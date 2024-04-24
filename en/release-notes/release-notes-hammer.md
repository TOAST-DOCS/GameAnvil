## Game > GameAnvil > Release Notes > GameHammer

### 1.4.0 (2023.12.13)

#### New

* Improved usability of scenario testing
  - Added a way to pass methods directly through ScenarioActor
  - Added a way to attach annotations to register methods as handlers
  - Added the feature to measure how long it takes for a scenario test to request and receive a response from the server and output it to the result log
  - Added the feature to manage state movements in one place by allowing scenarios to be created at once without setting state changes inside the state
  - Modified the scenario test to make it known through the log when the server is not running
  - Improved the feature to send and query scenario test logs
* Added the feature to support compressed packets in Payload
* Updated to the latest version of protobuf 3
* Improvement has been made that you don't need to specify an index when registering Protocol

#### Change

* Modified to give a Login failure response instead of a SystemError response when entering the wrong ChannelId at login
* Added Overloading to ConfigLoader to allow you to load CustomConfig from the received string as a parameter
* Changed the order of parameters of ScenarioActor.connect().
    * Modified callback to be preceded so that it is unified with other APIs
* Improved packet code/decode performance

#### Fix

* Fixed issues that do not leave a log if an exception occurs in the state's onEnter, onExit
* Fixed issues that leave warning logs if you send a request at the end of the test and do not wait for a response
* Fixed issues where Timer does not work on non-Connector
* Fixed bugs that do not shut down when testTimeout has passed but there are still scanarioActors that need to be started

---

### 1.3.0 (2022.12.27)
#### New
* Added the feature to load settings through vmOption

---
### 1.2.1 (2021.11.30)

#### New 

* Added SecureSocket support feature.
	* Added useSecureSocket option to RemoteInfo class (default : false)
    * Added useSecureSocket field to targetServerList item of GameHammerConfig.(default : false)
	* Added overloading to Tester.Bulder.addRemoteInfo() receiving useSecureSocket.

---

### 1.2.0(2021.07.13)
#### Change
* Organized the package structure
	* Grouped packages for internal use with gameanvilcore.
* ResultCode
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
    * Added FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID
    * Removed FORCE_CLOSE_DISCONNECT_ALARM
    * Added FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT
    * Added FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION
  * Added ResultCodeSessionClose

### 1.1.2 (2021.11.30)

#### New 

* Added SecureSocket support feature.
	* Added useSecureSocket option to RemoteInfo class (default : false)
    * Added useSecureSocket field to targetServerList item of GameHammerConfig.(default : false)
	* Added overloading to Tester.Bulder.addRemoteInfo() receiving useSecureSocket.

---

### 1.1.1 (2021.04.16)

#### New 

* Added `Connection.setSendPingPaced()` to enable ping feature on and off.

#### Fix

* Fixed bugs that do not apply pingIngerval in config
* Fixed config not to ping if pingIngerval is 0

---

### 1.1.0 (2021.04.15)

#### Change

* Raised to 1.1.0 to match the version on the server.

#### New

* Added sendPauseClientStateCheck()
* Added sendResumeClientStateCheck()
* Added feature to enable health check responses from the server to be turned on and off

---

### 1.0.2 (2020.02.10)

#### Fix
* Fixed an issue where if a client did not send any packets to a game node for a specified amount of time (default 10 seconds), the server would send a status check request to the client, and GameHammer would incorrectly respond to this status check request, resulting in a disconnect.
* Fixed an issue that caused packetSeq to overflow and not respond from the server when the number of requested packets became very large due to prolonged scenario testing.  
* Fixed problems that increase packetSeq even when sent

#### Change
* Enhanced log content.
    * Added accountId, userId
    * Added last received packet.
    * Excluded from Ping/Pong last packet
* To reduce the size of the packet 
    * limited packetSeq maximum value to 16383
    * limited subId maximum value to 127, minimum value to 1

---

### 1.0.1 (2020.12.28)

#### Fix
* Fixed a bug where all the overlapping queues were released at the first response when waitFor was used overlapping for the same message
* Fixed a bug where getPayloads() in ResultAuthentication returns null
* Fixed an issue that HandlerPing.onPingTime() occurs intermittently in HandlerPing.onPingTime() at the end of the test
* Fixed an issue that ConcurrentModificationException occurs intermittently in Statistics.record() during testing

#### Change
* Changed the output log from error to warn if the GameHammerConfig.json file does not exist.

---

### 1.0.0 (2020.12.18)

#### Fix
* Fixed an issue that failed to test for a long time in EA version.
* Significantly improved TPS performance over EA version (approximately 2x)
* Added missing features.
    * Connector
        * getChannelInfo
        * addListenerAdminKickoutNoti
        * addListenerForceCloseNoti
        * addListenerDisconnect
    * User
        * moveChannel
        * snapShot
        * addListenerMatchPartyStartNoti
        * addListenerMatchPartyCancelNoti
        * addListenerForceLogoutNoti
        * addListenerForceLeaveRoomNoti
        * addListenerMoveChannelNoti
        * addListenerNotice

#### Change
* Tester
    * Sync/Async support for all request method features
        * Sync: A method of receiving Future as a return value when requested, waiting until it is completed with Future.get() and receiving and processing the result.
        * Async : A method of handing over the callback as a factor when requested, and receiving and processing the completion result from the callback.
* Scenario
    * Removed the concept of TRANSACTION and EVENT
        * Instead, use changeState() from each State to go directly to the desired State

#### New
* Added waitForXXX feature so that noti sent from the server can be waited and processed.

---

### 1.0.0-EA (2020.08.03)

#### New
* Tester - Supports syncing feature with servers on behalf of GameAnvil Connector
    * Connection - Support for functions handled by the Connection Agent on the GameAnvil Connector
        * connect
        * authenticate
        * getChannelList
        * send
        * request
        * createUser
    * User - Support for feature handled by the User Agent on the GameAnvil Connector
        * login
        * logout
        * createRoom
        * namedRoom
        * joinRoom
        * leaveRoom
        * matchRoom
        * matchUser
        * matchPartyStart/Cancel
        * moveChannelStart/Cancel
        * send
        * request
        * addListenerMatchUserTimeout
        * addListenerMatchUserDone
* ScenarioTest - Supports scenario testing with Tester. 
    * Scenario Machine - a collection of different states that make up the scenario
    * State - Express a specific state of the entire scenario that you define
    * ScenarioActor - One virtual user performing a scenario