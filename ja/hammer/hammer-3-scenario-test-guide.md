## Game > GameAnvil > テスト開発ガイド > シナリオテスト開発ガイド

## 大規模負荷テストとそのためのシナリオ作成

GameHammerは大規模負荷テストのために大量の接続を同時に処理できる機能を提供します。そして、複雑なテストをより楽に管理できるように、状態ベースのシナリオテストをサポートします。

### シナリオテスト作成例

ここでは、次のような簡単なシナリオテストの作成例を説明します。

![img](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

まず、各シナリオを実行する主体となるScenarioActorを定義します。

```java
public static class TestActor extends ScenarioActor<TestActor> {
}
```

状態ごとにStateを継承したクラスを定義します。 

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

状態を管理するScenarioMachineを作成します。 そして、先に定義したStateA、StateB、StateCオブジェクトを作成してScenarioMachineに追加します。

```java
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());

sceanrio.editState(StateA.class) // StateAでは
    .addActionOnEnter(changeState(StateB.class), (sceanrioActor) -> new Random().nextBoolean()) // ランダムにStateBに移動
    .addActionOnEnter(changeState(StateC.class)) // 残りの場合はStateCに移動
    .endEdit();

scenario.
    .editState(StateB.class) // StateBでは
    .addActionOnEnter(changeState(StateC.class)) // 常にStateCに移動
    .endEdit();

scenario
    .editState(StateC.class) // StateCでは
    .addActionOnEnter(finishWithSuccess(), (scenarioActor) -> new Random().nextBoolean()) // ランダムに成功終了として処理
    .addActionOnEnter(finishWithFail()) // 残りの場合は失敗終了として処理
    .endEdit();
```

シナリオテストを実行します。

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

実行結果はRandomを使うので毎回違いますが、概ね下記のようになります。

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

シナリオテストは大規模な負荷テストをサポートするための概念で、シナリオテストを実行する時、同時に実行するScenarioActorの数、開始状態、最大実行時間を指定して実行します。

### ScenarioActor

同時に実行される各主体がScenarioActorです。同時実行する数を100に指定した場合、100個のScenarioActorオブジェクトが作成され、それぞれシナリオを実行します。ScenarioActorは一つのConnectionを持ち、これを利用してGameAnvilサーバーの機能を実行できます。ScenarioActorを継承して実装し、シナリオの実行に必要な機能や設定などを追加して利用できます。`changeState()` を使って現在の状態を別の状態に変更することができ、 `finish()` を使ってシナリオを終了できます。

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

ScenarioActorの現在の状態を別の状態に変更します。注意すべき点は、`ChangeState()`が呼び出された時点ですぐに状態が変更されるわけではないということです。実際に状態が変更されるのは次のメッセージループの開始時であり、この時、現在の状態を実装したStateオブジェクトの`onExit()`と次の状態を実装したStateオブジェクトの`onEnter()`が順番に呼び出されます。

```java
scenarioActor.changeState(NextState.class);
```

#### Finish

ScenarioActorが実行中のシナリオを終了します。`ChangeState()`と同様に`Finish()`が呼び出された後、次のメッセージループの開始時に実行され、現在の状態を実装したStateオブジェクトの`onExit()`が呼び出されます。 そして、ScenarioActorが実行された回数がScenarioLoopCountより少ない場合、シナリオを再実行し、ScenarioLoopCountだけ実行されるまで繰り返します。ScenarioLoopCount <= 0の場合、指定されたTestTime(デフォルト:30秒)の間、継続的に繰り返します。 boolean値を引数として受け取り、シナリオが成功で終了したか、失敗で終了したかを区別して統計を記録します。

```java
scenarioActor.finish(true);
```

#### Connection

ScenarioActorは一つのConnectionを持っていて、これを利用してGameAnvilサーバーの機能を実行できます。

```java
Connection connection = scenarioActor.getConnection();
```

シナリオテストでは、機能テストのようにFutureを使ってテストコードを作成する場合、`Future.get()`でブロックになるため、複数のテストを同時に実行できません。 代わりに、コールバック方式のAPIを使って同時に実行できます。機能テストで紹介した全てのAPIには対応するコールバック方式のAPIが提供されるので、シナリオテストではこのコールバック方式のAPIを使うことができます。

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

Stateを継承実装して各状態を表すStateを定義します。

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

ScenarioActorが各状態に変更する度に、各状態を表すStateの`onEnert()`が呼び出され、その状態に変更されたScenarioActorが引数として渡されます。 各状態から抜け出す時は`onExit()`が呼び出され、その状態から抜け出したScenarioActorが引数として渡されます。 

##### ScenarioMachine

ScenarioMachineで一つのシナリオを定義し、一つのシナリオは複数の状態と状態間の遷移で定義されます。

```java
ScenarioMachine<STATE, EVENT> scenario = new ScenarioMachine("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());
```

##### ScenarioTest

まず、Testerを生成します。この時、同時に実行するScenarioActorの数、繰り返す回数、テスト時間などをオプションで指定できます。 そしてScenarioMachineを引数で受けてScenarioTestを作成します。先ほど作成したTester、ScenarioActoのClass、開始状態を表すStateのClassを渡して実行します。テストは全てのScenarioActorが指定された回数だけ繰り返したり、指定したテスト時間になるまで実行されます。

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

テストが完了した後、 `printStatistics()` を使ってテスト結果を取得できます。

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

### 改善されたコールバック登録機能

この方法で登録したリスナーはUserエージェントで登録したリスナーとは違ってStateの終了時に自動で整理されるので、onExitでリスナーを削除する動作をする必要がなくなります。

サンプルコードではrequestに対してコールバックを登録して、コールバックでresponseを処理して文字列を返すように設定しました。返された値はScanarioMachineで次に移動するStateを決定するために使われます。

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

または、サーバーから一方的にメッセージを受け取る場合は、アノテーション方式でコールバックを登録すると便利です。下記のように `@Listener` アノテーションを使って特定のパケットに対してどう処理するかを定義できます。
アノテーションの引数に受信が予想されるメッセージのclassを設定し、そのハンドラにアタッチします。

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

リクエストに対するレスポンスハンドラを登録する時にもアノテーションを使うことができます。この場合、コールバックでサーバーから送信されたパケットが渡されますが、アノテーションのrequest引数にリクエストのタイプを指定する必要があります。
サーバーから受け取るタイプについては、アノテーションに指定しません。

```java
@Listener(request = RequestToServer.class)
@SuppressWarnings("unused")
public void requestToServerListener(PacketResult res, TestActor scenarioActor) {
    if (!res.isSuccess()) {
        scenarioActor.finish(false);
    }
}
```

他のUserエージェントでサポートする機能はほとんどのScenarioActorで下記のようにアノテーションを付けたり、メソッド参照を利用したコールバック登録方式を使うことができます。

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

以下はサポートするリスナーのリストです。下記のシグネチャを持つメソッドをStateに宣言した後、`@Listener` アノテーションを付けてください。

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
