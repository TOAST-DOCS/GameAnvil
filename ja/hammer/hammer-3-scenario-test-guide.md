## Game > GameAnvil > テスト開発ガイド > シナリオテスト開発ガイド

### シナリオテストとは？

シナリオテストとは、予め定められたルール通りにサーバーへ負荷をかけた後、TPSなど性能に関連する指標を得るテストを指します。ここでテストを進めるルールをシナリオと呼びます。また、サーバーに負荷をかけるためには多数のコネクションを生成して維持する必要がありますが、このコネクションそれぞれをシナリオアクターと呼びます。

シナリオは必要に応じて構成して実行できます。シナリオ構成はステートベースで行われます。シナリオには多数のステートが存在します。各ステートはシナリオアクターが実行する動作について定義しています。シナリオアクターは自身のステートに従って行動します。シナリオアクターは一度にこの中のひとつのステートのみを持つことができます。そして特定条件に従って別のステートへ移動できます。このような移動に必要な条件やステート間の連結情報もシナリオで定義します。

ステートがシナリオアクターの実行する動作について定義する方法は、ステートへの初回進入時点にユーザーが定義したコードを挿入することです。通常はサーバーに対して何らかのリクエストを送信するコードを挿入します。そしてリクエストの処理が完了しレスポンスが到着すると、他のステートへ移動するようにする方式が最も多く使われます。

簡単な例を通じてシナリオとステート、シナリオアクターを構成する方法を見てみましょう。3つのステートを持つシナリオを構成してみます。まず、最初のステートAで特定の動作を実行します。そしてこの動作の成否に従い、成功ならステートB、失敗ならステートCへ移動します。図で表すと以下のようになります。

![img](http://static.toastoven.net/prod_gameanvil/images/scenario_1.png)

まず各シナリオを実行する主体となるシナリオアクターを定義します。ScenarioActorを定義するには、まずGameHammerが提供するScenarioActorクラスを継承したクラスを生成します。そして内部にはシナリオアクターが維持すべきデータを定義すればよいです。この例ではシナリオの成否を保存するようにしてみます。

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

次はステートを定義します。まず開始ステートを定義してみます。シナリオアクターを定義する時と同様に、GameHammerが提供するStateを継承したクラスを生成します。この時、タイプパラメータとして先ほど定義したシナリオアクタークラスを入れます。

各ステートではシナリオアクターのステート進入時点と退出時点に呼び出される関数をオーバーライドできるように提供しています。また、このステートがシナリオで最初に進行されるステートである場合にのみ呼び出される関数であるonScenarioTestStartも提供しています。各関数が呼び出される時点が分かるように、全ての関数でログを出力するように設定します。

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

残りのステートB、Cも生成します。

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

これでステート間の移動のために各クラスを修正してみます。

まず、最初のステートであるStateAでは動作の成否に従って、それぞれ異なるステートへ移動しなければなりません。動作の成否は例示のためにランダム値で決定します。他のステートへの移動はchangeState関数を利用します。

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

State Bでは、次の実行時点ですぐにStateCへ移動するように設定します。

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

State Cでは、複数のステートを継続して移動できるようにシナリオを終了処理します。シナリオが終了すると、シナリオアクターは最初のステートへ移動してテストを継続します。シナリオ終了時にシナリオを評価し、正常に進行して終了したかどうかを記録できます。finish()関数を通じてシナリオを終了し、成否を記録します。

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

これで定義した状態を管理するScenarioMachineを生成します。そして先ほど定義したStateA、StateB、StateCオブジェクトを生成してScenarioMachineに追加します。

```java
ScenarioMachine<TestActor> scenario = new ScenarioMachine<>("Sample A");
scenario.addState(new StateA());
scenario.addState(new StateB());
scenario.addState(new StateC());
```

これで作成したシナリオを持ってシナリオテストを実行します。Testerオブジェクトを生成し、テストに使用するユーザー数と、シナリオが終了した際に何回まで繰り返すかを設定します。その後、シナリオテストオブジェクトを生成し、作成したシナリオを引数として提供します。シナリオテストオブジェクトのstart()関数を呼び出す際、引数として先ほど生成したTester、シナリオアクタークラス、最初のステートを引数に入れてください。

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

実行結果は毎回異なりますが、およそ以下のようになります。

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

シナリオ作成のために各ステートで必ずクラスを生成する必要はありません。全てのStateクラスを継承し、以下のようにシナリオ上で各Stateの動作を一度に簡単に定義できます。

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

### シナリオテスト作成例

以下の例は、実際にサーバーへ負荷をかけるように作成された例です。

| サンプルサーバー                                                                           | サンプルテスター                                                                          |
|-----------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| [GameAnvil Scenario Server](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvilScenarioServer.zip) | [GameAnvil Scenario Tester](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvilScenarioTester.zip) |

### アクション

シナリオ中にエンジンへリクエストするなど実行できる動作をアクションと呼びます。アクションはシナリオ中の特定時点に条件と組み合わせて登録できます。

#### ステート移動

```java
changeState("StateA")
```

```java
changeState(StateA.class)
```

#### シナリオ終了

```java
bool isSuccessful = true;
ScenarioAction.finish(isSuccessful);
```

#### 接続進行

```java
ScenarioAction.connect()
```

#### 認証進行

```java
ScenarioAction.authenticate()
```

#### ログイン進行

```java
ScenarioAction.login()
```

#### ユーザーマッチメイキング進行

```java
ScenarioAction.matchUserStart()
```

#### ルームマッチメイキング進行

```java
ScenarioAction.matchRoom()
```

#### ルーム退場進行

```java
ScenarioAction.leaveRoom()
```

#### ログアウト進行

```java
ScenarioAction.logout()
```

#### ネームドルーム動作リクエスト進行

```java
ScenarioAction.namedRoom()
```

#### パーティーマッチメイキング進行

```java
ScenarioAction.matchPartyStart()
```

#### パーティーマッチメイキングキャンセル進行

```java
ScenarioAction.matchPartyCancel()
```

#### チャンネル情報リクエスト進行

```java
ScenarioAction.getChannelInfo()
```

#### 全チャンネル情報リクエスト進行

```java
ScenarioAction.getAllChannelInfo()
```

#### チャンネルユーザー数、ルーム数リクエスト進行

```java
ScenarioAction.getChannelCountInfo()
```

#### 全チャンネルのユーザー数、ルーム数リクエスト進行

```java
ScenarioAction.getAllChannelCountInfo()
```

#### チャンネル移動進行

```java
ScenarioAction.moveChannel()
```

#### スナップショットリクエスト進行

```java
ScenarioAction.snapshot()
```

### アクション登録

以下はアクションを登録するメソッド一覧です。

#### シナリオ進入時点

シナリオ進入時点に実行するアクションを登録できます。

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"));
```

#### シナリオ移動時点

シナリオ移動時点に実行するアクションを登録できます。

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnExit(scenarioActor -> scenarioActor.resetChannle());
```

#### パケット受信時点

パケット受信時点に実行するアクションを登録できます。

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"));
```


#### タイマー呼び出し時点

タイマー呼び出し時点に実行するアクションを登録できます。

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .setActionInTimer("TimerName", changeState("OtherStateNameToEnter"));
```

### 条件機能

条件に従ってアクションを実行するかどうかを設定できます。

#### 常に

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), always());
```

#### 結果成功時

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnReceive(ResultConnect.class, changeState("OtherStateNameToEnter"), ifSuccess());
```

#### 結果失敗時

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnReceive(ResultConnect.class, changeState("OtherStateNameToEnter"), ifSuccess());
```

#### 条件反転

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), NOT(scenarioActor -> scenarioActor.valid()));
```

#### 条件結合

```java
scenario
    .addState("StateNamd")
    .editState("StateName")
    .addActionOnEnter(changeState("OtherStateNameToEnter"), AND(scenarioActor -> scenarioActor.valid(), ifSuccess());
```

### 向上したコールバック登録機能

この方式で登録したリスナーは、Userエージェントを通じて登録したリスナーとは異なり、Stateの終了時点に自動的に整理されるため、onExitでリスナーを削除する動作を行う必要がなくなります。

サンプルコードではrequestに対してコールバックを登録し、コールバックでresponseを処理して文字列を返すように設定しました。返された値はScanarioMachineで次に移動するStateを決定するために使用されます。

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

またはサーバーから一方的にメッセージを受け取る場合には、アノテーション方式でコールバックを登録すると便利です。以下のように`@Listener`アノテーションを使用して、特定パケットに対してどのように処理するか定義できます。
アノテーションの引数に受信が予想されるメッセージのclassを設定し、該当ハンドラに付与します。

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

リクエストに対するレスポンスハンドラを登録する際にもアノテーションを使用できます。この場合にはコールバックとしてサーバーから送信されたパケットが伝達されますが、アノテーションのrequest引数にリクエストタイプを代わりに明示する必要があります。
サーバーから受け取るタイプについてはアノテーションに明示しません。

```java
@Listener(request = RequestToServer.class)
@SuppressWarnings("unused")
public void requestToServerListener(PacketResult res, TestActor scenarioActor) {
    if (!res.isSuccess()) {
        scenarioActor.finish(false);
    }
}
```

その他Userエージェントでサポートする機能は、大部分ScenarioActorで以下のようにアノテーション付与またはメソッド参照を利用したコールバック登録方式を使用できます。

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

以下はサポートするリスナー一覧です。以下のシグネチャを持つメソッドをStateに宣言した後、`@Listener`アノテーションを付与してください。

| リスナー名 | シグネチャ |
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
