# AGENTS.md

## 基本ルール

- 回答は必ず日本語を使用する。
- Markdownファイル内の文章は、原則として日本語を使用する。
- ただし、コード内の引用、コマンド、設定値、ライブラリ名、フレームワーク名、サービス名、その他の固有名詞については、必要に応じて英語を使用してもよい。

## プロジェクトの前提

このプロジェクトは、Docker上でLaravel、Vue.js、PostgreSQL、pgAdmin、nginxを使用する開発環境である。

LaravelはBladeによる画面描画には使用せず、REST APIを提供するバックエンドとして扱う。
フロントエンドはVue.jsを使用し、Laravel APIと通信する独立したクライアントアプリケーションとして扱う。

## コンテナ内パスのルール

このプロジェクトでは、コンテナ内の作業ディレクトリとして`/project_dir`を使用する。

- Laravelバックエンドは`/project_dir/backend`に配置する。
- Vue.jsフロントエンドは`/project_dir/frontend`に配置する。
- nginxのLaravel公開ディレクトリは`/project_dir/backend/public`を参照する。

## Laravelバックエンドのルール

LaravelはREST APIを提供するバックエンドとして実装する。

- フロントエンド画面の実装にBladeを使用しない。
- Controllerは原則としてJSONレスポンスを返す。
- バリデーションは必要に応じてForm Requestに分離する。
- レスポンス形式が複雑になる場合は、API ResourceやDTOの利用を検討する。
- 画面表示の責務をLaravel側に持たせず、UIはVue.js側で実装する。

## Vue.jsフロントエンドのルール

Vue.jsは画面表示とユーザー操作を担当するフロントエンドとして実装する。

- フロントエンド画面は`frontend/`配下に実装する。
- Laravel APIとの通信を通じてデータを取得・更新する。
- APIのベースURLは環境変数で管理する。
- Laravel側のBladeに画面表示の責務を持たせない。
- API通信処理は、必要に応じて専用のAPIクライアント層やcomposableに分離する。

## データベースのルール

このプロジェクトでは、データベースとしてPostgreSQLを使用する。

- LaravelのDB接続はPostgreSQLを前提とする。
- Docker環境内からDBへ接続する場合は、Composeのサービス名である`db`をホスト名として使用する。
- マイグレーションやクエリを書く際は、PostgreSQLで動作することを前提とする。
- SQLiteやMySQLを前提とした設定・SQL・型定義を追加しない。
- DB管理には必要に応じてpgAdminを使用する。

## DDDの基本方針

このプロジェクトでは、必要に応じてDDDの考え方を取り入れる。

- 業務ルールはControllerに直接書かず、Domain層またはApplication層に配置する。
- ControllerはHTTPリクエストを受け取り、UseCaseを呼び出し、レスポンスを返す役割に留める。
- Domain層には、Entity、Value Object、Domain Service、Repository Interfaceなどを配置する。
- Application層には、UseCase、DTO、アプリケーションサービスなどを配置する。
- Infrastructure層には、Eloquentを使ったRepository実装や外部サービス連携を配置する。
- Laravel固有の機能への依存は、可能な範囲でDomain層に持ち込まない。

## Docker操作のルール

このプロジェクトの開発作業は、原則としてDocker Composeを通じて行う。

- Laravel関連のコマンドは`app`サービス上で実行する。
- Vue.js関連のコマンドは`frontend`サービス上で実行する。
- PostgreSQLへの接続確認や操作は`db`サービスを前提とする。
- Docker Composeの設定を変更した場合は、必要に応じて`docker compose config`で構文を確認する。
- 環境全体の起動確認が必要な場合は、`docker compose up -d --build`を使用する。

## Git管理のルール

生成物や環境依存ファイルはコミットしない。

コミットしないものの例:

- `backend/vendor/`
- `frontend/node_modules/`
- `frontend/dist/`
- `backend/.env`
- `backend/storage/logs/*.log`
- `backend/database/database.sqlite`

依存関係の再現に必要なlockファイルはコミット対象とする。

- `backend/composer.lock`
- `frontend/package-lock.json`

コミットは作業単位ごとに分け、変更内容が分かるメッセージを使用する。

## ブランチ運用のルール

このプロジェクトでは、以下のブランチ運用を基本とする。

- メインブランチは`main`とする。
- `main`から`develop`ブランチを作成し、基本的な開発作業は`develop`を基準に行う。
- 個別作業を行う場合は、`develop`から`feature`ブランチを切り出す。
- `feature`ブランチ名は、`feature/YYMMDD-short-description`の形式とする。
- `short-description`には、作業内容を簡潔な英語で記述する。
- 例: `feature/260629-add-user-api`
- `feature`ブランチは、必ず`develop`へマージする。
- `main`および`develop`への直接マージは原則として行わない。
- ただし、開発初期など、運用上必要な場合はこの限りではない。
- 本プロジェクトでは、現時点で`hotfix`ブランチ運用は定義しない。

## 動作確認・テストのルール

変更後は、内容に応じて必要な確認を行う。

- Docker設定を変更した場合は、`docker compose config`で構文を確認する。
- 環境全体に関わる変更をした場合は、`docker compose up -d --build`で起動確認を行う。
- Laravel側を変更した場合は、必要に応じて`php artisan test`や`php artisan migrate:status`を実行する。
- Vue.js側を変更した場合は、必要に応じて`npm run build`を実行する。
- 確認コマンドは原則としてDocker Composeの各サービス上で実行する。

## 変更作業時のルール

変更作業を行う際は、既存の構成と方針を優先する。

- 依頼された範囲に関係しない変更は行わない。
- 既存のディレクトリ構成、命名、実装方針に合わせる。
- 必要性が明確でない大きなリファクタリングは行わない。
- 新しいライブラリやツールを追加する場合は、目的と影響範囲を明確にする。
- ユーザーが作成した変更を、明示的な依頼なしに戻さない。
