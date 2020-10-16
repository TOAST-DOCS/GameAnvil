# 기본 개념



## Connector

GameAnvil Connector를 사용하기 위한 기본 모듈입니다. 기본 설정과 Agent의 관리를 담당합니다. 또 내부 동작과 관련된 로그를 볼 수 있도록 delegate를 등록할 수 있습니다.

### 초기화

Connector를 사용하기 위해서는 Connector객체를 생성하면 됩니다.

```c#
Connector connector = new Connector();
```

또 서버와의 주고받는 메세지의 처리를 위해 Update() 함수를 주기적으로 호출해줍니다. Update()함수의 호출 주기는 자유롭게 설정해도 무방하나 호출하지 않을 경우 서버로부터 메세지를 받더라도 이에대한 알림을 받을 수 없습니다. 

```c#
connector.Update();
```

이제 GameAnvil Connector의 사용 준비가 완료되었습니다.

### 설정

Connector의 설정은 모두 기본값을 가지고 있으므로 별도로 설정하지 않아도 사용하는데 문제가 없습니다. 또 프로젝트에서 필요한 경우 간단하게 변경할 수 있습니다. 설정할 수 있는 값들은 다음과 같습니다.

```c#
// 서버와의 연결을 확인하기 위한 ping 주기 설정(단위 : sec, 0일 경우 사용 안함, 기본값 3)
connector.config.pingInterval = 3;

// 패킷이 지정된 시간안에 업데이트 되지 않으면, disconnect 되었다고 판단됩니다.
// pingInterval 보다 높게 설정되야 합니다.
connector.config.packetTimeout = 5;

// TimeOut 기본 대기시간 설정(단위 : sec, 기본값 15)
connector.config.defaultReqestTimeout = 15;

// 접속시 IPv6 주소로 변환 여부. (기본값 : false)
connector.config.useIPv6 = false;

// 소켓의 Nodelay 사용 여부. (기본값 : true)
connector.config.useSocketNoDelay = true;
```

### Agent 관리

GameAnvil Connector가 제공하는 기능의 대부분은 Agent를 통해 제공하며, 이 Agent의 생성및 관리는 Connector에서 담당합니다. 
ConnectionAgent는 Connector가 초기화 될 때 자동으로 생성되며 GetConnectionAgent() 함수를 이용해 얻을 수 있습니다. 

```c#
ConnectionAgent connection = connector.GetConnectionAgent();
```

UserAgent를 사용하려면 CreateUserAgent()함수를 이용해 생성해야 합니다. 생성된 UserAgent는 Connector내부에서 관리되며 GetUserAgent() 함수를 이용해 다시 얻을 수 있습니다.

```c#
// serviceId    : UserAgent가 사용할 serviceId. 서버 구현에 따라 사용하지 않을 수 있음.
// subId        : 서비스별 UserAgent식별 할 수 있는 고유 ID. 서버 구현에 따라 사용하지 않을 수 있음.
UserAgent user = connector.CreateUserAgent(serviceId, subId);
UserAgent user = connector.GetUserAgent(serviceId, subId);
```

### 로그

GameAnvil Connector의 내부 동작에 대한 로그를 받기 위해 콜백을 등록할 수 있습니다.

```c#
connector.logger += logging;
connector.lvNetLogger += lvNetLogging;

//...

public void logging(GameAnvil.Defines.Enums.LogLevel lv, String log)
{
    Console.Out.WriteLine("{0}:{1}", lv.ToString(), log);
}

public void lvNetLogging(GameAnvil.Defines.Enums.NetLogLevel lv, String log)
{
    Console.Out.WriteLine("{0}:{1}", lv.ToString(), log);
}
```

### 종료

게임 플레이 종료전 CloseSoket() 함수를 호출해야 합니다.  특히 UnityEditor의 경우 CloseSoket()을 호출하지 않고 종료를 할 경우 다음 실행시  UnityEditor가 응답을 하지 않아 강제종료를 해야합니다. 플레이 종료전에 반드시 CloseSoket()가 호출되도록 해주세요.




## ConnectionAgent

GameAnvil Server의 Connection Node와 관련된 작업을 담당합니다. 접속(Connect()), 인증(Authentication()) 등 기본 세션 관리 기능 및 채널 목록 등을 제공하며, 서버 구현에 따라 채널 정보를 제공하거나 사용자 정의 메시지를 주고받을 수 있습니다. ConnectionAgent는 Connector가 초기화 될 때 자동으로 생성되며 GetConnectionAgent() 함수를 이용해 얻을 수 있습니다.

```c#
ConnectionAgent connection = connector.GetConnectionAgent();
```

### ConnectionAgent의 기본 기능 사용

ConnectionAgent가 제공하는 기본 기능은 모두 비동기 함수를 통해 제공됩니다. 그리고 각 기능의 결과를 얻기 위해 크게 나눠 2가지 방식을 제공하고 있습니다.  첫째는 listener나 delegate를 미리 등록해 놓고 이를 통해 결과에 대한 알림을 받는 방법이고, 나머지는 기능을 요청 할 때 알림을 받을 delegate를 같이 전달하는 방법입니다. 두 방법을 같이 사용하는 것도 가능하고 목적에 따라 listener나 delegate를 여러개를 등록하여 사용할 수도 있습니다. 한번 등록한 listener나 delegate는 등록을 해지하기 전까지 모두 알림을 받습니다.

#### ConnectionAgent의 기본 기능

##### Connect

```c#
/// GameAnvil Server에 연결한다.
/// <param name="ip">ip address</param>
/// <param name="port">port</param>
/// <param name="tardisSocket">연결에 사용할 ITardisSocket 객체.(TardisTCPSecureSocket, TardisWebSocket)</param>
/// <param name="onConnect">결과를 전달 받을 delegate</param>
public void Connect(string ip, int port);
public void Connect(string ip, int port, Interface.DelconnectionOnConnect onConnect);
public void Connect(string ip, int port, Socket.ITardisSocket tardisSocket);
public void Connect(string ip, int port, Socket.ITardisSocket tardisSocket, Interface.DelconnectionOnConnect onConnect);
    
/// Connect의 결과
/// <param name="connectionAgent">ConnectionAgent</param>
/// <param name="result">Connect의 결과</param>
public delegate void DelConnectionOnConnect(ConnectionAgent connectionAgent, Defines.ResultCodeConnect result);

/// Connect 결과 코드.
public enum ResultCodeConnect
{
    /// 성공.
    CONNECT_SUCCESS = 0,
    /// 실패.
    CONNECT_FAIL = 1,
    /// 이미 연결되어있음.
    CONNECT_ALREADY_CONNECTED = 2,
    /// 이미 연결 시도중임.
    CONNECT_ALREADY_REQUEST = 3,
}
```

##### Authenticate

```c#
/// GameAnvil Server에 인증을 요청 한다.
/// 인증에 성공해야 GameAnvil Connector의 다른 기능을 사용할 수 있다.
/// <param name="deviceId">사용자 기기를 식별 할 수 있는 고유 ID.서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="accountId">사용자 계정을 식별 할 수 있는 고유 ID.</param>
/// <param name="passwd">사용자 계정의 password. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="inPayload"></param>
/// <param name="onAuthentication">결과를 전달 받을 delegate</param>
public void Authenticate(string deviceId, string accountId, string passwd);
public void Authenticate(string deviceId, string accountId, string passwd, Payload inPayload);
public void Authenticate(string deviceId, string accountId, string passwd, Interface.DelconnectionOnAuthentication onAuthentication);
public void Authenticate(string deviceId, string accountId, string passwd, Payload inPayload, Interface.DelconnectionOnAuthentication onAuthentication);

/// Authentication의 결과
/// <param name="connectionAgent">ConnectionAgent</param>
/// <param name="result">Authentication의 결과</param>
/// <param name="loginedUserInfoList">서버에 남아있는 로그인 정보 목록</param>
/// <param name="message">서버로부터 받은 Message. 인증 실패 이유 등.</param>
/// <param name="payload">추가 정보</param>
public delegate void DelConnectionOnAuthentication(ConnectionAgent connectionAgent, Defines.ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload);

/// Authenticate 결과 코드.
public enum ResultCodeAuth
{
	/// 성공.
    AUTH_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    AUTH_FAIL_CONTENT = 201,
    /// 실패. 점검중.
    AUTH_FAIL_MAINTENANCE = 202,
    /// 실패. 잘못된 AccountId.
    AUTH_FAIL_INVALID_ACCOUNT_ID = 203
}
```

##### GetChannelList

```c#
/// 특정 서비스에서 사용 가능한 채널 목록을 요청한다. 
/// <param name="serviceCode">채널 목록을 요청할 service Code.</param>
/// <param name="onChannelList">결과를 전달 받을 delegate.</param>
public void GetChannelList(int serviceCode);
public void GetChannelList(int serviceCode, Interface.DelconnectionOnChannelList onChannelList);

/// GetChannelList의 결과
/// <param name="connectionAgent">ConnectionAgent</param>
/// <param name="result">GetChannelList의 결과</param>
/// <param name="channelIdList">체널 목록</param>
public delegate void DelConnectionOnChannelList(ConnectionAgent connectionAgent, Defines.ResultCodeChannelList result, List<string> channelIdList);

/// GetChannelList 결과 코드.
public enum ResultCodeChannelList
{
	/// 성공.
    CHANNEL_LIST_SUCCESS = 0,
    /// 실패. 잘못된 서비스 아이디.
    CHANNEL_LIST_FAIL_INVALID_SERVICEID = 1801
}
```

##### GetChannelInfo

```c#
/// 특정 채널의 정보를 요청한다.<br></br>서버에서 지원할 경우 사용할 수 있다.
/// <param name="serviceCode">채널 정보를 요청할 service Code.</param>
/// <param name="channelId">채널 정보를 요청할 channel의 Id.</param>
/// <param name="onChannelInfo">결과를 전달 받을 delegate.</param>
public void GetChannelInfo(int serviceCode, string channelId);
public void GetChannelInfo(int serviceCode, string channelId, Interface.DelconnectionOnChannelInfo onChannelInfo);

/// GetChannelInfo의 결과
/// <param name="connectionAgent">ConnectionAgent</param>
/// <param name="result">GetChannelInfo의 결과</param>
/// <param name="channelInfoList">체널 정보 목록</param>
public delegate void DelConnectionOnChannelInfo(ConnectionAgent connectionAgent, Defines.ResultCodeChannelInfo result, List<Payload> channelInfoList);

/// GetChannelInfo 결과 코드.
public enum ResultCodeChannelInfo
{
	/// 성공.
    CHANNEL_INFO_SUCCESS = 0,
    /// 실패. 잘못된 서비스 아이디.
    CHANNEL_INFO_FAIL_INVALID_SERVICEID = 1901
}
```

##### Disconnect

```c#
/// GameAnvil Server와의 연결을 해제한다.
/// <param name="onDisconnect">결과를 전달 받을 delegate</param>
public void Disconnect();
public void Disconnect(Interface.DelconnectionOnDisconnect onDisconnect);

/// Disconnect의 결과 또는 또는 강제 접속종료 알림
/// <param name="connectionAgent">ConnectionAgent</param>
/// <param name="result">Disconnect의 결과 또는 또는 강제 접속종료 알림</param>
/// <param name="force">강제 종료 여부</param>
/// <param name="payload">추가 정보</param>
public delegate void DelConnectionOnDisconnect(ConnectionAgent connectionAgent, Defines.ResultCodeDisconnect result, bool force, Payload payload);

/// Disconnect가 된 이유
public enum ResultCodeDisconnect
{
	/// 시스템 오류로 인한 강제 종료. 
    /// 일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람.
    FORCE_CLOSE_SYSTEM_ERROR = 2000,
    
    /// 서버에서 BaseConnection의 close() 호출
    FORCE_CLOSE_BASE_CONNECTION = 2010,
    /// 서버에서 BaseUser의 closeConnection() 호출
    FORCE_CLOSE_BASE_USER = 2011,
    /// Admin에서 강제 종료.
    FORCE_CLOSE_ADMIN_KICK = 2012,
    
    /// 서버 점검으로 인한 강제 종료.
    FORCE_CLOSE_MAINTENANCE = 2020,
    /// GameNode가 invalid 상태로 변경된 경우.
    FORCE_CLOSE_INVALID_NODE = 2021,
    /// 유저 트렌스퍼가 실패한 경우.
    FORCE_CLOSE_USER_TRANSFER_FAIL = 2022,
    /// 유저 트렌스퍼중 시스템 에러가 발생한 경우.
    FORCE_CLOSE_USER_TRANSFER_ERROR = 2023,
    
    /// 인증 실패로 인한 강제 종료.
    FORCE_CLOSE_AUTHENTICATION_FAIL = 2030,
    /// 중복접속으로 인한 강제 종료.
    FORCE_CLOSE_DUPLICATE_LOGIN = 2031,

    /// 같은 계정 정보로 새로운 로그인 요청이 들어온 경우. 
    /// 네트워크 순단 등으로 재접속을 하면 이전의 접속을 종료하면서 이 코드를 사용한다. 
    /// 일반적으로 클라이언트에서 이 코드를 받을 일은 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람.
    FORCE_CLOSE_BY_NEW_CONNECTION = 2040,
    /// 세션 정보를 찾을 수 없는 경우.
    /// 일반적으로 클라이언트에서 이 코드를 받을 일은 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람.
    FORCE_CLOSE_DISCONNECT_ALARM = 2041,
    /// 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우.
    /// 일반적으로 클라이언트에서 이 코드를 받을 일은 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람.
    FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL = 2042,
    /// 고스트 유저인 경우.
    /// 일반적으로 클라이언트에서 이 코드를 받을 일은 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람.
    FORCE_CLOSE_GHOST_USER = 2043,
    
    /// 네트워크 연결이 끊어짐
    SOCKET_DISCONNECT = 2100,
    /// 타임아웃이 발생, 컨넥터에서 연결을 끊음
    SOCKET_TIME_OUT = 2101,
    /// 소켓 에러가 발생하여 연결을 끊음
    SOCKET_ERROR = 2102,
}

```

#### IConnectionListener 등록

IConnectionListener 인터페이스를 구현해 등록하고 이를 통해 각 기능의 결과에 대한 알림을 받을 수 있습니다. 

```
IConnectionListener {
	/// <summary>
    /// Connect의 결과
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="result">Connect의 결과</param>
    void OnConnect(ConnectionAgent connectionAgent, Defines.ResultCodeConnect result);
    
    /// <summary>
    /// Authentication의 결과
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="result">Authentication의 결과</param>
    /// <param name="loginedUserInfoList">서버에 남아있는 로그인 정보 목록</param>
    /// <param name="message">서버로부터 받은 Message. 인증 실패 이유 등.</param>
    /// <param name="payload">추가 정보</param>
    void OnAuthentication(ConnectionAgent connectionAgent, Defines.ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload); 
    
    /// <summary>
    /// GetChannelList의 결과
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="result">GetChannelList의 결과</param>
    /// <param name="channelIdList">체널 목록</param>
    void OnChannelList(ConnectionAgent connectionAgent, Defines.ResultCodeChannelList result, List<string> channelIdList);
    
    /// <summary>
    /// GetChannelInfo의 결과
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="result">GetChannelInfo의 결과</param>
    /// <param name="channelInfoList">체널 정보 목록</param>
    void OnChannelInfo(ConnectionAgent connectionAgent, Defines.ResultCodeChannelInfo result, List<Payload> channelInfoList); 
        
	/// <summary>
    /// Disconnect의 결과 또는 또는 강제 접속종료 알림
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="result">Disconnect의 결과 또는 또는 강제 접속종료 알림</param>
    /// <param name="force">강제 종료 여부</param>
    /// <param name="payload">추가 정보</param>
    void OnDisconnect(ConnectionAgent connectionAgent, Defines.ResultCodeDisconnect result, bool force, Payload payload); 
    
    /// <summary>
    /// 기본 기능 사용중 Error발생
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="errorCode">에러 코드</param>
    /// <param name="commands">에러가 발생한 기능</param>
    void OnError(ConnectionAgent connectionAgent, Defines.ErrorCode errorCode, Connection.Defines.Commands commands); 
    
    /// <summary>
    /// Request, Send 전송후 Error발생
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="errorCode">에러 코드</param>
    /// <param name="command">에러가 발생한 패킷의 MessageName</param>
    void OnError(ConnectionAgent connectionAgent, Defines.ErrorCode errorCode, string command);
}
```

#### Delegate를 등록하는 방법

ConnectionAgent에 정의되어 있는 각 기능별 delegate에 직접 등록하여 알림을 받을 수 있습니다.

```c#
ConnectionAgent connection = Connector.GetConnectionAgent();
connection.onConnectListeners += OnConnectListeners;
connection.onReconnectStartListeners += OnReconnectStartListeners;
connection.onReconnectFinishListeners += OnReconnectFinishListeners;
connection.onAuthenticationListeners += OnAuthenticationListeners;
connection.onChannelListListeners += OnChannelListListeners;
connection.onChannelInfoListeners += OnChannelInfoListeners;
connection.onDisconnectListeners += OnDisconnectListeners;
connection.onErrorCommandListeners += OnErrorCommandListeners;
connection.onErrorCustomCommandListeners += OnErrorCustomCommandListeners;
```



## UserAgent

GameAnvil Server의 Game Node와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 Room 관리 등 기본 기능을 제공하며, 서버 구현에 따라 사용자 정의 기능을 추가적으로 제공할 수도 있습니다. UserAgent를 사용하기 위해서는 Connector.CreateUserAgent() 함수를 이용해 새로운 UserAgent를 생성해야 합니다. ServiceId 와 SubId로 구분되는 여러개의 UserAgent를 생성할 수 있으며 생성된 각 UserAgent는 독립적으로 사용할 수 있습니다. 생성된 UserAgent는 Connector 에서 내부적으로 관리되며 Connector.GetUser()함수를 이용해 다시 사용할 수 있습니다.

```c#
// serviceId    : UserAgent가 사용할 serviceId. 서버 구현에 따라 사용하지 않을 수 있음.
// subId        : 서비스별 UserAgent식별 할 수 있는 고유 ID. 서버 구현에 따라 사용하지 않을 수 있음.
GameAnvil.UserAgent user = connector.CreateUserAgent(serviceId, subId);
user = connector.GetUser(serviceId, subId);
```

### UserAgent의 기본 기능 사용

UserAgent가 제공하는 기본 기능은 모두 비동기 함수를 통해 제공됩니다. 그리고 각 기능의 결과를 얻기 위해 크게 나눠 2가지 방식을 제공하고 있습니다. 첫째는 listener나 delegate를 미리 등록해 놓고 이를 통해 결과에 대한 알림을 받는 방법이고 나머지는 기능을 요청 할 때 알림을 받을 delegate를 같이 전달하는 방법입니다. 두가지 방법을 같이 사용하는 것도 가능하고 목적에 따라 listener나 delegate를 여러개 등록하여 사용할 수도 있습니다. 

#### UserAgent의 기본 기능

##### Login

```c#
/// 지정한 채널에 로그인한다.
/// <param name="userType">User의 타입. </param>
/// <param name="payload">버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="channelId">로그인할 channel의 id. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onLogin">결과를 전달 받을 delegate</param>
public void Login(string userType);
public void Login(string userType, User.Interface.DelUserOnLogin onLogin);
public void Login(string userType, string channelId, Payload payload);
public void Login(string userType, string channelId, Payload payload, User.Interface.DelUserOnLogin onLogin);

/// Login의 결과.
/// <param name="userAgent">Login을 요청한 UserAgent객체</param>
/// <param name="result">Login결과</param>
/// <param name="loginInfo">Login 정보</param>
public delegate void DelUserOnLogin(UserAgent userAgent, Defines.ResultCodeLogin result, UserAgent.LoginInfo loginInfo);

/// 로그인 결과 코드.
public enum ResultCodeLogin
{
    /// 성공.
    LOGIN_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    LOGIN_FAIL_CONTENT = 301,
    /// 실패. 노드가 존재하지 않음.
    LOGIN_FAIL_NOT_EXIST_NODE = 302,
    /// 실패. 점검중.
    LOGIN_FAIL_MAINTENANCE = 303,
    /// 실패. 게임서버가 응답하지 않음.
    LOGIN_FAIL_TIMEOUT_GAME_SERVER = 304,

    /// 실패. 잘못된 서비스 아이디.
    LOGIN_FAIL_INVALID_SERVICEID = 310,
    /// 실패. 잘못된 유저 타입.
    LOGIN_FAIL_INVALID_USERTYPE = 311,
    /// 실패. 잘못된 유저 아이디.
    LOGIN_FAIL_INVALID_USERID = 312,
    /// 실패. 잘못된 SubId.
    LOGIN_FAIL_INVALID_SUB_ID = 313,

    /// 실패. 다른 서비스에 로그인 되어있음.
    LOGIN_FAIL_LOGINED_OTHER_SERVICE = 320,
    /// 실패. 다른 채널에 로그인 되어있음.
    LOGIN_FAIL_LOGINED_OTHER_CHANNEL = 321,
    /// 실패. 동일 Account 아이디로 다른 UserType이 로그인 되어있음.
    LOGIN_FAIL_LOGINED_OTHER_USER_TYPE = 322,
    /// 실패. 동일 Account 아이디로 다른 DeviceId가 로그인 되어있음.
    LOGIN_FAIL_LOGINED_OTHER_DEVICE = 323
}
    
```



##### Logout

```c#
/// 현재 채널에서 로그아웃 한다.
/// <param name="onLogout">로그아웃 콜백.</param>
public void Logout();
public void Logout(User.Interface.DelUserOnLogout onLogout);

/// Logout 결과 코드.
public enum ResultCodeLogout
{
    /// 성공.
    LOGOUT_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    LOGOUT_FAIL_CONTENT = 401
}
```



##### CreateRoom

```c#
/// 임의의 방을 생성하고 해당 방에 입장한다.
/// <param name="roomType">생성할 방의 roomType.</param>
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onCreateRoom">결과를 전달 받을 delegate.</param>
public void CreateRoom(string roomType);
public void CreateRoom(string roomType, Payload payload);
public void CreateRoom(string roomType, Interface.DelUserOnCreateRoom onCreateRoom);
public void CreateRoom(string roomType, Payload payload, Interface.DelUserOnCreateRoom onCreateRoom);

/// CreateRoom의 결과
/// <param name="userAgent">CreateRoom을 요청한 UserAgent 객체</param>
/// <param name="result">CreateRoom 결과</param>
/// <param name="roomName">생성된 방 이름</param>
/// <param name="roomId">생성된 방의 RoomId</param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnCreateRoom(UserAgent userAgent, Defines.ResultCodeCreateRoom result, int roomId, string roomName, Payload payload);

/// CreateRoom 결과 코드.
public enum ResultCodeCreateRoom
{
    /// 성공.
    CREATE_ROOM_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    CREATE_ROOM_FAIL_CONTENT = 601,
    /// 실패. 이미 룸에 들어가 있음.
    CREATE_ROOM_FAIL_ALREADY_JOINED_ROOM = 602,
}
```



##### JoinRoom

```c#
/// 지정한 id의 방에 입장한다. 지정한 id의 방이 없을 경우 실패한다.
/// <param name="roomType">입장할 방의 roomType.</param>
/// <param name="roomId">입장하고자 하는 방의 id.</param>
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onJoinRoom">결과를 전달 받을 delegate.</param>
public void JoinRoom(string roomType, string roomId);
public void JoinRoom(string roomType, string roomId, Payload payload);
public void JoinRoom(string roomType, string roomId, Payload payload, User.Interface.DelUserOnJoinRoom onJoinRoom);

/// JoinRoom의 결과
/// <param name="userAgent">JoinRoom을 요청한 UserAgent 객체</param>
/// <param name="result">JoinRoom 결과</param>
/// <param name="roomId">입장한 방의 RoomId</param>
/// <param name="roomName"></param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnJoinRoom(UserAgent userAgent, Defines.ResultCodeJoinRoom result, int roomId, string roomName, Payload payload);

/// JoinRoom 결과 코드.
public enum ResultCodeJoinRoom
{
	/// 성공.
    JOIN_ROOM_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    JOIN_ROOM_FAIL_CONTENT = 701,
    /// 실패. 방이 존재하지 않음.
    JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST = 702,
    /// 실패. 이미 방에 들어가 있음.
    JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM = 703
}
```



##### LeaveRoom

```c#
/// 현재 방에서 퇴장한다.
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onLeaveRoom">결과를 전달 받을 delegate.</param>
public void LeaveRoom();
public void LeaveRoom(Payload payload);
public void LeaveRoom(Interface.DelUserOnLeaveRoom onLeaveRoom);
public void LeaveRoom(Payload payload, Interface.DelUserOnLeaveRoom onLeaveRoom);

/// LeaveRoom의 결과
/// <param name="userAgent">방에서 퇴장한 UserAgent 객체</param>
/// <param name="result">LeaveRoom의 결과</param>
/// <param name="force">강퇴 여부</param>
/// <param name="roomId">퇴장한 방의 RoomId</param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnLeaveRoom(UserAgent userAgent, Defines.ResultCodeLeaveRoom result, bool force, int roomId, Payload payload);

/// LeaveRoom 결과 코드.
public enum ResultCodeLeaveRoom
{
	/// 성공.
    LEAVE_ROOM_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    LEAVE_ROOM_FAIL_CONTENT = 801
}
```



##### NamedRoom

```c#
/// 지정한 이름의 방에 입장한다.
/// 지정한 이름의 방이 없을 경우 지정한 이름의 방을 생성하고 해당 방에 입장한다.
/// <param name="roomType">입장할 방의 roomType.</param>
/// <param name="roomName">입장하고자 하는 방의 이름.</param>
/// <param name="isParty"></param>
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onNamedRoom">결과를 전달 받을 delegate</param>
public void NamedRoom(string roomType, string roomName, bool isParty);
public void NamedRoom(string roomType, string roomName, bool isParty, Payload payload);
public void NamedRoom(string roomType, string roomName, bool isParty, Payload payload, User.Interface.DelUserOnNamedRoom onNamedRoom);

/// NamedRoom의 결과
/// <param name="userAgent">NamedRoom을 요청한 UserAgent 객체</param>
/// <param name="result">NamedRoom 결과</param>
/// <param name="roomName">방 이름</param>
/// <param name="roomId">입장한 방의 RoomId.</param>
/// <param name="created">입장한 방의 생성 여부(방장 여부)</param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnNamedRoom(UserAgent userAgent, Defines.ResultCodeNamedRoom result, int roomId, string roomName, bool created, Payload payload);

/// NamedRoom 결과 코드.
public enum ResultCodeNamedRoom
{
	/// 성공.
    NAMED_ROOM_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    NAMED_ROOM_FAIL_CONTENT = 1001,
    /// 실패. 방 생성이 실패하여 방을 찾을 수 없음.
    NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST = 1002,
    /// 실패. 이미 방에 들어가 있음.
    NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM = 1003,
    /// 실패. 잘못된 방 이름을 보냈을 경우.
    NAMED_ROOM_FAIL_INVALID_ROOM_NAME = 1004
}
```



##### MatchRoom

```c#
/// <summary>
/// 자동으로 방에 입장한다.
/// 방이 없을 경우 임의의 방을 생성하고 해당 방에 입장할 수도 있습니다.
/// </summary>
/// <param name="roomType">입장할 방의 roomType.</param>
/// <param name="isCreateRoomIfNotJoinRoom">true - 방이 없을 경우 임의의 방을 생성. false - 방이 없을 경우 실패</param>
/// <param name="isMoveRoomIfJoinedRoom">true - 방에 입장한 상태에서 요청했을 경우 다른 방으로 이동. false - 방에 입장한 상태에서 요청했을 경우 실패</param>
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="leaveRoomPayload">입장한 방을 떠날때 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onMatchRoom">결과를 전달 받을 delegate.</param>
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, Interface.DelUserOnMatchRoom onMatchRoom);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, Payload payload);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, Payload payload, Interface.DelUserOnMatchRoom onMatchRoom);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, bool isMoveRoomIfJoinedRoom);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, bool isMoveRoomIfJoinedRoom, Interface.DelUserOnMatchRoom onMatchRoom);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, bool isMoveRoomIfJoinedRoom, Payload payload);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, bool isMoveRoomIfJoinedRoom, Payload payload, Interface.DelUserOnMatchRoom onMatchRoom);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, bool isMoveRoomIfJoinedRoom, Payload payload, Payload leaveRoomPayload);
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, bool isMoveRoomIfJoinedRoom, Payload payload, Payload leaveRoomPayload, User.Interface.DelUserOnMatchRoom onMatchRoom);

/// MatchRoom의 결과
/// <param name="userAgent">MatchRoom을 요청한 UserAgent 객체</param>
/// <param name="result">MatchRoom 결과</param>
/// <param name="roomId">매치된 방의 RoomId</param>
/// <param name="roomName">매치된 방의 이름</param>
/// <param name="created">매치된 방의 생성 여부(방장 여부)</param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnMatchRoom(UserAgent userAgent, Defines.ResultCodeMatchRoom result, int roomId, string roomName, bool created, Payload payload);

/// MatchRoom 결과 코드.
public enum ResultCodeMatchRoom
{
	/// 성공.
    MATCH_ROOM_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    MATCH_ROOM_FAIL_CONTENT = 901,
    /// 실패. 방이 존재하지 않음.
    MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST = 902,
    /// 실패. 이미 방에 들어가 있음.
    MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM = 903,
    /// 실패. 방 이동 기능으로 매칭시킬 때, 기존방에서 나가기가 실패한 경우.
    MATCH_ROOM_FAIL_LEAVE_ROOM = 904,
    /// 실패. 이미 매치 매이킹이 진행중인 경우.
    MATCH_ROOM_FAIL_IN_PROGRESS = 905,
    /// 실패. 조건에 맞는 방을 찾아 방에 참가 시키는 도중, 방이 사라짐.
    MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST = 906
}
```



##### MatchUserStart

```c#
/// <summary>
/// 매치를 요청한다.
/// 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패할 수 있습니다.
/// 매칭이 성공한 경우 OnMatchUserDone을 통해 알림을 받을 수 있습니다.
/// </summary>
/// <param name="roomType">매치를 요청할 roomType.</param>
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onMatchUserStart">결과를 전달 받을 delegate</param>
public void MatchUserStart(string roomType);
public void MatchUserStart(string roomType, Payload payload);
public void MatchUserStart(string roomType, User.Interface.DelUserOnMatchUserStart onMatchUserStart);
public void MatchUserStart(string roomType, Payload payload, User.Interface.DelUserOnMatchUserStart onMatchUserStart);

/// MatchUserStart의 결과
/// <param name="userAgent">MatchUserStart를 요청한 UserAgent 객체</param>
/// <param name="result">MatchUserStart 결과</param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnMatchUserStart(UserAgent userAgent, Defines.ResultCodeMatchUserStart result, Payload payload);

/// MatchUserStart 결과 코드.
public enum ResultCodeMatchUserStart
{
	/// 성공.
    MATCH_USER_START_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨
    MATCH_USER_START_FAIL_CONTENT = 1101,
    /// 실패. 이미 방에 들어가 있음.
    MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM = 1102
}
```



##### MatchUserCancel

```c#
/// <summary>
/// 매치 요청을 취소한다.
/// 이미 매칭이 성공했거나 Timeout이 발생했을 경우 실패할 수 있습니다.
/// </summary>
/// <param name="roomType">매치를 요청할 roomType.</param>
/// <param name="onMatchUserCancel">결과를 전달 받을 delegate</param>
public void MatchUserCancel(string roomType);
public void MatchUserCancel(string roomType, User.Interface.DelUserOnMatchUserCancel onMatchUserCancel);

/// MatchUserCancel의 결과
/// <param name="userAgent">MatchUserCancel을 요청한 UserAgent 객체</param>
/// <param name="result">MatchUserCancel 결과</param>
public delegate void DelUserOnMatchUserCancel(UserAgent userAgent, Defines.ResultCodeMatchUserCancel result);

/// MatchUserCancel 결과 코드.
public enum ResultCodeMatchUserCancel
{
	/// 성공.
    MATCH_USER_CANCEL_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    MATCH_USER_CANCEL_FAIL_CONTENT = 1201,
    /// 실패. 이미 매칭이 이루어짐.
    MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM = 1202
}
```



##### MatchPartyStart

```c#
/// 파티 매치를 요청한다.
/// 서버의 조건에 따라 요청이 실패할 수 있습니다.
/// 매칭이 성공한 경우 OnMatchPartyDone을 통해 알림을 받을 수 있습니다.
/// <param name="roomType">매치를 요청할 roomType.</param>
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onMatchPartyStart">결과를 전달 받을 delegate</param>
public void MatchPartyStart(string roomType, Payload payload);
public void MatchPartyStart(string roomType, Payload payload, User.Interface.DelUserOnMatchPartyStart onMatchPartyStart);

/// MatchPartyStart의 결과
/// <param name="userAgent">MatchPartyStart을 요청한 UserAgent 객체</param>
/// <param name="result">MatchPartyStart 결과</param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnMatchPartyStart(UserAgent userAgent, Defines.ResultCodeMatchPartyStart result, Payload payload);

/// MatchPartyStart 결과 코드.
public enum ResultCodeMatchPartyStart
{
	/// 성공.
    MATCH_PARTY_START_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    MATCH_PARTY_START_FAIL_CONTENT = 1301,
    /// 실패. 파티매칭을 요청할 때, 룸이 파티매칭용 룸이 아닌경우.
    MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD = 1302
}
```



##### MatchPartyCancel

```c#
/// 파티 매치 요청을 취소한다.
/// 이미 매칭이 성공했거나 Timeout이 발생했을 경우 실패할 수 있습니다.
/// <param name="roomType">매치를 요청할 roomType.</param>
/// <param name="onMatchPartyCancel">결과를 전달 받을 delegate</param>
public void MatchPartyCancel(string roomType);
public void MatchPartyCancel(string roomType, User.Interface.DelUserOnMatchPartyCancel onMatchPartyCancel);

/// MatchPartyCancel의 결과
/// <param name="userAgent">MatchPartyCancel 을 요청한 UserAgent 객체</param>
/// <param name="result">MatchPartyCancel의 결과</param>
public delegate void DelUserOnMatchPartyCancel(UserAgent userAgent, Defines.ResultCodeMatchPartyCancel result);

/// MatchPartyCancel 결과 코드.
public enum ResultCodeMatchPartyCancel
{
	/// 성공.
    MATCH_PARTY_CANCEL_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨
    MATCH_PARTY_CANCEL_FAIL_CONTENT = 1401,
    /// 실패. 파티매칭을 취소할 때, 룸이 파티매칭용 룸이 아닌경우.
    MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD = 1402
}
```



##### MoveChannel

```c#
/// 지정한 채널로 이동한다.
/// <param name="channelId">이동할 channel의 id.</param>
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onMoveChannel">결과를 전달 받을 delegate.</param>
public void MoveChannel(string channelId);
public void MoveChannel(string channelId, Payload payload);
public void MoveChannel(string channelId, Payload payload, User.Interface.DelUserOnMoveChannel onMoveChannel);

/// MoveChannel의 결과 또는 서버에서 강제로 수행한 체널 이동 알림
/// <param name="userAgent">체널을 이동한 UserAgent 객체</param>
/// <param name="result">체널이동 결과 코드</param>
/// <param name="force">서버에서 강제로 체널을 이동했는지 여부</param>
/// <param name="channelId">이동한 체널의 ChannelId</param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnMoveChannel(UserAgent userAgent, Defines.ResultCodeMoveChannel result, bool force, string channelId, Payload payload);

/// MoveChannel 결과 코드.
public enum ResultCodeMoveChannel
{
	/// 성공.
    MOVE_CHANNEL_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    MOVE_CHANNEL_FAIL_CONTENT = 1601,
    /// 실패. 체널 노드를 찾을 수 없음.
    MOVE_CHANNEL_FAIL_NODE_NOT_FOUND = 1602,
    /// 실패. 이미 요청한 체널에 있음.
    MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL = 1603,
    /// 실패. 이미 방에 입장하여 체널 이동을 할 수 없음.
    MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM = 1604
}
```



##### Snapshot

```c#
/// 현재 user의 snapshot을 얻어옵니다.
/// 재접속 등 user의 모든 정보를 갱싱해야 할 필요가 있을때 사용할 수 있습니다.
/// 서버 구현에 따라 내용이 달라집니다.
/// <param name="payload">서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onSnapshot">결과를 전달 받을 delegate.</param>
public void Snapshot();
public void Snapshot(Payload payload);
public void Snapshot(Payload payload, User.Interface.DelUserOnSnapshot onSnapshot);

/// Snapshot의 결과
/// <param name="userAgent">Snapshot을 요청한 UserAgent 객체</param>
/// <param name="result">Snapshot의 결과</param>
/// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
public delegate void DelUserOnSnapshot(UserAgent userAgent, Defines.ResultCodeSnapshot result, Payload payload);

/// Snapshot 결과 코드.
public enum ResultCodeSnapshot
{
	/// 성공.
    SNAPSHOT_SUCCESS = 0,
    /// 실패. 컨텐츠에서 거부됨.
    SNAPSHOT_FAIL_CONTENT = 501
}
```



#### Listener 등록

IUserListener 인터페이스를 구현해 등록하고 이를 통해 각 기능의 결과에 대한 알림을 받을 수 있습니다. 

```c#
public sealed class LoginInfo
{
    // 로그인한 service의 이름.
    public string ServiceName { get; private set; }
    // 로그인한 channel의 Id.
    public string ChannelId { get; private set; }
    // 로그인 실패시 추가 메시지.
    public string Message { get; private set; }
    // 서버로부터 전달 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.
    public Payload Payload { get; private set; }
    // 재 로그인 여부.
    public bool IsRelogined { get; private set; }
    // 방 입장 여부.
    public bool isJoinedRoom { get; private set ; }
    // 입장한 방의 Id
    public string RoomId { get; private set; }
    // 서버로부터 전달 받은 방에 대한 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.
    public Payload RoomPayload { get; private set; }
    // 매칭 중인지 여부.
    public bool IsMatching { get; private set; }
}

public class UserListener : IUserListener
{
	void OnLogin(UserAgent userAgent, GameAnvil.Defines.ResultCodeLogin result, UserAgent.LoginInfo loginInfo)
    {
    	// public void Login(Payload payload, string channelId, string lobbyId)에 대한 응답.
    	// result : 요청 결과.
    	// loginInfo : 로그인 정보.
    }

	void OnReconnect(UserAgent userAgent, UserAgent.LoginInfo loginInfo)
    {
    	// 재접속이 성공한 경우
    	// loginInfo : 재접속 로그인 정보.
    }

	void OnSnapshot(UserAgent userAgent, GameAnvil.Defines.ResultCodeSnapshot result, Payload payload)
	{
		// public void Snapshot(Payload payload = null)에 대한 응답.
		// result : 요청 결과.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
	}

	void OnLogout(UserAgent userAgent, GameAnvil.Defines.ResultCodeLogout result, bool force, Payload payload)
	{
		// public void Logout()에 대한 응답.
		// result : 요청 결과.
		// force : 강제 로그아웃 여부.
		// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
	}
	
	void OnCreateRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeCreateRoom result, string roomId, Payload payload)
	{
		// public void CreateRoom(Payload payload = null)에 대한 응답.
		// result : 요청 결과.
    	// roomId : 생성한 방 id.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
	}
	
	void OnJoinRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeJoinRoom result, string roomId, Payload payload)
	{
		// public void JoinRoom(string roomId, Payload payload)에 대한 응답.
		// result : 요청 결과.
    	// roomId : 생성한 방 id.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
	}

	void OnMatchRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchRoom result, string roomId, Payload payload)
	{
		// public void MatchRoom(bool isCreateRoomIfNotJoinRoom, Payload payload)에 대한 응답.
		// result : 요청 결과
    	// roomId : 재입장한 방 id.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
	}

	void OnNamedRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeNamedRoom result, string roomName, Payload payload)
	{
		// public void NamedRoom(string roomName, Payload payload = null)에 대한 응답.
		// result : 요청 결과.
    	// roomName : 입장한 방 이름.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
	}

	void OnMatchUserStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserStart result, Payload payload)
    {
    	// public void MatchUserStart(Payload payload = null)에 대한 응답. 매칭을 시작했다는 응답이며, 실제 매칭이 이루어졌을 경우에는 OnMatchUserDone 으로 알림을 받게됨. 서버의 상태에 따라 요청이 실패할 수 있음.(ex. 이미 방에 들아가 있는 경우 등.)
		// result : 요청 결과.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
    }
    
    void OnMatchUserDone(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, string roomId, Payload payload)
    {
    	// public void MatchUserStart(Payload payload = null)로 매칭 요청이 성공한 후 매칭이 이루어지면 받을 알림
		// result : 요청 결과.
		// created : 방을 생성했는지 여부. (방장 여부.)
		// roomId : 매칭된 room의 Id.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
    }
    
    void OnMatchUserTimeout(UserAgent userAgent)
    {
    	// public void MatchUserStart(Payload payload = null)로 매칭 요청이 성공한 후 서버의 설정된 시간동안 매칭이 이루어지지 못하면 받을 알림
    }
    
    void OnMatchUserCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserCancel result)
    {
    	// public void MatchUserCancel(string roomType)에 대한 응답. MatchUserCancel을 서버에 요청했을때 이미 매칭이 이루어졌거나 Timeout이 발생했을 수 있음. 
		// result : 요청 결과.
    }
    
    void OnMatchPartyStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchPartyStart result, Payload payload)
    {
    	// public void MatchPartyStart(Payload payload = null)에 대한 응답. 매칭을 시작했다는 응답이며, 실제 매칭이 이루어졌을 경우에는 OnMatchUserDone 으로 알림을 받게됨. 서버의 상태에 따라 요청이 실패할 수 있음.(ex. 이미 방에 들아가 있는 경우 등.)
		// result : 요청 결과.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
    }
    
    void OnMatchPartyCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchPartyCancel result)
    {
    	// public void MatchPartyCancel(string roomType)에 대한 응답. MatchUserCancel을 서버에 요청했을때 이미 매칭이 이루어졌거나 Timeout이 발생했을 수 있음. 
		// result : 요청 결과.
    }
    
    void OnLeaveRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeLeaveRoom result, bool force, string roomId, Payload payload)
	{
		// public void LeaveRoom(Payload payload = null)에 대한 응답.
		// result : true - 방 퇴장 성공, false - 방 퇴장 실패
		// force : 강제 퇴장 여부.
    	// roomId : 퇴장한 방 id.
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
	}

	void OnMoveChannel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMoveChannel result, bool force, string channelId, Payload payload)
    {
    	// 채널 이동이 이루어졌을 경우에 대한 알림.
    	// force : 강제 이동 여부.
    	// channelId : 이동한 채널 id
    	// payload : 추가 정보. 서버 구현에 따라 내용이 달라질 수 있음.
    }
    
	void OnError(UserAgent userAgent, GameAnvil.Defines.ErrorCode errorCode, User.Defines.Commands command)
	{
		// 기능 요청이 실패했을 경우에 대한 알림.
		// errorCode : TIMEOUT - 타임아웃으로 인한 실패. SERVER_ERROR - 서버 에러로 인한 실패.
		// command : 실패한 기능.
	}

	void OnError(UserAgent userAgent, GameAnvil.Defines.ErrorCode errorCode, string command)
	{
		// 사용자 정의 기능 요청이 실패했을 경우에 대한 알림.
		// errorCode : TIMEOUT - 타임아웃으로 인한 실패. SERVER_ERROR - 서버 에러로 인한 실패.
		// command : 실패한 사용자 정의 기능.
	}
	
	void OnNotice(UserAgent userAgent, string message)
	{
		// 서버에서 보낸 알림 
		// message : 알림 내용.
	}
	
	void OnAdminKickout(UserAgent userAgent, string message)
	{
		// 어드민에서 강제 종료.
		// message : 알림 내용.
	}
}
```

#### Delegate를 등록

UserAgent에 정의되어 있는 각 기능별 delegate에 직접 등록하여 알림을 받을 수 있습니다.

```c#
GameAnvil.UserAgent user = connector.GetUserAgent(serviceId, subId);

user.onLoginListeners += OnLoginListeners;
user.onLogoutListeners += OnLogoutListeners;
user.onReconnectListeners += OnReconnectListeners;
user.onMatchRoomListeners += OnMatchRoomListeners;
user.onCreateRoomListeners += OnCreateRoomListeners;
user.onJoinRoomListeners += OnJoinRoomListeners;
user.onNamedRoomListeners += OnNamedRoomListeners;
user.onMatchPartyStartListeners += OnMatchPartyStartListeners;
user.onMatchPartyCancelListeners += OnMatchPartyCancelListeners;
user.onMatchUserStartListeners += OnMatchUserStartListeners;
user.onMatchUserDoneListeners += OnMatchUserDoneListeners;
user.onMatchUserTimeoutListeners += OnMatchUserTimeoutListeners;
user.onMatchUserCancelListeners += OnMatchUserCancelListeners;
user.onLeaveRoomListeners += OnLeaveRoomListeners;
user.onMoveChannelListeners += OnMoveChannelListeners;
user.onSnapshotListeners += OnSnapshotListeners;
user.onErrorCommandListeners += OnErrorListeners;
user.onErrorCustomCommandListeners += OnErrorListeners;
user.onNoticeListeners += OnNoticeListeners;
user.onAdminKickout += OnAdminKickout;


```



## 메시지

ConnectionAgent, UserAgent의 기본 기능 외에 Request()와 Send()를 이용하여 메시지를 서버로 전송할 수 있습니다. 메시지를 전송하기 위해서는 메시지를 생성하고 등록하는 과정이 필요합니다.



### 메시지 생성

GameAnvil은 기본 메시지 프로토콜로 [ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/overview)를 사용하고 있습니다.  .proto파일에 메시지를 정의하고, protoc 컴파일러로 실제 클래스 소스 코드를 생성하게 됩니다. 생성된 소스 코드를 프로젝트에 추가하여 사용할 수 있습니다. protoc 컴파일러는 [여기](https://github.nhnent.com/game-server-engine/GameAnvil-connector-charp/tree/master/bin)에서 찾을 수 있습니다. 

```protobuf
// Messages.proto
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
```



### 메시지 등록

새로 생성한 메시지를 사용하기 위해서는 사용하려는 메시지를 ProtocolManager에 서버와 같은 값으로 미리 등록해야합니다. 등록하지 않거나 서버와 다를 경우 동작하지 않거나 오동작 하거나 예외가 발생 할 수 있습니다

```c#
ProtocolManager.getInstance().RegisterProtocol(0, Messages.Messagesflection.Descriptor);
```



### 메시지 전송

Request()로 메시지를 전송하면 서버 응답을 기다립니다. 서버 응답을 기다리는 동안 추가적인 Request()는 queue에 저장되고 서버 응답을 처리한 후 순차적으로 처리하게 됩니다. 서버 응답을 받아 처리하려면 Listener를 등록해야 합니다. 리스너를 등록하지 않을 경우 서버 응답을 받아도 별도의 알림을 주지 않고 다음 메시지를 처리하게 됩니다. 지정된 시간동안 응답이 오지 않으면 Timeout을 발생시키고 다음 메시지를 처리합니다. Timeout은 OnError() 리스너에 ErrorCode.TIMEOUT으로 전달이 됩니다.

Send()로 메시지를  전송하면 Send()의 호출 즉시 서버로 전송되며 별도의 응답을 기다리지 않습니다. Request() 에 대한 응답을 기다리는 중에도 Send()를 사용한 메시지는 바로 서버로 전송됩니다.

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
connection.AddListener((ConnectionAgent connection, Messages.SampleConnectionReceive msg)=> { });

Messages.SampleConnectionSend sampleConnectionSend= new Messages.SampleConnectionSend (); 
connection.Send(sampleConnectionSend);

Messages.SampleConnectionRequest sampleConnectionRequest = new Messages.SampleConnectionRequest();
connection.Request(sampleConnectionRequest);
```

```c#
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
user.AddListener((UserAgent user, Messages.SampleUserReceive msg)=> { });

Messages.SampleUserRequest SampleUserRequest = new Messages.SampleUserRequest();
user.Request(SampleUserRequest);
    
Messages.SampleUserSend sampleUserSend = new Messages.SampleUserSend (); 
user.Send(sampleUserSend);
```



### 커스텀 메시지

Packet 클래스를 이용하여 ProtocolBuffer 외의 임의의 데이터를 바이트 스트림으로 직렬화해 사용할 수 있습니다. Packet에 대한 자세한 내용은 [여기](##Packet)를 참고합니다.

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
int reqMsgId = 1;
int resMsgId = 2;

connection.AddListener(resMsgId, (ConnectionAgent connection, Packet packet)=> { });

Messages.SampleConnectionSend sampleConnectionSend= new Messages.SampleConnectionSend (); 
Packet sampleConnectionSendPacket = new Packet(reqMsgId, sampleConnectionSend.ToByteArray())
connection.Send(sampleConnectionSendPacket);

Messages.SampleConnectionRequest sampleConnectionRequest = new Messages.SampleConnectionRequest();
Packet sampleConnectionRequestPacket = new Packet(reqMsgId, sampleConnectionRequest.ToByteArray())
connection.Request(sampleConnectionRequestPacket);
connection.Request(sampleConnectionRequestPacket, (ConnectionAgent connection, Packet packet)=> { });
```

```c#
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
int reqMsgId = 1;
int resMsgId = 2;

user.AddListener(resMsgId, (UserAgent user, Packet packet)=> { });

Messages.SampleUserSend sampleUserSend = new Messages.SampleUserSend (); 
Packet sampleUserSendPacket = new Packet(reqMsgIndex, sampleUserSend.ToByteArray())
user.Send(sampleUserSendPacket);

Messages.SampleUserRequest sampleUserRequest = new Messages.SampleUserRequest();
Packet sampleUserRequestPacket = new Packet(reqMsgIndex, sampleUserRequest.ToByteArray())
user.Request(sampleUserRequestPacket);
user.Request(sampleUserRequestPacket, (UserAgent user, Packet packet)=> { });
```



## Packet

서버와 주고 받는 모든 메시지는 Packet 모듈에 실려서 처리 되며 Packet 모듈이 제공하는 인터페이스를 이용하게 됩니다.



### 생성

GameAnvil Connector는 Google Protocol Buffer를 기본 프로토콜로 사용하고 있습니다 . Google Protocol Buffer를 이용하는 Packet 생성은 다음과 같습니다.

```C#
CustomMessage.SampleRequest requestMsg = new CustomMessage.SampleRequest();
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
CustomMessage.SampleRequest requestMsg = new CustomMessage.SampleRequest();
Packet packet = new Packet(requestMsg);
packet.compress();

if (packet.isCompress())
    packet.decompress();

CustomMessage.SampleResponse responseMsg = packet.GetMessage<CustomMessage.SampleResponse>();

```



## Payload

GameAnvil에서 제공하는 기본 API를 이용할 때 추가적인 데이터가 필요할 수 있습니다. 이를 위해 기본 API들에는 추가 데이터를 넘겨줄 수 있는 Payload라는 파라메터가 포함되어 있습니다. 이 Payload에 필요한 데이터를 Packet에 담아 List형식으로 저장할 수 있습니다. 여기에 추가 데이터를 메시지형태로 넣어 서버로 보내거나, 서버에서 보낸 메시지를 꺼낼 수 있습니다. 
