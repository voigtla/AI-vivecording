# 🎋 中級者向け Discord Bot 制作講座
### 〜 無料AIだけでBotを作る・2026年最高のバイブコーディングエンジニアへの近道 〜

---

## ✉️ はじめに — この講座に込めた思い

この講座を書こうと思ったきっかけは、シンプルな問いから始まった。

**「お金をかけずに、AIを使いこなして、本物のものを作れるか？」**

2026年現在、AIはもはや「便利なツール」ではなく、開発の中心にある。  
ChatGPT、Claude、Gemini、そしてGCP。それぞれ無料枠があり、組み合わせれば想像以上のことができる。  
問題は「どう組み合わせるか」を誰も体系的に教えてくれないことだった。

私自身のやり方は、ある意味でめちゃくちゃだ。  
ChatGPTでアイデアを出し、Geminiに整理させ、ClaudeにZIPを作らせ、うまく動かなければまたZIPをChatGPTに投げる。  
「AIをAIで補完する」——この循環が、2026年のバイブコーディングの本質だと思っている。

この講座は、その「めちゃくちゃだけど合理的なやり方」を一冊にまとめたものだ。  
コードを書く量を最小化して、**思考と設計に時間を使う**。  
AIはあなたの手を動かす代理人であり、あなたは「何を作るか」の意思決定者だ。

そのマインドセットと実際の手順を、Discord Botというわかりやすいお題で体験してほしい。

---

## 🎯 誰が読むべきか

### ✅ この講座が最も刺さる人

| タイプ | 説明 |
|--------|------|
| **JS中級者** | `async/await` は読めるが、本格的なBot開発は初めて |
| **AI使いたい初心者** | ChatGPTは使ったことあるが、開発への活かし方がわからない |
| **個人開発者** | Node.js / Discord.js で何か作りたいと思っている |
| **無課金戦士** | お金はかけたくないが、質は落としたくない |
| **バイブコーダー志望** | コードより設計・思考に時間を使いたい |

### ⚠️ 前提として持っていてほしいもの

- JavaScriptの基礎（`const`, `let`, `async/await` が読める）
- `npm install` で何かをインストールした経験がある
- Discordサーバーの管理者権限を持てる（テスト用）
- 「わからないことをAIに聞く」ことへの抵抗がない

### 🙅 この講座が向かない人

- プログラミング経験がゼロの方（まずJavaScript基礎から）
- 完全自動化・ノーコードを求める方（ある程度は手を動かします）
- 既にBot開発の経験が豊富な上級者（基礎部分が物足りないかも）

---

## 📚 全Partの構成と読み方

```
discord-bot-course-part1.md  ← まずここから読む（必須）
discord-bot-course-part2.md  ← AI連携ワークフローの実践
discord-bot-course-part3.md  ← 実際のコード（動かしたいならここを先に見ても可）
discord-bot-course-part4.md  ← GCPデプロイ（公開したいときに読む）
discord-bot-course-part5.md  ← 運用・育て方（長く使いたいときに読む）
discord-bot-course-part6.md  ← AIゼロから生成実践（最終到達点）
```

### 🔀 読み方のパターン

**「とりあえず動かしたい」人:**
```
Part 1（環境構築まで） → Part 3（コードをコピー） → 動かす → 詰まったらPart 2
```

**「ちゃんと理解したい」人:**
```
Part 1 → Part 2 → Part 3 → Part 4 → Part 5 → Part 6
```

**「GCPデプロイだけ知りたい」人:**
```
Part 4 だけ読む（前提はPart 1の環境構築セクション）
```

**「AIワークフローを極めたい」人:**
```
Part 2 → Part 6 → 実際に試す → Part 3でコードを見比べる
```

---

## 🔧 ファイル構成

```
discord-bot-course/
├── README.md                        ← 今読んでいるこれ
├── discord-bot-course-part1.md      ← 環境・思想・AI選定・IDE比較
├── discord-bot-course-part2.md      ← AI連携ワークフロー実践
├── discord-bot-course-part3.md      ← Botコード完全公開
├── discord-bot-course-part4.md      ← GCP無料枠デプロイ
├── discord-bot-course-part5.md      ← 運用・監視・スケール
└── discord-bot-course-part6.md      ← AIゼロ生成実践（最終章）
```

---

## 💡 この講座のユニークな点

### 1. AIを「使う」のではなく「組み合わせる」

単一のAIに頼るのではなく、ChatGPT・Gemini・ClaudeをそれぞれAIの「個性」に合わせて使い分ける。  
この「AI三角形」の使い方を体系化したのが本講座の核心だ。

### 2. ソースコードはあえて「参考」として提示

Part 3にコードは載っているが、本講座の本当のゴールはそれを写すことではない。  
**Part 6** で、AIと対話しながらゼロから自分のコードを生成させることが最終目標だ。  
Part 3のコードは「答え合わせ」のために使ってほしい。

### 3. 「詰まったらどうするか」まで書いてある

デプロイでの落とし穴、よくあるエラーとその原因、AIへのデバッグ依頼の仕方——  
実際に詰まるポイントを先回りして解説している。

### 4. 2026年3月時点で情報を最新化

AIサービスのモデル名・無料枠・IDEの比較など、古い情報ではなく今使える情報に基づいている。  
ただし、AIサービスは変化が早いので、常に公式ページも確認すること。

---

## ⚡ クイックスタート

```bash
# 1. このリポジトリをクローン（またはファイルをダウンロード）
git clone https://github.com/yourusername/discord-bot-course

# 2. Part 3 のサンプルBotをすぐ試したい場合
mkdir omikuji-bot && cd omikuji-bot
npm init -y
npm install discord.js dotenv

# 3. Part 3 のコードをコピーしてファイルを作成
# 4. .env に Discord トークンを設定
# 5. Bot を起動
node src/deploy-commands.js
node src/index.js
```

---

## 📅 更新について

この講座は2026年3月時点の情報に基づいている。  
AIサービスの仕様・無料枠・モデル名は頻繁に変わる。  
特に以下の情報は定期的に公式サイトで確認してほしい：

- [ChatGPT モデル情報](https://help.openai.com/en/articles/9624314-model-release-notes)
- [Gemini API 料金](https://ai.google.dev/gemini-api/docs/pricing)
- [GCP Always Free](https://cloud.google.com/free/docs/free-cloud-features)
- [Claude 料金](https://www.anthropic.com/pricing)

---

## 🤝 フィードバック・改善について

詰まった箇所、わかりにくかった箇所、情報が古くなった箇所があれば、  
本講座をベースに自分のBotを作りながら「Claudeに改善案を聞く」のが一番の練習になる。  
そのプロセス自体が、バイブコーディングエンジニアへの道だ。

---

**🎋 大吉。あなたのBot開発は必ずうまくいく。**
