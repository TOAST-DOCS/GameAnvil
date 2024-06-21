## Game > GameAnvil > Unity深層開発ガイド > メッセージハンドリング

## メッセージハンドリング

ConnectionAgent、UserAgentの基本機能の他にRequest()とSend()を利用してメッセージをサーバーに送信できます。

### メッセージの送信

Request()でメッセージを送信すると、サーバーのレスポンスを待ちます。サーバーのレスポンスを受け取って処理する方法には、まず、[Unity基礎開発ガイド > メッセージハンドリング](../unity-basic/unity-basic-06-message-handling.md)で紹介したようにコールバックパラメータを渡す方法があります。

もう1つの方法としては、リスナーを登録する方法があります。どちらの方法も適用しない場合、サーバーのレスポンスを受け取っても、通知なしに次のメッセージを処理することになります。

リスナーを登録してRequest()のレスポンスを受ける例を見てみましょう。

[Unity基礎開発ガイド > メッセージハンドリング](../unity-basic/unity-basic-06-message-handling.md)で説明しなかったConnectionAgentを介したRequest()、Send()の使用例を紹介します。

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
// ConnectionAgentで伝達されるサーバーの通知を受けるリスナーを登録
connection.AddListener((ConnectionAgent connection, Messages.SampleReceive msg)=> { });

// ConnectionAgent Send
Messages.SampleSend sampleSend= new Messages.SampleSend(); 
connection.Send(sampleSend);

// ConnectionAgent Request
Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
connection.Request(sampleRequest, (ConnectionAgent connection, Packet packet)=> { });
```

### カスタムパケット

Packetクラスを利用して、ProtocolBuffer以外の任意のデータをバイトストリームとしてシリアル化して使うことができます。パケットに関する詳細は[Unity詳細開発ガイド > パケット](unity-advanced-05-packet.md)を参照してください。

[Unity基礎開発ガイド > メッセージハンドリング](../unity-basic/unity-basic-06-message-handling.md)で説明しなかったConnectionAgentを介したRequest()、Send()の使用例を紹介します。

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
int reqMsgId = 1;
int resMsgId = 2;

connection.AddListener(resMsgId, (ConnectionAgent connection, Packet packet)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
// パケットクラス利用
Packet sampleSendPacket = new Packet(reqMsgId, sampleSend.ToByteArray())
connection.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
// パケットクラス利用
Packet sampleRequestPacket = new Packet(reqMsgId, sampleRequest.ToByteArray())
connection.Request(sampleRequestPacket, (ConnectionAgent connection, Packet packet)=> { });
```
