## Game > GameAnvil > サーバー開発ガイド > 始める

## はじめに

このドキュメントは、GameAnvilを利用してサーバーを実装する際に必要な基本要素と実装方法について説明します。このドキュメントと共に提供されるチュートリアルプロジェクト[GameAnvilチュートリアル](../tutorial/tutorial-01-basic.md)を参考にすることを推奨します。

GameAnvilサーバーは基本的にノード(Node)単位で構成します。そのうちユーザーのコードが駆動するノードは、以下の図のように計4種類です。ここでは、これら4種類のノードの実装方法を中心にサーバー開発方法を説明します。

![Nodes on Network.png](https://static.toastoven.net/prod_gameanvil/images/user_nodes_on_network_.png)

## コールバックの再定義

基本的にGameAnvilの大部分の機能はコールバック形式で提供されます。つまり、エンジンユーザーはGameAnvilが提供する基本インターフェース(IGatewayNode、ISupportNode、IGameNode)を実装後、このようなコールバックメソッドを再定義する形式で大部分の機能を使用することになります。この過程でエンジンユーザーが必要なコールバックメソッドのみ実装すればよいため、一部のコールバックメソッドは無視することもできます。これらの基本インターフェースは全て「I」で始まる名前を持ち、com.nhn.gameanvilパッケージまたはその下位パッケージとして提供されます。

![callback-1.png](https://static.toastoven.net/prod_gameanvil/images/callback-1.png)

## 全ての実装の始まり、ノード

例えば、全てのノードは共通して以下のコールバックメソッドを再定義する必要があります。そして、それぞれのノードはその役割に合った追加のコールバックメソッドの実装を要求する場合があります。以下のコードで例に挙げたSampleGatewayNodeは、GatewayNodeの基本インターフェースであるIGatewayNodeを実装しています。

IGatewayNodeを含む全てのインターフェースノードは、共通して以下のようなコールバックメソッドを提供します。onCreate()メソッドのみ、実装するタイプに応じたコンテキストインターフェースをパラメータとして受け取ります。ユーザーがこれらのコールバックメソッドを実装すると、エンジンが特定の時点で該当コールバックを呼び出します。これこそがGameAnvilの最も基本的な使用法です。このような使用法はドキュメント全体を通して大同小異なので、大きな違和感なく各章を理解できるでしょう。

```java
@GameAnvilGatewayNode // エンジンにこのクラスをGatewayとして登録
public class SampleGatewayNode implements IGatewayNode {
    private IGatewayNodeContext gatewayNodeContext;

    /**
     * ゲートウェイノードコンテキストを伝達するために呼び出し
     * <p/>
     * オブジェクトが生成された後、一度呼び出される
     *
     * @param gatewayNodeContextゲートウェイノードコンテキスト
     */
    @Override
    public void onCreate(IGatewayNodeContext gatewayNodeContext) {
        this.gatewayNodeContext = gatewayNodeContext;
    }

    /**
     * ノードが初期化される時に呼び出し
     */
    @Override
    public void onInit() {

    }

    /**
     * Readyになる前に処理する部分のために呼び出し
     */
    @Override
    public void onPrepare() {

    }

    /**
     * Readyになる時に呼び出し
     */
    @Override
    public void onReady() {

    }

    /**
     * Pauseになる時に呼び出し
     *
     * @param payloadコンテンツから伝達したい追加情報
     */
    @Override
    public void onPause(IPayload payload) {

    }

    /**
     * Shutdown命令を受け取ると呼び出し
     */
    @Override
    public void onShuttingdown() {

    }

    /**
     * Resumeになる時に呼び出し
     *
     * @param payloadコンテンツから伝達したい追加情報
     */
    @Override
    public void onResume(IPayload payload) {

    }
}
```

このようなノード共通コールバックの意味と用途は以下の表を参考にしてください。

| コールバック名       | 意味   | 説明                                                                                                                                        |
|----------------|-------|---------------------------------------------------------------------------------------------|
| onCreate       | オブジェクト生成 | オブジェクトが生成された時に呼び出されます。生成されたタイプで使用可能なAPIを使用できるコンテキストを受け取ります。コンテンツで必要であれば保存して使用できます。 |
| onInit         | 初期化  | ノードが最初の初期化を行う時に呼び出します。ノード駆動前に必要な初期化作業がある場合、このコールバックが適しています。この時、ノードはまだメッセージを処理しません。    |
| onPrepare      | 準備   | ノード初期化が完了した後に呼び出されます。ユーザーはノードの準備が完了する前に任意の作業をここで処理できます。この時、ノードはメッセージを処理できます。  |
| onReady        | 準備完了 | ノードが全ての準備を終えた後、駆動完了段階です。この時、ノードはReady状態なのでユーザーはこの時から全ての機能を使用できます。                              |
| onPause        | 一時停止 | ノードを一時停止すると呼び出されます。ユーザーはノードが一時停止される時に追加で処理したいコードをここに実装できます。                                         |
| onShuttingdown | ノード停止 | ノードがShutdown命令を受け取る時に呼び出されます。停止したノードは再開(Resume)できません。                                                                  |
| onResume       | 再開   | ノードが一時停止状態で駆動を再開する時に呼び出されます。ユーザーは再開状態で処理したいコードをここに実装できます。                                  |

## _(underscore)で始まるメソッドと変数

エンジンを使用していると、ユーザーが実装インターフェースで_で始まるメソッドや変数を見かけることがあります。これはエンジン内部でのみ使用することを意味します。つまり、ユーザーは_で始まる変数やメソッドにアクセスしてはいけません。これはJavaのスコープ制御が柔軟でないために一部の公開を許容しているものですので、注意が必要です。

## gameanvilパッケージとgameanvilcoreパッケージ

エンジンは大きく2つの上位パッケージで構成されます。そのうちの1つであるgameanvilパッケージはユーザーのためのものです。このパッケージ内の全てのクラスやAPIは自由に使用可能です。反面、gameanvilcoreパッケージはエンジンコアロジックを含んでいるため、ユーザーが直接アクセスすることを許可しません。それにもかかわらずユーザーに公開されているのは、現在GameAnvilがサポートするJavaバージョンのスコープ制御の限界のためです。ユーザーコードを作成する過程でgameanvilcoreパッケージの内容が含まれないよう、格別の注意が必要です。

## IntelliJテンプレートでプロジェクト構成

GameAnvilプロジェクトを最初から1つずつ構成することは、様々な複雑な過程を要求します。エンジンライブラリを読み込むことに加え、構成ファイルを作成しなければならず、プロトコル仕様を作成するスクリプトとこれをコンパイルするコンパイラも必要です。このような一連の過程により開発時間を不必要に浪費することを防ぐために、GameAnvilはこれらの内容を全て含んだIntelliJ用テンプレートを提供します。以下のリンクからテンプレートファイルをダウンロードした後、次の手順に従ってください。

[GameAnvilテンプレートダウンロード](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil%20Template.zip?disposition=attachment)

IntelliJを実行した後、ショートカットキー`Shift Shift`で全体検索ウィンドウを表示し、**Import Settings...** を検索します。またはFile > Manage IDE Settings > **Import Settings...** を選択します。

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/search_import_settings.png" />

ファインダー画面からダウンロードしたテンプレートファイルを選択します。下の図のようにインポートする要素を選択するウィンドウが表示されたら、ファイルテンプレートとプロジェクトテンプレートを全てチェックした後、OKをクリックします。 

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/select_import.png" />

インポートが完了し新しいプロジェクト作成ウィンドウが開いたら、左側メニューのUser-definedタブからGameAnvil Templateを選択します。

<img src="https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/01-getting-started/new_project_project_template.png" />

完了すると次の図のようにGameAnvilサーバー開発をすぐに開始できるテンプレートプロジェクトが作成されます。

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/project_template_view.png" />

GameAnvilで提供する様々なクラスをすぐに作成できるファイルテンプレートも提供しています。プロジェクトで希望するパッケージを選択した後、右クリックでコンテキストメニューを開きます。この時、次の図のように各クラス別ファイルテンプレートを利用できます。

<img src="https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/01-getting-started/file_template.png" />

次はファイルテンプレートとして提供されるリストです。それぞれのファイルテンプレートは1つのクラステンプレートを実装します。

| ファイルテンプレート                       | 説明                    |
|--------------------------------|-------------------------|
| Connection                     | コネクション基本実装             |
| GameNode                       | ゲームノードの基本実装          |
| GatewayNode                    | ゲートウェイノードの基本実装       |
| MessageHandler for GameNode    | ゲームノードのメッセージハンドラ基本実装  |
| MessageHandler for GatewayNode | ゲートウェイノードのメッセージハンドラ基本実装 |
| MessageHandler for Room        | ルームメッセージハンドラ基本実装        |
| MessageHandler for SupportNode | サポートノードのメッセージハンドラ基本実装 |
| MessageHandler for User        | ユーザーメッセージハンドラ基本実装      |
| RestMessageHandler             | Restメッセージハンドラ基本実装    |
| Room                           | ルーム基本実装                  |
| RoomMatchForm                  | ルームマッチング申請基本実装         |
| RoomMatchInfo                  | ルームマッチング情報基本実装         |
| RoomMatchMaker                 | ルームマッチメーカーの基本実装        |
| Session                        | セッション基本実装              |
| SupportNode                    | サポートノードの基本実装         |
| User                           | ゲームユーザーの基本実装          |
| UserMatchInfo                  | ユーザーマッチ情報の基本実装       |
| UserMatchMaker                 | ユーザーマッチメーカーの基本実装       |

 
