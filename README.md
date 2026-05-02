ZennGitHubActionsYAML
Zenn の記事を GitHub にプッシュしたタイミングで、Slack と Discord に自動通知する GitHub Actions ワークフローです。
articles/*.md の変更を検知し、published: true かつ Zenn 側で実際に公開されている記事だけを通知します。
特徴

main ブランチへのプッシュ時のみ発火（articles/*.md のみ監視）
frontmatter の published: true を判定して、下書きは通知しない
Zenn 側のページが HTTP 200 を返すまで最大 60 秒リトライ（記事のデプロイ反映待ち）
Slack（HTTP 200）と Discord（HTTP 204）の成功ステータスを個別に検証
jq で JSON を組み立てるため、タイトルに " や \ が含まれていても安全
1 件でも通知に失敗するとジョブ全体を失敗扱いにする（CI でも気付ける）

動作の流れ
push to main
  └─ articles/*.md が変更されているか？
       └─ 変更ファイル毎に
            ├─ published: true か？        → No なら skip
            ├─ frontmatter から title 抽出 → 失敗なら error
            ├─ Zenn の URL に GET（最大6回リトライ）
            │    └─ 200 が返るまで待つ
            ├─ Slack に POST（200 を期待）
            └─ Discord に POST（204 を期待）
使い方
1. ワークフローの配置
notify-zenn.yml をご自身の Zenn 連携リポジトリの .github/workflows/ 配下にコピーします。
.github/
  └─ workflows/
       └─ notify-zenn.yml
articles/
  ├─ my-first-article.md
  └─ ...
2. Webhook URL の取得
Discord
通知したいチャンネルの「設定」→「連携サービス」→「ウェブフックを作成」から URL を発行します。
Slack
Slack API: Incoming Webhooks でアプリを作成し、通知先チャンネルを指定して URL を発行します。
3. GitHub Secrets の登録
リポジトリの Settings → Secrets and variables → Actions から以下の 3 つを登録します。
Name用途SLACK_WEBHOOK_URLSlack の Incoming Webhook URLDISCORD_WEBHOOK_URLDiscord のウェブフック URLZENN_USER_IDZenn のユーザー ID（zenn.dev/<ID> の部分）

Slack か Discord のどちらか一方だけ使う場合は、片方の Secret を未設定にしておけば該当の送信処理はスキップされます。

4. 記事を書いてプッシュ
articles/<slug>.md を作成し、frontmatter で published: true にしてから main にプッシュすれば通知が飛びます。
yaml---
title: "記事のタイトル"
emoji: "✨"
type: "idea"
topics: ["zenn"]
published: true
---
通知メッセージのサンプル
Slack / Discord ともに以下のような内容が投稿されます。
🎉 新しいZenn記事が公開・更新されました！
記事のタイトル
https://zenn.dev/<ZENN_USER_ID>/articles/<slug>
カスタマイズ
監視対象ディレクトリの変更
本（books）も含めたい場合は、on.push.paths と tj-actions/changed-files の files を両方変更します。
yamlon:
  push:
    branches:
      - main
    paths:
      - 'articles/*.md'
      - 'books/**/*.md'
メッセージ文面の変更
SLACK_PAYLOAD と DISCORD_PAYLOAD を組み立てている jq -n の引数を編集します。Slack は text、Discord は content フィールドを使う点に注意してください。
Zenn 反映待ちの調整
デフォルトは 10 秒間隔 × 6 回（最長 60 秒）です。Zenn のデプロイが遅れがちな場合は、リトライ回数や sleep の秒数を増やしてください。
トラブルシューティング
症状原因と対処Skipped <file> (published: false or not set) でスキップされるfrontmatter が published: true になっているか確認Article not published on Zenn after retries (HTTP 404)Zenn 側のデプロイが失敗している可能性あり。frontmatter の必須項目（emoji / type / topics 等）の漏れを確認Slack webhook failed with HTTP 4xxSLACK_WEBHOOK_URL の Secret 値が無効。再発行して Secret を更新Discord webhook failed with HTTP 4xxDISCORD_WEBHOOK_URL の Secret 値が無効。同上Failed to extract titlefrontmatter の title: 行が欠落している、または書式が崩れている
必要なツール
ワークフローは ubuntu-latest ランナーで動作します。利用しているコマンドは以下のとおりで、いずれも標準で利用可能です。

bash / awk / grep / curl / jq
actions/checkout@v4
tj-actions/changed-files@v42

ライセンス
MIT License
関連記事
このワークフローの背景や失敗談は Zenn の記事にまとめています。あわせてどうぞ。
