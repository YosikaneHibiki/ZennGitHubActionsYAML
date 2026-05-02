# notify-zenn.yml — Zenn記事公開通知ワークフロー

Zennに記事が公開・更新されたとき、SlackおよびDiscordへ自動で通知を送るGitHub Actionsワークフローです。

---

## 目次

- [概要](#概要)
- [動作フロー](#動作フロー)
- [前提条件](#前提条件)
- [Secrets の設定](#secrets-の設定)
- [動作条件](#動作条件)
- [処理の詳細](#処理の詳細)
- [通知メッセージ例](#通知メッセージ例)
- [トラブルシューティング](#トラブルシューティング)

---

## 概要

| 項目 | 内容 |
|------|------|
| **ファイル名** | `notify-zenn.yml` |
| **トリガー** | `main` ブランチへの `push`（`articles/*.md` が変更対象） |
| **通知先** | Slack / Discord |
| **使用Action** | `actions/checkout@v4`、`tj-actions/changed-files@v42` |

---

## 動作フロー

```
mainブランチへpush
  └─ articles/*.md に変更あり？
       ├─ なし → ワークフロー終了
       └─ あり → 変更ファイルを1件ずつ処理
                  ├─ ファイルが存在しない → スキップ（削除ケース）
                  ├─ published: true でない → スキップ
                  ├─ Zenn同期待ち（最大60秒、10秒間隔でリトライ）
                  │    └─ HTTP 200 未達 → エラー記録してスキップ
                  ├─ Slack へ通知（SLACK_WEBHOOK_URL が設定されている場合）
                  └─ Discord へ通知（DISCORD_WEBHOOK_URL が設定されている場合）
```

---

## 前提条件

- GitHubリポジトリでZennの記事管理をしていること
- 記事ファイルが `articles/` ディレクトリ配下に `.md` 形式で存在すること
- 記事のフロントマターに `published: true` が設定されていること

```yaml
---
title: "記事タイトル"
published: true   # ← これが必要
---
```

---

## Secrets の設定

GitHubリポジトリの **Settings → Secrets and variables → Actions** で以下を登録してください。

| Secret名 | 必須 | 説明 |
|----------|------|------|
| `SLACK_WEBHOOK_URL` | 任意 | Slack Incoming Webhook URL |
| `DISCORD_WEBHOOK_URL` | 任意 | Discord Webhook URL |
| `ZENN_USER_ID` | 必須 | ZennのユーザーID（例: `your_zenn_id`） |

> Slack・Discordはどちらか一方だけの設定でも動作します。

---

## 動作条件

通知が送信されるのは、以下の条件をすべて満たした場合です。

1. `main` ブランチへの `push` イベントが発生した
2. `articles/` 配下の `.md` ファイルに変更がある
3. 変更されたファイルに `published: true` が含まれている
4. Zennへの同期が完了している（60秒以内にHTTP 200を確認）

---

## 処理の詳細

### Zenn同期待ち

記事URLに対してHTTPリクエストを送り、200が返るまで最大6回（60秒）リトライします。  
タイムアウト後も200にならない場合はエラーとして記録し、次のファイルに進みます。

### Slack通知

```
🎉 新しいZenn記事が公開・更新されました！
https://zenn.dev/{ZENN_USER_ID}/articles/{slug}
```

### Discord通知

```
🎉 新しいZenn記事が公開・更新されました！
https://zenn.dev/{ZENN_USER_ID}/articles/{slug}
```

> SlackはHTTP `200`、DiscordはHTTP `204` を成功レスポンスとして判定します。

---

## 通知メッセージ例

```
🎉 新しいZenn記事が公開・更新されました！
https://zenn.dev/your_zenn_id/articles/my-new-article
```

---

## トラブルシューティング

| 症状 | 原因 | 対処 |
|------|------|------|
| 通知が届かない | Secretsが未設定または誤り | `SLACK_WEBHOOK_URL` / `DISCORD_WEBHOOK_URL` を再確認 |
| `ZENN_USER_ID` エラー | Secretが未設定 | `ZENN_USER_ID` をSecretsに登録 |
| Zenn同期タイムアウト | Zenn側の反映遅延 | 60秒待っても反映されない場合はZennの状態を確認 |
| 記事がスキップされる | `published: true` でない | フロントマターを確認 |
| ワークフローが起動しない | パスが `articles/*.md` 以外 | ファイルの配置場所を確認 |
