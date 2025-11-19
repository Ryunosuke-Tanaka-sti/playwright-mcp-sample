# Docker直接実行版 セットアップガイド

**Playwright MCP** をDockerで都度起動型として実行する、最もセキュアな方法です。

## 特徴

- ✅ **最もセキュア** - セッション毎に新しいコンテナが起動
- ✅ **リソース効率的** - 使用時のみ起動、終了後は自動削除
- ✅ **常に最新** - `--pull=always`で最新イメージを自動取得
- ✅ **クリーン** - `--rm`でコンテナ自動削除
- ✅ **公式推奨** - Playwright MCP公式が推奨する方式
- ✅ **並列実行可能** - 複数セッションが完全に独立

## 前提条件

| 項目 | 要件 |
|------|------|
| Docker | v20以上 |
| Claude Code | インストール済み |

**確認方法:**
```bash
docker --version  # Docker version 20.x.x 以上
```

## セットアップ手順（2分）

### Step 1: `.mcp.json`を配置

このディレクトリにはすでに`.mcp.json`が含まれています。

**設定内容:**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--init",
        "--pull=always",
        "mcr.microsoft.com/playwright/mcp",
        "--isolated"
      ]
    }
  }
}
```

**重要なフラグ:**
- `-i`: 標準入力を開く（stdio接続に必要）
- `--rm`: コンテナ終了時に自動削除
- `--init`: プロセス管理の改善
- `--pull=always`: 常に最新イメージを使用
- `--isolated`: メモリ内でプロファイル保持

### Step 2: VSCodeをリロード

```bash
# VSCodeを開く
code .

# VSCodeをリロード
# Cmd/Ctrl + Shift + P → "Developer: Reload Window"
```

### Step 3: 動作確認

Claude Codeのチャットで以下を試してください:

```
Playwright MCPを使用して、https://example.com を開いてページタイトルを取得してください
```

**初回実行:**
- Dockerイメージが自動的にダウンロードされます（約1GB、数分）
- 2回目以降は`--pull=always`により最新チェックが行われますが、変更がなければ即座に起動します

**期待される結果:**
```
✅ ページURL: https://example.com
✅ ページタイトル: "Example Domain"
✅ エラーなし
```

## 動作の仕組み

### 実行フロー

1. **Claude Codeがコマンド実行**: `docker run ...`
2. **Dockerイメージ取得**: 最新イメージをチェック（`--pull=always`）
3. **コンテナ起動**: 新しいコンテナが起動
4. **ブラウザ操作実行**: Playwrightが操作を実行
5. **コンテナ削除**: 終了後、コンテナが自動削除（`--rm`）

### セッションの独立性

各Claude Codeセッションは完全に独立したDockerコンテナで実行されます:

```
セッション1: docker run ... → コンテナA → 終了 → 削除
セッション2: docker run ... → コンテナB → 終了 → 削除
セッション3: docker run ... → コンテナC → 終了 → 削除
```

## パフォーマンス

| 操作 | 時間 | 備考 |
|------|------|------|
| 初回起動 | 数分 | イメージダウンロード（約1GB） |
| 2回目以降（最新） | 即座 | イメージキャッシュ利用 |
| 2回目以降（チェック） | 数秒 | `--pull=always`による確認 |
| コンテナ起動 | 1-2秒 | 毎回新規作成 |
| ブラウザ操作 | < 1-2秒 | 操作による |

**注意:**
- `--pull=always`により毎回最新イメージをチェックするため、起動時に数秒かかります
- ネットワーク接続が必要です

## カスタマイズ

### バージョン固定（本番環境推奨）

最新版を追わず、特定バージョンに固定:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--init",
        "mcr.microsoft.com/playwright/mcp:v0.0.47",
        "--isolated"
      ]
    }
  }
}
```

**メリット:**
- ✅ 予期せぬ動作変更を回避
- ✅ `--pull=always`不要で高速起動
- ✅ 再現性の確保

### ボリュームマウント追加

スクリーンショットをホスト側に保存:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--init",
        "-v", "${PWD}/screenshots:/home/pwuser/.playwright-mcp",
        "mcr.microsoft.com/playwright/mcp",
        "--isolated"
      ]
    }
  }
}
```

**注意:**
- `${PWD}`は環境により動作しない場合があります
- 絶対パスを指定することを推奨

### ネットワーク設定

他のDockerコンテナと通信する場合:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--init",
        "--network", "host",
        "mcr.microsoft.com/playwright/mcp",
        "--isolated"
      ]
    }
  }
}
```

### リソース制限

メモリ・CPU制限:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--init",
        "--memory", "2g",
        "--cpus", "2",
        "mcr.microsoft.com/playwright/mcp",
        "--isolated"
      ]
    }
  }
}
```

## トラブルシューティング

### 問題1: "Cannot connect to Docker daemon"

**原因:**
Dockerが起動していない、または権限がない

**解決策:**
```bash
# Dockerが起動しているか確認
docker ps

# Dockerサービスを起動（Linux）
sudo systemctl start docker

# ユーザーをdockerグループに追加（Linux）
sudo usermod -aG docker $USER
# ログアウト・ログインして反映
```

### 問題2: イメージダウンロードに失敗

**原因:**
ネットワーク問題、またはDockerレジストリへのアクセス制限

**解決策:**
```bash
# 手動でイメージをダウンロード
docker pull mcr.microsoft.com/playwright/mcp

# プロキシ設定（必要な場合）
# ~/.docker/config.json に設定
```

### 問題3: 起動が遅い

**原因:**
`--pull=always`により毎回最新チェックが行われる

**解決策:**
バージョンを固定して`--pull=always`を削除:
```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--init",
        "mcr.microsoft.com/playwright/mcp:v0.0.47",
        "--isolated"
      ]
    }
  }
}
```

### 問題4: "OCI runtime create failed"

**原因:**
Dockerコンテナの起動に失敗

**解決策:**
```bash
# Dockerログを確認
docker logs $(docker ps -a | grep playwright | awk '{print $1}' | head -1)

# Dockerデーモンを再起動
sudo systemctl restart docker
```

## メリット・デメリット

### メリット

1. ✅ **最もセキュア** - セッション毎に完全に独立
2. ✅ **リソース効率的** - 使用時のみ起動
3. ✅ **常に最新** - `--pull=always`で自動更新
4. ✅ **クリーン** - コンテナ自動削除でストレージ状態が残らない
5. ✅ **並列実行可能** - 複数セッションが独立して動作
6. ✅ **公式推奨** - Playwright MCP公式が推奨
7. ✅ **CI/CD最適** - クリーンな環境、再現性

### デメリット

1. ❌ **起動がやや遅い** - 毎回新しいコンテナを作成（1-2秒 + チェック時間）
2. ❌ **ネットワーク依存** - `--pull=always`によりインターネット接続必要
3. ❌ **初回ダウンロード** - 約1GBのイメージダウンロードが必要
4. ❌ **Docker必須** - Docker環境が必要

## 推奨ケース

### Docker直接実行版が適している場合

- ✅ **本番環境** - セキュリティ重視
- ✅ **CI/CD環境** - クリーンな環境、再現性
- ✅ **各実行が独立が必要** - テストの独立性
- ✅ **リソース効率重視** - 使用時のみリソース消費
- ✅ **常に最新版を使用** - 自動更新が必要

### 他の方式が適している場合

- **npx版** - Node.js環境があり、高速起動が必要
- **Docker Compose版** - 複数プロジェクトで共有、常駐型が必要

## CI/CD環境での使用例

### GitHub Actions

```yaml
name: E2E Tests with Playwright MCP

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Claude Code CLI
        run: |
          # Claude Code CLIのインストール手順
          # （実際の手順は環境により異なる）

      - name: Run E2E Tests
        run: |
          # .mcp.jsonが既に含まれている前提
          # Claude Code経由でテストを実行
```

### GitLab CI

```yaml
playwright-mcp-test:
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker pull mcr.microsoft.com/playwright/mcp
    # Claude Code経由でテストを実行
```

## 管理コマンド

### イメージ管理

```bash
# イメージのダウンロード
docker pull mcr.microsoft.com/playwright/mcp

# イメージの確認
docker images | grep playwright

# イメージの削除（必要な場合）
docker rmi mcr.microsoft.com/playwright/mcp
```

### コンテナ確認

```bash
# 実行中のコンテナ確認
docker ps | grep playwright

# すべてのコンテナ確認（終了済み含む）
docker ps -a | grep playwright

# 注意: --rmフラグにより通常は自動削除されます
```

## 参考リンク

- [Playwright MCP GitHub](https://github.com/microsoft/playwright-mcp)
- [Playwright公式Docker Image](https://playwright.dev/docs/docker)
- [Docker公式ドキュメント](https://docs.docker.com/)

---

**推奨度**: ⭐⭐⭐⭐⭐（本番環境/CI/CD）
**最終更新**: 2025-11-19
