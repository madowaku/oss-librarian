# Game Source Mode v0.2

Use this reference when the target is a video game, game mechanic, gameplay system, game engine integration, or a feature best understood from complete game source rather than from a small library alone.

The goal is not to find a game that looks similar. The goal is to find source that exposes a useful implementation boundary, then decide whether to reuse code, adapt a pattern, or use it only as architectural evidence.

## Entry conditions

Activate game-source mode when one or more of these is true:

- the request names a gameplay mechanic such as movement, spawning, combat, inventory, dialogue, camera, save/load, AI, economy, racing, networking, or procedural generation;
- the target stack is a game engine or game framework such as Godot, Unity, Bevy, Phaser, Three.js, SDL, or a custom engine;
- a complete game is likely to provide better evidence than an isolated package;
- the user asks for prior art from shipped or playable games.

Do not activate only because the project has graphics. Generic UI, rendering utilities, build tooling, or backend infrastructure should continue through the normal OSS Librarian workflow unless game-specific behavior matters.

## Discovery sources

Treat curated game lists as discovery indexes, not as evidence that every linked repository is reusable.

A useful seed index is:

- `bobeff/open-source-games` - categorized links to open-source games, source releases, ports, reimplementations, and reverse-engineering projects. Its index license does not determine the licenses of linked projects.

After an index produces candidates, inspect each candidate repository directly. Prefer canonical repositories over mirrors and forks unless the fork is the maintained implementation being evaluated.

## Search escalation ladder

Do not stop at repository-title search or README evidence. For every retained candidate, climb this ladder only as far as needed:

1. **Discovery** - curated index, repository search, or known project name.
2. **Mechanic signals** - search behavior terms and source-level signals such as `velocity`, `mouse look`, `spawn table`, `wave`, `timer`, `pool`, `tick`, `production`, or `demand`.
3. **Repository code search** - search the candidate repository for symbols, filenames, data types, and implementation terms.
4. **Manifest-guided directory walk** - if code search is empty/unindexed, use the engine manifest and repository tree to inspect likely source directories.
5. **Exact file fetch** - fetch the smallest implementation file(s), then find the scene/prefab/resource/usage/data file that wires them into runtime state.
6. **Fallback or stop** - if no evidence pair can be established within budget, downgrade the candidate to `reference`, reject it, or try one focused alternative. Do not promote README-only evidence to direct reuse.

A failed candidate is useful information. Drop it when source inspection shows that the title, description, or genre was misleading.

### Engine-specific directory fallback

When indexed code search fails, use these defaults before giving up.

**Godot**

`project.godot -> scenes/ or *.tscn -> scripts/ or *.gd/*.cs -> *.tres/resources -> input/autoload ownership`

Inspect `project.godot` first for engine generation, main scene, input map, autoloads, and feature flags. Then follow scene script references rather than crawling every file.

**Unity**

`ProjectSettings/ProjectVersion.txt -> Packages/manifest.json -> Assets/ -> likely Scripts or Runtime directories -> prefab/scene references`

Search common source roots such as `Assets/**/Scripts`, `Assets/**/Runtime`, feature-specific folders, and assembly definitions. If a README names a class such as `WaveSpawner`, use directory listing plus exact file fetch when repository code search cannot resolve it.

For other engines, identify the equivalent manifest, source roots, scene/entity composition, and data definition path.

## Candidate provenance classes

Classify every game candidate before reading implementation details:

| Class | Meaning | Default reuse posture |
| --- | --- | --- |
| `native-open-game` | Original game developed as open source | inspect code and asset licenses separately |
| `engine-native-example` | Game intentionally built on the target engine/framework | strong architecture/reference candidate |
| `source-release` | Source published from a formerly proprietary game | code may be inspectable; game data often remains restricted |
| `source-port` | Port or modernization of released source | useful for systems and platform adaptation; verify upstream obligations |
| `reimplementation` | New implementation compatible with an existing game | useful architecture evidence; may depend on original data |
| `decompilation` | Reconstructed source intended to match an original binary | default to reference-only until rights and license are clear |
| `reverse-engineering` | Compatibility or behavior reconstructed from an existing title | default to reference-only unless explicit licensing supports reuse |

This class is independent from repository popularity or technical quality.

## Game candidate card

For each surviving candidate capture:

- `repository`
- `ref`
- `provenance_class`
- `genre`
- `dimension`: `2D`, `2.5D`, `3D`, `mixed`, or `unknown`
- `engine_framework`
- `language`
- `engine_version`
- `mechanics`
- `architecture_signals`
- `target_fit`
- `code_license`
- `license_scope_text`: the repository wording that defines what the license covers, such as `source code`, `entire project`, or `assets`; use `unknown` when not explicit
- `asset_license`
- `original_data_required`: `yes`, `no`, or `unknown`
- `tests_ci`
- `maintenance_signal`
- `implementation_files`
- `usage_or_scene_files`
- `evidence_pair_status`: `complete`, `partial`, or `missing`
- `extractability`: `high`, `medium`, `low`
- `reuse_scope`: `reuse`, `partial`, `reference`, or `reject`
- `evidence_type`

Unknown values must stay `unknown` rather than being inferred from genre, engine, or a broad license filename.

## Evidence-pair gate

Before classifying a candidate as `reuse` or `partial`, require an **evidence pair**:

1. at least one implementation file or symbol that directly exposes the requested behavior; and
2. at least one usage, scene, prefab, resource, data definition, test, or integration artifact that shows how the behavior is owned or invoked.

If only one half exists, `evidence_pair_status=partial` and the candidate cannot exceed `reference` unless the missing half is irrelevant and that exception is explained. README prose can corroborate an evidence pair but cannot substitute for both halves.

This gate prevents attractive architecture descriptions from becoming unsupported copy/adaptation recommendations.

## Mechanic-first search

Search for behavior and implementation signals, not only game names or genre labels.

Examples:

| Mechanic | Search concepts | Source signals worth finding |
| --- | --- | --- |
| lane movement | lane switch, runner movement | lane index, target x, clamp, tween, input action |
| spawn system | wave spawn, encounter spawn | spawn table, timer, pool, factory, scene instantiate |
| inventory | item inventory, equipment | item id, slot, stack, serialization, resource/data object |
| combat | weapon, projectile, damage | hitbox, hurtbox, cooldown, damage event, projectile pool |
| FPS controller | character controller, weapon sway | velocity, floor check, mouse look, recoil, weapon state |
| racing | vehicle controller, checkpoint | wheel, traction, lap, checkpoint, spline, respawn |
| RTS | unit selection, command queue | selection set, order queue, pathfinding, formation |
| city builder | simulation, economy, zoning | tick, demand, production chain, agent, grid, budget |
| save system | game save, checkpoint | version, migration, snapshot, serialization, slot |

A genre match is not enough. Keep a candidate only when its source exposes the requested mechanic or a directly useful architectural boundary.

## Engine-aware inspection

### Godot

Prefer evidence from:

1. `project.godot` for engine/version and autoload/input signals;
2. the relevant `.gd` / `.cs` scripts;
3. `.tscn` scenes showing composition and ownership;
4. `.tres` / resource definitions for data-driven patterns;
5. tests or small playable scenes when present.

Record whether the useful behavior depends on scene hierarchy, signals, resources, autoload singletons, physics nodes, or engine-version-specific APIs.

### Unity

Prefer evidence from:

1. `ProjectSettings/ProjectVersion.txt` and package manifests;
2. focused `MonoBehaviour`, `ScriptableObject`, ECS, or DOTS implementation files;
3. prefab/scene ownership when source-readable metadata is available;
4. tests and sample scenes;
5. package dependencies and render/input stack assumptions.

Record whether the pattern depends on old/new Input System, CharacterController/Rigidbody, URP/HDRP, NavMesh, Addressables, ECS, or editor-only tooling.

### Other engines/frameworks

Identify the equivalent project manifest, runtime ownership model, scene/entity composition, data model, and the smallest source-to-usage path that proves the mechanic.

## Code and content license split

Never collapse game licensing into one field.

Check separately:

1. **code license** - source files and libraries;
2. **license scope wording** - what the project actually says the license covers; record `source code`, `software`, `project`, `assets`, or `unknown` instead of silently expanding scope;
3. **asset/content license** - art, audio, maps, fonts, dialogue, levels, shaders, and bundled data;
4. **original-data dependency** - whether the project expects files from a commercial or separately licensed game;
5. **name/trademark boundary** - do not treat an open code license as permission to reuse a title, logo, or branded content.

A curated index license applies only to that index unless the linked project says otherwise.

If code is permissively licensed but assets are unclear, code adaptation may still be possible while assets remain excluded. If the project requires original commercial data, treat that data boundary as a hard `do not import` item.

For permissive licenses, record concrete obligations such as notice/copyright preservation without claiming legal certainty beyond the repository evidence.

## Extractability

Use extractability to estimate how cleanly the useful behavior can cross into the target project.

- `high`: behavior lives in a small module with explicit inputs/outputs and limited dependencies;
- `medium`: behavior is understandable but coupled to several engine systems or project conventions;
- `low`: behavior is entangled with global state, generated code, proprietary/original data, custom engine internals, or broad project architecture.

Low extractability does not mean low value. It often means `algorithm/reference only` is the right decision.

## Cross-engine architecture handoff

When the useful candidate is in a different engine or language, do not translate files line-by-line. Extract a small engine-neutral state/data-flow model first.

Prefer a compact form such as:

`clock/tick -> read immutable inputs -> compute production/consumption -> commit ledger/state mutations -> emit events -> presentation reacts`

Name ownership boundaries, ordering constraints, deterministic inputs, and side effects. Then map those roles to the target engine only after the architecture is explicit.

## Game-source output

When game-source mode is active, add these items to the normal report:

- provenance class for each candidate;
- mechanic match rather than genre similarity alone;
- code license, license scope wording, and asset/content license as separate evidence;
- original-data requirement;
- evidence-pair status;
- extractability;
- exact implementation file plus a usage/scene/data file that proves integration when available;
- explicit `Do not import` for assets, original data, trademarks, unrelated engine subsystems, or copied architecture that is too coupled;
- an engine-neutral state/data-flow model when the recommendation crosses engines.

## Stop conditions

Stop when one candidate exposes the mechanic with direct source evidence, a complete evidence pair, acceptable rights boundaries, and a clear reuse boundary, even if a more famous game exists.

Also stop when all remaining candidates are only visually similar, require inaccessible proprietary data, have unclear source licensing, cannot produce an evidence pair within the research budget, or are so coupled that a focused local implementation is cheaper than further excavation.
