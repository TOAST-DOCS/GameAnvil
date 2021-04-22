## Game > GameAnvil > テスト開発ガイド > 機能テスト開発ガイド

## Tester

GameHammerを使用するための基本モジュールです。基本設定とConnectionオブジェクトの管理を担当します。Testerオブジェクトを作成するには、まずBuilderを作成し、テストに必要なオプションを設定した後、`build()`を呼び出してください。

```
Tester tester = Tester.newBuilder()
                    .addProtoBufClass(0, RPSGame.getDescriptor())
                    .build();
```

オプション設定を外部から読み込むこともできます。vmoption `config.file=PATH`でパスを指定するか、resourceフォルダの下にGameHammerConfg.jsonファイルを作成しておくと、TesterConfigLoaderで自動的に読み込みます。次のような方法で読み込んだオプション設定が適用されたTesterを作成できます。

```
Tester tester = Tester.newBuilderWithConfig()
                    .addProtoBufClass(0, RPSGame.getDescriptor())
                    .build();
```

## Connection

ゲームサーバーとの接続、認証などの機能を処理し、ユーザーの管理を担当します。次のようにTesterを利用して作成します。

```
Connection connection = tester.createConnection(uuid);
```

作成されたConnectionオブジェクトはTesterで管理し、uuidで区別されます。作成されたオブジェクトのuuidが入力された場合は、そのオブジェクトを返します。

Connectionは次のような機能を提供します。 

### Connect

GameAnvilサーバーに接続します。

```
Future<ResultConnect> future = connection.connect(new RemoteInfo("127.0.0.1", 11200));
ResultConnect resultConnect = futre.get(); // blocked
if(resultConnect.isSuccess()){
    // connect success
}
```

`connect()`から返されたFutureの`get()`を呼び出すと、接続が成功するか失敗するまで待機した後、ResultConnectオブジェクトを返します。返されたResultConnectオブジェクトで結果を確認できます。`connect()`の2つ目の引数でコールバックを渡して結果を受け取ることもできます。`connect()`以外の他のAPIもFutureをリターンして結果を待つか、コールバックを渡して結果を受け取ることができます。

### Authentication

GameAnvilサーバーに認証をリクエストします。認証に成功するとGameAnvilコネクタの他の機能を使用できます。

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

サーバーにメッセージを送り、レスポンスを待ちます。

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

接続を切ります。接続終了時、作成したユーザーを全てログアウトするには、引数にtrueを入力します。

```
connection.close(true);
```

### WaitForAdminKickoutNoti

Admin強制終了通知を受け取るまで待機します。Adminで強制終了した場合に伝達されます。

```
Future<ResultAdminKickoutNoti> future = connection.waitForAdminKickoutNoti()
ResultAdminKickoutNoti resultAdminKickoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultAdminKickoutNoti 
```

### WaitForForceCloseNoti

強制終了通知を受け取るまで待機します。次のような場合に強制終了通知を受け取ります。1．サーバーで`BaseUser.closeConnection()`を呼び出す。2．Authenticationが失敗する。3．同じアカウントで重複ログインを行う。4．UserTrasnfer中に例外が発生する。

```
Future<ResultForceCloseNoti> future = connection.waitForForceCloseNoti();
ResultForceCloseNoti resultForceCloseNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceCloseNoti 
```

### WaitForDisconnect

ネットワーク接続切断通知を受け取るまで待機します。次のような場合に通知を受け取ります。1．サーバーで`BaseConnection.close()`を呼び出す。2．ソケットエラーが発生する。3．`Connection.close()`、`Tester.Close()`を呼び出す。

```
Future<ResultDisconnect> future = connection.waitForDisconnect();
ResultDisconnect resultDisconnect = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultDisconnect 
```

## User

ログインをはじめ、ルーム作成、入場、マッチングなど、ゲームに必要な主な機能を担当します。次のようにUserを作成できます。

```
User user = connection.getUserAgent(serviceName, subId);
if(user == null){
    user = connection.createUser(serviceName, subId);
}
```

`Connection.getUserAgent()`でServiceNameとSubIdにマッチするUserがいるかを確認し、いない場合、`Connection.createUser()`で新しいUserを作成します。 

Userは次のような機能を提供します。 

### Login

指定したユーザータイプで指定したチャンネルにログインします。ユーザータイプとチャンネルはサーバーで設定した文字列を使用します。

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

強制ログアウト通知を受け取るまで待機します。サーバーで`BaseUser.kickout()`を呼び出す場合に伝達されます。

```
Future<ResultForceLogoutNoti> future = connection.waitForForceLogoutNoti();
ResultForceLogoutNoti resultForceLogoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceLogoutNoti 
```

### CreateRoom

指定したルームタイプで指定した名前のルームを作成し、そのルームに入場します。ルームタイプはサーバーで設定した文字列を使用します。

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

指定したIDのルームに入場します。指定したIDのルームがない場合は失敗します。

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

指定した名前のルームに入場します。指定した名前のルームがない場合は、ルームを作成し、そのルームに入場します。パーティマッチング用のルームの場合はusePartyをtrueにします。

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

現在のルームから退出します。

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

ルーム強制退出通知を受け取るまで待機します。サーバーでBaseUser.kickoutRoom()を呼び出す場合に伝達されます。

```
Future<ResultForceLeaveRoomNoti> future = connection.waitForForceLeaveRoomNoti();
ResultForceLeaveRoomNoti resultForceLeaveRoomNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultForceLeaveRoomNoti 
```

### MatchUserStart

ユーザーマッチメイキングをリクエストします。すでにルームに入場している場合など、サーバーの条件によってリクエストが失敗することがあります。WaitForMatchUserDoneNotiを介してマッチ成功通知を、WaitForMatchUserTimeoutNotiを介してマッチタイムアウト通知を受け取れます。

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

ユーザーマッチメイキングリクエストをキャンセルします。マッチリクエスト中ではない場合や、すでにマッチングが成功していたり、タイムアウトが発生した場合は失敗することがあります。

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

ユーザーマッチメイキングまたはパーティマッチメイキング完了通知を受け取るまで待機します。

```
Future<ResultMatchUserDone> future = connection.waitForMatchUserDoneNoti();
ResultMatchUserDone resultMatchUserDone = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchUserDone 
```

### WaitForMatchUserTimeoutNoti

ユーザーマッチメイキングまたはパーティマッチメイキングのタイムアウト通知を受け取るまで待機します。

```
Future<ResultMatchUserTimeout> future = connection.waitForMatchUserTimeoutNoti();
ResultMatchUserTimeout resultMatchUserTimeout = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchUserTimeout 
```

### MatchPartyStart

パーティマッチメイキングをリクエストします。パーティマッチメイキング用のルームに入場した状態でリクエストできます。WaitForMatchUserDoneNotiを介してマッチ成功通知を、WaitForMatchUserTimeoutNotiを介してマッチタイムアウト通知を受け取れます。

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

パーティマッチメイキング開始通知を受け取るまで待機します。パーティマッチメイキング用のルームで他の人がパーティマッチメイキングを始めた場合に伝達されます。

```
Future<ResultMatchPartyStart> future = connection.waitForMatchPartyStartNoti();
ResultMatchPartyStart resultMatchPartyStart = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchPartyStart 
```

### MatchPartyCancel

パーティマッチメイキングリクエストをキャンセルします。パーティマッチメイキング中ではない場合や、すでにパーティマッチメイキングに成功していたり、タイムアウトが発生した場合は、失敗することがあります。

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

パーティマッチメイキングキャンセル通知を受け取るまで待機します。パーティマッチメイキング中に他の人がパーティマッチメイキングをキャンセルした場合に伝達されます。

```
Future<ResultMatchPartyCancel> future = connection.waitForMatchPartyCancelNoti();
ResultMatchPartyCancel resultMatchPartyCancel = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMatchPartyCancel 
```

### MatchRoom

ルームマッチメイキングをリクエストします。ルームがない場合、任意のルームを作成し、そのルームに入場することもできます。

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

チャンネル移動通知を受け取るまで待機します。ルーム入場、マッチメイキングなどの理由でチャンネルを移動する場合に伝達されます。

```
Future<ResultMoveChannelNoti> future = connection.waitForMoveChannelNoti();
ResultMoveChannelNoti resultMoveChannelNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultMoveChannelNoti 
```

### Request

サーバーにメッセージを送り、レスポンスを待ちます。

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

告知通知を受け取るまで待機します。Adminから告知を送ったり、REST APIを利用して告知を送る場合に伝達されます。

```
Future<ResultNotice> future = connection.waitForMoveChannelNoti();
ResultNotice resultNotice = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
// resultNotice 
```

## テストコード作成

### 基本設定

JUnitを利用してテストコードを作成する時、次のようにBeforeClass、AfterClass、Afterコードを作成します。

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

このように作成すると、先行テストで使用したユーザーが残って、次のテストに影響を及ぼすことを防ぐことができます。

### Request/Responseテスト

GameHammerを利用したテストコードは、大きく2つのタイプに分けられます。1つ目はRequest/Response形式のテストコードです。クライアントからRequestを送る場合、サーバーからは必ずResponseを送る必要があります。レスポンスを送らない場合、timeoutが発生します。GameAnvilコネクタが提供するAPIの大部分がこのようなRequest/Response方式で、GameHammerでもこのようなRequest/Response方式のテストをサポートします。

例えば、Connectionのconnectのテストコードは次のように作成できます。

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

`connection.connect()`を呼び出し、サーバーに接続します。このとき返された`Future`の`get()`を呼び出すと、`connection.connect()`が完了するまで待機し、完了するとその結果として`ResultConnect`をリターンします。ここでリターンされた`ResultConnect`を利用して成否を判断できます。 

また他の例として、ゲームで使用するメッセージのRequest/Response方式テストコードは次のように作成できます。 

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

まずメッセージを作成し、これを`connection.request()`の引数に入れ、サーバーに送信します。このとき返された`Future`の`get()`を呼び出すとサーバーでレスポンスするか、タイムアウトが発生するまで待機し、レスポンスを受け取るか、タイムアウトが発生すると、`PacketResult`を返します。ここで返された`PacketResult`を利用して成否を判断できます。成功すると`PacketResult.getStream()`を利用してメッセージを解析して、内容を確認できます。 

Connnection、UserのAPIのうち、FutureをリターンするAPIは全てこのような方式でテストできます。

### Send/Receive

クライアントはSendを送り、レスポンスを待機しません。そしてサーバーでもクライアントの動作に関係なくSendを送れます。テストは次のように作成できます。

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
