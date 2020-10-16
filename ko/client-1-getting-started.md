# GameAnvil Connector

* [GameAnvil](https://github.nhnent.com/game-server-engine/GameAnvil.git) 서버 접속을 위한 전용 Connector 모듈입니다.



## 시스템 요구사항

- #### 지원하는 언어

  - C# .Net 4.5 이상
  - Typescript
  - C++ (예정)

- #### 타겟 개발환경

  - Unity 3D
  - CocosCreator
  - Unreal 3D(예정)

- #### 지원하는 네트워크 프로토콜

  - TCP/IP
  - SSL over TCP/IP
  - WebSocket
  - WebSocket with SSL

- #### 사용 가능한 응용 프로토콜 형식

  - Google Protocol Buffers
  - Google FlatBuffers (예정)
  - 커스텀 바이트 스트림
  - HTTP/HTTPS (특정한 용도로 한정)



## 레퍼런스 프로젝트

| 서버 | 클라이언트 | 설명 |
| --- | ----- | --- |
| [참고](https://nhnent.dooray.com/project/posts/2776353636252511965) | [참고](https://nhnent.dooray.com/project/posts/2776355680192301702) | 실제 게임 서버와 유니티 클라이언트 레퍼런스 코드 및 설명 |


GameAnvilConnector를 사용하기 위한 간단한 방법을 소개합니다.



### 1\. GameAnvilConnector 설치 및 초기화

GameAnvilConnector DLL 파일들을 프로젝트에 포함시킵니다. GameAnvilConnector DLL 파일들은 [여기](https://github.nhnent.com/game-server-engine/GameAnvil-connector-charp/tree/master/dll)에서 구할 수 있습니다.

- Google.Protobuf.dll
- LZ4.dll
- SuperSocket.ClientEngine.dll
- GameAnvil.dll



### 2. Connector 객체 생성

 Connector객체를 생성합니다.

```c#
using GameAnvil;

Connector connector = new Connector();
```


서버와의 주고받는 메세지의 처리를 위해 Update() 함수를 주기적으로 호출합니다. Update()함수의 호출 주기는 자유롭게 설정해도 무방하나 호출하지 않을 경우 서버로부터 메세지를 받더라도 이에대한 알림을 받을 수 없습니다.

```c#
connector.Update();
```
이제 GameAnvil Connector의 사용 준비가 완료되었습니다.



### 3. 서버 접속 및 인증

[ConnectionAgent](4z2.basic##ConnectionAgent)를 이용해 서버 접속 및 인증을 진행합니다. 접속 및 인증 결과는 [IConnectionListener](4z2.basic####IConnectionListener-등록)를 구현한 Listener를 등록해 알 수 있습니다.

```c#
class ConnectionListener : IConnectionListener{
    ...
}

ConnectionListener listener = new ConnectionListener();

ConnectionAgent connection = connector.GetConnectionAgent();
connection.AddConnectionListener(listener);

// ip	: ip address
// port	: port
session.Connect(ip, port);

// deviceId    : 사용자 기기를 식별 할 수 있는 고유 ID. 서버 구현에 따라 사용하지 않을 수 있음.
// accountId   : 사용자 계정을 식별 할 수 있는 고유 ID.
// password    : 사용자 계정의 password. 서버 구현에 따라 사용하지 않을 수 있음.
session.Authentication(deviceId, accountId, password);
```



### 4\. 로그인 및 기본 기능 사용

[UserAgent](4z2.basic#UserAgent)를 통해 로그인/로그아웃 및 기본 기능을 사용할 수 있습니다. 로그인/로그아웃 및 기본 기능의 결과는 [IUserListener](4z2.basic####IUserListener)를 등록해 알 수 있습니다. [UserAgent](4z2.basic##UserAgent)는 Connector.CreateUserAgent() 함수를 이용해 생성할 수 있습니다. ServiceId 와 SubId로 구분되는 여러개의 UserAgent를 생성할 수 있으며 생성된 각 UserAgent는 독립적으로 사용할 수 있습니다. 생성된 [UserAgent](4z2.basic##UserAgent)는 Connector 내부에서 캐싱하게 되며 Connector.GetUserAgent() 함수를 이용해 다시 사용할 수 있습니다. 

```c#
// serviceId    : UserAgent가 사용할 serviceId. 
// subId        : 서비스별 UserAgent식별 할 수 있는 고유 ID. 서버 구현에 따라 사용하지 않을 수 있음.
UserAgent userAgent = connector.CreateUserAgent(serviceId, subId);

userAgent = connector.GetUserAgent(serviceId, subId);
```

```c#
class UserListener : IUserListener{
    ...
}

UserListener listener = new UserListener();

UserAgent user = connector.GetUserAgent(serviceId, subId);
user .AddUserListener(listener)

// payload     : 서버로 전달할 추가적인 값. 서버 구현에 따라 사용하지 않을 수 있음.
// channelId   : UserAgent가 로그인할 채널 ID. 서버 구현에 따라 사용하지 않을 수 있음.
user.Login(payload, channelId);

user.Logout();

user.CreateRoom();

user.MatchRoom();

user.LeaveRoom();

...
```



### 5\. 메시지 전송

[ConnectionAgent](4z2.basic##ConnectionAgent), [UserAgent](4z2.basic##UserAgent)의 기본 기능 외에 Request()와 Send()를 이용하여 메시지를 서버로 전송할 수 있습니다. Request()로 메시지를 전송하면 서버 응답을 기다립니다. 서버 응답을 기다리는 동안 추가적인 Request()는 queue에 저장되고 서버 응답을 처리한 후 순차적으로 처리하게 됩니다.  Send()로 메시지를 전송하면 Send()의 호출 즉시 서버로 전송되며 별도의 응답을 기다리지 않습니다. Request() 에 대한 응답을 기다리는 중에도 Send()를 사용한 패킷은 바로 서버로 전송됩니다. 자세한 내용은 [여기](4z2.basic##메시지)를 참고합니다.

```c#
ConnectionAgent connection = connector.GetConnectionAgent();
CustomMessage.SampleConnectionSend sampleConnectionSend= new CustomMessage.SampleConnectionSend (); 
connection.Send(sampleConnectionSend);

CustomMessage.SampleConnectionRequest sampleConnectionRequest = new CustomMessage.SampleConnectionRequest();
connection.Request(sampleConnectionRequest);
connection.Request(sampleConnectionRequest, (UserAgent userAgent, CustomMessage.sampleConnectionResponse res)=> { // todo });
```
```c#
UserAgent user = Connector.GetUserAgent(serviceId, subId);
CustomMessage.SampleUserSend sampleUserSend = new CustomMessage.SampleUserSend (); 
user.Send(sampleUserSend);

CustomMessage.SampleUserRequest SampleUserRequest = new CustomMessage.SampleUserRequest();
user.Request(SampleUserRequest);
user.Request(SampleUserRequest, (UserAgent userAgent, CustomMessage.SampleUserResponse res)=> { // todo });
```


### 6. GameAnvil Connector 종료

게임 플레이 종료전 Connector.CloseSoket() 함수를 호출해야 합니다.  특히 UnityEditor의 경우 Connector.CloseSoket()을 호출하지 않고 종료를 할 경우 다음 실행시  UnityEditor가 응답을 하지 않아 강제종료를 해야하기 때문에 꼭 Connector.CloseSoket() 함수를 호출해야 합니다.
