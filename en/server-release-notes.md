## Game > GameAnvil > Release Note



### 1.0.5 (2020.11.17)

#### Fix

- Fixed an issue in which packet header would not restored when responding to the error in RoomMatchReq



### 1.0.4 (2020.10.29)

#### Fix

- Fixed an issue in which NPE (Null Pointer Exception) would occur when no packet was passed to an arbitrary Session
- An issue would occur when using publishToClient (included Notice)
- Fixed an issue in which room destruction would be delayed because of the repeat Timer used by Room
- Fixed an issue in which an exception would occur when de-serializing the Class that has the protobuf structure as a variable
- java.lang.UnsupportedOperationException occurs



### 1.0.3 (2020.10.26)

#### Change

- Reactivated the notification API that was previously excluded from specifications



### 1.0.2 (2020.10.12)

#### New

- Added a feature that is used to allow users to freely set the Node information displayed in Log
- Added a feature that is used to manually enter HostId
- Added a feature that is used to pass information to each HostId from nodeInfoByHostId

#### Fix

- Fixed an issue in which an error would occur while binding IP 0.0.0.0
- Fixed an issue in which the success value would respond as failure when using the /config/get of ConfigModule
- Added the exception handling for the problems that would cause an error while processing Publish Packet when another GameNode is active while a GameNode is already running

#### Change

- Edited so that the value range of ShardIndex that is used in the last digit when creating UserId or RoomId can be specified
- Added the getRemoteAddress value of RestObject
- Edited so that the number of NodeKey requests can be specified



### 1.0.1 (2020.09.07)

### Fix

- Fixed an issue in which Payload would not be passed to the client when onLogin() returned false
- Added the missing part of the log level check statement in front of the Debug and Trace log statements



### 1.0.0 (2020.08.31)

#### New

- Dramatically increased performance and stability (approximately 7 times better than those of version 0.9)
- Fixed issues and optimized several logics that directly affect performance
- Overhauled the GameNode load balancing
- Distributed load to the optimal GameNodes quickly and accurately, reflecting the status of cloud VM
- Introduced Integer type ID
- Optimized the core IDs by replacing them from String to Integer
- Added the API that can be used to allow the server Log level to be changed during runtime
- Log level can be changed while servicing in Node unit

#### Fix

- Fixed a user transfer problem, improved usage, and optimization
- GameAnvil users can easily use the user transfer feature
- Completely fixed uninterrupted patch
- Fixed all the issues in which room transfer and uninterrupted patch would not work correctly
- Fixed an issue in which some Nodes would not correctly appear due to an unknown timing issue when running the server
- Fixed memory leak in which some of the Timer objects of Node persist instead of being disabled
- Fixed an issue in which newly connected user would be unintentionally disconnected while checking the client connection

#### Change

- Changed the Node name into more intuitive one as shown in the table below

| v1.0       | ~ v0.10       | Required | Feature                                       | Implement content | Network access     |
| ---------- | ------------- | ---- | ------------------------------------------- | ----------- | ----------------- |
| Gateway    | Session       | Required | Handles client connection and authentication             | Possible        | public            |
| Game       | Space         | Required | Handles content as actual game server            | Possible        | private           |
| Support    | Service       | Optiona | Supports independent services as necessary | Possible        | private or public |
| Match      | Match         | Optiona | Performs matchmaking                           | Possible        | private           |
| Location   | Location      | Required | Store and manage user and room location information     | Impossible      | private           |
| Management | Management    | Required | Collect server information and communicate with Admin/Agent        | Impossible      | private           |
| Ipc        | Communication | Required | Process the Inter-process communication of the Gameflex server     | Impossible      | private           |

- Improved the readability and usability of the GameAnvil code
- Effectively improved package structure
- Improved the class inherit/include structure
- Modified naming to make it more intuitive and consistent
- The disable status of Node is subdivided by Invalid and Disable
- Invalid is a state in which the node is temporarily non-responsive
- Disable is a state in which the node is permanently disabled
- The Loopback address is now automatically bound
- Updated all the libraries used by GameAnvil to the latest version
- Integrated entire ErrorCode
- Removed the required ReConnect and MoveService feature
- Added a new timer that is used to measure the next time from the time of running instead of from the time of registering



### 0.10.2 (2020.04.08)

#### New

- GameAnvil API Reference (JavaDoc) Site Open

- Support AOT Instrumentation

- AOT at compile time without having to JIT Instrumentation the engine at runtime

- Start the guide so that it will use G1GC as the official GC

- The most basic VM option

  `java -javaagent:QUASAR_PATH\quasar-core-0.7.10-jdk8.jar=bm -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication`

#### Fix

- Edited the Quasar stack bug
- Fixed an issue in which room matching users would be matched to different rooms
- Fixed an issue in which compressed packets would not be transferred



### 0.10.1 (2020.02.11)

#### New

- Provided the Util class that were provided to users
- Organized the tools that were previously scattered throughout the engine into a single class

#### Change

- Improved the Topic process code performance



### 0.10.0 (2020.02.06)

#### New

- Newly refactored the asynchronous support API that was previously problematic
- Added new asynchronous API as support Redis Cluster and official support Lettuce become available
- Added asynchronous HttpRequest and HttpResponse API
- Support custom protocol based on ByteString
- Can serialize messages in a way the user other than Google protobuf

#### Fix

- Fixed all the issues in which the status of Fiber would be broken, which was the most critical problem in the previous version

#### Change

- Optimized the size of the packet and header used by the engine
- Performance improved by 2.5 times compared to version 0.9
- Embedded Redis Spec-out
- Guided users so that they would need the service provided by infrastructure



### 0.9.9 (2019.10.25)

#### New

- Opened additional port so that WatchDog can be linked in the unit of process

#### Fix

- Fixed the ghost user bug
- Fixed a Concurrent Modification error that would occur in SpaceNode
- Fixed a Comparator error in NodeInfoManager
- Fixed an issue in which the session object of logged out user would not be organized
- Fixed an issue in which matching groups would not be correctly applied to the users in many different channels
- Fixed an issue in which the callback for Timeout would not be correctly received when party matchmaking

#### Change

- Added error code used when a duplicate request is received while room matchmaking is in progress
- MATCH_ROOM_FAIL_IN_PROGRESS



### 0.9.8 (2019.09.05)

#### New

- Added addTopic() and removeTopic() API that is used to add or remove ClientTopic to Session
- Added the Packet TTL feature
- Added a logic that periodically checks and organizes Dangling Location

#### Fix

- Fixed the system so that ForceLogoutNoti would not be transferred to the client when kicking a user in Admin
- Fixed all Spot malfunctions of LocationNode

#### Change

- Changed the system so that ForceCloseNoti would be transferred before the socket is closed when onAuthenticate() callback failed



### 0.9.7 (2019.07.27)

#### New

- Added a logic that is used to regularly check the Rooms that abnormally remain
- As previous user matchmaking may not be canceled and remain when a user matchmaking is requested after logging in again, added a new flag that responds to the status of previous user matchmaking process to re-login response
- Added the Packet Expire feature (default: 30 seconds)

#### Fix

- Fixed an issue in which Connection ID (CID) would be duplicated
- Fixed a bug in which room Fiber would be closed by a disconnected user
- Fixed an issue in which the serviceId field would not be correctly set in the response message if login failed
- Fixed the part where try-catch is missing in the content callback process
- Fixed an issue in which the client would take more than 3 seconds to reconnect
- Fixed an issue in which thread simultaneity would occur when checking ghost users
- Fixed an issue in which room matchmaking would be processed in duplication
- Fixed an issue in which disconnected users would remain in the room

#### Change

- Added a log for every instance of Session Disconnect
- Changed CreateRoom so that it would directly create room in the current node



### 0.9.6 (2019.06.23)

#### New

- Added the canTransfer() interface to User and Room so that the time of transfer could be adjusted in content

#### Fix

- Fixed memory leak that would exist in internal engine code
- Fixed a connection bug of the ZMQ Router and Publisher socket
- Fixed an issue in which Login would be infinitely repeated
- Fixed an issue in which isLogined() API would not work correctly
- Fixed a NodeInfoManager synchronization issue
- Fixed an issue in which user objects would not be properly organized

#### Change

- Changed the name of AsyncAwaitHttpRequest to FiberHttpRequest
- Added an asynchronous function to FiberHttpRequest and HttpRequest
- Added a log related to Location Index
- Changed the SpaceNode distribution method from idle to roundRobin
- Changed the system so that it would sleep only when there is LocationNode in NodeStarter



### 0.9.5 (2019.06.21)

#### Fix

- Fixed an issue in which the corresponding user would not be organized in a paused node when a new user logs in or a user is being transferred while performing uninterrupted patch

#### Change

- Changed the serviceId from string to integer
- Changed the service_code of a protocol to service_id and changed the related code
- Changed the internal implementation and usage of APIs supporting RESTful



### 0.9.4 (2019.05.24)

#### New

- Added a feature that can be used to register the protocol used by content in the file unit
- It is to effectively process internal packets
- Added the MoveService feature

#### Fix

- Fixed an error that would occur in the Comparator used in room matchmaking

#### Change

- By adding the latest login information to the Authentication response so that it could be used in the next login
- Changed setReady() so that it can be explicitly called and passed with the onReady status
- Changed LocationNode so that it can be reset at CommunicationNode's onPrepare



### 0.9.3 (2019.04.10)

#### Fix

- Fixed an issue in which NPE (Null Pointer Exception) would occur when timeout occurred in AsyncAwait.call() API
- Fixed an issue in which Epoll would stop when it was enabled
- Fixed an Invalid System Target Location error that would occur while logging in

#### Change

- ManagementNode can no longer be expanded by engine users
- Use ServiceNode instead
- Changed the system so that the Machine information registration, edit, or deletion request can be instantly applied in Admin



### 0.9.2 (2019.02.11)

#### New

- Added the scheduled notification and scheduled maintenance feature using Admin
- Added the White IP, White User, and White Device features using Admin

#### Fix

- Fixed a piece of code that used to process exceptions in a wrong way in AsyncAwait.run() API
- Changed ManagementNode so that it will be reset before other Nodes
- Fixed an issue in which publish related to some channels would be missing when channel ID was not used
- Fixed an issue in which incorrect error code would be added to header when content returns fail on CreateRoom and JoinRoom
- Fixed memory leak that would occur when creating packet header
- Fixed memory leak that would occur in room matchmaking

#### Change

#### Change

- Added payload to the onLogout() callback
- Removed duplicate fields from some protocols as ResultCode was edited
- Improved the usability of Dynamic Module



### 0.9.1 (2018.11.12)

#### New

- Added MatchNode dedicated to matchmaking
- Added the refill function to user matchmaking
- Added party matchmaking, a new type of user matchmaking
- Added additional information by node type to the Machine of Admin
- Added the onLoginByOtherConnection() callback that is called when the user logs in again using the same DeviceId and UserId

#### Fix

- Fixed a Dynamic Module bug
- Fixed an issue in which Admin would not be connected
- Fixed a reply bug of User
- Fixed a Timer add/delete error
- Applied the previously missing timeout to AsyncAwait.run()
- Fixed an error that would occur when a node is shutdown
- Fixed an issue in which NPE (Null Pointer Exception) would occur when matching information was deleted while not using room matchmaking

#### Change

- Relocated some interface components
- Changed the system so that it uses re-login instead of processing as duplicate logins when the user logs into different DeviceIds
- Modified the payload added by the onMatchRoom() callback of room matchmaking would be included in the flow afterward



### 0.9.0 (2018.06.27)

#### New

- Added the user matchmaking feature
- Added the user and room list per channel feature
- Added Embedded Redis
- Added the Reconnect feature
- (Added the old) Test Agent

#### Fix

- Fixed a duplicate login in the Account unit issue

#### Change

- Changed OracleJDK to AdpotOpenJDK
- Changed the ZMQ communication unit from Node (thread) to process



### 0.8.6 (2018.06.28)

#### New

- Added the Send from Session to Management feature
- Added the Request from Session to Management feature
- Added User MatchMaker
- Added the Channel User List feature
- Added the Custom Serializer feature
- Custom Serializer can be plugged into RoomFinder

#### Fix

- Added an empty string exception process for the publish in which a channel is used as a topic in the channel user manager
- Added an empty string exception process for the publish in which a channel is used as a topic
- Fixed an issue that would be occurred by DefaultRoomFinder when calling delNamedRoomInfo() and delRoomInfo() in an environment that does not use AutoJoinRoom
- Fixed the NullPointerException that would occur due to missing null check in RoomFinder
- Fixed an issue in which a timed out request would not be removed in user matchmaking
- Added an exception that is used to be processed if password is set in redis
- Fixed a setRestIp() error of cfgManagement
- Fixed an issue in which HTTP handling would not be done in the Service node.
- Branch: 0.8.6.2_fix_service_node_rest_handling
- Tag: tardis-0.8.6.2.1
- Fixed an issue related to the set gameId while processing AutoJoinRoom
- Edited MatchMaker and strengthened examples
- Added the previously missing ISession::onDisconnect()
- Fixed a Node information exchange error among Communication Nodes
- Fixed a FindRoomList performance issue
- Fixed a Shutdown bug
- Fixed an issue in which BaseNode would not receive publishToNode messages
- Fixed a FindRoomList performance issue
- Fixed a Shutdown bug
- Fixed an issue in which BaseNode would not receive publishToNode messages
- Fixed a NamedAutoJoinRoom bug
- Changed UpdateRoomInfo/DelRoomInfo so that they can be instantly applied to the corresponding Node
- Fixed an error in which blocking call would be displayed in Fiber

#### Change

- Changed FindRoomList so that it serializes/de-serializes the container for the entire room list
- Changed the existing LocationException, TardisException and others to RuntimeException, organized Exceptions
- Renamed forceLeaveRoom to kickOutRoom
- Renamed forceLogout to kickOut
- Changed the tardisConfig configuration



### 0.8.5 (2017.12.18)

#### New

- Added a feature to be used to authenticate and manage HTTP sessions with the JWT authentication token
- Added a feature to be used to allow multiple services to log in using the same ID
- Added a feature to be used to allow a single GameNode to use multiple RoomTypes
- Added a feature to be used to view all the information of Server on the web (NodeInfoPage)

#### Fix

- Fixed a bug in which timers would not be removed
- Fixed a bug in which onPostLeaveRoom would be called twice

#### Change

- Improved ghost user handling
- Added ghost user logging
- Improved packet compress logic
- Improved RestObject, added a function to be used with it
- Changed the error handling of a packet, it now places error code to protocol header and processes it



### 0.8.4 (2018.01.10)

#### New

- Applied Bootstrap
- Implemented like NGB Bootstrap
- Simplified code by specifying only the class instance of each class in the implementing part
- Added the Multi Dispatch feature that is used to send and receive multiple messages
- Added the ServiceNodeSender (previously CustomServiceSender) publishToLobby method

#### Change

- Renamed AsyncAwait and RAsyncAwait
- Moved the Bootstrap RoomFinder settings to space
- Renamed Cache and Custom Exception (Location, Service)
- Changed the Payload class so that it retrieves the first packet using the getFirst method
- Removed the requestToLobby method from ServiceNodeSender (the requestToNode method works the same)
- Changed the makeDecompress method so that it does not use async when it is called by the Packet class
- Overloaded the reply method of UserSender (Packet list handling)
- Renamed some classes
- AsyncCall -> AsyncWaitingCall
- AsyncRun -> AsyncWaitingRun
- ReverseAsyncCall -> RAsyncWaitingCall
- ReverseAsyncRun -> RAsyncWaitingRun

#### Fix

- Fixed a Session IP error



### 0.8.3 (2017.10.26)

#### New

- Added the REST handling feature
- Implemented the ACL feature based on Path & IP for the REST handling with permissionGroups in TardisConfig.json
- Channel List (channel name list), Channel Information (detailed information of a specific channel)
- Added the requestToCustom and sendToCustom method to Space LobbySender
- Added the "State" item to the Management information of Space node
- Added void onShutdown() throws SuspendExecution to the Ilobby , ISessionLobby, and IService interface
- Added the onShutdown() interface so that contents can disable resource when a node is shutdown
- Added the publishToNode function to ServiceNodeSender

#### Fix

- Changed and added the JMX Management API of Management (SessionGateway, CustomGateway)
- Added the Channel List (channel name list) and Channel Information (detailed information of a specific channel) feature
- Changed the REST handling logic of the Session and Custom node
- Changed the return value of the Gateway node (Session, Custom) isConnectableType method
- Fixed a round-robin issue of SessionGateway and CustomGateway, changed the JMX return information
- Changed the code that uses the key value of cache node targetMap
- Changed the SessionGateway node REST RoundRobin code
- Improved login process (changed related to the Invalid Target Location error log)
- Returns the node if its status is not READY when the room deleting delRoomInfo is called

#### Change

- Integrated Session node and client's End-Point (Session Gateway)
- Integrated the REST End-Points of custom nodes (Custom Gateway)
- Changed code that directly uses Fiber to ReverseAsyncCall
- Added the SpaceSingular Dispatch time stamp
- Register onTimer and periodically check it (every 10 minutes)
- Applied internalLogout to timed out users
- Applied Async timeout
- Added and improved ghost user handling logic, added the needed values to TardisConfig.json
- Changed the result_code format of each Response protocol from the existing integer value to enum, reduced the number of formats. removed the is_payload field
- Renamed classes, packages, functions, and cfg contents
- Changed the class Payload data type to Packet to allow the Request and Response protocols to handle multiple payloads, removed class OutPayload
- Changed the parameter type of the user Transfer-In interface to byte array



### 0.8.2 (2017.09.21)

#### New

- Added the RestObject feature to CustomService
- Added RestHandler to CustomService

#### Fix

- Fixed a bug in which user information would not be deleted from Cache
- Added an exception handling for the NPE (Null Pointer Exception) that would occur in the engine
- Fixed an issue in which CID (Connection ID) would not be correctly applied while transferring packet

#### Change

- Changed the Custom module naming
- Ex) CustomLobby -> CustomService
- Changed some part of the Custom module interface
- Added the serviceId parameter to the sendToUser() and requestToUser() API of CustomServiceSender
