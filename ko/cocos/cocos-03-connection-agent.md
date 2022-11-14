## Game > GameAnvil > CocosCreator 개발 가이드 > ConnectionAgent

## ConnectionAgent

ConnectionAgent는 GameAnvil 서버의 커넥션 노드와 관련된 작업을 담당합니다. 접속(Connect()), 인증(Authentication()) 등 기본 세션 관리 기능 및 채널 목록 등을 제공하며, 직접 정의한 프로토콜을 기반으로 여러 가지 컨텐츠를 구현할 수 있습니다. ConnectionAgent는 커넥터가 초기화될 때 자동으로 생성되며 Connector.GetConnectionAgent() 함수를 이용해 얻을 수 있습니다.

```typescript
let connectionAgent = connector.GetConnectionAgent();
```

### 서버 접속

ConnectionAgent의 Connect 함수를 이용해 서버에 접속합니다. 

```typescript
// GameAnvil 서버에 연결
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void;
//  ipAddress : 접속할 서버 주소. 
//  callback : 결과를 전달받아 처리할 콜백
connectionAgent.Connect(ipAddress,(agent: ConnectionAgent, resultCode: ResultCodeConnect) => {
    //  agent : Connect()를 호출한 ConnectionAgent 객체.
	//  resultCode : Connect 결과.
    if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

### 인증

ConnectionAgent의 Authenticate 함수를 이용해 인증절차를 진행합니다. 인자로 deviceId, accountId, password, payload, 응답을 처리할 콜백을 넘겨줍니다. deviceId는 중복 접속에 대한 처리에 활용되며, accountId와 password를 활용하여 서버에서 인증 처리를 할 수 있습니다. 인증을 위해 deviceId, accountId, password 외의 추가 정보가 필요할 경우 payload에 담아 보낼수 있습니다. Authenticate 함수를 호출하면 서버에서는 BaseConnection의 onAuthentication() 콜백이 호출되며 이 콜백의 처리 결과로 인증의 성공, 실패가 결정됩니다.

```typescript
/**
 * GameAnvil 서버에 인증을 요청
 * 
 * 인증에 성공해야 GameAnvil 커넥터의 다른 기능을 사용 가능
 * @param deviceId 사용자 기기를 식별 할 수 있는 고유 아이디. 같은 사용자의 중복 접속을 체크하는데 사용
 * @param accountId 사용자 계정을 식별 할 수 있는 고유 아이디
 * @param password 사용자 계정의 패스워드
 * @param payload 서버로 전달 할 추가 정보
 * @param callback 결과를 전달 받아 처리 할 콜백
 */
connectionAgent.Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    /**
     * Authentication()의 결과
     * @param connection Authentication()을 요청한 커넥션에이전트
     * @param resultCode 인증의 결과 코드
     * @param loginedUserInfoList 서버에 남아있는 로그인 정보 목록
     * @param message 서버로부터 받은 메세지, 인증 실패 이유 등
     * @param payload 서버로 부터 받은 추가 정보
     */
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
		// 성공
    } else {
		// 실패
    }
});
```

### 채널 정보

GameAnvil은 설정으로 자유롭게 채널을 구성할 수 있습니다. 이런 채널구성은 서버와 클라이언트간에 미리 약속하여 고정된 형태로 사용할 수도 있지만, 상황에 따라 다양하게 변경하여 사용할 수도 있습니다. ConnectionAgent에서는 이렇게 변경된 채널 정보를 얻어올 수 있도록 몇가지 함수를 제공합니다. 

GetChannelList()는 특정 서비스의 채널 아이디 목록을 요청하여 받아올 수 있습니다. 

```typescript
/**
 * 특정 서비스에서 사용 가능한 채널 목록을 요청
 * @param serviceName 채널 정보를 요청할 서비스의 이름
 * @param callback 결과를 전달 받아 처리 할 콜백
 */
connectionAgent.GetChannelList(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeChannelList, channelIdList: Array<string>)=>{
    /**
     * GetChannelList()의 결과
     * @param connection GetChannelList()를 요청한 커넥션에이전트
     * @param resultCode GetChannelList()의 결과 코드
     * @param channelIdList 체널 목록
     */
    if (ResultCodeChannelList.CHANNEL_LIST_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

GetChannelCountInfo()는 특정 채널의 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 

```typescript
/**
 * 특정 채널의 유저와 방 개수를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param serviceName 채널 정보를 요청할 서비스의 이름
 * @param channelId 채널 정보를 요청할 체널의 아이디
 * @param callback 결과를 전달받아 처리 할 콜백
 */
connectionAgent.GetChannelCountInfo(serviceName, channelId, (agent: ConnectionAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo)=>{
    /**
     * GetChannelCountInfo()의 결과
     * @param connection GetChannelCountInfo()를 요청한 커넥션에이전트
     * @param resultCode GetChannelCountInfo()의 결과 코드
     * @param channelCountInfo 체널의 유저와 방 개수
     */
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

GetChannelInfo()는 특정 채널의 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 

```typescript
/**
 * 특정 채널의 정보를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param serviceName 채널 정보를 요청할 서비스의 이름
 * @param channelId 채널 정보를 요청할 체널의 아이디
 * @param callback 결과를 전달받아 처리 할 콜백
 */
connectionAgent.GetChannelInfo(serviceName, channelId, (agent: ConnectionAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload)=>{
    /**
     * GetChannelInfo()의 결과
     * @param connection GetChannelInfo()를 요청한 커넥션에이전트
     * @param resultCode GetChannelInfo()의 결과 코드
     * @param channelInfo 체널 정보
     */
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

GetAllChannelCountInfo()는 특정 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 

```typescript
/**
 * 특정 서비스에 있는 모든 채널의 유저와 방의 개수를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param serviceName 채널 정보를 요청할 서비스의 이름
 * @param callback 결과를 전달받아 처리 할 콜백
 */
connectionAgent.GetAllChannelCountInfo(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo)=>{
    /**
     * GetAllChannelCountInfo()의 결과
     * @param connection GetAllChannelCountInfo()를 요청한 커넥션에이전트
     * @param resultCode GetAllChannelCountInfo()의 결과 코드
     * @param channelInfoList 체널의 유저와 방 개수
     */
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

GetAllChannelInfo()는 특정 서비스의 모든 채널에 대한 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 

```typescript
/**
 * 특정 서비스에 있는 모든 채널의 유저와 방의 개수를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param serviceName 채널 정보를 요청할 서비스의 이름
 * @param callback 결과를 전달받아 처리 할 콜백
 */
connectionAgent.GetAllChannelInfo(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo)=>{
    /**
     * GetAllChannelInfo()의 결과
     * @param connection GetAllChannelInfo()를 요청한 커넥션에이전트
     * @param resultCode GetAllChannelInfo의 결과 코드
     * @param allChannelInfo 체널 정보
     */
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

### 접속 종료

ConnectionAgent의 Disconnect 함수를 이용해 서버와의 접속을 종료합니다. 

```typescript
/**
 * GameAnvil 서버와의 연결을 해제
 * @param reason 콜백에 전달할 연결 해제 이유
 * @param callback 접속 종료시 호출될 콜백
 */
connectionAgent.Disconnect(reason, (agent: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string) =>{
    /**
     * Disconnect()의 결과 또는 또는 강제 접속종료 알림
     * @param connection Disconnect()를 요청한 커넥션에이전트
     * @param resultCode Disconnect()의 결과 코드
     * @param reason 종료 이유. ConnectionAgent.Disconnect()에 인자로 넘긴 이유
     */
    if (ResultCodeDisconnect.SOCKET_DISCONNECT == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

### Listener

IConnectionListener는 ConnectionAgent의 모든 동작의 결과 또는 알림을 정의한 인터페이스입니다. 이 인터페이스를 구현한 리스너를 ConnectionAgent.AddConnectionListener()로 등록하면 등록한 리스너로 응답을 받을 수 있습니다. 

```typescript
class ConnectionListener implements IConnectionListener{
    /**
     * Connect()의 결과
     * @param connection Connect()을 요청한 커넥션에이전트
     * @param resultCode 연결의 결과 코드
     */
    OnConnect(connection: ConnectionAgent, resultCode: ResultCodeConnect): void { }
    /**
     * Authentication()의 결과
     * @param connection Authentication()을 요청한 커넥션에이전트
     * @param resultCode 인증의 결과 코드
     * @param loginedUserInfoList 서버에 남아있는 로그인 정보 목록
     * @param message 서버로부터 받은 메세지, 인증 실패 이유 등
     * @param payload 서버로 부터 받은 추가 정보
     */
    OnAuthentication(connection: ConnectionAgent, resultCode: ResultCodeAuth, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload): void { }
    /**
     * GetChannelList()의 결과
     * @param connection GetChannelList()를 요청한 커넥션에이전트
     * @param resultCode GetChannelList()의 결과 코드
     * @param channelIdList 체널 목록
     */
    OnChannelList(connection: ConnectionAgent, resultCode: ResultCodeChannelList, channelIdList: Array<string>): void { }
    /**
     * GetChannelInfo()의 결과
     * @param connection GetChannelInfo()를 요청한 커넥션에이전트
     * @param resultCode GetChannelInfo()의 결과 코드
     * @param channelInfo 체널 정보
     */
    OnChannelInfo(connection: ConnectionAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload): void { }
    /**
     * GetChannelCountInfo()의 결과
     * @param connection GetChannelCountInfo()를 요청한 커넥션에이전트
     * @param resultCode GetChannelCountInfo()의 결과 코드
     * @param channelCountInfo 체널의 유저와 방 개수
     */
    OnChannelCountInfo(connection: ConnectionAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo): void { }
    /**
     * GetAllChannelInfo()의 결과
     * @param connection GetAllChannelInfo()를 요청한 커넥션에이전트
     * @param resultCode GetAllChannelInfo의 결과 코드
     * @param allChannelInfo 체널 정보
     */
    OnAllChannelInfo(connection: ConnectionAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo): void { }
    /**
     * GetAllChannelCountInfo()의 결과
     * @param connection GetAllChannelCountInfo()를 요청한 커넥션에이전트
     * @param resultCode GetAllChannelCountInfo()의 결과 코드
     * @param channelInfoList 체널의 유저와 방 개수
     */
    OnAllChannelCountInfo(connection: ConnectionAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo): void { }
    /**
     * Disconnect()의 결과 또는 또는 강제 접속종료 알림
     * @param connection Disconnect()를 요청한 커넥션에이전트
     * @param resultCode Disconnect()의 결과 코드
     * @param reason 종료 이유. ConnectionAgent.Disconnect()에 인자로 넘긴 이유
     * @param force 강제 종료 여부
     * @param payload 서버로 부터 받은 추가 정보
     */
    OnDisconnect(connection: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload): void { }
    /**
     * 오류 발생
     * @param connection 오류 발생 한 커넥션에이전트
     * @param errorCode 에러 코드
     * @param msgName 에러가 발생한 메세지의 이름
     */
    OnErrorCommand(connection: ConnectionAgent, errorCode: ErrorCode, msgName: string): void { }
};

connectionAgent.AddListener(new ConnectionListener());
```

