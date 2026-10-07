<!-- pre-align:aligned sig=d9d45df7e543 -->

<a id="game-gameanvil-virtual-thread"></a>
## Game > GameAnvil > Server Concept Description > Virtual Thread { #game-gameanvil-virtual-thread }

<a id="virtual-thread"></a>
## Virtual Thread { #virtual-thread }

Virtual Thread is a feature added in JDK 21 and is a lightweight user thread. GameAnvil uses Virtual Thread as the basic flow unit of server code. The single-threaded node described earlier splits code flow across multiple Virtual Threads to effectively handle large numbers of sessions, users, and rooms simultaneously. That is, GameAnvil uses Continuation based on Virtual Thread.

Standard Virtual Threads are scheduled in a thread pool inside the JDK. GameAnvil uses a customized thread pool scheduler to improve the convenience of concurrent processing. If the size of the JDK's internal thread pool is fixed at 1, it becomes the model of a GameAnvil node. In other words, a node is a scheduler for processing multiple Virtual Threads concurrently. The image shows the following:

![VirtualThread_concept.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-basic/02-vt/VirtualThreadConcept1.png)

One of the advantages of using Virtual Thread in this way is that you can write code sequentially. Server code becomes very similar to writing ordinary blocking code. You don't need to worry about separate callback handling or completion notifications at all. In addition to these advantages of Virtual Thread, GameAnvil users don't need to pay much attention to the Virtual Thread unit itself. Since the GameAnvil engine manages all Virtual Threads, you can develop just as you would write ordinary single-threaded code.

![vt-context-switching.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-basic/02-vt/vt-context-switching.png)

GameAnvil server code is based on asynchronous processing. To support this, it provides an [Asynchronous Support API](../server-impl/server-impl-10-async). When you make a blocking call on an arbitrary Virtual Thread using these asynchronous APIs, only that Virtual Thread is parked (put into a waiting state).

<a id="virtual-thread-2"></a>
## Asynchronous processing based on Virtual Thread { #virtual-thread-2 }
The GameAnvil engine implements a customized Virtual Thread Executor. This Executor operates with a single Platform Thread and can run multiple Virtual Threads on a single node. When a Virtual Thread makes an I/O call that takes an arbitrary amount of time, that Virtual Thread yields execution rights to another Virtual Thread until the call completes. This implementation allows you to write server code without worrying about multi-thread synchronization issues, but you must write your code so that it can be processed asynchronously on Virtual Threads. The methods provided by GameAnvil operate using this asynchronous processing by default, allowing engine users to write readable code without worrying about synchronization issues.


<a id="virtual-thread-3"></a>
## Caution on Using Virtual Thread { #virtual-thread-3 }
Operations that can cause thread-blocking calls (most notably, using `synchronized`) should be avoided. If called, the Platform Thread will be blocked, which in turn blocks all Virtual Threads, causing the entire node to stop.

```java
void someProblematicMethod() {
    synchronized (this) {
        someThreadBlockingCall(); // 해당 Virtual Thread뿐만 아니라 전체 스레드가 블로킹!
    }

}
```
You can replace code that uses `synchronized` with code that uses `ReentrantLock`.
```java
private final ReentrantLock myLock = new ReentrantLock();
void someProblematicMethod() {
    myLock.lock();
    try {
        someThreadBlockingCall(); // 이제 이 Virtual Thread만 대기합니다
    } finally {
        myLock.unlock();
    }
}
```

The problem where using `synchronized` causes a Platform Thread to stop is called Pinning. For more information on such pinning, see the section [Virtual Threads#Pinning](https://openjdk.org/jeps/444#Pinning) in openJDK.

