## Game > GameAnvil > 서버 개념 설명 > Virtual Thread

## Virtual Thread

Virtual Thread는 JDK 21 에서 추가된 기능으로 경량 유저 스레드(Lightweight User Thread)입니다. GameAnvil에서는 Virtual Thread 를 서버 코드의 기본 흐름 단위로 사용하고 있습니다. 앞서 살펴본 노드의 싱글 스레드는 많은 수의 세션, 유저 그리고 방 등을 동시에 효과적으로 처리하기 위해 다수의 Virtual Thread 에서 코드 흐름이 나뉩니다. 즉, GameAnvil은 Virtual Thread 기반의 Continuation을 사용합니다.

표준 구현의 Virtual Thread들은 JDK 내부의 스레드풀에서 스케줄링됩니다. GameAnvil 에서는 동시 처리의 편의성을 높이기 위해 커스텀한 스레드풀 스케쥴러를 사용하고 있습니다. JDK 내부의 스레드풀의 크기를 1로 고정하면 바로 GameAnvil 노드의 모델이 됩니다. 즉, 노드는 다수의 Virtual Thread를 동시에 처리하기 위한 스케줄러인 것입니다. 이를 그림으로 표현하면 아래와 같습니다.

![VirtualThread_concept.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-basic/02-vt/VirtualThreadConcept1.png)

이와 같이 Virtual Thread를 사용할 때의 장점은 순차적인 코드 작성이 가능하다는 점입니다. 서버 코드는 일반적인 블로킹 코드를 작성하는 것과 매우 흡사해집니다. 별도의 콜백 처리나 완료 통보에 신경 쓸 필요가 전혀 없습니다. 이런 Virtual Thread의 장점에 더해 GameAnvil 사용자는 이 Virtual Thread 단위에 대해 크게 신경 쓸 필요가 없습니다. GameAnvil 엔진단에서 모든 Virtual Thread를 관리하고 있으므로 사용자는 일반적인 싱글 스레딩 코드를 작성하듯이 개발하면 됩니다.

![vt-context-switching.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-basic/02-vt/vt-context-switching.png)

GameAnvil 서버 코드는 비동기 처리를 기반으로 합니다. 이를 위해 [비동기 지원 API](../server-impl/server-impl-10-async.md)를 제공합니다. 이러한 비동기 API를 사용하여 임의의 Virtual Thread상에서 블로킹 호출을 할 경우에는 해당 Virtual Thread만 park(대기 상태)됩니다.

## Virtual Thread 기반의 비동기 처리
GameAnvil 엔진은 커스텀한 Virtual Thread Executor 를 구현하였습니다. 이 Executor 는 1개의 Platform Thread로 동작하며 1개의 노드에서 여러 Virtual Thread 를 실행 시킬 수 있습니다. 하나의 Virtual Thread에서 임의의 시간이 소요되는 I/O 호출을 했을 경우에 해당 Virtual Thread는 호출이 완료될 때까지 실행 권한을 다른 Virtual Thread에게 양보하여 동작합니다. 이러한 구현은 멀티 스레드 동기화 문제를 고민하지 않고 서버를 작성할 수 있게 하는 대신 코드를 Virtual Thread상에서 비동기로 처리될 수 있게 코드를 작성해야 합니다. GameAnvil 에서 제공하는 메서드는 기본적으로 이러한 비동기 처리를 사용하여 동작하며 엔진 사용자는 동기화 문제를 신경쓰지 않고 읽기 쉬운 코드를 작성할 수 있습니다. 


## Virtual Thread 사용 시 주의 사항 (JDK 21 사용 시)
스레드 블로킹 호출이 될 수 있는 작업(대표적으로 synchronized 사용)은 사용하지 않도록 해야하며 만일 호출 시 해당 Platform Thread가 블로킹되고 그 결과 모든 Virtual Thread들이 블로킹되어서 노드 전체가 멈추게 되는 결과를 초래하게 됩니다. 

```java
void someProblematicMethod() {
    synchronized (this) {
        someThreadBlockingCall(); // 해당 Virtual Thread뿐만 아니라 전체 스레드가 블로킹!
    }

}
```
synchronized를 사용하는 코드를 발견했을 때는 ReentrantLock 을 사용하는 코드로 변경할 수 있습니다
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

synchronized를 사용하여 Platform Thread가 멈추는 문제는 Pinning 이라고 합니다. Pinning 에 대한 자세한 설명은 openJDK 의 [Virtual Threads#Pinning](https://openjdk.org/jeps/444#Pinning) 항목 에서 확인할 수 있습니다.

