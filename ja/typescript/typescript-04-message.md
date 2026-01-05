## Game > GameAnvil > CocosCreator開発ガイド > メッセージハンドリング

## メッセージ

GameAnvilConnector、GameAnvilUserの基本機能の他に、request()とsend()を利用してカスタムメッセージをサーバーに送信できます。メッセージを送信するには、メッセージを事前に登録する必要があり、メッセージオブジェクトを作成するプロセスが必要です。

### メッセージ生成

GameAnvilは、メッセージプロトコルとして[Google Protocol Buffers](https://developers.google.com/protocol-buffers/docs/proto3)を標準で提供しています。.protoファイルにメッセージを定義し、protocで実際のクラスのソースコードにトランスパイルします。このように生成されたソースコードをプロジェクトに追加して使用します。

まず、使用するメッセージを作成し、protoフォルダ内に配置します。

```protocolbuffer
// message.proto
syntax = "proto3";

package Messages;

message UserInfo
{
  string name = 1;
  int32 age = 2;
  string job = 3;
}

message EchoReq
{
  string message = 1;
}

message EchoRes
{
  string message = 1;
}
```

次に、message.protoファイルをトランスパイルするためのライブラリをインストールします。

```shell
npm install protobufjs@7.2.6 protobufjs-cli@1.1.2
```

まず、pbjsを利用して.jsファイルへのトランスパイルを完了した後、game-server-connectorのinsertCode.jsを利用して必要なコードを挿入します。その後、pbtsを使用して.tsファイルにトランスパイルします。以下は、特定のディレクトリに対して説明した作業を自動的に実行するサンプルコードです。

```js
import fs from 'fs';
import { execSync } from 'child_process';
import path from 'path';

((srcDirectoryPath, distDirectoryPath) => {

    console.log(`Compile protobuf files in path: ${srcDirectoryPath}`);

    const srcFiles = fs.readdirSync(srcDirectoryPath);
    for ( const srcFileName of srcFiles){
        const distFileName = srcFileName.replace('.proto', '.js');

        const srcFilePath = path.join(srcDirectoryPath, srcFileName);
        const distFilePath = path.join(distDirectoryPath, distFileName);

        const packageName = getPackageNameFromProtoFile(srcFilePath);
        execSync(`pbjs --force-message --no-verify --no-convert --no-delimited -t static-module --wrap es6 --out ${distFilePath} ${srcFilePath}`);
        execSync(`node ../../game-server-connector/GameAnvil-connector-typescript/bin/insertCode.js ${distFilePath} ${srcFileName} ${packageName}`);
        execSync(`pbts --no-comments --out ${distFilePath.replace('.js', '.d.ts')} ${distFilePath}`);
    }

})(...process.argv.slice(2));

function getPackageNameFromProtoFile(filePath){
    const content = fs.readFileSync(filePath, {encoding:'utf8', flag:'r'});
    const packageLine = content.split(/\r?\n/).find(line => line.indexOf('package ') > -1);
    
    return packageLine.replace('package ', '').replace(';', '');
}
```

上記のスクリプトをbin/protoc.jsとして保存し、package.jsonに以下のように登録して使用します。これは、./protoフォルダにある.protoファイルをトランスパイルした結果を、./protocolに保存するスクリプトです。

```json
{
  "name": "my-game",
  "scripts": {
    "protoc" : "node ./bin/protoc.js ./proto ./protocol"
  }
}
```

ファイルが全て正しい場所にあるかをもう一度確認します。

* /bin/protoc.js
* /proto/message.proto
* - /protocolフォルダを事前に作成

問題がなければ、npmを利用してprotoc.jsを実行します。

```shell
npm run protoc
```

すると、./protocolフォルダに.tsと.d.tsファイルが作成されたことを確認できます。

### メッセージ登録

新しく作成したメッセージを使用するには、使用したいメッセージをGameAnvilProtocolManagerに事前に登録する必要があります。Authenticateの前に登録しなかったり、サーバーとプロトコルが異なったりすると、誤動作する可能性があります。

```typescript
import { Messages } from "./protocol/message";

GameAnvilProtocolManager.registerProtocol(Messages);
```

登録したメッセージの使用を解除するには、unregisterProtol()メソッドを使用します。

```typescript
GameAnvilProtocolManager.unregisterProtocol(Messages);
```

### メッセージ送信

コネクタやユーザーを通じて作成したメッセージを送信できます。以下は、コネクタを通じてメッセージを送信する例です。

```typescript
import { Messages } from "./protocol/message";

const name: string;
const age: number;
const job: string;

const userInfo: UserInfo = new UserInfo({name, age, job});
connector.send(userInfo);
```

以下は、コネクタを通じてメッセージを送信し、サーバーの応答を待つ例です。

```typescript
const message = "Hello World!";
const echoReq: EchoReq = new EchoReq({message});

const result = await connector.requestMessage(echoReq, EchoRes.descriptor);

if (result.resultCode === ResultCode.Success) {
    const echoRes: EchoRes = result.data;
    console.log(echoRes.message); // Hello World! と出力
}
```

以下は、ユーザーを通じてメッセージを送信する例です。

```typescript
import { Messages } from "./protocol/message";

const name = "John Doe";
const age = 20;
const job = "artist";

const message: UserInfo = new UserInfo({name, age, job});
user.sendUser(message);
```

以下は、ユーザーを通じてメッセージを送信し、サーバーの応答を待つ例です。

```typescript
const echoResult = await user.requestUser<EchoRes>(new EchoReq({ message: "Hello World!"}), EchoRes);

console.log(echoResult.message); // Hello World! と出力
```
