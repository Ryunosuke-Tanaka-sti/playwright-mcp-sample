# Docker Compose版 セットアップガイド

**Playwright MCP** をDocker Composeで常駐型サーバーとして実行する方法です。

## 特徴

- ✅ **複数プロジェクト共有** - 常駐型で複数から利用可能
- ✅ **環境分離** - Dockerコンテナで完全に独立
- ✅ **自動再起動** - `restart: unless-stopped`
- ✅ **管理が簡単** - Docker Composeで一元管理
- ✅ **2つの接続方式** - HTTP接続とstdio接続の両方に対応

## 前提条件

| 項目 | 要件 |
|------|------|
| Docker | v20以上 |
| Docker Compose | v2以上 |
| Claude Code | インストール済み |

**確認方法:**
```bash
docker --version         # Docker version 20.x.x 以上
docker compose version   # Docker Compose version v2.x.x 以上
```

## セットアップ手順（5分）

### Step 1: Docker Composeで起動

```bash
# このディレクトリに移動
cd docker-compose-setup

# サーバーを起動
docker compose up -d
```

**初回起動の流れ:**
1. Dockerイメージのダウンロード（約743MB、約2-3分）
2. Chromeのインストール（約112MB、約1-2分）
3. Playwright MCPのインストール（約1分）

**2回目以降:**
```bash
docker compose up -d
# → 約10秒で起動完了
```

### Step 2: サーバー起動確認

```bash
# コンテナの状態を確認
docker compose ps

# 期待される出力:
# NAME                    STATUS
# playwright-mcp-server   Up X seconds
```

**ログ確認:**
```bash
docker compose logs playwright-mcp

# 期待される出力に以下が含まれる:
# Listening on http://localhost:8931
```

### Step 3: `.mcp.json`を配置

このディレクトリにはすでに`.mcp.json`が含まれています。

**設定ファイル:**
- `.mcp.json` - stdio接続版（検証済み）

**デフォルト設定（stdio接続）:**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": [
        "exec",
        "-i",
        "playwright-mcp-server",
        "node",
        "/root/.npm/_npx/9833c18b2d85bc59/node_modules/.bin/mcp-server-playwright",
        "--isolated",
        "--no-sandbox"
      ]
    }
  }
}
```

### Step 4: VSCodeをリロード

```bash
# VSCodeを開く
code .

# VSCodeをリロード
# Cmd/Ctrl + Shift + P → "Developer: Reload Window"
```

### Step 5: 動作確認

Claude Codeのチャットで以下を試してください:

```
Playwright MCPを使用して、https://example.com を開いてページタイトルを取得してください
```

**期待される結果:**
```
✅ ページURL: https://example.com
✅ ページタイトル: "Example Domain"
✅ エラーなし
```

## 管理コマンド

### サーバー管理

```bash
# 起動
docker compose up -d

# 停止
docker compose down

# 再起動
docker compose restart

# 状態確認
docker compose ps

# リソース使用状況
docker stats playwright-mcp-server
```

### ログ確認

```bash
# リアルタイムでログを表示
docker compose logs -f playwright-mcp

# 最新100行を表示
docker compose logs --tail 100 playwright-mcp

# エラーのみ表示
docker compose logs playwright-mcp | grep -i error
```

### コンテナ内の操作

```bash
# コンテナ内に入る
docker exec -it playwright-mcp-server bash

# Chromeプロセス確認
docker exec playwright-mcp-server ps aux | grep chrome

# プロファイルロック削除（トラブル時）
docker exec playwright-mcp-server rm -rf /ms-playwright/mcp-chrome-*
```

## 接続方式の選択

### stdio接続（検証済み・推奨）

**メリット:**
- ✅ ネットワーク設定不要
- ✅ 公式推奨の方式と同じ
- ✅ **検証済み** - Phase 1-4で100%動作確認

**デメリット:**
- ⚠️ npxのパスが環境により異なる可能性

**設定（`.mcp.json`）:**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": [
        "exec",
        "-i",
        "playwright-mcp-server",
        "node",
        "/root/.npm/_npx/9833c18b2d85bc59/node_modules/.bin/mcp-server-playwright",
        "--isolated",
        "--no-sandbox"
      ]
    }
  }
}
```

**注意:** npxのパス（`/root/.npm/_npx/...`）は環境により異なる場合があります。

**パスの確認方法:**
```bash
docker exec -it playwright-mcp-server bash
find /root/.npm/_npx -name "mcp-server-playwright"
```

### HTTP接続（代替）

**メリット:**
- ✅ 設定がシンプル
- ✅ ログが見やすい
- ✅ デバッグが簡単
- ✅ 複数クライアントから接続可能

**デメリット:**
- ⚠️ **未検証** - test-results.mdには記載があるが実際の動作確認はstdio接続

**設定（`.mcp.http.json`）:**
```json
{
  "mcpServers": {
    "playwright": {
      "url": "http://localhost:8931/mcp"
    }
  }
}
```

**切り替え方法:**
```bash
mv .mcp.json .mcp.stdio.json
mv .mcp.http.json .mcp.json
# VSCodeをリロード
```

## パフォーマンス実測値

| 操作 | 時間 |
|------|------|
| 初回起動 | 約5分 |
| 2回目以降の起動 | 約10秒 |
| ブラウザ起動 | 1-2秒 |
| example.com アクセス | 27-94秒（ばらつき） |
| スクリーンショット取得 | < 1秒 |

**注意:**
- Docker Compose版はページアクセスにばらつきがあります（27秒〜94秒）
- 安定したパフォーマンスが必要な場合はnpx版を推奨

## トラブルシューティング

### 問題1: コンテナが起動しない

**診断:**
```bash
# コンテナの状態を確認
docker compose ps

# ログを確認
docker compose logs playwright-mcp
```

**よくある原因:**
1. ポート8931が既に使用されている
2. Dockerのリソース不足

**解決策:**
```bash
# ポート使用状況を確認
netstat -an | grep 8931
# または
lsof -i :8931

# 既存のコンテナを停止
docker compose down

# 再起動
docker compose up -d
```

### 問題2: MCPサーバーに接続できない

**診断:**
```bash
# サーバーが起動しているか確認
docker compose ps

# ログでエラーを確認
docker compose logs playwright-mcp | grep -i error

# ポートが開いているか確認
curl -v http://localhost:8931
```

**解決策:**
1. サーバーが起動しているか確認
2. ポート8931が開いているか確認
3. コンテナを再起動
   ```bash
   docker compose restart
   ```

### 問題3: "Session not found"エラー

**原因:**
HTTP接続の初期化に問題がある可能性

**解決策:**
```bash
# コンテナを再起動
docker compose restart

# VSCodeをリロード

# それでも解決しない場合はstdio接続を試す
mv .mcp.json .mcp.http.json
mv .mcp.stdio.json .mcp.json
# VSCodeをリロード
```

### 問題4: リソース不足

**現象:**
コンテナが頻繁に停止する、または動作が遅い

**診断:**
```bash
docker stats playwright-mcp-server
```

**解決策:**
`compose.yml`にリソース制限を追加:
```yaml
services:
  playwright-mcp:
    # ... 既存の設定 ...
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
```

## カスタマイズ

### ボリュームマウント追加

スクリーンショットをホスト側に保存:

```yaml
services:
  playwright-mcp:
    # ... 既存の設定 ...
    volumes:
      - ./screenshots:/home/pwuser/.playwright-mcp/screenshots
```

### 別のブラウザを使用

```yaml
services:
  playwright-mcp:
    # ... 既存の設定 ...
    command: >
      sh -c "
        npx playwright install firefox &&
        npm install -g @playwright/mcp@latest &&
        npx @playwright/mcp@latest --port 8931 --headless --isolated --no-sandbox --browser firefox
      "
```

### 複数ブラウザを並行実行

```yaml
services:
  playwright-mcp-chrome:
    # ... Chrome設定 ...
    container_name: playwright-mcp-chrome
    ports:
      - "8931:8931"

  playwright-mcp-firefox:
    # ... Firefox設定 ...
    container_name: playwright-mcp-firefox
    ports:
      - "8932:8932"
```

## メリット・デメリット

### メリット

1. ✅ **複数プロジェクト共有** - 常駐型で複数から利用可能
2. ✅ **環境分離** - Dockerコンテナで独立
3. ✅ **自動再起動** - `restart: unless-stopped`
4. ✅ **管理が簡単** - Docker Composeで一元管理
5. ✅ **2つの接続方式** - HTTP/stdio両対応

### デメリット

1. ❌ **初回起動時間** - 約5分（イメージ743MB + Chrome 112MB）
2. ❌ **リソース常時消費** - メモリ・CPU常駐（約1GB）
3. ❌ **ポート管理必要** - 8931ポート占有
4. ❌ **パフォーマンスばらつき** - 27秒〜94秒
5. ❌ **Docker知識必要** - Docker/Docker Composeの理解が必要

## 推奨ケース

### Docker Compose版が適している場合

- ✅ 複数のプロジェクトで共有したい
- ✅ 開発環境での頻繁な使用
- ✅ チーム全体で統一環境を使用したい
- ✅ リソースを常時確保できる環境がある

### npx版が適している場合

- ✅ 単一プロジェクトでのみ使用
- ✅ リソース消費を最小限に抑えたい
- ✅ Docker環境を構築したくない
- ✅ 安定したパフォーマンスが必要

## 参考リンク

- [Playwright公式Docker Image](https://playwright.dev/docs/docker)
- [Docker Compose公式ドキュメント](https://docs.docker.com/compose/)
- [Playwright MCP GitHub](https://github.com/microsoft/playwright-mcp)

---

**検証済み**: ✅ Phase 1-4完了（成功率100%）
**最終更新**: 2025-11-19
