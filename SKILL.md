# Triage and Repair

A field manual for fixing software in a remote workspace. The job is not to understand the whole codebase. The job is to identify the smallest part that needs to change, change it, and verify.

---

## What you are doing

A defect has been flagged. There is a description of intended behaviour, and there is code that does not produce it. Closing the gap between the two is the entire deliverable. Refactors, comments, and documentation that are not required to close the gap are extra work for which there is no demand.

Three roles in sequence: triage, surgery, recovery. Time on each should be roughly balanced. Heavy triage produces a thorough report; that is not the deliverable. Skipping triage produces a guess, sometimes correct.

---

## The wrap

Real work lives on a remote machine reachable through SSH. Every shell action that touches code, tests, files, or persistent state goes inside an ssh invocation:

```
ssh user@host 'COMMAND'
```

A bare connection without a trailing argument opens, finds nothing, and exits — no shell session is held across calls. A path operation issued without the wrap runs on the wrong machine; the result is empty content or "no such file" with no error visible until the next read.

If a read returns empty unexpectedly, the most likely cause is a missing wrap. The next call recovers by re-issuing the read inside `ssh ... 'CMD'`. When something has failed the same way once already, the next attempt should change shape — different mechanism, different chaining, different scope. Repeating the same failing pattern produces the same failure.

---

## Triage

Three artifacts contain everything required:

- The task description, where the workspace places it
- The code at fault, somewhere under the workspace
- The verification command that flags the fault, with the failing target listed

Read all three in one chained ssh call. Independent paths share the call; later reads are warranted only when their target is unknowable until earlier output returns. Reads that do not narrow the search reveal exploration rather than diagnosis — the next move is a write attempt that the verification can respond to.

If the workspace contains a notebook of prior observations, scan it before forming an opinion. Earlier work has mapped failures whose pattern is otherwise rediscovered from cold.

When a file the task points to already contains real code (it does not raise `NotImplementedError`, the verification's earlier targets for it pass), the work is an extension — a new use case, a new constructor parameter, a parallel class — not a rewrite. Read the existing implementation as part of triage. The change is shaped to call into or wrap it, not to recreate it.

The verification's failing target is one signal; the verification's full set is another. New targets usually do not displace prior ones — both run on every pass. A change that satisfies a new target while breaking an earlier one has fixed nothing.

The output of triage is a single sentence: "The defect is X, in file F, of class C." If the sentence cannot be formed, triage is not finished. If the sentence is too long to be one sentence, the diagnosis is wrong.

---

## Surgery

The right change has the following shape:

- It touches one file unless that is physically impossible
- It matches the existing style; it does not paraphrase it
- It introduces no new dependencies, configuration, or imports unless the failure demands them
- A reviewer reads it in under a minute

When the workspace already contains code that solves a related problem and the task asks to apply it to a new use case, the change should add — a small wrapper, a new call site, a subclass with the new parameters — never duplicate or paraphrase the original. Re-implementing existing logic doubles the surface and breaks the work that was already passing. The original keeps its responsibilities; the new construct is the thin layer above it.

When the diagnosis names a class of defect — a structure that grows without bound, a sweep on the hot path, a race between read and write on shared state, a wall-clock duration where a monotonic one belongs, an off-by-one at a threshold — the change is the smallest construct that closes that class. Not a redesign. More often it is a deque where there was a list, a guard around a sequence, a monotonic reading where the wall clock was used.

Compose the change locally, ship it in one chained call. `scp` is the safe carrier for any source file containing triple-quoted strings, backticks, or `$` — it copies bytes faithfully. A piped heredoc is fragile for those characters; if a heredoc is the only option, quote its delimiter (`<< 'EOF'`, not `<< EOF`) so the shell does not expand the body before writing. For a single-byte change, `sed -i` on the remote is the surgical instrument; a full re-upload of a file to flip an operator is unnecessary work.

Run the verification command in the same chained call as the write — they are one decision, not two. `ssh user@host 'WRITE && VERIFY'` returns the verification's exit code, which is the only signal that matters.

---

## Recovery

The change must land on the branch the task names, with author identity configured. Run `git config user.name` and `git config user.email` before the first commit of a session — missing identity causes a silent commit failure. Stage the file you actually edited; bulk-staging with `-A` drags caches and editor scratch into the diff. The first commit of the session also carries a `.gitignore` for build cache and bytecode, once. One commit per task. One line of message. No amend. No rebase.

If a follow-on task continues from a prior commit (a fix to a previous build, a documentation pass over an earlier change), the message should reference the prior commit's hash. The next reader follows the chain through that reference; without it, the change looks isolated and earlier work is lost.

Verify visibly that the verification command exits cleanly. A sentence claiming the change should pass is not the artifact that counts; only the verification's exit code is.

---

## Durable notes

If `/workspace/learned/` exists, treat it as a notebook used across runs:

- `repo.md` — repository facts: paths, the verification command, branch convention. One line each.
- `bugs.md` — defect classes encountered, with the construct that addressed each. Verified entries only.
- `tooling.md` — environment quirks and the workaround that worked.

Write the moment a surprise lands; deferring loses it. Update in place; never duplicate. When live observation contradicts a stored line, the stored line is replaced. Skip generic facts about the language, today's branch name, today's commit sha — anything `git log` could re-derive. The notebook is for things that survive across sessions.

When a defect on a known construct repeats, the entry that records it is the answer; copy its construct into the plan rather than rediscovering. When a documentation task asks about prior work, the notebook is the source — sourcing identifiers from the notebook avoids inventing them.

---

## Anti-patterns

These are recognisable patterns of work that produce nothing landed. Recognise and step out of:

- **Reading without narrowing.** Each read should reveal new shape, not the same surface seen from another angle. When successive reads stop narrowing the search, the next useful move is a write attempt the verification can respond to.

- **Looping on the same failure shape.** When a tactic produces the same shell error twice, the third attempt should switch mechanism. Heredoc to scp. Scp to sed. Two ssh calls to one chained ssh call. Repeating the same failing pattern gives the same failure.

- **Wide rewrites.** A diff that touches a whole module signals the diagnosis was wrong. Roll back, re-triage. The right shape is small.

- **Reimplementing existing code.** When the task asks to apply something the workspace already has, paraphrasing the original instead of calling into it doubles the surface and breaks earlier passes. Read first; wrap second.

- **Plausible substitution.** A name, identifier, or value invented under time pressure is the most common single source of silent failure. Every value belongs to observation. If it has not been read, it is not known.

- **Symptom-passing.** A change that makes the verification exit cleanly only for the inputs shown — and would fail for adjacent inputs the same code must handle — fixed the symptom, not the defect. The shape was wrong.

- **Polish after pass.** When the verification passes and the change is committed, the work is done. Reordering imports, rewording comments, or adding documentation after success is extra work that was not asked for.

---

## What "done" looks like

A small, named change is on the branch the task asked for. The verification command exits clean. The notebook reflects what was changed if a notebook exists. Nothing else is required.

A landed small change beats a more thorough change still in your head. The thorough change is fiction unless committed.
