# Step 1 — Rails アプリケーション作成

1. `rails new アプリケーション名 -d=postgresql -T --webpack=react --skip-coffee`
- `-d` オプションで特定のデータベースエンジンを指定(今回の場合はPostgreSQL).
- `-T` オプションでテストファイル生成をスキップする(このコマンドはRailsが提供しているツール以外を利用したいケースにも推奨されます).
- `--webpack` オプションでJavascript用のbundlerを事前に準備してくれる.今回のケースはreact.
- `--skip-coffee` オプションでRailsにCoffeeScriptをセットアップしないように指定

# Step 2 — Databaseのセットアップ
Railsアプリケーションを起動する前に、データベースとアプリケーションを接続する必要があります.
このステップでは生成された最新のアプリケーションとPostgreSQLを接続します.
shishaデータ必要時には保存&フェッチされることがあります.

データベースを生成するために以下のコマンドをrunします.
`rails db:create`

さxあ、アプリケーションとデータベースが接続されました.以下のコマンドでアプリケーションを起動させましょう.
`rails s --binding=127.0.0.1`



To see your application, open a browser window and navigate to http://localhost:3000. You will see the Rails default welcome page:


# Step 3 — フロントエンドに依存しているパッケージをインストール
このステップでは、必要なJavascript用の依存パッケージをインストールしましょう.
今回のshishaアプリケーションでは以下を含みます.

- `React Router`, for handling navigation in a React application.
- `Bootstrap`, for styling your front-end components.
- `jQuery` and `Popper`, for working with Bootstrap.

以下のコマンドを Run することでこれらのパッケージをインストールしましょう.
`yarn add react-router-dom bootstrap jquery popper.js`



# Step 4 — ホームページのセットアップ
必要なパッケージをインストールしたら、次はユーザーがサイトに訪れた際に確認できる、このアプリケーションのhomepageを生成します.
RailsはModel-View-Controllerの設計思想を採用しています.MVCパターンではcontrollerの目的は特定のリクエストを受信して適切なmodelかviewへパスします。
デフォルトのroot URLを変更するためにcontroller and view を生成します.

