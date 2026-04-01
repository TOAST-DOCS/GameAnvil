## Game > GameAnvil > テスト開発ガイド > 機能テスト開発ガイド

### Tester

GameHammerを使用するための基本モジュールです。機能テストや性能テストを行うには、必須で1つのTesterオブジェクトを生成する必要があります。基本設定とConnectionオブジェクトの管理を担当します。

#### オブジェクト生成

Testerオブジェクトの生成は、以下のようにビルダーを通じて行えます。

```java
Tester tester = Tester.newBuilder().build();
```

#### オプション追加

オブジェクト生成時にビルダーを通じて詳細オプションを追加できます。

```java
Tester tester = Tester.newBuilder()
                    .setDefaultPacketTimeoutSeconds(10)
                    .setEnableTimeoutCallback(false)
                    .setPingIntervalSeconds(3)
                    .addProtoBufClass(RPSGame.getDescriptor())
                    .build();
```

オプション設定を外部ファイルで作成し、ランタイムに読み込むこともできます。vmoption `config.file=PATH`でパスを指定するか、resourceフォルダ配下にGameHammerConfg.jsonファイルを作成しておくと、TesterConfigLoaderで自動的に読み込まれます。以下の方法で、読み込んだオプション設定が適用されたTesterを生成できます。

```json
{
  "requestTimeoutSeconds": 10,
  "isTimeoutCallbackEnabled": false,
  "pingIntervalSeconds": 3
}
```

```java
Tester tester = Tester.newBuilderWithConfig()
                    .addProtoBufClass(RPSGame.getDescriptor())
                    .build();
```

### Connection

Connectionはゲームサーバーとの接続、認証などの機能を処理し、ユーザーの管理を担当します。

#### オブジェクト生成

以下のようにTesterを通じて生成します。

```java
private static int uuid = 0;
Connection connection = tester.createConnection(uuid++);
```

生成されたConnectionオブジェクトはTesterでまとめて管理され、UUIDで区分されます。すでに生成されたオブジェクトのUUIDが入力されると、該当オブジェクトを返します。

#### Connect

GameAnvilサーバーに接続します。

```java
Future<ResultConnect> future = connection.connect(new RemoteInfo("127.0.0.1", 18200));
ResultConnect resultConnect = futre.get(); // blocked
if (resultConnect.isSuccess()) {
    // connect success
}
```

`connect()`から返されたFutureの`get()`を呼び出すと、接続が成功するか失敗するまで待機した後、ResultConnectオブジェクトを返します。返されたResultConnectオブジェクトを通じて結果を知ることができます。

`connect()`の2番目の引数にコールバックを渡して結果を受け取ることもできます。

```java
connection.connect(this::connectResponseListener, new RemoteInfo("127.0.0.1", 18200));

public void connectResponseListener(ResultConnect resultConnect, ConsoleTestActor scenarioActor) {
    if (resultConnect.isSuccess()) {
        // connect success
    }
}
```

`connect()`以外に他のAPIもFutureを返して結果を待つか、コールバックを渡して結果を受け取ることができます。このガイドではFuture方式を代表として説明します。

#### Authentication

GameAnvilサーバーに認証をリクエストします。認証に成功して初めてGameAnvilコネクタの他の機能を使用できます。

```java
Future<ResultAuthentication> future = connection.authentication(accountId, password, deviceId, payload);
ResultAuthentication resultAuthentication = future.get(); // blocked
if (resultAuthentication.isSuccess) {
    // authentication success
}
```

#### GetChannelList

指定したサービスで使用可能なチャンネルリストをリクエストします。

```java
Future<ResultChannelList> future = connection.getChannelList(serviceName);
ResultChannelList resultChannelList = future.get(); // blocked
if (resultChannelList.isSuccess) {
    // get channel list success
}
```

#### GetChannelInfo

指定したチャンネルの情報をリクエストします。

```java
Future<ResultChannelInfo> future = connection.getChannelInfo(serviceName, String channelId);
ResultChannelInfo resultChannelInfo = future.get(); // blocked
if (resultChannelInfo.isSuccess) {
    // get channel info success
}
```

#### Request

サーバーへメッセージを送信し、レスポンスを待ちます。

```java
GeneratedMessageV3 message;

Future<PacketResult> future = connection.request(message);
PacketResult packetResult = future.get(); // blocked
if (packetResult.isSuccess()) {
    // request success
}
```

#### Send

サーバーへメッセージを送信します。

```java
GeneratedMessageV3 message;

connection.send(message);
```

#### Close

接続を切断します。接続終了時に生成したユーザーを全てログアウト処理するには、引数にtrueを入力します。

```java
connection.close(true);
```

#### WaitForAdminKickoutNoti

Admin強制終了通知を受け取るまで待ちます。Adminから強制終了する場合、通知が送信されます。

```java
Future<ResultAdminKickoutNoti> future = connection.waitForAdminKickoutNoti();
ResultAdminKickoutNoti resultAdminKickoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

#### WaitForDisconnect

ネットワーク接続切断通知を受け取るまで待ちます。サーバーで`IConnection::close()`を呼び出すか、ソケットエラーが発生するか、`Connection::close()`, `Tester::Close()`を呼び出す場合に送信されます。

```java
Future<ResultDisconnect> future = connection.waitForDisconnect();
ResultDisconnect resultDisconnect = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

### User

ログインをはじめ、ルーム生成、入室、マッチングなどゲームに必要な主要機能を担当します。次のようにUserを生成できます。

```java
User user = connection.getUserAgent(serviceName, subId);
if (user == null) {
    user = connection.createUser(serviceName, subId);
}
```

`Connection::getUserAgent()`でServiceNameとSubIdでマッチングするUserがいるか確認し、いない場合は`Connection::createUser()`で新しいUserを生成します。

Userは次のような機能を提供します。

#### Login

指定したユーザータイプで指定したチャンネルにログインします。ユーザータイプとチャンネルはサーバーで設定した文字列を使用します。

```java
Future<ResultLogin> future = user.login(userType, channelId, payload);
ResultLogin resultLogin = future.get(); // blocked
if (resultLogin.isSuccess()) {
    // login success
}
```

#### Logout

ログインしたチャンネルからログアウトします。

```java
Future<ResultLogout> future = user.logout(userType, channelId, payload);
ResultLogout resultLogout = future.get(); // blocked
if (resultLogout.isSuccess()) {
    // logout success
}
```

#### WaitForForceLogoutNoti

強制ログアウト通知を受け取るまで待ちます。サーバーで`IUser::kickout()`を呼び出す場合に送信されます。

```java
Future<ResultForceLogoutNoti> future = connection.waitForForceLogoutNoti();
ResultForceLogoutNoti resultForceLogoutNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

#### CreateRoom

指定したルームタイプで指定した名前のルームを生成し、そのルームに入室します。ルームタイプはサーバーで設定した文字列を使用します。

```java
Future<ResultCreateRoom> future = user.createRoom(roomType, roomName, payload);
ResultCreateRoom resultCreateRoom = future.get(); // blocked
if (resultCreateRoom.isSuccess()) {
    // createRoom success
}
```

#### JoinRoom

指定したIDのルームに入室します。指定したIDのルームがない場合は失敗します。

```java
Future<ResultJoinRoom> future = user.joinRoom(roomType, roomId, payload);

ResultJoinRoom resultJoinRoom = future.get(); // blocked
if (resultJoinRoom.isSuccess()) {
    // joinRoom success
}
```

#### NamedRoom

指定した名前のルームに入室します。指定した名前のルームがない場合はルームを生成し、そのルームに入室します。パーティーマッチングのためのルームである場合はusePartyにtrueを入力します。

```java
Future<ResultNamedRoom> future = user.namedRoom(roomType, roomName, useParty, payload);
ResultNamedRoom resultNamedRoom = future.get(); // blocked
if (resultNamedRoom.isSuccess()) {
    // namedRoom success
}
```

#### LeaveRoom

現在のルームから退場します。

```java
Future<ResultLeaveRoom> future = user.leaveRoom(payload);
ResultLeaveRoom resultLeaveRoom = future.get(); // blocked
if (resultLeaveRoom.isSuccess()) {
    // leaveRoom success
}
```

#### WaitForForceLeaveRoomNoti

ルーム強制退場通知を受け取るまで待ちます。サーバーでBaseUser.kickoutRoom()を呼び出す場合に送信されます。

```java
Future<ResultForceLeaveRoomNoti> future = connection.waitForForceLeaveRoomNoti();
ResultForceLeaveRoomNoti resultForceLeaveRoomNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

#### MatchUserStart

ユーザーマッチメイキングをリクエストします。すでにルームに入室している場合など、サーバーの条件によってリクエストが失敗することがあります。WaitForMatchUserDoneNotiを通じてマッチ成功通知、WaitForMatchUserTimeoutNotiを通じてマッチタイムアウト通知を受け取ることができます。

```java
Future<ResultMatchUserStart> future = user.matchUserStart(roomType, payload);
ResultMatchUserStart resultMatchUserStart = future.get(); // blocked
if (resultMatchUserStart.isSuccess()) {
    // matchUserStart success
}
```

#### MatchUserCancel

ユーザーマッチメイキングリクエストをキャンセルします。マッチリクエスト中でない場合、すでにマッチングが成功しているかタイムアウトが発生している場合は失敗することがあります。

```java
Future<ResultMatchUserCancel> future = user.matchUserCancel(roomType);
ResultMatchUserCancel resultMatchUserCancel = future.get(); // blocked
if (resultMatchUserCancel.isSuccess()) {
    // matchUserCancel success
}
```

#### WaitForMatchUserDoneNoti

ユーザーマッチメイキングまたはパーティーマッチメイキング完了通知を受け取るまで待ちます。

```java
Future<ResultMatchUserDone> future = connection.waitForMatchUserDoneNoti();
ResultMatchUserDone resultMatchUserDone = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

#### WaitForMatchUserTimeoutNoti

ユーザーマッチメイキングまたはパーティーマッチメイキングに対するタイムアウト通知を受け取るまで待ちます。

```java
Future<ResultMatchUserTimeout> future = connection.waitForMatchUserTimeoutNoti();
ResultMatchUserTimeout resultMatchUserTimeout = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

#### MatchPartyStart

パーティーマッチメイキングをリクエストします。パーティーマッチメイキングのためのルームに入室した状態でリクエストできます。WaitForMatchUserDoneNotiを通じてマッチ成功通知、WaitForMatchUserTimeoutNotiを通じてマッチタイムアウト通知を受け取ることができます。

```java
Future<ResultMatchPartyStart> future = user.matchPartyStart(roomType, payload);
ResultMatchPartyStart resultMatchPartyStart = future.get(); // blocked
if (resultMatchPartyStart.isSuccess()) {
    // matchPartyStart success
}
```

#### WaitForMatchPartyStartNoti

パーティーマッチメイキング開始通知を受け取るまで待ちます。パーティーマッチメイキングのためのルームで他の人がパーティーマッチメイキングを開始した場合に送信されます。

```java
Future<ResultMatchPartyStart> future = connection.waitForMatchPartyStartNoti();
ResultMatchPartyStart resultMatchPartyStart = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

#### MatchPartyCancel

パーティーマッチメイキングリクエストをキャンセルします。パーティーマッチメイキング中でない場合、すでにパーティーマッチメイキングに成功しているかタイムアウトが発生した場合は失敗することがあります。

```java
Future<ResultMatchPartyCancel> future = user.matchPartyCancel(roomType);
ResultMatchPartyCancel resultMatchPartyCancel = future.get(); // blocked
if (resultMatchPartyCancel.isSuccess()) {
    // matchPartyCancel success
}
```

#### WaitForMatchPartyCancelNoti

パーティーマッチメイキングキャンセル通知を受け取るまで待ちます。パーティーマッチメイキング中に他の人がパーティーマッチメイキングをキャンセルした場合に送信されます。

```java
Future<ResultMatchPartyCancel> future = connection.waitForMatchPartyCancelNoti();
ResultMatchPartyCancel resultMatchPartyCancel = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

#### MatchRoom

ルームマッチメイキングをリクエストします。ルームがない場合は任意のルームを生成し、そのルームに入室することもできます。

```java
Future<ResultMatchRoom> future = user.matchRoom(roomType);
ResultMatchRoom resultMatchRoom = future.get(); // blocked
if (resultMatchRoom.isSuccess()) {
    // matchRoom success
}
```

#### MoveChannel

指定したチャンネルに移動します。

```java
Future<ResultMoveChannel> future = user.moveChannel(roomType);
ResultMoveChannel resultMoveChannel = future.get(); // blocked
if (resultMoveChannel.isSuccess()) {
    // moveChannel success
}
```

#### WaitForMoveChannelNoti

チャンネル移動通知を受け取るまで待ちます。ルーム入室、マッチメイキングなどの理由でチャンネルを移動することになると送信されます。

```java
Future<ResultMoveChannelNoti> future = connection.waitForMoveChannelNoti();
ResultMoveChannelNoti resultMoveChannelNoti = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

#### Request

サーバーへメッセージを送信し、レスポンスを待ちます。

```java
Future<PacketResult> future = user.request(message);
PacketResult packetResult = future.get(); // blocked
if(packetResult.isSuccess()){
    // request success
}
```

#### Send

サーバーへメッセージを送信します。

```java
user.send(message);
```

#### WaitForNotice

お知らせ通知を受け取るまで待ちます。Adminからお知らせを送信するか、REST APIを利用してお知らせを送信する場合に送信されます。

```java
Future<ResultNotice> future = connection.waitForMoveChannelNoti();
ResultNotice resultNotice = future.get(WAIT_TIME_OUT, TimeUnit.MILLISECOND); // blocked
```

### ユニットテストコード作成

JUnitを利用してテストコードを作成する方法を代表として説明します。

#### 基本設定

次のようにBeforeClass、AfterClass、Afterコードを作成します。

beforeClass、afterClassはテストの開始時点と終了時点に一度だけ呼び出されます。テストを進めるために必須となるTesterオブジェクトをこの時に生成して整理します。

各テストケース終了時に呼び出されるafterで`Tester::closeAllConnections()`を実行し、先行テストで使用したユーザーが残って次のテストに影響を与えるのを防ぐことができます。

```java
public class TestWithGameHammer {
    static Tester tester;

    @BeforeClass
    public static void beforeClass() {
        tester = Tester.newBuilder()
                    .addProtoBufClass(RPSGame.getDescriptor())
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
        Connection connection = tester.createConnection(0);
        ResultConnect resultConnect = connection.connect(connection.getConfig().getNextTargetServer()).get(1000, TimeUnit.MILLISECONDS);
        assertEquals(ResultCodeConnect.CONNECT_SUCCESS, resultConnect.getResultCode());
    }

    @Test
    public void someTest() {
        // some test code
    }
```

#### Request/Responseテスト

クライアントからRequestを送信する場合、サーバーでは必ずResponseを送信する必要があります。レスポンスを送信しないとtimeoutが発生します。GameAnvilコネクタが提供するAPIの大部分がこのようなRequest/Response方式であり、GameHammerでもこのようなRequest/Response方式のテストをサポートしています。

例えばConnectionのconnectに対するテストコードは、次のように作成できます。

```java
@Test
public void Connect() {
    Connection connection = tester.createConnection(uuid);

    Future<ResultConnect> future = connection.connect(new RemoteInfo("127.0.0.1", 11200));
    ResultConnect resultConnect = futre.get(); // blocked
    Assert.assertTrue("Connection fail", resultConnect.isSuccess());

    // more test code
}
```

`Connection::connect()`を呼び出してサーバーに接続します。この時返された`Future::get()`を呼び出すとコネクトが完了するまで待機し、完了するとその結果である`ResultConnect`オブジェクトを返します。ここで返された`ResultConnect`オブジェクトを利用して成否を判断できます。

もう1つの例として、ゲームで使用するメッセージに対するRequest/Response方式のテストコードは、次のように作成できます。

```java
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

まずメッセージを作成し、これを`Connection::request()`の引数に入れてサーバーへ送信します。この時返された`Future`の`Future::get()`を呼び出すと、サーバーから応答があるかタイムアウトが発生するまで待機し、終了すると`PacketResult`を返します。ここで返された`PacketResult`を利用して成否を判断できます。成功した場合、`PacketResult::getStream()`を利用してメッセージをパースし、内容を確認できます。

`Connection`、`User`のAPIのうち、`Future`を返すAPIは全てこの方式を使用してテストできます。

#### Send/Receiveテスト

Request/Responseとは異なり、Sendは送信後にレスポンスを待ちません。そしてサーバーでもクライアントの動作に関係なくSendを送信できます。テストは次のように作成できます。

```java
@Test
public void SendTest() {
    Connection connection = tester.createConnection(uuid);
    // assume aleady connected.

    Messages.SendToServer sendToServer = Messages.SendToServer.newBuilder().build();
    connection.send(sendToServer);

    Future<PacketResult> future = connection.waitFor(Messages.SendToClient.getDescription());
    PacketResult packetResult = future.get(3000, TimeUnit.MILLISECONDS); // blocked
    Assert.assertTrue("Receive fail", packetResult.isSuccess());

    Messages.SendToClient sendToClient = Messages.SendToClient.parseFrom(packetResult.getStream());

    // more test code
}
```
