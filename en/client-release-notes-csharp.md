## Game > GameAnvil > 릴리스 노트 > Connector-CSharp 

### 1.1.2 (2021.04.15) [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.2.unitypackage)
#### GameAnvil 1.1.0 이상
#### New
* GameAnvil Server의 ClientStateCheck기능을 일시 정지시키는 ConnectionAgent.PauseClientStateCheck() 추가. 앱이 백그라운드로 진입하는 등 메시지 루프가 동작하지 않게 되는 경우 호출해준다.
* 일시 정지했던 ClientStateCheck기능을 다시 동작 시키는 ConnectionAgent.ResumeClientStateCheck() 추가. 앱이 백그라운드에서 복귀하는 등 메시지 루프가 다시 동작하게 되는 경우 호출해준다.
* Singleserver.SetOnPauseClientStateCheck() 추가.
* Singleserver.SetOnResumeClientStateCheck() 추가.
#### Fix
* OnDisconnect에서 force가 false일 때 ResultCodeDisconnect 값이 0으로 넘어오는 버그 수정

------



### 1.1.1 (2021.04.07) [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.1.unitypackage)
#### GameAnvil 1.1.0 이상
#### Change
* listener개별 등록 여부를 확인 할 수 있는 ContainsListener 오버 로딩 추가. 
* 메시지 별 모든 listener를 삭제할 수 있는 RemoveAllListenersForMsg, RemoveAllListenersForMsgId 추가
* ContainsUserListener, ContainsUserNotificationListener, ContainsUserErrorListener 추가.
* ContainsConnectionListener, ContainsConnectionNotificationListener, ContainsConnectionErrorListener, ContainsConnectionErrorListener 추가.
#### Fix
* 강제 종료, 로그아웃, 로그인 실패 등의 상황에서 UserAgent의 일부 정보가 초기화 되지 않는 버그 수정

------



### 1.1.0 (2020.12.18) [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.0.unitypackage)

#### GameAnvil 1.1.0 이상
#### Change
* .Net 4.5 이상 지원으로 완전 전환
	* 사용 라이브러리 모두 .Net 4.5 용 최신 버전으로 교체
* ConnectionAgent 
	* IsConnected() : 접속 여부 확인 API 추가.
	* IsAuthenticated() : 인증되었는지 확인 API 추가.
* SingleServer의 GameAnvil.Action -> GameAnvil.Handler로 이름 변경.

------


------



### 1.0.0 (2020.08.31) [다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.0.0.unitypackage)

#### GameAnvil 1.0.0 이상
#### Change

* MoveService 제거
* Reconnect, Retry 기능 제거
* PacketHelper의 GetMessage에 Type을 파라메터로 받는 API추가.
* 각 ResultCode에 고유 숫자 적용

* ResultCode 추가 및 이름 변경

	* <span style="color:#eb6420">현제 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 FORCE_CLOSE_BY_NEW_CONNECTION 가 넘어오는 문제가 있다. 추후 수정 될 예정.</span>

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
		* <span style="color:#eb6420">현제 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 이 코드가 넘어오는 문제가 있다.
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
