## Game > GameAnvil > Unity基本開発ガイド > メッセージハンドリング

## メッセージハンドリング

UserAgentの基本機能以外にRequest()とSend()を利用してメッセージをサーバーに送信できます。メッセージを送信するためにはメッセージを作成して登録する過程が必要です。

### メッセージの作成

GameAnvilは基本メッセージプロトコルとして[ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/proto3)を使います。.protoファイルにメッセージを定義し、protocコンパイラで実際のクラスのソースコードを作成します。作成されたソースコードをプロジェクトに追加して使うことができます。 protocコンパイラはGameAnvil/protocフォルダで見つけることができます。 protocの詳しい説明は[こちら](https://developers.google.com/protocol-buffers/docs/proto3#generating)を参照してください。

メッセージを作るため、Assetsフォルダの下にprotocolsフォルダを作成し、次のようにmessages.protoファイルを作成します。

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

そして、Windowsコマンドプロンプト(cmd)を実行してprotocolsフォルダに移動して下記のように入力します。

```
../GameAnvil/protoc/protoc --csharp_out=./ messages.proto
```

すると、protocolsフォルダにMessages.csファイルが作成されたことを確認できます。

### メッセージの登録

新しく作成したメッセージを使うためには、使うメッセージをProtocolManagerに事前に登録する必要があります。事前に登録しないと、動作しないか、誤動作したり、例外が発生する可能性があります。

```c#
ProtocolManager.getInstance().RegisterProtocol(Messages.MessagesReflection.Descriptor);
```

### メッセージの送信

Request()でメッセージを送信すると、サーバーのレスポンスを待ちます。サーバーのレスポンスを待っている間、追加のRequest()はキューに保存され、サーバーのレスポンスを処理した後、順次処理することになります。サーバーのレスポンスを受け取って処理するため、コールバック引数を渡す必要があります。

```c#
/// <summary>
/// ユーザーエージェントを使ってプロトバフメッセージを送信
/// </summary>
/// <typeparam name="T">プロトバフタイプのメッセージ</typeparam>
/// <param name="agent">送信するユーザーエージェント</param>
/// <param name="message">送信するプロトバフメッセージ</param>
/// <param name="action">レスポンス処理する動作</param>
static public void Request<T>(User.UserAgent agent, IMessage message, Action<User.UserAgent, T> action) where T : IMessage;

/// <summary>
/// ユーザーエージェントを使ってパケットを送信
/// </summary>
/// <param name="agent">送信するユーザーエージェント</param>
/// <param name="packet">送信するパケット</param>
/// <param name="action">レスポンス処理する動作</param>
static public void Request(User.UserAgent agent, Packet packet, Action<User.UserAgent, Packet> action);
```

Requestレスポンスを受けるためにリスナーを登録する方法もありますが、これは[Unity深層開発ガイド > メッセージハンドリング](../unity-advanced/unity-advanced-04-message-handling.md)で確認できます。

指定された時間内にレスポンスが来ない場合、タイムアウトを発生させて次のメッセージを処理します。タイムアウトはUserAgent.OnErrorCommandListenersリスナーとUserAgent.OnErrorCustomCommandListenersリスナーにErrorCode.TIMEOUTとして渡されます。

Send()でメッセージを送信すると、Send()の呼び出しと同時にサーバーに送信され、レスポンスを待たずに送信されます。 Request()のレスポンスを待っている間もSend()を使ったメッセージはすぐにサーバーに送信されます。

```c#
Connector connector = new Connector();
UserAgent user = GameAnvilConnector.getUserAgent();
// UserAgentに伝達されるサーバーの通知を受けるリスナーを登録
user.AddListener((UserAgent user, Messages.SampleReceive msg)=> { }); 

// UserAgent Send
Messages.SampleSend sampleSend = new Messages.SampleSend(); 
user.Send(sampleSend);

// UserAgent Request
Messages.SampleRequest SampleRequest = new Messages.SampleRequest();
user.Request(SampleRequest, (UserAgent user, Messages.SampleResponse res) => { }); // コールバック引数の伝達
```

### カスタムパケット

パケットクラスを利用してProtocolBuffer以外の任意のデータをバイトストリームでシリアル化して使用できます。パケットの詳細は[Unity深層開発ガイド > パケット](../unity-advanced/unity-advanced-05-packet.md)を参照してください。

```c#
Connector connector = new Connector();
UserAgent user = GameAnvilConnector.getUserAgent();
int reqMsgId = 1;
int resMsgId = 2;

user.AddListener(resMsgId, (UserAgent user, Packet packet)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
// パケットクラス利用
Packet sampleSendPacket = new Packet(reqMsgIndex, sampleSend.ToByteArray())
user.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
// パケットクラス利用
Packet sampleRequestPacket = new Packet(reqMsgIndex, sampleRequestPacket.ToByteArray())
user.Request(sampleRequestPacket, (UserAgent user, Packet packet)=> { });
```
