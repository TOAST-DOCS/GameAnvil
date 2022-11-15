## Game > GameAnvil > 테스트 개발 가이드 > 기능 테스트 개발 가이드

## Tester

GameHammer를 사용하기 위한 기본 모듈입니다. 기본 설정과 Connection 객체의 관리를 담당합니다. Tester 객체를 생성하려면 먼저 Builder를 만들고 테스트에 필요한 옵션을 설정한 후, `build()`를 호출하면 됩니다.

```
Tester tester = Tester.newBuilder()
                    .addProtoBufClass(0, RPSGame.getDescriptor())
                    .build();
```

옵션 설정을 외부로부터 불러올 수도 있습니다. vmoption `config.file=PATH`로 경로를 지정하거나, resource 폴더 아래에 GameHammerConfg.json 파일을 만들어 놓으면 TesterConfigLoader에서 자동으로 읽어오게 됩니다. 아래와 같은 방법으로 읽어온 옵션 설정이 적용된 Tester를 생성할 수 있습니다.

```
Tester tester = Tester.newBuilderWithConfig()
                    .addProtoBufClass(0, RPSGame.getDescriptor())
                    .build();
```

## Connection

게임 서버와의 연결, 인증 등의 기능을 처리하고, 유저의 관리를 담당합니다. 아래와 같이 Tester를 통해 생성합니다.

```
Connection connection = tester.createConnection(uuid);
```

생성된 Connection 객체는 Tester에서 관리되며 uuid로 구분됩니다. 이미 생성된 객체의 uuid가 입력되면 해당 객체를 반환합니다.

Connection은 다음과 같은 기능을 제공합니다. 

### Connect

GameAnvil 서버에 연결합니다.

```
Future<ResultConnect> future = connection.connect(new RemoteInfo("127.0.0.1", 11200));
ResultConnect resultConnect = futre.get(); // blocked
if(resultConnect.isSuccess()){
    // connect success
}
```

`connect()` 에서 반환 받은 Future의 `get()`을 호출하면 연결이 성공하거나 실패 할 때까지 대기후 ResultConnect 객체를 반환합니다. 반환 받은 ResultConnect 객체를 통해 결과를 알 수 있습니다. `connect()`의 두 번째 인자로 콜백을 넘겨 결과를 받을 수도 있습니다. `connect()` 외 다른 API들도 Future를 리턴해 결과를 기다리거나 콜백을 넘겨 결과를 전달 받을 수 있습니다. 

### Authentication

GameAnvil 서버에 인증을 요청합니다. 인증에 성공해야 GameAnvil 커넥터의 다른 기능을 사용할 수 있습니다.

```
Future<ResultAuthentication> future = connection.authentication(accountId, password, deviceId, payload);
ResultAuthentication resultAuthentication = future.get(); // blocked
if(resultAuthentication.isSuccess){
    // authentication success
}
```

### GetChannelList

지정한 서비스에서 사용 가능한 채널 목록을 요청합니다.

```
Future<ResultChannelList> future = connection.getChannelList(serviceName);
ResultChannelList resultChannelList = future.get(); // blocked
if(resultChannelList.isSuccess){
    // authentication success
}
```

### GetChannelInfo

지정한 체널의 정보를 요청합니다.

```
Future<ResultChannelList> future = connection.getChannelList(serviceName, String channelId);
ResultChannelList resultChannelList = future.get(); // blocked
if(resultChannelList.isSuccess){
    // authentication success
}
```

### Request

서버로 메시지를 보내고, 응답을 기다립니다.

```
Future<PacketResult> future = connection.request(message);
PacketResult packetResult = future.get(); // blocked
if(packetResult.isSuccess()){
    // request success
}
```

### Send

서버로 메시지를 보냅니다.

```
connection.send(message);
```

### Close

접속을 끊습니다. 접속 종료 시 생성했던 유저들을 모두 로그아웃하려면 인자로 true를 입력합니다.

```
connection.close(true);
```

### WaitForAdminKickoutNoti

어드민 강제 종료 알림을 받을 때까지 기다립니다. 어드민에서 강제 종료할 경우 전달됩니다.

```
Future<ResultAdminKickoutNoti> future = connection.waitForAdminKickoutNoti()
ResultAdminKickoutNoti resultAdminKickoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultAdminKickoutNoti 
```

### WaitForForceCloseNoti

강제 종료 알림을 받을 때까지 기다립니다. 서버에서 `BaseUser.closeConnection()`을 호출하거나, Authentication이 실패하거나, 같은 계정으로 중복 로그인을 하거나, UserTrasnfer 도중 예외가 발생하는 등의 경우 전달됩니다.

```
Future<ResultForceCloseNoti> future = connection.waitForForceCloseNoti();
ResultForceCloseNoti resultForceCloseNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceCloseNoti 
```

### WaitForDisconnect

네트워크 연결 끊김 알림을 받을 때까지 기다립니다. 서버에서 `BaseConnection.close()`를 호출하거나, 소켓 오류가 발생하거나 `Connection.close()`, `Tester.Close()`를 호출할 경우 전달됩니다.

```
Future<ResultDisconnect> future = connection.waitForDisconnect();
ResultDisconnect resultDisconnect = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultDisconnect 
```

## User

로그인을 비롯하여 방 생성, 입장, 매칭 등 게임에 필요한 주요 기능을 담당합니다. 다음과 같이 User를 생성할 수 있습니다.

```
User user = connection.getUserAgent(serviceName, subId);
if(user == null){
    user = connection.createUser(serviceName, subId);
}
```

`Connection.getUserAgent()`로 ServiceName과 SubId로 매칭되는 User가 있는지 확인하고, 없을 경우 `Connection.createUser()`로 새로운 User를 생성합니다. 

User는 다음과 같은 기능을 제공합니다. 

### Login

지정한 유저 타입으로 지정한 채널에 로그인합니다. 유저 타입과 채널은 서버에서 설정한 문자열을 사용합니다.

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

### Logout

로그인된 채널에서 로그아웃합니다.

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

### WaitForForceLogoutNoti

강제 로그아웃 알림을 받을 때까지 기다립니다. 서버에서 `BaseUser.kickout()`을 호출할 경우 전달됩니다.

```
Future<ResultForceLogoutNoti> future = connection.waitForForceLogoutNoti();
ResultForceLogoutNoti resultForceLogoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceLogoutNoti 
```

### CreateRoom

지정한 룸 타입으로 지정한 이름의 방을 생성하고 해당 방에 입장합니다. 룸 타입은 서버에서 설정한 문자열을 사용합니다.

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

### JoinRoom

지정한 ID의 방에 입장합니다. 지정한 ID의 방이 없을 경우 실패합니다.

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

### NamedRoom

지정한 이름의 방에 입장합니다. 지정한 이름의 방이 없을 경우 방을 생성하고 해당 방에 입장합니다. 파티 매칭을 위한 방인 경우에는 useParty를 true로 입력합니다.

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

### LeaveRoom

현재 방에서 퇴장합니다.

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

### WaitForForceLeaveRoomNoti

방 강제 퇴장 알림을 받을 때까지 기다립니다. 서버에서 BaseUser.kickoutRoom()을 호출할 경우 전달됩니다.

```
Future<ResultForceLeaveRoomNoti> future = connection.waitForForceLeaveRoomNoti();
ResultForceLeaveRoomNoti resultForceLeaveRoomNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceLeaveRoomNoti 
```

### MatchUserStart

유저 매치메이킹을 요청합니다. 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패할 수 있습니다.  WaitForMatchUserDoneNoti를 통해 매치 성공 알림, WaitForMatchUserTimeoutNoti를 통해 매치 타임아웃 알림을 받을 수 있습니다.

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

### MatchUserCancel

유저 매치메이킹 요청을 취소합니다. 매치 요청 중이 아닌 경우, 이미 매칭이 성공했거나 타임아웃이 발생했을 경우 실패할 수 있습니다.

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

### WaitForMatchUserDoneNoti

유저 매치메이킹 또는 파티 매치메이킹 완료 알림을 받을 때까지 기다립니다.  

```
Future<ResultMatchUserDone> future = connection.waitForMatchUserDoneNoti();
ResultMatchUserDone resultMatchUserDone = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchUserDone 
```

### WaitForMatchUserTimeoutNoti

유저 매치메이킹 또는 파티 매치메이킹에 대한 타임아웃 알림을 받을 때까지 기다립니다.  

```
Future<ResultMatchUserTimeout> future = connection.waitForMatchUserTimeoutNoti();
ResultMatchUserTimeout resultMatchUserTimeout = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchUserTimeout 
```

### MatchPartyStart

파티 매치메이킹을 요청합니다. 파티 매치메이킹을 위한 방에 입장한 상태에서 요청할 수 있습니다. WaitForMatchUserDoneNoti를 통해 매치 성공 알림, WaitForMatchUserTimeoutNoti를 통해 매치 타임아웃 알림을 받을 수 있습니다.

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

### WaitForMatchPartyStartNoti

파티 매치메이킹 시작 알림을 받을 때까지 기다립니다. 파티 매치메이킹을 위한 방에서 다른 사람이 파티 매치메이킹을 시작했을 경우 전달됩니다.

```
Future<ResultMatchPartyStart> future = connection.waitForMatchPartyStartNoti();
ResultMatchPartyStart resultMatchPartyStart = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchPartyStart 
```

### MatchPartyCancel

파티 매치메이킹 요청을 취소합니다. 파티 매치메이킹 중이 아닌 경우, 이미 파티 매치메이킹에 성공했거나 타임아웃이 발생하면 실패할 수 있습니다.

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

### WaitForMatchPartyCancelNoti

파티 매치메이킹 취소 알림을 받을 때까지 기다립니다. 파티 매치메이킹 중 다른 사람이 파티 매치메이킹을 취소한 경우 전달됩니다.

```
Future<ResultMatchPartyCancel> future = connection.waitForMatchPartyCancelNoti();
ResultMatchPartyCancel resultMatchPartyCancel = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchPartyCancel 
```

### MatchRoom

방 매치메이킹을 요청합니다. 방이 없을 경우 임의의 방을 생성하고 해당 방에 입장할 수도 있습니다.

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

### MoveChannel

지정한 채널로 이동합니다.

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

### WaitForMoveChannelNoti

체널 이동 알림을 받을 때까지 기다립니다. 방 입장, 매치매이킹 등의 이유로 체널을 이동하게 되면 전달됩니다.

```
Future<ResultMoveChannelNoti> future = connection.waitForMoveChannelNoti();
ResultMoveChannelNoti resultMoveChannelNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMoveChannelNoti 
```

### Request

서버로 메시지를 보내고, 응답을 기다립니다.

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

### Send

서버로 메시지를 보냅니다.

```
user.send(message);
```

### WaitForNotice

공지 알림을 받을 때까지 기다립니다. 어드민에서 공지를 보내거나, REST API를 이용해 공지를 보낼 경우 전달됩니다.

```
Future<ResultNotice> future = connection.waitForMoveChannelNoti();
ResultNotice resultNotice = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultNotice 
```

## 테스트 코드 작성

### 기본 설정

JUnit을 이용해 테스트 코드를 작성할 때 다음과 같이 BeforeClass, AfterClass, After 코드를 작성합니다.

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

이렇게 작성하면 선행 테스트에서 사용한 유저가 남아 다음 테스트에 영향을 주는 것을 막을 수 있습니다. 

### Request/Response 테스트

GameHammer를 이용한 테스트 코드는 크게 2가지 유형으로 나눌 수 있습니다. 첫 번째는 Request/Response 형태의 테스트 코드입니다. 클라이언트에서 Request를 보낼 경우 서버에서는 반드시 Response를 보내야 합니다. 응답을 보내지 않으면 timeout이 발생합니다. GameAnvil 커넥터가 제공하는 API 대부분이 이런 Request/Response 방식이며 GameHammer에서도 이런 Request/Response 방식의 테스트를 지원합니다.

예를 들면 Connection의 connect에 대한 테스트 코드는 다음과 같이 작성할 수 있습니다.

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

`connection.connect()`를 호출해 서버에 접속합니다. 이때 반환받은 `Future`의 `get()`을 호출하면 `connection.connect()`가 완료될 때까지 대기하고,  완료되면 그 결과인 `ResultConnect`를 리턴합니다.  여기서 리턴받은 `ResultConnect`이용해 성공 여부를 판단할 수 있습니다. 

또 다른 예로 게임에서 사용하는 메시지에 대한 Request/Response 방식 테스트 코드는 다음과 같이 작성할 수 있습니다. 

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

먼저 메시지를 만들고, 이것을 `connection.request()`의 인자로 넣어 서버로 전송합니다. 이때 리턴받은 `Future`의 `get()`를 호출하면 서버에서 응답을 하거나 타임아웃이 발생할 때까지 대기하고, 응답을 받거나 타임아웃이 발생하면 `PacketResult`를 리턴합니다. 여기서 리턴받은 `PacketResult`를 이용해 성공 여부를 판단할 수 있습니다. 성공하면 `PacketResult.getStream()`을 이용해 메시지를 파싱하여 내용을 확인할 수 있습니다. 

Connnection, User의 API 중 Future를 리턴하는 API들은 모두 이런 방식을 사용해 테스트할 수 있습니다.

### Send/Receive

클라이언트는 Send를 보내고 응답을 기다리지 않습니다. 그리고 서버에서도 클라이언트의 동작과 상관없이 Send를 보낼 수 있습니다. 테스트는 다음과 같이 작성할 수 있습니다.

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