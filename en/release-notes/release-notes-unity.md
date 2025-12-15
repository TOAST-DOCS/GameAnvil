## Game > GameAnvil > Release Notes > Unity Connector

### 2.1.0 (June 30, 2025)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector.unitypackage)
#### GameAnvil 2.1.0 or later
#### New
###### GameAnvil 2.1.0 Connector
* To coincide with the release of GameAnvil 2.1.0 server, Connector version 2.1.0 is also being released.
  * There are no significant functional changes compared to version 2.0.0, but some bug fixes, ResultCode name changes, typos, and incorrect descriptions have been modified.

#### Change
* Updated engine protocol for GameAnvil 2.1 servers.
  * Servers prior to GameAnvil 2.1 are no longer supported.
* Added Packet support to GameAnvilConnector.
  * Added ability to receive Packets as a parameter to Request().
  * Added SetPacketCallback() to receive Packets as a callback parameter.
    * Support for custom packets.
* Added support for custom packets in Payload.
* Modified GameAnvilConnector to return Result instead of ErrorResult.
  * Modified the name of the class ErrorResult used to return asynchronous call results to Result.
* Modified some ResultCode.

    | AS-IS | TO-BE |
    | ---- | ---- |
    | LOGIN\_FAIL\_INVALID\_SERVICE\_ID<br>Failed. Invalid service ID | LOGIN_FAIL_INVALID_SERVICE_NAME<br>Failed. Invalid service name |
    | LOGIN\_FAIL\_INVALID\_USERTYPE<br>Failed. Invalid user type | LOGIN_FAIL_INVALID_USER_TYPE<br>Failed. Invalid user type |
    | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>Failed. Invalid service ID | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>Failed. Invalid service name |
    | ALL\_CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>Failed. Invalid service ID | ALL\_CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>Failed. Invalid service name |
    | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>Failed. Invalid service ID | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>Failed. Invalid service name |
    | ALL\_CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>Failed. Channel information not found | ALL\_CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>Failed. Invalid service name |
    | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_FORM\_NULL<br>Failed. If the matching request is null | MATCH\_ROOM\_FAIL\_MATCH\_FORM\_NULL<br>Failed. If the matching request is null |
    | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_INFO\_NULL<br>Failed. If the matching information is null | MATCH\_ROOM\_FAIL\_MATCH\_INFO\_NULL<br>Failed. If the matching information is null |
    | FORCE\_CLOSE\_BASE\_CONNECTION<br>Calling close() of BaseConnection on the server | FORCE\_CLOSE\_CONNECTION<br>Calling close() of IConnection on the server |
    | FORCE\_CLOSE\_BASE\_USER<br>Calling closeConnection() of BaseUser on the server | FORCE\_CLOSE\_USER<br>Call closeConnection() on IUser on server |

#### Fix
* Fixed the issue where the onDisconnect callback would be called after the user's status changed when the server was forcibly terminated
* Fixed an issue where the server and Hammer protocol buffers could be incompatible depending on the creation environment when created in different environments
* Fixed an issue where a response could not be received if the internally used seq value exceeded the maximum value during a request

---


### 2.0.0 (December 4, 2024)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-2.0.0.unitypackage)
#### GameAnvil 2.0.0 or later
#### <span style="color: #e11d21">New</span>
###### GameAnvil 2.0 Connector
* A new connector using async await has been released.
* Completion is now detected by the return value, rather than registering a function to handle completion.
* The GameAnvil connection and authentication code is as follows:

```c#
var (err, res) = await connector.ConnectAndAuthentication(
                                             /*host*/"127.0.0.1", 
                                             /*port*/18200, 
                                             "device_id", 
                                             "account_id", 
                                             "password");
```

#### <span style="color: #e11d21">Remove</span>
###### Removed ConnectionAgent
* The distinction between ConnectionAgent and GameAnvilConnector was ambiguous.
* GameAnvilConnector now includes the behavior of the existing ConnectionAgent.
* Other classes have been renamed to reflect this change:
   * UserAgnet has been renamed to GameAnvilUser.
   * ProtocolManager has been renamed to GameAnvilProtocolManager.
   * Components provided for Unity's convenience have also been renamed to GameAnvilManager.
###### Removed Connector.Update
* Now it works automatically inside the Connector.

###### Removed the delegate for receiving a request response
* Request responses are now received as return values.
* Messages received from the server without a request still use the delegate.
    * SetMessageCallback method

#### <span style="color: #e11d21">Change</span>

###### Use ProtoBuffer 4.28.3
* Modified the internal protobuf dependency to 4.28.3.

###### Directly manage UserAgent instances
* Modified to create and use UserAgent instances directly, as shown in the example below.
* Note that UserAgent.Dispose does not automatically log you out. UserAgent.Dispose only releases objects managed internally.
```csharp
using var myUser = new GameAnvilUser(connector, "ServiceName", subId);
```

###### Modified Packet Class Usability
* No longer always attempts to convert to byte[].
* Packet compression can now be added at creation time.
* Packet decompression now happens automatically.

###### GameAnvilConnector Concurrency
* Fixed an issue where sending and receiving multiple requests would only send one at a time.
* Multiple requests can now be sent and received.

###### Removed MultiRequest
* As the MultiRequest feature was removed from the server, the corresponding features were also removed from GameAnvilConnector.
    * `public void Request(List<Packet> packetList)`
    * `public bool Send(List<Packet> packetList)`

###### Messy Overloading Cleanup
* The excessive overloading has been cleaned up and consolidated into a minimal API for each function.

    ``` c#
    // Existing API example
    public void Login(string userType)
    public void Login(string userType, Interface.DelUserOnLogin onLogin)
    public void Login(string userType, Interface.DelUserOnLogin onLogin, Interface.DelUserOnError onError)
    public void Login(string userType, string channelId, Payload payload)
    public void Login(string userType, string channelId, Payload payload, Interface.DelUserOnLogin onLogin)
    public void Login(string userType, string channelId, Payload payload, Interface.DelUserOnLogin onLogin, Interface.DelUserOnError onError)

    // Modified API example
    public async Task<ErrorResult<ResultCodeLogin, LoginResult>> Login(string userType, string channelId, Payload? requestPayload = null)
    ```

#### <span style="color: #e11d21">Fix</span>
* Fixed an issue where the connection would sometimes not work properly in fast internet environments.

### 1.4.0 (2023.12.13)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.4.0.unitypackage)

#### New
Support for compressed packets in payloads. Updated to protobuf 3.24.1 and improved to not require index to be specified when registering a protocol.

#### Change
Modified to give a Login failure response instead of a SystemError response if you enter the wrong ChannelId at login.

#### Fix
Fixed an issue that does not disconnect in CONNECT_ALREADY_REQUEST state.

### 1.3.0 (2022.12.27)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.3.0.unitypackage)

#### GameAnvil 1.3.0 or later

#### New
New features added including fast connectivity, changing log levels, and synchronization. New features are available through the GameAnvil connector component.
###### Fast connectivity
Fast connectivity allows three processes to access, authenticate, and log in to a GameAnvil engine-based server in one method call.
###### Feature to change log level 
You can adjust the frequency of the provided logs by setting the log level to info, debug, warn, error, etc. through the Examiner window of the dedicated component.
###### Provide synchronization components
The provision of components greatly enhances the convenience of integrating with the Unity engine. Even without the complex logic implementation of the server, various elements of the game object are automatically synchronized between remote clients.
 
* Create GameObject, automatically synchronize destruction
* Synchronize Animation 
  * You can synchronize the animation of the game object with the synchronization of the animation parameters.
* Synchronize Transform 
  * Synchronize transform elements of game objects such as Position, Transform and Scale.
  * It detects any deformation through any path other than the deformation through the code.
* Synchronize RigidBody and RigidBody2D 
  * Synchronize by sending the rigid state calculated by the local client to the remote client.
  * It is optimized not to send packets if there is no change in the rigid state.
* Feature to synchronize Custom value 
  * You can set and use custom values in key-value pairs on a room-by-room basis.
  * It supports CAS method value setting, making timing issues easy to resolve.

#### Fix

* Fixed an issue where resending a match request after a successful match would incorrectly record whether a room was entered or not.
* Fixed client not to automatically disconnect when setting packetTimeout option to 0
* Fixed an issue with garbage being generated when calling update() method
* Fixed an issue when an error occurs without registering a listener for the error
 
#### Change

* API changes: Modified to receive ErrorCode as a name change and factor

| dcgm 버전이 3.1.7에서 3.1.8로 변경되었습니다. | CentOS 7.9 - Container (2023.08.22) |
|--|--|
| OnTimeout(msgId) | OnError(msgId, ErrorCode) |
| OnCustomTimeout(command) | OnCustomError(command, ErrorCode)| 

* Add ErrorCode

| Name | Description |
|--|--|
| ErrorCode.PARSE_ERROR | Indicates protocol parsing failed|

* Add ResultCode to Custom Protocol Callback

| Name | Description |
|--|--|
| ResultCode.PARSE_ERROR | Indicates packet parsing failed |
| ResultCode.SYSTEM_ERROR| Server system error |
| ResultCode.TIMEOUT | Timeout |
| ResultCode.SUCCESS | Successful |

* Add Exception

| Name | Description |
|--|--|
| ParseInvalidProtocol | Occurs when protocol parsing fails |

* Improved logging systems
  * Added the feature to optionally set ping pong log output
  * Modified a situation where message information is output from the log to output a name instead of message ID
  * Added feature to adjust log levels to output


* Added optional requestDirect to send packets immediately upon request
* Added option of setArgumentDelegateOrListenersOnError 

---

### 1.2.3 (2022.01.28)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.3.unitypackage)

#### GameAnvil 1.2.0 and later

#### Fix
* Fixed problems that fail to process compressed packets sent from the server and cause errors

------
### 1.2.2 (2021.11.30)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.2.unitypackage)

#### GameAnvil 1.2.0 and later

#### Fix

* Fixed a problem in which IsJoinedRoom() changes to false if you call MatchRoom while entering the room and fail

------
### 1.2.1 (2021.08.10) 

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.1.unitypackage)

#### GameAnvil 1.2.0 and later

#### Fix

* Fixed a bug in which OnDisconnect is called twice when SocketException occurs

------

### 1.2.0 (2021.07.13) 

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.0.unitypackage)

#### GameAnvil 1.2.0 and later

#### Change

* Changed the return type of Send() to void
* Changed Config.defaultReqestTimeout Defaults to 3 Seconds
* If Request() or other API calls are transferred to a factor, changed the response to only the callback transferred to a factor
    * Added Config.useArgumentDelegateOnly (Default true)
    * If Config.useArgumentDelegateOnly is false, callback handed over as a factor even if Request() or other API calls are handed over as a factor and a pre-registered listener is called at the same time. (Previous version of the operation method)
* Exceptions
    * Added NotConnected 
    * Added NotAuthenticated 
    * Added NotLoggedIn 
    * Added MatchUserInProgress 
    * Added MatchPartyInProgress 
    * Added MatchUserNotInProgress 
    * Added MatchPartyNotInProgress 
* ConnectionAgent
    * NotConnected Exception occurs when sending a request without connecting to the server
    * Not Authenticated Exception occurs when Sending a Request Without Authenticated
    * Excluded PauseClientStateCheck(), ResumeClientStateCheck() .
    * Removed LoginedUserInfo.payload
    * Added GetAllChannelInfo()
    * Added GetChannelCountInfo()
    * Added GetAllChannelCountInfo()
* UserAgent
    * NotLoggedIn Exception occurs when Sending Request Without Login
    * Removed LoginInfo.Message 
    * Added GetChannelInfo()
    * Added GetAllChannelInfo()
    * Added GetChannelCountInfo()
    * Added GetAllChannelCountInfo()
    * Added matchingGroup parameter to CreateRoom()
    * Added matching UserCategory parameter to JoinRoom()
    * Added matchingUserCategory parameter to matchingGroup of MatchRoom().
    * Added matchingGroup parameter to MatchUserStart()
    * Added matchingGroup parameter to MatchPartyStart()
    * Added IsUserMatchInProgress(), IsPartyMatchInProgress(), IsMatchInProgress() 
    * UserMatchInProgress exception occurs when making an api call except for UserMatchCancel during UserMatch
        * UserMatchNotInProgress exception occurs when calling UserMatchCancel when not during UserMatch
        * UserMatchInProgress exception occurs when calling api except PartyMatchCancel during PartyMatch
        * PartyMatchNotInProgress exception occurs when calling UserMatchCancel when not in PartyMatch
* ResultCode
	* ResultCodeConnect
    	* Added CONNECT_FAIL_INVALID_IP 
	* ResultCodeAuth
    	* Removed AUTH_FAIL_MAINTENANCE  
	* ResultCodeCreateRoom
        * Added CREATE_ROOM_FAIL_CREATE_ROOM_ID 
        * Added CREATE_ROOM_FAIL_CREATE_ROOM
    * ResultCodeChannelInfo
        * Added CHANNEL_INFO_FAIL_NO_CHANNEL_INFO 
        * Added CHANNEL_INFO_FAIL_INVALID_SERVICE_ID 
        * Added CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND 
    * Added ResultCodeAllChannelInfo 
    * Added ResultCodeChannelCountInfo 
    * Added ResultCodeAllChannelCountInfo 
    * ResultCodeChannelList
        * Removed CHANNEL_LIST_FAIL_INVALID_SERVICEID 
        * Added CHANNEL_LIST_FAIL_NO_CHANNEL_LIST
    * ResultCodeJoinRoom
        * Added JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM 
        * Added JOIN_ROOM_FAIL_ALREADY_FULL 
        * Added JOIN_ROOM_FAIL_ROOM_MATCH 
    * ResultCodeLogin
        * Removed LOGIN_FAIL_MAINTENANCE
    * ResultCodeMatchUserCancel
        * MATCH_USER_CANCEL_FAIL_CONTENT -> MATCH_USER_CANCEL_FAIL name changed
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
        * Added NAMED_ROOM_FAIL_CREATE_ROOM 
    * ResultCodeDisconnect
        * Removed FORCE_CLOSE_MAINTENANCE 
        * Added FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID.
        * Removed FORCE_CLOSE_DISCONNECT_ALARM
        * Added FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT 
        * Added FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION 
    * Added ResultCodeSessionClose 

------
### 1.1.6 (2023.01.20)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.6.unitypackage)

#### GameAnvil 1.1.0 and later

#### Fix
* Fixed issues that can be blocked when connect() is called with useIpv6 option enabled 

------

### 1.1.5 (2022.01.28)

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.5.unitypackage)

#### GameAnvil 1.1.0 and later

#### Fix
* Fixed problems that fail to process compressed packets sent from the server and cause errors

------
### 1.1.4 (2021.11.30) 

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.4.unitypackage)

#### GameAnvil 1.1.0 and later

#### Fix
* Fixed a problem in which IsJoinedRoom() changes to false if you call MatchRoom while entering the room and fail

------
### 1.1.3 (2021.08.10) 

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.3.unitypackage)

#### GameAnvil 1.1.0 and later
#### Fix
* Fixed a bug in which OnDisconnect is called twice when SocketException occurs

------

### 1.1.2 (2021.04.15) 

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.2.unitypackage)

#### GameAnvil 1.1.0 and later
#### New
* Added ConnectionAgent.PauseClientStateCheck() to pause the ClientStateCheck feature of GameAnvil Server. It calls when the message loop stops working, such as the app entering the background.
* Added the connection agent.ResumeClientStateCheck() to re-operate the paused ClientStateCheck feature. It calls when the message loop works again, such as when the app returns in the background.
* Added Singleserver.SetOnPauseClientStateCheck().
* Added Singleserver.SetOnResumeClientStateCheck().
#### Fix
* Fixed a bug with the ResultCodeDisconnect value crossing to 0 when force is false on OnDisconnect

------



### 1.1.1 (2021.04.07) 

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.1.unitypackage)

#### GameAnvil 1.1.0 and later
#### Change
* Added ContainsListener overloading to check individual registration. 
* Added RemoveAllListenersForMsg, RemoveAllListenersForMsgId to delete all listers by message
* Added ContainsUserListener, ContainsUserNotificationListener, ContainsUserErrorListener.
* Added ContainsConnectionListener, ContainsConnectionNotificationListener, ContainsConnectionErrorListener, ContainsConnectionErrorListener.
#### Fix
* Fixed bugs that do not initialize some information in the UserAgent in situations such as forced shutdown, logout, or login failure

------



### 1.1.0 (2020.12.18) 

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.0.unitypackage)

#### GameAnvil 1.1.0 and later
#### Change
* Full transition to .Net 4.5 and later support
	* Replaced all of your libraries with the latest version for .Net 4.5
* ConnectionAgent 
	* IsConnected(): Added connection verification API.
	* IsAuthenticated(): Added an authenticated API.
* SingleServe’s GameAnvil.Action -> GameAnvil.Handler named changed.

------

### 1.0.0 (2020.08.31) 

#### [Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.0.0.unitypackage)

#### GameAnvil 1.0.0 or later
#### Change

* Removed MoveService
* Removed the Reconnect and Retry feature
* Added the API that receives Type as a parameter for GetMessage of PacketHelper.
* Applied a unique number to each ResultCode

* Added and renamed ResultCode

	* <span style="color:#eb6420">Currently, there is a problem that FORCE_CLOSE_DUPLICATE_LOGIN case has FORCE_CLOSE_BY_NEW_CONNECTION. To be revised later.</span>

#### Fix

* Fixed an issue in which the isLogin of UserAgent returns true even when disconnected



#### Detailed ResultCode changes

* ResultCodeAuth
	* Add
		* AUTH_FAIL_INVALID_ACCOUNT_ID
* ResultCodeLogin
	* Rename
		* LOGIN_FAIL_EMPTY_SUB_ID -> LOGIN_FAIL_INVALID_SUB_ID
	* Removals
		* LOGIN_FAIL_LOGINED_SAME_SERVICE
	* Add
		* LOGIN_FAIL_INVALID_USERID: An invalid user ID.
		* LOGIN_FAIL_LOGINED_OTHER_USER_TYPE: Another UserType is logged in with the same account ID.
		* LOGIN_FAIL_LOGINED_OTHER_DEVICE: Another DeviceId is logged in with the same account ID.
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
		* NAMED_ROOM_FAIL_INVALID_ROOM_NAME: When the wrong room name is sent.
* ResultCodeDisconnect
	* Change Name
		* FORCE_CLOSE_SYSTEM -> FORCE_CLOSE_SYSTEM_ERROR
		* FORCE_CLOSE_CONTENT -> 
FORCE_CLOSE_BASE_CONNECTION: Calls the close() of BaseConnection from server 
FORCE_CLOSE_BASE_USER: Calls the closeConnection() of BaseUser from server
	* Add
		* FORCE_CLOSE_INVALID_NODE: When GameNode is changed to the invalid status.
		* FORCE_CLOSE_USER_TRANSFER_FAIL: When user transfer fails.
		* FORCE_CLOSE_USER_TRANSFER_ERROR: When a system error occurs during user transfer.
	* If it is added but the client cannot receive it, the server determines that the client is already disconnected and forcibly disconnects the client and uses the code below. As the client is disconnected from the server, the code below cannot be received. 
If you received this code, please contact the GameAnvil development team.
		* FORCE_CLOSE_BY_NEW_CONNECTION: When a new login request is received from the same account information. 
		* <span style="color:#eb6420">Currently FORCE_CLOSE_DUPLICATE_LOGIN case has a problem with this code coming over. 
To be revised later.</span>
		* FORCE_CLOSE_DISCONNECT_ALARM: When the client does not respond to the server's status check.
		* FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL: When the client does not respond to the server's status check.
		* FORCE_CLOSE_GHOST_USER: If it is a ghost user.


-----



### 1.0.0-EA2 (2020.07.07)

#### C-Sharp
##### Change

* Name change: Gameflex - GameAnvil
* RemoveAllListeners() 
	* Added bool parameter.
	* If false, userListener and connectionListener are not removed.
	* defalult = true,
* Added RemoveAllUserListeners().
* Added RemoveAllConnectionListeners().

-----

### 1.0.0-EA (2020.06.29)

#### C-Sharp
##### Change

* Name change: Tardis - Gameflex
	* TardisSocket -> Socket
	* SessionAgent -> ConnectionAgent
* Type change
	* Changed the type of UserId from string to int
	* Changed the type of SubId from string to int
	* Changed the type of RoomId from string to int
* CreateUserAgent(), GetUserAgent()
	* Changed parameter `string SubId ` -> `int SubId`
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
	* Added CreateRoom parameter `string roomName`.
	* Added onCreateRoom parameter `string roomName`.
	* Added onJoinRoom parameter `string roomName`.
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

-----



### 0.12.1.1 (2020.06.23)

#### C-Sharp

##### Change

* Fixed an issue in which an exception would occur when ProtocolManager.unregister is used

-----



### 0.12.1 (2020.04.06)

#### C-Sharp

##### Change

* Applied the XML document comment
* Agent.LvNetLogger and Logger removed.
* Moved Agent.ContainsListener(int customMsgId) to-> AgentHelper
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

-----



### 0.12.0 (2020.02.14)

#### C-Sharp

##### Change

* Parameter name change: msgIndex -> customMsgId
	* Agent
		* public bool ContainsListener(int customMsgId)
	* SessionAgent
		* public void AddCustomMsgListener(int custonMsgId, Action<SessionAgent, Packet> listener)
		* public void RemoveCustomMsgListener(int customMsgId, Action<SessionAgent, Packet> listener)
	* UserAgent
		* public void AddCustomMsgListener(int custonMsgId, Action<UserAgent, Packet> listener)
		* public void RemoveCustomMsgListener(int customMsgId, Action<UserAgent, Packet> listener)
	* Payload
		* public Packet getPacket(int customMsgId)

* Packet
	* public Packet(int descId, int msgIndex, byte[] buffer) removed
	* public Packet(int msgIndex, byte[] buffer) removed
	* public string GetFileDescriptorName() removed
	* public void SetDescId(int descId) removed
	* public int GetDescId() removed
	* public int GetMsgIndex() removed
	* public Packet setTimeout(int timeout) removed
	* public int GetTimeout() removed

##### New

* ResultCodeMatchRoom
	* MATCH\_ROOM\_FAIL\_UNKNOWN\_ERROR = 6,
	* MATCH\_ROOM\_FAIL\_MATCHED\_ROOM\_DOES\_NOT\_EXIST = 7
* Packet
	* public static Packet CreateWithCustomMsg(int customMsgId, byte[] buffer)
	* public int GetMsgId()
