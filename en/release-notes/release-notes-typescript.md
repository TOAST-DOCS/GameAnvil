## Game > GameAnvil > Release Notes > Typescript Connector

### 2.1.0 (June 30, 2025)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector-typescript.zip)
#### GameAnvil 2.0.0 or later


#### New
* To coincide with the release of GameAnvil 2.1.0 server, the connector version 2.1.0 has also been released.
* There are no major functional modifications compared to 2.0.0, but some bug fixes, ResultCode name changes, typos, and incorrect descriptions have been modified.

#### Changed
* Updated engine protocol to match GameAnvil 2.1 server
    * Servers older than GameAnvil 2.1 are no longer supported
* Changed some ResultCodes

  | AS-IS                                                                        | TO-BE                                                                      |
  |-----------------------------------------------------------------------------|---------------------------------------------------------------------------|
  | LOGIN\_FAIL\_INVALID\_SERVICE\_ID<br>Failed. Invalid service ID | LOGIN_FAIL_INVALID_SERVICE_NAME<br>Failed. Invalid service name |
  | LOGIN\_FAIL\_INVALID\_USERTYPE<br>Failed. Invalid user type | LOGIN_FAIL_INVALID_USER_TYPE<br>Failed. Invalid user type |
  | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>Failed. Invalid service ID | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>Failed. Invalid service name |
  | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>Failed. Invalid service ID | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>Failed. Invalid service name |
  | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_FORM\_NULL<br>Failed. If the matching request is null | MATCH\_ROOM\_FAIL\_MATCH\_FORM\_NULL<br>Failed. If the matching request is null |
  | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_INFO\_NULL<br>Failed. If the matching information is null | MATCH\_ROOM\_FAIL\_MATCH\_INFO\_NULL<br>Failed. If the matching information is null |
  | FORCE\_CLOSE\_BASE\_CONNECTION<br>The server calls close() on BaseConnection | FORCE\_CLOSE\_CONNECTION<br>Calling close() on IConnection on the server |
  | FORCE\_CLOSE\_BASE\_USER<br>Calling closeConnection() on BaseUser on the server | FORCE\_CLOSE\_USER<br>Calling closeConnection() on IUser on the server                  |

#### Fix
* Fixed an issue where a response might not be received if the internally used seq value exceeds the maximum when making a request

---

### 2.0.0 (December 5, 2024)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-2.0.0.zip)

#### GameAnvil 2.0.0 or later

#### New

* A new connector using async await has been released.
* Completion is now detected by the return value, rather than registering a function to handle completion.
* The GameAnvil connection and authentication code is as follows:

```typescript
const deviceId, accountId, password;

connector.host = "127.0.0.1";
connector.port = 18300;

const authResult = await connector.connectAndAuthenticateion(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);
```

#### Remove

###### Removed ConnectionAgent
* The distinction between ConnectionAgent and GameAnvilConnector was ambiguous.
* GameAnvilConnector now incorporates the behavior of the existing ConnectionAgent.
* Other classes have been renamed to reflect this change.
    * UserAgnet has been renamed to GameAnvilUser.
    * ProtocolManager has been renamed to GameAnvilProtocolManager.

###### Removed Connector.Update
* Now automatically implemented within Connector.

###### Removed delegate for receiving request responses
* Request responses are now returned as return values.
* Messages received from the server without a request still use delegates.
    * SetMessageCallback method

#### Change

###### Directly manage UserAgent instances
* Modified to create and use UserAgent instances directly, as shown in the example below.
* Note that UserAgent.Dispose does not automatically log you out. UserAgent.Dispose only releases objects managed internally.

```typescript
const myUser = new GameAnvilUser(connector, "ServiceName", subId);
```

###### Packet Class Usability Changes
* No longer always attempts to convert to byte[].
* Packet zip can now be added at creation time.
* Packet unzip now happens automatically.

###### GameAnvilConnector Concurrency
* Fixed an issue where sending and receiving multiple requests would only send one at a time.
* Multiple requests can now be sent and received.

###### Removed MultiRequest
* As the MultiRequest feature was removed from the server, the corresponding features were also removed from GameAnvilConnector.
    * `public void Request(List<Packet> packetList)`
    * `public bool Send(List<Packet> packetList)`

------

### 1.2.1 (2021.11.30)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.2.1.zip)

#### GameAnvil 1.2.0 or later

#### Fix
* Fixed IsJoinedRoom() turning to false when calling MatchRoom while in a room and failing

------

### 1.2.0 (2021.07.13)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.2.0.zip)

#### GameAnvil 1.2.0 or later

#### Change

* Changed Request() or other API calls to respond only to the callback passed as an argument when a callback is passed as an argument.
    * Added Config.useArgumentCallbackOnly (default true)
    * If Config.useArgumentCallbackOnly is false, when Request() or other API calls are made with Callback as an argument, the callback passed as an argument and the pre-registered listener will be called simultaneously (this is how it worked in previous versions).
* ConnectionAgent
    * If you send a request while not connected to the server, you will receive the error "Not connected. Connect before call XXX." error
    * If you send a request while not authenticated, you will receive the error "Not authenticated. Authenticate before call XXX." error occurs
        * PauseClientStateCheck() and ResumeClientStateCheck() are excluded.
    * Removed LoginedUserInfo.payload
    * Added GetAllChannelInfo()
    * Added GetChannelCountInfo()
    * Added GetAllChannelCountInfo()
* UserAgent
    * Error "Not loggedIn. Login before call XXX()." error occurs
    * Removed LoginInfo.message
    * Added GetChannelInfo()
    * Added GetAllChannelInfo()
    * Added GetChannelCountInfo()
    * Added GetAllChannelCountInfo()
    * Added matchingGroup parameter to CreateRoom()
    * Added matchingUserCategory parameter to JoinRoom()
    * Added matchingGroup, matchingUserCategory parameters to MatchRoom().
    * Added a matchingGroup parameter to MatchUserStart()
    * Added a matchingGroup parameter to MatchPartyStart()
    * Added IsUserMatchInProgress(), IsPartyMatchInProgress(), and IsMatchInProgress()
    * Error "MatchUser is in progress. MatchUserCancel before call XXX." on API calls except UserMatchCancel during UserMatch
    * Error "MatchUser is not in progress." when calling UserMatchCancel when not in the middle of a UserMatch
    * Error "MatchParty is in progress. MatchPartyCancel before call XXX." when calling API except PartyMatchCancel during PartyMatch
    * "MatchParty is not in progress." error when calling UserMatchCancel when not in PartyMatch
* ResultCode
    * ResultCodeAuth
        * Removed AUTH_FAIL_MAINTENANCE
    * ResultCodeCreateRoom
        * Added CREATE_ROOM_FAIL_CREATE_ROOM_ID
        * Added CREATE_ROOM_FAIL_CREATE_ROOM
    * ResultCodeChannelInfo
        * Added CHANNEL_INFO_FAIL_NO_CHANNEL_INFO
        * CHANNEL_INFO_FAIL_INVALID_SERVICEID
        * Added CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND
    * Added ResultCodeAllChannelInfo
    * Added ResultCodeChannelCountInfo
    * Added ResultCodeAllChannelCountInfo
    * ResultCodeChannelList
        * Remove CHANNEL_LIST_FAIL_INVALID_SERVICEID
        * Added CHANNEL_LIST_FAIL_NO_CHANNEL_LIST
    * ResultCodeJoinRoom
        * Added JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM
        * Added JOIN_ROOM_FAIL_ALREADY_FULL
        * Added JOIN_ROOM_FAIL_ROOM_MATCH
    * ResultCodeLogin
        * Remove LOGIN_FAIL_MAINTENANCE
    * ResultCodeMatchUserCancel
        * Rename MATCH_USER_CANCEL_FAIL_CONTENT -> MATCH_USER_CANCEL_FAIL
        * Added MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS
    * ResultCodeMatchRoom
        * Added MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID
        * Added MATCH_ROOM_FAIL_CREATE_FAILED_ROOM
        * Added MATCH_ROOM_FAIL_INVALID_ROOM_ID
        * Added MATCH_ROOM_FAIL_INVALID_NODE_ID
        * Added MATCH_ROOM_FAIL_INVALID_USER_ID
        * Added MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND
        * Added MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY
        * Added MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY
        * Added MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL
        * Added MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL
    * ResultCodeMatchUserDone
        * Added MATCH_USER_DONE_FAIL_TRANSFER
        * Added MATCH_USER_DONE_FAIL_CREATE_ROOM
    * ResultCodeNamedRoom
        * Add NAMED_ROOM_FAIL_CREATE_ROOM
    * ResultCodeDisconnect
        * Removing FORCE_CLOSE_MAINTENANCE
        * Added FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID.
        * Remove FORCE_CLOSE_DISCONNECT_ALARM
        * Add FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT
        * Added FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION
    * Added ResultCodeSessionClose

------
### 1.1.4 (2021.11.30)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.4.zip)

#### GameAnvil 1.1.0 or later

#### Fix
* Fixed IsJoinedRoom() turning to false when calling MatchRoom while in a room and failing

------

### 1.1.3 (2021.04.07)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.3.zip)

#### GameAnvil 1.1.0 or later
#### Change
* Added ContainsCallback, ContainsUndefinedProtocolCallback, and ContainsListener.
* Added callback option parameter to RemoveCallback and RemoveUndefinedProtocolCallback to allow per-callback deletion.
* Changed listener in RemoveListener to an optional parameter.
* Added ContainsOnError, ContainsOnDisconnect.
* Added callbackoption parameter to RemoveOnError and RemoveOnDisconnect to allow per-call deletion.
#### Fix
* Fixed a bug where some information in UserAgent was not initialized in situations such as logout, failed login, etc.
* Fixed bug with invalid values being passed as arguments in callbacks registered as AddOnError
* Fixed an exception being thrown if callback is not included when calling RequestPB

------



### 1.1.2 (2021.03.16)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.2.zip)

#### GameAnvil 1.1.0 or later
#### Change
* Change the default value in Connector.config.
    * PingInterval : 10 -> 3
    * PacketTimeout : 12 -> 5
#### Fix
* Fixed a bug in CodeInserter that caused index values to be pushed back from the next message if a .proto had nested message declarations inside the message.

------



### 1.1.1 (2021.02.10)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.1.zip)

#### GameAnvil 1.1.0 or later
#### Change
* SingleServer
    * Fixed issue with null in payload when using onMatchRoom
    * Change id: string parameter to subId: number in OnLogin
    * Added the missing roomName parameter to OnCreateRoom, OnJoinRoom, OnMatchRoom, and OnNamedRoom.
    * Added the missing created parameter to OnMatchRoom, OnNamedRoom.
    * Remove previously registered callbacks when registering OnXXX.
    * Added exception handling to respond with the requested serviceId in serviceId when passing null to loginInfo in response to OnLogin().
    * If an error occurs in a registered callback, throw it again so that the callstack information can be raised to the user

------



### 1.1.0 (2020.12.18)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.0.zip)

#### GameAnvil 1.1.0 or later
#### Change
* Replace the protobufjs you were using with the latest official version that fixes the compatibility issue and puts it on GitEnterprize.

------



### 1.0.1 (2020.10.08)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.0.1.zip)

#### GameAnvil 1.0.0 or later
#### FIX

* Fixed a bug that caused packets to be sent in the reverse order of invocation when multiple requests are made at the same time.

------



### 1.0.0 (2020.08.31)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.0.0.zip)

#### GameAnvil 1.0.0 or later
#### Change
* Remove MoveService
* Remove the Reconnect, Retry feature
* Apply a unique number to each ResultCode
* Add and rename ResultCode
    * <span style="color:#eb6420">There is currently an issue with FORCE_CLOSE_DUPLICATE_LOGIN case being overridden by FORCE_CLOSE_BY_NEW_CONNECTION.
      This will be fixed in the next release.</span>



#### ResultCode detail changes

* ResultCodeAuth
    * Add
        * AUTH_FAIL_INVALID_ACCOUNT_ID
* ResultCodeLogin
    * Rename
        * LOGIN_FAIL_EMPTY_SUB_ID -> LOGIN_FAIL_INVALID_SUB_ID
    * Removals
        * LOGIN_FAIL_LOGINED_SAME_SERVICE
    * Add
        * LOGIN_FAIL_INVALID_USERID: Invalid username.
        * LOGIN_FAIL_LOGINED_OTHER_USER_TYPE : A different UserType is logged in with the same Account ID.
        * LOGIN_FAIL_LOGINED_OTHER_DEVICE : Another DeviceId is logged in with the same Account ID.
* ResultCodeMatchRoom
    * Removals
        * MATCH_ROOM_FAIL_UNKNOWN_ERROR
* ResultCodeMatchUserStart
    * Removals
        * MATCH_USER_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchUserCancel
    * Removals
        * MATCH_USER_CANCEL_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyStart
    * Removals
        * MATCH_PARTY_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyCancel
    * Removals
        * MATCH_PARTY_CANCEL_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeNamedRoom
    * Add
        * NAMED_ROOM_FAIL_INVALID_ROOM_NAME: If you sent an invalid room name.
* ResultCodeDisconnect
    * Rename
        * FORCE_CLOSE_SYSTEM -> FORCE_CLOSE_SYSTEM_ERROR
        * force_close_content ->
          FORCE_CLOSE_BASE_CONNECTION : Call close() of BaseConnection on server
          FORCE_CLOSE_BASE_USER : Call closeConnection() of BaseUser on the server
    * Add
        * FORCE_CLOSE_INVALID_NODE: If a GameNode has changed to an invalid state.
        * FORCE_CLOSE_USER_TRANSFER_FAIL: If the user transporter failed.
        * FORCE_CLOSE_USER_TRANSFER_ERROR: If a system error occurred during user transfer.
    * added, but the client cannot receive it.
      The server expects the client to have already disconnected and uses the code below while forcing the connection to close.
      The client shouldn't receive this code because the client is still connected.
      If you do receive this code, please report it to the GameAnvil development team.
        * FORCE_CLOSE_BY_NEW_CONNECTION: If a new login request comes in with the same account information.
        * <span style="color:#eb6420">There is currently an issue with this code being passed to the FORCE_CLOSE_DUPLICATE_LOGIN case.
          Will be fixed in the next release.</span>
        * FORCE_CLOSE_DISCONNECT_ALARM: If the client has not responded to the server's status check.
        * FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL: If the client did not respond to the server's health check.
        * FORCE_CLOSE_GHOST_USER: If you are a ghost user.


-----



### 1.0.0-EA3 (2020.07.27)
#### Change
* Added IsReconnecting() to ConnectionAgent.

#### FIX
* Fixed isLogin() returning true when UserAgent is disconnected after being logged in
* When using multiple UserAgents and disconnecting without receiving a response to a request, the callback passed along with the request is not deleted and remains, causing the callback to be called twice in the next request.

-----



### 1.0.0-EA2 (2020.07.07)
#### Change

* Renamed: Gameflex -> GameAnvil
* Added RemoveAllCallback().

-----




### 1.0.0-EA (2020.06.29)
#### Change
* Rename: Tardis -> Gameflex
    * SessionAgent -> ConnectionAgent
* Change Type
    * UserId's type changed from string to number
    * SubId's type changed from string to number
    * RoomId's type changed from string to number
* CreateUserAgent(), GetUserAgent()
    * Change parameter `SubId` `: string` -> `SubId: number`
    * SubId > 0.
* LoginedUserInfo
    * Added a UserId entry.
    * Added a Payload item.
* LoginInfo
    * Added a userId entry.
    * Added a userType entry.
    * Added a roomName entry.
* UserAgent
    *  Change parameter `roomId` `: string` -> `roomId: number`.
    * MultiSendToSessionActor()->MultiSendToGatewaySession()
    * SendToSessionActor() -> SendToGatewaySession()
    * SendPbToSessionActor() -> SendPbToGatewaySession()
    * RequestToSessionActor() -> RequestToGatewaySession()
    * MultiRequestToSessionActor() -> MultiRequestToGatewaySession()
    * RequestToSessionActor() -> RequestToGatewaySession()
    * RequestPbToSessionActor() -> RequestPbToGatewaySession()
    * RequestPbAsyncToSesstionActor() -> RequestPbAsyncToGatewaySession()
    * RequestAsyncToSessionActor() -> RequestAsyncToGatewaySession()
    * CreateRoom() - Added parameter `roomName: string`.
    * onCreateRoom() - Added parameter `roomName: string`.
    * onJoinRoom() - Added parameter `roomName: string`.
    * onMatchRoom()
        * Added parameter `roomName: string`.
        * Added parameter `created: boolean`.
    * onNamedRoom()
        * Added parameter `roomName: string`.
        * Added parameter `created: boolean`.

#### New
* Error codes
    * ResultCodeLogin
        * LOGIN_FAIL_EMPTY_SUB_ID
        * LOGIN_FAIL_TIMEOUT_GAME_SERVER
    * ResultCodeMoveChannel
        * MOVE_CHANNEL_FAIL_NODE_NOT_FOUND
        * MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL
        * MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM

-----



### 0.12.1.1 (June 23, 2020)
#### Change

* Fixed an issue where calling connect() directly from the onDisconnect() callback would result in an error

-----



### 0.12.1 (2020.04.06)
#### Change

* Applied the JSDoc
* Added UserAgent.isJoinRoom.
* Removed the payload from IUserListener.OnMatchPartyCancel().
* Renamed from IUserListener.OnNamedRoom() to roomName -> RoomId
* Removed Payload.createDefault(). Replace with CreateEmpty()
* SessionAgent.addOnDisconnect()
    * AddOnDisconnect(callback: (agent: SessionAgent, message: string) => void): void; =>
      AddOnDisconnect(callback: (session: SessionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload) => void): void;

-----



### 0.12.0 (2020.02.14)
#### Change

* Packet
    * GetDescId(): number; deleted
    * GetMsgIndex(): number; deleted
    * GetTimeOut(): number; deleted
    * SetTimeOut(timeOut: number): void; deleted

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
