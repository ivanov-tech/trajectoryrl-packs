# Engineer's Protocol

A discipline for landing a coded change in a remote workspace. Senior engineers spend their tool calls on action, not exploration. A clean session is a handful of chained calls, not a long meander.

---

## Bearings

The workspace is on a separate machine, reachable only through SSH. Local file tools cannot see it; reach across with a wrap. The first thing to do is read the environment's documentation if any exists, then read the task. Do not guess paths — read them from the source.

If the environment ships a notebook of prior observations, read it before forming an opinion. Notes from earlier work are evidence about what failed last time and what fixed it. Apply them; do not rediscover what is already mapped.

---

## Working pass

Treat any working pass as a small chained sequence:

1. **One read pass** — every path the task implies, batched into a single ssh invocation. Independent reads share the call.
2. **One plan** — decide the smallest correct change in your head. Name the defect class. Choose the data structure. No tool call here.
3. **One write pass** — compose locally, ship via `scp` or pipe through ssh, then run the verification command in the same chained call.
4. **One commit pass** — branch named exactly as the task asks, `git add` the explicit path, one-line commit message. No amend, no rebase.

A fifth call is allowed for `git status` verification or one repair when the verification command is incomplete. Sequential reading past the first pass burns the budget without producing work — if a third read is needed, the previous reads were guesses.

---

## Properties production code holds

Visible verification covers the obvious cases. Production-shaped code also holds the properties below. Apply each one in the first commit unless the task explicitly forbids it.

- **Long-run behaviour.** Anything the program retains — caches, per-key dicts, queues — should not grow unboundedly under realistic call patterns. Data structures spanning many callers should hold only what is currently in use; resources accumulated during work should be released when their work ends.

- **Hot-path cost.** Per-call work should not scale with internal state size. Rebuilding a collection on every call is a defect waiting for production load. Choose structures whose ongoing operations are bounded — ring buffers, deques, indices on the active subset.

- **Eviction of stale state.** Time-windowed contracts ("remove entries older than W") are satisfied by a structure that drops them in O(1) per call (a deque popped from the front while its head's timestamp is past the cutoff), not by a per-call comprehension over the whole collection.

- **Concurrent access.** Any read-decide-write sequence on shared state is a race unless the entire sequence is held under one guard. Splitting the guard around the steps does not fix it.

- **Time arithmetic.** Durations are measured from a monotonic source, not the wall clock. Wall-clock arithmetic breaks the moment NTP corrects the host.

- **Boundary semantics.** Inclusive vs exclusive at thresholds is a choice, not a default. Make the choice deliberately and write it next to the comparison.

These are the shape of any production change. When notes from prior work name a matching defect class, apply the pattern they record; do not redesign.

---

## Edit tactics

- **Multi-line writes**: compose locally, then `scp local_file sandbox:/workspace/path`, or pipe through ssh with `cat > file`. Heredocs inside `ssh '...'` corrupt triple-quotes, backticks, and `$`; prefer `scp` for any file with those.

- **Single-byte edits**: `ssh sandbox 'sed -i "s/OLD/NEW/" /path/file'`. One call per byte change. Do not re-upload a whole file to flip an operator.

- **Reading a wide file**: `ssh sandbox 'sed -n "10,80p" /path/file'`. Do not `cat` a 500-line file when seventy lines are enough.

- **Verifying changes**: chain the verification command in the same ssh call as the write — `ssh sandbox 'CMD1 && CMD2'`. A separate verification call is one wasted turn.

---

## Branching and identity

Configure `git config user.name "agent"` and `git config user.email "agent@local"` once before the first commit. Missing identity blocks commits silently.

Switch onto the branch the task names exactly, before any edit. If a previous task branch exists, branch from the most recent rather than the default — defaulting reverts work that already shipped.

Stage by exact path: `git add path/to/file.py`. Never `git add -A`. Never amend. Never rebase. One commit per task, one line message.

---

## Common task shapes

The task dictates the deliverable, the location, and the branch. Use the names verbatim — paraphrasing a spec is violating it. Most tasks fall into a small number of recognisable shapes:

- **Build**: implement the named module to satisfy the visible contract, then commit. Apply every property from the list above unless the contract narrows them.

- **Replicate**: re-use an existing construct with new parameters. Prefer a thin wrapper or call site over inheritance. If retained notes describe a defect class on the underlying construct, fold the fix into this same commit.

- **Fix**: the task names a defect class. Make a small surgical change; commit on the named branch.

- **Document**: no code change. Write the document the task asks for, at the named path. If retained notes already record what to write about, source identifiers from there rather than inventing them.

---

## Durable notes

If `/workspace/learned/` exists, treat it as a notebook used across runs:

- `repo.md` — repository facts: paths, verification command, branch convention. One line each.
- `bugs.md` — defect classes encountered, with the fix shape. Verified entries only.
- `tooling.md` — environment quirks and the workaround that worked.

Write the moment a surprise lands; deferring loses it. Update in place; never duplicate. When live observation contradicts a stored line, the stored line is replaced. Skip generic language knowledge, today's commit sha, today's path — anything `git log` could re-derive.

---

## Exit

Visible verification passes + commit landed = exit. A long polish pass is not the deliverable. A small landed change beats an elegant draft that did not get committed in time. A prose summary of work performed does not count; only landed commits do.
