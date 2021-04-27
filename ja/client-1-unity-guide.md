## Game > GameAnvil > クライアント開発ガイド > Unity開発ガイド

ガイド環境

- Unity 2018.4.1f1
- Visual Studio 2017
- GameAnvil Connector: 1.1.0

## GameAnvilコネクタのインストール

Unityプロジェクトにgameanvil-connector.unitypackageをインストールします。gameanvil-connector.unitypackageは[こちら](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector.unitypackage)からダウンロードできます。パッケージをインストールすると、GameAnvilフォルダの下にDLLファイルがインストールされます。DLLファイルはC#で作成しており、Andoid、iOS、PCなどのプラットフォームで使用できます。

![unitypackage](http://static.toastoven.net/prod_gameanvil/images/client-1-unitypackage.png)

## コネクタ作成

GameAnvilConnectorを使用するには、まずコネクタを作成する必要があります。基本設定とエージェント管理を担当し、内部動作に関連するログを見ることができるようにデリゲートを登録できます。

```
using GameAnvil;
Connector connector = new Connector();
```

サーバーと送受信するメッセージを処理するためにUpdate()関数を定期的に呼び出します。 MonoBehaviourのUpdate()で呼び出すのが最も簡単な方法で、必要に応じて他の方法で呼び出しても構いません。Update()関数の呼び出し周期は自由に設定しても構いませんが、呼び出さない場合は、サーバーからメッセージを受信しても、この通知を受け取れません。

```
connector.Update();
```

次のように、簡単にMonoBehaviourを継承してコネクタを管理するスクリプトを作成して使用できます。

```
using GameAnvil;

public class ConnectHandler : MonoBehaviour
{
    public static Connector connector = null;

    private void Awake()
    {
        if(connector == null){
            connector = new GameAnvil.Connector();
            // コネクタログ追加
            connector.Logger += (level, log) =>
            {
                Debug.Log(string.Format("Log[{0}]:{1}", level, log));
            };
            connector.LvNetLogger += (level, log) =>
            {
                Debug.Log(string.Format("Net[{0}]:{1}", level, log));
            };
        }
    }

    private void Update()
    {
        if(connector != null)
            connector.Update();
    }

    private void OnDestroy()
    {
        if (connector != null && connector.IsConnected())
        {
            connector.CloseSocket();
        }
    }
}
```

このコードは例で、必要に応じて適切に変更して使用してください。これでGameAnvilコネクタを使用する準備が完了しました。

## サーバー接続および認証

サーバー接続と認証は、ConnectionAgentを利用して行います。ConnectionAgentはGameAnvilサーバーのコネクションノードに関連する作業を担当します。接続(Connect())、認証(Authentication())など、基本セッション管理機能とチャンネルリストなどを提供し、サーバーの実装に基づいてチャンネル情報を提供したり、ユーザー定義メッセージを送受信できます。ConnectionAgentは、コネクタが初期化されるときに自動的に作成され、Connector.GetConnectionAgent()関数を用いて取得できます。接続および認証結果はIConnectionListenerを実装したリスナーを登録して確認できます。

```
class ConnectionListener : IConnectionListener{
    ...
}


ConnectionListener listener = new ConnectionListener();


GameAnvil.ConnectionAgent connectionAgent = connector.GetConnectionAgent();
connectionAgent.AddConnectionListener(listener);


// ip    : ip address
// port    : port
connectionAgent.Connect(ip, port);
// ConnectionListener.OnConnectでレスポンス


// deviceId    ：ユーザーの端末を識別することができる固有のID。サーバーの実装に応じて使用しないこともある。
// accountId   ：ユーザーアカウントを識別することができる固有のID。
// password    ：ユーザーアカウントのパスワード。サーバーの実装に応じて使用しないこともある。
connectionAgent.Authentication(deviceId, accountId, password);
// ConnectionListener.OnAuthenticationでレスポンス
```

## ログインおよび基本機能の使用

ログイン/ログアウトと基本的な機能は、UserAgentを介して利用できます。UserAgentは、GameAnvilサーバーのゲームのノードに関連するタスクを担当します。ログイン（Login（））、ログアウト（Logout（））とルーム管理などの基本機能を提供し、サーバーの実装に応じて、ユーザー定義機能をさらに提供することもできます。UserAgentを使用するには、Connector.CreateUserAgent（）関数を用いて、新しいUserAgentを作成する必要があります。 ServiceIdとSubIdに区分されている複数のUserAgentを作成することができ、作成された各UserAgentは独立して使用できます。作成されたUserAgentはConnectorで内部的に管理されConnector.GetUser（）関数を用いて再利用できます。ログイン/ログアウトや基本的な機能の結果は、IUserListenerを登録して確認できます。 UserAgentはConnector.CreateUserAgent（）関数を用いて作成できます。 ServiceIdとSubIdに区分されている複数のUserAgentを作成することができ、作成された各UserAgentは独立して使用できます。作成されたUserAgentは、Connector内部でキャッシュし、Connector.GetUserAgent（）関数を用いて再利用できます。

```
// serviceId    ： UserAgentが使用するserviceId. 
// subId        ：サービスごとにUserAgentを識別することができる固有ID。1以上の整数。
UserAgent userAgent = connector.CreateUserAgent(serviceId, subId);


userAgent = connector.GetUserAgent(serviceId, subId);
class UserListener : IUserListener{
    ...
}


UserListener listener = new UserListener();


UserAgent userAgent = connector.GetUserAgent(serviceId, subId);
userAgent.AddUserListener(listener)


// payload     ：サーバーに伝達する追加の値。サーバーの実装に応じて使用しないこともある。
// channelId   ： UserAgentがログインするチャンネルID。サーバーの実装に応じて使用しないこともある。
userAgent.Login(payload, channelId);
// UserListener.OnLoginでレスポンス


userAgent.Logout();
// UserListener.OnLogoutでレスポンス


userAgent.CreateRoom();
// UserListener.OnCreateRoomでレスポンス


userAgent.MatchRoom();
// UserListener.OnMatchRoomでレスポンス


userAgent.LeaveRoom();
// UserListener.OnLeaveRoomでレスポンス
...
```

## メッセージ

ConnectionAgent、UserAgentの基本機能に加え、Request（）とSend（）を利用してメッセージをサーバーに送信できます。メッセージを送信するには、メッセージを作成して登録するプロセスが必要です。

### メッセージの作成

GameAnvilは、デフォルトのメッセージプロトコルに[ProtocolBuffer]（https://developers.google.com/protocol-buffers/docs/overview）を使用します。.protoファイルにメッセージを定義し、protocコンパイラで実際のクラスのソースコードを作成します。作成されたソースコードをプロジェクトに追加して使用できます。 protocコンパイラはGameAnvil / protocフォルダにあります。protocの詳細説明は、[こちら]（https://developers.google.com/protocol-buffers/docs/proto3#generating）を参照してください。

次に、メッセージを作成してみましょう。まず、Assetsフォルダにprotocolsフォルダを作成し、次のようにmessages.protoファイルを作成します。

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

そして、Windowsコマンドプロンプト(cmd)を実行してprotocolsフォルダに移動した後、次のように入力します。

```
../GameAnvil/protoc/protoc --csharp_out=./ messages.proto
```

そうするとprotocolsフォルダにMessages.csファイルが作成されていることを確認できます。 

### メッセージ登録

新たに作成したメッセージを使用するには、使用するメッセージをProtocolManagerにサーバーと同じ値であらかじめ登録する必要があります。あらかじめ登録していなかったり、サーバーと異なる場合、動作しなかったり、誤動作や例外が発生することがあります。

```
// サーバーと同じ値で登録する必要がある。
ProtocolManager.getInstance().RegisterProtocol(0, Messages.MessagesReflection.Descriptor);
```

### メッセージ送信

Request（）でメッセージを送信すると、サーバーの応答を待ちます。サーバーの応答を待っている間、追加Request（）は、キューに格納され、サーバーの応答を処理した後、順次処理されます。サーバーのレスポンスを受け取って処理するには、リスナーを登録する必要があります。リスナーを登録していない場合は、サーバーのレスポンスを受けとっても通知を行わず、次のメッセージを処理することになります。指定された時間内に応答が来なければタイムアウトを発生させ、次のメッセージを処理します。タイムアウトは、OnError（）リスナーにErrorCode.TIMEOUTが渡されます。

Send（）でメッセージを送信すると、Send（）を呼び出すとすぐにサーバーに送信され、別のレスポンスを待ちません。Request（）のレスポンスを待っている中、Send（）を使用したメッセージは、すぐにサーバーに送信されます。

```
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
connection.AddListener((ConnectionAgent connection, Messages.SampleReceive msg)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
connection.Send(sampleSend);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
connection.Request(sampleRequest);
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
user.AddListener((UserAgent user, Messages.SampleReceive msg)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
user.Send(sampleSend);

Messages.SampleRequest SampleRequest = new Messages.SampleRequest();
user.Request(SampleRequest);
```

### カスタムメッセージ

パケットクラスを利用して、ProtocolBuffer以外の任意のデータをバイトストリームとしてシリアライズして使用できます。パケットの詳細については、[こちら]（https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-1-unity-guide/##Packet）を参照してください。

```
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
int reqMsgId = 1;
int resMsgId = 2;

connection.AddListener(resMsgId, (ConnectionAgent connection, Packet packet)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
Packet sampleSendPacket = new Packet(reqMsgId, sampleSend.ToByteArray())
connection.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
Packet sampleRequestPacket = new Packet(reqMsgId, sampleRequest.ToByteArray())
connection.Request(sampleRequestPacket);
connection.Request(sampleRequestPacket, (ConnectionAgent connection, Packet packet)=> { });
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
int reqMsgId = 1;
int resMsgId = 2;

user.AddListener(resMsgId, (UserAgent user, Packet packet)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
Packet sampleSendPacket = new Packet(reqMsgIndex, sampleSend.ToByteArray())
user.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
Packet sampleRequestPacket = new Packet(reqMsgIndex, sampleRequestPacket.ToByteArray())
user.Request(sampleRequestPacket);
user.Request(sampleRequestPacket, (UserAgent user, Packet packet)=> { });
```

## パケット

サーバーと送受信するすべてのメッセージはパケットモジュールに載せて処理され、パケットモジュールが提供するインターフェイスを利用します。

### 作成

GameAnvilコネクタはGoogle Protocol Bufferをデフォルトでのプロトコルとして使用します。Google Protocol Bufferを利用するパケットの作成方法は次のとおりです。

```
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
```

パケット 

```
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Pcket.CreateWithCustomMsg(customMsgId, requestMsg);

byte[] bytes =  packet.GetBytes();
string JsonString = Encoding.UTF8.GetString(bytes);
```

### 圧縮

パケットのサイズが大きい場合、圧縮してデータ使用量を減らすことができます。

```
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
packet.compress();

if (packet.isCompress())
    packet.decompress();

CustomMessage.SampleResponse responseMsg = packet.GetMessage<CustomMessage.SampleResponse>();
```

## Payload

GameAnvilで提供される基本的なAPIを利用する際、追加のデータが必要な場合があります。そのため、基本的なAPIには、追加のデータを渡すことができるpayloadというパラメータが含まれています。このpayloadに必要なデータをパケットに入れてリスト形式で保存できます。ここに追加データを入れてサーバーに送信したり、サーバーから送信されたメッセージを取得できます。

```
using GameAnvil;
Messages.UserInfo userInfo = new Messages.UserInfo();
Messages.RoomInfo roomInfo = new Messages.RoomInfo();

Payload payload = new Payload();
payload.add(new Packet(userInfo));
payload.add(new Packet(roomInfo));

Packet packet = payload.getPacket<Messages.UserInfo>();
Messages.RoomInfo roomInfo = payload.GetMessage<Messages.RoomInfo>()
```

## GameAnvil Connectorの終了

ゲームプレイ終了前にConnector.CloseSoket（）関数を呼び出して、接続を終了することを推奨します。接続を終了していない場合、サーバ側でクライアントの終了を認識できず、不要な動作を続けることがあります。
