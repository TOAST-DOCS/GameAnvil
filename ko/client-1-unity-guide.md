## Game > GameAnvil > 클라이언트 개발 가이드 > Unity 개발 가이드
가이드 환경

* Unity 2018.4.1f1
* Visual Studio 2017
* GameAnvil Connector : 1.1.0



## GameAnvil Connector 설치

Unity 프로젝트에 GameAnvil.unitypackage를 설치합니다. GameAnvil.unitypackage는 [여기](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector.unitypackage)에서 다운받을 수 있습니다. 패키지를 설치하면 다음과 같이 GameAnvil 폴더 아래에 DLL파일들이 설치됩니다.  이 DLL파일들은 C#으로 만들어진 DLL이므로 Andoid, iOS, PC 등의 플렛폼에서 모두 사용가능합니다. 

![unitypackage](http://static.toastoven.net/prod_gameanvil/images/client-1-unitypackage.png)



## Connector 생성.

GameAnvilConnector를 사용하기 위해서 제일먼저 해야 할 것은 Connector를 생성하는것 입니다. 기본 설정과 Agent의 관리를 담당하며, 내부 동작과 관련된 로그를 볼수 있도록 delegate를 등록할 수 있습니다.

```c#
using GameAnvil;
Connector connector = new Connector();
```


서버와의 주고받는 메세지의 처리를 위해 Update() 함수를 주기적으로 호출합니다. MonoBehaviour의 Update() 에거 호출하도록 하는것이 제간 간편한방법이며, 필요에 따라 다른 방식으로 호출해주어도 무방합니다. Update()함수의 호출 주기는 자유롭게 설정해도 되지만 호출하지 않을 경우 서버로부터 메세지를 받더라도 이에대한 알림을 받을 수 없습니다.

```c#
connector.Update();
```

간단하게 다음과 같이 MonoBehaviour를 상속받아 Connector를 관리하는 스크립트를 만들어 사용할 수 있습니다.

```c#
using GameAnvil;

public class ConnectHandler : MonoBehaviour
{
    public static Connector connector = null;

    private void Awake()
    {
        if(connector == null){
            connector = new GameAnvil.Connector();
            // 커넥터 로그 추가
            connector.Logger += (level, log) =>
            {
                Debug.Log(string.Format("Log[{0}]:{1}", level, log));
            };
            connector.LvNetLogger += (level, log) =>
            {
                Debug.Log(string.Format("Net[{0}]:{1}", level, log));
            };
        }
    }

    private void Update()
    {
        if(connector != null)
	        connector.Update();
    }

    private void OnDestroy()
    {
        if (connector != null && connector.IsConnected())
        {
            connector.CloseSocket();
        }
    }
}
```

이코드는 예시로 제공되는것이며 프로젝트의 필요에 따라 적절하게 변경하여 사용하시면 됩니다. 이제 GameAnvil Connector의 사용 준비가 완료되었습니다.



## 서버 접속 및 인증

서버 접속 및 인증은 ConnectionAgent를 이용해 진행합니다. ConnectionAgent는 GameAnvil Server의 Connection Node와 관련된 작업을 담당합니다. 접속(Connect()), 인증(Authentication()) 등 기본 세션 관리 기능 및 채널 목록 등을 제공하며, 서버 구현에 따라 채널 정보를 제공하거나 사용자 정의 메시지를 주고받을 수 있습니다. ConnectionAgent는 Connector가 초기화 될 때 자동으로 생성되며 Connector.GetConnectionAgent() 함수를 이용해 얻을 수 있습니다. 접속 및 인증 결과는 IConnectionListener를 구현한 Listener를 등록해 알 수 있습니다.
```c#
class ConnectionListener : IConnectionListener{
    ...
}


ConnectionListener listener = new ConnectionListener();


GameAnvil.ConnectionAgent connectionAgent = connector.GetConnectionAgent();
connectionAgent.AddConnectionListener(listener);


// ip    : ip address
// port    : port
connectionAgent.Connect(ip, port);
// ConnectionListener.OnConnect으로 응답


// deviceId    : 사용자 기기를 식별 할 수 있는 고유 ID. 서버 구현에 따라 사용하지 않을 수 있음.
// accountId   : 사용자 계정을 식별 할 수 있는 고유 ID.
// password    : 사용자 계정의 password. 서버 구현에 따라 사용하지 않을 수 있음.
connectionAgent.Authentication(deviceId, accountId, password);
// ConnectionListener.OnAuthentication으로 응답
```


## 로그인 및 기본 기능 사용
로그인/로그아웃 및 기본 기능는 UserAgent를 통해 사용할 수 있습니다. UserAgent는 GameAnvil Server의 Game Node와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 Room 관리 등 기본 기능을 제공하며, 서버 구현에 따라 사용자 정의 기능을 추가적으로 제공할 수도 있습니다. UserAgent를 사용하기 위해서는 Connector.CreateUserAgent() 함수를 이용해 새로운 UserAgent를 생성해야 합니다. ServiceId 와 SubId로 구분되는 여러개의 UserAgent를 생성할 수 있으며 생성된 각 UserAgent는 독립적으로 사용할 수 있습니다. 생성된 UserAgent는 Connector 에서 내부적으로 관리되며 Connector.GetUser()함수를 이용해 다시 사용할 수 있습니다. 로그인/로그아웃 및 기본 기능의 결과는 IUserListener를 등록해 알 수 있습니다. UserAgent는 Connector.CreateUserAgent() 함수를 이용해 생성할 수 있습니다. ServiceId 와 SubId로 구분되는 여러개의 UserAgent를 생성할 수 있으며 생성된 각 UserAgent는 독립적으로 사용할 수 있습니다. 생성된 UserAgent는 Connector 내부에서 캐싱하게 되며 Connector.GetUserAgent() 함수를 이용해 다시 사용할 수 있습니다. 
```c#
// serviceId    : UserAgent가 사용할 serviceId. 
// subId        : 서비스별 UserAgent식별 할 수 있는 고유 ID. 1 이상의 정수.
UserAgent userAgent = connector.CreateUserAgent(serviceId, subId);


userAgent = connector.GetUserAgent(serviceId, subId);
```


```c#
class UserListener : IUserListener{
    ...
}


UserListener listener = new UserListener();


UserAgent userAgent = connector.GetUserAgent(serviceId, subId);
userAgent.AddUserListener(listener)


// payload     : 서버로 전달할 추가적인 값. 서버 구현에 따라 사용하지 않을 수 있음.
// channelId   : UserAgent가 로그인할 채널 ID. 서버 구현에 따라 사용하지 않을 수 있음.
userAgent.Login(payload, channelId);
// UserListener.OnLogin으로 응답


userAgent.Logout();
// UserListener.OnLogout으로 응답


userAgent.CreateRoom();
// UserListener.OnCreateRoom으로 응답


userAgent.MatchRoom();
// UserListener.OnMatchRoom으로 응답


userAgent.LeaveRoom();
// UserListener.OnLeaveRoom으로 응답
...
```


## 메시지
ConnectionAgent, UserAgent의 기본 기능 외에 Request()와 Send()를 이용하여 메시지를 서버로 전송할 수 있습니다. 메시지를 전송하기 위해서는 메시지를 생성하고 등록하는 과정이 필요합니다.



### 메시지 생성

GameAnvil은 기본 메시지 프로토콜로 [ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/overview)를 사용하고 있습니다.  .proto파일에 메시지를 정의하고, protoc 컴파일러로 실제 클래스 소스 코드를 생성하게 됩니다. 생성된 소스 코드를 프로젝트에 추가하여 사용할 수 있습니다. protoc 컴파일러는 GameAnvil/protoc 폴더에서 찾을 수 있습니다.  protoc에 대한 자세한 설명은 [여기](https://developers.google.com/protocol-buffers/docs/proto3#generating)를 참고하세요.

이제 메시지를 만들어 보도록 하겠습니다.  먼저 Assets 폴더 아래에 protocols 폴더를 생성하고 다음과 같이 messages.proto파일을 생성합니다.

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

그리고 윈도우 cmd 창을 띄우고 protocols 폴더로 이동한 다음 아래와 같이 입력합니다.

```
../GameAnvil/protoc/protoc --csharp_out=./ messages.proto
```

그러면 protocols 폴더에 Messages.cs파일이 생성된 것을 확인할 수 있습니다. 



### 메시지 등록

새로 생성한 메시지를 사용하기 위해서는 사용하려는 메시지를 ProtocolManager에 서버와 같은 값으로 미리 등록해야합니다. 등록하지 않거나 서버와 다를 경우 동작하지 않거나 오동작 하거나 예외가 발생 할 수 있습니다

```c#
// 서버와 같은 값으로 등록해야한다.
ProtocolManager.getInstance().RegisterProtocol(0, Messages.MessagesReflection.Descriptor);
```



### 메시지 전송

Request()로 메시지를 전송하면 서버 응답을 기다립니다. 서버 응답을 기다리는 동안 추가적인 Request()는 queue에 저장되고 서버 응답을 처리한 후 순차적으로 처리하게 됩니다. 서버 응답을 받아 처리하려면 Listener를 등록해야 합니다. 리스너를 등록하지 않을 경우 서버 응답을 받아도 별도의 알림을 주지 않고 다음 메시지를 처리하게 됩니다. 지정된 시간동안 응답이 오지 않으면 Timeout을 발생시키고 다음 메시지를 처리합니다. Timeout은 OnError() 리스너에 ErrorCode.TIMEOUT으로 전달이 됩니다.

Send()로 메시지를  전송하면 Send()의 호출 즉시 서버로 전송되며 별도의 응답을 기다리지 않습니다. Request() 에 대한 응답을 기다리는 중에도 Send()를 사용한 메시지는 바로 서버로 전송됩니다.

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
connection.AddListener((ConnectionAgent connection, Messages.SampleReceive msg)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
connection.Send(sampleSend);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
connection.Request(sampleRequest);
```

```c#
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
user.AddListener((UserAgent user, Messages.SampleReceive msg)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
user.Send(sampleSend);

Messages.SampleRequest SampleRequest = new Messages.SampleRequest();
user.Request(SampleRequest);
```



### 커스텀 메시지

Packet 클래스를 이용하여 ProtocolBuffer 외의 임의의 데이터를 바이트 스트림으로 직렬화해 사용할 수 있습니다. Packet에 대한 자세한 내용은 [여기](##Packet)를 참고합니다.

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
int reqMsgId = 1;
int resMsgId = 2;

connection.AddListener(resMsgId, (ConnectionAgent connection, Packet packet)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
Packet sampleSendPacket = new Packet(reqMsgId, sampleSend.ToByteArray())
connection.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
Packet sampleRequestPacket = new Packet(reqMsgId, sampleRequest.ToByteArray())
connection.Request(sampleRequestPacket);
connection.Request(sampleRequestPacket, (ConnectionAgent connection, Packet packet)=> { });
```

```c#
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
int reqMsgId = 1;
int resMsgId = 2;

user.AddListener(resMsgId, (UserAgent user, Packet packet)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
Packet sampleSendPacket = new Packet(reqMsgIndex, sampleSend.ToByteArray())
user.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
Packet sampleRequestPacket = new Packet(reqMsgIndex, sampleRequestPacket.ToByteArray())
user.Request(sampleRequestPacket);
user.Request(sampleRequestPacket, (UserAgent user, Packet packet)=> { });
```

## Packet

서버와 주고 받는 모든 메시지는 Packet 모듈에 실려서 처리 되며 Packet 모듈이 제공하는 인터페이스를 이용하게 됩니다.



### 생성

GameAnvil Connector는 Google Protocol Buffer를 기본 프로토콜로 사용하고 있습니다 . Google Protocol Buffer를 이용하는 Packet 생성은 다음과 같습니다.

```C#
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
```

Google Protocol Buffer를 사용하지 않고도 Packet을 생성할 수 있습니다 .

```c#
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Pcket.CreateWithCustomMsg(customMsgId, requestMsg);

byte[] bytes =  packet.GetBytes();
string JsonString = Encoding.UTF8.GetString(bytes);
```



### 압축

패킷 크기가 클경우 압축하여 데이터 사용량을 줄일 수 있습니다 .  

```c#
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
packet.compress();

if (packet.isCompress())
    packet.decompress();

CustomMessage.SampleResponse responseMsg = packet.GetMessage<CustomMessage.SampleResponse>();

```



## Payload

GameAnvil에서 제공하는 기본 API를 이용할 때 추가적인 데이터가 필요할 수 있습니다. 이를 위해 기본 API들에는 추가 데이터를 넘겨줄 수 있는 Payload라는 파라메터가 포함되어 있습니다. 이 Payload에 필요한 데이터를 Packet에 담아 List형식으로 저장할 수 있습니다. 여기에 추가 데이터를 넣어 서버로 보내거나, 서버에서 보낸 메시지를 꺼낼 수 있습니다. 

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



## GameAnvil Connector 종료


게임 플레이 종료전 Connector.CloseSoket() 함수를 호출해 연결을 종료하는것이 좋습니다. 그렇지 않을 경우 서버에서 클라이언트의 종료를 인지하지 못할 수 있으며, 그럴 경우 불필요한 동작을 계속할 수 있습니다. 
