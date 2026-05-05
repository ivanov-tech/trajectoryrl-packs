# Iteration Index — sv.1127

| rev  | base | change                                  | cancel-async | log-summary | break-filter | sum / 3 | mean |
|------|------|-----------------------------------------|-------------:|------------:|-------------:|--------:|-----:|
| r1   | —    | initial sv.v1 (Solve and Verify)        |        3/6   |        2/2  |        0/1   |   1.500 | 0.500|

## r1
- **on-chain hash**: fd9f1589faa4b54d827345a628dcdd53a6f47217026943779d727d6093ab0039
- **size**: 6325 bytes
- **submitted ep1127**: yes (00:43 UTC May 5)
| r2   | r1   | drop "Durable notes" /workspace/learned section |    0/6   |        2/2  |        0/1   |   1.000 | 0.333|

## r2
- size: 5880 bytes (-445 vs r1)
- hash: bd33c473cf18be15443904d7f0d6c7d99adef4b8cf2d44eba674d458e7ed7471
- regression: cancel-async-tasks 3/6 → 0/6 (agent only 3 tool calls, 317 output tokens — exposition trap)
- log-summary still 2/2, html still 0/1
- diagnosis: removing /workspace/learned/ section didn't help. Variance dominates with such small SKILLs.
| r3   | r2   | add "Sequence" 6-step at top + "first attempt on disk" tagline |    5/6   |        0/2  |        0/1   |   0.833 | 0.278|

## r3
- size: 6467 bytes (+587 vs r2)
- hash: 0d90a965e647c7bda1da5164bf62530f74ac09029dd1636b0219d37d3060e8a2
- cancel: BEST EVER (5/6 = 0.833) — Sequence section worked here
- log/html: regression (3-4 tool calls, agent paused after exposition)
- diagnosis: qwen variance dominates. r3 cancel was best but log/html got unlucky.
| r4   | r3   | "do not summarise" + concrete `cat`/`ls` commands in Sequence | 0/6 | 0/2 | 0/1 | 0.000 | 0.000|
| r1t2 | r1   | (re-run r1 to estimate variance) | 5/6 | 0/2 | 0/1 | 0.833 | 0.278|

## r4
- size: 6761 bytes, hash: 97205aa5e22d28afcdd213e42de5465bd6834ed338580f0bbaffa2abc96d3440
- catastrophic: all 3 scenarios scored 0. 3-4 tool calls each, agent stalled.
- "do not summarise" backfired — agent took it too literally, never resumed action

## VARIANCE FINDING
- r1 baseline runs: trial1=1.500, trial2=0.833 — variance ~0.66!
- LLM (qwen) variance dominates SKILL content for this task structure
- Best individual scenario peaks observed: cancel=5/6, log=2/2, html=0/1
- HTML scenario fails consistently (5/5 trials so far) — adversarial task, generic SKILL insufficient
| r5   | r1   | + "task types" 3-paragraph hint | 0/6 | 0/2 | 0/1 | 0.000 | 0.000 |
| r1t3 | r1   | (variance trial 3) | 0/6 | 2/2 | 0/1 | 1.000 | 0.333 |
| r6   | r1   | + "write first attempt early" for adversarial | 0/6 | 2/2 | 0/1 | 1.000 | 0.333 |
| r1t4 | r1   | (variance trial 4) | 0/6 | 2/2 | 0/1 | 1.000 | 0.333 |

## r1 baseline summary (4 trials)
- trials: [1.500, 0.833, 1.000, 1.000]
- mean: 1.083 / 3.000
- best: 1.500 (t1)
- HTML scenario: 0/4 trials passed — universally fails
- cancel: 1 trial out of 4 hit 3/6, 1 hit 5/6, 2 hit 0/6 — variance 0..0.833
- log: 3 trials hit 2/2, 1 trial hit 0/2 — usually solved

## r6 vs r1
- r6 single trial: 1.000 (same as r1t3, r1t4)
- "write first attempt early" addition: did not help html in this trial
| r7   | (new) | radical short 2KB SKILL "Operating Notes" | 0/6 | 2/2 | 0/1 | 1.000 | 0.333 |
| r8   | r1   | + "Default opening sequence" structured | 0/6 | 0/2 | 0/1 | 0.000 | 0.000 |

## Final aggregate (10 trials total across revisions)

| rev | trials | sum scores | mean sum |
|-----|-------:|------------|---------:|
| **r1** | 4 | 1.500, 0.833, 1.000, 1.000 | **1.083** |
| r2 | 1 | 1.000 | 1.000 |
| r3 | 1 | 0.833 | 0.833 |
| r4 | 1 | 0.000 | 0.000 |
| r5 | 1 | 0.000 | 0.000 |
| r6 | 1 | 1.000 | 1.000 |
| r7 | 1 | 1.000 | 1.000 |
| r8 | 1 | 0.000 | 0.000 |

## Conclusions

1. **r1 has the highest mean (1.083) and only revision with multi-trial data.**
2. Variance per trial: ~±0.5 (range 0.000-1.500 even within r1).
3. **No revision has shown statistically significant improvement over r1.** Most "improvements" hit qwen variance and got 0.000.
4. HTML scenario (break-filter-js-from-html): **0/10 trials passed**. Generic SKILL cannot solve this adversarial task.
5. Cancel-async-tasks variance: sometimes 5/6, sometimes 0/6 — same SKILL, different runs.
6. Log-summary-date-ranges: most reliable, 2/2 in 6+ of 10 trials.

## Recommendation
**Keep r1 on-chain (currently submitted as sv.v1, hash fd9f1589...).** No revision tested produced a robust improvement.
| r9   | r6   | + L3 "two-process disagreement" hint (abstract) | 0/6 | 1/2 | 0/1 | 0.500 | 0.167 |

## r9 — L3 hint trial
- Did NOT submit (local test only)
- html still failed in 3 tool calls — hint did not change agent behavior
- log got 1/2 instead of usual 2/2 — ambiguity may have confused
- Conclusion: even abstract L3 doesn't help break-filter-js-from-html within 15 turns

## Phase 2: r11/r11s/r12 — Phase+Glossary structure (after analyzing competitors)

| rev | size | trials | trial scores | mean |
|-----|-----:|-------:|---|-----:|
| r10 | 4.1KB | 1 | 1.833 | 1.833 (REJECTED — copied phrases from uid68 fresh) |
| r11 (medium) | 3.5KB | 3 | 1.833, 2.000, 1.833 | **1.889** |
| **r11s** (short) | **1.8KB** | 3 | 1.833, 2.000, 2.000 | **1.944** ← best |
| r11v (verbose) | 7.6KB | 1 | 1.000 | 1.000 (overload) |
| r12 (r11s + tactical loop V1+V2) | 2.8KB | 3 | 2.000, 1.000, 1.500 | **1.500** ⚠ |

## r12 detail
- size: 2784 bytes, hash: a4104174642cbe2588eca1ca3206cb2ea55f7cdc7b3ec3a329d78113e4cf5852
- cancel: 6/6 in ALL 3 trials (robust improvement vs r11s 5-6/6)
- log: regressed (2/2, 0/2, 1/2) — extra hint may have confused
- html: still 0/3 — V1+V2 (tactical loop) did not crack it
- net: worse mean (1.500 < r11s 1.944)
