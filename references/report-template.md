# OSS Librarian Report

Use this as a compact handoff. Omit sections that have no evidence rather than filling them with guesses.

## Request and budget

- **Goal:**
- **Target stack/version/platform:**
- **Reuse intent:** whole component / partial adaptation / algorithm only
- **License constraints:**
- **Exploration budget:** 5 candidates / 3 deep reads / 10 final files
- **Mode:** standard / game-source

## Capability brief

- **Capability:**
- **Non-goals:**
- **Local integration boundary:**
- **Search terms and implementation signals:**

## Game-source context (optional)

Use only when game-source mode is active.

- **Mechanic:**
- **Dimension:** 2D / 2.5D / 3D / mixed / unknown
- **Target engine/version:**
- **Content reuse in scope:** yes / no / unknown
- **Discovery indexes used:**
- **Search escalation used:** discovery / mechanic signals / code search / directory walk / exact fetch

## Candidate comparison

| Candidate | Capability evidence | Compatibility | License evidence | Quality/maintenance | Scope | Decision | Confidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `owner/repo` | `path:Symbol` (direct/corroborated/inferred/unknown) | fit / caveat | file + license | tests, CI, recent activity | module/pattern | reuse / partial / reference / reject | high/medium/low |

### Game-source candidate details (optional)

| Candidate | Provenance | Mechanic match | Engine/dimension | Code license + scope | Asset/content license | Original data | Evidence pair | Extractability |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `owner/repo` | native-open-game / source-port / reimplementation / ... | direct / partial / weak | engine + 2D/3D | license + `source code/project/...` | evidence or unknown | yes/no/unknown | complete/partial/missing | high/medium/low |

Candidate cards may also be emitted against `references/game-source-card.schema.json` when machine-readable handoff is useful.

## Evidence to read

For each selected file, include the repository ref and why it matters:

1. `owner/repo@ref:path` - reason; evidence type; symbol or line anchor.
2. `owner/repo@ref:path` - reason; evidence type; symbol or line anchor.

For game-source mode, pair the implementation file with one scene/prefab/resource/entity/usage/data file that proves runtime integration before recommending `reuse` or `partial`.

## Cross-engine model (optional)

Use when the recommendation crosses engines or languages.

`clock/input -> read state -> compute -> commit mutations -> emit events -> presentation`

- **State owner:**
- **Ordering constraints:**
- **Deterministic inputs:**
- **Side effects:**
- **Target-engine mapping:**

## Recommendation

- **Preferred candidate:**
- **Why it fits:**
- **Reuse boundary:**
- **Do not import:**
- **License/attribution obligations:**
- **Compatibility risks or open questions:**
- **Next implementation step:**
- **Stop condition:**

For game-source mode, explicitly include assets/content, original commercial data, names/logos/trademarks, and tightly coupled engine subsystems in `Do not import` when applicable.

## FIELD_NOTES.md entry (optional)

```markdown
### OSS discovery - YYYY-MM-DD
- repo/ref: `owner/repo@ref`
- license evidence: `path` - license/unknown; scope: source code/project/assets/unknown
- useful pattern: ...
- adopted boundary: ...
- avoided: ...
```
