## Game > GameAnvil > テスト開発ガイド > 機能テスト開発ガイド

## Tester

GameHammerを使うための基本モジュールです。基本設定とConnectionオブジェクトの管理を担当します。Testerオブジェクトを生成するには、まず、Builderを作成し、テストに必要なオプションを設定した後、`build()`を呼び出します。

```
Tester tester = Tester.newBuilder()
                    .addProtoBufClass(RPSGame.getDescriptor())
                    .build();
```

オプション設定を外部から読み込むこともできます。 vmoption `config.file=PATH` でパスを指定するか、resourceフォルダの下にGameHammerConfg.jsonファイルを作成しておくと、TesterConfigLoaderで自動的に読み込みます。下記のような方法で、読み込んだオプション設定が適用されたTesterを作成できます。

```
Tester tester = Tester.newBuilderWithConfig()
                    .addProtoBufClass(RPSGame.getDescriptor())
                    .build();
```

## Connection

ゲームサーバーとの接続、認証などの機能を処理して、ユーザーの管理を担当します。下記のようにTesterを使って作成します。

```
Connection connection = tester.createConnection(uuid);
```

作成されたConnectionオブジェクトはTesterで管理され、UUIDで区分されます。すでに作成されているオブジェクトのUUIDが入力された場合は、そのオブジェクトを返します。

Connectionは次のような機能を提供します。

### Connect

GameAnvilサーバーに接続します。

```
Future<ResultConnect> future = connection.connect(new RemoteInfo("127.0.0.1", 18200));
ResultConnect resultConnect = futre.get(); // blocked
if(resultConnect.isSuccess()){
    // connect success
}
```

`connect()`から返されたFutureの`get()`を呼び出すと、接続が成功するか失敗するまで待ってからResultConnectオブジェクトを返します。返されたResultConnectオブジェクトで結果を知ることができます。`connect()`の第二引数としてコールバックを渡して結果を受け取ることもできます。`connect()`以外の他のAPIもFutureを返して結果を待ったり、コールバックを渡して結果を受け取ることができます。

### Authentication

GameAnvilサーバーに認証をリクエストします。認証に成功すると、GameAnvilコネクタの他の機能を使用できます。

```
Future<ResultAuthentication> future = connection.authentication(accountId, password, deviceId, payload);
ResultAuthentication resultAuthentication = future.get(); // blocked
if(resultAuthentication.isSuccess){
    // authentication success
}
```

### GetChannelList

指定したサービスで使用可能なチャンネルリストをリクエストします。

```
Future<ResultChannelList> future = connection.getChannelList(serviceName);
ResultChannelList resultChannelList = future.get(); // blocked
if(resultChannelList.isSuccess){
    // authentication success
}
```

### GetChannelInfo

指定したチャンネルの情報をリクエストします。

```
Future<ResultChannelList> future = connection.getChannelList(serviceName, String channelId);
ResultChannelList resultChannelList = future.get(); // blocked
if(resultChannelList.isSuccess){
    // authentication success
}
```

### Request

サーバーにメッセージを送って、レスポンスを待ちます。

```
Future<PacketResult> future = connection.request(message);
PacketResult packetResult = future.get(); // blocked
if(packetResult.isSuccess()){
    // request success
}
```

### Send

サーバーにメッセージを送ります。

```
connection.send(message);
```

### Close

接続を切ります。 接続終了時に作成したユーザーを全てログアウトするには、引数にtrueを入力します。

```
connection.close(true);
```

### WaitForAdminKickoutNoti

管理者の強制終了通知が来るまで待ちます。管理者が強制終了する場合、通知されます。

```
Future<ResultAdminKickoutNoti> future = connection.waitForAdminKickoutNoti()
ResultAdminKickoutNoti resultAdminKickoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultAdminKickoutNoti 
```

### WaitForForceCloseNoti

強制終了通知が来るまで待ちます。サーバーから`BaseUser.closeConnection()`を呼び出したり、Authenticationが失敗したり、同じアカウントで重複ログインをしたり、UserTrasnferの途中で例外が発生した場合などに渡されます。

```
Future<ResultForceCloseNoti> future = connection.waitForForceCloseNoti();
ResultForceCloseNoti resultForceCloseNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceCloseNoti 
```

### WaitForDisconnect

ネットワーク切断通知を受け取るまで待ちます。サーバーから`BaseConnection.close()`を呼び出したり、ソケットエラーが発生したり、`Connection.close()`, `Tester.Close()` を呼び出した場合に渡されます。

```
Future<ResultDisconnect> future = connection.waitForDisconnect();
ResultDisconnect resultDisconnect = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultDisconnect 
```

## User

ログインをはじめルームの作成、入場、マッチングなどゲームに必要な主要機能を担当します。次のようにUserを作成できます。

```
User user = connection.getUserAgent(serviceName, subId);
if(user == null){
    user = connection.createUser(serviceName, subId);
}
```

`Connection.getUserAgent()`でServiceNameとSubIdでマッチングするUserがいるか確認し、いない場合は`Connection.createUser()`で新しいUserを作成します。 

Userは次のような機能を提供します。

### Login

指定したユーザータイプで指定したチャンネルにログインします。ユーザータイプとチャンネルはサーバーで設定した文字列を使います。

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

ログインしたチャンネルからログアウトします。

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

強制ログアウト通知を受け取るまで待ちます。サーバーで`BaseUser.kickout()` を呼び出す場合に渡されます。

```
Future<ResultForceLogoutNoti> future = connection.waitForForceLogoutNoti();
ResultForceLogoutNoti resultForceLogoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceLogoutNoti 
```

### CreateRoom

指定したルームタイプで指定した名前のルームを作成し、そのルームに入場します。ルームタイプはサーバーで設定した文字列を使います。

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

指定したIDのルームに入室します。指定したIDのルームがない場合は失敗します。

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

指定した名前のルームに入場します。指定した名前のルームがない場合、ルームを作成してそのルームに入場します。パーティーマッチングのためのルームの場合はusePartyをtrueにします。

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

現在のルームから退室します。

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

強制退室通知が来るまで待ちます。サーバーでBaseUser.kickoutRoom()を呼び出す場合に渡されます。

```
Future<ResultForceLeaveRoomNoti> future = connection.waitForForceLeaveRoomNoti();
ResultForceLeaveRoomNoti resultForceLeaveRoomNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceLeaveRoomNoti 
```

### MatchUserStart

ユーザーのマッチメイキングをリクエストします。すでにルームに入場している場合など、サーバーの条件によってリクエストが失敗する場合があります。 WaitForMatchUserDoneNotiでマッチ成功通知、WaitForMatchUserTimeoutNotiでマッチタイムアウト通知を受け取ることができます。

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

ユーザーのマッチメイキングリクエストをキャンセルします。マッチリクエスト中でない場合や、すでにマッチングが成功した場合、タイムアウトが発生した場合は、失敗することがあります。

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

ユーザーマッチメイキングまたはパーティーマッチメイキング完了通知を受け取るまで待ちます。 

```
Future<ResultMatchUserDone> future = connection.waitForMatchUserDoneNoti();
ResultMatchUserDone resultMatchUserDone = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchUserDone 
```

### WaitForMatchUserTimeoutNoti

ユーザーマッチメイキングまたはパーティーマッチメイキングのタイムアウト通知を受け取るまで待ちます。 

```
Future<ResultMatchUserTimeout> future = connection.waitForMatchUserTimeoutNoti();
ResultMatchUserTimeout resultMatchUserTimeout = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchUserTimeout 
```

### MatchPartyStart

パーティーマッチメイキングをリクエストします。パーティーマッチメイキングのためのルームに入場した状態でリクエストできます。WaitForMatchUserDoneNotiでマッチ成功通知、WaitForMatchUserTimeoutNotiでマッチタイムアウト通知を受け取ることができます。

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

パーティーマッチメイキング開始通知を受け取るまで待ちます。パーティーマッチメイキングのためのルームで他の人がパーティーマッチメイキングを開始した場合、通知されます。

```
Future<ResultMatchPartyStart> future = connection.waitForMatchPartyStartNoti();
ResultMatchPartyStart resultMatchPartyStart = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchPartyStart 
```

### MatchPartyCancel

パーティーマッチメイキングのリクエストをキャンセルします。パーティーマッチメイキング中でない場合や、すでにパーティーマッチメイキングに成功した場合、またはタイムアウトが発生した場合は失敗することがあります。

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

パーティーマッチメイキングのキャンセル通知を受け取るまで待ちます。パーティーマッチメイキング中に他の人がパーティーマッチメイキングをキャンセルした場合、通知されます。

```
Future<ResultMatchPartyCancel> future = connection.waitForMatchPartyCancelNoti();
ResultMatchPartyCancel resultMatchPartyCancel = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchPartyCancel 
```

### MatchRoom

ルームのマッチメイキングをリクエストします。ルームがない場合、任意のルームを作成してそのルームに入場することもできます。

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

指定したチャンネルに移動します。

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

チャンネル移動通知を受け取るまで待ちます。ルーム入場、マッチメイキングなどの理由でチャンネルを移動すると通知されます。

```
Future<ResultMoveChannelNoti> future = connection.waitForMoveChannelNoti();
ResultMoveChannelNoti resultMoveChannelNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMoveChannelNoti 
```

### Request

サーバーにメッセージを送って、レスポンスを待ちます。

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

サーバーにメッセージを送ります。

```
user.send(message);
```

### WaitForNotice

告知の通知を受け取るまで待ちます。管理者から告知を送ったり、REST APIを使って告知を送った場合に伝達されます。

```
Future<ResultNotice> future = connection.waitForMoveChannelNoti();
ResultNotice resultNotice = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultNotice 
```

## テストコードの作成

### 基本設定

JUnitを使ってテストコードを作成する時、次のようにBeforeClass、AfterClass、Afterコードを作成します。

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

このように書くと、先行テストで使ったユーザーが残って次のテストに影響を与えるのを防ぐことができます。

### Request/Responseテスト

GameHammerを使ったテストコードは大きく2つのタイプに分けられます。一つ目はRequest/Response形式のテストコードです。クライアントでRequestを送る場合、サーバーは必ずResponseを送る必要があります。レスポンスを送らない場合、timeoutが発生します。GameAnvilコネクタが提供するAPIのほとんどがこのようなRequest/Response方式であり、GameHammerでもこのようなRequest/Response方式のテストをサポートします。

例えば、Connectionのconnectに対するテストコードは次のように書くことができます。

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

`connection.connect()` を呼び出してサーバーに接続します。この時、返された `Future` の `get()` を呼び出すと `connection.connect()` が完了するまで待って、完了したらその結果である `ResultConnect` を返します。 ここで返された `ResultConnect` を利用して成功したかどうかを判断できます。

もう一つの例としてゲームで使うメッセージに対するRequest/Response方式のテストコードは次のように書くことができます。

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

まずメッセージを作成し、これを `connection.request()` の引数としてサーバーに送信します。この時、返された `Future` の `get()` を呼び出すと、サーバーからレスポンスが来るかタイムアウトが発生するまで待ち、レスポンスが来るかタイムアウトが発生したら `PacketResult` を返します。ここで返された `PacketResult` を使って成功したかどうかを判断できます。成功したら `PacketResult.getStream()` を使ってメッセージを解析して内容を確認できます。

Connection、UserのAPIのうち、Futureを返すAPIは全てこの方法でテストできます。

### Send/Receive

クライアントはSendを送ってレスポンスを待たないです。 そして、サーバーでもクライアントの動作と関係なくSendを送ることができます。テストは次のように書くことができます。

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
