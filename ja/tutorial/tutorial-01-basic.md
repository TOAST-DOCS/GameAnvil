## Game > GameAnvil > 基礎チュートリアル

### GameAnvilでマルチプレイヤーゲームを簡単に作成する

GameAnvilはリアルタイムマルチプレイヤーゲームサーバー構築プラットフォームです。
GameAnvilを使えば、簡単にゲームサーバーとクライアントを開発・運営できます。

この文書はGameAnvilの基本的な機能を利用して、実際のプレイが可能なマルチプレイヤー同期ゲームを開発する過程を説明します。
サーバーの概念とAPIを単純に列挙するのではなく、直接マルチプレイヤーゲームサーバーとサンプルクライアントを開発することで、GameAnvilの基本概念とプロジェクト構成及び実装方法を自然に習得できるようにしました。

GameAnvilはサーバーエンジンだけでなく、クライアントをサーバーに接続するのに役立つコネクタも提供しています。サーバーとクライアントが相互作用する様子を見ることができるサンプルを完成させながら、GameAnvilを使ってゲームを開発する全体的な流れに慣れることができます。

## 実習環境の準備 - サーバープロジェクト

マルチプレイヤーゲームを作るためには、クライアントと対応するサーバープログラムが必要です。ゲームサーバーを構築した後、続いてクライアントを実装する形でチュートリアルを進めます。

この例では、クライアントプログラムの作成にはUnityとGameAnvilコネクタを使用し、サーバープログラムの作成には先に紹介したサーバーエンジンGameAnvilを使用します。まず、GameAnvilを利用したサーバープログラムプロジェクトを作成してみましょう。

下記の段階を進めて作成される最終サーバーサンプルプロジェクトは下記のリンクからダウンロードできます。初期テンプレートからいくつかの段階を経てサーバー機能を実装したらどんな構造になるのか事前に確認したい場合は、該当プロジェクトをダウンロードして参照することができます。

[サーバーサンプルプロジェクトのダウンロード](https://static.toastoven.net/prod_gameanvil/files/tutorial/basic-tutorial/GameAnvilServerTutorial_1213.zip?disposition=attachment)

### プロジェクト構成

この章では、開発を始めるために初期設定を完了することを目標とします。実際のプロセスを実行してサーバーを駆動するのは次の章で説明します。

例では、サーバープロジェクトIDEをJetBrain社のIntelliJを使用しています。例で使ったIntelliJのバージョンはIDEA Ultimate 2023.1.2です。もし、ライセンスを購入していない場合は、IntelliJ IDEA Community Editionを使用しても構いません。他のバージョンのIntelliJを使用しても問題なく動作すると予想されますが、すべてのケースをテストしたわけではないので、サンプル実行バージョンと同じ環境で進めることを推奨します。

プロジェクトにGameAnvilを適用するには、MavenリポジトリにGameAnvilライブラリをダウンロードし、GameAnvilを駆動するために必要な設定ファイルを作成する必要があります。最後に少しのボイラープレートコードを書けば、開発初期設定が完了します。

GameAnvilではこのような一連の過程を代行してくれるIntelliJテンプレートを提供し、より簡単に初期作業を完了できます。次のリンクからIntelliJ用プロジェクトファイルテンプレートをダウンロードできます。ダウンロードしたテンプレートは解凍しないようにしてください。

[テンプレートダウンロード](https://static.toastoven.net/prod_gameanvil/files/tutorial/basic-tutorial/GameAnvil%20Template%202.1.0.zip?disposition=attachment)

ダウンロードしたテンプレートを適用するためにIntelliJを実行します。 **Welcome to InteliJ IDEA**画面の左側のメニューから**Customize**を選択し、**Import Settings...**をクリックします。または全体検索窓で**Import Settings...**を検索します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/import_gameanvil_template.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/search_import_settings.png)

<br>

ファインダーまたはファイルエクスプローラーウィンドウでテンプレートをダウンロードしたパスに移動し、圧縮ファイルを選択します。**Select Components to Import**ウィンドウが開いたら、**File templates**項目と**Project Templates** 項目をすべてチェックして選択します。 **OK**をクリックした後、**Import and Restart**をクリックすると、IntelliJが再起動し、テンプレートの適用が完了します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/select_import.png)

IntelliJ右上のボタングループで**New Project**をクリックした後、左側のリストをスクロールして下部の**Templates**にある**GameAnvil Template**を選択します。プロジェクト名を**SynchronizeTutorial**に設定します。名前に空白があってはいけません。プロジェクトの位置とベースパッケージ名を確認した後、プロジェクトを作成します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/imported_gameanvil_template.png)

これでIntelliJにサーバープロジェクトの骨格が構成されました。Projectパネルを見ると、コードと設定ファイルが作成されたことが確認できます。

* Main:プログラムの入口であるMain関数を含むクラスです。
* protocolパッケージ: javaでコンパイルされたプロトコルバッファファイルを含むパッケージです。
* protoパッケージ: Google Protobufライブラリを使って作成されたプロトコルファイルです。
* build.sh / build.bat:プロトコルファイルをjavaでコンパイルしてプロトコルバッファファイルを作成する実行ファイルです。
* GameAnvilConfig.json: GameAnvilの駆動に必要なサーバー設定情報を記録したファイルです。サーバーの実装に合わせて修正できます。
* logback.xml: Javaプロジェクトでロギングを設定するために使用するファイルです。Logbackフレームワークの設定ファイルで、ロギングシステムの動作方法やログの形式、保存場所などを指定します。このファイルを使用して、ロギングレベル、ログ形式、ログファイルのパスと名前、ログローリングポリシーなどを設定できます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_project_view_init_1213.png)

## GameAnvilサーバー設定ファイルの修正

プロジェクトパネルのresourcesパッケージの下にあるGameAnvilConfig.jsonファイルを介してGameAnvilサーバーの設定を変更できます。

* common:サーバーの全体的な設定を扱う部分
* location:ロケーションノードに関する設定を扱う部分
* match:マッチノード関連の設定を扱う部分
* gateway:ゲートウェイノード関連の設定を扱う部分
* game:ゲームノード関連の設定を扱う部分

テンプレートを使ってプロジェクトを構成したので、GameAnvilConfig.jsonファイルにサーバーの動作に必要な基本設定情報が設定されていることが確認できます。この例題で注意深く見るべき部分は大きく3つです。

1. gameのnodeCntの値
2. gameのserviceNameの値
3. gameのchannelIDsの値

ゲームノードは必要な量に応じて、またはサーバーの性能によって複数のVMで構成して実行できます。ゲームノードを何個実行させるかについての設定をすると、サーバーの実行時に自動的に設定ファイルを読み込んで決められた数のノードを起動するようになっています。テンプレート設定にはゲームノードを1つ起動するように設定されているので、このまま使ってください。追加修正する部分はgame部分のserviceNameとchannelIDsです。

GameAnvilConfig.jsonファイルのgameの最後の部分を見ると、Todoと書かれた部分があります。ここを修正してサービス名とチャンネル情報を設定します。

```json
  "game": [
    {
      "nodeCnt": 1,
      "serviceId": 1,
      "serviceName": "Todo - Input My Service Name",
      "channelIDs": ["ToDo - Input My ChannelName","ToDo - Input My ChannelName"], // ノードごとに付与するチャンネルID(一意である必要はありません。""はチャンネルを使用しないことを意味)
      "userTimeout": 5000 // クライアントの接続が切断された後、ユーザーオブジェクトをサーバーから削除せずに管理する時間を設定
    }
  ]
```

### サービスについて

サービスとは、1つのサーバーが複数のゲームを提供する場合、各ゲームサービスを区別して呼ぶ名前です。サービス名は特定のサービスを表すサーバーとクライアントの間で約束された文字列です。後々の過程でサービス名を入力する時に使うので覚えておく必要があります。

ここではSyncという名前のサービスを使います。 game部分のserviceNameに下記のように内容を修正します。

```
"serviceName" : "Sync",
```

### チャンネルについて

チャンネルは単一サーバー群を論理的に分ける方法の1つです。例題ではチャンネルを使わないので、この文書では詳しい説明は省略します。チャンネルを使わないので、game部分のchannelIDsに下記のように内容を修正します。

```
"channelIDs" : [""],
```

このように作成したGameAnvilサーバー設定ファイルの内容は次のとおりです。

```json
"game": [
    {
      "nodeCnt": 1,
      "serviceId": 1,
      "serviceName": "Sync",
      "channelIDs": [""], // ノードごとに付与するチャンネルID。(一意である必要はありません。""はチャンネルを使用しないことを意味)
      "userTimeout": 5000 // クライアントの接続が切れた後、ユーザーオブジェクトをサーバーから削除せずに管理する時間を設定
    }
  ]
```

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_config_json_1213.png)

参考までにgatewayの設定を見ると、TCP_SOCKETコネクションは18200ポートを使うように設定されていることが確認できます。これはクライアントと接続するポートで、今後クライアントプロジェクトでサーバー接続情報を記入する部分でこのポート番号を使うことになります。

## GameAnvilサーバー駆動

### Javaバージョン設定

GameAnvilはJava 8バージョンと11バージョンをサポートします。バージョンによって一部設定方法が違う場合があり、ここではJava 11バージョンを使用しました。

まず、jdkの設定を確認します。左上のメニューから**File > Project Structure**を選択して**Project Structure**ウィンドウを開きます。Macユーザーの場合、**Command + ;**ショートカットキーを使用できます。

**Project**タブでSDKの設定を確認します。もし、SDKが設定されていない場合は、**Add SDK > Download JDK**で希望のバージョンのJDKをダウンロードして設定します。**Language level**は**SDK default**に設定します。次に**Modules**タブで**Language level**を**Project default**に設定します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/project_structure_1213.png)

**設定**メニューで**gradle**で使うJVMを確認します。**プロジェクト**SDK**と同じ**gradle**バージョンに設定します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gradle_sdk_config_1213.png)

### サーバー駆動

実行設定が完了したら、Mainクラスのmain()関数の左側の緑色の三角形アイコンをクリックしてMain.main()実行を選択します。このように一度実行した後は、IntelliJ右上の緑色の三角形のRunアイコンをクリックしてもサーバーが実行されます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run1_1213.png)

build.gradleには便利なJVMオプションがあらかじめ設定されています。このような設定を活用してサーバーを実行するには、IntelliJのGradleウィンドウでTask > others > runMainを右クリックした後、GameAnvilTutorial実行をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run2_1213.png)

サーバーが正常に駆動されると、サーバー駆動状態に関するログが多数出力されます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run_log.png)

GameAnvilサーバーは複数のノードで構成されています。これらのノードはサーバーが実行する機能を複数の役割で分担します。まだ、サーバーの初期駆動を確認しただけで、ノードや他のサーバーを駆動するためのコードを作成していないため、完全に準備された状態ではありません。

それぞれのノードはコードを実行するために準備する時間が必要で、各ノードが準備完了したらonReadyログを出力します。クライアントがサーバーへ接続するのに直接的な役割をするノードはゲートウェイノードです。ゲートウェイノードが準備されてGatewayNodeのonReadyログが出力されたら、GameAnvilサーバーはいつでも接続可能な状態になったことになります。

次の章ではGameAnvilの様々なノードのうち、サンプルゲームの動作に必要なBasicGameNodeを実装してみます。

## GameAnvilサーバー機能の実装

### ゲームノードの実装

GameAnvilは`Base-`という接頭辞をつけた複数のノードクラスを提供しています。基本的なノードの機能はエンジン内部にすでに実装されており、ユーザーはこのBaseクラスを継承して様々なコールバック機能を使うことができます。今回の例では、BaseGameNodeクラスを継承したゲームノードクラスを作成して使ってみます。

プロジェクトパネルでMainクラスがあるパスを右クリックした後、**New** > Package**を選択して**node**という名前の新しいパッケージを作成します。そして、nodeパッケージをもう一度右クリックした後、**New** > BaseGameNode**を選択します。ファイル作成ダイアログが開いたら、**File name**に**SyncGameNode**、**Service Name**に**Sync**を入力し、**OK**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/create_sync_game_node.png)

この機能は先にテンプレートをインストールする時、File templates(schemes)を一緒に適用したため、使うことができます。**New > BaseGameNode**の項目が見えない場合、**New > Java Class** を選択して空のクラスを作成します。

自動的に作成されたコードは次のとおりです。

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.BaseGameNode;
import com.nhn.gameanvil.node.game.data.BaseChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.BaseChannelUserInfo;
import com.nhn.gameanvil.node.game.data.ChannelUpdateType;
import com.nhn.gameanvil.packet.Payload;
import com.nhn.gameanvil.packet.message.MessageDispatcher;

@ServiceName("Sync")
public final class SyncGameNode extends BaseGameNode {

    private static final MessageDispatcher<SyncGameNode> messageDispatcher = new MessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public MessageDispatcher<SyncGameNode> getMessageDispatcher() {
        return messageDispatcher;
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
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, BaseChannelUserInfo baseChannelUserInfo, int userId, String accountId) throws SuspendExecution {

    }

    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, BaseChannelRoomInfo baseChannelRoomInfo, int userId) throws SuspendExecution {

    }

    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {

    }
}
```

### ノードについて

全てのノードは何かの処理を開始できるループが開始されたかどうかによって状態を持ちます。以下はノードが持つことができる状態の一部です。

* INIT
* PREPARE
* READY
* SHUTDOWN

ノードはINIT状態から始まり、記載されている順に状態を変えながらREADY状態に到達します。READY状態は、ノードが与えられたタスクを処理して実行できる状態であることを示します。

自動作成されたコードには、各ノードの状態にフックされたコールバックをオーバーライドするコードが含まれています。例えば、onInit()メソッドに特定のロジックを記述すると、ノードが初期化(Init)を開始する直前の段階でそのコールバックが挿入され、呼び出されます。

GameAnvilはほとんどのコードがあらかじめ用意されているため、この段階で追加で作成するコードはありません。作成したままゲームノードを使うことができます。

### ユーザータイプについて

各ゲームノードでルームに参加してパケットを送受信する主体がユーザーですが、各ユーザー実装を区別する約束された文字列です。

GameAnvilで提供されるルームベースの実装を使用するためには、上記で実装したノードの他に**ゲームユーザー**と**ゲームルーム**クラスが必要です。クラスの継承とアノテーションを付けるだけで簡単に実装する方法を説明します。

### ゲームユーザーの実装

クライアントがサーバーにログインすると、サーバーはそのクライアントの情報を**ゲームユーザー**というオブジェクトを作成してメモリに保存します。ゲームユーザーがどのような情報を表現するかは、ユーザーのニーズに応じて自由に実装できます。ゲームユーザーの実装も、クラスの継承とコールバックのオーバーライドを通じて一貫して実装できます。

プロジェクトパネルでMainクラスが位置するパスを右クリックした後、**New > Package**を選択して**user**という名前の新しいパッケージを作成します。そして、**user**パッケージをもう一度右クリックした後、**New > BaseUser**を選択します。ファイル作成ダイアログが開いたら、**File name**に**SyncGameUser**、**Service Name**に**Sync**、**UserType**に**USER_TYPE_SYNC**を入力し、**OK**をクリックします。

ユーザータイプは、各ユーザーの実装を区別するサーバーとクライアントの間で約束された文字列であり、以後、クライアントプロジェクトの実装時にユーザータイプを使用しなければならないので、覚えておいてください。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/new-game-user-server.png)

自動的に作成されたコードは次のとおりです。

```java

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.annotation.UserType;
import com.nhn.gameanvil.node.game.BaseUser;
import com.nhn.gameanvil.node.game.data.RoomMatchResult;
import com.nhn.gameanvil.packet.Payload;
import com.nhn.gameanvil.packet.message.MessageDispatcher;
import com.nhn.gameanvil.serializer.TransferPack;

@ServiceName("Sync")
@UserType("USER_TYPE_SYNC")
public final class SyncGameUser extends BaseUser {

    private static final MessageDispatcher<SyncGameUser> messageDispatcher = new MessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public MessageDispatcher<SyncGameUser> getMessageDispatcher() {
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
    public boolean onReLogin(Payload payload, final Payload sessionPayload, Payload outPayload) throws SuspendExecution {
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

ゲームユーザーはクライアントがサーバーへログインリクエストを行うことで作成されます。サーバーでは、クライアントから送信されたペイロードなどを通じてログインを許可するかどうかを判断し、戻り値としてエクスポートできます。主なロジックのみエンジンユーザーが作成し、ログインの成功や失敗の処理はエンジンが担当します。

このチュートリアルでは、特別な検証プロセスなしでログインを許可するために、onLogin関数で常にtrueを返すようにしました。このようにすると、クライアントからログイン要求があった時、常にユーザーオブジェクトを作成して成功レスポンスを与えることになります。

### ゲームルームの実装

正常にゲームユーザーとしてゲームノードに接続したら、他のユーザーとゲームルームを通じてパケットを送受信できます。ゲームルームとはパケットを送受信するユーザーを論理的に束ねるグループです。ゲームルームの実装もクラス継承とコールバックのオーバーライドで実装できます。

プロジェクトパネルでMainクラスが位置するパスを右クリックし、**New** > Package**を選択し、**room**という名前の新しいパッケージを作成します。そして、**room**パッケージをもう一度右クリックし、**New** > BaseRoom**を選択します。ファイル作成ダイアログが開いたら、**File name**に**SyncGameRoom**、**Service Name**に**Sync**、**Room Type**に**ROOM_TYPE_SYNC**、**User Class**に**SyncGameUser**を入力し、**OK**をクリックします。

ルームタイプは、各ルームの実装を区別するサーバーとクライアントの間で約束された文字列で、以後、クライアントプロジェクトの実装時にルームタイプを入力する必要があるので、覚えておいてください。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/new_game_room_server.png)

自動的に作成されたコードは次のとおりです。

```java

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.BaseRoom;
import com.nhn.gameanvil.node.game.RoomMessageDispatcher;
import com.nhn.gameanvil.packet.Payload;
import com.nhn.gameanvil.serializer.TransferPack;

import java.util.List;

@ServiceName("Sync")
@RoomType("ROOM_TYPE_SYNC")
public final class SyncGameRoom extends BaseRoom<SyncGameUser> {

    private static final RoomMessageDispatcher<SyncGameRoom, SyncGameUser> messageDispatcher = new RoomMessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public RoomMessageDispatcher<SyncGameRoom, SyncGameUser> getMessageDispatcher() {
        return messageDispatcher;
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onDestroy() throws SuspendExecution {
    }

    @Override
    public boolean onCreateRoom(SyncGameUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public boolean onJoinRoom(SyncGameUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public boolean onLeaveRoom(SyncGameUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public void onLeaveRoom(SyncGameUser sampleUser) throws SuspendExecution {

    }

    @Override
    public void onPostLeaveRoom() throws SuspendExecution {

    }

    @Override
    public void onRejoinRoom(SyncGameUser user, Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean canTransfer() throws SuspendExecution {
        return true;
    }

    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {

    }

    @Override
    public void onTransferIn(List<SyncGameUser> userList, TransferPack transferPack) throws SuspendExecution {
    }

    @Override
    public void onPause() throws SuspendExecution {

    }

    @Override
    public void onResume() throws SuspendExecution {

    }
}
```

ゲームルームはゲームユーザーがサーバーにルームの作成をリクエストすると作成されます。クライアント側では単純にメソッドを呼び出すだけでルームを作成し、存在するルームに入室することができます。ユーザーがルームに入室する時点またはルームが作成される時点でカスタムコードを挿入したい場合は、適切なコールバックをオーバーライドして簡単にコードを挿入できます。

## サーバーの実装を終えて

ここまでで基礎チュートリアルのサンプル実行のためのサーバー構築が完了しました。再度サーバーを実行してみると、ログの中に`{"message":"All nodes are ready!!"}`という文言が表示されます。このログが出たということは、GameAnvilサーバーが正常に実行されたことを意味します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/all_nodes_are_ready.png)

これでクライアントのリクエストを受けるサーバーの準備ができました。次の段階ではGameAnvilコネクタとUnityサンプルプロジェクトを活用してクライアントを実装してみます。

## 実習環境の準備 - クライアントプロジェクト

下記の段階を進めながら修正が完了する最終的なクライアントサンプルプロジェクトは下記のリンクからダウンロードできます。Unityパッケージをダウンロードして構成した初期Unityプロジェクトで様々な段階を経てクライアント機能を実装した後、最終的にどのような構造になるのか事前に確認したい場合は、該当プロジェクトをダウンロードして参考にしてください。

[最終クライアントサンプルプロジェクトのダウンロード](https://static.toastoven.net/prod_gameanvil/files/tutorial/basic-tutorial/BasicSyncTutorial.zip?disposition=attachment)

### GameAnvilConnectorのダウンロード

GameAnvilコネクタdllを使うため下記のファイルをダウンロードします。

[GameAnvil-Connector.unitypackage](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-2.1.0.unitypackage)

### Unity Packageをダウンロード

GameAnvilコネクタの使用実習のため、下記のリンクからUnityパッケージをダウンロードします。

[GameAnvil-Tutorial-Sample.unitypackage](https://static.toastoven.net/prod_gameanvil/files/tutorial/basic-tutorial/GameAnvil-Tutorial-Sapmple.unitypackage)

### Unityプロジェクトの作成

Unityハブを実行した後、右上のNew Projectボタンをクリックします。Unityハブのバージョンは関係ありません。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/unity-hub.png)

テンプレートとして**2D**を選択し、プロジェクト名と保存場所を確認した後、**Create project**をクリックします。この例で使用したUnityのバージョンは2020.3.37f1であり、他のバージョンを使用しても構いませんが、すべての場合をテストしたわけではないので、サンプル実行バージョンと同じ環境で進めることを推奨します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/new-unity-project.png)

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/new-unity-project-done.png)

### GameAnvilConnector及びUnity Packageインポート

プロジェクトビューを右クリックし、**Import Package > Custom Package...** を選択し、ファインダーやファイルエクスプローラーが開いたら、前段階でダウンロードしたUnityパッケージを選択します。GameAnvilConnector、Tutorial-Sampleの順にImportを実行します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/import-unity-package-connector.png)

GameAnvilSampleフォルダ内のSceneフォルダでIntroSceneを開いて下記のような画面を確認します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/unity-after-import-package.png)

**File > Build Settings**で**Add Open Scene**をクリックしてビルド時に含まれるように設定します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/intro_scene_to_build_settings.png)

## GameAnvilConnectorの追加

Hierarchyビューで右クリックし、**GameAnvil > GameAnvilConnector**をクリックします。GameAnvilConnectorゲームオブジェクトが作成され、GameAnvilConnectorゲームオブジェクトのインスペクタ上で下記のように設定を修正できます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/add-gameanvilconnector-done.png)

* QuickConnect:クイック接続の進行状況を表示します。
* GameAnvil Connector Configuration:コネクタ関連の設定郡です。
* Connect Configuration:クイック接続の接続情報を修正できます。
* Authentication Configuration:クイック接続の認証情報を修正できます。
* Login Configuration:クイック接続のログイン情報を修正できます。
* LogListener: GameAnvilコネクタ内部で発生するログ出力を管理します。

今は詳細設定について詳しく知らなくても大丈夫です。チュートリアルを進めながら各項目の説明を確認できます。

## クイック接続

GameAnvilクライアントがGameAnvilサーバーに接続するためには、Connect、Authentication、Loginの3つのステップを経る必要があります。

* Connect:サーバーとクライアント間で通信できるようにソケットを作成して接続します。
* Auth:クライアントがサーバーを介してデータを送受信することを許可するかどうかをサーバーで決定します。
* Login:サーバーのメモリにクライアントの情報を表現するオブジェクト、つまり、ゲームユーザーを作成します。

各ステップは順番に行われ、前のステップが正常に完了しないと次のステップを進めることができません。各ステップの処理の成否は、コールバックで渡されたパラメータで値を取得できます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/connect-auth-login.gif)

ここでは、Hierarchyビュー上のCanvasゲームオブジェクトにコンポーネントとして追加されているQuickConnectUIManagerスクリプトをソースコードエディタで開き、実装を追加しながら各過程を直接実習します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/quickconnect_ui_manager.png)

### Connect関連フィールドの設定

接続するサーバー情報を記載します。ローカルで直接サーバーを立ち上げる場合なので、ipは`127.0.0.0.1`を使用します。 portはゲートウェイノードのデフォルトポートである`18200`を使用します。特に設定する必要はなく、GameAnvilConnectorのデフォルト値をそのまま使用します。ipとport情報は必要に応じてプレイモードで修正できるようにUnityのInputFieldと接続するコードが作成されていることが確認できます。

```c#
void Start()
{
    ipInputField.text = GameAnvilConnector.getInstance().ip;
    portInputField.text = GameAnvilConnector.getInstance().port.ToString();

    ...省略...

    ipInputField.onValueChanged.AddListener(delegate { ipChanged(); });
    portInputField.onValueChanged.AddListener(delegate { portChanged(); });
}

void ipChanged()
{
    GameAnvilConnector.getInstance().ip = ipInputField.text;
}

void portChanged()
{
    if (!int.TryParse(portInputField.text, out GameAnvilConnector.getInstance().port))
    {
        GameAnvilConnector.getInstance().port = 11200;
    }
}
```

### Authentication関連フィールド設定

認証に必要な情報を記載します。認証に必要な情報はaccountId、deviceId、passwordの3つがあります。今は認証段階を無条件に通過するようにサーバーが実装されている状態なので、どの値を設定しても動作に異常がないはずです。 GameAnvilConnectorの基本値である`test`を使うようにし、必要な場合は、プレイモードでUnityのInputFieldを使って入力された値を使えるようにコードが作成されていることが確認できます。

```c#
void Start()
{
    ...省略...
    
    accountIdInputField.text = GameAnvilConnector.getInstance().accountId;
    deviceIdInputField.text = GameAnvilConnector.getInstance().deviceId;
    passwordInputField.text = GameAnvilConnector.getInstance().password;

    ...省略...

    accountIdInputField.onValueChanged.AddListener(delegate { accountIdChanged(); });
    deviceIdInputField.onValueChanged.AddListener(delegate { deviceIdChanged(); });
    passwordInputField.onValueChanged.AddListener(delegate { passwordChanged(); });

    ...省略...
}

void accountIdChanged()
{
    GameAnvilConnector.getInstance().accountId = accountIdInputField.text;
}

void deviceIdChanged()
{
    GameAnvilConnector.getInstance().deviceId = deviceIdInputField.text;
}

void passwordChanged()
{
    GameAnvilConnector.getInstance().password = passwordInputField.text;
}
```

### Login関連フィールドの設定

ログインに必要な情報を入力します。ログインに必要な情報としては、ユーザータイプ、チャンネルID、そしてサービス名があります。サーバーを実装する時に作成したユーザータイプとサービス名を使う必要があります。基本的に実行される時、サーバーで指定した値を適用するように作成します。必要な場合、UnityのInputFieldを使って値を修正できるように設定されています。

```c#
void Start()
{
    ...省略...

    userTypeInputField.text = GameAnvilConnector.getInstance().userType;
    channelIdInputField.text = GameAnvilConnector.getInstance().channelId;
    serviceNameInputField.text = GameAnvilConnector.getInstance().serviceName;

    ...省略...

    userTypeInputField.onValueChanged.AddListener(delegate { userTypeChanged(); });
    channelIdInputField.onValueChanged.AddListener(delegate { channelIdChanged(); });
    serviceNameInputField.onValueChanged.AddListener(delegate { serviceNameChanged(); });

    ...省略...
}

void userTypeChanged()
{
    GameAnvilConnector.getInstance().userType = userTypeInputField.text;
}

void channelIdChanged()
{
    GameAnvilConnector.getInstance().channelId = channelIdInputField.text;
}

void serviceNameChanged()
{
    GameAnvilConnector.getInstance().serviceName = serviceNameInputField.text;
}
```

### クイック接続リクエストAPI呼び出し

クイック接続リクエストはGameAnvilConnectorを使って下記のようにリクエストできます。

```c#
GameAnvilConnector.getInstance().QuickConnect(DelOnQuickConnect);
```

クイック接続が終わったら、リクエスト時に渡したデリゲートを通じて結果を知ることができます。

```c#
void DelOnQuickConnect(GameAnvilConnector.ResultCodeQuickConnect resultCode, UserAgent userAgent, GameAnvilConnector.QuickConnectResult quickConnectResult)
{
    Debug.Log(resultCode);
}
```

ボタンをクリックしてクイック接続をリクエストできるように、QuickConnectメソッドを下記のように実装します。例題では基本的に内容が書かれているので、コメント処理だけ解除します。

```c#
public void QuickConnect()
{
    GameAnvilConnector.getInstance().QuickConnect(DelOnQuickConnect);
    state = UIState.QUICK_CONNECTING;
}
```

クイック接続が終わったら、UIの状態を切り替えるため下記のようにDelOnQuickConnectメソッドを修正します。例題では基本的に内容が作成されているので、コメント処理だけ解除します。

```c#
void DelOnQuickConnect(GameAnvilConnector.ResultCodeQuickConnect resultCode, UserAgent userAgent, GameAnvilConnector.QuickConnectResult quickConnectResult)
{
    if (quickConnectResult.resultCodeQuickConnect.Equals(GameAnvilConnector.ResultCodeQuickConnect.QUICK_CONNECT_SUCCESS))
    {
        state = UIState.QUICK_CONNECT_COMPLETE;
    }
    else
    {
        state = UIState.QUICK_CONNECT_FAIL;
    }
}
```

### クイック接続終了APIの呼び出し

クイック接続の終了も接続リクエストAPIと同じように下記のようにリクエストできます。

```c#
GameAnvilConnector.getInstance().QuickDisconnect();
```

ボタンを押してクイック接続終了APIを呼び出せるようにQuickDisconnectメソッドを下記のように実装します。例では基本的に内容が書かれているので、コメント処理だけ解除します。

```c#
public void QuickDisconnect()
{
    GameAnvilConnector.getInstance().QuickDisconnect();
    state = UIState.NOT_QUICK_CONNECTED;
}
```

### クイック接続の進行状況を出力

クイック接続の進行状況は下記のように読み込むことができます。

```c#
GameAnvilConnector.getInstance().GetQuickConnectState().ToString();
```

クイックコネクトの進行状況が常に分かるように画面に表示するコードが下記のようにUpdate関数に書かれています。

```c#
void Update()
{
    quickConnectResultText.text = GameAnvilConnector.getInstance().GetQuickConnectState().ToString();
}
```

### クイック接続に使用する値をGameAnvilConnectorに直接入力

サーバーの実装段階で使ったサービス名やユーザータイプの文字列値をクライアントでも同じように使う必要があります。
GameAnvilConnectorのインスペクタウィンドウでLogin Configurationに下記のようにUser TypeとService Nameをサーバーと同じ値に設定します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil-login-configuration.png)

### クイック接続リクエスト/接続終了テスト

サーバーが実行中であることを確認した後、Unityエディタでプレイモードに入ります。**Quick Connect** をクリックし、正常に接続が行われることを確認します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/quick-connect.gif)

クイック接続を試行すると、クイック接続ステータスウィンドウに下記のような順番でConnect, Authenticate, Loginのプロセスが行われます。

* NOT_CONNECTED
* CONNECT_IN_PROGRESS
* CONNECT_COMPLETE
* AUTHENTICATE_IN_PROGRESS
* AUTHENTICATE_COMPLETE
* LOGIN_IN_PROGRESS
* LOGIN_COMPLETE
* READY 

接続が完了したら、READYの状態となります。この状態で接続終了ボタンを押して、正常に接続が終了することを確認します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/quick-disconnect.gif)

## ゲームルームの作成と入室

### ゲームルーム作成リクエストAPIを使用

GameAnvilコネクタのルーム作成要求APIを呼び出すことで、クライアントは簡単にサーバーにゲームルームの作成をリクエストできます。ゲームルーム作成リクエストメソッドを呼び出す時、パラメータでルームタイプを渡さなければなりませんが、サーバーと事前に合意したルームタイム値を渡せばいいです。

```c#
GameAnvilConnector.getInstance().getUserAgent().CreateRoom("ROOM_TYPE_SYNC", DelOnCreateRoom);
```

ルーム作成リクエストの結果は、一緒に渡したデリゲートを通じて受け取ることができます。作成されたゲームルーム情報(ルームID、ルーム名など)も一緒に受け取ることができます。

```c#
public void DelOnCreateRoom(UserAgent userAgent, ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) {
    Debug.Log(result);
}
```

ボタンをクリックしてルームの作成を要求するため、CreateRoomメソッドを下記のように実装します。ドロップダウンで選択したルームタイプを使えるように設定しました。例では基本的に内容が作成されているので、コメント処理だけ解除します。

```c#
public void CreateRoom()
{ 
    GameAnvilConnector.getInstance().getUserAgent().CreateRoom(roomTypeDropdown.options[roomTypeDropdown.value].text, DelOnCreateRoom);
}
```

ルーム作成リクエスト結果を受け取る関数も下記のように修正してUIの状態を切り替えることができるようにします。作成されたゲームルームのIDを画面に表示するように設定します。例では基本的に内容が作成されているので、コメント処理だけ解除します。

```c#
public void DelOnCreateRoom(UserAgent userAgent, ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) {
    Debug.Log(result);
    if (result.Equals(GameAnvil.Defines.ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)){
        state = UIState.JOIN_ROOM_COMPLETE;
        RoomIdText.text =  roomId.ToString();
    }
    else
    {
        state = UIState.JOIN_ROOM_FAIL;
    }
}
```

ゲームルーム作成機能の実装が終わりました。テストは少し後回しにして、ゲームルーム入室機能を先に実装します。

### ゲームルーム入室リクエストAPIを使う

サーバーにゲームルームが作成されたとします。そのルームに接続するためには、GameAnvilコネクタでゲームルーム入室リクエストメソッドを呼び出します。この時、ルーム作成時に渡されたゲームルームIDを渡します。

```c#
GameAnvilConnector.getInstance().getUserAgent().JoinRoom("ROOM_TYPE_SYNC", {ルームID入力});
```

現在ルームにユーザーが入室しているかどうかは下記のようにIsJoinedRoom()メソッドで確認します。

```c#
GameAnvilConnector.getInstance().getUserAgent().IsJoinedRoom()
```

ボタンクリックでJoinRoomリクエストができるように、既存のJoinRoomメソッドに下記のように実装を追加します。ルームIDは入力フィールドに入力された値を使うようにしました。例題では基本的に内容が書かれているので、コメント処理だけ解除します。

```c#
public void JoinRoom()
{
    int roomId = 0;
    int.TryParse(joinRoomIdInputField.text, out roomId);
    GameAnvilConnector.getInstance().getUserAgent().JoinRoom(roomTypeDropdown.options[roomTypeDropdown.value].text, roomId);
}
```

これでゲームルームの作成機能と入室機能が全て完成しました。

### ゲームルームのテスト

Unityエディタでショートカットキー`CMD + b`または`Ctrl + b`を押してビルドします。ビルド結果が表示されたウィンドウでボタンをクリックしてゲームルームが作成されたことを確認します。ゲームルームが作成されると画面にゲームルームのIDが表示されます。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/create-room.gif)

その後、Unityエディタでプレイモードに入ります。以前に表示されたゲームルームのIDを入力フィールドに入力し、**Join Room**をクリックしてゲームルームに参加するか確認します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/join-room.gif)

## 同期コントローラー入門

これで、同じゲームルームに接続したゲームユーザー間では、パケットを送受信できます。このパケットを通して必要な情報をクライアントプロセス間で同期するようにコードを書くことができます。もっと簡単な方法としては、同期したいゲームオブジェクトに同期コンポーネントを付けるだけで同期を実装できます。

### 同期コントローラーの追加

Hierarchyビューで右クリックし、**GameAnvil > SyncController**を選択します。

この例ではシーン移動が行われるため、シーン移動後に手動で同期オブジェクトを作成するため、SyncControllerオブジェクトのインスペクタ上でInstantiate Sync Object Immediatlyのチェックを外します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/add-syn-controller-done.png)

これでGameAnvilの全ての同期機能を使うことができます。次は、一番単純な例題を使って同期コンポーネントの取り付けや使い方を説明します。

### 同期オブジェクトの作成

プロジェクトビューでResourcesフォルダ内に移動した後、Anvilプレハブをダブルクリックしてプレハブの修正画面に切り替えます。インスペクタウィンドウで、AddComponentボタンをクリックし、`GameAnvil > GameAnvil Sync > TransformSync`をクリックします。これで、このプレハブはゲームユーザー間でゲームオブジェクトのTransform情報を同期する準備が整いました。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/anvil-add-sync-component-done.png)

完成した同期ゲームオブジェクトプレハブをUnityプレイモードで使用するには、SyncControllerが提供するゲームオブジェクト作成APIを使用してゲームオブジェクトを作成し、シーンに追加します。最初の引数としてプレハブの名前を渡す必要があります。

注意点は、GameAnvilが提供する同期コンポーネントを付けた同期ゲームオブジェクトは、Unityが基本的に提供するGameObject.Instantiate()メソッドではなく、GameAnvilが提供するSyncControllerのInstantiate APIを使用しなければ、正常な同期ができません。

```c#
SyncController.Instance.Instantiate("Anvil", new Vector3(0, 0, 0), Quaternion.identity);
```

### ゲームシーンの作成

具体的な使用例を見るために、プロジェクトの**View > GameAnvilSample > Scene**フォルダでSpawnAnvilシーンを開きます。そして、** File > Build Settings**メニューで**Add Open Scene**をクリックして、ビルド時に含まれるように設定します。

SpawnAnvilSampleゲームオブジェクトに割り当てられているコンポーネントのSpawnAnvilSampleスクリプトを修正して実装を追加します。例では基本的に内容が作成されているので、コメント処理だけ解除します。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace GameAnvil
{
    public class SpawnAnvilSample : MonoBehaviour
    {
        void Start()
        {
            SyncController.Instance.InstantiateSyncObject();
        }

        void Update()
        {
            if (Input.GetMouseButtonDown(0))
            {
                Vector3 mousePos = Input.mousePosition;
                mousePos.z = Camera.main.nearClipPlane;
                Vector3 worldPosition = Camera.main.ScreenToWorldPoint(mousePos);

                SyncController.Instance.Instantiate("Anvil", worldPosition, Quaternion.identity);
            }

            if (GameAnvilConnector.getInstance() == null || GameAnvilConnector.getInstance().getUserAgent() == null || !GameAnvilConnector.getInstance().getUserAgent().IsJoinedRoom())
            {
                SceneManager.LoadScene("IntroScene");
            }
        }
    }

}
```

Start関数では、シーン移動直後に同期を開始するためにInstantiateSyncObject()を実行します。

Update関数では、クリックするたびにマウスの座標を取得し、前段階で修正したプレハブを作成するようにしました。接続が切れた場合に備えて再接続できるようにするため、元のシーンに移動するようにしました。

### 同期テスト

Unityエディタで`CMD + b`または`Ctrl + b`ショートカットを押してビルドします。ビルド結果ウィンドウでゲームルームを作成し、Spawn Anvilボタンを押してIntroSceneからSpawnAnvil Sceneにシーンを移動します。

その後、Unityエディタでプレイモードに入ります。ビルド結果と同じゲームルームに接続した後、Spawn Anvilボタンを押してIntroSceneからSpawnAnvil Sceneにシーンを移動します。その後、画面の任意の場所をクリックして新しい同期ゲームオブジェクトを作成します。一方のクライアントでゲームオブジェクトを作成すると、同じルームに入った他のクライアントの画面にも同じように表示されることを確認します。

<video src="https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/synchronize-create-done.mov" controls="controls" autoplay style="max-width: 640px;
  display: block;
  margin: auto;">
</video>

## 同期コントローラーの深化

前段階では、オブジェクト作成の同期を確認しました。ここでは、より複雑な例であるゲームオブジェクトのRigidbodyの同期について説明します。前の例を実装したときと同じ手順で実装を完了します。

### 同期オブジェクトの作成

プロジェクトビューでResourcesフォルダ内に移動し、Playerプレハブをダブルクリックしてプレハブの修正画面に切り替えます。インスペクタウィンドウで**AddComponent**をクリックした後、**GameAnvil > GameAnvil Sync > TransformSync**をクリックします。該当プレハブのゲームユーザー間でゲームオブジェクトのTransform情報を同期する準備が完了します。

プロジェクトビューで**Resources > Player**をダブルクリックしてプレハブの修正画面に切り替えます。インスペクタウィンドウで**AddComponent**をクリックし、**GameAnvil > GameAnvil Sync > RigidBodySync**をクリックします。該当プレハブのゲームユーザー間でゲームオブジェクトのRigidbody情報を同期する準備が完了します。

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/add-rigidbody-sync-done.png)

### ゲームシーンの作成

プロジェクトビューでGameAnvilSampleフォルダ内のSceneフォルダからKeyboardToMoveシーンを開きます。そして、**File > Build Settings**メニューから**Add Open Scene** をクリックして、ビルド時に含まれるように設定します。

KeyboardToMoveSampleゲームオブジェクトに割り当てられているコンポーネントのKeyboardToMoveSampleスクリプトを修正して実装を追加します。例では基本的に内容が作成されているので、コメント処理だけ解除します。

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Animations;
using GameAnvil;
using UnityEngine.SceneManagement;

public class KeyboardToMoveSample : MonoBehaviour
{
    private static KeyboardToMove player;
    private float moveForce = 10;
    public Transform playerPointer;

    // Start is called before the first frame update
    void Start()
    {
        SyncController.Instance.InstantiateSyncObject();
    }

    // Update is called once per frame
    void Update()
    {
        if (GameAnvilConnector.getInstance() == null || GameAnvilConnector.getInstance().getUserAgent() == null || !GameAnvilConnector.getInstance().getUserAgent().IsJoinedRoom())
        {
            SceneManager.LoadScene("IntroScene");
        }

        if (player != null)
        {
            playerPointer.position = player.transform.position;
            player.GetComponent<Rigidbody>().AddForce(Vector3.right * Input.GetAxis("Horizontal") * moveForce, ForceMode.Force);
            player.GetComponent<Rigidbody>().AddForce(Vector3.up * Input.GetAxis("Vertical") * moveForce, ForceMode.Force);
        }
    }

    public static void SpawnPlayer()
    {
        Vector3 mousePos = Input.mousePosition;
        Vector3 worldPosition = Camera.main.ScreenToWorldPoint(mousePos);
        worldPosition.z = 0;

        player = SyncController.Instance.Instantiate("player", worldPosition, Quaternion.identity).GetComponent<KeyboardToMove>();
    }

    public static void SetSelected(KeyboardToMove selected)
    {
        if ( player == selected)
        {
            player = null;
            GameObject.Destroy(selected.gameObject);
        }
        else
        {
            player = selected;
        }
    }
}
```
Start関数では、シーン移動直後に同期を開始するためにInstantiateSyncObject()を実行します。

SpawnPlayer関数は呼び出されるたびにマウスの位置に新しいPlayerオブジェクトを作成します。

Update関数はルームの入室とログインが維持されていることを確認し、最後に作成したオブジェクトをキーボードで操作できるようにします。キー入力によって剛体にAddForce()関数を実行して位置を更新するように誘導します。

### 同期テスト

Unityエディタでショートカットキー`CMD + b`または`Ctrl + b`を押してビルドします。ビルド結果ウィンドウでゲームルームを作成し、**KeyboardToMove**をクリックしてIntroSceneからKeyboardToMove Sceneに移動します。

その後、Unityエディタでプレイモードに入ります。ビルド結果と同じゲームルームに接続し、**KeyboardToMove**をクリックしてIntroSceneからKeyboardToMove Sceneに移動します。その後、画面の任意の場所をクリックして新しい同期ゲームオブジェクトを作成し、キーボードでゲームオブジェクトの位置を移動すると、他のクライアントでも反映されることを確認します。

<video src="https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/synchronize-rigidbody-done.mov" controls="controls" autoplay style="max-width: 640px;
  display: block;
  margin: auto;">
</video>

## チュートリアルの終わりに

この文書では、GameAnvilコネクタの便利な機能である簡易接続と同期機能について実習を通じて学びました。チュートリアルの冒頭で紹介したように、GameAnvilにはゲームサーバーの構築に必要なすべての機能が用意されており、チュートリアルではその一部だけを軽く取り上げました。次の文書でより詳しい使い方を学ぶことができます。
