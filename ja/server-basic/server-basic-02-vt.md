## Game > GameAnvil > サーバー概念説明 > Virtual Thread

## Virtual Thread

Virtual ThreadはJDK 21で追加された機能で、軽量ユーザースレッド(Lightweight User Thread)です。GameAnvilではVirtual Threadをサーバーコードの基本フロー単位として使用しています。先ほど見てきたノードのシングルスレッドは、多数のセッション、ユーザー、そしてルームなどを同時に効果的に処理するために多数のVirtual Threadでコードフローが分かれます。つまり、GameAnvilはVirtual ThreadベースのContinuationを使用します。

標準実装のVirtual ThreadはJDK内部のスレッドプールでスケジューリングされます。GameAnvilでは同時処理の利便性を高めるためにカスタムスレッドプールスケジューラを使用しています。JDK内部のスレッドプールのサイズを1に固定すると、まさにGameAnvilノードのモデルになります。つまり、ノードは多数のVirtual Threadを同時に処理するためのスケジューラなのです。これを図で表現すると以下のようになります。

![VirtualThread_concept.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-basic/02-vt/VirtualThreadConcept1.png)

このようにVirtual Threadを使用する際の長所は、順次的なコード作成が可能だという点です。サーバーコードは一般的なブロッキングコードを作成することと非常に似ています。別途コールバック処理や完了通知に気を使う必要が全くありません。このようなVirtual Threadの長所に加え、GameAnvilユーザーはこのVirtual Thread単位について大きく気を使う必要がありません。GameAnvilエンジン側で全てのVirtual Threadを管理しているため、ユーザーは一般的なシングルスレッディングコードを作成するように開発すればよいです。

![vt-context-switching.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-basic/02-vt/vt-context-switching.png)

GameAnvilサーバーコードは非同期処理をベースにします。このために[非同期サポートAPI](../server-impl/server-impl-10-async.md)を提供します。このような非同期APIを使用して任意のVirtual Thread上でブロッキング呼び出しをする場合には、該当Virtual Threadのみpark(待機状態)されます。

## Virtual Threadベースの非同期処理
GameAnvilエンジンはカスタムVirtual Thread Executorを実装しました。このExecutorは1つのPlatform Threadで動作し、1つのノードで複数のVirtual Threadを実行させることができます。1つのVirtual Threadで任意の時間がかかるI/O呼び出しをした場合に、該当Virtual Threadは呼び出しが完了するまで実行権限を他のVirtual Threadに譲って動作します。このような実装はマルチスレッド同期問題を悩まずサーバーを作成できるようにする代わりに、コードをVirtual Thread上で非同期に処理されるようにコードを作成する必要があります。GameAnvilで提供するメソッドは基本的にこのような非同期処理を使用して動作し、エンジンユーザーは同期問題を気にせず読みやすいコードを作成できます。


## Virtual Thread使用時の注意事項
スレッドブロッキング呼び出しになり得る作業(代表的にsynchronized使用)は使用しないようにすべきであり、もし呼び出した時には該当Platform Threadがブロックされ、その結果全てのVirtual Threadがブロックされてノード全体が止まる結果を招くことになります。 

```java
void someProblematicMethod() {
    synchronized (this) {
        someThreadBlockingCall(); // 該当Virtual Threadだけでなく全体スレッドがブロッキング！
    }

}
```
synchronizedを使用するコードを発見した時は、ReentrantLockを使用するコードに変更できます
```java
private final ReentrantLock myLock = new ReentrantLock();
void someProblematicMethod() {
    myLock.lock();
    try {
        someThreadBlockingCall(); // これでこのVirtual Threadのみ待機します
    } finally {
        myLock.unlock();
    }
}
```

synchronizedを使用してPlatform Threadが止まる問題はPinningといいます。Pinningに関する詳細な説明はopenJDKの[Virtual Threads#Pinning](https://openjdk.org/jeps/444#Pinning)項目で確認できます。
