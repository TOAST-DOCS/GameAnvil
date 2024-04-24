## Game > GameAnvil > サーバー開発ガイド > ゲートウェイノードの実装



## Gateway Node

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gatewaynode_on_network.png)

GatewayNodeはクライアントが接続する入り口(Gateway)です。すなわち、クライアントのコネクションとゲームサービスのセッションを管理します。この時、クライアントはゲームを進行するために必ずGatewayNodeに接続して、認証を完了する必要があります。これらの関係については、次の図を参照してください。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

通常、クライアントはGatewayNodeで1つのコネクションを結びます。この時、該当コネクションに対して認証手続きを行い、成功した場合に限って、1つ以上のセッションを作成できます。それぞれのセッションは、クライアントとユーザー間の論理的接続単位です。上図はクライアントが1つのコネクションを通じて、GameサービスとChatサービスでセッションを作成したものです。このような仕組みは、意図せずクライアントの接続が切れても、簡単に[セッションの復元(Session Recovery)](#21-session-recovery)を可能にします。



### GatewayNodeの実装

このようなGatewayNodeは、BaseGatewayNodeクラスを継承して共通コールバックメソッドを再定義するだけで構いません。このような共通コールバックメソッドは、その名前が用途を明確に説明しています。また、こうして実装したクラスをエンジンに登録するために@GatewayNodeアノテーションを使用します。

```java
@GatewayNode // このGatewayNodeクラスをエンジンに登録
public class SampleGatewayNode extends BaseGatewayNode {

    /**
     * 初期化する際に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onInit() throws SuspendExecution {      
    }

    /**
     * Ready前に処理する部分用に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPrepare() throws SuspendExecution {
    }

    /**
     * Ready呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onReady() throws SuspendExecution {      
    }

    /**
     * pause呼び出し
     *
     * @param payload contentsから送信したい追加情報
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPause(Payload payload) throws SuspendExecution {      
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
}
```



### Connectionの実装

コネクションは、クライアントの物理的な接続自体を意味します。クライアントは固有のAccountIdを利用して、コネクション上で認証手続きを進行できます。認証が成功した場合、該当AccountIdは作成されたコネクションにマッピングされます。

このようなコネクションは、次のようにBaseConnectionを継承した後、コールバックメソッドを再定義します。この時、任意のプラットフォームで認証した後、取得するユーザーのキー値などをAccountIdとして使用できます。例えば、Gamebaseを通じて認証した後、UserIdを取得すると、この値をGameAnvilの認証プロセスでAccountIdとして使用できます。また、GatewayNodeの実装と同様に、実装したクラスをエンジンに登録するために@Connectionアノテーションを使用しています。

```java
@Connection // このコネクションクラスをエンジンに登録
public class SampleConnection extends BaseConnection<SampleGameSession> {

    /**
     * authenticate時に呼び出し
     *
     * @param accountId アカウントID
     * @param password アカウントパスワード
     * @param deviceId クライアントのデバイスID
     * @param payload  クライアントから送信されるペイロード
     * @param outPayload クライアントで送信するペイロード
     * @return認証成否: falseの場合は、クライアントとの接続が終了
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public boolean onAuthenticate(final String accountId, final String password, final String deviceId, final Payload payload, final Payload outPayload) throws SuspendExecution {      
    }

    @Override
    public void onPause() throws SuspendExecution {
    }

    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * クライアントとの接続が切断された場合に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    public void onDisconnect() throws SuspendExecution {      
    }
}
```

これらのコールバックの意味と用途については次の表を参照してください。


| コールバック名    | 意味               | 説明                                                                                                                                                                                                                                                  |
| ---------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onAuthenticate | 認証              | クライアントがAuthentication()APIを使用して、コネクションに認証をリクエストした際に呼び出されます。ユーザーは、ここでクライアントが送信した認証情報に基づいて認証処理を実行できます。認証が成功した場合はtrueを返し、失敗した場合はfalseを返す必要があります。 |
| onPause        | 一時停止         | コンソールを介してGatewayNodeを一時停止すると、該当GatewayNodeのすべてのコネクションに対して呼び出されます。ユーザーは、ノードが一時停止された時、コネクションにおいて追加で処理したいコードをここに実装できます。                                                             |
| onResume       | 再開              | コンソールを介してGatewayNodeが一時停止状態から動作を再開すると、該当GatewayNodeのすべてのコネクションに対して呼び出されます。ユーザーは再開状態でコネクションに対して処理したいコードをここに実装できます。                                              |
| onDisconnect   | 接続終了         | クライアントからの接続が切断された時に呼び出されます。この時、追加で処理するコードをここに実装します。                                                                                                                                                           |
| getMessageDispatcher | 処理するパケット有り | ノードに処理するメッセージがある場合に返します。ユーザーは、自分が宣言したディスパッチャーを使用できます。詳細については[メッセージ処理](./server-impl-07-message-handling#13-getMessageDispatcher)を参照してください。 |



### Session実装

コネクションの接続に成功したクライアントは、該当コネクション上でサービスごとに1つずつGameNodeに対する論理的なセッションを結べます。GameAnvilは内部的にコネクションのAccountIdとセッションのSubIdを組み合わせて、サーバー全体で固有のセッションを区別できます。

この時、SubIdはユーザーが任意で決めたルールに合わせて、該当コネクション内のどの固有値を割り当てても構いません。すなわち、互いに異なるコネクションは、同一のSubIdを持つこともできます。しかし、異なるAccountIdを持つため、区別可能です。また、実装したセッションクラスをエンジンに登録するために@Sessionアノテーションを使用しています。

```java
@Session // このセッションクラスをエンジンに登録
public class SampleSession extends BaseSession {

    private static MessageDispatcher<SampleSession> dispatcher = new MessageDispatcher<>();

    static {
        dispatcher.registerMsg(SampleGame.MsgToSessionReq.class, _MsgToSessionHandler.class);
    }

    @Override
    public MessageDispatcher<SampleSession> getMessageDispatcher() {
        return dispatcher;
    }

    /**
     * loginを呼び出す前に呼び出し
     *
     * @param outPayloadクライアントで送信するペイロード
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPreLogin(Payload outPayload) throws SuspendExecution {      
    }

    /**
     * ログイン成功後に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPostLogin() throws SuspendExecution {      
    }

    /**
     * ログアウト後に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPostLogout() throws SuspendExecution {
    }
}
```

これらのコールバックの意味と用途については次の表を参照してください。


| コールバック名  | 意味               | 説明                                                                                                                                                                                                                                        |
| -------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onPreLogin   | ログイン前の処理     | GameNodeにログインをリクエストする前に呼び出されます。この時、ユーザーは引数で送信された出力用のペイロード(outPayload)に任意の値を入れて、ログインリクエストに載せて送信できます。このペイロードは、ゲームノードでログインコールバックを処理する時にそのまま送信されます。 |
| onPostLogin  | ログイン後の処理     | GameNodeにログイン完了した後、呼び出されます。ログイン完了後にセッションで処理するコードがある場合は、ここに実装します。                                                                                                                                   |
| onPostLogout | ログアウト後の処理   | ログアウト処理が完了した後、呼び出されます。ログアウト後にセッションで処理するコードがある場合は、ここに実装します。                                                                                                                                       |
| getMessageDispatcher | 処理するパケット有り | ノードに処理するメッセージがある場合に返します。ユーザーは自分が宣言したディスパッチャーを使用できます。詳細については[メッセージ処理](./server-impl-07-message-handling#13-getMessageDispatcher)を参照してください。 |



## ConnectionとSession

クライアントはゲートウェイノードに接続します。すなわち、コネクションを作成します。このコネクションを通じて、アカウントとユーザー情報に基づいて認証とログインを行うことができます。ログインまで完了すると、任意のゲームノードにユーザーオブジェクトが作成されます。これは、ゲートウェイノードと該当ゲームノード間に論理的なセッションが作成されたことを意味します。こうしてコネクションとセッションの作成が完了すると、該当ユーザーはゲームの進行が可能になります。これについては、ゲームノードを説明する際にもう一度見てみましょう。

### Session Recovery

万が一、クライアントとゲートウェイノード間で再接続が発生した場合、次の図のようにセッションの復元(Session Recovery)が行われます。再接続プロセスでクライアントは、複数のゲートウェイノードのうち、以前とは異なる場所にコネクションを試みることもできます。この場合、ユーザーオブジェクトが存在するゲームノードの位置情報に基づいて、新たにセッションを復元します。そのため、ユーザーはゲーム進行中に再接続しても、それまでのゲーム状態を維持できます。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

### Location Node

前述したコネクション復元図でロケーションノードが表示されています。ロケーションノードは、GameAnvilが内部的にユーザーやルームなどの位置情報を管理するシステムノードです。ユーザーはロケーションノードに直接実装または、使用することはできません。しかし、位置情報を管理するロケーションノードの役割を理解することは、GameAnvilシステムの全体的なフローを理解するのに役立つため、ここで簡単に説明します。

上記のコネクション復元を例に挙げてみます。クライアントが初回接続して、ゲームノードにログインを試みる過程で、これに関連するセッションとユーザーの位置情報はすべてのロケーションノードに保存されます。そのため、再接続した場合は、以前の接続過程で保存していたこの位置情報を照会できます。このような位置情報は、GameAnvil内部的にユーザーやルームの位置情報を照会し、これをもとにメッセージを送信する用途などに使用されます。
