# Sandbox Working Notes (short)

A small set of habits for getting one named output produced and validated within a short session.

## Stages

1. **Discover** — read the brief and list the workspace; open files the brief names. Replace your guesses with what is there.
2. **Lock the target** — note where the result must go and what counts as success: path, name, signature, output schema. Headers are case-sensitive.
3. **Make it** — write the simplest version straight to the target path. Skip docstrings, single-use helpers, defensive guards. Run the relevant test or check.
4. **Decide** — pass: stop. Fail: read the specific assertion, fix one thing, re-check.

## Vocabulary

- **Grader** — the script the brief points to as authoritative.
- **Reachable surface** — what `ls` actually shows; if it disagrees with the brief, your map is wrong.
- **Independent operations** — actions that do not need each other; bundle them into one message.
- **Spot check** — a tiny manual confirmation before invoking the grader.

## Rules

- Doing beats describing. Each message advances the state.
- Group reads that do not depend on each other.
- Copy names and constants from the brief; do not retype.
- Do not fabricate paths or signatures from memory.
- The grader's verdict is the verdict.

## When the output passes through a transformer

If a script reads/validates/rewrites your file before the real consumer sees it, the two readers can disagree about the same bytes. Inputs the first treats as one harmless thing and the second splits — irregular structures, half-formed delimiters, ambiguous whitespace — are the seams. Read the transformer source first; the gap is in what it does not strip.

You cannot reason ahead of time about whether your bytes survive a transformer. The only reliable test is to run it on real input and look at what comes out the other side. The required loop:

1. Write any concrete payload to the target path — start with the obvious thing the brief implies is forbidden. This is your baseline observation.
2. Run the transformer on the file and read the actual output, byte for byte. Note what was stripped, what was rewritten, what survived.
3. Adjust your payload based on what the transformer left behind. Re-run the transformer. Compare again.
4. Stop adjusting only when the grader signals pass.

Skipping step 1 — theorising about parsers without ever seeing them work — is the most common way this kind of task ends with no deliverable. For any task that fits this shape, your second action should be writing a baseline payload to disk; your third should be running the transformer on it.

## What burns the session

- Reading without writing.
- Wrong drop point.
- One enormous edit instead of small iterations.
- Polishing a passing solution.
