## Game > GameAnvil > 서버 개념 설명 > Suspendable



## 1. Suspendable

다음의 몇 가지 내용은 파이버 기반의 코드를 작성할 때 주의할 사항입니다. 사용자는 파이버 단위에 크게 신경 쓸 필요가 없지만 아래의 주의 사항만큼은 반드시 지켜야 합니다.

- **Suspend**는 파이버가 코드 실행을 다른 파이버로 양보하고 대기 상태로 들어가는 것을 의미합니다. 이를 파이버 블로킹으로 표현할 수도 있습니다.
- Suspend될 수 있는 메서드는 **throws SuspendExecution**이라는 예외 시그니처를 반드시 추가해야 합니다. 이는 해당 메서드가 파이버를 중단시킬 수도 있는 블로킹 코드가 포함되어 있음을 엔진에 알려주는 약속입니다.

```
void someSuspendableMethod() throws SuspendExecution {

// some fiber-blocking call can be here

}
```

- 만일 특수한 이유로 throws SuspendExecution 예외 시그니처를 명시할 수 없다면 **@Suspendable** 애너테이션(annotation)을 사용할 수 있습니다. 그 외의 경우에는 반드시 throws SuspendExecution 예외 시그니처를 우선해서 사용하세요.

```
@Suspendable
void someSuspendableMethod() {

// some blocking call can be here

}
```

- 당연하게도 이러한 Suspendable 메서드를 호출하는 메서드 또한 Suspendable 합니다. 예를 들어 앞서 본 someSuspendableMethod()를 호출하는 someCaller() 라는 메서드가 있다고 할 때, someCaller() 또한 반드시 Suspendable하게 선언되어야 합니다.

```
void someCaller() throws SuspendExecution {

    someSuspendableMethod();

}
```

- SuspendExecution은 엔진에서 사용하는 Quasar 라이브러리가 파이버를 처리하기 위해 사용하는 특수한 기법입니다. 이는 **실제 예외가 아니며** 파이버가 아직 Java의 표준이 아니기 때문에 Quasar 라이브러리가 쓰는 일종의 우회 기법 정도로 이해할 수 있습니다. **그러므로 절대 이 SuspendExecution 예외를 catch해서는 안됩니다!** 흔하게 발생할 수 있는 오류이자 매우 중요한 부분입니다.

```
// !! 잘못된 사용 법 !!

void someCaller() {

    try {

        someSuspendableMethod();

    } catch (SuspendExecution e) {
        // 절대 SuspendExecution을 명시적으로 catch하면 안 된다!
    }
}
```

- 하지만 전체 예외를 catch하는 것은 문제없습니다. 이 부분에 대해서는 알아서 처리해 줍니다.

```
void someCaller() throws SuspendExeuction {

    try {

        someSuspendableMethod();

    } catch (Exception e) {
        // 문제 없음
    }
}
```

