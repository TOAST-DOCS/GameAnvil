## Game > GameAnvil > サーバー開発ガイド > 実装ガイド

基本的なサーバーを実装するために知っておくべきエンジンの中核的な部分を中心に、基本的な要素と、これを実装する方法について説明します。コードレベルの資料は[GameAnvilレファレンスサーバー](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-reference-server)を参照してください。また、[GameAnvil API Reference](http://10.162.4.61:9090/gameanvil)からJavaDoc文書も利用できます。可能であればレファレンスサーバープロジェクトとこのガイド文書もみていただくことを推奨します。



### Note

*GameAnvilは、エンジン内部はもちろん、すべてのサンプルコードでメッセージハンドラについて_で始まるネーミングを使用します。すなわち、これから登場するサンプルコードでも_で始まるすべてのクラスはメッセージハンドラです。このようなメッセージハンドラの実装は、この文書では説明しません。前述したレファレンスサーバープロジェクトやJavaDocを利用してコードを確認するのが最も簡単で正確な方法です。*



GameAnvilの大部分の機能はコールバック方式で提供します。すなわち、エンジンのユーザーは**GameAnvilが提供する基本クラス(Base Classes)を継承した後、コールバックメソッドを再定義**する形で大部分の機能を使用することになります。この過程でエンジンのユーザーが必要なコールバックメソッドだけを実装すればよいので、一部のコールバックメソッドは無視することもできます。この文書は、このようなコールバックメソッドの実際の実装方法までは説明しません。コードレベルでは、前述したレファレンスサーバープロジェクトを参考にするのが最も良い方法です。



## 1. Node共通

すべての種類のノードは共通して以下のコールバックメソッドを再定義する必要があります。そして各ノードは用途に合わせて追加コールバックメソッドを実装することになります。以下のコードのGatewayNodeは、追加コールバックがないため、共通コールバックメソッドを示すのに適しています。特に、onPrepare()で呼び出したsetReady() APIはエンジンにノードが全て準備できたので、実際の動作を始めなさいというコマンドです。一般的にonPrepare()で呼び出します。しかし、他のノードとの通信や、任意の非同期結果に基づいて準備コマンドを下したい場合は、他のコードブロックで呼び出しても構いません。

```
public class SampleGatewayNode extends BaseGatewayNode {

    /**
     * 初期化する時に発生。
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onInit() throws SuspendExecution {        
    }

    /**
     * Prepare時に呼び出される。
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPrepare() throws SuspendExecution {
        setReady(); // 注意！
    }

    /**
     * Ready段階で呼び出される。
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onReady() throws SuspendExecution {        
    }

    /**
     * パケットが送信される時に呼び出される。
     *
     * @param packet処理するパケット
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {        
    }

    /**
     * pauseする時に呼び出される。
     *
     * @param payload contentsから送信する追加情報
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPause(Payload payload) throws SuspendExecution {        
    }

    /**
     * resumeする時に呼び出される。
     *
     * @param payload contentsから送信する追加情報
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onResume(Payload payload) throws SuspendExecution {        
    }

    /**
     * NonNetworkNode終了用のshutdownを命じた時に呼び出される。
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onShutdown() throws SuspendExecution {        
    }
}
```



## 2. Gateway Node

GatewayNodeはクライアントが接続するノードです。 BaseGatewayNodeクラスを継承して共通コールバックメソッドだけを再定義してください。その他、GatewayNode自体で特別に追加で実装する事項はありません。GatewayNodeはクライアントからのコネクションとセッションを管理します。これらの関係は次の図のとおりです。

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

一般的にクライアントはGatewayNodeで1つのコネクションを確立します。この時、そのコネクションに対して認証手順を進行して成功した場合に限り、1つ以上のセッションを作成できます。それぞれのセッションは異なるゲームサービスへの論理的な接続単位です。上の図はクライアントが1つのコネクションを介してGameサービスとChatサービスセッションを作成した様子です。



### 2-1. コネクション(Connection)

コネクションは、クライアントの物理的接続自体を意味します。クライアントは固有のAccountIdを利用してコネクション上で認証手続きを行うことができます。認証が成功すると、AccountIdは作成されたコネクションにマッピングされます。コネクションは以下のようなコールバックメソッドを再定義します。このとき、AccountIdは任意のプラットフォームで認証した後、取得するユーザーのキー値などを使用できます。例えば、Gamebaseを介して認証した後、UserIdを取得すると、この値をGameAnvilの認証過程でAccountIdとして使用できます。

```
public class SampleConnection extends BaseConnection<SampleGameSession> {

    /**
     * authenticate時に呼び出される。
     *
     * @param accountIdアカウントのID
     * @param passwordアカウントパスワード
     * @param deviceIdクライアントの端末ID
     * @param payloadクライアントから伝達されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @return falseクライアントとの接続が終了
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onAuthenticate(final String accountId, final String password, final String deviceId, final Payload payload, final Payload outPayload) throws SuspendExecution {        
    }

    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {        
    }

    /**
     * login呼び出し前に呼び出される。
     *
     * @param outPayloadクライアントに伝達するペイロード
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPreLogin(Payload outPayload) throws SuspendExecution {        
    }

    /**
     * ログイン成功後に呼び出される。
     *
     * @param sessionログインに使用したセッションオブジェクト
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPostLogin(U session) throws SuspendExecution {        
    }

    /**
     * ログアウト後に呼び出される。
     *
     * @param sessionゲームノードのユーザーに接続されたセッションオブジェクト
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPostLogout(U session) throws SuspendExecution {
    }

    @Override
    public void onPause() throws SuspendExecution {
    }

    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * クライアントと接続が切断された時に呼び出される。
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    public void onDisconnect() throws SuspendExecution {        
    }
}
```



### 2-2. セッション(Session)

コネクションを確立したクライアントは、そのコネクション上でサービスごとに1つずつGameNodeに論理的なセッションを結ぶことができます。GameAnvilは内部的にコネクションのAccountIdとセッションのSubIdを"AccoundId:SubId"の形で組み合わせて固有のセッションとそのセッションを使用中のユーザーを区別します。これらのセッションは次のようにそのセッションで処理するメッセージとハンドラを登録した後、コールバックメソッドを利用してディスパッチできます。この時、subIdはユーザーが任意で定めたルールに合わせてそのコネクション内で固有の値として割り当ててください。

```
public class SampleSession extends BaseSession {

    private static PacketDispatcher dispatcher = new PacketDispatcher();

    static {
        dispatcher.registerMsg(SampleGame.MsgToSessionReq.getDescriptor(), _MsgToSessionHandler.class);
    }

    /**
     * セッションにパケットが送信される時に呼び出される。
     *
     * @param packet処理するパケット
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        dispatcher.dispatch(this, packet);
    }
}
```



### 2-3. Bootstrap

このように作成したGatewayNodeとコネクション、そしてセッションは、Bootstrapの段階で以下のように登録できます。

```
public class Main {

    public static void main(String[] args) {

        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();
        // 登録
        bootstrap.setGateway()
            .connection(SampleConnection.class)
            .session(SampleSession.class)
            .node(SampleGatewayNode.class)
            .enableWhiteModules();

        bootstrap.run();
    }
}
```



## 3. Game Node

GameNodeは、実際のゲームオブジェクトが作成され、ゲームコンテンツを実装するノードです。GameNodeは基本的にBaseGameNode抽象クラスを継承して実装します。GameNodeが管理する代表的なオブジェクトは、GameUserとGameRoomです。これに関しては、下でさらに詳しく説明します。下記のコードはGameNodeで基本的に再定義することができるコールバックメソッドを示しています。ノード共通コールバックにチャンネル間の同期化を行うためのコールバックが追加されました。またGameNodeで処理したいプロトコルは任意のハンドラとマッピングした後、onDispatch()コールバックを利用してそのパケットを処理できます。以下のコードで1～3に該当するコメントの下のコードがこのための処理の流れを順序どおりに示しています。



```
public class SampleGameNode extends BaseGameNode {

    // 1.パケットディスパッチャーを作成 
    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    // 2.SampleGameNodeで処理したいプロトコルとハンドラをマッピング
    static {
        packetDispatcher.registerMsg(SampleGame.GameNodeTest.getDescriptor(), _GameNodeTest.class);
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onPrepare() throws SuspendExecution {
        setReady(); // 注意
    }

    /**
     * パケットが送信される時に呼び出される。
     *
     * @param packet処理するパケット
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {
        // 3.SampleGameNodeのメッセージ処理
        if (packetDispatcher.isRegisteredMessage(packet))
            packetDispatcher.dispatch(this, packet);
    }

    @Override
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {
    }

    @Override
    public void onResume(Payload payload) throws SuspendExecution {
    }

    @Override
    public void onShutdown() throws SuspendExecution {
    }

    /**
     * 同じチャンネルの他のnodeにユーザーの変化がある時に呼び出されるコールバック
     * <p>
     * updateChannelUser()を呼び出すと発生。
     *
     * @param type        チャンネル情報変更タイプ(更新/削除)
     * @param channelUserInfo変更されるユーザー情報
     * @param userId      変更対象のユーザーId
     * @param accountId   変更対象のAccount Id
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onChannelUserUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {        
    }

    /**
     * 同じチャンネルの他nodeにroomの状態の変化がある時に呼び出されるコールバック
     * <p>
     * updateChannelRoomInfo()の呼び出し時に発生。
     *
     * @param type        チャンネル情報変更タイプ(更新/削除)
     * @param channelRoomInfo変更されるRoom情報
     * @param roomId      変更対象のRoom Id
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onChannelRoomUpdate(ChannelUpdateType type, RoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {        
    }

    /**
     * クライアントでチャンネル情報をリクエストした時に呼び出されるコールバック(Base.GetChannelInfoReq)
     *
     * @param outPayloadクライアントに伝達されるChannel情報
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {        
    }
}
```



### 3-1. GameUser

ゲーム上のユーザーを表すGameUserオブジェクトは、ログインプロセスを経てGameNodeに作成されます。ユーザー単位のすべてのメッセージは、ここで処理できます。そのため、ユーザー関連のコンテンツはこのクラスを中心に実装するのが望ましいです。前述のすべての例と同様に、GameUserも処理するプロトコルとハンドラをマッピングできます。

以下のサンプルコードは、GameUserの全コールバックメソッドリストです。このうち一部はデフォルトで実装が提供されるため必要な状況でなければ再定義する必要はありません。これはGameUserだけでなく、エンジンで提供する大部分のコールバックメソッドに該当する事項です。このような部分は継承する各基本クラス(e.g.BaseUser)について[GameAnvil API Reference](http://10.162.4.61:9090/gameanvil)でJavaDoc文書を確認するか、[レファレンスサーバープロジェクト](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-reference-server)を参照してください。

以下のコールバックメソッドのうち、onLogin()は最初にゲームユーザーを作成するためにログインを試みる過程で呼び出されます。このonLogin()コールバックが成功すると、GameNode上に該当ゲームユーザーオブジェクトが作成されます。ログインが完了した後はユーザーが作成したプロトコルをベースにクライアントと直接メッセージをやりとりしてさまざまなコンテンツを処理できます。

```
public class SampleGameUser extends BaseUser {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameUserTest.getDescriptor(), _GameUserTest.class);
    }

    /**
     * login時に発生するコールバック
     *
     * @param payload    クライアントから送信した任意のペイロード
     * @param sessionPayload onPreLoginから渡されたペイロード
     * @param outPayload クライアントに渡す任意のペイロード
     * @return boolean typeがtrue：ログイン成功。 false：ログイン失敗を返す。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onLogin(final Payload payload,
                           final Payload sessionPayload,
                           Payload outPayload) throws SuspendExecution {        
    }

    /**
     * Login処理後に発生するコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPostLogin() throws SuspendExecution {

    }

    /**
     * 1つのIDでユーザーがログインしている状況で、他のデバイスで同じユーザーがログインした時に呼び出されるコールバック
     *
     * @param newDeviceId          新しく接続したユーザーのdeviceId値
     * @param outPayloadForKickUserクライアントに伝達するペイロード
     * @return boolean typeがtrue：新たに接続したユーザーがlogin success、現在のユーザーはforceLogout. false：新たに接続したユーザーがlogin failを返す。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendすることがある。/
     */
    @Override
    public boolean onLoginByOtherDevice(final String newDeviceId,
                                        Payload outPayloadForKickUser) throws SuspendExecution {
        return true;
    }

    /**
     * MultiTypeユーザーを使用する時、あるtypeのユーザーがloginした状況で他のtypeのユーザーがloginする時に呼び出されるコールバック
     *
     * @param userType  新たにログインを試行するユーザーのタイプ
     * @param outPayloadクライアントに伝達するペイロード
     * @return boolean typeがtrue：新たにログイン成功。false：新たにログイン失敗を返す。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onLoginByOtherUserType(final String userType,
                                          Payload outPayload) throws SuspendExecution {
        return true;
    }

    /**
     * Loginした状況で同じdevice同じidでloginする時に呼び出されるコールバック
     *
     * @param outPayloadクライアントに伝達するペイロード
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {        
    }

    /**
     * サーバーにユーザーオブジェクトが生きている状態でloginが呼び出された場合に呼び出されるコールバック
     *
     * @param payload    クライアントから送信した任意のペイロード
     * @param sessionPayload onPreLoginから渡されたペイロード
     * @param outPayload クライアントに渡す任意のペイロード
     * @return boolean typeがtrue：ReLogin成功、false：ReLogin失敗を返す。
     * @throws SuspendExecution：このメソッドはファイバーがsuspendすることがある。
     */
    @Override
    public boolean onReLogin(final Payload payload,
                             final Payload sessionPayload,
                             Payload outPayload) throws SuspendExecution {
    }

    /**
     * クライアントとの接続が切断された時に呼び出されるコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onDisconnect() throws SuspendExecution {        
    }

    /**
     * ユーザーに送信するパケットがある時に呼び出されるコールバック
     *
     * @param packet処理するパケット
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, packet);
    }

    /**
     * nodeがpauseした時にnodeが管理しているユーザーに呼び出されるコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPause() throws SuspendExecution {        
    }

    /**
     * nodeがresumeした時、nodeが管理しているユーザーに呼び出されるコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onResume() throws SuspendExecution {        
    }

    /**
     * ユーザーがlogout時に呼び出されるコールバック
     *
     * @param payloadクライアントから伝達されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onLogout(final Payload payload, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * ユーザーがログアウト可能かを確認するコールバック。
     *
     * @return boolean typeで返す：falseの場合、ログアウトできず、その後定期的に再確認を行う。：trueの場合、ユーザーはログアウトされる。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean canLogout() throws SuspendExecution {        
    }

    /**
     * クライアントからルームメイキングをリクエストした場合に発生するコールバック
     *
     * @param roomTypeマッチングするルームのタイプ
     * @param payloadクライアントから渡されたペイロード
     * @returnマッチングしたルームの情報を返す、nullを返す時はクライアントのリクエストオプションに基づいて新しいルームが作成されたり、失敗処理されることがある。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public MatchRoomResult onMatchRoom(final String roomType,
                                       final Payload payload) throws SuspendExecution {
        return null;
    }

    /**
     * クライアントからユーザーマッチメイキングをリクエストした場合に呼び出されるコールバック
     *
     * @param roomType  マッチングするルームのタイプ
     * @param payloadクライアントから伝達されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @return true：ユーザーマッチメイキングリクエスト成功、 false：ユーザーマッチメイキングリクエスト失敗。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onMatchUser(final String roomType,
                               final Payload payload,
                               Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * クライアントからuserMatchがキャンセルされた時に呼び出されるコールバック
     *
     * @param reasonキャンセルされた理由(TIMEOUT/CANCEL)
     * @return boolean typeで返す。true：ユーザーマッチメイキングキャンセル成功、 false：ユーザーマッチメイキングキャンセル失敗。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
        return false;
    }

    /**
     * タイマーハンドラが登録されて呼び出されるコールバック
     */
    @Override
    public void onRegisterTimerHandler() {        
    }

    /**     
	 * ユーザーが他のノードに移動する時、任意のデータを渡すために呼び出されるコールバック
     *
     * @param transferPack必要なデータを設定するオブジェクト
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onTransferOut(final TransferPack transferPack) throws SuspendExecution {        
    }

    /**
     * ユーザーが他のnodeに移動した後、user defineを復旧するために呼び出されるコールバック
     * <p>
     * この時点ではユーザーが完全に復旧される前のため、他の場所へrequestの呼び出しが制限される。
     *
     * @param transferPack移動前に渡されたオブジェクト。そのオブジェクトから設定値を取得する。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onTransferIn(final TransferPack transferPack) throws SuspendExecution {        
    }

    /**
     * ユーザーのnode間transferが完了した後に呼び出されるコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPostTransferIn() throws SuspendExecution {        
    }

    /**
     * クライアントからsnapshotをリクエストした時に呼び出されるコールバック
     * <p>
     * 主に接続が切断され、サーバーの状態が変わる確率がある場合に呼び出してクライアントとサーバーの状態情報を同期するのに使われる。
     *
     * @param payloadクライアントから伝達されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onSnapshot(final Payload payload, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * クライアントからuserに他のチャンネルに移動リクエストを送った場合に呼び出されるコールバック
     *
     * @param destinationChannelId移動しようとする対象channel ID
     * @param payload          クライアントから渡されたペイロード
     * @param errorPayload         channel移動失敗時にサーバーからクライアントへ渡すペイロード。成功の場合は渡されない。
     * @return false：チャンネル移動リクエスト失敗、 true：チャンネル移動リクエスト成功。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onCheckMoveOutChannel(final String destinationChannelId,
                                         final Payload payload,
                                         Payload errorPayload) throws SuspendExecution {
        return false;
    }

    /**
     * クライアントからチャンネル移動リクエストが入った場合、onCheckMoveOutChannel()がtrueを返した場合に呼び出されるコールバック
     * <p>
     * チャンネル移動時、移動するチャンネルに情報を渡すために呼び出しサーバーのmoveChannel()を呼び出した時、onCheckMoveOutChannel()が呼び出されず、すぐに呼び出される。
     *
     * @param destinationChannelId移動するchannel ID
     * @param outPayload       移動するchannelに渡すペイロード
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onMoveOutChannel(final String destinationChannelId,
                                 Payload outPayload) throws SuspendExecution {
    }

    /**
     * onMoveOutChannel()呼び出し後に呼び出されるコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPostMoveOutChannel() throws SuspendExecution {
    }

    /**
     * 移動したチャンネルに入場する時に発生するコールバック
     *
     * @param sourceChannelId移動する前のchannel ID
     * @param payload     クライアントから渡されるペイロード
     * @param outPayload  クライアントに渡すペイロード
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onMoveInChannel(final String sourceChannelId,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * onMoveInChannel()を実行した後に呼び出されるコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPostMoveInChannel() throws SuspendExecution {
    }

    /**
     * User Transfeが可能かを確認。
     *
     * @return boolean typeで返す。true：Transfer可能。false：Transfer不可。Transferが不可の場合、NonStopPatchが終了するまでは継続的に呼び出される。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean canTransfer() throws SuspendExecution;
}
```



### 3-2. GameRoom

2人以上のゲームユーザーはGameRoomを介して同期されたメッセージの流れを作ることができます。つまり、ゲームユーザーのリクエストはGameRoom内で全ての順序が保障されます。もちろん1人のゲームユーザーのためのGameRoom作成もコンテンツに応じて意味を持つこともあります。GameRoomをどのように使用するかは、どこまでもエンジンユーザーの自由です。このようにGameRoomは、GameUserと同様に基本クラス(BaseRoom)をもとにさまざまな独自のコールバックメソッドを再定義することができ、独自のプロトコルを処理することもできます。以下のサンプルコードはSampleGameUserのためのSampleGameRoomクラスです。

```
public class SampleGameRoom extends BaseRoom<SampleGameUser> {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameRoomTest.getDescriptor(), _GameRoomTest.class);
    }

    /**
     * Roomが初期化される時に発生するコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onInit() throws SuspendExecution {        
    }

    /**
     * Roomが削除される時に発生するコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onDestroy() throws SuspendExecution {        
    }

    /**
     * Roomに{@link Packet}が渡される時に呼び出されるコールバック
     *
     * @param userそのパケットを処理するユーザーオブジェクト。そのパケットを処理するユーザーがいない場合はnull。
     * @param packet処理するパケット
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onDispatch(SampleGameUser user, final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, user, packet);
    }

    /**
     * クライアントからルームの作成をリクエストした時に発生するコールバック
     *
     * @param user      ルームの作成をリクエストしたユーザーオブジェクト
     * @param inPayloadクライアントから渡されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @return true：ルーム作成成功、false：ルーム作成失敗。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onCreateRoom(SampleGameUser user,
                                final Payload inPayload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * クライアントからRoom入場リクエストを送った時に発生するコールバック
     *
     * @param user       Room入場をリクエストするユーザーオブジェクト
     * @param inPayloadクライアントから渡されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @return true：Room入場成功。false：Room入場失敗。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onJoinRoom(SampleGameUser user,
                              final Payload inPayload,
                              Payload outPayload) throws SuspendExecution {
    }

    /**
     * クライアントからRoom退出リクエストを送った時に発生するコールバック
     *
     * @param user      退出リクエストを送ったユーザーオブジェクト
     * @param inPayloadクライアントから渡されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @return boolean typeを返す。true：Room退出成功。false：Room退出失敗。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onLeaveRoom(SampleGameUser user,
                               final Payload inPayload,
                               Payload outPayload) throws SuspendExecution {
    }

    /**
     * onLeaveRoom呼び出し後、戻り値がtrueの場合に呼び出されるコールバック
     *
     * @param user onLeaveRoom処理が終わったユーザーオブジェクト
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPostLeaveRoom(SampleGameUser user) throws SuspendExecution {
    }

    /**
     * ユーザーがRoomにいる状態でReJoinする場合呼び出されるコールバック
     *
     * @param user       ReJoinを行ったユーザーオブジェクト
     * @param outPayloadクライアントに伝達するペイロード
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onRejoinRoom(SampleGameUser user, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * タイマーハンドラが登録されて呼び出されるコールバック
     */
    @Override
    public void onRegisterTimerHandler() {
    }

    /**
     * room transfer発生時、roomのdefineをserializeするために呼び出されるコールバック
     *
     * @param transferPack移動前に渡された情報パッケージ。そのオブジェクトから情報を取得する。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * room transfer発生時、roomのdefineを復元するために呼び出されるコールバック
     *
     * @param userList そのRoomにいるユーザーオブジェクトリスト。
     * @param transferPack 復元するRoom情報パッケージ
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onTransferIn(List<SampleGameUser> userList,
                             final TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * nodeがpauseした後、各roomオブジェクトで呼び出されるコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onPause() throws SuspendExecution {
    }

    /**
     * nodeがresumeした後、各roomで呼び出されるコールバック
     *
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * クライアントからpartyMatchをリクエストした時に呼び出されるコールバック
     *
     * @param roomType  ルームのタイプ
     * @param user      パーティマッチングをリクエストしたルームリーダー
     * @param payloadクライアントから伝達されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @return boolean typeで返す。true：user matchingリクエスト成功。false：user matchingリクエスト失敗。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean onMatchParty(final String roomType,
                                final SampleGameUser user,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * room transferを現在行うことができるかをチェック。
     * <p>
     * 最初のコールバックを呼び出した後、NonNetworkNode状態がReadyになるまで継続的にチェック。
     *
     * @return boolean typeで返す。true：room transfer可能。false：room transfer不可。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    @Override
    public boolean canTransfer() throws SuspendExecution {        
    }
}
```



### 3-3. ブートストラップ(Bootstrap)

作成したGameNodeとGameUser、そしてGameRoomはBootstrap段階で以下のように登録できます。

```
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGateway()
            .connection(SampleGameConnection.class)
            .session(SampleGameSession.class)
            .node(SampleGameGatewayNode.class)
            .enableWhiteModules();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)
            .room("SampleGameRoomType", SampleGameRoom.class);

        bootstrap.run();
    }
}
```



## 4. ユーザー転送(UserTransfer)

ゲームノードにはゲームユーザーオブジェクトが作成されます。このゲームユーザーオブジェクトは、ゲームノード間でいつでも転送されることがあります。例えば1番ゲームノードのユーザーオブジェクトが2番ゲームノードに作成されたルームに入る過程でゲームノード間でユーザーオブジェクトの転送が行われます。この技術はGameAnvilの最も根幹となり、ユーザーオブジェクトを正しくシリアル化/逆シリアル化した時に意図したとおりに動作します。これらのシリアル化/逆シリアル化対象は、GameAnvilユーザーが直接実装できます。つまり、そのゲームユーザーオブジェクトのどの値を送信するのかを選択可能です。

ユーザーの転送が発生するタイミングは、大きく3つに分けられます。

- ①マッチメイキングを含むルーム作成や、ルーム参加ロジックによりユーザーオブジェクトは他のゲームノードに転送されることがあります。この時、実装に応じて他のチャンネルのルームに入ることもできます。
- ②他のチャンネルに移動する時に発生します。1つのゲームノードは1つのチャンネルに属すことができます。そのため、チャンネルを移動することは、ゲームノード間の移動を意味します。
- ①の場合のようにマッチメイキングなどを利用して他のチャンネルのルームに入る場合は、エンジン内部で自動的に処理します。
- クライアントは明示的にチャンネル移動をリクエストすることもできます。
- ③任意のゲームノードに対して無停止メンテナンス(NonStopPatch)を行うと、そのノードのユーザーオブジェクトは、他の有効なゲームノードに分散されて転送されます。これは、運営側でGameAnvil Consoleを使用して命令を下した場合です。



### 4-1. ユーザー転送の実装

実際のユーザーの転送は、GameAnvilが内部的に処理します。このとき、クライアントは自分のゲームユーザーオブジェクトがサーバー間で転送されたかを認知できません。つまり、他のゲームノードのルームに入ってもクライアントはただ1つのGameAnvilサーバー郡で任意のルームに入っただけです。

ただし、他のゲームノードにユーザーオブジェクトを転送する時、どのようなデータを一緒に転送するかはユーザーが指定できます。指定だけしておけば、そのデータはユーザーオブジェクトの一部として一緒にシリアル化されます。転送する対象ゲームノードでは逆シリアル化を気にする必要はなく、簡単にそのオブジェクトにアクセスして使用できます。

次は、このようなユーザー転送に関するコールバックメソッドです。まず「一緒に転送するデータの指定」です。これを行うためにBaseUser抽象クラスを継承したゲームユーザークラスに以下のコールバックを実装します。方法は簡単です。以下の例のように転送するデータをユーザーが任意のkey値を利用してkey-valueのペアで引数として渡されたtransferPackに入れるだけです。

もしここで転送するデータを指定しなければ、対象ゲームノードに転送されたユーザーオブジェクトのデータは全てデフォルト値で初期化されるため注意が必要です。

```
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("TotalMoney", myTotalMoney);
    transferPack.put("Message", myMessage);
}
```

次にユーザーの転送が完了した後、対象ゲームノードで処理するコールバックメソッドを見ていきます。以下のように転送前に指定したkeyを利用して任意のオブジェクトにアクセスできます。このような方法で転送完了したユーザーオブジェクトのデータを元の状態にします。

```
@Override
public void onTransferIn(TransferPack transferPack) throws SuspendExecution {
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setTotalMoney(transferPack.getToLong("TotalMoney"));
    setTransferMessage(transferPack.getToString("Message"));
}
```

上の2つのメソッドは、GameAnvilが呼び出します。ユーザーはただ実装するだけで構いません。



### 4-2. 転送可能なユーザーのタイマー

ユーザーが転送される時、ユーザーに登録しておいたタイマーで一緒に転送できます。転送可能なタイマーを使用するには、別途、以下のコールバックメソッドで任意のkeyを利用して事前に登録しておく必要があります。Timerハンドラを次のように任意のkeyでマッピングしておきます。

```
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferUserTimerHandler1", transferUserTimerHandler1());
     registerTimerHandler("transferUserTimerHandler2", (timerObject, object) -> {
         GameUser user = (GameUser) object;
         logger.warn("GameUser::transferUserTimerHandler2() : userId({})", user.getId());
     });
 }
```

登録が完了したタイマーオブジェクトは、いつでもそのkeyを利用して以下のようにゲームユーザーの実装部で使用できます。単に登録しただけのタイマーは効果がないため、実際に使用するには必ず以下のようにユーザーオブジェクトに追加する必要があります。

```
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

このようにユーザーオブジェクトに追加しておいたタイマーは、別途転送の処理を行わなくても全て自動的に転送されます。つまり、対象ゲームノードでそのユーザーオブジェクトに同じタイマーの追加プロセスを踏む必要がありません。



## 5. ID

GameAnvilは、複数の種類のIDを使用します。その中の一部はサーバーが独自に発行し、他の一部はユーザーがGameAnvilConfigに直接設定します。接続に必要なアカウント情報などは、クライアントから入力された情報をサーバーへ伝達します。次はGameAnvilで使用する代表的なIDの説明です。

| 名前  | 説明                                                     | データ型 | 範囲  |
| --------- | ------------------------------------------------------------ | ------ | --------- |
| ServiceId | - 設定した各サービスを区別するためのID。1つのサービスIDは複数のノードで構成することができる。 | int    | 0<id< 100 |
| HostId    | - ホストの固有ID。1つのホストに複数のGameAnvilプロセスを実行する場合にはGameAnvilConfigに別のvmIdを設定 | long   | -         |
| NodeId    | - ノードの固有ID。HostId + ServiceId + 内部Counter値で構成 | long   | -         |
| AccountId | - クライアントが接続する時に入力。コネクション(Connection)1つにつき1つのアカウントIDがマッピング | string | -         |
| UserId    | - ゲームユーザーオブジェクトの固有ID。ゲームユーザーオブジェクトが作成される時、サーバーが発行。同じユーザーが再接続する場合でも新しいゲームユーザーオブジェクトが作成されると新しいIDを発行 | int    | -         |
| RoomId    | -ルームの固有ID。ルームオブジェクトが作成される時、サーバーが発行     | int    | -         |
| SubId     | - 1つのアカウント(accountId)内で固有の補助ID。セッション(Session)1つにつきAccountId + SubIdがマッピング。コネクション(Connection)単位で複数のセッション(Session)を使用する時、1つの接続内でそれぞれのセッションユーザーを区別するID。クライアントが接続する時に入力 | int    | 0 < id    |

### 5-1.IDサポートAPI

前述したIDに関する一部の機能を以下の表のようにエンジンユーザーに提供します。そのIDを取得または確認するには、以下のAPIを使用する必要があります。

- ServiceIdクラス

| メソッド                            | 説明                                |
| ------------------------------------- | --------------------------------------- |
| boolean isValid(int serviceId)        | 有効なserviceIdであることを確認           |
| int findServiceId(String serviceName) | ServiceNameに該当するServiceIdを取得 |
| String findServiceName(int serviceId) | ServiceIdに該当するServiceNameを取得 |

- HostIdクラス

| メソッド                   | 説明                 |
| ---------------------------- | ------------------------ |
| boolean isValid(long hostId) | 有効なhostIdであることを確認 |
| long get()                   | プロセスのhostIdを取得 |

- NodeIdクラス

| メソッド                    | 名前                      |
| ----------------------------- | ----------------------------- |
| boolean isValid(long nodeId)  | 有効なnodeIdであることを確認    |
| long getHostId(long nodeId)   | nodeIdからhostIdを取得   |
| int getServiceId(long nodeId) | nodeIdからserviceIdを取得 |
| int getNodeNum(long nodeId)   | nodeIdからnodeNumを取得  |

- UserIdクラス

| メソッド                  | 説明               |
| --------------------------- | ---------------------- |
| boolean isValid(int userId) | 有効なuserIdであることを確認 |

- RoomIdクラス

| メソッド                  | 説明               |
| --------------------------- | ---------------------- |
| boolean isValid(int roomId) | 有効なroomIdであることを確認 |

- SubIdクラス

| メソッド                 | 説明              |
| -------------------------- | --------------------- |
| boolean isValid(int subId) | 有効なsubIdであることを確認 |



## 6. チャンネル(Channel)

チャンネルは単一サーバー郡を論理的に分割することができる方法の1つです。GameAnvilは、1つ以上のゲームノードが含まれている場合にチャンネルを設定できます。基本的にGameAnvilConfigを利用してゲームノードに以下の例のようにチャンネルを設定します。この例で4個のゲームノードに対して、それぞれch1、ch1、ch2、ch2を設定します。

```
"game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "Game",
      "channelIDs": [
        "ch1",
        "ch1",
        "ch2",
        "ch2"
      ],
    }
```

この時、同じチャンネルのゲームノードは、互いにチャンネル関連情報を共有します。例えば、チャンネルに新しいユーザーが出入りした時、GameAnvilエンジンがあらかじめ実装したコールバックを呼び出します。チャンネル間でユーザーとルーム情報の同期を行うために次のようにGameUserInfo、そしてGameRoomInfoを使用します。前述したGameNodeのBootstrapの内容に追加でSampleChannelUserInfoとSampleChannelRoomInfoが登録されていることを確認できます。

```
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class, SampleChannelUserInfo.class)
            .room("SampleGameRoomType", SampleGameRoom.class, SampleChannelRoomInfo.class);

        bootstrap.run();
    }
}
```

それでは、このようなChannelUserInfoとChannelRoomInfoを実装する方法について説明します。まずチャンネルユーザー情報は、GameAnvilが提供するChannelUserInfoインターフェイスとSerializableインターフェイスを実装します。同じチャンネルに属すGameNodeの間で転送される必要があるためSerializableは必須です。残りはエンジンユーザーが任意の情報を入れてください。

```
public class SampleChannelUserInfo implements ChannelUserInfo, Serializable {

    private int userId = 0;
    private String userType = "";
    private String accountId = "";

    public SampleChannelUserInfo(String userType, int userId, String accountId) {
        this.userType = userType;
        this.userId = userId;
        this.accountId = accountId;
    }

    public SampleChannelUserInfo() {}

    public void setUserType(String userType) {
        this.userType = userType;
    }

    public void setUserId(int userId) {
        this.userId = userId;
    }

    public void setAccountId(String accountId) {
        this.accountId = accountId;
    }

    /**
     * {@link KryoSerializer}でシリアル化する。
     *
     * @return ByteBufferで逆シリアル化されたオブジェクトを返す。
     */
    @Override
    public ByteBuffer serialize() {
        return KryoSerializer.write(this);
    }

    /**
     * {@link KryoSerializer}で逆シリアル化する。
     *
     * @param inputStreamシリアル化されたストリーム
     * @return ChannelUserInfoで逆シリアル化されたオブジェクトを返す。
     */
    @Override
    public ChannelUserInfo deserialize(InputStream inputStream) {
        return (GameChannelUserInfo) KryoSerializer.read(inputStream);
    }

    /**
     * 変更されるChannel User情報のUser Id。
     *
     * @return int typeでUserIdを返す。
     */
    @Override
    public int getUserId() {
        return userId;
    }

    /**
     * 変更されるChannel User情報のAccount Id。
     *
     * @return String typeでAccountIdを返す。
     */
    @Override
    public String getAccountId() {
        return accountId;
    }

    @Override
    public int compareTo(GameChannelUserInfo o) {
        if (o.userId != this.userId)
            return -1;
        else
            return 0;
    }

}
```

チャンネルルーム情報も同じ方法で実装します。GameAnvilが提供するRoomInfoインターフェイスとSerializableを実装します。インターフェイス名がChannelRoomInfoではなく、RoomInfoである点に注意してください。このクラスはチャンネルユーザー情報と同様に、エンジンユーザーがチャンネル間の同期に使用する情報をもとに実装してください。

```
public class SampleChannelRoomInfo implements Serializable, RoomInfo {

    public static final int MAX_ENTRY_USER = 4;
    private int roomId = 0;

    private int userCnt;
    private int gameState;
    private long createTime;

    public SampleChannelRoomInfo() {
    }

    //... コンテンツで必要な情報でクラスを実装

    /**
     * Room情報のRoom Id。
     *
     * @return int typeでRoomIdを返す。
     */
    @Override
    public int getRoomId() {
        return roomId;
    }

    /**
     * Room情報を{@link KryoSerializer}利用してserialize処理を行う。
     *
     * @return ByteBuffer typeでシリアル化された内容を返す。
     */
    @Override
    public ByteBuffer serialize() {
        return KryoSerializer.write(this);
    }

    /**
     * 渡された情報を{@link KryoSerializer}を利用して逆シリアル化処理を行う。
     *
     * @param inputStream逆シリアル化するデータ
     * @return RoomInfoでRoom情報を返す。
     */
    @Override
    public RoomInfo deserialize(InputStream inputStream) {
        return (RoomInfo) KryoSerializer.read(inputStream);
    }

    /**
     * Room情報をコピーする。
     *
     * @return RoomInfoでコピーされたRoom情報を返す。
     * @throws CloneNotSupportedExceptionのコピーができない場合。
     */
    @Override
    public RoomInfo copy() throws CloneNotSupportedException {
        GameRoomInfo roomInfo = (GameRoomInfo) super.clone();
        roomInfo.testListInfo = new ArrayList<>(testListInfo);

        return roomInfo;
    }
}
```

前述したチャンネルのユーザー、ルーム情報は、実際のユーザーやルームの更新が必要な時に同じチャンネル内のGameNode間に伝わります。この時、関連GameNodeは以下の2つのコールバックメソッドをそれぞれのチャンネルユーザー情報とチャンネルルーム情報更新について呼び出します。

```
    /**
     * 同じチャンネルの他のnodeにユーザーの変化がある時に呼び出されるコールバック
     * <p>
     * updateChannelUser()を呼び出すと発生。
     *
     * @param type            Channel情報変更タイプ(更新/削除)
     * @param channelUserInfo変更されるユーザー情報
     * @param userId      変更対象のUser Id
     * @param accountId   変更対象のAccount Id
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    public void onChannelUserUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {        
    }
```

パラメータに渡されたチャンネルユーザー、ルーム情報をもとにエンジンユーザーが任意の方法で実装してください。

```
    /**
     * 同じチャンネルの他nodeにroomの状態の変化がある時に呼び出されるコールバック
     * <p>
     * updateChannelRoomInfo()呼び出し時に発生。
     *
     * @param type            Channel情報変更タイプ(更新/削除)
     * @param channelRoomInfo変更されるRoom情報
     * @param roomId      変更対象のRoom Id
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    public void onChannelRoomUpdate(ChannelUpdateType type, RoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {        
    }
```

最後に、クライアントはサーバーにチャンネル情報をリクエストできます。このとき、以下のコールバックメソッドが呼び出されます。これもエンジンユーザーが任意の方法で実装してください。

```
    /**
     * クライアントからChannel情報をリクエストする時に呼び出されるコールバック(Base.GetChannelInfoReq)
     *
     * @param outPayloadクライアントに伝達されるChannel情報
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {        
    }
```



## 7. マッチンググループ(MatchingGroup)

マッチンググループもチャンネルと同様に単一サーバー郡を論理的に分割できる方法の1つです。ただし、マッチンググループはチャンネルと異なり、あらかじめ設定しません。またチャンネルはGameNodeを論理的に分割するための方法であるが、マッチンググループはマッチメイキングを論理的に分割するための方法です。前述したユーザーマッチメイキングコールバックメソッドをもう一度確認します。この例で設定したマッチンググループは、「Newbie」です。この「Newbie」という値はクライアントからサーバーへpayloadを介して渡すこともでき、エンジンユーザーがサーバー上に該当ユーザーに対してあらかじめ保存しておいた値かもしれません。以下のようにハードコーディングした値を使うこともできますが、実際のゲームサーバーコードには望ましくありません。とにかく「Newbie」というマッチンググループに対して1つのマッチメイカーが作成され、その後にすべての「Newbie」リクエストはこのマッチメイカーに蓄積されて一緒に処理されることが保障されます。

```
    /**
     * クライアントからuserMatchをリクエストした時に呼び出されるコールバック
     *
     * @param roomType  マッチングするroomのタイプ
     * @param payloadクライアントから伝達されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @return boolean typeで返す。true：user matchingリクエスト成功。false：user matchingリクエスト失敗。
     * @throws SuspendExecutionこのメソッドはファイバーがsuspendする場合がある。
     */
    public boolean onMatchUser(final String roomType,
                               final Payload payload, Payload outPayload) throws SuspendExecution {

        String matchingGroup = "Newbie";
        SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId());
        sampleUserMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, term, payload);
    }
```

マッチメイキング関連のすべてのプロトコルにはマッチンググループフィールドが存在します。そのため、クライアントとサーバーの間にあらかじめマッチンググループを定義しておいて使用するのが最も理想的です。例えば、「初心者」、「中級者」、「上級者」のように実力をもとにマッチンググループを定義することもでき、「韓国」、「日本」、「米国」のように国別のマッチンググループを定義することもできます。これはどこまでもエンジンユーザーが自由に使用できる部分です。



## 8. MatchNode & MatchMaker

エンジンユーザーはマッチングロジックを実装することでマッチメイキングを適用できます。マッチメーカーは、ロジックに基づいてユーザーを同じルームに入場させます。GameAnvilは、RoomMatchMakerとUserMatchMakerの2つのマッチメーカーを提供します。このマッチメイカーは、MatchNodeで独自に動作します。このMatchNodeは、マッチメイキングを実行する用途以外の追加のコンテンツを実装できません。そのためユーザーは、MatchNodeではなく、マッチメイカーにのみ集中してください。

### Note

*マッチメイカーは、マッチンググループ単位で作成されます。つまり、同じマッチンググループ同士でマッチメイキングを実行します。*



### 6-1. UserMatchMaker

ユーザーマッチメイキングは、ゲームユーザーのマッチングリクエストをキューに格納します。特定時間ごとにこのリクエストキューの内容を比較、分析してユーザーが望む基準で任意のユーザーを1つのルームに入場させます。ここでエンジンユーザーはリクエストキューの内容をどのように比較し分析して、どのような基準でユーザーをマッチングするかというロジックにのみ集中してください。参考に最も代表的なユーザーマッチメイキングゲームは「*リーブオブレジェンド*」があります。

このようなユーザーマッチメイキングの最も基本は、マッチングリクエスト自体です。マッチングリクエストを以下のようにエンジンで提供するUserMatchInfo抽象クラスを継承して実装します。リクエスタを区別することができるゲームユーザーのIDを提供できるように、getId()メソッドは必ず実装する必要があります。以下の例は、マッチングリクエスト間の比較を行うためにComparableインターフェイスを追加で実装しています。

```
public class SampleUserMatchInfo extends UserMatchInfo implements Serializable, Comparable<SampleUserMatchInfo> {

    private int userId;
    private int rating;

    public SampleUserMatchInfo(int userId, int rating) {
        this.userId = id;
        this.rating = rating;
    }

    public int getRating() {
        return rating;
    }

    /**
     * パーティマッチングリクエストの場合はRoomId、ユーザーマッチングリクエストの場合はUserIdを返す。
     *
     * @return int typeでパーティッチングリクエストの場合はRoomId、ユーザーマッチングリクエストは場合UserIdを返す。
     */
    @Override
    public int getId() {
        return userId;
    }

    /**
     * パーティマッチングリクエストの場合はパーティのサイズ(人数)、ユーザーマッチングリクエストの場合は0を返す。
     *
     * @return int typeでパーティマッチングリクエストの場合はパーティのサイズ(人数)、ユーザーマッチングリクエストの場合は0を返す。
     */
    @Override
    public int getPartySize() {
        return 0;
    }

    // もしSampleUserMatchInfoオブジェクト間で比較が必要な場合は、Comparableインターフェイスを実装します。
    @Override
    public int compareTo(SampleUserMatchInfo o) {
        if (this.rating < o.getRating())
            return -1;
        else if (this.rating > o.getRating())
            return 1;

        if (this.id < o.getId())
            return -1;
        else if (this.id > o.getId())
            return 1;

        return 0;
}
```



次にユーザーマッチメイカーを作ります。ユーザーマッチメイカーは、エンジンで提供するUserMatchMakerの抽象クラスを継承して実装します。特にmatch()メソッドは、実際のマッチングを実行するために呼び出されるコールバックのため注意してください。refill()メソッドは、すでに完了したマッチメイキングに対して増員リクエストが入った時に呼び出されるコールバックです。例えば、4人がマッチメイキングした状態で1人がゲームを終了してしまった時、1人を増員するために使用できます。以下のサンプルコードは、このようなUserMatchMakerをどのように実装できるかを示しています。

```
public class SampleUserMatchMaker extends UserMatchMaker<SampleUserMatchInfo> {

    private static final Logger logger = getLogger(SampleUserMatchMaker.class);

    public SampleUserMatchMaker() {
        super(2, 5000);
    }

    private Multiset<SampleUserMatchInfo> ratingSet = TreeMultiset.create();
    private final int matchMultiple = 1; // match定員の何倍まで人員が集まった後にratingごとにソートしてマッチングするか
    private int currentMultiple = matchMultiple;
    private long lastMatchTime = System.currentTimeMillis();
    private int totalMatchMakings = 0;

    @Override
    public void match() {
        List<SampleUserMatchInfo> matchRequests = getMatchRequests(matchSize * currentMultiple);

        // 最小数(minAmount)まで格納されなかった
        if (matchRequests == null) {
            if (System.currentTimeMillis() - lastMatchTime >= 10000)
                currentMultiple = Math.max(--currentMultiple, 1);

            return;
        }

        // matchingが成立していない項目は、ratingSetにそのまま残ることがあるが、別途保管する必要はない。
        // この項目は、次のgetMatchRequests()で再び伝達される。
        ratingSet.clear();
        ratingSet.addAll(matchRequests);

        if (ratingSet.size() >= matchSize) {

            // ratingSetの順序どおりにmatchingAmount*matchSize分の項目を消費API
            int matchingAmount = matchSingles(ratingSet);

            if (matchingAmount > 0) {
                totalMatchMakings += matchingAmount;
                logger.info("{} match(s) made (total: {}) - {}", matchingAmount, totalMatchMakings, this.getMatchingGroup());

                lastMatchTime = System.currentTimeMillis();
                currentMultiple = matchMultiple;
            }
        }
    }

    @Override
    public boolean refill(SampleUserMatchInfo matchReq) {
        try {
            List<SampleUserMatchInfo> refillRequests = getRefillRequests();

            if (refillRequests.isEmpty()) {
                return false;
            }

            for (SampleUserMatchInfo refillInfo : refillRequests) {
                // 100点以上差がなければリフィル
                if (Math.abs(matchReq.getRating() - refillInfo.getRating()) < 100) {
                    if (refillRoom(matchReq, refillInfo)) { // このマッチングリクエストをリフィルが必要なルームにマッチング
                        return true;
                    }
                }
            }
        } catch (Exception e) {
            logger.error("SampleUserMatchMaker::refill()", e);
        }

        return false;
    }
}
```



こうして作成されたUserMatchMakerとUserMatchInfoは、Bootstrapの段階で登録する必要があります。1つ注意する点は、マッチメイカーは以下のようにゲームノード構成の延長線上にあるという点です。つまり、GameNodeで使用するGameUserと、GameRoomを構成することに加えて、これらをマッチングするためにどのようなUserMatchMakerを使用するか構成することです。

```
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)
            .room("SampleUserMatchType", SampleGameRoom.class)

            // 登録
            .userMatchMaker("MyUserMatchRoomType", SampleUserMatchMaker.class, SampleUserMatchInfo.class);

        bootstrap.run();
    }
}
```



これでクライアントはサーバーにユーザーマッチメイキングをリクエストできます。このリクエストは、GameUserに渡された後、エンジンにより以下のコールバックメソッドを呼び出します。このコールバックメソッドは、エンジンユーザーにGameAnvilが提供するユーザーマッチメイカーだけでなく、第3のソリューションを使用する機会を提供します。以下の例は、エンジンで提供するユーザーマッチメイカーを使用するためにmatchUser() APIを呼び出す様子です。エンジンユーザーは、独自にマッチメイカーを作って使用したり、他のライブラリなどを使用することもできます。この場合はそれに合わせて以下のコールバックメソッドを実装します。特に、このサンプルコードで注意すべき部分は、マッチンググループです。マッチンググループは、マッチメイキングを論理的に分割できる概念で、この文書の途中で詳しく説明します。例で使用した「Newbie」マッチンググループは、同じ「Newbie」マッチンググループ同士で同じマッチングキューを共有します。

```
    /**
     * クライアントからuserMatchをリクエストした時に呼び出されるコールバック
     *
     * @param roomType  マッチングするルームのタイプ
     * @param payloadクライアントから伝達されたペイロード
     * @param outPayloadクライアントに伝達するペイロード
     * @return boolean typeで返す。true：user matchingリクエスト成功。false：user matchingリクエスト失敗。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
    public boolean onMatchUser(final String roomType,
                               final Payload payload, Payload outPayload) throws SuspendExecution {

        String matchingGroup = "Newbie";
        SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId());
        sampleUserMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, term, payload);
    }
```



最後に、クライアントは、いつでもリクエストしたマッチメイキングをキャンセルできます。この時、エンジンはキャンセル処理が成功すると、以下のようなGameUserのコールバックメソッドを呼び出します。エンジンユーザーは、このコールバックでキャンセルタイミングに処理したい部分を実装してください。

```
    /**
     * クライアントからuserMatchがキャンセルされた時に呼び出されるコールバック
     *
     * @param reasonキャンセルされた理由(TIMEOUT/CANCEL)
     * @return boolean typeで返す。true：user matchingキャンセル成功。false：user matchingキャンセル失敗。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
    }
```



### 6-2. RoomMatchMaker

ルームマッチメイキングは、ゲームユーザーを適切なルームに自動入場させます。ルームマッチメイキングをリクエストしたゲームユーザーをどのルームに入場させるかはエンジンユーザーが自由に実装できます。最もユーザー数が多いルームに入場させたり、最も空いているルームに入場させることもできます。また平均スコアが最も高いルームに入場させることもできます。エンジンユーザーは、このようなマッチングロジックにのみ集中できます。最も代表的なルームマッチメイキングゲームは「*ハンゲームポーカー*」や「*カートライダー*」などがあります。

- 注意> RoomMatchMakerとUserMatchMakerは、独立して運営されます。つまり、同じマッチンググループにユーザーマッチングとルームマッチングをそれぞれリクエストしても、この2つのリクエストが一緒にマッチングすることはありません。

このようなルームマッチメイキングの最も基本は、マッチングリクエスト自体です。マッチングリクエストを以下のようにエンジンで提供するRoomMatchInfoインターフェイスを実装します。リクエスタを区別することができるゲームユーザーのIDを提供できるようにgetId()メソッドは必ず実装する必要があります。

```
public class SampleRoomMatchInfo implements Serializable, RoomMatchInfo {

    private int roomId;
    private int userCurrentCount = 0;
    private int maxUserCount = 4;

    public SampleRoomMatchInfo() {
    }

    public void setRoomId(int roomId) {
        this.roomId = roomId;
    }

    public int getUserCurrentCount() {
        return userCurrentCount;
    }

    public void setUserCurrentCount(int userCurrentCount) {
        this.userCurrentCount = userCurrentCount;
    }

    public int getMaxUserCount() {
        return maxUserCount;
    }

    /**
     * RoomIdを取得する。
     * <p>
     * 状況に応じて他の意味で使われる。
     * <p>
     * RoomMatchInfoがマッチング可能なRoomまたはマッチングしたRoomの情報を意味する場合、そのRoomのRoomIdを意味する。
     * <p>
     * RoomMatchInfoがマッチング条件を意味する場合、コンテンツで指定したRoomIdを返す。
     * <p>
     * コンテンツで任意の方法で使用できる。
     * <p>
     * ex)マッチングで除外するRoomId。
     *
     * @return int typeでRoomIdを返す。
     */
    int getRoomId();
    @Override
    public int getRoomId() {
        return roomId;
    }
}
```



ルームマッチメイカーを作ります。ルームマッチメイカーは、エンジンで提供するRoomMatchMaker抽象クラスを継承して実装します。特にmatch()メソッドは、実際のマッチングを実行するために呼び出されるコールバックのため注意してください。以下のサンプルコードは、RoomMatchMakerをどのように実装できるかを示しています。UserMatchMakerはUserMatchInfoがComparableインターフェイスを実装している反面、RoomMatchMakerはComparatorを提供しています。これは単にエンジンユーザーが柔軟に好きな方法で実装できることを示すためです。

```
public class SampleGameRoomMatchMaker extends RoomMatchMaker<SampleGameRoomMatchRoomInfo> {

    /**
     * MatchRoomリクエストを処理する。
     * <p>
     * 登録されたRoomの中からマッチング条件に合ったRoomを探してマッチング結果を返す。
     * <p>
     * {@link BaseUser#onMatchRoom(String, Payload)}で{@link BaseUser#matchRoom(String, String, RoomMatchInfo)}を呼び出すと、呼び出されてマッチングを処理できる。
     * <p>
     * マッチング条件RoomMatchInfoがtermsに伝達される。
     * <p>
     * {@link #getRooms()}を使用して取得したRoomMatchInfoのリストがマッチング可能なRoomの情報です。
     * <p>
     * この情報とtermsを比較してマッチング可能なRoomを探す。
     * <p>
     * マッチング可能なRoomが見つかったらそのRoomのRoomMatchInfoを返し、見つからなかった場合はnullを返す。
     *
     * @param termsマッチング条件
     * @param args追加で渡されたデータ
     * @return {@link RoomMatchInfo}でマッチングしたRoomの情報を返す。ない場合はnullを返す。
     */
    @Override
    public SampleGameRoomMatchRoomInfo match(SampleGameRoomMatchRoomInfo terms, Object... args) {

        int bypassRoomId = terms.getRoomId();
        List<SampleGameRoomMatchRoomInfo> rooms = getRooms();

        for (SampleGameRoomMatchRoomInfo info : rooms) {
            // moveRoomオプションがtrueの場合は参加中のルームは除外する
            if (info.getRoomId() == bypassRoomId)
                continue;

            if (info.getMaxUserCount() != terms.getMaxUserCount())
                continue;

            if (info.getMaxUserCount() == info.getUserCurrentCount())
                continue;

            return info;
        }

        return null;
    }

    @Override
    public Comparator<SampleGameRoomMatchRoomInfo> getComparator() {
        return new Comparator<SampleGameRoomMatchRoomInfo>() {
            @Override
            public int compare(SampleGameRoomMatchRoomInfo o1, SampleGameRoomMatchRoomInfo o2) {
                return o1.getMaxUserCount() - o2.getMaxUserCount();
            }
        };
    }
}
```



このように作成したRoomMatchMakerとRoomMatchInfoは、Bootstrapの段階で登録する必要があります。注意する点は、ユーザーマッチメイカーと同じです。ルームマッチメイカーも以下のようにゲームノード構成の延長線上にあります。つまり、GameNodeで使用するGameUserとGameRoomを構成することに加えて、これらをマッチングするためにどのようなRoomMatchMakerを使用するか構成することです。また、ユーザーマッチメイカーとルームマッチメイカーを一緒に使用できます。以下のサンプル例コードは、これを示すために2つのマッチメイカーを一緒に登録しています。ただし、2つのマッチメイカーが使用するRoomとMatchInfoクラスが分かれていることに注意してください。

```
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)

            .room("SampleGameUserMatchRoomType", SampleUserMatchRoom.class)
            .userMatchMaker("SampleGameUserMatchRoomType", SampleUserMatchMaker.class, SampleUserMatchInfo.class)

            // 登録
            .room"SampleGameRoomMatchRoomType", SampleRoomMatchRoom.class)
            .roomMatchMaker("SampleGameRoomMatchRoomType", SampleRoomMatchMaker.class, SampleRoomMatchInfo.class);

        bootstrap.run();
    }
}
```



これでクライアントはサーバーにルームマッチメイキングをリクエストできます。このリクエストはGameUserに渡された後、エンジンにより以下のコールバックメソッドを呼び出します。このコールバックメソッドは、エンジンユーザーにGameAnvilが提供するルームマッチメイカーだけでなく、第3のソリューションを使用する機会を提供します。以下の例はエンジンで提供するルームマッチメイカーを使用するためにmatchRoom() APIを呼び出す様子です。エンジンユーザーは、独自にマッチメイカーを作って使用したり、他のライブラリなどを使用することもできます。この場合はそれに合わせて以下のコールバックメソッドを実装します。特に、このサンプルコードで注意すべき部分は、マッチンググループです。マッチンググループは、マッチメイキングを論理的に分割できる概念で、この文書の途中で詳しく説明します。例で使用した「Newbie」マッチンググループは、同じ「Newbie」マッチンググループ同士で同じマッチングキューを共有します。

```
    /**
     * クライアントでroomMatchをリクエストした場合に発生するコールバック
     *
     * @param roomTypeマッチングするルームのタイプ
     * @param payloadクライアントから渡されたペイロード
     * @return {@link MatchRoomResult}でmatchingしたroomの情報を返す。nullを返す時はクライアントリクエストオプションに基づいて新しいRoomが作成されるか、リクエスト失敗処理される。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
    public MatchRoomResult onMatchRoom(final String roomType,
                                       final Payload payload) throws SuspendExecution {

        String matchingGroup = "Newbie";

        SampleRoomMatchInfo sampleRoomMatchInfo = new SampleRoomMatchInfo(getChannelId());
        sampleRoomMatchInfo.setMatchingGroup(matchingGroup);
        sampleRoomMatchInfo.setRoomMode(RoomMode.NORMAL);

        return matchRoom(matchingGroup, roomType, terms);
    }
```



## 7. Support Node

SupportNodeは補助的な機能を実行するためのノードです。ゲームユーザーやルームオブジェクトに関係なく任意の機能を実装できます。例えばログを収集して転送したり、ビリングサーバーとの通信を担うなどの役割で使用できます。また、エンジンのRESTful機能を使用して簡単なWebサーバーの代わりとして使用することも可能です。

このようなSupportNodeは、基本的にBaseSupportNodeの抽象クラスを継承して実装します。ノード共通コールバックメソッドに追加でRESTful処理のためのonDispatch()コールバックが提供されます。以下のコードで1～3に該当するコメントの下のコードがそのための処理の流れを順に示しています。

```
public class SampleSupportNode extends BaseSupportNode {

    // 1.RESTディスパッチャー作成
    private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher<>();

    // 2.SampleSupportNodeで処理したいURLとハンドラをマッピング
    static {
        // pathとmethod(GET, POST, ...)の組み合わせで登録。
        restMsgHandler.registerMsg("/auth", RestObject.GET, _RestAuthReq.class);
        restMsgHandler.registerMsg("/echo", RestObject.GET, _RestEchoReq.class);
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onPrepare() throws SuspendExecution {
    }

    @Override
    public void onReady() throws SuspendExecution {
    }

    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {
    }

    public abstract boolean onDispatch(RestObject restObject) throws SuspendExecution;


    /**
     * rest callが渡される時に呼び出される。
     *
     * @param restObject渡された{@link RestObject}
     * @return boolean typeで返す。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
    @Override
    public boolean onDispatch(RestObject restObject) throws SuspendExecution {
        // 3.SampleSupportNodeのRESTリクエスト処理    
        if (!restMsgHandler.isRegisteredMessage(restObject))
            return false;

        restMsgHandler.dispatch(this, restObject);
        return true;
    }

    @Override
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {}

    @Override
    public void onResume(Payload outPayload) throws SuspendExecution {}

    @Override
    public void onShutdown() throws SuspendExecution {}
}
```



こうして作成したSupportNodeは、Bootstrapの段階で次のように登録できます。

```
public class Main {

    public static void main(String[] args) {

        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        // 登録
        bootstrap.setSupport("SampleWebSupport")
            .node(SampleSupportNode.class);

        bootstrap.run();
    }
}
```



## 8. ルーム転送(RoomTranfer)

ルームの転送は、途中で説明したユーザー転送ととても似た機能です。ユーザーより大きな概念のルームが転送される差があるだけです。ゲームノードには1人以上のゲームユーザーが参加したルームオブジェクトが作成されることがあります。このルームオブジェクトは、ユーザーオブジェクトと同じように、ゲームノード間で転送されることがあります。例えば、1番ゲームノードのルームオブジェクトが2番ゲームノードに転送されるということです。このようにルームが転送される時に当然、ルーム内に存在するユーザーも転送が行われます。そのおかげでルームが転送される前後のゲームの流れは連続して行われることがあります。つまり、そのルームオブジェクトが転送される過程はとても早く進行するため、ルーム内のユーザーは認知できない状態でゲームを継続できます。この技術はGameAnvilの無停止メンテナンスパッチ(NonStopPatch)の根幹であり、ルームオブジェクトを正しくシリアル化/逆シリアル化した時に意図したとおりに動作します。これらのシリアル化/逆シリアル化対象は、GameAnvilユーザーが直接実装できます。つまり、そのゲームユーザーオブジェクトのどの値を送信するのかを選択可能です。

このようなルーム転送を発生させるのは、無停止メンテナンスパッチ(NonStopPatch)コマンドだけです。このコマンドは一般的にGameAnvil Consoleを利用してゲーム運営者が実行します。



### 8-1. ルーム転送の実装

実際のルーム転送は、GameAnvilが内部的に処理します。この時、クライアントは自分のゲームユーザーとともに自分が属すルームオブジェクトがサーバー間で転送されたかを認知できない可能性が高いです。特別な問題が発生しない限り、全体の流れがとても速く進行するため、転送前のゲームの流れを転送後に引き継げます。

この時、他のゲームノードにルームオブジェクトを転送する時、どのようなデータを一緒に転送するかはユーザーが指定できます。指定だけしておけば、このデータはルームオブジェクトの一部として一緒にシリアル化されます。転送する対象ゲームノードでは逆シリアル化を気にする必要はなく、簡単にそのオブジェクトにアクセスして使用できます。

次は、このようなユーザー転送に関するコールバックメソッドです。まず「一緒に転送するデータの指定」です。これを行うためにBaseRoom抽象クラスを継承したゲームルームクラスに以下のコールバックを実装します。方法は簡単です。以下の例のように転送するデータをユーザーが任意のkey値を利用してkey-valueのペアで引数として渡されたtransferPackに入れるだけです。

もしここで転送するデータを指定しなければ、対象ゲームノードに転送されたルームオブジェクトのデータは全てデフォルト値で初期化されるため注意が必要です。

**参考**> *先に説明したように、ルームが転送されるときは当然、ルーム内のユーザーが一緒に転送されます。ユーザーの転送に関しては途中で説明したため、ここでは説明しません。*

```
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("RoomInfo", myRoomInfo);
}
```

では、ルームの転送が完了した後、対象ゲームノードで処理するコールバックメソッドを見てみましょう。以下のように転送前に指定したkeyを利用した目的のオブジェクトにアクセスできます。このような方法で転送完了したルームオブジェクトのデータを元の状態にします。特に、転送されたルーム内のユーザーオブジェクトリストを引数として渡されていることを確認できます。この部分を除けば全体的な流れはユーザー転送とよく似ています。

```
@Override
public void onTransferIn(List<GameUser> userList, TransferPack transferPack) throws SuspendExecution {
    this.users.clear();
    for (GameUser user : userList)
        this.users.put(user.getUserId(), user);

    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setRoomInfo(transferPack.getToLong("RoomInfo"));
}
```



### 8-2. 転送可能なルームタイマー

ルームが転送されるとき、ルームに登録しておいたタイマーも一緒に転送できます。転送可能なタイマーを使用するには、別途、以下のコールバックメソッドでkeyを利用してあらかじめ登録しておく必要があります。Timerハンドラを次のようにkeyにマッピングしておきます。これはユーザー転送とまったく同じです。

```
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferRoomTimerHandler1", transferRoomTimerHandler1());
     registerTimerHandler("transferRoomTimerHandler2", (timerObject, object) -> {
         GameRoom room = (GameRoom) object;
         logger.warn("GameRoom::transferRoomTimerHandler2() : roomId({})", room.getId());
     });
 }
```

登録が完了したタイマーオブジェクトは、いつでもそのkeyを利用して以下のようにゲームルームの実装部で使用できます。単に登録しただけのタイマーは効果がないため、実際に使用するには必ず以下のようにユーザーオブジェクトに追加する必要があります。

```
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

このようにルームオブジェクトに追加しておいたタイマーは、別途転送の処理を行わなくても全て自動的に転送されます。つまり、対象ゲームノードでそのルームオブジェクトに同じタイマーの追加プロセスを踏む必要がありません。



## 9. 無停止メンテナンス(NonStopPatch)

一般的にメンテナンスを行う時は、ゲームサービス全体にメンテナンスをかけてユーザーのゲームプレイを遮断します。その後、必要なパッチを進行します。しかし、GameAnvilを使用すれば、サーバーバイナリの互換性がない場合でなければサービスを止めずにパッチを進行できます。これを無停止メンテナンスパッチと呼び、基本的にGameAnvil Console運営ツールを利用して進行できます。

無停止メンテナンスパッチのポイントは、先に説明したユーザー転送とルーム転送技術です。この2つの機能をもとにパッチを進行するゲームサーバーのユーザーとルームを全て有効な他のゲームサーバーに転送した後、ゲームサーバーが空の状態になった時、パッチを進行します。このような無停止メンテナンスパッチは、サービスの過程で進行する部分のため、必ずサービス前にテストを行い、問題がないことを確認するプロセスを踏んでください。また、パッチするサーバーバイナリと以前のサーバーバイナリ間の互換性が保たれているかチェックする必要があります。

無停止メンテナンスパッチを利用するには、ユーザーはGameNodeクラスに関連コールバックメソッドを再定義する必要があります。

```
    /**
     * 無停止メンテナンスの開始時、そのnodeが出発地nodeの場合に呼び出されるコールバック
     *
     * @return boolean typeで成否を返す。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
@Override
public boolean onNonStopPatchSrcStart() throws SuspendExecution {
    return true;
}

    /**
     * 無停止メンテナンスが終了した時、そのnodeが出発地nodeの場合に呼び出されるコールバック
     *
     * @return boolean typeで成否を返す。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
@Override
public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}

    /**
     * 無停止メンテナンスが始まり、そのnodeが出発地nodeの場合に無停止メンテナンスを終了するかを確認するために呼び出されるコールバック
     *
     * @return boolean typeで成否を返す。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
@Override
public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}

    /**
     * 無停止メンテナンスの開始時に、そのnodeが到着地nodeの場合に呼び出されるコールバック
     *
     * @return boolean typeで成否を返す。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
@Override
public boolean onNonStopPatchDstStart() throws SuspendExecution {
    return true;
}

    /**
     * 無停止メンテナンスが終了した時、そのnodeが到着地nodeの場合に呼び出されるコールバック
     *
     * @return boolean typeで成否を返す。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
@Override
public boolean onNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}

    /**
     * 無停止メンテナンスが始まり、そのnodeが到着地nodeの場合に無停止メンテナンスを終了するかを確認するために呼び出されるコールバック
     *
     * @return boolean typeで成否を返す。
     * @throws SuspendExecutionこのメソッドは、ファイバーがsuspendすることがある。
     */
@Override
public boolean canNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
```



### 3-1. 無停止メンテナンスパッチガイド

無停止メンテナンスパッチは、GameAnvilの複数の要素が正しく噛み合ったときに良い結果を得られます。そのため、必ず以下のガイド文書を読み、すべての内容を理解して使用するようにしてください。

#### [無停止メンテナンスパッチガイド](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-nonstop-patch)

## 10. プロトコル定義とコンパイル

GameAnvilは[Google Protocol Buffers](https://developers.google.com/protocol-buffers)を使用してプロトコルを定義してビルドします。以下の例は、プロトコルを定義する方法と、ビルドする方法を説明します。まずSampleGame.protoファイルをテキストエディタで作成し、プロトコルを定義します。プロトコルバッファの詳しい文法は、[公式Protocol Buffersガイド](https://developers.google.com/protocol-buffers/docs/overview)を参照してください。

```
package [パッケージ名];

// 参照する必要がある他のprotoファイルがあればimport
import [protoファイル名]

enum SampleUserTypeEnum {
    USER_TYPE_NONE = 0;
    USER_TYPE_ONE = 1;
}

message SampleUserData {
    int32 data = 1;
}

enum SampleRoomTypeEnum {
    ROOM_TYPE_NONE = 0;
    ROOM_TYPE_ONE = 1;
}

message SampleRoomData {
    int32 data = 1;
}

message SampleUserMessage {
    SampleUserTypeEnum sampleUserTypeEnum = 1;
    int32 id = 2;
    int64 money = 3;
    string nickname = 4;
    double score = 5;
    repeated SampleUserData sampleUserData = 6; // list
}

message SampleRoomMessage {
    SampleRoomTypeEnum sampleRoomTypeEnum = 1;
    int32 id = 2;
    int64 potMoney = 3;
    string roomName = 4;
    repeated SampleRoomData sampleRoomData = 5; // list
}
```



そして、次のようなコマンドラインで作成したプロトコルスクリプトをコンパイルできます。この時、build.batのようなバッチファイルを作成しておくと便利です。もしGameAnvilテンプレートを利用してプロジェクトを構成している場合は、プロジェクト内のprotoフォルダからプロトコルスクリプトとバッチファイルを参照できます。コンパイルが完了すると、プロトコルの処理に必要なすべてのJavaクラスが自動的に作成されます。

```
protoc  ./MyGame.proto --java_out=../java
```



## 11. Configurations

GameAnvilはGameAnvilConfig.jsonファイルを利用してサーバー構成を変更できます。構成ファイルは合計7個の項目に分類され、commonを除くとそれぞれ1種類のノードを設定します。この時、設定する値の一部はBootstrapの段階で使用する値と一致する必要があります。例えば、先に私たちは以下のようにGameNodeを構成しました。ここに使われているServiceNameの「SampleGameService」は、必ずGameAnvilConfig.jsonに設定されている必要があります。設定されていない場合は、そのサービスを見つけられず、サーバー側でエラーが発生します。

```
bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
```



### common

- 必須の共通設定

| 名前                        | 説明                                                     | デフォルト値 |
| ------------------------------- | ------------------------------------------------------------ | ------- |
| vmId                            | 1つのmachineで複数のJVMを起動する場合に使用する固有識別ID ( 0 <= vmId < 100) | 0       |
| ip                              | ノードごとに共通で使用するIP。(マシンのIPを指定)値がない場合はprivate ipに自動指定される。 |         |
| meetEndPoints                   | 対象ノードのcommon IPとcommunicatePort登録該当サーバーendpointを含む可能リストに複数登録可能 |         |
| keepAliveTime                   | communicateNode接続持続時間(ms)                          | 60000   |
| nodeTimeout                     | ノード状態チェックタイムアウト(ms) 0の場合はチェックしない。             | 180000  |
| msgExpiredTime                  | メッセージ有効期限                                         | 30000   |
| defaultReqTimeout               | ノード間メッセージルーティングタイムアウト(ms)0の場合はチェックしない。        | 30000   |
| defaultAsyncAwaitTimeout        | ブロッキング呼び出し時のタイムアウト(ms)                                  | 30000   |
| ipcPort                         | 他のipc nodeと通信する時に使用されるポート                  | 11100   |
| publisherPort                   | publish socketのためのポート                               | 11101   |
| debugMode                       | デバッグ時に、各種timeoutが発生しないようにするオプション。リアルでは必ずfalseにする必要がある。 | false   |
| nodeInfoUpdateTime              | JVMにあるノード情報を他のJVMに伝える周期        | 500(ms) |
| nodeInfoUpdateBusyTime          | ノード情報がアップデートされる時、この時間より時間がかかった場合、そのノードはBUSY状態と判断する。 | 510(ms) |
| nodeInfoManagerRefreshTime      | 渡されたノード情報を利用してNodeInfoManagerを更新する周期。 | 100(ms) |
| loopInfoSamplingCount           | Message Loopの状態をチェックする時、Samplingするloop回数     | 100(回) |
| enableLoopInfoMonitor           | NodeInfoPageでloop情報を更新するかどうか                        | true    |
| maxUserCount                    | IdleNode判断時、idleと判断する最大User数                  | 2500    |
| addLocWeight                    | UserLoc割り当て時、情報を補正するために使用する倍数。例)ゲームノードが8個の場合、1人が割り当てられるが、8人のユーザーが割り当てられたと任意で指定。 | 1       |
| allocatorType                   | 使用するbytebuf type 1 (POOLED_DIRECT_BUFFER) 2 (POOLED_HEAP_BUFFER) 3 (UNPOOLED_DIRECT_BUFFER) 4 (UNPOOLED_HEAP_BUFFER) | 4       |
| httpClientOption                | Gameflexで使用するhttpClientOption                       |         |
| httpClientOption-keepAlive      | httpClientOption接続を持続するかどうか                             | true    |
| httpClientOption-connTimeout    | httpClientOption connectionタイムアウト                     | 5000    |
| httpClientOption-requestTimeout | httpClientOption  requestタイムアウト                       | 10000   |
| httpClientOption-maxConnPerHost | httpClientOption hostあたりの最大connection数                | 10000   |
| httpClientOption-maxConn        | httpClientOption connectionの最大数                       | 12000   |
| moduleURL                       | config、dynamicModule情報を読み込むURL情報該当オプションが設定されている場合はURLを使用してmodule情報を取得する。 | ""      |
| configPath                      | configをFileで読み込む時、configのJsonファイルを読み込むパス。moduleURLが設定されている場合は無視 | /config |

- 使用例

```
"common": {
    "ip": "127.0.0.1",
    "meetEndPoints": ["127.0.0.1:16000"],
    "keepAliveTime": 10000,
    "nodeTimeout": 180000,
    "msgExpiredTime:":30000,
    "defaultReqTimeout": 30000,
    "defaultAsyncAwaitTimeout": 30000,
    "communicatePort": 16000,
    "publisherPort" : 13300,
    "debugMode": false,
    "allocatorType": 1,
    "nodeInfoUpdateTime": 10,
    "nodeInfoUpdateBusyTime": 12,
    "nodeInfoManagerRefreshTime": 4,
    "loopInfoSamplingCount": 5,
    "maxLoopTime" : 10,
    "enableLoopInfoMonitor" : true,
    "maxUserCount" : 2500,
    "addLocWeight": 16,
    "httpClientOption": {
        "keepAlive": true,
        "connTimeout": 5000,
        "requestTimeout": 10000,
        "maxConnPerHost" : 10000,
        "maxConn": 12000
    }
},
```



### location

- 設定値がないか、clusterSizeが0の場合、LocationNodeを作成しません。

| 名前    | 説明                                                     | デフォルト値 |
| ----------- | ------------------------------------------------------------ | ------ |
| clusterSize | 構成されているサーバーの数(VM)                                       |        |
| replicaSize | レプリケーショングループのサイズmaster + slaveの数                      |        |
| shardFactor | shardingのための引数 <br />-全shardの数= clusterSize x replicaSize x shardFactor <br />-1つのマシン(VM)で動作するshardの数= replicaSize x shardFactor <br />-固有のshardの合計数(masterシャードの数) = clusterSize x shardFactor |        |

- 使用例

```
location: {
    "clusterSize": 1,
    "replicaSize": 3,
    "shardFactor": 3
},
```



### match

- 設定値がない場合やnodeCntが0の場合は、MatchNodeを作成しません。

| 名前 | 説明        | デフォルト値 |
| ------- | --------------- | ------ |
| nodeCnt | match nodeの数 |        |

- 使用例

```
match: {
    "nodeCnt": 3
},
```



### gateway

- 設定値がない場合やnodeCntが0の場合は、GatewayNodeを作成しません。

| 名前                                 | 説明                                                     | デフォルト値 |
| ---------------------------------------- | ------------------------------------------------------------ | ------ |
| nodeCnt                                  | ノードの数                                                   |        |
| ip                                       | クライアントと接続されているIP設定値がない場合は、private ipに自動設定 |        |
| dns                                      | クライアントと接続されているドメインアドレス                        |        |
| maintenance                              | サーバー起動時、メンテナンスするかどうか                                      | false  |
| checkWhiteListBeforeOnAuth               | onAuthenticateコールバックを呼び出すまえにWhiteListをチェックするかどうか             | true   |
| connectGroup                             | TCP_SOCKET WEB_SOCKET                                        |        |
| connectGroup - port                      | クライアントと接続されているポート                               |        |
| connectGroup - idleClientTimeout         | データ送受信がない状態以降のタイムアウト。0の場合は使用していない | 4000   |
| connectGroup - secure                    | セキュリティ設定の設定値がない場合は使用していない。                  |        |
| connectGroup - secure - useSelf          | keyCertChainPath, privateKeyPath設定値に関係なく、セキュリティ設定を使用するかどうか | false  |
| connectGroup - secure - keyCertChainPath | 証明書パス <br />-Dsecure設定パスで開始             |        |
| connectGroup - secure - privateKeyPath   | 秘密鍵パス <br />-Dsecure設定パスで開始             |        |
| duplicateLoginServices                   | 重複ログイン可能サービス指定                             |        |

- 使用例

```
"gateway": {
    "nodeCnt": 4,
    "ip": "127.0.0.1",
    "dns": "",
    "maintenance": false,
    "checkWhiteListBeforeOnAuth": true,
    "connectGroup": {
        "TCP_SOCKET": {
            "port": 11200,
            "idleClientTimeout": 0,
            "secure": {
                "useSelf": true,
                "keyCertChainPath": "gameflex.crt",
                "privateKeyPath": "privatekey.pem"
            }
        },
        "WEB_SOCKET": {
            "port": 11400,
            "idleClientTimeout": 0
        }
    },
    "duplicateLoginServices": {
        "RPSGame": ["GroupChatting"],
        "GroupChatting": ["SampleGameService"],
        "ForTestCase": []
    }
},
```



### game

- 設定値がない場合や、nodeCntが0の場合はGameNodeを作成しません。先に説明したように、エンジンのBootstrap段階で使用したServiceNameをここに設定する必要があります。

| 名前    | 説明                                                     | デフォルト値 |
| ----------- | ------------------------------------------------------------ | ------ |
| nodeCnt     | ノードの数                                                   |        |
| serviceId   | Service ID                                                   |        |
| serviceName | Service名                                             |        |
| channelIDs  | ノードごとに付与するチャンネルID。ユニークでなくてもいい。""文字列でチャンネルの区別なく重複使用も可能 |        |
| userTimeout | disconnect以降のユーザーオブジェクト削除タイムアウト                 | 10000  |

- 使用例

```
"game": [
    {
        "nodeCnt": 8,
        "serviceId": 0,
        "serviceName": "SampleGameService",
        "channelIDs": ["ch1","ch1","ch2","ch2","ch3","ch3","ch4","ch4"],
        "userTimeout": 3000
    },
    {
        "nodeCnt": 4,
        "serviceId": 1,
        "serviceName": "GroupChatting",
        "channelIDs": ["", "", "", ""],
        "userTimeout": 4000
    }
],
```



### support

- 設定値がない場合やnodeCntが0の場合やSupportNodeを作成しません。SupportNodeもGameNodeと同様に、Bootstrapの段階で使用したServiceNameが設定されている必要があります。

| 名前                         | 説明                                                     | デフォルト値 |
| -------------------------------- | ------------------------------------------------------------ | ------ |
| nodeCnt                          | ノードの数                                                   |        |
| serviceId                        | service ID                                                   |        |
| serviceName                      | service名                                             |        |
| restIp                           | rest ip設定値がない場合、private ipに自動設定        |        |
| restPort                         | rest port                                                    |        |
| restPermissionGroups             | ACL設定を許可する特定PathとIPを登録。空の場合、すべての接続を許可 |        |
| restSecure                       | 設定値がない場合は使用しない                            |        |
| restSecure - keyCertChainPath    | 証明書パス -Dsecure設定パスで開始                   |        |
| restSecure - privateKeyPath      | 秘密鍵パス -Dsecure設定パスで開始                   |        |
| restSecure - authorizationSecret | 認証トークン(JWT)を使用する場合のみ必要。暗号化シグネチャキー      |        |
| restSecure - authorizationPath   | 認証トークン(JWT)を使用する場合のみ必要。認証を行うパス   |        |
| restCorsAllowDomains             | CORS許可domain例-127.0.0.1:1234                           |        |

- 使用例

```
support: [
    {
        "nodeCnt": 2,
        "serviceId": 3,
        "serviceName": "PLACE",
        "restIp": "127.0.0.1",
        "restPort": 17260,
        "restPermissionGroups": {
            "/": []
        },
        "restSecure": {
            "keyCertChainPath": "",
            "privateKeyPath": "",
            "authorizationSecret": "authSecret",
            "authorizationPath": "/auth"
        },
        "restCorsAllowDomains":[
        ]
    },
    {
        "nodeCnt": 2,
        "serviceId": 4,
        "serviceName": "DB",
        "restIp": "127.0.0.1",
        "restPort": 17251,
        "restPermissionGroups": {
            "/": []
        }
    },
    {
        "nodeCnt": 2,
        "serviceId": 5,
        "serviceName": "SampleWebSupport",
        "restIp": "127.0.0.1",
        "restPort": 17250
    }
],
```



### management

- 設定値がない場合や、nodeCntが0の場合はManagementNodeを作成しません。

| 名前      | 説明                                                     | デフォルト値                      |
| ------------- | ------------------------------------------------------------ | ------------------------------- |
| nodeCnt       | ノードの数                                                   |                                 |
| restIp        | rest ip設定値がない場合、private ipに自動設定        |                                 |
| restPort      | rest port                                                    |                                 |
| db            | managementで使用するdb設定                          |                                 |
| db - user     | db接続ID                                                   |                                 |
| db - password | db接続パスワード                                         |                                 |
| db - url      | db接続url - h2db : jdbc:h2:mem:gameflex_admin;DB_CLOSE_DELAY'-1 - mysql : jdbc:mysql://localhost:3306/gameflex_admin?useSSL=false - mariadb : jdbc:mariadb://localhost:3306/gameflex_admin?useSSL=false |                                 |
| db - driver   | db接続driver- h2db : org.h2.Driver- mysql : com.mysql.jdbc.Driver- mariadb : org.hibernate.dialect.MySQLDialect | org.h2.Driver                   |
| db - dialect  | db接続dialect- h2db : org.hibernate.dialect.H2Dialect- mysql : org.hibernate.dialect.MySQLDialect- mariadb : org.hibernate.dialect.MySQLDialect | org.hibernate.dialect.H2Dialect |
| db - ddlAuto  | ddlオプション - update：自動的にTable作成 - none ：自動的にTableを作成しない。 | update                          |

- 使用例

```
management: {
    "nodeCnt": 2,
    "restIp": "127.0.0.1",
    "restPort": 25150,

    "db": {
        "user": "root",
        "password": "1234",
        "url": "jdbc:h2:mem:gameflex_admin;DB_CLOSE_DELAY'-1"
    }
}
```



## 12. 非同期サポートAPI

先に説明したファイバー上での非同期処理を行うためにGameAnvilはAsyncクラスを提供します。以下のようなimport分を使用してAsyncクラスを利用すると、一般的なブロッキング/ノンブロッキング呼び出しを全てファイバー化できます。

```
import com.nhn.gameanvil.async.Async;
```

*すべてのAPImp説明は、[GameAnvil API Reference](http://10.162.4.61:9090/gameanvil)より、JavaDocで作成された文書を確認できます。*

呼び出し用のAPIは、大きくcallとrunに分けられ、それぞれ戻り値がある場合とそうでない場合に使用します。その他のスレッドベースのfutureをファイバーベースで使用できるように切り替えます。それぞれの用途に応じた使用方法は文書の次の部分で詳しく扱います。

### **12-1.ブロッキング呼び出し処理**

一般的なブロッキングの呼び出しは、スレッドブロッキングを意味します。つまり、現在のコードが実行中のファイバーだけでなく、このファイバーをスケジューリングするスレッドまでブロッキングさせるという意味です。これは、すぐにノードが止まるということなので絶対にブロッキング呼び出しを直接的に使用してはいけません。GameAnvilは、このようなスレッドブロッキング呼び出しをファイバーブロッキングに切り替えるAsync APIを提供します。このAPIは、外部のexecutorを使用して該当のブロッキング呼び出しを処理した後、完了した後のコードの流れを再びファイバー化します。戻り値の有無に応じてrunBlocking()とcallBlocking()のいずれかを使用してください。また、基本概念で説明したように、ファイバーブロッキングAPIは、Suspendableなので、API呼び出しメソッドは必ずSuspendExecution例外シグネチャを明示する必要があります。

```
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() throws SuspendExecution {

    Async.runBlocking(executor, runnable); // スレッドブロッキング呼び出しをファイバーブロッキング呼び出しに変換

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() throws SuspendExecution {

    return Async.callBlocking(executor, callable);  // スレッドブロッキング呼び出しをファイバーブロッキング呼び出しに変換

}
```

### **12-2.Future処理**

Futureの待機は、スレッドブロッキングを誘発します。例えば、以下のようなコードは、呼び出しスレッドをブロッキングします。

```
Future<SomeObject> future = someAsyncJob();

SomeObject ret = future.get(); // スレッドブロッキングを誘発
```

GameAnvilは、このようなfutureの待機をスレッドブロッキングからファイバーブロッキングに変換するAPIを提供します。ただし、このAPIはJavaのCompletableFutureとGuavaのListenableFutureのみサポートします。幸い大部分のライブラリは、この2つのfutureをもとに非同期をサポートしているため、無理なく適用できるでしょう。以下のコードは、Async APIを利用してfutureの待機をファイバーブロッキングで処理する例です。

```
Future<SomeObject> future = someAsyncJob();

SomeObject ret = Async.awaitFuture(future); // このファイバーのみブロッキング
```

### **12-3.ブロッキング処理の委任**

ブロッキング呼び出し処理を説明しました。Async APIのrunBlocking()やcallBlocking()は、ブロッキング処理が完了した後に、再びそのファイバーの実行フローを継続する場合に使用します。反面、外部スレッドにブロッキングの呼び出しを委任した後、その結果を気にする必要がない場合は、実行フローを継続できるでしょう。その場合は、以下のAPIを使用してください。このAPIは、ブロッキング呼び出しの結果を待機しないため、Suspendableではないことに注意してください。

```
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() { // NOT suspendable

    Async.exec(executor, runnable); // 外部スレッドにブロッキング呼び出しを委任したため、このファイバーはブロッキングされない。.

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() {  // NOT suspendable

    return Async.exec(executor, callable);  // 外部スレッドにブロッキング呼び出しを委任したため、このファイバーはブロッキングされない。

}
```



## 13. 非同期Redisサポート

これまで、どのような種類のRedisクライアントを使用するかは、GameAnvilユーザーが選択していました。しかし、これによりRedis関連イシューの種類と複雑さが、ユーザーが選択したRedisクライアントの種類と使用方法の違いに比例して増加することがわかりました。私たちは、これを防止するために、GameAnvilの基本的なガイドラインにRedisクライアントに関する使用方法を含めることに決定しました。このガイドラインは、GameAnvilでサポートするRedisクライアントの種類と、その基本的な使用方法をAPI化して、統一させることを目標としてしています。もちろん、提供されるAPではなく、他の種類のRedisクライアントを選択して別途使用することも可能ですが、特別な理由がなければ避けてください。

### **[Note]**

*以降の内容で、GameAnvilで提供するLettuceクラスと製品名の「Lettuce」を区別するために、前者は可能な限り「Lettuceクラス」と表記し、一部は内容上必要に応じて「Lettuce」と表記する場合があります。これと区別するために、製品名は全て大文字で***LETTUCE***と表記します。この文書で説明するLETTUCEはGameAnvilでの使用方法にフォーカスを当てているため、それ以上の説明が必要な場合は[LETTUCE公式ページ](https://github.com/lettuce-io/lettuce-core)を参照してください。*

GameAnvilはRedisクライアントでLETTUCEを使用することを推奨します。GameAnvilで提供するRedisラップAPIもLETTUCEを使用します。ちなみにLETTUCEは非同期Redisクライアントで、大部分の非同期APIはCompletableFutureをもとにしています。これは、すぐにGameAnvilのAsync APIを利用してファイバーベースの非同期化に切り替えられることを意味します。

GameAnvilで提供するRedisラップAPIは、大きく3つのクラス(Lettuce、RedisCluster、RedisSingle)に分かれています。Lettuceは、最も一般的な形の使用方法を提供し、内部的にLETTUCEオブジェクトを管理しないstaticクラスです。そのため、LETTUCEを最も一般的な形で使用したい場合は、Lettuceクラスが最も適しています。RedisClusterとRedisSingleは、それぞれRedisクラスターとスタンドアローンに対応するためのクラスで、内部的にLETTUCEオブジェクトを管理します。

### **13-1.Lettuce**

Lettuceクラスは、ファイバー単位の処理を行うための根幹となるstatic APIを提供します。内部的にRedisに関するいかなる状態も保管しないため、別のオブジェクトを作る必要がなく、すぐに使用可能です。Lettuceライブラリにある程度慣れたら、Lettuceクラスを直接使用するのがいいでしょう。

```
import com.nhn.gameanvil.async.redis.Lettuce;
```

次の3つの注意事項の他には基本的なLettuceの使用方法をそのまま維持できます。

- ①connectはGameAnvilのLettuce.connect()またはLettuce.connectAsync()を使用する。コネクションは基本的にスレッドをブロッキングするため、これのファイバー化処理を含みます。
- ②shutdownまたはコネクションと同じ理由でLettuce.shutdown()を使用する必要があります。
- ③RedisFutureの待機は、必ずLettuce.awaitFuture()を使用してファイバーブロッキング化する必要があります。

Lettuceクラスを使用してRedisに接続するコードは以下のとおりです。

```
RedisURI clusterURI = RedisURI.Builder.redis(IP_ADDRESS, 7500).build();
clusterClient = RedisClusterClient.create(Arrays.asList(clusterURI));
clusterConnection = Lettuce.connect(RpsConfig.DB_THREAD_POOL, clusterClient);

if (clusterConnection.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### **13-2.RedisCluster**

```
import com.nhn.gameanvil.async.redis.RedisCluster;
```

Redis ClusterのAPIをラップします。基本的に先に説明したLettuceと使用方法は大きく違いません。しかし、このクラスはLettuce関連オブジェクト(e.g.RedisClusterClient、StatefulRedisClusterConnectionなど)を独自に管理します。Lettuceオブジェクトを直接管理するより、RedisClusterを利用して管理したい場合に使用するのを考慮してください。

注意事項はLettuceの場合と完全に同じです。以下はRedisClusterを利用してRedisに接続するコードです。

```
redisClient = RedisSingle.create("redis://IP_ADDRESS:6379");
redisAsyncCommands = Lettuce.connect(RpsConfig.DB_THREAD_POOL, redisClient).async();

if (redisClient.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### **13-3.RedisSingle**

```
import com.nhn.gameanvil.async.redis.RedisSingle;
```

RedisClusterと比較した時、対象Rediがスタンドアローンであるという違いしかありません。

注意事項はLettuceの場合と完全に同じです。以下はRedisSingleを利用してRedisに接続するコードです。

```
redisCluster = new RedisCluster<>(IP_ADDRESS, 7500);
redisCluster.connect(RpsConfig.DB_THREAD_POOL, StringCodec.UTF8);

if (redisCluster.isConnected()) {
    logger.warn("============= Connected to Redis using Lettuce =============");
}
```

### **13-4.RedisFutureをファイバーから使用する**

Lettuce、RedisCluster、そしてRedisSingleは、全てLettuceライブラリがサポートするRedisFutureをファイバー上で待機できるAPIを提供します。内部実装は全てエンジンで提供するAsync.awaitFuture()を同じように使用するため、混用しても構いません。以下の4つのコードは、全て同じコードです。GameAnvilのファイバー上でRedisFutureに対するget()は、必ずこの4つのいずれかの方法を使用する必要があります。

- Async.awaitFuture()

```
try {
    Async.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- Lettuce.awaitFuture()

```
try {
    Lettuce.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- RedisCluster.awaitFuture()

```
try {
    redisCluster.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- RedisSingle.awaitFuture()

```
try {
    redisSingle.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- **間違った使い方**：直接Futureの待機を行う場合、そのNode(Thread)がブロッキングされるため、絶対に以下のようなコードは使用してはいけません。

```
try {
    RedisFuture future = clusterAsyncCommands.mget("testKey", getUserId()));
    future.get(); // スレッドブロッキングを誘発
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

### **13-5.set/get**

最も基本となるsetとgetは、RedisCluserとRedisSingleで基本提供します。

- RedisClusterを利用したset/get例

```
String setResult = redisCluster.set(key, value);
String getResult = redisCluster.get(key);
```

- RedisSingleを利用したset/get例

```
String setResult = redisSingle.set(key, value);
String getResult = redisSingle.get(key);
```

- 直接LETTUCEのRedisAsyncCommandsオブジェクトを使用した例

```
RedisFuture<String> setFuture = redisAsyncCommands.set(key, value);
RedisFuture<String> getFuture = redisAsyncCommands.get(key);

String setResult = Async.awaitFuture(setFuture);
String getResult = Async.awaitFuture(getFuture);
```

### **13-6.本格的なLETTUCE非同期処理**

Redisが提供するさまざまなコマンドは、LETTUCEのCommandsオブジェクトを利用して使用できます。基本的にLETTUCEは、Sync方式のCommandsオブジェクトと、Async方式のCommandsオブジェクトを提供しますが、GameAnvilはAync方式を使用することを推奨します。基本的にAsyncCommandsはRedis Clusterの場合とStandAloneの場合について、それぞれ以下のとおりです。

- RedisAdvancedClusterAsyncCommands
- RedisAsyncCommands

以下の例は、AsyncCommandsオブジェクトを利用してmgetを実行する例です。LETTUCEの非同期処理は、基本的にRedisFutureを使用し、このRedisFutureはCompletableFutureです。CompletableFutureの詳しい内容は[Java公式レファレンス](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)で確認できます。ちなみに以下の例は、LETTUCEの非同期処理のごく一部の方法だけを示しているため、そのまま使用せず、開発中のコードに合わせて作成してください。完璧な非同期コードの制御を行うには、必ず[LETTUCE](https://github.com/lettuce-io/lettuce-core)と[CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)の内容を熟知する必要があります。

### **[Note]**

thenApply()とthenAccept()などは、任意の外部スレッドから呼び出されるため、Nodeから管理する内部リソースにアクセスしたり、リソースにLockを使用してはいけません。

- 例1> key1とkey2の値を非同期で取得

```
Lettuce.awaitFuture(asyncCommands.mget("key1", "key2"));
```

- 例2> 以降のコードの流れと関係ない場合、future chainで外部スレッドに処理を委任(つまり、mgetで値の取得が完了するまで待機する必要がない場合)

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

future.thenApplyAsync(r -> {
    Map<String, String> map = new HashMap<>();
    for (KeyValue<String, String> kv : r)
        map.put(kv.getKey(), kv.getValue());
    return map;
}).thenAccept(r -> {
    for (Entry<String, String> entry : r.entrySet())
        logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
});
```

- 例3> 以降のコードの流れと関係がある場合は、futureを待機した後、処理

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

CompletionStage<Map<String, String>> cs = future.thenApplyAsync(r -> {
    Map<String, String> map = new HashMap<>();
    for (KeyValue<String, String> kv : r)
        map.put(kv.getKey(), kv.getValue());
    return map;
});

// do something here

try {
    // ファイバー上でfutureを待機するためにLettuce.awaitFuture()を使用する必要がある点に気をつけてください。
    Map<String, String> map = Lettuce.awaitFuture(cs);

    for (Entry<String, String> entry : map.entrySet())
        logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin()", e);
}
```



## 14. 非同期HttpReqeust & HttpResponseの使用方法

Http処理に関する部分も、Redisと同様にGameAnvilで基本的なAPIとガイドラインを提供します。もちろん、他の種類のHttp使用方法も選択できますが、特別な理由がなければ避けてください。GameAnvilは非同期ベースのHttpを使用するために内部的に[AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)を使用します。次で説明するAPIとその使用範囲を超える場合は、私たちが提供するAPIより直接[AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)を使用することを推奨します。 LETTUCEと同様に、AsyncHttpClientも内部的にCompletableFutureを使用するため、futureの待機をAsync.awaitFuture()を利用してファイバー化すれば、残りは一般スレッド上での使用方法と完全に同じです。

```
Async.awaitFuture(future.get()); // ファイバー上でfutureを待機します。
```

GameAnvilで提供するHttp APIは、リクエストとレスポンスのためのHttpRequest、HttpResponseクラス、そして結果の一般的な処理を行うためのHttpResultTemplateクラスで構成されています。このクラスを利用すれば、簡単で直感的にHttpリクエストとレスポンスを処理することができ、その結果を好きな形で取得することもできます。また、すべてのコードは非同期のため、特別な処理が必要ありません。次は、これを使用したサンプルコードです。

- 例1> 基本的な使用方法は、内部的にファイバー単位のfuture処理を勝手にしてくれるため、最も直感的な方法です。特別な理由がなければ、これらの基本的な使用方法だけで十分です。

```
HttpRequest request = new HttpRequest(URL);
HttpResponse response = request.GET();
```

- 例2> futureベースの非同期方式は、HTTPリクエストとレスポンス待機の間に別の作業を行いたい場合に、以下のようにfutureを直接利用できます。

```
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

// Do something here

HttpResponse response = new HttpResponse(Async.awaitFuture(future, 10000, TimeUnit.MILLISECONDS));
```

- 例3> HTTPリクエストheader構成は、以下の例のように、AsyncHttpClientはさまざまなAPIを提供します。AsyncHttpClientの詳細な使用方法は、[公式ページ](https://github.com/AsyncHttpClient/async-http-client)を参照してください。

```
HttpRequest request = new HttpRequest(url);
request.getBuilder()
    .addHeader("if-none-check-node", "false")
    .setRequestTimeout(timeout);
    .addQueryParam("serviceId", serviceId);

HttpResponse httpResponse = request.GET();
```

- 例4> 以降のコードの流れと関係ない場合、future chainで外部スレッドに処理を委任(Lettuceの場合と同じ方法)

```
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

future.thenApplyAsync(r -> {
    try {
        HttpResponse response = new HttpResponse(r);
        String body = response.getContents(String.class);
        HttpResultTemplate<JsonObject> result = GameAnvilUtil.Gson().fromJson(body, new RestResponseParamType(JsonObject.class));
        if (!result.getHeader().getIsSuccessful()) {
            logger.warn("GET failed : resultCode {}, resultMessage {}",
                result.getHeader().getResultCode(),
                result.getHeader().getResultMessage());
        }
        return result.getContents();
    } catch (IOException e) {
        logger.error("Exception occurred: ", e);
        return null;
    }
}).thenAccept(r -> {
    if (r != null) {
        JsonElement element = r.get(ELEMENT_NAME);
        if (element != null) {
            logger.info("The response code: {}", element.getAsString());
        }
    }
});
```

- 例5> 以降のコードの流れと関係ある場合は、futureを待機してから処理

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

CompletionStage<JsonObject> cs = future.thenApplyAsync(r -> {
    try {
        HttpResponse response = new HttpResponse(r);
        String body = response.getContents(String.class);
        HttpResultTemplate<JsonObject> result = GameAnvilUtil.Gson().fromJson(body, new RestResponseParamType(JsonObject.class));
        if (!result.getHeader().getIsSuccessful()) {
            logger.warn("GET failed : resultCode {}, resultMessage {}",
                result.getHeader().getResultCode(),
                result.getHeader().getResultMessage());
        }
        return result.getContents();
    } catch (IOException e) {
        logger.error("GameUser::onLogin()", e);
        return null;
    }
});

// do something here

try {
    // ファイバー上でfutureを待機するためにAsync.awaitFuture()を使用する必要がある点に気をつけてください。
    JsonObject jsonObject = Async.awaitFuture(cs);
    if (jsonObject != null) {
        JsonElement element = jsonObject.get(ELEMENT_NAME);
        if (element != null) {
            logger.info("The response code: {}", element.getAsString());
        }
    }
} catch (TimeoutException e) {
    logger.error("Exception occurred: ", e);
}
```



## 15. RDBMS非同期処理

RDBMSへのクエリーは、一般的にブロッキングです。ブロッキングクエリーをGameAnvil上で処理する方法は、先に説明した他のAsync使用方法と大きく変わりません。RDBMSを使用していたSQLクエリーのコードは、同じ方法で実装できます。また、エンジンユーザーは、DBにアクセスするために自由にSQL MapperやORMなどを選択できます。



### Note

*DBのクエリーを実装する過程で最も重要ですがよく見落とす部分は、DBのCP(ConnectionPool)サイズと、これを非同期で処理するTP(ThreadPool)の数の設定と、これらの間の関係についての理解です。一般的に、これらの2つの数値は、処理するクエリーの量を考慮して同じ値に設定するか、TPをCPより少し多めに設定してください。ちなみに、GameAnvilを利用した大規模性能テストの結果、サーバープロセス1つあたり6000～8000人処理基準TPとCP250個設定が最も良い結果を示しました。これは、どこまでもクエリーの複雑さと頻度など、複合的な要素を考慮してできるだけ多くのテストを行って最適な値を見つけるのがいいでしょう。*



### 15-1.Async Query

まず、クエリーの非同期処理は、大きく2つに分けることができます。クエリーの結果が必要な場合と、そうでない場合です。この2つの場合は、クエリーの結果有無の違いがあるだけで、全てのクエリーの実行が完了するまでファイバーが待機するのは同じです。つまり、非同期でリクエストしたクエリーが完了した後、次のコードに進むため、エンジンユーザーは一般的なブロッキングコードを作成するように実装できます。



まず、クエリーの結果を取得しようとする場合は、次の例のようにAsyncクラスのcallBlocking APIを使用します。callBlockingは、ファイバー上で任意のブロッキング呼び出しを実行した後、結果を返します。

```
try {
    return Async.callBlocking("MyThreadPool", new Callable<List<T>>() {
        @Override
        public List<T> call() throws Exception {
            return myQueryCode();
        }
    });
} catch (TimeoutException e) {
    logger.error("TimeoutException occured: ", e);
}

logger.info("Query has finished.");
```

この時、非同期処理のためのスレッドプールは、Bootstrapの段階であらかじめ作成しておくことができます。

```
bootstrap.createExecutorService("MyThreadPool", 250);
```

またはエンジンユーザーが必要に応じて直接作成した外部スレッドプールを使うこともできます。

```
bootstrap.createExecutorService(myExecutorService, 250);
```



2つ目に、クエリーの結果が必要ない場合は、次の例のようにAsyncクラスのrunBlocking APIを使用します。runBlockingは、ファイバー上で任意のブロッキング呼び出しを実行します。

```
try {
    Async.runBlocking("MyThreadPool", new Runnable() {
        @Override
        public void run() {
            try {
                myQueryCode();
            } catch (Exception e) {
                logger.error("Exception occured during query code: ", e);
            }
        }
    });
} catch (TimeoutException e) {
    logger.error("TimeoutException occured: ", e);
}

logger.info("Query has finished.");
```

この場合も同様に、任意のスレッドプールをrunBlocking APIに引数として渡すことができます。
