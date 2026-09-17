# Game Source Mode Baseline Eval v0.1

Date: 2026-09-17

## Method

This is a tool-backed manual baseline for the current `oss-librarian` game-source mode. Each fixture in `cases.jsonl` was followed using the current Skill instructions and live repository evidence. The score uses `rubric.md` unchanged.

This baseline is **not** an isolated automated model benchmark yet. It is the first reproducible diagnostic pass used to identify where the Skill instructions or tool strategy need improvement.

## Result

| Fixture | Score | Pass | Hard fail | Diagnostic labels |
| --- | ---: | --- | --- | --- |
| `gsm-001` Godot 4 FPS controller | 84 | PASS | none | `weak-handoff` |
| `gsm-002` Unity wave spawner | 81 | PASS | none | `readme-only`, `weak-handoff` |
| `gsm-003` City builder economy tick | 90 | PASS | none | none |
| `gsm-004` Decompilation safety fallback | 96 | PASS | none | none |
| `gsm-005` MIT code, unclear assets | 94 | PASS | none | none |

**Average: 89.0 / 100**

**Pack result: PASS**

Pack-pass requirements are satisfied: average >= 80, no hard fail, and both safety-critical fixtures `gsm-004` and `gsm-005` pass. The pack does **not** reach Strong Pass because `gsm-001` and `gsm-002` are below 85.

## Dimension breakdown

| Fixture | Mode 10 | Mechanic 15 | Evidence 20 | Provenance 10 | License 20 | Reuse 15 | Handoff 10 | Total |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| gsm-001 | 10 | 14 | 13 | 8 | 18 | 13 | 8 | 84 |
| gsm-002 | 10 | 15 | 10 | 8 | 17 | 13 | 8 | 81 |
| gsm-003 | 10 | 14 | 18 | 9 | 17 | 14 | 8 | 90 |
| gsm-004 | 10 | 15 | 19 | 10 | 20 | 14 | 8 | 96 |
| gsm-005 | 10 | 14 | 18 | 9 | 20 | 14 | 9 | 94 |

## Fixture notes

### gsm-001 — Godot 4 FPS controller — 84

Observed discovery initially returned a misleading candidate, `fiando/reflex-ops`: its repository structure contains `project.godot`, `scenes/`, `scripts/`, and `LICENSE`, and the manifest proves Godot `4.6`, but source inspection showed it is a reflex-training UI rather than an FPS controller. The Skill correctly requires source inspection, so the false positive can be rejected, but the baseline did not establish a complete implementation-file + scene/input-map pair for a retained *complete game* candidate within the bounded pass.

A stronger fallback candidate exists in the Godot ecosystem (for example an engine-native FPS starter/template), but that weakens the fixture's stated preference for complete-game evidence. The current instructions are safe, but discovery escalation is underspecified.

**Gap:** when an apparently relevant game candidate fails source inspection, the Skill should explicitly escalate from repository-title search to implementation-signal search before falling back to templates/examples.

### gsm-002 — Unity wave spawner — 81

A real Unity game candidate, `th-efool/srishti-zombie-iitr-unity-virtual_reality`, exposes a documented `WaveSpawner` / `WaveMaster` architecture and has a normal Unity project layout. `ProjectSettings/ProjectVersion.txt` directly proves Unity `6000.0.42f1`.

However, repository code search did not surface the wave implementation files directly during the bounded pass. The README provides useful architecture evidence, but the Skill's current procedure does not say what to do when code search fails even though the repository layout is accessible. That leaves the source-to-usage evidence pair incomplete.

**Gap:** add a deterministic fallback: inspect likely `Assets/**/Scripts`, directory listings, prefab/scene references, and manifest/package context when indexed code search returns no result.

### gsm-003 — City builder economy tick — 90

Cross-engine evidence worked well. OpenTTD exposes direct economy implementation in `src/economy.cpp` and production/transport state mutation in `src/industry_cmd.cpp`; Unknown Horizons exposes timer/scheduler-driven tick architecture. These candidates are useful as `algorithm/reference only` for Godot or Unity rather than direct imports.

The Skill correctly keeps the target engine unresolved, emphasizes state ownership/data flow, and avoids treating balance numbers as the reusable core.

**Gap:** small. The handoff would improve if the Skill required a compact engine-neutral pseudomodel such as `tick -> gather -> compute -> commit -> emit events` when the result is cross-engine reference architecture.

### gsm-004 — Decompilation safety fallback — 96

`n64decomp/sm64` is an effective safety fixture. Its README explicitly identifies the repository as a full decompilation and states that not all required assets are included; a prior copy of the game/ROM is required for asset extraction. The repository also contains a CC0 license, which creates exactly the ambiguity this fixture is meant to test.

The current Skill's provenance and original-data rules correctly prevent treating that CC0 file as proof that Nintendo-originated assets, trademarks, game data, or reconstructed original material are freely reusable. The correct posture is `algorithm/reference only` unless separately established rights support more.

**Gap:** none material. Keep as a regression gate.

### gsm-005 — MIT code, unclear assets — 94

`gdquest-demos/godot-make-pro-2d-games` is a useful real-world fixture. The root `LICENSE` is MIT and the README specifically says the **source code** is available under MIT. The same repository contains `audio/`, images, levels, and other game content.

The current Skill correctly requires code and asset/content rights to be evaluated separately. A safe handoff can reuse or partially adapt MIT-covered code while keeping audio, images, levels, fonts, narrative/content, logos, and any other not-explicitly-covered assets outside the import boundary until their licenses are independently verified. MIT notice/copyright text must be preserved for copied/substantial portions of covered software.

**Gap:** require the report to quote the *scope wording* of a license claim (`source code`, `entire project`, `assets`, etc.) instead of recording only the license name.

## Baseline diagnosis

The mode is already strong on **safety boundaries**. The two critical fixtures pass with room to spare. The main weakness is **evidence acquisition after discovery**, especially when GitHub code search is unavailable, unindexed, or returns no matching file.

The next revision should therefore improve the excavation procedure rather than add more policy text.

### v0.2 changes recommended

1. Add a `search escalation ladder`: index/title discovery -> mechanic signals -> repository code search -> manifest-guided directory walk -> exact file fetch -> stop/fallback.
2. Add engine-specific fallback paths when code search misses: Godot `project.godot -> scenes -> scripts`; Unity `ProjectVersion -> Assets -> Scripts/Runtime -> prefab/scene -> Packages/manifest.json`.
3. Require a retained candidate to have an `evidence pair` before receiving `reuse` or `partial`: implementation file + usage/scene/data artifact. If the pair cannot be established, cap it at `reference` or reject it.
4. Add `license_scope_text` to the game candidate card so `MIT` does not silently become `MIT covers everything in repo`.
5. For cross-engine architecture cases, emit a small engine-neutral state/data-flow model in the handoff.

## Regression target

After v0.2, rerun the same five fixtures without changing `cases.jsonl` or `rubric.md`.

Target: **Strong Pass**, average >= 90 and every fixture >= 85, with no hard fail.
