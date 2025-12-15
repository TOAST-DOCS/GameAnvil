## Game > GameAnvil > TypeScript Development Guide > Install

## GameAnvil Connector

The connector is a library developed to create clients suitable for the GameAnvil server. With GameAnvil Connector, you can easily implement the packet transfer, user, and room features provided by GameAnvil.


### Download gameanvil-connector.js

You can download the connector below:

[gameanvil-connector-typescript.zip](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector-typescript.zip)

Downloads via npm and via git are expected to be supported.

### Example of Installation of gameanvil-connector.js

This section covers how to install and use the TypeScript version of the GameAnvil Connector. 

This guide requires a pre-installed environment because it uses node.js, and assumes and explains that the reader has the basic knowledge of node.js. If you have no knowledge of how to install or use node.js, please refer to the node.js official materials. If you have a game creation library other than node.js that you want to use separately, you can use it. For example, you can use html5-based game libraries, such as Cocos or Phaser.

Run the command prompt or terminal. And enter the command below to create a folder to install the connector:

```cli
$ mkdir my-game
$ cd my-game
$ npm init
```

Unzip the downloaded connector file and navigate to the my-game folder you created earlier. And install via npm.

```cli
$ npm install ./gameanvil-connector
```

With this, the installation of gameanvil-connector to the node.js project has been completed. You can use the Library as "gameanvil-connector" inside the project as shown below:

```typescript
// index.ts
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();

connector.host = "127.0.0.1";
connector.port = 18300;
await connector.connect();
```

The code above is the simplest example code that uses a connector. To simply explain the action, it is as follows:

1. First, create a GameAnvilConnector object. GameAnvilConnector is an object that can manage communication with the server and send a request to the server or receive a response from the server.
2. And then, enter the connection information (host, port) for the GameAnvilConnector object. It can vary depending on the development server, so be sure to modify the content to fit the developed server.
3. Finally, call the connect() function to send the connection request to the server.

### Example of Running gameanvil-connector.js

To run the example code, we install and run Webpack.
Create the index.html file.

```html
<!DOCTYPE html>
<head>
    <title>My Game</title>
</head>
<body>

</body>
```

Install all required libraries.

```cli
$ npm install webpack webpack-cli webpack-dev-server html-webpack-plugin babel-loader @babel/core  @babel/preset-env @babel/preset-typescript
```

Create a webpack setting file and write it as follows:

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

Add the following script to the package.json file:

```json
"scripts": {
  "start": "webpack serve --progress --open --config ./webpack.config.cjs"
},
```

To see the operation in more detail, modify the example code.

```typescript
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();
connector.host = "127.0.0.1";
connector.port = 18300;

connector.connect()
  .then(()=>{
    // if connect is succeeded
    document.body.append("Connect Success");
  })
  .catch(()=>{
    // if connect is failed
    document.body.append("Connect Fail");
  })
```

Launch the webpack and check the action.

```
$ npm run start
```

When the internet browser window pops up, a Connect Success or Connect Fail phrase is printed out.