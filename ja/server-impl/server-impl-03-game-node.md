## Game > GameAnvil > サーバー開発ガイド > ゲームノード実装

## Game Node

![GameNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gamenode_on_network.png)

GameNodeは実際のゲームオブジェクトが生成され、ゲームコンテンツを実装するノードです。クライアントはGatewayNodeで認証を完了した後、GameNodeにログインを完了して初めてこのようなゲームコンテンツを開始できます。

以下の画像で見られるように、クライアントは固有のAccountIdで認証を完了したコネクションを基盤に複数の論理セッションを生成できます。この時、セッションはクライアントとユーザーオブジェクトの間に作られます。下の図で赤色の点線矢印がこのようなセッションを表します。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

それぞれのセッションは該当コネクション内で固有の値で区分でき、私たちはこの値をSubIdと呼びます。つまり、AccountIdとSubIdの組み合わせでユーザーは望むだけセッションを生成できます。この時、同一サービスに対するセッションも同様に望むだけ生成できます。つまり、画像でAccountIdが1であるコネクションは"Game"サービスあるいは"Chat"サービスに対するセッションをいくらでも追加で生成できるのです。

このようなセッションが向かう場所はまさにユーザーオブジェクトです。GameNodeはこのようなユーザーオブジェクトと彼らのグループであるルームオブジェクトを管理します。今回の章はこのようなGameNodeとGameUserそしてGameRoomについて説明します。

## GameNode 実装

GameNodeはIGameNodeインターフェースを実装します。以下のサンプルコードはGameNodeで基本的に再定義できるコールバックメソッドを示しています。ノード共通コールバックに加え、チャンネル管理のためのコールバックが存在します。

```java
@GameAnvilGameNode(gameServiceName = "MyGame") // (1) "MyGame"というサービスのためのGameNodeとしてエンジンに登録
public class SampleGameNode implements IGameNode {
    private IGameNodeContext gameNodeContext;

    /**
     * ゲームノードコンテキストを伝達するために呼び出し
     * <p/>
     * オブジェクトが生成された後、一度呼び出される
     *
     * @param gameNodeContextゲームノードコンテキスト
     */
    @Override
    public void onCreate(IGameNodeContext gameNodeContext) {
        this.gameNodeContext = gameNodeContext;
    }

    /**
     * 同じチャンネルの他のノードでユーザー変化があった時に呼び出し
     * <p>
     * updateChannelUser()呼び出し時に発生。
     *
     * @param type            チャンネル情報変更タイプ(更新/削除) である{@link ChannelUpdateType}
     * @param channelUserInfo変更されるユーザー情報である{@link IChannelUserInfo}
     * @param userId          変更対象のユーザーID
     * @param accountId       変更対象のアカウントID
     */
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {

    }

    /**
     * 同じチャンネルの他のノードでルーム状態変化があった時に呼び出し
     * <p>
     * updateChannelRoomInfo()呼び出し時に発生
     *
     * @param type            チャンネル情報変更タイプ(更新/削除) {@link ChannelUpdateType}
     * @param channelRoomInfo変更されるルーム情報である{@link IChannelRoomInfo}
     * @param roomId          変更対象のルームID
     */
    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {

    }

    /**
     * クライアントからチャンネル情報をリクエスト時に呼び出し  (Base.GetChannelInfoReq)
     *
     * @param outPayloadクライアントへ伝達されるチャンネル情報
     */
    @Override
    public void onChannelInfo(IPayload payload) {

    }

    /**
     * ノードが初期化される時に呼び出し
     */
    @Override
    public void onInit() {

    }

    /**
     * Readyになる前に処理する部分のために呼び出し
     */
    @Override
    public void onPrepare() {

    }

    /**
     * Readyになる時に呼び出し
     */
    @Override
    public void onReady() {

    }

    /**
     * Pauseになる時に呼び出し
     *
     * @param payloadコンテンツから伝達したい追加情報
     */
    @Override
    public void onPause(IPayload payload) {

    }

    /**
     * Shutdown命令を受け取ると呼び出し
     */
    @Override
    public void onShuttingdown() {

    }

    /**
     * Resumeになる時に呼び出し
     *
     * @param payloadコンテンツから伝達したい追加情報
     */
    @Override
    public void onResume(IPayload payload) {

    }
}
```

```java
// プロトコルバッファ MyGame.GameNodeTest の入力があった場合に動作するメッセージ処理クラス
@GameAnvilController
public class _GameNodeTest {
    // (2) SampleGameNodeで処理したいプロトコルとハンドラをマッピング
    @GameNodeMapping(
        value = MyGame.GameNodeTest.class, // 処理するプロトコルバッファ
        loadClass = SampleGameNode.class   // メッセージを受信する対象(SampleGameNode)
    )
    public void execute(IGameNodeDispatchContext ctx) {
        // ここで行う作業を作成
    }
}
```

コールバックの意味と用途は以下の表を参考にしてください。

| コールバック名                    | 意味            | 説明                                                                                                                                                                                                  |
|-------------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate                  | オブジェクト生成         | オブジェクトが生成された時に呼び出されます。生成されたタイプで使用可能なAPIを使用できるコンテキストを受け取ります。コンテンツで必要であれば保存して使用できます。                                                                                       |
| onChannelUserInfoUpdate | チャンネルのユーザー情報更新 | 同じチャンネルに属する複数のGameNodeのうち、1つのGameNodeでチャンネルのユーザー情報が変更された時、同じチャンネル内の残りの全てのGameNodeで同期のために呼び出されます。この時、ユーザーは受け取った情報をもとに現在のGameNodeのチャンネル情報を更新できます。                 |
| onChannelRoomInfoUpdate | チャンネルのルーム情報更新 | 同じチャンネルに属する複数のGameNodeのうち、1つのGameNodeでチャンネルのルーム情報が変更された時、同じチャンネル内の残りの全てのGameNodeで同期のために呼び出されます。この時、ユーザーは受け取った情報をもとに現在のGameNodeのチャンネル情報を更新できます。                  |
| onChannelInfo           | チャンネル情報リクエスト     | クライアントがチャンネル情報をリクエストする時に呼び出されます。ユーザーはこのコールバックで任意にチャンネル情報を構成してクライアントへ伝達できます。                                                                                                       |
| onInit                  | 初期化          | ノードが最初の初期化を行う時に呼び出します。ノード駆動前に必要な初期化作業がある場合、このコールバックが適しています。この時、ノードはまだメッセージを処理しません。                                                                                 |
| onPrepare               | 準備           | ノード初期化が完了した後に呼び出されます。ユーザーはノードが準備完了する前に任意の作業をここで処理できます。この時、ノードはメッセージを処理できます。                                                                                 |
| onReady                 | 準備完了         | ノードが全ての準備を終えた後、駆動完了段階です。この時、ノードはReady状態なのでユーザーはこの時から全ての機能を使用できます。                                                                                                 |
| onPause                 | 一時停止         | ノードを一時停止すると呼び出されます。ユーザーはノードが一時停止される時に追加で処理したいコードをここに実装できます。                                                                                                          |
| onShuttingdown          | ノード停止         | ノードがShutdown命令を受け取る時に呼び出されます。停止したノードは再開(Resume)できません。                                                                                                                         |
| onResume                | 再開           | ノードが一時停止状態で駆動を再開する時に呼び出されます。ユーザーは再開状態で処理したいコードをここに実装できます。                                                                                                          |

全てのノードはユーザー定義メッセージを処理するためのメッセージハンドラ登録過程が必要です。特にGameNodeはゲームコンテンツのためにこのような過程が必須です。サーバー実行前にメインクラスで設定を行います。(1)GameAnvilConfigに設定されているゲームサービス名を生成します。ゲームサービス名は必ずGameAnvilConfigに定義されている名前を使用する必要があります。(2) そして処理したいメッセージを実装しておいた[ハンドラ](server-impl-07-message-handling.md#_2)と接続します。1つのGameNodeクラスはただ1つのサービスに対してのみ登録できます。

このようなGameNodeの主目的はノードに接続された全てのGameUserとGameRoomオブジェクトに対する処理を実行することです。これについてはすぐ後で説明します。

## ユーザー実装

ユーザーオブジェクトはログイン過程を経てGameNodeに生成されます。ユーザーベースのコンテンツはこのクラスを中心に実装する必要があります。先ほど見てきた全ての例と同様に、ユーザーも処理する固有のメッセージとハンドラを接続できます。以下のサンプルコードを見ると、ユーザーはかなり多くのコールバックメソッドを提供していることがわかります。このうち一部は基本実装が提供されるため、特別必要な状況でなければ再定義しなくても構いません。これはユーザーだけでなくエンジンで提供する大部分のクラスに該当します。

```java
@GameAnvilUser(
    gameServiceName = "MyGame", // ユーザーが所属するノード(上記のSampleGameNodeと同じサービス名)
    gameType = "BasicUser",     // ユーザーの固有タイプ、"BasicUser"というユーザータイプのユーザーをエンジンに登録
    useChannelInfo = true       // チャネル間の情報同期設定
)
public class SampleGameUser implements IUser {
    private IUserContext userContext;

    /**
     * ユーザーコンテキストを伝達するために呼び出し
     * <p/>
     * オブジェクトが生成された後、一度呼び出される
     *
     * @param userContextユーザーコンテキスト
     */
    @Override
    public void onCreate(IUserContext userContext) {
        this.userContext = userContext;
    }

    /**
     * ログイン時に呼び出し
     *
     * @param payload        クライアントから受け取った{@link IPayload}
     * @param sessionPayload onBeforeLoginから伝達された{@link IPayload}
     * @param outPayload     クライアントへ伝達する{@link IPayload}
     * @return戻り値がtrueならログイン成功、falseならログイン失敗
     */
    @Override
    public boolean onLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * ログイン成功後に必要な後処理のために呼び出し
     * <p/>
     * (つまり、onLoginあるいはonReLoinが成功した後に呼び出し)
     *
     * @param isRelogined 再ログインの有無
     */
    @Override
    public void onAfterLogin(boolean isRelogined) {

    }

    /**
     * すでにログインしている状態で、再度ログインを試みる時に呼び出し
     * <p/>
     * ログインしている状態では接続が切れても、ユーザーのゲームユーザーオブジェクトがゲームノードに一定期間有効な状態で残っている
     *
     * @param payload        クライアントから伝達した任意の{@link IPayload}
     * @param sessionPayload onBeforeLoginから伝達された{@link IPayload}
     * @param outPayload     クライアントへ伝達する任意の{@link IPayload}
     * @return 戻り値がtrueなら再ログイン成功、falseなら再ログイン失敗
     */
    @Override
    public boolean onReLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * クライアントとの接続が切れた時に呼び出されるコールバック
     */
    @Override
    public void onDisconnect() {

    }

    /**
     * ユーザーが属するノードがPauseになる時、該当ユーザーもPauseになり呼び出し
     */
    @Override
    public void onPause() {

    }

    /**
     * ユーザーが属するノードがResumeになる時、該当ユーザーもResumeになり呼び出し
     */
    @Override
    public void onResume() {

    }

    /**
     * ユーザーがログアウトする時に呼び出し
     *
     * @param payload   クライアントから受け取った{@link IPayload}
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     */
    @Override
    public void onLogout(IPayload payload, IPayload outPayload) {

    }

    /**
     * 該当ユーザーがログアウト可能か確認するために呼び出し
     * <p/>
     * エンジンユーザーはこのコールバックで現在のゲームユーザーがログアウトしても問題ないか決定できる
     *
     * @return 戻り値がfalseならログアウト進行が止まり、以後に定期的に再度コールバックが呼び出される。戻り値がtrueならログアウトを進行する
     */
    @Override
    public boolean canLogout() {
        return true;
    }

    /**
     * ルームのonLeavingRoomが実行されルームからユーザーが完全に出た後に呼び出し
     * <p/>
     * ルームから出たユーザーが処理すべき作業を行う
     */
    @Override
    public void onAfterLeaveRoom() {

    }

    /**
     * ユーザーが他のノードへ移動(転送)可能な状態か確認するために呼び出し
     *
     * @return 戻り値がtrueなら転送可能な状態、falseなら不可能な状態。不可能な状態の場合、もしSafePauseが進行中ならSafePauseが終了するまでは該当ユーザーを転送するために持続的に呼び出される
     */
    @Override
    public boolean canTransfer() {
        return true;
    }

    /**
     * すでにログインしている状況で他のデバイスから同じユーザーがログインする時に呼び出し
     *
     * @param newDeviceId              新しく接続したユーザーのデバイスID値
     * @param outPayloadForKickUserクライアントへ伝達する{@link IPayload}。kickOutあるいはLoginRes情報含む
     * @return 戻り値がtrueなら新しく接続したユーザーがログインし、既存ユーザーは強制ログアウト処理される。falseなら新しく接続したユーザーがログイン失敗
     */
    @Override
    public boolean onLoginByOtherDevice(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 任意のユーザータイプですでにログインしている状態で、他のユーザータイプでログインを試みる時に呼び出し
     *
     * @param userType   新しくログインを試みるユーザーのタイプ
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     * @return 戻り値がtrueなら新しいログイン成功、falseなら失敗
     */
    @Override
    public boolean onLoginByOtherUserType(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * すでにログインしている状態で(再接続などの理由で) 他のコネクションを通じてログインを試みる場合に呼び出し
     *
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     */
    @Override
    public void onLoginByOtherConnection(IPayload payload) {

    }

    /**
     * ルームマッチメイキングリクエストを受け取ると呼び出される
     *
     * @param roomType                クライアントとサーバー間で事前定義したルーム種類を区分する任意の値
     * @param matchingGroup           マッチングされるルームマッチンググループ伝達
     * @param matchingUserCategory マッチングされるマッチングユーザーカテゴリーを伝達
     * @param payload                 クライアントから受け取った{@link IPayload}
     * @return {@link RoomMatchResult}タイプでマッチングされたルームの情報を返す。nullを返す場合クライアントリクエストオプションに従って新しいルームが生成されるかリクエスト失敗処理
     */
    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        return RoomMatchResult.FAILED;
    }

    /**
     * クライアントのルームマッチメイキングリクエストを処理する際に失敗する場合呼び出し
     *
     * @param matchRoomFailCode ルームマッチメイキングが失敗した理由
     */
    @Override
    public void onMatchRoomFail(MatchRoomFailCode matchRoomFailCode) {

    }

    /**
     * クライアントのユーザーマッチメイキングリクエストを処理する際に失敗する場合呼び出し
     *
     * @param matchUserFailCodeユーザーマッチメイキングが失敗した理由
     */
    @Override
    public void onMatchUserFail(MatchUserFailCode matchUserFailCode) {

    }

    /**
     * クライアントからユーザーマッチメイキングをリクエストした場合に呼び出されるコールバック
     *
     * @param roomType         クライアントとサーバー間で事前定義したルーム種類を区分する任意の値
     * @param matchingGroupマッチンググループ
     * @param payload          クライアントから受け取った{@link IPayload}
     * @param outPayload       クライアントへ伝達する{@link IPayload}
     * @return戻り値がtrueならユーザーマッチメイキングリクエスト成功、falseなら失敗
     */
    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * ユーザーマッチメイキングがキャンセルされる時に呼び出し
     *
     * @param reasonキャンセルされた理由。タイムアウト(TIMEOUT)、ユーザーのリクエストによるキャンセル(CANCEL)、マッチノードの終了によるキャンセル(SHUTDOWN)
     */
    @Override
    public void onMatchUserCancel(MatchCancelReason matchCancelReason) {

    }

    /**
     * ユーザーが他のノードへ移動(転送)する時、出発ノードから伝達するデータをまとめるために呼び出される
     *
     * @param transferPack他のノードへ持っていくデータを保存するためのパッケージ
     */
    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    /**
     * ユーザーが他のノードから移動(転送)されてきた時、到着地ノードへデータ、再登録すべきタイマーキーを伝達するために呼び出される
     * <p>
     * TimerHandlerTransferPackを通じてuserに登録されていたtimerHandlerKeyリストを確認する
     * <p/>
     * TimerHandlerTransferPackのreRegister()を活用して使用するtimerHandlerを再登録する
     *
     * @param transferPack             他のノードから持ってきたデータを伝達するためのパッケージ
     * @param timerHandlerTransferPack
     */
    @Override
    public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    /**
     * ユーザーが他のノードから移動(転送)されてきた時、転送が完了した後に呼び出される
     */
    @Override
    public void onAfterTransferIn() {

    }

    /**
     * クライアントからスナップショットリクエスト時に呼び出される
     * <p>
     * 主に接続が切れてサーバー状態が変わる可能性がある場合に呼び出して、クライアントとサーバー状態情報を同期するのに使用される
     *
     * @param payload    クライアントからサーバーへ伝達されるDefineがセットされた{@link IPayload}伝達
     * @param outPayload サーバーからクライアントへ伝達されるDefineがセットされた{@link IPayload}を伝達
     */
    @Override
    public void onSnapshot(IPayload payload, IPayload outPayload) {

    }

    /**
     * クライアントから他のチャンネルへの移動リクエストをする時、現在ユーザーがチャンネル移動可能な状態か確認するために呼び出される
     * <p>
     * 注意！もし、ユーザーが明示的にmoveChannel() APIを呼び出してチャンネルを移動する場合にはcanMoveOutChannel()は呼び出されない
     * <p/>
     * エンジンにより暗黙的なチャンネル移動が発生する時のみ呼び出される
     *
     * @param destinationChannelId移動対象チャンネルのID
     * @param payload                 クライアントから受け取った{@link IPayload}
     * @param errorPayload         チャンネル移動に失敗した場合、サーバーからクライアントへ伝達するエラー{@link IPayload}。成功の場合は伝達されない
     * @return戻り値がfalseならチャンネル移動が不可能なためリクエストは失敗、trueなら成功
     */
    @Override
    public boolean canMoveOutChannel(String channelId, IPayload payload, IPayload outPayload) {
        return false;
    }

    /**
     * 他のノードへチャンネル移動をする時、出発ノードで呼び出される
     *
     * @param destinationChannelId移動対象チャンネルのID
     * @param outPayload            移動するチャンネルへ伝達する{@link IPayload}
     */
    @Override
    public void onMoveOutChannel(String channelId, IPayload payload) {

    }

    /**
     * 他のノードへチャンネル移動が完了した後、出発ノードで呼び出される
     */
    @Override
    public void onAfterMoveOutChannel() {

    }

    /**
     * 他のノードへチャンネル移動をする時、対象ノードへ進入しながら呼び出される
     *
     * @param sourceChannelId移動する前のチャンネルID
     * @param payload       クライアントから受け取った{@link IPayload}
     * @param outPayload    クライアントへ伝達する{@link IPayload}
     * @throws GameAnvilException IOException, ExecutionException, InterruptedException発生時にGameAnvilExceptionとしてまとめてthrow
     */
    @Override
    public void onMoveInChannel(String channelId, IPayload payload, IPayload outPayload) {

    }
}

```

```java
@GameAnvilController
public class _GameUserTest {
   // SampleGameUserで処理したいプロトコルとハンドラをマッピング
    @GameUserMapping(
        value = MyGame.GameUserTest.class, // 処理するプロトコルバッファ
        loadClass = SampleGameUser.class   // メッセージを受け取る対象(SampleGameUser)
    )
    public void execute(IUserDispatchContext ctx) {
       // ここで行う作業を作成
    }
}
```

特に、ユーザーはエンジンに登録するための設定が他のクラスに比べて多く要求されます。まず、どのゲームサービスのためのユーザーなのか登録した後、ユーザータイプを登録します。このユーザータイプは名前の通りユーザーの種類を区別するための用途として、クライアントでも同様にAPIを呼び出す時に使用されます。つまり、該当APIがサーバーのどのユーザータイプに対する呼び出しかを明示するものです。したがって必ずサーバーとクライアント間にこのようなユーザータイプを任意の文字列で事前定義しておく必要があります。このサンプルコードでは「BasicUser」というユーザータイプを使用します。これについてのより詳細な説明は別の章で改めて扱います。最後にこのユーザー情報が[チャンネル間ユーザー情報同期](server-impl-09-channel.md#_4)に使用されるかどうかを決定できます。もしチャンネル間情報同期が必要なければ省略できます。

上記のサンプルコードでonLoginで始まるコールバックは全てログインに関連して呼び出されます。例えば最初のログインリクエストに対してはonLoginコールバックが呼び出され、すでにログインされている状態で再ログインを処理する場合はonReLoginコールバックが呼び出されます。同様にログアウトを処理する時にはonLogoutコールバックが呼び出されます。このようにGameAnvilのコールバックはその名前とJavaDoc注釈を通じてその用途が大部分明確に説明されます。

ユーザーはいつでもチャンネル間での移動が可能です。このようなチャンネルに関連した内容は後で[別の章](server-impl-09-channel)で個別に説明するので、ここではひとまず進むことにします。それだけでなくユーザーは複数のGameNode間で転送可能なオブジェクトです。これに関連した機能もまた[別の章](server-impl-08-object-transfer)でより詳しく説明します。 

GameAnvilは2種類のマッチメイキング機能、ルームマッチメイキングとユーザーマッチメイキングを提供します。このようなマッチメイキングリクエストがユーザーに到達するとonMatchRoomあるいはonMatchUserコールバックが呼び出されます。ユーザーはこのコールバックでGameAnvilが提供するマッチメーカーを使用することもでき、直接実装したりサードパーティーから提供された他のマッチメーカーを使用することもできます。これについてのより詳細な説明はすぐ[次の章](server-impl-04-match-node)でMatchNodeを扱いながら改めて行います。

このようなユーザーのコールバックを整理すると以下の表の通りです。

| コールバック名                   | 意味                     | 説明                                                                                                                                                                                                                                                                |
|--------------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate                 | オブジェクト生成                  | オブジェクトが生成された時に呼び出されます。生成されたタイプで使用可能なAPIを使用できるコンテキストを受け取ります。コンテンツで必要であれば保存して使用できます。                                                                                                                                                   |
| onLogin                  | ログイン                    | 初めてログインをする時に呼び出されます。一般的にユーザーはこのコールバックでDBなどのストレージからユーザー情報を取得し、ゲームユーザーオブジェクトを初期化する作業を行います。                                                                                                                                                     |
| onAfterLogin             | ログイン成功後処理              | onLoginが成功した後に呼び出されます。ログインに対する後処理作業をこのコールバックで行うことができます。                                                                                                                                                                                            |
| onReLogin                | 再ログイン                   | すでにログインされている状態で再度ログインをする場合にはonLoginではなくonReLoginが呼び出されます。つまり、再ログインに対する作業はこのコールバックで処理します。                                                                                                                                                  |
| onDisconnect             | 接続終了                  | クライアントから接続が切れた時に呼び出されます。この時、追加で処理するコードをここに実装します。                                                                                                                                                                                              |
| onPause                  | 一時停止                  | コンソールを通じてGameNodeを一時停止すると、該当GameNodeの全てのユーザーに対して呼び出されます。ユーザーはノードが一時停止される時にユーザーで追加で処理したいコードをここに実装できます。                                                                                                                                   |
| onResume                 | 再開                     | コンソールを通じてGameNodeが一時停止状態で駆動を再開すると、該当GameNodeの全てのユーザーに対して呼び出されます。ユーザーは再開状態でユーザーに対して処理したいコードをここに実装できます。                                                                                                                                |
| onLogout                 | ログアウト                   | ユーザーがログアウトする時に呼び出されます。これはユーザーが明示的にリクエストしたログアウトの場合もあり、接続切れ状態で設定された時間を超過した場合にエンジンにより自動的にログアウトされる場合もあります。                                                                                                                                        |
| canLogout                | ログアウト可能性確認             | 該当ユーザーが現在ログアウト可能な状態かチェックするために呼び出されます。ゲームプレイ中だったり情報を失ってはいけない状況ではfalseを返してログアウトを遅らせることができます。もしfalseを返すとエンジンは任意の時間以後に持続的にこのコールバックを呼び出します。                                                                                |
| onAfterLeaveRoom         | ルーム退室後処理                | ルームのonLeavingRoomが実行されルームからユーザーが完全に出た後に呼び出されます。ルームから出たユーザーが処理すべき作業を進めます。                                                                                                                                                                  |
| canTransfer              | ユーザー転送が可能な状態か確認      | 該当ユーザーが他のノードへ転送されうる状態かチェックするために呼び出されます。ゲームプレイ中だったりまだ準備ができていない場合にはfalseを返して転送を遅らせることができます。falseを返した場合にはエンジンが任意の時間以後に持続的にこのコールバックを呼び出します。                                                                           |
| onLoginByOtherDevice     | 他のデバイスでのログイン試行     | すでにログインされている状態で他のデバイスから追加ログインリクエストが来た時に呼び出されます。ユーザーは既存のユーザーと新しいユーザーのどちらをログインさせるか戻り値で決定できます。                                                             |
| onLoginByOtherUserType   | 他のユーザータイプでのログイン試行    | すでにログインされている状態で他のユーザータイプでログインリクエストが来た時に呼び出されます。新しいユーザータイプに対してログインを進めるかどうかを戻り値で決定できます。                                                                   |
| onLoginByOtherConnection | 他のコネクションでのログイン試行      | すでにログインされている状態で(再接続により) 以前とは異なるコネクションでログイン試行をする場合に呼び出されます。もし、再接続に対する追加作業が必要であればこのコールバックで行うことができます。                                                          |
| onMatchRoom              | ルームマッチメイキングリクエスト            | ユーザーがルームマッチメイキングをリクエストすると呼び出されます。この時、ユーザーはこのコールバックでエンジンが提供するルームマッチメイキングAPIあるいは第三のマッチメイキングソリューションを任意に使用できます。                                                               |
| onMatchRoomFail          | ルームマッチメイキングリクエスト失敗         | ユーザーがルームマッチメイキングをリクエストすると呼び出されます。                                                                                                                                                      |
| onMatchUserFail          | ユーザーマッチメイキングリクエスト失敗        | ユーザーがユーザーマッチメイキングをリクエストすると呼び出されます。ユーザーはこのコールバックでエンジンが提供するユーザーマッチメイキングAPIあるいは第三のマッチメイキングソリューションを任意に使用できます。                                                               |
| onMatchUser              | ユーザーマッチメイキングリクエスト           | ユーザーがユーザーマッチメイキングをリクエストすると呼び出されます。ユーザーはこのコールバックでエンジンが提供するユーザーマッチメイキングAPIあるいは第三のマッチメイキングソリューションを任意に使用できます。                                                               |
| onMatchUserCancel        | ユーザーマッチメイキングキャンセル           | ユーザーが以前に申請したユーザーマッチメイキングをキャンセルすると呼び出されます。すでにマッチングが完了した状況のようにキャンセルできない場合には失敗することもあります。                                                                                 |
| onTransferOut            | 既存ノードから転送されて出ていく準備   | ユーザーが他のGameNodeへ転送される時、ソースノードで転送を開始する時に呼び出されます。ユーザーはこのコールバックでユーザーオブジェクトと共に転送するデータパッケージをまとめることができます。                                                               |
| onTransferIn             | 新しいノードへの転送完了処理     | ユーザーが他のGameNodeへ転送される時、対象ノードで転送完了しながら呼び出されます。ユーザーは一緒に持ってきたデータパッケージを展開して元のユーザー状態に復旧できます。                                                              |
| onAfterTransferIn        | 転送完了後処理             | ユーザー転送が成功した場合、対象ノードで後処理のために呼び出されます。                                                                                                                                         |
| onSnapshot               | クライアントからスナップショットリクエスト        | クライアントからスナップショットリクエスト時に呼び出されます。主に接続が切れてサーバー状態が変わる確率がある場合に呼び出してクライアントとサーバー状態情報を同期するのに使用されます。                                                                    |
| canMoveOutChannel        | チャンネル移動が可能な状態か確認    | ユーザーが他のチャンネルへ移動できる状態かチェックするために呼び出されます。もし、ユーザーが明示的にmoveChannel() APIを呼び出してチャンネルを移動する場合には呼び出されません。ただエンジンにより暗黙的なチャンネル移動が発生する時のみ呼び出されます。               |
| onMoveOutChannel         | 既存チャンネルから他のチャンネルへ移動準備   | ユーザーが他のチャンネルへ移動する時、ソースノードで呼び出されます。ユーザーは希望する情報をoutPayloadに入れて対象チャンネルへ持っていくことができます。                                                                           |
| onAfterMoveOutChannel    | 既存チャンネルから他のチャンネルへ移動準備完了 | onMoveOutChannelが成功すれば後処理のために呼び出されます。                                                                                                                                        |
| onMoveInChannel          | 新しいチャンネルへの移動処理         | ユーザーが他のチャンネルへ移動する時、対象ノードで呼び出されます。ユーザーは任意の情報をoutPayloadに入れてクライアントへ伝達できます。                                                                            |

### ログインとは？

先ほど説明した内容とサンプルコードでログインに関する内容が頻繁に登場します。また、このようなログインはクライアントがサーバーに接続した後、GameNodeに自身のユーザーオブジェクトを作る過程だと定義できます。コールバックメソッドのうちonLogin()は最初にユーザーを生成するためにログインを試みる過程で呼び出されます。この時、ユーザーはユーザーオブジェクトを構成するための情報をDBなどから獲得できます。このようなonLogin()コールバックが成功すればGameNode上に該当ユーザーオブジェクトが生成されます。このようにログインが完了すると、直接定義したプロトコルを基盤にクライアントは自身のユーザーオブジェクトを通じて他のオブジェクトとメッセージをやり取りしながら様々なコンテンツを実装できます。

### ログアウト

ログアウトはログインの反対概念です。つまり、GameNode上で自身のユーザーオブジェクトを除去する過程です。ログアウトを開始すると該当ユーザーオブジェクトはonLogout()コールバックを呼び出してメモリ上で削除される前にDBなどで自身の最終状態を保管できます。このようなログアウトはクライアントが明示的にリクエストすることもでき、クライアントの接続が切れた状態で任意の時間が経過した後、エンジンにより自動的に処理されたりもします。したがって、もしモバイルゲームのように頻繁な接続切れが予想される場合にはすぐにログアウトが進行されないよう適切な[設定](server-impl-16-config-vm.md#game)をしておくことができます。

## ルーム実装

2名以上のユーザーはルームを通じて同期されたメッセージフローを作ることができます。つまり、ユーザーのリクエストはルームの中で全て順序が保証されます。もちろん1名のユーザーのためのルーム生成もコンテンツによっては意味を持つこともあります。ルームをどのように使用するかはあくまでエンジンユーザーの役割です。このようなルームはユーザーと同様に基本クラスであるIRoomインターフェースを実装して様々なコールバックメソッドを再定義でき、独自にメッセージを処理することもできます。以下のサンプルコードはSampleUserのためのSampleRoomクラスです。

 ```java
 @GameAnvilRoom(
    gameServiceName = "MyGame", // ルームが所属するノード(上記のSampleGameNodeと同じサービス名)
    gameType = "BasicRoom",      // ルームの固有タイプ、"BasicRoom"というタイプのルームをエンジンに登録
    useChannelInfo = true       // チャネル間の情報同期設定
)
public class SampleRoom implements IRoom<SampleUser> {
    private IRoomContext roomContext;

    /**
     * ルームコンテキストを伝達するために呼び出し
     * <p/>
     * オブジェクトが生成された後、一度呼び出される
     *
     * @param roomContextルームコンテキスト
     */
    @Override
    public void onCreate(IRoomContext<SampleUser> roomContext) {
        this.roomContext = roomContext;
    }

    /**
     * ルームが初期化される時に呼び出し
     */
    @Override
    public void onInit() {

    }

    /**
     * ルームが削除される時に呼び出される
     */
    @Override
    public void onDestroy() {

    }

    /**
     * 新しいルームを生成する時に呼び出し
     * <p/>
     * 戻り値によってルームを生成するかどうかが決定される
     *
     * @param user       リクエストしたユーザーオブジェクト
     * @param inPayload  クライアントから受け取った{@link IPayload}
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     * @return戻り値がtrueならルーム生成成功、falseなら失敗
     */
    @Override
    public boolean onCreateRoom(SampleUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 任意のルームに参加する時に呼び出される
     * <p/>
     * 戻り値によってルームに参加するかどうかが決定される
     *
     * @param user       リクエストしたユーザーオブジェクト
     * @param inPayload  クライアントから受け取った{@link IPayload}
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     * @return戻り値がtrueなら入室成功、falseなら失敗
     */
    @Override
    public boolean onJoinRoom(SampleUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * ルームから出る時に呼び出される
     * <p/>
     * 戻り値によってルームから出るかどうかが決定される
     *
     * @param user       リクエストをしたユーザーオブジェクト
     * @param inPayload  クライアントから受け取った{@link IPayload}
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     * @return戻り値がtrueなら退室成功、falseなら失敗
     */
    @Override
    public boolean canLeaveRoom(SampleUser user, IPayload payload, IPayload outPayload) {
        return true;
    }

    /**
     * ユーザーがルームから出る時に呼び出し
     * <p/>
     * ユーザーがルームから出る前に最後の作業を処理する
     *
     * @param userルームから出るユーザー
     */
    @Override
    public void onLeaveRoom(SampleUser user) {

    }

    /**
     * ユーザーがルームから完全に出た後に呼び出される
     * <p/>
     * ルームとルームに残っているユーザーに関する作業を処理する
     */
    @Override
    public void onAfterLeaveRoom() {

    }

    /**
     * 再ログイン時に該当ルームへ自動的に再進入する際に呼び出される
     *
     * @param user       ルームへ入るユーザーオブジェクト
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     */
    @Override
    public void onRejoinRoom(SampleUser user, IPayload payload) {

    }

    /**
     * 該当ルームが他のノードへ移動(転送)する時、出発ノードから伝達するデータをまとめるために呼び出される
     *
     * @param transferPack 他のノードへ持っていくデータを保存するためのパッケージである {@link ITransferPack}
     */
    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    /**
     * 該当ルームが他のノードへ移動(転送)する時、対象ノードで新しく生成されたルームオブジェクトを元の状態に復元、処理するタイマーハンドラを登録するために呼び出される
     * <p>
     * TimerHandlerTransferPackを通じてルームに登録されていたtimerKeyリストを確認する
     * <p/>
     * TimerHandlerTransferPackのreRegister()を活用して使用するtimerHandlerを再登録する
     * <p/>
     * この時点ではルームは完全に復旧されていないため、他の場所(ノード、ユーザーなど)へのメッセージリクエストが制限される
     *
     * @param userList               移動するユーザーリスト
     * @param transferPack              他のノードから持ってきたデータパッケージである {@link ITransferPack}
     * @param timerHandlerTransferPack
     */
    @Override
    public void onTransferIn(List<SampleUser> list, ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    /**
     * 該当ルームが他のノードへ移動(転送)完了した後に呼び出される
     */
    @Override
    public void onAfterTransferIn() {

    }

    /**
     * ルームが属するノードが Pause になる時、該当ルームも Pause になり呼び出される
     */
    @Override
    public void onPause() {

    }

    /**
     * ルームが属するノードが Resume になる時、該当ルームも Resume になり呼び出される
     */
    @Override
    public void onResume() {

    }

    /**
     * クライアントからパーティーマッチメイキングをリクエストした場合に呼び出される
     * <p/>
     * パーティーマッチメイキングのためには2名以上のユーザーがパーティータイプのネームドルームに入室しなければならない
     *
     * @param roomType         クライアントとサーバー間で事前定義したルーム種類を区分する任意の値
     * @param matchingGroupマッチンググループ
     * @param user           パーティーマッチメイキングをリクエストしたユーザー(ルームリーダー)
     * @param payload          クライアントから受け取った{@link IPayload}
     * @param outPayload       クライアントへ伝達する{@link IPayload}
     * @return 戻り値が true ならパーティーマッチメイキングリクエスト成功、false なら失敗
     */
    @Override
    public boolean onMatchParty(String roomType, String matchingGroup, SampleUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * パーティーマッチメイキングがキャンセルされる時に呼び出される
     *
     * @param reasonキャンセルされた理由。タイムアウト(TIMEOUT)、ユーザーのリクエストによるキャンセル(CANCEL)、マッチノードの終了によるキャンセル(SHUTDOWN)
     */
    @Override
    public void onMatchPartyCancel(MatchCancelReason matchCancelReason) {

    }

    /**
     * ルームマッチメイキングがキャンセルされる時に呼び出される
     *
     * @param reason キャンセルされた理由。(SHUTDOWN: マッチノードの終了によるキャンセル)
     */
    @Override
    public void onForceMatchRoomUnregistered(MatchCancelReason matchCancelReason) {

    }

    /**
     * ルームが他のノードへ移動(転送)可能な状態か確認するために呼び出される
     *
     * @return 戻り値が true なら転送可能な状態、false なら不可能な状態
     * <p/>
     * 不可能な状態の場合、もし SafePause が進行中なら SafePause が終了するまでは該当ルームを転送するために持続的に呼び出される
     */
    @Override
    public boolean canTransfer() {
        return true;
    }
}
```

```java
// プロトコルバッファ MyGame.GameRoomTest 入力が入ってきた時に動作するメッセージ処理クラス
@GameAnvilController
public class _GameRoomTest {
    // SampleRoomで処理したいプロトコルとハンドラをマッピング
    @GameRoomMapping(
        value = MyGame.GameRoomTest.class, // 処理するプロトコルバッファ
        loadClass = SampleRoom.class   // メッセージを受け取る対象(SampleRoom) 
    )
    public void execute(IRoomDispatchContext ctx) {
        // ここで行う作業を作成
    }
}
```

ルームもやはりエンジンに登録するために様々な設定が必要です。まずどのゲームサービスのためのルームなのか登録した後、ルームタイプを登録します。このルームタイプは名前の通りルームの種類を区別するための用途として、クライアントでも同様にAPIを呼び出す時に使用されます。つまり、該当APIがサーバーのどのルームタイプに対する呼び出しかを明示するものです。したがって必ずサーバーとクライアント間にこのようなルームタイプを任意の文字列で事前に定義しておく必要があります。このサンプルコードでは「BasicRoom」というルームタイプを使用します。これについてのより詳細な説明は別の章で改めて扱います。最後にこのルーム情報が[チャンネル間ルーム情報同期](server-impl-09-channel.md#_4)に使用されるかどうかを決定できます。もしチャンネル間情報同期が必要なければ省略できます。

ルームは最も基本的な3つのコールバックであるonCreateRoom、onJoinRoom、onLeaveRoomを提供します。それぞれルームの生成、参加、そして退室について呼び出されます。ユーザーは該当コールバックで関連する機能を直接実装できます。例えば、ルームを生成しながらルームリーダーを指定し基本的なデータ構造を初期化できます。また任意のルーム参加リクエストに対してはルームのユーザーリストを更新したり各ユーザー間の状態同期などを実行できます。

このようなルームの生成と参加はクライアントの明示的リクエストだけでなく、マッチメイキングによりエンジンが自動的に処理することもあります。例えば定員が2名のゲームでユーザーAとユーザーBがマッチングされたなら、一人は自動的にルームを生成し、もう一人は該当ルームに参加することになります。この過程はユーザーAとユーザーBのリクエストではなくエンジンで自動的に処理するものです。この過程でももちろんonCreateRoomとonJoinRoomコールバックが呼び出されます。

このようなルームのコールバックを整理すると以下の表の通りです。

| コールバック名                        | 意味                | 説明                                                                                                                                                           |
|------------------------------|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate                     | オブジェクト生成             | オブジェクトが生成された時に呼び出されます。生成されたタイプで使用可能なAPIを使用できるコンテキストを受け取ります。コンテンツで必要であれば保存して使用できます。                                                                            |
| onInit                       | 初期化               | ルームが生成される時に初期化のために呼び出されます。トピック登録などの該当ルームに対する初期化コードを作成できます。                                                                                                  |
| onDestroy                    | ルーム消滅             | ルームから最後のユーザーが退出し、これ以上処理するメッセージがなければ該当ルームは消滅します。この時呼び出されるコールバックです。                                                                                                  |
| onCreateRoom                 | ルーム生成              | クライアントがルーム生成をリクエストすると呼び出されます。コンテンツで使用するユーザーリストのためのデータ構造を生成したり、その他ルーム生成と共に処理すべきコードを作成します。                                                                            |
| onJoinRoom                   | ルーム参加              | クライアントがサーバー上に存在する任意のルームへの参加をリクエストすると呼び出されます。コンテンツで使用するユーザーリストを更新したり、ルーム内のユーザー間で同期する情報を処理できます。                                                                        |
| canLeaveRoom                 | ルーム退室確認           | ルームから退出する時に呼び出されます。戻り値によってルームから出るかどうかが決定されます。                                                                                                                    |
| onLeaveRoom                  | ルーム退室              | クライアントがルーム退室リクエストをすると呼び出されます。コンテンツで使用するユーザーリストのためのデータ構造を更新したり、その他ルームから出ながら共に処理すべきコードを作成します。                                                                          |
| onAfterLeaveRoom             | ルーム退室後処理          | onLeaveRoomが成功すれば後処理のために呼び出されます。ルームを出た直後に処理するコードを作成します。                                                                                                        |
| onRejoinRoom                 | ルーム再参加           | ユーザーがルームに入っている状態で接続切れなどにより再ログイン(ReLogin)をする場合、エンジンにより自動的に該当ルームへ再進入(ReJoin)します。この時、呼び出されるコールバックです。再進入過程で同期に必要な情報などを処理できます。                      |
| onTransferOut                | 既存ノードから転送されて出ていく準備 | ルームが他のGameNodeへ転送される時、ソースノードで転送を開始する時に呼び出されます。ユーザーはこのコールバックでルームオブジェクトと共に転送するデータパッケージをまとめることができます。ちなみにルーム転送はルーム内のユーザーそれぞれに対するユーザー転送を含みます。                                     |
| onTransferIn                 | 新しいノードへの転送完了処理  | ルームが他のGameNodeへ転送される時、対象ノードで転送完了しながら呼び出されます。ユーザーは一緒に持ってきたデータパッケージを展開して元のルーム状態に復旧できます。ちなみにルーム転送はルーム内のユーザーそれぞれに対するユーザー転送を含みます。                                    |
| onAfterTransferIn            | 新しいノードへの転送完了後処理 | ルームが他のGameNodeへ転送される時、対象ノードで転送完了しながら呼び出されます。ユーザーは一緒に持ってきたデータパッケージを展開して元のルーム状態に復旧できます。ちなみにルーム転送はルーム内のユーザーそれぞれに対するユーザー転送を含みます。                                    |
| onPause                      | 一時停止             | コンソールを通じてGameNodeを一時停止すると、該当GameNodeの全てのルームに対して呼び出されます。ユーザーはノードが一時停止される時にルームで追加で処理したいコードをここに実装できます。                                                           |
| onResume                     | 再開                | コンソールを通じてGameNodeが一時停止状態で駆動を再開すると、該当GameNodeの全てのルームに対して呼び出されます。ユーザーは再開状態でルームに対して処理したいコードをここに実装できます。                                                          |
| onMatchParty                 | パーティーマッチメイキングリクエスト        | ユーザーがパーティーマッチメイキングをリクエストすると呼び出されます。パーティーマッチメイキングは任意のNamedRoomをパーティー用途で生成した後、ルーム内の全てのユーザーが1つのパーティーとしてマッチングをリクエストする機能です。ユーザーはこのコールバックでエンジンが提供するパーティーマッチメイキングAPIあるいは第三のマッチメイキングソリューションを任意に使用できます。 |
| onMatchPartyCancel           | パーティーマッチメイキングリクエストキャンセル     | ユーザーがパーティーマッチメイキングをリクエストすると呼び出されます。パーティーマッチメイキングは任意のNamedRoomをパーティー用途で生成した後、ルーム内の全てのユーザーが1つのパーティーとしてマッチングをリクエストする機能です。ユーザーはこのコールバックでエンジンが提供するパーティーマッチメイキングAPIあるいは第三のマッチメイキングソリューションを任意に使用できます。 |
| onForceMatchRoomUnregistered | ルームマッチメイキングキャンセル        | ルームマッチメイキングがキャンセルされる時に呼び出されます。                                                                                                                                                            |
| canTransfer                  | ルーム転送が可能な状態か確認   | 該当ルームが他のノードへ転送されうる状態かチェックするために呼び出されます。 もしルームでまだゲームがプレイ中だったり準備ができていない場合にはfalseを返して転送を遅らせることができます。falseを返した場合にはエンジンが任意の時間以後に持続的にこのコールバックを呼び出します。ちなみにルーム転送は無停止パッチを進行する時にのみ使用されます。 |

## ルーム種類

先ほど見てきたルームの実装法とは別にエンジンで提供するルームの種類は大きく2つです。この2つのルームを計4つの方法で使用します。

| ルーム種類       | 説明                                                                                                                                                                                                                                                                                               |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Normal Room | クライアントが明示的にCreateRoomをリクエストして生成します。他のクライアントも該当ルームのIDを利用して明示的にJoinRoomをして参加できます。したがって参加するユーザーはあらかじめ該当ルームのIDを共有してもらわなければなりません。                                                                                                                                                              |
| Named Room  | NamedRoomはその名前のように唯一のルーム名を中心に命名されたルーム(NamedRoom)に対して動作を実行します。クライアントはサーバー群内で唯一のルーム名でNamedRoomをリクエストします。この時、もしサーバーに該当ルーム名が存在しなければリクエスト者は直接ルームを生成することになります。反対にもしサーバーに該当ルーム名がすでに存在する場合はそのルームに自動進入することになります。例えば、スタークラフトのカスタムゲームリストに現れる「3:3ハンター初心者！」のようなあらゆるルームタイトルを考えれば理解しやすいです。 |

このような2つのルーム種類は計4つの方法で使用されます。

| ルーム種類        | 使用法                                                                                                                                                                                                            |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Normal Room | 1. クライアントはCreateRoom / JoinRoomリクエストを通じて生成及び参加します。<br>2. ルームマッチメイキングを通じてNormalRoomを生成または参加できます。またCreateRoomで作成したルームもルームマッチメイキング対象として登録可能です。この時、ルームIDはエンジンにより管理されマッチングされるルームとユーザー間で自動的に共有されます。                                                                                       |
| Named Room  | 3. クライアントはNamedRoomリクエストを通じて生成及び参加します。<br>4. ユーザーマッチメイキングを通じてNamedRoomを生成または参加できます。この時、生成されるNamedRoomのルーム名はエンジンで固有に生成して管理します。ルームマッチメイキングと異なり一般的なNamedRoomとして生成したルームはユーザーマッチメイキング対象にはなりません。ただし、パーティーマッチメイキングのためにNamedRoomとしてパーティールームを生成した後、複数名のユーザーが1つのパーティーとしてマッチメイキングをリクエストすることはできます。 |

## チャンネル

GameNodeはその用途に合わせて論理的にグループ化できます。このような論理グループを[チャンネル](server-impl-09-channel)といいます。例えば、GameNode 1と2を「Beginner」チャンネルにまとめ、GameNode 3と4を「Expert」チャンネルにまとめることができます。これについての詳細な説明は[別の章](server-impl-09-channel)でより詳しく扱います。
