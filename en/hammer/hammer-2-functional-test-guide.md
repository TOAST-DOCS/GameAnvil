<!-- pre-align:aligned sig=ef059736c23a -->

<a id="game-gameanvil-test-development-guide-how-to-implement-functional-test"></a>
## Game > GameAnvil > Test Development Guide > How to implement Functional Test { #game-gameanvil-test-development-guide-how-to-implement-functional-test }

<a id="tester"></a>
### Tester { #tester }

It is the default module for using GameHammer. It is responsible for the default settings and the Connection object. To create a Tester object, create a Builder first, configure the necessary options for a test and call `build()`.

```
Tester tester = Tester.newBuilder() 
                    .addProtoBufClass(RPSGame.getDescriptor()) 
                    .build();
```

Option settings can be imported from outside. Specify a path using vmoption `config.file=PATH` or create the GameHammerConfg.json file under the resource folder so that the settings can be automatically read from TesterConfigLoader. The Tester to which the option settings read using the aforementioned method can be created.

```
Tester tester = Tester.newBuilderWithConfig() 
                    .addProtoBufClass(RPSGame.getDescriptor()) 
                    .build();
```

<a id="tester-create-object"></a>
#### Create Object

<!-- TODO: translate body -->

<a id="tester-add-option"></a>
#### Add Option

<!-- TODO: translate body -->

<a id="connection"></a>
### Connection { #connection }

It handles the connection to the game server and authentication and takes care of users. Create Tester using as below:

```
Connection connection = tester.createConnection(uuid);
```

The created Connection object is managed by Tester and it is separated by UUID. If the UUID of an existing object is entered, it returns the corresponding object.

Connection provides the following features: 

<a id="connection-create-object"></a>
#### Create Object

<!-- TODO: translate body -->

<a id="connection-connect"></a>
#### Connect

Connects to the GameAnvil server.

```
Future<ResultConnect> future = connection.connect(new RemoteInfo("127.0.0.1", 18200)); 
ResultConnect resultConnect = futre.get(); // blocked 
if(resultConnect.isSuccess()){ 
    // connect success 
}
```

When you call `get()`on the Future returned by `connect()`, it waits for the connection to succeed or fail and returns a ResultConnect object. You can get the result from the returned ResultConnect object. You can also pass a callback as the second argument to `connect()`to get the result. Other APIs besides `connect()` can also return a Future and wait for the result, or pass a callback to get the result. 

<a id="connection-authentication"></a>
#### Authentication

Request authentication to the GameAnvil server. The other features of the GameAnvil connector can be used only when the authentication is successful.

```
Future<ResultAuthentication> future = connection.authentication(accountId, password, deviceId, payload); 
ResultAuthentication resultAuthentication = future.get(); // blocked 
if(resultAuthentication.isSuccess){ 
    // authentication success 
}
```

<a id="connection-getchannellist"></a>
#### GetChannelList

Request the list of channels available to the specified service.

```
Future<ResultChannelList> future = connection.getChannelList(serviceName); 
ResultChannelList resultChannelList = future.get(); // blocked 
if(resultChannelList.isSuccess){ 
    // authentication success 
}
```

<a id="connection-getchannelinfo"></a>
#### GetChannelInfo

Request the information of the specified channel.

```
Future<ResultChannelList> future = connection.getChannelList(serviceName, String channelId); 
ResultChannelList resultChannelList = future.get(); // blocked 
if(resultChannelList.isSuccess){ 
    // authentication success 
}
```

<a id="connection-request"></a>
#### Request

Send message to server and wait for response.

```
Future<PacketResult> future = connection.request(message); 
PacketResult packetResult = future.get(); // blocked 
if(packetResult.isSuccess()){ 
    // request success 
}
```

<a id="connection-send"></a>
#### Send

Send message to server.

```
connection.send(message);
```

<a id="connection-close"></a>
#### Close

Severs the connection. To have the users created when disconnected log out, enter true as a factor.

```
connection.close(true);
```

<a id="connection-waitforadminkickoutnoti"></a>
#### WaitForAdminKickoutNoti

Wait until the forcibly closed by admin notification is received. It is passed when the admin forcibly disconnects.

```
Future<ResultAdminKickoutNoti> future = connection.waitForAdminKickoutNoti() 
ResultAdminKickoutNoti resultAdminKickoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultAdminKickoutNoti 
```

<a id="connection-waitfordisconnect"></a>
#### WaitForDisconnect

Wait until a network disconnected notification is received. It is passed when the server calls `BaseConnection.close()`, a socket error occurs or calls either `Connection.close()` or `Tester.Close()`.

```
Future<ResultDisconnect> future = connection.waitForDisconnect(); 
ResultDisconnect resultDisconnect = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultDisconnect 
```

<a id="user"></a>
### User { #user }

It is responsible for the major features required for the game such as login, room creation, join, and matching. User can be created as below:

```
User user = connection.getUserAgent(serviceName, subId); 
if(user == null){ 
    user = connection.createUser(serviceName, subId); 
}
```

Check if User that is matched with ServiceName and SubId using `Connection.getUserAgent()`. If there is none, create a new User with `Connection.createUser()`. 

User provides the following features: 

<a id="user-login"></a>
#### Login

Log into the specified channel using the specified user type. User type and channel use the character strings specified by server.

```
Future<ResultLogin> future = user.login(resultLogin -> { 
    if(resultLogin.isSuccess()){ 
        // login success 
    } 
}, userType, channelId, payload); 
 
ResultLogin resultLogin = future.get(); // blocked 
if(resultLogin.isSuccess()){ 
    // login success 
}
```

<a id="user-logout"></a>
#### Logout

Log out from the logged in channel.

```
Future<ResultLogout> future = user.logout(resultLogout -> { 
    if(resultLogout.isSuccess()){ 
        // logout success 
    } 
}, userType, channelId, payload); 
 
ResultLogout resultLogout = future.get(); // blocked 
if(resultLogout.isSuccess()){ 
    // logout success 
}
```

<a id="user-waitforforcelogoutnoti"></a>
#### WaitForForceLogoutNoti

Wait until a forcible logout notification is sent. It is passed when the server calls `BaseUser.kickout()`.

```
Future<ResultForceLogoutNoti> future = connection.waitForForceLogoutNoti(); 
ResultForceLogoutNoti resultForceLogoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultForceLogoutNoti 
```

<a id="user-createroom"></a>
#### CreateRoom

Create a room, name it with the specified room type and join the room. Room type uses the character strings configured from server.

```
Future<ResultCreateRoom> future = user.createRoom(resultCreateRoom -> { 
    if(resultCreateRoom.isSuccess()){ 
        // createRoom success 
    } 
}, roomType, roomName, payload); 
 
ResultCreateRoom resultCreateRoom = future.get(); // blocked 
if(resultCreateRoom.isSuccess()){ 
    // createRoom success 
}
```

<a id="user-joinroom"></a>
#### JoinRoom

Join the room that has the specified ID. It fails when there is no room with the specified ID.

```
Future<ResultJoinRoom> future = user.joinRoom(resultJoinRoom -> { 
    if(resultJoinRoom.isSuccess()){ 
        // joinRoom success 
    } 
}, roomType, roomId, payload); 
 
ResultJoinRoom resultJoinRoom = future.get(); // blocked 
if(resultJoinRoom.isSuccess()){ 
    // joinRoom success 
}
```

<a id="user-namedroom"></a>
#### NamedRoom

Join the room with the specified name. If there is no room with the specified name, create such a room and join it. If the room is for party matching, enter true as useParty.

```
Future<ResultNamedRoom> future = user.namedRoom(resultNamedRoom -> { 
    if(resultNamedRoom.isSuccess()){ 
        // namedRoom success 
    } 
}, roomType, roomName, useParty, payload); 
 
ResultNamedRoom resultNamedRoom = future.get(); // blocked 
if(resultNamedRoom.isSuccess()){ 
    // namedRoom success 
}
```

<a id="user-leaveroom"></a>
#### LeaveRoom

Leave the current room.

```
Future<ResultLeaveRoom> future = user.leaveRoom(resultLeaveRoom -> { 
    if(resultLeaveRoom.isSuccess()){ 
        // leaveRoom success 
    } 
}, payload); 
 
ResultLeaveRoom resultLeaveRoom = future.get(); // blocked 
if(resultLeaveRoom.isSuccess()){ 
    // leaveRoom success 
}
```

<a id="user-waitforforceleaveroomnoti"></a>
#### WaitForForceLeaveRoomNoti

Wait until a forcible kickout notification is sent. It is passed when the server calls BaseUser.kickoutRoom.

```
Future<ResultForceLeaveRoomNoti> future = connection.waitForForceLeaveRoomNoti(); 
ResultForceLeaveRoomNoti resultForceLeaveRoomNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultForceLeaveRoomNoti 
```

<a id="user-matchuserstart"></a>
#### MatchUserStart

Request user matchmaking. If the user already joined a room, the request may fail depending on server requirements. Matching success notification can be received by WaitForMatchUserDoneNoti and the match timeout notification can be received by WaitForMatchUserTimeoutNoti.

```
Future<ResultMatchUserStart> future = user.matchUserStart(resultMatchUserStart -> { 
    if(resultMatchUserStart.isSuccess()){ 
        // matchUserStart success 
    } 
}, roomType, payload); 
 
ResultMatchUserStart resultMatchUserStart = future.get(); // blocked 
if(resultMatchUserStart.isSuccess()){ 
    // matchUserStart success 
}
```

<a id="user-matchusercancel"></a>
#### MatchUserCancel

Cancel the user matchmaking request. If match is not requested, matching was successful, or a timeout occurred, it may fail.

```
Future<ResultMatchUserCancel> future = user.matchUserCancel(resultMatchUserCancel -> { 
    if(resultMatchUserCancel.isSuccess()){ 
        // matchUserCancel success 
    } 
}, roomType); 
 
ResultMatchUserCancel resultMatchUserCancel = future.get(); // blocked 
if(resultMatchUserCancel.isSuccess()){ 
    // matchUserCancel success 
}
```

<a id="user-waitformatchuserdonenoti"></a>
#### WaitForMatchUserDoneNoti

Wait until the user matchmaking or party matchmaking request complete notification is sent.  

```
Future<ResultMatchUserDone> future = connection.waitForMatchUserDoneNoti(); 
ResultMatchUserDone resultMatchUserDone = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultMatchUserDone 
```

<a id="user-waitformatchusertimeoutnoti"></a>
#### WaitForMatchUserTimeoutNoti

Wait until the user matchmaking or party matchmaking request timed out notification is sent.  

```
Future<ResultMatchUserTimeout> future = connection.waitForMatchUserTimeoutNoti(); 
ResultMatchUserTimeout resultMatchUserTimeout = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultMatchUserTimeout 
```

<a id="user-matchpartystart"></a>
#### MatchPartyStart

Request party matchmaking. It can be requested when the user joined a room for party matchmaking. Matching success notification can be received by WaitForMatchUserDoneNoti and the match timeout notification can be received by WaitForMatchUserTimeoutNoti.

```
Future<ResultMatchPartyStart> future = user.matchPartyStart(resultMatchPartyStart -> { 
    if(resultMatchPartyStart.isSuccess()){ 
        // matchPartyStart success 
    } 
}, roomType, payload); 
 
ResultMatchPartyStart resultMatchPartyStart = future.get(); // blocked 
if(resultMatchPartyStart.isSuccess()){ 
    // matchPartyStart success 
}
```

<a id="user-waitformatchpartystartnoti"></a>
#### WaitForMatchPartyStartNoti

Wait until the user receives a party matchmaking start notification. It is passed when someone else started party matchmaking in a room for party matchmaking.

```
Future<ResultMatchPartyStart> future = connection.waitForMatchPartyStartNoti(); 
ResultMatchPartyStart resultMatchPartyStart = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultMatchPartyStart 
```

<a id="user-matchpartycancel"></a>
#### MatchPartyCancel

Cancel the party matchmaking request. If party matchmaking is not in progress, party matchmaking was successful, or a timeout occurred, it may fail.

```
Future<ResultMatchPartyCancel> future = user.matchPartyCancel(resultMatchPartyCancel -> { 
    if(resultMatchPartyCancel.isSuccess()){ 
        // matchPartyCancel success 
    } 
}, roomType); 
 
ResultMatchPartyCancel resultMatchPartyCancel = future.get(); // blocked 
if(resultMatchPartyCancel.isSuccess()){ 
    // matchPartyCancel success 
}
```

<a id="user-waitformatchpartycancelnoti"></a>
#### WaitForMatchPartyCancelNoti

Wait until the party matchmaking cancel notification is sent. It is passed when someone else cancels party matchmaking while party matchmaking is in progress.

```
Future<ResultMatchPartyCancel> future = connection.waitForMatchPartyCancelNoti(); 
ResultMatchPartyCancel resultMatchPartyCancel = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultMatchPartyCancel 
```

<a id="user-matchroom"></a>
#### MatchRoom

Request the room matchmaking. If there is no room, the user may create an arbitrary room and join that room.

```
Future<ResultMatchRoom> future = user.matchRoom(resultMatchRoom -> { 
    if(resultMatchRoom.isSuccess()){ 
        // matchRoom success 
    } 
}, roomType); 
 
ResultMatchRoom resultMatchRoom = future.get(); // blocked 
if(resultMatchRoom.isSuccess()){ 
    // matchRoom success 
}
```

<a id="user-movechannel"></a>
#### MoveChannel

Move to the specified channel.

```
Future<ResultMoveChannel> future = user.moveChannel(resultMoveChannel -> { 
    if(resultMoveChannel.isSuccess()){ 
        // moveChannel success 
    } 
}, roomType); 
 
ResultMoveChannel resultMoveChannel = future.get(); // blocked 
if(resultMoveChannel.isSuccess()){ 
    // moveChannel success 
}
```

<a id="user-waitformovechannelnoti"></a>
#### WaitForMoveChannelNoti

Wait until you receive a channel move notification. If you move the channel for reasons such as room entry, matchmaking, etc., it will be delivered.

```
Future<ResultMoveChannelNoti> future = connection.waitForMoveChannelNoti(); 
ResultMoveChannelNoti resultMoveChannelNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultMoveChannelNoti 
```

<a id="user-request"></a>
#### Request

Send message to server and wait for response.

```
Future<PacketResult> future = user.request(packetResult -> { 
    if(packetResult.isSuccess()){ 
        // request success 
    } 
}, message); 
 
PacketResult packetResult = future.get(); // blocked 
if(packetResult.isSuccess()){ 
    // request success 
}
```

<a id="user-send"></a>
#### Send

Send message to server.

```
user.send(message);
```

<a id="user-waitfornotice"></a>
#### WaitForNotice

Wait until a notification is sent. It is passed when an admin sent a notification or a notification is sent by REST API.

```
Future<ResultNotice> future = connection.waitForMoveChannelNoti(); 
ResultNotice resultNotice = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked 
// resultNotice 
```

<a id="write-test-code"></a>
### Write test code { #write-test-code }

<a id="write-test-code-basic-settings"></a>
#### Basic Settings

Write the BeforeClass, AfterClass, and After codes as follows when writing test code using JUnit.

```
public class TestWithGameHammer { 
    static Tester tester; 
 
    @BeforeClass 
    public static void beforeClass() { 
        tester = Tester.newBuilder() 
                    .addProtoBufClass(0, RPSGame.getDescriptor()) 
                    .build(); 
    } 
 
    @Before 
    public void before() { 
    } 
 
    @After 
    public void after() { 
        tester.closeAllConnections(); 
    } 
 
    @AfterClass 
    public static void afterClass() { 
        tester.close(); 
        tester = null; 
    } 
 
    @Test 
    public void connectTest() { 
        ResultConnect resultConnect = connect(connection); 
        assertEquals(ResultCodeConnect.CONNECT_SUCCESS, resultConnect.getResultCode()); 
    } 
 
    @Test 
    public void someTest() { 
        // some test code 
    }
```

If it is written in this way, you can stop the users participated in a preceding test remain from affecting the following test. 

<a id="write-test-code-requestresponse-test"></a>
#### Request/Response test

Test codes using GameHammer can be categorized by 2 types. The former is a group of test codes in Request/Response type. If Request is sent by the client, Response must be sent. If response is not sent, a timeout occurs. Most of the APIs provided by the GameAnvil connector use the Request/Response method and GameHammer supports tests in this Request/Response type.

For example, the test code for the connect of Connection can be written as below:

```
@Test 
public void Connect() { 
    Connection connection = tester.createConnection(uuid); 
 
    Future<ResultConnect> future = connection.connect(new RemoteInfo("127.0.0.1", 11200)); 
    ResultConnect resultConnect = futre.get(); // blocked 
    Assert.assertTrue("Connection fail", resultConnect.isSuccess()); 
 
    // more test code 
}
```

Call `connection.connect()` to connect to the server. If you call `get()` of `Future` returned at this time, wait until `connection.connect()` is complete, and return the resulting `ResultConnect`. You can use the returned `ResultConnect` to determine whether or not it is a success or failure. 

For another example, the test code for messages using the Request/Response method can be written as below: 

```
@Test 
public void RequestTest() { 
    Connection connection = tester.createConnection(uuid); 
    // assume aleady connected. 
 
    Messages.Request request = Messages.Request.newBuilder().build(); 
    Future<PacketResult> future = connection.request(request); 
    PacketResult packetResult = future.get(); // blocked 
    Assert.assertTrue("Response fail", packetResult.isSuccess()); 
    Messages.Response response = Messages.Response.parseFrom(packetResult.getStream()); 
 
    // more test code 
}
```

First, create a message and send it to the server as a factor in `connection.request()`. When `get()` of `Future` is called, the server responds or waits for a timeout to occur and returns `PacketResult` when a timeout occurs. The `PacketResult` returned here can be used to determine success or failure. If successful, you can parse the message using `PacketResult.getStream()`. 

The APIs of Connection and User that return Future can be tested by using this method.

<a id="write-test-code-sendreceive"></a>
#### Send/Receive

The client sends Send, but it does not wait for response. The server may send Send regardless of the client's actions as well. The test can be written as below:

```
@Test 
public void RequestTest() { 
    Connection connection = tester.createConnection(uuid); 
    // assume aleady connected. 
 
    Messages.SendToServer sendToServer = Messages.SendToServer.newBuilder().build(); 
    connection.send(sendToServer); 
 
    Future<PacketResult> future = connection.waitFor(Messages.SendToClient.getDescription()); 
    PacketResult packetResult = future.get(3000, TimeUnit.MILLISECONDS); // blocked 
    Assert.assertTrue("receive fail", packetResult.isSuccess()); 
    Messages.SendToClient sendToClient = Messages.SendToClient.parseFrom(packetResult.getStream()); 
 
    // more test code 
}
```