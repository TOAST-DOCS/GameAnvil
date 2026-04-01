## Game > GameAnvil > サーバー開発ガイド > サポートノード実装

## Support Node

![SupportNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_supportnode_on_network.png)

SupportNodeは名前の通り補助的な機能を実行するためのノードです。ゲームユーザーやルームオブジェクトに関係なく任意の機能を実装できます。また、SupportNodeはGatewayNodeを通じた接続を要求しないため、別途のコネクションやセッション管理が必要ありません。このような特徴を基に、SupportNodeは主に独自の機能を担当します。例えば、ログを集約して送信したり、課金サーバーとの通信を専任するなどの役割として使用できます。また、エンジンのRESTful機能を使用して簡単なWebサーバーの代用として使用することも可能です。このとき、SupportNodeは上の図のように内部ネットワークだけでなく**外部ネットワーク(Public)**に公開することもできるため、GatewayNodeへ接続する前/後に必要な機能を担当することもできます。このように任意の補助的な機能を柔軟に実装して配置できることがSupportNodeの長所です。

## SupportNode実装

このようなSupportNodeは基本的にISupportNodeインターフェースを実装します。ノード共通コールバックメソッドのみを持っています。SupportNodeは先に確認した他のノードとは異なり、ユーザーが定義したRESTful処理ができるということです。言い換えると、SupportNodeはユーザーが定義した一般的なパケット処理に加えてRESTfulメッセージを処理できる唯一のユーザーノードです。

```java
@GameAnvilSupportNode(gameServiceName = "MySupport") // (1) "MySupport"というサービスのためのSupportNodeとしてエンジンに登録
public class SampleSupportNode implements ISupportNode {
    private ISupportNodeContext supportNodeContext;

    /**
     * サポートノードコンテキストを伝達するために呼び出し
     * <p/>
     * オブジェクトが生成された後、一度呼び出される
     *
     * @param supportNodeContextサポートノードコンテキスト
     */
    @Override
    public void onCreate(ISupportNodeContext supportNodeContext) {
        this.supportNodeContext = supportNodeContext;
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
public class _SupportNodeTest {
    // (2) SampleSupportNodeで処理したいプロトコルとハンドラをマッピング
    @GameNodeMapping(
        value = MyGame.SupportNodeTest.class,   // 処理するプロトバッファ
        loadClass = SampleSupportNode.class     // メッセージを受け取る対象(SampleSupportNode)
    )
    public void runGameNodeTest(IGameNodeDispatchContext ctx) {
        // ここで行う作業を作成
    }
}
```


全てのノードはユーザー定義メッセージを処理するためのメッセージハンドラ登録過程が必要です。特にGameNodeはゲームコンテンツのためにこのような過程が必須です。サーバー実行前にメインクラスで設定を行います。(1)GameAnvilConfigに設定されているゲームサービス名を生成します。ゲームサービス名は必ずGameAnvilConfigに定義されている名前を使用する必要があります。(2) そして処理したいメッセージを実装しておいた[ハンドラ](server-impl-07-message-handling.md#_2)と接続します。1つのGameNodeクラスはただ1つのサービスに対してのみ登録できます。
