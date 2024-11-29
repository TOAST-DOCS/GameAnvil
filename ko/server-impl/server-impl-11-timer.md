## Game > GameAnvil > 서버 개발 가이드 > 타이머 사용하기

## 타이머

정해진 시간 또는 주기마다 임의의 코드를 실행할 수 있습니다. 여기에서는 이와 같은 타이머를 설정하는 방법을 설명합니다.

### TimerHandler 구현

타이머 처리를 하기 위해서는 반드시 이 인터페이스를 구현해야 합니다. GameAnvil의 타이머 시스템은 이 인터페이스에 대한 호출을 수행합니다. 원하는 만큼 여러 개의 TimerHandler를 아래와 같은 형태로 만들어도 됩니다.

```java
private ITimerHandler getMyTimerHandler() {
    return new ITimerHandler() {
        @Override
        public void onTimer(final ITimer timer) {
            // 타이머에서 할 작업
        }
    };
}
```

이 코드와 동일한 람다 형태는 다음과 같습니다.

```java
private ITimerHandler getMyTimerHandler() {
    return (timer) -> {
         // 타이머에서 할 작업
    };
}
```

이때, onTimer 메서드의 인자로 전달되는 timer 객체는 타이머의 정보가 들어 있습니다. 타이머 객체를 활용하여 타이머를 일시정지, 재개, 정지하는 등의 추가적인 작업을 할 수 있습니다.

### 타이머 추가

앞서 살펴본 타이머 핸들러는 엔진에 추가해야 실제 구동됩니다. 타이머 추가를 위해서 addTimer API를 사용합니다.

```java
/**
 * 한 번 작업을 수행할 타이머를 추가 후 반환받는다
 *
 * @param timerKey 타이머 키
 * @param delay    지연 간격
 * @param timeUnit 시간 단위
 * @param handler  등록할 핸들러
 * @return {@link ITimer} 타입으로 등록된 타이머 반환
 */
ITimer scheduleTimer(String timerKey, int delay, TimeUnit timeUnit, ITimerHandler handler);
```

각각의 매개변수에 대한 설명은 다음의 표와 같습니다.

| 이름        | 설명                                                           | 예시                                        |
|-----------|--------------------------------------------------------------|-------------------------------------------|
| timerKey  | 호출 함수 Key<br>User/Room에서 Transfer해야 하는 Timer를 구분하기 위해 사용     | "MyTimer"                                 |
| delay     | 호출 시간 간격                                                     | 100                                       |
| TimeUnit  | 호출 시간 단위                                                     | TimeUnit.SECONDS<br>TimeUnit.MILLISECONDS |
| handler   | 실행할 핸들러 객체                                                   | 사용자 정의                                    |


### 타이머 제거

엔진에 등록된 타이머는 언제든 제거할 수 있습니다. 이때, removeTimer API를 사용합니다. 타이머 제거를 위해 등록할 때 반환된 타이머 객체를 잘 저장하고 있어야 합니다.

```java

userContext.scheduleTimer("MyTimer", 1000, TimeUnit.MILLISECONDS, new ITimerHandler() {
    @Override
    public void onTimer(final ITimer timer) {
        // 타이머에서 할 작업
    }
});

// 타이머 이름으로 객체를 제거합니다
userContext.removeTimer("MyTimer");
```

## 타이머 전송

[전송 가능 객체](server-impl-08-object-transfer)에서 객체의 전송에 대해 살펴보았습니다. 기본적으로 게임 유저와 방 객체는 언제든 게임 노드 사이에서 전송될 수 있습니다. 그러므로 이러한 유저와 방 객체에 등록한 타이머도 함께 전송될 수 있어야 합니다. 엔진 내부에서 타이머를 전송할 때 타이머 핸들러 코드는 전송할 수 없기 때문에 타이머 핸들러 키 목록을 전송합니다. 따라서 타이머 핸들러 키에 해당하는 타이머를 유저 또는 방 전송 이후에도 사용할 것이라면 onTransferIn 콜백에서 사용할 타이머 핸들러를 재등록하도록 코드를 작성해야 합니다.


### 타이머 핸들러 재등록

유저와 방은 전송 이후에도 사용할 타이머를 등록할 수 있도록 onTransferIn 콜백을 호출합니다. 이때, 사용자는 문자열 키와 그에 대응하는 ITimerHandler를 등록할 수 있습니다. 문자열 키가 전송된 타이머 핸들러 키 목록에 존재하는 경우, 재등록할 타이머 핸들러를 등록합니다. 이 콜백에서 재등록하지 않은 타이머는 유저 또는 방 전송 이후 더 이상 사용되지 않습니다.

```java
@Override
public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {
    if (timerHandlerTransferPack.containsTimerKey("MyTimer")) {
        timerHandlerTransferPack.reRegister("MyTimer", newMyTimer());
    }
}
```
