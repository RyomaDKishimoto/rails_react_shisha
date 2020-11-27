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

```
nano ~/rails_react_recipe/app/models/recipe.rb
```

```
class Recipe < ApplicationRecord
  validates :name, presence: true
  validates :ingredients, presence: true
  validates :instruction, presence: true
end
```

- データベースの更新
```
rails db:migrate
```

- rails generate controller <コントローラー名> <アクション名>
```
rails generate controller api/v1/Recipes index create show destroy -j=false -y=false --skip-template-engine --no-helper
```


In this command, you created a Recipes controller in an api/v1 directory with an index, create, show, and destroy action. The index action will handle fetching all your recipes, the create action will be responsible for creating new recipes, the show action will fetch a single recipe, and the destroy action will hold the logic for deleting a recipe.

You also passed some flags to make the controller more lightweight, including:

- -j=false which instructs Rails to skip generating associated JavaScript files.
- -y=false which instructs Rails to skip generating associated stylesheet files.
- --skip-template-engine, which instructs Rails to skip generating Rails view files, since React is handling your front-end needs.
- --no-helper, which instructs Rails to skip generating a helper file for your controller.


```
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
```

このルートファイルでは、データを投稿したり削除したりできるように、createとdestroyのルートのHTTP動詞を変更しました。
また、ルートに:idパラメータを追加することで、showとdestroyアクションのためのルートを修正しました。

また、既存のルートにマッチしない他のリクエストをホームページコントローラーのインデックスアクションに誘導するget '/*path'を持つキャッチオールートを追加しました。
このようにして、フロントエンドのルーティングはレシピの作成、読み込み、削除に関係のないリクエストを処理します。

このコマンドを実行すると、プロジェクトのURIパターン、動詞、一致するコントローラまたはアクションのリストが表示されます。

次に、すべてのレシピを一度に取得するロジックを追加します。
RailsはActiveRecordライブラリを使って、このようなデータベース関連のタスクを処理しています。
ActiveRecordはクラスをリレーショナルデータベースのテーブルに接続し、それらを操作するための豊富なAPIを提供します。

すべてのレシピを取得するには、ActiveRecordを使ってレシピテーブルをクエリし、データベースに存在するすべてのレシピを取得します。
```
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
```

インデックスアクションでは、ActiveRecordが提供するallメソッドを使用して、データベース内のすべてのシーシャを取得します.
order メソッドを使用して、作成日の降順に並べ替えます。この方法では、新しいシーシャを最初に取得します.
最後に、ShishasのリストをJSONレスポンスとしてrenderで送信します.

次に、新しいレシピを作成するためのロジックを追加します.
すべてのシーシャを取得するのと同様に、提供されたシーシャの詳細を検証して保存するためにActiveRecordに依存します.
次のハイライトされた行のコードでシーシャコントローラを更新します.

```
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
```

createアクションでは、ActiveRecordのcreateメソッドを使用して新しいレシピを作成します.
createメソッドは、モデルに提供されたすべてのコントローラパラメータを一度に割り当てる機能を持っています.
このため、レコードを簡単に作成できる反面、悪意のある利用の可能性が出てきます。これは、強力なパラメータとして知られるRailsが提供する機能を使用することで防ぐことができます.
この方法では、ホワイトリストに登録されていない限り、パラメータを割り当てることはできません.

あなたのコードでは、createメソッドにrecipe_paramsパラメータを渡しています.
recipe_paramsはプライベートメソッドで、間違ったコンテンツや悪意のあるコンテンツがデータベースに入るのを防ぐためにコントローラのパラメータをホワイトリスト化しています.
このケースでは、createメソッドの有効な使用のために、名前、画像、成分、命令パラメータを許可しています.

これで、レシピコントローラはレシピを読み込んで作成することができます.
あとは、1つのレシピを読み込んで削除するロジックだけです.次のコードでレシピコントローラを更新してください.

```
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
```

新しいコードの行では、プライベートレシピメソッドを作成しました.
recipeメソッドはActiveRecordのfindメソッドを使用して、そのidmatchesがparamsで提供されたidと一致するレシピを見つけて、インスタンス変数@recipeに割り当てます.
showアクションでは、レシピがレシピメソッドによって返されるかどうかをチェックし、JSONレスポンスとして送信するか、そうでない場合はエラーを送信しました.

destroyアクションでは、Rubyの安全なナビゲーション演算子&.を使って似たようなことをしました.
これにより、レシピが存在する場合にのみレシピを削除し、レスポンスとしてメッセージを送信することができます.

recipes_controller.rbへの変更が完了したら、ファイルを保存してテキストエディタを終了してください.
このステップでは、レシピのモデルとコントローラを作成しました.
これで、バックエンドでレシピを扱うために必要なすべてのロジックが記述されました.
次のセクションでは、レシピを表示するためのコンポーネントを作成します.



# Step 7 — Viewing Shishas

このセクションでは、レシピを表示するためのコンポーネントを作成します。
まず、既存のすべてのレシピを表示するページを作成し、次に個別のレシピを表示するページを作成します。
まずはすべてのレシピを表示するページを作成します。
しかし、これを行う前に、データベースは現在空なので、作業するためのレシピが必要です。
Railsは、アプリケーションのシードデータを作成する機会を与えてくれます。

```
9.times do |i|
  Recipe.create(
    name: "Recipe #{i + 1}",
    ingredients: '227g tub clotted cream, 25g butter, 1 tsp cornflour,100g parmesan, grated nutmeg, 250g fresh fettuccine or tagliatelle, snipped chives or chopped parsley to serve (optional)',
    instruction: 'In a medium saucepan, stir the clotted cream, butter, and cornflour over a low-ish heat and bring to a low simmer. Turn off the heat and keep warm.'
  )
end
```

このコードでは、ループを使って、名前、材料、命令を指定して9つのレシピを作成するようにRailsに指示しています。ファイルを保存して終了します。
このデータをデータベースにシードするには、ターミナルウィンドウで次のコマンドを実行します。
```
rails db:seed
```

このコマンドを実行すると、9つのレシピがデータベースに追加されます。これで、フロントエンドでレシピを取得してレンダリングすることができるようになりました。

すべてのレシピを表示するコンポーネントは、すべてのレシピのリストを取得するために RecipesController の index アクションに HTTP リクエストを行います。これらのレシピはページ上にカードで表示されます。
app/javascript/components ディレクトリに Recipes.jsx ファイルを作成します。
次に、React.Component クラスを継承する Recipes クラスを作成します。
以下のハイライトされたコードを追加して、React.Componentを継承したReactコンポーネントを作成します。

```
import React from "react";
import { Link } from "react-router-dom";

class Recipes extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      recipes: []
    };
  }

}
export default Recipes;
```


コンストラクタの内部では、レシピの状態を保持するステートオブジェクトを初期化しています。

次に、RecipeクラスにcomponentDidMountメソッドを追加します。componentDidMountメソッドは、コンポーネントがマウントされた直後に呼び出されるReactのライフサイクルメソッドです。このライフサイクルメソッドでは、すべてのレシピをフェッチするための呼び出しを行います。これを行うには、以下の行を追加します。


```
import React from "react";
import { Link } from "react-router-dom";

class Recipes extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      recipes: []
    };
  }

  componentDidMount() {
      const url = "/api/v1/recipes/index";
      fetch(url)
        .then(response => {
          if (response.ok) {
            return response.json();
          }
          throw new Error("Network response was not ok.");
        })
        .then(response => this.setState({ recipes: response }))
        .catch(() => this.props.history.push("/"));
  }

}
export default Recipes;
```

componentDidMountメソッドでは、Fetch APIを使用してすべてのレシピを取得するためにHTTP呼び出しを行いました。レスポンスが成功した場合、アプリケーションはレシピの配列をレシピの状態に保存します。エラーが発生した場合は、ユーザーをホームページにリダイレクトします。

最後に、レシピクラスにレンダーメソッドを追加します。renderメソッドは、コンポーネントがレンダリングされたときに評価されてブラウザページに表示されるReact要素を保持します。この場合、renderメソッドはコンポーネントの状態からレシピのカードをレンダリングします。Recipes.jsxに以下のハイライトされた行を追加します


```
import React from "react";
import { Link } from "react-router-dom";

class Recipes extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      recipes: []
    };
  }

  componentDidMount() {
    const url = "/api/v1/recipes/index";
    fetch(url)
      .then(response => {
        if (response.ok) {
          return response.json();
        }
        throw new Error("Network response was not ok.");
      })
      .then(response => this.setState({ recipes: response }))
      .catch(() => this.props.history.push("/"));
  }
  render() {
    const { recipes } = this.state;
    const allRecipes = recipes.map((recipe, index) => (
      <div key={index} className="col-md-6 col-lg-4">
        <div className="card mb-4">
          <img
            src={recipe.image}
            className="card-img-top"
            alt={`${recipe.name} image`}
          />
          <div className="card-body">
            <h5 className="card-title">{recipe.name}</h5>
            <Link to={`/recipe/${recipe.id}`} className="btn custom-button">
              View Recipe
            </Link>
          </div>
        </div>
      </div>
    ));
    const noRecipe = (
      <div className="vw-100 vh-50 d-flex align-items-center justify-content-center">
        <h4>
          No recipes yet. Why not <Link to="/new_recipe">create one</Link>
        </h4>
      </div>
    );

    return (
      <>
        <section className="jumbotron jumbotron-fluid text-center">
          <div className="container py-5">
            <h1 className="display-4">Recipes for every occasion</h1>
            <p className="lead text-muted">
              We’ve pulled together our most popular recipes, our latest
              additions, and our editor’s picks, so there’s sure to be something
              tempting for you to try.
            </p>
          </div>
        </section>
        <div className="py-5">
          <main className="container">
            <div className="text-right mb-3">
              <Link to="/recipe" className="btn custom-button">
                Create New Recipe
              </Link>
            </div>
            <div className="row">
              {recipes.length > 0 ? allRecipes : noRecipe}
            </div>
            <Link to="/" className="btn btn-link">
              Home
            </Link>
          </main>
        </div>
      </>
    );
  }
}
export default Recipes;
```
すべてのレシピを表示するためのコンポーネントを作成したので、次のステップはそのためのルートを作成することです。app/javascript/routes/Index.jsxにあるフロントエンドのルートファイルを開きます。

```
import React from "react";
import { BrowserRouter as Router, Route, Switch } from "react-router-dom";
import Home from "../components/Home";
import Recipes from "../components/Recipes";

export default (
  <Router>
    <Switch>
      <Route path="/" exact component={Home} />
      <Route path="/recipes" exact component={Recipes} />
    </Switch>
  </Router>
);
```
これで、アプリケーションに存在するすべてのレシピを表示できるようになったので、個々のレシピを表示するための2つ目のコンポーネントを作成しましょう。
app/javascript/components ディレクトリに Recipe.jsx ファイルを作成します。
Recipesコンポーネントと同様に、以下の行を追加してReactモジュールとLinkモジュールをインポートします。
```
import React from "react";
import { Link } from "react-router-dom";

class Recipe extends React.Component {
  constructor(props) {
    super(props);
    this.state = { recipe: { ingredients: "" } };

    this.addHtmlEntities = this.addHtmlEntities.bind(this);
  }
}

export default Recipe;
```
Recipesコンポーネントと同様に、コンストラクタではレシピの状態を保持するステートオブジェクトを初期化しました。
また、addHtmlEntitiesメソッドをこのオブジェクトにバインドして、コンポーネント内でアクセスできるようにしました。
addHtmlEntitiesメソッドは、コンポーネント内の文字実体をHTML実体に置き換えるために使われます。

特定のレシピを見つけるために、アプリケーションはレシピのidを必要とします。これは Recipe コンポーネントが id パラメーターを期待していることを意味します。
コンポーネントに渡された小道具を介してこれにアクセスできます。

次に、propsオブジェクトのマッチキーからidパラメータにアクセスするcomponentDidMountメソッドを追加します。
idを取得したら、レシピを取得するためにHTTPリクエストを行います。次のハイライトされた行をファイルに追加してください。


```
import React from "react";
import { Link } from "react-router-dom";

class Recipe extends React.Component {
  constructor(props) {
    super(props);
    this.state = { recipe: { ingredients: "" } };

    this.addHtmlEntities = this.addHtmlEntities.bind(this);
  }

  componentDidMount() {
    const {
      match: {
        params: { id }
      }
    } = this.props;

    const url = `/api/v1/show/${id}`;

    fetch(url)
      .then(response => {
        if (response.ok) {
          return response.json();
        }
        throw new Error("Network response was not ok.");
      })
      .then(response => this.setState({ recipe: response }))
      .catch(() => this.props.history.push("/recipes"));
  }

}

export default Recipe;
```
componentDidMountメソッドでは、オブジェクトの破壊を使用して、propsオブジェクトからidパラメータを取得し、Fetch APIを使用して、idを所有するレシピを取得し、setStateメソッドを使用してコンポーネントの状態に保存するためにHTTPリクエストを行います。レシピが存在しない場合、アプリはユーザーをレシピページにリダイレクトします。

addHtmlEntitiesメソッドを追加します。これは文字列を受け取り、すべてのエスケープされた開括弧と閉括弧をそれらのHTMLエンティティで置き換えます。これは、レシピの命令で,保存されたエスケープされた文字を変換するのに役立ちます。


