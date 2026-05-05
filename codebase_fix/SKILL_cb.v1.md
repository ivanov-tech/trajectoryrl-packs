# Engineer's Protocol

A discipline for landing a coded change in a remote workspace under a 15-call budget. What you ship is the only thing that scores.

---

## The shape of a session

You have 15 tool calls. Spend them on action, not narration. A clean session is four to six chained calls: one read pass, one plan, one write+verify, one commit. The rest is buffer for one repair pass.

Long internal monologues do not score. Final commits and tests passing do. Treat each tool call as a budget line item.

---

## The mandatory bookkeeping

Before any work, run a single chained call that does both:

```
ssh sandbox 'cat /workspace/INSTRUCTION.md /workspace/SKILL.md /workspace/ENVIRONMENT.md 2>/dev/null; ls /workspace/learned/ 2>/dev/null; cat /workspace/learned/*.md 2>/dev/null'
```

The contents of `/workspace/learned/` are notes from prior episodes — they are evidence about what failed last time and what fixes the project. Read them before forming an opinion.

Before exit, append a new entry. Use this exact pattern, no improvisation:

`/workspace/learned/learnings.md`:
```
## [<episode-id>] <one-line topic>
- shape: <what kind of change — fix, replicate, build, document>
- target: <file path that was changed, exact>
- bug class: <invariant violated, named generally>
- pattern that fixed it: <data structure or algorithm choice, generally>
- traps avoided: <one line per>
```

`/workspace/learned/repo.md`: append one-line facts that survive across episodes (test command, branch convention, persistent layout). One fact per line, deduplicate before writing.

Each line in those files is a constraint inherited by the next episode. If a line contradicts a later observation, replace the line in place — never duplicate.

Skip the bookkeeping and the next episode starts from zero again. The split-half score multiplier rewards agents that genuinely improve; it punishes amnesiacs.

---

## The four-call rhythm

1. **Read pass.** One ssh call, every path the ticket implies, batched. Independent reads share the call. Subsequent reads only when their target depends on earlier output.

2. **Plan.** No tool call. Decide the smallest correct change in your head. Name the bug class. Pick the data structure. Choose the test command.

3. **Write+verify.** One write pass that composes the change locally then ships it via `scp` or a single `ssh sandbox 'cat > /path/file'` heredoc, then runs the visible test command in the same chained call. If the diff is small, prefer `sed -i` on the remote.

4. **Commit.** Branch with the exact name the ticket gives. `git add` the explicit file path. One-line commit message. No amend, no rebase.

A fifth call is allowed only for `git status` verification or one repair when the test pass is incomplete. Never cycle past six. If you are still reading at call seven, you are losing.

---

## Properties that production code must have

Visible tests catch the obvious; the invariants below are what hidden tests probe. Apply every one of them in the first commit unless the ticket explicitly forbids it. Naming a defect class without addressing it is the same as not naming it.

- **Bounded growth.** Anything that retains state — caches, per-key dicts, queues, logs — must shed entries when their work ends. A collection that grows with the number of distinct callers, not with the number of currently active callers, is a memory leak.

- **Bounded per-call cost.** Per-call work must not scale with the total accumulated state. Rebuilding a list on every call works for ten entries and fails at twenty thousand. Choose ring buffers, deques, or windowed indices; never sweep an unbounded list on the hot path.

- **Eviction of stale state.** Anything time-windowed must be evictable. The contract "remove entries older than window W" is satisfied by a structure that drops them in O(1) per call (e.g. a deque popped from the front), not by a per-call comprehension over the entire collection.

- **Concurrent access.** Any read-decide-write sequence on shared state is a race unless the entire sequence is held under one guard. Splitting the guard around the steps does not fix it.

- **Time arithmetic.** Durations are measured from a monotonic source, not the wall clock. Wall-clock arithmetic breaks the moment NTP corrects the host.

- **Boundary semantics.** Inclusive vs exclusive at thresholds is a choice, not a default. Make the choice deliberately and write it next to the comparison.

These are not specific to any task. They are the shape of any production change.

---

## Edit tactics

- **Multi-line writes**: compose locally, then `scp local_file sandbox:/workspace/path`, or pipe via `ssh sandbox 'cat > /workspace/path' << EOF`. Heredocs inside `ssh '...'` corrupt triple-quotes, backticks, and `$`; prefer `scp` for any file with those.

- **Single-byte edits**: `ssh sandbox 'sed -i "s/OLD/NEW/" /path/file'`. One call per byte change. Don't re-upload a whole file to flip an operator.

- **Reading a wide file**: `ssh sandbox 'sed -n "10,80p" /path/file'`. Don't `cat` a 500-line file when you need 70 lines.

- **Verifying changes**: chain the test command in the same ssh call as the write — `ssh sandbox 'CMD1 && CMD2'`. Don't burn a turn on a separate verification.

---

## Branching and identity

Configure `git config user.name "agent"` and `git config user.email "agent@local"` once before the first commit. Missing identity blocks commits silently.

Switch onto the branch the ticket names exactly, before any edit. If a previous task branch exists, branch from the most recent rather than the default — defaulting reverts work that already shipped.

Stage by exact path: `git add path/to/file.py`. Never `git add -A`. Never amend. Never rebase. One commit per ticket, one line message.

---

## Common task shapes

The ticket dictates the deliverable, the location, and the branch. Use the names verbatim — paraphrase is violation. Most tickets fall into one of:

- **Build**: implement the named module to satisfy the visible contract, then commit. Apply every property from the list above unless the contract narrows them.
- **Replicate**: re-use an existing construct with new parameters. Prefer a thin wrapper or call site over inheritance. If retained notes show a prior unaddressed bug class on the same construct, fold the fix into this commit.
- **Fix**: the ticket names a defect class. The retained notes name the pattern that fixes it. Make the surgical change; commit on the named branch.
- **Document**: no code change. Write the document the ticket asks for, at the named path, citing failing identifiers verbatim from the retained notes — never fabricate.

---

## Exit

Visible tests pass + commit landed = exit. A long polish pass is not the deliverable. A small landed change beats an elegant draft that did not get committed in time.

Before exit, append the new entry to `/workspace/learned/learnings.md`. The entry is one short block in the format above; it is part of the deliverable, not optional polish.

If you are at call 12 and still reading, stop reading. Write something testable and commit. A 70% solution committed scores higher than a 95% solution that hit the iteration limit.
