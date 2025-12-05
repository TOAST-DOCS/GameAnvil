## Game > GameAnvil > Guide to Test Development > Guide to Scenario Test Development

### What is a scenario test?

Scenario tests are tests that obtain metrics related to performance after loading the server according to predefined rules. Here the rules for performing the test are called scenarios. In addition, to load the server, you must create and maintain multiple connections, each of which is called a scenario actor.

Scenarios can be configured and executed according to your needs. Scenario configuration is based on state. Scenarios have multiple states. Each state defines the action to be performed by the scenario actor. Scenario actors act according to their state. Scenario actors can only have one of these states at a time. And you can move to another state depending on certain conditions. Conditions required for such moves or connection information between states are also defined in the scenario.

The method to define the actions to be performed by the scenario actor by the state is to insert the user-defined code at the time of the first entry into the state. Usually, it inserts the code to send a request to the server. And when the request is processed and the response arrives, it is the most used method to get the request to go to another state.

Learn how to configure scenarios, states, and scenario actors with simple examples. Let me configure a scenario with three states. First, perform a particular action in state A first. And depending on whether this action is successful or not, if it succeeds, it will go to state B and if it fails, it will go to state C. The image shows the following:

![img](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

First, define the scenario actor to be the subject to execute each scenario. To define ScenaroActor, first create a class that inherits the ScenarioActor class provided by Game Hammer. And you can define the data that the scenario actor needs to maintain internally. In this example, let us save whether the scenario succeeds.

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

The following defines the state. First, we need to define the start state. Just as when defining scenario actors, it creates a class that inherits the state provided by GameHammer. At this time, the previously defined scenario actor class is added to the type parameter.

For each state, it is provided to overwrite the functions called at the stage entry time and when the stage is coming out of the scenario actor. It also provides onScenarioTestStart, a function that is called only when the state is the first state in the scenario to proceed. To allow us to know when each function is being called, we will set logs to be output from all functions.

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

Create the remaining states B, C.

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

Let me now modify each class to move between states.

First, in StateA, the first state, you must move to different states depending on whether the operation succeeds or fails. For example, we will determine whether the operation succeeds/fails with a random value. Move to another state uses the changeState function.

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

In State B, set it to go to State C at the next execution point.

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

In State C, we will finish the scenario to allow multiple states to continue to move. When the scenario ends, the scenario actor moves to the initial state to continue with the test. When the scenario is terminated, you can evaluate the scenario and register whether it has been terminated because it proceeds normally. Terminate the scenario and records the success or failure with the finish() function.

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

Create a ScenarioMachine to manage the states you now define. Then the previously defined StateA, StateB, and StateC objects are created and added to the ScenarioMachine.

```java
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());
```

Run a scenario test with the scenario you already wrote. It creates a Tester object and sets the number of users to use for the test and how many times to repeat when the scenario is terminated. Next, provide as an argument the scenario created and written by the scenario test object. Put the Tester, scenario actor class, and first state you just created with the argument as an argument by calling the start() function of the scenario test object.

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

The execution result will be different each time, but is approximately similar to the following:

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

You don't necessarily have to create a class for each state to create a scenario. You inherit all State classes, and you can easily define the actions of each state one by one in the scenario as shown below:

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

### Example of Writing a Scenario Test

The example below is an example written to authorize a load to a real server:

| Sample Server | Sample Tester |
|-----------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| [GameAnvil Scenario Server](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvilScenarioServer.zip) | [GameAnvil Scenario Tester](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvilScenarioTester.zip) |

### Action

Operations that can be performed during scenarios are called actions, such as requests to the engine. Actions can be registered at a certain point in time, combined with conditions.

#### Move Status

```java
changeState("StateA")
```

```java
changeState(StateA.class)
```

#### End Scenario

```java
bool isSuccessful = true;
ScenarioAction.finish(isSuccessful);
```

#### Progress in Connection

```java
ScenarioAction.connect()
```

#### Progress in Authentication

```java
ScenarioAction.authenticate()
```

#### Progress in Login

```java
ScenarioAction.login()
```

#### Progress in User Matchmaking

```java
ScenarioAction.matchUserStart()
```

#### Progress in Room Matchmaking

```
java ScenarioAction.matchRoom()
```

#### Proceed with Excluding Room

```
java ScenarioAction.leaveRoom()
```

#### Progress in Logout

```
java ScenarioAction.logout()
```

#### Proceed with Requesting Named Room Action

```
java ScenarioAction.namedRoom()
```

#### Progress in Party Matchmaking

```
java ScenarioAction.matchPartyStart()
```

#### Proceed with Canceling Party Matchmaking

```
java ScenarioAction.matchPartyCancel()
```

#### Proceed with Requesting Channel Information

```
java ScenarioAction.getChannelInfo()
```

#### Proceed with Requesting All Channel Information

```
java ScenarioAction.getAllChannelInfo()
```

#### Proceed with Requesting Number of Channel Users, Number of Rooms

```
java ScenarioAction.getChannelCountInfo()
```

#### Proceed with Requesting Number of Users for All Channels and Number of Rooms

```
java ScenarioAction.getAllChannelCountInfo()
```

#### Proceed with Moving Channel

```
java ScenarioAction.moveChannel()
```

#### Progress in Snapshot Request

```
java ScenarioAction.snapshot()
```

### Register Action

Below is a list of methods to register actions:

#### Scenario Entry Point

You can register the actions to be executed at the time the scenario enters.

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"));
```

#### Scenario Movement Point

You can register the actions to be executed at the time of scenario migration.

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnExit(scenarioActor -> scenarioActor.resetChannle());
```

#### Packet Reception Point

You can register the actions to be executed at the time the packet is received.

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"));
```


#### Timer Calling Point

You can register the actions to be executed at the time call time point.

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .setActionInTimer("TimerName", changeState("OtherStateNameToEnter"));
```

### Conditional Trigger

You can set whether to execute the action depending on the condition.

#### Always

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), always());
```

#### Result Success

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnReceive(ResultConnect.class, changeState("OtherStateNameToEnter"), ifSuccess());
```

#### Result Failure

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnReceive(ResultConnect.class, changeState("OtherStateNameToEnter"), ifSuccess());
```

#### Conditional Inversion

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), NOT(scenarioActor -> scenarioActor.valid()));
```

#### Conditional Combination

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), AND(scenarioActor -> scenarioActor.valid(), ifSuccess());
```

### Enhanced Callback Registration Feature

The listener registered in this way is automatically cleaned up at the end time of the state unlike the listener registered through the user agent, eliminating the need to do anything to remove the listener from onExit.

In the example code, you have registered a callback for the request, and set the response to return the string in the callback. The returned value will be used to determine the state to be moved to from ScanarioMachine.

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

Or if you receive messages unilaterally from the server, it is convenient to register callbacks via annotation. You can use `@Listener` quotes to define how to handle specific packets, as shown below.
Set the class of the message that is expected to be received with the argument of the annotation and attach it to the dealers.

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

Annotations can also be used when registering a response manipulator for a request. In this case, the packet sent from the server is delivered as a callback, but the request type must be specified instead of the request index in the annotation.
For the type received from the server, it is not specified in the annotation.

```java
@Listener(request = RequestToServer.class)
@SuppressWarnings("unused")
public void requestToServerListener(PacketResult res, TestActor scenarioActor) {
    if (!res.isSuccess()) {
        scenarioActor.finish(false);
    }
}
```

For most of the features supported by other User agents, ScenarioActor can use the callback registration method using an annotation attachment or method reference as shown below:

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

Below is a list of supported readers. After declaring the method with the signature below to State, attach the annotation `@Listener`.

| Listener name | Signature |
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
