## Game > GameAnvil > 테스트 개발 가이드 > 시나리오 테스트 개발 가이드

## 대규모 부하 테스트와 이를 위한 시나리오 작성

GameHammer는 대규모 부하테스트를 위해 대량의 커넥션을 동시에 처리할 수 있는 기능을 제공합니다. 그리고 복잡한 테스트를 좀 더 편하게 관리할 수 있도록 상태 기반의 시나리오 테스트를 지원합니다. 

### 시나리오 테스트 작성 예제

다음과 같은 간단한 시나리오 테스트를 작성해 봅시다.

![img](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

우선 각 시나리오를 수행할 주체가 될 ScenarioActor를 정의합니다.

```
public static class TestActor extends ScenarioActor<TestActor> {
}
```

각 상태별로 State를 상속받은 클래스를 정의합니다.  

```
public class StateA extends State<TestActor> {

    @Override
    protected void onScenarioTestStart(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onScenarioTestStart " + getStateName());
    }

    @Override
    protected void onEnter(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onEnter " + getStateName());
        scenarioActor.changeState(new Random().nextBoolean() ? StateB.class : StateC.class);
    }

    @Override
    protected void onExit(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onExit " + getStateName());
    }
}
public class StateB extends State<TestActor> {

    @Override
    protected void onEnter(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onEnter " + getStateName());
        scenarioActor.changeState(StateC.class);
    }

    @Override
    protected void onExit(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onExit " + getStateName());
    }
}
public class StateC extends State<TestActor> {

    @Override
    protected void onEnter(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onEnter " + getStateName());
        scenarioActor.finish(true);
    }

    @Override
    protected void onExit(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onExit " + getStateName());
    }
}
```

상태를 관리할 ScenarioMachine을 생성합니다. 그리고 앞서 정의한 StateA, StateB, StateC 객체를 생성해 ScenarioMachine에 추가합니다. 

```
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());
```

이제 시나리오 테스트를 실행합니다.

```
Tester tester = Tester.newBuilder()
    .setUserCount(2)
    .setScenarioLoopCount(1)
    .Build();

ScenarioTest<TestActor> scenarioTest = new ScenarioTest<>(scenario);
scenarioTest.start(tester,
                   TestActor.class,
                   StateA.class
                  );
```

실행 결과는 Random을 사용하기 때문에 매번 다르겠지만 대략 아래와 비슷합니다.

```
ScenarioActor 0 - onScenarioTestStart StateA
ScenarioActor 0 - onEnter StateA
ScenarioActor 0 - onExit StateA
ScenarioActor 0 - onEnter StateC
ScenarioActor 0 - onExit StateC
ScenarioActor 1 - onScenarioTestStart StateA
ScenarioActor 1 - onEnter StateA
ScenarioActor 1 - onExit StateA
ScenarioActor 1 - onEnter StateB
ScenarioActor 1 - onExit StateB
ScenarioActor 1 - onEnter StateC
ScenarioActor 1 - onExit StateC
```

## 시나리오 테스트 기본 개념

시나리오 테스트는 대규모 부하 테스트를 지원하기 위한 개념으로, 시나리오 테스트를 실행할 때 동시에 실행할 ScenarioActor 수, 시작 상태, 최대 실행 시간을 지정해서 실행합니다. 

### ScenarioActor

동시에 실행되는 각각의 주체가  ScenarioActor입니다. 동시에 실행할 개수를 100으로 지정할 경우 100개의 ScenarioActor 객체가 생성되어 각각 시나리오를 수행하게 됩니다. ScenarioActor는 하나의 Connection을 가지고 있어 이를 이용해 GameAnvil 서버의 기능을 실행할 수 있습니다. ScenarioActor를 상속 구현하면서 시나리오 수행에 필요한 기능이나 설정 등을 추가하여 이용할 수 있습니다. `changeState()`를 이용해 현재 상태를 다른 상태로 변경할 수 있고, `finish()`를 이용해 시나리오를 종료할 수 있습니다. 

```
public static class TestActor extends ScenarioActor<TestActor> {
    private int value1;
    private String value2;

    public void funcion() {
        // some code
    }
}
```

#### ChangeState

ScenarioActor를 현재 상태를 다른 상태로 변경합니다. 주의 할점은 `ChangeState()`가 호출된 시점에 바로후 상태가 변경되는것이 아니라는 점 입니다. 실제 상태가 변경되는 시점은 다음 메시지 루프의 시작시점이며, 이때 현재 상태를 구현한 State 객체의 `onExit()`와 다음 상태를 구현한 State 객체의 `onEnter()`가 순차적으로 호출됩니다.

```
scenarioActor.changeState(NextState.class);
```

#### Finish

ScenarioActor가 수행 중인 시나리오를 종료합니다. `ChangeState()` 와 마찬가지로 `Finish()`가 호출된 후 다음 메시지 루프의 시작 시점에 실행되며, 이때 현제 상태를 구현한 State 객체의 `onExit()`가 호출됩니다. 그리고 ScenarioActor가 수행된 횟수가 ScenarioLoopCount보다 적은 경우 시나리오를 다시 수행하여 ScenarioLoopCount만큼 수행할 때 까지 반복합니다. ScenarioLoopCount <= 0일 경우 지정된 TestTime(기본값: 30초) 동안 계속 반복합니다. boolean값을 인자로 받아 시나리오가 성공으로 종료되었는지, 실패로 종료되었는지 구분하여 통계를 기록합니다.

```
scenarioActor.finish(true);
```

#### Connection

ScenarioActor는 하나의 Connection를 가지고 있어 이를 이용해 GameAnvil 서버의 기능을 실행할 수 있습니다. 

```
Connection connection = scenarioActor.getConnection();
```

시나리오 테스트에서는 기능 테스트에서처럼 Future를 이용해 테스트 코드를 작성할 경우, `Future.get()`에서 블록이 되기 때문에 여러개의 테스트를 동시에 수행할 수 없습니다. 대신 콜백 방식의 API를 사용하여 동시에 수행하도록 할 수 있습니다. 기능 테스트에서 소개한 모든 API에는 대응하는 콜백 방식의 API가 제공되므로 시나리오 테스트에서는 이 콜백 방식 API를 사용하면 됩니다.

```
Connection connection = scenarioActor.getConnection();
connection.connect(new RemoteInfo("127.0.0.1", 11200), resultConnect -> {
    if (ResultCodeConnect.CONNECT_SUCCESS == resultConnect.getResultCode()) {
        System.out.println("Connect success.");
        scenarioActor.changeState(NextState.class);
    } else {
        System.out.println("Connect success.");
        scenarioActor.finish(false);
    }
});
```

### State

State를 상속 구현하여 각 상태를 나타내는 State를 정의합니다. 

```
public class StateA extends State<TestActor> {

    @Override
    protected void onScenarioTestStart(RPSActor scenarioActor) {
    }

    @Override
    protected void onEnter(ScenarioActor<STATE, EVENT> scenarioActor) {
    }

    @Override
    protected void onExit(ScenarioActor<STATE, EVENT> scenarioActor) {
    }
}
```

ScenarioActor가 각 상태로 변경할 때 마다 각 상태를 나타내는 State의 `onEnert()`가 호출되며 그 상태로 변경된 ScenarioActor가 인자로 넘어옵니다. 각 상태에서 빠져나갈때는`onExit()`가 호출되며 그 상태에서 빠져나간 ScenarioActor가 인자로 넘어옵니다.  

##### ScenarioMachine

ScenarioMachine으로 하나의 시나리오를 정의하며, 하나의 시나리오는 여러 개의 상태로 정의됩니다. 

```
ScenarioMachine<STATE, EVENT> scenario = new ScenarioMachine("Sample A");
scenario.setState(new StateA(scenario, STATE.A));
scenario.setState(new StateB(scenario, STATE.B));
scenario.setState(new StateC(scenario, STATE.C));
```

##### ScenarioTest

먼저 Tester를 생성합니다. 이때 동시에 실행할 ScenarioActor의 수, 반복할 횟수, 테스트 시간 등을 옵션으로 지정할 수 있습니다.  그리고 ScenarioMachine을 인자로 받아 ScenarioTest를 생성합니다. 이제 앞서 생성한 Tester, ScenarioActo의 Class, 시작 상태를 나타내는 State의 Class를 넘겨 실행합니다. 테스트는 모든 ScenarioActor가 지정된 횟수만큼 반복하거나 지정한 테스트 시간이 될 때까지 수행됩니다. 

```
Tester tester = Tester.newBuilder()
    .setActorCount(3)
    .setScenarioLoopCount(2)
    .Build();

ScenarioTest<TestActor> scenarioTest = new ScenarioTest<>(scenario);
scenarioTest.start(tester,
                   TestActor.class,
                   StateA.class
                  );
```

테스트를 완료한 후 `printStatistics()`를 이용해 테스트 결과를 얻을 수 있습니다.

```
logger.info(scenarioTest.printStatistics("Finished"));
```
```
## State - Statistics ##
                                         StateC                      StateA                      StateB
Total   |                             6                           6                           3
##################

## Packets - Statistics ##
TPS
    Total   0
    Avg 0
    Max 0
    Time    0
PPS
    Total   0
    Avg 0
    Max 0
    Time    0
Heap Memory Avg : NaN MBytes
## Timeout Total : 0
##################
## Packet Count
## total : 0

## Success : 6
## Fail : 0
## Disconnected : 0
## ForceDisconnected : 0
## SocketException : 0
```