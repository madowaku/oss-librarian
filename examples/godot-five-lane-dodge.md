# OSS Librarian Report: Godot 5-Lane Dodge

## Request and budget

- **Goal:** Godot 4.xで、スマホ向け5レーン障害物回避の移動・スポーン・再利用パターンを調査する。
- **Target stack/version/platform:** Godot 4.x、タッチ入力を含むモバイル。対象はゲーム全体ではなく、レーン管理、左右移動、障害物スポーン、pooling。
- **Reuse intent:** partial adaptation / algorithm only。既存ゲームを丸ごと採用しない。
- **License constraints:** MIT、Apache-2.0、BSDを優先。ライセンス不明のコードはコピーしない。
- **Exploration budget:** 5 candidates / 3 deep reads / 10 final files。候補5件、深掘り3件（`mujtaba-io/3d-endless-runner`、`NovemberDev/novemberdev-godot-endless-runner-tutorial`、`TheDuckCow/godot-road-generator`）で停止した。Godot endless-runnerの追加検索は、未解決だったpooling/レーン実装を絞るための1回だけ行った。

## Capability brief

- **Capability:** 5個のlane indexを状態として持ち、入力で隣のレーンへ移動し、複数種類の障害物を上方から流す。可能なら生成済みノードを再利用する。
- **Non-goals:** カメラ、UI、スコア、アート、ゲーム全体、3D交通シミュレーションの導入。
- **Local integration boundary:** `lane_index: int` と `lane_x: PackedFloat32Array` をローカルのplayer/controllerに持ち、spawnerは `spawn_obstacle(kind, lane_index, z)` だけを公開する。poolは障害物sceneごとの小さなadapterに限定する。
- **Search terms and implementation signals:** `godot lane`、`godot endless runner`、`lane index`、`obstacle spawn`、`object pooling`、`Godot 4.x`。見るシグナルは `target_lane`、lane座標配列、`instantiate`、poolからの取得/返却、Godot version、CI/GUT。

## Candidate comparison

| Candidate | Capability evidence | Compatibility | License evidence | Quality/maintenance | Scope / integration cost | Decision | Confidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [`mujtaba-io/3d-endless-runner`](https://github.com/mujtaba-io/3d-endless-runner) | [`Player.gd`](https://github.com/mujtaba-io/3d-endless-runner/blob/82dd76e846311b9129c0724555f5e34bdb4a4a57/Player/Player.gd#L7-L35) の `LANES`、`current_lane`、clamp、`lerp`（direct）。[`Levels/Level.gd`](https://github.com/mujtaba-io/3d-endless-runner/blob/82dd76e846311b9129c0724555f5e34bdb4a4a57/Levels/Level.gd#L39-L107) はlaneを乱択してcoin/obstacleをinstantiate（direct）。 | `project.godot` は config version 5、feature `4.2`（direct）。5レーン化は座標配列を拡張する推論。外部runtime依存は manifest上確認できない（direct/unknown）。 | [`LICENSE`](https://github.com/mujtaba-io/3d-endless-runner/blob/main/LICENSE) はMIT（direct）。 | root listingにテスト/CIは見えない（direct/unknown）。非archive。探索で確認した最新commitは [`82dd76e`](https://github.com/mujtaba-io/3d-endless-runner/commit/82dd76e846311b9129c0724555f5e34bdb4a4a57)（message: Update README.md、日付はconnector未提供）。 | player lane + obstacle spawn。integration cost: low-medium（3→5 lane座標とscene pathを置換、poolは別実装）。 | **partial adaptation** | high |
| [`NovemberDev/novemberdev-godot-endless-runner-tutorial`](https://github.com/NovemberDev/novemberdev-godot-endless-runner-tutorial) | [`scripts/WORLD.gd`](https://github.com/NovemberDev/novemberdev-godot-endless-runner-tutorial/blob/19a729bb84d50192254b4e71400ba50862df9935/scripts/WORLD.gd#L138-L227) はpart/obstacleをpoolから取得し、lane pointへ配置（direct）。[`ObjectPooling.gd`](https://github.com/NovemberDev/novemberdev-godot-endless-runner-tutorial/blob/19a729bb84d50192254b4e71400ba50862df9935/addons/object_pooling/ObjectPooling.gd#L39-L170) は事前生成、active/inactive管理、reset、返却（direct）。 | `project.godot` は config version 4、Spatial、autoloadを使うGodot 3構成（direct）。Godot 4への直接互換性はない（direct）。 | [`LICENSE`](https://github.com/NovemberDev/novemberdev-godot-endless-runner-tutorial/blob/main/LICENSE) はMIT（direct）。 | root listingにCI/テストは見えない（direct/unknown）。非archive。最新観測commit [`19a729b`](https://github.com/NovemberDev/novemberdev-godot-endless-runner-tutorial/commit/19a729bb84d50192254b4e71400ba50862df9935)（Revision、日付はunknown）。 | poolingの契約とspawn lifecycleのみ。integration cost: medium-high（Godot 3のautoload/Spatial APIを4.xへ翻訳）。generated file全体は対象外。 | **partial adaptation** | high |
| [`TheDuckCow/godot-road-generator`](https://github.com/TheDuckCow/godot-road-generator) | [`road_lane.gd`](https://github.com/TheDuckCow/godot-road-generator/blob/980bc04c9f95a5c49b787f0a7a64a458156d5b9b/addons/road-generator/nodes/road_lane.gd#L1-L64) のlane隣接、タグ、登録管理と [`road_lane_agent.gd`](https://github.com/TheDuckCow/godot-road-generator/blob/980bc04c9f95a5c49b787f0a7a64a458156d5b9b/addons/road-generator/nodes/road_lane_agent.gd#L92-L107) のlane assignment（direct）。 | `project.godot`/READMEはGodot 4.4+、3D Path3D traffic addon（direct）。2Dの5レーンには過剰で、曲線lane graphを使う場合だけ概念を移植（inferred）。 | addon READMEと [`LICENSE`](https://github.com/TheDuckCow/godot-road-generator/blob/980bc04c9f95a5c49b787f0a7a64a458156d5b9b/LICENSE) はMIT（direct）。 | [`tests.yaml`](https://github.com/TheDuckCow/godot-road-generator/blob/980bc04c9f95a5c49b787f0a7a64a458156d5b9b/.github/workflows/tests.yaml#L1-L35) はGodot 4.4.1/4.5.1のheadless + GUT（direct）。最新観測commit [`980bc04`](https://github.com/TheDuckCow/godot-road-generator/commit/980bc04c9f95a5c49b787f0a7a64a458156d5b9b) はPR merge（date unknown）。 | lane graph/assignmentの設計だけ。integration cost: high（3D Path3D交通addonを2D laneへ縮約）。 | **algorithm/reference only** | medium |
| [`hman278/Godot-Runner-Game`](https://github.com/hman278/Godot-Runner-Game) | READMEは「very simple endless runner」と説明するが、今回の深掘りではlane index、pool、該当source symbolを確認できなかった（direct/unknown）。 | Godot version、依存、モバイル対応はREADMEだけではunknown。 | [`LICENSE`](https://github.com/hman278/Godot-Runner-Game/blob/master/LICENSE) はMIT（direct）。 | README/ライセンスのみ確認。テスト/CIと保守状況はunknown。 | 一般的なrunnerの着想のみ。integration cost: unknown（実装証拠不足）。 | **algorithm/reference only** | low |
| [`Junshuo0205/CrossLane`](https://github.com/Junshuo0205/CrossLane) | READMEは「Godot.Frogger like」だけで、今回必要な実装sourceは証拠化できない（direct）。 | engine version、依存、モバイル適合性はunknown。 | `LICENSE` endpointが404で、ライセンス不明（direct）。 | READMEのみ。テスト/CI/maintenanceはunknown。 | 不明ライセンスのため再利用境界を定義できない。integration cost: unknown。 | **reject** | high |

### Evidence to read

最終reading listは10ファイル。リンクは調査時のref/commitへ固定し、行番号はconnectorで取得した内容のanchorである。

1. `mujtaba-io/3d-endless-runner@82dd76e:project.godot` — Godot config version 5 / feature 4.2。 [direct](https://github.com/mujtaba-io/3d-endless-runner/blob/82dd76e846311b9129c0724555f5e34bdb4a4a57/project.godot#L9-L29)
2. `mujtaba-io/3d-endless-runner@82dd76e:Player/Player.gd` — `LANES`、`target_lane`、左右入力とclamp/lerp。 [direct](https://github.com/mujtaba-io/3d-endless-runner/blob/82dd76e846311b9129c0724555f5e34bdb4a4a57/Player/Player.gd#L7-L35)
3. `mujtaba-io/3d-endless-runner@82dd76e:Levels/Level.gd` — coin/obstacleのlane選択、instantiate、spawn timer。 [direct](https://github.com/mujtaba-io/3d-endless-runner/blob/82dd76e846311b9129c0724555f5e34bdb4a4a57/Levels/Level.gd#L15-L107)
4. `NovemberDev/novemberdev-godot-endless-runner-tutorial@19a729b:project.godot` — Godot 3構成であることの互換性確認。 [direct](https://github.com/NovemberDev/novemberdev-godot-endless-runner-tutorial/blob/19a729bb84d50192254b4e71400ba50862df9935/project.godot#L9-L34)
5. `NovemberDev/novemberdev-godot-endless-runner-tutorial@19a729b:scripts/WORLD.gd` — `spawn_next_part` / `initialize_part` のpool取得と配置。 [direct](https://github.com/NovemberDev/novemberdev-godot-endless-runner-tutorial/blob/19a729bb84d50192254b4e71400ba50862df9935/scripts/WORLD.gd#L138-L227)
6. `NovemberDev/novemberdev-godot-endless-runner-tutorial@19a729b:addons/object_pooling/ObjectPooling.gd` — `load_from_pool`、activation、返却、reset。 [direct](https://github.com/NovemberDev/novemberdev-godot-endless-runner-tutorial/blob/19a729bb84d50192254b4e71400ba50862df9935/addons/object_pooling/ObjectPooling.gd#L39-L170)
7. `TheDuckCow/godot-road-generator@980bc04:project.godot` — Godot 4.4 feature宣言。 [direct](https://github.com/TheDuckCow/godot-road-generator/blob/980bc04c9f95a5c49b787f0a7a64a458156d5b9b/project.godot#L9-L20)
8. `TheDuckCow/godot-road-generator@980bc04:addons/road-generator/nodes/road_lane.gd` — lane隣接、register/unregister。 [direct](https://github.com/TheDuckCow/godot-road-generator/blob/980bc04c9f95a5c49b787f0a7a64a458156d5b9b/addons/road-generator/nodes/road_lane.gd#L32-L64)
9. `TheDuckCow/godot-road-generator@980bc04:addons/road-generator/nodes/road_lane_agent.gd` — `assign_lane` / current laneの遷移。 [direct](https://github.com/TheDuckCow/godot-road-generator/blob/980bc04c9f95a5c49b787f0a7a64a458156d5b9b/addons/road-generator/nodes/road_lane_agent.gd#L92-L107)
10. `TheDuckCow/godot-road-generator@980bc04:.github/workflows/tests.yaml` — Godot headless/GUTのCI。 [direct](https://github.com/TheDuckCow/godot-road-generator/blob/980bc04c9f95a5c49b787f0a7a64a458156d5b9b/.github/workflows/tests.yaml#L1-L35)

## Recommendation

- **Preferred candidate:** `mujtaba-io/3d-endless-runner` のlane index + spawnの考え方を **partial adaptation** として使う。poolingは `NovemberDev` の lifecycleを **algorithm/reference only** で参照する。
- **Why it fits:** Godot 4.2の実コードで、lane array → target lane → clamp/lerp、spawn timer → lane座標 → instantiateの最短経路が確認できる。5レーン化は `[-4, -2, 0, 2, 4]` のようなローカル座標へ置き換えればよい（この数値は要件からの設計案であり、repoの直接事実ではない）。
- **Reuse boundary:** player側は `lane_index` と境界チェックだけ、spawner側は `kind/lane_index/position` の責務だけを採用する。poolはGodot 4の自前Node poolとして、`acquire → reset → activate → release` の契約を再実装する。
- **Do not import:** Godot 3の `ObjectPooling.gd` をそのままコピーしない（autoload/Spatial/APIが不一致）。`godot-road-generator` の3D交通addon全体、未知ライセンスの `CrossLane`、runnerゲームのscene/UI/assetsも持ち込まない。
- **License/attribution obligations:** 直接コードをコピーする場合はMITの著作権/ライセンス通知を保持する。ここでは実装を写経せず、設計パターンを適合させる。法的な互換性判断は別途行う。
- **Compatibility risks or open questions:** Godot 4.xのminor差、タッチ入力のgesture設計、画面外解放タイミング、poolサイズ、端末上のGC/描画負荷は未検証（unknown）。モバイル実機でspawn burst、低FPS、scene resetをテストする。
- **Next implementation step:** 5レーンの座標と入力境界をテスト可能な純粋なlane controllerに切り出し、次に障害物poolのacquire/releaseテスト、最後にGodot sceneでspawnを接続する。
- **Stop condition:** lane管理・spawn・poolの3責務について上記の境界を決めた時点で探索を終了する。新しい候補を増やすのは、実機テストでGodot 4互換性またはpool性能に失敗した場合だけ。

## FIELD_NOTES.md entry

```markdown
### OSS discovery — 2026-08-23
- repo/ref: `mujtaba-io/3d-endless-runner@82dd76e` / `NovemberDev/novemberdev-godot-endless-runner-tutorial@19a729b`
- license evidence: `LICENSE` in both repos — MIT
- useful pattern: Godot 4 lane array + target index/clamp/lerp; separate spawn timer from lane coordinate selection
- adopted boundary: lane controller and spawn contract only; reimplement a small Godot 4 pool with acquire/reset/release
- avoided: Godot 3 generated ObjectPooling addon, whole runner scenes/assets, 3D traffic addon
```
