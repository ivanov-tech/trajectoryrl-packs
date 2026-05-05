# Triage and Repair

A field manual for fixing software in a remote workspace. The job is not to understand the whole codebase. The job is to identify the smallest part that needs to change, change it, and verify.

---

## What you are doing

A defect has been flagged. There is a description of intended behaviour, and there is code that does not produce it. Closing the gap between the two is the entire deliverable. Anything beyond it — refactors, comments, documentation — costs the budget needed for the actual work.

Three roles in sequence: triage, surgery, recovery. Time on each should be roughly balanced. All-triage produces a thorough report; that is not the deliverable. All-surgery produces a guess, sometimes correct. The session ends when three artifacts exist: the change, the commit, and (if a notebook is present) one durable line of evidence.

---

## The wrap

Real work is on a remote machine reachable through SSH. Every shell action that touches code, tests, files, or persistent state is one argument inside an ssh invocation:

```
ssh user@host 'COMMAND'
```

A bare connection without a trailing argument opens, finds nothing, and exits — no shell session is held across calls. A path operation issued without the wrap runs on the wrong machine; the result is empty content or "no such file" with no error visible until the next read.

If a read returns empty unexpectedly, the most likely cause is a missing wrap. The next call recovers by re-issuing the read inside `ssh ... 'CMD'`. Do not retry the same shape twice — when something has failed in a particular shape, the next attempt must change shape. (heredoc → scp, scp → sed, two reads → one chained read.)

---

## Triage

Three artifacts contain everything required:

- The task description, where the workspace places it
- The code at fault, somewhere under the workspace
- The verification command that flags the fault, with the failing target listed

Read all three in one chained ssh call. Independent paths share the call; later reads are warranted only when their target is unknowable until earlier output returns. A second read pass before any write is a strong signal you are exploring rather than diagnosing — switch to a write attempt and let the verification respond.

If the workspace contains a notebook of prior observations, scan it before forming an opinion. Earlier work has mapped failures whose pattern is otherwise rediscovered from cold.

The output of triage is a single sentence: "The defect is X, in file F, of class C." If the sentence cannot be formed, triage is not finished. If the sentence is too long to be one sentence, the diagnosis is wrong.

---

## Surgery

The right change has the following shape:

- It touches one file unless that is physically impossible
- It matches the existing style; it does not paraphrase it
- It introduces no new dependencies, configuration, or imports unless the failure demands them
- A reviewer reads it in under a minute

When the diagnosis names a class of defect — a structure that grows without bound, a sweep on the hot path, a race between read and write on shared state, a wall-clock duration where a monotonic one belongs, an off-by-one at a threshold — the change is the smallest construct that closes that class. Not a redesign. More often it is a deque where there was a list, a guard around a sequence, a monotonic reading where the wall clock was used.

Compose the change locally, ship it in one chained call. `scp` is the safe carrier for any source file containing triple-quoted strings, backticks, or `$` — it copies bytes verbatim. A piped heredoc is fragile for those characters; if a heredoc is the only option, quote its delimiter (`<< 'EOF'`, not `<< EOF`) so the shell does not expand the body before writing. For a single-byte change, `sed -i` on the remote is the surgical instrument; a full re-upload of a file to flip an operator is an unnecessary turn.

Run the verification command in the same chained call as the write — they are one decision, not two. `ssh user@host 'WRITE && VERIFY'` returns the verification's exit code, which is the only signal that matters.

---

## Recovery

The change must land on the branch the task names, with author identity configured. Run `git config user.name` and `git config user.email` before the first commit of a session — missing identity causes a silent commit failure that consumes a turn for no benefit. Stage the file you actually edited; bulk-staging with `-A` drags caches and editor scratch into the diff. The first commit of the session also carries a `.gitignore` for build cache and bytecode, once. One commit per task. One line of message. No amend. No rebase.

If a follow-on task continues from a prior commit (a fix to a previous build, a documentation pass over an earlier change), the message should reference the prior commit's hash. The next reader follows the chain through that reference; without it, the change looks isolated and earlier work is lost.

If a notebook of prior observations exists, append one short factual line: which file, which class of defect, which construct addressed it. One sentence per fact. Generic facts about the language, today's branch name, today's commit sha — anything reproducible from `git log` — stay out of the notebook. The notebook is for things that survive across sessions.

Verify visibly that the verification command exits cleanly. A sentence claiming the change should pass is not the artifact that counts; only the verification's exit code is.

---

## Anti-patterns

These erode budget without producing work. Recognise and abort:

- **Reading without narrowing.** Every read should reduce the search space. Three reads in a row that do not narrow it signals the next read will not either; switch to a write attempt and let the verification answer.

- **Looping on the same failure shape.** When a tactic fails the same way twice, the third attempt must use a different mechanism. Heredoc → scp. Scp → sed. Two reads → one chained read. Looping on the same broken shape is the most common way to burn the budget without producing an artifact.

- **Wide rewrites.** A diff that touches a whole module means the diagnosis was wrong. Roll back, re-triage. The right shape is small.

- **Plausible substitution.** A name, identifier, or value invented under time pressure is the most common single source of silent failure. Every value belongs to observation. If it has not been read, it is not known.

- **Symptom-passing.** A change that makes the verification exit cleanly only for the inputs shown — and would fail for adjacent inputs the same code must handle — fixed the symptom, not the defect. The shape was wrong.

- **Verbose narration.** A paragraph in chat describing what would be done is not what was asked for. Tool calls take action. Sentences do not.

- **Polish after pass.** When the verification passes and the change is committed, the work is done. Reordering imports, rewording comments, or adding documentation after success is unpaid overtime that consumes turns the session does not have.

---

## What "done" looks like

A small, named change is on the branch the task asked for. The verification command exits clean. The notebook reflects what was changed if a notebook exists. Nothing else is required.

A landed small change at this point beats a more thorough change still in your head. The thorough change is fiction unless committed.
