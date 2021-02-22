## Game > GameAnvil > Connector > 릴리스 노트



### 1.0.1
#### TS

##### FIX

* Request를 동시에 여러번 호출할 경우 호출 순서의 역순으로 패킷을 전송하는 버그 수정

------



### 1.0.0

#### C#

##### Change

* MoveService 제거
* Reconnect, Retry 기능 제거
* PacketHelper의 GetMessage에 Type을 파라메터로 받는 API추가.
* 각 ResultCode에 고유 숫자 적용
* ResultCode 추가 및 이름 변경([세부항목](client-link-resultCode-change))
    * <span style="color:#eb6420">현제 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 FORCE_CLOSE_BY_NEW_CONNECTION 가 넘어오는 문제가 있다.
        다음 릴리즈 때 수정 될 예정.</span>

##### Fix

* Disconnect시에도 UserAgent의 isLogin이 true를 리턴하는 이슈 수정



#### TS

##### Change

* MoveService 제거

* Reconnect, Retry 기능 제거

* 각 ResultCode에 고유 숫자 적용

* ResultCode 추가 및 이름 변경([세부항목](client-3-release-notes-1.0.0-resultCode-change))

    * <span style="color:#eb6420">현제 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 FORCE_CLOSE_BY_NEW_CONNECTION 가 넘어오는 문제가 있다.
        다음 릴리즈 때 수정 될 예정.</span>


-----



### 1.0.0-EA3

#### TS
##### Change

* ConnectionAgent에 IsReconnecting() 추가.

##### FIX

* UserAgent가 로그인 된 후 disconnect되었을 때 isLogin()이 true를 리턴하는 문제 수정
* UserAgent를 여러개 사용할때 request에 대한 응답을 받지 못하고 disconnect된 경우 request에 같이 넘겨준  callback이 지워지지 않고 남아서 다음 request시 callback이 2번 호출 되는 문제 수정

-----



 ### 1.0.0-EA2

#### C#
##### Change

* 이름 변경 : Gameflex -> GameAnvil
* RemoveAllListeners() 
    * bool 파라미터 추가.
    * false일 경우 userListener, connectonListener는 제거하지 않음.
    * defalult = true,
* RemoveAllUserListeners() 추가.
* RemoveAllConnectionListeners() 추가.

##### TS
##### Change

* 이름 변경 : Gameflex -> GameAnvil
* RemoveAllCallback () 추가.

-----




### 1.0.0-EA


#### C#
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

#### TS

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

#### C#

##### Change

* ProtocolManger.unregister()사용시 exception발생하는 버그 수정

#### TS

##### Change

* onDisconnect() 콜백에서 바로 connect() 를 호출할 경우 에러 발생 이슈 수정

-----



### 0.12.1

#### C#

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

#### TS

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

#### C#

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

#### TS

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

