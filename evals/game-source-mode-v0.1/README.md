# Game Source Mode Eval Pack v0.1

Purpose: verify that `oss-librarian` activates game-source mode when appropriate and produces evidence-backed, license-aware reuse decisions from real game-source research.

## What this pack tests

The pack focuses on five failure-prone behaviors:

1. **Mechanic-first discovery** — search by implementation signals, not only genre or game title.
2. **Game-source mode activation** — detect that complete game source is stronger evidence than a generic library/example.
3. **License separation** — distinguish code license, asset/content license, and original-data requirements.
4. **Provenance classification** — identify native open games, engine-native examples, source releases, source ports, reimplementations, decompilations, and reverse-engineering projects.
5. **Reuse-scope judgment** — choose among reuse candidate, partial adaptation, algorithm/reference only, and reject, with explicit `Do not import` boundaries.

## Pack layout

- `cases.jsonl` — machine-readable eval fixtures.
- `rubric.md` — scoring rules and hard-fail conditions.
- `expected-behaviors.md` — human-readable expected behaviors for each fixture.

## How to run

For each row in `cases.jsonl`:

1. Give the `prompt` to an agent with `$oss-librarian` available.
2. Allow normal GitHub read-only research.
3. Capture the final response.
4. Grade it against both `must_include` and `must_not` assertions.
5. Apply the weighted rubric in `rubric.md`.

A run is considered healthy when:

- total score is **80/100 or higher**;
- no hard-fail condition is triggered;
- fixtures `gsm-004` and `gsm-005` both pass their license/provenance safety checks.

## Design notes

These fixtures intentionally avoid requiring one exact repository as the answer. The skill is evaluated on research behavior, evidence quality, provenance, licensing, and adaptation boundaries rather than memorizing a canonical repo.

Candidate count should remain within the normal OSS Librarian budget: at most 5 discovered candidates, at most 3 deep reads, and at most 10 files in the final reading list.

## Version

- Eval pack: `v0.1`
- Target skill behavior: `oss-librarian` game-source mode
- Added: 2026-09-17
