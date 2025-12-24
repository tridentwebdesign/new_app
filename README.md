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

## Render.com と Neon でのデプロイ

Render.com（ホスティング）と Neon（PostgreSQL）を組み合わせることで、完全無料で Rails アプリをデプロイできます。

### 前提条件

1. GitHub アカウント
   - このプロジェクトを GitHub にプッシュしておいてください

### デプロイ手順

#### ステップ 1: Neon で PostgreSQL データベースを作成

1. [Neon](https://neon.tech/)にアクセス
2. 「Sign up」ボタンをクリック
3. GitHub アカウントで認証（「Continue with GitHub」をクリック）
4. プロジェクト作成ウィザードが開く

   - **Project name**: `allray-db`（任意の名前）
   - **Database name**: デフォルトのまま（`neondb`）
   - **Cloud service provider**: `AWS` を選択
   - **Region**: 以下から選択：
     - 日本にいる場合：`Asia Pacific (Singapore)` または `ap-northeast-1`（最速）
     - それ以外の場合：最も近い地域のリージョンを選択
   - 「Create project」をクリック

5. ダッシュボードで「Connection string」を確認
   - 右上の「Connection string」タブを選択
   - 「Pooled connection」を選択
   - 接続文字列が表示されます（例：`psql 'postgresql://user:password@host/database?sslmode=require'`）
   - **重要**: `psql '` と末尾の `'` を除いた、`postgresql://` で始まる URL 部分のみをコピーしてください
   - 正しい形式：`postgresql://user:password@host/database?sslmode=require`

#### ステップ 2: master.key を準備

1. ローカルで master.key を生成（まだない場合）：

```bash
docker compose exec web bin/rails credentials:edit
```

2. 生成された master.key を確認：

```bash
cat config/master.key
```

出力された 32 文字の文字列（例：`d07ad57a73bbe74564b481ecec01cc05`）をメモしておいてください。

#### ステップ 3: GitHub にコードをプッシュ

```bash
# リポジトリを作成（まだない場合）
git remote add origin https://github.com/your-username/new_app.git
git branch -M main
git push -u origin main
```

#### ステップ 4: Render.com でアプリをデプロイ

1. [Render.com](https://render.com/)にアクセス
2. 「Get Started」ボタンをクリック
3. GitHub アカウントで認証（「Continue with GitHub」をクリック）
4. リポジトリへのアクセスを許可
5. ダッシュボードで「New」ボタンをクリック
6. 「Web Service」を選択

7. リポジトリを選択

   - 自分のアプリリポジトリ（`new_app`）を選択

8. Web Service を設定

   - **Name**: `allray`（任意）
   - **Environment**: `Ruby`
   - **Build Command**: `bundle install; bundle exec rails assets:precompile; bundle exec rails db:migrate`
   - **Start Command**: `bundle exec puma -C config/puma.rb`
   - **Plan**: `Free`

9. 「Advanced」セクションを展開して環境変数を設定

   - **Environment Variables** セクションで **[+ Add Environment Variable]** をクリック
   - 以下の環境変数を 1 つずつ追加：

   | Key                   | Value                                                                     |
   | --------------------- | ------------------------------------------------------------------------- |
   | `DATABASE_URL`        | ステップ 1 でコピーした Neon の接続文字列（`postgresql://` で始まる URL） |
   | `RAILS_ENV`           | `production`                                                              |
   | `RAILS_LOG_TO_STDOUT` | `true`                                                                    |
   | `RAILS_MASTER_KEY`    | ステップ 2 で確認した master.key の内容（32 文字の文字列）                |

   **注意**: `DATABASE_URL` には `psql '...'` 形式ではなく、純粋な URL のみを入力してください

10. 「Create Web Service」をクリック
11. デプロイが開始されます（数分かかる場合があります）
12. デプロイ完了後、ダッシュボードの URL（`https://allray.onrender.com` など）をクリックしてアプリを確認

### 環境変数の設定詳細

Render.com のダッシュボードで「Environment」を選択し、以下を設定：

| キー                  | 値                  | 説明                        |
| --------------------- | ------------------- | --------------------------- |
| `DATABASE_URL`        | Neon の接続文字列   | PostgreSQL データベース接続 |
| `RAILS_ENV`           | `production`        | 本番環境モード              |
| `RAILS_LOG_TO_STDOUT` | `true`              | ログを標準出力に出す        |
| `RAILS_MASTER_KEY`    | `config/master.key` | Rails 暗号化キー            |

### デプロイ後の更新

コードを更新して GitHub に push すると、自動的に Render.com がデプロイを開始します。

```bash
git add .
git commit -m "Update app"
git push origin main
```

### トラブルシューティング

#### `ActiveSupport::MessageEncryptor::InvalidMessage` エラーが出る場合

このエラーは `RAILS_MASTER_KEY` と `config/credentials.yml.enc` が一致していないことを示します。

1. **ローカルで credentials.yml.enc が存在するか確認**：

```bash
ls -la config/credentials.yml.enc
```

2. **存在しない場合は、credentials を生成**：

```bash
docker compose exec web bin/rails credentials:edit
```

3. **master.key の内容を確認**：

```bash
cat config/master.key
```

4. **GitHub にプッシュ**：

```bash
git add config/credentials.yml.enc
git commit -m "Add credentials"
git push origin main
```

**注意**: `config/master.key` は GitHub にプッシュしないでください（.gitignore に含まれています）。この値は Render.com の環境変数 `RAILS_MASTER_KEY` として設定します。

5. **Render.com の環境変数を確認**：
   - ダッシュボードの「Environment」タブを開く
   - `RAILS_MASTER_KEY` の値が、ステップ 3 で確認した master.key の内容と一致しているか確認
   - 一致していない場合は、正しい値に更新して「Save Changes」をクリック

#### デプロイに失敗した場合

1. Render.com ダッシュボードの「Logs」タブでエラーを確認
2. よくあるエラー：
   - `DATABASE_URL が設定されていない` → Environment で設定を確認
   - `bad URI (is not URI?)` → DATABASE_URL に `psql '...'` 形式で入力されている。`postgresql://` で始まる URL のみを入力
   - `ActiveSupport::MessageEncryptor::InvalidMessage` → 上記の手順を実行

#### アプリにアクセスできない場合

- デプロイ完了を待つ（数分かかる場合があります）
- Logs タブでエラーメッセージを確認

### 重要な注意事項

1. **無料プランの制限**:

   - Render.com: 無料プランはメモリ 512MB、月 750 時間まで
   - Neon: 無料プランは月 3GB の storage まで

2. **自動スリープ機能**:

   - Render.com の無料プランはアイドル 15 分後にスリープ状態になります
   - 次のリクエストで自動的に起動します

3. **ファイルストレージ**: Render.com のファイルシステムは一時的です。画像などのファイルアップロードには、AWS S3 などの外部ストレージを使用してください

4. **機密情報**: API キーやパスワードはコードに直接書かず、Environment の環境変数に設定してください
