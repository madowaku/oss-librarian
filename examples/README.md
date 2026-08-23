# OSS Librarian field examples

実在のGitHub repositoryを、候補比較 → source/license/test確認 → 再利用境界の決定まで行った実戦例です。

- [Godot 5レーン障害物回避](./godot-five-lane-dodge.md) — lane管理、spawn、poolingを別候補から **partial adaptation / algorithm-reference** として組み合わせる例
- [Reactドラッグ並べ替えUI](./react-drag-sort.md) — dnd-kitの **whole-library reuse candidate** と、依存を抑える **partial adaptation** を比較する例
- [SQLiteセーブデータとMigration](./sqlite-save-migrations.md) — Node/TSでは **whole-library reuse**、他環境では **algorithm/reference** から小さなrunnerへ落とす例

## Skill実戦評価

- **探索予算:** 各レポートで候補5件、深掘り3件、最終reading list 10ファイル以内を守れた。Godotだけはpoolingの未解決点を絞るため、追加検索を1回使った。探索を続けるより、責務ごとの境界を決める方が利益が大きい時点で停止できた。
- **GitHub MCP:** repository search、repo metadata、directory、file、commitを取得するには十分だった。connectorがcommitの日付を返さないケース、CI fileが空/取得不能なケースがあり、その場合は日付やCIを`unknown`と明記し、取得できたcommit/file linkを根拠にした。
- **Evidence分類:** `direct`（ファイル/metadataで確認）、`corroborated`（READMEとsourceが一致）、`inferred`（互換性・統合コストの推論）、`unknown`（取得できない/未検証）を分けると、starsやREADMEだけの過信を避けやすかった。
- **templateの不足:** 既存templateはhandoffの骨格として足りた。実戦では候補表に「tests/CI」「maintenance」「dependencies」「integration cost」を明示し、各recommendationに`Do not import`を追加すると読み手がさらに迷わない。今回は既存Skill本体とtemplateを変更していない。
- **license:** `CrossLane`や`ruckusing-migrations`のようにLICENSE取得が404/不明な候補があり、コードをコピーせず`reject`できた。MIT/Apache-2.0も「安全」の法的断定ではなく、確認したfile/metadataと通知保持の要件として記録した。
- **改善提案:** 次のtemplate改訂では、候補表に `Evidence type` と `Dependencies / tests / maintenance` の専用列、本文に `Do not import` と `Stop condition` の必須見出しを追加するとよい。重大な仕様問題ではないため、この作業ではSkill本体を変更しない。
