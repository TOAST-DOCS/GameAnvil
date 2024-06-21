## Game > GameAnvil > Unity深層開発ガイド > パケット

## パケット

サーバーと送受信する全てのメッセージはパケットモジュールで処理され、パケットモジュールが提供するインターフェイスを利用します。

### 作成

コネクタはGoogle Protocol Buffersを基本プロトコルとして使用します。Google Protocol Buffersを利用するパケットの作成は次のとおりです。

```c#
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
```

    Google Protocol Buffersを使わなくてもパケットを作成できます。

```c#
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Pcket.CreateWithCustomMsg(customMsgId, requestMsg);

byte[] bytes =  packet.GetBytes();
string JsonString = Encoding.UTF8.GetString(bytes);
```

パケットの最大サイズは64Kbytesに制限されています。パケットサイズが64Kbytesを超える場合、圧縮によってサイズ制限を回避できます。
ペイロードも内部的にはパケットとして処理されるため、同様に64Kbytesを超えることはできません。

### 圧縮

パケットサイズが大きい場合、圧縮してデータ使用量を減らすことができます。 

```c#
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
packet.compress();

if (packet.isCompress())
    packet.decompress();

CustomMessage.SampleResponse responseMsg = packet.GetMessage<CustomMessage.SampleResponse>();
```

### Payload

GameAnvilが提供する基本APIを利用する時、追加的なデータが必要な場合があります。このため、基本APIには追加データを渡すことができるペイロードというパラメータが含まれています。このペイロードに必要なデータをパケットに入れてリスト形式で保存できます。ここに追加データを入れてサーバーに送ったり、サーバーから送ったメッセージを取り出すことができます。

```c#
using GameAnvil;
Messages.UserInfo userInfo = new Messages.UserInfo();
Messages.RoomInfo roomInfo = new Messages.RoomInfo();

Payload payload = new Payload();
payload.add(new Packet(userInfo));
payload.add(new Packet(roomInfo));

Packet packet = payload.getPacket<Messages.UserInfo>();
Messages.RoomInfo roomInfo = payload.GetMessage<Messages.RoomInfo>()
```
