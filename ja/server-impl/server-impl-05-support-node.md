## Game > GameAnvil > サーバー開発ガイド > サポートノード実装



## Support Node

![SupportNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_supportnode_on_network.png)

SupportNodeは、その名のとおり補助的な機能を行うためのノードです。ゲームユーザーやルームオブジェクトに関係なく、任意の機能を実装できます。また、SupportNodeはGatewayNodeによる接続を必要としないため、別途のコネクションやセッション管理が必要ありません。このような特徴を基に、SupportNodeは主に独自の機能を担当しています。例えば、ログをまとめて送信したり、ビリングサーバーと通信を専任したりなどの役割で使用できます。また、エンジンのRESTful機能を使用して、簡単なWebサーバーの代わりに使用することもできます。この時、SupportNodeは上図のように内部ネットワークだけでなく、外部ネットワーク(Public)に表示させることもできるため、GatewayNodeに接続する前/後に必要な機能を担当することもできます。このように任意の補助的な機能を柔軟に実装、バッチできるのがSupportNodeの長所です。



## SupportNodeの実装

SupportNodeは基本的にBaseSupportNode抽象クラスを継承実装します。ノード共通コールバックメソッドを除くと、追加でRESTful処理のためのgetRestMessageDispatcherメソッドでは唯一です。つまり、SupportNodeは前述した他のノードとまさにこの部分で差別化されています。ユーザーが定義したRESTful処理ができます。SupportNodeは、ユーザーが定義した一般的なパケット処理に加えて、RESTfulメッセージを処理できる唯一のユーザーノードです。

すべてのノードは、ユーザー定義メッセージを処理するためのディスパッチャー作成とメッセージハンドラ登録プロセスで必要です。SupportNodeはGameNodeと同様にユーザーが任意のコンテンツを実装できるノードの1つです。まず、(1)静的パケットディスパッチャーを1つ作成します。この時、必ずメモリや性能面で利点を得られるように静的(static)で作成します。(2)処理したいメッセージを実装しておいた[ハンドラ](server-impl-07-message-handling.md)と接続します。(3)最後に、RestMessageDispatcherで、(1)で作成したディスパッチャーを利用してパケットを処理します。この時、サンプルコードで使用したパケットディスパッチャーはRestMessageDispatcherであることに注意します。もちろん、一般パケットディスパッチャーを使用することも可能ですが、この例ではRESTfulリクエストを処理するためにRestMessageDispatcherを選択しました。両者の使用方法はほとんど同じです。

これらの一連のプロセスは、以下のサンプルコードで(1)～(3)に該当する注釈のすぐ下にあるコードで調べることができます。

```java
public class SampleSupportNode extends BaseSupportNode {

    // 1.RESTディスパッチャー作成
    private static RestMessageDispatcher<SampleSupportNode> restMessageDispatcher = new RestMessageDispatcher<>();

    // 2.SampleSupportNodeで処理したいURLとハンドラをマッピング
    static {
        // pathとmethod(GET, POST, ...)の組み合わせで登録。
        restMessageDispatcher.registerMsg("/auth", RestObject.GET, _RestAuthReq.class);
        restMessageDispatcher.registerMsg("/echo", RestObject.GET, _RestEchoReq.class);
    }
    
    @Override
    public MessageDispatcher<PublicSupportNode> getMessageDispatcher() {
        return null;
    }


    @Override
    public RestMessageDispatcher<PublicSupportNode> getRestMessageDispatcher() {
        return restMessageDispatcher;
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
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {}

    @Override
    public void onResume(Payload outPayload) throws SuspendExecution {}

    @Override
    public void onShutdown() throws SuspendExecution {}
}
```

SupportNode固有のコールバックは次のとおりです。


| コールバック名 | 意味                      | 説明                                                                                                                                                                                                           |
| ------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| getRestMessageDispatcher | 処理するRESTfulリクエスト有り | ノードに処理するメッセージがある場合に返します。ユーザーは自分が宣言したディスパッチャーを使用できます。詳細については[メッセージ処理](./server-impl-07-message-handling#13-getMessageDispatcher)を参照してください。 |
