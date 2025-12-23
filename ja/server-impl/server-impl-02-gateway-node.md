## Game > GameAnvil > サーバー開発ガイド > ゲートウェイノード実装

## Gateway Node

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gatewaynode_on_network.png)

GatewayNodeはクライアントが接続する関門(Gateway)です。つまり、クライアントのコネクションとゲームサービスに対するセッションを管理します。この時、クライアントはゲームを進行するために必ずGatewayNodeに接続した後、認証を完了しなければなりません。これらの関係は次の図の通りです。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

一般的にクライアントはGatewayNodeと1つのコネクションを結びます。この時、該当コネクションに対して認証手続きを進め、成功した場合に限り1つ以上のセッションを作成できます。それぞれのセッションはクライアントとユーザー間の論理的接続単位です。上記の図はクライアントが1つのコネクションを通じてGameサービスとChatサービスでセッションを作成した様子です。このような構造は意図せずクライアントの接続が切れても、簡単に[セッション復旧 (Session Recovery)](#session-recovery)を可能にします。

### GatewayNode実装

このようなGatewayNodeは、@GameAnvilGatewayNodeアノテーションを宣言してエンジンに登録し、IGatewayNodeインターフェースを実装してコールバックメソッドのみをオーバーライドすれば済みます。これらの共通コールバックメソッドは、その名前が用途を明確に説明しています。
```java
@GameAnvilGatewayNode // エンジンにこのクラスをGatewayとして登録
public class SampleGatewayNode implements IGatewayNode {
    private IGatewayNodeContext gatewayNodeContext;

    /**
     * ゲートウェイノードコンテキストを伝達するために呼び出し
     * <p/>
     * オブジェクトが生成された後、一度呼び出される
     *
     * @param gatewayNodeContextゲートウェイノードコンテキスト
     */
    @Override
    public void onCreate(IGatewayNodeContext gatewayNodeContext) {
        this.gatewayNodeContext = gatewayNodeContext;
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


### Connection実装

コネクションはクライアントの物理的接続自体を意味します。クライアントは固有のAccountIdを利用してコネクション上で認証手続きを進めることができます。認証が成功した場合、該当AccountIdは作成されたコネクションにマッピングされます。

このようなコネクションは次のようにIConnectionを実装した後、コールバックメソッドを再定義します。この時、任意のプラットフォームで認証した後に獲得するユーザーのキー値などをAccountIdとして使用できます。例えばGamebaseを通じて認証した後UserIdを獲得すれば、この値をGameAnvilの認証過程でAccountIdとして使用できます。 

```java
@GameAnvilGatewayConnection // エンジンにこのクラスをConnectionとして登録
public class SampleConnection implements IConnection {
    private IConnectionContext connectionContext;
    
    /**
     * コネクションコンテキストを伝達するために呼び出し
     * <p/>
     * オブジェクトが生成された後、一度呼び出される
     *
     * @param connectionContextコネクションコンテキスト
     */
    @Override
    public void onCreate(IConnectionContext connectionContext) {
        this.connectionContext = connectionContext;
    }

    /**
     * 認証リクエスト時に呼び出し
     *
     * @param accountId アカウントID
     * @param password  アカウントパスワード
     * @param deviceId  クライアントのデバイスID
     * @param payload   クライアントから受け取った{@link IPayload}
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     * @return戻り値がtrueなら認証成功、falseならクライアントとの接続終了
     */
    @Override
    public boolean onAuthenticate(String accountId, String password, String deviceId, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * コネクションが属するノードがPauseになる時に呼び出し
     */
    @Override
    public void onPause() {

    }

    /**
     * コネクションが属するノードがResumeになる時に呼び出し
     */
    @Override
    public void onResume() {

    }

    /**
     * クライアントとの接続が切れた時に呼び出し
     */
    @Override
    public void onDisconnect() {

    }
}
```


このようなコールバックの意味と用途は以下の表を参考にしてください。

| コールバック名         | 意味     | 説明                                                                                                                                                      |
|----------------|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate       | オブジェクト生成  | オブジェクトが生成された時に呼び出されます。生成されたタイプで使用可能なAPIを使用できるコンテキストを受け取ります。コンテンツで必要であれば保存して使用できます。                                  |
| onAuthenticate | 認証     | クライアントがAuthentication() APIを使用してコネクションに対する認証をリクエストする時に呼び出されます。ユーザーはここでクライアントが送った認証情報をもとに認証処理を進めることができます。もし認証が成功すればtrueを返し、失敗すればfalseを返さなければなりません。 |
| onPause        | 一時停止  | コンソールを通じてGatewayNodeを一時停止すると、該当GatewayNodeの全てのコネクションに対して呼び出されます。ユーザーはノードが一時停止される時にコネクションで追加で処理したいコードをここに実装できます。                               |
| onResume       | 再開     | コンソールを通じてGatewayNodeが一時停止状態で駆動を再開すると、該当GatewayNodeの全てのコネクションに対して呼び出されます。ユーザーは再開状態でコネクションに対して処理したいコードをここに実装できます。                             |
| onDisconnect   | 接続終了  | クライアントから接続が切れた時に呼び出されます。この時、追加で処理するコードをここに実装します。                                                                                                  |

### Session実装

コネクションを正常に結んだクライアントは、該当コネクション間でサービスごとに1つずつGameNodeに対する論理的なセッションを結ぶことができます。GameAnvilは内部的にコネクションのAccountIdとセッションのSubIdを組み合わせて全体サーバーで固有のセッションを区分できます。

この時、SubIdはユーザーが任意に決めたルールに合わせて該当コネクション内の任意の固有な値として割り当てればよいです。つまり、別々のコネクションは同一のSubIdを持つこともあります。しかし別々のAccountIdを持つため区分が可能です。

```java
@GameAnvilGatewaySession // エンジンにこのクラスをSessionとして登録
public class SampleSession implements ISession {
    private ISessionContext sessionContext;

    /**
     * セッションコンテキストを伝達するために呼び出し
     * <p/>
     * オブジェクトが生成された後、一度呼び出される
     *
     * @param sessionContextセッションコンテキスト
     */
    @Override
    public void onCreate(ISessionContext sessionContext) {
        this.sessionContext = sessionContext;
    }

    /**
     * ログイン呼び出し以前に呼び出し
     *
     * @param outPayloadクライアントへ伝達する{@link IPayload}
     */
    @Override
    public void onBeforeLogin(IPayload payload) {

    }

    /**
     * ログイン成功以後に呼び出し
     */
    @Override
    public void onAfterLogin(boolean isReLogined) {

    }

    /**
     * ログアウト以後に呼び出し
     */
    @Override
    public void onAfterLogout() {

    }
}
```

このようなコールバックの意味と用途は以下の表を参考にしてください。


| コールバック名         | 意味      | 説明                                                                                                                                              |
|----------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate       | オブジェクト生成   | オブジェクトが生成された時に呼び出されます。生成されたタイプで使用可能なAPIを使用できるコンテキストを受け取ります。コンテンツで必要であれば保存して使用できます。                                       |
| onBeforeLogin  | ログイン前処理 | GameNodeにログインをリクエストする直前に呼び出されます。この時、ユーザーはパラメータとして渡された出力用ペイロード(outPayload)に任意の値を入れてログインリクエストに乗せて送ることができます。このペイロードはゲームノードでログインコールバックを処理する際にそのまま伝達されます。 |
| onAfterLogin   | ログイン後処理 | GameNodeにログインを完了した後に呼び出されます。ログイン完了後にセッションで処理するコードがあればここに実装します。                                                                        |
| onAfterLogout  | ログアウト後処理 | ログアウト処理が完了した後に呼び出されます。ログアウト以後にセッションで処理するコードがあればここに実装します。                                                                             |

## ConnectionとSession

クライアントはゲートウェイノードに接続します。つまり、コネクションを生成します。このコネクションを通じてアカウントとユーザー情報をもとに認証とログインを進めることができます。ログインまで完了すると任意のゲームノードにユーザーオブジェクトが生成されます。これはゲートウェイノードと該当ゲームノードの間に論理的なセッションが生成されたことを意味します。このようにコネクションとセッション生成が完了すると、該当ユーザーはゲーム進行が可能になります。これについてはすぐ後でゲームノードを説明する際にもう一度見ていくことにします。

### Session Recovery

もし、クライアントとゲートウェイノードの間に再接続が発生すると、下の図のようにセッション復旧(Session Recovery)が行われます。再接続をする過程でクライアントは複数台のゲートウェイノードのうち、以前とは異なる場所にコネクションを試みることもあります。この場合、ユーザーオブジェクトが存在するゲームノードに対する位置情報をもとに新しくセッションを復旧します。したがってユーザーはゲーム進行中に再接続をしても以前のゲーム状態を続けることができます。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

### Location Node

先ほど見てきたコネクション復旧の図でロケーションノードが見えます。ロケーションノードはGameAnvilが内部的にユーザーやルームなどの位置情報を管理するシステムノードです。ユーザーはロケーションノードについて直接実装したり使用することはできません。しかし位置情報を管理するロケーションノードの役割を理解することは、全体的なGameAnvilシステムの流れを理解するのに役立つため、ここで簡単に言及したいと思います。

上記のコネクション復旧を例に挙げてみます。クライアントが初回接続をしてゲームノードへログインを試みる過程で、これに関連するセッションとユーザーに対する位置情報は全てロケーションノードに保存されます。したがって再接続を行う場合には、以前の接続過程で保存しておいたこの位置情報を照会できます。このような位置情報はGameAnvil内部的にユーザーやルームの位置情報を照会し、これをもとにメッセージを伝達する用途などで非常に重要に使用されます。
