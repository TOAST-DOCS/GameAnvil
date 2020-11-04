## Game > GameAnvil > 서버 개발 가이드 > 기본 개념

## 1.Node

GameAnvil 서버 구성의 가장 기본이 되는 단위는 노드입니다. 각각의 노드는 그 역할에 맞는 기능을 독립적으로 수행합니다.  몇 개의 노드로 어떤 역할을 수행할지는 별도로 설정 가능합니다. 노드에 관해 자세한 설명을 하기에 앞서 그 역할별로 노드를 나누면 다음과 같습니다.

| 노드 | 필수 | 기능 | 컨텐츠 구현 | 네트워크 접근 |
| ---- | ---- | ---- | ---- | ---- |
| Gateway | 필수 | 클라이언트 접속과 인증을 처리 | 가능 | public |
| Game | 필수 | 실제 게임 서버로서 컨텐츠를 처리 | 가능 | private |
| Support | 선택 | 필요에 따라 독립된 서비스로 구현하도록 지원 | 가능 | private or public |
| Match | 선택 | 매치메이킹을 수행 | 가능 | private |
| Location | 필수 | 유저와 방 등의 위치 정보를 저장 및 관리 | 불가능 | private |
| Management | 필수 | 서버 정보 취합 및 Admin/Agent와 통신 | 불가능 | private |
| Ipc | 필수 | GameAnvil 서버의 Inter-process 통신 처리 | 불가능 | private |

GameAnvil 사용자는 이 중에서 컨텐츠 구현이 가능한 노드들(Gateway, Game, Support, Match)에만 집중하면 됩니다. 나머지 노드는 GameAnvil 서버 엔진이 내부적으로 사용하며 사용자가 추가로 구현할 부분은 없습니다. 또한 필수 노드는 반드시 서버군에 하나 이상 존재해야 구동이 가능합니다.

이러한 노드의 계층 구조는 아래의 그림과 같은 모습입니다.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)

<br>

## 2.Single-Threaded


노드에 관한 가장 중요한 부분은 하나의 노드가 하나의 스레드로 처리된다는 것입니다. 즉, 각 노드는 기본적으로 모든 처리를 비동기적으로 해야하며 이 노드 스레드는 블러킹 없이 지속적으로 구동되는 것이 보장되어야 합니다. 이러한 구동 모델은 Vert.x나 Node.js의 그것과 매우 흡사합니다.

싱글 스레드의 가장 큰 장점은 역시 Lock-free 입니다. 사용자는 특수한 경우를 제외하고는 명시적인 Lock을 걸 필요가 없고 걸어서도 안됩니다. 앞서 설명했듯이 Lock을 거는 순간 해당 노드 스레드가 처리를 멈출 수 있습니다. 이는 해당 노드 전체의 로직이 멈추는 것을 의미하므로 GameAnvil 상에서 개발할 때는 절대 노드 스레드에 대해서는 Lock을 사용해서는 안됩니다. 그러므로 노드 스레드와 외부 스레드 사이에 절대 게임 관련 객체가 공유되어서는 안됩니다. 단, 외부 스레드를 사용하여 작업을 위임하는 경우에 외부 스레드들 사이의 Lock은 사용해도 무방합니다. 만일 외부 스레드로 작업을 위임하거나 위임한 작업에 대한 결과를 획득할 필요가 있을 경우에는 GameAnvil에서 제공하는 [비동기 지원 API](3z5.async)를 사용하십시오.

<br>

## 3.Fiber

앞서 살펴본 노드의 싱글 스레드는 많은 수의 세션, 유저 그리고 방 등을 동시에 효과적으로 처리하기 위해 다시 다수의 Fiber로 코드 흐름이 나누어집니다. 즉, Fiber는 일종의 경량 유저 스레드(Lightweight User Thread)로서 GameAnvil 서버 코드의 기본 흐름 단위입니다. GameAnvil은 Fiber 기반의 Continuation을 지원합니다. 

다수의 Fiber들은 스레드풀(Executor) 상에서 스케쥴링 됩니다. 이 때, 스레드풀의 크기를 1로 고정하면 바로 GameAnvil 노드의 모델이 됩니다. 즉, 노드는 다수의 Fiber를 동시에 처리하기 위한 싱글스레드 스케쥴러인 것입니다. 이를 그림으로 표현하면 아래와 같습니다.

![image.png](http://static.toastoven.net/prod_gameanvil/images/FiberConcept.png)

이와 같이 Fiber를 사용할 때의 장점은 순차적인 코드 작성이 가능하다는 점입니다. 서버 코드는 일반적인 블러킹 코드를 작성하는 것과 매우 흡사해지는거죠. 별도의 콜백 처리나 완료 통보에 신경쓸 필요가 전혀 없습니다. 이런 Fiber와 관련하여 한 가지 다행인 점은 GameAnvil 사용자가 이 Fiber 단위에 대해 크게 신경 쓸 필요가 없다는 점입니다. GameAnvil 엔진단에서 모든 Fiber를 관리하고 있으므로 사용자는 일반적인 싱글 스레딩 코드를 작성하듯이 개발하면 됩니다.

![image.png](http://static.toastoven.net/prod_gameanvil/images/FiberConcept2.png)

GameAnvil 서버 코드는 비동기 처리를 기반으로 합니다. 이를 위해 비동기 전용 [비동기 지원 API](3z5.async)를 제공합니다. 이러한 비동기 API를 사용하여  임의의 Fiber 상에서 블러킹 호출을 할 경우에는 해당 Fiber만 Suspend 됩니다. 이 부분은 문서의 다음 부분에서 더 자세히 다루도록 하겠습니다.

<br>

### **[주의 사항]**

다음의 몇 가지 내용은 Fiber 기반의 코드를 작성함에 있어 주의할 사항입니다. 앞서 설명 드렸듯이 사용자는 Fiber 단위에 대해 크게 신경 쓸 필요가 없습니다. 하지만 아래의 주의 사항 만큼은 반드시 숙지해서 지켜주세요.

* **Suspend**는 Fiber가 코드 실행을 다른 Fiber로 양보하고 대기 상태로 들어가는 것을 의미합니다. 이를 Fiber 블러킹으로 표현할 수도 있습니다.
* 이런 Suspend 될 수 있는 메서드는 **throws SuspendExecution** 이라는 예외 시그니쳐를 반드시 추가해야 합니다. 이는 해당 메서드가 Fiber를 중단시킬 수도 있는 블러킹 코드가 포함되어 있음을 엔진에 알려주는 약속입니다.

```java
void someSuspendableMethod() throws SuspendExecution {

// some fiber-blocking call can be here

}
```

* 만일 특수한 이유로 throws SuspendExecution 예외 시그니쳐를 명시할 수 없는 경우에는 **@Suspendable** 어노테이션을 사용할 수 있습니다. 그 외의 경우에는 반드시 throws SuspendExecution 예외 시그니쳐를 우선해서 사용하세요.

```java
@Suspendable
void someSuspendableMethod() {

// some blocking call can be here

}
```

* 당연하게도 이러한 Suspendable 메서드를 호출하는 메서드 또한 Suspendable 합니다. 예를 들어 앞서 본 someSuspendableMethod()를 호출하는 someCaller() 라는 메서드가 있다고 할 때, someCaller() 또한 반드시 Suspendable하게 선언되어야 합니다.

```java
void someCaller() throws SuspendExecution {

    someSuspendableMethod();

}
```

* 마지막으로 가장 흔히 실수를 하는 부분이자 중요한 부분입니다. 위에서 말한 SuspendExecution은 엔진에서 사용하는 Quasar 라이브러리가 Fiber를 처리하기 위해 사용하는 특수한 기법입니다. 이는 **실제 예외가 아니며** Fiber가 아직 Java의 표준이 아니기 때문에 Quasar 라이브러리가 쓰는 일종의 우회 기법 정도로 이해할 수 있습니다. **그러므로 절대 이 SuspendExecution 예외를 catch해서는 안됩니다!**

```java
// !! 잘못된 사용 법 !!

void someCaller() {

    try {

        someSuspendableMethod();

    } catch (SuspendExecution e) {
        // 절대 SuspendExecution을 명시적으로 catch하면 안된다!
    }
}
```

* 하지만 전체 예외를 catch하는 것은 문제 없습니다. 이 부분에 대해서는 알아서 처리를 해줍니다.

```java
void someCaller() throws SuspendExeuction {

    try {

        someSuspendableMethod();

    } catch (Exception e) {
        // 문제 없음
    }
}
```

<br>

## 4.Asynchronous on fibers

코드는 Fiber 상에서 비동기로 처리될 수 있어야 합니다. 즉, 하나의 Fiber에서 임의의 시간이 소요되는 I/O 호출을 했을 경우에 해당 Fiber는 호출이 완료될 때까지 실행 권한을 다른 Fiber에게 양보할 수 있습니다. 심지어 스레드 블러킹 호출조차도 이를 Fiber 블러킹 호출로 전환하여 비동기화할 수 있도록 API를 제공합니다. 즉, 스레드 블러킹 호출은 반드시 [비동기 지원 API](3z5.async)를 사용해서 처리해야 하며, 만일 이를 어기고 직접 호출하면 해당 스레드가 블러킹되고 그 결과 스레드 상의 모든 Fiber들이 블러킹되어서 노드 전체가 멈추게 되는 결과를 초래하게 됩니다.

```java
    void someProblematicMethod() {

        someThreadBlockingCall(); // 해당 Fiber 뿐만 아니라 전체 스레드가 블러킹 된다!

    }
```

이에 대한 자세한 설명은 [비동기 지원](server-5-async)에서 자세히 다룹니다.

<br>

## 5.핵심 라이브러리

아래의 네 가지가 GameAnvil에서 사용하는 핵심 라이브러리입니다. Qusar와 ZeroMQ 그리고 Netty는 엔진 내부에서 사용하므로 GameAnvil 사용자가 직접 사용할 일은 없습니다. Protocol Buffers는 메시지를 직렬화/역직렬화 하는 과정에서 사용하게 됩니다. 직접 사용 여부와 관계없이 아래의 네 가지 라이브러이에 대해 잘 이해하고 있다면 엔진 사용에 많은 도움이 될 것입니다.

| 라이브러리       | 용도                           |
| ---------------- | ------------------------------ |
| Quasar           | Fiber 기반의 Continuation 지원 |
| ZeroMQ           | 서버의 IPC                     |
| Netty            | 서버-클라이언트 통신           |
| Protocol Buffers | 서버-클라이언트 메시지 직렬화  |

