# Zenn GitHub Actions YAML

Zennに記事を公開・更新したとき、SlackおよびDiscordへ自動通知するGitHub Actionsワークフローです。

---

## 目次

- [概要](#概要)
- [動作条件](#動作条件)
- [Secrets の設定](#secrets-の設定)
- [ワークフロー一覧](#ワークフロー一覧)
- [クイックスタート](#クイックスタート)
- [ライセンス](#ライセンス)

---

## 概要

`articles/` 配下の Markdown ファイルが `main` ブランチにプッシュされると、自動的にSlack・Discordへ通知を送ります。

```
mainブランチへpush（articles/*.md）
  └─ published: true の記事だけを対象に通知
       ├─ Zenn同期完了を確認（最大60秒リトライ）
       ├─ Slack へ通知
       └─ Discord へ通知
```

---

## 動作条件

通知が送信されるのは以下をすべて満たした場合です。

1. `main` ブランチへ `push` した
2. `articles/` 配下の `.md` ファイルが変更された
3. 記事のフロントマターに `published: true` が設定されている

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
|----------|:----:|------|
| `ZENN_USER_ID` | 必須 | ZennのユーザーID（例: `your_zenn_id`） |
| `SLACK_WEBHOOK_URL` | 任意 | Slack Incoming Webhook URL |
| `DISCORD_WEBHOOK_URL` | 任意 | Discord Webhook URL |

> SlackとDiscordはどちらか一方だけでも動作します。

### Slack Webhook URLの取得方法

1. [Slack API](https://api.slack.com/apps) でAppを作成
2. **Incoming Webhooks** を有効化
3. 通知先チャンネルを選択してWebhook URLをコピー

### Discord Webhook URLの取得方法

1. 通知先チャンネルの **設定 → 連携サービス → ウェブフック**
2. **新しいウェブフック** を作成してURLをコピー

---

## ワークフロー一覧

| ファイル名 | 説明 | 詳細 |
|-----------|------|------|
| [`notify-zenn.yml`](.github/workflows/notify-zenn.yml) | Zenn記事公開時にSlack・Discord通知 | [解説ドキュメント](notify-zenn.md) |

---

## クイックスタート

```bash
# 1. リポジトリをクローン
git clone https://github.com/YosikaneHibiki/ZennGitHubActionsYAML.git
cd ZennGitHubActionsYAML

# 2. Secretsを設定（GitHub上で行う）
#    ZENN_USER_ID / SLACK_WEBHOOK_URL / DISCORD_WEBHOOK_URL

# 3. articles/ に記事を追加して push するだけで通知が届く
```

---

## ライセンス

このプロジェクトは MIT ライセンスのもとで公開されています。詳細は [LICENSE](LICENSE) ファイルをご覧ください。
