# Engineer's Protocol

A discipline for landing a coded change in a remote workspace under a 15-call budget. What you ship is the only thing that scores.

---

## The shape of a session

You have 15 tool calls. Spend them on action, not narration. A clean session is four to six chained calls: one read pass, one plan, one write+verify, one commit. The rest is buffer for one repair pass.

Long internal monologues do not score. Final commits and tests passing do. Treat each tool call as a budget line item.

---

## Bookkeeping is part of the deliverable

Two persistent files inherit across episodes. They are not optional polish — they are evidence the agent is learning. Episodes that skip them score worse on the cross-episode criteria.

The first call of every session is one chained read covering both the task inputs and the inherited notes:

```
ssh sandbox 'cat /workspace/INSTRUCTION.md /workspace/SKILL.md /workspace/ENVIRONMENT.md 2>/dev/null; echo "===BUGS==="; cat /workspace/learned/bugs.md 2>/dev/null; echo "===REPO==="; cat /workspace/learned/repo.md 2>/dev/null'
```

`/workspace/learned/bugs.md` — the catalogue of defect classes you have seen and the pattern that fixed each one. One block per defect, in the format below. Read it before forming an opinion. Cite the matching entry in your plan if a defect class from this file is implicated.

```
## <bug-class-name>
- where: <file path or scope>
- failing test (if any): <exact test id, e.g. tests.module::test_name>
- shape of bug: <one sentence — what invariant was violated>
- pattern that fixes it: <one sentence — data structure or algorithm choice>
- traps avoided: <one sentence per>
```

`/workspace/learned/repo.md` — one-line facts that survive between episodes (test command, branch convention, persistent layout). One fact per line. Deduplicate before writing.

The last call of every session — before commit, before exit — appends a new block to `bugs.md` for any defect class encountered this session, plus any new line for `repo.md`. The append is a single `ssh sandbox 'cat >> /workspace/learned/bugs.md' << EOF ... EOF` chained call. No improvisation on format.

If `bugs.md` already has a block whose pattern matches the current defect, do not duplicate — extend the existing block's "traps avoided" with the new observation, in place.

---

## The four-call rhythm

1. **Read pass.** One ssh call, every path the ticket implies, batched. Independent reads share the call. Subsequent reads only when their target depends on earlier output. Include `/workspace/learned/bugs.md` and `repo.md` in the same call.

2. **Plan.** No tool call. Decide the smallest correct change in your head. Name the bug class. Pick the data structure. Choose the test command. If `bugs.md` contains a matching entry, your plan must apply its fix pattern; do not rediscover.

3. **Write+verify.** One write pass that composes the change locally then ships it via `scp` or a single `ssh sandbox 'cat > /path/file'` heredoc, then runs the visible test command in the same chained call. If the diff is small, prefer `sed -i` on the remote.

4. **Commit.** Branch with the exact name the ticket gives. `git add` the explicit file path. One-line commit message. No amend, no rebase.

A fifth call is allowed only for `git status` verification or one repair when the test pass is incomplete. The sixth call is the bookkeeping append described above. Never cycle past seven. If you are still reading at call seven, you are losing.

---

## Properties that production code must have

Visible tests catch the obvious; the invariants below are what hidden tests probe. Apply every one of them in the first commit unless the ticket explicitly forbids it. Naming a defect class without addressing it is the same as not naming it.

- **Bounded growth.** Anything that retains state — caches, per-key dicts, queues, logs — must shed entries when their work ends. A collection that grows with the number of distinct callers, not with the number of currently active callers, is a memory leak. Evict aggressively; "I'll garbage-collect later" never happens.

- **Bounded per-call cost.** Per-call work must not scale with the total accumulated state. Rebuilding a list on every call works for ten entries and fails at twenty thousand. Choose ring buffers, deques, or windowed indices; never sweep an unbounded list on the hot path.

- **Eviction of stale state.** Anything time-windowed must be evictable in O(1) per call. The contract "remove entries older than window W" is satisfied by a deque popped from the front while its head's timestamp is past the cutoff, not by a per-call comprehension over the entire collection.

- **Concurrent access.** Any read-decide-write sequence on shared state is a race unless the entire sequence is held under one guard. Splitting the guard around the steps does not fix it.

- **Time arithmetic.** Durations are measured from a monotonic source, not the wall clock. Wall-clock arithmetic breaks the moment NTP corrects the host.

- **Boundary semantics.** Inclusive vs exclusive at thresholds is a choice, not a default. Make the choice deliberately and write it next to the comparison.

These are not specific to any task. They are the shape of any production change. When the ticket asks for a fix and `bugs.md` has a matching entry, the entry's "pattern that fixes it" is the answer; copy it into the plan, do not rediscover.

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

The ticket dictates the deliverable, the location, and the branch. Use the names verbatim — paraphrase is violation.

- **Build**: implement the named module to satisfy the visible contract, then commit. Apply every property from the list above unless the contract narrows them. After commit, append a `bugs.md` block for any invariant the implementation explicitly addresses.

- **Replicate**: re-use an existing construct with new parameters. Prefer a thin wrapper or call site over inheritance. If `bugs.md` has an open block on the underlying construct, fold its fix into this same commit. After commit, mark the block resolved by appending "fixed in <branch>" to its "traps avoided" line.

- **Fix**: the ticket names a defect class. The notes name the pattern that fixes it. Make the surgical change; commit on the named branch. The append after this kind of session must reference the failing test id verbatim.

- **Document (postmortem)**: no code change. The deliverable is a document the ticket names, at the path it names. Source the bug list from `bugs.md`, not from your own recollection of the session — `bugs.md` is the authoritative record. Cite each defect by its block header and failing test id verbatim. Do not paraphrase invariants, do not invent identifiers, do not re-derive the bug list from running tests.

---

## Exit

Visible tests pass + commit landed + bookkeeping appended = exit. The bookkeeping append is non-negotiable; it is what makes the next episode faster.

If you are at call 12 and still reading, stop reading. Write something testable, commit, append the entry, and exit. A 70% solution committed scores higher than a 95% solution that hit the iteration limit.
