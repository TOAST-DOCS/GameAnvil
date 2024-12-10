## Game > GameAnvil > CocosCreator 개발 가이드 > 패킷

### 패킷

서버와 주고 받는 모든 메시지는 패킷에 실려서 처리됩니다.

#### 생성

Protocol Buffer를 이용한 생성 방법은 아래와 같습니다.

```typescript
const message: IMessage;

const packet: Packet = PacketFactory.makePacket(message);
```

그 외 Uint8Array 형식의 패킷 생성 방식은 아래와 같습니다.
```typescript
const data: Unit8Array;

const packet: Packet = PacketFactory.makeCustomPacket(1, data);
```

#### 압축

패킷 크기가 클 경우 압축하여 데이터 사용량을 줄일 수 있습니다.

```typescript
const packet: Packet = PacketFactory.makePacket(message, PacketOption.compress);
```

### 패이로드

GameAnvil에서 제공하는 기본 API를 이용할 때 추가적인 데이터가 필요할 수 있습니다. 이를 위해 기본 API들에는 추가 데이터를 넘겨줄 수 있는 payload라는 매개변수가 포함되어 있습니다. 이 payload에 필요한 데이터를 패킷에 담아 list 형식으로 저장할 수 있습니다. 여기에 추가 데이터를 넣어 서버로 보내거나, 서버에서 보낸 메시지를 꺼낼 수 있습니다.

```typescript
const message: IMessage;
const packet: Packet = PacketFactory.makePacket(message);

const payload: Payload = new Payload();

payload.addPacket(message);
payload.getPacket(IMessage.descriptor);
```