## Game > GameAnvil > Release Note > GameAnvil
### 1.4.2 (2024.02.26)

#### New
* Improved Safe-Pause feature. 
  * Even if Safe-Pause is already in progress, improvements have been made to run new Safe-Pause by designating non-in-progress nodes as origin/destination nodes. 
  * In addition to GameNode, MatchNode also supports Safe-Pause. 

#### Fix
* Enhanced and improved the readability of the engine's logs. 
  * Added more information about the packets sent when a request fails.
  * Separated the markers for system information logs from a single SYSTEM_INFO to SYSTEM_INFO and SYSTEM_WARN.
  * Modified to log which requests failed when individual requests failed in MultiRequest
  * Enhanced logs related to machine-to-machine connection status. 

#### Change
* Modified GameAnvilConfig.json to enable managementPort as well as managementIp.

------

### 1.4.1 (2023.12.13)

#### New
###### Updated engine Protobuf version to 3.24.1
###### Added Protobuf-related convenience features
  * You can use Protobuf directly in send, reply.
  * You can change packets directly to Protobuf (`MyProto.MyMessageResMsg =resPacket.toProtoBufferMessage();`)
  * In most situations, it works normally without calling a retain release.
###### When registering for Protocol, we have improved so that you don't need to specify an index. 
  * Existing numbers are automatically assigned by the engine.

#### Fix
* Improved the server to give an immediate failure response if no target is found when making a request.
    * Improved the engine's request APIs, such as requestToGameUser, to return a failure immediately if the target is not found, instead of waiting for a timeout.
* Enhanced engine log content and readability
    * We systematically separated both DEBUG and TRACE levels in the engine log and organized the contents to make it easier to understand.
    * Enhanced the log content of the engine related to the user.
    * Improved some engine error messages to make them easier to understand.
* Improved management nodes to only run together when a location node is configured.
    * Because the management node runs automatically in the location node configuration location, you no longer need to set it up and manage it yourself and you don't need to worry about the order of operations.
* Improved the response to incorrect channelIDs during the login process.
    * Modified to give a login failure response instead of a vague SystemError response, which is clearer.
* Improved the server to handle errors and pass responses to the client if an unregistered protocol is passed to the game server in the payload.
    * In the case of leaving only the WARN logs on the existing server, there is a problem that the logs may be missed during the debugging process, so we improved it to forward the response to the client.
* Provides new HttpClient2 based on Apache HttpComponent.
    * Because existing AsyncHttpClient open sources are no longer updated, we provide a new library-based HttpClient2 for optional use.
* Support for compressed packets in the payload.
    * It has been now improved that the availability of compressed packets for payloads as well.
* It forwards notifications to the client when matchmaking fails.
    * Improved delivery of appropriate results to clients even when matchmaking fails.

#### Change
* Set
    * GameAnvilConfig.json uses ipcIp and managementIp instead of ip entries.
    * We fixed an issue that did not serialize successfully in BaseChannelInfo. Instead of BaseChannelInfo, you can serialize by calling the setObjectSerializer in GameAnvilServer.
* It does not include Payload when calling matchUser, matchParty. It keeps it on its own.
 * It is fixed so that all location nodes (master nodes) can only send requests to location nodes after clustering is complete. And added the feature to ensure that all location nodes are clustered and to leave an error log if the entire clustering is not completed within a certain time.
 * During user matching, it was changed from the existing method in which matched rooms were created in random channels to the method in which matched rooms were created among the channels requested for matching.
  
------

### 1.3.1 (2023.04.20)

#### New

#### Fix

* Fixed the issue that the channel information of the newly started node is not updated

#### Change

- Changed the Active Status Check API for the Support node that you see in the Console
- Changed the Management node to start only on the server where the location node is set up

------

### 1.3.0 (2022.12.27)

#### New

###### Syncing with Console 1.3 
GameAnvil 1.3 works perfectly with the completely new Console 1.3. through the console, you can intuitively manage the status of each node and issue commands. You can conveniently try features such as proceeding with SafePause or creating and removing specific nodes through the console.
</br> ※ Earlier versions of GameAnvil are not available in Console 1.3.

###### Easy-to-use synchronization feature
The convenience of integrating with the Unity engine has been greatly enhanced by the provision of synchronization components. Without the complex implementation of logic on the server, various elements of the game object are automatically synchronized between remote clients.

* Create GameObject, automatically synchronize destruction
* Synchronize Animation
* Synchronize Transform
* Synchronize RigidBody and RigidBody2D
* Custom value synchronization feature

###### Optimize Performance
GameAnvil 1.3 improves packet processing by more than 5% compare to previous versions. It operates at a faster rate by optimizing the internal action code.

#### Fix

- Fixed Safe Pause-Related Bugs
  - We have fixed the issue of matchmaking to the room on the origin node during Safe Pause.
  - Fixed MatchRoomException issue when simultaneously performing stress tests and Safe Pause
  - Fixed the problem of NPE occurring due to TimerObject when transmitting a room during the Safe Pause process.
  - Addressed issues where room match making fails when proceeding with Safe Pause
  - Fixed the issue of creating a room with the same ID as that room on the destination node of the room sent through Safe Pause
    - Fixed the location node to be responsible for creating room IDs
  - Additionally, bugs related to user transfers and room transfers were fixed.
    - Fixed problem that does not roll back when a room transfer fails 
    - Fixed the result packet to be forwarded with a failure error code when the room transmission fails
    - When the room transmission fails, the room and users created were inquired at the node and fixed to organize themselves without organizing them.
  	- Fixed the likelihood that some messages sent by clients will be lost during the room transfer process
    - Fixed the situation where the room remains on the existing node and the user can enter when the transfer to another node is completed
      - The rooms you are transferring are now removed from the matchmaker's room list
      - Fixed to avoid entering an existing room where the transfer has been completed and needs to be removed, or a room where the transfer has failed
    - Fixed a problem where no user objects are found on the destination node when reconnected before the user transfer is complete
    - Fixed not to check the state value for the target node during user transfer
    - During room transfer, change the user's transfer-in related callback function to be called

- Fixed the failure of room matchmaking when proceeding with Safe-Pause
  - Add TRANSFER_IN_PROGRESS to the type of room condition being transferred
  - Fixed the transferred room so that it does not change to the PAUSE state
  - Fixed the room entry to stop processing and proceed with the transfer if the room is being transferred when the room entry is about to be processed by the room fiber
	
* Fixed bugs related to Room and matchmaking
  - Fixed an issue that causes an exception when a room is shut down after registering a room created with the CreateRoom method for room matchmaking
    - Fixed not to set match user categories for game users if no match room is found during room match

  - Fixed a problem that allows new users to enter the room with timing issues
    - Has to be checked whether it is an empty room or not when the user enters the room
    - Organize channel information and cancel Match Refill immediately after the last user leaves the room

  - Fixed a Server Error when a Client Calls to LeaveRoom
    - Changed UserPostLeaveRoom packet forward object to game user fiber
  
  - Added ClosedRoomException, an exception to API calls using a room when the room is closed

* Fixed Gateway and Session-Related Bugs
  * Fixed issue of session object of gateway node being leaked to transfer state

  - Fixed problems that can remain unremoved if an exception occurs during the session deletion process

* Fixed node-related bugs
  - Fixed problems that leave an error or warning log on the rest of the server when shut down any node
    - Fixed to share shut down node information between servers
    - Added API to clear shut down node information

  - Fixed possible problems when sending packets from another node to a node in the shutdown state
    - There will be a one-second delay from the end notice to the actual end

* Unknown error
   * Fixed issue that does not receive a response to a request from the client

  - Fixed problems where all nodes are treated as Ready even if the server is running with an incorrect service name
    - Changed the logic to increase the number of nodes to the correct point of time
    - Fixed node manager to exit immediately if you attempt to run a node using an incorrect service name


  - Fixed bugs that can be forced to terminate connections during duplicate logins
    - Fixed not to close the session in duplicate if you log out an existing user when DeviceId is different and Connection ID and Gateway Node ID are the same

  - Fixed ghostTimeout values to apply correctly

  - Add constraints so that the ghotsTimeout value is at least 3 seconds greater than the demandClientStateCheck value

#### Change

- Advanced Safe Pause
  - Changed to select both the origin node and the destination node
  - Changed the origin node and destination node not selected at the start of Safe Pause
  - Enhanced report feature: Add the feature to generate reports in json format other than HTML format

- Changed the name of the Safe Pause-related protocol and modify the behavior of the ReportSender in a request manner
  - Before change: PubNonStopPatchReportStartCommand / PubNonStopPatchReportStartNode
  - After change: NonStopPatchReportStartCommandReq / NonStopPatchReportStartCommandRes / PubNonStopPatchReportStartNode		

- Changed to handle exceptions when a request is sent to the same fiber

- Changed how to use BaseRoom
  - Added Overloading of the onLeaveRoom method to BaseRoom
  - Changed usability by removing parameters from onPostLeaveRoom in BaseRoom

- Changed how to use BaseUser
  - Added an onPostLeaveRoom method called after the user leaves the room completely

- Disconnected status is subdivided into isDetectedDisconnected, and isCompleteDisconnected
  - isDetectedDisconnected is the state value when the server determines that the client is disconnected
  - isCompleteDisconnected is the status value when the server has finished disconnecting

- Unique HostId generation logic was enhanced
  - Fixed to create HostId based on the subnet mask entered by the user

- Updated latest libraries used by the engine

- Added API getUserTimer, getRoomTimer to return a reference to UserTimer, RoomTimer

- Delete DNS related setting

---

### 1.2.0 (2021.07.13)

For more information, see the [Deployment Note](https://nhnent.dooray.com/share/posts/sGAj_STlTEWDr5LPgKgIhg)

#### New

* Apply User Licenses
  
* [GameAnvil User Licenses ](https://gameplatform.toast.com/kr/services/gameanvil/license)
  
* Supports node-by-node drive and scaling

  * Added feature to raise new nodes at runtime
  * Improved clustering more reliably than before
  * Improved node behavior according to node status
  * Runtime can be run or shut down on a node-by-node basis

* Simplified bootstrapping
  * Set to GameAnvil Bootstrap Deprecate. To be deleted in future versions.
  * Run a GameAnvil server using a GameAnvil Server instance instead of a GameAnvil Bootstrap.
  * Improved usability to read BaseClass from GameAnvil using the provided annotation.

```java
// Register GameNode 
@ServiceName("GameNode") 
public class GameNode extends BaseGameNode { 
} 
 
// Register GameNode
@ServiceName("GameNode") 
@UserType("GameUserType1") 
@UseChannelInfo 
public class GameUser extends BaseUser { 
}
```


* Support for asynchronous MySQL queries
  * Add Jasync-sql / XDev API class for asynchronous sql processing
  * Supports users handle asynchronous sql easily
  
* User Guide Documents enhanced
  * We upgraded the quality and quantity of documents by modularizing several important topics that were implicitly listed in one document.
  * With tutorials, you can develop your own easy and simple but pretty cool jigsaw puzzle game (server and client). 
  * We made up for all the missing or missing parts in the JavaDoc API reference. Also refined all the sentences that might be ambiguous or misinterpreted.



#### Fix

* Refactoring channel information management and synchronization features
  * Improved channel information to efficiently manage it
  * Improved features to send channel information to clients
  * Added the feature to get the number of rooms and users in the channel
  * Improved to no longer using the channel feature if you do not set the channel ID (i.e., using ""), if you want to use the channel, you must use at least one valid string as the channel ID.

* Fixed a bug that does not release outPayload when creating a room
* Fixed bugs where room matchmaking continues to fail until reconnected from clients on the same account
* Fixed bugs that did not send a response to the client if an error occurred during user matchmaking
* Corrected publish API
  * Improved usability
  * Fixed a bug where publish messages were sent to nodes that the user did not intend. In addition, we improved the usability of publish APIs.
  * Fixed a bug where the pushlist message could not find the target node when using Hangul as the channel ID.



#### Change

* Default port has been changed for the cloud ACL environment.

| Name | Port | Previous port |
| --- | --- | --- |
| ipcPort | 18000 | 16000 |
| publisherPort | 18100 | 13300 |
| gateway tcp | 18200 | 11200 |
| gateway web | 18300 | 11400 |
| management | 18400 | 25150 |
| support | 18600~18999 | Define User  |
| GameAnvil Agent | 19080 |  |
| GameAnvil Admin | 19090 |  |

* Admin is no longer supported, so we no longer use our own DB inside GameAnvil.

* Dynamic Module is no longer supported.

* Replaced the ZMQ Library
	
	* Replaced libzmq and jzmq combinations with Jeromq
	
* Improved room matchmaking refactoring and usability
  * Improved the flow of room match making
  * Stress test can be conducted by solving the problem of not being able to find a room during the stress test
  * Room attendance information manage feature was added so that users do not have to implement room attendance management features.
  
* Optimized authentication, login and connection termination code

    * Moved the callback that Connection was calling to Session
    * Improvements have been made to facilitate identification of the cause of DisconnectAlarm.

* Clean up user-provided topics

| Name | Descriptions | Default Registration Location |
| --- | --- | --- |
| GameAnvilTopic.GAME_NODE | All game nodes | GameNode |
| GameAnvilTopic.GATEWAY_NODE | All Gateway nodes | GatewayNode |
| GameAnvilTopic.SUPPORT_NODE | All port nodes | SupportNode |
| GameAnvilTopic.ALL_CLIENT | All connected clients | Session |
| GameAnvilTopic.ALL_GAME_USER | All game user objects on the game node | GameUser |


* Optimized exception handling and method signature


  * Fixed to clean up indiscriminately used exception handling and method signatures and hand users only the information they need

* Added PATCH method to HttpRequest
* The feature setting name that allows SSL to be used for testing without a certificate has been changed from useSelf to useSelfSignedCert.

---

### 1.1.12 (2021.06.07)
#### Change

* Added API to SupportNode to receive gateway access information
    * Provides a list of Gateway access information using the BaseSupportNode.getGatewayConnectionInfoList function

```json
// Information list converts to Json 
{ 
  "header": { 
    "isSuccessful": true, 
    "resultCode": 0, 
    "resultMessage": "SUCCESS" 
  }, 
  "body": [ 
    { 
      "ip": "127.0.0.1", 
      "dns": "", 
      "connectionInfo": { 
        "TCP_SOCKET": 11200,    // Provide port information according to the corresponding Connection Type
        "WEB_SOCKET": 11400 
      } 
    } 
  ] 
}
```
---

### 1.1.11 (2021.05.06)

#### Change

* Add API to HttpReqest to enable PATCH method
    * httpRequest.PATCH()
    * httpRequest.PATCH(AsyncHttpCompletionHandler asyncHttpCompletionHandler)
    * httpRequest.PATCHAsync()

---

### 1.1.10 (2021-04-21)

#### Fix

* Password support APIs added that were missed in RedisSingle

---

### 1.1.9 (2021-04-16)

#### Fix

* Fixed the idleClientTimeout check to also stop when receiving the client's PauseClientStateCheck.
* Fixed the unit of pauseTime received from the client's PauseClientStateCheck to count in seconds.

---

### 1.1.8 (2021-04-15)

#### New

* Added features to receive the client's PauseClientStateCheck and not check the client's status for as long as the time it was entered.
* Added the feature to resume health checks by receiving the client's ResumeClientStateCheck.
* Added setting value to GameAnvilConfig.json's Gateway to limit PauseClientStateCheck-related tolerances
    * pauseClientStateCheckMaxDuration = Limit the time value sent by the client to that value if it is greater than that value
    * pauseClientStateCheckMinDuration = Limit the time value sent by the client to that value if it is less than that value

---

### 1.1.7 (2021-04-02)

#### Fix

* Fixed a problem with /game-data/get, in which GameData values are transferred to the String instead of to the JsonObject

---

### 1.1.6 (2021-03-30)

#### Change

* Dynamic Module feature has been specified out of GameAnvil and deleted.
  * For existing users, we create and share the Dynamic Module Source Disclosure and User Guide. Please refer to this guide to add to use Dynamic Module Source to your non-GameAnvil user project.
  * [GameAnvil-Guide/81 Dynamic Module 
Source delivery and usage due to spec out](https://nhnent.dooray.com/share/posts/xtwdO6FJTXmr7l28cZSM-w)

---

### 1.1.5 (2021-03-19)

#### Change

* Deleted GameAnvil DB and Admin feature (excluding GameData, Dynamic Module )

    * GameAnvil engine end no longer uses DB.
    * Note: [GameAnvil-Guide/76 Admin 1.1.0 -> 1.1.1 Migration document](https://nhnent.dooray.com/share/posts/GIHoRQ8kTjSi7D4wQjUcWw)
    * All the Admin features that previously used DB have been moved to GameAnvil Admin.

```json
"management": { 
  "nodeCnt": 2, 
  "restIp": "127.0.0.1", 
  "restPort": 25150, 
 
  // Delete 
  //"db": { 
  //  "user": "root", 
  //  "password": "1234", 
  //  "url": "jdbc:h2:mem:gameanvil_admin;DB_CLOSE_DELAY=-1" 
  //} 
}
```

---

### 1.1.4 (2021-03-18)

#### Fix

* Fixed the problem of not joining intermittently created rooms when applying MatchingGroup to room matching

---

### 1.1.3 (2021-02-05)

#### Fix

* Fixed issues that do not recover from disabled state upon instance re-start
    * It can be reproduced with low possibility, but it is restored when re-started

* Fixed a problem with Double.infinity errors when using nodeInfoByHostId in Management Node

---

### 1.1.2 (2021-01-07)

#### Fix

* Fixed an issue where DynamicModule fails to call properly due to missing exception handling for code calls that may cause SuspendExcussion exceptions

---

### 1.1.1 (2021-01-05)

#### Fix

* Fixed the issue of NPE when calling getNodeId() on GatewayNode

* Fixed issues where room matching continues to fail with the appropriate GameUser until reconnected from the client of the same account

---

### 1.1.0 (2020-12-17)

For more information, see the [Deployment Note](https://nhnent.dooray.com/share/posts/bWby9jGjQri2cFNR_KYbnw)

#### New

* Jdk11 support

  * In addition to the Jdk8 supported version, we have distributed the Jdk11 supported version.
  * GameAnvilversion name has been changed.

```xml
// jdk8 
<dependency> 
    <groupId>com.nhn.gameanvil</groupId> 
    <artifactId>gameanvil</artifactId> 
    <version>1.1.0-jdk8</version> 
</dependency>
```

```xml
// jdk11 
<dependency> 
    <groupId>com.nhn.gameanvil</groupId> 
    <artifactId>gameanvil</artifactId> 
    <version>1.1.0-jdk11</version> 
</dependency>
```

* Alarm

    * Added the ability to receive situations that have occurred within the server at the specified URL.
    * Use the "-Dalarm.url" VM option to specify the URL to receive the alarm.
    * [How to use GameAnvil-Guide/69 Alarm ](https://nhnent.dooray.com/share/posts/Ap6DJT9KSaGv916_tj-xAA)

#### Change

* Changed BaseObject's findAllUserLocsOfAccount return value to UserLoc

* Added feature to obtain Node information through Rest API.
    * /management/ipcNetworkNodeInfo
    * /management/managementNetworkNodeInfo
    * /management/locationLookupNodeInfo
    * /management/matchNodeInfo

#### Fix

* If you use a different SubId with the same AccountId, the login succeeds, but we have since fixed the problem of not receiving the response packet.
* Fixed an issue where NodeNum for GatewayNode, MatchNode, SupportNode and ManagementNode starts from 1.
* Fixed that Netty's send, recv buffer size optimization and modification to be applied for watermarking appropriately

---

### 1.0.7 (2021.01.05)

#### Fix
- Fixed the issue of NPE when calling getNodeId() on GatewayNode

---

### 1.0.6 (2020.12.28)

#### Fix
- `RoomMatchMakingFailure. The matched-room({}) does not exist in the game node.` If the room matching fails with the log remaining, the user will continue to fix the issue of room matching failing until reconnection school

---

### 1.0.5 (2020.11.17)

#### Fix

- Fixed problems that fail to restore packet header upon error response in RoomMatchReq

---

### 1.0.4 (2020.10.29)

#### Fix

- Fixed an issue in which NPE(Null Pointer Exception) would occur when no packet was passed to an arbitrary Session
- Problems with publicToClients (including Notice)
- Fixed an issue in which room destruction would be delayed because of the repeat Timer used by Room
- Fixed an issue in which an exception would occur when de-serializing the Class that has the protobuf structure as a variable
- java.lang.UnsupportedOperationException occurs

---

### 1.0.3 (2020.10.26)

#### Change

- Reactivated the notification API that was previously excluded from specifications

---

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

---

### 1.0.1 (2020.09.07)

#### Fix

- Fixed an issue in which Payload would not be passed to the client when onLogin() returned false
- Added the missing part of the log level check statement in front of the Debug and Trace log statements

---

### 1.0.0 (2020.08.31)

For more information, see the [Deployment Note](https://nhnent.dooray.com/share/posts/5Fvh0aszQ5u6d_ZZxRWPvA)

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

- Fixed a user transfer problem, improved usage and optimization
- GameAnvil users can now more easily use user transfers
- Completely fixed uninterrupted patch
- Fixed all the issues in which room transfer and uninterrupted patch would not work correctly
- Fixed an issue in which some Nodes would not correctly appear due to an unknown timing issue when running the server
- Fixed memory leak in which some of the Timer objects of Node persist instead of being disabled
- Fixed an issue in which newly connected user would be unintentionally disconnected while checking the client connection

#### Change

- Changed the Node name into a more intuitive one as shown in the table below

| v1.0       | -v0.10       | Required | Features                                        | Implement content | Network access     |
| ---------- | ------------- | ---- | ------------------------------------------- | ----------- | ----------------- |
| Gateway    | Session       | Required | Handles client connection and authentication               | Available        | public            |
| Game       | Space         | Required | Handles content as actual game server            | Available        | private           |
| Support    | Service       | Select | Supports independent services as necessary | Available        | private or public |
| Match      | Match         | Select | Performs matchmaking                           | Available        | private           |
| Location   | Location      | Required | Stores and manages user and room location information     | Unavailable      | private           |
| Management | Management    | Required | Collects server information and communicate with Admin/Agent        | Unavailable      | private           |
| Ipc        | Communication | Required | Processes the Inter-process communication of the Gameflex server     | Unavailable      | private           |

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

---

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

---

### 0.10.1 (2020.02.11)

#### New

- Provided the Util class that was provided to users
- Organized the tools that were previously scattered throughout the engine into a single class

#### Change

- Improved the Topic process code performance


---

### 0.10.0 (2020.02.06)

#### New

- Newly refactored the asynchronous support API that was previously problematic
- Added new asynchronous API as support Redis Cluster and official support Lettuce become available
- Added asynchronous HttpRequest and HttpResponse API
- Support custom protocol based on ByteString
- Serialize messages the way you want, not just Google protobufs

#### Fix

- Fixed all the issues in which the status of Fiber would be broken, which was the most critical problem in the previous version

#### Change

- Optimized the size of the packet and header used by the engine
- Performance improved by 2.5 times compared to version 0.9
- Embedded Redis Spec-out
- Guided users so that they would need the service provided by infrastructure

---

### 0.9.9 (2019.10.25)

#### New

- Opened additional port so that WatchDog can be linked in the unit process

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

---

### 0.9.8 (2019.09.05)

#### New

- Added addTopic() and removeTopic() API that are used to add or remove ClientTopic to Session
- Added the Packet TTL feature
- Added a logic that periodically checks and organizes Dangling Location

#### Fix

- Fixed the system so that ForceLogoutNoti would not be transferred to the client when kicking a user in Admin
- Fixed all Spot malfunctions of LocationNode

#### Change

- Changed the system so that ForceCloseNoti would be transferred before the socket is closed when onAuthenticate() callback failed


---

### 0.9.7 (2019.07.27)

#### New

- Added a logic that is used to regularly check the Rooms that abnormally remain
- As previous user matchmaking may not be canceled and remain when a user matchmaking is requested after logging in again, added a new flag that responds to the status of previous user matchmaking process to re-login response
- Added Packet Expire feature (default 30 seconds)

#### Fix

- Fixed a problem with overlapping Connection IDs (CIDs)
- Fixed a bug in which room Fiber would be closed by a disconnected user
- Fixed an issue in which the serviceId field would not be correctly set in the response message if login failed
- Fixed the part where try-catch is missing in the content callback process
- Fixed an issue in which the client would take more than 3 seconds to reconnect
- Fixed an issue in which thread simultaneity would occur when checking ghost users
- Fixed an issue in which room matchmaking would be processed in duplicate
- Fixed an issue in which disconnected users would remain in the room

#### Change

- Added a log for every instance of Session Disconnect
- Changed CreateRoom so that it would directly create room in the current node


---

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


---

### 0.9.5 (2019.06.21)

#### Fix

- Fixed an issue in which the corresponding user would not be organized in a paused node when a new user logs in or a user is being transferred while performing uninterrupted patch

#### Change

- Changed the serviceId from string to integer
- Changed the service_code of a protocol to service_id and changed the related code
- Changed the internal implementation and usage of APIs supporting RESTful


---

### 0.9.4 (2019.05.24)

#### New

- Added a feature that can be used to register the protocol used by content in the file unit
- It is to effectively process internal packets
- Added the MoveService feature

#### Fix

- Corrected errors in the comparator used in room matchmaking

#### Change

- By adding the latest login information to the Authentication response so that it could be used in the next login
- Changed setReady() so that it can be explicitly called and passed with the onReady status
- Changed LocationNode so that it can be reset at CommunicationNode's onPrepare


---

### 0.9.3 (2019.04.10)

#### Fix

- Fixed a problem that also causes Null Pointer Exception (NPE) when timeout occurs in the AsyncAwait.call() API
- Fixed an issue in which Epoll would stop when it was enabled
- Fixed an Invalid System Target Location error that would occur while logging in

#### Change

- ManagementNode can no longer be expanded by engine users
- Use ServiceNode instead
- Changed the system so that the Machine information registration, edit, or deletion request can be instantly applied in Admin


---

### 0.9.2 (2019.02.11)

#### New

- Added the scheduled notification and scheduled maintenance feature using Admin
- Added the White IP, White User and White Device features using Admin

#### Fix

- Fixed a piece of code that is used to process exceptions in a wrong way in AsyncAwait.run() API
- Changed ManagementNode so that it will be re-set before other Nodes
- Fixed an issue in which publications related to some channels would be missing when channel ID was not used
- Fixed an issue in which incorrect error code would be added to header when content returns fail on CreateRoom and JoinRoom
- Fixed memory leak that would occur when creating a packet header
- Fixed memory leak that would occur in room matchmaking

#### Change

- Added payload to the onLogout() callback
- Removed duplicate fields from some protocols as ResultCode was edited
- Improved the usability of Dynamic Module


---

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
- Applied missing timeout to AsyncAwait.run()
- Fixed an error that would occur when a node is shutdown
- Fixed problems with Null Pointer Exception (NPE) while deleting matching information even though room matchmaking is not being used

#### Change

- Relocated some interface components
- Changed the system so that it uses re-login instead of processing as duplicate logins when the user logs into different DeviceIds
- Modified the payload added by the onMatchRoom() callback of room matchmaking would be included in the flow afterward


---

### 0.9.0 (2018.06.27)

#### New

- Added the user matchmaking feature
- Added the user and room list per channel feature
- Added Embedded Redis
- Added the Reconnect feature
- Added the (old) Test Agent

#### Fix

- Fixed a duplicate login in the Account unit issue

#### Change

- Changed OracleJDK to AdpotOpenJDK
- Changed the ZMQ communication on a per-node(thread) basis to a per-process basis

---

### 0.8.6 (2018.06.28)

#### New

- Added the Send from Session to Management feature
- Added the Request from Session to Management feature
- Added User MatchMaker
- Added the Channel User List feature
- Added the Custom Serializer feature
- Custom Serializer can be plugged into RoomFinder

#### Fix

- Added an empty string exception process for the publication where a channel is used as a topic in the channel user manager
- Added an empty string exception process for the publication where a channel is used as a topic
- Fixed an issue that would be occurred by DefaultRoomFinder when calling delNamedRoomInfo() and delRoomInfo() in an environment that does not use AutoJoinRoom
- Fixed the NullPointerException that would occur due to a missing null check in RoomFinder
- Fixed an issue in which a timed out request would not be removed in user matchmaking
- Added an exception that is used to be processed if password is set in redis
- Fixed a setRestIp() error of cfgManagement
- Fixed an issue where HTTP handling would not be done in the Service node.
- Branch: 0.8.6.2_fix_service_node_rest_handling
- Tag: tardis-0.8.6.2.1
- Fixed an issue related to the set gameId while processing AutoJoinRoom
- Edited MatchMaker and strengthened examples
- Added the previously missing ISession::onDisconnect()
- Fixed a Node information exchange error among Communication Nodes
- Fixed a FindRoomList performance issue
- Fixed a Shutdown bug
- Fixed an issue where a BaseNode would not receive publishToNode messages
- Fixed a FindRoomList performance issue
- Fixed a Shutdown bug
- Fixed an issue where a BaseNode would not receive publishToNode messages
- Fixed a NamedAutoJoinRoom bug
- Changed UpdateRoomInfo/DelRoomInfo so that they can be instantly applied to the corresponding Node
- Fixed an error where a blocking call would be displayed in Fiber

#### Change

- Changed FindRoomList so that it serializes/de-serializes the container for the entire room list
- Changed the existing LocationException, TardisException and others to RuntimeException, organized Exceptions
- Renamed forceLeaveRoom to kickOutRoom
- Renamed forceLogout to kickOut
- Changed the tardisConfig configuration

---

### 0.8.5 (2017.12.18)

#### New

- Added a feature to be used to authenticate and manage HTTP sessions with the JWT authentication token
- Added a feature to be used to allow multiple services to log in using the same ID
- Added a feature to be used to allow a single GameNode to use multiple RoomTypes
- Added a feature to be used to view all the information of the Server on the web (NodeInfoPage)

#### Fix

- Fixed a bug in which timers would not be removed
- Fixed a bug in which onPostLeaveRoom would be called twice

#### Change

- Improved ghost user handling
- Added ghost user logging
- Improved packet compress logic
- Improved RestObject, added a function to be used with it
- Changed the error handling of a packet, it now places error code to protocol header and processes it

---

### 0.8.4 (2018.01.10)

#### New

- Applied Bootstrap
- Implemented like NGB Bootstrap
- Simplified code by specifying only the class instance of each class in the implementing part
- Added the Multi Dispatch feature that is used to send and receive multiple messages
- Added the ServiceNodeSender (previously CustomServiceSender) publishToLobby method

#### Change

- Re-named AsyncAwait and RAsyncAwait
- Moved the Bootstrap RoomFinder settings to space
- Renamed Cache and Custom Exception (Location, Service)
- Changed the Payload class so that it retrieves the first packet using the getFirst method
- Removed the requestToLobby method from ServiceNodeSender (the requestToNode method works the same)
- Changed the makeDecompress method so that it does not use async when it is called by the Packet class
- Overloaded the reply method of UserSender (Packet list handling)
- Re-named some classes
- AsyncCall -> AsyncWaitingCall
- AsyncRun -> AsyncWaitingRun
- ReverseAsyncCall -> RAsyncWaitingCall
- ReverseAsyncRun -> RAsyncWaitingRun

#### Fix

- Fixed a Session IP error

---

### 0.8.3 (2017.10.26)

#### New

- Added the REST handling feature
- Implemented the ACL feature based on Path and IP for the REST handling with permissionGroups in TardisConfig.json
- Channel List (channel name list), Channel Information (detailed information of a specific channel)
- Added the requestToCustom and sendToCustom method to Space LobbySender
- Added the "State" item to the Management information of Space node
- Added void onShutdown() throws SuspendExecution to the Ilobby, ISessionLobby and IService interface
- Added the onShutdown() interface so that contents can disable resource when a node is shutdown
- Added the publishToNode function to ServiceNodeSender

#### Fix

- Changed and added the JMX Management API of Management (SessionGateway, CustomGateway)
- Added Channel List, Channel Information (detailed information of specific channel) features
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
- Re-named classes, packages, functions and cfg contents
- Changed the class Payload data type to Packet to allow the Request and Response protocols to handle multiple payloads, removed class OutPayload
- Changed the parameter type of the user Transfer-In interface to byte array

---

### 0.8.2 (2017.09.21)

#### New

- Added the RestObject feature to CustomService
- Added RestHandler to CustomService

#### Fix

- Fixed a bug in which user information would not be deleted from Cache
- Added exception handling for NPE (Null Pointer Exception) that occurred inside the engine
- Fixed the problem of not properly applying the Connection ID (CID) to packet transmission

#### Change

- Changed the Custom module naming
- Ex) CustomLobby -> CustomService
- Changed some part of the Custom module interface
- Added the serviceId parameter to the sendToUser() and requestToUser() API of CustomServiceSender