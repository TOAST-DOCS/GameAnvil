## Game > GameAnvil > CocosCreator開発ガイド > メッセージハンドリング

## メッセージ

ConnectionAgent、UserAgentの基本機能以外にRequest()とSend()を使ってメッセージをサーバーに送信できます。メッセージを送信するためにはメッセージを作成して登録する過程が必要です。

### メッセージの作成

GameAnvilは基本メッセージプロトコルとして[Google Protocol Buffers](https://developers.google.com/protocol-buffers/docs/proto3)を使っています。.protoファイルにメッセージを定義して、pbjsで実際のクラスのソースコードを作成します。 そして、GameAnvilで使う追加コードを作成されたコードに挿入します。このように作成されたソースコードをプロジェクトに追加して使うことができます。

追加コードを挿入するには `CodeInserter` をインストールする必要があります。CodeInserter` は[ここ](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-CodeInserter.zip)からダウンロードできます。ダウンロードしたファイルを解凍してプロジェクトルートの下(assetsフォルダの外)にフォルダを作って格納します。

![codeInserter](https://static.toastoven.net/prod_gameanvil/images/client-2-codeInserter.png)

`CodeInserter`はacornを使用します。ターミナルで次のコードを入力してacornをインストールします。

```
npm i acorn@5.5.3 --save-dev
```

package.jsonのdevDependenciesにacornが追加されたことが確認できます。

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
  },j
  "devDependencies": {
    "acorn": "^5.5.3"
  }
}
```

次はメッセージを作ってみます。 まず、assetsフォルダの下にprotocolsフォルダを作成して次のようにmessages.protoファイルを作成します。

```protobuf
// messages.proto
syntax = "proto3";

package Messages;

message SampleRequest
{
  string msg = 1;
}

message SampleResponse
{
  repeated string msgs = 1;
}

message SampleSend
{
  string msg = 1;
}

message SampleReceive
{
  repeated string msgs = 1;
}
```

次に、messages.protoファイルをコンパイルするためのスクリプトをpackage.jsonに追加します。

```json
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "messages.js": "pbjs --force-message --no-verify --no-convert --no-delimited -t static-module -w default -r base -o assets/protocols/messages.js assets/protocols/messages.proto",
    "messages.codeInsert": "node codeInserter/CodeInserter assets/protocols/messages.js",
    "messages.ts": "pbts --no-comments -o assets/protocols/messages.d.ts assets/protocols/messages.js",
    "messages": "npm run messages.js && npm run messages.codeInsert && npm run messages.ts",
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "protobufjs": "^6.10.2"
  },
  "devDependencies": {
    "acorn": "^5.5.3"
  }
}
```

そしてターミナルで次のコードを入力してメッセージを作成します。

```
npm run messages
```

次のように`message.js`, `message.d.ts`ファイルが作成されたことを確認できます。

![messages](https://static.toastoven.net/prod_gameanvil/images/client-2-messages.png)

### メッセージの登録

新しく作成したメッセージを使うためには、使用するメッセージをProtocolManagerにサーバーと同じ値で事前に登録する必要があります。 登録をしなかったり、サーバーと違う場合、動作しないか、誤動作したり、例外が発生する可能性があります。

```typescript
// サーバーと同じ値で登録する必要があります。
ProtocolManager.RegisterProtocol(0, message);
```

### メッセージの送信

RequestPb()でメッセージを送信すると、サーバーレスポンスを待ちます。サーバーレスポンスを待っている間、追加のRequestPb()はキューに保存され、サーバーレスポンスを処理した後、順次処理します。サーバーレスポンスを受信して処理するにはリスナーを登録する必要があります。リスナーを登録していない場合、サーバーレスポンスを受け取っても別途の通知をせずに次のメッセージを処理することになります。指定された時間内にレスポンスが来ない場合、タイムアウトを発生させて次のメッセージを処理します。タイムアウトはOnError()リスナーにErrorCode.TIMEOUTが伝達されます。

SendPb()でメッセージを送信すると、SendPb()の呼び出しと同時にサーバーに送信され、別途のレスポンスを待たずに送信されます。RequestPb()のレスポンスを待っている間も、SendPb()を使ったメッセージはすぐにサーバーに送信されます。

```typescript
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();

connection.AddCallback(Messages.SampleReceive, (connectionAgent, receive)=>{
    // Messages.SampleReceive
});
let sampleSend= new Messages.SampleSend (); 
connection.SendPb(sampleSend);


connection.AddCallback(Messages.SampleResponse, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
let sampleRequest = new Messages.SampleRequest();
connection.RequestPb(sampleRequest);
// レスポンス Messages.SampleResponse.

connection.RequestPb<Messages.SampleResponse>(sampleRequest, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
user.AddCallback(Messages.SampleReceive, (connectionAgent, receive)=>{
    // Messages.SampleReceive
});
let sampleSend= new Messages.SampleSend (); 
user.SendPb(sampleSend);


user.AddCallback(Messages.SampleResponse, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
let sampleRequest = new Messages.SampleRequest();
user.RequestPb(sampleRequest);
// レスポンス Messages.SampleResponse.

user.RequestPb<Messages.SampleResponse>(sampleRequest, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
```

### カスタムメッセージ

パケットクラスを利用してProtocolBuffer以外の任意のデータをバイトストリームでシリアル化して使うことができます。パケットについての詳細は[こちら](cocos-06-packet.md)を参照してください。

```typescript
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
let reqMsgId = 1;
let resMsgId = 2;
let enc = new TextEncoder();

connection.AddUndefinedProtocolCallback(resMsgId, this.onUndefinedProtocolResponse);
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data"));
connection.Request(packet);
// onUndefinedProtocolResponseレスポンス

connection.Request(packet, (connectionAgent, packet) => {
    // ここにレスポンス
});
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
let reqMsgId = 1;
let resMsgId = 2;
let enc = new TextEncoder();

user.AddUndefinedProtocolCallback(resMsgId, this.onUndefinedProtocolResponse);
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data"));
user.Request(packet);
// onUndefinedProtocolResponseレスポンス

user.Request(packet, (connectionAgent, packet) => {
    // ここにレスポンス
});
```
