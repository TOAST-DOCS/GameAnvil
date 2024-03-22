## Game > GameAnvil > CocosCreator開発ガイド > GameAnvilコネクタのインストール

## GameAnvilコネクタのインストール

まず、新しいCocosCreatorプロジェクトを作成します。**Cocos Creator Dashboard > New Project > Empty Project**を選択して新しいプロジェクトを作成します。

![new-project](https://static.toastoven.net/prod_gameanvil/images/client-2-new-project.png)

![new-project-empty](https://static.toastoven.net/prod_gameanvil/images/client-2-new-project-empty.png)

作成されたプロジェクトにGameAnvilコネクタを追加します。GameAnvilコネクタは[ここ](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript.zip)からダウンロードできます。ダウンロードしたファイルを解凍してassetsの下にフォルダを作って入れます。 この時、次のような通知が表示されたら**No**をクリックします。

![set-as-a-plugin](https://static.toastoven.net/prod_gameanvil/images/client-2-set-as-a-plugin.png)

![gameanvil-project](https://static.toastoven.net/prod_gameanvil/images/client-2-gameanvil-project.png)

GameAnvilコネクタはprotobufjsを使用します。それでGameAnvilコネクタをプロジェクトに入れると上のようなエラーが出ることが確認できます。

protobufjsはNode.jsのinstall機能を使ってプロジェクトに含めます。VSCodeターミナルで次のコードを入力してpackage.jsonファイルを作成します。

```
npm init
```

上のコードを入力したらpackage.jsonファイル生成プロセスが始まってパッケージ名を入力します。 この時、Enterを押すとデフォルトでプロジェクト名を使います。以降に入力される値もEnterを押すとデフォルト値を使います。詳しく内容は[ここ](https://docs.npmjs.com/cli/v6/commands/npm-init)を参照してください。 

![npm-init](https://static.toastoven.net/prod_gameanvil/images/client-2-npm-init.png)

デフォルト値で作成したpackage.jsonファイルを開いてみると次のとおりです。 

```json
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

package.jsonファイルを生成したら、ターミナルで次のコードを入力してprotobufjsをインストールします。

```
npm i protobufjs --save
```

![protobufjs](https://static.toastoven.net/prod_gameanvil/images/client-2-protobufjs.png)

インストールが完了したら、プロジェクトフォルダの下に下記のようなフォルダが作成されます。

- node_modules
- .bin
- @protobufjs
- @types
- long
- protobufjs

package.jsonファイルのdependenciesにgameanvil_connectorが追加されたことも確認できます。

```json
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "protobufjs": "^6.10.2"
  }
}
```

Internet Explorerなど古いバージョンのブラウザをサポートするためcore-js, regenerator-runtimeを追加でインストールします。

```
npm i core-js regenerator-runtime
```
