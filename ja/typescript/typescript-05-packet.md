<!-- pre-align:aligned sig=53c95f95568b -->

<a id="game-gameanvil-cocoscreator-development-guide-packet"></a>
## Game > GameAnvil > CocosCreator 開発ガイド > パケット { #game-gameanvil-cocoscreator-development-guide-packet }

<a id="packet"></a>
## パケット { #packet }

サーバーとやり取りする全てのメッセージは、パケットに載せて処理されます。

<a id="create"></a>
### 生成 { #create }

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

<a id="compress"></a>
### 圧縮 { #compress }

パケットサイズが大きい場合、圧縮してデータ使用量を減らすことができます。

```typescript
const packet: Packet = PacketFactory.makePacket(message, PacketOption.compress);
```

<a id="payload"></a>
### ペイロード { #payload }

GameAnvilが提供する基本APIを利用する際、追加のデータが必要になる場合があります。このために基本APIには、追加データを渡すことができるpayloadというパラメータが含まれています。このpayloadに必要なデータをパケットに込めてlist形式で保存できます。ここに追加データを入れてサーバーへ送ったり、サーバーから送られたメッセージを取り出すことができます。

```typescript
const message: IMessage;
const packet: Packet = PacketFactory.makePacket(message);

const payload: Payload = new Payload();

payload.addPacket(message);
payload.getPacket(IMessage.descriptor);
```
