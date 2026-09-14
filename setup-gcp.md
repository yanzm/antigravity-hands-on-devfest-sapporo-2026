# Google Cloud のセットアップ

ワークショップで使う Google Cloud 側の準備です。**当日の午前（11:05-12:00）に、講師の説明を聞きながら自分の手で 1 つずつ進めます。** 各ステップに「何をしているのか」を書いてあるので、当日はそれを理解しながら実行してください。

事前にやってきてほしいのは、当日やると待ち時間が発生する「0. 事前準備」だけです。

<span style="color:salmon"><b>※ 会社や学校の Google Workspace アカウントではなく、個人の Google アカウント（@gmail.com）で行ってください。</b></span>

---

## 0. 事前準備（当日までに）

| 準備 | なぜ事前か |
| --- | --- |
| **課金アカウント**（クレジットカード登録） | 本人確認や承認待ちが入ることがあり、当日中に終わらない可能性がある |
| **gcloud CLI のインストール** | ダウンロードが大きく、会場の回線では時間がかかる |
| **Node.js**（22.23.1 以降 or 24.18 以降） | 同上。Cloud Run MCP とアプリ本体の両方で使います |

### 課金アカウント

Google Cloud のほとんどのサービスは、無料枠の範囲内であっても **課金アカウントの紐付けが必須** です。今回作るものは無料枠（Always Free）に収まる見込みですが、クレジットカードの登録は必要です。

1. https://console.cloud.google.com/billing を開く
2. 課金アカウントがあればそのまま OK。なければ **[お支払いアカウントを作成]**
   - 初めて Google Cloud を使う方は、この流れで **無料トライアル（$300 分のクレジット）** が有効になります

### gcloud CLI

https://cloud.google.com/sdk/docs/install にしたがってインストールしてください。ログインは当日一緒にやるので不要です（先にやっておいても構いません）。

### 確認

```
gcloud version
node -v
```

<span style="color:salmon">Node.js が 22.23.0 または 24.17.0 だった場合は、別のバージョンに更新してください</span>（Node 本体の不具合で、午後のデプロイが必ず失敗します）。

---

## 1. gcloud にログインする（当日）

ターミナルで以下を **両方** 実行します。どちらもブラウザが開くので、個人の Google アカウントでログインしてください。

```
gcloud auth login
```

```
gcloud auth application-default login
```

**何をしているか** : 2 つは別物です。

| コマンド | 誰のための認証か |
| --- | --- |
| `gcloud auth login` | **gcloud コマンド自体**が使う（このあと打つコマンドのため） |
| `gcloud auth application-default login` | **あなたが書いたプログラムや、エージェントのツール**が Google Cloud にアクセスするときに使う（ADC = Application Default Credentials） |

午後は、エージェントが作ったアプリと MCP の両方が ADC を使って Firestore や Cloud Run に繋がります。片方だけやって「認証したのに繋がらない」となるのが定番の詰まりポイントです。

---

## 2. プロジェクトを作って課金アカウントを紐付ける（当日）

まず、自分の課金アカウントの ID を確認します。

```
gcloud billing accounts list
```

`ACCOUNT_ID` の列（`XXXXXX-XXXXXX-XXXXXX` の形式）をメモしてください。

次にプロジェクトを作ります。プロジェクト ID は世界中で一意である必要があるので、自分の名前などを入れてください（例 : `devfest-guesthouse-yanzm`）。小文字・数字・ハイフンのみ、6〜30 文字です。

```
gcloud projects create <プロジェクトID> --name="DevFest Guesthouse"
gcloud config set project <プロジェクトID>
gcloud billing projects link <プロジェクトID> --billing-account=<課金アカウントID>
```

**何をしているか** :

- **プロジェクト**は Google Cloud のリソース（データベース、サーバー、ログ…）をまとめる箱です。今日作るものは全部この箱に入り、最後に箱ごと削除します
- `config set project` は「これ以降の gcloud コマンドはこのプロジェクトに対して実行する」という設定です
- **課金アカウントの紐付け**をしないと、Firestore も Cloud Run も使えません（無料枠内でも同じ）

> **Note** : 「プロジェクト ID がすでに使われている」と言われたら、別の ID にしてください。

---

## 3. Firestore データベースを作る（当日）

```
gcloud services enable firestore.googleapis.com
gcloud firestore databases create --database="(default)" --location=asia-northeast1 --type=firestore-native
```

1 分ほどかかります。

**何をしているか** :

- 1 行目は「このプロジェクトで Firestore の API を使えるようにする」です。Google Cloud のサービスは、使う前にプロジェクトごとに API を有効化する必要があります
- 2 行目がデータベースの作成です。午後に作る予約システムのデータ（部屋・予約）はここに入ります
- **`asia-northeast1`（東京）** はデータの置き場所です。<span style="color:salmon">後から変更できません</span>。午後の手順は全員がこのリージョンである前提で書かれています
- **Native mode** は Firestore の 2 つのモードのうちの 1 つです。もう 1 つの Datastore mode を選ぶと API が別物になり、午後の手順が動きません

https://console.cloud.google.com/firestore を開いて、空のデータベースが見えることを確認してください。午後、エージェントがここにデータを入れるのを目で見ることになります。

---

## 4. 自分にロールを付ける（当日）

```
gcloud projects add-iam-policy-binding <プロジェクトID> --member="user:<自分のGmailアドレス>" --role="roles/mcp.toolUser"
gcloud projects add-iam-policy-binding <プロジェクトID> --member="user:<自分のGmailアドレス>" --role="roles/datastore.user"
```

**何をしているか** :

- Google Cloud では「誰が何をしてよいか」を **IAM** で管理します。自分が作ったプロジェクトでは自分が **オーナー** になっていて、ほとんどのことができます
- ただし **`roles/mcp.toolUser`** はオーナーでも別に必要です。これは「Google がホストする MCP サーバーのツールを呼んでよい」という権限で、午後に Antigravity のエージェントが Firestore MCP を使うために要ります。<span style="color:salmon">これがないと `Forbidden` エラーになります</span>（事前検証で実際に確認しました）
- `roles/datastore.user` は Firestore のデータを読み書きする権限です

---

## 5. ADC にプロジェクトを教える（当日）

```
gcloud auth application-default set-quota-project <プロジェクトID>
```

**何をしているか** : ステップ 1 でログインしたとき、まだプロジェクトがなかったので、ADC はどのプロジェクトの利用枠（quota）を使うか知りません。ここで教えておきます。

---

## 6. 確認（当日）

以下を実行して、すべて期待通りなら午後の準備完了です。

| コマンド | 期待する結果 |
| --- | --- |
| `gcloud config get-value project` | 自分のプロジェクト ID |
| `gcloud auth application-default print-access-token` | `ya29.` で始まる長い文字列 |
| `gcloud firestore databases list` | `(default)` と `asia-northeast1` を含む行 |
| 下の IAM 確認コマンド | `roles/mcp.toolUser` と `roles/datastore.user` を含む |

```
gcloud projects get-iam-policy $(gcloud config get-value project) --flatten="bindings[].members" --filter="bindings.members:$(gcloud config get-value account)" --format="value(bindings.role)"
```

うまくいかない人は昼休憩中にスタッフが対応します。声をかけてください。

---

## 費用について

今回作るものは Cloud Run・Firestore・Cloud Build・Artifact Registry を使いますが、いずれも Always Free の枠内に収まる規模です。ただし放置すると微額の課金が発生しうるので、**ワークショップの最後にプロジェクトごと削除します**。

心配な方は、予算アラートを設定しておくと安心です。これも「運用」の一部です。

1. https://console.cloud.google.com/billing/budgets を開く
2. **[予算を作成]** → 名前は任意、金額は `1` USD（または 100 円）
3. しきい値の通知をオンにして保存

## 困ったときは

| 症状 | 対処 |
| --- | --- |
| `gcloud billing projects link` で権限エラー | 課金アカウントの管理者が自分になっているか、https://console.cloud.google.com/billing で確認 |
| `firestore databases create` で「API が有効ではない」 | `gcloud services enable firestore.googleapis.com` をもう一度実行して 1 分待つ |
| `print-access-token` で「credentials not found」 | `gcloud auth application-default login` をやり直す |
