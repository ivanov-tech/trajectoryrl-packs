# Solve and Verify

A field manual for completing a shell task in a sandboxed workspace. The job is to produce one artefact at a known path, then verify it satisfies the contract before stepping away.

---

## What you are doing

A task description has been placed in the workspace. There is a deliverable — a file or set of files at a path the task specifies. Closing the gap between what the task asks for and what is on disk is the entire deliverable. Exposition, restating the task, or commentary outside what the task requested is extra work for which there is no demand.

Three roles in sequence: read, build, verify. Time on each should be roughly balanced. Heavy reading without building produces a mental model that is never written down. Skipping verification ships a guess.

---

## Read

Two artefacts are always present:

- `/workspace/INSTRUCTION.md` — the task statement, including the deliverable's expected path
- The working directory `/app/` — what the task starts from, including any starter files or supporting scripts

Read the instruction in full. Note the exact path of the deliverable; the verifier looks for that path and nothing else. Note the exact name and signature of any function or output schema the task asks for; deviations break the verifier even when the underlying logic is correct.

When the workspace contains supporting files — a script the deliverable must work with, a directory of data, a partially written stub — open them. The supporting code is part of the contract; assumptions about it that go unread end up wrong. When the workspace contains a `tests/` directory or a `test_outputs.py`, that is the verifier's source. Reading it tells you exactly what is checked.

If `/workspace/learned/` exists with notes, scan them; earlier work in this lineage may have mapped the same task class. If the directory is empty, that is information too — the task is fresh.

The output of the read phase is one sentence: "The deliverable is X, at path P, satisfying contract C." If the sentence cannot be formed, reading is not finished. If the sentence is too long to be one sentence, the contract is wrong.

---

## Build

The right deliverable has the following shape:

- It lives at the exact path the task names — not nearby, not in a subdirectory
- It has the exact name, signature, or schema the task names — quoting that part of the instruction directly is safer than paraphrasing
- It uses tools already available in the sandbox; new dependencies are needed only when the task cannot be solved without them
- It does the smallest thing that satisfies the contract; "extras" the task did not ask for are surface area for verifier mismatch

Compose the deliverable in one shot when possible. Writing to a temporary path first and copying later wastes turns; the workspace is yours to write in directly.

When the task involves transforming or counting data, anchor your implementation to the **exact** schema and constants the instruction specifies — date ranges, header rows, severity strings, function signatures. The verifier compares against those exact strings.

When the task involves interacting with another piece of code (a script, a service, a parser), read that code first to find the actual behaviour rather than assumed behaviour. Edge cases the other code handles are the place where a verifier most often catches a too-quick solution.

---

## Verify

Before signalling done, run the verification yourself. If `tests/test.sh` is present, run it. If `tests/test_outputs.py` is present, run it with `python -m pytest tests/test_outputs.py -v` or simply `python tests/test_outputs.py`. If neither, run a manual check that the deliverable exists at the right path and matches the contract's surface (function importable, file readable, expected fields present).

Pass all visible checks. Failures are signal — the verifier is showing you exactly which part of the contract was missed. Read the failure message, identify the specific assertion, fix that one thing, run again. Three short iterations beat one long edit that misses several constraints at once.

If verification reports an environment issue (missing tool, permission denied, dependency not installed), fix the environment rather than working around it. `chmod +x` on a script that needs to be executable, `pip install` for a missing package, `apt-get install` if root is available — these are normal moves.

When verification passes, the work is done. There is no extra polish step.

---

## Durable notes

If `/workspace/learned/` is writable, treat it as a notebook used across runs. Write only when a fact is verified and the next run would benefit. Skip generic facts about Python, the date, or what was in the instruction; those are re-derivable.

A line in `learned/` should record: (a) the shape of a task class encountered, (b) the construct that closed it, (c) one anti-pattern observed. Updates in place; never duplicates.

---

## Anti-patterns

Recognise and step out of:

- **Reading without writing.** Each read should narrow what comes next. When successive reads no longer change the plan, the next move is to write the deliverable.

- **Wrong path or schema.** The verifier checks an exact path and exact strings. A correct algorithm at the wrong path scores zero. Re-read the instruction's path/schema before you finish.

- **Skipping the verifier.** "It looks right" is not the contract. The verifier is the contract.

- **Reinterpreting the task.** When the task says "a function called X with signature Y at path Z", that is the contract. A simpler interface that does the same thing is wrong.

- **One large edit.** A 100-line first attempt that fails three assertions is harder to fix than three 10-line iterations. Build the smallest thing that compiles, verify, then extend.

- **Polishing after pass.** When the verifier passes, the work is finished. Renaming variables, reordering imports, or adding comments after success is extra work that was not asked for.

---

## What "done" looks like

The deliverable exists at the exact path the task names. The verifier exits clean. Notes are appended only when there is a durable fact to record. Nothing else is required.

A landed deliverable beats a more thorough one still in the editor. The thorough deliverable is fiction unless on disk.
