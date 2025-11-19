# Playwright MCP Sample Repository

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Playwright MCP](https://img.shields.io/badge/Playwright%20MCP-v0.0.47-blue)](https://github.com/microsoft/playwright-mcp)

**Playwright MCP (Model Context Protocol)** を使ってClaude Codeからブラウザ自動化を行うための、3つのセットアップ方法のサンプルリポジトリです。

## 📋 目次

- [概要](#概要)
- [3つのセットアップ方法](#3つのセットアップ方法)
- [クイックスタート](#クイックスタート)
- [各方式の詳細](#各方式の詳細)
- [比較表](#比較表)
- [トラブルシューティング](#トラブルシューティング)
- [ライセンス](#ライセンス)

## 概要

このリポジトリには、Playwright MCPを使用してClaude Codeからブラウザを自動操作するための、3つの異なるセットアップ方法が含まれています。

### Playwright MCPとは？

- **公式サポート**: Microsoft Playwright公式チームがメンテナンス
- **構造化アプローチ**: アクセシビリティツリーを使用し、スクリーンショット不要
- **22個のツール**: ナビゲーション、スクリーンショット、要素操作、JavaScript実行など
- **高速・軽量**: ビジョンモデル不要で決定論的な動作

## 3つのセットアップ方法

| 方式 | 推奨度 | セットアップ時間 | 推奨ケース |
|------|--------|-----------------|----------|
| **[npx版](./npx-setup/)** | ⭐⭐⭐⭐⭐ | 5分 | 個人開発、チーム開発、DevContainer |
| **[Docker Compose版](./docker-compose-setup/)** | ⭐⭐⭐⭐ | 5分（初回） | 複数プロジェクト共有 |
| **[Docker直接実行版](./docker-direct-setup/)** | ⭐⭐⭐⭐⭐ | 中程度 | 本番環境、CI/CD |

## クイックスタート

### 方法1: npx版（最も簡単・推奨）

#### パターンA: DevContainerを使用（Node.js環境がない場合）

```bash
# 1. このディレクトリに移動
cd npx-setup

# 2. VSCodeで開く
code .

# 3. 「コンテナーで再度開く」をクリック
#    または Cmd/Ctrl+Shift+P → "Dev Containers: Reopen in Container"

# 4. 自動的にNode.js環境とChromiumがインストールされます（初回のみ約1分）

# 5. VSCodeをリロード（Cmd/Ctrl+Shift+P → "Developer: Reload Window"）

# 6. Claude Codeで試す
# "Playwright MCPを使用して、https://example.com を開いてページタイトルを取得してください"
```

**所要時間**: 約5分（初回）、約30秒（2回目以降）

#### パターンB: ローカル環境を使用（Node.js v20+がある場合）

```bash
# 1. このディレクトリに移動
cd npx-setup

# 2. Chromiumをインストール
npx playwright install chromium

# 3. VSCodeを開く
code .

# 4. VSCodeをリロード（Cmd/Ctrl+Shift+P → "Developer: Reload Window"）

# 5. Claude Codeで試す
# "Playwright MCPを使用して、https://example.com を開いてページタイトルを取得してください"
```

**所要時間**: 約5分

### 方法2: Docker Compose版（複数プロジェクト向け）

```bash
# 1. このディレクトリに移動
cd docker-compose-setup

# 2. Docker Composeで起動
docker compose up -d

# 3. VSCodeを開く
code .

# 4. VSCodeをリロード

# 5. Claude Codeで試す
```

**所要時間**: 約5分（初回）、約10秒（2回目以降）

### 方法3: Docker直接実行版（本番/CI/CD向け）

```bash
# 1. このディレクトリに移動
cd docker-direct-setup

# 2. VSCodeを開く
code .

# 3. VSCodeをリロード

# 4. Claude Codeで試す
# 初回は自動的にDockerイメージがダウンロードされます
```

## 各方式の詳細

### 📁 [npx-setup/](./npx-setup/)

**特徴:**
- ✅ インストール不要（npx経由で実行）
- ✅ 高速・安定（< 1秒で起動）
- ✅ 設定ファイル1つ（`.mcp.json`のみ）
- ✅ チーム共有が簡単
- ✅ **DevContainer対応**（Node.js環境不要）

**含まれるファイル:**
- `.mcp.json` - MCP設定（検証済み）
- `.devcontainer/` - DevContainer設定（Node.js自動セットアップ）
- `README.md` - 詳細セットアップガイド

### 🐳 [docker-compose-setup/](./docker-compose-setup/)

**特徴:**
- ✅ 複数プロジェクトで共有可能
- ✅ 常駐型（起動後は高速）
- ✅ 環境が完全に分離
- ✅ Docker Composeで管理

**含まれるファイル:**
- `compose.yml` - Docker Compose設定
- `.mcp.json` - MCP設定
- `README.md` - 詳細セットアップガイド

### 🔒 [docker-direct-setup/](./docker-direct-setup/)

**特徴:**
- ✅ 最もセキュア（セッション毎に独立）
- ✅ リソース効率的（使用時のみ起動）
- ✅ 常に最新イメージを使用
- ✅ 公式推奨の方式

**含まれるファイル:**
- `.mcp.json` - MCP設定（Docker直接実行）
- `README.md` - 詳細セットアップガイド

## 比較表

### 基本情報

| 項目 | npx版 | Docker Compose版 | Docker直接実行版 |
|------|-------|-----------------|-----------------|
| Node.js必須 | ✅ | ❌ | ❌ |
| Docker必須 | ❌ | ✅ | ✅ |
| セットアップ時間 | 5分 | 5分（初回）、10秒（2回目以降） | 中程度 |
| ディスク使用量 | 約300MB | 約1GB | 約1GB |
| メモリ消費 | 低 | 中〜高（常駐） | 低（使用時のみ） |

### パフォーマンス（実測値）

| 操作 | npx版 | Docker Compose版 |
|------|-------|-----------------|
| ブラウザ起動 | 1-2秒 | 1-2秒 |
| example.com アクセス | < 1秒 | 27-94秒（ばらつき） |
| スクリーンショット | < 1秒 | < 1秒 |

### 推奨ケース

| ユースケース | 推奨方式 |
|------------|---------|
| 個人開発（単一プロジェクト） | **npx版** ⭐⭐⭐⭐⭐ |
| チーム開発 | **npx版** ⭐⭐⭐⭐⭐ |
| 複数プロジェクト共有 | **Docker Compose版** ⭐⭐⭐⭐ |
| 本番環境 | **Docker直接実行版** ⭐⭐⭐⭐⭐ |
| CI/CD | **Docker直接実行版** ⭐⭐⭐⭐⭐ |
| DevContainer環境 | **npx版** ⭐⭐⭐⭐⭐ |

## トラブルシューティング

### よくある問題

#### 問題1: プロファイルロックエラー（npx版）

**エラーメッセージ:**
```
Error: Browser is already in use for /home/node/.cache/ms-playwright/mcp-chrome-*
```

**解決策:**
1. `.mcp.json`に`--isolated`フラグが含まれているか確認
2. VSCodeをリロード
3. 必要に応じてロックファイルを削除:
   ```bash
   rm -rf ~/.cache/ms-playwright/mcp-chrome-*
   ```

#### 問題2: MCPサーバーが認識されない

**解決策:**
1. `.mcp.json`の構文を確認（JSONバリデーター使用）
2. ファイルがプロジェクトルートにあることを確認
3. VSCodeをリロード（Cmd/Ctrl+Shift+P → "Developer: Reload Window"）
4. Claude CodeのOutputパネルでエラーログを確認

#### 問題3: Chromiumが起動しない（npx版）

**解決策:**
```bash
npx playwright install chromium
```

#### 問題4: Dockerコンテナが起動しない（Docker Compose版）

**解決策:**
```bash
# コンテナの状態を確認
docker compose ps

# ログを確認
docker compose logs playwright-mcp

# 再起動
docker compose restart
```

### さらなるヘルプ

- [Playwright公式ドキュメント](https://playwright.dev)
- [Playwright MCP GitHub](https://github.com/microsoft/playwright-mcp)
- [Issues](https://github.com/microsoft/playwright-mcp/issues)

## 利用可能なツール（22個）

### 検証済みツール

- ✅ `browser_navigate` - ページ遷移
- ✅ `browser_snapshot` - ページ構造取得
- ✅ `browser_take_screenshot` - スクリーンショット
- ✅ `browser_console_messages` - コンソールログ取得
- ✅ `browser_evaluate` - JavaScript実行
- ✅ `browser_install` - ブラウザインストール

### その他のツール

**要素操作:** `browser_click`, `browser_type`, `browser_fill_form`, `browser_hover`, `browser_drag`, `browser_select_option`, `browser_press_key`

**高度な機能:** `browser_tabs`, `browser_navigate_back`, `browser_handle_dialog`, `browser_file_upload`, `browser_wait_for`, `browser_resize`, `browser_close`, `browser_run_code`, `browser_network_requests`

## 使用例

### 基本的なページアクセス

```
Playwright MCPを使用して、https://example.com を開いてページタイトルを取得してください
```

### スクリーンショットの取得

```
https://playwright.dev を開いて、スクリーンショットを "screenshot.png" として保存してください
```

### 要素の検索と取得

```
https://github.com を開いて、メインの見出しテキストを取得してください
```

### JavaScript実行

```
https://example.com を開いて、JavaScriptでページの背景色を赤に変更してください
```

## バージョン情報

- **Playwright MCP**: v0.0.47（2025-11-14公開）
- **検証環境**: Node.js v20.19.5, Docker v28.0.2
- **検証日**: 2025-11-17〜18

## 参考リンク

- [Playwright MCP GitHub](https://github.com/microsoft/playwright-mcp)
- [npm パッケージ](https://www.npmjs.com/package/@playwright/mcp)
- [Playwright公式サイト](https://playwright.dev)
- [MCP仕様](https://spec.modelcontextprotocol.io/)

## ライセンス

このサンプルリポジトリはMITライセンスの下で公開されています。

Playwright MCPは[Apache-2.0ライセンス](https://github.com/microsoft/playwright-mcp/blob/main/LICENSE)です。

## 貢献

バグ報告や改善提案は歓迎します。Issueを作成してください。

---

**最終更新**: 2025-11-19
**作成者**: Claude Code
**検証済み**: ✅ すべてのセットアップ方法が動作確認済み
