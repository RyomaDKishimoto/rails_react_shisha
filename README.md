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

以下のコマンドでcontrollerを生成します.
`rails g controller Homepage index` Homepageというcontroller

以下ファイルも一緒に生成されます.
- `homepage_controller.rb`ファイルはhomepageに関するリクエストをレシーブします.
- `homepage.js`Homepageコントローラーに関するJavascriptの振る舞いを追記するファイルです.
- `homepage.scss`Homepageコントローラーに関するstylesを追記するファイルです.
- `homepage_helper.rb` helper methodsを追記するhelperファイル
- An index.html.erb file which is the view page for rendering anything related to the homepage.



# Step 5 — Reactの設定 Railsのフロントエンドとして


# Step 6 — Shisa ControllerとModelを生成
このステップではShisaのControllerとModelを作成します.
Shisa ModelはControllerから受け取ってハンドリングする情報を保持するデータベースのtableを表します.
ユーザーがShishaをリクエストする際、Shisha controller がリクエストを受け取ってrecipe modelへパスします, 取得する、リクエストされたデータをデータベースから.
Modelがcontrollerからの返り値としてShishaデータを返す.


スタートしましょう. Shisha Model を作成することでRailsで提供されているModelを生成するサブコマンドを使うことで,そしてModelを名前を指定して.
Start by creating a Recipe model by using the generate model subcommand provided by Rails and by specifying the name of the model along with its columns and data types. 

ターミナルで以下のコマンドをターミナルで Run してShisha Modelを生成します.
Run the following command in your Terminal window to create a Recipe model:

`rails generate model Recipe name:string ingredients:text instruction:text image:string`

上記コマンドでカラムと名前を一緒にmodelを生成できます.生成する際にカラムのタイプを一緒に指定してあげることができます.
`generate model` コマンドで 以下のファイルが生成されます.

- recipe.rb file that holds all the model related logic.
- A `20190407161357_create_recipes.rb` file (the number at the beginning of the file may differ depending on the date when you run the command). This is a migration file that contains the instruction for creating the database structure.


次に, shishaファイルにeditします. databaseへの保存が有効なデータを確認します.
それを達成するために, modelにいくつかのバリデーションを追記します.

`nano ~/rails_react_recipe/app/models/recipe.rb`

`
class Recipe < ApplicationRecord
  validates :name, presence: true
  validates :ingredients, presence: true
  validates :instruction, presence: true
end
`

- データベースの更新
`rails db:migrate`

- rails generate controller <コントローラー名> <アクション名>
`rails generate controller api/v1/Recipes index create show destroy -j=false -y=false --skip-template-engine --no-helper`


In this command, you created a Recipes controller in an api/v1 directory with an index, create, show, and destroy action. The index action will handle fetching all your recipes, the create action will be responsible for creating new recipes, the show action will fetch a single recipe, and the destroy action will hold the logic for deleting a recipe.

You also passed some flags to make the controller more lightweight, including:

- -j=false which instructs Rails to skip generating associated JavaScript files.
- -y=false which instructs Rails to skip generating associated stylesheet files.
- --skip-template-engine, which instructs Rails to skip generating Rails view files, since React is handling your front-end needs.
- --no-helper, which instructs Rails to skip generating a helper file for your controller.


`
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      get 'recipes/index'
      post 'recipes/create'
      get '/show/:id', to: 'recipes#show'
      delete '/destroy/:id', to: 'recipes#destroy'
    end
  end
  root 'homepage#index'
  get '/*path' => 'homepage#index'
  For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
`
このルートファイルでは、データを投稿したり削除したりできるように、createとdestroyのルートのHTTP動詞を変更しました。
また、ルートに:idパラメータを追加することで、showとdestroyアクションのためのルートを修正しました。

また、既存のルートにマッチしない他のリクエストをホームページコントローラーのインデックスアクションに誘導するget '/*path'を持つキャッチオールートを追加しました。
このようにして、フロントエンドのルーティングはレシピの作成、読み込み、削除に関係のないリクエストを処理します。

このコマンドを実行すると、プロジェクトのURIパターン、動詞、一致するコントローラまたはアクションのリストが表示されます。

次に、すべてのレシピを一度に取得するロジックを追加します。
RailsはActiveRecordライブラリを使って、このようなデータベース関連のタスクを処理しています。
ActiveRecordはクラスをリレーショナルデータベースのテーブルに接続し、それらを操作するための豊富なAPIを提供します。

すべてのレシピを取得するには、ActiveRecordを使ってレシピテーブルをクエリし、データベースに存在するすべてのレシピを取得します。
`
class Api::V1::RecipesController < ApplicationController
  def index
    recipe = Recipe.all.order(created_at: :desc)
    render json: recipe
  end

  def create
  end

  def show
  end

  def destroy
  end
end
`

インデックスアクションでは、ActiveRecordが提供するallメソッドを使用して、データベース内のすべてのシーシャを取得します.
order メソッドを使用して、作成日の降順に並べ替えます。この方法では、新しいシーシャを最初に取得します.
最後に、ShishasのリストをJSONレスポンスとしてrenderで送信します.

次に、新しいレシピを作成するためのロジックを追加します.
すべてのシーシャを取得するのと同様に、提供されたシーシャの詳細を検証して保存するためにActiveRecordに依存します.
次のハイライトされた行のコードでシーシャコントローラを更新します.

`
class Api::V1::RecipesController < ApplicationController
  def index
    recipe = Recipe.all.order(created_at: :desc)
    render json: recipe
  end

  def create
    recipe = Recipe.create!(recipe_params)
    if recipe
      render json: recipe
    else
      render json: recipe.errors
    end
  end

  def show
  end

  def destroy
  end

  private

  def recipe_params
    params.permit(:name, :image, :ingredients, :instruction)
  end
end
`

createアクションでは、ActiveRecordのcreateメソッドを使用して新しいレシピを作成します.
createメソッドは、モデルに提供されたすべてのコントローラパラメータを一度に割り当てる機能を持っています.
このため、レコードを簡単に作成できる反面、悪意のある利用の可能性が出てきます。これは、強力なパラメータとして知られるRailsが提供する機能を使用することで防ぐことができます.
この方法では、ホワイトリストに登録されていない限り、パラメータを割り当てることはできません.

あなたのコードでは、createメソッドにrecipe_paramsパラメータを渡しています.
recipe_paramsはプライベートメソッドで、間違ったコンテンツや悪意のあるコンテンツがデータベースに入るのを防ぐためにコントローラのパラメータをホワイトリスト化しています.
このケースでは、createメソッドの有効な使用のために、名前、画像、成分、命令パラメータを許可しています.

これで、レシピコントローラはレシピを読み込んで作成することができます.
あとは、1つのレシピを読み込んで削除するロジックだけです.次のコードでレシピコントローラを更新してください.

`
class Api::V1::RecipesController < ApplicationController
  def index
    recipe = Recipe.all.order(created_at: :desc)
    render json: recipe
  end

  def create
    recipe = Recipe.create!(recipe_params)
    if recipe
      render json: recipe
    else
      render json: recipe.errors
    end
  end

  def show
    if recipe
      render json: recipe
    else
      render json: recipe.errors
    end
  end

  def destroy
    recipe&.destroy
    render json: { message: 'Recipe deleted!' }
  end

  private

  def recipe_params
    params.permit(:name, :image, :ingredients, :instruction)
  end

  def recipe
    @recipe ||= Recipe.find(params[:id])
  end
end
`
