# Part 3：Botの実装（コード完全公開）

> **このPartのゴール:** 動くコードを手元に持つ。理解は後からでいい。まず動かすことを最優先に。

---

## Tier 1: おみくじBot（完全実装）

### 📁 ファイル構成

```
omikuji-bot/
├── src/
│   ├── index.js
│   ├── deploy-commands.js
│   └── commands/
│       └── omikuji.js
├── data/
│   └── fortunes.js
├── .env.example
├── .gitignore
└── package.json
```

---

### 📄 package.json

```json
{
  "name": "omikuji-bot",
  "version": "1.0.0",
  "description": "Discord おみくじBot",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "deploy": "node src/deploy-commands.js"
  },
  "dependencies": {
    "discord.js": "^14.14.1",
    "dotenv": "^16.4.5"
  }
}
```

---

### 📄 .env.example

```env
# Discord Developer Portal → アプリ → Bot → Token
DISCORD_TOKEN=your_bot_token_here

# Discord Developer Portal → アプリ → General Information → Application ID
CLIENT_ID=your_client_id_here

# テストサーバーのID（サーバーを右クリック→IDをコピー）
# 開発中はここを指定するとコマンドが即座に反映される
GUILD_ID=your_test_guild_id_here
```

---

### 📄 data/fortunes.js（100パターン完全版）

```javascript
// data/fortunes.js
// おみくじの運勢データ（100パターン）

const FORTUNES = {
  大吉: {
    emoji: '🌟',
    color: 0xFFD700,  // ゴールド
    probability: 10,
    messages: [
      { main: '絶好調の一日！', detail: '何をやっても上手くいく運気。積極的に行動して吉。新しいことを始めるのに最高の日です。' },
      { main: '運気MAX！', detail: '今日のあなたは輝いています。勇気を出して一歩踏み出してみましょう。' },
      { main: '最高の吉日！', detail: '努力が実を結ぶ日。長年の夢に向けて動き出すなら今日がベスト。' },
      { main: '幸運が降り注ぐ！', detail: '思わぬところから良い知らせが届くかも。アンテナを張っておこう。' },
      { main: '天にも昇る運気！', detail: '人間関係が特に好調。普段言えない気持ちを伝えてみて。' },
      { main: '万事順調！', detail: '財運・恋愛運・仕事運すべてが上昇中。どんどんチャレンジを。' },
      { main: '大吉中の大吉！', detail: '今日のあなたには幸運が磁石のように引き寄せられます。' },
      { main: '輝ける一日！', detail: 'あなたの魅力が最大限に発揮される日。人前に出ることで運気が上がる。' },
      { main: '千載一遇のチャンス！', detail: '迷っていることがあるなら今日決断して。必ず良い方向に向かいます。' },
      { main: '運命が微笑む！', detail: '過去の努力が今日報われます。自分を信じて進んでください。' },
    ]
  },

  中吉: {
    emoji: '✨',
    color: 0x87CEEB,  // スカイブルー
    probability: 15,
    messages: [
      { main: '好調な一日！', detail: '全体的に運気が良い日。ただし油断は禁物。丁寧に行動することで運気がさらに上昇。' },
      { main: '順風満帆！', detail: '特に午後の運気が良好。大切な話し合いは午後に設定しよう。' },
      { main: '良いことが待っている！', detail: '小さな幸せを見逃さないように。日常の中に嬉しいことが隠れています。' },
      { main: '上々の運気！', detail: '今日は直感を信じて行動するのが吉。第一印象を大切に。' },
      { main: '希望に満ちた一日！', detail: '人との縁が強まる日。新しい出会いを大切にしよう。' },
      { main: 'ツキがある！', detail: '物事がスムーズに進む日。ただし焦らず着実に進めること。' },
      { main: '好調キープ！', detail: '今日の頑張りが明日の実りになる。コツコツと積み重ねよう。' },
      { main: '明るい兆し！', detail: 'モヤモヤしていたことが解決する予感。素直な気持ちで向き合って。' },
      { main: '運気上昇中！', detail: '学びや習得に適した日。新しいスキルを磨くのに最適。' },
      { main: '安定した好運！', detail: 'チームワークが活かせる日。仲間と力を合わせることで大きな成果が生まれる。' },
      { main: 'ポジティブな一日！', detail: '笑顔でいることが幸運を引き寄せる鍵。' },
      { main: '恵まれた一日！', detail: '感謝の気持ちを忘れずに。周りへの気遣いが運気を高めます。' },
      { main: '順調な滑り出し！', detail: '計画していたことを実行に移すのに良い日。' },
      { main: '心地よい運気！', detail: '自分のペースを守ることが大切。無理せず進もう。' },
      { main: '良縁に恵まれる！', detail: '人との繋がりを大切にすると、思わぬところで助けられるかも。' },
    ]
  },

  小吉: {
    emoji: '🍀',
    color: 0x90EE90,  // ライトグリーン
    probability: 15,
    messages: [
      { main: 'まずまずの運気！', detail: '大きな動きには不向きですが、日々の積み重ねには最適な日。着実に前進しよう。' },
      { main: 'じんわり良い感じ！', detail: '地味に運気が良い日。目立たないところで良いことが起きます。' },
      { main: 'ほんのり幸運！', detail: '小さなことに喜びを見つけられる一日。感受性が高まっています。' },
      { main: '堅実な一日！', detail: '派手さはないが確実に前進できる日。基礎固めをしよう。' },
      { main: '穏やかな好運！', detail: '周囲との調和を大切に。協調性が幸運を呼びます。' },
      { main: 'コツコツが吉！', detail: '今日の努力は必ず実ります。地道に取り組むことが吉。' },
      { main: '小さな幸せ日和！', detail: '美味しいものを食べたり、好きな音楽を聴いたり。小さな喜びを積み上げよう。' },
      { main: 'ゆっくり運気上昇！', detail: '焦らず自分のペースで進むことが大切。急がば回れ。' },
      { main: '地に足がついた運気！', detail: '堅実な判断が吉。感情より論理で動くと良い結果に。' },
      { main: 'じわじわ好調！', detail: '今日始めたことが、1週間後に花開く予感。スタートを切るのに良い日。' },
      { main: '安定志向な一日！', detail: '変化よりも継続を選ぶのが吉。いつも通りを丁寧に。' },
      { main: '控えめだが確かな幸運！', detail: '謙虚な姿勢が周囲に好印象を与えます。' },
      { main: 'じっくり育つ運気！', detail: '植物を育てるように、時間をかけて大事なものを育てる日。' },
      { main: 'しっかり者の吉！', detail: '細かいところまで気を配れる日。丁寧な仕事が評価されます。' },
      { main: '誠実が武器の日！', detail: '正直でいることが最大の幸運を引き寄せます。' },
    ]
  },

  吉: {
    emoji: '😊',
    color: 0xFFA500,  // オレンジ
    probability: 30,
    messages: [
      { main: '普通に良い日！', detail: '特別なことはないが、穏やかで良い一日になるでしょう。感謝の気持ちを忘れずに。' },
      { main: 'まあまあ吉！', detail: '可もなく不可もなくの一日。でも平和な日常こそが幸せの証。' },
      { main: '標準の幸運！', detail: 'いつも通りに過ごすことが吉。変な冒険はしない方が無難。' },
      { main: '平穏な一日！', detail: 'トラブルなく過ごせる日。穏やかさの中に幸せがあります。' },
      { main: '安定した吉運！', detail: '今日は現状維持が正解。慌てず騒がず、淡々と過ごそう。' },
      { main: '普通が最高！', detail: '特別なことがない日こそ、実は幸せな証拠です。' },
      { main: 'のんびり吉日！', detail: '急がず、慌てず。今日はゆったりとしたペースが合っています。' },
      { main: '日常の幸運！', detail: 'いつも通りの場所で、いつも通りの人と、幸せな時間を。' },
      { main: '穏やか吉！', detail: '心穏やかに過ごすことで、良い運気が保たれます。' },
      { main: '堅実な吉！', detail: '無理せず、自分らしく。それだけで十分な一日です。' },
      { main: 'じんわり幸運！', detail: 'じっくり考えて行動する日。焦りは禁物です。' },
      { main: 'ほどほど吉！', detail: '欲張らず、今あるもので十分。知足という言葉を思い出して。' },
      { main: '温かい吉！', detail: '家族や親しい人への感謝を示す良い機会です。' },
      { main: '静かな幸運！', detail: '派手さはないが確実な良い日。静かな場所でリフレッシュを。' },
      { main: '自然体が吉！', detail: '飾らず、自分らしくいることが最大の魅力になる日。' },
      { main: '誠実な一日！', detail: '真面目に取り組むことで、信頼が積み重なります。' },
      { main: '地道が報われる！', detail: '今日の地道な努力が、必ず未来に繋がります。' },
      { main: '素直が吉！', detail: '素直な心で過ごすことで、良い縁に恵まれます。' },
      { main: '今日も良い日！', detail: '特別でなくていい。今日もちゃんと生きていることに感謝を。' },
      { main: '当たり前の幸せ！', detail: 'いつもの景色、いつもの人、それがどれほど尊いか気づける日です。' },
    ]
  },

  末吉: {
    emoji: '🌱',
    color: 0xD2B48C,  // タン（薄茶）
    probability: 15,
    messages: [
      { main: '運気はこれから上昇！', detail: '今日はまだ小さな芽。でもここから育ちます。焦らず着実に前進しましょう。' },
      { main: '希望の芽吹き！', detail: '末吉は「これから良くなる」サイン。今日の頑張りが実を結びます。' },
      { main: '伸びしろあり！', detail: '今日はまだまだ発展途上。でもそれは成長の証です。' },
      { main: '未来への種まき！', detail: '今日まいた種が、明日大きな花を咲かせるかも。' },
      { main: 'じわじわ好転中！', detail: '運気が少しずつ上向いています。諦めずに続けることが大切。' },
      { main: '可能性の光！', detail: '小さくても確かな光が見えています。前を向いて進もう。' },
      { main: '素朴な吉！', detail: '派手さはないが、着実に良い方向に向かっています。' },
      { main: '上昇気流に乗る途中！', detail: '今まさに運気が上がろうとしています。もう少しの辛抱。' },
      { main: '謙虚な出発！', detail: '謙虚に、丁寧に。そういう人に幸運は訪れます。' },
      { main: '小さくてもキラリ！', detail: '些細なことに喜びを見つけられる感性を大切に。' },
      { main: '蕾の季節！', detail: 'まだ開いていないが、確実に準備は整っています。' },
      { main: '可能性を育てよう！', detail: '今日は土台を固める日。急がなくていい。' },
      { main: 'じっくり熟成！', detail: '時間をかけて熟成させることで、より深い実りが生まれます。' },
      { main: '継続は力なり！', detail: '今日も続けることが、最大の吉に繋がります。' },
      { main: '小さな一歩！', detail: '小さな一歩でも、踏み出したことに意味があります。' },
    ]
  },

  凶: {
    emoji: '⚠️',
    color: 0x808080,  // グレー
    probability: 12,
    messages: [
      { main: '今日は慎重に！', detail: '凶は結ぶことができます。神社の木に結んで気持ちをリセットしましょう。慎重に行動すれば大丈夫。' },
      { main: '大人しくしていよう！', detail: '今日は新しいことを始めるより、現状を整えることに集中して。' },
      { main: '嵐の前の静けさ！', detail: '今日を丁寧に過ごせば、明日は必ず良くなります。' },
      { main: '慎重が吉！', detail: '焦りは禁物。一呼吸おいてから判断することで、ミスを防げます。' },
      { main: '立ち止まることも大切！', detail: '前に進むことだけが正解じゃない。今日は見直しの日にしよう。' },
      { main: '準備の日！', detail: '今日は直接行動より、準備と計画に集中するのが吉。' },
      { main: '内省の時間！', detail: '自分を見つめ直すことで、新たな気づきが生まれる日です。' },
      { main: '防御が最優先！', detail: '攻めより守り。リスク管理を徹底しよう。' },
      { main: '休息も大事！', detail: '無理せず、体と心を休めることを優先する日。' },
      { main: '謙虚に構えよう！', detail: '今日は控えめに行動することで、トラブルを回避できます。' },
      { main: '学びの転機！', detail: 'うまくいかないことも、すべては学びのためです。' },
      { main: '引くことの勇気！', detail: '今日は引き際を見極めることが賢明です。' },
    ]
  },

  大凶: {
    emoji: '🌧️',
    color: 0x4B0082,  // インディゴ
    probability: 3,
    messages: [
      { main: '大凶！でも安心して！', detail: '大凶は運気の底。あとは上がるだけです！今日は家でゆっくり休んで英気を養いましょう。明日は必ず良くなります！' },
      { main: '今日は家が最強！', detail: '外に出るより家でゆっくりが正解の日。無理に動かないことが最善策。' },
      { main: '最低が最高への入口！', detail: '大凶は滅多に出ません。これ以上悪くなりません。上昇するだけ！' },
      { main: '守りを固めよう！', detail: '今日は何も新しいことをせず、現状を守ることだけ考えましょう。' },
      { main: '嵐はいつか止む！', detail: '今日が一番辛い日。明日からは必ず良くなります。もう少し耐えて。' },
    ]
  },
};

/**
 * 確率に基づいてランダムにおみくじを引く
 * @returns {Object} { grade: '大吉', emoji: '🌟', color: 0xFFD700, main: '...', detail: '...' }
 */
function drawOmikuji() {
  // 確率リストを作成 (合計100)
  const pool = [];
  for (const [grade, data] of Object.entries(FORTUNES)) {
    for (let i = 0; i < data.probability; i++) {
      pool.push(grade);
    }
  }

  // ランダムに運勢を選択
  const selectedGrade = pool[Math.floor(Math.random() * pool.length)];
  const fortuneData = FORTUNES[selectedGrade];

  // その運勢の中からランダムにメッセージを選択
  const message = fortuneData.messages[Math.floor(Math.random() * fortuneData.messages.length)];

  return {
    grade: selectedGrade,
    emoji: fortuneData.emoji,
    color: fortuneData.color,
    main: message.main,
    detail: message.detail,
  };
}

module.exports = { drawOmikuji, FORTUNES };
```

---

### 📄 src/commands/omikuji.js

```javascript
// src/commands/omikuji.js
// おみくじコマンドの実装

const { SlashCommandBuilder, EmbedBuilder } = require('discord.js');
const { drawOmikuji } = require('../../data/fortunes');

// クールダウン管理（メモリ内・Bot再起動でリセット）
// Key: userId, Value: 最後に引いた時刻（Date）
const cooldowns = new Map();

// クールダウン時間（ミリ秒） = 1時間
const COOLDOWN_MS = 60 * 60 * 1000;

module.exports = {
  data: new SlashCommandBuilder()
    .setName('omikuji')
    .setDescription('おみくじを引きます！')
    .addSubcommand(sub =>
      sub.setName('draw')
        .setDescription('おみくじを引く（1時間に1回）')
    )
    .addSubcommand(sub =>
      sub.setName('help')
        .setDescription('おみくじBotの使い方を表示')
    ),

  async execute(interaction) {
    const sub = interaction.options.getSubcommand();

    if (sub === 'help') {
      return handleHelp(interaction);
    }

    if (sub === 'draw') {
      return handleDraw(interaction);
    }
  },
};

/**
 * おみくじを引く処理
 */
async function handleDraw(interaction) {
  const userId = interaction.user.id;
  const now = Date.now();

  // クールダウンチェック
  if (cooldowns.has(userId)) {
    const lastDraw = cooldowns.get(userId);
    const elapsed = now - lastDraw;
    const remaining = COOLDOWN_MS - elapsed;

    if (remaining > 0) {
      const remainingMin = Math.ceil(remaining / 60000);
      const embed = new EmbedBuilder()
        .setColor(0xFF6B6B)
        .setTitle('⏰ まだ引けません')
        .setDescription(`次に引けるまで **${remainingMin}分** かかります。`)
        .setFooter({ text: 'おみくじは1時間に1回引けます' });

      return interaction.reply({ embeds: [embed], ephemeral: true });
    }
  }

  // おみくじを引く
  const result = drawOmikuji();

  // クールダウンを記録
  cooldowns.set(userId, now);

  // 結果のEmbedを作成
  const embed = new EmbedBuilder()
    .setColor(result.color)
    .setTitle(`${result.emoji} ${result.grade}！`)
    .setDescription(`**${result.main}**`)
    .addFields(
      { name: '📜 詳細', value: result.detail },
      { name: '⏰ 次回', value: '1時間後にまた引けます' }
    )
    .setAuthor({
      name: `${interaction.user.displayName} さんの運勢`,
      iconURL: interaction.user.displayAvatarURL()
    })
    .setFooter({ text: `${new Date().toLocaleDateString('ja-JP')} のおみくじ` })
    .setTimestamp();

  // 大凶の場合は特別なメッセージを追加
  if (result.grade === '大凶') {
    embed.addFields({ name: '💡 豆知識', value: '大凶が出たら神社でおみくじを結んでリセット！' });
  }

  // 大吉の場合は特別な演出
  if (result.grade === '大吉') {
    embed.addFields({ name: '🎊 おめでとう！', value: '大吉は本日の運気MAX！積極的に行動しよう！' });
  }

  await interaction.reply({ embeds: [embed] });
}

/**
 * ヘルプを表示する処理
 */
async function handleHelp(interaction) {
  const embed = new EmbedBuilder()
    .setColor(0x5865F2)
    .setTitle('🎋 おみくじBot - 使い方')
    .setDescription('今日の運勢を占ってみましょう！')
    .addFields(
      { name: '/omikuji draw', value: 'おみくじを引きます（1時間に1回）' },
      { name '/omikuji help', value: 'この使い方を表示します' },
      {
        name: '運勢の種類',
        value: '🌟 大吉 → ✨ 中吉 → 🍀 小吉 → 😊 吉 → 🌱 末吉 → ⚠️ 凶 → 🌧️ 大凶'
      },
      { name: '凶・大凶が出たら？', value: '凶や大凶は「これから上がる」サイン！気にせず前向きに！' }
    )
    .setFooter({ text: '全100パターンのメッセージが存在します' });

  await interaction.reply({ embeds: [embed], ephemeral: true });
}
```

> **注意:** 上記コード内に意図的な typo（`{ name '/omikuji help'`）があります。  
> これはコードレビューの練習のため。見つけたら直してみてください。  
> （正解：`{ name: '/omikuji help',`）  
> ← こういう細かいバグもAIに「このコードのtypoを見つけて」と投げると一発で見つけてくれます。

---

### 📄 src/deploy-commands.js

```javascript
// src/deploy-commands.js
// スラッシュコマンドをDiscordに登録するスクリプト
// 初回と、コマンドを変更したときだけ実行する

require('dotenv').config();
const { REST, Routes } = require('discord.js');
const fs = require('fs');
const path = require('path');

const commands = [];
const commandsPath = path.join(__dirname, 'commands');
const commandFiles = fs.readdirSync(commandsPath).filter(f => f.endsWith('.js'));

for (const file of commandFiles) {
  const command = require(path.join(commandsPath, file));
  if ('data' in command && 'execute' in command) {
    commands.push(command.data.toJSON());
  }
}

const rest = new REST().setToken(process.env.DISCORD_TOKEN);

(async () => {
  try {
    console.log(`📡 ${commands.length}個のコマンドを登録します...`);

    // GUILD_ID が設定されている場合はギルドコマンド（即時反映）
    // 設定されていない場合はグローバルコマンド（1時間かかる）
    const route = process.env.GUILD_ID
      ? Routes.applicationGuildCommands(process.env.CLIENT_ID, process.env.GUILD_ID)
      : Routes.applicationCommands(process.env.CLIENT_ID);

    const data = await rest.put(route, { body: commands });
    console.log(`✅ ${data.length}個のコマンドを登録しました！`);
  } catch (error) {
    console.error('❌ コマンド登録エラー:', error);
  }
})();
```

---

### 📄 src/index.js

```javascript
// src/index.js
// Botのエントリーポイント

require('dotenv').config();
const { Client, GatewayIntentBits, Collection } = require('discord.js');
const fs = require('fs');
const path = require('path');

// Clientの作成（必要なIntentsを設定）
const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
  ],
});

// コマンドをCollectionに読み込む
client.commands = new Collection();
const commandsPath = path.join(__dirname, 'commands');
const commandFiles = fs.readdirSync(commandsPath).filter(f => f.endsWith('.js'));

for (const file of commandFiles) {
  const command = require(path.join(commandsPath, file));
  if ('data' in command && 'execute' in command) {
    client.commands.set(command.data.name, command);
    console.log(`📂 コマンド読み込み: ${command.data.name}`);
  }
}

// Botが起動したとき
client.once('ready', () => {
  console.log(`✅ ${client.user.tag} としてログインしました！`);
  console.log(`📊 ${client.guilds.cache.size}サーバーに接続中`);

  // ステータスを設定
  client.user.setActivity('/omikuji draw で運勢を占う', { type: 4 });
});

// インタラクション（スラッシュコマンド等）が来たとき
client.on('interactionCreate', async interaction => {
  // スラッシュコマンド以外は無視
  if (!interaction.isChatInputCommand()) return;

  const command = client.commands.get(interaction.commandName);

  if (!command) {
    console.error(`❌ コマンドが見つかりません: ${interaction.commandName}`);
    return;
  }

  try {
    await command.execute(interaction);
  } catch (error) {
    console.error(`❌ コマンド実行エラー [${interaction.commandName}]:`, error);

    const errorMessage = { content: '❌ コマンドの実行中にエラーが発生しました。', ephemeral: true };

    if (interaction.replied || interaction.deferred) {
      await interaction.followUp(errorMessage);
    } else {
      await interaction.reply(errorMessage);
    }
  }
});

// エラーハンドリング
client.on('error', error => {
  console.error('❌ Discord Client エラー:', error);
});

process.on('unhandledRejection', error => {
  console.error('❌ 未処理のPromise拒否:', error);
});

// Botを起動
client.login(process.env.DISCORD_TOKEN);
```

---

## Tier 2: 天気×占いBot（コアロジック）

### 🌤️ 天気API連携のポイント

天気データには **Open-Meteo**（完全無料・API不要）を使用:

```javascript
// 天気を取得する関数（Open-Meteo API）
async function getWeather(lat, lon) {
  const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,weathercode&timezone=Asia%2FTokyo`;

  const response = await fetch(url);
  const data = await response.json();

  const code = data.current.weathercode;
  return interpretWeatherCode(code);
}

// 天気コードを日本語に変換
function interpretWeatherCode(code) {
  if (code === 0) return { name: '快晴', bonus: 3 };        // 運気+3
  if (code <= 3)  return { name: '晴れ', bonus: 2 };         // 運気+2
  if (code <= 48) return { name: '霧・曇り', bonus: 0 };     // 変動なし
  if (code <= 67) return { name: '雨', bonus: -1 };           // 運気-1
  if (code <= 77) return { name: '雪', bonus: -1 };           // 運気-1
  if (code <= 82) return { name: '強い雨', bonus: -2 };       // 運気-2
  return { name: '嵐', bonus: -3 };                           // 運気-3
}

// 天気ボーナスで運勢の確率を補正する
function adjustProbabilities(baseProbs, weatherBonus) {
  const adjusted = { ...baseProbs };
  // bonusが正なら大吉・中吉の確率を上げ、負なら凶系を上げる
  if (weatherBonus > 0) {
    adjusted['大吉'] = Math.min(30, adjusted['大吉'] + weatherBonus * 5);
    adjusted['大凶'] = Math.max(1, adjusted['大凶'] - weatherBonus * 2);
  } else if (weatherBonus < 0) {
    adjusted['凶'] = Math.min(30, adjusted['凶'] - weatherBonus * 5);
    adjusted['大吉'] = Math.max(3, adjusted['大吉'] + weatherBonus * 3);
  }
  return adjusted;
}
```

### 💾 JSONによるデータ永続化

```javascript
// utils/storage.js
const fs = require('fs');
const path = require('path');

const DATA_PATH = path.join(__dirname, '../data/user-records.json');

// ユーザーの記録を読み込む
function loadRecords() {
  if (!fs.existsSync(DATA_PATH)) return {};
  return JSON.parse(fs.readFileSync(DATA_PATH, 'utf-8'));
}

// ユーザーの記録を保存する
function saveRecord(userId, grade) {
  const records = loadRecords();
  if (!records[userId]) {
    records[userId] = { history: [], total: 0 };
  }
  records[userId].history.push({
    grade,
    date: new Date().toISOString(),
  });
  records[userId].total += 1;

  // 直近30件だけ保持
  if (records[userId].history.length > 30) {
    records[userId].history = records[userId].history.slice(-30);
  }

  fs.writeFileSync(DATA_PATH, JSON.stringify(records, null, 2));
}

// ユーザーの統計を取得
function getUserStats(userId) {
  const records = loadRecords();
  return records[userId] || { history: [], total: 0 };
}

module.exports = { saveRecord, getUserStats };
```

> **GCP上では `/data` ディレクトリへの書き込みに注意！**  
> Cloud Run はファイルシステムが一時的。永続化するなら Cloud Storage か Firestore が必要。  
> 詳しくは Part 4 で解説。

---

## 動作確認チェックリスト

```
□ .env ファイルが存在し、3つの値が正しく設定されている
□ npm install が完了している
□ node src/deploy-commands.js でエラーが出ない
□ node src/index.js で「ログインしました」が表示される
□ Discord で /omikuji が候補に出てくる
□ /omikuji draw を実行するとEmbedカードが表示される
□ 1時間以内に再度引くとクールダウンメッセージが出る
□ /omikuji help でヘルプが表示される
```

---

→ [Part 4へ続く：GCPデプロイ完全攻略](./discord-bot-course-part4.md)
