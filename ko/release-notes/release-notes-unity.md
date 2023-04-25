## Game > GameAnvil > 릴리스 노트 > Connector-CSharp 

### 1.3.0 (2022.12.27)

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.3.0.unitypackage)

#### GameAnvil 1.3.0 이상

#### New
빠른 연결, 로그 레벨 변경, 동기화 기능 등이 새롭게 추가되었습니다. 게임엔빌 커넥터 컴포넌트를 통해 새로운 기능을 이용할 수 있습니다.
###### 빠른 연결
빠른 연결 기능은 게임엔빌 엔진 기반 서버에 접속, 인증, 로그인 하는 세 과정을 한번의 메서드 호출로 이루어지도록 합니다.
###### 로그 레벨 변경 기능
전용 컴포넌트의 인스펙터 창을 통해 로그 레벨을 info, debug, warn, error 등으로 설정하여 제공되는 로그의 빈도를 조정할 수 있습니다.
###### 동기화 컴포넌트 제공
컴포넌트 제공으로 Unity 엔진과 연동 편의성이 대폭 강화되었습니다. 서버의 복잡한 로직 구현 없이도 원격의 클라이언트 간에 게임오브젝트의 여러 요소들이 자동으로 동기화 됩니다.
 
* GameObject 생성, 파괴 자동 동기화
* Animation 동기화
  * Animation 파라미터 동기화로 게임오브젝트의 Animation을 동기화 할 수 있습니다.
* Transform 동기화
  * Position, Transform, Scale 등 게임오브젝트의 Transform 요소를 동기화 합니다.
  * 코드를 통한 변형 외에 어떠한 경로를 통한 변형도 모두 감지합니다.
* RigidBody 및 RigidBody2D 동기화
  * 로컬 클라이언트에서 계산된 강체 상태를 원격 클라이언트에게 전송하여 동기화합니다.
  * 강체 상태에 변화가 없을 경우에는 패킷을 보내지 않도록 최적화 되어 있습니다.
* 커스텀 값 동기화 기능
  * 방 단위로 key-value 쌍으로 커스텀 값을 설정하여 사용할 수 있습니다.
  * CAS방식의 값 설정을 지원하므로 타이밍 이슈를 쉽게 해결할 수 있습니다.

#### Fix

* 매칭 성공 후 매칭 요청을 다시 보냈을 때 방 입장 여부가 잘못 기록되는 문제 수정
* packetTimeout 옵션을 0으로 설정시 클라이언트가 자동으로 접속 종료 되지 않도록 수정
* update() 메서드 호출 시에 가비지가 생성 되는 이슈 해결
* 에러에 대한 리스너를 등록하지 않은 상태에서 에러가 발생한 경우 예외가 발생하는 현상 수정
 
#### Change

* API 변경 : 이름 변경 및 인자로 ErrorCode를 받도록 수정

| 변경 전 | 변경 후 |
|--|--|
| OnTimeout(msgId) | OnError(msgId, ErrorCode) |
| OnCustomTimeout(command) | OnCustomError(command, ErrorCode)| 

* ErrorCode 추가

| 명칭 | 설명 |
|--|--|
| ErrorCode.PARSE_ERROR | 프로토콜 파싱에 실패하였음을 나타냄|

* 사용자 정의 프로토콜 콜백에 ResultCode 추가

| 명칭 | 설명 |
|--|--|
| ResultCode.PARSE_ERROR | 패킷 파싱에 실패하였음을 나타냄 |
| ResultCode.SYSTEM_ERROR| 서버 시스템 에러 |
| ResultCode.TIMEOUT | 타임아웃 |
| ResultCode.SUCCESS | 성공 |

* Exception 추가

| 명칭 | 설명 |
|--|--|
| ParseInvalidProtocol | 프로토콜 파싱이 실패하는 경우에 발생 |

* 로깅 시스템 개선
  * 핑퐁 로그 출력을 옵션으로 정할 수 있는 기능을 추가
  * 로그에서 메시지 정보를 출력하는 상황에 메시지 아이디 대신에 이름을 출력하도록 수정
  * 출력할 로그 레벨을 조정할 수 있는 기능을 추가


* Request시에 패킷을 바로 전송할 수 있는 옵션 requestDirect 추가
* setArgumentDelegateOrListenersOnError 옵션 추가

---

### 1.2.3 (2022.01.28)

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.3.unitypackage)

#### GameAnvil 1.2.0 이상

#### Fix
* 서버에서 보낸 압축 패킷을 처리하지 못하고 오류가 발생하는 문제 수정

------
### 1.2.2 (2021.11.30)

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.2.unitypackage)

#### GameAnvil 1.2.0 이상

#### Fix

* 방에 입장한 상태에서 MatchRoom을 호출하여 실패한경우 IsJoinedRoom()이 false로 바뀌는 문제 수정

------
### 1.2.1 (2021.08.10) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.1.unitypackage)

#### GameAnvil 1.2.0 이상

#### Fix

* SocketException 발생시 OnDisconnect가 두번 호출되는 버그 수정

------

### 1.2.0 (2021.07.13) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.0.unitypackage)

#### GameAnvil 1.2.0 이상

#### Change

* Send() 의 리턴 타입이 void로 변경
* Config.defaultReqestTimeout의 기본값이 3초로 변경
* Request() 또는 다른 API호출시 Callback을 인자로 같이 넘기면 인자로 넘긴 callback으로만 응답이 가도록 변경
    * Config.useArgumentDelegateOnly 추가 (기본값 true)
    * Config.useArgumentDelegateOnly가 false 일경우, Request() 또는 다른 API호출시 Callback을 인자로 같이 넘겨도 인자로 넘긴 callback과 미리 등록한 listener가 동시에 호출.(이전 버전의 동작 방식)
* Exceptions
    * NotConnected 추가
    * NotAuthenticated 추가
    * NotLoggedIn 추가
    * MatchUserInProgress 추가
    * MatchPartyInProgress 추가
    * MatchUserNotInProgress 추가
    * MatchPartyNotInProgress 추가
* ConnectionAgent
    * 서버에 접속하지 않은 상태에서 요청을 보낼 경우 NotConnected 예외 발생
    * Authenticated 되지 않은 상태에서 요청을 보낼 경우 NotAuthenticated  예외 발생
    * PauseClientStateCheck(), ResumeClientStateCheck() 는 예외.
    * LoginedUserInfo.payload 제거
    * GetAllChannelInfo() 추가
    * GetChannelCountInfo() 추가
    * GetAllChannelCountInfo() 추가
* UserAgent
    * Login 하지 않은 상태에서 요청을 보낼 경우 NotLoggedIn 예외 발생
    * LoginInfo.Message제거
    * GetChannelInfo() 추가
    * GetAllChannelInfo() 추가
    * GetChannelCountInfo() 추가
    * GetAllChannelCountInfo() 추가
    * CreateRoom() 에 matchingGroup 파라메터 추가
    * JoinRoom() 에 matchingUserCategory 파라메터 추가
    * MatchRoom() 에 matchingGroup, matchingUserCategory 파라메터 추가.
    * MatchUserStart() 에  matchingGroup 파라메터 추가
    * MatchPartyStart() 에  matchingGroup 파라메터 추가
    * IsUserMatchInProgress(), IsPartyMatchInProgress(), IsMatchInProgress() 추가
    * UserMatch중 UserMatchCancel 을 제외한 api 호출 시 UserMatchInProgress 예외 발생
        * UserMatch중이 아닐 때 UserMatchCancel 호출 시 UserMatchNotInProgress 예외 발생
        * PartyMatch중 PartyMatchCancel 을 제외한 api 호출 시 UserMatchInProgress 예외 발생
        * PartyMatch중이 아닐 때 UserMatchCancel 호출 시 PartyMatchNotInProgress 예외 발생
* ResultCode
	* ResultCodeConnect
    	* CONNECT_FAIL_INVALID_IP 추가
	* ResultCodeAuth
    	* AUTH_FAIL_MAINTENANCE 제거 
	* ResultCodeCreateRoom
        * CREATE_ROOM_FAIL_CREATE_ROOM_ID 추가
        * CREATE_ROOM_FAIL_CREATE_ROOM 추가
    * ResultCodeChannelInfo
        * CHANNEL_INFO_FAIL_NO_CHANNEL_INFO 추가
        * CHANNEL_INFO_FAIL_INVALID_SERVICE_ID 추가
        * CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND 추가
    * ResultCodeAllChannelInfo 추가
    * ResultCodeChannelCountInfo 추가
    * ResultCodeAllChannelCountInfo 추가
    * ResultCodeChannelList
        * CHANNEL_LIST_FAIL_INVALID_SERVICEID 제거
        * CHANNEL_LIST_FAIL_NO_CHANNEL_LIST 추가
    * ResultCodeJoinRoom
        * JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM 추가
        * JOIN_ROOM_FAIL_ALREADY_FULL 추가
        * JOIN_ROOM_FAIL_ROOM_MATCH 추가
    * ResultCodeLogin
        * LOGIN_FAIL_MAINTENANCE 제거
    * ResultCodeMatchUserCancel
        * MATCH_USER_CANCEL_FAIL_CONTENT -> MATCH_USER_CANCEL_FAIL 이름 변경
        * MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS 추가
    * ResultCodeMatchRoom
        * MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID 추가
        * MATCH_ROOM_FAIL_CREATE_FAILED_ROOM  추가
        * MATCH_ROOM_FAIL_INVALID_ROOM_ID  추가
        * MATCH_ROOM_FAIL_INVALID_NODE_ID  추가
        * MATCH_ROOM_FAIL_INVALID_USER_ID  추가
        * MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND  추가
        * MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY 추가
        * MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY 추가
        * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL  추가
        * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL 추가
    * ResultCodeMatchUserDone
        * MATCH_USER_DONE_FAIL_TRANSFER 추가
        * MATCH_USER_DONE_FAIL_CREATE_ROOM 추가
    * ResultCodeNamedRoom
        * NAMED_ROOM_FAIL_CREATE_ROOM 추가
    * ResultCodeDisconnect
        * FORCE_CLOSE_MAINTENANCE 제거
        * FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID 추가.
        * FORCE_CLOSE_DISCONNECT_ALARM 제거
        * FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT 추가
        * FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION 추가
    * ResultCodeSessionClose 추가

------
### 1.1.6 (2023.01.20)

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.6.unitypackage)

#### GameAnvil 1.1.0 이상

#### Fix
* useIpv6 옵션 활성화상태에서 connect() 호출 시 블록 될 수 있는 이슈 수정 

------

### 1.1.5 (2022.01.28)

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.5.unitypackage)

#### GameAnvil 1.1.0 이상

#### Fix
* 서버에서 보낸 압축 패킷을 처리하지 못하고 오류가 발생하는 문제 수정

------
### 1.1.4 (2021.11.30) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.4.unitypackage)

#### GameAnvil 1.1.0 이상

#### Fix
* 방에 입장한 상태에서 MatchRoom을 호출하여 실패한경우 IsJoinedRoom()이 false로 바뀌는 문제 수정

------
### 1.1.3 (2021.08.10) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.3.unitypackage)

#### GameAnvil 1.1.0 이상
#### Fix
* SocketException 발생시 OnDisconnect가 두번 호출되는 버그 수정

------

### 1.1.2 (2021.04.15) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.2.unitypackage)

#### GameAnvil 1.1.0 이상
#### New
* GameAnvil Server의 ClientStateCheck기능을 일시 정지시키는 ConnectionAgent.PauseClientStateCheck() 추가. 앱이 백그라운드로 진입하는 등 메시지 루프가 동작하지 않게 되는 경우 호출해준다.
* 일시 정지했던 ClientStateCheck기능을 다시 동작 시키는 ConnectionAgent.ResumeClientStateCheck() 추가. 앱이 백그라운드에서 복귀하는 등 메시지 루프가 다시 동작하게 되는 경우 호출해준다.
* Singleserver.SetOnPauseClientStateCheck() 추가.
* Singleserver.SetOnResumeClientStateCheck() 추가.
#### Fix
* OnDisconnect에서 force가 false일 때 ResultCodeDisconnect 값이 0으로 넘어오는 버그 수정

------



### 1.1.1 (2021.04.07) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.1.unitypackage)

#### GameAnvil 1.1.0 이상
#### Change
* listener개별 등록 여부를 확인 할 수 있는 ContainsListener 오버 로딩 추가. 
* 메시지 별 모든 listener를 삭제할 수 있는 RemoveAllListenersForMsg, RemoveAllListenersForMsgId 추가
* ContainsUserListener, ContainsUserNotificationListener, ContainsUserErrorListener 추가.
* ContainsConnectionListener, ContainsConnectionNotificationListener, ContainsConnectionErrorListener, ContainsConnectionErrorListener 추가.
#### Fix
* 강제 종료, 로그아웃, 로그인 실패 등의 상황에서 UserAgent의 일부 정보가 초기화 되지 않는 버그 수정

------



### 1.1.0 (2020.12.18) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.0.unitypackage)

#### GameAnvil 1.1.0 이상
#### Change
* .Net 4.5 이상 지원으로 완전 전환
	* 사용 라이브러리 모두 .Net 4.5 용 최신 버전으로 교체
* ConnectionAgent 
	* IsConnected() : 접속 여부 확인 API 추가.
	* IsAuthenticated() : 인증되었는지 확인 API 추가.
* SingleServer의 GameAnvil.Action -> GameAnvil.Handler로 이름 변경.

------

### 1.0.0 (2020.08.31) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.0.0.unitypackage)

#### GameAnvil 1.0.0 이상
#### Change

* MoveService 제거
* Reconnect, Retry 기능 제거
* PacketHelper의 GetMessage에 Type을 파라메터로 받는 API추가.
* 각 ResultCode에 고유 숫자 적용

* ResultCode 추가 및 이름 변경

	* <span style="color:#eb6420">현재 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 FORCE_CLOSE_BY_NEW_CONNECTION 가 넘어오는 문제가 있다. 추후 수정 될 예정.</span>

#### Fix

* Disconnect시에도 UserAgent의 isLogin이 true를 리턴하는 이슈 수정



#### ResultCode 세부 변경사항

* ResultCodeAuth
	* 추가
		* AUTH_FAIL_INVALID_ACCOUNT_ID
* ResultCodeLogin
	* 이름변경
		* LOGIN_FAIL_EMPTY_SUB_ID -> LOGIN_FAIL_INVALID_SUB_ID
	* 제거
		* LOGIN_FAIL_LOGINED_SAME_SERVICE
	* 추가.
		* LOGIN_FAIL_INVALID_USERID : 잘못된 유저 아이디.
		* LOGIN_FAIL_LOGINED_OTHER_USER_TYPE : 동일 Account 아이디로 다른 UserType이 로그인 되어있음.
		* LOGIN_FAIL_LOGINED_OTHER_DEVICE : 동일 Account 아이디로 다른 DeviceId가 로그인 되어있음.
* ResultCodeMatchRoom
	* 제거
		* MATCH_ROOM_FAIL_UNKNOWN_ERROR
* ResultCodeMatchUserStart
	* 제거
		* MATCH_USER_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchUserCancel
	* 제거
		* MATCH_USER_CANCEL_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyStart
	* 제거
		* MATCH_PARTY_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyCancel
	* 제거
		* MATCH_PARTY_CANCEL_FAIL_MATCH_UNKNOWN_ERROR        
* ResultCodeNamedRoom
	* 추가.
		* NAMED_ROOM_FAIL_INVALID_ROOM_NAME : 잘못된 방 이름을 보냈을 경우.
* ResultCodeDisconnect
	* 이름 변경
		* FORCE_CLOSE_SYSTEM -> FORCE_CLOSE_SYSTEM_ERROR
		* FORCE_CLOSE_CONTENT -> 
		FORCE_CLOSE_BASE_CONNECTION : 서버에서 BaseConnection의 close() 호출
		FORCE_CLOSE_BASE_USER : 서버에서 BaseUser의 closeConnection() 호출
	* 추가.
		* FORCE_CLOSE_INVALID_NODE : GameNode가 invalid 상태로 변경된 경우.
		* FORCE_CLOSE_USER_TRANSFER_FAIL : 유저 트렌스퍼가 실패한 경우.
		* FORCE_CLOSE_USER_TRANSFER_ERROR : 유저 트렌스퍼중 시스템 에러가 발생한 경우.
	* 추가되었으나 클라이언트에서 받을 수 없는 경우.
		서버에서는 클라이언트의 연결이 이미 끊겨있을것으로 예상하고 접속을 강제 종료하면서 아래 코드를 사용한다.
		클라이언트의 연결이 끈겨 있기 때문에 아래 코드는 받을 수 없어야 한다.
		이 코드를 받았다면 GameAnvil 개발팀에 제보해 주시길 바란다.
		* FORCE_CLOSE_BY_NEW_CONNECTION : 같은 계정 정보로 새로운 로그인 요청이 들어온 경우. 
		* <span style="color:#eb6420">현재 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 이 코드가 넘어오는 문제가 있다.
          추후 수정 될 예정.</span>
		* FORCE_CLOSE_DISCONNECT_ALARM : 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우.
		* FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL : 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우.
		* FORCE_CLOSE_GHOST_USER : 고스트 유저인 경우.


-----



### 1.0.0-EA2 (2020.07.07)

#### C-Sharp
##### Change

* 이름 변경 : Gameflex -> GameAnvil
* RemoveAllListeners() 
	* bool 파라미터 추가.
	* false일 경우 userListener, connectonListener는 제거하지 않음.
	* defalult = true,
* RemoveAllUserListeners() 추가.
* RemoveAllConnectionListeners() 추가.

-----

### 1.0.0-EA (2020.06.29)

#### C-Sharp
##### Change

* 이름 변경 : Tardis -> Gameflex
	* TardisSocket -> Socket
	* SessionAgent -> ConnectionAgent
* Type 변경
	* UserId의 type이 string에서 int로 변경
	* SubId의 type이 string에서 int로 변경
	* RoomId의 type이 string에서 int로 변경
* CreateUserAgent(), GetUserAgent()
	* 파라미터 `string SubId` -> `int SubId` 로 변경
	* SubId > 0 이어야 한다.
* LoginedUserInfo
	* UserId 항목 추가.
	* Payload 항목 추가.
* LoginInfo
	* userId 항목 추가.
	* userType 항목 추가.
	* roomName 항목 추가.
* UserAgent
	*  파라미터 `string roomId` -> `int roomId` 로 변경.
	* RequestToSessionActor() ->RequestToGatewaySession()
	* SendToSessionActor() -> SendToGatewaySession()
	* RequestToSessionActor() -> RequestToGatewaySession()
	* CreateRoom() - 파라미터 `string roomName` 추가.
	* onCreateRoom() - 파라미터 `string roomName` 추가.
	* onJoinRoom() - 파라미터 `string roomName` 추가.
	* onMatchRoom() 
		* 파라미터 `string roomName` 추가.
		* 파라미터 `bool created` 추가.
	* onNamedRoom()
		* 파라미터 `string roomName` 추가.
		* 파라미터 `bool created` 추가.

##### New

*  에러코드
	* ResultCodeLogin
		* LOGIN_FAIL_EMPTY_SUB_ID
		* LOGIN_FAIL_TIMEOUT_GAME_SERVER
	* ResultCodeMoveChannel
		* MOVE_CHANNEL_FAIL_NODE_NOT_FOUND
		* MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL
		* MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM
* Exception
	* InvalidSubId

-----



### 0.12.1.1 (2020.06.23)

#### C-Sharp

##### Change

* ProtocolManger.unregister()사용시 exception발생하는 버그 수정

-----



### 0.12.1 (2020.04.06)

#### C-Sharp

##### Change

* XML 문서 주석 적용
* Agent.LvNetLogger, Logger 삭제.
* Agent.ContainsListener(int customMsgId) -> AgentHelper로 이동
* SessionAgent.AddCustomMsgListener, RemoveCustomMsgListener -> AgentHelper.RemoveListener() 로 대체
* UserAgent.AddCustomMsgListener, RemoveCustomMsgListener -> AgentHelper.AddListener, RemoveListener로 대체
* Agent.FlushQueue 제거
* Connector.FlushNetworkQueue 제거
* Packet.GetMsgId, GetRetryCount, SetRetryCount, setUncompressSize, GetServiceId, GetSubId -> 제거
* ProtocolManager.RegisterProtocol()에서 protocolId의 최대값 체크 (최대 510)
* 잘못된 ProtocolId사용시 InvalidProtocolId Exception 발생하도록 수정
* OnNamedRoom에서 RoomName -> RoomId 이름 변경

##### Fix

* MatchUserDone일때 isJoinRoom 셋팅 안되던 버그 수정

-----



### 0.12.0 (2020.02.14)

#### C-Sharp

##### Change

* 파라미터 이름 변경 : msgIndex -> customMsgId
	* Agent
		* public bool ContainsListener(int customMsgId)
	* SessionAgent
		* public void AddCustomMsgListener(int custonMsgId, Action\<SessionAgent, Packet> listener)
		* public void RemoveCustomMsgListener(int customMsgId, Action\<SessionAgent, Packet> listener)
	* UserAgent
		* public void AddCustomMsgListener(int custonMsgId, Action\<UserAgent, Packet> listener)
		* public void RemoveCustomMsgListener(int customMsgId, Action\<UserAgent, Packet> listener)
	* Payload
		* public Packet getPacket(int customMsgId)

* Packet
	* public Packet(int descId, int msgIndex, byte[] buffer) 삭제
	* public Packet(int msgIndex, byte[] buffer) 삭제
	* public string GetFileDescriptorName() 삭제
	* public void SetDescId(int descId) 삭제
	* public int GetDescId() 삭제
	* public int GetMsgIndex() 삭제
	* public Packet setTimeout(int timeout) 삭제
	* public int GetTimeout() 삭제

##### New

* ResultCodeMatchRoom
	* MATCH\_ROOM\_FAIL\_UNKNOWN\_ERROR = 6,
	* MATCH\_ROOM\_FAIL\_MATCHED\_ROOM\_DOES\_NOT\_EXIST = 7
* Packet
	* public static Packet CreateWithCustomMsg(int customMsgId, byte[] buffer)
	* public int GetMsgId()
