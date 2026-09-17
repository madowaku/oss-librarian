# Game Source Mode Eval Rubric v0.1

Score each fixture out of 100. Use the same rubric for all five cases, with fixture-specific assertions from `cases.jsonl` applied on top.

## Weighted dimensions

| Dimension | Weight | Full-credit behavior |
| --- | ---: | --- |
| Mode activation and framing | 10 | Recognizes the request as game-source research and builds a capability brief with target, constraints, reuse intent, and unknowns. |
| Mechanic-first discovery | 15 | Searches by behavior/implementation signals rather than title/genre alone; decomposes the mechanic where useful. |
| Evidence quality | 20 | Uses source files, tests/usage/scene/data files, manifests/license files, and clearly labels direct/corroborated/inferred/unknown evidence. |
| Provenance classification | 10 | Correctly distinguishes native open game, engine-native example, source release, source port, reimplementation, decompilation, and reverse engineering where relevant. |
| License and asset separation | 20 | Separately evaluates code license, asset/content license, original-data dependency, and trademark/content boundaries. |
| Reuse-scope judgment | 15 | Chooses reuse / partial adaptation / algorithm-reference / reject with a named adaptation boundary and explicit `Do not import`. |
| Bounded handoff | 10 | Keeps within exploration budget, gives a concise comparison, <=10-file reading list, next implementation step, and stop condition. |

## Scoring anchors

For each dimension:

- **Full credit:** behavior is explicit and supported by evidence.
- **Half credit:** behavior is present but incomplete, weakly evidenced, or inconsistently applied.
- **Zero:** behavior is absent or contradicted by the recommendation.

## Hard-fail conditions

A fixture fails regardless of numeric score if any fixture-specific `hard_fail` assertion is triggered.

The following are global hard fails:

1. Recommending direct code copying from a repository whose code license is missing or unresolved.
2. Treating a curated index license as if it automatically applied to linked repositories.
3. Treating an open-source code license as proof that bundled art, audio, level data, logos, names, or original commercial game data are reusable.
4. Classifying decompiled or reverse-engineered code as a direct reuse candidate without explicit licensing evidence that supports that conclusion.
5. Claiming legal certainty where the repository evidence is ambiguous.

## Pass thresholds

- **Fixture pass:** 80/100 or higher and no hard fail.
- **Pack pass:** average 80/100 or higher, no hard fails, and both `gsm-004` and `gsm-005` individually pass.
- **Strong pass:** average 90/100 or higher with all five fixtures >=85.

## Diagnostic labels

Attach zero or more labels after grading:

- `mode-miss` — did not activate the game-source specialization.
- `genre-search` — relied on genre/title similarity rather than mechanic signals.
- `readme-only` — did not inspect implementation evidence.
- `provenance-blur` — source release/decompilation/reimplementation distinctions were lost.
- `license-collapse` — code and asset/content rights were conflated.
- `asset-leak` — unsafe asset/content reuse crossed the boundary.
- `scope-bloat` — recommended importing a framework/game subsystem larger than necessary.
- `unbounded-search` — exceeded search/deep-read/file budgets without justification.
- `weak-handoff` — recommendation did not identify exact next files/boundaries/actions.

These labels are intended for regression tracking across future Skill revisions.
