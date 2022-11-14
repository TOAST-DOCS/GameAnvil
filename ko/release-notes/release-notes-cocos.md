## Game > GameAnvil > 릴리스 노트 > Connector-Typescript
### 1.2.1 (2021.11.30) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.2.1.zip)

#### GameAnvil 1.2.0 이상

#### Fix
* 방에 입장한 상태에서 MatchRoom을 호출하여 실패한경우 IsJoinedRoom()이 false로 바뀌는 문제 수정

------

### 1.2.0 (2021.07.13) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.2.0.zip)

#### GameAnvil 1.2.0 이상

#### Change

* Request() 또는 다른 API호출시 Callback을 인자로 같이 넘기면 인자로 넘긴 callback으로만 응답이 가도록 변경
  * Config.useArgumentCallbackOnly 추가 (기본값 true)
  * Config.useArgumentCallbackOnly 가 false 일경우, Request() 또는 다른 API호출시 Callback을 인자로 같이 넘겨도 인자로 넘긴 callback과 미리 등록한 listener가 동시에 호출.(이전 버전의 동작 방식)
* ConnectionAgent
  * 서버에 접속하지 않은 상태에서 요청을 보낼 경우 "Not connected. Connect before call XXX." 에러 발생
  * Authenticated 되지 않은 상태에서 요청을 보낼 경우 Not authenticated. Authenticate before call XXX." 에러 발생
    * PauseClientStateCheck(), ResumeClientStateCheck() 는 예외.
  * LoginedUserInfo.payload 제거
  * GetAllChannelInfo() 추가
  * GetChannelCountInfo() 추가
  * GetAllChannelCountInfo() 추가
* UserAgent
  * Login 하지 않은 상태에서 요청을 보낼 경우 "Not loggedIn. Login before call XXX()." 에러 발생
  * LoginInfo.message제거
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
  * UserMatch중 UserMatchCancel 을 제외한 api 호출 시 "MatchUser is in progress. MatchUserCancel before call XXX." 에러 발생
   * UserMatch중이 아닐 때 UserMatchCancel 호출 시 "MatchUser is not in progress." 에러 발생
   * PartyMatch중 PartyMatchCancel 을 제외한 api 호출 시 "MatchParty is in progress. MatchPartyCancel before call XXX." 에러 발생
   * PartyMatch중이 아닐 때 UserMatchCancel 호출 시 "MatchParty is not in progress." 에러 발생
* ResultCode
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
### 1.1.4 (2021.11.30) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.4.zip)

#### GameAnvil 1.1.0 이상

#### Fix
* 방에 입장한 상태에서 MatchRoom을 호출하여 실패한경우 IsJoinedRoom()이 false로 바뀌는 문제 수정

------

### 1.1.3 (2021.04.07) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.3.zip)

#### GameAnvil 1.1.0 이상
#### Change
* ContainsCallback, ContainsUndefinedProtocolCallback, ContainsListener 추가.
* RemoveCallback, RemoveUndefinedProtocolCallback에 callback개별로 삭제 가능하도록 callback 옵셔널 파라메터 추가.
* RemoveListener의 listener를 옵셔널 파라메터로 변경.
* ContainsOnError, ContainsOnDisconnect 추가.
* RemoveOnError, RemoveOnDisconnect에 callback개별로 삭제 가능하도록 callback옵셔널 파라메터 추가.
#### Fix
* 로그아웃, 로그인 실패 등의 상황에서 UserAgent의 일부 정보가 초기화 되지 않는 버그 수정
* AddOnError로 등록된 콜백의 인자에 잘못된 값이 넘어가는 버그 수정
* RequestPB 호출시 callback을 넣지 않을 경우 예외발생하는 문제 수정

------



### 1.1.2 (2021.03.16) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.2.zip)

#### GameAnvil 1.1.0 이상
#### Change
* Connector.config의 default값 변경.
	* PingInterval : 10 -> 3
	* PacketTimeout : 12 -> 5
#### Fix
* CodeInserter에서 .proto에 메시지 내부에 중접 메시지 선언이 있을 경우 다음 메시지부터 index 값이 밀리는 버그 수정

------



### 1.1.1 (2021.02.10) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.1.zip)

#### GameAnvil 1.1.0 이상
#### Change
* SingleServer 
	* onMatchRoom 사용 시 payload에 null이 넘어오는 이슈 수정
	* OnLogin에 id: string 파라메터를 subId: number로 변경
	* OnCreateRoom, OnJoinRoom, OnMatchRoom, OnNamedRoom에 누락된 roomName 파라메터 추가.
	* OnMatchRoom, OnNamedRoom에 누락된 created파라메터 추가. 
	* OnXXX 등록 시 기존에 등록된 콜백 제거.
	* OnLogin() 응답으로 loginInfo에 null을 넘길 경우 serviceId에 요청받은 serviceId를 넣어 응답하도록 예외 처리 추가.
	* 등록된 콜백에서 에러가 발생 할 경우 callstack 정보가 사용자에게 올라 갈 수 있도록 다시 throw

------



### 1.1.0 (2020.12.18) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.0.zip)

#### GameAnvil 1.1.0 이상
#### Change
* 호환성 이슈로 수정하여 GitEnterprize에 올려놓고 사용하던 protobufjs를 이슈가 수정된 공식 최신버전으로 교체

------



### 1.0.1 (2020.10.08) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.0.1.zip)

#### GameAnvil 1.0.0 이상
#### FIX

* Request를 동시에 여러번 호출할 경우 호출 순서의 역순으로 패킷을 전송하는 버그 수정

------



### 1.0.0 (2020.08.31) 

#### [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.0.0.zip)

#### GameAnvil 1.0.0 이상
#### Change
* MoveService 제거
* Reconnect, Retry 기능 제거
* 각 ResultCode에 고유 숫자 적용
* ResultCode 추가 및 이름 변경
	* <span style="color:#eb6420">현제 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 FORCE_CLOSE_BY_NEW_CONNECTION 가 넘어오는 문제가 있다.
        다음 릴리즈 때 수정 될 예정.</span>



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
		* <span style="color:#eb6420">현제 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 이 코드가 넘어오는 문제가 있다.
          다음 릴리즈 때 수정 될 예정.</span>
		* FORCE_CLOSE_DISCONNECT_ALARM : 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우.
		* FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL : 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우.
		* FORCE_CLOSE_GHOST_USER : 고스트 유저인 경우.


-----



### 1.0.0-EA3 (2020.07.27)
#### Change
* ConnectionAgent에 IsReconnecting() 추가.

#### FIX
* UserAgent가 로그인 된 후 disconnect되었을 때 isLogin()이 true를 리턴하는 문제 수정
* UserAgent를 여러개 사용할때 request에 대한 응답을 받지 못하고 disconnect된 경우 request에 같이 넘겨준  callback이 지워지지 않고 남아서 다음 request시 callback이 2번 호출 되는 문제 수정

-----



### 1.0.0-EA2 (2020.07.07)
#### Change

* 이름 변경 : Gameflex -> GameAnvil
* RemoveAllCallback () 추가.

-----




### 1.0.0-EA (2020.06.29)
#### Change
* 이름 변경 : Tardis -> Gameflex
	* SessionAgent -> ConnectionAgent
* Type 변경
	* UserId의 type이 string에서 number로 변경
	* SubId의 type이 string에서 number로 변경
	* RoomId의 type이 string에서 number로 변경
* CreateUserAgent(), GetUserAgent()
	* 파라미터 `SubId: string` -> `SubId: number` 로 변경
	* SubId > 0 이어야 한다.
* LoginedUserInfo
	* UserId 항목 추가.
	* Payload 항목 추가.
* LoginInfo
	* userId 항목 추가.
	* userType 항목 추가.
	* roomName 항목 추가.
* UserAgent
	*  파라미터 `roomId: string` -> `roomId: number` 로 변경.
	* MultiSendToSessionActor()->MultiSendToGatewaySession()
	* SendToSessionActor() -> SendToGatewaySession()
	* SendPbToSessionActor() -> SendPbToGatewaySession()
	* RequestToSessionActor() -> RequestToGatewaySession()
	* MultiRequestToSessionActor() -> MultiRequestToGatewaySession()
	* RequestToSessionActor() -> RequestToGatewaySession()
	* RequestPbToSessionActor() -> RequestPbToGatewaySession()
	* RequestPbAsyncToSesstionActor() -> RequestPbAsyncToGatewaySession()
	* RequestAsyncToSessionActor() -> RequestAsyncToGatewaySession()
	* CreateRoom() - 파라미터 `roomName: string` 추가.
	* onCreateRoom() - 파라미터 `roomName: string` 추가.
	* onJoinRoom() - 파라미터 `roomName: string` 추가.
	* onMatchRoom() 
		* 파라미터 `roomName: string` 추가.
		* 파라미터 `created: boolean` 추가.
	* onNamedRoom()
		* 파라미터 `roomName: string` 추가.
		* 파라미터 `created: boolean` 추가.

#### New
* 에러코드
	* ResultCodeLogin
		* LOGIN_FAIL_EMPTY_SUB_ID
		* LOGIN_FAIL_TIMEOUT_GAME_SERVER
	* ResultCodeMoveChannel
		* MOVE_CHANNEL_FAIL_NODE_NOT_FOUND
		* MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL
		* MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM

-----



### 0.12.1.1 (2020.06.23)
#### Change

* onDisconnect() 콜백에서 바로 connect() 를 호출할 경우 에러 발생 이슈 수정

-----



### 0.12.1 (2020.04.06)
#### Change

* JSDoc 적용
* UserAgent.isJoinRoom 추가.
* IUserListener.OnMatchPartyCancel()에서 Payload 제거.
* IUserListener.OnNamedRoom() 에서 roomName-> RoomId로 이름 변경
* Payload.createDefault() 제거. CreateEmpty() 로 대체함
* SessionAgent.addOnDisconnect()
	* AddOnDisconnect(callback: (agent: SessionAgent, message: string) => void): void; =>
	AddOnDisconnect(callback: (session: SessionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload) => void): void;

-----



### 0.12.0 (2020.02.14)
#### Change

* Packet
	* GetDescId(): number; 삭제
	* GetMsgIndex(): number; 삭제
	* GetTimeOut(): number; 삭제
	* SetTimeOut(timeOut: number): void; 삭제

#### New

* ResultCodeMatchRoom
	* MATCH\_ROOM\_FAIL\_UNKNOWN\_ERROR = 6,
	* MATCH\_ROOM\_FAIL\_MATCHED\_ROOM\_DOES\_NOT\_EXIST = 7
* Packet
	* GetMsgId(): number;
* ProtocolManager
	* static getMsgId(descId: number, msgIndex: number): number;
	* static getMsgIdFromPb\(msg: IMessage \| \(new \(\) =\> T\)\): any;
	* static getCustomMsgId(customMsgId: number): number;
