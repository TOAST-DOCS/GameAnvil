## Game > GameAnvil > Unity 기초 개발 가이드 > 메시지 핸들링

## 메시지 핸들링

GameAnvilUserController의 RequestUser()와 SendUser() 메소드를 이용하여 사용자가 정의한 메시지를 서버로 전송할 수 있습니다. 메시지를 전송하기 위해서는 메시지를 생성하고 등록하는 과정이 필요합니다.

### 메시지 생성

GameAnvil은 기본 메시지 프로토콜로 [ProtocolBuffers](https://developers.google.com/protocol-buffers/docs/proto3)를 사용합니다. .proto 파일에 메시지를 정의하고, protoc 컴파일러로 실제 클래스 소스 코드를 생성하게 됩니다. 생성된 소스 코드를 프로젝트에 추가하여 사용할 수 있습니다. protoc에 대한 자세한 설명은 [여기](https://developers.google.com/protocol-buffers/docs/proto3#generating)를 참고하십시오. 

이제 메시지를 만들기 위해 Assets 폴더 아래에 protocols 폴더를 생성하고 다음과 같이 messages.proto 파일을 생성합니다.

```protobuf
// messages.proto
syntax = "proto3";

package Messages;

message SampleRequest
{
  string msg = 1;
}

message SampleResponse
{
  repeated string msgs = 1;
}

message SampleSend
{
  string msg = 1;
}

message SampleReceive
{
  repeated string msgs = 1;
}
```

그리고 윈도우즈 명령 프롬프트(cmd)에서 protoc를 이용해 메시지를 생성합니다.

```
/protoc --csharp_out=./ messages.proto
```

### 메시지 등록

새로 생성한 메시지를 사용하려면 사용할 메시지를 ProtocolManager에 미리 등록해야 합니다. 미리 등록하지 않으면, 동작하지 않거나 오동작하거나 예외가 발생할 수 있습니다.

```c#
GameAnvilProtocolManager.RegisterProtocol(Messages.MessagesReflection.Descriptor);
```

### 메시지 전송

#### RequestUser

RequestUser()로 메시지를 전송하고 응답을 받을 수 있습니다.

```c#
public async void ManagerRequest()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        var (resultCode, response) = await userController.RequestUser<Protocol.SampleResponse>(new Protocol.SampleRequest());
        if (resultCode == ResultCode.SUCCESS)
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

RequestUser\<TProtoBuffer\>()은 다음과 같이 1개의 타입 매개변수와 1개의 매개변수를 가지고 있습니다.

| 타입       | 이름           | 설명             |
|----------|--------------|----------------|
| 타입 매개변수  | TProtoBuffer | 응답으로 받을 메시지 타입 |
| IMessage | message      | 서버로 보낼 메시지     |

응답으로 ErrorResult<ResultCode, TProtoBuffer>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. RequestUser() 가 성공하면 ErrorCode 필드의 값이 ResultCode.SUCCESS 가 되며, 아닌 경우 메시지 전송이 실패한 것입니다. Data 필드를 통해 응답 메시지를 얻을 수 있습니다.

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

SendUser()로 메시지를 전송하면 SendUser()의 호출 즉시 서버로 전송되며 별도의 응답을 기다리지 않습니다.

```c#
public async void ManagerSend()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        userController.SendUser(new Protocol.SampleSend());
    } catch (Exception e)
    {
        // 예외
    }
}
```

SendUser()은 다음과 같이 1개의 매개변수를 가지고 있습니다.

| 타입       | 이름      | 설명         |
|----------|---------|------------|
| IMessage | message | 서버로 보낼 메시지 |

#### MessageCallback

SendUser() 로 보내는 메시지와 상관없이 서버에서 보내는 메시지를 수신하기 위해서는 SetMessageCallback\<TProtoBuffer\>() 을 이용해 콜백을 등록할 수 있습니다. 등록된 콜백을 해제할 때는 RemoveMessageCallback\<TProtoBuffer\>()을 이용하면 됩니다.

```c#
public async void ManagerMessageCallback()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    userController.SetMessageCallback((GameAnvilUserController user, ResultCode resultCode, Protocol.SampleReceive receive) =>
    {
        return Task.CompletedTask;
    });
    
    userController.RemoveMessageCallback<Protocol.SampleReceive>();
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

