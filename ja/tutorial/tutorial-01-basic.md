## Game > GameAnvil > 基礎チュートリアル

### GameAnvilでマルチプレイヤーゲームを簡単に作成する

GameAnvilはリアルタイムマルチプレイヤーゲームサーバー制作プラットフォームです。
GameAnvilを使用すると、手軽にゲームサーバーとクライアントを開発・運用できます。

このドキュメントでは、GameAnvilの基本的な機能を利用して、実際にプレイ可能なマルチプレイヤー同期ゲームを開発する過程を扱います。
サーバーの概念とAPIを単に羅列するのではなく、直接マルチプレイヤーゲームサーバー及びサンプルクライアントを開発することで、GameAnvilの基本概念とプロジェクト構成、及び実装方法を自然に習得できるようにしました。

GameAnvilはサーバーエンジンだけでなく、クライアントをサーバーに接続するのを支援するコネクタも提供します。サーバーとクライアントが相互作用する様子を確認できるサンプルを完成させながら、GameAnvilを使用してゲームを開発する全体的な流れに慣れることができます。

## 実習環境の準備 - サーバープロジェクト

マルチプレイヤーゲームを作成するには、クライアントと対応するサーバープログラムが必要です。ゲームサーバーを構築した後、続いてクライアントを実装する方式でチュートリアルが進行します。

この例では、クライアントプログラムの制作にUnityとGameAnvilコネクタを使用し、サーバープログラムの制作に先ほど紹介したサーバーエンジンGameAnvilを使用します。まずGameAnvilを利用したサーバープログラムプロジェクトを作成してみます。

以下の手順を進めると作成される最終的なサーバーサンプルプロジェクトは、以下のリンクからダウンロードできます。初期テンプレートからいくつかの段階を経てサーバー機能を実装するとどのような構造になるかあらかじめ確認したい場合は、該当プロジェクトをダウンロードして参考にしてください。

[サーバーサンプルプロジェクトのダウンロード](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil_Tutorial_Basic_Server.zip?disposition=attachment)

### プロジェクト構成

今回のチャプターでは、開発を開始するために初期設定を完了することを目標とします。実際のプロセスを実行してサーバーを起動することは、次のチャプターで扱います。

例ではサーバープロジェクトIDEとしてJetBrain社のIntelliJを使用します。例で使用したIntelliJのバージョンはIDEA Ultimate 2024.2.1です。ライセンスを購入していない場合は、IntelliJ IDEA Community Editionを使用しても問題ありません。他のバージョンのIntelliJを使用しても問題なく動作すると予想されますが、全てのケースをテストしたわけではないため、サンプル実行バージョンと同じ環境で進めることを推奨します。

プロジェクトにGameAnvilを適用するには、MavenリポジトリからGameAnvilライブラリをダウンロードし、GameAnvilを駆動するために必須の設定ファイルを作成する必要があります。最後に若干のボイラープレートコードを作成すれば、開発初期設定が完了します。

GameAnvilでは、このような一連の過程を代わりに行ってくれるIntelliJテンプレートを提供しており、より簡単に初期作業を完了できます。次のリンクからIntelliJ用プロジェクトファイルテンプレートをダウンロードできます。ダウンロードしたテンプレートは解凍しないでください。

[テンプレートのダウンロード](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil%20Template.zip?disposition=attachment)

ダウンロードしたテンプレートを適用するためにIntelliJを実行します。**Welcome to IntelliJ IDEA**画面の左側メニューで**Customize**を選択した後、**Import Settings...**をクリックします。または全体検索ウィンドウで**Import Settings...**を検索します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/1_import_gameanvil_template.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/2_search_import_settings.png)

<br>

Finderまたはファイルエクスプローラーウィンドウでテンプレートをダウンロードしたパスへ移動し、圧縮ファイルを選択します。**Select Components to Import**ウィンドウが開いたら、**File templates**項目と**Project Templates**項目の両方をチェックして選択します。**OK**をクリックした後、**Import and Restart**をクリックするとIntelliJが再起動し、テンプレートの適用が完了します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/3_select_import.png)

IntelliJ右上のボタングループで**New Project**をクリックした後、左側のリストをスクロールして下段の**Templates**にある**GameAnvil 2.1.0 Template**を選択します。プロジェクト名を設定します。名前にスペースを含めることはできません。プロジェクトの場所とベースパッケージ名を確認した後、プロジェクトを作成します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/4_imported_gameanvil_template.png)

これでIntelliJにサーバープロジェクトの骨格が構成されました。Projectパネルを見ると、コードと設定ファイルが生成されたことを確認できます。

- Main: プログラムのエントリーポイントであるMain関数を含むクラスです。
- GameAnvilConfig.json: GameAnvilの駆動に必要なサーバー設定情報を記録したファイルです。サーバーの実装に合わせて修正できます。
- logback.xml: Javaプロジェクトでロギングを構成するために使用されるファイルです。Logbackフレームワークの設定ファイルとして,ロギングシステムの動作方式とログの形式,保存場所などを指定します。このファイルを使用してロギングレベル,ログ形式,ログファイルのパス及び名前,ログローテーションポリシーなどを設定できます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/5_gameanvil_project_view_init.png)

## GameAnvilサーバー設定ファイルの修正

プロジェクトパネルのresourcesパッケージ配下にあるGameAnvilConfig.jsonファイルを通じて、GameAnvilサーバー設定を変更できます。

- common: サーバー全般の設定を扱う部分
- location: ロケーションノード関連の設定を扱う部分
- match: マッチノード関連の設定を扱う部分
- gateway: ゲートウェイノード関連の設定を扱う部分
- game: ゲームノード関連の設定を扱う部分

テンプレートを通じてプロジェクトを構成したため、GameAnvilConfig.jsonファイルにサーバー動作に必要な基本設定情報がセットされていることを確認できます。この例で注意深く見るべき部分は大きく3つです。

1. gameのnodeCnt値
2. gameのserviceName値
3. gameのchannelIDs値

ゲームノードは必要量に応じて、またはサーバーの性能に応じて複数のVMで構成して実行できます。ゲームノードをいくつ実行させるか設定すると、サーバー実行時に自動的に設定ファイルを読み込み、指定された個数のノードを立ち上げるようになっています。テンプレート設定ではゲームノードを1つ立ち上げるようにセットされており、このまま使用すれば問題ありません。追加修正する部分はgame部分のserviceNameとchannelIDsです。

GameAnvilConfig.jsonファイルのgame側の最後を見ると、Todoと表示された部分があります。ここを修正してサービス名とチャンネル情報を設定してみます。

```json
  "game": [
    {
      "nodeCnt": 1,
      "serviceId": 1,
      "serviceName": "Todo - Input My Service Name",
      "channelIDs": [["ToDo - Input My ChannelName"]], // ノードごとに付与するチャンネルID。(一意でなくても良い。""はチャンネルを使用しないことを意味)
      "userTimeout": 5000 //クライアントの接続断後、ユーザーオブジェクトをサーバーから削除せずに管理する時間を設定
    }
  ]
```

### サービスについて

サービスとは、1つのサーバーが複数のゲームを提供する場合、各ゲームサービスを区別して呼ぶ名前です。サービス名は特定サービスを表すサーバーとクライアント間で約束された文字列です。以降の過程でサービス名を入力する際に使用するため、覚えておく必要があります。

ここではSyncという名前を持つサービスを使用します。game部分のserviceNameを以下のように修正します。

```
"serviceName" : "Sync",
```

### チャンネルについて

チャンネルは単一サーバー群を論理的に分割できる方法の1つです。例ではチャンネルを使用しないため、このドキュメントでは詳細な説明を省略します。チャンネルを使用しないため、game部分のchannelIDsを以下のように修正します。

```
"channelIDs" : [[""]],
```

このように作成完了したGameAnvilサーバー設定ファイルの内容は次のとおりです。

```json
"game": [
    {
      "nodeCnt": 1,
      "serviceId": 1,
      "serviceName": "Sync",
      "channelIDs": [[""]], // ノードごとに付与するチャンネルID。(一意でなくても良い。""はチャンネルを使用しないことを意味)
      "userTimeout": 5000 //クライアントの接続断後、ユーザーオブジェクトをサーバーから削除せずに管理する時間を設定
    }
  ]
```

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/6_gameanvil_config_json.png)

参考までに、gateway設定を見るとTCP_SOCKETコネクションは18200ポートを使用するように設定されていることが確認できます。これはクライアントと接続されるポートで、以降クライアントプロジェクトでサーバー接続情報を記入する部分でこのポート番号を使用することになります。

## GameAnvilサーバー起動

### Javaバージョン設定

GameAnvilはJava 21バージョンをサポートします。バージョンによって一部の設定方法が異なる場合があり、ここではJava 21バージョンを使用しました。

まずJDK設定を確認します。左上のメニューから**File > Project Structure**を選択して**Project Structure**ウィンドウを開きます。Macユーザーの場合は**Command + ;**ショートカットキーを使用できます。

**Project**タブでSDK設定を確認します。もしSDKが設定されていない場合は、**Add SDK > Download JDK**を通じて希望するバージョンのJDKをダウンロードして設定します。**Language level**は**SDK default**に設定します。次に**Modules**タブで**Language level**を**Project default**に設定します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/7_project_structure.png)

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/8_module_language_level.png)

**設定** メニューで**gradle** 設定を確認します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/9_gradle_config.png)

### サーバー起動

実行設定が完了したら、右側のgradleメニューからTasks > other > `runMain`実行をダブルクリックします。このように一度実行した後は、IntelliJ右上の緑色の三角形のRunアイコンをクリックしてもサーバーが実行されます。

`runMain`で実行することで、GameAnvilサーバーの実行に必要なVMオプションが適用されます。もしMainクラスのmain()関数をそのまま実行する場合は、**Edit Configurations...**で以下の必須VMオプションを追加する必要があります。

```
"--add-opens", "java.base/java.lang=ALL-UNNAMED",
"--add-opens", "java.base/java.lang.invoke=ALL-UNNAMED"
"-XX:+UseG1GC"
```

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/10_gameanvil_run.png)

サーバーが正常に起動すると、サーバー起動状態に関するログが多数出力されます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/11_gameanvil_run_log.png)

GameAnvilサーバーは複数のノードで構成されています。これらのノードはサーバーが実行する機能を複数の役割で分担します。まだサーバーの初期起動を確認しただけで、ノードや他のサーバー起動のためのコード作成を行っていないため、完全に準備された状態ではありません。

各ノードはコードを実行するための準備に時間を要し、各ノードが準備完了するとonReadyログを出力します。クライアントがサーバーに接続する際に直接的な役割を果たすノードはゲートウェイノードです。ゲートウェイノードが準備されGatewayNodeのonReadyログが出力されれば、GameAnvilサーバーはいつでも接続可能な状態になったということです。

次のチャプターではGameAnvilの複数のノードのうち、サンプルゲーム動作のために必要なBasicGameNodeを実装してみます。

## GameAnvilサーバー機能の実装

### ゲームノードの実装

GameAnvilは`I-`プレフィックスを付けた複数のノードインターフェースを提供します。基本的なノードの機能はエンジン内部にすでに実装されており、ユーザーはこれらのインターフェースを実装して多様なコールバック機能を使用できます。今回の例ではIGameNodeインターフェースを実装したゲームノードクラスを作成して使用してみます。

プロジェクトパネルでMainクラスが位置するパスをマウスの右ボタンでクリックした後、**New > Package**を選択して**node**という名前の新しいパッケージを作成します。そしてnodeパッケージを再度マウスの右ボタンでクリックした後、**New > GameAnvil GameNode**を選択します。ファイル生成ダイアログが開いたら、**File name**に**SyncGameNode**、**Service name**に**Sync**を入力した後、**OK**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/13_select_game_node_file_template.png)

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/14_create_sync_game_node.png)

この機能は、先ほどテンプレートをインストールした際にFile templates(schemes)を一緒に適用したため使用できます。**New > GameAnvil GameNode**項目が見えない場合は、**New > Java Class**を選択して空のクラスを作成します。

自動的に作成されたコードは次のとおりです。

```java
package com.tutorial.gameanvil.node;

import com.nhn.gameanvil.game.GameAnvilGameNode;
import com.nhn.gameanvil.node.game.ChannelUpdateType;
import com.nhn.gameanvil.node.game.IGameNode;
import com.nhn.gameanvil.node.game.context.IGameNodeContext;
import com.nhn.gameanvil.node.game.data.IChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.IChannelUserInfo;
import com.nhn.gameanvil.packet.IPayload;

@GameAnvilGameNode(gameServiceName = "Sync")
public class SyncGameNode implements IGameNode {
    private IGameNodeContext gameNodeContext;

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

### ノードについて

全てのノードは、何か処理を開始できるループが始まったかどうかに応じて状態を持ちます。以下はノードが持ちうる状態の一部です。

- INIT
- PREPARE
- READY
- SHUTDOWN

ノードはINIT状態から始まり、記載された順序で状態を変えながらREADY状態に到達します。READY状態は、ノードが与えられた作業を処理し実行できる状態であることを示します。

自動生成されたコードには、各ノードの状態にフックされたコールバックをオーバーライドするコードが含まれています。例えば、onInit()メソッドに特定のロジックを作成すると、ノードが初期化(Init)を開始する直前の段階で該当コールバックが挿入され呼び出されます。

GameAnvilは大部分のコードがあらかじめ用意されているため、この段階でさらに作成するコードはありません。生成されたそのままゲームノードを使用すればよいです。

### ユーザータイプについて

各ゲームノードでルームに参加してパケットをやり取りする主体がユーザーですが、各ユーザー実装を区別する約束された文字列です。

GameAnvilで提供されるルームベースの実装を使用するには、上記で実装したノード以外に**ゲームユーザー**と**ゲームルーム**クラスが必要です。インターフェースの実装だけで簡単に実装する方法を説明します。

### ゲームユーザーの実装

クライアントがサーバーにログインすると、サーバーでは該当クライアント情報を**ゲームユーザー**というオブジェクトとして作成し、メモリに保存して維持します。ゲームユーザーがどのような情報を表現するかは、ユーザーが必要に応じて自由に実装可能です。ゲームユーザーの実装も、クラスの継承とコールバックのオーバーライドを通じて一貫性を持って実装できます。

プロジェクトパネルでMainクラスが位置するパスをマウスの右ボタンでクリックした後、**New > Package**を選択して**user**という名前の新しいパッケージを作成します。そして**user**パッケージを再度マウスの右ボタンでクリックした後、**New > GameAnvil User**を選択します。ファイル生成ダイアログが開いたら、**File name**に**SyncGameUser**、**Service name**に**Sync**、**User type**に**USER_TYPE_SYNC**を入力した後、**OK**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/15_create_sync_game_user.png)

自動的に生成されたコードは次のとおりです。

```java
package com.tutorial.gameanvil.user;

import com.nhn.gameanvil.game.GameAnvilUser;
import com.nhn.gameanvil.node.game.IUser;
import com.nhn.gameanvil.node.game.context.IUserContext;
import com.nhn.gameanvil.node.game.data.MatchCancelReason;
import com.nhn.gameanvil.node.game.data.MatchRoomFailCode;
import com.nhn.gameanvil.node.game.data.MatchUserFailCode;
import com.nhn.gameanvil.node.game.data.RoomMatchResult;
import com.nhn.gameanvil.packet.IPayload;
import com.nhn.gameanvil.serializer.ITimerHandlerTransferPack;
import com.nhn.gameanvil.serializer.ITransferPack;

@GameAnvilUser(
        gameServiceName = "Sync",
        gameType = "USER_TYPE_SYNC",
        useChannelInfo = false
)
public class SyncGameUser implements IUser {
    private IUserContext userContext;

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
        return RoomMatchResult.FAILED;
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

ゲームユーザーはクライアントがサーバーにログインリクエストを行うことで生成されます。サーバーではクライアントから送信されたペイロードなどを通じてログイン許可の可否を決定し、戻り値として返すことができます。主要ロジックのみエンジンユーザーが作成し、ログインの成功や失敗処理はエンジンが担当します。

このチュートリアルでは特別な検証過程なしにログインを許可するため、onLogin関数で常にtrueを返すようにしました。このようにすると、クライアントからログインリクエストがあった際に常にユーザーオブジェクトを生成し、成功レスポンスを返すことになります。

### ゲームルームの実装

正常にゲームユーザーとしてゲームノードに接続すると、他のユーザーとゲームルームを通じてパケットをやり取りできるようになります。ゲームルームとは、パケットをやり取りするユーザーを論理的にまとめたグループです。ゲームルームもインターフェースの実装を通じて生成できます。

プロジェクトパネルでMainクラスが位置するパスをマウスの右ボタンでクリックした後、**New > Package**を選択して**room**という名前の新しいパッケージを作成します。そして**room**パッケージを再度マウスの右ボタンでクリックした後、**New > GameAnvil Room**を選択します。ファイル生成ダイアログが開いたら、**File name**に**SyncGameRoom**、**Service name**に**Sync**、**Room type**に**ROOM_TYPE_SYNC**、**User**に**SyncGameUser**を入力した後、**OK**をクリックします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/16_create_sync_game_room.png)

自動的に生成されたコードは次のとおりです。

```java
package com.tutorial.gameanvil.room;

import com.nhn.gameanvil.game.GameAnvilRoom;
import com.nhn.gameanvil.node.game.IRoom;
import com.nhn.gameanvil.node.game.context.IRoomContext;
import com.nhn.gameanvil.node.game.data.MatchCancelReason;
import com.nhn.gameanvil.packet.IPayload;
import com.nhn.gameanvil.serializer.ITimerHandlerTransferPack;
import com.nhn.gameanvil.serializer.ITransferPack;
import com.tutorial.gameanvil.user.SyncGameUser;

import java.util.List;

@GameAnvilRoom(
        gameServiceName = "Sync",
        gameType = "ROOM_TYPE_SYNC",
        useChannelInfo = false
)
public class SynGameRoom implements IRoom<SyncGameUser> {
    private IRoomContext roomContext;

    @Override
    public void onCreate(IRoomContext<SyncGameUser> roomContext) {
        this.roomContext = roomContext;
    }

    @Override
    public void onInit() {

    }

    @Override
    public void onDestroy() {

    }

    @Override
    public boolean onCreateRoom(SyncGameUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean onJoinRoom(SyncGameUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean canLeaveRoom(SyncGameUser user, IPayload payload, IPayload outPayload) {
        return true;
    }

    @Override
    public void onLeaveRoom(SyncGameUser user) {

    }

    @Override
    public void onAfterLeaveRoom() {

    }

    @Override
    public void onRejoinRoom(SyncGameUser user, IPayload payload) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(List<SyncGameUser> list, ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

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
    public boolean onMatchParty(String roomType, String matchingGroup, SyncGameUser user, IPayload payload, IPayload outPayload) {
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

ゲームルームはゲームユーザーがサーバーにルーム生成リクエストを行うと生成されます。クライアント側では簡単にメソッド呼び出しだけでルームを生成し、存在するルームに入室できます。ユーザーがルームに入室する時点、またはルームが生成される時点にカスタムコードを挿入したい場合は、適切なコールバックをオーバーライドして簡単にコードを組み込むことができます。

## サーバー実装の仕上げ

ここまで、基礎チュートリアルサンプル実行のためのサーバー構築が完了しました。再度サーバーを実行してみると、ログの中に`All nodes are ready!!`という文言を確認できます。このログが表示されたということは、GameAnvilサーバーが正常に実行されたことを意味します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/17_all_nodes_are_ready.png)

これでクライアントのリクエストを受け付けるサーバーが準備できました。次の段階ではGameAnvilコネクタとUnityサンプルプロジェクトを活用してクライアントを実装してみます。

## 実習環境の準備 - クライアントプロジェクト

### GameAnvilConnectorのダウンロード

GameAnvilコネクタdll使用のために以下のファイルをダウンロードします。

[gameanvil_connector_2.0.0.unitypackage](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector.unitypackage)

### Unity Packageのダウンロード

GameAnvilコネクタ使用実習のために以下のリンクからUnityパッケージをダウンロードします。

[gameanvil_tutorial_basic.unitypackage](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil_tutorial_basic.unitypackage)

### Unityプロジェクトの作成

Unity Hubを実行した後、右上のNew Projectボタンをクリックします。Unity Hubのバージョンは問いません。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/18_unity_hub.png)

テンプレートとして**2D**を選択し、プロジェクト名と保存場所を確認した後、**Create project**をクリックします。この例で使用したUnityバージョンは2022.3.21f1であり、実習時に他のバージョンを使用しても問題ありませんが、全てのケースをテストしたわけではないため、サンプル実行バージョンと同じ環境で進めることを推奨します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/19_new_unity_project.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/20_new_unity_project_done.png)

### GameAnvilConnector及びUnity Packageのインポート

プロジェクトビューをマウスの右ボタンでクリックし、**Import Package > Custom Package...**を選択した後、Finderまたはファイルエクスプローラーが開いたら、前の段階でダウンロードしたUnityパッケージを選択します。gameanvil_connector、gameanvil_tutorial_basicの順にImportを実行します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/21_import_unity_package_gameanvil_connector.png)

GameAnvilSampleフォルダ内のSceneフォルダからIntroSceneを開き、以下のような画面を確認します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/22_unity_after_import_package.png)

もし次のようにCinemachine、InputSystem関連のパッケージエラーが発生する場合は、**Window > Package Manager**を選択してPackage Managerウィンドウを開いた後、**Packages: Unity Registry**を選択し、**Cinemachine**と**InputSystem**を検索してインストールします。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/23_package_error.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/24_add_cinemachine_package.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/25_add_inputsystem_package.png)

Canvasに追加されたUI Managerを通じて、例で使用されたUnity UIコンポーネントを確認できます。

**File > Build Settings**で**Add Open Scene**をクリックし、ビルド時に含まれるように設定します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/26_intro_scene_to_build_settings.png)

## GameAnvilManager

Hierarchyビューでマウスの右ボタンをクリックし、**GameAnvil > GameAnvilManager**をクリックします。GameAnvilManagerゲームオブジェクトが生成され、GameAnvilManagerゲームオブジェクトのインスペクター上で以下のように設定を修正できます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/27_gameanvil_manager.png)

- Connect Configuration: 接続情報を修正できます。
- Authentication Configuration: 認証情報を修正できます。
- Login Configuration: ログイン情報を修正できます。
- Pause Client Check : クライアントの接続状態を定期的に確認する時間間隔を調整できます。

今は詳細設定について詳しく知らなくても大丈夫です。チュートリアルを進めながら、各項目に関する説明を確認できます。

## サーバーとクライアントの接続

GameAnvilクライアントがGameAnvilサーバーに接続するためには、Connect、Authentication、Loginの3段階を経る必要があります。

- Connect: サーバーとクライアント間で通信できるようにソケットを生成して接続します。
- Auth: クライアントがサーバーを通じてデータを送受信できるように許可するかどうかをサーバーで決定します。
- Login: サーバーのメモリにクライアントの情報を表現するオブジェクト,すなわちゲームユーザーを生成します。

各段階は順次進行し、前の段階が正常に完了しなければ次の段階へ進むことができません。

コネクタのGameAnvilManagerが提供するLogin APIを使用すると、Connect、Authentication、Loginの過程を一度に統合して処理できます。

ここではHierarchyビュー上のTesterゲームオブジェクトにコンポーネントとして追加されているGameAnvilManagerTesterスクリプトをソースコードエディタで開き、実装を追加しながら各過程を直接実習します。

### Connect関連フィールド設定

接続するサーバー情報を記載します。ローカルでサーバーを直接立ち上げる場合なので、ipは`127.0.0.1`を使用します。portはゲートウェイノードのデフォルトポートである`18200`を使用します。ipとport情報は、プレイモードで修正できるようにUnityのInputFieldと接続するコードが作成されていることを確認できます。

```c#
public string managerIp
{
    get => UI.managerIpInputField.text;
    set => UI.managerIpInputField.text = value;
}

public int managerPort
{
    get => int.Parse(UI.managerPortInputField.text);
    set => UI.managerPortInputField.text = value.ToString();
}
```

### Authentication関連フィールド設定

認証に必要な情報を記載します。認証に必要な情報としては、accountId、deviceId、passwordの3つがあります。今は認証段階を無条件で通過するようにサーバー実装がされている状態なので、どのような値に設定しても動作に異常はないでしょう。プレイモードでUnityのInputFieldを通じて入力された値を使用できるように、コードが作成されていることを確認できます。

```c#
public string managerAccountId
{
    get => UI.managerAccountIdInputField.text;
    set => UI.managerAccountIdInputField.text = value;
}

public string managerPassword
{
    get => UI.managerPasswordInputField.text;
    set => UI.managerPasswordInputField.text = value;
}

public string managerDeviceId
{
    get => UI.managerDeviceIdInputField.text;
    set => UI.managerDeviceIdInputField.text = value;
}
```

### Login関連フィールド設定

ログインに必要な情報を記載します。ログインに必要な情報としては、ユーザータイプ、チャンネルID、そしてサービス名があります。サーバー実装時に作成したユーザータイプとサービス名を使用する必要があります。プレイモードでUnityのInputFieldを通じて値を修正できるように設定されています。

```c#
public string managerServiceName
{
    get => UI.managerServiceNameInputField.text;
    set => UI.managerServiceNameInputField.text = value;
}

public string managerUserType
{
    get => UI.managerUserTypeInputField.text;
    set => UI.managerUserTypeInputField.text = value;
}

public string managerChannelId
{
    get => UI.managerChannelIdInputField.text;
    set => UI.managerChannelIdInputField.text = string.IsNullOrEmpty(value) ? "" : value;
}
```

### フィールド自動入力設定

プレイモードで毎回入力値を入力する手間を省くために、ConstantManagerで使用する値をあらかじめ入力しておくことができます。例で使用した入力値は以下のとおりです。

```c#
public class ConstantManager : MonoBehaviour
{
    private GameAnvilConnectorTester connectorTester;
    private GameAnvilManagerTester managerTester;

    public static string ip = "127.0.0.1";
    public static int port = 18200;
    [Space]
    public static string accountId = "test";
    public static string password = "test";
    public static string deviceId = "test";
    [Space]
    public static string serviceName = "Sync";
    [Space]
    public static string userType = "USER_TYPE_SYNC";
    public static string channelId = "";
    [Space]
    public static string roomType = "ROOM_TYPE_SYNC";

    ...(省略)...
}
```

### Unity UI設定

```c#
void Start()
{
    UI = GameObject.FindWithTag("UIManager").GetComponent<UIManager>();

    gameAnvilManager = GameAnvilManager.Instance;
    gameAnvilManager.onStateChange.AddListener(OnManagerStateChanged);

    UI.managerLoginButton.onClick.AddListener(ManagerLogin);
    UI.managerLogoutButton.onClick.AddListener(ManagerLogout);
    UI.managerCreateRoomButton.onClick.AddListener(ManagerCreateRoom);
    UI.managerJoinRoomButton.onClick.AddListener(ManagerJoinRoom);
    UI.managerLeaveRoomButton.onClick.AddListener(ManagerLeaveRoom);

    ...(省略)...
}

void Update()
{
    UI.managerLoginState.Set(gameAnvilManager.State == GameAnvilManager.LoginState.LOGIN_COMPLETE);
    UI.managerJoinedRoomState.Set(gameAnvilManager.UserController != null && gameAnvilManager.UserController.IsJoinedRoom);
}

public void OnManagerStateChanged(GameAnvilManager.LoginState oldState, GameAnvilManager.LoginState newState)
{
    UI.consoleInputField.text += newState.ToString() + '\n';
}
```

### ログインAPI呼び出し

Connect、Authentication、Loginの過程が統合されたGameAnvilManagerのLogin APIは、次のように使用します。result値を通じて呼び出し結果を確認できます。

```c#
public async void ManagerLogin()
{
    gameAnvilManager.ip = managerIp;
    gameAnvilManager.port = managerPort;
    gameAnvilManager.accountId = managerAccountId;
    gameAnvilManager.deviceId = managerDeviceId;
    gameAnvilManager.password = managerPassword;
    gameAnvilManager.userType = managerUserType;
    gameAnvilManager.channelId = managerChannelId;
    gameAnvilManager.serviceName = managerServiceName;

    try
    {
        var result = await gameAnvilManager.Login(null);
        UI.managerResultCodeInputField.text = result.loginResultCode.ToString();
        UI.managerExceptionInputField.text = null;
        if (result.loginResultCode != GameAnvilManager.LoginResultCode.SUCCESS)
            UI.consoleInputField.text += result.loginResultCode.ToString() + '\n';
        UI.consoleInputField.text += result.loginResultCode.ToString() + '\n';
    }
    catch (Exception e)
    {
        UI.managerResultCodeInputField.text = null;
        UI.managerExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

### ログアウトAPI呼び出し

```c#
public async void ManagerLogout()
{
    await gameAnvilManager.Logout();
}
```

### サーバー接続及びログインテスト

サーバーが実行中か確認した後、Unityエディタでプレイモードに入ります。**Login**をクリックして、正常にサーバー接続及びログインが進行することを確認します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/28_login_success.png)

ログインが完了すると、上部の状態表示ウィンドウに緑色のランプ表示と共にLOGIN状態で表示されます。この状態で**Logout**ボタンを押し、正常にログアウトされるか確認します。

ログアウトが完了すると、上部の状態表示ウィンドウに赤色のランプ表示と共にLOGOUT状態で表示されます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/29_logout_success.png)

## ゲームルーム生成及び入室

### ルーム生成及び入室関連フィールド設定

```c#
public string managerRoomType
{
    get => UI.managerRoomTypeInputField.text;
    set => UI.managerRoomTypeInputField.text = value;
}

public int managerRoomId
{
    get => int.Parse(UI.managerRoomIdInputField.text);
    set => UI.managerRoomIdInputField.text = value.ToString();
}
```

### ゲームルーム生成リクエストAPI使用

GameAnvilコネクタのルーム生成リクエストAPIを呼び出して、クライアントは簡単にサーバーへゲームルーム生成をリクエストできます。ゲームルーム生成リクエストメソッドを呼び出す際、パラメータとしてルームタイプを渡す必要がありますが、サーバーと事前に合意したルームタイプ値を渡せば問題ありません。

```c#
public async void ManagerCreateRoom()
{
    try
    {
        var result = await gameAnvilManager.UserController.CreateRoom(string.Empty, managerRoomType, ConstantManager.channelId);
        UI.managerRoomResultInputField.text = result.ErrorCode.ToString();
        UI.managerRoomIdResultInputField.text = result.Data.RoomId.ToString();
        UI.managerRoomExceptionInputField.text = null;
        UI.consoleInputField.text += result.ErrorCode.ToString() + '\n';
        UI.consoleInputField.text += "Room Id : " + result.Data.RoomId + '\n';
    }
    catch(Exception e)
    {
        UI.managerRoomResultInputField.text = null;
        UI.managerRoomIdResultInputField.text = null;
        UI.managerRoomExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

ゲームルーム生成機能の実装が終わりました。テストは少し後回しにして、ゲームルーム入室機能を先に実装します。

### ゲームルーム入室リクエストAPI使用

サーバーにゲームルームが生成されたと仮定しましょう。該当ルームに接続するためには、GameAnvilコネクタでゲームルーム入室リクエストメソッドを呼び出せばよいです。このとき、ルーム生成時に受け取ったゲームルームIDを渡します。プレイモードでUnityのInputFieldを通じて、入室するルームIDを入力します。

```c#
public async void ManagerJoinRoom()
{
    try
    {
        var result = await gameAnvilManager.UserController.JoinRoom(managerRoomType, managerRoomId, string.Empty);
        UI.managerRoomResultInputField.text = result.ErrorCode.ToString();
        UI.managerRoomIdResultInputField.text = result.Data.RoomId.ToString();
        UI.managerRoomExceptionInputField.text = null;
        UI.consoleInputField.text += result.ErrorCode.ToString() + '\n';
        UI.consoleInputField.text += "Room Id : " + result.Data.RoomId + '\n';
    }
    catch (Exception e)
    {
        UI.managerRoomResultInputField.text = null;
        UI.managerRoomIdResultInputField.text = null;
        UI.managerRoomExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

これでゲームルーム生成機能と入室機能が全て完成しました。

### ゲームルームテスト

Unityエディタでショートカットキー`CMD + b`または`Ctrl + b`を押してビルドします。ビルド結果として出たウィンドウでボタンをクリックし、ゲームルームが生成されることを確認します。ゲームルームが生成されると、画面にゲームルームのIDが表示されます。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/30_create_room_success.png)

その後、Unityエディタでプレイモードに入ります。必要な場合、画面を1080 X 1920に設定します。以前に表示されたゲームルームのIDを入力フィールドに入力し、**Join Room**をクリックしてゲームルームに参加できるか確認します。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/31_join_room_success.png)

## 同期コントローラー入門

これで同じゲームルームに接続したゲームユーザー間では、パケットをやり取りできます。このパケットを通じて、必要な情報をクライアントプロセス間で同期するようにコードを作成できます。より簡単な方法としては、同期したいゲームオブジェクトに同期コンポーネントをアタッチするだけでも同期を実装できます。

### 同期コントローラー

Hierarchyビュー上でSyncControllerゲームオブジェクトを探し、次のようにSyncControllerがコンポーネントとして追加されていることを確認します。

[写真]

もし該当ゲームオブジェクトが存在しない場合は、Hierarchyビューで右クリックし、**GameAnvil > SyncController**を選択して追加します。

これでGameAnvilの全ての同期機能を利用できます。次は最も単純な例を通じて、同期コンポーネントのアタッチ及び使用方法を確認してみます。

### 同期オブジェクト

プロジェクトビューでResourcesフォルダ内部へ移動した後、Characterプレハブをダブルクリックしてプレハブ修正画面へ切り替えます。インスペクターウィンドウでSync、TransformSync、RigidBody2DSync、AnimatorSyncコンポーネントが追加されていることを確認します。

該当コンポーネントは、GameAnvilコネクタが提供する同期機能のためのコンポーネントです。コンポーネントの追加だけで、該当プレハブはゲームユーザー間でゲームオブジェクトのTransform、Rigidbody2D、Animation情報を同期する準備が完了したことになります。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/32_character_sync_prefab.png)

完成した同期ゲームオブジェクトプレハブをUnityプレイモードで使用するには、SyncControllerが提供するゲームオブジェクト生成APIを通じてゲームオブジェクトを生成し、シーンに追加すればよいです。第1引数としてプレハブ名を渡す必要があります。

注意すべき点は、GameAnvilが提供する同期コンポーネントを付けた同期ゲームオブジェクトは、Unityが基本提供するGameObject.Instantiate()メソッドではなく、GameAnvilが提供するSyncControllerのInstantiate APIを使用しないと、正常な同期が行われないということです。

ルームに入室した後、同期ゲームオブジェクトを生成するようにコードを作成します。

```c#
public async void ManagerCreateRoom()
{
    try
    {
        var result = await gameAnvilManager.UserController.CreateRoom(string.Empty, managerRoomType, ConstantManager.channelId);
        UI.managerRoomResultInputField.text = result.ErrorCode.ToString();
        UI.managerRoomIdResultInputField.text = result.Data.RoomId.ToString();
        UI.managerRoomExceptionInputField.text = null;
        UI.consoleInputField.text += result.ErrorCode.ToString() + '\n';
        UI.consoleInputField.text += "Room Id : " + result.Data.RoomId + '\n';

        // コード追加
        if (result.ErrorCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
        {
            OnEnterRoom();
        }
    }
    catch(Exception e)
    {
        UI.managerRoomResultInputField.text = null;
        UI.managerRoomIdResultInputField.text = null;
        UI.managerRoomExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

```c#
public async void ManagerJoinRoom()
{
    try
    {
        var result = await gameAnvilManager.UserController.JoinRoom(managerRoomType, managerRoomId, string.Empty);
        UI.managerRoomResultInputField.text = result.ErrorCode.ToString();
        UI.managerRoomIdResultInputField.text = result.Data.RoomId.ToString();
        UI.managerRoomExceptionInputField.text = null;
        UI.consoleInputField.text += result.ErrorCode.ToString() + '\n';
        UI.consoleInputField.text += "Room Id : " + result.Data.RoomId + '\n';

        // コード追加
        if (result.ErrorCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS)
        {
            OnEnterRoom();
        }
    }
    catch (Exception e)
    {
        UI.managerRoomResultInputField.text = null;
        UI.managerRoomIdResultInputField.text = null;
        UI.managerRoomExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

```c#
private void OnEnterRoom()
{
    Debug.Log("User enter the room.");
    SyncController.Instance.InstantiateSyncObject();
    SyncController.Instance.Instantiate("Character", Vector3.zero, Quaternion.identity);
}
```

### 同期テスト

Unityエディタで`CMD + b`または`Ctrl + b`ショートカットキーを押してビルドします。ビルド結果として出たウィンドウでログイン後、ゲームルームを生成し、**Toggle Hide**ボタンを押して同期ゲームオブジェクトをテストする画面が見えるようにします。

その後、Unityエディタでプレイモードに入ります。ビルド結果として出たウィンドウと同じゲームルームに接続した後、**Toggle Hide**ボタンを押して同期ゲームオブジェクトのテスト画面が見えるようにします。

このとき、別々のユーザーとして識別されるために、ログイン時すでに使用したaccountIdとは異なるaccountIdを使用する必要があります。

ルームに入室しつつ、片方のクライアントでゲームオブジェクトを生成すると、同じルームに入室したもう片方のクライアントの画面にも同様に現れることを確認します。

キーボードのW、S、A、Dキーまたは上下左右キーを入力してゲームオブジェクトを移動させてみます。ゲームオブジェクトが移動し、その過程でアニメーションが変化することを確認します。同じルームに入室した他のクライアント画面でも同様に現れることを確認します。

これでGameAnvilが提供する同期機能を通じ、ゲームオブジェクト生成の同期及びTransform、Rigidbody、Animationの同期を体験しました。

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/33_sync_test.gif)

## チュートリアルの仕上げ

このドキュメントでは、GameAnvilコネクタの便利機能である接続、認証、ログイン過程を統合した簡易ログイン機能と同期機能について、実習を通じて学びました。チュートリアルの冒頭で紹介したように、GameAnvilにはゲームサーバー制作に必要な全ての機能が用意されており、チュートリアルではその一部のみ軽く扱いました。続くドキュメントで、より詳細な使用方法を学ぶことができます。
