## Game > GameAnvil > 서버 개념 설명 > 파이버



## 1. 파이버 (Fiber)

파이버는 일종의 경량 유저 스레드(Lightweight User Thread)로서 GameAnvil 서버 코드의 기본 흐름 단위입니다. 앞서 살펴본 노드의 싱글 스레드는 많은 수의 세션, 유저 그리고 방 등을 동시에 효과적으로 처리하기 위해 다시 다수의 파이버로 코드 흐름이 나뉩니다. 즉, GameAnvil은 파이버 기반의 Continuation을 지원합니다.

다수의 파이버들은 스레드풀(Executor)상에서 스케줄링됩니다. 이때, 스레드풀의 크기를 1로 고정하면 바로 GameAnvil 노드의 모델이 됩니다. 즉, 노드는 다수의 파이버를 동시에 처리하기 위한 애너테이션(annotation) 스케줄러인 것입니다. 이를 그림으로 표현하면 아래와 같습니다.

![image.png](https://static.toastoven.net/prod_gameanvil/images/FiberConcept.png)

이와 같이 파이버를 사용할 때의 장점은 순차적인 코드 작성이 가능하다는 점입니다. 서버 코드는 일반적인 블로킹 코드를 작성하는 것과 매우 흡사해집니다. 별도의 콜백 처리나 완료 통보에 신경쓸 필요가 전혀 없습니다. 이런 파이버의 장점에 더해 GameAnvil 사용자는 이 파이버 단위에 대해 크게 신경 쓸 필요가 없습니다. GameAnvil 엔진단에서 모든 파이버를 관리하고 있으므로 사용자는 일반적인 싱글 스레딩 코드를 작성하듯이 개발하면 됩니다.

![fiber-context-switching.png](https://static.toastoven.net/prod_gameanvil/images/fiber-context-switching.png)

GameAnvil 서버 코드는 비동기 처리를 기반으로 합니다. 이를 위해 [비동기 지원 API](server-10-async)를 제공합니다. 이러한 비동기 API를 사용하여  임의의 파이버 상에서 블로킹 호출을 할 경우에는 해당 파이버만 suspend(대기 상태)됩니다. 이 내용은 문서 아래에서 더 자세히 다루겠습니다.



## 2. 파이버 기반의 비동기 처리 (Asynchronous on fibers)

코드는 파이버 상에서 비동기로 처리될 수 있어야 합니다. 즉, 하나의 파이버에서 임의의 시간이 소요되는 I/O 호출을 했을 경우에 해당 파이버는 호출이 완료될 때까지 실행 권한을 다른 파이버에게 양보할 수 있습니다. 심지어 스레드 블로킹 호출조차도 이를 파이버 블로킹 호출로 전환하여 비동기화할 수 있도록 API를 제공합니다. 즉, 스레드 블로킹 호출은 반드시 [비동기 지원 API](server-10-async)를 사용해서 처리해야 하며, 만일 이를 어기고 직접 호출하면 해당 스레드가 블로킹되고 그 결과 스레드 상의 모든 파이버들이 블로킹되어서 노드 전체가 멈추게 되는 결과를 초래하게 됩니다.

```
    void someProblematicMethod() {

        someThreadBlockingCall(); // 해당 파이버 뿐만 아니라 전체 스레드가 블로킹된다!

    }
```

이에 대한 자세한 설명은 구현 방법에서 [비동기 지원](server-10-async)에서 자세히 다룹니다.

