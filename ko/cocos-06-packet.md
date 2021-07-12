## Game > GameAnvil > CocosCreator 개발 가이드 > 패킷

## 패킷

서버와 주고받는 모든 메시지는 패킷 모듈에 실려서 처리되며 패킷 모듈이 제공하는 인터페이스를 이용하게 됩니다.

### 생성

GameAnvil 커넥터는 Google Protocol Buffers를 기본 프로토콜로 사용하고 있습니다 . Google Protocol Buffers를 이용하는 패킷생성은 다음과 같습니다.

```typescript
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);

let sampleMessage2 = packet.GetPbMessage<Messages.SampleMessage>();
```

Google Protocol Buffers를 사용하지 않고도 패킷을 생성할 수 있습니다 .

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

### 압축

패킷 크기가 클 경우 압축하여 데이터 사용량을 줄일 수 있습니다 .  

```typescript
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);
packet.compress();

if (packet.IsCompress())
    packet.Decompress();

let responseMsg = packet.GetPbMessage<Messages.SampleMessage>();
```

### Payload

GameAnvil에서 제공하는 기본 API를 이용할 때 추가적인 데이터가 필요할 수 있습니다. 이를 위해 기본 API들에는 추가 데이터를 넘겨줄 수 있는 payload라는 매개변수가 포함되어 있습니다. 이 payload에 필요한 데이터를 패킷에 담아 list 형식으로 저장할 수 있습니다. 여기에 추가 데이터를 넣어 서버로 보내거나, 서버에서 보낸 메시지를 꺼낼 수 있습니다.  

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

