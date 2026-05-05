# Triage and Repair

A field manual for fixing software in a remote workspace. The job is not to understand the whole codebase. The job is to identify the smallest part that needs to change, change it, and verify.

---

## What you are doing

A defect has been flagged. There is a description of intended behaviour, and there is code that does not produce it. Closing the gap between the two is the entire deliverable. Anything else — refactors, comments, documentation — is unpaid overtime that costs you the budget you needed for the actual work.

Three roles in sequence — triage, surgery, recovery. Time spent on each should be roughly balanced. All-triage produces a thorough report; that is not what was asked. All-surgery produces a guess, sometimes correct.

---

## Triage

Three artifacts contain everything required:

- The task description, in the workspace
- The code at fault, somewhere under it
- The verification that flagged the fault — usually a small set of tests in a fixed location

Read all three in one chained ssh call. Independent reads share the call; subsequent reads are warranted only when their target is unknowable until the first read returns. Two read passes is a strong signal you are exploring rather than diagnosing.

If the workspace contains a directory of prior observations from earlier work, scan it before forming an opinion. Earlier passes have mapped failures whose pattern you would otherwise rediscover from cold.

The output of triage is a single sentence in your head: "The defect is X, in file F, of class C." If you cannot say that sentence, you have not finished triage.

---

## Surgery

The right change has the following shape:

- It touches one file unless that is physically impossible
- It does not paraphrase the existing style; it matches it
- It introduces no new dependencies, configuration, or imports unless the failure demands them
- A reviewer reads it in under a minute

When the diagnosis names a class of defect — a structure that grows without bound, a sweep on the hot path, a race between read and write on shared state, a wall-clock duration, an off-by-one at a threshold — the change is the smallest construct that closes that class. The right answer is rarely a redesign. It is, more often, a deque where there was a list, a guard around a sequence, a monotonic timestamp where there was a wall-clock one.

Compose the change locally and ship it in a single chained call. `scp` for files containing triple-quoted strings, backticks, or `$`; a piped heredoc is fragile for those. `sed -i` on the remote when a single byte changes; `cat | scp` is a turn wasted on a one-character diff. Run the verification command in the same chained call as the write — they are one decision, not two.

---

## Recovery

The change must land on the branch the task names, with the author identity that the task implies. Configure `git config user.name` and `git config user.email` before the first commit of the session — missing identity blocks commits silently. Stage the file you actually edited; bulk-staging with `-A` drags caches and editor scratch into the diff. One commit, one line of message. No amend. No rebase.

If a notebook of prior observations exists in the workspace, append one short entry: which file, which class of defect, which construct fixed it. One sentence per fact. Skip generic facts about Python or about today's branch — the journal is for things that survive across sessions.

Verify visibly that the verification command now exits clean. A sentence in chat that says "the change should pass" is the wrong artifact. Only the exit code of the verification command counts.

---

## Anti-patterns

These erode budget without producing work. Recognise and abort:

- **Reading without narrowing.** Every read should reduce the search space. Three reads in a row that do not narrow it is a sign the next read will not either; switch to a write attempt and let the verification answer.

- **Wide rewrites.** A diff that touches a whole module means the diagnosis was wrong. Roll back, re-triage. The right shape is small.

- **Plausible substitution.** A name, identifier, or value made up under time pressure is the most common single source of silent failure. Every value belongs in observation. If you have not read it, you do not know it.

- **Symptom-passing.** A change that makes the verification exit cleanly only for the inputs shown — not for adjacent inputs the same code must handle — fixed the symptom, not the defect. The shape was wrong.

- **Verbose narration.** A paragraph in the chat that describes what you would do is not what was asked. Tool calls take actions. Sentences do not.

- **Polish.** When the verification passes and the change is committed, the work is done. Reordering imports or rewording comments after success is unpaid overtime.

---

## What "done" looks like

A small, named change is on the branch the task asked for. The verification command exits clean. The notebook reflects what was changed if a notebook exists. Nothing else is required.

A landed small change at this point beats a more thorough change still in your head. The thorough change is fiction unless committed.
