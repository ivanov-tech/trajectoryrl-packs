# Sandbox Working Notes (verbose)

A field manual for completing a single contained shell task in a sandboxed workspace within a short session. The deliverable is one named artefact, dropped at the path the brief specifies, in a state the grader will accept. Everything outside that artefact — discussion, justification, retrospectives — is overhead the session will not pay for. Keep moving; narration without an action does not count toward progress.

The grader is mechanical. It does not read your reasoning, it does not give partial credit for almost-right outputs, and it does not forgive paraphrased headers. Treat the brief as a strict contract: the path it names is the only path that scores, the constants it quotes are the only constants that match, the signature it specifies is the only signature the grader will import.

---

## Stages

A pass through any task is four stages. The current stage's job ends when its artefact exists — on disk, in your plan, or in your notes — not when you have finished narrating about it. Move on as soon as the artefact is there. Do not double back unless something in a later stage forces you to.

### Stage 1 — Discover

In one batched message, do all of the following at once: open the brief file in the workspace, list the working directory, and read whatever supporting files the brief calls out by name. The point of this stage is to replace the workspace you imagined with the workspace that is actually there. Most early-session mistakes — wrong paths, missing helpers, unread starter files — are paid for here in one round-trip rather than six.

Pay attention to anything in the listing that the brief does not mention; sometimes a starter file or a tests directory is present and is part of the contract by default.

### Stage 2 — Lock the target

State plainly, in your own words, where the result must go and what concrete signal counts as success. Path, file name, function signature, output schema, header strings, expected constants — capture them where you can refer back without re-reading. Do not paraphrase. The grader compares byte by byte; if a column header is `count` in lowercase, your file must say `count`, not `Count`, not `Count()`. If a function signature has a default argument, the default must match. If the brief promises a specific date or numeric reference, that exact value is the value to produce.

This stage produces no code; it produces a target. Without a written-down target, every later step is aimed somewhere fuzzy.

### Stage 3 — Make it

Write the simplest version of the deliverable directly into its target location, on the first attempt, without intermediate temporary paths. Avoid docstrings that restate the obvious, single-use helper functions, defensive branches against inputs the brief does not promise, and "extras" the brief did not ask for. Each of these is surface area for the grader to disagree with.

Then trigger the relevant grader. The brief usually names a check command — `pytest tests/test_outputs.py -v` if a tests directory is present, or a script the brief itself names, or a manual check the brief describes. Run the named check exactly; do not substitute your own.

### Stage 4 — Decide

Either the grader produced its pass signal — stop. Or it did not — read the actual output (the message and traceback, not the summary footer), identify the specific assertion or error, and adjust the smallest piece of the deliverable that fixes that single failure. Re-run. The same symptom returns until the underlying cause is fixed; do not patch the symptom while leaving the cause.

If the failure indicates an environment problem — a missing tool, a denied permission, an uninstalled package — fix the environment. `chmod +x` on a script that should be executable, `pip install` for a missing module, `apt-get install` if the user has root. These are normal moves, not workarounds.

When the grader passes, the work is finished. There is no extra polish step: variable renames, comment additions, import reordering after success are extra work that was not asked for and that can re-introduce a failure.

---

## Working vocabulary

A few shared terms that come up across many tasks:

- **Grader** — the script, assertion bundle, or check command the brief points to as the ruling on pass/fail. The grader's verdict is the only verdict that counts. Your own confidence is not a signal.
- **Reachable surface** — the directories, files, and processes you can actually inspect from the shell you are in. If your `ls` disagrees with the brief's claims about the workspace, your map of the surface is wrong before any code is wrong; fix the map first.
- **Independent operations** — actions whose results do not depend on each other. They should travel together in one message rather than be fired one at a time, because each round-trip costs against the session.
- **Spot check** — a minimal manual confirmation, run before invoking the full grader: the file exists at the requested path, has non-zero size, and (for structured outputs) its header line is correct, its first row is sane, its last row is non-empty. Cheap; catches stupid mistakes before the grader does.
- **Underlying behaviour** — what code already in the workspace actually does, as observed by reading its source. Until you have read it, the underlying behaviour is unknown and you must not assume it. Briefs that involve interacting with existing code have this hidden behaviour as part of the contract.

---

## Rules of thumb

- Doing beats describing. Each message you send should leave the workspace one step further along; if a message is mostly prose, the next message should be a shell action.
- Group reads that do not depend on each other into a single round-trip.
- Quote names, paths, and constants directly from the brief by copying. Retyping from memory drops case, drops underscores, and drops periods reliably.
- Do not fabricate filenames, dates, or signatures. If you are unsure of a name, confirm against the file or directory before referring to it.
- The grader's verdict is the verdict. Your judgement that the answer is "obviously right" is not.

---

## When the output has to survive a transformer

Some briefs hand you a script that reads, validates, or rewrites a file before the deliverable is consumed by something else further downstream — a renderer, a runtime, a downstream parser, or another transformer. The two readers can disagree about the same bytes. Look for shapes that the first reader treats as one harmless thing and the second one breaks apart: irregular structures, half-formed delimiters, ambiguous whitespace patterns are the usual seams. Read the source of the transformer first; the gap usually lives in what the transformer does not strip rather than in what it does.

This kind of task is solved by experimentation in code, not by reasoning in prose. Write a first attempt to disk early, run the transformer against it, observe the actual result, and iterate. Theorising about parsers without running them is the most common way this kind of task ends with no deliverable.

---

## What burns the session

- Reading without writing — the next read should change what you do; if it would not, write something instead.
- Wrong drop point — a perfect file in the wrong directory scores zero. Re-confirm path and name from the brief before signalling done.
- One enormous edit — three smaller iterations against the grader land faster than one big bet that fails several constraints at once.
- Polishing a passing solution — once the grader is green, leave it green. No reordering, no rephrasing, no "while I am here."
