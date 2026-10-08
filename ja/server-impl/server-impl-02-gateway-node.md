<!-- machine_translated: true -->

<!-- pre-align:aligned sig=4435e036d920 -->

<a id="game-gameanvil-server-development-guide-implement-gateway-node"></a>
## Game > GameAnvil > サーバー開発ガイド > ゲートウェイノード実装 { #game-gameanvil-server-development-guide-implement-gateway-node }

<a id="gateway-node"></a>
## Gateway Node { #gateway-node }

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gatewaynode_on_network.png)

GatewayNodeはクライアントが接続する関門(Gateway)です。つまり、クライアントのコネクションとゲームサービスに対するセッションを管理します。この時、クライアントはゲームを進行するために必ずGatewayNodeに接続した後、認証を完了しなければなりません。これらの関係は次の図の通りです。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

一般的にクライアントはGatewayNodeと1つのコネクションを結びます。この時、該当コネクションに対して認証手続きを進め、成功した場合に限り1つ以上のセッションを作成できます。それぞれのセッションはクライアントとユーザー間の論理的接続単位です。上記の図はクライアントが1つのコネクションを通じてGameサービスとChatサービスでセッションを作成した様子です。このような構造は意図せずクライアントの接続が切れても、簡単に[セッション復旧 (Session Recovery)](#session-recovery)を可能にします。

<a id="implement-gatewaynode"></a>
### GatewayNode実装 { #implement-gatewaynode }

このようなGatewayNodeは、@GameAnvilGatewayNodeアノテーションを宣言してエンジンに登録し、BaseGatewayNodeクラスを実装してコールバックメソッドのみをオーバーライドすれば済みます。これらの共通コールバックメソッドは、その名前が用途を明確に説明しています。
```java
@GameAnvilGatewayNode // このクラスをGatewayとしてエンジンに登録
public class SampleGatewayNode extends BaseGatewayNode {
 
    /**
     * ノードが初期化されるときに呼び出される
     */
    @Override
    public void onInit() {

    }

    /**
     * Readyになる前に処理する部分のために呼び出される
     */
    @Override
    public void onPrepare() {

    }

    /**
     * Readyになるときに呼び出される
     */
    @Override
    public void onReady() {

    }

    /**
     * Shutdownコマンドを受け取ると呼び出される
     */
    @Override
    public void onShuttingdown() {

    }
}
```


<a id="implement-connection"></a>
### Connection実装 { #implement-connection }

コネクションはクライアントの物理的接続自体を意味します。クライアントは固有のAccountIdを利用してコネクション上で認証手続きを進めることができます。認証が成功した場合、該当AccountIdは作成されたコネクションにマッピングされます。

このようなコネクションは次のようにBaseConnectionを実装した後、コールバックメソッドを再定義します。この時、任意のプラットフォームで認証した後に獲得するユーザーのキー値などをAccountIdとして使用できます。例えばGamebaseを通じて認証した後UserIdを獲得すれば、この値をGameAnvilの認証過程でAccountIdとして使用できます。 

```java
@GameAnvilGatewayConnection // このクラスをConnectionとしてエンジンに登録
public class SampleConnection extends BaseConnection {

    /**
     * 認証リクエスト時に呼び出される
     *
     * @param accountId  アカウントID
     * @param password   アカウントパスワード
     * @param deviceId   クライアントのデバイスID
     * @param payload    クライアントから受け取った {@link IPayload}
     * @param outPayload クライアントへ送信する {@link IPayload}
     * @return 戻り値がtrueの場合は認証成功、falseの場合はクライアントとの接続を切断
     */
    @Override
    public boolean onAuthenticate(String accountId, String password, String deviceId, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }
}
```


このようなコールバックの意味と用途は以下の表を参考にしてください。

| コールバック名         | 意味     | 説明                                                                                                                                                      |
|----------------|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onAuthenticate | 認証     | クライアントがAuthentication() APIを使用してコネクションに対する認証をリクエストする時に呼び出されます。ユーザーはここでクライアントが送った認証情報をもとに認証処理を進めることができます。もし認証が成功すればtrueを返し、失敗すればfalseを返さなければなりません。 |

<a id="perform-session"></a>
### Session実装 { #perform-session }

コネクションを正常に結んだクライアントは、該当コネクション間でサービスごとに1つずつGameNodeに対する論理的なセッションを結ぶことができます。GameAnvilは内部的にコネクションのAccountIdとセッションのSubIdを組み合わせて全体サーバーで固有のセッションを区分できます。

この時、SubIdはユーザーが任意に決めたルールに合わせて該当コネクション内の任意の固有な値として割り当てればよいです。つまり、別々のコネクションは同一のSubIdを持つこともあります。しかし別々のAccountIdを持つため区分が可能です。

```java
@GameAnvilGatewaySession  // このクラスをSessionとしてエンジンに登録
public class SampleSession extends BaseSession {

    /**
     * ログイン呼び出し前に呼び出される
     *
     * @param outPayload クライアントに渡す {@link IPayload}
     */
    @Override
    public void onBeforeLogin(IPayload payload) {

    }

    /**
     * ログイン成功後に呼び出される
     */
    @Override
    public void onAfterLogin(boolean isReLogined) {

    }

    /**
     * ログアウト後に呼び出される
     */
    @Override
    public void onAfterLogout() {

    }
}
```

このようなコールバックの意味と用途は以下の表を参考にしてください。


| コールバック名         | 意味      | 説明                                                                                                                                              |
|----------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| onBeforeLogin  | ログイン前処理 | GameNodeにログインをリクエストする直前に呼び出されます。この時、ユーザーはパラメータとして渡された出力用ペイロード(outPayload)に任意の値を入れてログインリクエストに乗せて送ることができます。このペイロードはゲームノードでログインコールバックを処理する際にそのまま伝達されます。 |
| onAfterLogin   | ログイン後処理 | GameNodeにログインを完了した後に呼び出されます。ログイン完了後にセッションで処理するコードがあればここに実装します。                                                                        |
| onAfterLogout  | ログアウト後処理 | ログアウト処理が完了した後に呼び出されます。ログアウト以後にセッションで処理するコードがあればここに実装します。                                                                             |

<a id="connection-and-session"></a>
## ConnectionとSession { #connection-and-session }

クライアントはゲートウェイノードに接続します。つまり、コネクションを生成します。このコネクションを通じてアカウントとユーザー情報をもとに認証とログインを進めることができます。ログインまで完了すると任意のゲームノードにユーザーオブジェクトが生成されます。これはゲートウェイノードと該当ゲームノードの間に論理的なセッションが生成されたことを意味します。このようにコネクションとセッション生成が完了すると、該当ユーザーはゲーム進行が可能になります。これについてはすぐ後でゲームノードを説明する際にもう一度見ていくことにします。

<a id="session-recovery"></a>
### Session Recovery { #session-recovery }

もし、クライアントとゲートウェイノードの間に再接続が発生すると、下の図のようにセッション復旧(Session Recovery)が行われます。再接続をする過程でクライアントは複数台のゲートウェイノードのうち、以前とは異なる場所にコネクションを試みることもあります。この場合、ユーザーオブジェクトが存在するゲームノードに対する位置情報をもとに新しくセッションを復旧します。したがってユーザーはゲーム進行中に再接続をしても以前のゲーム状態を続けることができます。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

<a id="location-node"></a>
### Location Node { #location-node }

先ほど見てきたコネクション復旧の図でロケーションノードが見えます。ロケーションノードはGameAnvilが内部的にユーザーやルームなどの位置情報を管理するシステムノードです。ユーザーはロケーションノードについて直接実装したり使用することはできません。しかし位置情報を管理するロケーションノードの役割を理解することは、全体的なGameAnvilシステムの流れを理解するのに役立つため、ここで簡単に言及したいと思います。

上記のコネクション復旧を例に挙げてみます。クライアントが初回接続をしてゲームノードへログインを試みる過程で、これに関連するセッションとユーザーに対する位置情報は全てロケーションノードに保存されます。したがって再接続を行う場合には、以前の接続過程で保存しておいたこの位置情報を照会できます。このような位置情報はGameAnvil内部的にユーザーやルームの位置情報を照会し、これをもとにメッセージを伝達する用途などで非常に重要に使用されます。