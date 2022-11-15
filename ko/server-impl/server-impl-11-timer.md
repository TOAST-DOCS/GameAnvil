## Game > GameAnvil > 서버 개발 가이드 > 타이머 사용하기

## 1. 타이머

사용자는 정해진 시간에 혹은 정해진 주기마다 임의의 코드를 실행할 수 있습니다. 이런 타이머 관련 처리 방법을 한 번 살펴보도록 하겠습니다.

### 1.1. TimerHandler 구현

타이머 처리를 하기 위해서는 반드시 이 인터페이스를 구현해야 합니다. GameAnvil의 타이머 시스템은 이 인터페이스에 대한 호출을 수행합니다. 원하는 만큼 여러개의 TimerHandler를 아래와 같은 형태로 만들어도 됩니다.

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

이 때, 인자로 전달되는 timer 객체는 타이머의 정보가 들어있습니다. 또한 object는 해당 타이머를 등록한 객체입니다. 예를 들어 사용자의 GameUser에 타이머를 등록했다면 저 object는 해당 GameUser 객체입니다.

### 1.2. 타이머 추가

앞 서 살펴본 타이머 핸들러는 엔진에 추가해야 실제 구동됩니다. 타이머 추가를 위해서 addTimer API를 사용합니다. 이 API는 두 가지 방식의 상용법을 제공합니다. 특별한 이유가 없다면 반드시 handlerKey를 사용하는 방식을 사용하세요. 자세한 사용법은 뒤에서 다시 살펴보도록 하겠습니다.

```java
/**
 * 타이머를 추가
 *
 * @param interval      지연 간격
 * @param timeUnit      시간 단위
 * @param times         반복 횟수(0 == 무한반복)
 * @param handlerKey    핸들러 키 전달
 * @param isTickFromRun false 이면 타이머는 가장 최근의 구동 시점 기준으로 interval 후에 구동된다. true 이면 타이머는 등록 시점 기준으로 interval 주기로 반복
 * @return 등록된 타이머 반환
 */
public Timer addTimer(int interval, TimeUnit timeUnit, int times, String handlerKey, boolean isTickFromRun)

/**
 * 타이머를 추가
 *
 * @param interval      지연 간격
 * @param timeUnit      시간 단위
 * @param times         반복 횟수(0 == 무한반복)
 * @param handler       등록할 TimerHandler
 * @param isTickFromRun false 이면 타이머는 가장 최근의 구동 시점 기준으로 interval 후에 구동된다. true 이면 타이머는 등록 시점 기준으로 interval 주기로 반복
 * @return 등록된 타이머 반환
 */
public Timer addTimer(int interval, TimeUnit timeUnit, int times, TimerHandler handler, boolean isTickFromRun)
```

각각의 매개변수에 대한 설명은 다음의 표와 같습니다.

| 이름 | 설명 | 예시 |
| --- | --- | --- |
| Interval | 호출 시간 간격 | 1 |
| TimeUnit | 호출 시간 단위 | TimeUnit.SECONDS<br>TimeUnit.MILLISECONDS |
| Times | 호출 횟수 | 20 |
| TimerHandler | 호출 함수 | 위의 TimerHandler 참고 |
| TimerHandlerKey | 호출 함수 Key<br>User/Room에서 Transfer해야 하는 Timer일 경우 사용 | timerHandlerKey |
| IsTickFromRun | 다음 Timer 호출 시간의 기준을 바로 전 실행된 Timer의 시간으로 할 것인지Timer가 등록된 시간을 기준으로 할 것인지를 결정<br><br>false이면 <br>  - Timer가 등록된 시간을 기준으로 호출<br>  - 예를 들어, 1초 주기의 타이머는 직전의 타이머가 부하로 인해 0.9 초 밀려서  0.1초 전에 호출되었더라도 타이머 등록 시간 기준으로 바로 0.1초 후에 다시 호출됨 <br>  - 정해진 시간 내의 호출 횟수가 중요할 경우 추천<br>  - 부하가 걸려도 호출 횟수는 보장이 되지만, 한꺼번에 밀린 타이머 처리가 동시에 여러번 호출 될 수 있음<br>  - 사용 예시: 1초에 한번식 버프를 거는 아이템<br><br>true이면<br>  - 바로전 실행된 Timer의 시간을 기준으로 호출<br>  - 예를 들어 1초 주기의 타이머는 이전 타이머가 처리된 시간을 기준으로 1초 후에 다시 호출됨<br>  - 호출 횟수의 정확도보다 서버 부하에 맞춰 유연하게 호출되는 것이 중요할 경우 추천<br>  - 부하가 걸리면 전체적인 호출 횟수는 줄어즐 수 있지만, 호출이 몰려서 발생하지는 않음<br>  - 사용 예시: 주기적인 DB 쿼리 | true/false |

### 1.3. 타이머 제거

엔진에 등록된 타이머는 언제든 제거할 수 있습니다. 이 때, removeTimer API를 사용합니다. 타이머 제거를 위해 등록할 때 반환받은 타이머 객체를 잘 저장하고 있어야 합니다.

```java
// 반환값인 Timer 객체를 잘 보관합니다.
Timer timer = addTimer(1, TimeUnit.SECONDS, 20, "sampleTimerHandler1", false);

// 보관해둔 Timer 객체를 사용하여 엔진에서 제거합니다.
removeTimer(timer);
```

## 2. 전역 타이머

앞서 우리는 [전송 가능 객체](server-08-object-transfer)에서 객체의 전송에 대해 살펴 보았습니다. 기본적으로 게임 유저와 방 객체는 언제든 게임 노드 사이에서 전송될 수 있습니다. 그러므로 이러한 유저와 방 객체에 등록한 타이머도 함께 전송될 수 있어야 최선입니다. 지금 설명하는 전역 타이머가 바로 이러한 요구사항을 만족하는 안전한 사용법입니다. 전체 서버에서 유저나 방 객체가 어느 노드로 전송되든 이 타이머는 항상 자동으로 따라다닙니다. 만일 유저나 방 객체에 타이머를 등록할 경우에는 반드시 이 사용법을 따라주시기 바랍니다.

### 2.1. 타이머 등록

유저와 방은 타이머를 등록할 수 있도록 onRegisterTimerHandler 콜백을 호출합니다. 이 때, 사용자는 임의의 문자열 키와 그에 대응는 TimerHandler를 등록할 수 있습니다. 타이머 등록은 반드시 이 콜백에서만 유효합니다.

```java
@Override
public void onRegisterTimerHandler() {

    registerTimerHandler("transferableTimerHandler1", transferRoomTimerHandler1());

    registerTimerHandler("transferableTimerHandler2", (timerObject, object) -> {
        GameRoom room = (GameRoom) object;
        ...
    });
}
```

### 2.2. Timer 추가

이제 앞에서 등록한 타이머를 엔진에 추가할 차례입니다. 등록할 때 사용한 문자열 키를 이용해서 추가합니다. 타이머는 엔진에 추가해야 비로소 동작합니다.

```java
public class GameRoom extends BaseRoom<GameUser> {
    ...
    @Override
    public final boolean onCreateRoom(final GameUser user, final Payload payload, Payload outPayload) throws SuspendExecution {
        ...
        // 1초마다 transferRoomTimerHandler1 Timer를 20번 호출
        addTimer(1, TimeUnit.SECONDS, 20, "transferableTimerHandler1", false);
        // 2초마다 transferRoomTimerHandler1 Timer 호출
        addTimer(2, TimeUnit.SECONDS, 0, "transferableTimerHandler2", false);
    }
}
```

이러한 타이머에 대한 제거 방법은 앞서 살펴본 것과 동일합니다.

## 3. 지역 타이머

지역 타이머는 전역 타이머와 달리 하나의 노드 내에서만 유효한 타이머입니다. 이 타이머는 유저와 같은 객체가 전송되더라도 함께 따라가지 않습니다. 그러므로 전송된 후 사용자가 알아서 다시 생성/추가 해주어야 합니다.

### 3.1. 타이머 추가

타이머의 추가 방법은 거의 흡사합니다. addTimer API를 사용하여 원하는 시간값과 TimerHandler를 전달하면 끝입니다. 전역 타이머와 달리 별도의 등록 절차는 없습니다. 만일 지역 타이머를 추가해둔 상태에서 해당 타이머를 사용하는 객체가 다른 노드로 전송되면 이 타이머는 제거됩니다. 즉, 새롭게 전송된 노드에서 해당 객체는 새롭게 타이머를 추가해주어야 합니다. 이런 경우에는 전역 타이머를 사용하는 것이 좋습니다. 지역 타이머는 타이머가 전송될 필요 없을 경우에 가장 쉽고 간단한 방법입니다.

```java
@Override
public void onInit() throws SuspendExecution {

    addTimer(1, TimeUnit.SECONDS, 0, nonTransferableTimerHandler1(), false);

    addTimer(1, TimeUnit.SECONDS, 0, new TimerHandler() {
        @Override
        public void onTimer(Timer timer, Object object) throws SuspendExecution {
            ...
        }
    }, false);
}
```

타이머의 제거 방법은 앞서 살펴본 것과 동일합니다.
