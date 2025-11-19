# DevContainer セットアップガイド

このディレクトリにはPlaywright MCP npx版を試すためのDevContainer設定が含まれています。

## 前提条件

- VSCode がインストールされている
- Dev Containers 拡張機能がインストールされている
- Docker Desktop が起動している

## 使い方

### 方法1: VSCodeで開く

1. このディレクトリ（`npx-setup/`）をVSCodeで開く
2. VSCodeが「コンテナーで再度開く」を提案するので、クリック
3. または、`Cmd/Ctrl+Shift+P` → "Dev Containers: Reopen in Container"

### 方法2: コマンドラインから

```bash
# このディレクトリに移動
cd playwright-mcp-samples/npx-setup

# VSCodeで開く
code .

# VSCodeでコンテナを開く
# Cmd/Ctrl+Shift+P → "Dev Containers: Reopen in Container"
```

## 初回起動時の処理

DevContainerが起動すると、以下が自動的に実行されます：

1. Node.js v20環境の構築
2. Chromiumブラウザのインストール（約174MB、30秒程度）

**初回起動ログ例:**
```
Running the postCreateCommand from devcontainer.json...

> npx playwright install chromium

Downloading Chromium 141.0.7390.37 (playwright build v1194)...
173.4 MiB [====================] 100% 0.0s
Chromium 141.0.7390.37 downloaded to /home/node/.cache/ms-playwright/chromium-1194
```

## セットアップ完了後

### 1. MCP設定の確認

`.mcp.json` がプロジェクトルートに既に配置されています。

```bash
# 確認
cat .mcp.json
```

### 2. VSCodeをリロード

```
Cmd/Ctrl + Shift + P → "Developer: Reload Window"
```

### 3. Claude Codeで動作確認

Claude Codeのチャットで以下を試してください：

```
Playwright MCPを使用して、https://example.com を開いてページタイトルを取得してください
```

**期待される結果:**
```
✅ ページURL: https://example.com
✅ ページタイトル: "Example Domain"
✅ エラーなし
```

## トラブルシューティング

### 問題1: DevContainerが起動しない

**解決策:**
- Docker Desktop が起動しているか確認
- VSCodeの Dev Containers 拡張機能がインストールされているか確認
- Docker のリソース割り当てを確認（最低2GB推奨）

### 問題2: Chromiumインストールに失敗

**解決策:**
```bash
# DevContainer内で手動実行
npx playwright install chromium

# インストール状態を確認
ls -la ~/.cache/ms-playwright/
```

### 問題3: MCPサーバーが認識されない

**解決策:**
1. `.mcp.json` がプロジェクトルート（`/workspaces/npx-setup/`）にあることを確認
2. VSCodeをリロード
3. Claude CodeのOutputパネルでエラーログを確認

## 環境情報

### 含まれるもの

- **Node.js**: v20.x
- **npm**: v10.x
- **OS**: Debian-based Linux
- **ユーザー**: node

### インストールされるもの

- Playwright Chromium: 約174MB
- Playwright Headless Shell: 約104MB（必要に応じて）

### ディスク使用量

- DevContainerイメージ: 約1GB
- Chromium: 約174MB
- npm cache: 数十MB
- **合計**: 約1.2GB

## カスタマイズ

### 他のブラウザを追加

`devcontainer.json` の `postCreateCommand` を編集:

```json
{
  "postCreateCommand": "npx playwright install chromium firefox webkit"
}
```

### Git設定を追加

```json
{
  "features": {
    "ghcr.io/devcontainers/features/git:1": {}
  }
}
```

### Python環境を追加

```json
{
  "features": {
    "ghcr.io/devcontainers/features/python:1": {
      "version": "3.11"
    }
  }
}
```

## 参考リンク

- [Dev Containers 公式ドキュメント](https://code.visualstudio.com/docs/devcontainers/containers)
- [devcontainer.json リファレンス](https://containers.dev/implementors/json_reference/)
