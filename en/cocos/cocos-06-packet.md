## Game > GameAnvil > CocosCreator Development Guide > Packet

## Packet

All the messages transferred between server are processed in a packet module and use the interface provided by packet module.

### Create

The GameAnvil connector uses Google Protocol Buffer as the default protocol. Creating a packet using Google Protocol Buffer is as below:

```typescript
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);

let sampleMessage2 = packet.GetPbMessage<Messages.SampleMessage>();
```

A packet can be created without using Google Protocol Buffers.

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

### Compression

If the size of a packet is too large, it can be compressed to reduce data usage.  

```typescript
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);
packet.compress();

if (packet.IsCompress())
    packet.Decompress();

let responseMsg = packet.GetPbMessage<Messages.SampleMessage>();
```

### Payload

When using the default API provided by GameAnvil, you may need additional data. For this, the default API includes the payload parameter to deliver additional data. The data required for the payload can be stored in a packet in a list form. You can add more data here and send it to the server or extract the message sent to the server.  

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

