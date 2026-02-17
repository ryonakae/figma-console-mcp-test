# Figma Console MCP ツール選択ガイド

Figma Console MCP で利用できるツールの選択ガイド。

- **Local Mode**（NPX/Git）: 56+ツール、読み書き可能。Desktop Bridge Plugin が必要
- **Remote Mode**（SSE）: 21ツール、読み取り専用
- 全ツールの完全なパラメータ仕様: `reference/figma-console-mcp/docs/tools.md`

---

## デザイン作成・編集（Local Mode のみ）

| やりたいこと | 使うツール |
|-------------|-----------|
| フレーム・コンポーネント・テキスト・シェイプを作成 | `figma_execute` |
| ノードをリサイズ | `figma_resize_node` |
| ノードを移動 | `figma_move_node` |
| ノードを複製 | `figma_clone_node` |
| ノードを削除 | `figma_delete_node` |
| ノードをリネーム | `figma_rename_node` |
| テキスト内容を変更 | `figma_set_text` |
| 塗り色を変更 | `figma_set_fills` |
| ストロークを変更 | `figma_set_strokes` |
| 子ノードを作成 | `figma_create_child` |

## コンポーネント操作（Local Mode のみ）

| やりたいこと | 使うツール |
|-------------|-----------|
| コンポーネントを名前で探す | `figma_search_components` |
| コンポーネントの詳細を取得 | `figma_get_component_details` |
| コンポーネントをキャンバスに配置 | `figma_instantiate_component` |
| バリアントをグリッド整理・ラベル付け | `figma_arrange_component_set` |
| コンポーネントに説明を追加 | `figma_set_description` |
| コンポーネントプロパティを追加 | `figma_add_component_property` |
| コンポーネントプロパティを編集 | `figma_edit_component_property` |
| コンポーネントプロパティを削除 | `figma_delete_component_property` |

## 変数・デザイントークン管理（Local Mode のみ）

| やりたいこと | 使うツール |
|-------------|-----------|
| 新規コレクション作成 | `figma_create_variable_collection` |
| 変数を1〜2件作成 | `figma_create_variable` |
| 変数を3件以上作成 | `figma_batch_create_variables`（10〜50x高速） |
| 変数を1〜2件更新 | `figma_update_variable` |
| 変数を3件以上更新 | `figma_batch_update_variables`（10〜50x高速） |
| コレクション＋モード＋変数を一括セットアップ | `figma_setup_design_tokens` |
| 変数をリネーム | `figma_rename_variable` |
| 変数を削除 | `figma_delete_variable` |
| コレクションを削除 | `figma_delete_variable_collection` |
| モードを追加（ライト/ダーク等） | `figma_add_mode` |
| モードをリネーム | `figma_rename_mode` |

### 変数管理のルール

- **3件以上の作成**は `figma_batch_create_variables` を使う（個別呼び出しより10〜50倍高速）
- **3件以上の更新**は `figma_batch_update_variables` を使う
- **新規デザインシステム全体のセットアップ**は `figma_setup_design_tokens` で一括実行（モード名で値を指定できる）
- `figma_setup_design_tokens` はコレクション→モード→変数を1回のCDPラウンドトリップで完結

## デザインシステム読み取り（全モード）

| やりたいこと | 使うツール |
|-------------|-----------|
| 変数・デザイントークンを取得 | `figma_get_variables` |
| カラー・テキスト・エフェクトスタイルを取得 | `figma_get_styles` |
| コンポーネントのデータを取得 | `figma_get_component` |
| コンポーネントの実装用スペック＋画像を取得 | `figma_get_component_for_development` |
| コンポーネントの画像だけ取得 | `figma_get_component_image` |
| ファイル構造を取得 | `figma_get_file_data` |
| プラグイン開発向けファイルデータ取得 | `figma_get_file_for_plugin` |
| デザインシステム概要を取得 | `figma_get_design_system_summary` |
| コレクション別のトークン値を取得 | `figma_get_token_values` |

## プラグインデバッグ（全モード）

| やりたいこと | 使うツール |
|-------------|-----------|
| コンソールログを取得 | `figma_get_console_logs` |
| コンソールをリアルタイム監視 | `figma_watch_console` |
| ログバッファをクリア | `figma_clear_console` |
| スクリーンショットを撮る | `figma_take_screenshot` |
| スクリーンショットを撮る（プラグイン経由） | `figma_capture_screenshot` |
| プラグイン（ページ）をリロード | `figma_reload_plugin` |

## デザインとコードのパリティチェック（全モード）

| やりたいこと | 使うツール |
|-------------|-----------|
| Figmaスペックとコードを比較 | `figma_check_design_parity` |
| コンポーネントドキュメントを生成 | `figma_generate_component_doc` |

## コメント（全モード）

| やりたいこと | 使うツール |
|-------------|-----------|
| コメントを取得 | `figma_get_comments` |
| コメントを投稿・ノードにピン留め | `figma_post_comment` |
| コメントを削除 | `figma_delete_comment` |

## ナビゲーション

| やりたいこと | 使うツール |
|-------------|-----------|
| Figma URL を開く | `figma_navigate` |
| 接続状態を確認 | `figma_get_status` |
| Figma Desktop に再接続 | `figma_reconnect` |

---

## 注意事項

- **`figma_delete_*` 系ツールはundo不可**（プログラム的に元に戻せない）
- **`figma_delete_variable_collection` はコレクション内の全変数を削除**する
- `figma_post_comment` で `@mention` を含めてもプレーンテキストとして扱われる（Figma UIのメンション機能とは別物）
- `figma_get_variables` はFigma Enterprise プランが必要。Local Modeなら Desktop Bridge Plugin 経由で無料プランでも取得可能
- **`figma_capture_screenshot`** は nodeId を省略するとページ全体をキャプチャする。変更直後の検証には `figma_take_screenshot`（REST API）より `figma_capture_screenshot`（プラグイン経由）が確実（ステールデータの問題がない）
- **`figma_take_screenshot`** は常に特定ノード単位でレンダリングし、ページ全体の撮影はできない
