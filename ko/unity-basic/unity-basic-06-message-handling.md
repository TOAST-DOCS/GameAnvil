## Game > GameAnvil > Unity 기초 개발 가이드 > 메시지 핸들링

## 메시지 핸들링

UserAgent의 기본 기능 외에 Request()와 Send()를 이용하여 메시지를 서버로 전송할 수 있습니다. 메시지를 전송하기 위해서는 메시지를 생성하고 등록하는 과정이 필요합니다.

### 메시지 생성

GameAnvil은 기본 메시지 프로토콜로 [ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/proto3)를 사용합니다. .proto 파일에 메시지를 정의하고, protoc 컴파일러로 실제 클래스 소스 코드를 생성하게 됩니다. 생성된 소스 코드를 프로젝트에 추가하여 사용할 수 있습니다. protoc 컴파일러는 GameAnvil/protoc 폴더에서 찾을 수 있습니다.  protoc에 대한 자세한 설명은 [여기](https://developers.google.com/protocol-buffers/docs/proto3#generating)를 참고하세요.

이제 메시지를 만들어 보도록 하겠습니다. 먼저 Assets 폴더 아래에 protocols 폴더를 생성하고 다음과 같이 messages.proto 파일을 생성합니다.

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

그리고 윈도우즈 명령 프롬프트(cmd)를 실행해 protocols 폴더로 이동한 다음 아래와 같이 입력합니다.

```
../GameAnvil/protoc/protoc --csharp_out=./ messages.proto
```

그러면 protocols 폴더에 Messages.cs 파일이 생성된 것을 확인할 수 있습니다. 

### 메시지 등록

새로 생성한 메시지를 사용하려면 사용할 메시지를 ProtocolManager에 서버와 같은 값으로 미리 등록해야 합니다. 미리 등록하지 않거나 서버와 다르면, 동작하지 않거나 오동작 하거나 예외가 발생할 수 있습니다.

```c#
// 서버와 같은 값으로 등록해야 한다.
ProtocolManager.getInstance().RegisterProtocol(0, Messages.MessagesReflection.Descriptor);
```

### 메시지 전송

Request()로 메시지를 전송하면 서버 응답을 기다립니다. 서버 응답을 기다리는 동안 추가적인 Request()는 큐에 저장되고 서버 응답을 처리한 후 순차적으로 처리하게 됩니다. 서버 응답을 받아 처리하려면 콜백 매개변수를 전달해야 합니다.

```c#
/// <summary>
/// 유저 에이전트를 사용 해서 프로토 버프 메세지 전송
/// </summary>
/// <typeparam name="T">프로토 버프 타입 메세지</typeparam>
/// <param name="agent">전송할 유저 에이전트</param>
/// <param name="message">전송할 프로토 버프 메세지</param>
/// <param name="action">응답 처리 할 동작</param>
static public void Request<T>(User.UserAgent agent, IMessage message, Action<User.UserAgent, T> action) where T : IMessage;

/// <summary>
/// 유저 에이전트를 사용 해서 패킷 전송
/// </summary>
/// <param name="agent">전송할 유저 에이전트</param>
/// <param name="packet">전송할 패킷</param>
/// <param name="action">응답 처리 할 동작</param>
static public void Request(User.UserAgent agent, Packet packet, Action<User.UserAgent, Packet> action);
```

Request 응답을 받기 위해 리스너를 등록하는 방법도 있는데, 이는 [Unity 심화 개발 가이드 > 메세지 핸들링](../unity-advanced/unity-advanced-04-message-handling.md)에서 확인하실 수 있습니다.

지정된 시간 내에 응답이 오지 않으면 타임아웃을 발생시키고 다음 메시지를 처리합니다. 타임아웃은 UserAgent.OnErrorCommandListeners 리스너와 UserAgent.OnErrorCustomCommandListeners 리스너에 ErrorCode.TIMEOUT으로 전달됩니다.

Send()로 메시지를 전송하면 Send()의 호출 즉시 서버로 전송되며 별도의 응답을 기다리지 않습니다. Request()에 대한 응답을 기다리는 중에도 Send()를 사용한 메시지는 바로 서버로 전송됩니다.

```c#
Connector connector = new Connector();
UserAgent user = GameAnvilConnector.getUserAgent();
// UserAgent로 전달되는 서버 알림을 받는 리스너 등록
user.AddListener((UserAgent user, Messages.SampleReceive msg)=> { }); 

// UserAgent Send
Messages.SampleSend sampleSend = new Messages.SampleSend(); 
user.Send(sampleSend);

// UserAgent Request
Messages.SampleRequest SampleRequest = new Messages.SampleRequest();
user.Request(SampleRequest, (UserAgent user, Messages.SampleResponse res) => { }); // 콜백 매개변수 전달
```

### 커스텀 패킷

패킷 클래스를 이용하여 ProtocolBuffer 외의 임의의 데이터를 바이트 스트림으로 직렬화해 사용할 수 있습니다. 패킷에 대한 자세한 내용은 [Unity 심화 개발 가이드 > 패킷](../unity-advanced/unity-advanced-05-packet.md)을 참고합니다.

```c#
Connector connector = new Connector();
UserAgent user = GameAnvilConnector.getUserAgent();
int reqMsgId = 1;
int resMsgId = 2;

user.AddListener(resMsgId, (UserAgent user, Packet packet)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
// 패킷 클래스 이용
Packet sampleSendPacket = new Packet(reqMsgIndex, sampleSend.ToByteArray())
user.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
// 패킷 클래스 이용
Packet sampleRequestPacket = new Packet(reqMsgIndex, sampleRequestPacket.ToByteArray())
user.Request(sampleRequestPacket, (UserAgent user, Packet packet)=> { });
```