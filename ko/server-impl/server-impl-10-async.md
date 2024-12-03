## Game > GameAnvil > 서버 개발 가이드 > 비동기 지원

## 비동기 지원

GameAnvil은 다음과 같은 목적을 위해 비동기 처리를 지원합니다.

| 관점        | 설명                                                      |
|-----------|---------------------------------------------------------|
| 코드 생산성    | 간결하고 짧아지는 코드비동기화 방식에 따라서는 콜백 처리나 스레드 위임 등의 처리는 모두 생략 가능 |
|           | 비동기화 방식에 따라서는 콜백 처리나 스레드 위임 등의 처리는 모두 생략 가능             |
| 성능        | 드라이버나 라이브러리 수준에서 최적의 구현을 제공해 줄 경우 성능 UP                 |
|           | 외부 스레드 풀을 이용한 스레드 단위의 비동기 처리는 리소스 소비와 비용이 큼             |
| 가상 스레드 지원 | 스레드가 아닌 가상 스레드 중심의 비동기 처리                               |
|           | Future 지원 강화                                            |

GameAnvil의 API도 다른 많은 비동기 방식 API들처럼 Future를 리턴하는 방식으로 동작합니다. 더욱 자유롭게 코드 흐름을 만들 수 있습니다. GameAnvil에서 Virtual Thread를 공식 지원하기 때문에, 블로킹 코드를 호출하더라도 Java 21을 지원하는 코드라면 Platform Thread 가 아닌 Virtual Thread가 블록이 됩니다. 

```java

Future<Packet> packetFuture = user.requestToNode(nodeId, myRequest1);
Future<Response> httpFuture = getHttpClient().executeRequest(myRequest2);
Packet packetResponse = packetFuture.get();  
Response httpResponse = httpFuture.get();  // Java 21 에서는 Virtual Thread 만 블락

```

> [참고]
> 
> Platform Thread : 기존 Java에서 사용한 Thread. OS 스레드 혹은 자바 스레드입니다.
> 
> Virtual Thread: Java 21에서 추가된 새로운 Thread입니다 이전 버전 GameAnvil의 Fiber 와 유사한 동작을 합니다 자세한 동작은 [여기](https://openjdk.org/jeps/444)를 참고하십시오.


## RDBMS 지원

기존 Java 에서 많은 RDBMS 드라이버는 `java.sql.DriverManager` 를 사용하고 있으므로 쿼리는 블로킹입니다. 그러나 Java 21 에서는 Virtual Thread 위에서 실행 시 이러한 블로킹 쿼리를 Virtual Thread 만 정지하는 형태로 바꿔 실행하여 비동기를 활용한 향상의 이점을 누릴 수 있습니다. GameAnvil 역시 Virtual Thread 위에서 실행하여 비동기 쿼리를 통한 성능 향상이 가능합니다. 이러한 드라이버는 대표적으로 [MySQL Connector/J](https://github.com/mysql/mysql-connector-j) 가 있겠습니다. 

비동기 규칙을 자세하게 설정하고 블록킹 방식의 드라이버보다 성능 향상을 필요로 할 때는 Future 방식으로 비동기 처리를 해주는  [jasync-sql](https://github.com/jasync-sql/jasync-sql)과 같은 드라이버를 사용할 수 도 있습니다. [jasync-sql](https://github.com/jasync-sql/jasync-sql) 를 사용 시 높은 유연성과 성능을 기대할 수 있지만 GameAnvil 에서 실행 시 몇가지 주의할 점이 있습니다. 이에 대한 내용은 아래 [Pinning 문제](#pinning-문제) 문단을 참고하십시오.

> [참고]
>
> 모든 라이브러리가 Virtual Thread 를 지원하는 것은 아닙니다. 이전 버전에 맞춰 제작된 라이브러리를 GameAnvil 에서 실행 시 정상적으로 동작하지 않을 수 있습니다. 예를 들어 [MySQL Connector/J](https://github.com/mysql/mysql-connector-j) 는 9.x 버전 부터 Virtual Thread 를 지원합니다. 8.x 버전을 사용 시 정상적으로 동작하지 않을 수 있습니다.


## Redis 지원
많이 사용되고 있는 라이브러리는 [Jedis](https://github.com/redis/jedis) 를 사용하고 있는 곳이 더 많지만 엔진팀 내부 확인 결과 [Jedis](https://github.com/redis/jedis) 는 Virtual Thread 사용 시 스레드가 잠기는 문제가 발생할 수 있습니다. GameAnvil 에서는 Virtual Thread 를 사용하고 있어 이러한 문제가 자주 발생하며 문제 발생 시 디버깅이 어려워 매우 탐지하기 어렵습니다. 만약 Jedis 사용을 고려하고 있다면 [Lettuce](https://github.com/redis/lettuce) 사용을 권장합니다. 이미 Jedis 를 사용하고 있어 마이그레이션이 어려울 때는 다음과 같이 다른 스레드 풀에서 jedis 를 실행하는 코드로 사용하여 스레드가 잠기는 문제를 회피할 수 있습니다.

```java
final String myKey = ForkJoinPool.commonPool().submit(() -> {
    return jedis.get("my_key");
});
```

GameAnvil 에서는 Redis 사용을 위해 [Lettuce](https://github.com/redis/lettuce) 사용을 권장합니다. [Lettuce](https://github.com/redis/lettuce) 를 GameAnvil 에서 사용 시 높은 성능을 기대할 수 있지만 몇가지 주의할 점이 있습니다. 이에 대한 내용은 아래 [Pinning 문제](#pinning-문제) 문단을 참고하십시오



## Pinning 문제
* Virtual Thread 의 synchronized 블록 안에서 Virtual Thread 를 일시 정지하는 코드를 사용 시 Virtual Thread 가 잠기는 문제가 발생할 수 있습니다. 간단한 재현 방법은 다음과 같습니다.

```java
// Main.java
int loopCount = Runtime.getRuntime().availableProcessors() * 2;
List<Thread> threads = new ArrayList<>();
for (int i = 0; i < loopCount; i++) {
    Thread t = Thread.ofVirtual().start(() -> {
        System.out.println("hello"); // System.out.println 안에서 ReentrantLock 를 사용합니다
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
위 코드를 실행시 프로그램이 종료되지 않습니다. System.out.println 안에서 ReentrantLock 를 사용 하고 있기 때문에 잠기는 문제인데 이렇게 잠기게된 Virtual Thread 는 혼자서는 탈출 할 수 없으며 다른 스레드가 Virtual Thread 를 풀어줘야만 계속 동작할 수 있습니다. 위 예제에서는 `availableProcessors * 2` 로 작성되어 항상 프로그램이 멈추게 됩니다만 이 코드를 `availableProcessors - 1` 로 변경 후 실행 시 정상적으로 종료됩니다. Virtual Thread 를 실행하는 Carrier Thread 풀의 기본 값이 `availableProcessors` 이기 때문입니다.

이러한 문제를 테스트 환경에서 제거하기 위하여 아래 JVM 옵션을 추가할 수 있습니다
```
-Djdk.tracePinnedThreads=full
```
VM 옵션 추가 후 위 프로그램을 다시 실행 시 아래와 같은 경고가 출력되어 문제가 발생하는 코드를 확인할 수 있습니다.
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

GameAnvil 에서는 엔진 사용을 위해 커스텀한 Virtual Thread 를 사용하고 있기 때문에 위와 같은 문제가 발생 시 잠금이 해제되지 않을 수 있습니다. [jasync-sql](https://github.com/jasync-sql/jasync-sql), [Lettuce](https://github.com/redis/lettuce) 사용 시 에도 Connection 을 만드는 부분에서 Pinning 경고가 출력합니다. Connection 연결 이외에서는 문제가 발생하지 않으므로 Connection 생성 시 메인 스레드 혹은 다른 스레드 에서 생성하는 작업을 넣어주는 것이 필요합니다. 아래는 [jasync-sql](https://github.com/jasync-sql/jasync-sql) 을 사용하여 GameAnvil 서버 시작 전 먼저 MySql ConnectionPool 을 만드는 예제입니다.

```java
public static Connection connectionPool;
public static void main(String[] args) {
    // 먼저 Database를 연결합니다
    connectionPool = MySQLConnectionBuilder.createConnectionPool(
        "jdbc:mysql://localhost/test", config -> {
            config.setUsername("db_user");
            config.setPassword("db_password");
            config.setMaxActiveConnections(30);
            return Unit.INSTANCE;
        });

    // 이후 GameAnvil 초기화를 합니다
    GameAnvilServer gameAnvilServer = GameAnvilServer.getInstance();
}
```

이러한 Pinning 에 대한 자세한 설명은 openJDK 의 [Virtual Threads#Pinning](https://openjdk.org/jeps/444#Pinning) 항목 에서 확인할 수 있습니다.
Virtual Thread 에서 synchronized 제한은 이후 Java 릴리즈에서 제거될 수 있습니다. 자세한 사항은 [Synchronize Virtual Threads without Pinning](https://openjdk.org/jeps/491) 에서 확인할 수 있습니다.