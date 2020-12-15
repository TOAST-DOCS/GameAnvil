## Game > GameAnvil > Connector > 릴리스 노트



### 1.0.1
#### Typescript

##### FIX

* Request를 동시에 여러번 호출할 경우 호출 순서의 역순으로 패킷을 전송하는 버그 수정

------



### 1.0.0

#### C-Sharp

##### Change

* MoveService 제거
* Reconnect, Retry 기능 제거
* PacketHelper의 GetMessage에 Type을 파라메터로 받는 API추가.
* 각 ResultCode에 고유 숫자 적용

* ResultCode 추가 및 이름 변경

  * <span style="color:#eb6420">현제 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 FORCE_CLOSE_BY_NEW_CONNECTION 가 넘어오는 문제가 있다.
    다음 릴리즈 때 수정 될 예정.</span>

##### Fix

* Disconnect시에도 UserAgent의 isLogin이 true를 리턴하는 이슈 수정



#### Typescript

##### Change

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



### 1.0.0-EA3

#### Typescript
##### Change

* ConnectionAgent에 IsReconnecting() 추가.

##### FIX

* UserAgent가 로그인 된 후 disconnect되었을 때 isLogin()이 true를 리턴하는 문제 수정
* UserAgent를 여러개 사용할때 request에 대한 응답을 받지 못하고 disconnect된 경우 request에 같이 넘겨준  callback이 지워지지 않고 남아서 다음 request시 callback이 2번 호출 되는 문제 수정

-----



 ### 1.0.0-EA2

#### C-Sharp
##### Change

* 이름 변경 : Gameflex -> GameAnvil
* RemoveAllListeners() 
    * bool 파라미터 추가.
    * false일 경우 userListener, connectonListener는 제거하지 않음.
    * defalult = true,
* RemoveAllUserListeners() 추가.
* RemoveAllConnectionListeners() 추가.

#### Typescript
##### Change

* 이름 변경 : Gameflex -> GameAnvil
* RemoveAllCallback () 추가.

-----




### 1.0.0-EA


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

#### Typescript

##### Change

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

##### New

* 에러코드
    * ResultCodeLogin
        * LOGIN_FAIL_EMPTY_SUB_ID
        * LOGIN_FAIL_TIMEOUT_GAME_SERVER
    * ResultCodeMoveChannel
        * MOVE_CHANNEL_FAIL_NODE_NOT_FOUND
        * MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL
        * MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM

-----



### 0.12.1.1

#### C-Sharp

##### Change

* ProtocolManger.unregister()사용시 exception발생하는 버그 수정

#### Typescript

##### Change

* onDisconnect() 콜백에서 바로 connect() 를 호출할 경우 에러 발생 이슈 수정

-----



### 0.12.1

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

#### Typescript

##### Change

* JSDoc 적용
* UserAgent.isJoinRoom 추가.
* IUserListener.OnMatchPartyCancel()에서 Payload 제거.
* IUserListener.OnNamedRoom() 에서 roomName-> RoomId로 이름 변경
* Payload.createDefault() 제거. CreateEmpty() 로 대체함
* SessionAgent.addOnDisconnect()
    * AddOnDisconnect(callback: (agent: SessionAgent, message: string) => void): void; =>
    AddOnDisconnect(callback: (session: SessionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload) => void): void;

-----



### 0.12.0

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

#### Typescript

##### Change

* Packet
  * GetDescId(): number; 삭제
  * GetMsgIndex(): number; 삭제
  * GetTimeOut(): number; 삭제
  * SetTimeOut(timeOut: number): void; 삭제

##### New

* ResultCodeMatchRoom
    * MATCH\_ROOM\_FAIL\_UNKNOWN\_ERROR = 6,
    * MATCH\_ROOM\_FAIL\_MATCHED\_ROOM\_DOES\_NOT\_EXIST = 7
* Packet
    * GetMsgId(): number;
* ProtocolManager
    * static getMsgId(descId: number, msgIndex: number): number;
    * static getMsgIdFromPb\(msg: IMessage \| \(new \(\) =\> T\)\): any;
    * static getCustomMsgId(customMsgId: number): number;

