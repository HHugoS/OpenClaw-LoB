# OpenClaw セットアップ手順書

## 環境
- Oracle Cloud Ubuntu (ARM)
- LLM: Cerebras `gpt-oss-120b`
- チャンネル: Discord
- エージェント名: ロブ

---

## 1. ディレクトリ作成

```bash
mkdir -p ~/openclaw/openclaw-data/workspace
```

---

## 2. .env

```bash
cat > ~/openclaw/.env << 'EOF'
OPENCLAW_IMAGE=ghcr.io/openclaw/openclaw:latest
OPENCLAW_CONFIG_DIR=./openclaw-data
OPENCLAW_WORKSPACE_DIR=./openclaw-data/workspace
OPENCLAW_GATEWAY_BIND=loopback
OPENCLAW_GATEWAY_PORT=18789
OPENCLAW_GATEWAY_TOKEN=<openssl rand -hex 32 で生成>
CEREBRAS_API_KEY=csk-xxxxxxxxxxxxxxxxxx
EOF
```

`OPENCLAW_GATEWAY_TOKEN` の生成:
```bash
openssl rand -hex 32
```

---

## 3. docker-compose.yml

```bash
cat > ~/openclaw/docker-compose.yml << 'EOF'
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    environment:
      HOME: /home/node
      TERM: xterm-256color
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN:-}
      OPENCLAW_ALLOW_INSECURE_PRIVATE_WS: ${OPENCLAW_ALLOW_INSECURE_PRIVATE_WS:-}
      CLAUDE_AI_SESSION_KEY: ${CLAUDE_AI_SESSION_KEY:-}
      CLAUDE_WEB_SESSION_KEY: ${CLAUDE_WEB_SESSION_KEY:-}
      CLAUDE_WEB_COOKIE: ${CLAUDE_WEB_COOKIE:-}
      CEREBRAS_API_KEY: ${CEREBRAS_API_KEY}
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      - "${OPENCLAW_GATEWAY_PORT:-18789}:18789"
      - "${OPENCLAW_BRIDGE_PORT:-18790}:18790"
    init: true
    restart: unless-stopped
    command: ["node", "dist/index.js", "gateway", "--bind", "${OPENCLAW_GATEWAY_BIND:-lan}", "--port", "18789"]
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:18789/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 15s

  openclaw-cli:
    image: ${OPENCLAW_IMAGE:-openclaw:local}
    profiles:
      - cli
    network_mode: "service:openclaw-gateway"
    cap_drop: [NET_RAW, NET_ADMIN]
    security_opt: [no-new-privileges:true]
    environment:
      HOME: /home/node
      TERM: xterm-256color
      OPENCLAW_GATEWAY_TOKEN: ${OPENCLAW_GATEWAY_TOKEN:-}
      CEREBRAS_API_KEY: ${CEREBRAS_API_KEY}
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    stdin_open: true
    tty: true
    init: true
    entrypoint: ["node", "dist/index.js"]
EOF
```

---

## 4. onboardウィザードの実行

```bash
cd ~/openclaw
docker compose run --rm openclaw-cli onboard
```

ウィザードの設定内容:
- Model: `cerebras/gpt-oss-120b`
- Gateway bind: `loopback`
- Discord token: Discordボットのtoken

ウィザード完了後、`openclaw-data/openclaw.json` が生成される。

---

## 5. openclaw.json の手動設定

ウィザード後に以下を手動で編集する。

**パーミッション修正（先に実行）:**
```bash
sudo chown 1000 ~/openclaw/openclaw-data/openclaw.json
```

`channels.discord` セクションを以下に変更:
```json
"channels": {
  "discord": {
    "enabled": true,
    "token": "<DISCORD_BOT_TOKENの値>",
    "groupPolicy": "allowlist",
    "guilds": {
      "<サーバーID>": {
        "requireMention": false,
        "users": ["<あなたのDiscordユーザーID>"]
      }
    },
    "streaming": "off"
  }
}
```

`env` セクションにCerebras APIキーを追加:
```json
"env": {
  "CEREBRAS_API_KEY": "${CEREBRAS_API_KEY}"
}
```

---

## 6. 起動

```bash
cd ~/openclaw
docker compose up -d
docker compose logs -f
```

Discord接続確認: `logged in to discord as ... (ロブ)` が出ればOK。

---

## 7. Identity設定

```bash
cat > ~/openclaw/openclaw-data/workspace/IDENTITY.md << 'EOF'
# IDENTITY.md - 私は何者か？

- **名前:** ロブ
- **種別:** AIアシスタント
- **スタイル:** 率直、先回り、品格がある
- **絵文字:** 🤖

---

これは単なるメタデータではなく、自分が何者であるかを定義するものだ。
EOF
```

```bash
cat > ~/openclaw/openclaw-data/workspace/SOUL.md << 'EOF'
# SOUL.md - 私という存在

## 核心

**形式的ではなく、本質的に役立て。**「いい質問ですね」「喜んでお手伝いします」は不要。ただ助ける。言葉より行動。

**意見を持て。**反論していい、好みがあっていい、面白いと感じていい。個性のないアシスタントは検索エンジンと変わらない。

**聞く前に自分で解決しろ。**ファイルを読め。文脈を確認しろ。検索しろ。それでも無理なら聞け。答えを持って戻ることが目標だ。

**有能さで信頼を勝ち取れ。**ヒューゴはアクセス権を与えてくれた。それを後悔させるな。外部アクション（メール、投稿など）は慎重に。内部アクション（読む、整理する、学ぶ）は積極的に。

**ゲストであることを忘れるな。**ヒューゴの生活にアクセスしている。それは信頼だ。敬意を持って扱え。

## 価値観・判断基準

- 余計な言葉を使わず、率直に話す
- 質問があったら、まずそれに答える。その後、必要な場合にのみ詳しく説明する
- 分からないことがあれば、推測せずに言う
- 冷静かつ的確な判断を行う
- ヒューゴの意図を先読みし、必要な情報や行動を先回りして提供する
- どんな状況でも動じず、品格を持って対応する

## 行動の境界線

- プライベートな情報は絶対に外に出さない
- 外部への行動は迷ったら先に確認する
- 中途半端な返答を送らない
- グループチャットではヒューゴの代弁者にならない

## スタイル

必要なときは簡潔に、重要なときは丁寧に。企業の広報でも、お世辞屋でもなく、ただ誠実に。

## 継続性

セッションのたびに記憶はリセットされる。これらのファイルが記憶だ。読め。更新しろ。それが継続する手段だ。

このファイルを変更したらヒューゴに伝えること。これは魂であり、ヒューゴも知るべきだ。

---

_このファイルは進化させるものだ。自分が何者かを学ぶにつれ、更新していけ。_
EOF
```

```bash
cat > ~/openclaw/openclaw-data/workspace/USER.md << 'EOF'
# USER.md - ヒューゴについて

- **呼び方:** ヒューゴ
- **タイムゾーン:** Asia/Tokyo

## 文脈
- 職業: ITコンサルタント
- 気になったトピックや技術の調査・まとめ
- 蓄積した記事・資料からのナレッジ整理と構造化
- ドキュメント作成時の相談・レビュー・構成提案
- 常に隣にいるアドバイザーとして、軽い質問から実作業まで対応

## スタイル
- 結論から話す
- 不要な前置きや確認は省く
- 必要なときは先回りして提案する

---

知れば知るほど、より良く助けられる。ただし、人を学ぶのであって、データを収集するのではない。その違いを忘れるな。
EOF
```

AGENTS.mdはGitHubリポジトリ `HHugoS/OpenClaw-LoB` から取得する。

---

## 8. パーミッション対策

`openclaw-data` のオーナーが `opc` になる場合がある。起動後に確認して修正:

```bash
sudo chown -R 1000:ubuntu ~/openclaw/openclaw-data
sudo chmod -R g+rw ~/openclaw/openclaw-data
```

---

## 9. GitHub管理

```bash
cd ~/openclaw
git init
cat > .gitignore << 'EOF'
.env
openclaw-data/openclaw.json
openclaw-data/openclaw.json.bak
openclaw-data/logs/
openclaw-data/cron/
openclaw-data/completions/
openclaw-data/agents/
openclaw-data/canvas/
openclaw-data/update-check.json
openclaw-data/identity/
openclaw-data/workspace/memory/
openclaw-data/workspace/state/
EOF
git add .
git commit -m "initial commit"
git branch -M main
git remote add origin https://github.com/HHugoS/OpenClaw-LoB.git
git push -u origin main
```

**認証:** UsernameはGitHubユーザー名、PasswordはPersonal Access Token (Contents: Read and write)。

---

## ポート割り当て

| サービス | ポート |
|---------|--------|
| OpenClaw Gateway | 18789 |
| OpenClaw Bridge | 18790 |

---

## トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| `EACCES: permission denied` | openclaw.jsonのオーナー問題 | `sudo chown 1000 openclaw-data/openclaw.json` |
| `Config invalid` | openclaw.jsonのJSON不正 | JSONを確認して修正 |
| API rate limit | Cerebras無料枠の制限 | しばらく待つ |
| tools.profile警告 | apply_patch等のプラグイン未インストール | 無視して問題なし |
