# OSS Librarian Report: SQLite Save Data and Migrations

## Request and budget

- **Goal:** SQLiteのsave databaseを、アプリ更新後も壊さずにschema versionを順に移行する設計を調査する。
- **Target stack/version/platform:** 小規模ゲームまたはdesktop app。言語は未確定なので、SQLite固有の設計を優先し、Node/TypeScript、Go、Swiftの実装を比較する。
- **Reuse intent:** migration frameworkをそのまま使う案と、最小の自前migration runnerへpartial adaptationする案の比較。
- **License constraints:** MIT/Apache-2.0/BSDを優先。ライセンス不明の実装はコピーしない。
- **Exploration budget:** 5 candidates / 3 deep reads / 10 final files。候補は5件、深掘りは `kriasoft/node-sqlite`、`pressly/goose`、`garriguv/SQLiteMigrationManager.swift` の3件で停止。`drizzle sqlite migrations` の追加検索は候補の直接適合性が低く、1回で打ち切った。

## Capability brief

- **Capability:** 初回DB作成、schema versionの記録、version順のmigration、各migrationのtransaction、失敗時rollback、テスト可能なmigration runner。
- **Non-goals:** 分散DB、オンラインschema変更、複数writer同期、暗号化、バックアップ製品の選定。
- **Local integration boundary:** `schema_migrations(version, name, checksum)`（最小ならversion/nameのみ）と、immutableな`Migration[]`をアプリ層から渡す。runnerは`applyUpTo(targetVersion)`と結果/エラーを返し、save dataのdomain modelは触らない。
- **Search terms and implementation signals:** `sqlite migration`、`schema_migrations`、`BEGIN COMMIT ROLLBACK`、`version_id`、`pending migrations`、`migration tests`、`SQLite.swift`、`TypeScript migration API`。

## Candidate comparison

| Candidate | Capability evidence | Compatibility | License evidence | Quality/maintenance | Scope / integration cost | Decision | Confidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [`kriasoft/node-sqlite`](https://github.com/kriasoft/node-sqlite) | [`readMigrations`](https://github.com/kriasoft/node-sqlite/blob/f86c213863353d3701d71d8993975a04138dbe99/src/utils/migrate.ts#L10-L62) は`001-name.sql`をid順に並べ、up/downを分離（direct）。[`migrate`](https://github.com/kriasoft/node-sqlite/blob/f86c213863353d3701d71d8993975a04138dbe99/src/utils/migrate.ts#L64-L137) はmetadata table作成、pending適用、migration+version insertを同一transaction、error時rollback（direct）。 | Node.js/TypeScriptのSQLite wrapper。`package.json`はversion 5.1.1、SQL migration API、Jest/TypeScript/sqlite3 test stack（direct）。他言語へは設計移植が必要。 | [`package.json`](https://github.com/kriasoft/node-sqlite/blob/f86c213863353d3701d71d8993975a04138dbe99/package.json#L1-L10) のlicenseはMIT（direct）。 | `test`/`test:ci`/coverage script（direct）、[`CircleCI config`](https://github.com/kriasoft/node-sqlite/blob/f86c213863353d3701d71d8993975a04138dbe99/.circleci/config.yml#L23-L58) はtest/build（direct）。非archive。最新観測commit [`f86c213`](https://github.com/kriasoft/node-sqlite/commit/f86c213863353d3701d71d8993975a04138dbe99) は5.1.1（date unknown）。 | 小規模Node/TS appならrunnerごと採用可能。integration cost: low（Node/TS）/high（他言語）。full SQLをmetadataへ保存する設計は要件次第で削る。 | **reuse candidate**（Node/TSの場合） | high |
| [`garriguv/SQLiteMigrationManager.swift`](https://github.com/garriguv/SQLiteMigrationManager.swift) | [`SQLiteMigrationManager.swift`](https://github.com/garriguv/SQLiteMigrationManager.swift/blob/c46cc4566ea3d2bb73d34fe620131f0f64589a2d/Sources/SQLiteMigrationManager.swift#L1-L33) はversion順にmigrationをsort、`schema_migrations` tableを定義。`migrateDatabase`（同#L164-L179）はmigrationごとにtransaction内でschema変更とversion insertを行う（direct）。 | Swift Package、SQLite.swift 0.15.3以上（[`Package.swift`](https://github.com/garriguv/SQLiteMigrationManager.swift/blob/c46cc4566ea3d2bb73d34fe620131f0f64589a2d/Package.swift#L1-L22)）。設計は他言語へ移せるがAPIはSwift依存。 | [`LICENSE`](https://github.com/garriguv/SQLiteMigrationManager.swift/blob/main/LICENSE) はMIT（direct）。 | test targetあり。[`SQLiteMigrationManagerTests.swift`](https://github.com/garriguv/SQLiteMigrationManager.swift/blob/c46cc4566ea3d2bb73d34fe620131f0f64589a2d/Tests/SQLiteMigrationManagerTests.swift#L255-L280) は成功migrationと失敗rollbackを確認（direct）。最新観測commit [`c46cc45`](https://github.com/garriguv/SQLiteMigrationManager.swift/commit/c46cc4566ea3d2bb73d34fe620131f0f64589a2d) はSQLite checkout更新（date unknown）。 | language-agnosticな最小設計の参照。integration cost: medium（Swift APIを他言語へ翻訳）。 | **algorithm/reference only** | high |
| [`pressly/goose`](https://github.com/pressly/goose) | SQLite dialectは`version_id`/`is_applied`/timestampを持つversion tableを作成（[`sqlite3.go`](https://github.com/pressly/goose/blob/16922709fc608486f866e6a8770ca60cffba9940/internal/dialects/sqlite3.go#L18-L50)、direct）。[`provider_run.go`](https://github.com/pressly/goose/blob/16922709fc608486f866e6a8770ca60cffba9940/provider_run.go#L124-L168) は全migrationを先にparseし、順番に個別適用、失敗時`PartialError`。同#L203-L219はtransaction内のmigration+version update（direct）。 | Go 1.25、SQLiteを含む多数DB/adapter依存（[`go.mod`](https://github.com/pressly/goose/blob/16922709fc608486f866e6a8770ca60cffba9940/go.mod#L1-L25)）。小規模gameにはdependency/運用コストが大きい。 | [`LICENSE`](https://github.com/pressly/goose/blob/main/LICENSE) はMIT（direct）。 | [`provider_run_test.go`](https://github.com/pressly/goose/blob/16922709fc608486f866e6a8770ca60cffba9940/provider_run_test.go#L60-L136) はup/down/by-one/version整合を検証（direct）。非archive。最新観測commit [`1692270`](https://github.com/pressly/goose/commit/16922709fc608486f866e6a8770ca60cffba9940) はMySQL locker実装（date unknown）。 | frameworkのトランザクション境界、順序、partial failureの設計。integration cost: high（multi-DB dependency/CLI/運用層）。全frameworkは採用しない。 | **partial adaptation** | high |
| [`golang-migrate/migrate`](https://github.com/golang-migrate/migrate) | [`database/sqlite3/sqlite3.go`](https://github.com/golang-migrate/migrate/blob/18966c755f83de2b250dd8e766bad8b769cd0e25/database/sqlite3/sqlite3.go#L65-L88) は初回version table、同#L202-L255はqueryとversion更新のtransaction/rollback、`NoTxWrap`を示す（direct）。 | Goのmulti-DB migration framework。`go.mod`は多くのDB adapterを含むため小規模appには過剰（direct）。 | [`LICENSE`](https://github.com/golang-migrate/migrate/blob/master/LICENSE) はMIT（direct）。 | SQLite testはtemp DBでdriver/migrateを検証（[`sqlite3_test.go`](https://github.com/golang-migrate/migrate/blob/18966c755f83de2b250dd8e766bad8b769cd0e25/database/sqlite3/sqlite3_test.go#L17-L54)、direct）。最新観測commit [`18966c7`](https://github.com/golang-migrate/migrate/commit/18966c755f83de2b250dd8e766bad8b769cd0e25) はYugabyte test更新（date unknown）。 | transaction/no-transaction boundaryの比較対象。integration cost: high（multi-DB adapter群）。 | **algorithm/reference only** | medium |
| [`ruckus/ruckusing-migrations`](https://github.com/ruckus/ruckusing-migrations) | READMEはPHP5 migration toolとSQLite対応を説明するが、今回のschema/transaction sourceを深掘りしなかった（direct/unknown）。 | PHP5前提で現行desktop/game stackとの互換性が低い（direct/inferred）。 | `LICENSE` file fetchが404で、ライセンス不明（direct）。 | legacy signal、テスト/CI/maintenanceはunknown。 | 不明licenseかつlegacyのためコードは使わない。integration cost: high/unknown。 | **reject** | high |

### Evidence to read

最終reading listは9ファイル（10ファイル上限内）。migration順序、初回作成、transaction、failure、testsを直接追えるものに絞った。

1. `kriasoft/node-sqlite@f86c213:package.json` — TypeScript migration API、MIT、test/CI scripts。 [direct](https://github.com/kriasoft/node-sqlite/blob/f86c213863353d3701d71d8993975a04138dbe99/package.json#L1-L100)
2. `kriasoft/node-sqlite@f86c213:src/utils/migrate.ts` — numeric order、metadata table、up/down、BEGIN/COMMIT/ROLLBACK。 [direct](https://github.com/kriasoft/node-sqlite/blob/f86c213863353d3701d71d8993975a04138dbe99/src/utils/migrate.ts#L10-L137)
3. `kriasoft/node-sqlite@f86c213:src/utils/__tests__/migrate.test.ts` — in-memory DBでapply/forceと結果を検証。 [direct](https://github.com/kriasoft/node-sqlite/blob/f86c213863353d3701d71d8993975a04138dbe99/src/utils/__tests__/migrate.test.ts#L9-L67)
4. `garriguv/SQLiteMigrationManager.swift@c46cc45:Package.swift` — Swift/SQLite.swift dependencyとtest target。 [direct](https://github.com/garriguv/SQLiteMigrationManager.swift/blob/c46cc4566ea3d2bb73d34fe620131f0f64589a2d/Package.swift#L1-L22)
5. `garriguv/SQLiteMigrationManager.swift@c46cc45:Sources/SQLiteMigrationManager.swift` — sorted migrations、初回table、pending、transaction。 [direct](https://github.com/garriguv/SQLiteMigrationManager.swift/blob/c46cc4566ea3d2bb73d34fe620131f0f64589a2d/Sources/SQLiteMigrationManager.swift#L27-L179)
6. `garriguv/SQLiteMigrationManager.swift@c46cc45:Tests/SQLiteMigrationManagerTests.swift` — order、pending、成功、失敗rollback。 [direct](https://github.com/garriguv/SQLiteMigrationManager.swift/blob/c46cc4566ea3d2bb73d34fe620131f0f64589a2d/Tests/SQLiteMigrationManagerTests.swift#L109-L280)
7. `pressly/goose@1692270:internal/dialects/sqlite3.go` — `version_id/is_applied` tableとlatest version query。 [direct](https://github.com/pressly/goose/blob/16922709fc608486f866e6a8770ca60cffba9940/internal/dialects/sqlite3.go#L18-L50)
8. `pressly/goose@1692270:provider_run.go` — parse-before-run、個別適用、`PartialError`、transaction boundary。 [direct](https://github.com/pressly/goose/blob/16922709fc608486f866e6a8770ca60cffba9940/provider_run.go#L124-L219)
9. `pressly/goose@1692270:provider_run_test.go` — up/down/by-oneとversion整合のテスト。 [direct](https://github.com/pressly/goose/blob/16922709fc608486f866e6a8770ca60cffba9940/provider_run_test.go#L60-L136)

## Recommendation

- **Preferred candidate:** 言語がNode/TypeScriptなら `kriasoft/node-sqlite` を **reuse candidate** として採用する。言語未確定またはゲームの小さなsave runnerなら、`SQLiteMigrationManager.swift` と `goose` の境界をもとにした **partial adaptation**（小さな自前runner）を推奨する。
- **Why it fits:** 初回`CREATE TABLE IF NOT EXISTS`、migration filename/versionの決定的順序、migration本体とversion記録を同じtransactionに入れる構造、失敗時rollback、in-memory testが揃っている。gooseはmigrationを先にparseしてから順に適用し、失敗時に適用済み/失敗を返すため、起動時の部分適用を可視化する設計も参考になる。
- **Reuse boundary:** `schema_migrations(version, name)`を最小のsource of truthにし、`Migration {version, up}`を昇順に一度ずつ適用する。各migrationは `BEGIN → schema change → version insert → COMMIT`、例外は `ROLLBACK`して起動を止める。必要ならchecksumを追加するが、既存saveとの互換規則を先に定義する。
- **Do not import:** goose/golang-migrateの全multi-DB dependency、CLI/locking/driver層、SwiftのSQLite.swift API、node-sqliteのfull SQL (`up/down`)を無条件に持ち込まない。down migrationを本番の自動rollback手段とみなさず、backup/restoreとforward-only policyを別に決める。不明licenseのRuckusingからコードをコピーしない。
- **License/attribution obligations:** node-sqlite、SQLiteMigrationManager、goose、golang-migrateは確認したLICENSE/package metadataでMIT。直接コードを移植する場合は通知を保持する。Ruckusingはlicense不明なので再利用を推奨しない。法的断定ではなく、調査時のrepository evidenceである。
- **Compatibility risks or open questions:** SQLite DDLのtransaction挙動、同時起動、save file破損、巨大migrationの時間、schema checksumの導入、旧versionから飛び級したときのテストはアプリ固有。backupを作ってからmigrationし、途中失敗後に再実行できることを実機/desktopで確認する。
- **Next implementation step:** version 0のDB作成、version 1/2の小さなfixture、失敗するversion 3を用意し、`applyAll`の順序・rollback・再実行・既存save読み出しをテストする。Node/TSならnode-sqliteを薄くwrapし、他言語なら同じcontractを実装する。
- **Stop condition:** 初回作成、順序、transaction rollback、失敗後の再実行、旧save fixtureの5ケースが通れば探索終了。checksum/backup仕様が未決定なら実装前に設計判断を残すが、追加OSS探索は行わない。

## FIELD_NOTES.md entry

```markdown
### OSS discovery — 2026-08-23
- repo/ref: `kriasoft/node-sqlite@f86c213` / `garriguv/SQLiteMigrationManager.swift@c46cc45`
- license evidence: `package.json` / `LICENSE` — MIT
- useful pattern: deterministic numeric order; create metadata table on first run; wrap each schema change and version write in one transaction
- adopted boundary: small runner exposes `applyAll/applyTo`, owns only schema_migrations and migration result
- avoided: multi-DB framework dependencies, generated CLI/locking layers, automatic production down migrations, unknown-license code
```
