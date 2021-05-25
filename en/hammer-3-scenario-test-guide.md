## Game > GameAnvil > Test Development Guide > Scenario Test Development Guide

## Large scale load test and write the scenario for it

GameHammer provides a feature that can be used to simultaneously process a large number of connections for a large scale load test. It also provides scenario tests based on status for more convenient testing. 

### An example of scenario test

Let's make a simple scenario test.

![img](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

First, define ScenarioActor, which is the main agent of each scenario.

```
public static class TestActor extends ScenarioActor<TestActor> {
}
```

Define the class that inherits the State for each status.  

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

Create ScenarioMachine to be used to manage status. And create the StateA, StateB, and StateC objects that are defined earlier and add them to ScenarioMachine. 

```
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());
```

Now, run the scenario test.

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

The run results may vary as they use Random, they are similar to the below:

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

## Basic Concepts of Scenario Test

Scenario test is a concept used to support large load tests. It specifies the number of ScenarioActors to be concurrently run, starting status, maximum run time and run them. 

### ScenarioActor

The concurrently run agents are ScenarioActors. If the number of concurrent runs is set to 100, 100 ScenarioActor objects are created and they run each scenario. Because ScenarioActor has one Connection, it can be used to run the features of the GameAnvil server. While inheriting and implementing ScenarioActor, the features and settings that are needed to run scenarios can be added and used. The current status can be changed using `changeState()` and the scenario can be closed using `finish()`.

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

Change the current status of ScenarioActor to another. Note that the status of `ChangeState()` is not immediately changed when it is called. The time of actual status change is when the next message loop starts. At that time, the `onExit()` of the State object that implements the current status and the `onEnter()` of the State object that implements the next status are called one after another.

```
scenarioActor.changeState(NextState.class);
```

#### Finish

ScenarioActor ends the running scenario. Similar to `ChangeState()`, it is run when the next message loop starts after `Finish()` is called. At that time, the `onExit()` of the State object that implements the current status is called. If the number of ScenarioActor runs is less than the number of ScenarioLoopCount, the scenario is run again until the number matches to the number of ScenarioLoopCounts. If ScenarioLoopCount<=0, it is repeated for the specified TestTime (default: 30 seconds). It receives boolean value to determine whether the scenario is successfully ended or erroneously ended and records the result as statistics.

```
scenarioActor.finish(true);
```

#### Connection

As ScenarioActor has one Connection, it can be used to run the features of the GameAnvil server. 

```
Connection connection = scenarioActor.getConnection();
```

When the user writes test code using Future, like the functionality test, in a scenario test, it is blocked from `Future.get()`. For this reason, multiple tests cannot be run at the same time. Instead, run multiple tests using the API with the callback method. In functionality test, APIs with callback method for all the APIs introduced by functionality test are provided. In a scenario test, use APIs using this method.

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

Inherit and implement State to define the State presenting each status. 

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

Whenever the status of ScenarioActor changes, the `onEnert()` of the State that represents each state and the ScenarioActor changed to the corresponding status is passed. `onExit()` is called when leaving from each status and the left ScenarioActor is passed as a factor.

##### ScenarioMachine

A single scenario is defined by ScenarioMachine and a single scenario can be defined to several states. 

```
ScenarioMachine<STATE, EVENT> scenario = new ScenarioMachine("Sample A");
scenario.setState(new StateA(scenario, STATE.A));
scenario.setState(new StateB(scenario, STATE.B));
scenario.setState(new StateC(scenario, STATE.C));
```

##### ScenarioTest

First, create Tester. At this time, the user can specify the number of ScenarioActors to be run, number of repeats, and test time as an option. And take ScenarioMachine as a factor and create ScenarioTest. Pass the State of the previously created Tester and ScenarioActor and run them. The test is repeated until ScenarioActor is repeated to the specified number or the specified test time is met.

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

Once the test is done, `printStatistics()` can be used to get the test result.

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
