## Game > GameAnvil > サーバー概念説明 > メッセージ処理とパケット

## パケット

パケットはGameAnvilでサーバーとクライアント間でメッセージを伝達する単位です。Javaのプロトコルバッファやビルダーを使用してパケットを生成できます。パケットはGameAnvilエンジンでサポートする様々なタイプで作成できますが、簡単な使用法は次のとおりです。
```java
// レスポンスプロトコルバッファ作成
EchoSend res = EchoSend.newBuilder()
    .setData("hello");

// 伝達パケット生成
Packet packet = Packet.makePacket(res);

// クライアントへ伝達
userContext.send(packet);
```

上記のコードはゲームユーザーからクライアントへパケットを伝達するコードです。まず伝達するプロトコルバッファを定義した後、パケットを生成してクライアントへ伝達します。 

## パケット活用性能最適化 

パケットは内部的にプロトコルバッファをシリアライズしたデータのキャッシュを行っています。したがって複数のユーザーにプロトコルバッファメッセージを送る時は、パケットを何度も作って送る代わりに一度だけ作って送るのが良いです。以下の例では2名のユーザーにパケットの浅いコピーをして伝達する方法を扱っています。 
```java
EchoSend.Builder res = EchoSend.newBuilder()
    .setData("hello");

List<MyUser> allUsers = roomContext.getAllUsers();

// ユーザーが複数名いると仮定して 
// 全てのユーザーにメッセージを送ります。

// 悪い方法
for (GameUser myUser : allUsers) {
    IUserContext userContext = user.getUserContext();
    userContext.send(Packet.makePacket(res)); // この時resを送信すると、ユーザー数分だけシリアライズします注意！
}

// 良い方法
Packet packet = Packet.makePacket(res); // このパケットはメンバー変数などで保存後、何度も再利用可能です
                                        // ただしパケットを作った後、resの修正は反映されません
for (GameUser myUser : allUsers) {
    IUserContext userContext = user.getUserContext();
    userContext.send(packet.duplicate()); // パケットの浅いコピーをしますシリアライズは1回！
}
```
`duplicate`関数は浅いコピーをします。GameAnvilエンジンは内部的にパケットが正常に処理されなかったか検査をしているため、再利用時に正常に処理されない可能性があります。この時パケットの浅いコピーをして新しいパケットのように扱えば、GameAnvilは別のパケットとして認識し再度カウントするようになります。 

もちろんこのような追加はコードが読みづらくなる可能性があり、上記の例のように単純なコードはシリアライズ作業を多くしても実際の性能上大きく差がない場合があります。多くのデータが入るパケットを送信し性能最適化が必要な時点でこのような構文を作成するのが良いでしょう。 
