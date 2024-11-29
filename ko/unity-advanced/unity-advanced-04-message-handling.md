## Game > GameAnvil > Unity 심화 개발 가이드 > 메시지 핸들링

## 메시지 핸들링

GameAvnilConnector, GameAvilUser의 기본 기능 외에도 사용자가 정의한 메시지를 서버로 전송할 수 있습니다.

### 메시지 생성 및 등록
메시지를 생성하고 등록하는 방법은 GameAnvilManager를 사용하는 경우와 동일 합니다. [Unity 기초 개발 가이드 > 메시지 핸들링](../unity-basic/unity-basic-06-message-handling)에서 소개한 설명을 참고하세요. 

### 메시지 전송

#### Request

GameAvnilConnector에서는 Request(), GameAvilUser에서는 RequestUser()를 호출하여 메시지를 전송하고 응답을 받을 수 있습니다. 

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

Request\<TResponse\>(), RequestUser\<TResponse\>()은 다음과 같이 1개의 타입 매개변수와 1개의 매개변수를 가지고 있습니다.

| 타입       | 이름           | 설명             |
|----------|--------------|----------------|
| 타입 매개변수  | TResponse | 응답으로 받을 메시지 타입 |
| IMessage | message      | 서버로 보낼 메시지     |

응답으로 ErrorResult<ResultCode, TResponse>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. RequestUser() 가 성공하면 ErrorCode 필드의 값이 ResultCode.SUCCESS 가 되며, 아닌 경우 메시지 전송이 실패한 것입니다. Data 필드를 통해 응답 메시지를 얻을 수 있습니다.

ResultCode의 상세 내용은 다음과 같습니다.

| 이름                | 값  | 설명                                         |
|-------------------|----|--------------------------------------------|
| PARSE_ERROR       | -2 | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT           | -1 | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR      | 1  | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL  | 2  | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| HANDLER_NOT_EXIST | 10 | 실패. 서버에 핸들러 없음.                            |
| HANDLER_ERROR     | 11 | 실패. 서버의 핸들러에서 예외 발생.                       |
| SUCCESS           | 0  | 성공                                         |

#### SendUser

GameAvnilConnector에서는 Send(), GameAvilUser에서는 SendUser()를 호출여 서버로 메시지를 전송하며 별도의 응답을 기다리지는 않습니다.

```c#
public async void SendMessage()
{
    try
    {
        connector.Send(new Protocol.SampleSend());
    } catch (Exception e)
    {
        // 예외
    }
}
```

Send(), SendUser()는 다음과 같이 1개의 매개변수를 가지고 있습니다.

| 타입       | 이름      | 설명         |
|----------|---------|------------|
| IMessage | message | 서버로 보낼 메시지 |

#### MessageCallback

Send(), SendUser() 로 보내는 메시지와 상관없이 서버에서 보내는 메시지를 수신하기 위해서는 SetMessageCallback\<TProtoBuffer\>() 을 이용해 콜백을 등록할 수 있습니다. 등록된 콜백을 해제할 때는 RemoveMessageCallback\<TProtoBuffer\>()을 이용하면 됩니다.

```c#
public async void MessageCallback()
{
    connector.SetMessageCallback((GameAnvilConnector connector, ResultCode resultCode, Protocol.SampleReceive receive) =>
    {
        return Task.CompletedTask;
    });

    connector.RemoveMessageCallback<Protocol.SampleReceive>();
}
```

SetMessageCallback\<TProtoBuffer\>()은 다음과 같이 1개의 타입 매개변수와 1개의 매개변수를 가지고 있습니다.

| 타입                                                            | 이름           | 설명                         |
|---------------------------------------------------------------|--------------|----------------------------|
| 타입 매개변수                                                       | TProtoBuffer | 서버로부터 받을 메시지 타입            |
| Func<GameAnvilUserController, ResultCode, TProtoBuffer, Task> | callback     | 서버에서 메시지를 보냈을 때 호출될 콜백 메소드 |

RemoveMessageCallback\<TProtoBuffer\>()은 다음과 같이 1개의 타입 매개변수를 가지고 있습니다.

| 타입      | 이름           | 설명                |
|---------|--------------|-------------------|
| 타입 매개변수 | TProtoBuffer | 등록한 콜백이 받을 메시지 타입 |
