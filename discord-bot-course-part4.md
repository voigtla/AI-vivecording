# Part 4：GCP無料枠完全攻略 〜 ゼロコストで24時間稼働させる 〜

> **このPartのゴール:** ローカルでしか動かなかったBotをGCPの無料枠でインターネットに常時公開する。  
> ⚠️ **初心者注意:** このPartは詰まりやすいです。一つ一つ確認しながら進めてください。詰まったらChatGPTかClaudeにエラーを投げましょう。

---

## 1. GCPの無料枠を理解する

### 💰 GCP無料枠の全体像（2026年3月時点）

```
【新規アカウント特典】
・$300のクレジット（90日間有効）
・クレジットカード登録必須だが、自動課金はされない
・90日後または$300使い切ったら常時無料枠へ移行

【Always Free（常時無料）- Bot運用に使えるもの】

Compute Engine (e2-micro):
  - 月720時間（= 24時間365日分 = 常時稼働できる！）
  - us-west1, us-central1, us-east1 リージョンのみ対象
  - 30GB の標準永続ディスク
  - 1GBのネットワーク送信/月
  ★ Discord Bot には十分すぎるスペック

Cloud Run:
  - 月200万リクエスト無料
  - 360,000 GB秒のメモリ使用無料
  ★ Discord Botはリクエスト数が少ないので実質無料

Cloud Storage:
  - 5GBの標準ストレージ（US リージョン）
  ★ JSONデータの保存に使える

Cloud Scheduler:
  - 月3ジョブ無料
  ★ 定期的なヘルスチェックやリセット処理に使える

Secret Manager:
  - 月6アクセス操作、10,000回のアクセス無料
  ★ Discord Tokenの安全な管理に使える
```

### ⚠️ 無料枠の落とし穴

```
【こうなると課金が発生する】

❌ e2-micro を2台起動する（1台目は無料、2台目から課金）
❌ us-west1 以外のリージョンに建てる
❌ 静的IPアドレスを取得して使わない（月数百円課金される）
❌ Compute Engine の長期スナップショットを大量に取る
❌ Cloud Run で大量リクエストを受ける（月200万超えたら課金）
❌ 90日の試用クレジット切れ後に大きなインスタンスを動かしたまま

【防衛策】
✅ 予算アラートを設定する（月$1で警告を受け取る）
✅ インスタンスの起動数を1に制限するポリシーを設定
✅ 定期的にコンソールで料金ページを確認する
```

---

## 2. GCP プロジェクトのセットアップ

### 🔧 Step 1: GCPアカウントとプロジェクト作成

```
1. https://cloud.google.com/ にアクセス
2. 「無料で開始」→ Googleアカウントでログイン
3. クレジットカード情報を入力（自動課金なし）
4. コンソールへ移動

5. 新しいプロジェクトを作成:
   - 左上「Google Cloud」→「プロジェクト選択」
   - 「新しいプロジェクト」
   - プロジェクト名: discord-omikuji-bot
   - 「作成」
```

### 🔧 Step 2: gcloud CLI のインストールと認証

```bash
# macOS
brew install google-cloud-sdk

# Windows
# https://cloud.google.com/sdk/docs/install から installer をダウンロード

# Ubuntu/Debian
curl https://sdk.cloud.google.com | bash
exec -l $SHELL

# 認証
gcloud auth login

# プロジェクトを設定
gcloud config set project discord-omikuji-bot

# 確認
gcloud config list
```

---

## 3. 方法A: Compute Engine（おすすめ・常時稼働）

### 📍 なぜ Compute Engine か？

```
Discord Bot は「常時起動して待機している」必要がある。
Cloud Run はリクエストが来たときだけ起動する設計なので、
WebSocketベースのDiscord Botには不向き。

Compute Engine の e2-micro は常時稼働できて無料枠内に収まる。
これが Discord Bot のベストな選択肢。
```

### 🖥️ VM インスタンスの作成

```bash
# コマンドラインから作成（推奨）
gcloud compute instances create discord-bot-vm \
  --project=discord-omikuji-bot \
  --zone=us-west1-b \
  --machine-type=e2-micro \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-standard \
  --tags=discord-bot

# 確認
gcloud compute instances list
```

> **ゾーンは `us-west1-b` を指定する。**  
> `us-west1` リージョン内ならどこでも無料枠対象だが、  
> `-b` が最も安定している（経験則）。

### 🔐 SSH でVMに接続

```bash
# ブラウザ内SSH（最も簡単）
# GCPコンソール → Compute Engine → VMインスタンス → 「SSH」ボタン

# またはgcloud経由（ローカルターミナルから）
gcloud compute ssh discord-bot-vm --zone=us-west1-b
```

### 📦 VM上でのNode.js セットアップ

```bash
# SSH接続後、VM上で実行

# Node.js 20.x のインストール
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# バージョン確認
node -v
npm -v

# Git のインストール（あれば省略）
sudo apt-get install -y git

# プロジェクトをGitHubからクローン（または直接アップロード）
git clone https://github.com/あなた/omikuji-bot.git
cd omikuji-bot

# 依存関係インストール
npm install
```

### 🔑 Secret Manager でTokenを安全に管理

```bash
# ローカルで実行（VMに設定する前に）

# Secret Manager APIを有効化
gcloud services enable secretmanager.googleapis.com

# DISCORD_TOKENをSecretとして登録
echo -n "your_actual_discord_token" | \
  gcloud secrets create DISCORD_TOKEN \
    --data-file=- \
    --replication-policy=automatic

# CLIENT_IDも同様に
echo -n "your_client_id" | \
  gcloud secrets create CLIENT_ID --data-file=-

# GUILD_IDも同様に（本番では不要かも）
echo -n "your_guild_id" | \
  gcloud secrets create GUILD_ID --data-file=-
```

```javascript
// VM上のコードでSecretを取得する場合
// npm install @google-cloud/secret-manager

const { SecretManagerServiceClient } = require('@google-cloud/secret-manager');
const client = new SecretManagerServiceClient();

async function getSecret(name) {
  const projectId = 'discord-omikuji-bot';
  const [version] = await client.accessSecretVersion({
    name: `projects/${projectId}/secrets/${name}/versions/latest`,
  });
  return version.payload.data.toString();
}

// 起動時に環境変数に設定
async function loadSecrets() {
  process.env.DISCORD_TOKEN = await getSecret('DISCORD_TOKEN');
  process.env.CLIENT_ID = await getSecret('CLIENT_ID');
}
```

> **開発中はシンプルに `.env` で十分。**  
> GCP上での本番運用時にSecret Managerを使うとベスト。

### 🔄 PM2 でBotを常時稼働させる

Node.jsプロセスをデーモン化するには **PM2** を使う:

```bash
# VM上で実行

# PM2 インストール
sudo npm install -g pm2

# Botを起動
pm2 start src/index.js --name discord-omikuji-bot

# VM再起動後も自動起動するよう設定
pm2 startup
# → 表示されるコマンドをコピーして実行する

pm2 save

# 状態確認
pm2 status
pm2 logs discord-omikuji-bot

# 再起動
pm2 restart discord-omikuji-bot
```

### 📊 便利なPM2コマンド

```bash
pm2 list                      # 全プロセス一覧
pm2 logs discord-omikuji-bot  # ログをリアルタイム表示
pm2 stop discord-omikuji-bot  # 停止
pm2 delete discord-omikuji-bot # 削除
pm2 monit                     # リソース使用量モニター
```

---

## 4. 方法B: Cloud Run（高度・自動スケール）

> **Cloud Run は Discord Bot には基本的に向かないが、**  
> HTTPエンドポイントが必要な場合（Webhook Bot等）には有効。  
> 参考程度に記載する。

```bash
# Dockerfileを作成
cat > Dockerfile << 'EOF'
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
CMD ["node", "src/index.js"]
EOF

# .dockerignore
cat > .dockerignore << 'EOF'
node_modules
.env
.git
*.log
EOF

# Cloud Run にデプロイ
gcloud run deploy discord-omikuji-bot \
  --source . \
  --region us-west1 \
  --platform managed \
  --allow-unauthenticated \
  --set-env-vars DISCORD_TOKEN=xxx,CLIENT_ID=yyy

# ※ Cloud Run のWebSocket非対応問題を回避するには
# Gateway IntentsをHTTP Webhook方式に変更する必要がある（複雑）
```

---

## 5. デプロイ後の確認とトラブルシューティング

### ✅ 確認チェックリスト

```bash
# VM上で確認

# 1. プロセスが動いているか
pm2 status

# 2. ログにエラーがないか
pm2 logs discord-omikuji-bot --lines 50

# 3. メモリ使用量が常識的か（e2-microは1GBしかない）
free -h

# 4. ディスク使用量
df -h

# 5. ネットワーク接続（Discordサーバーに繋がるか）
curl -I https://discord.com/api/v10/gateway
```

### 🔴 よくある問題と解決策

```
【問題】pm2 が起動後すぐに停止する
解決策: pm2 logs でエラーを確認する
       多くの場合は .env の設定漏れか、npm install 忘れ

【問題】Bot はオンラインだがコマンドが反応しない
解決策:
  1. deploy-commands.js を実行したか確認
  2. GUILD_ID が正しいか確認
  3. BotのIntentsが正しく設定されているか確認

【問題】VM が突然落ちる
解決策:
  - メモリ不足の可能性（e2-microは1GB）
  - pm2 monit でメモリ使用量を確認
  - 不要なプロセスを終了する

【問題】Git pull してもBot に反映されない
解決策: pm2 restart discord-omikuji-bot を実行する
```

### 🔄 コード更新のワークフロー

```bash
# ローカルで開発・GitHubにpush
git add .
git commit -m "feat: おみくじの新パターンを追加"
git push origin main

# VM上で更新を反映
gcloud compute ssh discord-bot-vm --zone=us-west1-b --command="cd omikuji-bot && git pull && pm2 restart discord-omikuji-bot"
```

---

## 6. 予算アラートの設定（重要！）

**これを設定しないと、誤操作で思わぬ課金が発生する可能性がある。**

```
GCPコンソール:
1. 左メニュー「お支払い」→「予算とアラート」
2. 「予算を作成」
3. 設定:
   - 予算名: discord-bot-budget
   - スコープ: 全サービス
   - 予算金額: $1 （1ドルを超えたら警告）
   - アラートのしきい値: 50%, 90%, 100%
   - メール通知: チェックを入れる
4. 「保存」
```

---

## 7. 無料枠との付き合い方・長期戦略

### 📅 月次チェックルーティン

```
毎月1日にやること:
  □ GCPコンソール → 料金 → 現在の費用を確認
  □ $0.00 であることを確認
  □ VM が1台だけ動いているか確認
  □ 静的IPが割り当てられていないか確認
  □ 不要なスナップショットを削除
```

### 💡 無料で戦い続けるための哲学

```
【Bot開発の段階別戦略】

ステージ1: 開発・テスト（0〜1ヶ月目）
  - ローカル（自分のPC）で開発・テスト
  - GCPは使わない（費用ゼロ）
  - AIのコスト: ほぼゼロ（無料枠内）

ステージ2: 初期デプロイ（1〜3ヶ月目）
  - GCP $300クレジットを活用
  - より大きなインスタンスで試せる
  - 余裕があるのでいろいろ試す期間

ステージ3: 安定運用（3ヶ月目以降）
  - e2-micro + PM2 + Secret Manager の組み合わせに固定
  - 月$0を維持する
  - Bot の機能拡張に集中

ステージ4: スケールが必要になったとき
  - ユーザーが増えてきたら課金を検討
  - まずClaude Pro ($20/月) で開発効率を上げる
  - GCPの課金は最後の手段
```

---

→ [Part 5へ続く：運用・監視・スケールアップ](./discord-bot-course-part5.md)

---

*GCPは最初の設定が大変だが、一度動いてしまえば何もしなくていい。  
詰まったら「GCP e2-micro discord bot」でGoogle検索すると日本語の情報が豊富にある。  
ChatGPTにエラーを投げれば9割は解決する。残り1割は公式ドキュメントへ。*
