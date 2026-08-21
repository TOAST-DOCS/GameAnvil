<!-- pre-align:aligned sig=53c95f95568b -->

<a id="game-gameanvil-cocoscreator-development-guide-packet"></a>
## Game > GameAnvil > CocosCreator Development Guide > Packet { #game-gameanvil-cocoscreator-development-guide-packet }

<a id="packet"></a>
## Packet { #packet }

All messages sent and received by the server are uploaded and processed in a packet.

<a id="create"></a>
### Create { #create }

The following are the creation methods using Protocol Buffer:

```typescript
const message: IMessage;

const packet: Packet = PacketFactory.makePacket(message);
```

Other Uint8Array format packet creation methods are as follows:
```typescript
const data: Unit8Array;

const packet: Packet = PacketFactory.makeCustomPacket(1, data);
```

<a id="compress"></a>
### Compress { #compress }

If the packet size is large, you can compress it to reduce data usage.

```typescript
const packet: Packet = PacketFactory.makePacket(message, PacketOption.compress);
```

<a id="payload"></a>
### Payload { #payload }

Additional data may be required when using the default API provided by GameAnvil. To this end, basic APIs include a parameter called payload that can release additional data. You can store the data required for this payload in list format in a packet. You can put additional data here to send to the server or extract messages sent from the server.

```typescript
const message: IMessage;
const packet: Packet = PacketFactory.makePacket(message);

const payload: Payload = new Payload();

payload.addPacket(message);
payload.getPacket(IMessage.descriptor);
```