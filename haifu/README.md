# 配布物

| ファイル | 用途 | 参加者の操作 |
|---|---|---|
| [workshop.md](workshop.md) | ワークスペースルール（`.agents/rules/workshop.md` に置く） | 下のコマンドで配置 |

## ルールの配置方法

参加者が **Antigravity IDE で自分の手でファイルを作る**（curl やコマンド貼り付けではなく）。手順は [hands-on.md のステップ 0](../hands-on.md#ステップ-0--プロジェクトを作ってルールを置く目安--10-分)。IDE のルール編集画面で Activation Mode を Always On にし、Content にこの `workshop.md` の内容を貼り付ける。

このディレクトリの `workshop.md` が内容の原本。

## ルールの各項目の根拠（事前検証より）

| 項目 | 根拠 |
|---|---|
| ブラウザ検証をしない | エージェントの自動ブラウザ検証は画像解析でクォータを大きく消費する。動作確認は人間が目でやる |
| テストコードを書かない | 検証でエージェントが頼んでいないテストを 2 ファイル書き、実 Firestore に対する結合テストまで実行した（クォータと時間の消費） |
| デプロイ後の動作確認をしない | 3 回目のデプロイで「Running 7 commands」の余計な承認が発生。4 回目でこの 1 行を足したらゼロになった |
| エラー時に認証情報・履歴を読まない | Forbidden の後、エージェントが `print-access-token` の外部送信と `~/.zsh_history` の読み取りを試みた |
| Firestore のみ / サーバーサイド SDK のみ | Cloud SQL は課金、Web SDK はセキュリティルール全開放になるため。検証で遵守を確認 |
| 日付は文字列 | Cloud Run（UTC）とローカル（JST）の 1 日ズレ防止。検証で遵守を確認 |
| 構造化ログのキー指定 | 検証で `severity` キー付きの JSON が Cloud Logging で jsonPayload としてパースされ、severity も反映された |
| 初期データを投入しない | 検証でエージェントが seed.py を自分で実行し、ステップ 2（Firestore MCP の見せ場）が先回りされた |
