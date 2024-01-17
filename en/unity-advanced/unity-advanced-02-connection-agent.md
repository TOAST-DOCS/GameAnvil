## Game > GameAnvil > Unity 심화 개발 가이드 > ConnectionAgent

## ConnectionAgent

ConnectionAgent는 GameAnvil 서버의 커넥션 노드와 관련된 작업을 담당합니다. 접속(Connect()), 인증(Authentication()) 등 기본 세션 관리 기능 및 채널 목록 등을 제공하며, 직접 정의한 프로토콜을 기반으로 여러 가지 컨텐츠를 구현할 수 있습니다. ConnectionAgent는 커넥터가 초기화될 때 자동으로 생성되며 Connector.GetConnectionAgent() 함수를 이용해 얻을 수 있습니다.

```c#
ConnectionAgent connectionAgent = connector.GetConnectionAgent();
```

### 서버 접속

ConnectionAgent의 Connect 함수를 이용해 서버에 접속합니다. 

```c#
/// <summary>
/// GameAnvil 서버에 연결 시도
/// </summary>
/// <param name="ip">대상 아이피 주소</param>
/// <param name="port">대상 포트</param>
/// <param name="onConnect">연결 시도 결과를 전달받을 대리자</param>
connector.GetConnectionAgent().Connect(ip, port, (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
    /// <param name="connectionAgent">Connect()를 요청한 커넥션 에이전트</param>
    /// <param name="result">Connect() 요청 결과 코드</param>
    if (result == ResultCodeConnect.CONNECT_SUCCESS) {
		// 접속 성공
    } else {
	    // 접속 실패
    }
});
```

### 인증

ConnectionAgent의 Authenticate 함수를 이용해 인증 절차를 진행합니다. 매개변수로 deviceId, accountId, password, payload, 응답을 처리할 콜백을 넘겨줍니다. deviceId는 중복 접속에 대한 처리에 활용되며, accountId와 password를 활용하여 서버에서 인증 처리를 할 수 있습니다. 인증을 위해 deviceId, accountId, password 외의 추가 정보가 필요할 경우 payload에 담아 보낼 수 있으며 사용하지 않을 경우 payload 매개변수는 생략할 수 있습니다. 
Authenticate 함수를 호출하면 서버에서는 BaseConnection의 onAuthentication() 콜백이 호출되며 이 콜백의 처리 결과로 인증의 성공, 실패가 결정됩니다.

```c#
/// <summary>
/// GameAnvil 서버에 인증 요청<para></para>
/// 인증 성공 후 커넥터 사용 가능
/// </summary>
/// <param name="deviceId">사용자 기기 식별용 고유 아이디. 서버 구현에 따라 사용하지 않는 경우 빈 문자열 전달</param>
/// <param name="accountId">사용자 계정을 식별 할 수 있는 고유 아이디</param>
/// <param name="passwd">사용자 계정의 비밀번호. 서버 구현에 따라 사용하지 않는 경우 빈 문자열 전달</param>
/// <param name="onAuthentication">요청 결과를 전달받을 대리자</param>
connector.GetConnectionAgent().Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    /// <param name="connectionAgent">Authentication()를 요청한 커넥션 에이전트</param>
    /// <param name="result">Authentication() 요청 결과</param>
    /// <param name="loginedUserInfoList">서버에 남아있는 로그인 정보 목록</param>
    /// <param name="message">서버로부터 받은 메시지</param>
    /// <param name="payload">서버로부터 받은 추가 정보</param>
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
		// 인증 성공
    } else {
		// 인증 실패
    }
});
```

### 채널 정보

GameAnvil은 설정에서 자유롭게 채널을 구성할 수 있습니다. 이런 채널 구성은 서버와 클라이언트 간에 미리 약속하여 고정된 형태로 사용할 수도 있지만, 상황에 따라 다양하게 변경하여 사용할 수도 있습니다. ConnectionAgent에서는 이렇게 변경된 채널 정보를 얻어올 수 있도록 몇 가지 함수를 제공합니다. 

| 함수 | 설명 |
| --- | --- |
| GetChannelList() | 특정 서비스의 채널 아이디 목록 요청 | 
| GetChannelCountInfo() | 특정 채널의 카운트 정보(유저와 방 개수) 요청 | 
| GetChannelInfo() | 특정 채널의 정보(사용자 정의) 요청 |
| GetAllChannelCountInfo() | 특정 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수) 요청 |

<br>

아래에서 코드를 통해 더욱 자세하게 살펴보겠습니다.


GetChannelList()는 특정 서비스의 채널 아이디 목록을 요청하여 받아올 수 있습니다. 

```c#
/// <summary>
/// 서비스의 사용 가능한 채널 목록 요청 
/// </summary>
/// <param name="serviceName">대상 서비스의 이름</param>
/// <param name="onChannelList">요청 결과를 전달받을 대리자</param>
connector.GetConnectionAgent().GetChannelList(serviceName, (ConnectionAgent connection, ResultCodeChannelList result, List<string> channelIdList) => {
    /// <param name="connectionAgent">GetChannelList()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetChannelList() 요청 결과</param>
    /// <param name="channelIdList">서버에서 받은 체널 목록</param>
	if(result == ResultCodeChannelList.CHANNEL_LIST_SUCCESS){
		// 채널 목록 요청 성공
	} else {
		// 채널 목록 요청 실패
	}
});
```
<br>

GetChannelCountInfo()는 특정 채널의 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 

```c#
/// <summary>
/// 채널의 유저와 방 개수 요청<para></para>
/// 서버에서 지원하는 경우 사용 가능
/// </summary>
/// <param name="serviceName">대상 채널이 속한 서비스의 이름</param>
/// <param name="channelId">대상 채널의 아이디</param>
/// <param name="onChannelCountInfo">요청 결과를 전달 받을 대리자</param>
connector.GetConnectionAgent().GetChannelCountInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="connectionAgent">GetChannelCountInfo()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// 채널 카운트 정보 요청 성공
	} else {
		// 채널 카운트 정보 요청 실패
	}
});
```
<br>

GetChannelInfo()는 특정 채널의 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 

```c#
/// <summary>
/// 채널 정보 요청<para></para>
/// 서버에서 지원하는 경우 사용 가능
/// </summary>
/// <param name="serviceName">대상 채널이 속한 서비스의 이름</param>
/// <param name="channelId">대상 채널의 아이디</param>
/// <param name="onChannelInfo">요청 결과를 전달받을 대리자</param>
connector.GetConnectionAgent().GetChannelInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="connectionAgent">GetChannelInfo()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetChannelInfo() 요청 결과</param>
    /// <param name="channelInfo">서버에서 받은 체널 정보</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// 채널 정보 요청 성공
	} else {
		// 채널 정보 요청 실패
	}
});
```
<br>

GetAllChannelCountInfo()는 특정 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 

```c#
/// <summary>
/// 서비스 내 모든 채널의 유저와 방 개수 요청<para></para>
/// 서버에서 지원하는 경우 사용 가능
/// </summary>
/// <param name="serviceName">대상 서비스의 이름</param>
/// <param name="onAllChannelCountInfo">요청 결과를 전달 받을 대리자</param>
connector.GeConnectionAgent().GetAllChannelCountInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="connectionAgent">GetAllChannelCountInfo()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetAllChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보들</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// 모든 채널 카운트 정보 요청 성공
	} else {
		// 모든 채널 카운트 정보 요청 실패
	}
});
```
<br>

GetAllChannelInfo()는 특정 서비스의 모든 채널에 대한 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 

```c#
/// <summary>
/// 서비스 내 모든 채널의 정보 요청<para></para>
/// 서버에서 지원하는 경우 사용 가능
/// </summary>
/// <param name="serviceName">대상 서비스의 이름</param>
/// <param name="onAllChannelInfo">요청 결과를 전달 받을 대리자</param>
connector.GetConnectionAgent().GetAllChannelInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="connectionAgent"> GetAllChannelInfo()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetAllChannelInfo() 요청의 결과</param>
    /// <param name="channelInfo">서버에서 받은 체널 정보 목록</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// 모든 채널 정보 요청 성공
	} else {
		// 모든 채널 정보 요청 실패
	}
});
```

### 접속 종료

ConnectionAgent의 Disconnect 함수를 이용해 서버와의 접속을 종료합니다. 

```c#
/// <summary>
/// GameAnvil 서버와의 연결 해제 요청
/// </summary>
/// <param name="onDisconnect">요청 결과를 전달 받을 대리자</param>
connector.GetConnectionAgent().Disconnect((ConnectionAgent connectionAgent, ResultCodeDisconnect result) => {
    /// <param name="connectionAgent">Disconnect()발생한 커넥션 에이전트</param>
    /// <param name="result">>Disconnect() 결과</param>
    if (result == ResultCodeDisconnect.SOCKET_DISCONNECT) {
		// 정상 종료
    } else {
	    // 비정상 종료
    }
});
```

### Listener

ConnectionAgent에서 모든 요청에 대한 결과 또는 서버로부터의 알림을 전달받는 방법은 크게 두 가지입니다.
하나는 ConnectionAgent에 정의되어 있는 delegate에 함수를 추가하는 방법입니다. 다른 하나는 IConnectionListener 인터페이스를 구현한 리스너를 등록하는 방법입니다.

먼저, 첫 번째 방법입니다. ConnectionAgent는 모든 동작의 결과 또는 알림을 받을 수 있도록 각각의 delegate를 맴버로 가지고 있습니다. 앞서 설명한 API에 콜백 매개변수를 생략하고 호출했거나 서버에서 알림을 보냈을 경우 delegate에 등록된 함수로 응답을 받을 수 있습니다.  

```c#
/// <summary>
/// Connect()의 결과
/// </summary>
/// <param name="connectionAgent">Connect()를 요청한 커넥션 에이전트</param>
/// <param name="result">Connect()의 결과</param>
public Interface.DelConnectionOnConnect onConnectListeners;

/// <summary>
/// Authentication() 결과
/// </summary>
/// <param name="connectionAgent">Authentication()를 요청한 커넥션 에이전트</param>
/// <param name="result">Authentication() 요청 결과</param>
/// <param name="loginedUserInfoList">서버에 남아있는 로그인 정보 목록</param>
/// <param name="message">서버로부터 받은 메시지</param>
/// <param name="payload">서버로부터 받은 추가 정보</param>
public Interface.DelConnectionOnAuthentication onAuthenticationListeners;

/// <summary>
/// GetChannelList() 요청 결과
/// </summary>
/// <param name="connectionAgent">GetChannelList()를 요청한 커넥션 에이전트</param>
/// <param name="result">GetChannelList() 요청 결과</param>
/// <param name="channelIdList">서버에서 받은 체널 목록</param>
public Interface.DelConnectionOnChannelList onChannelListListeners;

/// <summary>
/// GetChannelInfo() 요청 결과
/// </summary>
/// <param name="connectionAgent">GetChannelInfo()를 요청한 커넥션 에이전트</param>
/// <param name="result">GetChannelInfo() 요청 결과</param>
/// <param name="channelInfo">서버에서 받은 체널 정보</param>
public Interface.DelConnectionOnChannelInfo onChannelInfoListeners;

/// <summary>
/// GetAllChannelInfo() 요청 결과
/// </summary>
/// <param name="connectionAgent"> GetAllChannelInfo()를 요청한 커넥션 에이전트</param>
/// <param name="result">GetAllChannelInfo() 요청의 결과</param>
/// <param name="channelInfo">서버에서 받은 체널 정보 목록</param>
public Interface.DelConnectionOnAllChannelInfo onAllChannelInfoListeners;

/// <summary>
/// GetChannelCountInfo() 요청 결과
/// </summary>
/// <param name="connectionAgent">GetChannelCountInfo()를 요청한 커넥션 에이전트</param>
/// <param name="result">GetChannelCountInfo() 요청 결과</param>
/// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보</param>
public Interface.DelConnectionOnChannelCountInfo onChannelCountInfoListeners;

/// <summary>
/// GetAllChannelCountInfo() 요청 결과
/// </summary>
/// <param name="connectionAgent">GetAllChannelCountInfo()를 요청한 커넥션 에이전트</param>
/// <param name="result">GetAllChannelCountInfo() 요청 결과</param>
/// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보들</param>
public Interface.DelConnectionOnAllChannelCountInfo onAllChannelCountInfoListeners;

/// <summary>
/// Disconnect() 알림
/// </summary>
/// <param name="connectionAgent">Disconnect()발생한 커넥션 에이전트</param>
/// <param name="result">>Disconnect() 이유</param>
/// <param name="force">강제 종료 여부</param>
/// <param name="payload">서버로부터 받은 추가 정보</param>
public Interface.DelConnectionOnDisconnect onDisconnectListeners;

/// <summary>
/// 커넥션의 기본 기능 사용중 오류 발생
/// </summary>
/// <param name="connectionAgent">오류 발생한 커넥션 에이전트</param>
/// <param name="errorCode">오류 코드</param>
/// <param name="commands">오류 발생 기능</param>
public Interface.DelConnectionOnErrorCommand onErrorCommandListeners;

/// <summary>
/// 패킷 전송 후 오류 발생
/// </summary>
/// <param name="connectionAgent">처리를 요청할 커넥션 에이전트</param>
/// <param name="errorCode">오류 코드</param>
/// <param name="command">패킷의 메시지 이름</param>
public Interface.DelConnectionOnErrorCustomCommand onErrorCustomCommandListeners;
```
<br>

다음으로, 두 번째 방법입니다. IConnectionListener는 ConnectionAgent의 모든 동작의 결과 또는 알림을 정의한 인터페이스입니다. 이 인터페이스를 구현한 리스너를 ConnectionAgent.AddConnectionListener()로 등록하면 등록한 리스너로 응답을 받을 수 있습니다. 

```c#
public class ConnectionListener : IConnectionListener
{
    /// <summary>
    /// Connect()의 결과
    /// </summary>
    /// <param name="connectionAgent">Connect()를 요청한 커넥션 에이전트</param>
    /// <param name="result">Connect()의 결과</param>
    public void OnConnect(ConnectionAgent connectionAgent, ResultCodeConnect result) { }
    
    /// <summary>
    /// Authentication() 결과
    /// </summary>
    /// <param name="connectionAgent">Authentication()울 요청한 커넥션 에이전트</param>
    /// <param name="result">Authentication() 요청 결과</param>
    /// <param name="loginedUserInfoList">서버에 남아있는 로그인 정보 목록</param>
    /// <param name="message">서버로부터 받은 메시지</param>
    /// <param name="payload">서버로부터 받은 추가 정보</param>
    public void OnAuthentication(ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) { }

    /// <summary>
    /// GetChannelList() 요청 결과 
    /// </summary>
    /// <param name="connectionAgent">GetChannelList()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetChannelList() 요청 결과</param>
    /// <param name="channelIdList">서버에서 받은 채널 목록</param>
    public void OnChannelList(ConnectionAgent connectionAgent, ResultCodeChannelList result, List<string> channelIdList) { }

    /// <summary>
    /// GetChannelInfo의 결과
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="result">GetChannelInfo의 결과</param>
    /// <param name="channelInfo">채널 정보</param>
    public void OnChannelInfo(ConnectionAgent connectionAgent, ResultCodeChannelInfo result, Payload channelInfo) { }

    /// <summary>
    /// 모든 채널 정보 요청 결과
    /// </summary>
    /// <param name="connectionAgent">GetAllChannelInfo()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetAllChannelInfo() 요청의 결과</param>
    /// <param name="channelInfo">서버에서 받은 체널 정보</param>
    public void OnAllChannelInfo(ConnectionAgent connectionAgent, ResultCodeAllChannelInfo result, Dictionary<string, Payload> channelInfo) { }

    /// <summary>
    /// GetChannelCountInfo() 요청 결과
    /// </summary>
    /// <param name="connectionAgent">GetChannelCountInfo()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보</param>
    public void OnChannelCountInfo(ConnectionAgent connectionAgent, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) { }

    /// <summary>
    /// GetAllChannelCountInfo() 정보 요청 결과
    /// </summary>
    /// <param name="connectionAgent">GetAllChannelCountInfo()를 요청한 커넥션 에이전트</param>
    /// <param name="result">GetAllChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보들</param>
    public void OnAllChannelCountInfo(ConnectionAgent connectionAgent, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) { }
    
    /// <summary>
    /// Disconnect() 또는 강제 접속 종료 결과 알림
    /// </summary>
    /// <param name="connectionAgent">Disconnect()발생한 커넥션 에이전트</param>
    /// <param name="result">Disconnect() 결과 또는 강제 접속 종료 이유</param>
    /// <param name="force">종료 강제 여부</param>
    /// <param name="payload">서버로부터 받은 추가 정보</param>
    public void OnDisconnect(ConnectionAgent connectionAgent, ResultCodeDisconnect result, bool force, Payload payload) { }
    
    /// <summary>
    /// 커넥션의 기본 기능 사용중 오류 발생
    /// </summary>
    /// <param name="connectionAgent">오류 발생한 커넥션 에이전트</param>
    /// <param name="errorCode">오류 코드</param>
    /// <param name="commands">오류 발생 기능</param>
    public void OnError(ConnectionAgent connectionAgent, ErrorCode errorCode, Commands commands) { }
    
    /// <summary>
    /// 패킷 전송 후 오류 발생
    /// </summary>
    /// <param name="connectionAgent">처리를 요청할 커넥션 에이전트</param>
    /// <param name="errorCode">오류 코드</param>
    /// <param name="command">패킷의 메시지 이름</param>
    public void OnError(ConnectionAgent connectionAgent, ErrorCode errorCode, string command) { }
}

connector.GetConnectionAgent().AddConnectionListener(new ConnectionListener);
```

