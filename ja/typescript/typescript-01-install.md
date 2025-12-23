## Game > GameAnvil > TypeScript開発ガイド > インストール

## GameAnvil Connector

コネクタは、GameAnvilサーバーに合わせたクライアントを制作するために開発されたライブラリです。GameAnvil Connectorを利用すると、GameAnvilが基本的に提供するパケット送信、ユーザー及びルーム機能などを簡単に実装できます。


### gameanvil-connector.jsのダウンロード

コネクタは以下からダウンロードできます。

[gameanvil-connector-typescript.zip](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector-typescript.zip)

npmを通じたダウンロードやgitを通じたダウンロードは、今後サポートされる予定です。

### gameanvil-connector.jsのインストール例

このセクションでは、GameAnvil ConnectorのTypeScriptバージョンのインストールと利用方法について説明します。

このガイドではnode.jsを使用するため、事前にインストールが完了した環境が必要です。また、読者が基本的なnode.jsの知識を持っていることを前提に説明します。もしnode.jsのインストールや利用について詳しくない場合は、node.jsの公式資料をご参照ください。node.js以外に、別途使用したいゲーム制作用のライブラリがあれば、それを使用することもできます。例えば、CocosやPhaserなどのHTML5ベースのゲームライブラリを使用できます。

コマンドプロンプトまたはターミナルを実行します。そして、以下のコマンドを入力してコネクタをインストールするフォルダを作成します。

```cli
$ mkdir my-game
$ cd my-game
$ npm init
```

ダウンロードしたコネクタファイルを解凍し、先ほど作成したmy-gameフォルダの下に移動します。そして、npmを通じてインストールします。

```cli
$ npm install ./gameanvil-connector
```

これで、node.jsプロジェクトへのgameanvil-connectorのインストールは完了です。プロジェクト内部で、以下のように「gameanvil-connector」としてライブラリを使用できます。

```typescript
// index.ts
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();

connector.host = "127.0.0.1";
connector.port = 18300;
await connector.connect();
```

上記のコードは、コネクタを使用する最も簡単なサンプルコードです。動作を簡単に説明すると、以下のようになります。

1. まず、GameAnvilConnectorオブジェクトを作成します。GameAnvilConnectorは、サーバーとの通信を管理し、サーバーにリクエストを送信したり、サーバーのレスポンスを受け取ったりできるオブジェクトです。
2. 次に、GameAnvilConnectorオブジェクトに接続情報(host, port)を入力します。これは開発サーバーによって内容が異なる場合があるため、必ず開発したサーバーに合わせて内容を修正してください。
3. 最後に、connect()関数を呼び出してサーバーに接続リクエストを送信します。

### gameanvil-connector.jsの実行例

サンプルコードを実行するために、Webpackをインストールして実行してみましょう。
index.htmlファイルを作成します。

```html
<!DOCTYPE html>
<head>
    <title>My Game</title>
</head>
<body>

</body>
```

必要なライブラリを全てインストールします。

```cli
$ npm install webpack webpack-cli webpack-dev-server html-webpack-plugin babel-loader @babel/core  @babel/preset-env @babel/preset-typescript
```

webpackの設定ファイルを作成し、以下のように記述します。

```javascript
// webpack.config.cjs
const webpack = require( 'webpack');
const path = require( 'path');
const HtmlWebpackPlugin = require( 'html-webpack-plugin');


module.exports = {
    mode: "development",
    devtool: 'source-map',
    entry: {
        entry: ['./index.ts']
    },
    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'bundle.js',
    },
    resolve: {
        extensions: ['.ts', '.js'],
    },
    devServer: {
        watchFiles: path.join(__dirname, 'dist'),
        compress: false,
        port: 8081
    },
    module: {
        rules: [
            {
                test: /\.(d\.ts|ts|js)$/,
                exclude: '/node_modules/',
                loader: 'babel-loader',
                options: {
                    presets: [
                        '@babel/preset-env', '@babel/preset-typescript'
                    ]
                },
                resolve: {
                    fullySpecified: false
                }
            },
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './index.html',
        })
    ]
};
```

package.jsonファイルに以下のスクリプトを追加します。

```json
"scripts": {
  "start": "webpack serve --progress --open --config ./webpack.config.cjs"
},
```

動作をより詳しく確認するため、サンプルコードを修正します。

```typescript
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();
connector.host = "127.0.0.1";
connector.port = 18300;

connector.connect()
  .then(()=>{
    // connectに成功した場合
    document.body.append("Connect Success");
  })
  .catch(()=>{
    // connectに失敗した場合
    document.body.append("Connect Fail");
  })
```

Webpackを実行して動作を確認します。

```
$ npm run start
```

インターネットブラウザのウィンドウが開くと、「Connect Success」または「Connect Fail」という文言が出力されます。
