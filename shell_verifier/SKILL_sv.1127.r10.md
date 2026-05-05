# Shell-Task Working Protocol

A loop of four phases under a tight schedule. Each phase has a single artefact; the phase ends when its artefact exists on disk or in your plan, not when you finish narrating about it.

---

## Phases

### Phase 1 — Scan
Read the instruction file and list the working directory in one batched message. Then in a second batched message read every file the instruction names. By the end of two messages you should know the actual environment, not the one you imagined.

### Phase 2 — Contract
In one line, state the deliverable's absolute path and the observable pass signal. This is your finish line. You cannot aim for a finish line you have not written down.

Note the exact name, signature, schema, header strings, and constants the instruction mentions. The check command compares those character by character — paraphrasing is the same as wrong.

### Phase 3 — Build
Write the smallest implementation that should satisfy the contract to its named path on the first attempt. No comments stating what code obviously does. No helpers used only once. No defensive branches the spec does not promise. Get bytes on disk.

Then verify: if the instruction names a check command, run it. Otherwise look in `tests/` and run what is there.

### Phase 4 — Land or Loop
- **Land** — the verifier produced its pass signal. Done.
- **Loop** — the verifier failed. Read the actual output (the message, not the summary). Identify the root cause, not the symptom. Make one targeted change in Phase 3. Re-run. Same symptom returns until the root cause is fixed.

---

## Glossary

- **Verifier** — the command the instruction treats as authoritative for pass or fail.
- **Surface** — the filesystem and processes you can reach. If your listing disagrees with the brief, your surface is wrong before your code is wrong.
- **Burst** — a set of shell actions whose results do not depend on each other. Send them in one batched message together; serial single actions quietly burn the schedule.
- **Smoke** — the smallest manual check that exercises your last change before running the full verifier (file exists at the exact path with non-zero size; for structured outputs, header, first row, last row are sane).
- **Mechanism** — the exact behaviour of code already present in the workspace. Until you have read the source, the mechanism is unknown; do not assume it.

---

## Cross-cutting rules

- Action over analysis. A shell action advances the work; a paragraph about the action does not.
- Parallelise independent operations within a single message.
- Names, paths, and constants from the instruction are the contract. Use them character-for-character; do not retype from memory.
- Never invent paths, dates, signatures, or filenames. Confirm against the live environment first.
- The verifier — not your judgment — decides "done."

---

## When the deliverable interacts with another piece of code

If the instruction references a script, parser, validator, or service that the deliverable must work with, read that source first. Edge cases the other code handles are the place where a verifier most often catches a too-quick solution. Behaviour the other code does not handle is where the deliverable usually lives.

For tasks that require constructing input that survives a checker and is then re-read by a separate consumer, the two readers do not always interpret identical bytes the same way. Look for inputs one reader treats as a single unit while the other splits it; malformed structures and ambiguous delimiters are the usual divergence points.

---

## Anti-patterns

- **Reading without writing.** Each read should change what you do next. When successive reads stop changing the plan, the next move is to write.
- **Wrong path or name.** A correct artefact at the wrong path scores zero. Re-confirm path/name from the instruction before signalling done.
- **One-shot perfection.** Three small iterations against the verifier beat one large edit that fails several constraints at once.
- **Polish after pass.** Once the verifier passes, the work is finished. No reordering, no rephrasing, no "while I'm here."
