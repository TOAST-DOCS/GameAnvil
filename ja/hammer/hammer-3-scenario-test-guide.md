## Game > GameAnvil > 테스트 개발 가이드 > 시나리오 테스트 개발 가이드

## 대규모 부하 테스트와 이를 위한 시나리오 작성

GameHammer는 대규모 부하 테스트를 위해 대량의 커넥션을 동시에 처리할 수 있는 기능을 제공합니다. 그리고 복잡한 테스트를 좀 더 편하게 관리할 수 있도록 상태 기반의 시나리오 테스트를 지원합니다. 

### 시나리오 테스트 작성 예제

여기에서는 다음과 같은 간단한 시나리오 테스트 작성 예제를 다룹니다.

![img](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

우선 각 시나리오를 수행할 주체가 될 ScenarioActor를 정의합니다.

```java
public static class TestActor extends ScenarioActor<TestActor> {
}
```

각 상태별로 State를 상속 받은 클래스를 정의합니다.  

```java
public class StateA extends State<TestActor> {

    @Override
    protected void onScenarioTestStart(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onScenarioTestStart " + getStateName());
    }

    @Override
    protected void onEnter(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onEnter " + getStateName());
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
    }

    @Override
    protected void onExit(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onExit " + getStateName());
    }
}
```

상태를 관리할 ScenarioMachine을 생성합니다. 그리고 앞서 정의한 StateA, StateB, StateC 객체를 생성해 ScenarioMachine에 추가합니다. 

```java
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());

sceanrio.editState(StateA.class) // StateA에서는
    .addActionOnEnter(changeState(StateB.class), (sceanrioActor) -> new Random().nextBoolean()) // 랜덤으로 StateB로 이동
    .addActionOnEnter(changeState(StateC.class)) // 나머지 경우에는 StateC로 이동
    .endEdit();

scenario.
    .editState(StateB.class) // StateB에서는
    .addActionOnEnter(changeState(StateC.class)) // 항상 StateC로 이동
    .endEdit();

scenario
    .editState(StateC.class) // StateC에서는
    .addActionOnEnter(finishWithSuccess(), (scenarioActor) -> new Random().nextBoolean()) // 랜덤으로 성공 종료로 처리
    .addActionOnEnter(finishWithFail()) // 나머지 경우는 실패 종료로 처리
    .endEdit();
```

이제 시나리오 테스트를 실행합니다.

```java
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

```java
public static class TestActor extends ScenarioActor<TestActor> {
    private int value1;
    private String value2;

    public void funcion() {
        // some code
    }
}
```

#### ChangeState

ScenarioActor를 현재 상태를 다른 상태로 변경합니다. 주의할 점은 `ChangeState()`가 호출된 시점에 바로 상태가 변경되는 것이 아니라는 점 입니다. 실제 상태가 변경되는 시점은 다음 메시지 루프의 시작 시점이며, 이때 현재 상태를 구현한 State 객체의 `onExit()`와 다음 상태를 구현한 State 객체의 `onEnter()`가 순차적으로 호출됩니다.

```java
scenarioActor.changeState(NextState.class);
```

#### Finish

ScenarioActor가 수행 중인 시나리오를 종료합니다. `ChangeState()` 와 마찬가지로 `Finish()`가 호출된 후 다음 메시지 루프의 시작 시점에 실행되며, 이때 현재 상태를 구현한 State 객체의 `onExit()`가 호출됩니다. 그리고 ScenarioActor가 수행된 횟수가 ScenarioLoopCount보다 적은 경우 시나리오를 다시 수행하여 ScenarioLoopCount만큼 수행할 때 까지 반복합니다. ScenarioLoopCount <= 0일 경우 지정된 TestTime(기본값: 30초) 동안 계속 반복합니다. boolean 값을 인자로 받아 시나리오가 성공으로 종료되었는지, 실패로 종료되었는지 구분하여 통계를 기록합니다.

```java
scenarioActor.finish(true);
```

#### Connection

ScenarioActor는 하나의 Connection를 가지고 있어 이를 이용해 GameAnvil 서버의 기능을 실행할 수 있습니다. 

```java
Connection connection = scenarioActor.getConnection();
```

시나리오 테스트에서는 기능 테스트에서처럼 Future를 이용해 테스트 코드를 작성할 경우, `Future.get()`에서 블록이 되기 때문에 여러 개의 테스트를 동시에 수행할 수 없습니다. 대신 콜백 방식의 API를 사용하여 동시에 수행하도록 할 수 있습니다. 기능 테스트에서 소개한 모든 API에는 대응하는 콜백 방식의 API가 제공되므로 시나리오 테스트에서는 이 콜백 방식 API를 사용하면 됩니다.

```java
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

```java
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

ScenarioActor가 각 상태로 변경할 때 마다 각 상태를 나타내는 State의 `onEnert()`가 호출되며 그 상태로 변경된 ScenarioActor가 인자로 넘어옵니다. 각 상태에서 빠져나갈 때는`onExit()`가 호출되며 그 상태에서 빠져나간 ScenarioActor가 인자로 넘어옵니다.  

##### ScenarioMachine

ScenarioMachine으로 하나의 시나리오를 정의하며, 하나의 시나리오는 여러 개의 상태와 상태 간의 전이로 정의됩니다. 

```java
ScenarioMachine<STATE, EVENT> scenario = new ScenarioMachine("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());
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

### 향상된 콜백 등록 기능

이 방식으로 등록한 리스너는 User 에이전트를 통해 등록한 리스너와 다르게 State의 종료 시점에 자동으로 정리되므로 onExit에서 리스너를 제거해 주는 동작을 할 필요가 없어집니다.

예제 코드에서는 request에 대해 콜백을 등록하고, 콜백에서 response를 처리하여 문자열을 리턴하도록 설정하였습니다. 리턴된 값은 ScanarioMachine에서 다음으로 이동할 State를 결정하는 데 쓰일 것입니다. 

```java
public class EchoState extends State<TestActor> {
    EchoReq.Builder echoReq = EchoReq.newBuilder();

    @Override
    protected String onEnter(TestActor scenarioActor) {
        int random = (int)Math.ceil(Math.random() * 100);
        echoReq.setData("EchoReq" + random);

        scenarioActor.request(echoReq.build(), this::echoResListener);
        return null;
    }

    public String echoResListener(PacketResult res, ScenarioActor actor) {
        TestActor scenarioActor = (TestActor)actor;
        if (res.isSuccess()) {
            return "Success";
        } else {
            return "Fail";
        }
    }

    @Override
    protected void onExit(TestActor scenarioActor) {
    }
}
```

또는 서버로부터 일방적으로 메시지를 받는 경우에는 어노테이션 방식으로 콜백을 등록하면 편리합니다. 아래와 같이 `@Listener`어노테이션을 사용하여 특정 패킷에 대해 어떻게 처리할지 정의할 수 있습니다.
어노테이션의 인자로 수신이 예상되는 메시지의 class를 설정하고, 해당 핸들러에 부착합니다.

```java
@Listener(SendFromServer.class)
@SuppressWarnings("unused")
public void sendFromServerListener(PacketResult packetResult, TestActor scenarioActor) {
    try {
        SendFromServer send = SendFromServer.parseFrom(packetResult.getStream());
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

리퀘스트에 대한 응답 핸들러를 등록할 때에도 어노테이션을 사용할 수 있습니다. 이 경우에는 콜백으로 서버에서 전송된 패킷이 전달되지만, 어노테이션의 request 인자로 리퀘스트 타입을 대신 명시해야 합니다.
서버에서 받는 타입에 대해서는 어노테이션에 명시하지 않습니다.

```java
@Listener(request = RequestToServer.class)
@SuppressWarnings("unused")
public void requestToServerListener(PacketResult res, TestActor scenarioActor) {
    if (!res.isSuccess()) {
        scenarioActor.finish(false);
    }
}
```

기타 User 에이전트에서 지원하는 기능들은 대부분 ScenarioActor에서 아래와 같이 어노테이션 부착 또는 메서드 레퍼런스를 이용한 콜백 등록 방식을 사용할 수 있습니다.

```java
@Listener
@SuppressWarnings("unused")
public String connectListener(ResultConnect resultConnect, TestActor scenarioActor) {
    if (ResultCodeConnect.CONNECT_SUCCESS == resultConnect.getResultCode()) {
        System.out.println("Connect Success!");
        return "Success";
    } else {
        System.out.println("Connect Fail!");
        return "Fail";
    }
}
```

아래는 지원하는 리스너 목록입니다. 아래 시그니처를 가진 메서드를 State에 선언한 후 `@Listener` 어노테이션을 부착하십시오.

* connect 

```java
public void listener(ResultConnec result, ScenarioActor actor);
```

* authentication  

```java
public void listener(ResultAuthentication result, ScenarioActor actor);
```

* login  

```java
public void listener(ResultLogin result, ScenarioActor actor);
```

* matchUserStart  

```java
public void listener(ResultMatchUserStart result, ScenarioActor actor);
```

* logout  

```java
public void listener(ResultLogout result, ScenarioActor actor);
```

* logout  

```java
public void listener(ResultLogout result, ScenarioActor actor);
```

* leaveRoom  

```java
public void listener(ResultLeaveRoom result, ScenarioActor actor);
```

* leaveRoom  

```java
public void listener(ResultLeaveRoom result, ScenarioActor actor);
```

* createRoom  

```java
public void listener(ResultCreateRoom result, ScenarioActor actor);
```

* namedRoom  

```java
public void listener(ResultNamedRoom result, ScenarioActor actor);
```

* joinRoom  

```java
public void listener(ResultJoinRoom result, ScenarioActor actor);
```

* matchUserCancel  

```java
public void listener(ResultMatchUserCancel result, ScenarioActor actor);
```

* matchPartyStart  

```java
public void listener(ResultMatchPartyStart result, ScenarioActor actor);
```

* matchPartyCancel  

```java
public void listener(ResultMatchPartyCancel result, ScenarioActor actor);
```

* matchRoom  

```java
public void listener(ResultMatchRoom result, ScenarioActor actor);
```

* getChannelInfo  

```java
public void listener(ResultChannelInfo result, ScenarioActor actor);
```

* getAllChannelInfo  

```java
public void listener(ResultAllChannelInfo result, ScenarioActor actor);
```

* getChannelCountInfo  

```java
public void listener(ResultChannelCountIn result, ScenarioActor actor);
```

* getAllChannelCountInfo  

```java
public void listener(ResultAllChannelCoun result, ScenarioActor actor);
```

* moveChannel  

```java
public void listener(ResultMoveChannel result, ScenarioActor actor);
```

* snapshot  

```java
public void listener(ResultSnapshot result, ScenarioActor actor);
```

* request  

```java
public void listener(PacketResult result, ScenarioActor actor);
```

