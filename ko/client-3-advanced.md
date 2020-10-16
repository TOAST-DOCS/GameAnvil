# 고급 개념



## Connection Recovery 지원

무선 네트워크환경에서는 언제든 내트워크 연결이 끊어질 수 있습니다. 이때 매번 게임이 종료되고 처음부터 다시 시작해야 합니다면 게임 유저에게는 엄청난 불편을 안겨주게됩니다. GameAnvil은 네트워크 연결이 끊기더라도 서버에 해당 유저의 정보를 유지하고 있어서 재접속을 통해 기존의 상태를 그대로 이어서 플레이 하는것이 가능하다. 재접속후 로그인을 하면 OnLogin 콜백의 LoginInfo 파라메터를 통해 재접속 여부, 방 입장 여부, 추가정보 등을 확인 할 수 있으며 이를 이용해 이어서 플레이를 하도록 구현할 수 있습니다.



## One Connection - Multi User를 지원합니다.

GameAnvil은 하나의 Connection으로 여러 UserAgent를 동시에 사용할 수 있습니다. 각각의 UserAgent를 독립적으로 사용할 수 있기 때문에 다양한 방법으로 활용할 수 있습니다. 예를 들자면 게임을 위한 UserAgent와 채팅을 위한 UserAgent를 나눠서 처리할 수도 있으며, 게임 모드별로 UserAgent를 만들어 처리할 수도 있습니다. 같은 게임모드에 여러개의 UserAgent를 만드는것도 가능합니다.



## 커스텀 메시지

GameAnvil은 기본 메시지의 프로토콜로 [ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/overview)를 사용하고 있지만, 필요하다면 사용자가 원하는 임의의 데이터를 바이트 스트림으로 직렬화해 전송할 수 있습니다. 각 데이터 전송시 고유의 Id를 부여하여 어떤 목적의 데이터인지 구분하여 처리할 수도 있습니다. 



## SingleServer

실제 GameAnvil Server없이도 클라이언트와 패킷을 주고 받을 수 있는 환경을 만들어줍니다.  다음과 같은 경우 SingleServer를 이용하면 실제 서버와 통신하는 것 처럼 사용할 수 있습니다.

- 서버가 준비되지 않은 경우
- 테스트를 위한 mock 서버가 필요할 경우
- 기존 서버와 함께 새로 추가되는 프로토콜을 이용한 테스트가 필요한 경우
  (기존의 프로토콜은 실제 서버에서, 추가되는 프로토콜은 SingleServer에서 처리)

SingleServer를 사용하기 위해서는 SingleServer.dll 파일을 프로젝트에 포함시켜야 합니다. DLL 파일은 [여기](https://github.nhnent.com/game-server-engine/GameAnvil-connector-charp/tree/master/dll)에서 구할 수있습니다. 



### 시작

```C#
// class SingleServer

// SingleServer를 초기화 한다.
public static void Initialize(Connector connector)

// SingleServer에서 사용할 ServiceName과 ServiceId 쌍을 등록한다.
public static void AddServiceId(string serviceName, int serviceId)
    
// SingleServer의 응답에 딜레이를 지정한다.
// delay : 딜레이 시간(초)
public static void SetDelay(float delay)

// SingleServer Connect시 Authenticate까지 자동으로 처리할지 여부를 설정한다.
public static void SetAutoAuth(bool autoAuth, string deviceId = "", string accountId = "", string password = "", Tardis.Payload authPaylod = null)
```

GameAnvil Connector 와의 주고받는 메세지의 처리를 위해 Update() 함수를 주기적으로 호출해야 합니다. Update() 함수의 호출 주기는 자유롭게 설정해도 무방하나 호출하지 않을 경우 GameAnvil Connector로부터 메세지를 받더라도 이에대한 알림을 받을 수 없습니다.

```C#
SingleServer.Update();
```

SingleServer를 시작하기위해서는 SingleServer.Connect()를 호출해야 합니다.  Connect() 호출시 ip/port를 지정하지 않을 경우 SingleServer에 설정한 응답만 받게됩니다. GameAnvil Server의 ip/port를 지정하면 해당 ip/port로 접속하여 SingleServer에 설정하지 않은 응답은 지정된 GameAnvil Server로 요청을 보내고 서버로 부터 받은 응답을 TardisConnector로 전달하게됩니다. 

```C#
SingleServer.Connect("127.0.0.1", 11200);
```



### 기본 응답 설정

GameAnvil Connector의 기본 기능중 어느 기능에 응답할 것인지 설정할 수 있습니다. 설정하지 않은 기능의 경우 무시합니다.

```c#
public static void SetOnAuthenticate(DelOnAuthenticate onAuthenticate);
public static void SetOnGetChannelInfo(DelOnGetChannelInfo onGetChannelInfo);
public static void SetOnGetChannelList(DelOnGetChannelList onGetChannelList);

public static void SetOnLogin(DelOnLogin onLogin);
public static void SetOnLogout(DelOnLogout onLogout);
public static void SetOnCreateRoom(DelOnCreateRoom onCreateRoom);
public static void SetOnJoinRoom(DelOnJoinRoom onJoinRoom);
public static void SetOnMatchRoom(DelOnMatchRoom onMatchRoom);
public static void SetOnMatchUserStart(DelOnMatchUserStart onMatchUserStart);
public static void SetOnMatchUserCancel(DelOnMatchUserCancel onMatchUserCancel);
public static void SetOnMatchPartyStart(DelOnMatchPartyStart onMatchPartyStart);
public static void SetOnMatchPartyCancel(DelOnMatchPartyCancel onMatchPartyCancel);
public static void SetOnLeaveRoom(DelOnLeaveRoom onLeaveRoom);
public static void SetOnMoveChannel(DelOnMoveChannel onMoveChannel);
public static void SetOnMultiRequest(DelOnMultiRequest onMultiRequset);
public static void SetOnMultiSend(DelOnMultiSend onMultiSend);
public static void SetOnNamedRoom(DelOnNamedRoom onNamedRoom);
public static void SetOnSnapshot(DelOnSnapshot onSnapshot);
```

```
// example

SingleServer.SetOnLogin((req, res) =>
{
	if (msgReq.userType != Constants.UserType)
    	throw new SingleServerException(Defines.ErrorCode.TIMEOUT);
    return Defines.ResultCodeLogin.LOGIN_SUCCESS;
});

SingleServer.SetOnLogout((req, res) =>
{
	return Defines.ResultCodeLogout.LOGOUT_SUCCESS;
});
```



### 응답 설정

클라이언트에서 보낸 사용자 정의 메시지 중 어떤 메시지에 응답할 것인지 설정할 수 있습니다. 설정하지 않은 메시지의 경우 무시합니다.

```c#
// class SingleServer

// 클라이언트로부터 Request()함수를 통해 Rcv 패킷을 받으면 응답으로 Snd 패킷을 보낸다.
public static void SetResponse<Rcv, Snd>()

// 클라이언트로부터 Send()함수를 통해 Rcv 패킷을 받으면 응답으로 Snd 패킷을 보낸다.
public static void SetEcho<Rcv, Snd>() 

// 클라이언트로부터 Request()함수를 통해 Rcv 패킷을 받으면 응답으로 Snd 패킷을 보낸다.
// DelResponseMessage를 통해 응답 패킷의 값을 설정할 수 있다.
//		rcv : 클라이언트로부터 받은 요청 패킷. 패킷에 설정된 값을 확인할 수 있다.
//		snd : 클라이언트에 보낼 응답 패킷. 패킷에 값을 설정할 수 있다.
//		return : true - 응답 패킷 전송한다. false - 응답 프로토콜을 전송하지 않는다.
public delegate bool DelResponseMessage<Rcv, Snd>(Rcv rcv, Snd snd) where Rcv : IMessage where Snd : IMessage;
public static void SetResponse<Rcv, Snd>(Action.DelResponseMessage<Rcv, Snd> del) 

// 클라이언트로부터 Send()함수를 통해 Rcv 패킷을 받으면 응답으로 Snd 패킷을 보낸다.
// DelSendMessage 통해 응답 패킷의 값을 설정할 수 있다.
//		rcv : 클라이언트로부터 받은 요청 패킷. 패킷에 설정된 값을 확인할 수 있다.
//		snd : 클라이언트에 보낼 응답 패킷. 패킷에 값을 설정할 수 있다.
//		return : true - 응답 패킷 전송한다. false - 응답 프로토콜을 전송하지 않는다.
public delegate bool DelSendMessage<Rcv, Snd>(Rcv rcv, Snd snd) where Rcv : IMessage where Snd : IMessage;
public static void SetEcho<Rcv, Snd>(Action.DelSendMessage<Rcv, Snd> del)
```

```c#
// example
SingleServer.SetEcho((RPSGameProto.RPSGameStartSend req, RPSGameProto.RPSGameStartReceive res) =>
{
	if (!isMaster)
		return false;
	
	if (CurrentUser < 2)
		return false;

	return true;
});
```



### State

State를 생성하고 관리할 수 있습니다. 

- State는 하위 State를 가질 수 있다.
- "Root"라는 이름의 최상위 State를 기본으로 가지고 있으며 SingleServer에서 직접 관리한다. 
- State별로 응답 설정을 할 수 있으며 설정하지 않은 경우 상위 State에 처리를 위임한다. 

```c#
// class SingleServer

// 새로운 ServerState를 생성한다. 생성된 ServerState는 자동으로 추가된다. 
// 	name : 생성할 state의 이름.
//	return : 생성된 ServerStgate. 이름이 이미 등록되어있는 경우 등록된 ServerState를 리턴한다.
static public ServerState CreateState(string name)
    
// 새로운 ServerState를 추가한다. 이름이 이미 등록되어있는 경우 무시한다.
//	state : 새로운 ServerState나 ServerState를 상속 받은 객체.
static public void AddState(ServerState state)
    
// 등록되어있는 ServerState를 얻어온다.
//	name : 얻어올 ServerState의 이름.
//	return : 이름이 name인 ServerState. 등록된 이름의 ServerState가 없을 경우 null을 리턴한다.
static public ServerState GetState(string name)
    
// 등록되어있는 ServerState를 제거한다.
//	name : 제거할 ServerState의 이름.
//	return : 제거된 ServerState. 등록된 이름의 ServerState가 없을 경우 null을 리턴한다.
public ServerState RemoveState(string name)
    
// 등록되어있는 ServerState를 모두 제거한다.
static public void ClearState()
```

```c#
// class ServerState

// 새로운 ServerState를 생성한다. 생성된 ServerState는 자동으로 하위State로 추가된다. 
// 	name : 생성할 state의 이름. 
//	return : 생성된 ServerStgate. 이름이 이미 등록되어있는 경우 등록된 ServerState를 리턴한다.
static public ServerSubState CreateSubState(string name)
    
// 새로운 ServerState를 하위State로 추가한다. 이름이 이미 등록되어있는 경우 무시한다.
//	state : 새로운 ServerState나 ServerState를 상속 받은 객체.
static public void AddSubState(ServerState state)
    
// 등록되어있는 ServerState를 얻어온다.
//	name : 얻어올 ServerState의 이름. 
//	return : 이름이 name인 ServerState. 등록된 이름의 ServerState가 없을 경우 null을 리턴한다.
static public ServerState GetSubState(string name)

// 등록되어있는 ServerState를 제거한다.
//	name : 제거할 ServerState의 이름.
//	return : 제거된 ServerState. 등록된 이름의 ServerState가 없을 경우 null을 리턴한다.
public ServerState RemoveSubState(string name)
    
// 등록되어있는 ServerState를 모두 제거한다.
static public void ClearSubState()
    
// 클라이언트로부터 Request()함수를 통해 Rcv 패킷을 받으면 응답으로 Snd 패킷을 보낸다.
public void SetResponse<Rcv, Snd>()

// 클라이언트로부터 Send()함수를 통해 Rcv 패킷을 받으면 응답으로 Snd 패킷을 보낸다.
public void SetEcho<Rcv, Snd>() 

// 클라이언트로부터 Request()함수를 통해 Rcv 패킷을 받으면 응답으로 Snd 패킷을 보낸다.
// DelResponseMessage를 통해 응답 패킷의 값을 설정할 수 있다.
//		rcv : 클라이언트로부터 받은 요청 패킷. 패킷에 설정된 값을 확인할 수 있다.
//		snd : 클라이언트에 보낼 응답 패킷. 패킷에 값을 설정할 수 있다.
//		return : true - 응답 패킷 전송한다. false - 응답 프로토콜을 전송하지 않는다.
public delegate bool DelResponseMessage<Rcv, Snd>(Rcv rcv, Snd snd) where Rcv : IMessage where Snd : IMessage;
public void SetResponse<Rcv, Snd>(Action.DelResponseMessage<Rcv, Snd> del) 

// 클라이언트로부터 Send()함수를 통해 Rcv 패킷을 받으면 응답으로 Snd 패킷을 보낸다.
// DelSendMessage 통해 응답 패킷의 값을 설정할 수 있다.
//		rcv : 클라이언트로부터 받은 요청 패킷. 패킷에 설정된 값을 확인할 수 있다.
//		snd : 클라이언트에 보낼 응답 패킷. 패킷에 값을 설정할 수 있다.
//		return : true - 응답 패킷 전송한다. false - 응답 프로토콜을 전송하지 않는다.
public delegate bool DelSendMessage<Rcv, Snd>(Rcv rcv, Snd snd) where Rcv : IMessage where Snd : IMessage;
public void SetEcho<Rcv, Snd>(Action.DelSendMessage<Rcv, Snd> del)
```

```c#
// example

ServerState stateAuth = SingleServer.CreateState("Auth");
stateAuth.SetOnLogin((req, res) =>
{
    UserId = req.Id;
    res.Id = UserId;
    stateAuth.ChangeState("Lobby");
    return true;
});

ServerState stateLobby = SingleServer.CreateState("Lobby");
stateLobby.SetOnLogout((req, res) =>
{
	res.Id = UserId;
	UserId = "";
	stateLobby.ChangeState("Auth");
	return true;
});
```