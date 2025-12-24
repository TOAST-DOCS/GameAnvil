## Game > GameAnvil > 詳細なチュートリアル

### GameAnvilでゲームサーバーを簡単に構築する方法

GameAnvilはリアルタイムマルチプレイヤーゲームサーバー構築プラットフォームです。GameAnvilを使用すると、簡単にゲームサーバーとクライアントを開発して運営できます。GameAnvilを使うためには、ゲームサーバーとクライアントの開発から始める必要があります。この文書では、GameAnvilの基本的な機能を利用してマルチプレイヤーゲームサーバーを開発する過程を説明します。この文書を通じてGameAnvilの基本的な概念、開発プロジェクトの構成方法及び実装方法などを簡単に習得できます。

GameAnvilはサーバーエンジンだけでなく、サーバーにクライアントを接続するためのコネクトを提供します。この文書は、サーバー開発及びコネクタを利用したクライアント開発についても説明します。サーバーとクライアントが相互作用する様子を見ることができるサンプルを完成させながら、GameAnvilを使ってゲームを開発する全体的な流れに慣れることができます。

この文書は、サーバーの概念とAPIを単純に列挙するのではなく、より具体的な例で説明し、理解を助けるために実際のプレイが可能なマルチプレイヤージグソーパズルゲームを開発する過程を順番に説明します。ドキュメントの内容を一つ一つ直接見ながら、自然にGameAnvilとマルチプレイヤーゲームの開発に対する理解度を高めることができます。

## プロジェクトの構成

マルチプレイヤーゲームを作るためには、クライアントと対応するサーバープログラムが必要です。この例では、クライアントプログラムの作成にはUnityとGameAnvilコネクタを、サーバープログラムの作成には先に紹介したサーバーエンジンGameAnvilを使用します。まず、GameAnvilを利用したサーバープログラムプロジェクトを作成した後、UnityとGameAnvilコネクタを利用してクライアントプログラムプロジェクトを作成します。

### GameAnvilプロジェクトの構成

プロジェクトにGameAnvilを適用するためにはMavenリポジトリからGameAnvilライブラリをダウンロードしてGameAnvilを駆動するため必要な設定ファイルを作成する必要があります。最後に少しのボイラープレートコードを書けば、開発初期設定が終わります。この章では、開発を開始するための初期設定を完了することを目的としています。実際のプロセスを実行してサーバーを駆動することは、次の章で説明します。

GameAnvilではこのような一連の過程を代行してくれるIntelliJテンプレートを提供し、より簡単に初期作業を完了できます。次のリンクからIntelliJ用プロジェクトファイルのテンプレートをダウンロードできます。ダウンロードしたテンプレートは解凍しないようにしてください。

[プロジェクトテンプレートダウンロード](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil%20Template.zip?disposition=attachment)

ダウンロードしたテンプレートを適用するためにIntelliJを実行します。**Welcome to IntelliJ IDEA]画面の左側のメニューから**Customize**を選択し、**Import Settings...** をクリックします。または検索窓で**Import Settings...**を検索します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/1_import_gameanvil_template.png)

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/2_search_import_settings.png)

ファインダーまたはファイルエクスプローラーでテンプレートをダウンロードしたパスに移動して圧縮ファイルを選択します。**Select Components to Import**ウィンドウが開いたら**File templates**項目と**Project Templates**項目を全てチェックして選択します。**OK**をクリックしてインポートが完了したら、IntelliJを再起動してテンプレートの適用を完了します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/3_select_import.png)

テンプレートを利用してプロジェクトを構成することもできますが、ここではすでに初期設定が完了しているプロジェクトをダウンロードして使います。下記のリンクをクリックしてチュートリアル用プロジェクトをダウンロードし、解凍してIntelliJで実行します。

[チュートリアル用プロジェクトのダウンロード](https://static.toastoven.net/prod_gameanvil/files/GameAnvil%20Tutorial%20Project_1213.zip?disposition=attachment)

初めてプロジェクトを開くと、下記のようにGradleでスクリプトを実行することを許可するかどうか尋ねるプロンプトが表示されることがあります。**Trust Project**を選択して完全なプロジェクトを開きます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/4_imported_gameanvil_template.png)

これでIntelliJにサーバープロジェクトの骨格が構成されました。Projectパネルを見ると、コードと設定ファイルが作成されたことが確認できます。
- protocolパッケージ: javaでコンパイルされたプロトコルバッファファイルを含むパッケージです。
- Main:プログラムの入り口であるMain関数を含むクラスです。
- BasicProtocol.proto:プロトコルバッファを利用して作成されたプロトコルファイルです。
- GameAnvilConfig.json: GameAnvilの駆動に必要なサーバー設定情報を記録したファイルです。サーバーの実装に合わせて修正できます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/gameanvil_projectview_init.png)

まず、JDKの設定を確認します。左上メニューの** File > Project Structure** を選択します。Macユーザーの場合、`Command + ;`ショートカットキーを使用できます。

ProjectタブでSDKの設定を確認します。もし、設定されたSDKがない場合は、`Add SDK > Download JDK`を通じて希望するバージョンのJDKをダウンロードして設定します。Language levelはSDK defaultに設定します。次に、ModulesタブでLanguage levelをProject defaultに設定します。
**Project**タブでSDKの設定を確認します。もし、設定されたSDKがない場合は**Add SDK > Download JDK**をクリックして希望のバージョンのJDKをダウンロードして設定します。**Language level**は**SDK default**に設定します。次に**Modules**タブで**Language level**を**Project default**に設定します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/project_structure_1213.png)

**設定メニューでgradleで使っているJVMを確認します。プロジェクトSDKと同じgradleバージョンに設定します。**

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gradle_sdk_config_1213.png)

プロジェクトの準備がほぼ終わりましたが、実行するためにはいくつかの設定が必要です。ここではまず、クライアントプロジェクトを先に作成した後、サーバーの設定を終えて実行します。
<br>

### Unityプロジェクト構成

Unity Hubを実行します。右上の**NEW**をクリックして新しいプロジェクト作成ウィンドウを開きます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unityhub.png)

テンプレートリストから**2D**を選択し、プロジェクト名と位置を確認した後、**CREATE**をクリックしてプロジェクトの作成を完了します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unityhub_new_project.png)

次のリンクからGameAnvilコネクタをダウンロードしてください。コネクタはGameAnvilサーバーとの通信に必要なクライアントAPIを提供し、簡単なコードだけでクライアントを実装できるようにするパッケージです。

[[ gameanvil-connector.unitypackage ]](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector.unitypackage)

サーバープロジェクトの作成と同じように、早い進行のため作成されたボイラープレートコードをダウンロードして使います。次のリンクからチュートリアル用コードとイメージソースなどが含まれているパッケージをダウンロードします。

[[ GameAnvilTutorial.unitypacakge ]](https://static.toastoven.net/prod_gameanvil/files/GameAnvil Tutorial.unitypackage)

ダウンロードしたパッケージファイルをUnityプロジェクトにドラッグしてインポートします。または、**Asset > Import Package > Custom Package...**メニューを開き、ファインダーまたはファイルエクスプローラーでパッケージファイルを選択します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_import_custom_package.png)

Import Unity Packageダイアログボックスで、リストのすべてのチェックボックスを選択し、**Import**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_import_custom_package_window.png)

最後に、快適にテストするためには、Project SettingsウィンドウでPlayerタブを選択し、Resoultion and Presentation項目で下記のようにプロジェクトの基本ウィンドウの幅と高さをそれぞれ1920、1080に設定します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_project_setting_resolution.png)

クライアントプロジェクトの設定が完了しました。

## サーバーの駆動及び接続

再びIntelliJに移動してサーバープロジェクトを実行します。

### GameAnvilサーバーの駆動

実行設定が完了したら、Mainクラスのmain()関数の左側の緑色の三角形アイコンをクリックして**Main.main()実行**を選択します。最初の実行後は、IntelliJの右上の緑色の三角形のRunアイコンをクリックしてもサーバーが実行されます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run1_1213.png)

build.gradleには便宜上、JVMオプションがあらかじめ設定されています。この設定を活用してサーバーを実行するには、IntelliJのGradleウィンドウで**Task > others > runMain**を右クリックし、**GameAnvilTutorial実行**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run2_1213.png)


サーバーが正常に駆動されると、サーバー駆動状態に関するログが多数出力されます。GameAnvilサーバーは、実行する役割を複数に分担するイベントループであるノードで構成されています。各ノードは、コードを実行するために準備する時間が必要です。各ノードの準備が完了すると、onReadyログを出力します。ノードのうち、クライアントがサーバーに接続するための直接的な役割を実行するGatewayNodeが準備され、そのノードでonReadyログが出力されると、GameAnvilサーバーはいつでも接続可能な状態になります。

<br>

### コネクトハンドラの作成

それでは、Unityプロジェクトに移動してGameAnvilサーバーに接続できるようにコードを作成してみましょう。サーバーと接続するには、まず、コネクタオブジェクトを作成する必要があります。

アセットパネルでSceneフォルダ内のConnect.sceneをダブルクリックしてシーンを準備します。階層関係パネルでConnectHandlerゲームオブジェクトを選択して適用されたコンポーネントのスクリプトConnectHandler.csファイルの実装を追加修正してみましょう。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using GameAnvil.User;
using UnityEngine.SceneManagement;
using Protocol;

public class ConnectHandler : MonoBehaviour
{
    [SerializeField]
    GameAnvil.Connector.Config config;

    private GameAnvil.Connector connector;
    private static ConnectHandler instance;

    [Header("Object Reference")]
    public Text serverInfoText;
    public Text clientInfoText;
    public Text logText;

    public GameObject popupCanvas;
    public InputField roomIdInput;

    public GameAnvil.Connector GetConnector()
    {
        return connector;
    }

    public static ConnectHandler GetInstance()
    {
        return instance;
    }

    private void Awake()
    {
        if (instance != null)
        {
            DestroyImmediate(this.gameObject);
        }
        instance = this;
        DontDestroyOnLoad(this.gameObject);
        connector = new GameAnvil.Connector(config);
    }

    private void OnDisable()
    {
        if (connector.IsConnected())
        {
            connector.CloseSocket();
        }
    }

    private void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            // 入力した時間の間、サーバーのclientStateCheck機能を停止させる。
            // この時間が過ぎるとclientStateCheck機能が動作して接続が切れることがあります。
            connector.GetConnectionAgent().PauseClientStateCheck(10 * 60);

            connector.Update();
        }
        else
        {
            connector.Update();

            // サーバーのclientStateCheck機能を再開します。
            connector.GetConnectionAgent().ResumeClientStateCheck();
        }
    }
}
```

Awake()関数で新しいConnectorオブジェクトを作成してconnectorフィールドに保存します。サーバーと送受信するメッセージを処理するため、コネクタのUpdate()メソッドを周期的に呼び出す必要があることに注意してください。フレームごとにUnityで呼び出す関数であるUpdate()関数でconnectorのUpdate()を呼び出すように作成します。

GameAnvilサーバーとクライアントは定期的にステータスチェックパケットをやり取りする必要があります。クライアントが一時停止された状況などでステータスチェックパケットの送受信を停止するには、PauseClientStateCheck()メソッドを利用します。ステータスチェックを再開するには、ResumeClientStateCheck()メソッドを呼び出します。OnApplicationPause()関数でクライアントの一時停止を検出し、適切なメソッドを呼び出します。
<br>

### サーバー接続コードの追加

コネクタで様々な機能を利用するために使う窓口をAgentと言います。Agentは実行する機能によってConnectionAgentとUserAgentに分けられます。サーバー接続及び認証機能はConnectionAgentを使用してアクセス可能です。Unityが始まる時点で接続が行われるようにConnectionHandlerコードに下記のようにStart()メソッドとConnect()メソッドを追加します。

```c#
// Client-side

[SerializeField]
GameAnvil.Connector.Config config;

public GameAnvil.Connector connector = null;
private ConnectionAgent connectionAgent;

public string ip = "127.0.0.1";
public int port = 11200;


[Header("Object Reference")]
public Text serverInfoText;
public Text clientInfoText;
public Text logText;

void Start () {
    // 起動時にすぐに接続を試みます。
    Connect();
  
    // 接続情報を画面に出力します。
    serverInfoText.text = "サーバー情報:" + ip + ":" + port;
}

public void Connect(){
  connectionAgent = connector.GetConnectionAgent();
	connectionAgent.Connect(ip, port,
            (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
                Debug.Log(result);
            
              	// ゲーム画面上に結果を表示します。
                if (result == ResultCodeConnect.CONNECT_FAIL) {
                    logText.text = "接続情報:接続失敗";
                } else {
                    logText.text = "接続情報:接続成功";
                }
            }
        );
}
```

Connect()関数ではconnectorを使ってConnectionAgentオブジェクトを参照してConnectメソッドを呼び出しています。これにより、引数で指定されたipとport情報でサーバーへの接続を試みます。3つ目の引数には接続を試行した結果を受け取るコールバックを登録します。

コールバックは接続を実行したConnectionAgentオブジェクトと接続を試行した結果コードをパラメータで渡します。接続を試行した結果コードをコンソールに出力して、画面上でも分かるように結果によって違うフレーズを表示するように設定します。例題コードでは接続に失敗した場合、「接続情報：接続失敗」を画面に表示するようにしました。

これでサーバーが接続を受け入れる準備ができたように、クライアントもサーバーに接続する準備が完了しました。

<br>

### サーバー接続の確認

Unityクライアントでプレイモードに入り、コンソール上に結果コードが正しく出力されるか確認します。ゲーム画面のテキスト上にIP、Portの接続情報と一緒に接続成功メッセージを確認できます。ゲームサーバーに接続が完了したクライアントは、サーバーとメッセージをやり取りできます。

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_connection_test.png)

<br>

## Room及びUserの作成

サーバーに接続したクライアントを**ゲームユーザー**と呼びます。サーバーに接続したクライアントは、サーバー上で1つ以上のゲームユーザー(User)としてログインできます(この例では1つのユーザーとしてログインする場合を扱います)。ゲームユーザーは1つの**ゲームルーム**に属することで、同じルームに属する他のユーザーと通信できます。つまり、ユーザーが他のユーザーとゲームに関するメッセージを交換するには、そのユーザーは同じルーム(Room)内に属している必要があります。

GameAnvilでは、ゲームユーザーとゲームルームの基本的な実装をあらかじめ用意しているので、エンジンのクラスを拡張し、コネクタのAPIを利用して簡単にゲームユーザーとルームの構造を完成させることができます。エンジン側でゲームユーザーとルームを定義する方法を説明し、コネクタ側でルームの作成や参加などを要求するAPIを使用する例を説明します。

### サーバー側の作業

サーバーでは、ゲームユーザーとゲームルームの機能をクラスで定義します。定義したクラスにGameAnvilが提供するアノテーションをつけることで、ゲーム実行時に自動的にクラスをスキャンし、適切なタイミングでゲームユーザーとルームオブジェクトが作成されます。まず、ゲームユーザーを定義してみましょう。Ctrl + N`または`Cmd + N`で新規ファイル作成コンテキストメニューを開き、GameUserを選択します。 (先にファイルテンプレートが正しくインストールされていない場合、GameUser項目が存在しないでしょう。ファイルテンプレートをインストールしてから進めてください。)

ダイアログボックスが表示されたら、上記のようにフィールドを入力します。各値は、クライアントとサーバー間で事前に協議された値で指定する必要があります。もし、下のフィールドに他の値を入力した場合は、覚えておいて、クライアントでユーザーを作成するためにログインする時、同じ値を入力する必要があります。ここでは、単一サービスと単一ユーザータイプを使うゲームサーバーを構築しているので、GameAnvilのマルチサービスとユーザータイプの概念については、この文書では詳しく説明しません。

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gameuser.png)



GameAnvilで提供するBaseUserクラスを継承してゲームユーザーを実装する基本コードが書かれたファイルが作成されます。GameAnvilで希望する機能のゲームユーザーを実装するには、BaseUserクラスを継承した後、状況に合わせて呼び出される様々なコールバック関数をオーバーライドして希望するコードを実行するように設定します。以下はサポートするコールバック関数の一部です。

- onLogin:ログイン要求時に実行されるコールバックです。戻り値としてログイン要求を許可するかどうかを渡す必要があります。 falseを返すと、クライアントはログインに失敗します。パラメータでクライアントから受け取ったペイロードを参照することができ、クライアントに返すペイロードを参照することで、ログイン許可情報以外に追加情報をクライアントに渡すことができます。
- onPostLogin:ログイン後に実行されるコールバックです。
- onReLogin:既にログインしたことがあるユーザーに対して再度ログインリクエストが来る場合は、このコールバックで別途処理できます。
- onDisconnect:クライアントのリクエストによるログアウトまたはサーバーによる強制ログアウトによってサーバーに登録されたユーザー情報と接続が切れた時に呼び出されるコールバックです。

それ以外のコールバック関数はログイン過程に関わらないので、全てのコールバックが何を意味するのか完璧に知る必要はありません。

onLoginコールバックメソッドではログイン過程で実行される動作を実装します。この例では特にログインを実装することなく、無条件にログインが成功するようにtrueを返すようにします。

```java

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.annotation.UserType;
import com.nhn.gameanvil.node.game.BaseUser;
import com.nhn.gameanvil.node.game.data.RoomMatchResult;
import com.nhn.gameanvil.packet.Payload;
import com.nhn.gameanvil.packet.message.MessageDispatcher;
import com.nhn.gameanvil.serializer.TransferPack;

@ServiceName("BASIC_SERVICE")
@UserType("BASIC_USER")
public class BasicUser extends BaseUser {

    private static final MessageDispatcher<BasicUser> messageDispatcher = new MessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public MessageDispatcher<BasicUser> getMessageDispatcher() {
        return messageDispatcher;
    }

    @Override
    public boolean onLogin(Payload payload, Payload sessionPayload, Payload outPayload) throws SuspendExecution {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onPostLogin() throws SuspendExecution {

    }

    @Override
    public boolean onLoginByOtherDevice(String newDeviceId, Payload outPayloadForKickUser) throws SuspendExecution {
        return true;
    }

    @Override
    public boolean onLoginByOtherUserType(String userType, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean onReLogin(Payload payload, Payload sessionPayload, Payload outPayload) throws SuspendExecution {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onDisconnect() throws SuspendExecution {
    }


    @Override
    public void onPause() throws SuspendExecution {

    }

    @Override
    public void onResume() throws SuspendExecution {

    }

    @Override
    public void onLogout(Payload payload, Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean canLogout() throws SuspendExecution {
        boolean canLogout = true;
        return canLogout;
    }

    @Override
    public void onPostLeaveRoom() throws SuspendExecution {

    }

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, Payload payload) throws SuspendExecution {
        return null;
    }

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, Payload payload, Payload outPayload) throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canTransfer() throws SuspendExecution {
        return true;
    }

    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {

    }

    @Override
    public void onTransferIn(TransferPack transferPack) throws SuspendExecution {

    }

    @Override
    public void onPostTransferIn() throws SuspendExecution {

    }

    @Override
    public boolean onCheckMoveOutChannel(String destinationChannelId, Payload payload, Payload errorPayload) throws SuspendExecution {
        boolean canMoveOut = false;
        return canMoveOut;
    }

    @Override
    public void onMoveOutChannel(String destinationChannelId, Payload outPayload) throws SuspendExecution {
    }

    @Override
    public void onPostMoveOutChannel() throws SuspendExecution {
    }

    @Override
    public void onMoveInChannel(String sourceChannelId, Payload payload, Payload outPayload) throws SuspendExecution {
    }

    @Override
    public void onPostMoveInChannel() throws SuspendExecution {

    }
}
```

ログイン可能なユーザーの実装が完了しました。次に、ゲームルームを実装します。ユーザー作成方法と同じようにファイルテンプレートを使ってBaseRoomクラスのサブクラスを作成します。この時、ゲームユーザー作成時に入力したサービス名と同じサービス名を入力する必要があります。ルームタイプ情報は後日、クライアントからルーム作成を要求する際に必要なので、覚えておいてください。ユーザークラス入力フィールドには前段階で作成したBaseUser実装クラスのクラス名を入力します。

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gameroom.png"/>

**OK**をクリックすると、サポートするコールバックメソッドが自動的に作成されます。GameAnvilでサポートするコールバックを説明するために、クライアントのAPIを簡単に説明します。クライアントでは、コネクタでログインした後、他のユーザーと通信するためにルーム関連APIを呼び出すことができます。ルームを作成したり、他のユーザーが作ったルームに参加したり、ルームから出るなどの動作をサポートします。

- onCreateRoom:ユーザーがルームの作成を要求したときに実行されます。ルームの作成を要求したユーザー情報とルーム作成時にクライアントから渡された追加情報をパラメータとして提供するので、この情報を基にルームの作成を許可するかどうかを決定して返して実装します。
- onJoinRoom:他のユーザーが作成したルームへの入室を要求した時に実行されます。他のコールバックと同じように、ルームへの入室を要求したユーザー情報と、ルームへの入室時にクライアントから渡された追加情報を提供するので、この情報を基にルームへの入室を許可するかどうかを決定して返して実装します。
- onLeaveRoom:ユーザーがルームから退室するときに実行されます。他のコールバックと同じように、ルームから退室を要求したユーザー情報と、ルームから退室を要求する時に一緒に渡した追加情報を提供するので、この情報を基にルームから退室を許可するかどうかを決定して返して実装します。

GameAnvilコールバックの呼び出し時点と戻り値の意味に一貫性があるため、他のコールバックの動作を推測しやすくなります。GameAnvilコールバックはどのような動作が要求され、処理される直前または直後に呼び出され、戻り値はリクエストをサーバーで処理することを許可するかどうかを決定します。サーバーがリクエストを完全に処理したかどうかは、パケットの形でエンジンを介してクライアントに渡されます。

ここでは、これらのコールバックを利用して、ルームに入場しているユーザーの情報を保存する機能を実装します。ユーザーがルームを作成したり、ルームに入場した時点でリクエストを渡したユーザーをルームオブジェクトのフィールドusersマップに保存します。ユーザーがルームから退場する時点では、usersマップから該当ユーザーを削除します。下記のコードを参考にしてゲームルームクラスの実装を完了します。

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.BaseRoom;
import com.nhn.gameanvil.node.game.BaseUser;
import com.nhn.gameanvil.node.game.RoomMessageDispatcher;
import com.nhn.gameanvil.packet.Payload;

import java.util.HashMap;
import java.util.Map;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoom extends BaseRoom<BasicUser> {
    private static RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>();
    private Map<Integer, BaseUser> users = new HashMap<>();

    @Override
    public final RoomMessageDispatcher<BasicRoom, BasicUser> getMessageDispatcher() {
        return dispatcher;
    }

    @Override
    public boolean onCreateRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(user.getUserId(), user);
        return true;
    }

    @Override
    public boolean onJoinRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(user.getUserId(), user);
        return true;
    }

    @Override
    public void onInit() throws SuspendExecution {

    }

    @Override
    public void onDestroy() throws SuspendExecution {

    }

    @Override
    public boolean onLeaveRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.remove(user);
        return true;
    }

    @Override
    public void onLeaveRoom(BasicUser user) throws SuspendExecution {

    }

    @Override
    public void onPostLeaveRoom() throws SuspendExecution {

    }

    @Override
    public void onRejoinRoom(BasicUser user, Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean canTransfer() throws SuspendExecution {
        return false;
    }
}
```

ゲームユーザーとゲームルームの準備ができました。しかし、ゲームユーザー/ゲームルームの作成と削除要求を処理するノードがまだありません。ゲームユーザーとゲームルームを管理する役割をするノードはGameNodeです。このノードは、一般的にゲームサーバーが期待するほとんどのゲームロジックを処理する役割をするノードです。GameAnvilにノードを追加する方法は自然で簡単です。ゲームユーザーとゲームルームを定義したのと同じように、あらかじめ定義されたクラスを継承してクラスを作成した後、必要な機能を追加実装するだけです。GameAnvilが提供するアノテーションを付けると、実行時に適切なタイミングでノードインスタンスが作成され、自動的に実行されます。

サービス名を入力する時、先に作成したゲームユーザーとゲームルームで使ったサービス名と同じサービス名を入力する必要があります。チュートリアルをそのまま踏襲して作成した場合、下記のように入力した後、**OK**をクリックして完了します。


<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gamenode.png"/>

ノードが役割を実行するためにはまず、ノードがループを実行する必要があります。ノードが実行される時、一連の過程を経るので少し時間が必要です。ノードが実行中かどうか、または実行過程のどの段階にあるかを示す指標をノードの状態と言います。ノードの状態は通常、下記の順番で順次変化し、READY状態になります。

- INIT
- PREPARE
- READY

READYステータスに到達したノードは、ユーザーが事前に作成したロジックを実行する準備が整った状態です。各準備段階に到達したときに特定のコードを実行したい場合は、コールバックメソッドを実装して、エンジンがコールバックメソッドを呼び出したときにそのコードが実行されるように設定できます。今は特別に実行するコードがないので、作成されたコードをそのまま使います。


```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.BaseGameNode;
import com.nhn.gameanvil.node.game.data.BaseChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.BaseChannelUserInfo;
import com.nhn.gameanvil.node.game.data.ChannelUpdateType;
import com.nhn.gameanvil.packet.Packet;
import com.nhn.gameanvil.packet.message.MessageDispatcher;
import com.nhn.gameanvil.packet.Payload;

@ServiceName("BASIC_SERVICE")
public class BasicGameNode extends BaseGameNode {

    private static final MessageDispatcher<BasicGameNode> messageDispatcher = new MessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public void onInit() throws SuspendExecution {

    }

    @Override
    public void onPrepare() throws SuspendExecution {

    }

    @Override
    public void onReady() throws SuspendExecution {

    }

    @Override
    public MessageDispatcher<BasicGameNode> getMessageDispatcher() {
        return messageDispatcher;
    }

    @Override
    public void onPause(Payload payload) throws SuspendExecution {

    }

    @Override
    public void onResume(Payload payload) throws SuspendExecution {

    }

    @Override
    public void onShutdown() throws SuspendExecution {

    }

    @Override
    public boolean onNonStopPatchSrcStart() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchDstStart() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchDstEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canNonStopPatchDstEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, BaseChannelUserInfo baseChannelUserInfo, int i, String s) throws SuspendExecution {

    }

    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, BaseChannelRoomInfo baseChannelRoomInfo, int i) throws SuspendExecution {

    }

    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {

    }

}
```

最後に作成したゲームユーザーとゲームルームをGameNodeに連動するため設定ファイルにも登録します。GameAnvilConfig.jsonファイルの'game'項目に下記の内容を追加します。


```json
// ゲームロビーの役割をするノード(ゲームルーム、ユーザーを含んでいる)
  "game": [
    {
      "nodeCnt": 8,
      "serviceId": 1,
      "serviceName": "BASIC_SERVICE",
      "channelIDs": [
        "",
        "",
        "",
        "",
        "",
        "",
        "",
        ""
      ],
      "userTimeout": 5000,                // ノードごとに付与するチャネルID(ユニークである必要はありません。""文字列でチャネルを区別せずに重複して使用することも可能)
      "safeCreateTime": 1000,             //テストのために設定。通常は設定不要。基本60秒。

	  "checkClientStateCycle": 10000, 		// チェックルーチン呼び出し周期
	  "demandClientStateTimeout": 10000,	// 最後のメッセージを受け取ってからdemandClientStateTimeoutが経過したらdemandClientStateを実行する。
	  "clientStateOkDeadline": 10000		// demandClientStateをリクエストに対してClientStateOKをレスポンスすべきデッドライン(時間)
    }
  ],
```

これで、クライアントがサーバーに接続し、ゲームユーザーとしてログインし、ゲームルームを作成する機能が実装完了しました。しかし、サーバーに接続したからといって、すぐにゲーム関連機能(ゲームユーザーの作成、ゲームルームの作成など)を要求できるわけではありません。今の状態でサーバーとクライアントを実行しても、クライアントはゲームサーバーの機能を使うことができないでしょう。サーバーにこれらのことを要求するには、サーバーに接続した後にクライアント認証プロセスが必要です。次の章では、サーバーとクライアントで認証をどのように処理するかについて説明します。


<br>

## 接続認証

クライアントがサーバーに接続した後、ゲームにログインする前に、ユーザーの身元を確認し、認証する必要があります。
 ゲームノードがゲームユーザーとゲームルームの作成を担当するなら、ゲートウェイノードはユーザーの接続と認証機能を担当します。ゲームノードをクラス作成を通じて実装したように、ゲートウェイノードも一貫性のある方法で実装できます。GameAnvilでは基本的なゲートウェイを自動的に作成するので、この章ではサーバー側の実装は省略します。

### 接続認証コードの追加

Unityプロジェクトに移動して、クライアント側の実装を行います。クライアントでは、接続要求と同様に、認証要求をコネクタ接続エージェントAPIを通じて要求できます。認証要求は接続要求の後にだけ成立することができるので、接続に成功した直後に認証要求を行うようにコードを修正します。認証要求をする時は、認証要求の結果によるコールバックを渡して結果を受け取って画面上のテキストとコンソールを通じて出力して接続情報を知ることができるようにします。

```c#
// Client-side

[SerializeField]
GameAnvil.Connector.Config config;

public GameAnvil.Connector connector = null;
private ConnectionAgent connectionAgent;

public string ip = "127.0.0.1";
public int port = 11200;

public string deviceId = "";
public string password = "";
public string accountId = "";

public Text connectInfoText;
public Text accountIdText;

void Start () {
    Connect();
  
    serverInfoText.text = "サーバー情報: " + ip + ":" + port;
}

public void Connect(){
    connectionAgent = connector.GetConnectionAgent();
    connectionAgent.Connect(ip, port,
        (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
            Debug.Log(result);

           if (result == ResultCodeConnect.CONNECT_FAIL) {
                logText.text = "接続情報:接続失敗";
            } else {
                logText.text = "接続情報:接続成功";
           
             	// 接続に成功したら、すぐに認証を試みます。
                Auth();
            }
        }
    );
}
public void Auth(){
    accountId = Random.Range(1000,9999) + "";
  	clientInfoText.text = "クライアント情報: " + accountId;
		connectionAgent.Authenticate(deviceId, accountId, password,
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
                Debug.Log(result);
         
                if (result == ResultCodeAuth.AUTH_SUCCESS) {
                    logText.text = "認証情報:認証成功";
                } else {
                    logText.text = "認証情報:認証失敗";
                }
            }
			);
}
```

これでクライアントがサーバーに接続するだけでなく、認証プロセスまで要求できるように設定されました。

<br>

### 接続認証の確認

Unityクライアントでプレイモードに入ります。コンソール上にログが接続、認証の順に出力されることを確認します。ゲーム画面上に認証情報と認証の成否が表示されることを確認できます。まとめると、クライアントのAuthリクエストにより、サーバーのゲートウェイノードでユーザー認証が完了し、サーバーにログインできる状態になりました。

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_authentication_test.png)

<br>

## ログイン

最後に行うプロセスはゲームノードにログインしてゲームユーザーを作成することです。ゲームユーザーはサーバーに接続した他のクライアントと通信するために必要な概念で、クライアント間の通信をするためには各クライアントはゲームユーザーを作成してゲームルームを通じてメッセージをやり取りするようにします。

### ログインコードの追加

接続及び認証が完了したクライアントは、ゲームノードにログインできます。サーバーにログインすると、ゲームノードにそのクライアント用のゲームユーザーオブジェクトが作成されます。クライアントは自分のサーバー側ゲームユーザーオブジェクトを通じてサーバーや他のユーザーとメッセージをやり取りできます。先に作った認証コードを下記のように修正して認証が成功した場合、すぐにログインを行うようにします。

```c#
// Client-side

public void Auth(){
    accountId = Random.Range(1000,9999) + "";
    connectionAgent.Authenticate(deviceId, accountId, password,
     (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
        Debug.Log(result);
 
        if (result == ResultCodeAuth.AUTH_SUCCESS) {
            logText.text = "認証情報:認証成功";
              
            Login(); // 認証に成功した場合、すぐにログインを試みます。
        } else {
            logText.text = "認証情報:認証失敗";
        }
    });
}

public void Login()
{
    connector.CreateUserAgent("BASIC_SERVICE", 1).Login("BASIC_USER", "", null, (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => {
        Debug.Log(result);

        if (result == ResultCodeLogin.LOGIN_SUCCESS) {
            logText.text = "ログイン情報:ログイン成功";
        } else {
            logText.text = "ログイン情報:ログイン失敗";
        }
    });
}
```

ログインをリクエストする時、一緒に渡したコールバックメソッドがログインリクエスト結果をログで出力するようにします。

<br>

### ログイン確認

Unityテストモードで正常にログインできることを確認します。

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_login_test.png"/>

<br>

## ルームの作成及び参加申請ページに戻り、次の段階を進めてください。

### クライアントの作業

UnityプロジェクトでConnectHandlerコードにルームの作成をリクエストするメソッドを追加します。この時、userAgent.CreateRoomメソッドの最初の引数であるRoomTypeは必ずサーバーで指定した値と同じでなければならないことに注意してください。一般的にこのようなRoomTypeなどのプロトコルは、サーバーとクライアントの開発者が事前に値を定義しておきます。サーバー開発段階でチュートリアルと同じ値を指定したら、下記のコードのサービス名をそのまま使ってください。ルームの作成結果を受け取るコールバックを登録し、結果コードをコンソールに出力します。また、ルームの作成に成功したら、roomIdをクライアント側に保存しておいて、ゲームシーンに移動するようにコードを書きます。

```c#
using UnityEngine.SceneManagement;

public int roomId;
public UserAgent userAgent;

public void CreateRoom(){
    connector.GetUserAgent("BASIC_SERVICE", 1).CreateRoom("BASIC_ROOM", (UserAgent ua, ResultCodeCreateRoom resultCode, int roomId, string roomName, Payload payload) => {
        Debug.Log(resultCode); 
        if (resultCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
            this.roomId = roomId;
       
            SceneManager.LoadScene("GameScene");
        }
    });
}
```

コードを作成したら、階層関係パネルで**Create Room**をクリックします。インスペクタの**Button**コンポーネントから**OnClick**リスナー項目にConnectHandlerコンポーネントをドラッグして参照を登録し、ドロップダウンメニューから**CreateRoom**メソッドを選択して画面上のボタンでメソッドを実行できるように設定します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_create_room_on_click.png)

ここまで完了したら、クライアントを通じてルームを作成する機能を実装したことになります。実際に実行する前にルーム参加機能まで実装してテストしてみます。今回はConnectHandlerにルーム参加コードを追加します。

userAgent.JoinRoomメソッドの最初の引数がサーバー開発で使った事前定義されたRoomType文字列と一致するか確認します。コールバックでルーム参加リクエストに対するサーバー側のレスポンスを確認してルーム参加に成功したら、ルームの作成と同じ動作をするように作成します。メソッドの作成が終わったら、Join RoomボタンのOnClickイベントと接続します。

```c#
private void Update() {
    connector.Update();

    if (popupCanvas && popupCanvas.activeInHierarchy && Input.GetKeyDown(KeyCode.Return)){
        JoinRoom();
    }
}

public void JoinRoom(){
    if (!string.IsNullOrEmpty(roomIdInput.text)){
        connector.GetUserAgent("BASIC_SERVICE", 1).JoinRoom("BASIC_ROOM", int.Parse(roomIdInput.text), null, (UserAgent userAgent, ResultCodeJoinRoom resultCode, int roomId, string roomName, Payload payload)=>{
            Debug.Log(resultCode);
            if ( resultCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS){
                this.roomId = roomId;
            
                SceneManager.LoadScene("GameScene");
            }
        });
    }
}
```

次はGameSceneに移動してルーム参加後どんな動作をするのか実装してみます。まず、GameManagerコードを下記のように修正してルームIDをゲーム上で表示できるようにします。

```c#
public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }
}
```

<br>

これでクライアントにサーバーへの接続、認証、ログイン機能だけでなく、ルームの参加と作成機能まで全て実装しました。

### Roomの作成及び参加テスト

Unityプロジェクトの上部のツールバーから**File > Build Setting**を選択します。下記のように必要なシーンをビルドするシーンリストに順番に追加します。もし、シーンの順番が間違っている場合は、リスト上の項目をドラッグしてConnectシーンが一番上に来るように順番を調整します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/add_scene.png)

Unityで`cmd+b`または`ctrl+b`でビルド後にプレイします。新しいウィンドウが開き、Connectシーンがロードされます。接続と認証、ログインが完了したことを確認します。もし、中間過程で失敗ログが残る場合は、失敗コードで原因を知ることができます。もしそれで足りない場合は、サーバーのログを確認して原因を分析する必要があります。ログインまで成功したら、Create Roomボタンを押してルームの作成をリクエストします。シーンが移動されて作成されたルームのIDが一緒に出力されます。ルームIDは次の例題イメージと違う場合があります。

その状態でUnityプレイモードを実行した後、ログインまで完了することを確認します。そして、Join Roomボタンをクリックした後、先に作成したルームのRoomIdを直接入力してルームに参加するようにします。ルームへの参加に成功したら、ゲームシーンに移動され、2つのゲーム画面で同じルームIDを持っていることが確認できるはずです。テスト画面は下記のように表示されたら成功です。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_room_test.png)

<br>

うまくいかない場合は、下記の内容を再確認してください。
- サーバー実装の修正後、サーバープロセスを再起動したか？
- サービス名はGameAnvilConfig.jsonに設定したものと同じようにサーバー/クライアントに実装されているか？

## ゲーム内チャットの実装

これで、クライアント間でサーバーを介して通信できる環境が構成されました。ここでは、クライアントで作成したデータをリモートのクライアントが受け取ることができる簡単な例を実装してみます。例のプロジェクト内部にあらかじめ実装されたプロトコルを利用して、チャット履歴を送受信する方法を説明します。この例に限って、通信データをクライアントからサーバーへ送信する時はMessageRequestクラスを使います。サーバーからクライアントへ通信データを送る時はMessageResponseまたはMessageBroadcastクラスを使います。

(プロジェクトテンプレートを使って直接プロジェクトを作成した場合、Messageクラスは利用できません。チュートリアル用に作られたプロジェクトをダウンロードして、内部のBasicProtocol.javaがプロジェクトに含まれるように設定してください。また、Bootstrap時にプロトコルを登録する過程も必要です。サーバー側のプロトコル関連設定は、予め準備されたチュートリアル用プロジェクトに予め完了しているので、チュートリアルをそのまま踏襲した場合は、行う必要がありません)。

<br>

### クライアント側送信の実装

まず、クライアントからサーバーへメッセージを送るコードを書いてみます。一度ログインしたユーザーとしてサーバーに接続したら、ユーザーエージェントを使ってサーバーの機能を活用できます。ここでは、ユーザーエージェントのSend機能を利用してルームにパケットを送信します。メッセージを送信するためには、送信する内容を入れたパケットを引数として渡す必要があります。

UnityプロジェクトのGameManagerスクリプトにメッセージ送信スクリプトを追加します。SendMessageメソッドが呼び出されると、画面上の入力フィールドにユーザーが入力したメッセージと自分のユーザーIDを入れたパケットをuserAgentを使ってサーバーへ送信します。そして、Enterキーを押すたびにSendMessageメソッドが実行されるようにUpdate関数に実装を追加します。下記のサンプルコードはSendMessageメソッドの実装です。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using Protocol;


public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }

    void Update(){
	 	 if ( Input.GetKeyDown(KeyCode.Return) && !string.IsNullOrEmpty(ChatInputText.text)){
	 	 		SendMessage();
	 	 }
	}

	public void SendMessage(){
        MessageRequest messageRequest = new MessageRequest();
        messageRequest.Message = "\n[" + connectHandler.accountId +  "]:" + ChatInputText.text;
        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).Send(new Packet(messageRequest));
        ChatInputText.text = string.Empty;
	}
}
```

このようにすると、クライアントからサーバーへパケットを送信する機能が実装されました。しかし、今はパケットをサーバーへ送ってもサーバーからは何のレスポンスもないでしょう。その理由は、サーバー側でそのパケットを受け取った時、内容をどのように分析してどのような動作をするのか定義していないからです。次の章ではサーバー側の実装を説明します。

<br>

### サーバー側レスポンスの実装

ここでは、ゲームユーザーが送信したメッセージをサーバーが受け取り、ルーム内のユーザーに送信する機能を作成します。クライアントから送信されたメッセージをサーバーのゲームルームで処理するためには、ハンドラを使用します。ハンドラとは、特定のプロトコルを処理するためのコードの束を意味します。ハンドラはプロトコルの種類によって複数のハンドラがあり、ルームに複数のハンドラを登録できます。したがって、ルームは複数のプロトコルを処理できます。

サーバープロジェクトに移動してBasicRoomクラスをもう一度確認します。先ほどBaseRoomクラスを継承したBasicRoomクラスでRoomMessageDispatcherを作成しました。パケットディスパッチャーはメッセージを受信した時、メッセージが適切な(プログラマが意図した)ハンドラを探して実行する役割をします。今回作成する受信メッセージに対するハンドラもこのパケットディスパッチャーに登録されます。

ハンドラもクラス作成で作成します。新しいクラスファイルをBasicHandlerという名前で作成します。そして、RoomPacketHandlerを継承するようにし、executeメソッドをオーバーライドします。その後、パケットディスパッチャーに登録することで、MessageRequestを受信した時にこのメソッド内の内容が実行されます。

```Java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import org.slf4j.Logger;
import protocol.BasicProtocol;

import static org.slf4j.LoggerFactory.getLogger;

public class BasicHandler implements RoomMessageHandler<BasicRoom, BasicUser, BasicProtocol.MessageRequest> {
    private static final Logger logger = getLogger(BasicHandler.class);

    @Override
    public void execute(BasicRoom room, BasicUser requester, BasicProtocol.MessageRequest request) throws SuspendExecution {

    }
}
```

メソッド内部を実装する前に、BasicRoomクラスにハンドラで使ったbroadcastメソッドを実装します。ルームの全てのユーザーにメソッドで入ってきたパケットを複製して送る役割をするメソッドです。そして、MessageRequestを受信したら、先に作成したハンドラを実行するように登録するため、registerMsgメソッドでプロトコルのディスクリプタとハンドラクラスを登録します。この時、メッセージハンドラの登録は必ずstaticブロック内に実装する必要があります。

```Java
public class BasicRoom extends BaseRoom<BasicUser> {
    private static RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>();
    private Map<Integer, BaseUser> users = new HashMap<>();

    static {
        dispatcher.registerMsg(BasicProtocol.MessageRequest.class, BasicHandler.class); // 処理するメッセージのハンドラを登録
    }

    @Override
    public final RoomMessageDispatcher<BasicRoom, BasicUser> getMessageDispatcher() {
        return dispatcher;
    }

    ...省略...

    public void broadcast(com.google.protobuf.GeneratedMessageV3 message) {
        users.values().stream().forEach(user -> user.send(message));
    }

}
```

実行される内容、つまり、executeメソッドの内部実装は下記のように作成します。下記のハンドラの実装例では、受信したメッセージに対して送信者に応答メッセージを送信すると共に、ルーム全体のユーザーにルーム単位ブロードキャスト用のメッセージを追加で送信します。この時、クライアントはルーム単位のブロードキャストメッセージを基準にゲームを同期するように実装されています。

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import org.slf4j.Logger;
import protocol.BasicProtocol;

import static org.slf4j.LoggerFactory.getLogger;

public class BasicHandler implements RoomMessageHandler<BasicRoom, BasicUser, BasicProtocol.MessageRequest> {
    private static final Logger logger = getLogger(BasicHandler.class);
  
  	@Override
    public void execute(BasicRoom room, BasicUser requester, BasicProtocol.MessageRequest request) throws SuspendExecution {
        BasicProtocol.MessageResponse response = BasicProtocol.MessageResponse.newBuilder().setMessage(request.getMessage()).build();
        BasicProtocol.MessageBroadcast broadcast = BasicProtocol.MessageBroadcast.newBuilder().setMessage(request.getMessage()).build();

        requester.reply(response); // 送信者にレスポンス
        room.broadcast(broadcast); // ルームの全ユーザーにメッセージを送信
    }
}

```

上のコードではまず、受信したパケットのストリームを解析してMessageRequestオブジェクトを作ります。その後、MessageRequestオブジェクトのMessage値を使ってMessageResponse、MessageBroadcastオブジェクトをそれぞれ新しく作成します。MessageResponseタイプのオブジェクトはパケットをルームに送信したユーザーオブジェクトに送信します。MessageBroadcastオブジェクトはroomを通じてルーム内の全てのユーザーに送信します。

このように、クライアントが送信したパケットをサーバーが受信し、少しの処理をした後、再び返す機能がサーバーに追加されました。この時、クライアントもサーバーが送信したパケットをどのように処理するかを指定する必要があります。
<br>

### クライアント側の受信の実装

クライアント側でもサーバー側のパケットを処理するために、あらかじめハンドラを登録する必要があります。そうしないと、パケットを受け取ったときに、処理方法がわからないプロトコルと判断して内容を捨ててしまいます。サーバーから送信する内容を受信した時にこれを検知して内容を処理するには、サーバーから送信するパケットのプロトコルタイプのハンドラを登録します。つまり、MessageBroadcastタイプのメッセージを処理するハンドラ登録コードを作成して登録します。

再びUnityプロジェクトに移動してGameManagerのStartメソッドに実装を追加します。まず、プロトコルマネージャーを使ってサーバーとクライアント間で合意したプロトコルを登録します。ここでは0番にBasicProtocolを登録しました。番号については後述します。

その後、ユーザーエージェントを使ってリスナーを登録します。メソッドタイプにはプロトコルのタイプを使用します。サーバーから送信するように設定したプロトコルはMessageBroadcastなので、これを使用します。ハンドラの実装は、受信したメッセージを画面に用意されたテキストにそのまま出力するシンプルなロジックで作成します。これがチャットの最も基本的な機能です。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using Protocol;

public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        ProtocolManager.GetInstance().RegisterProtocol(BasicProtocolReflection.Descriptor);
        connectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).AddListener<MessageBroadcast>((sendUserAgent, messageBroadcast) => {
            ChatLogText.text += messageBroadcast.Message;
        });
    }
  
		...省略...
}

```

<br>

### メッセージ伝達確認

サーバーを修正した後、新しく実行したことを確認し、Unityで`cmd+b`または`ctrl+b`でビルドしてからプレイします。ビルドされたゲームでルームを作成し、サーバー側のログを確認します。その状態でUnityエディタでプレイモードに入った後、先に作成したルームのRoomIdを入力してそのルームに参加します。

任意のゲームプロセスで入力欄にテキストを入力した後、Enterキーを押すと、他のゲームプロセスでも同じテキストが表示されます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_chat_test.png)

簡単なチャットサーバーの実装を通じてメッセージの処理過程を学習しました。次は、もっと実用的な例の実装過程を見てみましょう。

<br>

## パズルゲームの実装

ゲームシーンにはシングルプレイが可能なパズルゲームがあらかじめ実装されています。プレイモードに入り、パズルのピースをドラッグして適切な位置に配置すると、グリッド上の正確な位置に補正されます。この章では、このゲームをマルチプレイヤーゲームにしてみます。

同じルーム内のユーザー間でやり取りするメッセージは、MessageRequest、MessageResponse、MessageBroadcast以外にも、どのようなタイプの値でも表現できます。この章では、直接プロトコルを定義し、サーバーとクライアントに登録する方法を説明します。

メッセージはあらかじめ定義したプロトコルに基づいて定義するだけで、サーバーとクライアント間で送受信できます。XML、jsonなど様々な表現手段がありますが、GameAnvilはGoogle Protocol Buffersを使います。これは速度と安定性の面で最も良いソリューションの1つです。

<br>

### Google Protocol Buffersを利用したメッセージのシリアライズ/逆シリアル化

プロトコルバッファを使用するには、まず、メッセージをどのように定義するかの仕様を作成する必要があります。例えば、MessageRequetの仕様には単一の文字列を含めます。その後、プロトコル仕様をコンパイルし、目的の言語のファイルに変換します。その後、MessageRequestを使用した方法と同様に、パケットのメッセージプロトコルとして使用できます。

パズルの位置同期のためのメッセージプロトコルの作成を始めます。サーバープロジェクトのsrc/main/protoのパスにPuzzle.protoファイルを追加した後、下記のようにプロトコル仕様を作成します。

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto.png)

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}
```

PuzzlePositionプロトコルはパズルのピースの最新位置を表すため、各パズルのピースの固有番号情報(Index)とパズルの位置情報(PositionXとPositionY)で構成します。これでプロトコルの定義は終わりました。次はこのようなプロトコル定義を各開発言語に合うクラスにコンパイルします。

protoc実行ファイルはプロジェクトの最上位パスにあります。該当プロジェクトに移動して次のコマンドラインを使ってJavaとC#のためのコンパイルを実行します。コマンドラインを実行した時、結果メッセージが何も表示されなければ成功です。

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

src/main/javaのパスに新しくPuzzle.javaクラスが作成されたことが確認できます。

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto_java.png)

C#クラスファイルは、ファインダーやファイルエクスプローラーなどのプログラムを利用してUnityプロジェクトのAsset/Protocolパスに移動します。

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto_csharp.png)

パズル位置同期のためのメッセージプロトコルの作成が終わりました。

<br>

### プロトコルの登録

プロトコルを定義してコンパイルまで無事に終わったら、サーバーとクライアントの両方に該当プロトコルクラスを登録する必要があります。ちなみに、チャットプロトコル(MessageRequestなど)の場合、基本的にテンプレートに登録されているので追加で登録する必要がありませんでした。しかし、追加で実装したPuzzleプロトコルは直接登録する必要があります。GameAnvilサーバーのMainメソッドで下記のようにプロトコルを登録します。

```C#
public class Main {

    public static void main(String[] args) {
        GameAnvilServer server = GameAnvilServer.getInstance();

        server.addProtoBufClass(BasicProtocol.class);
        server.addProtoBufClass(Puzzle.class);

        server.run();
    }

}
```

UnityプロジェクトはConnectHandler Startメソッドに下記のようにプロトコルを追加します。この時、サーバーと同じインデックスである1で登録します。

```c#
using Protocol;

public class ConnectHandler : MonoBehaviour {
    void Start()
    {
        // プロトコル登録
        ProtocolManager.GetInstance().RegisterProtocol(PuzzleReflection.Descriptor);
    }
  
  	...省略...
}
```

<br>

### クライアント側送信の実装

これで、ゲーム用のプロトコルの定義と登録まで全て完了しました。これからは、これらのプロトコルに基づいたメッセージを実際に送信する機能を実装します。まず、クライアント側でデータを送信する部分を先に実装します。パズルのピースをドラッグする間、その位置をサーバーに送信してみます。

Unityプロジェクトに移動してPuzzle.csファイルを開いて下記のようにコードを追加します。ドラッグしている間、パズルの固有の位置情報を入れたメッセージオブジェクトを作成します。そして、userAgentを使ってサーバーに送信します。メッセージのプロトコルだけ違うだけで先に実習したMessageRequestと同じ方式であることが分かります。

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;
using Protocol;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using GameAnvil.User;

public class Puzzle : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public int index;

    public Transform puzzleHolder;
    public float tolerance;

  
    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();
    }

    public void OnBeginDrag(PointerEventData eventData)
    {
        transform.position = eventData.position;
    }
  
    public void OnDrag(PointerEventData eventData)
    {
        transform.position = eventData.position;
        SendPuzzlePosition(transform.position, false);
    }
  
    public void OnEndDrag(PointerEventData eventData)
    {
        FixPosition();
        SendPuzzlePosition(transform.position, true);
    }

    public void FixPosition(){
        if (Vector2.Distance(transform.position, puzzleHolder.position) < tolerance){
            transform.position = puzzleHolder.position;
        }
    }

    public void SendPuzzlePosition(Vector2 puzzlePosition, bool onEndDrag){
		PuzzlePosition position = new PuzzlePosition();
		position.Index = index;
        position.PositionX = (int)puzzlePosition.x;
        position.PositionY = (int)puzzlePosition.y;
        position.OnEndDrag = onEndDrag;
      
        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).Send(position);
    }
}
```



<br>

### サーバー側レスポンスの実装

クライアント側では継続的にパズルの位置をサーバーに送信するようになりました。次に、パズルの位置をサーバーでどのように処理するかを作成する必要があります。MessageRequestを処理してMessageResponse、MessageBroadcastでユーザーに返したように、パズルの位置を再びゲームルームの全てのユーザーに返すように実装してみます。

サーバープロジェクトに戻ってPuzzlePositionHandler.javaクラスファイルを作成します。そして、受け取ったパケットをそのままルーム内の全てのユーザーに渡すようにbroadcastメソッドを使います。

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import protocol.Puzzle;

public class PuzzlePositionHandler implements RoomMessageHandler<BasicRoom, BasicUser, Puzzle.PuzzlePosition> {

    @Override
    public void execute(BasicRoom room, BasicUser user, Puzzle.PuzzlePosition position) throws SuspendExecution {
        room.broadcast(position);
    }
}
```

先に実装したハンドラをBasicRoomのRoomDispacherに登録するようにします。ハンドラークラスファイルを作成してもRoomDispatcherに登録しないと、メッセージが届いた時、そのハンドラを実行できません。これでBasicRoomはBasicProtocol以外にもPuzzlePositionプロトコルを処理できるようになりました。

```java
public class BasicRoom extends BaseRoom<BasicUser> {
    private static final Logger logger = getLogger(BasicRoom.class);
    private static RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>();
    private Map<Integer, BaseUser> users = new HashMap<>();

    static {
        dispatcher.registerMsg(BasicProtocol.MessageRequest.class, BasicHandler.class);
        dispatcher.registerMsg(Puzzle.PuzzlePosition.class, PuzzlePositionHandler.class);
    }
  
    ...省略...
  
}
```

<br>

### クライアント側の受信の実装

先にサーバーから送信したMessageBroadcastを処理するため、あらかじめハンドラを登録する必要がありました。今回も同様にパズルの位置を処理するハンドラを作るため、GameManagerのStartメソッドにPuzzlePositionメッセージを処理するハンドラ登録コードを作成します。このハンドラはメッセージを受信したら、パズルのオブジェクトを探してサーバーから受け取った位置に移動させます。

```c#
public class GameManager : MonoBehaviour{
  
    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        ProtocolManager.GetInstance().RegisterProtocol(0, BasicProtocolReflection.Descriptor);

        // チャットメッセージリスナー
        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).AddListener<MessageBroadcast>((sendUserAgent, messageBroadcast) => {
            ChatLogText.text += messageBroadcast.Message;
        });
        // パズル位置リスナー
        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).AddListener<PuzzlePosition>((sendUserAgent, puzzlePosition) => {
            Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);

            if (puzzlePosition.OnEndDrag)
            {
                puzzle.FixPosition();
            }
        });
    }
    ... 省略 ...
}
```

<br>

### パズル位置の同期確認

Unityで`cmd+b`または`ctrl+b`でビルド後、プレイします。ビルドされたゲームでルームを作成した後、Unityプレイモードを実行して作成されたルームのRoomIdを入力してルームに参加します。これで、パズルのピースをドラッグして位置を移動すると、そのパズルの位置が同期されてリモートクライアントに反映されることが確認できます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_puzzle_position_playmode.gif)

<br>

### 途中参加ユーザーを処理する

ゲーム中にランダムなパズルのピース位置が変更された後、新しいユーザーがルームに入る場合を考えてみましょう。このとき、既存ユーザーと新規ユーザー間のパズルの状態は異なります。新しく入ってきたユーザーは初期のパズル状態を持っているので、既存ユーザーとパズルの状態を同期させる必要があります。これを解決するためにサーバー側のロジックを修正します。サーバーはパズルの位置情報を全て保存し、新しいユーザーが入室したら、その情報を利用して同期するようにロジックを修正します。

BasicRoomにpuzzlePositionsマップを追加します。このマップは各パズルピースの位置情報を管理します。

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    private static final RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>();
  
    private Map<Integer, BaseUser> users = new HashMap<>();
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();
  
  	...省略...
  
}
```

PuzzlePositionHandlerのコードを修正してパズルの位置を各ルームに保存するようにします。

```Java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import protocol.Puzzle;

public class PuzzlePositionHandler implements RoomMessageHandler<BasicRoom, BasicUser, Puzzle.PuzzlePosition> {

    @Override
    public void execute(BasicRoom room, BasicUser user, Puzzle.PuzzlePosition position) throws SuspendExecution {
        room.puzzlePositions.put(position.getIndex(), position); // マップに位置情報を保存
        room.broadcast(position);
    }
}

```

新しいユーザーがルームに入る時、保存していたパズルの位置情報を受け取れるようにBasicRoomのonJoinRoomを修正してサーバーに保存していたパズルの位置情報を送信するようにします。

```java
public class BasicRoom extends BaseRoom {
    @Override
    public boolean onJoinRoom(BasicUser basicUser, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(user.getUserId(), user);
        puzzlePositions.values().stream().forEach(this::broadcast); // パズルの位置の同期化
        return true;
    }
}
```

このように修正した後、もう一度テストしてみます。1つのクライアントで先にパズルの位置を修正した後、他のクライアントでゲームルームに参加してパズルの位置が正常に同期されるか確認します。意図と異なりまだ動作しないことが確認できます。

これは、onJoinRoomの呼び出し後にクライアントでシーンの移動が行われるため、リスナーの登録が完了する前に同期メッセージが配信され、問題が発生したためです。この問題を解決するには、クライアントでシーンの移動が終了し、リスナーの登録が完了した後にパズルの位置情報を同期する必要があります。

この問題は今すぐ解決することはせず、後述します。このような問題は、次に説明するパズルのシャッフルでも顕著に現れます。そのため、パズルのシャッフルから説明した後、この問題を解決するために修正した内容を再度説明します。

<br>

## パズルのシャッフルの実装

パズルの位置をランダムにシャッフルするロジックを実装してみましょう。クライアントからパズルの位置のシャッフルをリクエストすると、サーバーでパズルをシャッフルした後、新しい位置を決定します。そして、変更された位置情報をクライアントに返すのが基本的な考え方です。

### プロトコルの登録

まず、パズルのシャッフルリクエストをするためのプロトコルを作成してみましょう。サーバープロジェクトに移動し、Puzzle.protoファイルにプロトコル仕様を追加します。このプロトコルは特にクライアントからサーバーに送る情報がないので、フィールドが1つもありません。これもプロトコルとして十分意味があります。

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}

message ScatterPuzzle { } // パズルのシャッフルリクエストプロトコル
```

プロトコルバッファファイルを修正したら、コンパイルもやり直す必要があります。

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

作成されたC#クラスを再びファインダーやファイルエクスプローラーなどのプログラムを利用してUnityプロジェクトのAsset/Protocolフォルダに移動します。この時、サーバーは出力パスのおかげで新しくコンパイルされたクラスに自動置換されるので、別途作業する必要はありません。

<br>

### クライアント側の実装

Unityプロジェクトに移動してGameManager.csに下記のようにシャッフルリクエストのためのコードを作成します。Scatterメソッドが呼び出されると、ユーザーエージェントを通じて新しいScatterPuzzleタイプのメッセージがゲームルームに送信されます。

```csharp
public class GameManager : Monobehaviour {
	public void Scatter() {
        	ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).Send(new ScatterPuzzle());
    }
}
```

**Hierarchy**パネルの**Scatter Puzzle Button**をクリックします。**Inspector**の **Button** コンポーネントで**OnClick**リスナーに項目を追加した後、GameManagerコンポーネントをドラッグして登録し、ドロップダウンからScatterメソッドを選択します。

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_scatter_puzzle_on_click.png"/>

### サーバー側の実装

シャッフルリクエストが来た時の処理は先ほどと同じようにハンドラを利用します。新しいJavaファイルを作成してハンドラScatterPuzzleHandlerを作成します。16個の各パズルの位置をランダムに設定し、PuzzlePositonタイプのメッセージを送信します。また、サーバーのpuzzlePositionsマップも新しい位置情報に更新します。


<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/scatter_puzzle_handler.png"/>

```Java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import org.slf4j.Logger;
import protocol.Puzzle;

import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

import static org.slf4j.LoggerFactory.getLogger;

public class ScatterPuzzleHandler implements RoomMessageHandler<BasicRoom, BasicUser, Puzzle.ScatterPuzzle> {
    private static final Logger logger = getLogger(ScatterPuzzleHandler.class);
    private static final int mapSize = 400;

    private class Point {
        public int x;
        public int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    @Override
    public void execute(BasicRoom room, BasicUser user, Puzzle.ScatterPuzzle scatterPuzzle) throws SuspendExecution {
        room.puzzlePositions = new HashMap<>();
        
        List<Point> random = IntStream.rangeClosed(-4, 4).boxed()
            .map(i -> new Point(i * mapSize / 4, i%2==0 ? mapSize : -mapSize))
            .collect(Collectors.toList());
        random = Stream.concat(random.stream(), random.stream().map(p -> new Point(p.y, p.x))).collect(Collectors.toList());
        Collections.shuffle(random);

        for (int i = 0; i < 16; i++) {
            Point point = random.get(i);
            int x = 1300 + point.x;
            int y = 550 + point.y;

            Puzzle.PuzzlePosition puzzlePosition = Puzzle.PuzzlePosition.newBuilder().setIndex(i).setPositionX(x).setPositionY(y).build();

            room.puzzlePositions.put(i, puzzlePosition);

            room.broadcast(puzzlePosition);
        }
    }
}

```

このように実装したハンドラをRoomに登録します。

```java
public class BasicRoom extends BaseRoom<BasicUser> {
    private static final Logger logger = getLogger(BasicRoom.class);
    private static final RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>();
    
    private Map<Integer, BaseUser> users = new HashMap<>();
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();


    static {
        dispatcher.registerMsg(BasicProtocol.MessageRequest.class, BasicHandler.class); // 処理するメッセージのハンドラを登録
        dispatcher.registerMsg(Puzzle.PuzzlePosition.class, PuzzlePositionHandler.class);
        dispatcher.registerMsg(Puzzle.ScatterPuzzle.class, ScatterPuzzleHandler.class);
    }
    ...省略...  
}
```

これで、クライアント側の送信機能とサーバー側のレスポンス機能の両方が完成しました。
<br>

### パズルのシャッフル機能確認

Unityエディタでプレイモードに入ります。Scatter Puzzleボタンをクリックして、パズルのシャッフル機能がうまく動作することを確認します。

<br>

## より良い途中参加ユーザーの処理

先ほどは、途中参加ユーザーを処理する過程で同期の問題が発生しました。その原因は、ユーザーがルームに入るタイミングをパズルのピースの位置同期のタイミングとして考えていたためです。しかし、私たちが実装しているパズルゲームでは、ユーザーがルームに入るタイミングでシーンの移動が発生します。

そのため、リスナー登録とonJoinRoomコールバックの呼び出しの2つのタイミングに分けて考える必要があります。これは、ゲームクライアントの実装によって問題がない場合もあれば、問題がある場合もあります。 onJoinRoomコールバックの呼び出し後にシーンの移動が開始されるため、シーンの移動が完了した直後にクライアントが直接サーバーにパズルの位置同期をリクエストしたいと考えています。

<br>

### プロトコルの登録

サーバープロジェクトに移動し、Puzzle.protoにパズルの位置同期をリクエストするためのプロトコルを追加します。

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}

message ScatterPuzzle {}

message PuzzlePositionReq {}
```

プロトコルを再コンパイルした後、作成されたC#クラスをUnityプロジェクトに移動します。

<br>

### クライアント側の実装

シーン移動直後にサーバーにパズルの位置をリクエストするためには、シーン移動直後に実行されるStart関数を修正する必要があります。GameManagerのStartメソッドに次のように新しく作成したPuzzlePositionReqプロトコルメッセージを送信するコードを追加します。

```c#
public class GameManager : MonoBehaviour
{
    void Start(){
        ...省略...

        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).Send(new PuzzlePositionReq());
    }
  
  	...省略...
}
```

<br>

### サーバー側の実装

onJoinRoomで誤って実装していたパズル位置送信のコードを、正しい位置に移動してPuzzlePositionReqHandlerに新たに作成します。

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import protocol.Puzzle;

public class PuzzlePositionReqHandler implements RoomMessageHandler<BasicRoom, BasicUser, Puzzle.PuzzlePositionReq> {
    @Override
    public void execute(BasicRoom room, BasicUser user, Puzzle.PuzzlePositionReq req) throws SuspendExecution {
        room.puzzlePositions.values().stream().forEach(room::broadcast);
    }
}
```

そして、作成したハンドラをBasicRoomに登録します。

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    static {
        dispatcher.registerMsg(BasicProtocol.MessageRequest.class, BasicHandler.class); // 処理するメッセージのハンドラを登録
        dispatcher.registerMsg(Puzzle.PuzzlePosition.class, PuzzlePositionHandler.class);
        dispatcher.registerMsg(Puzzle.ScatterPuzzle.class, ScatterPuzzleHandler.class);
        dispatcher.registerMsg(Puzzle.PuzzlePositionReq.class, PuzzlePositionReqHandler.class);
    }
  
    ...省略...
  
}
```

<br>

### 途中参加ユーザー処理確認

Unityで`cmd+b`または`ctrl+b`でビルドしてからプレイします。ビルドされたゲームでルームを作成し、パズルのシャッフルを実行します。その状態でUnityエディタのプレイモードに入り、このルームに参加した後、パズルの位置が同期されることを確認します。

<br>

## ユーザーマッチメイキングの実装

### サーバー側の実装

ユーザーマッチメイキングは、ユーザーのマッチメイキングリクエストを集め、適切な基準に合わせて、似たようなレベルのユーザー同士が同じルームでゲームを開始できるようにします。勝ち点やスコアなど様々な要素をユーザーが直接実装して、ユーザーを適切に区別してマッチングできます。ここでは、2人のユーザーを1つのゲームにマッチングするロジックを実装します。

UserMatchInfoクラスを作成します。ファイル名はBasicUserMatchInfoとします。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_user_match_info.png)

このクラスにはマッチングに使用するユーザーの情報を入れます。マッチメイキングに使う要素があれば、ここに追加します。今回の例題では特別な要素は追加せず、必ず実装するメソッドだけ作成して使います。 1つ注意する点は、getId()メソッドが必ずリクエストしたユーザーのIDを返すように実装するようにすることです。そして、パーティーマッチメイキング機能は使わないので、0を返すように設定します。

```java
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.match.BaseUserMatchInfo;

import java.io.Serializable;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicUserMatchInfo extends BaseUserMatchInfo implements Serializable {

    private int id;

    public BasicUserMatchInfo(int id) {
        this.id = id;
    }

    @Override
    public int getId() {
        return id;
    }

    @Override
    public int getPartySize() {
        return 0;
    }
}
```

このようなUserMatchInfoはクライアントがユーザーマッチメイキングをリクエストする時、サーバーのゲームユーザーでonMatchUserコールバックを実装する過程で作成して使います。GameAnvilは基本的なユーザーマッチメーカーを提供します。下記のonMatchUserは、このようなエンジンの基本的なユーザーマッチメイキングをmatchUser APIを通じて使用しています。

```java

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.exceptions.NodeNotFoundException;
import match.BasicUserMatchInfo;
import org.slf4j.Logger;

import java.util.concurrent.TimeoutException;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@UserType("BASIC_USER")
public class BasicUser extends BaseUser {
    private static final Logger logger = getLogger(BasicUser.class);

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, Payload payload, Payload outPayload) throws SuspendExecution {
   try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(getUserId());
            return matchUser(matchingGroup, roomType, term); // ユーザーマッチメイキングリクエスト
        } catch (Exception e) {
            logger.error("BasicUser::onMatchUser()", e);
        }
        return false;
    }
}

```

ユーザーマッチメイキングを使うための基本的な準備ができたら、実際にマッチメイキングをするマッチメーカーを作成します。下記のようにBasicUserMatchMakerクラスを追加します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_user_match_maker.png)

コンストラクタでは親クラスのコンストラクタを呼び出しながら引数でマッチ人数とマッチ申請の有効時間を渡します。有効時間が過ぎると、そのマッチリクエストは自動的にキャンセルされます。そして、実際のマッチメイキングを行うmatchメソッドは、エンジンによって1秒に1回ずつ呼び出されます。

getMatchRequestsは、マッチングのための最少人数を引数として受け取り、現在のマッチングプール全体を照会し、最適なマッチングを可能な限り作成します。つまり、マッチングリクエストがたくさん溜まっている場合、getMatchRequestsによって一度に100個または1,000個のマッチングが作成されることもあります。この時、マッチングが成功すると、マッチングされたユーザーのUserMatchInfoリストを返します。このリストをエンジンが提供するmatchSingles APIに渡すと、そのリストの中のユーザーに対するルームの作成や移動が自動的に行われます。

もし、マッチングに十分なリクエストが溜まっていなかったり、条件に合う相手がいない場合はnullを返します。この場合は、次の1秒後のmatch呼び出しで再度同じマッチング検索が行われます。

```Java
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.match.BaseUserMatchMaker;
import org.slf4j.Logger;

import java.util.List;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicUserMatchMaker extends BaseUserMatchMaker<BasicUserMatchInfo> {
    private static final Logger logger = getLogger(BasicUserMatchMaker.class);
    private static final int NUM_USER = 2;
    private static final int TIMEOUT = 10000;

    public BasicUserMatchMaker() {
        super(NUM_USER, TIMEOUT);
    }

    @Override
    public void onMatch() {
        List<BasicUserMatchInfo> matchRequests = getMatchRequests(2);

        if (matchRequests == null) {
            return;
        }

        matchSingles(matchRequests);
    }

    @Override
    public boolean onRefill(BasicUserMatchInfo roomInfo) {
        return false;
    }
}
```

<br>

### クライアント側の実装

マッチメイキングロジックは全てサーバーに実装されているので、クライアントはマッチメイキングが必要なタイミングでリクエストを送るだけです。ConnectHandlerにUserMatchMakingメソッドを追加します。そして、マッチメイキングが終わった時点でシーンを移動させるハンドラを追加します。

```csharp
public class ConnectHandler : MonoBehaviour
{
	...省略...
	
    void Start () {
  
        ...省略...
     
            connector.GetUserAgent("BASIC_SERVICE", 1).onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone resultCode, bool created, int roomId, Payload payload) =>{
            this.roomId = roomId;
            SceneManager.LoadScene("GameScene");
        };
    }
	
	...省略...

    public void UserMatchMaking(){
            connector.GetUserAgent("BASIC_SERVICE", 1).MatchUserStart("BASIC_ROOM", "BASIC_MATCHING_GROUP",(UserAgent userAgent, ResultCodeMatchUserStart resultCode, Payload payload) =>{
            Debug.Log(resultCode);
        });
    }
}

```

シーンでUser Match MakingボタンのOnClickリスナーにConnectHandlerコンポーネントをドラッグして登録し、ドロップダウンからUserMatchMakingメソッドを選択します。

<br>

### ユーザーマッチメイキングのテスト

Unityで`cmd+b`または`ctrl+b`でビルド後にプレイします。その状態でUnityエディタでプレイモードに入ります。両方でUser Match Makingボタンを押して、マッチングが成立し、同じルーム番号で結ばれることを確認します。

<br>

## ルームマッチメイキングの実装

ルームマッチメイキングは、マッチメーカーが管理しているルームの中から、ユーザーのリクエストに最も適したルームに自動的に入場させることができる機能です。つまり、ユーザーマッチメイキングがユーザーとユーザーをマッチングさせる機能であるのに対し、ルームマッチメイキングはユーザーとルームをマッチングさせる機能です。このとき、実装方法によって様々な条件でユーザーをルームにマッチングさせることができます。ここでは、まだ定員に達していないルームの中で、最も人数が少ないルームに入場するマッチメイキングを実装します。

まず、マッチメイキングを実際に行うクラスが必要です。そして、ルームマッチメーカーでルームを管理するための情報を格納するインスタンスがルームごとに1つずつ必要です。最後に、ユーザーがマッチメイキングを申請するたびに、ユーザーの要求事項を含む申請書インスタンスが必要です。このように3つの新しいクラスを作成します。

さらに、既存のロジックを一部修正します。ルームマッチメイキングは全てのルームではなく、ルームマッチメイキング対象として申請したルームだけを対象として行われます。そのため、ルーム作成時にルームマッチメイキングを対象として申請するコードを追加します。
<br>

### サーバー側の実装

まず、マッチメイキングのリクエストを表すクラスを実装します。BasicRoomMatchFormクラスを作成します。 

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_maker_form.png)

ユーザーがマッチメイキングをリクエストするたびにBasicRoomMatchFormオブジェクトが作成されて使用されます。コンストラクタから親コンストラクタに渡す「BASIC_MATCHING_USER_CATEGORY」はこのドキュメントでは気にする必要はありません。

```java
import com.nhn.gameanvil.node.match.BaseRoomMatchForm;
import java.io.Serializable;

public class BasicRoomMatchForm extends BaseRoomMatchForm implements Serializable {
    public BasicRoomMatchForm() {
        super("BASIC_MATCHING_USER_CATEGORY");
    }
}
```



次はマッチング対象のルームの情報を表現するクラスを実装します。下記のようにBasicRoomMatchInfoクラスを作成します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_info.png)

この時、ルームの最大定員はstaticフィールドで指定された4人です。この最大定員とユーザータイプを継承したBaseRoomMatchInfoのコンストラクタに引数として必ず渡さなければなりません。

```java

import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.match.BaseRoomMatchInfo;

import java.io.Serializable;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoomMatchInfo extends BaseRoomMatchInfo implements Serializable {
    private static final int MAX_USER = 4;

    public BasicRoomMatchInfo(int roomId) {
        super(roomId, "BASIC_USER", MAX_USER);
    }
}
```

次に、実際にルームマッチメイキングを処理するルームマッチメーカーを作成します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_maker.png)

次のようにBasicRoomMatchMakerを作成します。

```Java
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.data.RoomMatchResultCode;
import com.nhn.gameanvil.node.match.BaseRoomMatchMaker;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoomMatchMaker extends BaseRoomMatchMaker<BasicRoomMatchForm, BasicRoomMatchInfo> {
    @Override
    public RoomMatchResultCode onPreMatch(BasicRoomMatchForm basicRoomMatchForm, Object... objects) {
        return RoomMatchResultCode.SUCCESS;
    }

    @Override
    public boolean canMatch(BasicRoomMatchForm basicRoomMatchForm, BasicRoomMatchInfo basicRoomMatchInfo, Object... objects) {
        return true;
    }

    @Override
    public void onPostMatch(BasicRoomMatchForm basicRoomMatchForm, BasicRoomMatchInfo basicRoomMatchInfo, Object... objects) {

    }

    @Override
    public int compare(BasicRoomMatchInfo o1, BasicRoomMatchInfo o2) {
        int o1UserCount = getUserCount(o1.getRoomId());
        int o2UserCount = getUserCount(o2.getRoomId());
        if (o1UserCount > o2UserCount) {
            return -1;
        } else if (o1UserCount < o2UserCount) {
            return 1;
        } else {
            return 0;
        }
    }

    @Override
    public void onIncreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {

    }

    @Override
    public void onDecreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {

    }
}
```

compareメソッドはマッチングプールに入っているルームをソートする条件を実装します。例題は人数によってルームをソートするように実装しました。

これでルームマッチメイキングの準備がほぼ終わりました。クライアントがルームマッチリクエストを送信すると、サーバーのBasicUserはonMatchRoomコールバックを呼び出します。このコールバックで先ほど説明したBasicRoomMatchFormオブジェクトを作成した後、matchRoom APIに引数として渡して呼び出します。つまり、クライアントが送ったルームマッチングリクエストをここでマッチメーカーに渡しました。

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.exceptions.NodeNotFoundException;
import match.BasicUserMatchInfo;
import org.slf4j.Logger;

import java.util.concurrent.TimeoutException;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@UserType("BASIC_USER")
public class BasicUser extends BaseUser {

	@Override
	public final RoomMatchResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final Payload payload) throws SuspendExecution {
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();

        try {
            return matchRoom(matchingGroup, roomType, gameRoomMatchForm); // マッチメーカーにリクエスト伝達
        } catch (Exception e) {
            logger.error("BasicUser::onMatchRoom()", e);
        }
        return RoomMatchResult.FAILED;
    }
}
```

ルームが作成された時点でルームマッチメイキング対象にするため、BasicRoomのonCreateRoomコールバックで下記のようにregisterRoomMatch APIを呼び出します。

```java
public class BasicRoom extends BaseRoom<BasicUser> {
    ...省略...

    @Override
    public boolean onCreateRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(user.getUserId(), user);

        BasicRoomMatchInfo gameRoomMatchInfo = new BasicRoomMatchInfo(getId());
        try {
            registerRoomMatch(gameRoomMatchInfo, "BASIC_MATCHING_USER_CATEGORY", user.getUserId()); // ルームをルームマッチメイキング対象として登録
        } catch (Exception e) {
            logger.error("BasicRoom::onCreateRoom():registerRoomMatch", e);
        }

        logger.debug("onCreateRoom(userId:{})", user);
        return true;
    }
  
}
```

また、ルーム情報が変動するたびに、ルームマッチメイキング情報も更新する必要があります。ちなみに、ルームの人数変動はルームマッチメーカーが自動的に同期するので、人数を除いた追加情報の更新を行うだけです。

以下は、onJoinRoomコールバックを修正して、ルームにユーザーが参加した時にマッチメイキング情報を更新するコードです。

```java
public class BasicRoom extends BaseRoom<BasicUser> {
    ...省略...

    @Override
    public boolean onJoinRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(user.getUserId(), user);

        BasicRoomMatchInfo gameRoomMatchInfo = new BasicRoomMatchInfo(getId());
        try {
            updateRoomMatch(gameRoomMatchInfo);
        } catch (Exception e) {
            logger.error("BasicRoom::onCreateRoom():registerRoomMatch", e);
        }

        logger.debug("onJoinRoom(userId:{})", user);
        return true;
    }
}
```

<br>

### クライアントの実装

ユーザーマッチメイキングと同じように、クライアントはマッチメイキングが必要なタイミングでリクエストを送るだけです。ConnectHandlerにRoomMatchMakingメソッドを追加します。

```csharp
public class ConnectHandler : MonoBehaviour {
    public void RoomMatchMaking(){
            connector.GetUserAgent("BASIC_SERVICE", 1).MatchRoom("BASIC_ROOM", "BASIC_MATCHING_GROUP", "BASIC_MATCHING_USER_CATEGORY", true, 
            (UserAgent userAgent, ResultCodeMatchRoom resultCode, int integer, int roomId, string roomName, bool created, Payload payload) =>{
            Debug.Log(resultCode);
            if (resultCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS){
            	this.roomId = roomId;

            	SceneManager.LoadScene("GameScene");
            }
        });
    }
}

```

シーンでRoom Match MakingボタンのOnClickリスナーにConnectHandlerコンポーネントをドラッグして登録し、ドロップダウンメニューからRoomMatchMakingメソッドを選択します。

マッチンググループがあるルームを作るため、ルーム作成コードを修正します。この時、マッチンググループは、マッチメイキング対象のルームを論理的に分けるために、サーバーとクライアントの間で事前に定義した任意の文字列です。ここではBASIC_MATCHING_GROUPという文字列を使います。

```c#
public class ConnectHandler : MonoBehaviour {
	public void CreateRoom(){
            connector.GetUserAgent("BASIC_SERVICE", 1).CreateRoom("", "BASIC_ROOM", "BASIC_MATCHING_GROUP", null (UserAgent ua, ResultCodeCreateRoom resultCode, int roomId, string roomName, Payload payload) => {
            Debug.Log(resultCode); 
            if (resultCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
                this.roomId = roomId;
              
                SceneManager.LoadScene("GameScene");
            }
        });
    }
}
```

<br>

### ルームマッチメイキングテスト

Unityで`cmd+b`または`ctrl+b`でビルド後、プレイ状態でルームを作成します。その状態でUnityエディタでプレイモードに入ります。プレイモードでRoom Match Makingボタンを押して、ビルドモードで作成したルームに移動することを確認します。

<br>

## Room退室の実装

最後にルームを退室する機能を実装するためUnityクライアントのGameManagerに下記のメソッドを追加します。

```c#
public void LeaveRoom() {
		ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).LeaveRoom((userAgent, resultCode, force, roomId, payload) =>{
      	if(resultCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS){
      		Destroy(connectHandler.gameObject);
    			SceneManager.LoadScene("ConnectScene");
    		}
    });
}
```

シーンでLeave RoomボタンのOnClickリスナーにGameManagerコンポーネントをドラッグして登録し、ドロップダウンメニューからLeaveRoomメソッドを選択します。

## プロジェクトの終わりに

以上、GameAnvilとUnityを使ってリアルタイムマルチプレイが可能なパズルゲームを実装してみました。その過程でGameAnvilのコア機能の多くを使用しました。しかし、GameAnvilはこのチュートリアルに含まれていないもっと多様な機能をサポートしています。これらの機能については、次のドキュメントを参照してください。また、一緒に提供されるリファレンスサンプルプロジェクトとJavaDocもGameAnvilを理解するのに役立つと思います。
