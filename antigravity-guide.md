# Antigravity 2.0 ガイド

ハンズオンで使う Antigravity 2.0 について、知っておくと便利な情報をまとめています。

<span style="color:salmon"><b>※ このドキュメントの内容は 2026/09/07 時点の Antigravity 2.0 で確認したものです。Antigravity は活発にアップデートされているため、最新の情報は公式ドキュメント ( https://antigravity.google/docs ) を確認してください。</b></span>

## Antigravity 2.0 と Antigravity IDE

2026 年 6 月のリリースで、Antigravity は 2 つのアプリに分かれました。

| | Antigravity 2.0 | Antigravity IDE |
| --- | --- | --- |
| 正体 | エージェントを動かすためのデスクトップアプリ | 従来の VS Code ベースのエディタ |
| 画面 | 会話 + エージェントの作業パネル + ターミナル。ファイルツリーやエディタはない | ファイルツリー、エディタ、ターミナル、エージェントパネル |
| 向いている作業 | 「作って」「デプロイして」と任せる作業。複数のプロジェクトやタスクの並行 | コードを見ながら行単位で変更を確認したい作業 |
| ダウンロード | https://antigravity.google/download | 同じページ。2.0 の右上の **Install IDE** からも |

ハンズオンでは **2.0** を使います。2.0 の右上にある **Open IDE** で同じフォルダを IDE で開けますが、**ログインは 2.0 と IDE で別**なので、IDE も使う人は両方でログインが必要です。

公式サイト : https://antigravity.google/

公式ドキュメント : https://antigravity.google/docs


## レートリミットとクォータ

Antigravity にはプランに応じた利用量の上限（クォータ）があります。無料プランのクォータは **週ごと** にリセットされます。

レートリミットはエージェントが行った作業量に比例します。計画＋複数ファイル編集のような大きなタスクは消費が大きく、**MCP ツールの呼び出しもエージェントの作業として消費されます**。

詳しくはこちら : https://antigravity.google/docs/plans

> **Note** : 2.0 でプランやクォータの体系が変わっていないか、当日前に上記ページで確認してください。

### クォータを節約するコツ

- モデルは **Gemini 3.8 Flash (High)** で十分なことが多い。Pro は計画や難しい修正のときだけ
- 1 つの会話で多くのことをやりすぎず、タスクを分けて会話を開始する
- プロンプトは具体的に書く（やり直しが減る）
- 失敗したら同じ会話で続けず、新しい会話でやり直す（エージェントの原因調査に消費させない）
- 「ブラウザで確認して」と頼まない（画像解析にクォータを使う）


## 画面構成

![Antigravity 2.0 の画面](images/new-conversation.png)

**左パネル**

- **+ New Conversation** : 新しい会話を開始
- **Conversation History** : 過去の会話
- **Scheduled Tasks** : 定期実行するタスク（今日は使いません）
- **Projects** : 開いているフォルダと、その中の会話の一覧。**+** で新しいフォルダを開く
- **Settings** : 設定

**中央** : 会話。エージェントが何をしているか（Explored / Edited / Ran / MCP Tool …）が順に表示されます

**右パネル**（右上のアイコンで開閉）: ファイル、変更差分、**Terminals**。エージェントが起動したサーバーのログもここに流れます

**右上の Open IDE** : 同じフォルダを Antigravity IDE で開く

### 入力欄

- **モデル選択** : Gemini 3.8 Flash (High) / 3.7 Flash / 3.6 Flash / 3.1 Pro (Low) / Claude Sonnet 4.6 / Claude Opus 4.6 / GPT-OSS 120B から選べます。**View Usage** でクォータの使用状況を確認できます

![モデル選択](images/model-select.png)

- **+** ボタン : Media（画像を添付）/ Mentions（ファイルを参照）/ Actions / Browser
- **Local** : 実行場所（今日はローカルのまま）

> **Note** : 最初の依頼では自動的に **Implementation Plan** が作られ、**Proceed** を押してから実装が始まります。計画だけ見たいときは `/plan` を使います。


## スラッシュコマンド

入力欄で `/` を打つと使えます。Antigravity 2.0 と CLI の両方で利用できます。

| コマンド | 機能 | ハンズオンでの扱い |
| --- | --- | --- |
| `/plan` | コードを調査して、レビュー可能な実装計画を作る（コードは書かない） | 大きめの変更の前に |
| `/grill-me` | コードを書く前に、エージェントがあなたにインタビューして計画をすり合わせる | 自由テーマのときに便利。今日は題材が決まっているので不要 |
| `/btw` | 作業を止めずに、バックグラウンドで軽い質問をする | **おすすめ**。生成されたコードについて聞く |
| `/learn` | この会話でのフィードバックや学びを、永続的なルールやスキルに蒸留する | **おすすめ**。最後にやると今日の体験が資産になる |
| `/goal` | 途中確認なしに、目標を達成するまで自律的に実行する | 今日は使わない（承認とクォータを大量に消費） |
| `/schedule` | タスクを一回限り・または cron で定期実行する | 15:00 のパートで「定期的にログを確認する」等に使えるかも |
| `/browser` | サンドボックス化されたブラウザで Web 調査や UI の検査をする | 今日は使わない（クォータ） |
| `/boost` | 複雑なバグやレース条件のための、複数エージェントによる深い推論 | 今日は使わない |
| `/teamwork-preview` | リポジトリ規模の移行や調査のための協調エージェントチーム | 今日は使わない |

参考 : https://antigravity.google/docs/slash-commands/


## サブエージェント

複雑な問題を並列に処理するために、エージェントが動的にサブエージェントを起動します。あらかじめ用意されているのは **research**（調査）、**browser**（ブラウザ操作）、**self**（自分自身のコピー）の 3 種類で、カスタムエージェントを定義することもできます。

**Browser サブエージェント** は Chrome を開いて操作できます（Chrome と、デバッグセッション開始の許可が必要）。動作確認を自動でやってくれて便利ですが、画像解析でクォータを消費するので、ハンズオンではルールで止めています。


## アーティファクト

エージェントが作業の節目に生成する成果物です。

| アーティファクト | 内容 | ユーザーの関わり |
| --- | --- | --- |
| **Task List** | エージェントが進捗を管理するための作業項目 | 通常は見るだけ |
| **Implementation Plan** | 変更の設計と技術詳細。最初の依頼のあとに生成される | **Proceed** で承認してから実装に入る（設定で Always Proceed にもできる） |
| **Code Diff** | 作業中に作られるコードの差分 | 受け入れ / 拒否できる。**Review** ボタンで確認 |
| **Walkthrough** | 実装完了後のレポート。変更の要約、起動方法、確認手順 | 読んで動作確認する |
| **Screenshot & Recording** | ブラウザサブエージェントが撮った画面や録画 | Walkthrough に添付される（今日は使わない） |

![Implementation Plan](images/implementation-plan.png)


## 承認ダイアログとサンドボックス

Antigravity 2.0 は、エージェントが実行するコマンドを **サンドボックス**（ネットワークやファイルへのアクセスが制限された隔離環境）で実行しようとします。サンドボックスの外での実行が必要なときや、MCP ツールを呼ぶとき、プロジェクトの外のファイルを読むときに **承認ダイアログ** が表示されます。

![コマンド実行の承認](images/allow-command.png)

| 選択肢 | 意味 |
| --- | --- |
| 1 Yes, allow this time | 今回だけ許可 |
| 2 Yes, and always allow in this conversation | この会話では同じ操作を常に許可 |
| 3 Yes, and always allow in this project | このプロジェクトでは常に許可 |
| 4 Yes, and always allow | どこでも常に許可 |
| 5 No (tell the agent what to do instead) | 拒否して、代わりの指示を書く |

**ダイアログは読んでから押しましょう。** エージェントは失敗すると原因を探し始め、gcloud の認証情報、アクセストークン、シェルの履歴などを読もうとすることがあります。おかしいと思ったら **5 No** で止めて、新しい会話でやり直すのが安全です。

> **Note** : サンドボックスの中では `~/.config/gcloud` に書き込めないため、エージェントが `gcloud` コマンドを実行すると「Could not setup log file」「Unable to create private file」という警告が出ます。これは正常で、そのあとサンドボックス外での実行の承認を求めてきます。


## ルール

エージェントの動作をガイドする指針です。

- **グローバルルール** : すべてのプロジェクトに適用。`~/.gemini/GEMINI.md` に保存されます。Settings → Customizations の **Rules** に表示されます
- **ワークスペースルール** : 特定のプロジェクトにのみ適用。`<プロジェクト>/.agents/rules/*.md` に置きます

ワークスペースルールのファイルは、先頭に適用条件を書きます。

```markdown
---
trigger: always_on
---

- テストコードは書かないでください。
- 日付はすべて "YYYY-MM-DD" 形式の文字列として扱ってください。
```

> **Note** : 2026/09 時点では、2.0 の Customizations 画面からワークスペースルールを **作成できません**（表示もされません）。ターミナルでファイルを置くか、IDE で作成してください。置いたルールはちゃんと効きます。

ハンズオンで使うルールと各項目の理由は [hands-on.md のステップ 0](hands-on.md#ステップ-0--プロジェクトを作ってルールを置く目安--10-分) にあります。


## ワークフローとスキル

ルールと似た「カスタマイズ」が 2 つあります。違いは **誰が発動するか** です。

| | 正体 | 発動 | 置き場所 |
| --- | --- | --- | --- |
| **Rules** | システム指示（常に有効） | 常時 | `~/.gemini/GEMINI.md` / `.agents/rules/` |
| **Workflows** | 保存したプロンプト | **ユーザー**が `/` で呼ぶ | `~/.gemini/config/global_workflows/` / `.agents/workflows/` |
| **Skills** | プロンプト + スクリプト + 例のパッケージ | **エージェント**が必要と判断したとき | `~/.gemini/config/skills/` / `.agents/skills/` |

（Mete Atamel さんの [Getting started with Google Antigravity](https://speakerdeck.com/meteatamel/getting-started-with-google-antigravity) より : Rules ≅ system instructions, Workflows ≅ saved prompts, Skills ≅ agent-triggered）

### スキルの構造

```
my-skill/
├── SKILL.md      # 指示（必須）。先頭の frontmatter に name と description
├── scripts/      # 補助スクリプト（任意）
├── examples/     # 参考実装（任意）
└── resources/    # テンプレートなど（任意）
```

会話が始まると、エージェントは利用可能なスキルの **名前と説明の一覧** だけを見て、関係がありそうなら本文を読みに行きます（progressive disclosure）。なので `description` に「いつ使うか」を書くのが大事です。

`/learn` コマンドを使うと、会話の中での学びからルールやスキルを自動生成してくれます。


## プラグイン

**プラグイン** は、スキル・ルール・MCP・フックをひとまとめにした名前空間付きのパッケージです。Customizations の **Plugins** → **Build With Google Plugins** → **Customize** から、Google が用意したプラグインを有効化できます。

手元で確認できたものの例 :

| プラグイン | 中身 |
| --- | --- |
| **firebase** | Firestore / Auth / Hosting / Security Rules など Firebase 各サービスのスキル 11 個 |
| **chrome-devtools-plugin** | Chrome DevTools でのデバッグ・パフォーマンス分析・アクセシビリティのスキル |
| **modern-web-guidance-plugin** | モダン Web 開発のガイダンス（GoogleChrome/modern-web-guidance） |
| **android-cli-plugin** | Android 開発の基本ツールと知識 |
| **google-antigravity-sdk** | Antigravity Python SDK でエージェントを作るためのスキル |
| **science** | AlphaFold, PubMed, UniProt など科学系データベースのスキル 40 個以上（google-deepmind/science-skills） |

有効化したプラグインのスキルは、エージェントが自分の判断で使います。たとえば **firebase** プラグインを有効にしていると、Firestore を使うアプリを作るときに `firebase_firestore` スキルが読み込まれ、Firebase CLI（`npx firebase-tools`）でデータベースを確認しに行くことがあります。

> **Note** : 検証機では、2.0 への移行の直後に android-cli / chrome-devtools / firebase / google-antigravity-sdk / modern-web-guidance の 5 つが自動で入り、有効になっていました。**2.0 を新規インストールすると最初から入っている可能性が高い**ですが、未確認です（当日前に新規環境で確認します）。有効になっていると、たとえば Firestore を使うときに firebase プラグインのスキルが読み込まれてエージェントの動きが少し変わります。Customizations → Plugins → Customize で、何が有効かを一度見ておいてください。

自作プラグインは `~/.gemini/config/plugins/<名前>/`（グローバル）または `.agents/plugins/<名前>/`（ワークスペース）に `plugin.json` と一緒に置くと自動で読み込まれます。


## 設定ファイルの置き場所まとめ

| 種類 | グローバル（`~/.gemini/`） | ワークスペース（`<プロジェクト>/.agents/`） |
| --- | --- | --- |
| ルール | `GEMINI.md` | `rules/*.md` |
| ワークフロー | `config/global_workflows/` | `workflows/` |
| スキル | `config/skills/` | `skills/` |
| MCP | `config/mcp_config.json` | `mcp_config.json` |
| フック | `config/hooks.json` | `hooks.json` |
| カスタムエージェント | `config/agents/` | `agents/` |
| プラグイン | `config/plugins/` | `plugins/` |

出典 : https://atamel.dev/posts/2026/08-21_where_agy_configuration_summary/


## MCP（Model Context Protocol）

エージェントが外部のサービス（データベース、クラウド、GitHub など）を操作するための共通の仕組みです。MCP を入れると、エージェントに新しい「ツール」が増えます。

### インストール

Settings → **Customizations** → **Installed MCP Servers** の **Add MCP +** で MCP ストアが開きます。検索して **+ Add** を押すだけです。

![MCP ストア](images/mcp-store.png)

インストール後は Customizations の一覧に表示され、トグルで有効/無効を切り替えられます。ツールの一覧も確認できます。

![インストール済み MCP](images/mcp-installed.png)

### ローカル MCP とリモート MCP

| | ローカル MCP | リモート MCP |
| --- | --- | --- |
| 動く場所 | 自分の PC 上（Node.js などで起動） | サービス提供者のサーバー上 |
| 例 | Cloud Run MCP（`npx @google-cloud/cloud-run-mcp`） | Firestore MCP（`https://firestore.googleapis.com/mcp`） |
| 認証 | gcloud の ADC | gcloud の ADC（`authProviderType: google_credentials`）+ IAM |
| 前提 | Node.js が必要 | Google Cloud 側で **`roles/mcp.toolUser`** が必要 |

設定は `~/.gemini/config/mcp_config.json` に保存されます（Customizations の **Open MCP Config** で開けます）。

```json
{
  "mcpServers": {
    "google-cloud-firestore": {
      "serverUrl": "https://firestore.googleapis.com/mcp",
      "authProviderType": "google_credentials"
    },
    "cloudrun": {
      "command": "npx",
      "args": ["-y", "@google-cloud/cloud-run-mcp"]
    }
  }
}
```

### ハンズオンで使う MCP のツール

**Cloud Run MCP** : `deploy_local_folder`, `deploy_file_contents`, `deploy_container_image`, `list_services`, `get_service`, `get_service_log`, `list_projects`, `create_project`

**Firestore MCP** : `list_databases`, `list_collections`, `list_documents`, `add_document`, `get_document`, `update_document`, `delete_document` など

エージェントは、頼まなくても状況確認のために MCP を呼ぶことがあります（例 : アプリ生成中に `list_collections` で DB の状態を見る）。

### 知っておくとよい制約

- Cloud Run MCP の `deploy_local_folder` は `.gcloudignore` / `.dockerignore` を **無視してフォルダ全体を zip** します。`node_modules` 込みで 10MB 前後（Python の `.venv` だと 40MB 前後）のアップロードになります
- Cloud Run MCP の `get_service_log` は構造化ログ（JSON）の中身を `[object Object]` と表示してしまいます（2026/09 時点、v1.10.0）。エラーの詳細を読むには Google Cloud コンソールのログ画面の方が確実です
- Cloud Run MCP は Node.js **22.23.0 / 24.17.0 では動きません**（Node 本体の不具合 [nodejs/node#63989](https://github.com/nodejs/node/issues/63989)）。22.23.1 以降または 24.18 以降を使ってください


## Customization budget

Customizations の上部に **Token Usage** が表示されます。ルール・スキル・MCP のツール定義は毎回エージェントに渡されるため、合計に上限があり、超えると大きなものから自動で切り詰められます。

今日の構成（ルール 1 つ + MCP 2 つ）はごくわずか（数百トークン）しか使いません。過去に多くのスキルを入れている人は、ここで確認しておくと安心です。


## 権限（エージェントに何を許すか）

Antigravity 2.0 では、エージェントの操作は「**アクション(対象)**」という単位で許可・拒否が管理されます。承認ダイアログで **「Yes, and always allow in this project」** を選ぶと、その操作がプロジェクトの **許可リスト（allow list）** に保存され、次からはダイアログなしで実行されます。

保存先は `~/.gemini/config/projects/<プロジェクトID>.json` の `permissionGrants` で、中身はこんな形です（検証機の実物）。

```
mcp(google-cloud-firestore/add_document)        ← MCP ツールの呼び出し
mcp(cloudrun/deploy_local_folder)
unsandboxed(.venv/bin/python3 seed.py)           ← サンドボックス外でのコマンド実行
unsandboxed(gcloud auth)
read_file(/Users/.../mcp_config.json)           ← プロジェクト外のファイル読み取り
```

<span style="color:salmon"><b>「always allow」は取り消さない限り残ります。</b></span> 検証中、エージェントの調査モードのときに「always allow in this project」を選んでしまったコマンド（`gcloud auth`、トークンを外部に送る `curl` など）が、そのまま許可リストに残っていました。承認ダイアログで 2〜4 を選ぶのは、**中身を読んで「これは毎回許可してよい」と判断できるものだけ**にしてください。

> **Note** : Settings には、承認ダイアログを出さずに実行する範囲を広げる設定があります（許可リスト / 拒否リスト、Terminal Sandboxing）。ハンズオンでは **「全部許可」に相当する設定にはしない**でください。エージェントが失敗したときの「調査モード」を止められなくなります。


## 参考資料・コンテンツ

- Antigravity 公式サイト : https://antigravity.google/
- Antigravity 公式ドキュメント : https://antigravity.google/docs
- Antigravity 2.0 の紹介 : https://antigravity.google/blog/introducing-google-antigravity-2
- Getting started with Google Antigravity（Mete Atamel さんのスライド）: https://speakerdeck.com/meteatamel/getting-started-with-google-antigravity
- Antigravity の codelab 一覧 : https://codelabs.developers.google.com/?product=antigravity
- MCP のドキュメント : https://antigravity.google/docs/mcp/
- Cloud Run MCP（GitHub） : https://github.com/GoogleCloudPlatform/cloud-run-mcp
- Firestore リモート MCP（公式ドキュメント） : https://docs.cloud.google.com/firestore/native/docs/use-firestore-mcp
