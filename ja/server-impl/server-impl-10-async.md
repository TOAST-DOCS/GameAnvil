## Game > GameAnvil > サーバー開発ガイド > 非同期サポート

## 非同期サポート

GameAnvilは次のような目的のために非同期処理をサポートします。

| 観点        | 説明                                                          |
|-----------|---------------------------------------------------------|
| コード生産性    | 簡潔で短くなるコード。非同期方式によっては、コールバック処理やスレッド委譲などの処理は全て省略可能 |
|            | 非同期方式によっては、コールバック処理やスレッド委譲などの処理は全て省略可能              |
| 性能         | ドライバーやライブラリレベルで最適な実装を提供してくれる場合、性能UP                  |
|            | 外部スレッドプールを利用したスレッド単位の非同期処理は、リソース消費とコストが大きい              |
| 仮想スレッド対応 | スレッドではなく仮想スレッド中心の非同期処理                                |
|           | Futureサポート強化                                          |

GameAnvilのAPIも他の多くの非同期方式APIのようにFutureを返す方式で動作します。より自由にコードフローを作成できます。GameAnvilではVirtual Threadを公式サポートしているため、ブロッキングコードを呼び出してもJava 21をサポートするコードであれば、Platform ThreadではなくVirtual Threadがブロックされます。 

```java

Future<Packet> packetFuture = user.requestToNode(nodeId, myRequest1);
Future<Response> httpFuture = getHttpClient().executeRequest(myRequest2);
Packet packetResponse = packetFuture.get();  
Response httpResponse = httpFuture.get();  // Java 21ではVirtual Threadのみブロック

```

> [参考]
> 
> Platform Thread : 既存のJavaで使用していたThread。OSスレッドまたはJavaスレッドです。
> 
> Virtual Thread: Java 21で追加された新しいThreadです。以前のバージョンのGameAnvilのFiberと類似した動作をします。詳細な動作は[こちら](https://openjdk.org/jeps/444)を参照してください。


## RDBMSサポート

既存のJavaにおいて多くのRDBMSドライバーは `java.sql.DriverManager` を使用しているため、クエリはブロッキングです。しかしJava 21ではVirtual Thread上で実行する際、このようなブロッキングクエリをVirtual Threadのみ停止する形に変えて実行し、非同期を活用した向上のメリットを享受できます。GameAnvilもVirtual Thread上で実行し、非同期クエリによる性能向上が可能です。このようなドライバーには代表的に[MySQL Connector/J](https://github.com/mysql/mysql-connector-j)があります。

非同期ルールを詳細に設定し、ブロッキング方式のドライバーより性能向上を必要とする場合は、Future方式で非同期処理を行う [jasync-sql](https://github.com/jasync-sql/jasync-sql)のようなドライバーを使用することもできます。[jasync-sql](https://github.com/jasync-sql/jasync-sql)を使用する場合、高い柔軟性と性能を期待できますが、GameAnvilで実行する際にいくつか注意点があります。これに関する内容は、以下の[Pinning問題](#pinning)のセクションを参照してください。

> [参考]
>
> 全てのライブラリがVirtual Threadをサポートしているわけではありません。以前のバージョンに合わせて制作されたライブラリをGameAnvilで実行すると、正常に動作しない場合があります。例えば[MySQL Connector/J](https://github.com/mysql/mysql-connector-j)は、9.xバージョンからVirtual Threadをサポートします。8.xバージョンを使用すると、正常に動作しない場合があります。


## Redisサポート
多く使用されているライブラリには[Jedis](https://github.com/redis/jedis)がありますが、エンジンチーム内部での確認結果、[Jedis](https://github.com/redis/jedis)はVirtual Thread使用時にスレッドがロックされる問題が発生する可能性があります。GameAnvilではカスタムVirtual Threadを使用しており、このような問題が発生すると予期しない動作をし、デバッグが難しく、検知するのが非常に困難です。もしJedisの使用を検討しているなら、[Lettuce](https://github.com/redis/lettuce)の使用を推奨します。すでにJedisを使用していて移行が難しい場合は、次のように別のスレッドプールでjedisを実行するコードを使用して、スレッドがロックされる問題を回避できます。

```java
final String myKey = ForkJoinPool.commonPool().submit(() -> {
    return jedis.get("my_key");
});
```

GameAnvilではRedis使用のために[Lettuce](https://github.com/redis/lettuce)の使用を推奨します。[Lettuce](https://github.com/redis/lettuce)をGameAnvilで使用する場合、高い性能を期待できますが、いくつか注意点があります。これに関する内容は、以下の[Pinning問題](#pinning)のセクションを参照してください



## Pinning問題
* Virtual Threadのsynchronizedブロック内でVirtual Threadを一時停止するコードを使用すると、Virtual Threadがロックされる問題が発生する可能性があります。簡単な再現方法は次のとおりです。

```java
// Main.java
int loopCount = Runtime.getRuntime().availableProcessors() * 2;
List<Thread> threads = new ArrayList<>();
for (int i = 0; i < loopCount; i++) {
    Thread t = Thread.ofVirtual().start(() -> {
        System.out.println("hello"); // System.out.println内でReentrantLockを使用します
        synchronized (Main.class) {
            System.out.println("synchronized");
        }
    });
    threads.add(t);
}

for (Thread thread : threads) {
    thread.join();
}
```
上記のコードを実行するとプログラムが終了しません。System.out.println内でReentrantLockを使用しているためにロックされる問題ですが、このようにロックされたVirtual Threadは自力では脱出できず、他のスレッドがVirtual Threadを解放してあげないと動作を継続できません。上記の例では `availableProcessors * 2` と記述されているため常にプログラムが止まりますが、このコードを `availableProcessors - 1` に変更後に実行すると正常に終了します。Virtual Threadを実行するCarrier Threadプールのデフォルト値が `availableProcessors` だからです。

このような問題をテスト環境で除去するために、以下のJVMオプションを追加できます
```
-Djdk.tracePinnedThreads=full
```
VMオプション追加後に上記のプログラムを再実行すると、以下のような警告が出力され、問題が発生するコードを確認できます。
```
Thread[#46,ForkJoinPool-1-worker-3,5,CarrierThreads]
    java.base/java.lang.VirtualThread$VThreadContinuation.onPinned(VirtualThread.java:183)
    java.base/jdk.internal.vm.Continuation.onPinned0(Continuation.java:393)
    java.base/java.lang.VirtualThread.park(VirtualThread.java:582)
    java.base/java.lang.System$2.parkVirtualThread(System.java:2643)
    java.base/jdk.internal.misc.VirtualThreads.park(VirtualThreads.java:54)
    java.base/java.util.concurrent.locks.LockSupport.park(LockSupport.java:219)
    java.base/java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:754)
    java.base/java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire(AbstractQueuedSynchronizer.java:990)
    java.base/java.util.concurrent.locks.ReentrantLock$Sync.lock(ReentrantLock.java:153)
    java.base/java.util.concurrent.locks.ReentrantLock.lock(ReentrantLock.java:322)
    java.base/jdk.internal.misc.InternalLock.lock(InternalLock.java:74)
    java.base/java.io.PrintStream.writeln(PrintStream.java:824)
    java.base/java.io.PrintStream.println(PrintStream.java:1168)
    org.example.Main.lambda$main$0(Main.java:30) <== monitors:1
    java.base/java.lang.VirtualThread.run(VirtualThread.java:309)
```

GameAnvilではエンジン使用のためにカスタムVirtual Threadを使用しているため、上記のような問題が発生した場合、ロックが解除されない可能性があります。[jasync-sql](https://github.com/jasync-sql/jasync-sql)、[Lettuce](https://github.com/redis/lettuce)使用時にもConnectionを作成する部分でPinning警告が表示されます。Connection接続以外では問題が発生しないため、Connection生成時はメインスレッドまたは別のスレッドで生成する処理を入れる必要があります。以下は[jasync-sql](https://github.com/jasync-sql/jasync-sql)を使用してGameAnvilサーバー開始前に、まずMySql ConnectionPoolを作成する例です。

```java
public static Connection connectionPool;
public static void main(String[] args) {
    // まずDatabaseを接続します
    connectionPool = MySQLConnectionBuilder.createConnectionPool(
        "jdbc:mysql://localhost/test", config -> {
            config.setUsername("db_user");
            config.setPassword("db_password");
            config.setMaxActiveConnections(30);
            return Unit.INSTANCE;
        });

    // その後GameAnvilの初期化を行います
    GameAnvilServer gameAnvilServer = GameAnvilServer.getInstance();
}
```

このようなPinningに関する詳細な説明は、openJDKの[Virtual Threads#Pinning](https://openjdk.org/jeps/444#Pinning)項目で確認できます。
Virtual Threadでのsynchronized制限は、今後のJavaリリースで解除される可能性があります。詳細は[Synchronize Virtual Threads without Pinning](https://openjdk.org/jeps/491)で確認できます。
