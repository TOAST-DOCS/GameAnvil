## Game > GameAnvil > クライアント開発ガイド > CocosCreator開発ガイド

ガイド環境

- CocosCreator : 2.2.2.
- Node.js : 10.16.3
- Visual Studio Code : 1.43.0
- GameAnvil Connector : 1.1.0

## GameAnvilコネクタインストール

まず、新しいCocosCreatorプロジェクトを作成します。**Cocos Creator Dashboard > New Project > Empty Project**を選択し、新しいプロジェクトを作成します。 

![new-project](http://static.toastoven.net/prod_gameanvil/images/client-2-new-project.png)

![new-project-empty](http://static.toastoven.net/prod_gameanvil/images/client-2-new-project-empty.png)

次に、作成されたプロジェクトにGameAnvilコネクタを追加します。GameAnvilコネクタは[こちら](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript.zip)からダウンロードできます。ダウンロードしたファイルを解凍し、assetsの下にフォルダを作成して格納します。この時、次のような通知が表示されたら**No**をクリックします。

![set-as-a-plugin](http://static.toastoven.net/prod_gameanvil/images/client-2-set-as-a-plugin.png)

![gameanvil-project](http://static.toastoven.net/prod_gameanvil/images/client-2-gameanvil-project.png)

GameAnvilコネクタは、protobufjsを使用するので、GameAnvilコネクタをプロジェクトに含めると、上記のようなエラーが発生することがあります。

protobufjsはNode.jsのinstall機能を利用してプロジェクトに含めます。VSCode端末で、次のコードを入力して、package.jsonファイルを作成します。

```
npm init
```

上記のコードを入力すると、package.jsonファイルの作成プロセスが始まるので、パッケージの名前を入力します。この時、単にEnterを押すと、デフォルト値のプロジェクト名を使用します。その後に入力する値も、Enterキーを押すと、デフォルト値を使用します。詳細については、[こちら](https://docs.npmjs.com/cli/v6/commands/npm-init)を参照してください。

![npm-init](http://static.toastoven.net/prod_gameanvil/images/client-2-npm-init.png)

デフォルト値で作成したpackage.jsonファイルを開くと、次のようになっています。

```
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

package.jsonファイルを作成したら、ターミナルで次のコードを入力してprotobufjsをインストールします。

```
npm i protobufjs --save
```

![protobufjs](http://static.toastoven.net/prod_gameanvil/images/client-2-protobufjs.png)

インストールが完了したら、プロジェクトフォルダの下に次のようなフォルダが作成されます。

- node_modules
- .bin
- @protobufjs
- @types
- long
- protobufjs

package.jsonファイルのdependenciesにgameanvil_connectorが追加されていることも確認できます。

```
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

Internet Explorerなど、旧バージョンのブラウザをサポートするにはcore-js, regenerator-runtimeをインストールします。

```
npm i core-js regenerator-runtime
```

## コネクタの作成と基本設定

コネクタを使用するには、コネクタとProtocolManagerモジュールをインポートします(import)。

```
import { Connector, ProtocolManager} from 'GameAnvil/gameanvil';
```

ゲームで使用する[メッセージ](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##メッセージ)を登録し、コネクタオブジェクトを作成します。

```
export default class GameAnvilManager {
    private connector: Connector;
    constructor() {
        // プロトコル登録。
        ProtocolManager.RegisterProtocol(0, GameMessages);

        // コネクタ作成。
        this.connector = Connector.Create();
    }
}
```

サーバーとやりとりする[メッセージ](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##メッセージ)を処理するためUpdate()関数を定期的に呼び出す必要があります。Update()関数の呼び出し周期は自由に設定しても構いませんが、呼び出していない場合は、サーバーから[メッセージ](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##メッセージ)を受信しても、この通知を受け取れません。

```
setInterval(() => { this.connector.Update(); }, 10);
```

次のようなシングルトンクラスを作成して使用すると便利です。

```
/*
    Internet Explorerなどの古いブラウザでコネクタを使用するには
    core-js、regenerator-runtimeなどのプラグインを使用する必要がある。
*/
import 'core-js';
import 'regenerator-runtime';

import { Connector, ProtocolManager } from 'GameAnvil/gameanvil';

export default class GameAnvilManager {
    private static manager: GameAnvilManager;
    private connector: Connector;
    private constructor() {
        // プロトコル登録。
        ProtocolManager.RegisterProtocol(0, GameMessages);

        // コネクタ作成。
        this.connector = Connector.Create();

        // メッセージループ。10msごとに呼び出し。
        let updater = setInterval(() => { this.connector.Update(); }, 10);
    }

    static GetInstance() {
        if (this.manager == null) {
            this.manager = new GameAnvilManager();
        }
        return this.manager;
    }

    GetConnectionAgent() {
        return this.connector.GetConnectionAgent();
    }

    GetUserAgent(serviceName: string, subId: string) {
        return this.connector.GetUserAgent(serviceName, subId);
    }

    CreateUserAgent(serviceName: string, subId: string) {
        return this.connector.CreateUserAgent(serviceName, subId);
    }
}
```

## サーバー接続および認証

サーバー接続と認証は、ConnectionAgentを利用して行います。ConnectionAgentはGameAnvilサーバーのコネクションノードに関連する作業を担当します。接続(Connect())、認証(Authentication())など、基本セッション管理機能とチャンネルリストなどを提供し、サーバーの実装に基づいてチャンネル情報を提供したり、ユーザー定義[メッセージ](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##メッセージ)を送受信できます。ConnectionAgentは、コネクタが初期化されるときに自動的に作成され、Connector.GetConnectionAgent()関数を用いて取得できます。ConnectionAgentはコネクタが初期化される時に自動的に作成され、Connector.GetConnectionAgent()関数を利用して取得できます。

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// サーバー接続
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void;
//  ipAddress ：接続するサーバーアドレス。 
//  callback ：結果を受け取るコールバック。(オプション)
//    agent ： Connect()を呼び出したConnectionAgentオブジェクト。
//    resultCode ： Connect結果。
connection.Connect(
    this.edtIPAddress.string,
    (agent, resultCode) => {
        console.log("Connect : " + ResultCodeConnect[resultCode]);
        if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) {
            // 接続成功
        } else {
            // 接続失敗
        }
    }
);
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// 認証。 
// Authenticate(deviceId: string, accountId: string, password: string, payload?: Payload, callback?: (agent: ConnectionAgent, resultCode: ResultCodeAuth, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload) => void): void;
//  deviceId ：重複接続をチェックするために使用。
//  accountId ：ユーザーアカウント。 
//  password ：パスワード。
//  payload ：追加で必要なデータがある場合に使用。(オプション)
//  callback ：結果を受け取るコールバック。(オプション)
//   agent ： Authenticateを呼び出したConnectionAgentオブジェクト。
//   resultCode ： Authenticate結果
//   loginedUserInfoList ：このアカウントIdを利用してログイン中のユーザー情報。
//   message ：サーバーから送る追加メッセージ。
//   payload：サーバーコンテンツから送る追加データ。
connection.Authenticate(
    "deviceId",
    this.edtAccountId.string,
    this.edtPassword.string,
    null,
    (agent, resultCode, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload) => {
        if (ResultCodeAuth.AUTH_SUCCESS == resultCode) {
            // 認証成功
        } else {
            // 認証失敗
        }
    }
);
```

接続を終了する時はDisconnect()を呼び出します。

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// 接続終了
// Disconnect(reason: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string) => void, force: boolean, payload: Payload): void;
//   reason：接続終了理由。Disconnect()を呼び出すケースが複数ある場合、コールバックでそれぞれのケースを区別するために使用。
//   callback ：結果を受け取る。(オプション)
//     agent ： Disconnect()を呼び出したConnectionAgentオブジェクト。
//     resultCode ： Disconnect()の結果。
//     reason ：接続終了理由。
//     force ：サーバーで強制終了するかどうか。重複接続、Kick、AdminKickなど。
//     payload ：サーバーコンテンツから送る追加データ。
connection.Disconnect("click Disconnect");
```

Disconnect()の呼び出しの他に、ネットワーク環境が原因で接続終了する場合に対処するためにConnectionAgent.AddOnDisconnect()を利用して個別にコールバックを登録して使用します。

IConnectionListenerインタフェースを実装して登録すると、ConnectionAgentのすべての通知を受け取ることができます。

```
class ConnectionListener implements IConnectionListener{
    OnConnect?(connection: ConnectionAgent, resultCode: ResultCodeConnect): void { }
    OnAuthentication?(connection: ConnectionAgent, resultCode: ResultCodeAuth, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload): void { }
    OnChannelList?(connection: ConnectionAgent, resultCode: ResultCodeChannelList, channelIdList: Array<string>): void { }
    OnChannelInfo?(connection: ConnectionAgent, resultCode: ResultCodeChannelInfo, channelInfoList: Array<Payload>): void { }
    OnDisconnect?(connection: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload): void { }
    OnErrorCommand?(connection: ConnectionAgent, errorCode: ErrorCode, msgName: string): void { }
}


let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
connection.AddListener(new ConnectionListener());

// address      ：サーバーアドレス
connection.Connect(address);
// ConnectionListener.OnConnectでレスポンス


//  deviceId ：重複接続をチェックするために使用。
//  accountId ：ユーザーアカウント。 
//  password ：パスワード。
//  payload ：追加で必要なデータがある場合に使用。(オプション)
connection.Authenticate(deviceId, accountId, password, payload);
// ConnectionListener.OnAuthenticateでレスポンス
```

## ログインおよび基本機能使用

ログイン/ログアウトと基本的な機能は、UserAgentを介して利用できます。UserAgentは、GameAnvilサーバーのゲームのノードに関連するタスクを担当します。ログイン(Login())、ログアウト(Logout())とルーム管理などの基本機能を提供し、サーバーの実装に応じて、ユーザー定義機能をさらに提供することもできます。UserAgentを使用するには、Connector.CreateUserAgent()関数を用いて、新しいUserAgentを作成する必要があります。 ServiceIdとSubIdに区分されている複数のUserAgentを作成することができ、作成された各UserAgentは独立して使用できます。作成されたUserAgentはコネクタで内部的に管理され、Connector.GetUser()関数を用いて再利用できます。

```
// serviceId    ： UserAgentが使用するserviceId。
// subId        ：サービスごとにUserAgentを識別することができる固有のID。サーバーの実装に応じて使用しないこともある。
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
if (user == null) {
    user = GameAnvilManager.GetInstance().CreateUserAgent(this.ServiceName);
}
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
// Serviceにログインします。
// Login(userType: string, payload?: Payload, channelId?: string, callback?: (agent: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo) => void): void;
//   userType ：サーバーに登録されたUserType。
//   payload ：追加的で必要なデータがある場合に使用。(オプション)
//   channelId ：使用するチャンネル。(オプション。入力していない場合は、空白文字列として処理され、サーバーに空白の文字列のチャンネルが設定されている必要があります。)
//   callback ：結果を受け取るコールバック。(オプション)
//     agent ： Login()を呼び出したUserAgentオブジェクト。
//     resultCode ： Login()結果
// loginInfo：ログインしているユーザーの情報。
user.Login(this.UserType, null, this.editBoxChannel.string, (UserAgent, resultCode, loginInfo) => {
    if(resultCode == ResultCodeLogin.LOGIN_SUCCESS){
        // ログイン成功
    }else{
        // ログイン失敗
    }
});
```

IUserListenerインターフェイスを実装して登録すると、UserAgentのすべての通知を受け取ることもできます。

```
class UserListener : IUserListener{
    OnLogin?(user: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo): void { }
    OnLogout?(user: UserAgent, resultCode: ResultCodeLogout, force: boolean, payload: Payload): void { }
    OnMatchRoom?(user: UserAgent, resultCode: ResultCodeMatchRoom, roomId: string, payload: Payload): void { }
    OnCreateRoom?(user: UserAgent, resultCode: ResultCodeCreateRoom, roomId: string, payload: Payload): void { }
    OnJoinRoom?(user: UserAgent, resultCode: ResultCodeJoinRoom, roomId: string, payload: Payload): void { }
    OnNamedRoom?(user: UserAgent, resultCode: ResultCodeNamedRoom, roomId: string, payload: Payload): void { }
    OnLeaveRoom?(user: UserAgent, resultCode: ResultCodeLeaveRoom, force: boolean, roomId: string, payload: Payload): void { }
    OnMatchUserStart?(user: UserAgent, resultCode: ResultCodeMatchUserStart, payload: Payload): void { }
    OnMatchUserCancel?(user: UserAgent, resultCode: ResultCodeMatchUserCancel): void { }
    OnMatchUserDone?(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: string, payload: Payload): void { }
    OnMatchUserTimeout?(user: UserAgent): void { }
    OnMatchPartyStart?(user: UserAgent, resultCode: ResultCodeMatchPartyStart, payload: Payload): void { }
    OnMatchPartyCancel?(user: UserAgent, resultCode: ResultCodeMatchPartyCancel): void { }
    OnMoveChannel?(user: UserAgent, resultCode: ResultCodeMoveChannel, force: boolean, channelId: string, payload: Payload): void { }
    OnSnapshot?(user: UserAgent, resultCode: ResultCodeSnapshot, paload: Payload): void { }
    OnErrorCommand?(user: UserAgent, errorCode: ErrorCode, msgName: string): void { }
    OnNotice?(user: UserAgent, message: string): void { }
    OnAdminKickoutNoti?(user: UserAgent, message: string): void { }
}


let user = connector.GetUserAgent(serviceId, subId);
user.AddUserListener(new UserListener())


// payload     ：サーバーに伝達する追加の値。サーバーの実装に応じて使用しないこともある。
// channelId   ： UserAgentがログインするチャンネルID。サーバーの実装に応じて使用しないこともある。
user.Login(payload, channelId);
// UserListener.OnLoginでレスポンス


user.Logout();
// UserListener.OnLogoutでレスポンス


user.CreateRoom();
// UserListener.OnCreateRoomでレスポンス


user.MatchRoom();
// UserListener.OnMatchRoomでレスポンス


user.LeaveRoom();
// UserListener.OnLeaveRoomでレスポンス


...
```

## メッセージ

ConnectionAgent、UserAgentの基本機能に加え、Request()とSend()を利用してメッセージをサーバーに送信できます。メッセージを送信するには、メッセージを作成し、登録するプロセスが必要です。

### メッセージの作成

GameAnvilはデフォルトのメッセージプロトコルに[ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/overview)を使用しています。.protoファイルにメッセージを定義し、pbjsで実際のクラスソースコードを作成します。そして、GameAnvilで使用する追加コードを、作成されたコードに挿入します。こうして作成されたソースコードをプロジェクトに追加して使用できます。

追加コードを挿入するには`CodeInserter`をインストールする必要があります。`CodeInserter`は[こちら](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-CodeInserter.zip)からダウンロードできます。ダウンロードしたファイルを解凍し、プロジェクトルートの下(assetsフォルダの外に)にフォルダを作成して格納します。

![codeInserter](http://static.toastoven.net/prod_gameanvil/images/client-2-codeInserter.png)

`CodeInserter`は、acornを使用します。ターミナルで次のコードを入力してacornをインストールします。 

```
npm i acorn@5.5.3 --save-dev
```

package.jsonのdevDependenciesにacornが追加されたことを確認できます。

```
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
  },
  "devDependencies": {
    "acorn": "^5.5.3"
  }
}
```

メッセージを作成してみます。まず、assetsフォルダの下にprotocolsフォルダを作成し、次のようにmessages.protoファイルを作成します。

```
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

次にmessages.protoファイルをコンパイルするためのスクリプトをpackage.jsonに追加します。 

```
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

そして、ターミナルで次のコードを入力してメッセージを作成します。

```
npm run messages
```

次のように`message.js`、`message.d.ts`ファイルが作成されたことを確認できます。

![messages](http://static.toastoven.net/prod_gameanvil/images/client-2-messages.png)

### メッセージ登録

新たに作成したメッセージを使用するには、使用するメッセージをProtocolManagerにサーバーと同じ値であらかじめ登録する必要があります。登録していなかったり、サーバーと異なる場合、動作しなかったり、誤動作や例外が発生することがあります。

```
// サーバーと同じ値で登録する必要がある。
ProtocolManager.RegisterProtocol(0, message);
```

### メッセージ送信

RequestPb()でメッセージを送信すると、サーバーの応答を待ちます。サーバーの応答を待っている間、追加RequestPb()は、キューに格納され、サーバーの応答を処理した後、順次処理されます。サーバーのレスポンスを受け取って処理するには、リスナーを登録する必要があります。リスナーを登録していない場合は、サーバーのレスポンスを受けとっても通知を行わず、次のメッセージを処理することになります。指定された時間内に応答が来なければタイムアウトを発生させ、次のメッセージを処理します。タイムアウトは、OnError()リスナーにErrorCode.TIMEOUTが渡されます。

SendPb()でメッセージを送信すると、SendPb()を呼び出すとすぐにサーバーに送信され、別のレスポンスを待ちません。RequestPb()のレスポンスを待っている中、SendPb()を使用したメッセージは、すぐにサーバーに送信されます。

```
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
// レスポンスでMessages.SampleResponse.

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
// レスポンスでMessages.SampleResponse.

user.RequestPb<Messages.SampleResponse>(sampleRequest, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
```

### カスタムメッセージ

パケットクラスを利用して、ProtocolBuffer以外の任意のデータをバイトストリームとしてシリアライズして使用できます。パケットの詳細については、[こちら](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##Packet)を参照してください。

```
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

## パケット

サーバーと送受信するすべてのメッセージはパケットモジュールに載せて処理され、パケットモジュールが提供するインターフェイスを利用します。

### 作成

GameAnvilコネクタはGoogle Protocol Bufferをデフォルトでのプロトコルとして使用しています。 Google Protocol Bufferを利用するパケットの作成は次のとおりです。

```
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);

let sampleMessage2 = packet.GetPbMessage<Messages.SampleMessage>();
```

Google Protocol Bufferを使用しなくてもパケットを作成できます。

```
let enc = new TextEncoder();
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data"));

let dec = new TextDecoder("utf-8");
let bytes =  packet.GetBytes();
let string = dec.decode(bytes);
let uint8arr = JsonUtil.Serialize(obj); // object to UInt8Array;
let packet = Packet.CreateFromBuffer(reqMsgId, uint8arr);

let bytes = packet.GetBytes();
let obj = JsonUtil.Deserialize(bytes);
```

### 圧縮

パケットサイズが大きい場合、圧縮してデータ使用量を減らすことができます。

```
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);
packet.compress();

if (packet.IsCompress())
    packet.Decompress();

let responseMsg = packet.GetPbMessage<Messages.SampleMessage>();
```

## Payload

GameAnvilで提供される基本的なAPIを利用する際、追加のデータが必要な場合があります。そのため、基本的なAPIには、追加のデータを渡すことができるpayloadというパラメータが含まれています。このpayloadに必要なデータをパケットに入れてlist形式で保存できます。ここに追加データを入れてサーバーに送信したり、サーバーから送信されたメッセージを取り出すことができます。

```
let userInfo = Packet.CreateFromPbMsg(new Messages.UserInfo());
let roomInfo = Packet.CreateFromPbMsg(new Messages.RoomInfo());

let payload = Payload.CreateFromPackets([userInfo, roomInfo]);

let userInfoPacket: Packet = payload.GetPacket(Messages.UserInfo);
let roomInfo2: Messages.RoomInfo = payload.GetPBMessage(Messages.RoomInfo);
let userInfo = Packet.CreateFromPbMsg(new Messages.UserInfo());
let roomInfo = Packet.CreateFromPbMsg(new GameMessages.RoomInfo());

let payload = Payload.CreateDefault();
payload.add(userInfo);
payload.add(roomInfo);

let userInfoPacket: Packet = payload.GetPacket(Messages.UserInfo);
let roomInfo2: Messages.RoomInfo = payload.GetPBMessage(Messages.RoomInfo);
```

## GameAnvilコネクタ終了

ゲームプレイ終了前にConnector.CloseSoket()関数を呼び出して、接続を終了することを推奨します。接続を終了していない場合、サーバ側でクライアントの終了を認識できず、不要な動作を続けることがあります。
