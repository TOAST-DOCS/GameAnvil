<!-- machine_translated: true -->

<!-- pre-align:aligned sig=b2564be6b47a -->

<a id="game-gameanvil-server-development-guide-implement-game-node"></a>
## Game > GameAnvil > サーバー開発ガイド > ゲームノード実装 { #game-gameanvil-server-development-guide-implement-game-node }

<a id="game-node"></a>
## Game Node { #game-node }

![GameNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gamenode_on_network.png)

GameNodeは実際のゲームオブジェクトが生成され、ゲームコンテンツを実装するノードです。クライアントはGatewayNodeで認証を完了した後、GameNodeにログインを完了して初めてこのようなゲームコンテンツを開始できます。

以下の画像で見られるように、クライアントは固有のAccountIdで認証を完了したコネクションを基盤に複数の論理セッションを生成できます。この時、セッションはクライアントとユーザーオブジェクトの間に作られます。下の図で赤色の点線矢印がこのようなセッションを表します。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

それぞれのセッションは該当コネクション内で固有の値で区分でき、私たちはこの値をSubIdと呼びます。つまり、AccountIdとSubIdの組み合わせでユーザーは望むだけセッションを生成できます。この時、同一サービスに対するセッションも同様に望むだけ生成できます。つまり、画像でAccountIdが1であるコネクションは"Game"サービスあるいは"Chat"サービスに対するセッションをいくらでも追加で生成できるのです。

このようなセッションが向かう場所はまさにユーザーオブジェクトです。GameNodeはこのようなユーザーオブジェクトと彼らのグループであるルームオブジェクトを管理します。今回の章はこのようなGameNodeとGameUserそしてGameRoomについて説明します。

<a id="implement-gamenode"></a>
## GameNode 実装 { #implement-gamenode }

GameNodeはBaseGameNodeクラスを実装する必要があります。以下のサンプルコードはGameNodeで基本的に再定義できるコールバックメソッドを示しています。ノード共通コールバックに加え、チャンネル管理のためのコールバックが存在します。

```java
@GameAnvilGameNode(gameServiceName = "MyGame") // (1) "MyGame"というサービスのGameNodeとしてエンジンに登録
public class SampleGameNode extends BaseGameNode {

    /**
     * 同じチャンネルの別ノードでユーザーに変化があったときに呼び出される
     * <p>
     * updateChannelUser() 呼び出し時に発生。
     *
     * @param type            チャンネル情報変更タイプ(更新/削除) である {@link ChannelUpdateType}
     * @param channelUserInfo 変更されるユーザー情報である {@link IChannelUserInfo}
     * @param userId          変更対象のユーザーID
     * @param accountId       変更対象のアカウントID
     */
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {

    }

    /**
     * 同じチャンネルの別ノードでルームの状態に変化があったときに呼び出される
     * <p>
     * updateChannelRoomInfo() 呼び出し時に発生
     *
     * @param type            チャンネル情報変更タイプ(更新/削除) {@link ChannelUpdateType}
     * @param channelRoomInfo 変更されるルーム情報である {@link IChannelRoomInfo}
     * @param roomId          変更対象のルームID
     */
    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {

    }

    /**
     * クライアントからチャンネル情報をリクエストしたときに呼び出される (Base.GetChannelInfoReq)
     *
     * @param outPayload クライアントに渡されるチャンネル情報
     */
    @Override
    public void onChannelInfo(IPayload payload) {

    }

    /**
     * ノードが初期化されるときに呼び出される
     */
    @Override
    public void onInit() {

    }

    /**
     * Ready になる前に処理する部分のために呼び出される
     */
    @Override
    public void onPrepare() {

    }

    /**
     * Ready になるときに呼び出される
     */
    @Override
    public void onReady() {

    }

    /**
     * Pause になるときに呼び出される
     *
     * @param payload コンテンツから渡す追加情報
     */
    @Override
    public void onPause(IPayload payload) {

    }

    /**
     * Shutdown コマンドを受信したときに呼び出される
     */
    @Override
    public void onShuttingdown() {

    }

    /**
     * Resume になるときに呼び出される
     *
     * @param payload コンテンツから渡す追加情報
     */
    @Override
    public void onResume(IPayload payload) {

    }
}
```

```java
// プロトコルバッファ MyGame.GameNodeTest の入力が来たときに動作するメッセージ処理クラス
@GameAnvilController
public class _GameNodeTest {
    // (2) SampleGameNode で処理したいプロトコルとハンドラーをマッピング
    @GameNodeMapping(
        value = MyGame.GameNodeTest.class, // 処理するプロトコルバッファ
        loadClass = SampleGameNode.class   // メッセージを受け取る対象（SampleGameNode）
    )
    public void execute(IGameNodeDispatchContext ctx) {
        // ここで行う作業を記述
    }
}
```

コールバックの意味と用途は以下の表を参考にしてください。

| コールバック名                    | 意味            | 説明                                                                                                                                                                                                  |
|-------------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onChannelUserInfoUpdate | チャンネルのユーザー情報更新 | 同じチャンネルに属する複数のGameNodeのうち、1つのGameNodeでチャンネルのユーザー情報が変更された時、同じチャンネル内の残りの全てのGameNodeで同期のために呼び出されます。この時、ユーザーは受け取った情報をもとに現在のGameNodeのチャンネル情報を更新できます。                 |
| onChannelRoomInfoUpdate | チャンネルのルーム情報更新 | 同じチャンネルに属する複数のGameNodeのうち、1つのGameNodeでチャンネルのルーム情報が変更された時、同じチャンネル内の残りの全てのGameNodeで同期のために呼び出されます。この時、ユーザーは受け取った情報をもとに現在のGameNodeのチャンネル情報を更新できます。                  |
| onChannelInfo           | チャンネル情報リクエスト     | クライアントがチャンネル情報をリクエストする時に呼び出されます。ユーザーはこのコールバックで任意にチャンネル情報を構成してクライアントへ伝達できます。                                                                                                       |
| onInit                  | 初期化          | ノードが最初の初期化を行う時に呼び出します。ノード駆動前に必要な初期化作業がある場合、このコールバックが適しています。この時、ノードはまだメッセージを処理しません。                                                                                 |
| onPrepare               | 準備           | ノード初期化が完了した後に呼び出されます。ユーザーはノードが準備完了する前に任意の作業をここで処理できます。この時、ノードはメッセージを処理できます。                                                                                 |
| onReady                 | 準備完了         | ノードが全ての準備を終えた後、駆動完了段階です。この時、ノードはReady状態なのでユーザーはこの時から全ての機能を使用できます。                                                                                                 |
| onPause                 | 一時停止         | ノードを一時停止すると呼び出されます。ユーザーはノードが一時停止される時に追加で処理したいコードをここに実装できます。                                                                                                          |
| onShuttingdown          | ノード停止         | ノードがShutdown命令を受け取る時に呼び出されます。停止したノードは再開(Resume)できません。                                                                                                                         |
| onResume                | 再開           | ノードが一時停止状態で駆動を再開する時に呼び出されます。ユーザーは再開状態で処理したいコードをここに実装できます。                                                                                                          |

全てのノードはユーザー定義メッセージを処理するためのメッセージハンドラ登録過程が必要です。特にGameNodeはゲームコンテンツのためにこのような過程が必須です。サーバー実行前にメインクラスで設定を行います。(1)GameAnvilConfigに設定されているゲームサービス名を生成します。ゲームサービス名は必ずGameAnvilConfigに定義されている名前を使用する必要があります。(2) そして処理したいメッセージを実装しておいた[ハンドラ](server-impl-07-message-handling.md#implement-message-handler-and-connect-messages-to-handlers)と接続します。1つのGameNodeクラスはただ1つのサービスに対してのみ登録できます。

このようなGameNodeの主目的はノードに接続された全てのGameUserとGameRoomオブジェクトに対する処理を実行することです。これについてはすぐ後で説明します。

<a id="implement-user"></a>
## ユーザー実装 { #implement-user }

ユーザーオブジェクトはログイン過程を経てGameNodeに生成されます。ユーザーベースのコンテンツはこのクラスを中心に実装する必要があります。先ほど見てきた全ての例と同様に、ユーザーも処理する固有のメッセージとハンドラを接続できます。以下のサンプルコードを見ると、ユーザーはかなり多くのコールバックメソッドを提供していることがわかります。このうち一部は基本実装が提供されるため、特別必要な状況でなければ再定義しなくても構いません。これはユーザーだけでなくエンジンで提供する大部分のクラスに該当します。

```java
@GameAnvilUser(
    gameServiceName = "MyGame", // ユーザーが所属するノード（上記SampleGameNodeと同じサービス名）
    gameType = "BasicUser",     // ユーザーの固有タイプ。"BasicUser"というユーザータイプのユーザーをエンジンに登録する
    useChannelInfo = true       // チャンネル間の情報同期設定
)
public class SampleGameUser extends BaseGameUser {

/**
     * ログイン時に呼び出される
     *
     * @param payload        クライアントから受け取った {@link IPayload}
     * @param sessionPayload onBeforeLogin から渡された {@link IPayload}
     * @param outPayload     クライアントへ渡す {@link IPayload}
     * @return 戻り値がtrueの場合はログイン成功、falseの場合はログイン失敗
     */
    @Override
    public boolean onLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        return false;
    }

/**
     * ログイン成功後に必要な後処理のために呼び出される
     * <p></p>
     * （つまり、onLoginまたはonReLoginが成功した後に呼び出される）
     *
     * @param isRelogined 再ログインかどうか
     */
    @Override
    public void onAfterLogin(boolean isRelogined) {

}

/**
     * すでにログインされている状態で、再度ログインを試みる際に呼び出される
     * <p></p>
     * ログイン状態では接続が切れても、ユーザーのゲームユーザーオブジェクトがゲームノードに一定期間有効な状態で残る
     *
     * @param payload        クライアントから渡された任意の {@link IPayload}
     * @param sessionPayload onBeforeLogin から渡された {@link IPayload}
     * @param outPayload     クライアントへ渡す任意の {@link IPayload}
     * @return 戻り値がtrueの場合はReLogin成功、falseの場合はReLogin失敗
     */
    @Override
    public boolean onReLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        return false;
    }

/**
     * クライアントとの接続が切れた際に呼び出されるコールバック
     */
    @Override
    public void onDisconnect() {

}

/**
     * ユーザーが属するノードがPauseされる際、該当ユーザーもPauseされながら呼び出される
     */
    @Override
    public void onPause() {

}

/**
     * ユーザーが属するノードがResumeされる際、該当ユーザーもResumeされながら呼び出される
     */
    @Override
    public void onResume() {

}

/**
     * ユーザーがログアウトする際に呼び出される
     *
     * @param payload    クライアントから受け取った {@link IPayload}
     * @param outPayload クライアントへ渡す {@link IPayload}
     */
    @Override
    public void onLogout(IPayload payload, IPayload outPayload) {

}

/**
     * 該当ユーザーがログアウト可能かどうかを確認するために呼び出される
     * <p></p>
     * エンジンユーザーはこのコールバックで、現在のゲームユーザーがログアウトしても問題がないかどうかを決定できます
     *
     * @return 戻り値がfalseの場合はログアウト処理が停止し、その後定期的に再度コールバックが呼び出される。戻り値がtrueの場合はログアウトを進行する
     */
    @Override
    public boolean canLogout() {
        return false;
    }

/**
     * ルームのonLeavingRoomが実行され、ルームからユーザーが完全に退出した後に呼び出される
     * <p></p>
     * ルームから退出したユーザーが処理すべき作業を行います
     */
    @Override
    public void onAfterLeaveRoom() {

}

/**
     * ユーザーが別のノードへ移動（転送）可能な状態かどうかを確認するために呼び出される
     *
     * @return 戻り値がtrueの場合は転送可能な状態、falseの場合は不可能な状態。不可能な状態の場合、SafePauseが進行中であれば、SafePauseが終了するまで該当ユーザーを転送するために継続的に呼び出される
     */
    @Override
    public boolean canTransfer() {
        return false;
    }

/**
     * すでにログインされている状況で、別のデバイスから同じユーザーがログインする際に呼び出される
     *
     * @param newDeviceId           新たに接続したユーザーのデバイスID値
     * @param outPayloadForKickUser クライアントへ渡す {@link IPayload}。kickOutまたはLoginRes情報を含む
     * @return 戻り値がtrueの場合は新たに接続したユーザーがログインした後、既存ユーザーは強制ログアウト処理。falseの場合は新たに接続したユーザーのログイン失敗
     */
    @Override
    public boolean onLoginByOtherDevice(String newDeviceId, IPayload outPayloadForKickUser) {
        return false;
    }

/**
     * 任意のユーザータイプですでにログインしている状態で、別のユーザータイプでログインを試みる際に呼び出される
     *
     * @param userType   新たにログインを試みるユーザーのタイプ
     * @param outPayload クライアントへ渡す {@link IPayload}
     * @return 戻り値がtrueの場合は新しいログインが成功、falseの場合は失敗
     */
    @Override
    public boolean onLoginByOtherUserType(String userType, IPayload outPayload) {
        return false;
    }

/**
     * すでにログインされている状態で（再接続などの理由で）別のコネクションを通じてログインを試みる場合に呼び出される
     *
     * @param outPayload クライアントへ渡す {@link IPayload}
     */
    @Override
    public void onLoginByOtherConnection(IPayload outPayload) {

}

/**
     * ルームマッチメイキングリクエストを受け取った際に呼び出される
     *
     * @param roomType             クライアントとサーバー間で事前に定義したルーム種類を区別する任意の値
     * @param matchingGroup        マッチングされるルームマッチンググループを渡す
     * @param matchingUserCategory マッチングされるマッチングユーザーカテゴリーを渡す
     * @param payload              クライアントから受け取った {@link IPayload}
     * @return {@link RoomMatchResult} タイプでマッチングされたルームの情報を返す。nullを返す場合、クライアントリクエストオプションに応じて新しいルームが作成されるか、リクエスト失敗処理となる
     */
    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        return null;
    }

/**
     * クライアントのルームマッチメイキングリクエストを処理する際に失敗した場合に呼び出される
     *
     * @param matchRoomFailCode ルームマッチメイキングが失敗した理由
     */
    @Override
    public void onMatchRoomFail(MatchRoomFailCode matchRoomFailCode) {

}

/**
     * クライアントのユーザーマッチメイキングリクエストを処理する際に失敗した場合に呼び出される
     *
     * @param matchUserFailCode ユーザーマッチメイキングが失敗した理由
     */
    @Override
    public void onMatchUserFail(MatchUserFailCode matchUserFailCode) {

}

/**
     * クライアントからユーザーマッチメイキングをリクエストした場合に呼び出されるコールバック
     *
     * @param roomType      クライアントとサーバー間で事前に定義したルーム種類を区別する任意の値
     * @param matchingGroup マッチンググループ
     * @param payload       クライアントから受け取った {@link IPayload}
     * @param outPayload    クライアントへ渡す {@link IPayload}
     * @return 戻り値がtrueの場合はユーザーマッチメイキングリクエスト成功、falseの場合は失敗
     */
    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        return false;
    }

/**
     * ユーザーマッチメイキングがキャンセルされる際に呼び出される
     *
     * @param reason キャンセルされた理由。タイムアウト(TIMEOUT)、ユーザーのリクエストによるキャンセル(CANCEL)、マッチノードの終了によるキャンセル(SHUTDOWN)
     */
    @Override
    public void onMatchUserCancel(MatchCancelReason reason) {

}

/**
     * ユーザーが別のノードへ移動（転送）する際、出発ノードで渡すデータをまとめるために呼び出される
     *
     * @param transferPack 別のノードへ持っていくデータを保存するためのパッケージ
     */
    @Override
    public void onTransferOut(ITransferPack transferPack) {

}

/**
     * ユーザーが別のノードから移動（転送）されてきた際、到着先ノードへデータおよび再登録が必要なタイマーキーを渡すために呼び出される
     * <p>
     * TimerHandlerTransferPackを通じて、userに登録されていたtimerHandlerKeyのリストを確認します
     * <p></p>
     * TimerHandlerTransferPackのreRegister()を活用して、使用するtimerHandlerを再登録します
     *
     * @param transferPack             別のノードから持ってきたデータを渡すためのパッケージ
     * @param timerHandlerTransferPack 別のノードから持ってきたタイマー情報を渡すためのパッケージ
     */
    @Override
    public void onTransferIn(IReadOnlyTransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

}

/**
     * ユーザーが別のノードから移動（転送）されてきた際、転送が完了した後に呼び出される
     */
    @Override
    public void onAfterTransferIn() {

}

/**
     * クライアントからスナップショットリクエスト時に呼び出される
     * <p>
     * 主に接続が切れ、サーバーの状態が変わる可能性がある場合に呼び出して、クライアントとサーバーの状態情報を同期するために使用されます
     *
     * @param payload    クライアントからサーバーへ渡されるDefineが設定された {@link IPayload}
     * @param outPayload サーバーからクライアントへ渡されるDefineが設定された {@link IPayload}
     */
    @Override
    public void onSnapshot(IPayload payload, IPayload outPayload) {

}

/**
     * クライアントから別のチャンネルへの移動リクエストがあった際、現在のユーザーがチャンネル移動可能な状態かどうかを確認するために呼び出される
     * <p>
     * 注意！ユーザーが明示的にmoveChannel() APIを呼び出してチャンネルを移動する場合は、canMoveOutChannel()は呼び出されません
     * <p></p>
     * エンジンによる暗黙的なチャンネル移動が発生する場合にのみ呼び出されます
     *
     * @param destinationChannelId 移動先チャンネルのID
     * @param payload              クライアントから受け取った {@link IPayload}
     * @param errorPayload         チャンネル移動に失敗した場合、サーバーからクライアントへ渡すエラー {@link IPayload}。成功の場合は渡されない
     * @return 戻り値がfalseの場合はチャンネル移動が不可能なためリクエストは失敗、trueの場合は成功
     */
    @Override
    public boolean canMoveOutChannel(String destinationChannelId, IPayload payload, IPayload errorPayload) {
        return false;
    }

/**
     * 別のノードへチャンネル移動する際、出発ノードで呼び出される
     *
     * @param destinationChannelId 移動先チャンネルのID
     * @param outPayload           移動先チャンネルへ渡す {@link IPayload}
     */
    @Override
    public void onMoveOutChannel(String destinationChannelId, IPayload outPayload) {

}

/**
     * 別のノードへのチャンネル移動が完了した後、出発ノードで呼び出される
     */
    @Override
    public void onAfterMoveOutChannel() {

}

/**
     * 別のノードへチャンネル移動する際、対象ノードへ進入しながら呼び出される
     *
     * @param sourceChannelId 移動前のチャンネルID
     * @param payload         クライアントから受け取った {@link IPayload}
     * @param outPayload      クライアントへ渡す {@link IPayload}
     * @throws GameAnvilException IOException、ExecutionException、InterruptedException 発生時にGameAnvilExceptionとしてラップしてthrow
     */
    @Override
    public void onMoveInChannel(String sourceChannelId, IPayload payload, IPayload outPayload) throws GameAnvilException {

}
}
```

```java
@GameAnvilController
public class _GameUserTest {
   // SampleGameUserで処理したいプロトコルとハンドラーをマッピング
    @GameUserMapping(
        value = MyGame.GameUserTest.class, // 処理するプロトコルバッファ
        loadClass = SampleGameUser.class   // メッセージを受け取る対象(SampleGameUser)
    )
    public void execute(IUserDispatchContext ctx) {
       // ここで行う作業を記述
    }
}
```

特に、ユーザーはエンジンに登録するための設定が他のクラスに比べて多く要求されます。まず、どのゲームサービスのためのユーザーなのか登録した後、ユーザータイプを登録します。このユーザータイプは名前の通りユーザーの種類を区別するための用途として、クライアントでも同様にAPIを呼び出す時に使用されます。つまり、該当APIがサーバーのどのユーザータイプに対する呼び出しかを明示するものです。したがって必ずサーバーとクライアント間にこのようなユーザータイプを任意の文字列で事前定義しておく必要があります。このサンプルコードでは「BasicUser」というユーザータイプを使用します。これについてのより詳細な説明は別の章で改めて扱います。最後にこのユーザー情報が[チャンネル間ユーザー情報同期](server-impl-09-channel.md#channel-user-information)に使用されるかどうかを決定できます。もしチャンネル間情報同期が必要なければ省略できます。

上記のサンプルコードでonLoginで始まるコールバックは全てログインに関連して呼び出されます。例えば最初のログインリクエストに対してはonLoginコールバックが呼び出され、すでにログインされている状態で再ログインを処理する場合はonReLoginコールバックが呼び出されます。同様にログアウトを処理する時にはonLogoutコールバックが呼び出されます。このようにGameAnvilのコールバックはその名前とJavaDoc注釈を通じてその用途が大部分明確に説明されます。

ユーザーはいつでもチャンネル間での移動が可能です。このようなチャンネルに関連した内容は後で[別の章](server-impl-09-channel)で個別に説明するので、ここではひとまず進むことにします。それだけでなくユーザーは複数のGameNode間で転送可能なオブジェクトです。これに関連した機能もまた[別の章](server-impl-08-object-transfer)でより詳しく説明します。

GameAnvilは2種類のマッチメイキング機能、ルームマッチメイキングとユーザーマッチメイキングを提供します。このようなマッチメイキングリクエストがユーザーに到達するとonMatchRoomあるいはonMatchUserコールバックが呼び出されます。ユーザーはこのコールバックでGameAnvilが提供するマッチメーカーを使用することもでき、直接実装したりサードパーティーから提供された他のマッチメーカーを使用することもできます。これについてのより詳細な説明はすぐ[次の章](server-impl-04-match-node)でMatchNodeを扱いながら改めて行います。

このようなユーザーのコールバックを整理すると以下の表の通りです。

| コールバック名                   | 意味                     | 説明                                                                                                                                                                                                                                                                |
|--------------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
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
| onLoginByOtherDevice     | 他のデバイスでのログイン試行     | すでにログインされている状態で他のデバイスから追加ログインリクエストが来た時に呼び出されます。ユーザーは既存のユーザーと新しいユーザーのどちらをログインさせるか戻り値で決定できます。                                                                             |
| onLoginByOtherUserType   | 他のユーザータイプでのログイン試行    | すでにログインされている状態で他のユーザータイプでログインリクエストが来た時に呼び出されます。新しいユーザータイプに対してログインを進めるかどうかを戻り値で決定できます。                                                                   |
| onLoginByOtherConnection | 他のコネクションでのログイン試行      | すでにログインされている状態で(再接続により) 以前とは異なるコネクションでログイン試行をする場合に呼び出されます。もし、再接続に対する追加作業が必要であればこのコールバックで行うことができます。                                                          |
| onMatchRoom              | ルームマッチメイキングリクエスト            | ユーザーがルームマッチメイキングをリクエストすると呼び出されます。この時、ユーザーはこのコールバックでエンジンが提供するルームマッチメイキングAPIあるいは第三のマッチメイキングソリューションを任意に使用できます。                                                               |
| onMatchRoomFail          | ルームマッチメイキングリクエスト失敗         | ユーザーがルームマッチメイキングをリクエストすると呼び出されます。                                                                                                                                      |
| onMatchUserFail          | ユーザーマッチメイキングリクエスト失敗        | ユーザーがユーザーマッチメイキングをリクエストすると呼び出されます。ユーザーはこのコールバックでエンジンが提供するユーザーマッチメイキングAPIあるいは第三のマッチメイキングソリューションを任意に使用できます。                                                               |
| onMatchUser              | ユーザーマッチメイキングリクエスト           | ユーザーがユーザーマッチメイキングをリクエストすると呼び出されます。ユーザーはこのコールバックでエンジンが提供するユーザーマッチメイキングAPIあるいは第三のマッチメイキングソリューションを任意に使用できます。                                                               |
| onMatchUserCancel        | ユーザーマッチメイキングキャンセル           | ユーザーが以前に申請したユーザーマッチメイキングをキャンセルすると呼び出されます。すでにマッチングが完了した状況のようにキャンセルできない場合には失敗することもあります。                                                                                 |
| onTransferOut            | 既存ノードから転送されて出ていく準備   | ユーザーが他のGameNodeへ転送される時、ソースノードで転送を開始する時に呼び出されます。ユーザーはこのコールバックでユーザーオブジェクトと共に転送するデータパッケージをまとめることができます。                                                               |
| onTransferIn             | 新しいノードへの転送完了処理     | ユーザーが他のGameNodeへ転送される時、対象ノードで転送完了しながら呼び出されます。ユーザーは一緒に持ってきたデータパッケージを展開して元のユーザー状態に復旧できます。                                                              |
| onAfterTransferIn        | 転送完了後処理             | ユーザー転送が成功した場合、対象ノードで後処理のために呼び出されます。                                                                                                                         |
| onSnapshot               | クライアントからスナップショットリクエスト        | クライアントからスナップショットリクエスト時に呼び出されます。主に接続が切れてサーバー状態が変わる確率がある場合に呼び出してクライアントとサーバー状態情報を同期するのに使用されます。                                                                    |
| canMoveOutChannel        | チャンネル移動が可能な状態か確認    | ユーザーが他のチャンネルへ移動できる状態かチェックするために呼び出されます。もし、ユーザーが明示的にmoveChannel() APIを呼び出してチャンネルを移動する場合には呼び出されません。ただエンジンにより暗黙的なチャンネル移動が発生する時のみ呼び出されます。               |
| onMoveOutChannel         | 既存チャンネルから他のチャンネルへ移動準備   | ユーザーが他のチャンネルへ移動する時、ソースノードで呼び出されます。ユーザーは希望する情報をoutPayloadに入れて対象チャンネルへ持っていくことができます。                                                                           |
| onAfterMoveOutChannel    | 既存チャンネルから他のチャンネルへ移動準備完了 | onMoveOutChannelが成功すれば後処理のために呼び出されます。                                                                                                                        |
| onMoveInChannel          | 新しいチャンネルへの移動処理         | ユーザーが他のチャンネルへ移動する時、対象ノードで呼び出されます。ユーザーは任意の情報をoutPayloadに入れてクライアントへ伝達できます。                                                            |

<a id="what-is-a-login"></a>
### ログインとは？ { #what-is-a-login }

先ほど説明した内容とサンプルコードでログインに関する内容が頻繁に登場します。また、このようなログインはクライアントがサーバーに接続した後、GameNodeに自身のユーザーオブジェクトを作る過程だと定義できます。コールバックメソッドのうちonLogin()は最初にユーザーを生成するためにログインを試みる過程で呼び出されます。この時、ユーザーはユーザーオブジェクトを構成するための情報をDBなどから獲得できます。このようなonLogin()コールバックが成功すればGameNode上に該当ユーザーオブジェクトが生成されます。このようにログインが完了すると、直接定義したプロトコルを基盤にクライアントは自身のユーザーオブジェクトを通じて他のオブジェクトとメッセージをやり取りしながら様々なコンテンツを実装できます。

<a id="logout"></a>
### ログアウト { #logout }

ログアウトはログインの反対概念です。つまり、GameNode上で自身のユーザーオブジェクトを除去する過程です。ログアウトを開始すると該当ユーザーオブジェクトはonLogout()コールバックを呼び出してメモリ上で削除される前にDBなどで自身の最終状態を保管できます。このようなログアウトはクライアントが明示的にリクエストすることもでき、クライアントの接続が切れた状態で任意の時間が経過した後、エンジンにより自動的に処理されたりもします。したがって、もしモバイルゲームのように頻繁な接続切れが予想される場合にはすぐにログアウトが進行されないよう適切な[設定](server-impl-16-config-vm.md#game)をしておくことができます。

<a id="implement-room"></a>
## ルーム実装 { #implement-room }

2名以上のユーザーはルームを通じて同期されたメッセージフローを作ることができます。つまり、ユーザーのリクエストはルームの中で全て順序が保証されます。もちろん1名のユーザーのためのルーム生成もコンテンツによっては意味を持つこともあります。ルームをどのように使用するかはあくまでエンジンユーザーの役割です。このようなルームはユーザーと同様に基本クラスであるIRoomインターフェースを実装して様々なコールバックメソッドを再定義でき、独自にメッセージを処理することもできます。以下のサンプルコードはSampleUserのためのSampleRoomクラスです。

```java
 @GameAnvilRoom(
    gameServiceName = "MyGame", // ルームが所属するノード（上記のSampleGameNodeと同じサービス名）
    gameType = "BasicRoom",     // ルーム固有のタイプ。"BasicRoom"というタイプのルームをエンジンに登録
    useChannelInfo = true       // チャンネル間の情報同期設定
)
public class SampleGameRoom extends BaseGameRoom<SampleGameUser> {

/**
     * ルームが初期化されるときに呼び出される
     */
    @Override
    public void onInit() {

}

/**
     * ルームが削除されるときに呼び出される
     */
    @Override
    public void onDestroy() {

}

/**
     * 新しいルームを作成するときに呼び出される
     * <p/>
     * 戻り値によってルームを作成するかどうかが決定される
     *
     * @param user       リクエストしたユーザーオブジェクト
     * @param inPayload  クライアントから受け取った {@link IPayload}
     * @param outPayload クライアントに渡す {@link IPayload}
     * @return 戻り値がtrueの場合はルーム作成成功、falseの場合は失敗
     */
    @Override
    public boolean onCreateRoom(SampleGameUser user, IPayload inPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

/**
     * 任意のルームに参加するときに呼び出される
     * <p/>
     * 戻り値によってルームに参加するかどうかが決定される
     *
     * @param user       リクエストしたユーザーオブジェクト
     * @param inPayload  クライアントから受け取った {@link IPayload}
     * @param outPayload クライアントに渡す {@link IPayload}
     * @return 戻り値がtrueの場合は入場成功、falseの場合は失敗
     */
    @Override
    public boolean onJoinRoom(SampleGameUser user, IPayload inPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

/**
     * ルームから退出するときに呼び出される
     * <p/>
     * 戻り値によってルームから退出するかどうかが決定される
     *
     * @param user       リクエストしたユーザーオブジェクト
     * @param inPayload  クライアントから受け取った {@link IPayload}
     * @param outPayload クライアントに渡す {@link IPayload}
     * @return 戻り値がtrueの場合は退出成功、falseの場合は失敗
     */
    @Override
    public boolean canLeaveRoom(SampleGameUser user, IPayload inPayload, IPayload outPayload) {
        return true;
    }

/**
     * ユーザーがルームから退出するときに呼び出される
     * <p/>
     * ユーザーがルームから退出する前の最後の処理を行う
     *
     * @param user ルームから退出するユーザー
     */
    @Override
    public void onLeaveRoom(SampleGameUser user) {

}

/**
     * ユーザーがルームから完全に退出した後に呼び出される
     * <p/>
     * ルームおよびルームに残っているユーザーに関する処理を行う
     */
    @Override
    public void onAfterLeaveRoom() {

}

/**
     * 再ログイン時に該当ルームへ自動的に再入場するときに呼び出される
     *
     * @param user       ルームに入るユーザーオブジェクト
     * @param outPayload クライアントに渡す {@link IPayload}
     */
    @Override
    public void onRejoinRoom(SampleGameUser user, IPayload outPayload) {

}

/**
     * 該当ルームが別のノードに移動（転送）する際、出発ノードから転送するデータをまとめるために呼び出される
     *
     * @param transferPack 別のノードに持っていくデータを保存するためのパッケージ {@link ITransferPack}
     */
    @Override
    public void onTransferOut(ITransferPack transferPack) {

}

/**
     * 該当ルームが別のノードに移動（転送）する際、対象ノードで新たに作成されたルームオブジェクトを元の状態に復元し、処理するタイマーハンドラーを登録するために呼び出される
     * <p>
     * TimerHandlerTransferPackを通じてルームに登録されていたtimerKeyのリストを確認する
     * <p/>
     * TimerHandlerTransferPackのreRegister()を活用して使用するtimerHandlerを再登録する
     * <p/>
     * この時点ではルームは完全に復元されていないため、他の場所（ノード、ユーザーなど）へのメッセージリクエストが制限される
     *
     * @param userList                 移動するユーザーリスト
     * @param transferPack             別のノードから持ってきたデータパッケージ {@link ITransferPack}
     * @param timerHandlerTransferPack タイマーハンドラーの再登録のための {@link ITimerHandlerTransferPack}
     */
    @Override
    public void onTransferIn(List<SampleGameUser> userList, IReadOnlyTransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

}

/**
     * 該当ルームが別のノードへの移動（転送）が完了した後に呼び出される
     */
    @Override
    public void onAfterTransferIn() {

}

/**
     * ルームが属するノードがPauseされるとき、該当ルームもPauseされながら呼び出される
     */
    @Override
    public void onPause() {

}

/**
     * ルームが属するノードがResumeされるとき、該当ルームもResumeされながら呼び出される
     */
    @Override
    public void onResume() {

}

/**
     * クライアントからパーティーマッチメイキングをリクエストした場合に呼び出される
     * <p/>
     * パーティーマッチメイキングには2名以上のユーザーがパーティータイプのネームドルームに入場する必要がある
     *
     * @param roomType      クライアントとサーバー間で事前定義したルーム種類を区別する任意の値
     * @param matchingGroup マッチンググループ
     * @param user          パーティーマッチメイキングをリクエストしたユーザー（ルームオーナー）
     * @param payload       クライアントから受け取った {@link IPayload}
     * @param outPayload    クライアントに渡す {@link IPayload}
     * @return 戻り値がtrueの場合はパーティーマッチメイキングリクエスト成功、falseの場合は失敗
     */
    @Override
    public boolean onMatchParty(String roomType, String matchingGroup, SampleGameUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

/**
     * パーティーマッチメイキングがキャンセルされるときに呼び出される
     *
     * @param reason キャンセルされた理由。タイムアウト（TIMEOUT）、ユーザーのリクエストによるキャンセル（CANCEL）、マッチノードの終了によるキャンセル（SHUTDOWN）
     */
    @Override
    public void onMatchPartyCancel(MatchCancelReason reason) {

}

/**
     * ルームマッチメイキングがキャンセルされるときに呼び出される
     *
     * @param reason キャンセルされた理由。（SHUTDOWN: マッチノードの終了によるキャンセル）
     */
    @Override
    public void onForceMatchRoomUnregistered(MatchCancelReason reason) {

}

/**
     * ルームが別のノードに移動（転送）可能な状態かどうかを確認するために呼び出される
     *
     * @return 戻り値がtrueの場合は転送可能な状態、falseの場合は不可能な状態
     * <p/>
     * 不可能な状態の場合、SafePauseが進行中であれば、SafePauseが終了するまで該当ルームを転送するために継続して呼び出される
     */
    @Override
    public boolean canTransfer() {
        return true;
    }
}
```

```java
// プロトコルバッファ MyGame.GameRoomTest の入力があったときに動作するメッセージ処理クラス
@GameAnvilController
public class _GameRoomTest {
    // SampleRoom で処理したいプロトコルとハンドラーをマッピング
    @GameRoomMapping(
        value = MyGame.GameRoomTest.class, // 処理するプロトコルバッファ
        loadClass = SampleRoom.class   // メッセージを受け取る対象 (SampleRoom) 
    )
    public void execute(IRoomDispatchContext ctx) {
        // ここで行う作業を記述
    }
}
```

ルームもやはりエンジンに登録するために様々な設定が必要です。まずどのゲームサービスのためのルームなのか登録した後、ルームタイプを登録します。このルームタイプは名前の通りルームの種類を区別するための用途として、クライアントでも同様にAPIを呼び出す時に使用されます。つまり、該当APIがサーバーのどのルームタイプに対する呼び出しかを明示するものです。したがって必ずサーバーとクライアント間にこのようなルームタイプを任意の文字列で事前に定義しておく必要があります。このサンプルコードでは「BasicRoom」というルームタイプを使用します。これについてのより詳細な説明は別の章で改めて扱います。最後にこのルーム情報が[チャンネル間ルーム情報同期](server-impl-09-channel.md#channel-room-information)に使用されるかどうかを決定できます。もしチャンネル間情報同期が必要なければ省略できます。

ルームは最も基本的な3つのコールバックであるonCreateRoom、onJoinRoom、onLeaveRoomを提供します。それぞれルームの生成、参加、そして退室について呼び出されます。ユーザーは該当コールバックで関連する機能を直接実装できます。例えば、ルームを生成しながらルームリーダーを指定し基本的なデータ構造を初期化できます。また任意のルーム参加リクエストに対してはルームのユーザーリストを更新したり各ユーザー間の状態同期などを実行できます。

このようなルームの生成と参加はクライアントの明示的リクエストだけでなく、マッチメイキングによりエンジンが自動的に処理することもあります。例えば定員が2名のゲームでユーザーAとユーザーBがマッチングされたなら、一人は自動的にルームを生成し、もう一人は該当ルームに参加することになります。この過程はユーザーAとユーザーBのリクエストではなくエンジンで自動的に処理するものです。この過程でももちろんonCreateRoomとonJoinRoomコールバックが呼び出されます。

このようなルームのコールバックを整理すると以下の表の通りです。

| コールバック名                        | 意味                | 説明                                                                                                                                                           |
|------------------------------|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
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
| canTransfer                  | ルーム転送が可能な状態か確認   | 該当ルームが他のノードへ転送されうる状態かチェックするために呼び出されます。もしルームでまだゲームがプレイ中だったり準備ができていない場合にはfalseを返して転送を遅らせることができます。falseを返した場合にはエンジンが任意の時間以後に持続的にこのコールバックを呼び出します。ちなみにルーム転送は無停止パッチを進行する時にのみ使用されます。 |

<a id="room-type"></a>
## ルーム種類 { #room-type }

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

<a id="channel"></a>
## チャンネル { #channel }

GameNodeはその用途に合わせて論理的にグループ化できます。このような論理グループを[チャンネル](server-impl-09-channel)といいます。例えば、GameNode 1と2を「Beginner」チャンネルにまとめ、GameNode 3と4を「Expert」チャンネルにまとめることができます。これについての詳細な説明は[別の章](server-impl-09-channel)でより詳しく扱います。