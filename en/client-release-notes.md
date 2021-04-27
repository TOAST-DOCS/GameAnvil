## Game > GameAnvil > Connector > Release Note



### 1.0.1
#### Typescript

##### FIX

* Fixed a bug in which packets were sent in the reverse order when a request is called several times at the same time

------



### 1.0.0

#### C-Sharp

##### Change

* Removed MoveService
* Removed the Reconnect and Retry feature
* Added the API that receives Type as a parameter for GetMessage of PacketHelper.
* Applied a unique number to each ResultCode

* Added and renamed ResultCode

  * <span style="color:#eb6420">Currently, there is an issue where FORCE_CLOSE_BY_NEW_CONNECTION comes to the FORCE_CLOSE_DUPLICATE_LOGIN case. It will be fixed on the next release.</span>

##### Fix

* Fixed an issue in which the isLogin of UserAgent returns true even when disconnected



#### Typescript

##### Change

* Removed MoveService

* Removed the Reconnect and Retry feature

* Applied a unique number to each ResultCode

* Added and renamed ResultCode

    * <span style="color:#eb6420">Currently, there is an issue where FORCE_CLOSE_BY_NEW_CONNECTION comes to the FORCE_CLOSE_DUPLICATE_LOGIN case. It will be fixed on the next release.</span>



#### Detailed ResultCode changes

* ResultCodeAuth
  * Add
    * AUTH_FAIL_INVALID_ACCOUNT_ID
* ResultCodeLogin
  * Rename
    * LOGIN_FAIL_EMPTY_SUB_ID -> LOGIN_FAIL_INVALID_SUB_ID
  * Remove
    * LOGIN_FAIL_LOGINED_SAME_SERVICE
  * Add.
    * LOGIN_FAIL_INVALID_USERID : An invalid user ID.
    * LOGIN_FAIL_LOGINED_OTHER_USER_TYPE : Another UserType is logged in with the same account ID.
    * LOGIN_FAIL_LOGINED_OTHER_DEVICE : Another DeviceId is logged in with the same account ID.
* ResultCodeMatchRoom
  * Remove
    * MATCH_ROOM_FAIL_UNKNOWN_ERROR
* ResultCodeMatchUserStart
  * Remove
    * MATCH_USER_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchUserCancel
  * Remove
    * MATCH_USER_CANCEL_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyStart
  * Remove
    * MATCH_PARTY_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyCancel
  * Remove
    * MATCH_PARTY_CANCEL_FAIL_MATCH_UNKNOWN_ERROR        
* ResultCodeNamedRoom
  * Add.
    * NAMED_ROOM_FAIL_INVALID_ROOM_NAME : When the wrong room name is sent.
* ResultCodeDisconnect
  * Rename
    * FORCE_CLOSE_SYSTEM -> FORCE_CLOSE_SYSTEM_ERROR
    * FORCE_CLOSE_CONTENT -> 
      FORCE_CLOSE_BASE_CONNECTION : Calls the close() of BaseConnection from server
      FORCE_CLOSE_BASE_USER : Calls the closeConnection() of BaseUser from server
  * Add.
    * FORCE_CLOSE_INVALID_NODE : When GameNode is changed to the invalid status.
    * FORCE_CLOSE_USER_TRANSFER_FAIL : When user transfer fails.
    * FORCE_CLOSE_USER_TRANSFER_ERROR : When a system error occurs during user transfer.
  * If it is added but the client cannot receive it,
    the server determines that the client is already disconnected and forcibly disconnects the client and uses the code below.
    As the client is disconnected from the server, the code below cannot be received.
    If you received this code, please contact the GameAnvil development team.
      * FORCE_CLOSE_BY_NEW_CONNECTION : When a new login request is received from the same account information. 
        * <span style="color:#eb6420">Currently, there is an issue in which this code would be passed to the FORCE_CLOSE_DUPLICATE_LOGIN case. It will be fixed on the next release.</span>
      * FORCE_CLOSE_DISCONNECT_ALARM : When the client does not respond to the server's status check.
      * FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL : When the client does not respond to the server's status check.
      * FORCE_CLOSE_GHOST_USER : If it is a ghost user.


-----



### 1.0.0-EA3

#### Typescript
##### Change

* Added IsReconnecting() to ConnectionAgent.

##### FIX

* Fixed an issue in which isLogin() would return true when UserAgent was logged in and then disconnected
* Fixed an issue in which the passed callback would remain and cause the callback to be called twice when requested when the response to a request was not received and disconnected while using multiple UserAgents

-----



 ### 1.0.0-EA2

#### C-Sharp
##### Change

* Name change : Gameflex -> GameAnvil
* RemoveAllListeners() 
    * Added bool parameter.
    * If false, userListener and connectionListener are not removed.
    * defalult = true,
* Added RemoveAllUserListeners().
* Added RemoveAllConnectionListeners().

#### Typescript
##### Change

* Name change : Gameflex -> GameAnvil
* Added RemoveAllCallback ().

-----




### 1.0.0-EA


#### C-Sharp
##### Change

* Name change : Tardis -> Gameflex
    * TardisSocket -> Socket
    * SessionAgent -> ConnectionAgent
* Type change
    * Changed the type of UserId from string to int
    * Changed the type of SubId from string to int
    * Changed the type of RoomId from string to int
* CreateUserAgent(), GetUserAgent()
    * Changed parameter `string SubId` -> `int SubId`
    * SubId must be > 0.
* LoginedUserInfo
    * Added the UserId item.
    * Added the Payload item.
* LoginInfo
    * Added the userId item.
    * Added the userType item.
    * Added the roomName item.
* UserAgent
    *  Changed parameter `string roomId` -> `int roomId`.
    * RequestToSessionActor() ->RequestToGatewaySession()
    * SendToSessionActor() -> SendToGatewaySession()
    * RequestToSessionActor() -> RequestToGatewaySession()
    * Added CreateRoom() - parameter `string roomName`.
    * Added onCreateRoom() - parameter `string roomName`.
    * Added onJoinRoom() - parameter `string roomName`.
    * onMatchRoom() 
        * Added parameter `string roomName`.
        * Added parameter `bool created`.
    * onNamedRoom()
        * Added parameter `string roomName`.
        * Added parameter `bool created`.

##### New

*  Error code
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

* Name change : Tardis -> Gameflex
    * SessionAgent -> ConnectionAgent
* Type change
    * Changed the type of UserId from string to number
    * Changed the type of SubId from string to number
    * Changed the type of RoomId from string to number
* CreateUserAgent(), GetUserAgent()
    * Changed parameter `SubId: string` -> `SubId: number`
    * SubId must be > 0.
* LoginedUserInfo
    * Added the UserId item.
    * Added the Payload item.
* LoginInfo
    * Added the userId item.
    * Added the userType item.
    * Added the roomName item.
* UserAgent
    *  Changed parameter `roomId: string` -> `roomId: number`.
    * MultiSendToSessionActor()->MultiSendToGatewaySession()
    * SendToSessionActor() -> SendToGatewaySession()
    * SendPbToSessionActor() -> SendPbToGatewaySession()
    * RequestToSessionActor() -> RequestToGatewaySession()
    * MultiRequestToSessionActor() -> MultiRequestToGatewaySession()
    * RequestToSessionActor() -> RequestToGatewaySession()
    * RequestPbToSessionActor() -> RequestPbToGatewaySession()
    * RequestPbAsyncToSesstionActor() -> RequestPbAsyncToGatewaySession()
    * RequestAsyncToSessionActor() -> RequestAsyncToGatewaySession()
    * Added CreateRoom() - parameter `roomName: string`.
    * Added onCreateRoom() - parameter `roomName: string`.
    * Added onJoinRoom() - parameter `roomName: string`.
    * onMatchRoom() 
        * Added parameter `roomName: string`.
        * Added parameter `created: boolean`.
    * onNamedRoom()
        * Added parameter `roomName: string`.
        * Added parameter `created: boolean`.

##### New

* Error code
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

* Fixed an issue in which an exception would occur when ProtocolManager.unregister() is used

#### Typescript

##### Change

* Fixed an issue in which an error would occur when calling connect() directly from the onDisconnect() callback

-----



### 0.12.1

#### C-Sharp

##### Change

* Applied the XML document comment
* Agent.LvNetLogger and Logger removed.
* Moved Agent.ContainsListener(int customMsgId) to> AgentHelper
* SessionAgent.AddCustomMsgListener, RemoveCustomMsgListener -> Replaced with AgentHelper.RemoveListener()
* UserAgent.AddCustomMsgListener, RemoveCustomMsgListener -> Replaced with AgentHelper.AddListener and RemoveListener
* Agent.FlushQueue removed
* Connector.FlushNetworkQueue removed
* Packet.GetMsgId, GetRetryCount, SetRetryCount, setUncompressSize, GetServiceId, GetSubId -> Removed
* Checks the maximum value of protocolId from ProtocolManager.RegisterProtocol() (Up to 510)
* Edited so that InvalidProtocolId Exception would occur when using invalid ProtocolId
* Changed the name RoomName -> RoomId in OnNamedRoom

##### Fix

* Fixed a bug in which isJoinRoom would not be set when MatchUserDone

#### Typescript

##### Change

* Applied JSDoc
* Added UserAgent.isJoinRoom.
* Removed payload from IUserListener.OnMatchPartyCancel().
* Changed the name roomName-> RoomId from IUserListener.OnNamedRoom()
* Removed Payload.createDefault(). Replaced by CreateEmpty()
* SessionAgent.addOnDisconnect()
    * AddOnDisconnect(callback: (agent: SessionAgent, message: string) => void): void; =>
    AddOnDisconnect(callback: (session: SessionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload) => void): void;

-----



### 0.12.0

#### C-Sharp

##### Change

* Parameter name change: msgIndex -> customMsgId
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
    * public Packet(int descId, int msgIndex, byte[] buffer) removed
    * public Packet(int msgIndex, byte[] buffer) removed
    * public string GetFileDescriptorName() removed
    * public void SetDescId (int descId) removed
    * public int GetDescId() removed
    * public int GetMsgIndex() removed
    * public Packet setTimeout (int timeout) removed
    * public int GetTimeout() removed

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
  * GetDescId(): number; removed
  * GetMsgIndex(): number; removed
  * GetTimeOut(): number; removed
  * SetTimeOut(timeOut: number): void; removed

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

