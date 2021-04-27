## Game > GameAnvil > テスト開発ガイド > シナリオテスト開発ガイド

## 大規模負荷テストと、テスト用シナリオの作成

GameHammerは、大規模負荷テストを行うために大量のコネクションを同時に処理できる機能を提供します。そして、複雑なテストをさらに簡単に管理できるように、状態ベースのシナリオテストをサポートします。 

### シナリオテスト作成例

次のような簡単なシナリオテストを作成します。

![img](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

まず、各シナリオを実行する主体になるScenarioActorを定義します。

```
public static class TestActor extends ScenarioActor<TestActor> {
}
```

状態ごとにStateを継承したクラスを定義します。

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

状態を管理するScenarioMachineを作成します。そして先に定義したStateA、StateB、StateCオブジェクトを作成してScenarioMachineに追加します。 

```
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());
```

シナリオテストを実行します。

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

実行結果はRandomを使用するため毎回変わりますが、おおよそ次のとおりです。

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

## シナリオテスト基本概念

シナリオテストは大規模負荷テストをサポートするための概念で、シナリオテストを実行する時、同時に実行するScenarioActorの数、開始状態、最大実行時間を指定して実行します。 

### ScenarioActor

同時に実行されるそれぞれの主体がScenarioActorです。同時に実行する数を100に指定した場合、100個のScenarioActorオブジェクトが作成され、各シナリオを実行します。ScenarioActorは1つのConnectionを持っており、これを利用してGameAnvilサーバーの機能を実行できます。ScenarioActorを継承実装してシナリオの実行に必要な機能や設定などを追加して利用できます。`changeState()`を利用して現在の状態を他の状態に変更でき、`finish()`を利用してシナリオを終了できます。

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

ScenarioActorを現在の状態を他の状態に変更します。注意する点は、`ChangeState()`が呼び出された時点ですぐに状態が変更されるわけではないという点です。実際に状態が変更されるのは、次のメッセージループの開始時点で、この時に現在の状態を実装したStateオブジェクトの`onExit()`と次の状態を実装したStateオブジェクトの`onEnter()`が順番に呼び出されます。

```
scenarioActor.changeState(NextState.class);
```

#### Finish

ScenarioActorが実行中のシナリオを終了します。`ChangeState()`と同様に`Finish()`が呼び出された後、次のメッセージループの開始時点で実行され、このとき現在の状態を実装したStateオブジェクトの`onExit()`が呼び出されます。そしてScenarioActorが実行された回数がScenarioLoopCountより少ない場合、シナリオを再度実行してScenarioLoopCountの値に達するまで実行を繰り返します。ScenarioLoopCount <= 0の場合、指定されたTestTime(デフォルト値：30秒)の間繰り返します。boolean値を引数として受け取り、シナリオが成功で終了したのか、失敗で終了したのかを区別して統計を記録します。

```
scenarioActor.finish(true);
```

#### Connection

ScenarioActorは1つのConnectionを持っており、これを利用してGameAnvilサーバーの機能を実行できます。

```
Connection connection = scenarioActor.getConnection();
```

シナリオテストでは機能テストのようにFutureを利用してテストコードを作成する場合は、`Future.get()`でブロックになるため、複数のテストを同時に実行できません。代わりにコールバック方式のAPIを使用して同時に実行するようにできます。機能テストで紹介したすべてのAPIには対応するコールバック方式のAPIが提供されるため、シナリオテストではこのコールバック方式APIを使用してください。

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

Stateを継承実装して各状態を表すStateを定義します。

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

ScenarioActorが各状態に変更されると、各状態を表すStateの`onEnert()`が呼び出され、その状態に変更されたScenarioActorが引数として渡されます。各状態から抜ける時は`onExit()`が呼び出され、その状態から抜けたScenarioActorが引数として渡されます。

##### ScenarioMachine

ScenarioMachineで1つのシナリオを定義し、1つのシナリオは複数の状態に定義されます。 

```
ScenarioMachine<STATE, EVENT> scenario = new ScenarioMachine("Sample A");
scenario.setState(new StateA(scenario, STATE.A));
scenario.setState(new StateB(scenario, STATE.B));
scenario.setState(new StateC(scenario, STATE.C));
```

##### ScenarioTest

まずTesterを作成します。このとき、同時に実行するScenarioActorの数、繰り返す回数、テスト時間などをオプションで指定できます。そしてScenarioMachineを引数として受け取りScenarioTestを作成します。作成したTester、ScenarioActoのClass、開始状態を表すStateのClassを渡して実行します。テストはすべてのScenarioActorが指定された回数繰り返すか、指定したテスト時間になるまで実行されます。 

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

テストを完了した後、`printStatistics()`を利用してテスト結果を取得できます。

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
