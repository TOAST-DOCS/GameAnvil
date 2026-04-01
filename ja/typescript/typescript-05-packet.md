## Game > GameAnvil > CocosCreator 開発ガイド > パケット

## パケット

サーバーとやり取りする全てのメッセージは、パケットに載せて処理されます。

### 生成

Protocol Bufferを利用した生成方法は以下のとおりです。

```typescript
const message: IMessage;

const packet: Packet = PacketFactory.makePacket(message);
```

その他、Uint8Array形式のパケット生成方式は以下のとおりです。
```typescript
const data: Unit8Array;

const packet: Packet = PacketFactory.makeCustomPacket(1, data);
```

### 圧縮

パケットサイズが大きい場合、圧縮してデータ使用量を減らすことができます。

```typescript
const packet: Packet = PacketFactory.makePacket(message, PacketOption.compress);
```

### ペイロード

GameAnvilが提供する基本APIを利用する際、追加のデータが必要になる場合があります。このために基本APIには、追加データを渡すことができるpayloadというパラメータが含まれています。このpayloadに必要なデータをパケットに込めてlist形式で保存できます。ここに追加データを入れてサーバーへ送ったり、サーバーから送られたメッセージを取り出すことができます。

```typescript
const message: IMessage;
const packet: Packet = PacketFactory.makePacket(message);

const payload: Payload = new Payload();

payload.addPacket(message);
payload.getPacket(IMessage.descriptor);
```
