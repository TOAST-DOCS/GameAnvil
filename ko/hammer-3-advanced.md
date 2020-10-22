## Game > GameAnvil > 테스트 개발 가이드 > 고급 주제

### 대규모 부하 테스트와 이를 위한 시나리오 작성

GameHammer는 대규모 부하테스트를 위해 대량의 Connection을 동시에 처리할 수 있는 기능을 제공합니다. 그리고 복잡한 테스트를 좀더 편하게 관리할 수 있도록 상태 기반의 시나리오를 지원하고 있습니다. 

#### 시나리오 작성 예제.

다음과 같은 간단한 시나리오를 작성해 봅시다.

![](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

우선 각 상태를 나타내는 Enum을 정의합니다.

```java
public enum STATE{
	A,B,C
}
```

발생할 수 있는 이벤트를 나타내는 Enum을 정의합니다.

```java
public enum EVENT {
	Success, Next, Fail
}
```

각 상태별로 State<STATE, EVENT>를 상속받은 클래스를 정의합니다.  

```java
public class StateA extends State<STATE, EVENT> {

    public StateA(ScenarioMachine<STATE, EVENT> scenarioMachine, STATE stateId) {
        super(scenarioMachine, stateId);
    }

    @Override
    protected void onEnter(ScenarioActor<STATE, EVENT> scenarioActor) {
        System.out.println("ScenarioActor" + scenarioActor.getUuid() + " - onEnter " + stateId);
        scenarioActor.takeEvent(new Random().nextBoolean() ? EVENT.Success : EVENT.Fail);
    }

    @Override
    protected void onExit(ScenarioActor<STATE, EVENT> scenarioActor) {
        System.out.println("ScenarioActor" + scenarioActor.getUuid() + " - onExit " + stateId);
    }
}
```

```java

public class StateB extends State<STATE, EVENT> {

    public StateB(ScenarioMachine<STATE, EVENT> scenarioMachine, STATE stateId) {
        super(scenarioMachine, stateId);
    }

    @Override
    protected void onEnter(ScenarioActor<STATE, EVENT> scenarioActor) {
        System.out.println("ScenarioActor" + scenarioActor.getUuid() + " - onEnter " + stateId);
        scenarioActor.takeEvent(EVENT.Next);
    }

    @Override
    protected void onExit(ScenarioActor<STATE, EVENT> scenarioActor) {
        System.out.println("ScenarioActor" + scenarioActor.getUuid() + " - onExit " + stateId);
    }
}
```

```java
public class StateC extends State<STATE, EVENT> {

    public StateC(ScenarioMachine<STATE, EVENT> scenarioMachine, STATE stateId) {
        super(scenarioMachine, stateId);
    }

    @Override
    protected void onEnter(ScenarioActor<STATE, EVENT> scenarioActor) {
        System.out.println("ScenarioActor" + scenarioActor.getUuid() + " - onEnter " + stateId);
    }

    @Override
    protected void onExit(ScenarioActor<STATE, EVENT> scenarioActor) {
        System.out.println("ScenarioActor" + scenarioActor.getUuid() + " - onExit " + stateId);
    }
}
```



 상태를 관리할 ScenarioMachine을 생성하고, 정의한 상태를 등록한 후, transition을 정의합니다.

```java
ScenarioMachine<STATE, EVENT> scenarioA = new ScenarioMachine("Sample A");
scenarioA.setState(new StateA(scenario, STATE.A));
scenarioA.setState(new StateB(scenario, STATE.B));
scenarioA.setState(new StateC(scenario, STATE.C));

scenarioA.setTransition(STATE.A, EVENT.Success, STATE.B);
scenarioA.setTransition(STATE.A, EVENT.Fail, STATE.C);
scenarioA.setTransition(STATE.B, EVENT.Next, STATE.C);
```

이제 시나리오 테스트를 실행합니다.

```java
Tester tester = Tester.newBuilder().Build();

ArrayList<ScenarioMachine<STATE, EVENT>> scenarioList = new ArrayList<>();
scenarioList.add(scenarioA);

ScenarioTest<STATE, EVENT> scenarioTest = new ScenarioTest<>(scenarioList);
scenarioTest.start(tester,
                   5,
                   STATE.A,
                   STATE.C,
                   (index, error) -> {
                       scenarioTest.setState(index, STATE.C);
                   },
                   100
                  );
```

실행결과는 Random을 사용하기때문에 매번 다르겠지만 대략 아래와 비슷합니다.

```
ScenarioActor0 - onEnter A
ScenarioActor0 - onExit A
ScenarioActor0 - onEnter C
ScenarioActor1 - onEnter A
ScenarioActor1 - onExit A
ScenarioActor1 - onEnter B
ScenarioActor1 - onExit B
ScenarioActor1 - onEnter C
ScenarioActor2 - onEnter A
ScenarioActor2 - onExit A
ScenarioActor2 - onEnter B
ScenarioActor2 - onExit B
ScenarioActor2 - onEnter C
ScenarioActor3 - onEnter A
ScenarioActor3 - onExit A
ScenarioActor3 - onEnter C
ScenarioActor4 - onEnter A
ScenarioActor4 - onExit A
ScenarioActor4 - onEnter C
```



#### 시나리오 기본 개념

시나리오는 대규모 부하 테스트를 지원하기 위한 개념으로, 시나리오를 실행할 때 동시에 실행할 ScenarioActor 수, 시작 State, 종료 State, 최대 실행 시간을 지정해서 실행합니다. 

##### ScenarioActor

동시에 실행되는 각각의 주체가  ScenarioActor입니다.  동시에 실행할 갯수를 100으로 지정할 경우 100개의 ScenarioActor가 각각 시나리오를 실행하게 됩니다. ScenarioActor는 하나의 Connector를 가지고 있어, 이를 이용해 GameAnvil 서버의 기능을 실행할 수 있습니다.



##### State

State<STATE, EVENT> 를 상속구현하여 각 State를 정의합니다. Type 파라메터 STATE는 각 State를 나타내는 고유의 값으로 enum을 정의해 사용하면 됩니다. 

Type 파라메터 EVENT는 각 State에 정의된 Transition을 구분하는데 사용하는 값으로 enum을 정의해 사용하면 됩니다. 

```java
public class StateA extends State<STATE, EVENT> {

    public StateA(ScenarioMachine<STATE, EVENT> scenarioMachine, STATE stateId) {
        super(scenarioMachine, stateId);
    }

    @Override
    protected void onEnter(ScenarioActor<STATE, EVENT> scenarioActor) {
    }

    @Override
    protected void onExit(ScenarioActor<STATE, EVENT> scenarioActor) {
    }
}
```

ScenarioActor가 각 State에 진입할 때 마다 각 State의 `onEnert`가 호출되며 그 State에 진입한 ScenarioActor가 인자로 넘어옵니다.  각 State에서 빠져나갈때는`onExit`가 호출되며 그 State에서 빠져나간 ScenarioActor가 인자로 넘어옵니다.  



##### ScenarioMachine

ScenarioMachine으로 하나의 시나리오를 정의하며, 하나의 시나리오는 여러개의 State와 각 State간의 전환를 표현하는 Transition으로 정의됩니다. 

```
ScenarioMachine<STATE, EVENT> scenarioA = new ScenarioMachine("Sample A");
scenarioA.setState(new StateA(scenario, STATE.A));
scenarioA.setState(new StateB(scenario, STATE.B));
scenarioA.setState(new StateC(scenario, STATE.C));

scenarioA.setTransition(STATE.A, EVENT.Success, STATE.B);
scenarioA.setTransition(STATE.A, EVENT.Fail, STATE.C);
scenarioA.setTransition(STATE.B, EVENT.Next, STATE.C);
```



##### ScenarioTest

ScenarioMachine이 담긴 ArrayList를 인자로 받아 ScenarioTest를 생성합니다.  그리고 `start()`에 동시에 실행할 ScenarioActor 수, 시작 State, 종료 State, 최대 실행 시간을 인자로 넘겨 테스트를 실행합니다.  테스르를 실행하면 이 리스트에 담긴 시나리오가 각 유저에게 순자로 배정되어 실행됩니다. 예를 들어 scenarioA, scenarioB 2개를 담아 실행하면서 유저수를 3으로 지정한다면 첫번째 ScenarioActor는 scenarioA, 두번째 ScenarioActor는 scenarioB, 세번째 ScenarioActor는 다시 scenarioA를 실행하게 됩니다. 테스트는 모든 ScenarioActor가 종료State에 진입하거나 최대실행시간이 될때 까지 수행됩니다. 

```java
ArrayList<ScenarioMachine<STATE, EVENT>> scenarioList = new ArrayList<>();
scenarioList.add(scenarioA);

ScenarioTest<STATE, EVENT> scenarioTest = new ScenarioTest<>(scenarioList);
scenarioTest.start(tester,
                   5,
                   STATE.A,
                   STATE.C,
                   (index, error) -> {
                       scenarioTest.setState(index, STATE.C);
                   },
                   100
                  );
```

테스트 완료후 다음 `printStatistics()`를 이용해 테스트 결과를 얻을 수 있습니다.

```
scenarioTest.printStatistics("Finished")
```



