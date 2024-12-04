## Game > GameAnvil > サーバー開発ガイド > ゲームノードの実装



## Game Node

![GameNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gamenode_on_network.png)

GameNodeは、実際のゲームオブジェクトが作成され、ゲームコンテンツを実装するノードです。クライアントはGatewayNodeで認証を完了した後、GameNodeにログインして初めてこのようなゲームコンテンツを開始できます。

以下の画像を見て分かるように、クライアントは固有のAccountIdで認証を完了したコネクションをベースに複数の論理セッションを作成できます。この時、セッションはクライアントとユーザーオブジェクト間に作成されます。以下の図では、赤い点線の矢印がこのようなセッションを示しています。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

それぞれのセッションは、該当コネクション内で固有値で区別でき、この値をSubIdと呼んでいます。すなわち、AccountIdとSubIdの組み合わせにより、ユーザーは好きなだけセッションを作成できます。この時、同じサービスに対するセッションも同様に好きなだけ作成できます。つまり、画像でAccountIdが1であるコネクションは、"Game"サービスまたは、"Chat"サービスのセッションをいくらでも追加で作成できるということです。

これらのセッションが向かうのはユーザーオブジェクトです。GameNodeは、これらのユーザーオブジェクトとそれらのグループであるルームオブジェクトを管理します。このチャプターでは、このようなGameNodeとGameUser、そしてGameRoomについて説明します。


## GameNode

GameNodeは、BaseGameNodeクラスを継承して実装します。以下のサンプルコードはGameNodeで基本的に再定義できるコールバックメソッドを示しています。ノード共通のコールバックに加えて、チャンネル管理用のコールバックが存在します。

すべてのノードは、ユーザー定義のメッセージを処理するためのディスパッチャーの作成とメッセージハンドラ登録プロセスが必要です。特にGameNodeはゲームコンテンツのために、このようなプロセスが必須です。まず、(1)静的パケットディスパッチャーを1つ作成します。この時、必ずメモリや性能面で利点を得られるように静的(static)で作成します。(2)処理したいメッセージを実装しておいた[ハンドラ](server-impl-07-message-handling.md)と接続します。(3)最後に、MessageDispatcherで(1)で作成したディスパッチャーを利用してパケットを処理します。

これらの一連のプロセスは、以下のサンプルコードで(1)～(3)に該当する注釈のすぐ下にあるコードで調べることができます。

また、こうして実装したクラスを@ServiceNameアノテーションを使用して、特定のサービスに対する用途でエンジンに登録します。1つのGameNodeクラスは、1つのサービスにのみ登録できます。

```java
@ServiceName("MyGame") // "MyGame"というサービス用にGameNodeでエンジンに登録
public class SampleGameNode extends BaseGameNode {

    // (1)パケットディスパッチャーを作成 
    private static final MessageDispatcher<SampleGameNode> messageDispatcher = new MessageDispatcher<>();

    // (2) SampleGameNodeで処理したいプロトコルとハンドラをマッピング
    static {
        messageDispatcher.registerMsg(SampleGame.GameNodeTest.class、_GameNodeTest.class);
    }

    @Override
    public MessageDispatcher<SampleGameNode> getMessageDispatcher() {
        return messageDispatcher;
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onPrepare() throws SuspendExecution {
    }

    /**
     * pause呼び出し
     *
     * @param payload contentsから送信したい追加情報
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPause(PauseType type,Payload payload) throws SuspendExecution {
    }

    /**
     * resume呼び出し
     *
     * @param payload contentsから送信したい追加情報
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onResume(Payload payload) throws SuspendExecution {
    }
  
    /**
     * shutdownコマンドを受け取ると呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onShutdown() throws SuspendExecution {
    }

    /**
     * 同じチャンネルの他のノードでユーザーが変化した時に呼び出し
     * すなわち、updateChannelUser() APIを呼び出した時に発生
     *
     * @param type            Channel情報変更タイプ(更新/削除)を送信。
     * @param channelUserInfo変更されるUser情報を送信。
     * @param userId        変更対象のUser Idを送信。
     * @param accountId     変更対象のAccount Idを送信。
     * @throws SuspendExecution、このメソッドはファイバーがsuspendされることがある。
     */
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {  
    }

    /**
     * 同じチャンネルの他のノードでルームの状態が変化した時に呼び出し
     * すなわち、updateChannelRoomInfo() APIを呼び出した時に発生
     *
     * @param type            Channel情報変更タイプ(更新/削除)を送信。
     * @param channelRoomInfo変更されるRoom情報を送信。
     * @param roomId        変更対象のRoom Idを送信。
     * @throws SuspendExecution、このメソッドはファイバーがsuspendされることがある。
     */
    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType type, ChannelRoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {  
    }

    /**
     * クライアントからチャンネル情報をリクエストした時に呼び出し
     *
     * @param outPayload Clientに送信されるChannel情報を送信。
     * @throws SuspendExecution、このメソッドはファイバーがsuspendされることがある。
     */
    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {  
    }
}
```

これらのGameNodeの主な目的は、ノードに接続されたすべてのGameUserとGameRoomオブジェクトに対する処理を実行することです。これについては、次で説明します。

共通コールバックを除外したコールバックの意味と用途については、次の表を参照してください。


| コールバック名             | 意味                 | 説明                                                                                                                                                             |
| ------------------------- | ----------------------- |------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onChannelUserInfoUpdate | チャンネルのユーザー情報更新 | 同じチャンネルに結ばれた複数のGameNodeのうち、1つのGameNodeでチャンネルのユーザー情報が変更された場合、同じチャンネル内の残りの全GameNodeで同期するために呼び出されます。この時、ユーザーは受信した情報に基づいて現在のGameNodeチャンネル情報を更新できます。 |
| onChannelRoomInfoUpdate | チャンネルのルーム情報更新 | 同じチャンネルに結ばれた複数のGameNodeのうち、1つのGameNodeでチャンネルのルーム情報が変更された場合、同じチャンネル内の残りの全GameNodeで同期するために呼び出されます。この時、ユーザーは受信した情報に基づいて現在のGameNodeチャンネル情報を更新できます。  |
| onChannelInfo           | チャンネル情報リクエスト      | クライアントがチャンネル情報をリクエストした場合に呼び出されます。ユーザーは、このコールバックで好きなようにチャンネル情報を構成して、クライアントに送信できます。                                                                                     |



## ユーザー実装

ユーザーオブジェクトはログインプロセスを経て、GameNodeに作成されます。ユーザーベースのコンテンツは、このクラスを中心に実装することが望ましいです。前述したすべての例と同様に、ユーザーも処理する固有のメッセージとハンドラを接続できます。次のサンプルコードを見ると、ユーザーはかなり多くのコールバックメソッドを提供していることが分かります。これらの一部は、基本実装が提供されるため、特に必要な場合でない限り、再定義する必要はありません。これはユーザーだけでなく、エンジンが提供するほとんどのクラスに該当します。

特に、ユーザーはエンジンに登録するためのアノテーションも他のクラスに比べて多く要求します。まず、どのようなゲームサービスのユーザーなのか登録した後、ユーザータイプを登録します。このユーザータイプは、その名のとおりユーザーの種類を区別するためのもので、クライアントでも同様にAPIを呼び出す時に使用されます。すなわち、該当APIがサーバーのどのユーザータイプに対する呼び出しなのかを明示することです。そのため、必ずサーバーとクライアント間にこれらのユーザータイプを任意の文字列で事前に定義しておく必要があります。このサンプルコードでは、"BasicUser"というユーザータイプを使用します。これに関する詳細は別のチャプターで取り扱います。最後に、このユーザー情報が[チャンネル間のユーザー情報同期](server-impl-09-channel.md)に使用するかどうかを決定できます。チャンネル間の情報同期が必要でない場合は、@UseChannelInfoアノテーションを省略できます。

```java
@ServiceName("MyGame")
@UserType("BasicUser")
@UseChannelInfo
public class SampleGameUser extends BaseUser {

    private static final MessageDispatcher<SampleGameUser> messageDispatcher = new MessageDispatcher<>();

    static {
        messageDispatcher.registerMsg(SampleGame.GameUserTest.class, _GameUserTest.class);
    }

    @Override
    public MessageDispatcher<SampleGameUser> getMessageDispatcher() {
        return messageDispatcher;
    }

    /**
     * ログインする時に呼び出し
     *
     * @param payload      クライアントから送信された情報
     * @param sessionPayload onPreLoginから送信されたペイロード
     * @param outPayload   クライアントに送信する情報
     * @returnログインが成功した場合はtrueを返し、それ以外の場合はfalseを返す 
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onLogin(final Payload payload,
                           final Payload sessionPayload,
                           Payload outPayload) throws SuspendExecution {    
    }

    /**
     * ログイン成功後に必要な後処理用の呼び出し(すなわち、onLoginもしくはonReLoinが成功後に呼び出し)
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPostLogin() throws SuspendExecution {

    }

    /**
     * すでにログインした状況で異なるデバイスで同じユーザーがログインした場合に呼び出し
     *
     * @param newDeviceId          新たに接続したユーザーのdeviceId値
     * @param outPayloadForKickUserクライアントに送信するペイロード
     * @return戻り値がtrueの場合は、新たに接続したユーザーがログインした後、既存ユーザーは強制ログアウト処理。Falseの場合は、新たに接続したユーザーがログイン失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onLoginByOtherDevice(final String newDeviceId,
                                        Payload outPayloadForKickUser) throws SuspendExecution {
        return true;
    }

    /**
     * 任意のユーザータイプでログインした状態で、異なるユーザータイプでログインを試みた時に呼び出し
     *
     * @param userType  新たにログインを試みるユーザーのタイプ
     * @param outPayloadクライアントに送信するペイロード
     * @return戻り値がtrueの場合は、新しいログインが成功し、falseの場合は失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onLoginByOtherUserType(final String userType,
                                          Payload outPayload) throws SuspendExecution {
        return true;
    }

    /**
     * すでにログインした状態で(再接続などの理由により)異なるコネクションを通じて、ログインを試みた場合に呼び出し
     *
     * @param outPayloadクライアントに送信するペイロード
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {    
    }

    /**
     * すでにログインした状態で、再ログインを試みた時に呼び出し
     * (ログインした状態では、ユーザーのGameUserオブジェクトがGameNodeに変わらず有効な状態である)
     *
     * @param payload      クライアントから送信した任意のペイロード
     * @param sessionPayload onPreLoginから送信されたペイロード
     * @param outPayload   クライアントに送信する任意のペイロード
     * @return戻り値がtrueの場合はReLogin成功、falseの場合はReLogin失敗
     * @throws SuspendExecution:このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onReLogin(final Payload payload,
                             final Payload sessionPayload,
                             Payload outPayload) throws SuspendExecution {
    }

    /**
     * クライアントとの接続が切断された時に呼び出されるコールバック
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onDisconnect() throws SuspendExecution {    
    }

    /**
     * ユーザーが属するノードがpauseされた時、該当ユーザーもpauseされて呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPause() throws SuspendExecution {    
    }

    /**
     * ユーザーが属するノードがresumeされた時、該当ユーザーもresumeされて呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onResume() throws SuspendExecution {    
    }

    /**
     * ユーザーがlogoutした時に呼び出し
     *
     * @param payload  クライアントから送信されたペイロード
     * @param outPayloadクライアントに送信するペイロード
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onLogout(final Payload payload, Payload outPayload) throws SuspendExecution {
    }

    /**
     * 該当ユーザーがログアウト可能かどうかを確認するための呼び出し
     * エンジンユーザーは、このコールバックで現在のGameUserがログアウトしても問題がないか決定できる
     *
     * @return戻り値がfalseの場合は、ログアウトの進行が止まり、その後定期的に再度コールバックを呼び出す。戻り値がtrueの場合は、ログアウトを進行
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean canLogout() throws SuspendExecution {
    }

    /**
     * ルームマッチメイキングリクエストを受け取ると呼び出し
     *
     * @param roomTypeクライアントとサーバー間にあらかじめ定義したルームの種類を区別する任意の値
     * @param payloadクライアントから送信されたペイロード
     * @returnマッチングされたルームの情報返還を返す。Nullの場合は、クライアントのリクエストオプションによって新たにルームが作成されることや、失敗処理されることもある
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public MatchRoomResult onMatchRoom(final String roomType,
                                       final Payload payload) throws SuspendExecution {
        return null;
    }

    /**
     * クライアントでユーザーマッチメイキングリクエストした場合に呼び出されるコールバック
     *
     * @param roomType クライアントとサーバー間にあらかじめ定義したルームの種類を区別する任意の値
     * @param payload  クライアントから送信されたペイロード
     * @param outPayloadクライアントに送信するペイロード
     * @return戻り値がtrueの場合は、ユーザーマッチメイキングリクエストに成功し、falseの場合は失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onMatchUser(final String roomType,
                               final Payload payload,
                               Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * ユーザーマッチングがキャンセルされた時に呼び出し
     *
     * @param reasonキャンセルされた理由。通常、タイムアウト(TIMEOUT)やユーザーのリクエストによるキャンセル(CANCEL)のうちどちらか1つ。
     * @return戻り値がtrueの場合は、マッチングキャンセル処理に成功し、falseの場合はキャンセルが失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
        return false;
    }

    /**
     * 処理するタイマーハンドラを登録するためのコールバック
     * このコールバックが呼び出された場合、ユーザーは処理したいタイマーハンドラを好きなだけ登録
     * (詳細については"タイマー"チャプターを参照してください。)
     */
    @Override
    public void onRegisterTimerHandler() {    
    }

    /**
     * ユーザーが他のノードに移動(転送)可能な状態なのか確認するために呼び出し
     *
     * @returnの戻り値がtrueの場合は、転送可能な状態であり、falseの場合は不可能な状態である。不可能な場合、もしNonStopPatchが進行中であれば、パッチが終了するまでは該当ユーザーを転送するために、継続的に呼び出す
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean canTransfer() throws SuspendExecution {
    }
    
    /**
	 * ユーザーが他のノードに移動(転送)する時に、ソースノードから送信するデータを構築するために呼び出す
	 * (詳細については"転送可能なオブジェクト"チャプターを参照してください。)
     *
     * @param transferPack他のノードに移動させるデータを保存するためのパッケージ
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onTransferOut(final TransferPack transferPack) throws SuspendExecution {    
    }

    /**
     * ユーザーが他のノードに移動(転送)する時に、対象ノードで新たに作成されたユーザーオブジェクトを本来の状態に復元するために呼び出す
     * (詳細については"転送可能なオブジェクト"チャプターを参照してください。)
     * 
     * この時点ではユーザーが完全に復元していないため、他の場所(ノード、ユーザーなど)へのメッセージリクエストが制限される。
     *
     * @param transferPack他のノードから持ってきたデータパッケージ
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onTransferIn(final TransferPack transferPack) throws SuspendExecution {    
    }

    /**
     * ノード間のユーザー移動(転送)が完了した後、呼び出す
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPostTransferIn() throws SuspendExecution {    
    }

    /**
     * クライアントから他のチャンネルに移動リクエストをした時、現在のユーザーがチャンネル移動が可能な状態かどうかを確認するために呼び出す
	 *
	 * 注意>万が一、ユーザーが明示的にmoveChannel()APIを呼び出してチャンネルを移動する場合、onCheckMoveOutChannel()は呼び出されません。エンジンによって暗黙のチャンネル移動が発生した場合にのみ呼び出されます。
     *
     * @param destinationChannelId 移動対象チャンネルのID
     * @param payload            クライアントから送信されたペイロード
     * @param errorPayload       チャンネル移動に失敗した場合、サーバーからクライアントに送信するペイロード(成功した場合は送信されない)
     * @returnの戻り値がfalseの場合、チャンネル移動は不可能であるため、リクエストは失敗し、trueの場合は成功
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onCheckMoveOutChannel(final String destinationChannelId,
                                         final Payload payload,
                                         Payload errorPayload) throws SuspendExecution {
        return false;
    }

    /**
     * 他のノードにチャンネル移動する時、ソースノードから呼び出す
     *
     * @param destinationChannelId移動するchannel ID
     * @param outPayload         移動するchannelに送信するペイロード
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onMoveOutChannel(final String destinationChannelId,
                                 Payload outPayload) throws SuspendExecution {
    }

    /**
     * 他のノードにチャンネル移動が完了した後、ソースノードから呼び出す
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPostMoveOutChannel() throws SuspendExecution {
    }

    /**
     * 他のノードにチャンネル移動する時、対象ノードに進入して呼び出す
     *
     * @param sourceChannelId移動する前のチャンネルID
     * @param payload       クライアントから送信されたペイロード
     * @param outPayload    クライアントに送信するペイロード
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onMoveInChannel(final String sourceChannelId,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * 他のノードにチャンネル移動が完了した後、対象ノードから呼び出す
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPostMoveInChannel() throws SuspendExecution {
    }
}
```

上記のサンプルコードでonLoginから始まるコールバックは、すべてログインに関連して呼び出されます。例えば、初回のログインリクエストに対しては、onLoginコールバックが呼び出され、すでにログインしている状態で再ログイン処理する場合は、onReLoginコールバックが呼び出されます。同様にログアウト処理する場合は、onLogoutコールバックが呼び出されます。このように、GameAnvilのコールバックは、その名前とJavaDoc注釈により、その用途が明確になっています。

ユーザーは、いつでもチャンネル間での移動が可能です。このようなチャンネルに関する内容は、後で[別途のチャプター](server-impl-09-channel)で説明しますので、ここでは先に進みます。それだけでなく、ユーザーは複数のGameNode間で転送可能なオブジェクトです。これに関連する機能についても[別途のチャプター](server-impl-08-object-transfer)で詳しく説明します。

GameAnvilは、2種類のマッチメイキング機能、ルームマッチメイキングとユーザーマッチメイキングを提供します。これらのマッチメイキングリクエストがユーザーに到達すると、onMatchRoomまたは、onMatchUserコールバックが呼び出されます。ユーザーは、このコールバックでGameAnvilが提供するマッチメイカーを使用したり、直接実装したり、3rdパーティとして提供された他のマッチメイカーを使用したりすることもできます。これに関する詳細は、[次のチャプター](server-impl-04-match-node)でMatchNodeについて取り扱いながら、もう一度説明します。

これらのユーザーのコールバックを整理すると、次の図のようになります。


| コールバック名              | 意味                                   | 説明                                                                                                                                                    |
| -------------------------- | ----------------------------------------- |---------------------------------------------------------------------------------------------------------------------------------------------------------|
| onLogin                  | ログイン                                | 初めてログインする時に呼び出されます。通常、ユーザーはこのコールバックでDBなどのストレージからユーザー情報を取得して、ゲームユーザーオブジェクトを初期化する作業を実行します。                                                              |
| onPostLogin              | ログイン成功後処理                    | onLoginが成功した後に呼び出されます。ログインの後処理作業をこのコールバックで実行できます。                                                                                                 |
| onLoginByOtherDevice     | チャンネル情報リクエスト                        | すでにログインしている状態で、他のデバイスに追加ログインリクエストが来た時に呼び出されます。ユーザーは、既存のユーザーと新たなユーザーのどちらをログインさせるかを戻り値で決定できます。                                                   |
| onLoginByOtherUserType   | 他のユーザータイプでログイン試行         | すでにログインしている状態で、他のユーザータイプにログインリクエストが来た時に呼び出されます。新たなユーザータイプに対してログインを進めるかどうかを戻り値で決定できます。                                                         |
| onLoginByOtherConnection | 他のコネクションでログイン試行            | すでにログインしている状態で、(再接続によって)以前とは異なるコネクションでログインを試みた場合に呼び出されます。もし、再接続に対する追加作業が必要な場合は、このコールバックで行うことができます。                                               |
| onReLogin                | 再ログイン                              | すでにログインしている状態で再ログインした場合は、onLoginではなくonReLoginが呼び出されます。すなわち、再ログインに対する作業は、このコールバックで処理します。                                                           |
| onDisconnect             | 接続終了                             | クライアントから接続が切断された時に呼び出されます。この時、追加で処理するコードをここに実装します。                                                                                                   |
| onPause                  | 一時停止                             | コンソールを介してGameNodeを一時停止すると、該当GameNodeのすべてのユーザーに呼び出されます。ユーザーは、ノードが一時停止した時にユーザーが追加で処理したいコードをここに実装できます。                                           |
| onResume                 | 再開                                  | コンソールを介してGameNodeが一時停止状態から動作を再開すると、該当GameNodeのすべてのユーザーに呼び出されます。ユーザーは再開状態でユーザーに対して処理したいコードをここに実装できます。                                  |
| onLogout                 | ログアウト                             | ユーザーがログアウトした時に呼び出されます。これは、ユーザーが明示的にリクエストしたログアウトである可能性もあり、接続が切断された状態で設定された時間を超過した場合、エンジンによって自動的にログアウトすることもあります。                                                 |
| canLogout                | ログアウト可能性確認                  | 該当ユーザーが現在ログアウトが可能な状態であるかをチェックするために呼び出されます。もしゲームプレイ中や情報を失ってはならない状況である場合は、falseを返してログアウトを延期できます。falseを返した場合、エンジンは任意の時間が経過した後、継続的にコールバックを呼び出します。 |
| onMatchRoom              | ルームマッチメイキングリクエスト                    | ユーザーがルームマッチメイキングをリクエストすると、呼び出されます。この時、ユーザーはこのコールバックでエンジンが提供するルームマッチメイキングAPIもしくは、第3のマッチメイキングソリューションを任意で使用できます。                                                        |
| onMatchUser              | ユーザーマッチメイキングリクエスト                  | ユーザーがユーザーマッチメイキングをリクエストすると、呼び出されます。ユーザーは、このコールバックでエンジンが提供するユーザーマッチメイキングAPIもしくは、第3のマッチメイキングソリューションを任意で使用できます。                                                          |
| onMatchUserCancel        | ユーザーマッチメイキングキャンセル                  | ユーザーが申請したユーザーマッチメイキングをキャンセルすると、呼び出されます。すでにマッチングが完了した場合など、キャンセルできない場合は失敗することもあります。                                                                         |
| canTransfer              | ユーザー転送が可能な状態であるかを確認       | 該当ユーザーが他のノードに転送可能な状態であるかをチェックするために呼び出されます。もしゲームプレイ中やまだ準備が完了していない場合は、falseを返して転送を延期できます。 falseを返した場合、エンジンが任意の時間が経過した後、継続的にこのコールバックを呼び出します。 |
| onTransferOut            | 既存ノードから転送される準備       | ユーザーが他のGameNodeに転送される時、ソースノードで転送を開始する時に呼び出されます。ユーザーは、このコールバックでユーザーオブジェクトと共に転送するデータパッケージを構築できます。                                                         |
| onTransferIn             | 新たなノードに転送完了処理           | ユーザーが他のGameNodeに転送される時に対象ノードからの転送が完了すると、呼び出されます。ユーザーは共にインポートされたデータパッケージを展開して、元のユーザー状態に復元できます。                                                       |
| onPostTransferIn         | 転送完了の後処理                      | ユーザーの転送に成功した場合、対象ノードで後処理のために呼び出されます。                                                                                                                   |
| onCheckMoveOutChannel    | チャンネル移動が可能な状態であるかを確認       | ユーザーが他のチャンネルに移動できる状態かどうかをチェックするために呼び出されます。ユーザーが明示的にmoveChannel()APIを呼び出して、チャンネルを移動する場合は呼び出されません。エンジンによって、暗黙のチャンネル移動が発生した場合にのみ呼び出されます。             |
| onMoveOutChannel         | 既存チャンネルから他のチャンネルへの移動準備    | ユーザーが他のチャンネルに移動する時にソースノードから呼び出されます。ユーザーは希望する情報をoutPayloadに入れて、対象チャンネルに移動させることができます。                                                                 |
| onPostMoveOutChannel     | 既存チャンネルから他のチャンネルへの移動準備完了 | onMoveOutChannelが成功すると、後処理のために呼び出されます。                                                                                                                   |
| onMoveInChannel          | 新しいチャンネルへの移動処理                | ユーザーが他のチャンネルに移動する時に対象ノードから呼び出されます。ユーザーは任意の情報をoutPayloadに入れて、クライアントに送信できます。                                                                        |
| onPostMoveInChannel      | 新しいチャンネルへの移動完了                | onMoveInChannelが成功すると、後処理のために呼び出されます。                                                                                                                    |
| getMessageDispatcher | 処理するパケット有り | 処理するメッセージがある場合に返します。ユーザーは、自分が宣言したディスパッチャーを使用できます。詳細については、[メッセージ処理](./server-impl-07-message-handling#13-getMessageDispatcher)を参照してください。            |

### ログインとは？

前述した内容とサンプルコードでログインに関する内容が頻繁に登場します。また、このようなログインは、クライアントがサーバーに接続した後、GameNodeに自分のユーザーオブジェクトを作る過程だと定義できます。コールバックメソッドの中でonLogin()は、最初にユーザーを作成するためにログインを試みる過程で呼び出されます。この時、ユーザーはユーザーオブジェクトを構成するための情報をDBなどから取得できます。このようなonLogin()コールバックが成功すると、GameNode上に該当ユーザーオブジェクトが作成されます。このようにログインが完了すると、直接定義したプロトコルに基づいてクライアントは自分のユーザーオブジェクトを通じて、他のオブジェクトとメッセージをやり取りすることでさまざまなコンテンツを実装できます。

### ログアウト
ログアウトはログインの反対概念です。すなわち、GameNode上で自分のユーザーオブジェクトを削除するプロセスです。ログアウトを開始すると、該当ユーザーオブジェクトは、onLogout()コールバックを呼び出して、メモリ上で削除される前にDBなどに自分の最終状態を保管できます。このようなログアウトは、クライアントが明示的にリクエストすることも可能で、クライアントの接続が切断された状態で任意の時間が経過した後、エンジンによって自動的に処理されることもあります。したがって、モバイルゲームのように頻繁な接続の切断が予想される場合は、すぐにログアウトが行われないように適切な[設定](server-impl-16-config-vm.md#_game)をしておくことができます。



## ルーム実装

2人以上のユーザーは、ルームを通じて同期されたメッセージの流れを作成できます。すなわち、ユーザーのリクエストはルーム内ですべて順番が保証されます。もちろん、1人のユーザーのためのルーム作成もコンテンツによって、意味を持つことができます。ルームをどのように使用するかは、あくまでエンジンユーザー次第です。このようなルームは、ユーザーと同様に基本クラスであるBaseRoomを継承して、さまざまなコールバックメソッドを再定義でき、自らメッセージを処理することもできます。次のサンプルコードは、SampleGameUserのためのSampleGameRoomクラスです。

ルームもエンジンに登録するためには、さまざまなアノテーションが必要です。まず、どのようなゲームサービス用のルームなのかを登録した後、ルームタイプを登録します。このルームタイプは、その名のとおり部屋の種類を区別するためのものであり、クライアントでも同様にAPIを呼び出す時に使用されます。すなわち、該当APIがサーバーのどのルームタイプへの呼び出しであるのかを明示することです。したがって、必ずサーバーとクライアント間にこれらのルームタイプを任意の文字列であらかじめ定義しておく必要があります。このサンプルコードでは、"BasicRoom"というルームタイプを使用します。これについての詳細は、別のチャプターで再度扱います。最後に、このルーム情報が[チャンネル間のルーム情報同期](server-impl-09-channel.md)に使用するかどうかを決定できます。チャンネル間の情報同期が不要な場合、@UseChannelInfoアノテーションは省略できます。

```java
@ServiceName("MyGame")
@RoomType("BasicRoom")
@UseChannelInfo
public class SampleGameRoom extends BaseRoom<SampleGameUser> {

    private static final RoomMessageDispatcher<SampleGameRoom, SampleGameUser> messageDispatcher = new RoomMessageDispatcher<>();

    static {
        messageDispatcher.registerMsg(SampleGame.GameRoomTest.class, _GameRoomTest.class);
    }

    @Override
    public RoomMessageDispatcher<SampleGameRoom, SampleGameUser> getMessageDispatcher() {
        return messageDispatcher;
    }

    /**
     * ルームが初期化される時に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onInit() throws SuspendExecution {    
    }

    /**
     * ルームがサーバーから消えた時に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onDestroy() throws SuspendExecution {    
    }

    /**
     * 新しいルームを作成する時に呼び出し
     *
     * @param user     リクエストしたユーザーオブジェクト
     * @param inPayloadクライアントから送信されたペイロード
     * @param outPayloadクライアントに送信するペイロード
     * @returnの戻り値がtrueの場合は、ルームの作成に成功し、falseの場合は失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onCreateRoom(SampleGameUser user,
                                final Payload inPayload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * 任意のルームに参加する時に呼び出し
     *
     * @param user     リクエストしたユーザーオブジェクト
     * @param inPayloadクライアントから送信されたペイロード
     * @param outPayloadクライアントに送信するペイロード
     * @return戻り値がtrueの場合は、入場に成功し、falseの場合は失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onJoinRoom(SampleGameUser user,
                              final Payload inPayload,
                              Payload outPayload) throws SuspendExecution {
    }

    /**
     * ルームから退出する時に呼び出し
     *
     * @param user     リクエストしたユーザーオブジェクト
     * @param inPayloadクライアントから送信されたペイロード
     * @param outPayloadクライアントに送信するペイロード
     * @return戻り値がtrueの場合は、退出に成功し、falseの場合は失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onLeaveRoom(SampleGameUser user,
                               final Payload inPayload,
                               Payload outPayload) throws SuspendExecution {
    }

    /**
     * onLeaveRoomコールバックが成功した場合、退出完了後、後処理のために呼び出し
     *
     * @param userルームから退出したユーザーオブジェクト
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPostLeaveRoom(SampleGameUser user) throws SuspendExecution {
    }

    /**
     * ユーザーがルームに参加した状態で接続切断などによって、再接続および再ログインした場合、該当ルームに自動的に再入場して呼び出し
     *
     * @param user      ルームに入場するユーザーオブジェクト
     * @param outPayloadクライアントに送信するペイロード
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onRejoinRoom(SampleGameUser user, Payload outPayload) throws SuspendExecution {    
    }

   /**
     * 処理するタイマーハンドラを登録するためのコールバック
     * このコールバックが呼び出された場合、ユーザーは処理したいタイマーハンドラを好きなだけ登録します。
     * (詳細については"タイマー"チャプターを参照してください。)
     */
    @Override
    public void onRegisterTimerHandler() {
    }


    /**
     * ルームが他のノードに移動(転送)可能な状態であるかどうかを確認するために呼び出し
     *
     * @returnの戻り値がtrueの場合は、転送可能な状態であり、falseの場合は不可能な状態である。不可能な場合、NonStopPatchが進行中であれば、パッチが終了する前に該当ルームを転送するために継続的に呼び出し
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean canTransfer() throws SuspendExecution {    
    }
  
    /**
	 * 該当ルームが他のノードに移動(転送)する時、ソースノードから送信するデータを構築するために呼び出し
	 * (詳細については「転送可能なオブジェクト」チャプターを参照してください。)
     *
     * @param transferPack他のノードに移動させるデータを保存するためのパッケージ
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * 該当ルームが他のノードに移動(転送)する時、対象ノードで新たに作成されたルームオブジェクトを元の状態に復元するために呼び出し
     * (詳細については"転送可能なオブジェクト"チャプターを参照してください。)
     * 
     * この時点ではルームは完全に復元していないため、他の場所(ノード、ユーザーなど)へのメッセージリクエストが制限される。
     *
     * @param transferPack他のノードから取得したデータパッケージ
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onTransferIn(List<SampleGameUser> userList,
                             final TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * ルームが属するノードがpauseされた時、該当ルームもpauseされて呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPause() throws SuspendExecution {
    }

    /**
     * 部屋が属するノードがresumeされた時、該当ルームもresumeされて呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * クライアントがパーティマッチメイキングをリクエストした場合、呼び出されるコールバック
     * (パーティマッチメイキングのためには、2人以上のユーザーがNamedRoomに入場していなければならない。この時、該当NamedRoomでこのコールバックが呼び出される。)
     *
     * @param roomType クライアントとサーバー間にあらかじめ定義したルームの種類を区別する任意の値
     * @param user     パーティマッチングをリクエストしたユーザー(ルームリーダー)
     * @param payload  クライアントから送信されたペイロード
     * @param outPayloadクライアントに送信するペイロード
     * @return戻り値がtrueの場合は、パーティマッチメイキングリクエストが成功し、falseの場合は失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */  
    @Override
    public boolean onMatchParty(final String roomType,
                                final SampleGameUser user,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
        return false;
    }
}
```

ルームは最も基本的な3つのコールバックであるonCreateRoom、onJoinRoom、onLeaveRoomを提供します。それぞれ、ルームの作成、参加、そして退出に対して呼び出されます。ユーザーは該当コールバックで関連する機能を直接実装できます。例えば、ルームを作成してルームリーダーを指定し、基本的なデータ構造を初期化できます。また、任意のルームへの参加リクエストに対しては、ルームのユーザーリスト更新や、各ユーザー間での状態同期を実行できます。

このようなルームの作成と参加は、クライアントの明示的なリクエストだけでなく、マッチメイキングによってエンジンが自動的に処理することもあります。例えば、定員が2人のゲームでユーザーAとユーザーBがマッチングした場合、1人が自動的にルームを作成し、もう1人はそのルームに参加することになります。このプロセスは、ユーザーAとユーザーBのリクエストではなく、エンジンが自動的に処理するものです。このプロセスでももちろん、onCreateRoomとonJoinRoomコールバックが呼び出されます。

これらのルームのコールバックを整理すると、次の表のようになります。


| コールバック名            | 意味                            | 説明                                                                                                                                                                                        |
| ------------------------ | ---------------------------------- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onInit                 | 初期化                         | ルームが作成された時に初期化するために呼び出されます。トピックの登録など、該当ルームの初期化コードを作成できます。                                                                                                                                 |
| onDestroy              | ルームの消滅                       | ルームから最後のユーザーが退出し、それ以上処理するメッセージが存在しない場合、該当ルームは消滅します。この時に呼び出されるコールバックです。                                                                                                                    |
| onCreateRoom           | ルーム作成                        | クライアントがルームの作成をリクエストすると呼び出されます。コンテンツで使用するユーザーリスト用のデータ構造の作成や、他のルームの作成と一緒に処理される必要があるコードを作成します。                                                                                               |
| onJoinRoom             | ルーム参加                       | クライアントがサーバー上に存在する任意のルームに参加をリクエストすると呼び出されます。コンテンツで使用するユーザーリストの更新や、ルーム内のユーザー間に同期する情報を処理できます。                                                                                                |
| onLeaveRoom            | ルーム退出                         | クライアントがルームからの退出をリクエストすると呼び出されます。コンテンツで使用するユーザーリスト用のデータ構造の更新や、他のルームからの退出と一緒に処理される必要があるコードを作成します。                                                                                           |
| onPostLeaveRoom        | ルーム退出の後処理                 | onLeaveRoomが成功すると、後処理のために呼び出されます。ルーム退出直後に処理するコードを作成します。                                                                                                                                   |
| onRejoinRoom           | ルームへの再入場                    | ユーザーがルームに入場している状態で接続切断などにより再ログイン(ReLogin)した場合、エンジンによって自動的に該当ルームに再入場(ReJoin)します。この時、呼び出されるコールバックです。再入場過程で同期に必要な情報などを処理できます。                                                                |
| canTransfer            | ルームの転送が可能な状態であるかを確認 | 該当ルームが他のノードに転送できる状態であるかどうかをチェックするために呼び出されます。もしルームでゲームプレイ中や、まだ準備が完了していない場合は、falseを返して転送を延期できます。falseを返した場合は、エンジンが任意の時間が経過した後、継続的にこのコールバックを呼び出します。また、ルームの転送は、無点検パッチを行う時にのみ使用されます。           |
| onTransferOut          | 既存ノードから転送される準備 | ルームが他のGameNodeに転送される時、ソースノードから転送を開始する際に呼び出されます。ユーザーは、このコールバックからルームオブジェクトと共に転送するデータパッケージを構築できます。また、ルームの転送はルーム内の各ユーザーへのユーザー転送が含まれます。                                                        |
| onTransferIn           | 新しいノードへの転送完了処理   | ルームが他のGameNodeに転送される時、対象ノードからの転送が完了すると呼び出されます。ユーザーは共にインポートされたデータパッケージを展開して、元のルームの状態に復元できます。また、ルームの転送はルーム内の各ユーザーへのユーザー転送が含まれます。                                                            |
| onPause                | 一時停止                      | コンソールを介してGameNodeを一時停止すると、該当GameNodeのすべてのルームに呼び出されます。ユーザーは、ノードが一時停止した時にルームで追加で処理したいコードをここに実装できます。                                                                                         |
| onResume               | 再開                           | コンソールを介してGameNodeが一時停止状態から動作を再開すると、該当GameNodeのすべてのルームに呼び出されます。ユーザーは再開状態でルームに対して処理したいコードをここに実装できます。                                                                                        |
| onMatchParty           | パーティマッチメイキングリクエスト           | ユーザーがパーティマッチメイキングをリクエストすると呼び出されます。パーティマッチメイキングは、任意のNamedRoomをパーティ用途で作成した後、ルーム内のすべてのユーザーが1つのパーティでマッチングをリクエストする機能です。ユーザーは、このコールバックでエンジンが提供するパーティマッチメイキングAPIもしくは、第3のマッチメイキングソリューションを任意で使用できます。 |
| getMessageDispatcher | 処理するパケット有り | ノードに処理するメッセージがある場合に返します。ユーザーは、自分が宣言したディスパッチャーを使用できます。詳細については[メッセージ処理](server-impl-07-message-handling.md)を参照してください。                                                                       |

## ルームの種類

前述したルームの実装方法と別に、エンジンが提供するルームの種類は、大きく分けて2種類あります。この2つのルームを計4つの方法で使用します。

| ルームの種類   | 説明                                                       |
| ----------- | ------------------------------------------------------------ |
| Normal Room | クライアントが明示的にCreateRoomをリクエストして作成します。他のクライアントも該当ルームのIDを利用し、明示的にJoinRoomすることで参加できます。したがって、参加するユーザーは、事前に該当ルームIDを共有してもらう必要があります。 |
| Named Room  | NamedRoomは、その名のとおり唯一のルーム名を中心に命名されたルーム(NamedRoom)に対して動作を実行します。クライアントはサーバー群内で唯一の名を持つルームとしてNamedRoomをリクエストします。この時、もしサーバーに該当ルーム名が存在しない場合、リクエスト者は直接ルームを作成します。一方で、もしサーバーに該当ルーム名がすでに存在する場合は、そのルームに自動的に入場します。例えば、スタークラフトのカスタムゲームリストに表示される"3：3ハンター初心者！"のようなルームタイトルを考えると、理解しやすいです。 |

この2つのルームは、計4つの方法で使用されます。

| ルームの種類   | 使用方法                                                     |
| ----------- | ------------------------------------------------------------ |
| Normal Room | 1. クライアントはCreateRoom / JoinRoomリクエストによって作成および参加します。<br>2.ルームマッチメイキングによって、NormalRoomを作成し、参加できます。また、CreateRoomで作成したルームもルームマッチメイキングの対象として登録可能です。この時、ルームIDはエンジンによって管理され、マッチングしたルームとユーザー間で自動的に共有されます。 |
| Named Room  | 3. クライアントはNamedRoomリクエストによって作成および参加します。<br>4.ユーザーマッチメイキングによって、NamedRoomを作成し、参加できます。この時、作成されるNamedRoomのルーム名はエンジンが作成し、管理します。ルームマッチメイキングとは異なり、一般的なNamedRoomとして作成したルームはユーザーマッチメイキングの対象になれません。ただし、パーティマッチメイキングのために、NamedRoomとしてパーティルームを作成した後、複数のユーザーが1つのパーティでマッチメイキングをリクエストできます。 |



## チャンネル

GameNodeは、その用途に合わせて論理的にグループ化できます。これらの論理グループを[チャンネル](server-impl-09-channel)といいます。例えば、GameNode 1と2を"Beginner"チャンネルで結び、GameNode 3と4を"Expert"チャンネルで結ぶことができます。これに関する詳細は[別途のチャプター](server-impl-09-channel)で再度扱います。
