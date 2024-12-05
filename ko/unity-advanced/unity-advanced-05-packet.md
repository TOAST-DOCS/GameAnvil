## Game > GameAnvil > Unity 심화 개발 가이드 > 패킷

## 패킷

GameAnvil은 기본 메시지 프로토콜로 ProtocolBuffer 메시지를 사용합니다. 그리고 이 메시지들은 패킷에 담겨서 처리됩니다. 대부분의 경우 GameAnvilConnector를 이용할 때 ProtocolBuffer 메시지만 사용해도 상관 없지만 상황에 따라서는 Packet을 이용해야 하는 경우도 있습니다. 

### 생성
다음과 같이 ProtocolBuffer 메시지를 이용해 패킷을 생성할 수 있습니다. 
```c#
Packet packet = Packet.MakePacket(new Protocol.SampleRequest());
```

ProtocolBuffer 메시지를 이용하지 않고도 패킷을 생성할 수 있습니다.
```c#
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Packet.MakeCustomPacket(customMsgId, requestMsg);

ByteString bytes = packet.ToByteString();
string JsonString = bytes.ToStringUtf8();
```
### 압축

서버로 전송할 수 있는 패킷의 최대 크기는 64Kbytes로 제한되어 있습니다. 패킷 크기가 64Kbytes를 넘을 경우 압축을 통해 크기 제한을 회피할 수 있습니다.
페이로드도 내부적으로는 패킷으로 처리되기 때문에 마찬가지로 64Kbytes를 넘을 수 없습니다.

```c#
Packet packet = Packet.MakePacket(new Protocol.SampleRequest(), PacketOption.Compress);
```

### 전송
이렇게 생성한 패킷은 ProtocolBuffer 메시지를 전송하는것와 같은 방법으로 서버로 전송할 수 있습니다. 
```c#
public async void RequestPacket()
{
    try
    {
        Packet packet = Packet.MakePacket(new Protocol.SampleRequest());
        ErrorResult<ResultCode, Protocol.SampleResponse> result = await connector.Request<Protocol.SampleResponse>(packet);
        if (result.ErrorCode == ResultCode.SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}
```

### Payload

GameAnvil에서 제공하는 기본 API를 이용할 때 추가적인 데이터가 필요할 수 있습니다. 이를 위해 기본 API들에는 추가 데이터를 넘겨줄 수 있는 페이로드라는 매개변수가 포함되어 있습니다. 이 페이로드에 추가로 필요한 데이터를 추가할 수 있으며, 이렇게 추가한 데이터를 서버로 보내거나, 서버로부터 받아 이용할 수 있습니다. 

```c#
Payload payload = new Payload(new Protocol.SampleRequest());
payload.Add(new Protocol.SampleSend());

Packet packet = payload.GetPacket(Protocol.SampleReceive.Descriptor);
Protocol.EchoRecv echoRecv = payload.GetProtoBuffer<Protocol.SampleReceive>();
```
