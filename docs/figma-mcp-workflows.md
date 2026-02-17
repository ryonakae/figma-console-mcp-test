# Figma Console MCP ワークフローガイド

Figma Console MCP を使ったデザイン作業の標準ワークフロー。

---

## セッション開始時の必須手順

1. `figma_get_status` で接続確認（`setup.valid: true` を確認）
2. コンポーネントを使う場合は `figma_search_components` で毎回検索する（nodeId はセッション固有で無効になる）
3. ページ作成前は重複チェックを行う

```javascript
// ページ重複チェックパターン
await figma.loadAllPagesAsync();
const existing = figma.root.children.find(p => p.name === 'PageName');
```

---

## ビジュアル検証ワークフロー（必須）

デザイン要素を作成・変更したら必ずこのフローを実行する：

1. **CREATE** — `figma_execute` でデザインコードを実行
2. **SCREENSHOT** — `figma_take_screenshot` または `figma_capture_screenshot` で結果を撮影
3. **ANALYZE** — アライメント・スペース・プロポーション・バランスを確認
4. **ITERATE** — 問題があれば修正して繰り返す（最大3回）
5. **VERIFY** — 最終スクリーンショットで確認

### スクリーンショットの対象粒度（重要）

個別ノードのスクリーンショットだけでは、他の要素との重なり・余白・配置関係は検出できない。
目的に応じて適切な粒度のスクリーンショットを撮ること。

| 確認したいこと | スクリーンショット対象 | 方法 |
|-------------|-------------------|------|
| 要素の内部構造（テキスト、色、余白） | 作成したノード単体 | `figma_capture_screenshot(nodeId: 'ノードID')` |
| 兄弟要素との位置関係（重なり、余白、アライン） | **親フレーム** | `figma_capture_screenshot(nodeId: '親フレームID')` |
| 画面全体の構成・バランス | **画面フレーム全体** | `figma_capture_screenshot(nodeId: '画面フレームID')` |
| Components ページ全体のレイアウト | **ページ全体** | `figma_capture_screenshot()`（nodeId 省略） |

**原則**: 要素を配置・移動した後は、**必ず親フレームまたは画面全体のスクリーンショットも撮り**、周囲の要素との関係を確認する。

### スクリーンショット取得が失敗した場合

`figma_take_screenshot` が 429 Rate Limit エラーで失敗した場合：

1. まず `figma_capture_screenshot`（プラグイン経由）で代替を試みる
2. それも失敗する場合は、**ユーザーに Figma 画面のスクリーンショット撮影を依頼する**

```
スクリーンショットの取得がレート制限でできませんでした。
Figma の画面をスクリーンショットで共有いただけますか？
```

ユーザーから画像を受け取ったら、通常の ANALYZE → ITERATE フローを続ける。

---

## `figma_execute` のベストプラクティス

```javascript
figma_execute({
  code: `
    // 1. Section/Frame内に配置する（浮いたコンポーネントを避ける）
    let section = figma.currentPage.findOne(n => n.type === 'SECTION' && n.name === 'Components');
    if (!section) {
      section = figma.createSection();
      section.name = 'Components';
    }

    // 2. async操作はawaitを使う
    await figma.loadFontAsync({ family: "Inter", style: "Medium" });
    const node = await figma.getNodeByIdAsync("123:456");

    // 3. viewport中心に配置して視認性を確保（画面フレーム等の最上位ノードの初期配置のみ。
    //    Section/Frame の子要素は Auto Layout で配置するため x/y 指定不要）
    frame.x = figma.viewport.center.x;
    frame.y = figma.viewport.center.y;

    // 4. 作成したノードを選択状態にする
    figma.currentPage.selection = [frame];

    // 5. nodeIdなど有用なデータをreturnする（後続操作で使用）
    return { nodeId: frame.id, name: frame.name };
  `
})
```

---

## よくあるレイアウト問題

- **`hug contents` と `fill container` の使い分けミス**: レイアウトが崩れる最多の原因。子要素が親を埋めるべき箇所では `fill container` を使う
- **浮いたコンポーネント**: 必ず Section または Frame の内部に配置する。キャンバスに直接置かない
- **絶対配置の使用**: Auto Layout を使えば絶対配置は不要。固定 x/y 値での配置は避ける
- **要素の重なり（見落とし）**: 個別ノードのスクリーンショットでは他要素との重なりが検出できない。親フレームやページ全体のスクリーンショットで確認する

---

## 詳細リファレンス

- ツール選択ガイド: `docs/figma-mcp-tool-guide.md`
- 公式ツールパラメータ仕様: `reference/figma-console-mcp/docs/tools.md`
- ユースケース別プロンプト例: `reference/figma-console-mcp/docs/use-cases.md`
- Local / Remote モード比較: `reference/figma-console-mcp/docs/mode-comparison.md`
