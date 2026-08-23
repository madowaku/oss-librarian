# OSS Librarian Report: React Drag-Sort UI

## Request and budget

- **Goal:** React + TypeScriptで、カードをmouse/touchでドラッグして並べ替え、確定した順序をstateとして受け取れる実装を比較する。
- **Target stack/version/platform:** React 18/19を第一候補、TypeScript、モバイルブラウザ。keyboard/accessibilityも評価する。
- **Reuse intent:** whole-library reuseとpartial adaptationの両案を比較する。
- **License constraints:** MIT/Apache-2.0/BSDを優先。ライセンスだけで採用を決めず、保守と依存も評価する。
- **Exploration budget:** 5 candidates / 3 deep reads / 10 final files。候補は5件、深掘りは `clauderic/dnd-kit`、`clauderic/react-sortable-hoc`、`SortableJS/react-sortablejs` の3件。追加検索は `dnd kit` の1回だけに限定した。

## Capability brief

- **Capability:** list stateを持つカード一覧、pointer（mouse/touch）drag、keyboard操作、drop後の並び順更新、モバイルでの誤操作抑制。
- **Non-goals:** kanban全体、複数window間のdrag、画像編集、仮想スクロール、デザインシステム。
- **Local integration boundary:** `items: Card[]` と `setItems` をアプリが所有し、並べ替えライブラリには `id/index/ref` とdrag end eventだけを渡す。保存はdrop後のstateを既存APIへ渡す。
- **Search terms and implementation signals:** `react sortable`、`dnd kit`、`useSortable`、`PointerSensor`、`KeyboardSensor`、`onDragEnd`、`setList`、`React 18`、`React 19`、`accessible`。

## Candidate comparison

| Candidate | Capability evidence | Compatibility | License evidence | Quality/maintenance | Scope / integration cost | Decision | Confidence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [`clauderic/dnd-kit`](https://github.com/clauderic/dnd-kit) | README/docsは軽量・accessibleなdrag/dropとpointer/keyboardを説明（corroborated）。[`sensors.mdx`](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/apps/docs/docs/react/guides/sensors.mdx#L11-L18) はPointerSensor（mouse/touch/pen）とKeyboardSensorを既定登録し、touch delay/距離制約とkeyboard keyを説明（direct）。[`sortable-state-management.mdx`](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/apps/docs/docs/react/guides/sortable-state-management.mdx#L40-L87) はdrop後のarray state更新（direct）。 | `packages/react/package.json` は `@dnd-kit/react` 0.5.0、React/React DOM `^18 || ^19` peer、`@dnd-kit/abstract/dom/state`等を依存（direct）。TypeScript source。 | [`packages/react/package.json`](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/packages/react/package.json#L1-L10) のlicenseはMIT（direct）。 | root scriptにbuild/test/lint、[`tests.yml`](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/.github/workflows/tests.yml#L1-L25) はBun install/test（direct）。非archive。最新観測commit [`6fb5783`](https://github.com/clauderic/dnd-kit/commit/6fb57833026e06bb3925eef78316ba56d59749c8) はPR merge（date unknown）。 | sortable hook + input sensors + app-owned state。integration cost: medium（monorepo由来のproviderと複数package）。 | **reuse candidate** | high |
| [`SortableJS/react-sortablejs`](https://github.com/SortableJS/react-sortablejs) | [`ReactSortable`](https://github.com/SortableJS/react-sortablejs/blob/5e6ed8ab656bce48136599d09e7126e8af62bb52/src/react-sortable.tsx#L37-L75) はSortable.createをReact componentに接続し、[`types.ts`](https://github.com/SortableJS/react-sortablejs/blob/5e6ed8ab656bce48136599d09e7126e8af62bb52/src/types.ts#L27-L37) は`list`/`setList`を要求（direct）。READMEのcontrolled list例もcorroborateする。 | React peerは `>=16.9`、SortableJS 1、classnames/tiny-invariant依存（direct）。React 18/19でも動く可能性はあるが、型/保守は別検証（inferred/unknown）。 | [`package.json`](https://github.com/SortableJS/react-sortablejs/blob/5e6ed8ab656bce48136599d09e7126e8af62bb52/package.json#L1-L13) はMIT（direct）。 | README自身が「not production-ready / bugs」を警告（direct）。packageにbuild/lint/Jest devDependencyはあるが、CI workflow内容はconnectorで取得できずunknown。最新観測commit [`5e6ed8a`](https://github.com/SortableJS/react-sortablejs/commit/5e6ed8ab656bce48136599d09e7126e8af62bb52)（PR merge、date unknown）。 | 完成componentを採用できるが、a11yの直接証拠はdnd-kitほど強くない。integration cost: low-medium（SortableJS 1とpeerを追加、a11yは別検証）。 | **partial adaptation** | medium |
| [`clauderic/react-sortable-hoc`](https://github.com/clauderic/react-sortable-hoc) | READMEはtouch/keyboard対応を説明するが、[`SortableElement`](https://github.com/clauderic/react-sortable-hoc/blob/caf3c4f5bfb30894639391ae75c1bfc2b707a127/src/SortableElement/index.js#L35-L75) は`findDOMNode`で登録（direct）。 | Reactの将来版で`findDOMNode`が壊れる可能性をREADME自身が指摘（direct）。現行React 18/19の採用候補ではない。 | [`LICENSE`](https://github.com/clauderic/react-sortable-hoc/blob/master/LICENSE) はMIT（direct）。 | READMEが「no longer maintained」「dnd-kitへ移行」と明記（direct）。最新観測commit [`caf3c4f`](https://github.com/clauderic/react-sortable-hoc/commit/caf3c4f5bfb30894639391ae75c1bfc2b707a127)（Minor tweak、date unknown）。 | old HOCの登録/manager設計を読むだけ。integration cost: high（legacy HOC/findDOMNodeの置換が必要）。 | **algorithm/reference only** | high |
| [`atlassian/react-beautiful-dnd`](https://github.com/atlassian/react-beautiful-dnd) | READMEはdrag/drop libraryを示すが、archive後のコードを今回のsource evidenceとして採用しない（direct）。 | archived、現行React/TypeScript要件に対する保守保証なし（direct/inferred）。 | [`LICENSE`](https://github.com/atlassian/react-beautiful-dnd/blob/master/LICENSE) はApache-2.0（direct）。 | GitHub metadataがarchived、READMEがdeprecatedを明記（direct）。 | 既存UIへの新規導入対象外。integration cost: high（移行・保守リスク）。 | **reject** | high |
| [`frontend-collective/react-sortable-tree`](https://github.com/frontend-collective/react-sortable-tree) | READMEはtree/hierarchical data専用で、今回のflat card listとはscopeが違う（direct）。 | READMEがnot actively maintained。tree virtualization等が不要な今回には過剰（direct）。 | [`LICENSE`](https://github.com/frontend-collective/react-sortable-tree/blob/master/LICENSE) はMIT（direct）。 | 保守停止の明記、flat listのsource evidenceなし。 | tree-specific APIを持ち込まない。integration cost: high（不要な階層/virtualization）。 | **reject** | high |

### Evidence to read

最終reading listは8ファイル（10ファイル上限内）。`@ref`は調査時のbranchと観測commitを示す。

1. `clauderic/dnd-kit@6fb5783:packages/react/package.json` — MIT、React 18/19 peer、依存境界。 [direct](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/packages/react/package.json#L1-L75)
2. `clauderic/dnd-kit@6fb5783:packages/react/src/sortable/useSortable.ts` — `useSortable({id,index,ref})` とgroup/sensor同期。 [direct](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/packages/react/src/sortable/useSortable.ts#L23-L65)
3. `clauderic/dnd-kit@6fb5783:apps/docs/docs/react/guides/sensors.mdx` — pointer/touch activation、keyboard、interactive element除外。 [direct](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/apps/docs/docs/react/guides/sensors.mdx#L11-L18)
4. `clauderic/dnd-kit@6fb5783:apps/docs/docs/react/guides/sortable-state-management.mdx` — `onDragEnd`/`move`でapp stateを更新。 [direct](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/apps/docs/docs/react/guides/sortable-state-management.mdx#L40-L87)
5. `clauderic/dnd-kit@6fb5783:packages/dom/src/core/sensors/pointer/PointerSensor.ts` — touch delay 250ms、distance/tolerance、interactive element guard。 [direct](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/packages/dom/src/core/sensors/pointer/PointerSensor.ts#L31-L86)
6. `clauderic/dnd-kit@6fb5783:packages/dom/src/core/sensors/keyboard/KeyboardSensor.ts` — Space/Enter/Escape/Arrow key map。 [direct](https://github.com/clauderic/dnd-kit/blob/6fb57833026e06bb3925eef78316ba56d59749c8/packages/dom/src/core/sensors/keyboard/KeyboardSensor.ts#L30-L79)
7. `SortableJS/react-sortablejs@5e6ed8a:src/react-sortable.tsx` — `Sortable.create`とlist更新callbackのadapter。 [direct](https://github.com/SortableJS/react-sortablejs/blob/5e6ed8ab656bce48136599d09e7126e8af62bb52/src/react-sortable.tsx#L37-L75)
8. `SortableJS/react-sortablejs@5e6ed8a:src/types.ts` — `list`/`setList`がstate boundaryになる型。 [direct](https://github.com/SortableJS/react-sortablejs/blob/5e6ed8ab656bce48136599d09e7126e8af62bb52/src/types.ts#L27-L37)

## Recommendation

- **Preferred candidate:** `clauderic/dnd-kit` を **reuse candidate** として採用候補にする。React 18/19、pointer（mouse/touch/pen）、keyboard、state handoffが一つの設計で揃っている。
- **Why it fits:** `useSortable` はid/index/refをReact componentに自然に接続し、sensor docsはtouch delay/距離とkeyboard操作を明示し、state docsはdrop後の配列更新をapp側に残す。accessibilityはKeyboardSensorを標準の設計対象として扱える。
- **Reuse boundary:** `@dnd-kit/react`のsortable hookと必要なprovider/sensorsだけを使い、カードの`items`は既存stateに保持する。drop eventから`move(items, oldIndex, newIndex)`相当のpure functionを呼び、永続化はアプリ側で行う。
- **Do not import:** `react-beautiful-dnd`（archived/deprecated）や`react-sortable-hoc`（findDOMNode/保守停止）を新規導入しない。dnd-kit内部のcollision/sensor実装をコピーせず、READMEだけでa11yを推測しない。依存を避ける場合でも、sortable stateのarray moveパターンだけを参考にする。
- **License/attribution obligations:** dnd-kit/SortableJS/react-sortable-hoc/react-sortable-treeの確認済みライセンスはMIT、react-beautiful-dndはApache-2.0。直接コードを取り込む場合は該当通知を保持する。ライセンスは法的判断ではなく、リポジトリのmetadata/LICENSEを根拠にしている。
- **Compatibility risks or open questions:** dnd-kitは0.5.0のmonorepoで、必要パッケージのtree-shakingとbundleサイズ、SSR、カード内のbutton/input、iOS Safariのtouch delayを実アプリで確認する。`react-sortablejs`のproduction warningも解消されていない（unknown）。
- **Next implementation step:** まずdnd-kitの最小sortable listをReact 19/TypeScriptで実装し、pointer/keyboardの両方で`items`の順序が一度だけ更新されるテストを書く。依存予算が超えた場合のみ、state move関数とsensor境界を部分自前実装する。
- **Stop condition:** React 18/19でmouse/touch/keyboard、drop後state、モバイルの誤起動、a11y smoke testが通れば探索終了。bundle/SSRに問題が出た場合だけ`react-sortablejs`を再評価する。

## FIELD_NOTES.md entry

```markdown
### OSS discovery — 2026-08-23
- repo/ref: `clauderic/dnd-kit@6fb5783`
- license evidence: `packages/react/package.json` — MIT; peer React 18/19
- useful pattern: separate pointer/keyboard sensors from sortable state; keep `items` in app state and update on drag end
- adopted boundary: `useSortable`/sensor wiring and a small state move adapter
- avoided: archived react-beautiful-dnd, unmaintained react-sortable-hoc, copying dnd-kit internals, assuming a11y from README alone
```
