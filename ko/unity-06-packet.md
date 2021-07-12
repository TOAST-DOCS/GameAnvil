## 패킷

서버와 주고받는 모든 메시지는 패킷 모듈에 실려서 처리되며 패킷 모듈이 제공하는 인터페이스를 이용합니다.

### 생성

GameAnvil 커넥터는 Google Protocol Buffer를 기본 프로토콜로 사용합니다. Google Protocol Buffer를 이용하는 패킷 생성은 다음과 같습니다.

```c#
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
```

패킷 

```c#
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Pcket.CreateWithCustomMsg(customMsgId, requestMsg);

byte[] bytes =  packet.GetBytes();
string JsonString = Encoding.UTF8.GetString(bytes);
```

### 압축

패킷 크기가 크면 압축하여 데이터 사용량을 줄일 수 있습니다.  

```c#
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
packet.compress();

if (packet.isCompress())
    packet.decompress();

CustomMessage.SampleResponse responseMsg = packet.GetMessage<CustomMessage.SampleResponse>();
```

### Payload

GameAnvil에서 제공하는 기본 API를 이용할 때 추가적인 데이터가 필요할 수 있습니다. 이를 위해 기본 API들에는 추가 데이터를 넘겨줄 수 있는 payload라는 매개변수가 포함되어 있습니다. 이 payload에 필요한 데이터를 패킷에 담아 목록 형식으로 저장할 수 있습니다. 여기에 추가 데이터를 넣어 서버로 보내거나, 서버에서 보낸 메시지를 꺼낼 수 있습니다. 

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

