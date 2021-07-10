## Game > GameAnvil > 클라이언트 개발 가이드 > Unity 개발 가이드

## 가이드 환경

- Unity 2018.4.1f1
- Visual Studio 2017
- GameAnvil Connector: 1.2.0

## GameAnvil 커넥터 설치

Unity 프로젝트에 gameanvil-connector.unitypackage를 설치합니다. gameanvil-connector.unitypackage는 [여기](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector.unitypackage)에서 다운로드할 수 있습니다. 패키지를 설치하면 다음과 같이 GameAnvil 폴더 아래에 DLL 파일들이 설치됩니다. DLL 파일들은 C#으로 만든 것으로 Andoid, iOS, PC 등의 플렛폼에서 모두 사용 가능합니다. 

![unitypackage](http://static.toastoven.net/prod_gameanvil/images/client-1-gameanvil.png)

gameanvil-sample-template.unitypackage는 서버 템플릿에 포함된 채팅 서버에 대응하는 채팅 클라이언트로 [여기](https://static.toastoven.net/prod_gameanvil/files/gameanvil-sample-template.unitypackage)에서 다운로드할 수 있습니다. gameanvil-sample-template.unitypackage를 설치하면 다음과 같이 GameAnvilSampleForTemplate 폴더에 샘플 파일들이 설치됩니다. 여기에 포함된 Login Scene을 열고 실행하면 서버 템플릿의 채팅 서버에 접속하여 채팅을 할 수 있습니다.

![unitypackage](http://static.toastoven.net/prod_gameanvil/images/client-1-gameanvil-sample-template.png)

![unitypackage](http://static.toastoven.net/prod_gameanvil/images/client-1-chatting-client.png)

## Connector
GameAnvil 커넥터를 사용하려면 먼저 Connector를 생성해야 합니다. 기본 설정과 에이전트 관리를 담당하며, 내부 동작과 관련된 로그를 볼 수 있도록 콜백을 등록할 수 있습니다.

### 생성
다음과 같이 Connecor를 생성할 수 있습니다.
``` C#
using GameAnvil;
Connector connector = new Connector();
```

### 설정
Connector가 동작에 사용되는 설정이 있습니다. 이 설정들은 Connector 생성시 기본값으로 설정되지만 필요하다면 다음과 같이 직접 값을 바꿀 수 있습니다. 
``` C#
using GameAnvil;
Connector connector = new Connector();
connector.config.packetTimeout = 10;
```
`Connector.Config`를 미리 생성해놓고 이를 인자로 받아서 Connector 를 생성할 수도 있습니다. 또 이렇게 생성한`Connector.Config`를 MonoBehavior의 맴버로 만들어 놓을 경우 Unity의 Inspector 창에서 설정값을 바꿀 수도 있습니다. 
``` C#
using GameAnvil;
var config = new Connector.Config();
config.packetTimeout = 10;
Connector connector = new Connector(config);
```

설정의 종류는 다음과 같습니다.
| 이름 | 설명 | 기본값|
|--- | --- | --- |
|defaultReqestTimeout | TimeOut 기본 대기시간 설정 | 3(sec) |
| packetTimeout | 패킷이 지정된 시간안에 업데이트 되지 않으면, disconnect 되었다고 판단된다. pingInterval 보다 높게 설정되야 한다. | 5(sec)|
|pingInterval |서버와의 연결을 확인하기 위한 ping 주기 설정. 0일경우 사용안함 3(sec) |
| useIPv6 | 접속시 IPv6 주소로 변환 여부 | false |
| useSocketNoDelay | 소켓의 Nodelay 사용 여부 | true |


### 로그
Connector는 직접 로그를 남기지 않고 콜백을 통해 로그를 전달합니다. 다음과 같이 콜백을 등록해야 Connector에서 발생하는 로그를 받을 수 있습니다. 
``` c#
connector = new GameAnvil.Connector();
connector.Logger += (level, log) =>
{
Debug.Log(string.Format("Log[{0}]:{1}", level, log));
};
connector.LvNetLogger += (level, log) =>
{
Debug.Log(string.Format("Net[{0}]:{1}", level, log));
};
```
Connector.Logger는 Connector에서 발생한 에러나 경고등에 대한 로그를 전달하고, Connector.LvNetLogger는 Connector와 서버가 주고받는 Packet을 처리에 대한 로그를 전달하며 그 양이 많습니다. Connector.LvNetLogger는 개발중에만 사용하길 권장합니다. 

### Update
서버와 주고받는 메시지 처리를 위해 Update() 함수를 주기적으로 호출해야 합니다. MonoBehaviour의 Update() 에서 호출하도록 하는것이 제일 간편한방법이지만, 필요에 따라 다른 방식으로 호출해도 무방합니다만 Thread safe하지 않으므로 별도의 Thread에서 호출하면 안됩니다. Update() 함수의 호출 주기는 자유롭게 설정해도 되지만 호출하지 않을 경우 서버로부터 메시지를 받더라도 이에 대한 알림을 받을 수 없습니다.

```c#
connector.Update();
```

### ConnectHandler
간단하게 다음과 같이 MonoBehaviour를 상속받아 커넥터를 관리하는 스크립트를 만들어 사용할 수 있습니다.

```c#
using GameAnvil;

public class ConnectHandler : MonoBehaviour
{
    public static Connector connector = null;

	[SerializeField]
	GameAnvil.Connector.Config config = null;

	private void Awake()
    {
        if(connector == null){
            connector = new GameAnvil.Connector(config);
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

	private void OnApplicationPause(bool pause){
        if (pause)
        {
            // 입력한 시간동안 서버의 clientStateCheck 기능을 정지시킨다
            // 이시간이 지나면 clientStateCheck 기능이 동작하여 연결이 끊어질 수 있다. 
            connector.GetConnectionAgent().PauseClientStateCheck(10 * 60);
          
            connector.Update();
        } else {
            connector.Update();

            // 서버의 clientStateCheck 기능을 다시 동작시킨다.
            connector.GetConnectionAgent().ResumeClientStateCheck();
        }
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
이 코드는 예시이며 필요에 따라 적절하게 변경해 사용하면 됩니다. 이제 GameAnvil 커넥터의 사용 준비가 완료되었습니다.

## ConnectionAgent
ConnectionAgent는 GameAnvil 서버의 커넥션 노드와 관련된 작업을 담당합니다. 접속(Connect()), 인증(Authentication()) 등 기본 세션 관리 기능 및 채널 목록 등을 제공하며, 서버 구현에 따라 채널 정보를 제공하거나 사용자 정의 메시지를 주고받을 수 있습니다. ConnectionAgent는 커넥터가 초기화될 때 자동으로 생성되며 Connector.GetConnectionAgent() 함수를 이용해 얻을 수 있습니다.
```c#
ConnectionAgent connectionAgent = connector.GetConnectionAgent();
```
### 서버 접속
ConnectionAgent의 Connect 함수를 이용해 서버에 접속합니다. 인자로 서버의 ip 주소와 port 번호, 응답을 처리할 콜백을 넘겨줍니다. 콜백의 ResultCodeConnect result 매개변수를 통해 성공/실패를 확인할 수 있습니다.
```c#
connector.GetConnectionAgent().Connect(ip, port, (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
    if (result == ResultCodeConnect.CONNECT_SUCCESS) {
		// 접속 성공
    } else {
	    // 접속 실패
    }
});
```

### 인증
ConnectionAgent의 Authenticate 함수를 이용해 인증절차를 진행합니다. 인자로 deviceId, accountId, password, payload, 응답을 처리할 콜백을 넘겨줍니다. deviceId는 중복 접속에 대한 처리에 활용되며, accountId와 password를 활용하여 서버에서 인증 처리를 할 수 있습니다. 인증을 위해 deviceId, accountId, password 외의 추가 정보가 필요할 경우 payload에 담아 보낼수 있으며 사용하지 않을 경우 payload 인자는 생략할 수 있습니다.  
Authenticate 함수를 호출하면 서버에서는 BaseConnection의 onAuthentication() 콜백이 호출되며 이 콜백의 처리 결과로 인증의 성공, 실패가 결정됩니다.

```c#
connector.GetConnectionAgent().Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
		// 인증 성공
    } else {
		// 인증 실패
    }
});
```

### 체널 정보
GameAnvil은 설정으로 자유롭게 채널을 구성할 수 있습니다. 이런 채널구성은 서버와 클라이언트간에 미리 약속하여 고정된 형태로 사용할 수도 있지만, 상황에 따라 다양하게 변경하여 사용할 수도 있습니다. ConnectionAgent에서는 이렇게 변경된 채널 정보를 얻어올 수 있도록 몇가지 함수를 제공합니다. 

GetChannelList()는 특정 서비스의 채널 아이디 목록을 요청하여 받아올 수 있습니다. 인자로 서비스 이름과 응답을 처리할 콜백을 넘겨줍니다. 
```c#
connector.GetConnectionAgent().GetChannelList(serviceName, (ConnectionAgent connection, ResultCodeChannelList result, List<string> channelIdList) => {
	if(result == ResultCodeChannelList.CHANNEL_LIST_SUCCESS){
		// 채널 목록 요청 성공
	} else {
		// 채널 목록 요청 실패
	}
});
```
GetChannelCountInfo()는 특정 채널의 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 인자로 서비스 이름과 채널 아이디, 응답을 처리할 콜백을 넘겨줍니다. 
```
connector.GetConnectionAgent().GetChannelCountInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// 채널 카운트 정보 요청 성공
	} else {
		// 채널 카운트 정보 요청 실패
	}
});
```
GetChannelInfo()는 특정 채널의 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 인자로 서비스 이름과 채널 아이디, 응답을 처리할 콜백을 넘겨줍니다. 
```c#
connector.GetConnectionAgent().GetChannelInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// 채널 정보 요청 성공
	} else {
		// 채널 정보 요청 실패
	}
});
```
GetAllChannelCountInfo()는 특정 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 인자로 서비스 이름과 응답을 처리할 콜백을 넘겨줍니다. 
```
connector.GeConnectionAgent().GetAllChannelCountInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// 모든 채널 카운트 정보 요청 성공
	} else {
		// 모든 채널 카운트 정보 요청 실패
	}
});
```
GetAllChannelInfo()는 특정 서비스의 모든 채널에 대한 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 인자로 서비스 이름과 응답을 처리할 콜백을 넘겨줍니다. 
```c#
connector.GetConnectionAgent().GetAllChannelInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
	if(result == ResultCodeChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// 모든 채널 정보 요청 성공
	} else {
		// 모든 채널 정보 요청 실패
	}
});
```
### 접속 종료
ConnectionAgent의 Disconnect 함수를 이용해 서버와의 접속을 종료합니다. 인자로 응답을 처리할 콜백을 넘겨줍니다. 콜백의 ResultCodeDisconnect result 매개변수를 통해 결과를 알 수 있습니다.
```c#
connector.GetConnectionAgent().Disconnect((ConnectionAgent connectionAgent, ResultCodeDisconnect result) => {
    if (result == ResultCodeDisconnect.SOCKET_DISCONNECT) {
		// 정상 종료
    } else {
	    // 비정상 종료
    }
});
```
### Listener

ConnectionAgent에는 모든 동작의 결과 또는 알림을 받을 수 있도록 각각의 delegate를 맴버로 가지고 있습니다. 이 delegate에 함수를 등록하면 앞서 설명한 API에 콜백 인자를 생략하고 호출했을 경우 또는 서버에서 일림을 보냈을 경우 등록한 함수로 응답을 받을 수 있습니다.  

```c#
connector.GetConnectionAgent().onConnectListeners += listener.OnConnect;
connector.GetConnectionAgent().onAuthenticationListeners += listener.OnAuthentication;
connector.GetConnectionAgent().onChannelListListeners += listener.OnChannelList;
connector.GetConnectionAgent().onChannelInfoListeners += listener.OnChannelInfo;
connector.GetConnectionAgent().onAllChannelInfoListeners += listener.OnAllChannelInfo;
connector.GetConnectionAgent().onChannelCountInfoListeners += listener.OnChannelCountInfo;
connector.GetConnectionAgent().onAllChannelCountInfoListeners += listener.OnAllChannelCountInfo;
connector.GetConnectionAgent().onDisconnectListeners += listener.OnDisconnect;
connector.GetConnectionAgent().onErrorCommandListeners += listener.OnError;
connector.GetConnectionAgent().onErrorCustomCommandListeners += listener.OnError;
```

IConnectionListener는 ConnectionAgent의 모든 동작의 결과 또는 알림을 정의한 인터페이스입니다. 이 인터페이스를 구현한 리스너를 ConnectionAgent.AddConnectionListener()로 등록하면 등록한 리스너로 응답을 받을 수 있습니다. 

```c#
public class ConnectionListener : IConnectionListener
{
    public void OnAllChannelCountInfo(ConnectionAgent connectionAgent, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) { }
    public void OnAllChannelInfo(ConnectionAgent connectionAgent, ResultCodeAllChannelInfo result, Dictionary<string, Payload> channelInfo) { }
    public void OnAuthentication(ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) { }
    public void OnChannelCountInfo(ConnectionAgent connectionAgent, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) { }
    public void OnChannelInfo(ConnectionAgent connectionAgent, ResultCodeChannelInfo result, Payload channelInfo) { }
    public void OnChannelList(ConnectionAgent connectionAgent, ResultCodeChannelList result, List<string> channelIdList) { }
    public void OnConnect(ConnectionAgent connectionAgent, ResultCodeConnect result) { }
    public void OnDisconnect(ConnectionAgent connectionAgent, ResultCodeDisconnect result, bool force, Payload payload) { }
    public void OnError(ConnectionAgent connectionAgent, ErrorCode errorCode, Commands commands) { }
    public void OnError(ConnectionAgent connectionAgent, ErrorCode errorCode, string command) { }
}

connector.GetConnectionAgent().AddConnectionListener(new ConnectionListener);
```



## UserAgent

UserAgent는 GameAnvil 서버의 게임 노드와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 룸 관리 등 기본 기능을 제공하며, 서버 구현에 따라 사용자 정의 기능을 추가적으로 제공할 수도 있습니다. UserAgent를 사용하기 위해서는 Connector.CreateUserAgent() 함수를 이용해 새로운 UserAgent를 생성해야 합니다. ServiceId 와 SubId로 구분되는 여러개의 UserAgent를 생성할 수 있으며 생성된 각 UserAgent는 독립적으로 사용할 수 있습니다. 생성된 UserAgent는 Connector 에서 내부적으로 관리되며 Connector.GetUser()함수를 이용해 다시 사용할 수 있습니다. 로그인/로그아웃 및 기본 기능의 결과는 IUserListener를 등록해 알 수 있습니다. UserAgent는 Connector.CreateUserAgent() 함수를 이용해 생성할 수 있습니다. ServiceId 와 SubId로 구분되는 여러 개의 UserAgent를 생성할 수 있으며 생성된 각 UserAgent는 독립적으로 사용할 수 있습니다. 생성된 UserAgent는 Connector 내부에서 캐싱하게 되며 Connector.GetUserAgent() 함수를 이용해 다시 사용할 수 있습니다. 
```
// serviceId    : UserAgent가 사용할 serviceId. 
// subId        : 서비스별 UserAgent식별 할 수 있는 고유 ID. 1 이상의 정수.
UserAgent userAgent = connector.GetUserAgent(serviceId, subId);
if(userAgent == null) {
	userAgent = connector.CreateUserAgent(serviceId, subId);
}

```
### 로그인 및 기본 기능 사용
로그인/로그아웃 및 기본 기능은 UserAgent를 통해 사용할 수 있습니다. 

```

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

GameAnvil은 기본 메시지 프로토콜로 [ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/overview)를 사용합니다. .proto파일에 메시지를 정의하고, protoc 컴파일러로 실제 클래스 소스 코드를 생성하게 됩니다. 생성된 소스 코드를 프로젝트에 추가하여 사용할 수 있습니다. protoc 컴파일러는 GameAnvil/protoc 폴더에서 찾을 수 있습니다.  protoc에 대한 자세한 설명은 [여기](https://developers.google.com/protocol-buffers/docs/proto3#generating)를 참고하세요.

이제 메시지를 만들어 보도록 하겠습니다.  먼저 Assets 폴더 아래에 protocols 폴더를 생성하고 다음과 같이 messages.proto 파일을 생성합니다.

```
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

새로 생성한 메시지를 사용하려면 사용할 메시지를 ProtocolManager에 서버와 같은 값으로 미리 등록해야 합니다. 미리 등록하지 않거나 서버와 다르면, 동작하지 않거나 오동작 하거나 예외가 발생할 수 있습니다

```
// 서버와 같은 값으로 등록해야한다.
ProtocolManager.getInstance().RegisterProtocol(0, Messages.MessagesReflection.Descriptor);
```

### 메시지 전송

Request()로 메시지를 전송하면 서버 응답을 기다립니다. 서버 응답을 기다리는 동안 추가적인 Request()는 큐에 저장되고 서버 응답을 처리한 후 순차적으로 처리하게 됩니다. 서버 응답을 받아 처리하려면 리스너를 등록해야 합니다. 리스너를 등록하지 않을 경우 서버 응답을 받아도 별도의 알림을 주지 않고 다음 메시지를 처리하게 됩니다. 지정된 시간 내에 응답이 오지 않으면 타임아웃을 발생시키고 다음 메시지를 처리합니다. 타임아웃은 OnError() 리스너에 ErrorCode.TIMEOUT으로 전달됩니다.

Send()로 메시지를  전송하면 Send()의 호출 즉시 서버로 전송되며 별도의 응답을 기다리지 않습니다. Request() 에 대한 응답을 기다리는 중에도 Send()를 사용한 메시지는 바로 서버로 전송됩니다.

```
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
connection.AddListener((ConnectionAgent connection, Messages.SampleReceive msg)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
connection.Send(sampleSend);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
connection.Request(sampleRequest);
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
user.AddListener((UserAgent user, Messages.SampleReceive msg)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
user.Send(sampleSend);

Messages.SampleRequest SampleRequest = new Messages.SampleRequest();
user.Request(SampleRequest);
```

### 커스텀 메시지

패킷 클래스를 이용하여 ProtocolBuffer 외의 임의의 데이터를 바이트 스트림으로 직렬화해 사용할 수 있습니다. 패킷에 대한 자세한 내용은 [여기](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-1-unity-guide/##Packet)를 참고합니다.

```
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

## 패킷

서버와 주고받는 모든 메시지는 패킷 모듈에 실려서 처리되며 패킷 모듈이 제공하는 인터페이스를 이용합니다.

### 생성

GameAnvil 커넥터는 Google Protocol Buffer를 기본 프로토콜로 사용합니다. Google Protocol Buffer를 이용하는 패킷 생성은 다음과 같습니다.

```
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
```

패킷 

```
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Pcket.CreateWithCustomMsg(customMsgId, requestMsg);

byte[] bytes =  packet.GetBytes();
string JsonString = Encoding.UTF8.GetString(bytes);
```

### 압축

패킷 크기가 크면 압축하여 데이터 사용량을 줄일 수 있습니다.  

```
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
packet.compress();

if (packet.isCompress())
    packet.decompress();

CustomMessage.SampleResponse responseMsg = packet.GetMessage<CustomMessage.SampleResponse>();
```

## Payload

GameAnvil에서 제공하는 기본 API를 이용할 때 추가적인 데이터가 필요할 수 있습니다. 이를 위해 기본 API들에는 추가 데이터를 넘겨줄 수 있는 payload라는 매개변수가 포함되어 있습니다. 이 payload에 필요한 데이터를 패킷에 담아 목록 형식으로 저장할 수 있습니다. 여기에 추가 데이터를 넣어 서버로 보내거나, 서버에서 보낸 메시지를 꺼낼 수 있습니다. 

```
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

게임 플레이 종료 전 Connector.CloseSoket() 함수를 호출해 연결을 종료하는 것이 좋습니다. 종료하지 않으면 서버에서 클라이언트의 종료를 인지하지 못할 수 있으며, 그럴 경우 불필요한 동작을 계속할 수 있습니다. 

## 백그라운드 접속 끊김 방지

## 재접속