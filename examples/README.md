# OSS Librarian field examples

実在のGitHub repositoryを、候補比較 → source/license/test確認 → 再利用境界の決定まで行った実戦例です。

- [Godot 5レーン障害物回避](./godot-five-lane-dodge.md) - lane管理、spawn、poolingを別候補から **partial adaptation / algorithm-reference** として組み合わせる例。ゲーム依頼なので、現在のSkillでは **game-source mode** の対象になる。
- [Reactドラッグ並べ替えUI](./react-drag-sort.md) - dnd-kitの **whole-library reuse candidate** と、依存を抑える **partial adaptation** を比較する例
- [SQLiteセーブデータとMigration](./sqlite-save-migrations.md) - Node/TSでは **whole-library reuse**、他環境では **algorithm/reference** から小さなrunnerへ落とす例

## Game-source mode

ゲーム機構、ゲームシステム、Godot/Unity等のengine-specific behaviorでは、通常のOSS探索に加えて [`references/game-source-mode.md`](../references/game-source-mode.md) を使います。

追加で確認する主な項目:

- mechanic match。見た目やgenreが似ているだけでは候補に残さない
- provenance class。native open game / source release / source port / reimplementation / decompilation / reverse engineering等を区別する
- code licenseとasset/content licenseを分離する
- original commercial dataが必要かを確認する
- implementation fileとscene/prefab/resource/usage側を対で読む
- extractabilityを high / medium / low で記録する
- 必要なら [`game-source-card.schema.json`](../references/game-source-card.schema.json) で候補カードを機械可読化する

`bobeff/open-source-games` のようなcurated listは、候補発見用のindexとして扱います。index自身のlicenseをリンク先repositoryへ継承させて判断してはいけません。

## Skill実戦評価

- **探索予算:** 各レポートで候補5件、深掘り3件、最終reading list 10ファイル以内を守れた。Godotだけはpoolingの未解決点を絞るため、追加検索を1回使った。探索を続けるより、責務ごとの境界を決める方が利益が大きい時点で停止できた。
- **GitHub MCP:** repository search、repo metadata、directory、file、commitを取得するには十分だった。connectorがcommitの日付を返さないケース、CI fileが空/取得不能なケースがあり、その場合は日付やCIを`unknown`と明記し、取得できたcommit/file linkを根拠にした。
- **Evidence分類:** `direct`（ファイル/metadataで確認）、`corroborated`（READMEとsourceが一致）、`inferred`（互換性・統合コストの推論）、`unknown`（取得できない/未検証）を分けると、starsやREADMEだけの過信を避けやすかった。
- **template:** 通常handoffに加えて、game-source modeでは provenance、mechanic match、code/content license split、original-data requirement、extractabilityを追記する。
- **license:** `CrossLane`や`ruckusing-migrations`のようにLICENSE取得が404/不明な候補があり、コードをコピーせず`reject`できた。MIT/Apache-2.0も「安全」の法的断定ではなく、確認したfile/metadataと通知保持の要件として記録した。
- **game-source safety rail:** curated index、source release、decompilation、reverse engineeringを同列の「自由に使えるOSS」とみなさない。コード、アセット、原作データ、名称/商標の境界を別々に記録する。
