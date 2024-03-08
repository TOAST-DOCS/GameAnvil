## Game > GameAnvil > 서버 개발 가이드 > 타이머 사용하기

## 타이머

정해진 시간 또는 주기마다 임의의 코드를 실행할 수 있습니다. 여기에서는 이와 같은 타이머를 설정하는 방법을 설명합니다.

### TimerHandler 구현

타이머 처리를 하기 위해서는 반드시 이 인터페이스를 구현해야 합니다. GameAnvil의 타이머 시스템은 이 인터페이스에 대한 호출을 수행합니다. 원하는 만큼 여러 개의 TimerHandler를 아래와 같은 형태로 만들어도 됩니다.

```java
private TimerHandler getMyTimerHandler() {
    return new TimerHandler() {
            @Override
            public void onTimer(Timer timer, Object object) throws SuspendExecution {
                ...
            }
        };
}
```

이 코드와 동일한 람다 형태는 다음과 같습니다.

```java
private TimerHandler getMyTimerHandler() {
    return (timer, object) -> {
        ...
    };
}
```

이때, 인자로 전달되는 timer 객체는 타이머의 정보가 들어 있습니다. 또한 object는 해당 타이머를 등록한 객체입니다. 예를 들어 사용자의 GameUser에 타이머를 등록했다면 저 object는 해당 GameUser 객체입니다.

### 타이머 추가

앞서 살펴본 타이머 핸들러는 엔진에 추가해야 실제 구동됩니다. 타이머 추가를 위해서 addTimer API를 사용합니다.

```java
/**
 * 타이머를 추가
 *
 * @param interval      지연 간격
 * @param timeUnit      시간 단위
 * @param times         반복 횟수(0 == 무한반복)
 * @param handlerKey    핸들러 키 전달
 * @param isTickFromRun ffalse이면 타이머는 가장 최근의 구동 시점 기준으로 interval 후에 구동. true이면 타이머는 등록 시점 기준으로 interval 주기로 반복
 * @return 등록된 타이머 반환
 */
public Timer addTimer(int interval, TimeUnit timeUnit, int times, String handlerKey, boolean isTickFromRun)
```

각각의 매개변수에 대한 설명은 다음의 표와 같습니다.

| 이름 | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 예시 |
| --- |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| --- |
| Interval | 호출 시간 간격                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | 1 |
| TimeUnit | 호출 시간 단위                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | TimeUnit.SECONDS<br>TimeUnit.MILLISECONDS |
| Times | 호출 횟수                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | 20 |
| HandlerKey | 호출 함수 Key<br>User/Room에서 Transfer해야 하는 Timer를 구분하기 위해 사용                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | timerHandlerKey |
| IsTickFromRun | 다음 Timer 호출 시간의 기준을 바로 전 실행된 Timer의 시간으로 할 것인지, Timer가 등록된 시간을 기준으로 할 것인지를 결정<br><br>false이면 <br>  - Timer가 등록된 시간을 기준으로 호출<br>  - 예를 들어 1초 주기의 타이머는 직전의 타이머가 부하로 인해 0.9초 밀려서 0.1초 전에 호출되었더라도 타이머 등록 시간 기준으로 바로 0.1초 후에 다시 호출됨 <br>  - 정해진 시간 내의 호출 횟수가 중요할 경우 추천<br>  - 부하가 걸려도 호출 횟수는 보장이 되지만, 한꺼번에 밀린 타이머 처리가 동시에 여러 번 호출될 수 있음<br>  - 사용 예시: 1초에 한 번씩 버프를 거는 아이템<br><br>true이면<br>  - 바로 전 실행된 Timer의 시간을 기준으로 호출<br>  - 예를 들어 1초 주기의 타이머는 이전 타이머가 처리된 시간을 기준으로 1초 후에 다시 호출됨<br>  - 호출 횟수의 정확도보다 서버 부하에 맞춰 유연하게 호출되는 것이 중요할 경우 추천<br>  - 부하가 걸리면 전체적인 호출 횟수는 줄어들 수 있지만, 호출이 몰려서 발생하지는 않음<br>  - 사용 예시: 주기적인 DB 쿼리 | true/false |

### 타이머 제거

엔진에 등록된 타이머는 언제든 제거할 수 있습니다. 이때, removeTimer API를 사용합니다. 타이머 제거를 위해 등록할 때 반환된 타이머 객체를 잘 저장하고 있어야 합니다.

```java
// 반환값인 Timer 객체를 잘 보관합니다.
Timer timer = addTimer(1, TimeUnit.SECONDS, 20, "sampleTimerHandler1", false);

// 보관해 둔 Timer 객체를 사용하여 엔진에서 제거합니다.
removeTimer(timer);
```

## 타이머 전송

[전송 가능 객체](server-impl-08-object-transfer)에서 객체의 전송에 대해 살펴보았습니다. 기본적으로 게임 유저와 방 객체는 언제든 게임 노드 사이에서 전송될 수 있습니다. 그러므로 이러한 유저와 방 객체에 등록한 타이머도 함께 전송될 수 있어야 합니다. 엔진 내부에서 타이머를 전송할 때 타이머 핸들러 코드는 전송할 수 없기 때문에 타이머 핸들러 키 목록을 전송합니다. 따라서 타이머 핸들러 키에 해당하는 타이머를 유저 또는 방 전송 이후에도 사용할 것이라면 onTransferInTimerHandler 콜백에서 사용할 타이머 핸들러를 재등록하도록 코드를 작성해야 합니다.


### 타이머 핸들러 재등록

유저와 방은 전송 이후에도 사용할 타이머를 등록할 수 있도록 onTransferInTimerHandler 콜백을 호출합니다. 이때, 사용자는 문자열 키와 그에 대응하는 TimerHandler를 등록할 수 있습니다. 문자열 키가 전송된 타이머 핸들러 키 목록에 존재하는 경우, 재등록할 타이머 핸들러를 등록합니다. 이 콜백에서 재등록하지 않은 타이머는 유저 또는 방 전송 이후 더 이상 사용되지 않습니다.

```java
@Override
public void onTransferInTimerHandler(TimerHandlerTransferPack timerHandlerTransferPack) {
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)){
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
    ...
}
```
