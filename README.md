# README

Rails 8 アプリケーション

## 必要な環境

- Docker
- Docker Compose

## セットアップと起動手順

### 1. Docker のインストール

まだ Docker をインストールしていない場合は、[Docker Desktop](https://www.docker.com/products/docker-desktop/)をダウンロードしてインストールしてください。

### 2. アプリケーションの起動

```bash
# リポジトリをクローンした後、プロジェクトディレクトリに移動
cd new_app

# Dockerコンテナを起動（初回は依存関係のインストールに時間がかかります）
docker compose up -d
```

### 3. アプリケーションへのアクセス

ブラウザで以下の URL にアクセスしてください：
http://localhost:3000

### その他のコマンド

```bash
# ログを確認
docker compose logs -f

# アプリケーションを停止
docker compose down

# Railsコンソールを起動
docker compose exec web bin/rails console

# データベースのマイグレーション
docker compose exec web bin/rails db:migrate

# テストの実行
docker compose exec web bin/rails test
```

## 技術スタック

- Ruby 3.4.2
- Rails 8.0.4
- SQLite3
- Puma (アプリケーションサーバー)
- Docker & Docker Compose

## プロジェクト構成

```
new_app/
├── app/              # アプリケーションコード
├── bin/              # 実行可能ファイル
├── config/           # 設定ファイル
├── db/               # データベース関連
├── public/           # 静的ファイル
├── storage/          # アクティブストレージファイル
├── test/             # テストファイル
├── docker-compose.yml # Docker Compose設定
└── Gemfile           # Ruby依存関係
```

## 開発について

このアプリケーションは Docker コンテナ内で実行されるため、ローカル環境に Ruby をインストールする必要はありません。すべての依存関係はコンテナ内で管理されます。

## Heroku へのデプロイ

### 前提条件

1. Heroku アカウントの作成
   - [Heroku](https://www.heroku.com/)でアカウントを作成してください

### デプロイ手順

#### 1. Heroku ダッシュボードでアプリケーションを作成

1. [Heroku ダッシュボード](https://dashboard.heroku.com)にログイン
2. 右上の「New」ボタンをクリック
3. 「Create new app」を選択
4. アプリ名を入力（例：`my-allray-app`）
5. 地域を選択（「United States」または「Europe」）
6. 「Create app」をクリック

#### 2. PostgreSQL アドオンを追加

1. 作成したアプリのダッシュボードを開く
2. 「Resources」タブをクリック
3. 「Add-ons」セクションで検索欄に「Postgres」と入力
4. 「Heroku Postgres」を選択
5. プランを「Hobby Dev - Free」に設定
6. 「Submit Order Form」をクリック

#### 3. デプロイ用の設定

アプリケーションをローカルで準備します：

```bash
# Git リポジトリを初期化（まだの場合）
git init
git add .
git commit -m "Initial commit - Prepare for Heroku deployment"
```

#### 4. Heroku リモートを追加してデプロイ

ダッシュボードの「Deploy」タブで以下の情報を確認し、実行してください：

```bash
# Heroku リモートを追加
heroku git:remote -a your-app-name

# メインブランチをデプロイ
git push heroku main
```

※ `your-app-name` は作成したアプリ名に置き換えてください

#### 5. 環境変数を設定

1. ダッシュボードの「Settings」タブをクリック
2. 「Config Vars」セクションで「Reveal Config Vars」をクリック
3. 以下の環境変数を追加：
   - **Key**: `RAILS_MASTER_KEY`
   - **Value**: `config/master.key` ファイルの内容をコピー＆ペースト
4. 「Add」ボタンをクリック

#### 6. データベースのマイグレーション

1. ダッシュボードの「More」ボタン（右上の …）をクリック
2. 「Run console」を選択
3. 以下のコマンドを実行：

```bash
rails db:migrate
```

4. コンソールが自動的に閉じれば完了

#### 7. アプリケーションを確認

ダッシュボードの右上にある「Open app」ボタンをクリックするか、ブラウザで以下にアクセス：

```
https://your-app-name.herokuapp.com
```

### 重要な注意事項

1. **データベース**: Heroku では本番環境で PostgreSQL を使用します（SQLite3 は使用できません）

2. **ファイルストレージ**: Heroku のファイルシステムは一時的です。画像などのファイルアップロードには、Amazon S3 などの外部ストレージサービスを使用してください

3. **環境変数**: 機密情報（API キー、パスワードなど）は必ず環境変数として設定し、コードには直接書かないでください

4. **本番環境設定**: `config/environments/production.rb` の設定が適切か確認してください
