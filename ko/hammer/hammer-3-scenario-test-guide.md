## Game > GameAnvil > 테스트 개발 가이드 > 시나리오 테스트 개발 가이드

### 시나리오 테스트란?

시나리오 테스트란, 미리 정해진 규칙 대로 서버에 부하를 가한 뒤 TPS 등 성능과 관련된 지표를 얻는 테스트를 말합니다. 여기에서 테스트를 진행하는 규칙을 시나리오라고 합니다. 또, 서버에 부하를 가하기 위해서는 다수의 커넥션을 생성하고 유지해야 하는데, 이 커넥션 각각을 시나리오 액터 라고 부릅니다.

시나리오는 필요에 따라서 구성하여 실행할 수 있습니다. 시나리오 구성은 스테이트 기반으로 이루어 집니다. 시나리오에는 다수의 스테이트가 존재합니다. 각 스테이트는 시나리오 액터가 수행할 동작에 대해 정의하고 있습니다. 시나리오 액터는 자신의 스테이트에 따라 행동합니다. 시나리오 액터는 한 번에 이 중 하나의 스테이트만 가질 수 있습니다. 그리고 특정 조건에 따라 또 다른 스테이트로 이동할 수 있습니다. 이러한 이동에 필요한 조건이나 스테이트 간의 연결 정보도 시나리오에서 정의합니다.

스테이트가 시나리오 액터가 수행할 동작에 대해서 정의하는 방법은, 스테이트 첫 진입 시점에 사용자가 정의한 코드를 삽입하는 것입니다. 보통은 서버에 대해 어떤 요청을 보내는 코드를 삽입합니다. 그리고 요청이 처리 완료 되어 응답이 도착하면 다른 스테이트로 이동하도록 하는 방식을 가장 많이 사용합니다.

간단한 예제를 통해 시나리오와 스테이트, 시나리오 액터를 구성하는 방법을 알아보겠습니다. 스테이트 세 개를 가지고 있는 시나리오를 구성해보겠습니다. 먼저, 처음 스테이트 A에서 특정 동작을 수행합니다. 그리고 이 동작의 성공 여부에 따라, 성공이면 스테이트 B, 실패하면 스테이트 C로 이동합니다. 그림으로 나타내면 아래와 같습니다.

![img](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

우선 각 시나리오를 수행할 주체가 될 시나리오 액터를 정의합니다. ScenaroActor를 정의하려면 먼저 게임 해머가 제공하는 ScenarioActor 클래스를 상속한 클래스를 생성합니다. 그리고 내부에는 시나리오 액터가 유지해야할 데이터를 정의 하면 됩니다. 이 예제에서는 시나리오 성공 여부를 저장하도록 해보겠습니다.

```java
public static class TestActor extends ScenarioActor<TestActor> {
    private boolean success;

    public void setSuccess(boolean value) {
        this.success = value
    }

    public boolean getSuccess() {
        return this.success;
    }

    public void reset() {
        this.success = false;
    }
}
```

다음은 스테이트를 정의합니다. 먼저 시작 스테이트를 정의해 보겟습니다. 시나리오 액터를 정의할 때와 마찬가지로, 게임 해머가 제공하는 State를 상속한 클래스를 생성합니다. 이 때 타입 파라미터로 아까 정의한 시나리오 액터 클래스를 넣습니다.

각 스테이트에서는 시나리오 액터의 스테이트 진입 시점과 진출 시점에 호출되는 함수를 오버라이딩 할 수 있도록 제공합니다. 또한, 이 스테이트가 시나리오에서 처음으로 진행 되는 스테이트 일 경우에만 호출 되는 함수인 onScenarioTestStart도 제공합니다. 각 함수가 호출 되는 시점을 알 수 있도록, 모든 함수에서 로그를 출력하도록 설정하겠습니다.

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
```

나머지 스테이트 B, C도 생성합니다.

```java
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

이제 스테이트간 이동을 위해서 각 클래스를 수정해보겠습니다.

먼저, 첫 번째 스테이트인 StateA에서는 동작 성공/실패 여부에 따라 각각 다른 스테이트로 이동 해야 합니다. 동작 성공/실패 여부는 예시를 위해서 랜덤 값으로 결정하겠습니다. 다른 스테이트로의 이동은 changeState 함수를 이용합니다.

```java
public class StateA extends State<TestActor> {

    @Override
    protected void onScenarioTestStart(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onScenarioTestStart " + getStateName());
    }

    @Override
    protected void onEnter(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onEnter " + getStateName());

        scenarioActor.reset();

        if (new Random().nextBoolean()) {
            scenarioActor.changeState(StateB);
        } else {
            scenarioActor.changeState(StateC);
        }
    }

    @Override
    protected void onExit(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onExit " + getStateName());
    }
}
```

State B에서는 다음 실행 시점에 곧바로 StateC로 이동하도록 설정합니다.

```java
public class StateB extends State<TestActor> {

    @Override
    protected void onEnter(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onEnter " + getStateName());

        sceanrioActor.setSuccess(true);

        scenarioActor.changeState(StateC);
    }

    @Override
    protected void onExit(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onExit " + getStateName());
    }
}
```

State C에서는 여러 스테이트를 계속해서 이동할 수 있도록 시나리오를 종료 처리 하겠습니다. 시나리오가 종료 되면, 시나리오 액터는 처음 스테이트로 이동해서 테스트를 계속합니다. 시나리오 종료 시에 시나리오를 평가하여 정상적으로 진행 되어서 종료 되었는지 여부를 기록할 수 있습니다. finish() 함수를 통해 시나리오를 종료하고 성공 여부를 기록합니다.

```java
public class StateC extends State<TestActor> {

    @Override
    protected void onEnter(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onEnter " + getStateName());

        scenarioActor.finish(scenarioActor.getSuccess());
    }

    @Override
    protected void onExit(TestActor scenarioActor) {
        System.out.println("ScenarioActor " + scenarioActor.getIndex() + " - onExit " + getStateName());
    }
}
```

이제 정의한 상태들을 관리할 ScenarioMachine을 생성합니다. 그리고 앞서 정의한 StateA, StateB, StateC 객체를 생성해 ScenarioMachine에 추가합니다.

```java
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());
```

이제 작성한 시나리오를 가지고 시나리오 테스트를 실행합니다. Tester 객체를 생성하며 테스트에 사용할 유저 수와, 시나리오가 종료 되었을 때 몇 번까지 반복할 것인지를 설정합니다. 그 다음, 시나리오 테스트 객체를 생성하고 작성한 시나리오를 인자로 제공합니다. 시나리오 테스트 객체의 start() 함수를 호출하면서 인자로 방금 생성한 Tester, 시나리오 액터 클래스, 첫 스테이트를 인자로 넣으십시오.

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

실행 결과는 매번 다르겠지만 대략 아래와 비슷합니다.

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

시나리오 작성을 위해 각 스테이트를 반드시 클래스를 생성할 필요는 없습니다. 모든 State 클래스를 상속하고, 아래와 같이 시나리오 상에서 각 State의 동작을 한번에 간편하게 정의할 수 있습니다.

```java
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState("StateA");
scenario.addState("StateB");
scenario.addState("StateC");

sceanrio
    .startEdit("StateA")
    .addActionOnEnter(changeState("StateB"), (sceanrioActor) -> new Random().nextBoolean())
    .addActionOnEnter(changeState("StateC"))
    .endEdit();

scenario.
    .startEdit("StateB")
    .addActionOnEnter((scenarioActor) -> scenarioActor.setSuccess(true));
    .addActionOnEnter(changeState("StateC"))
    .endEdit();

scenario
    .editState("StateC")
    .addActionOnEnter(finishWithSuccess(), (scenarioActor) -> scenarioActor.getSuccess())
    .addActionOnEnter(finishWithFail())
    .endEdit();
```

### 시나리오 테스트 작성 예시

아래 예시는 실제 서버에 부하를 인가하도록 작성된 예시입니다.

| 샘플 서버                                                                                                           | 샘플 테스터                                                                                                          |
|-----------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| [GameAnvil Scenario Server](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvilScenarioServer.zip) | [GameAnvil Scenario Tester](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvilScenarioTester.zip) |

### 액션

시나리오 중에 엔진에 요청 등 수행할 수 있는 동작을 액션이라고 합니다. 액션은 시나리오 중에 특정 시점에 조건과 함께 결합하여 등록할 수 있습니다.

#### 스테이트 이동

```java
changeState("StateA")
```

```java
changeState(StateA.class)
```

#### 시나리오 종료

```java
bool isSuccessful = true;
ScenarioAction.finish(isSuccessful);
```

#### 연결 진행

```java
ScenarioAction.connect()
```

#### 인증 진행

```java
ScenarioAction.authenticate()
```

#### 로그인 진행

```java
ScenarioAction.login()
```

#### 유저 매치메이킹 진행

```java
ScenarioAction.matchUserStart()
```

#### 룸 매치메이킹 진행

```java
ScenarioAction.matchRoom()
```

#### 방 퇴장 진행

```java
ScenarioAction.leaveRoom()
```

#### 로그아웃 진행

```java
ScenarioAction.logout()
```

#### 네임드 룸 동작 요청 진행

```java
ScenarioAction.namedRoom()
```

#### 파티 매치메이킹 진행

```java
ScenarioAction.matchPartyStart()
```

#### 파티 매치메이킹 취소 진행

```java
ScenarioAction.matchPartyCancel()
```

#### 채널 정보 요청 진행

```java
ScenarioAction.getChannelInfo()
```

#### 모든 채널 정보 요청 진행

```java
ScenarioAction.getAllChannelInfo()
```

#### 채널 유저 수, 룸 수 요청 진행

```java
ScenarioAction.getChannelCountInfo()
```

#### 모든 채널의 유저 수, 룸 수 요청 진행

```java
ScenarioAction.getAllChannelCountInfo()
```

#### 채널 이동 진행

```java
ScenarioAction.moveChannel()
```

#### 스냅샷 요청 진행

```java
ScenarioAction.snapshot()
```

### 액션 등록

아래는 액션을 등록하는 메서드 목록입니다.

#### 시나리오 진입 시점

시나리오 진입 시점에 실행할 액션을 등록할 수 있습니다.

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"));
```

#### 시나리오 이동 시점

시나리오 이동 시점에 실행할 액션을 등록할 수 있습니다.

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnExit(scenarioActor -> scenarioActor.resetChannle());
```

#### 패킷 수신 시점

패킷 수신 시점에 실행할 액션을 등록할 수 있습니다.

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"));
```


#### 타이머 호출 시점

타이머 호출 시점에 실행할 액션을 등록할 수 있습니다.

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .setActionInTimer("TimerName", changeState("OtherStateNameToEnter"));
```

### 조건 기능

조건에 따라 액션 수행 여부를 설정할 수 있습니다.

#### 항상

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), always());
```

#### 결과 성공시

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnReceive(ResultConnect.class, changeState("OtherStateNameToEnter"), ifSuccess());
```

#### 결과 실패시

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnReceive(ResultConnect.class, changeState("OtherStateNameToEnter"), ifSuccess());
```

#### 조건 반전

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), NOT(scenarioActor -> scenarioActor.valid()));
```

#### 조건 결합

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), AND(scenarioActor -> scenarioActor.valid(), ifSuccess());
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

| 리스너 이름 | 시그니처 |
| --- | --- |
| connect | `public void listener(ResultConnect result, ScenarioActor actor);` |
| authentication | `public void listener(ResultAuthentication result, ScenarioActor actor);` |
| login | `public void listener(ResultLogin result, ScenarioActor actor);` |
| logout | `public void listener(ResultLogout result, ScenarioActor actor);` |
| createRoom | `public void listener(ResultCreateRoom result, ScenarioActor actor);` |
| joinRoom | `public void listener(ResultJoinRoom result, ScenarioActor actor);` |
| namedRoom | `public void listener(ResultNamedRoom result, ScenarioActor actor);` |
| leaveRoom | `public void listener(ResultLeaveRoom result, ScenarioActor actor);` |
| matchRoom | `public void listener(ResultMatchRoom result, ScenarioActor actor);` |
| matchUserStart | `public void listener(ResultMatchUserStart result, ScenarioActor actor);` |
| matchUserCancel | `public void listener(ResultMatchUserCancel result, ScenarioActor actor);` |
| matchPartyStart | `public void listener(ResultMatchPartyStart result, ScenarioActor actor);` |
| matchPartyCancel | `public void listener(ResultMatchPartyCancel result, ScenarioActor actor);` |
| getChannelInfo | `public void listener(ResultChannelInfo result, ScenarioActor actor);` |
| getAllChannelInfo | `public void listener(ResultAllChannelInfo result, ScenarioActor actor);` |
| getChannelCountInfo | `public void listener(ResultChannelCountInfo result, ScenarioActor actor);` |
| getAllChannelCountInfo | `public void listener(ResultAllChannelCountInfo result, ScenarioActor actor);` |
| moveChannel | `public void listener(ResultMoveChannel result, ScenarioActor actor);` |
| snapshot | `public void listener(ResultSnapshot result, ScenarioActor actor);` |
| request | `public void listener(PacketResult result, ScenarioActor actor);` |
