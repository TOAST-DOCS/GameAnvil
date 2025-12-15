## Game > GameAnvil > Server Development Guide > Asynchronous Support

## Asynchronous Support

GameAnvil supports asynchronous processing for the following purposes:

| Viewpoint | Description |
|--------------|---------------------------------------------------------|
| Code Productivity | Concise and Short Code. Depending on the asynchronous method, callback processing and thread delegation can be omitted. |
| | Depending on the asynchronous method, callback processing and thread delegation can be omitted. |
| Performance | Performance increases when optimal implementations are provided at the driver or library level. |
| | Thread-level asynchronous processing using an external thread pool consumes significant resources and is expensive. |
| Virtual Thread Support | Asynchronous processing centered on virtual threads, not threads. |
| | Enhanced Future Support |

GameAnvil's API works in a way that returns Future like many other asynchronous APIs. Create code flows more freely. Since GameAnvil officially supports Virtual Thread, even if you call a blocking code, the code that supports Java 21 will be blocked by Virtual Thread instead of Platform Thread. 

```java

Future<Packet> packetFuture = user.requestToNode(nodeId, myRequest1);
Future<Response> httpFuture = getHttpClient().executeRequest(myRequest2);
Packet packetResponse = packetFuture.get();  
Response httpResponse = httpFuture.get(); // In Java 21, only Virtual Thread is blocked

```

> [Note]
> 
> Platform Thread : Thread used by existing Java. OS thread or Java thread.
> 
> Virtual Thread: This is a new thread added in Java 21 that works similarly to the Fiber of the previous version of GameAnvil. For more information, see [here](https://openjdk.org/jeps/444).


## RDBMS Support

In existing Java, many RDBMS drivers use `java.sql.DriverManager`, so queries are blocked. In Java 21, however, when running above Virtual Thread, you can benefit from improved utilization of asynchronous, by changing these block queries to the form that stops Virtual Thread only. GameAnvil can also be executed above Virtual Thread to improve performance through asynchronous queries. These drivers are mainly [MySQL Connector/J](https://github.com/mysql/mysql-connector-j). 

You can also use a driver such as [jasync-sql](https://github.com/jasync-sql/jasync-sql) that enables asynchronous processing in the Future method when setting asynchronous rules in detail and requiring performance improvements over a blocking-type driver. You can expect greater flexibility and performance when using [jasync-sql](https://github.com/jasync-sql/jasync-sql) , but there are several cautions to take when running with GameAnvil . For more information about this, see the [Pinning Issues](#pinning) section below.

> [Note]
>
> Not all libraries support Virtual Thread. Library created according to the previous version may not operate normally when running from GameAnvil. For example, [MySQL Connector/J](https://github.com/mysql/mysql-connector-j) supports Virtual Thread from version 9.x or later. 8.x version may not operate normally.


## Redis Support
Libraries that are used extensively have [Jedis](https://github.com/redis/jedis), but internal engine team checks suggest that [Jedis](https://github.com/redis/jedis) may cause threads to lock when using Virtual Thread. GameAnvil uses customized Virtual Thread, which makes it very difficult to detect unknown actions and debugs when these problems occur. If you are considering using Jedis, we recommend using [Lettuce](https://github.com/redis/lettuce). When you are already using Jedis and migration is difficult, you can avoid threads being locked by using the code that runs jedis from another thread pool, such as

```java
final String myKey = ForkJoinPool.commonPool().submit(() -> {
    return jedis.get("my_key");
});
```

GameAnvil recommends using [Lettuce](https://github.com/redis/lettuce) for Redis. You can expect high performance when using [Lettuce](https://github.com/redis/lettuce) on GameAnvil , but there are a few cautions. For more information about this, see the [Pinning Issues](#pinning) section below.



## Pinning Issues
* When using the code to temporarily stop Virtual Thread within the synchronized block of Virtual Thread, Virtual Thread may experience problems of locking. The simple reproduction methods are as follows:

```java
// Main.java
int loopCount = Runtime.getRuntime().availableProcessors() * 2;
List<Thread> threads = new ArrayList<>();
for (int i = 0; i < loopCount; i++) {
    Thread t = Thread.ofVirtual().start(() -> {
        System.out.println("hello"); // Use ReentrantLock within System.out.println
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
The program is not terminated when the above code is executed. The Virtual Thread locked in this way cannot be escaped by itself and can continue to operate only when another thread releases the Virtual Thread. In the example above, the program is written as `availableProcessors * 2` and will always stop, but the code will be changed to `availableProcessors - 1` and will end properly when the program runs. The default value for the Carrier Thread pool running Virtual Thread is `availableProcessors`.

To eliminate these problems in the test environment, you can add the following JVM options
```
-Djdk.tracePinnedThreads=full
```
VM options and, when you restart the above programs, you will receive the caution below to see the code that will cause the problem.
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

GameAnvil may not be unlocked in the event of the problem above because it is using customized Virtual Thread for engine use. When using [jasync-sql](https://github.com/jasync-sql/jasync-sql), when using [Lettuce](https://github.com/redis/lettuce), a Pinning warning is issued from the part creating Edo Connection. No problem occurs outside of the Connection connection, so when creating a Connection, you need to put the task to be created in the main thread or another thread. The following is an example of using [jasync-sql](https://github.com/jasync-sql/jasync-sql) to create MySql ConnectionPool before beginning the GameAnvil server:

```java
public static Connection connectionPool;
public static void main(String\[] args) {
    // Connect the Database first 
    onnectionPool = MySQLConnectionBuilder.createConnectionPool(
        "jdbc:mysql://localhost/test", config -> {
            config.setUsername("db\_user");
            config.setPassword("db\_password");
            config.setMaxActiveConnections(30);
            return Unit.INSTANCE;
        });

    // After that, reset GameAnvil
    GameAnvilServer gameAnvilServer = GameAnvilServer.getInstance();
}
```

For more information on such pinning, see the section [Virtual Threads#Pinning](https://openjdk.org/jeps/444#Pinning) in openJDK.