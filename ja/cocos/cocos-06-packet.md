## Game > GameAnvil > CocosCreator開発ガイド > パケット

## パケット

サーバーと送受信する全てのメッセージはパケットモジュールに載せて処理され、パケットモジュールが提供するインターフェイスを利用します。

### 作成

GameAnvilコネクタはGoogle Protocol Buffersを基本プロトコルとして使用しています。Google Protocol Buffersを利用するパケット作成は次のとおりです。

```typescript
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);

let sampleMessage2 = packet.GetPbMessage<Messages.SampleMessage>();
```

Google Protocol Buffersを使わずにパケットを生成できます。

```typescript
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

```typescript
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);
packet.compress();

if (packet.IsCompress())
    packet.Decompress();

let responseMsg = packet.GetPbMessage<Messages.SampleMessage>();
```

### Payload

GameAnvilが提供する基本APIを利用する時、追加的なデータが必要な場合があります。このため、基本APIには追加データを渡すことができるpayloadというパラメータが含まれています。このpayloadに必要なデータをパケットに入れてlist形式で保存できます。ここに追加データを入れてサーバーへ送ったり、サーバーから送ったメッセージを取り出すことができます。 

```typescript
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
