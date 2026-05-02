# Zenn GitHub Actions YAML

ZennへのCI/CDを自動化するためのGitHub Actions YAML設定集です。

---

## 目次

- [概要](#概要)
- [機能](#機能)
- [クイックスタート](#クイックスタート)
- [ワークフロー一覧](#ワークフロー一覧)
- [設定例](#設定例)
- [コントリビューション](#コントリビューション)
- [ライセンス](#ライセンス)

---

## 概要

このリポジトリには、開発ワークフローを自動化するための各種YAML設定ファイルが収録されています。ビルド・テスト・デプロイ・通知などをGitHub Actionsで一元管理できます。

---

## 機能

| 機能 | 説明 |
|------|------|
| **継続的インテグレーション (CI)** | プッシュ・PR時にコードを自動ビルド＆テスト |
| **継続的デリバリー (CD)** | テスト通過後、本番環境へ自動デプロイ |
| **通知** | ビルド・デプロイの結果をリアルタイム通知 |

---

## クイックスタート

### 1. リポジトリをクローン

```bash
git clone https://github.com/YosikaneHibiki/ZennGitHubActionsYAML.git
cd ZennGitHubActionsYAML
```

### 2. 依存関係をインストール

```bash
npm install
```

### 3. ワークフローを有効化

`.github/workflows/` 配下のYAMLファイルをリポジトリにプッシュするだけで自動的に有効になります。

---

## ワークフロー一覧

| ワークフロー名 | トリガー | 説明 |
|---------------|---------|------|
| CI | `push` / `pull_request` | テストを自動実行 |
| CD | `push` (mainブランチ) | 本番環境へ自動デプロイ |

---

## 設定例

### CI ワークフロー (`ci.yml`)

```yaml
name: CI

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install Dependencies
        run: npm install

      - name: Run Tests
        run: npm test
```

---

## コントリビューション

コントリビューションを歓迎します！  
プルリクエストを送る前に [コントリビューションガイドライン](CONTRIBUTING.md) をご確認ください。

1. このリポジトリをフォーク
2. フィーチャーブランチを作成 (`git checkout -b feature/your-feature`)
3. 変更をコミット (`git commit -m 'Add your feature'`)
4. ブランチをプッシュ (`git push origin feature/your-feature`)
5. プルリクエストを作成

---

## ライセンス

このプロジェクトは MIT ライセンスのもとで公開されています。詳細は [LICENSE](LICENSE) ファイルをご覧ください。
