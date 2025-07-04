## Game > GameAnvil > サーバー開発ガイド > 始まり



## 始まりの前に

この文書は、GameAnvilを利用してサーバーを実装する際に必要な基本要素と実装方法について説明します。この文書と共に提供されるリファレンスプロジェクト[GameAnvilリファレンスサーバー](../reference/reference-1-server)を参照してください。また、[JavaDoc文書](https://gameplatform.toast.com/docs/api/)でそれぞれのAPIの説明とUMLダイアグラムを参照してください。

GameAnvilサーバーは、基本的にノード(Node)単位で構成します。そのうち、ユーザーのコードが動作するノードは、以下の図のように計4つです。ここでは、これら4つのノードの実装方法を中心にサーバーの開発方法を説明します。

![Nodes on Network.png](https://static.toastoven.net/prod_gameanvil/images/user_nodes_on_network_.png)



## コールバックの再定義

基本的にGameAnvilの大部分の機能はコールバックの形で提供します。つまり、エンジンユーザーはGameAnvilが提供する基本クラス(Base Classes)を継承した後、このようなコールバックメソッドを再定義する形で大部分の機能を使用することになります。このプロセスで、エンジンユーザーが必要なコールバックメソッドのみを実装すればいいので、一部のコールバックメソッドは無視することもあります。このような基本クラスはすべて"Base"で始まる名前を持ち、com.nhn.gameanvilパッケージもしくは、そのサブパッケージで提供されます。

![callback-1.png](https://static.toastoven.net/prod_gameanvil/images/callback-1.png)

## すべての実装の始まり、ノード

例えば、すべてのノードは共通して以下のコールバックメソッドを再定義する必要があります。そして、それぞれのノードはその役割に合った追加コールバックメソッドの実装を要求できます。以下のコードで例に挙げたSampleGatewayNodeは、GatewayNodeの基本クラスであるBaseGatewayNodeを継承しています。

BaseGatewayNodeを含むすべてのBaseNodeは、共通して以下のようなコールバックメソッドを提供します。ユーザーがこのコールバックメソッドを実装すると、エンジンが特定の時点に該当コールバックを呼び出します。これがGameAnvilの最も基本的な使用方法です。このような使用方法は文書全体においてさほど変わらないため、違和感を抱くことなく、各チャプターを理解できるでしょう。

```java
@GatewayNode
public class SampleGatewayNode extends BaseGatewayNode {

    /**
     * 初期化する際に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onInit() throws SuspendExecution {  
    }

    /**
     * Ready前に処理する部分用に呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPrepare() throws SuspendExecution {
    }

    /**
     * Ready呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onReady() throws SuspendExecution {  
    }

    /**
     * pause呼び出し
     *
     * @param payload contentsから送信したい追加情報
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onPause(Payload payload) throws SuspendExecution {  
    }

    /**
     * resume呼び出し
     *
     * @param payload contentsから送信したい追加情報
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onResume(Payload payload) throws SuspendExecution {  
    }

    /**
     * shutdownコマンドを受け取ると呼び出し
     *
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public void onShutdown() throws SuspendExecution {  
    }
}
```

これらのノード共通コールバックの意味と用途については次の表を参照してください。


| コールバック名 | 意味              | 説明                                                                                                                                        |
| ------------ | -------------------- |---------------------------------------------------------------------------------------------------------------------------------------------|
| onInit     | 初期化           | ノードが最初の初期化を行う時に呼び出します。ノードが動作する前に必要な初期化作業がある場合は、このコールバックが適しています。この時、ノードはまだメッセージを処理しません。                                                   |
| onPrepare  | 準備             | ノードの初期化が完了した後に呼び出されます。ユーザーはノードが準備完了する前に任意の作業をここで処理できます。この時、ノードはメッセージを処理できます。                                                  |
| onReady    | 準備完了        | ノードがすべての準備を終えた動作完了段階です。この時、ノードはReady状態であるため、ユーザーはこの時からすべての機能を使用できます。                                                              |
| onPause    | 一時停止        | ノードを一時停止すると呼び出されます。ユーザーはノードが一時停止された時に追加で処理したいコードをここに実装できます。                                                                        |
| onResume   | 再開             | ノードが一時停止状態から動作を再開する際に呼び出されます。ユーザーは再開状態で処理したいコードをここに実装できます。                                                                 |
| onShutdown | ノード停止        | ノードが完全に停止された際に呼び出されます。停止されたノードは再開(Resume)できません。                                                                                            |
| getMessageDispatcher | 処理するパケット有り | ノードに処理するメッセージがある場合に返します。ユーザーは、自分が宣言したディスパッチャーを使用できます。詳細については[メッセージ処理](./server-impl-07-message-handling#13-getMessageDispatcher)を参照してください。 |



## throws SuspendExecution

前のサンプルコードですべてのコールバックメソッドの次のような例外シグネチャを見てきました。

```java
throws SuspendExecution
```

これは実際の例外ではありません。ファイバーのフローを制御するために[Quasar](https://docs.paralleluniverse.co/quasar/)で使用する一種の迂回法です。この例外シグネチャは該当メソッドがいつでもファイバーを一時停止(Suspend)できることを意味します。

そのため、絶対にこの例外を次のような方法で直接処理してはいけません。詳細については[Suspendable](../server-basic/server-basic-03-suspendable)を参照してください。

```java
try {
    ...
} catch (SuspendExecution e) 
{
    // 絶対にSuspendExecution例外はcatchしてはいけません。
}
```



## _(underscore)で始まるメソッドと変数

エンジンを使用すると、ユーザーが継承したBaseクラスにおいて _で始まるメソッドや変数を見ることがあります。これはエンジン内部でのみ使用するためのものを意味します。すなわち、ユーザーは _で始まる変数やメソッドにアクセスしてはいけません。これはC++と異なり、Javaのスコープ制御が柔軟ではなく、一部表示を許可した場合であるため注意が必要です。



## gameanvilパッケージとgameanvilcoreパッケージ

エンジンは大きく2つの上位パッケージで構成されます。そのうちの1つであるgameanvilパッケージはユーザー用のものです。このパッケージ内のすべてのクラスやAPIは自由に使用可能です。

一方でgameanvilcoreパッケージは、エンジンコアロジックを含んでいるため、ユーザーのアクセスを許可していません。それにもかかわらず、ユーザーに表示されるのは、現在GameAnvilがサポートするJavaバージョンのスコープ制御限界が原因です。ユーザーコードを作成するプロセスでgameanvilcoreパッケージの内容が含まれないように注意が必要です。



## IntelliJテンプレートでプロジェクトを構成

GameAnvilプロジェクトを最初から1つずつ構成するには、いくつかの複雑なプロセスが必要です。エンジンライブラリを読み込むとともに構成ファイルを作成する必要があり、プロトコルの詳細を作成するスクリプトとこれをコンパイルするコンパイラも必要です。このような一連のプロセスによる開発時間の浪費を防止するために、GameAnvilはこれらの内容をすべて含むIntelliJ用テンプレートを提供します。以下のリンクでテンプレートファイルをダウンロードして、次の手順に従ってください。



[GameAnvilテンプレートダウンロード](https://static.toastoven.net/prod_gameanvil/files/GameAnvil Template.zip?disposition=attachment)



IntelliJを実行後、ショートカットキー`Shift Shift`で全体検索ウィンドウを表示して**Import Settings...**を検索します。または、File > Manage IDE Settings > **Import Settings...**を選択します。

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/search_import_settings.png" />

ファインダー画面でダウンロードしたテンプレートファイルを選択します。次の図のようにインポートする要素を選択するウィンドウが表示されたら、ファイルテンプレートとプロジェクトテンプレートをすべてチェックした後、OKをクリックします。インポートが完了したら

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/select_import.png" />

新しいプロジェクト作成ウィンドウが開いたら、左メニューのUser-definedタブでGameAnvil Templateを選択します。

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/new_project_project_template.png" />

終了すると、次の図のようにGameAnvilサーバーの開発をすぐに開始できるテンプレートプロジェクトが作成されます。

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/project_template_view.png" />

GameAnvilが提供するさまざまなクラスをすぐに作成できるファイルテンプレートも提供しています。プロジェクトで希望するパッケージを選択した後、クリックしてコンテキストメニューを開きます。この時、次の図のように各クラス別のファイルテンプレートを利用できます。

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/file_template.png" />

次の図はファイルテンプレートで提供されるリストです。それぞれのファイルテンプレートは1つのクラステンプレートを実装します。

| ファイルテンプレート  | 説明             |
| -------------- |------------------|
| GatewayNode    | ゲートウェイノードの基本実装 |
| GameNode       | ゲームノードの基本実装   |
| SupportNode    | サポートノードの基本実装  |
| Connection     | コネクションクラスの基本実装 |
| Session        | セッションクラスの基本実装  |
| GameUser       | ゲームユーザーの基本実装   |
| GameRoom       | ゲームルームの基本実装    |
| UserMatchMaker | ユーザーマッチメイカーの基本実装 |
| UserMatchInfo  | ユーザーマッチ情報の基本実装 |
| RoomMatchMaker | ルームマッチメイカーの基本実装 |
| RoomMatchForm  | ルームマッチング申請の基本実装  |
| RoomMatchInfo  | ルームマッチング情報基本の実装  |
