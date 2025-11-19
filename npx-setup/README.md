# npx版 セットアップガイド

**Playwright MCP** をnpx経由で実行する最もシンプルな方法です。

## 特徴

- ✅ **インストール不要** - npx経由で直接実行
- ✅ **高速** - 1-2秒でブラウザ起動
- ✅ **安定** - 100%成功率（検証済み）
- ✅ **簡単** - `.mcp.json`のみで設定完了
- ✅ **チーム共有** - Git管理で簡単に配布

## 前提条件

| 項目 | 要件 |
|------|------|
| Node.js | v20以上 |
| npm | v10以上 |
| Claude Code | インストール済み |

**確認方法:**
```bash
node --version  # v20.x.x 以上
npm --version   # 10.x.x 以上
```

**Node.js環境がない場合:**

このディレクトリには**DevContainer設定**が含まれています。VSCodeのDev Containers機能を使用すると、Node.js環境を自動的に構築できます。

詳細は [.devcontainer/README.md](./.devcontainer/README.md) を参照してください。

## セットアップ手順（5分）

### Step 1: `.mcp.json`を配置

このディレクトリにはすでに`.mcp.json`が含まれています。

**設定ファイル:**
- `.mcp.json` - 検証済み設定（最新版使用）

**デフォルトで使用される設定:**
```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--headless",
        "--isolated",
        "--browser",
        "chromium",
        "--no-sandbox"
      ]
    }
  }
}
```

**重要なフラグ:**
- `@latest`: 常に最新版を使用
- `--headless`: DevContainer/Docker環境で必須
- `--isolated`: プロファイルロック問題を回避（必須）
- `--browser chromium`: ブラウザを明示的に指定
- `--no-sandbox`: DevContainer/Docker環境で必須

### Step 2: Chromiumをインストール

```bash
npx playwright install chromium
```

**実行結果例:**
```
Downloading Chromium 141.0.7390.37 (playwright build v1194)...
173.4 MiB [====================] 100% 0.0s
Chromium 141.0.7390.37 downloaded to ~/.cache/ms-playwright/chromium-1194
```

- ダウンロードサイズ: 約174MB
- 所要時間: 約30秒

### Step 3: VSCodeをリロード

MCP設定を反映させるため、VSCodeをリロードします。

**方法1: コマンドパレット**
1. `Cmd/Ctrl + Shift + P`
2. "Developer: Reload Window"

**方法2: VSCodeの再起動**
- VSCodeを閉じて再度開く

### Step 4: 動作確認

Claude Codeのチャットで以下を試してください:

```
Playwright MCPを使用して、https://example.com を開いてページタイトルを取得してください
```

**期待される結果:**
```
✅ ページURL: https://example.com
✅ ページタイトル: "Example Domain"
✅ スナップショット: アクセシビリティツリーが取得される
```

## バージョン管理オプション

デフォルト設定では`@latest`を使用していますが、バージョンを固定したい場合は以下のように変更できます：

**バージョン固定版（本番環境推奨）:**
```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "@playwright/mcp@0.0.47",    // バージョン固定
        "--headless",
        "--isolated",
        "--browser",
        "chromium",
        "--no-sandbox"
      ]
    }
  }
}
```

**メリット:**
- ✅ 予期せぬ動作変更を回避
- ✅ チーム全体で同じバージョンを使用
- ✅ 再現性の確保

## パフォーマンス実測値

| 操作 | 時間 |
|------|------|
| ブラウザ起動（初回） | 1-2秒 |
| ブラウザ起動（2回目以降） | < 1秒 |
| example.com ナビゲーション | < 1秒 |
| playwright.dev ナビゲーション | < 2秒 |
| スクリーンショット取得 | < 1秒 |

## 使用例

### 基本的なページアクセス

```
Playwright MCPを使用して、https://example.com を開いてください
```

### スクリーンショットの取得

```
https://playwright.dev を開いて、スクリーンショットを "screenshot.png" として保存してください
```

**保存先:**
- デフォルト: `.playwright-mcp/screenshot.png`
- カスタム: `screenshots/screenshot.png`

### 複雑なページの操作

```
https://github.com を開いて、以下を実行してください：
1. ページタイトルを取得
2. スクリーンショットを撮影
3. "Sign up"ボタンが存在するか確認
```

### JavaScript実行

```
https://example.com を開いて、h1要素のテキストをJavaScriptで取得してください
```

## トラブルシューティング

### 問題1: プロファイルロックエラー

**エラーメッセージ:**
```
Error: Browser is already in use for /home/node/.cache/ms-playwright/mcp-chrome-*
use --isolated to run multiple instances of the same browser
```

**原因:**
- `.mcp.json`に`--isolated`フラグが含まれていない
- または、既存のプロファイルがロックされている

**解決策:**
1. `.mcp.json`に`--isolated`フラグを追加（既に追加済みの場合はスキップ）
2. ロックファイルを削除:
   ```bash
   rm -rf ~/.cache/ms-playwright/mcp-chrome-*
   ```
3. VSCodeをリロード

### 問題2: MCPサーバーが認識されない

**診断方法:**
```bash
# MCP設定を確認
cat .mcp.json

# JSONの文法が正しいか確認（Node.jsがある場合）
node -e "console.log(JSON.parse(require('fs').readFileSync('.mcp.json', 'utf8')))"
```

**解決策:**
1. `.mcp.json`の構文を確認
2. ファイルがプロジェクトルートにあることを確認
3. VSCodeをリロード
4. Claude CodeのOutputパネルでエラーログを確認

### 問題3: Chromiumが起動しない

**エラーメッセージ:**
```
Error: Executable doesn't exist at ~/.cache/ms-playwright/chromium-1194/chrome-linux/chrome
```

**解決策:**
```bash
# Chromiumをインストール
npx playwright install chromium

# インストール状態を確認
ls -la ~/.cache/ms-playwright/
```

### 問題4: 初回起動が遅い

**現象:**
MCPサーバーの起動に時間がかかる

**原因:**
npxが初回実行時にパッケージをダウンロードしている

**解決策:**
事前にパッケージをキャッシュ:
```bash
npx @playwright/mcp@latest --version
```

2回目以降は高速に起動します。

## メリット・デメリット

### メリット

1. ✅ **インストール不要** - npx経由で即座に実行
2. ✅ **公式サポート** - Microsoft公式チームがメンテナンス
3. ✅ **高速** - 1-2秒でブラウザ起動
4. ✅ **安定** - 100%成功率（検証済み）
5. ✅ **チーム共有** - `.mcp.json`をGit管理するだけ
6. ✅ **構造化アプローチ** - アクセシビリティツリー使用
7. ✅ **22個のツール** - 豊富な機能

### デメリット

1. ❌ **初回起動遅い** - パッケージダウンロード（数秒〜数十秒）
2. ❌ **ネットワーク依存** - インターネット接続必要（初回）
3. ❌ **Node.js必須** - Node.js v20以上が必要
4. ❌ **キャッシュ管理** - npm cacheに約300MB消費

## DevContainer環境での使用

このディレクトリには**DevContainer設定が含まれています**（`.devcontainer/`）。

### DevContainerで開く

1. このディレクトリをVSCodeで開く
2. 「コンテナーで再度開く」をクリック
3. または `Cmd/Ctrl+Shift+P` → "Dev Containers: Reopen in Container"

### 自動セットアップ

DevContainer起動時に以下が自動実行されます：
- Node.js v20環境の構築
- Chromiumブラウザのインストール

### 詳細

詳しくは [.devcontainer/README.md](./.devcontainer/README.md) を参照してください。

**注意点:**
- `--headless`フラグが必須（GUI表示不可）
- 初回起動時のChromiumインストールに約30秒かかります

## 管理コマンド

### Chromium管理

```bash
# Chromiumインストール
npx playwright install chromium

# Chromium Headless Shellもインストール
npx playwright install chromium-headless-shell

# インストール状態確認
npx playwright install --dry-run chromium
```

### プロファイル管理

```bash
# プロファイルロック削除
rm -rf ~/.cache/ms-playwright/mcp-chrome-*

# 全プロファイル確認
ls -la ~/.cache/ms-playwright/
```

### npm cache管理

```bash
# キャッシュ確認
npm cache verify

# キャッシュ削除（問題がある場合）
npm cache clean --force
```

### パッケージ情報

```bash
# パッケージ情報確認
npm view @playwright/mcp

# バージョン確認
npx @playwright/mcp@latest --version
```

## 参考リンク

- [Playwright MCP GitHub](https://github.com/microsoft/playwright-mcp)
- [npm パッケージ](https://www.npmjs.com/package/@playwright/mcp)
- [Playwright公式サイト](https://playwright.dev)

---

**検証済み**: ✅ Phase 1-6A完了（成功率100%）
**最終更新**: 2025-11-19
