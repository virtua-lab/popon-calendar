# 開発ルール・設計方針 (2026/01/26 制定)

このプロジェクトにおけるデータ管理の絶対ルールです。
今後の機能追加・修正時は必ずこれを守ること。

## ⚠️ データ永続化の鉄則

**「UI（入力欄）をデータの保存場所にしないこと」**

### ❌ 悪い例（過去の失敗）
データ読み込み時、`localStorage` から直接 `input.value` に入れようとする。
```javascript
// ❌ 画面が表示されていないと input 要素が見つからず、失敗する
// ❌ タイミングによっては空文字で上書き保存してしまう
function loadData() {
    document.getElementById('myInput').value = localStorage.getItem('data');
}
```

### ⭕️ 正しい設計
データは必ず**「グローバル変数（アプリケーションデータ）」**として管理し、プログラム開始直後に読み込む。
UI（入力欄）はあくまでデータを表示・編集するための窓口にすぎない。

```javascript
// ⭕️ 1. プログラム開始直後にデータを変数にロード（UIの状態に関係なく成功する）
let appData = localStorage.getItem('data') || '';

// ⭕️ 2. 画面表示時、変数の値を入力欄に反映する
function render() {
    const input = document.getElementById('myInput');
    if (input) input.value = appData;
}

// ⭕️ 3. 保存時は、変数を更新してから localStorage に書く
function save(val) {
    appData = val;
    localStorage.setItem('data', appData);
}
```

---

## 現状のデータ構造
以下のデータはこのルールに従って実装されています。

1.  **スケジュールデータ (`poponSchedules`, `vspSchedules`)**
    *   スクリプト先頭でロード済み。
2.  **登録済みリスト (`registeredItems`)**
    *   スクリプト先頭でロード済み。
3.  **Discord設定 (`discordSettings`)**
    *   スクリプト先頭でロード済み（**今回修正**）。

今後設定項目を増やす場合も、必ず `discordSettings` オブジェクトに追加するか、同様のパターンで変数を定義すること。
