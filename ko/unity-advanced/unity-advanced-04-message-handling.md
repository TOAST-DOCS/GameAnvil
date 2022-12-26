## Game > GameAnvil > Unity 심화 개발 가이드 > 메시지 핸들링

## 메세지 핸들링

ConnectionAgent, UserAgent의 기본 기능 외에 Request()와 Send()를 이용하여 메시지를 서버로 전송할 수 있습니다.

### 메시지 전송

Request()로 메시지를 전송하면 서버 응답을 기다립니다. 서버 응답을 받아 처리하는 방법에는 먼저 [Unity 기초 개발 가이드 > 메세지 핸들링](../unity-basic/unity-basic-06-message-handling.md) 에서 소개한 것처럼 콜백 매개 변수를 전달하는 방법이 있습니다. 

또 다른 방법으로는 리스너를 등록하는 방법이 있습니다. 두 방법 중 어느 것도 적용하지 않을 경우 서버 응답을 받아도 별도의 알림 없이 다음 메시지를 처리하게 됩니다. 

리스너를 등록해서 Request()의 응답을 받는 예제를 살펴봅시다.

[Unity 기초 개발 가이드 > 메세지 핸들링](../unity-basic/unity-basic-06-message-handling.md) 에서 다루지 않았던 ConnectionAgent를 통한 Request(), Send() 사용 예제를 소개합니다.

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
// ConnectionAgent로 전달되는 서버 알림을 받는 리스너 등록
connection.AddListener((ConnectionAgent connection, Messages.SampleReceive msg)=> { });

// ConnectionAgent Send
Messages.SampleSend sampleSend= new Messages.SampleSend(); 
connection.Send(sampleSend);

// ConnectionAgent Request
Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
connection.Request(sampleRequest, (ConnectionAgent connection, Packet packet)=> { });
```

### 커스텀 패킷

패킷 클래스를 이용하여 ProtocolBuffer 외의 임의의 데이터를 바이트 스트림으로 직렬화해 사용할 수 있습니다. 패킷에 대한 자세한 내용은 [Unity 심화 개발 가이드 > 패킷](unity-advanced-05-packet.md) 을 참고합니다.

[Unity 기초 개발 가이드 > 메세지 핸들링](../unity-basic/unity-basic-06-message-handling.md) 에서 다루지 않았던 ConnectionAgent를 통한 Request(), Send() 사용 예제를 소개합니다.

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
int reqMsgId = 1;
int resMsgId = 2;

connection.AddListener(resMsgId, (ConnectionAgent connection, Packet packet)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
// 패킷 클래스 이용
Packet sampleSendPacket = new Packet(reqMsgId, sampleSend.ToByteArray())
connection.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
// 패킷 클래스 이용
Packet sampleRequestPacket = new Packet(reqMsgId, sampleRequest.ToByteArray())
connection.Request(sampleRequestPacket, (ConnectionAgent connection, Packet packet)=> { });
```