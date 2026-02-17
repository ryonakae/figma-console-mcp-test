# ZenBalance - パーソナライズド・ウェルネス・ダッシュボード

## プロジェクト概要

iOS向け（iPhone 16 フレーム）の Hi-Fi モバイルアプリプロトタイプを Figma 上に作成するプロジェクト。
Figma Console MCP（Local Mode）を使ってデザインを生成・管理する。

- **アプリ名**: ZenBalance
- **コンセプト**: 健康データ（睡眠・活動量・マインドフルネス）を集約し、AIがウェルビーイング向上の助言を行う
- **デザインの方向性**: 穏やか・信頼感・ミニマル
- **ターゲット**: iPhone 16（393 x 852pt）

---

## デザインシステム（デザイントークン）

### カラーパレット

| トークン名 | 値 | 用途 |
|-----------|-----|------|
| Primary | `#4A6C6F` | 主要ボタン、アクティブ状態（セージグリーン） |
| Secondary | `#E8F1F2` | 背景色（ペールミント） |
| Accent | `#F4A261` | 通知・アラート（ソフトオレンジ） |
| Text/Main | `#2D3436` | 見出し（ダークスレート） |
| Text/Body | `#636E72` | 本文（グレー） |
| Surface | `#FFFFFF` | カード背景 |

### タイポグラフィ

| スタイル | フォント | サイズ | ウェイト |
|---------|---------|--------|---------|
| H1 | SF Pro Display | 24px | Bold |
| H2 | SF Pro Display | 20px | Bold |
| Body | SF Pro Text | 16px | Regular |
| Caption | SF Pro Text | 12px | Medium |

### スペーシング & レイアウト

- **グリッド**: 4カラム、マージン 16px、ガター 16px
- **コンテナ内余白**: 16px
- **カード内余白**: 16px
- **カードの角丸**: 16px
- **ボタンの角丸**: 999px（完全な角丸）

---

## コアコンポーネント

| コンポーネント | レイヤー名 | 用途 |
|-------------|-----------|------|
| メトリックカード | `Card_Metric` | 睡眠・歩数・カロリー等の個別指標表示（正方形） |
| デイリースコアサークル | `Circle_DailyScore` | 総合ウェルネススコア（0-100）の円形プログレスバー |
| ボトムナビゲーション | `Nav_Bottom` | ホーム・分析・AIコーチ・設定の4タブ（下部固定） |
| アクションボタン | `Btn_Primary` | 横幅いっぱいのプライマリーボタン（白文字） |

コンポーネントの nodeId はセッション固有のため、使用前に必ず `figma_search_components` で検索すること。

### コンポーネント作成のワークフロー（必須）

新しいコンポーネントを作成する前に、**必ず既存コンポーネントを確認し、再利用できるものがないかを探すこと**。

1. **既存コンポーネントの確認**: `figma_search_components` で類似コンポーネントを検索する
2. **再利用の検討**: 既存コンポーネントをそのまま、またはバリアント・プロパティを調整して使えないか検討する
3. **新規作成は最終手段**: 既存コンポーネントで要件を満たせない場合のみ新規作成する

```javascript
// 例: ボタンコンポーネントが必要な場合
// まず既存コンポーネントを検索
const results = await figma_search_components({ query: 'button' });
// → 既存の Btn_Primary が見つかればそれを使用（figma_instantiate_component）
// → 見つからない場合のみ新規作成
```

### iOS 標準コンポーネント

**制約事項**: **iOS and iPadOS 26** ライブラリは Figma の GUI 上では参照できるが、Plugin API（figma_search_components 等）からはアクセスできない。

#### 運用方針

| 要素の種類 | 対応方法 |
|-----------|---------|
| ステータスバー、ホームインジケーター、アラートダイアログ、ナビゲーションバーなど、明らかに iOS ネイティブ UI を使うべき要素 | **ユーザーに依頼**：GUI から iOS and iPadOS 26 ライブラリのコンポーネントを現在の Figma ドキュメントにコピー＆ペーストしてもらう。その後、`figma_search_components` で検索して使用する |
| それ以外の UI 要素（カード、ボタン、メトリック表示、ナビゲーション等） | **自作する**：本ドキュメントのデザインシステムに従って実装する |

#### ユーザーへの依頼例（iOS ネイティブコンポーネントが必要な場合）

> iOS のステータスバーコンポーネントが必要です。Figma の「Assets」パネルから「iOS and iPadOS 26」ライブラリを開き、該当コンポーネントを現在のページにコピー＆ペーストしていただけますか？

---

## 生成ルール

### バリアブル・スタイルの使用（必須）

デザイン要素には、ファイル内の既存バリアブルとテキストスタイルを優先的に使用する。
ハードコードされた色値や数値を直接指定してはならない。

- **色（カラー変数）**: `ZenBalance/Colors` コレクションのバリアブルを使用
- **数値（スペーシング・角丸・フォントサイズ）**: `ZenBalance/Numbers` コレクションのバリアブルを使用
- **テキストスタイル**: `ZenBalance/H1`〜`ZenBalance/Caption` のスタイルを使用

既存のバリアブル・スタイルで実現できない場合（例: スコア表示用の 36px フォントなど）のみ新規作成する。

```javascript
// カラーバリアブルのバインド（fills）
const colorVar = await figma.variables.getVariableByIdAsync('VariableID:...');
const newFills = [...node.fills];
newFills[0] = figma.variables.setBoundVariableForPaint(newFills[0], 'color', colorVar);
node.fills = newFills;

// 数値バリアブルのバインド（cornerRadius, padding, fontSize 等）
const numVar = await figma.variables.getVariableByIdAsync('VariableID:...');
node.setBoundVariable('topLeftRadius', numVar);
node.setBoundVariable('paddingTop', numVar);
node.setBoundVariable('fontSize', numVar);

// テキストスタイルのバインド（非同期）
await textNode.setTextStyleIdAsync('S:...');
```

バリアブル ID はセッション内で `figma.variables.getLocalVariablesAsync()` で取得する（MCP の `figma_get_variables` ツールは動作しない場合あり）。

### Auto Layout（必須）

すべてのフレーム・コンポーネントに Auto Layout を適用する。
固定サイズの手動配置（絶対 x/y 指定）は禁止。

#### サイジング設定のルール（重要）

`layoutSizingHorizontal` / `layoutSizingVertical` は **親に追加した後に設定すること**。
親の Auto Layout フレームに appendChild する前に FILL を設定するとエラーになる。

```javascript
// NG: 親に追加する前に FILL を設定するとエラー
frame.layoutSizingHorizontal = 'FILL'; // Error: FILL can only be set on children of auto-layout frames
parent.appendChild(frame);

// OK: 親に追加してから設定する
parent.appendChild(frame);
frame.layoutSizingHorizontal = 'FILL';
```

#### `primaryAxisSizingMode` と `layoutSizingHorizontal` の競合に注意

HORIZONTAL フレームで `primaryAxisSizingMode = 'AUTO'`（HUG）を設定した後に `layoutSizingHorizontal = 'FILL'` を設定すると、後から設定した方が優先される。意図せず上書きされないよう、`primaryAxisSizingMode` は設定せず `layoutSizingHorizontal` / `layoutSizingVertical` のみで制御するのが安全。

#### 水平・垂直サイジングの独立性

- 幅 FILL ＋ 高さ HUG の組み合わせが頻出パターン
- **横方向 FILL にしたからといって縦方向が自動的に HUG になるわけではない**
- 必ず両軸を明示的に設定すること

```javascript
row.layoutSizingHorizontal = 'FILL'; // 幅は親に追従
row.layoutSizingVertical = 'HUG';    // 高さはコンテンツに合わせる（明示必須）
```

#### Components ページのレイアウト管理

新しいコンポーネントを Core Components ページに作成する際も、絶対座標（`x/y`）で配置せず Auto Layout で管理する。

- Section 内に配置用の Auto Layout フレーム（`ComponentLayout`）を作成し、そこにコンポーネントを追加する
- コンポーネント同士が重ならないよう、`itemSpacing` で自動的に間隔を確保する

```javascript
// Section 内の Auto Layout フレームにコンポーネントを追加
let layoutFrame = section.findOne(n => n.name === 'ComponentLayout');
if (!layoutFrame) {
  layoutFrame = figma.createFrame();
  layoutFrame.name = 'ComponentLayout';
  layoutFrame.layoutMode = 'VERTICAL';
  layoutFrame.itemSpacing = 48;
  layoutFrame.paddingTop = 32;
  layoutFrame.paddingRight = 32;
  layoutFrame.paddingBottom = 32;
  layoutFrame.paddingLeft = 32;
  layoutFrame.fills = [];
  layoutFrame.primaryAxisSizingMode = 'AUTO';
  layoutFrame.counterAxisSizingMode = 'AUTO';
  section.appendChild(layoutFrame);
}
layoutFrame.appendChild(newComponent);
```

### レイヤー命名規約

すべてのレイヤーに明確な名前をつける。デフォルト名（Frame 1、Rectangle 2 等）は使わない。

- コンポーネント: `Btn_Primary`, `Card_Metric`, `Nav_Bottom`
- 画面セクション: `Home/Header`, `Home/Content/MetricGrid`
- テキスト: `Text_Greeting`, `Label_SleepValue`

### アイコン

アイコンには **Material Icons フォント**（Google Fonts）を使用する。

> **注意**: テキストノードを作成する前に、Figma のデフォルトフォント **Inter Regular** も必ずロードすること。ロードしないと `Cannot write to node with unloaded font "Inter Regular"` エラーが発生する。

```javascript
// フォントのロード（Inter を忘れずに）
await figma.loadFontAsync({ family: "Inter", style: "Regular" }); // デフォルトフォント（必須）
await figma.loadFontAsync({ family: "Material Icons", style: "Regular" });

// テキストノードとして配置（ligature 名を characters に指定）
const icon = figma.createText();
icon.fontName = { family: "Material Icons", style: "Regular" };
icon.fontSize = 24;
icon.characters = 'home';  // Material Icons の ligature 名
```

主なアイコン対応表（ZenBalance で使用するもの）:

| 用途 | ligature 名 |
|------|------------|
| ホーム | `home` |
| 分析 | `bar_chart` |
| AIコーチ | `smart_toy` |
| 設定 | `settings` |
| 睡眠 | `bedtime` |
| 歩数・活動 | `directions_walk` |
| 心拍 | `favorite` |
| 水分 | `water_drop` |
| カロリー | `local_fire_department` |

### データ

リアルなデータを使用する。ダミーテキスト（Lorem ipsum）や空のプレースホルダーは禁止。

例: 「7h 32m」「8,432歩」「82 bpm」「水分 2.1L」

---

## Figma ページ構成

### ページの役割

| ページ名 | 用途 |
|---------|------|
| **Design** | すべての画面デザインを作成・管理するページ |
| **Components** | コアコンポーネントを管理するページ |

### ルール

- **画面デザインは「Design」ページ（[リンク](https://www.figma.com/design/A2QrFNxjE4zk4WvTCTsOTK/figma-console-mcp-test?node-id=1-65)）に集約する**
  - 機能ごとにページを分割しない（現状は画面数が少ないため不要）
  - 将来的に画面数が増えた場合はページ分割を検討する
- **コンポーネントは「Components」ページで管理する**
  - 新規コンポーネントは必ず「Components」ページに作成する
  - 「Design」ページへの直接配置はインスタンスのみ

---

## ツールとワークフロー

Figma Console MCP（Local Mode、56+ツール）を使用する。

- セッション手順・検証フロー: [`docs/figma-mcp-workflows.md`](docs/figma-mcp-workflows.md)
- ツール選択ガイド: [`docs/figma-mcp-tool-guide.md`](docs/figma-mcp-tool-guide.md)
- 公式ツールパラメータ仕様: `reference/figma-console-mcp/docs/tools.md`
- ユースケース別プロンプト例: `reference/figma-console-mcp/docs/use-cases.md`
