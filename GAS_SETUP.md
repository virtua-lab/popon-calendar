# 🤖 企画別 自動通知ロボットの作り方

社長、お待たせしました！
今度は**「POP ONの予定はPOP ONの部屋へ、ぶいしくぱの予定はぶいしくぱの部屋へ」**ときちんと振り分ける、ちょっと賢いロボットを作ります。

手順は前回とほぼ同じですが、**命令文（コード）だけ新しくなります！**

---

## 1. 宛先を2つ用意する

Discordでそれぞれの部屋（チャンネル）のウェブフックURLをコピーしてきてください。
メモ帳か何かに貼っておくと便利です。

1.  **POP ON用の部屋のURL**
2.  **ぶいしくぱ用の部屋のURL**

---

## 2. ロボットの命令文を書き換える

Google Apps Scriptの画面を開き、前のコードを消して、以下のコードを貼り付けてください。

```javascript
// 👇👇👇 設定エリア 👇👇👇

// 1. POP ON 用の宛先（URL）
const POPON_WEBHOOK = "ここにPOPON用のURLを貼り付けてください";

// 2. ぶいしくぱ 用の宛先（URL）
const VSP_WEBHOOK = "ここにぶいしくぱ用のURLを貼り付けてください";

// 👆👆👆 設定ここまで 👆👆👆


// === これより下は触らなくて大丈夫です ===

function notifyDiscord() {
  const today = new Date();
  const calendar = CalendarApp.getDefaultCalendar();
  const events = calendar.getEventsForDay(today);

  if (events.length === 0) return; // 予定なしなら終了

  // 予定を振り分ける箱を用意
  let poponMsg = "☀️ **本日の予定 (POP ON)**\n";
  let vspMsg   = "☀️ **本日の予定 (ぶいしくぱ)**\n";
  
  let hasPopon = false;
  let hasVsp   = false;

  // 予定をひとつずつチェック
  for (let i = 0; i < events.length; i++) {
    const title = events[i].getTitle();
    
    // タイトルを見て振り分け
    if (title.includes("POP ON")) {
      poponMsg += `・${title.replace("【POP ON】", "")}\n`; // 「【POP ON】」は見づらいので消しておく
      hasPopon = true;
    } 
    else if (title.includes("ぶいしくぱ")) {
      vspMsg += `・${title.replace("【ぶいしくぱ】", "")}\n`;
      hasVsp = true;
    }
    // どちらでもない場合（その他の予定）はどうするか？
    // 今回は一旦無視するか、両方に送るなどの調整が可能ですが、
    // 今のツールから登録したものは必ずどちらかの名前がついているはずです。
  }

  // POP ONの予定があれば通知
  if (hasPopon) {
    sendToDiscord(POPON_WEBHOOK, poponMsg);
  }

  // ぶいしくぱの予定があれば通知
  if (hasVsp) {
    sendToDiscord(VSP_WEBHOOK, vspMsg);
  }
}

// 送信するための裏方さん機能
function sendToDiscord(url, content) {
  if (!url || url.includes("ここにURL")) return; // URLが未設定なら何もしない

  const payload = { content: content };
  try {
    UrlFetchApp.fetch(url, {
      method: "post",
      contentType: "application/json",
      payload: JSON.stringify(payload)
    });
  } catch (e) {
    console.log("送信失敗: " + e);
  }
}
```

---

## 3. URLを貼り付ける

コードの一番上にある2つの項目を書き換えてください。

1.  `POPON_WEBHOOK` の `"ここ..."` に、POP ON用のURLを貼り付け。
2.  `VSP_WEBHOOK` の `"ここ..."` に、ぶいしくぱ用のURLを貼り付け。

---

## 4. 保存して完了！

**「💾 保存」** を押せば完了です！
（既にタイマー設定済みであれば、明日の朝から自動で振り分けられて通知されます）

もし「テスト送信」してみたい場合は、Googleカレンダーに
*   「【POP ON】テスト」
*   「【ぶいしくぱ】テスト」
というタイトルの予定を今日のところに作ってから、`notifyDiscord` を実行してみてください。
