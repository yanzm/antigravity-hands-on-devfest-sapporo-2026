# 配布プロンプト集

各ステップで Antigravity 2.0 に貼り付けるプロンプトです。`<PROJECT_ID>` は自分の Google Cloud プロジェクト ID に置き換えてください。

**モデル** : デフォルトの **Gemini 3.8 Flash (High)** のままで OK です（検証済み）。

**会話の分け方** : ステップごとに **New Conversation** で新しい会話を始めてください。失敗したときも、同じ会話で続けずに新しい会話でやり直してください。

---

## ステップ 1 : アプリを作る

> 実行前に `.agents/rules/workshop.md` がワークスペースに置いてあることを確認してください。

```
ゲストハウスの宿泊予約システムを作ってください。

## 機能（この 3 つだけ）
1. 部屋一覧ページ : 部屋の名前・定員・料金を表示
2. 空室確認 : チェックイン日・チェックアウト日を指定すると空いている部屋を表示
3. 予約登録 : 部屋・日付・宿泊者名を指定して予約を作成。既存予約と日程が重なる場合はエラーにする

## 技術要件
- Node.js + Express + EJS テンプレート（フロントエンドフレームワークは使わない）
- データは Google Cloud Firestore に保存（コレクション名 : rooms, reservations）
- rooms のドキュメントは name（文字列）, capacity（整数）, price（整数）のフィールドを持つ
- `npm start` で起動し、環境変数 PORT があればそのポートで 0.0.0.0 を listen する（デフォルトは 8080）
- Google Cloud プロジェクト ID : <PROJECT_ID>

## 注意
- 管理画面、ログイン、キャンセル機能は不要
- デザインはシンプルで良い
```

**期待される流れ** : 「Worked for 〜」のあとに **Implementation Plan** が出て **Proceed** ボタンが表示されます。内容をざっと見て Proceed を押してください。その後、`npm install` や Firestore MCP の呼び出しで承認ダイアログが何度か出ます。

> ※ 2026-09-09 に Python + FastAPI から Node.js + Express に変更（yoichiro さんの提案。事前準備のランタイムが Node だけになり、Cloud Run MCP のアップロードも 41MB → 約 10MB に減る）。**Node 版での生成は未検証**。再検証で Plan の内容・遵守状況を確認すること。

---

## ステップ 2 : 部屋データを入れる（Firestore MCP）

> アプリを起動して部屋一覧を開くと、まだ何も表示されません。エージェントに Firestore を直接操作させて部屋を登録します。

```
Firestore MCP を使って、rooms コレクションに以下の 4 部屋を登録してください。
ドキュメント ID は括弧内の値を使ってください。

- 男女共用ドミトリー (room-dormitory) : 定員 1 名、1 泊 3500 円
- 和室「さくら」 (room-sakura) : 定員 2 名、1 泊 6000 円
- 洋室「ツイン」 (room-twin) : 定員 2 名、1 泊 6500 円
- ファミリールーム「松」 (room-matsu) : 定員 4 名、1 泊 14000 円

登録後、rooms コレクションの内容を一覧して確認してください。
```

**期待される流れ** : `google-cloud-firestore/add_document` の承認ダイアログが出ます（「3 Yes, and always allow in this project」を選ぶと以降のダイアログが減ります）。終わったらブラウザをリロードして部屋が 4 つ表示されることを確認してください。Firestore コンソールでも見てみましょう。

---

## ステップ 3 : Cloud Run にデプロイする（Cloud Run MCP）

```
このフォルダを Cloud Run MCP を使って Cloud Run にデプロイしてください。
プロジェクト ID : <PROJECT_ID>、リージョン : asia-northeast1、
サービス名 : guesthouse-app、認証なしアクセスを許可してください。
デプロイ後の動作確認は私が行うので、curl や gcloud は実行しないでください。
```

**期待される流れ** : エージェントが Dockerfile と .dockerignore を作成し、`cloudrun/deploy_local_folder` の承認ダイアログが 1 回出ます。承認後 2〜3 分でサービス URL が表示されます。ブラウザで開いて、部屋一覧と予約が動くことを確認してください。

> ⚠ `Premature close` というエラーで失敗した場合は Node.js のバージョンが原因です（22.23.0 / 24.17.0 は NG）。エージェントに調査させず、`node -v` を確認してスタッフに声をかけてください。

---

## ステップ 4（15:00 のパートで使用）: ログを取らせる

```
guesthouse-app の直近のログを Cloud Run MCP で取得して、エラーや警告がないか確認してください。
警告があれば、それぞれ何が起きたのかを説明してください。
```

---

## 困ったときの対処

| 症状 | 原因 | 対処 |
|---|---|---|
| 入力欄に「⚠ MCP Error」、`sending "tools/call": Forbidden` | `roles/mcp.toolUser` が付いていない | 午前の手順で IAM ロールを付与。エージェントに調べさせない |
| デプロイで `Premature close` | Node.js 22.23.0 / 24.17.0 の回帰バグ | Node を 22.23.1 以降 or 24.18 以降に更新し、Antigravity を再起動 |
| 承認ダイアログが連発して `gcloud auth` や `~/.zsh_history` を読もうとする | エージェントがエラーの原因調査を始めた | **「5 No」で止めて、新しい会話でやり直す** |
| 「Allow read access to this path?」でプロジェクトの外のパスが出る | エージェントが親フォルダを読もうとしている | プロジェクトフォルダは `~/Projects/<名前>` のような独立した場所に作る |
