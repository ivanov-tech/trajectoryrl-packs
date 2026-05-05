# Sandbox Working Notes

A small set of habits for getting one named output produced and validated within a short session. Keep moving; narration without an action does not count.

---

## Stages

A pass through any task is four stages. Move on as soon as the current stage's job is done.

### 1. Discover

In one batched message: open the brief, list the work directory, and read whatever files the brief calls out by name. The point is to replace your guesses about the workspace with what is actually there.

### 2. Lock the target

Identify, in plain language: where the result must go, and what concrete signal counts as success. Path, name, function signature, output schema — write them down (mentally or in a `learned/` note). The grader checks bytes; if a column header is `count`, the file must say `count` and not `Count`.

### 3. Make it

Write the simplest version of the deliverable straight into its target location. Skip docstrings explaining the obvious, single-use helpers, and guard clauses against inputs the brief does not describe. Then trigger the relevant test or check — usually `pytest tests/test_outputs.py -v`, sometimes a script the brief names, sometimes a manual check.

### 4. Decide

Either the check passes (you are done — stop), or it does not. If it failed, scroll the output until you see the specific assertion or error message. Adjust the smallest piece that fixes that one failure. Re-check. Repeat.

---

## Working vocabulary

- **Grader** — the script or assertion the brief points to as the ruling on pass/fail.
- **Reachable surface** — the directories and processes you can actually inspect from the shell. If your `ls` does not match the brief, your map is wrong before your code is.
- **Independent operations** — actions that do not need each other's output; bundle them into a single message rather than firing them one at a time.
- **Spot check** — a tiny manual confirmation (file exists at the requested path, header line is correct, last line is non-empty) before invoking the full grader.
- **Underlying behaviour** — what code already in the workspace actually does. Do not assume; read it.

---

## Rules of thumb

- Doing beats describing. Each message should leave the workspace one step further along.
- Group reads that do not depend on each other into one round-trip.
- Quote names, paths, and constants from the brief by copy rather than retype; humans miss case and punctuation reliably.
- Do not fabricate filenames, dates, or signatures from memory. Confirm against the file or directory first.
- The grader's verdict is the verdict; your own confidence is not.

---

## When the output has to survive a transformer

Some briefs hand you a script that reads, validates, or rewrites a file before the deliverable is consumed by something else (a renderer, a runtime, a downstream parser). The two readers can disagree about the same bytes. Look for shapes that the first reader treats as one harmless thing and the second one breaks apart — irregular structures, half-formed delimiters, ambiguous whitespace are the usual seams. Read the source of the transformer first; the gap is in what it does not strip, not what it does.

---

## What burns the session

- Reading without writing: the next read should change what you do; if it does not, write something instead.
- Wrong drop point: a perfect file in the wrong directory scores nothing.
- One enormous edit: smaller iterations against the grader land faster than one big bet.
- Polishing a passing solution: once the grader is green, leave it green.
