## Game > GameAnvil > 応用チュートリアル

### GameAnvilでゲームサーバーを簡単に作成する

GameAnvilは、リアルタイムマルチプレイヤーゲームサーバー制作プラットフォームです。
GameAnvilはサーバーエンジンだけでなく、サーバーにクライアントを接続するためのコネクタを提供します。
GameAnvilを使用すると、手軽にゲームサーバーとクライアントを開発し、運用できます。

このドキュメントは、GameAnvilの基本的な機能に加え、ユーザーが定義したプロトコルを利用して、インゲームチャット及び実際にプレイ可能なマルチプレイヤーパズルゲームを開発する過程を扱います。

サーバーとクライアントが相互作用する様子を確認できるサンプルを完成させながら、GameAnvilを使用してゲームを開発する全体的な流れに慣れることができます。

提供されたテンプレートを活用して簡単にサーバーを構築し、GameAnvilコネクタが提供する機能を通じてUnityクライアントでチャット、マルチプレイヤー同期、マッチメイキングなどを実装できます。

このドキュメントは、サーバーの概念とAPIを単に羅列する代わりに、より具体的な例を通じて説明し、理解を助けるために、実際にプレイ可能なマルチプレイヤージグソーパズルゲームを開発する過程を順序立てて説明します。ドキュメントの内容を直接1つずつ実践しながら、自然とGameAnvilとマルチプレイヤーゲーム開発に対する理解度を高めることができます。

## プロジェクト構成

マルチプレイヤーゲームを作成するには、クライアントと対応するサーバープログラムが必要です。この例では、クライアントプログラムの制作にUnityとGameAnvilコネクタを使用し、サーバープログラムの制作に先ほど紹介したサーバーエンジンGameAnvilを使用します。まずGameAnvilを利用したサーバープログラムプロジェクトを作成した後、UnityとGameAnvilコネクタを利用してクライアントプログラムプロジェクトを作成します。

以下の段階を進めると作成される最終的なサーバーサンプルプロジェクトは、以下のリンクからダウンロードできます。初期テンプレートからいくつかの段階を経てサーバー機能を実装するとどのような構造になるかあらかじめ確認したい場合は、該当プロジェクトをダウンロードして参考にできます。

[サーバーサンプルプロジェクトのダウンロード](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil_Tutorial_Advanced_Server.zip?disposition=attachment)

### GameAnvilプロジェクト構成

プロジェクトにGameAnvilを適用するには、MavenリポジトリからGameAnvilライブラリをダウンロードし、GameAnvilを駆動するのに必須の設定ファイルを作成する必要があります。最後に若干のボイラープレートコードを作成すれば、開発初期設定が完了します。今回のチャプターでは、開発を開始するための初期設定を完了することを目標とします。実際のプロセスを実行してサーバーを起動することは、次のチャプターで扱います。

GameAnvilでは、このような一連の過程を代行するIntelliJテンプレートを提供しており、より簡単に初期作業を完了できます。次のリンクからIntelliJ用プロジェクトファイルテンプレートをダウンロードできます。ダウンロードしたテンプレートは解凍しないようにします。

[テンプレートのダウンロード](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil%20Template.zip?disposition=attachment)

ダウンロードしたテンプレートを適用するためにIntelliJを実行します。**Welcome to IntelliJ IDEA**画面の左側メニューで**Customize**を選択した後、**Import Settings...**をクリックします。または全体検索ウィンドウで**Import Settings...**を検索します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/1_import_gameanvil_template.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/2_search_import_settings.png)

<br>

Finderまたはファイルエクスプローラーでテンプレートをダウンロードしたパスへ移動し、圧縮ファイルを選択します。**Select Components to Import**ウィンドウが開いたら、`File templates`項目と`Project Templates`項目の両方をチェックして選択します。**OK**をクリックした後、インポートが完了したらIntelliJを再起動してテンプレートの適用を完了します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/3_select_import.png)

IntelliJ右上のボタングループで**New Project**をクリックした後、左側のリストをスクロールして下段の**Templates**にある`GameAnvil 2.1.0 Template`を選択します。プロジェクト名を設定します。名前にスペースを含めることはできません。プロジェクトの場所とベースパッケージ名を確認した後、プロジェクトを作成します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/4_imported_gameanvil_template.png)

これでIntelliJにサーバープロジェクトの骨格が構成されました。Projectパネルを見ると、コードと設定ファイルが生成されたことを確認できます。

- Main: プログラムのエントリーポイントであるMain関数を含むクラスです。
- protocolパッケージ: javaでコンパイルされたプロトコルバッファファイルを含むパッケージです。
- BasicProtocol.proto, Puzzle.proto: プロトコルバッファを利用して作成されたプロトコルファイルです。
- GameAnvilConfig.json: GameAnvilの駆動に必要なサーバー設定情報を記録したファイルです。サーバー実装に合わせて修正できます。
- logback.xml: Javaプロジェクトでロギングを構成するために使用されるファイルです。Logbackフレームワークの設定ファイルとして、ロギングシステムの動作方式とログの形式、保存場所などを指定します。このファイルを使用して、ロギングレベル、ログ形式、ログファイルのパス及び名前、ログローリングポリシーなどを設定できます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/5_gameanvil_project_view_init.png)

まずJDK設定を確認します。GameAnvilはJava 21バージョンをサポートします。バージョンによって一部の設定方法が異なる場合があり、ここではJava 21バージョンを使用しました。

左上のメニューから **File > Project Structure** を選択します。Macユーザーの場合、`Command + ;` ショートカットキーを使用できます。

ProjectタブでSDK設定を確認します。もし設定されたSDKがない場合、`Add SDK > Download JDK`を通じて希望するバージョンのJDKをダウンロードして設定します。Language levelはSDK defaultに設定します。次にModulesタブでLanguage levelをProject defaultに設定します。

**Project**タブでSDK設定を確認します。もしSDKが設定されていない場合は、**Add SDK > Download JDK**をクリックし、使用するバージョンのJDKをダウンロードして設定します。Language levelは `SDK default(Java 21)` に設定します。次に **Modules** タブでLanguage levelを `Project default (Java 21)` に設定します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/6_project_structure.png)

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/7_module_language_level.png)

**設定**メニューで**gradle**設定を確認します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/8_gradle_config.png)

プロジェクトの準備はほぼ完了しましたが、実行するにはいくつかの設定が必要です。ここではまずクライアントプロジェクトを作成し、サーバー設定を完了してから実行します。

### Unityプロジェクト構成

Unity Hubを実行します。右上の**NEW**をクリックして新しいプロジェクト作成ウィンドウを開きます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/9_unity_hub.png)

テンプレートリストから**2D**を選択し、プロジェクト名と場所を確認した後、**CREATE**をクリックしてプロジェクト作成を完了します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/10_new_unity_project.png)

次のリンクからGameAnvilコネクタをダウンロードしてください。コネクタはGameAnvilサーバーとの通信に必要なクライアントAPIを提供し、簡単なコードだけでクライアントを実装できるように支援するパッケージです。

[gameanvil-connector.unitypackage](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector.unitypackage)

実習に必要なクライアントプロジェクト作成のために、以下のリンクからチュートリアル用コードと画像ソースなどが含まれたUnityパッケージをダウンロードします。

[gameanvil_tutorial_advanced.unitypackage](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil_tutorial_advanced.unitypackage)

ダウンロードしたパッケージファイルをUnityプロジェクトへドラッグしてインポートします。または **Asset > Import Package > Custom Package...** メニューを開き、Finderまたはファイルエクスプローラーからパッケージファイルを選択します。Import Unity Packageダイアログでリストの全てのチェックボックスを選択した後、**Import**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/11_import_unity_package.png)

最後に、便利にテストするためにはProject SettingsウィンドウでPlayerタブを選択し、Resolution and Presentation項目で以下のようにプロジェクトの基本ウィンドウの幅と高さをそれぞれ1920、1080に設定します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/12_unity_project_setting_resolution.png)

クライアントプロジェクトの設定が完了しました。

## サーバー起動及び接続

### GameAnvilサーバー起動

実行設定が完了したら、右側のgradleメニューからTasks > other > `runMain`をダブルクリックして実行します。一度このように実行すると、その後はIntelliJの右上にある緑色の三角形の実行アイコンをクリックしてもサーバーが起動します。

`runMain`で実行してこそ、GameAnvilサーバー実行に必要なVMオプションが適用されます。もしMainクラスのmain()関数をそのまま実行する場合は、**Edit Configurations...**で以下の必須VMオプションを追加する必要があります。

```
"--add-opens", "java.base/java.lang=ALL-UNNAMED",
"--add-opens", "java.base/java.lang.invoke=ALL-UNNAMED" 
"-XX:+UseG1GC"
```

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/13_gameanvil_run.png)

サーバーが正常に起動すると、サーバー起動状態に関するログが多数出力されます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/14_gameanvil_run_log.png)

GameAnvilサーバーは、複数のノードで構成されています。これらのノードは、サーバーが実行する機能を複数の役割に分担します。まだサーバーの初期起動を確認しただけで、ノードや他のサーバー駆動のためのコード作成を行っていないため、完全に準備された状態ではありません。

それぞれのノードはコードを実行するための準備に時間を要し、各ノードの準備が完了するとonReadyログを出力します。クライアントがサーバーに接続するうえで直接的な役割を果たすノードは、ゲートウェイノードです。ゲートウェイノードが準備され、GatewayNodeのonReadyログが出力されれば、GameAnvilサーバーはいつでも接続可能な状態になったことになります。

### コネクトハンドラ作成

Unityプロジェクトへ移動し、GameAnvilサーバーに接続できるようにコードを作成します。サーバーと接続するには、まずコネクタオブジェクトを生成する必要があります。

AssetパネルでSceneフォルダ内のConnectSceneをダブルクリックしてシーンを準備します。Hierarchyビュー上のConnectHandlerゲームオブジェクトにコンポーネントとして追加されているConnectHandlerスクリプトをソースコードエディタで開き、実装内容を確認しながら各過程を直接実習します。

GameAnvilサーバーとクライアントは、定期的に状態チェックパケットをやり取りする必要があります。クライアントが一時停止した状況などで状態チェックパケットのやり取りを停止するには、PauseClientStateCheck()メソッドを利用します。状態チェックを再開するには、ResumeClientStateCheck()メソッドを呼び出します。OnApplicationPause()関数でクライアントの一時停止を検知し、適切なメソッドを呼び出します。

```c#
using System;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using Protocol;
using UnityEngine.SceneManagement;
using Random = UnityEngine.Random;

public class ConnectHandler : MonoBehaviour
{
    private static ConnectHandler instance;

    [Header("Object Reference")]
    public Text serverInfoText;
    public InputField serverInfoInput;
    public Text clientInfoText;
    public Text logText;

    public string ip = "127.0.0.1";
    public int port = 18200;

    public string deviceId = "";
    public string password = "";
    public string accountId = "";

    public int roomId;

    public GameObject popupCanvas;
    public InputField roomIdInput;

    private static GameAnvilConnector connector;
    private static GameAnvilUser user;
    
    void Start()
    {
        // 接続情報を画面に出力します。
        serverInfoText.text = "サーバー情報:";
        serverInfoInput.text = ip;
    }

    private void Awake()
    {
        DontDestroyOnLoad(this.gameObject);
    }

    private void OnApplicationPause(bool pause)
    {
        if(!getConnector().IsConnected)
            return;

        if (pause)
        {
            // 入力した時間の間、サーバーのclientStateCheck機能を停止させる。
            // この時間が過ぎるとclientStateCheck機能が動作し、接続が切れる可能性がある。
            getConnector().PauseClientStateCheck(10 * 60);
        }
        else
        {
            // サーバーのclientStateCheck機能を再度動作させる。
            getConnector().ResumeClientStateCheck();
        }
    }

    public void Quit()
    {
        if (getConnector().IsConnected)
        {
            getConnector().Disconnect();
        }
#if UNITY_EDITOR
        UnityEditor.EditorApplication.isPlaying = false;
#else
        Application.Quit();
#endif
    }
}
```

### ConnectorとUserの生成

コネクタで様々な機能を利用するために、ConnectorとUserを生成する必要があります。Connectorは主にサーバー接続、認証などの機能を提供し、Userはログイン、ルーム生成及び入室など、ユーザーに関連する機能を提供します。

getConnector()関数で新しいGameAnvilConnectorオブジェクトを生成してconnectorフィールドに保存します。そしてgetUser()関数で新しいGameAnvilUserオブジェクトを生成してuserフィールドに保存します。user生成時に使用するサービス名はサーバープロジェクトでも使用するため、覚えておきます。

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();
    }

    return connector;
}

public GameAnvilUser getUser()
{
    if (user == null)
    {
        user = new GameAnvilUser(getConnector(), "BASIC_SERVICE", 1);
    }

    return user;
}
```

<br>

### サーバー接続

コネクタが提供するAPIを使用してサーバーに接続するConnect()メソッドは次のとおりです。

```c#
// Client-side

public async void Connect()
{
    ip = serverInfoInput.text;

    getConnector().OnDisconnect += (resultCode, payload) =>
    {
        Debug.Log("Disconnected");
    };

    try
    {
        await getConnector().Connect(ip, port);
        Debug.Log("CONNECT_SUCCESS");
        logText.text = "CONNECT_SUCCESS";
    }
    catch (Exception e)
    {
        Debug.Log("CONNECT_FAIL");
        logText.text = "CONNECT_FAIL";
        throw;
    }
}
```

<br>

Connect()関数ではGameAnvilConnectorのConnectメソッドを呼び出しています。これを通じて、引数として与えられたipとport情報でサーバー接続を試みます。

これでサーバーが接続を受け入れる準備ができたように、クライアントもサーバーに接続する準備が完了しました。

### サーバー接続確認

これでUnityクライアントでプレイモードに入り、コンソール上に結果コードが正しく出力されるか確認します。ゲーム画面のテキスト上にIP、Portの接続情報と共に接続成功メッセージを確認できます。ゲームサーバーへの接続が完了したクライアントは、サーバーを通じてメッセージをやり取りできるようになります。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/15_connect_success.png)

## Room及びUser生成

サーバーに接続したクライアントを**ゲームユーザー**と呼びます。サーバーに接続したクライアントは、サーバー上で1つ以上のゲームユーザー(User)としてログインできます(この例では、1つのユーザーとしてログインする場合を扱います)。ゲームユーザーは1つの**ゲームルーム**に属することで、同一ルームに属する他のユーザーと通信できます。つまり、ユーザーが他のユーザーとゲーム関連のメッセージを交換するには、該当ユーザーは同じルーム(Room)内に属している必要があります。

GameAnvilではゲームユーザーとゲームルームの基本実装をあらかじめ用意しているため、エンジンのクラスを拡張し、コネクタのAPIを利用して簡単にゲームユーザーとルームの構造を完成させることができます。エンジン側ではゲームユーザーとルームを定義する方法を扱い、コネクタ側ではルーム生成や参加などをリクエストするAPIを使用する例を扱います。

### User

サーバーではゲームユーザーとゲームルームの機能をクラスとして定義します。まずゲームユーザーを定義してみます。

プロジェクトパネルでMainクラスが位置するパスを右クリックした後、**New > Package**を選択して **game** という名前の新しいパッケージを生成します。そして **game** パッケージを再度右クリックした後、**New > GameAnvil User**を選択します。ファイル生成ダイアログが開いたら、**File name**に **BasicUser**を、**Service name**、**User type**に **USER_TYPE_BASIC**を入力した後、**OK**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/16_create_user.png)

GameAnvilが提供するIUserインターフェースを実装してゲームユーザーを実装する基本コードが作成されたファイルが生成されます。GameAnvilで希望する機能のゲームユーザーを実装するには、IUserインターフェースを実装した後、状況に合わせて呼び出される複数のコールバック関数をオーバーライドして、希望するコードを実行するように設定すればよいです。次は、サポートするコールバック関数リストの一部です。

- onLogin: ログインリクエスト時に実行されるコールバックです。戻り値としてログインリクエストの許可可否を渡す必要があります。falseを返すと、クライアントはログインに失敗します。パラメータを通じてクライアントから受け取ったペイロードを参照でき、クライアントに返すペイロードへの参照を通じてログイン許可情報以外に追加情報をクライアントに伝達できます。
- onPostLogin: ログイン後に実行されるコールバックです。
- onReLogin: すでにログイン済みのユーザーに対して再度ログインリクエストが来る場合は、このコールバックで個別に処理できます。
- onDisconnect: クライアントのリクエストによるログアウト、またはサーバーによる強制ログアウトによって、サーバーに登録されたユーザー情報と接続が切れた時に呼び出されるコールバックです。

これ以外のコールバック関数はログイン過程に関与しないため、今すぐ全てのコールバックがどのような意味なのか完璧に知る必要はありません。

onLoginコールバックメソッドでは、ログイン過程で実行されるべき動作を実装します。この例では特別なログイン実装はなく、無条件でログインに成功するようにtrueを返すようにします。そしてuserContextのgetterを生成します。

```java
@GameAnvilUser(
        gameServiceName = "BASIC_SERVICE",
        gameType = "USER_TYPE_BASIC",
        useChannelInfo = true
)
public class BasicUser implements IUser {

    private static final Logger logger = getLogger(BasicUser.class);

    private IUserContext userContext;

    public IUserContext getUserContext() {
        return userContext;
    }

    @Override
    public void onCreate(IUserContext userContext) {
        this.userContext = userContext;
    }

    @Override
    public boolean onLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onAfterLogin(boolean isRelogined) {

    }

    @Override
    public boolean onReLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onDisconnect() {

    }

    @Override
    public void onPause() {

    }

    @Override
    public void onResume() {

    }

    @Override
    public void onLogout(IPayload payload, IPayload outPayload) {

    }

    @Override
    public boolean canLogout() {
        return true;
    }

    @Override
    public void onAfterLeaveRoom() {

    }

    @Override
    public boolean canTransfer() {
        return true;
    }

    @Override
    public boolean onLoginByOtherDevice(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean onLoginByOtherUserType(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onLoginByOtherConnection(IPayload payload) {

    }

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();
        try {
            return userContext.matchRoom(matchingGroup, roomType, gameRoomMatchForm);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicUser::onMatchRoom", e);
            return RoomMatchResult.FAILED;
        }
    }

    @Override
    public void onMatchRoomFail(MatchRoomFailCode matchRoomFailCode) {

    }

    @Override
    public void onMatchUserFail(MatchUserFailCode matchUserFailCode) {

    }

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;

        try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(userContext.getUserId());
            return userContext.matchUser(matchingGroup, roomType, term);
        } catch (TimeoutException | NodeNotFoundException | GameAnvilException e) {
            logger.error("BasicUser::onMatchUser", e);
        }

        return isSuccess;
    }

    @Override
    public void onMatchUserCancel(MatchCancelReason matchCancelReason) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    @Override
    public void onAfterTransferIn() {

    }

    @Override
    public void onSnapshot(IPayload payload, IPayload outPayload) {

    }

    @Override
    public boolean canMoveOutChannel(String channelId, IPayload payload, IPayload outPayload) {
        return false;
    }

    @Override
    public void onMoveOutChannel(String channelId, IPayload payload) {

    }

    @Override
    public void onAfterMoveOutChannel() {

    }

    @Override
    public void onMoveInChannel(String channelId, IPayload payload, IPayload outPayload) {

    }
}
```

<br>

### Room

ログイン可能なユーザー実装が完了しました。次はゲームルームを実装します。ユーザー生成方法と同様に**GameAnvil Room**ファイルテンプレートを利用してIRoomインターフェースを実装したクラスを生成します。**File name**には**BasicRoom**を、**Service name**には**BASIC_SERVICE**を、**Room type**には**ROOM_TYPE_BASIC**を、**User**には前の段階で生成したIUser実装クラスのクラス名**BasicUser**を入力します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/17_create_room.png)

**OK**をクリックすると、サポートされるコールバックメソッドが自動的に作成されます。GameAnvilでサポートするコールバックを説明するために、少しクライアントのAPIを簡単に説明します。クライアントではコネクタでログイン後に他のユーザーと通信するために、ルーム関連APIを呼び出すことができます。ルームを作成したり、他のユーザーが作成したルームに参加したり、ルームから退出するなどの動作をサポートします。

- onCreateRoom: ユーザーがルーム生成をリクエストした時に実行されます。ルーム生成をリクエストしたユーザー情報と共に、ルーム生成時にクライアントから渡された追加情報をパラメータとして提供するため、この情報をもとにルーム生成を許可するかどうかを決定して返すよう実装します。
- onJoinRoom: 他のユーザーが生成したルームへの入室をリクエストした時に実行されます。他のコールバックと同様に、ルーム入室をリクエストしたユーザー情報と共に、ルーム入室時にクライアントから渡された追加情報を提供するため、この情報をもとにルーム入室を許可するかどうかを決定して返すよう実装します。
- onLeaveRoom: ユーザーがルームからの退出をリクエストした時に実行されます。他のコールバックと同様に、ルーム退出をリクエストしたユーザー情報と共に、ルーム退出リクエスト時に一緒に渡された追加情報を提供するため、この情報をもとにルーム退出を許可するかどうかを決定して返すよう実装します。

ルームに属しているユーザーに関する情報は、**roomContext.getAllUsers()**関数を通じて取得できます。これを活用して、ルームに属している全てのユーザーに同一のパケットを送信する**broadcast()**関数を追加します。また、ルームマッチング情報を保存するフィールドと、パズル位置同期のためのデータ構造を追加します。

そして、**onInit**コールバックでルームマッチング情報オブジェクトを生成し、**onCreateRoom/onJoinRoom**コールバックでそれぞれルームマッチング情報を登録及び更新するコードを追加します。これにより、該当ルームがマッチングリストに追加され、他のユーザーが該当ルームにマッチングされるようにします。

```java
package org.example.game;

import com.nhn.gameanvil.exceptions.GameAnvilException;
import com.nhn.gameanvil.exceptions.NodeNotFoundException;
import com.nhn.gameanvil.game.GameAnvilRoom;
import com.nhn.gameanvil.node.game.IRoom;
import com.nhn.gameanvil.node.game.context.IRoomContext;
import com.nhn.gameanvil.node.game.data.MatchCancelReason;
import com.nhn.gameanvil.packet.IPayload;
import com.nhn.gameanvil.packet.Packet;
import com.nhn.gameanvil.serializer.ITimerHandlerTransferPack;
import com.nhn.gameanvil.serializer.ITransferPack;
import org.example.StringValues;
import org.example.match.BasicRoomMatchInfo;
import protocol.Puzzle;
import org.slf4j.Logger;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.TimeoutException;

import static org.slf4j.LoggerFactory.getLogger;

@GameAnvilRoom(
        gameServiceName = "BASIC_SERVICE",
        gameType = "ROOM_TYPE_BASIC",
        useChannelInfo = true
)
public class BasicRoom implements IRoom<BasicUser> {

    private static final Logger logger = getLogger(BasicRoom.class);

    private IRoomContext roomContext;

    // コード追加
    private BasicRoomMatchInfo gameRoomMatchInfo;
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();

    public void broadcast(Packet packet) {
        final var allUsers = roomContext.getAllUsers();

        for (final var user : allUsers) {
            final var userContext = ((BasicUser)user).getUserContext();
            userContext.send(packet);
        }
    }

    @Override
    public void onCreate(IRoomContext<BasicUser> roomContext) {
        this.roomContext = roomContext;
    }

    @Override
    public void onInit() {
        gameRoomMatchInfo = new BasicRoomMatchInfo(roomContext.getId());
    }

    @Override
    public void onDestroy() {

    }

    @Override
    public boolean onCreateRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean canLeaveRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        return true;
    }

    @Override
    public void onLeaveRoom(BasicUser user) {

    }

    @Override
    public void onAfterLeaveRoom() {

    }

    @Override
    public void onRejoinRoom(BasicUser user, IPayload payload) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(List<BasicUser> list, ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    @Override
    public void onAfterTransferIn() {

    }

    @Override
    public void onPause() {

    }

    @Override
    public void onResume() {

    }

    @Override
    public boolean onMatchParty(String roomType, String matchingGroup, BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onMatchPartyCancel(MatchCancelReason matchCancelReason) {

    }

    @Override
    public void onForceMatchRoomUnregistered(MatchCancelReason matchCancelReason) {

    }

    @Override
    public boolean canTransfer() {
        return true;
    }
}
```

<br>

### GameNode

これでゲームユーザーとゲームルームの準備ができました。しかし、まだゲームユーザー/ゲームルームの生成と削除リクエストを処理するノードがありません。ゲームユーザーとゲームルームを管理する役割を果たすノードはGameNodeです。このノードは、一般的にゲームサーバーが行うことを期待される大部分のゲームロジック処理の役割を遂行するノードです。GameAnvilにノードを追加する方法は自然で簡単です。ゲームユーザーとゲームルームを定義したのと同様に、あらかじめ定義されたインターフェースを実装してクラスを作成した後、希望する機能を追加実装すればよいです。
**GameAnvil GameNode**テンプレートを選択後、ファイル名を**BasicGameNode**に、サービス名を**BASIC_SERVICE**に設定し、**OK**ボタンを押してゲームノードクラスを生成します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/18_create_game_node.png)

ノードが役割を遂行するためには、まずノードがループを実行する必要があります。ノードが実行される時は一連の過程を経ることになるため、若干の時間が必要です。ノードの実行可否や実行過程のどの段階にあるかを示す指標をノードの状態と呼びます。ノードの状態は通常、以下の順序に従って順次変更されながらREADY状態に到達します。

- INIT
- PREPARE
- READY

READY状態に到達したノードは、ユーザーがあらかじめ作成したロジックを実行する準備ができた状態です。各準備段階に到達した際に特定のコードを実行したい場合は、コールバックメソッドを実装し、エンジンがコールバックメソッドを呼び出した時に該当コードが実行されるように設定できます。今は特別に実行すべきコードがないため、生成されたコードをそのまま使用します。

```java
import com.nhn.gameanvil.game.GameAnvilGameNode;
import com.nhn.gameanvil.node.game.ChannelUpdateType;
import com.nhn.gameanvil.node.game.IGameNode;
import com.nhn.gameanvil.node.game.context.IGameNodeContext;
import com.nhn.gameanvil.node.game.data.IChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.IChannelUserInfo;
import com.nhn.gameanvil.packet.IPayload;

@GameAnvilGameNode(gameServiceName = "BASIC_SERVICE")
public class BasicGameNode implements IGameNode {
    private IGameNodeContext gameNodeContext;

    public IGameNodeContext getContext() {
        return gameNodeContext;
    }

    @Override
    public void onCreate(IGameNodeContext gameNodeContext) {
        this.gameNodeContext = gameNodeContext;
    }

    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {

    }

    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {

    }

    @Override
    public void onChannelInfo(IPayload payload) {

    }

    @Override
    public void onInit() {

    }

    @Override
    public void onPrepare() {

    }

    @Override
    public void onReady() {

    }

    @Override
    public void onPause(IPayload payload) {

    }

    @Override
    public void onShuttingdown() {

    }

    @Override
    public void onResume(IPayload payload) {

    }
}
```

作成したゲームユーザーとゲームルームをGameNodeに連動させるため、設定ファイルにもこれを登録します。GameAnvilConfig.jsonファイルの'game'項目に以下の内容を追加します。

```json
// ゲームロビーの役割を果たすノード。(ゲームルーム、ユーザーを含んでいる)
  "game": [
    {
      "nodeCnt": 1,
      "serviceId": 1,
      "serviceName": "BASIC_SERVICE",
      "channelIDs": [["ch1"]], // ノードごとに付与するチャンネルID。(一意でなくても良い。""はチャンネルを使用しないことを意味)
      "userTimeout": 5000 // クライアントの接続切れ後、ユーザーオブジェクトをサーバーから削除せずにどの程度の間管理するかを設定
    }
  ],
```

<br>

### ゲームノード、ユーザー、ルーム設定

 マウスの右ボタンでクリックした後、**New > Java Class**を選択して直接クラスを生成することもできます。 

```java
@GameAnvilGameNode(gameServiceName = StringValues.serviceName)
public class BasicGameNode implements IGameNode {
    // ...
}

@GameAnvilRoom(
    gameServiceName = StringValues.serviceName,
    gameType = StringValues.roomType,
    useChannelInfo = false
)
public class BasicRoom implements IRoom<BasicUser> {
    // ...
}

@GameAnvilUser(
    gameServiceName = StringValues.serviceName,
    gameType = StringValues.userType,
    useChannelInfo = false)
public class BasicUser implements IUser {
    // ...
}
```

そしてエンジンが提供する@GameAnvilGameNodeアノテーションを通じてゲームノード、@GameAnvilUserアノテーションを通じてユーザー、@GameAnvilRoomアノテーションを通じてルーム関連の設定が自動的に登録されます。 

ユーザータイプは各ユーザー実装を区別するサーバーとクライアント間で約束された文字列であり、ルームタイプは各ルーム実装を区別するサーバーとクライアント間で約束された文字列です。

今後クライアントプロジェクトの実装時に該当タイプを使用する必要があるため、覚えておきます。例で使用したユーザーとルームタイプは次のとおりです。

```java
public class StringValues {
    public static final String serviceName = "BASIC_SERVICE";
    public static final String userType = "USER_TYPE_BASIC";
    public static final String roomType = "ROOM_TYPE_BASIC";
}
```

例で使用するBasicGameNode、BasicUser、BasicRoomコンストラクタをそれぞれパラメータとして入力します。

最後にconfigを登録する部分では、チャンネル情報の使用有無設定、プロトコル登録などの作業を行います。プロトコルを登録する部分は、後ほどインゲームチャットとジグソーパズルロジックを実装する部分で扱うことになります。

これでクライアントがサーバーに接続してゲームユーザーとしてログインし、ゲームルームを生成できる機能の実装が完了しました。しかし、サーバーに接続したからといって、すぐにゲーム関連機能(ゲームユーザー生成、ゲームルーム生成など)をリクエストできるわけではありません。今の状態でサーバーとクライアントを実行しても、クライアントはゲームサーバーの機能を使用できないでしょう。サーバーにこれらをリクエストするには、サーバー接続後にクライアント認証プロセスが必要です。次のチャプターでは、サーバーとクライアントで認証をどのように処理するかを扱います。

## サーバー接続

クライアントがサーバーに接続してからゲームにログインする前に、ユーザーの身元を確認し、認証プロセスを経る必要があります。
ゲームノードがゲームユーザーとゲームルーム生成の役割を担当するなら、ゲートウェイノードはユーザーの接続と認証機能を担当します。ゲームノードをクラス作成を通じて実装したように、ゲートウェイノードも一貫性のある方式で実装できます。
**GameAnvil GatewayNode**テンプレートを選択後、ファイル名を**BasicGatewayNode**に設定し、**OK**ボタンを押してゲートウェイノードクラスを生成します。ゲートウェイノードクラスは、基本生成されたコード以外に別途追加コードは必要ありません。

### プロトコル登録

認証プロセスを経ながら、サーバーとクライアントは互いに使用するプロトコルを確認するプロセスを経ます。したがって、認証プロセスを経る前にプロトコルを登録する必要があります。プロトコル登録は一度だけ行えばよいため、getConnector()コード内部にプロトコル登録コードを追加してみます。チュートリアルに従って進めると実装することになるインゲームチャットで使用するプロトコルをあらかじめ登録してみます。

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();

        // RegisterProtocolはAuthの前に行う必要がある。
        GameAnvilProtocolManager.RegisterProtocol(BasicProtocolReflection.Descriptor);
    }

    return connector;
}
```

<br>

### 認証コードの追加

Unityプロジェクトに移動してクライアント側の実装を行ってみます。クライアントでは接続リクエストと同様に、認証リクエストをコネクタGameAnvilConnector APIを通じてリクエストできます。認証リクエストは接続リクエスト後にのみ成立します。認証リクエストの結果を画面上のテキストとコンソールを通じて出力し、確認できるようにします。

```c#
// Client-side

public async void Auth()
{
    accountId = Random.Range(1000, 9999) + "";
    clientInfoText.text = "クライアント情報 : " + accountId;

    try
    {
        var result = await connector.Authentication(deviceId, accountId, password);
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
    }
    catch (Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
        throw;
    }
}
```

これでクライアントがサーバーに接続するだけでなく、認証プロセスまでリクエストできるように設定されました。

### 認証確認

Unityクライアントでプレイモードに入ります。コンソール上にログが接続、認証の順に順次出力されることを確認します。整理すると、クライアントのAuthリクエストに従い、サーバーのゲートウェイノードでユーザー認証が完了し、サーバーにログインできる状態になったということです。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/19_auth_success.png)

## ログイン

最後に行う過程は、ゲームノードにログインしてゲームユーザーを生成することです。ゲームユーザーはサーバーに接続した他のクライアントと通信するために必要な概念で、クライアント間で通信を行うには、各クライアントはゲームユーザーを生成し、ゲームルームを通じてメッセージをやり取りするようにしています。

### ログインコードの追加

接続及び認証が完了したクライアントは、ゲームノードへログインできます。サーバーにログインすると、ゲームノードに該当クライアントのためのゲームユーザーオブジェクトが生成されます。クライアントは、自身のサーバー側ゲームユーザーオブジェクトを通じてサーバーまたは他のユーザーとメッセージをやり取りできるようになります。先ほど作成した認証コードを以下のように修正し、認証が成功した場合にすぐログインを進めるようにします。

```c#
// Client-side

public async void Login()
{
    try
    {
        var result = await getUser().Login("USER_TYPE_BASIC", "ch1");
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
    }
    catch (Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
        throw;
    }
}
```

ログインをリクエストする際に一緒に渡したコールバックメソッドが、ログインリクエスト結果をログに出力するようにします。

### ログイン確認

Unityテストモードを通じて、正常にログインされることを確認します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/20_login_success.png)

## ルーム生成及び参加

### クライアント作業

UnityプロジェクトでConnectHandlerコードにルーム生成をリクエストするメソッドを作成します。このとき、CreateRoomメソッドの2番目の引数であるRoomTypeは、必ずサーバーで指定した値と同じである必要がある点に留意します。一般的にこのようなRoomTypeなどのプロトコルは、サーバーとクライアント開発者が事前に値をあらかじめ定義しておき使用します。ルーム生成結果コードをコンソールに出力します。また、ルーム生成に成功すればroomIdをクライアント側に保存しておき、ゲームシーンへ移動するようにコードを作成します。

```c#
public async void CreateRoom()
{
    try
    {
        var result = await getUser().CreateRoom(string.Empty, "ROOM_TYPE_BASIC", string.Empty);
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
        if (result.ErrorCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
        {
            Debug.Log("Room Id : " + result.Data.RoomId);
            this.roomId = result.Data.RoomId;
            SceneManager.LoadScene("GameScene");
        }
    }
    catch(Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
    }
}
```

ここまで完了すれば、クライアントを通じてルームを生成する機能を実装したことになります。Create RoomボタンのOnClickイベントとCreateRoomメソッドを接続します。実際に実行してみる前に、ルーム参加機能まで実装してからテストしてみます。今回はConnectHandlerにルーム参加コードを追加します。

JoinRoomメソッドの最初の引数がサーバー開発で使用した事前定義されたRoomType文字列と一致するか確認します。結果コードを通じてルーム参加リクエストに対するサーバー側の応答を確認し、ルーム参加に成功した場合はルーム生成と同様の動作をするように作成します。メソッド作成が完了したら、Join RoomボタンのOnClickイベントとJoinRoomメソッドを接続します。UnityのUpdate()関数でエンターボタンが入力されるとJoinRoom()を呼び出すようにコードを作成します。

```c#
private void Update()
{
    if (popupCanvas && popupCanvas.activeInHierarchy && Input.GetKeyDown(KeyCode.Return))
    {
        JoinRoom();
    }
}

public async void JoinRoom()
{
    if (!string.IsNullOrEmpty(roomIdInput.text))
    {
        try
        {
            var result = await getUser().JoinRoom("ROOM_TYPE_BASIC", int.Parse(roomIdInput.text), string.Empty);
            Debug.Log(result.ErrorCode.ToString());
            logText.text = result.ErrorCode.ToString();
            if (result.ErrorCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS)
            {
                Debug.Log("Room Id : " + result.Data.RoomId);
                this.roomId = result.Data.RoomId;
                SceneManager.LoadScene("GameScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
        }
    }
}
```

次にGameSceneへ移動して、ルーム参加後にどのような動作を行うかを実装してみます。まず初期GameManagerコードは以下のとおりです。ルームIDをゲーム上で表示できるようにします。

```c#
public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public InputField ChatInputText;

    private ConnectHandler connectHandler;


    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }
}
```

これでクライアントにサーバー接続、認証、ログイン機能だけでなく、ルーム参加と生成機能まで全て実装されました。

### Room生成及び参加テスト

Unityプロジェクトの上部ツールバーで**File > Build Settings**を選択します。以下のように必要なシーンをビルドするシーンリストに順次追加します。もしシーンの順序が間違っている場合は、リスト上の項目をドラッグしてConnectSceneが一番上に来るように順序を適切に調整します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/21_add_scene.png)

Unityで`cmd+b`または`ctrl+b`でビルド後にプレイします。新しいウィンドウが開きConnectSceneがロードされます。サーバーのIPアドレスを入力しConnectボタンを押してサーバー接続を試みます。例ではローカルサーバーを立ち上げて使用するため`127.0.0.1`を入力します。サーバー接続に成功した場合、コンソールログと共にゲーム画面上で接続状態が`CONNECT_SUCCESS`と表示されることを確認できます。もし途中の過程で失敗ログが残る場合、失敗コードを通じて原因を知ることができます。もしそれで不十分な場合は、サーバーのログを確認して原因を分析する必要があります。ログインまで成功したらCreate Roomボタンを押してルーム生成をリクエストします。GameSceneへシーンが移動し、生成されたルームのIDが一緒に出力されます。ルームIDは次の例の画像と異なる場合があります。

その状態でUnityプレイモードを実行した後、ログインまで完了することを確認します。そしてJoin Roomボタンをクリックした後、先ほど生成したルームのRoomIdを直接入力してルームに参加するようにします。ルーム参加に成功すればゲームシーンへ移動し、2つのゲーム画面で同じルームIDを持っていることが確認できるでしょう。テスト画面が以下のように表示されれば成功です。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/22_join_room.png)

もし正常に進行しない場合は、以下の事項を再確認してください。

- サーバー実装修正後、サーバープロセスを再起動したか？
- サービス名はGameAnvilConfig.jsonに設定したものと同じようにサーバー/クライアントに実装されているか？

## インゲームチャットの実装

これでクライアント間でサーバーを通じて通信できる環境が構築されました。ここではクライアントで生成したデータをリモートのクライアントが受信できる簡単な例を実装してみます。例のプロジェクト内部にあらかじめ実装されたプロトコルを利用して、チャット履歴をやり取りする方法を学びます。この例限定で、通信データをクライアントからサーバーへ送信する際はMessageRequestクラスを使用します。サーバーからクライアントへ通信データを送信する際はMessageResponseまたはMessageBroadcastクラスを使用します。

(プロジェクトテンプレートを使用して直接プロジェクトを作成した場合、Messageクラスを利用できません。チュートリアル用に作成されたプロジェクトをダウンロードし、内部のBasicProtocol.javaがプロジェクトに含まれるように設定してください。また、サーバー実行前にプロトコルを登録する過程も必要です。サーバー側のプロトコル関連設定は、あらかじめ用意されたチュートリアル用プロジェクトで完了しているため、チュートリアルプロジェクトをそのまま使用した場合は行わなくても構いません。)

### クライアント側の送信実装

まずクライアントからサーバー方向へメッセージを送信するコードを作成してみます。一度ログインしたユーザーとしてサーバーに接続されると、ユーザーエージェントを通じてサーバーの機能を活用できます。ここではユーザーエージェントのSend機能を活用してルームにパケットを送信します。メッセージ送信のためには、送信する内容を含むパケットを引数として渡す必要があります。

UnityプロジェクトのGameManagerスクリプトにメッセージ送信スクリプトを追加します。SendMessageメソッドが呼び出されると、画面上の入力フィールドにユーザーが入力したメッセージと自身のユーザーIDを含むパケットをuserAgentを通じてサーバーへ送信します。そしてEnterキーを押すたびにSendMessageメソッドが実行されるようにUpdate関数に実装を追加します。以下の例のコードはSendMessageメソッドの実装を示しています。

```c#
using System;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil.Defines;
using Protocol;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public InputField ChatInputText;

    private ConnectHandler connectHandler;

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Return) && !string.IsNullOrEmpty(ChatInputText.text))
        {
            SendMessage();
        }
    }

    public void SendMessage()
    {
        MessageRequest messageRequest = new MessageRequest();
        messageRequest.Message = "\n[" + connectHandler.accountId + "]:" + ChatInputText.text;
        connectHandler.getUser().SendUser(messageRequest);
        ChatInputText.text = string.Empty;
    }
}
```

このようにして、クライアントからサーバー方向へパケットを送信する機能が実装されました。しかし今はパケットをサーバーへ送っても、サーバーでは何の応答もないでしょう。その理由は、サーバー側で該当パケットを受け取った際に内容をどう分析し、どのような動作をするか定義していないためです。次のチャプターではサーバー側の実装を扱います。

### サーバー側のレスポンス実装

まずサーバーでチャットプロトコルを使用できるように、サーバー起動前にプロトコル登録を行います。

```java
@GameAnvilApplication
public class Main {
    public static void main(String[] args) {
        final var server = GameAnvilServer.getInstance();

        server.addProtocol(BasicProtocol.class);

        server.run(Main.class, args);
    }
}
```

ここではゲームユーザーが送信したメッセージをサーバーが受け取り、ルーム内のユーザーたちに送信する機能を作成します。クライアントが送信したメッセージをサーバーのゲームルームで処理するためにはハンドラを使用します。ハンドラとは、特定プロトコルを処理するためのコードの束を意味します。ハンドラはプロトコルの種類によって複数になることがあり、ルームにハンドラを複数登録できます。したがって、ルームは複数のプロトコルを処理可能です。

ハンドラもインターフェース実装を通じて生成されます。プロジェクトパネルでMainクラスがあるパスを右クリックした後、**New > Package**を選択して**handler**という名前の新しいパッケージを作成します。そして**handler**パッケージを再度右クリックした後、**New > GameAnvil RoomMessageHandler**を選択します。ファイル生成ダイアログが開いたら、**File name**に**BasicHandler**、**Message**に**BasicProtocol.MessageRequest**、**Room**に先ほど生成したIRoom実装クラスのクラス名**BasicRoom**を入力した後、**OK**をクリックします。

このようにするとBasicHandlerクラスが生成され、@GameAnvilControllerアノテーションと@GameRoomMappingアノテーションによりBasicRoomで使用するハンドラとして登録され、別途の登録手続きなしに使用できます。これでBasicRoomはBasicProtocolのMessageRequestメッセージをBasicHandlerを通じて処理できるようになりました。

実行される内容、つまりexecuteメソッド内部の実装は以下のように作成します。以下のハンドラ例の実装では、受信したメッセージに対して送信者にレスポンスメッセージを送信すると共に、ルーム全体のユーザーにルーム単位ブロードキャスト用メッセージを追加で送信します。このとき、クライアントはルーム単位ブロードキャストメッセージを基準にゲームを同期するように実装されています。

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.BasicProtocol;

@GameAnvilController
public class BasicHandler {

    @GameRoomMapping(value = BasicProtocol.MessageRequest.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, BasicProtocol.MessageRequest request) {
        BasicProtocol.MessageResponse response = BasicProtocol.MessageResponse.newBuilder().setMessage(request.getMessage()).build();
        BasicProtocol.MessageBroadcast broadcast = BasicProtocol.MessageBroadcast.newBuilder().setMessage(request.getMessage()).build();

        BasicRoom room = ctx.getRoom();
        room.broadcast(Packet.makePacket(broadcast));
        ctx.reply(response);
    }
}
```

上記のコードでは、まずMessageRequestオブジェクトのMessage値を利用してMessageResponse、MessageBroadcastオブジェクトをそれぞれ新しく生成します。MessageResponseタイプのオブジェクトは、パケットをルームへ送信したユーザーオブジェクトへ送信します。MessageBroadcastオブジェクトはroomを通じてルーム内部の全てのユーザーへ送信します。

こうしてクライアントが送信したパケットをサーバーが受信し、若干の処理を行った後に送り返す機能がサーバーに追加されました。このときクライアントもまた、サーバーから送信されたパケットをどのように処理するか指定する必要があります。

### クライアント側の受信実装

クライアントでもサーバー側のパケットを処理するために、あらかじめハンドラを登録する必要があります。そうでなければ、パケットを受け取った際に処理方法が分からないプロトコルだと判断して内容を破棄することになります。サーバーから送られる内容を受信した際にこれを検知して内容を処理するには、サーバーから送られるパケットのプロトコルタイプのハンドラを登録すればよいです。つまり、MessageBroadcastタイプのメッセージを処理するハンドラ登録コードを作成して登録します。

再度Unityプロジェクトへ移動し、GameManagerのStartメソッドに実装を追加します。まずプロトコルマネージャーを利用して、サーバーとクライアント間で合意されたプロトコルを登録します。ここでは0番としてBasicProtocolを登録しました。番号については以降に続く内容で説明します。

その後、ユーザーエージェントを通じてリスナーを登録します。メソッドタイプとしてはプロトコルのタイプを使用します。サーバーから送るように設定したプロトコルはMessageBroadcastなので、これを使用すればよいです。ハンドラ実装は、受信したメッセージを画面に用意されたテキストにそのまま出力する簡単なロジックで作成します。まさにチャットの最も基本的な機能です。

```c#
void Start()
{
    connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

    roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

    // コード追加
    connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
    {
        ChatLogText.text += messageBroadcast.Message;
    });
}
```

<br>

### メッセージ伝達確認

サーバー修正後に新しく実行したか確認し、Unityで`cmd+b`または`ctrl+b`でビルド後にプレイします。ビルドされたゲームでルームを生成し、サーバー側のログを確認します。その状態でUnityエディタでプレイモードに入った後、先ほど生成したルームのRoomIdを入力して該当ルームに参加します。

任意のゲームプロセスで入力欄にテキストを入力した後、エンターを押します。すると他のゲームプロセスでも同じテキストが表示されるでしょう。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/23_unity_chat_test.png)

簡単なチャットサーバー実装を通じてメッセージ処理過程を学習しました。次はもう少し実用的な例の実装過程を見ていきます。

## パズルゲームの実装

ゲームシーンにはシングルプレイが可能なパズルゲームがあらかじめ実装されています。プレイモードに入りパズルピースをドラッグして適切な位置の近くに置くと、格子上の正確な位置に補正されます。今回のチャプターでは、このゲームをマルチプレイヤーゲームにしてみます。

同じルーム内のユーザー間でやり取りするメッセージは、MessageRequest、MessageResponse、MessageBroadcast以外に、どのようなタイプの値も全て表現できます。今回のチャプターでは、直接プロトコルを定義し、サーバーとクライアントに登録する方法を見ていきます。

メッセージはあらかじめ定義したプロトコルに基づいて定義さえすれば、サーバーとクライアント間で送受信が可能です。XML、jsonなど様々な表現手段がありますが、GameAnvilはGoogle Protocol Buffersを使用します。これは速度と安定性の側面で最も良いソリューションの1つです。

### Google Protocol Buffersを利用したメッセージシリアライズ/デシリアライズ

プロトコルバッファを使用するには、まずメッセージをどのように定義するか仕様を作成する必要があります。例えばMessageRequestの仕様には単一文字列を含めます。その後プロトコル仕様をコンパイルし、希望する言語のファイルに変換します。その後はMessageRequestを使用した方式と同様に、パケットのメッセージプロトコルとして使用できます。

チュートリアルサンプルプロジェクトをそのまま使用する場合、以下のPuzzleプロトコル生成過程はすでに完了しているため、概念理解の参考にしてください。
パズル位置同期のためのメッセージプロトコル作成を始めます。サーバープロジェクトのsrc/main/protoパスにPuzzle.protoファイルを追加した後、以下のようにプロトコル仕様を作成します。

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

PuzzlePositionプロトコルは、パズルピースの最新位置を表すために、各パズルピースの固有番号情報(Index)とパズルの位置情報(PositionXとPositionY)で構成します。これでプロトコル定義は終わりました。次はこのようなプロトコル定義を各開発言語に合ったクラスにコンパイルします。

protoc実行ファイルはプロジェクト最上位パスにあります。該当プロジェクトへ移動し、次のコマンドラインを利用してJavaとC#のためのコンパイルを実行します。コマンドラインを実行した際、何の結果メッセージも表示されなければ成功したことになります。

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

これでsrc/main/javaパスに新しくPuzzle.javaクラスが生成されたことを確認できます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/24_puzzle_proto_java.png)

C#クラスファイルはFinderやファイルエクスプローラーなどのプログラムを利用して、UnityプロジェクトのAsset/Protocolパスへ移動します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/25_puzzle_proto_csharp.png)

パズル位置同期のためのメッセージプロトコル作成が終わりました。

### プロトコル登録

プロトコルを定義しコンパイルまで無事に終えたら、サーバーとクライアントの両方に該当プロトコルクラスを登録する必要があります。GameAnvilサーバーのMainメソッドで以下のようにプロトコルを登録します。

```java
@GameAnvilApplication
public class Main {
    public static void main(String[] args) {
        final var server = GameAnvilServer.getInstance();

        server.addProtocol(BasicProtocol.class);
        // コード追加
        server.addProtocol(Puzzle.class);

        server.run(Main.class, args);
    }
}
```

UnityプロジェクトはConnectHandlerスクリプトのgetConnector()メソッドに以下のようにプロトコルを追加します。

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();

        GameAnvilProtocolManager.RegisterProtocol(BasicProtocolReflection.Descriptor);
        // コード追加
        GameAnvilProtocolManager.RegisterProtocol(PuzzleReflection.Descriptor);
    }

    return connector;
}
```

<br>

### クライアント側の送信実装

これでゲームのためのプロトコル定義及び登録まで全て終わりました。今からはこのようなプロトコルに基づいたメッセージを実際に送信する機能を実装します。まずクライアント側でデータを送信する部分を先に実装します。パズルピースをドラッグする間、その位置をサーバーへ送信してみます。

Unityプロジェクトへ移動し、Puzzle.csファイルを開いて以下のようにコードを追加します。ドラッグされる間、パズルの位置情報を含んだメッセージオブジェクトを生成します。そしてGameAnvilUserを通じてサーバーへ送信します。メッセージのプロトコルが異なるだけで、先ほど実習したMessageRequestと同じ方式であることが分かります。

```c#
using UnityEngine;
using UnityEngine.EventSystems;
using Protocol;

public class Puzzle : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public int index;

    public Transform puzzleHolder;
    public float tolerance;


    private ConnectHandler connectHandler;

    void Start()
    {
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

    public void FixPosition()
    {
        if (Vector2.Distance(transform.position, puzzleHolder.position) < tolerance)
        {
            transform.position = puzzleHolder.position;
        }
    }

    public void SendPuzzlePosition(Vector2 puzzlePosition, bool onEndDrag)
    {
        PuzzlePosition position = new PuzzlePosition();
        position.Index = index;
        position.PositionX = (int)puzzlePosition.x;
        position.PositionY = (int)puzzlePosition.y;
        position.OnEndDrag = onEndDrag;

        connectHandler.getUser().SendUser(position);
    }
}
```

<br>

### サーバー側のレスポンス実装

クライアント側では継続的にパズルの位置をサーバーへ送ることになりました。次はパズルの位置をサーバーでどのように処理するか作成する必要があります。MessageRequestを加工してMessageResponse、MessageBroadcastでユーザーに送り返したように、パズル位置を再度ゲームルームの全ユーザーに送り返すように実装してみます。

サーバープロジェクトに戻り、**GameAnvil RoomMessageHandler**ファイルテンプレートを利用してPuzzlePositionHandlerクラスファイルを生成します。そして受け取ったパケットをそのままルーム内の全ユーザーに伝達するようにbroadcastメソッドを使用します。

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionHandler {

    @GameRoomMapping(value = Puzzle.PuzzlePosition.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePosition request) {
        BasicRoom room = ctx.getRoom();
        room.broadcast(Packet.makePacket(request)); // ルーム全体のユーザーにメッセージを送信
    }
}

```

PuzzlePositionHandlerもやはり@GameAnvilControllerアノテーションと@GameRoomMappingアノテーションによりBasicRoomで使用するハンドラとして登録されます。これでBasicRoomはPuzzleプロトコルのPuzzlePositionメッセージをPuzzlePositionHandlerを通じて処理できるようになりました。


<br>

### クライアント側の受信実装

先ほどサーバーから送信されたMessageBroadcastを処理するために、あらかじめハンドラを登録する必要がありました。今回も同様にパズル位置を処理するハンドラを作成するために、GameManagerのStartメソッドにPuzzlePositionメッセージを処理するハンドラ登録コードを作成します。このハンドラはメッセージを受信すると、パズルオブジェクトを探してサーバーから受け取った位置へ移動させます。

```c#
public class GameManager : MonoBehaviour{

    ...(省略)...

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
        {
            ChatLogText.text += messageBroadcast.Message;
        });

        // 追加
        connectHandler.getUser().SetMessageCallback<PuzzlePosition>((_, _, puzzlePosition) =>
        {
            Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);

            if (puzzlePosition.OnEndDrag)
            {
                puzzle.FixPosition();
            }
        });
    }

    ...(省略)...
}
```

<br>

### パズル位置同期確認

Unityで`cmd+b`または`ctrl+b`でビルド後にプレイします。ビルドされたゲームでルームを生成した後、Unityプレイモードを実行し、生成されたルームのRoomIdを入力してルームに参加します。これでパズルピースをドラッグして位置を移動すると、該当パズルピースの位置が同期され、リモートクライアントに反映されることを確認できます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/26_unity_puzzle_position_playmode.gif)

<br>

### 途中参加ユーザーの処理

ゲーム中に任意のパズルピースの位置が変更された後、新しいユーザーがルームに入室する場合を考えてみましょう。このとき、既存ユーザーと新規ユーザー間のパズル状態は異なります。新しく入ってきたユーザーは初期のパズル状態を持っているため、既存ユーザーとパズル状態を同期させる必要があります。これを解決するために、サーバー側のロジックを修正します。サーバーはパズルの位置情報を全て保管し、新しいユーザーが入室すれば該当情報を利用して同期するようにロジックを修正します。

BasicRoomにpuzzlePositionsマップを追加します。このマップはパズルピースごとの位置情報を管理します。

```java
public class BasicRoom implements IRoom<BasicUser> {

    private static final Logger logger = getLogger(BasicRoom.class);

    private IRoomContext roomContext;

    // コード追加
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();

    ...(省略)...
}
```

次にPuzzlePositionHandlerコードを修正して、パズル位置を各ルームに保存するようにします。

```Java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionHandler {

    @GameRoomMapping(value = Puzzle.PuzzlePosition.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePosition request) {
        BasicRoom room = ctx.getRoom();

        // コード追加
        room.puzzlePositions.put(request.getIndex(), request);
        room.broadcast(Packet.makePacket(request)); // ルーム全体のユーザーにメッセージを送信
    }
}

```

新しいユーザーがルームに入ってくる時、保存しておいたパズル位置情報を受け取れるようにBasicRoomのonJoinRoomを修正し、サーバーに保存しておいたパズルの位置情報を送信するようにします。

```java
public class BasicRoom implements IRoom<BasicUser> {

    ...(省略)...

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        // コード追加
        // 一時的に作成されたコードであり、次の過程で問題を解決しながら該当コードは削除されます。
        for(Puzzle.PuzzlePosition puzzlePosition : puzzlePositions.values()) {
            broadcast(Packet.makePacket(puzzlePosition));
        }
        return isSuccess;
    }

    ...(省略)...
}
```

このように修正した後、再度テストしてみます。クライアントの1つでまずパズル位置を修正した後、他のクライアントでゲームルームに参加してパズル位置が正常に同期されるか確認します。意図とは異なり、依然として動作しないことが確認できます。

これはonJoinRoom呼び出し以降にクライアントでシーン移動が起こるためです。つまり、リスナー登録が完了する前に同期メッセージが伝達されて問題が発生したのです。この問題を解決するためには、クライアントでシーン移動が終わりリスナー登録まで終えた後に、パズルの位置情報を同期する必要があります。

この問題はすぐには解決せず、以降の内容で扱います。このような問題は、すぐ次に説明するパズルシャッフルでも如実に現れます。そのためパズルシャッフルから確認した後、これを解決するために修正した内容を再度扱うことにします。

<br>

## パズルシャッフルの実装

パズル位置をランダムに混ぜるロジックを実装してみます。クライアントでパズル位置シャッフルをリクエストすると、サーバーでパズルを混ぜた後に新しい位置を決定します。そして変更された位置情報を再度クライアントへ送り返すのが基本的なアイデアです。

### プロトコル登録

まずパズルシャッフルをリクエストするためのプロトコルを作成してみます。サーバープロジェクトへ移動し、Puzzle.protoファイルにプロトコル仕様を追加します。このプロトコルは特にクライアントからサーバーへ送る情報がないため、フィールドが1つもありません。これもまたプロトコルとして十分に有意義です。

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

message ScatterPuzzle { } // パズルシャッフルリクエストプロトコル
```

プロトコルバッファファイルを修正すると、コンパイルも再度行う必要があります。

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

生成されたC#クラスを再度Finderやファイルエクスプローラーなどのプログラムを利用して、UnityプロジェクトのAsset/Protocolフォルダへ移動します。

<br>

### クライアント側の実装

Unityプロジェクトへ移動し、GameManager.csに以下のようにシャッフルリクエストのためのコードを作成します。Scatterメソッドが呼び出されると、GameAnvilUserを通じて新しいScatterPuzzleタイプのメッセージがゲームルームへ送信されます。

```c#
public class GameManager : Monobehaviour {

    ...(省略)...

    public void Scatter()
    {
        connectHandler.getUser().SendUser(new ScatterPuzzle());
    }

    ...(省略)...
}
```

**Hierarchy**パネルの**Scatter Puzzle Button**をクリックします。**Inspector**の**Button**コンポーネントで**OnClick**リスナーに項目を追加した後、GameManagerコンポーネントをドラッグして登録し、ドロップダウンからScatterメソッドを選択します。

### サーバー側の実装

シャッフルリクエストが入った時の処理は、先ほど使用した方式どおりハンドラを利用します。**GameAnvil RoomMessageHandler**ファイルテンプレートを通じてScatterPuzzleHandlerクラスを生成します。16個の各パズルの位置をランダムに設定した後、PuzzlePositonタイプのメッセージを送信します。またサーバーのpuzzlePositionsマップも新しい位置情報で更新します。

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

@GameAnvilController
public class ScatterPuzzleHandler {
    private static final int mapSize = 400;

    private static class Point {
        public int x;
        public int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    @GameRoomMapping(value = Puzzle.ScatterPuzzle.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.ScatterPuzzle request) {
        BasicRoom room = ctx.getRoom();
        room.puzzlePositions = new HashMap<>();
        List<ScatterPuzzleHandler.Point> random = IntStream.rangeClosed(-4, 4).boxed()
                .map(i -> new Point(i * mapSize / 4, i % 2 == 0 ? mapSize : -mapSize))
                .collect(Collectors.toList());
        random = Stream.concat(random.stream(), random.stream().map(p -> new Point(p.y, p.x))).collect(Collectors.toList());
        Collections.shuffle(random);

        for (int i = 0; i < 16; i++) {
            ScatterPuzzleHandler.Point point = random.get(i);
            int x = 1300 + point.x;
            int y = 550 + point.y;

            Puzzle.PuzzlePosition puzzlePosition = Puzzle.PuzzlePosition.newBuilder().setIndex(i).setPositionX(x).setPositionY(y).build();
            room.puzzlePositions.put(i, puzzlePosition);
            room.broadcast(Packet.makePacket(puzzlePosition));
        }
    }
}
```

これでクライアント側の送信機能とサーバー側の応答機能が全て完成しました。

### パズルシャッフル機能の確認

Unityエディタでプレイモードに入ります。Scatter Puzzleボタンをクリックしてパズルシャッフル機能が正しく動作するか確認します。

## より良い途中参加ユーザー処理

先ほど途中参加ユーザーを処理する過程で同期問題が発生しました。その原因は、ユーザーがルームに入室する時点をパズルピースの位置同期時点として適切だと考えたためです。しかし、私たちが実装中のパズルゲームは、ユーザーがルームに入室する時点でシーン移動が発生します。

そのため、私たちはリスナー登録とonJoinRoomコールバック呼び出しの2つの時点に分けて考える必要があります。これはゲームクライアントの実装によって問題がない場合もあれば、問題になる場合もあります。onJoinRoomコールバック呼び出し以降にシーン移動が始まるため、シーン移動が完了した直後に、クライアントが直接サーバーへパズル位置同期をリクエストするようにします。

### プロトコル登録

サーバープロジェクトへ移動し、パズル位置同期をリクエストするためのプロトコルをPuzzle.protoに追加します。

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

message PuzzlePositionReq {} // パズル同期リクエスト
```

プロトコルを再コンパイルした後、生成されたC#クラスをUnityプロジェクトへ移動します。

<br>

### クライアント側の実装

シーン移動直後にサーバーへパズル位置をリクエストするようにするためには、GameSceneへシーン移動した直後に実行されるStart関数を修正する必要があります。GameManagerのStartメソッドに、次のように新しく作成したPuzzlePositionReqプロトコルメッセージを送信するコードを追加します。

```c#
public class GameManager : MonoBehaviour
{
    ...(省略)...

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
        {
            ChatLogText.text += messageBroadcast.Message;
        });

        connectHandler.getUser().SetMessageCallback<PuzzlePosition>((_, _, puzzlePosition) =>
        {
            Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);

            if (puzzlePosition.OnEndDrag)
            {
                puzzle.FixPosition();
            }
        });

        // コード追加
        connectHandler.getUser().SendUser(new PuzzlePositionReq());
    }

  	...(省略)...
}
```

<br>

### サーバー側の実装

onJoinRoomに誤って実装していたパズル位置送信コードは削除し、Puzzle.PuzzlePositionReqメッセージに対するハンドラPuzzlePositionReqHandlerクラスを生成後、新しく作成します。

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionReqHandler {
    @GameRoomMapping(value = Puzzle.PuzzlePositionReq.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePositionReq request) {
        BasicRoom room = ctx.getRoom();
        room.puzzlePositions.values().stream().forEach(puzzlePosition -> { room.broadcast(Packet.makePacket(puzzlePosition)); });
    }
}

```

<br>

### 途中参加ユーザー処理確認

Unityで`cmd+b`または`ctrl+b`でビルド後にプレイします。これでビルドされたゲームでルームを生成し、パズルシャッフルを実行します。その状態でUnityエディタのプレイモードに入り、このルームに参加した後、パズル位置が同期されることを確認します。

## ユーザーマッチメイキングの実装

### サーバー側の実装

ユーザーマッチメイキングは、ユーザーのマッチメイキングリクエストを一箇所に集め、適切な基準に合わせて似たレベルのユーザー同士が同じルームでゲームを開始できるようにします。勝ち点やスコアなど様々な要素をユーザーが直接実装し、ユーザーを適切に区分してマッチングできます。ここではユーザー2人を1つのゲームとしてマッチングするロジックを実装します。

プロジェクトパネルでMainクラスがあるパスを右クリックした後、**New > Package**を選択して**match**という名前の新しいパッケージを作成します。そして**match**パッケージを再度右クリックした後、**New > GameAnvil UserMatchInfo**を選択します。ファイル生成ダイアログが開いたら、**File name**に**BasicUserMatchInfo**を入力した後、**OK**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/27_create_user_match_info.png)

このクラスにはマッチングに使用されるユーザーの情報を含めることになります。マッチメイキングに使用される要素があればここに追加すればよいです。今回の例では特別な要素を追加せず、基本的に実装されたメソッドのみを使用します。1つ注意すべき点は、getId()メソッドが必ずリクエストしたユーザーのIDを返すように実装されているか確認することです。そしてパーティーマッチメイキング機能は使用しないため、0を返すように設定します。

```java
package org.example.match;

import com.nhn.gameanvil.node.match.usermatch.AbstractUserMatchInfo;

public class BasicUserMatchInfo extends AbstractUserMatchInfo implements Comparable<BasicUserMatchInfo> {

    private int userId;

    public BasicUserMatchInfo(int userId) {
        this.userId = userId;
    }

    @Override
    public int getId() {
        return userId;
    }

    @Override
    public int getPartySize() {
        return 0;
    }

    @Override
    public int compareTo(BasicUserMatchInfo o) {
        return 0;
    }
}

```

このようなUserMatchInfoは、クライアントがユーザーマッチメイキングをリクエストする際、サーバーのゲームユーザーでonMatchUserコールバックを実装する過程で生成した後に使用します。GameAnvilは基本的なユーザーマッチメーカーを提供します。以下のonMatchUserは、このようなエンジンの基本ユーザーマッチメイキングをmatchUser APIを通じて使用しています。

```java
public class BasicUser implements IUser {

    ...(省略)...

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;

        try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(userContext.getUserId());
            return userContext.matchUser(matchingGroup, roomType, term);
        } catch (TimeoutException | NodeNotFoundException | GameAnvilException e) {
            logger.error("BasicUser::onMatchUser", e);
        }

        return isSuccess;
    }

    ...(省略)...
}
```

ユーザーマッチメイキングを使用するための基本的な準備ができたら、次は実際にマッチメイキングを実行するマッチメーカーを作成します。

プロジェクトパネルの**match**パッケージを右クリックした後、**New > GameAnvil UserMatchMaker**を選択します。ファイル生成ダイアログが開いたら、**File name**に**BasicUserMatchMaker**、**Room**に**BasicRoom**、**User Match Info**に**BasicUserMatchInfo**を入力した後、**OK**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/28_create_user_match_maker.png)

コンストラクタでは親クラスのコンストラクタを呼び出しながら、引数としてマッチ人数とマッチ申請有効時間を渡します。有効時間が過ぎると該当マッチリクエストは自動的にキャンセルされます。そして実際にマッチメイキングを実行するmatchメソッドは、内部的にエンジンによって1秒に1回ずつ呼び出されます。

getMatchRequestsは引数としてマッチングのための最小人数を受け取り、現在のマッチングプール全体を照会して最適なマッチングを可能な限り作成します。つまり、マッチングリクエストが多く溜まっていれば、getMatchRequestsによって一度に100個または1,000個のマッチングも作成される可能性があります。このとき、マッチングが成功すればマッチングされたユーザーたちのUserMatchInfoリストを返します。このリストをエンジンが提供するmatchSingles APIに渡すと、該当リスト内のユーザーに対するルーム生成及び移動が自動的に進行します。

もしマッチングに十分なリクエストが溜まっていないか、条件に合う対象がない場合はnullを返します。この場合には次の1秒後のmatch呼び出しで、再度同じマッチング検索が実行されます。

```java
package org.example.match;

import com.nhn.gameanvil.game.GameAnvilUserMatchMaker;
import com.nhn.gameanvil.node.match.AbstractUserMatchMaker;
import org.example.game.BasicRoom;

import java.util.List;

@GameAnvilUserMatchMaker(loadClass = BasicRoom.class)
public class BasicUserMatchMaker extends AbstractUserMatchMaker<BasicUserMatchInfo> {

    public BasicUserMatchMaker() {
        super(2, 5000);
    }

    private int matchSize = 2;

    @Override
    public void onMatch() {
        List<BasicUserMatchInfo> matchRequests = getMatchRequests(matchSize);

        if (matchRequests == null) {
            return;
        }

        int matchingAmount = matchSingles(matchRequests);
    }

    @Override
    public boolean onRefill(BasicUserMatchInfo matchReq) {
        return false;
    }
}
```

<br>

### クライアント側の実装

マッチメイキングロジックは全てサーバーに実装されているため、クライアントではマッチメイキングが必要な時点でリクエストを送るだけで済みます。ConnectHandlerにMatchUserメソッドを追加します。そしてマッチメイキングが終わった時点でシーンを移動するようにコードを追加します。

```c#
public class ConnectHandler : MonoBehaviour
{
	...(省略)...

    public void MatchUser()
    {
        try
        {
            getUser().OnMatchUserDone += OnMatchUserDone;
            var result = getUser().MatchUserStart("ROOM_TYPE_BASIC", string.Empty);
            Debug.Log("MATCHING_USER...");
            logText.text = "MATCHING_USER...";
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
            throw;
        }
    }

    private void OnMatchUserDone(GameAnvilUser user, ResultCodeMatchUserDone resultCode, MatchResult matchResult)
    {
        getUser().OnMatchUserDone -= OnMatchUserDone;
        Debug.Log("Room Id : " + matchResult.RoomId);
        this.roomId = matchResult.RoomId;
        SceneManager.LoadScene("GameScene");
    }

	...(省略)...
}
```

シーンでMatchUserボタンのOnClickリスナーにConnectHandlerコンポーネントをドラッグして登録し、ドロップダウンからMatchUserメソッドを選択します。

### ユーザーマッチメイキングテスト

Unityで`cmd+b`または`ctrl+b`でビルド後にプレイします。その状態でUnityエディタでプレイモードに入ります。両側で全てUser Match Makingボタンを押し、マッチングが成立して同じルーム番号でまとめられることを確認します。

## ルームマッチメイキングの実装

ルームマッチメイキングは、マッチメーカーが管理するルームの中でユーザーの要求事項に最も適したルームへ自動入室させることができる機能です。つまり、ユーザーマッチメイキングがユーザーとユーザーをマッチングさせる機能なら、ルームマッチメイキングはユーザーとルームをマッチングさせる機能です。このとき、実装方式によって多様な条件でユーザーをルームにマッチングできます。ここではまだ定員が埋まっていないルームの中で、人数が最も少ないルームへ入室するマッチメイキングを実装します。

まずマッチメイキングを実際に実行するクラスが必要です。そしてルームマッチメーカーでルームを管理するための情報を含むインスタンスがルームごとに1つずつ必要です。最後にユーザーがマッチメイキングを申請するたびに、ユーザーの要求事項を含んだ申請書インスタンスが必要です。このように3つの新しいクラスを作成してみます。

さらに既存のロジックを一部修正します。ルームマッチメイキングは全てのルームではなく、ルームマッチメイキング対象として申請したルームのみを対象に実行されます。したがって、ルーム生成時点でルームマッチメイキングを対象として申請するコードを追加します。

### サーバー側の実装

まずマッチメイキングリクエストを表すクラスを実装します。BasicRoomMatchFormクラスを生成します。

プロジェクトパネルの**match**パッケージを右クリックした後、**New > GameAnvil RoomMatchForm**を選択します。ファイル生成ダイアログが開いたら、**File name**に**BasicRoomMatchForm**を入力した後、**OK**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/29_create_room_match_form.png)

ユーザーがマッチメイキングリクエストを行うたびに、BasicRoomMatchFormオブジェクトが生成され使用されます。

```java
package org.example.match;

import com.nhn.gameanvil.node.match.roommatch.AbstractRoomMatchForm;

public class BasicRoomMatchForm extends AbstractRoomMatchForm {
    public BasicRoomMatchForm() {
        super();
    }
}
```

次はマッチング対象となるルームの情報を表現するクラスを実装します。以下のようにBasicRoomMatchInfoクラスを生成します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/30_create_room_match_info.png)

このとき、ルームの最大定員はMAX_ENTRY_USERフィールドに指定された4名です。このような最大定員とroomIdを必ず継承されたAbstractRoomMatchInfoコンストラクタに引数として渡す必要があります。

```java
package org.example.match;

import com.nhn.gameanvil.node.match.roommatch.AbstractRoomMatchInfo;

public class BasicRoomMatchInfo extends AbstractRoomMatchInfo {
    private static final int MAX_ENTRY_USER = 4;

    public BasicRoomMatchInfo(int roomId) {
        super(roomId, MAX_ENTRY_USER);
    }
}
```

次に実際にルームマッチメイキングを処理するルームマッチメーカーを生成します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/31_create_room_match_maker.png)

以下のようにBasicRoomMatchMakerが生成されます。

```java
package org.example.match;

import com.nhn.gameanvil.game.GameAnvilRoomMatchMaker;
import com.nhn.gameanvil.node.match.AbstractRoomMatchMaker;
import org.example.game.BasicRoom;

@GameAnvilRoomMatchMaker(loadClass = BasicRoom.class)
public class BasicRoomMatchMaker extends AbstractRoomMatchMaker<BasicRoomMatchForm, BasicRoomMatchInfo> {

    @Override
    public boolean onMatch(BasicRoomMatchForm roomMatchForm, BasicRoomMatchInfo roomMatchInfo, Object... args) {
        return false;
    }

    @Override
    public int compare(BasicRoomMatchInfo o1, BasicRoomMatchInfo o2) {
        return 0;
    }
}
```

compareメソッドはマッチングプールに入っているルームを整列する条件を実装します。例では人数に従ってルームを整列するように実装しました。

これでルームマッチメイキングのための準備がほぼ終わりました。クライアントがルームマッチリクエストを送ると、サーバーのBasicUserはonMatchRoomコールバックが呼び出されます。このコールバックで先ほど確認したBasicRoomMatchFormオブジェクトを生成した後、matchRoom APIに引数として渡して呼び出します。つまり、クライアントが送ったルームマッチングリクエストをここでマッチメーカーへ伝達しました。

```java
public class BasicUser implements IUser {

    ...(省略)...

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();
        try {
            return userContext.matchRoom(matchingGroup, roomType, gameRoomMatchForm);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicUser::onMatchRoom", e);
            return RoomMatchResult.FAILED;
        }
    }

    ...(省略)...
}
```

ルームが生成された時点でルームマッチメイキング対象にするために、BasicRoomのonCreateRoomコールバックで以下のようにregisterRoomMatch APIを呼び出します。

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    ...(省略)...

    @Override
    public boolean onCreateRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        try {
            roomContext.registerRoomMatch(gameRoomMatchInfo, user.getUserContext().getUserId());
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicRoom::onCreateRoom()", e);
        }
        return isSuccess;
    }

    ...(省略)...
}
```

またルーム情報が変動するたびにルームマッチメイキング情報も更新される必要があります。参考までにルームの人数変動はルームマッチメーカーが自動的に同期します。そのため、人数を除いた追加情報の更新のみ実行すればよいです。

以下はonJoinRoomコールバックを修正し、ルームにユーザーが参加する時にマッチメイキング情報を更新するコードです。

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    ...(省略)...

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        try {
            roomContext.updateRoomMatch(gameRoomMatchInfo);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicRoom::onJoinRoom()", e);
        }
        return isSuccess;
    }

    ...(省略)...
}
```

<br>

### クライアント実装

ユーザーマッチメイキングと同様に、クライアントはマッチメイキングが必要な時点でリクエストを送るだけで済みます。ConnectHandlerにRoomMatchMakingメソッドを追加します。

```c#
public class ConnectHandler : MonoBehaviour {

    ...(省略)...

    public async void MatchRoom()
    {
        try
        {
            var result = await user.MatchRoom(true, true, "ROOM_TYPE_BASIC", "", "");
            if (result.ErrorCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                Debug.Log("Room Id : " + result.Data.RoomId);
                this.roomId = result.Data.RoomId;
                SceneManager.LoadScene("GameScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
            throw;
        }
    }

    ...(省略)...
}
```

シーンでMatchRoomボタンのOnClickリスナーにConnectHandlerコンポーネントをドラッグして登録し、ドロップダウンメニューからMatchRoomメソッドを選択します。

<br>

### ルームマッチメイキングテスト

Unityで`cmd+b`または`ctrl+b`でビルド後、プレイ状態でルームを生成します。その状態でUnityエディタでプレイモードに入ります。プレイモードでRoom Match Makingボタンを押し、ビルドモードで生成したルームへ移動するか確認します。

<br>

## Room退出の実装

最後にルームを退出する機能実装のために、UnityクライアントのGameManagerに以下のメソッドを追加します。

```c#
public class GameManager : MonoBehaviour
{
    ...(省略)...

    public async void LeaveRoom()
    {
        try
        {
            var result = await connectHandler.getUser().LeaveRoom();
            Debug.Log(result.ErrorCode);
            if (result.ErrorCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
            {
                Destroy(connectHandler.gameObject);
                SceneManager.LoadScene("ConnectScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            throw;
        }
    }

    ...(省略)...
}
```

シーンでLeave RoomボタンのOnClickリスナーにGameManagerコンポーネントをドラッグして登録し、ドロップダウンメニューからLeaveRoomメソッドを選択します。

## プロジェクトの仕上げ

以上、GameAnvilとUnityを利用して、リアルタイムマルチプレイが可能なパズルゲームを実装してみました。その過程で、GameAnvilの核心機能の多くを使用しました。しかし、GameAnvilはこのチュートリアルに含まれていない、さらに豊富で多様な機能をサポートしています。これらの機能については、続くドキュメントを参照してください。また、共に提供されるリファレンスサンプルプロジェクトとJavaDocも、GameAnvilを理解するのに大いに役立つでしょう。
