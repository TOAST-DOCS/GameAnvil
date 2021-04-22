## Game > GameAnvil > サーバー開発ガイド > 基本概念

## 1. Node

GameAnvilサーバー構成の最も基本となる単位はノードです。各ノードはその役割に合った機能を独立して実行します。ノードごとにどのような役割を実行するかは設定できます。ノードを詳しく説明する前に、役割ごとにノードを分けると次のとおりです。

| ノード   | 必須 | 機能                                    | コンテンツ実装 | ネットワークアクセス   |
| ---------- | ---- | ------------------------------------------- | ----------- | ------------------- |
| Gateway    | 必須 | クライアント接続と認証を処理           | 可能    | public              |
| Game       | 必須 | 実際のゲームサーバーとしてコンテンツを処理        | 可能    | private             |
| Support    | 任意 | 必要に応じて独立したサービスで実装するようにサポート | 可能    | privateまたはpublic |
| Match      | 任意 | マッチメイキングを実行                         | 可能    | private             |
| Location   | 必須 | ユーザーとルームなどの位置情報を保存および管理 | 不可  | private             |
| Management | 必須 | サーバー情報の収集およびConsole/Agentと通信  | 不可  | private             |
| Ipc        | 必須 | GameAnvilサーバーのInter-process通信処理 | 不可  | private             |

GameAnvilのユーザーは、この中からコンテンツの実装が可能なノード(Gateway、Game、Support、Match)にのみ集中できます。残りのノードはGameAnvilサーバーエンジンが内部的に使用し、ユーザーが追加で実装する部分はありません。また必須ノードは、サーバー群に1つ以上存在するときのみ起動できます。

これらのノードの階層構造は、次の図のようになっています。

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



## 2. シングルスレッド(Single-Threaded)

GameAnvilで1つのノードは1つのスレッドで処理されます。これはとても重要です。各ノードは基本的にすべての処理を非同期で行わなければならず、このノードスレッドはブロッキングなしで継続的に動作することが保障されなければなりません。このような動作モデルはVert.xやNode.jsととても似ています。

シングルスレッドの最大の利点は、やはりlock-freeです。ユーザーは、特殊な場合を除き、明示的にlockを使用する必要がなく、使用してはなりません。先に説明したようにlockを使う瞬間、そのノードのスレッドが処理を停止することがあります。これは、そのノード全体のロジックが停止することを意味するので、GameAnvilで開発する場合は、絶対にノードのスレッドにlockを使用しないでください。したがって、ノードのスレッドと外部スレッドの間には絶対にゲーム関連オブジェクトが共有されてはいけません。ただし、別途外部スレッドを作成して作業を委任する場合に、外部のスレッド間のlockは使用してもかまいません。もし外部スレッドで作業を委任したり、委任した作業の結果を取得する必要があるときはGameAnvilで提供される[非同期サポートAPI]（https://alpha-docs.toast.com/ko/Game/GameAnvil/ko / server-3-implementation /＃12-api）を使用してください。



## 3. ファイバー(Fiber)

ファイバーは一種の軽量ユーザースレッド(Lightweight User Thread)で、GameAnvilサーバーコードの基本的な流れの単位です。前述したノードのシングルスレッドは、多数のセッション、ユーザーとルームなどを同時に効果的に処理するために再び多数のファイバーにコードの流れが分かれます。つまり、GameAnvilはファイバーベースのContinuationをサポートします。

多数のファイバーはスレッドプール(Executor)上でスケジューリングされます。この時、スレッドプールのサイズを1に固定するとすぐにGameAnvilノードのモデルになります。すなわち、ノードは多数のファイバーを同時に処理するためのアノテーション(annotation)スケジュールラです。これを図で表すと次のようになります。

![image.png](http://static.toastoven.net/prod_gameanvil/images/FiberConcept.png)

このように、ファイバーを利用する時の利点は、順番にコードの作成が可能である点です。サーバーのコードは一般的なブロッキングコードを作成するのと非常に似ています。別のコールバック処理や完了通知に気を使う必要がありません。このようなファイバーの利点に加え、GameAnvilユーザーはこのファイバーの単位についてあまり気にする必要がありません。GameAnvilエンジンですべてのファイバーを管理しているため、ユーザーは一般的なシングルスレッドのコードを作成するように開発できます。

![image.png](http://static.toastoven.net/prod_gameanvil/images/FiberConcept2.png)

GameAnvilサーバーコードは、非同期処理をベースにします。そのため、[非同期サポートAPI](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api)を提供します。これらの非同期APIを使用して任意のファイバー上でブロッキングを呼び出す場合にはそのファイバーのみsuspend(待機状態)になります。この内容は文書の下で詳しく説明します。



### **[注意事項]**

次のいくつかの内容は、ファイバーベースのコードを作成する時に注意する事項です。ユーザーはファイバーの単位を気にする必要がありませんが、以下の注意事項は必ず守ってください。

- **Suspend**はファイバーがコードの実行を他のファイバーに譲り、待機状態に入ることを意味します。これをファイバーブロッキングで表現することもできます。
- Suspendする場合があるメソッドは**throws SuspendExecution**という例外シグネチャを追加する必要があります。これは、そのメソッドがファイバーを中断させることもあるブロッキングコードが含まれていることをエンジンに知らせる約束です。

```
void someSuspendableMethod() throws SuspendExecution {

// some fiber-blocking call can be here

}
```

- もし特殊な理由でthrows SuspendExecution例外シグネチャを明示できない場合は、**@Suspendable**アノテーション(annotation)を使用できます。それ以外の場合には必ずthrows SuspendExecution例外シグネチャを優先して使用してください。

```
@Suspendable
void someSuspendableMethod() {

// some blocking call can be here

}
```

- 当然ですが、これらのSuspendableメソッドを呼び出すメソッドもSuspendableします。例えば前述のsomeSuspendableMethod()を呼び出すsomeCaller()というメソッドがあるとすると、someCaller()もSuspendableに宣言する必要があります。

```
void someCaller() throws SuspendExecution {

    someSuspendableMethod();

}
```

- SuspendExecutionは、エンジンで使用するQuasarライブラリがファイバーを処理するために使用する特殊な記法です。これは**実際の例外ではなく、**ファイバーがまだJavaの標準ではないため、Quasarライブラリが使う一種の迂回手法だと理解できます。**そのため、絶対にこのSuspendExecution例外をcatchしてはいけません。**頻繁に発生する可能性のあるエラーであり、非常に重要な部分です。

```
// !!誤った使用方法!!

void someCaller() {

    try {

        someSuspendableMethod();

    } catch (SuspendExecution e) {
        // 絶対にSuspendExecutionを明示的にcatchしてはいけません。
    }
}
```

- しかし全ての例外をcatchするのは問題ありません。この部分については自由に処理してください。

```
void someCaller() throws SuspendExeuction {

    try {

        someSuspendableMethod();

    } catch (Exception e) {
        // 問題なし
    }
}
```



## 4. ファイバーベースの非同期処理(Asynchronous on fibers)

コードは、ファイバー上で非同期で処理されることがあります。つまり、1つのファイバーで任意の時間がかかるI/O呼び出しを行った場合に、ファイバーは呼び出しが完了するまで実行権限を他のファイバーに譲ることができます。さらに、スレッドブロッキング呼び出しさえ、これをファイバーブロッキングの呼び出しに変換して非同期化できるようにAPIを提供します。つまり、スレッドブロッキングの呼び出しは、必ず[非同期サポートAPI](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api)を使用して処理する必要があり、もしこれを破って直接呼び出すと、そのスレッドがブロックされ、その結果、スレッド上のすべてのファイバーがブロッキングされてノード全体が停止する結果をもたらすことになります。

```
    void someProblematicMethod() {

        someThreadBlockingCall(); // そのファイバーだけでなく、全てのスレッドがブロッキングされる。

    }
```

これについての詳細な説明は、[実装方法で非同期サポート](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api)で扱います。



## 5. 分散サーバー

基本概念でGameAnvilのノード構成は次の図のとおりだと説明しました。すなわち、1つのプロセスは複数のノードを自由に構成して起動できます。ただし、すべてのGameAnvilプロセスは1つのIPC(Inter-Process Communication)ノードが必ず含まれている必要があります。このIPCノードはGameAnvilプロセス間の通信を担当します。実は実際のネットワーク処理を担当する低レベルノードがありますが、ユーザーはこの部分をひっくるめてIPCノードと理解して構いません。

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)

### 5-1. ノード間通信

これらのIPCノードを介して2つ以上のGameAnvilプロセスが通信する様子は、下の図のとおりです。この図で2つのGameAnvilプロセスは異なる構成のノードが動作します。この時、それぞれのノードは相互に通信が可能です。

異なるプロセスのノードと通信するために、各ノードはIPCノードを介してメッセージを送信します。一方、同じプロセス上のノードはキューを利用して相互に通信します。そのため、この場合はIPCノードを介しません。

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/IPC.png)

### 5-2. Meetpoint

では、このようなIPCのためのプロセスは、どのように相互接続するのでしょうか？その答えはMeetpointです。GameAnvilはMeetpointアドレスを設定できます。以下のような形式ですが、1つ以上のIPアドレスのペアを設定できます。GameAnvilプロセスは最初の起動時に設定されたMeetpointアドレスのいずれかに接続を試みて全サーバー群の情報を同期します。

```
"common": {

    "meetEndPoints": [
      "10.1.2.1:16000",
      "10.1.2.2:16000",
    ],

}
```



## 6. コネクション(Connection)とセッション(Session)

クライアントはGatewayノードにコネクション(Connection)を作成します。このコネクションを使用してアカウントとユーザー情報をもとに認証とログインを行うことができます。ログインまで完了すると、任意のGameノードにユーザーオブジェクトが作成されます。これはGatewayノードとそのGameノードの間には論理的なセッションが作成されたことを意味します。このようにコネクションとセッションの作成が完了すると、ユーザーはゲーム進行が可能になります。

### 6-1. Connection Recovery

クライアントとGatewayノード間のコネクションが切断されると、下図のようにコネクションの復旧(Connection Recovery)が行われます。再接続を行う過程でクライアントは複数のGatewayノードの中から、前と違う場所にコネクションを試みる場合もあります。この場合、ユーザーオブジェクトが存在するGameノードの位置情報をもとに新たにセッションを復旧します。そのためユーザーはゲーム進行中に再接続を行っても以前のゲーム状態を継続することができます。

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

### 6-2. Location Node

コネクション復旧(Connection Recovery)の図にLocationノードがあります。このLocationノードはGameAnvilが内部的にユーザーとルームなどの位置情報を管理する用途で使用します。ユーザーはLocationノードに直接実装したり、使用することはできません。しかし、位置情報を管理するLocationノードの存在を知ってこそ全体的なGameAnvilシステムの流れを理解することができるため、ここで簡単に説明します。

上記のコネクション復旧(Connection Recovery)を例に説明します。クライアントが最初に接続を行い、Gameノードにログインを試みる過程で、関連するセッションとユーザーの位置情報は全てLocationノードに保存されます。そのため、再接続を行う場合は、前の接続過程で保存しておいたこの位置情報を照会できます。これらの位置情報はGameAnvilの内部でユーザーやルームの位置情報を照会し、これをもとにメッセージを送信する用途などに使用される非常に重要なものです。



## 7. コアライブラリ

下の4つがGameAnvilで使用するコアライブラリです。QusarとZeroMQそしてNettyは、エンジン内部で使用するためGameAnvilユーザーが直接使用することはありません。Protocol Buffersはメッセージをシリアル化/逆シリアル化する過程で使用します。直接使用するかどうかに関係なく、下の4つのライブラリをよく理解しているとエンジンを使用する時に役立つでしょう。

| ライブラリ   | 用途                          |
| ---------------- | ------------------------------ |
| Quasar           | ファイバーベースのContinuationをサポート |
| ZeroMQ           | サーバーのIPC                     |
| Netty            | サーバー-クライアント通信       |
| Protocol Buffers | サーバー-クライアントメッセージのシリアル化 |



## 8. ByteCode Instrumentation

GameAnvilは、ファイバーベースのサーバーエンジンです。そのためQuasarライブラリを使用します。ファイバーベースの非同期処理を行うために事前に約束された特殊な例外を使用します。あるいは@Suspendableアノテーション(annotation)を使用することもできます。

```
throws SuspendExecution
```

事前に約束されたこれらのコードを解析するためにGameAnvilサーバーコードはQuasarライブラリを利用してByteCode Instrumentationを行う必要があります。ByteCode Instrumentationは2つの方法のうち1つを利用して行うことができます。

### 8-1. Runtime Instrumentation

サーバー実行VMオプションの最も前の部分に以下のようにQuasarバイナリをjavaagentに追加します。このようにしてランタイムにByteCode Instrumentaionを行います。

```
-javaagent:MY_PATH\quasar-core-0.7.10-jdk8.jar=bm
```

### Note

この項目はVMオプションの最も前の部分に追加する必要があります。この時、quasar-coreのパスは本人のquasar-coreをコピーしておいたパスに設定してください。

### 8-2. AOT Instrumentation

AOT(ahead-of-time) Instrumentationを進めたい場合は、以下の内容をプロジェクトオブジェクト管理ファイル(pom.xml)に追加した後、Mavenを利用してサーバーバイナリをpackage、install、deployのいずれかを行うとコンパイル完了後にInstrumentationを進めます。この場合は当然、最初の場合のようにVMオプションでjavaagentが必要ありません。

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>

    <executions>
        <execution>
            <id>instrument-classes</id>
            <phase>compile</phase>

            <configuration>
                <tasks>
                    <taskdef name="instrumentationTask" classname="co.paralleluniverse.fibers.instrument.InstrumentationTask" classpathref="maven.dependency.classpath"/>
                    <instrumentationTask>
                        <fileset dir="${project.build.directory}/classes/" includes="**/*.class"/>
                    </instrumentationTask>
                </tasks>
            </configuration>

            <goals>
                <goal>run</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
