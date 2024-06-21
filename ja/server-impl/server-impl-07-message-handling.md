## Game > GameAnvil > サーバー開発ガイド > メッセージハンドリング

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/three_steps_for_message_process_1213.png)

## 一般メッセージ処理

* GameAnvilのメッセージ処理は、上図のように大きく3つの部分に分けられます。この3つの部分が互いに相まってメッセージ処理の流れを作ります。
* このうち、onDispatchコールバックの実装はエンジンが独自に行っており、エンジンユーザーはDispatcherの宣言とメッセージハンドラを実装してパケットを処理します。

### パケットディスパッチャーの作成およびメッセージとハンドラ接続

まず、メッセージ処理を行うためのパケットディスパッチャーを作成します。このディスパッチャーは該当クラスに対して使用されるため、リソースの浪費を防ぐために必ずstaticで作成します。

```java
// (パケットディスパッチャー作成  
private static final MessageDispatcher<MY_USER_CLASS> messageDispatcher = new MessageDispatcher<>();
```

こうして作成したディスパッチャーに必要なメッセージとハンドラを接続します。

```java
// 処理したいメッセージとハンドラをマッピング
static {
    messageDispatcher.registerMsg(MyProto.MyMsg.class, _MyMsgHandler.class);
}
```


### メッセージハンドラの実装

次に該当メッセージハンドラを直接実装します。この時、GameAnvilはエンジン内部はもちろん、すべてのサンプルコードでメッセージハンドラに対して_で始まるネーミングを使用します。これから登場するすべてのサンプルコードでも_で始まるクラスはすべてメッセージハンドラです。最も基本的な形のメッセージハンドラは次のとおりです。RECEIVER_CLASSは該当メッセージを受け取るクラスを意味します。

```java
public class _MyGameMsg implements MessageHandler<RECEIVER_CLASS, MyProto.MyMsg> {

    private static final Logger logger = getLogger(_MyMsgHandler.class);

    @Override
    public void execute(RECEIVER_CLASS receiver, MyProto.MyMsg myMsg) throws SuspendExecution {
    }

}
```

例えば、パケット受信者がGameUserの場合、そのメッセージハンドラは次のとおりです。

```java
public class _MyGameMsg implements MessageHandler<GameUser, MyProto.MyMsg> {

    private static final Logger logger = getLogger(_MyMsgHandler.class);

    @Override
    public void execute(GameUser receiver, MyProto.MyMsg myMsg) throws SuspendExecution {
    }

}
```


### getMessageDispatcherコールバックの実装

パケットディスパッチャーとメッセージハンドラをすべて実装しました。これでエンジンが処理するパケットがある時に、ディスパッチャーを渡すだけです。

```java
public class GameUser {
    // .. 省略
    @Override
    public final MessageDispatcher<GameUser> getMessageDispatcher() {
        return packetDispatcher;
    }
    // .. 省略
}
```

ここまで一般的なパケット処理の流れを見てきました。それとともに、RESTfulリクエストに対する処理について見てみましょう。



## RESTfulリクエスト処理

パケットディスパッチャーは2つ存在します。1つは前述した一般的なパケットディスパッチャーであり、もう1つはRESTfulリクエストを処理するためのディスパッチャーです。全体的な使用方法は両者ともほとんど同じです。ただし、このようなRESTfulメッセージ処理は、SupportNodeだけサポートしています。 



### RESTパケットディスパッチャーの作成およびメッセージとハンドラ接続

次のサンプルコードは、RESTfulリクエストを処理するために以前とは異なるRestPacketDispatcherを作成しています。また、メッセージクラスではなく、URL形式のAPIをハンドラに接続しています。

```java
private static final RestMessageDispatcher<RECEIVER_CLASS> restDispatcher = new RestMessageDispatcher<>();

static {
    // pathとmethod(GET, POST, ...)の組み合わせで登録。
    restDispatcher.registerMsg("/auth", RestObject.GET, _RestAuthReq.class);
    restDispatcher.registerMsg("/echo", RestObject.GET, _RestEchoReq.class);
    restDispatcher.registerMsg("/testGET", RestObject.GET, _RestTestGET.class);
    restDispatcher.registerMsg("/testPOST", RestObject.POST, _RestTestPOST.class);
}
```

サンプルコードのようにRestMessageDispatcherを使用してユーザーが望むRESTful APIとメッセージハンドラを接続しています。



### ハンドラの実装

RESTfulメッセージハンドラは、Packetの代わりにRestObjectが引数で送信される点を除けば、違いはありません。

```java
public class _RestAuthReq implements RestMessageHandler<WebSupportNode> {
    private static final Logger logger = getLogger(_RestAuthReq.class);

    @Override
    public void execute(WebSupportNode node, RestObject restObject) throws SuspendExecution {
		...
    }  
}
```



### RestMessageDispatcherの実装

RESTfulメッセージ処理用のgetRestMessageDispatcherは、SupportNodeのみをサポートしています。restMessageDispatcherを渡すように簡単に実装できます。
```java
@Override
public final RestMessageDispatcher<WebSupportNode> getRestMessageDispatcher() {
    return restMessageDispatcher;
}
```
