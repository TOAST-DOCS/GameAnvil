## Game > GameAnvil > サーバー開発ガイド > メッセージハンドリング

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/07-message-handling/three_steps_for_message_process.png)

## 一般メッセージ処理

* GameAnvilのメッセージ処理は上の図のように大きく3つの部分に分かれます。この3つの部分が互いにかみ合ってメッセージ処理フローを作ることになります。
* このうちonDispatchコールバック実装はエンジンで独自に行っており、エンジンユーザーはDispatcherの宣言とメッセージハンドラを実装してパケットを処理します。

### メッセージハンドラ実装及びメッセージとハンドラの接続

このメッセージハンドラを直接実装します。このとき、GameAnvilはエンジン内部はもちろん、全てのサンプルコードでメッセージハンドラに対して「_」で始まる命名規則を使用します。今後登場する全てのサンプルコードでも、「_」で始まるクラスは全てメッセージハンドラです。最も基本的な形のメッセージハンドラは次のとおりです。CONTEXT_CLASSは該当メッセージの流れを示すクラスを意味し、MAPPING_CLASSは処理するメッセージを登録するクラスを意味します。Contextクラスで対象オブジェクトの取得、メッセージの応答などを行うことができます。

```java
@GameAnvilController // このクラスをメッセージハンドラとして使用します。
public class _MyEchoHandler {

    @MAPPING_CLASS // このメソッドがどのメッセージを処理するかを宣言します。
    public void execute(CONTEXT_CLASS ctx, EchoSend request) {
        System.out.println("Receive EchoSend Message!");
    }
}
```

例えば、パケット受信者がGameUserなら、そのメッセージハンドラは次のとおりです。

```java
@GameAnvilController // このクラスをメッセージハンドラとして使用します。
public class _MyEchoHandler {

    @GameUserMapping(
        value = EchoSend.class,             // 処理するプロトコルバッファ
        loadClass = SampleGameUser.class    // メッセージを受信する対象
    )
    public void execute(IUserDispatchContext ctx, EchoSend request) {
        System.out.println("Receive EchoSend Message!");
    }
}
```

## httpリクエストの処理

パケットディスパッチャは2種類存在します。1つは前述した一般的なパケットディスパッチャで、もう1つはhttpリクエストを処理するためのディスパッチャです。全体的な使用方法は、この二つでほぼ同じです。ただし、このようなhttpメッセージ処理は、SupportNodeのみがサポートします。

### httpパケットディスパッチャの作成及びメッセージとハンドラの接続

次のサンプルコードは、RESTfulリクエストを処理するために、以前とは異なる@SupportNodeHttpMappingを宣言します。また、メッセージクラスではなくURL形式のAPIをハンドラに接続しています。httpメッセージハンドラは、RestObjectが引数として渡されるという違いがあります。

```java 
@GameAnvilController
public class _RestAuthReq {

    @SupportNodeHttpMapping(path = "/hello", method = "GET", loadClass = WebSupportNode.class)
    public void execute(WebSupportNode node, RestObject restObject) {
		// ...
    }  
}
```
