## Game > GameAnvil > リリースノート > GameAnvil
### 2.1.0 (2025.06.30)
#### New
##### 依存関係の管理とspring bootの追加
* GameAnvilの依存関係管理にspring bootを使用し、Beanを追加または注入できます。
* 従来のインターフェースを使用してエンジンに登録していたコードも、今後はアノテーションを宣言してエンジンに登録します。
#### Remove
##### Gateway、Connectionでのメッセージ処理を非推奨化
* 使用性が低く、直感的でないAPIであるため、今後のリリースで削除される予定です。

##### Gateway、ConnectionからTimer、Topicを削除
* 使用性が低く、直感的でないAPIを削除しました。
* Room、Userなど、一般的に使用されるゲームノードでは引き続き使用できます。
#### FIX
* ChannelCountInfo APIをリクエストする際、予期せぬエラーメッセージが発生する可能性があった問題を修正しました。
* サーバー終了までのデフォルト待機時間が減少しました。
* プロトコルバッファ使用時、フルパスではなくファイル名を使用するようにし、同じファイルであれば競合が発生しないように修正しました。
* 内部でこれ以上使用しないプロトコルバッファパケットを削除しました。
* 発生はしないが宣言されていたChecked Exceptionの宣言を削除しました。
* 全クラスタリングを行っていないにもかかわらず、断続的に「All nodes are Ready」メッセージが出力される問題を修正しました。
* トピック追加時、内部的にデバッグログを追加しました。
* 内部で使用していたIpcノードを削除しました。従来のIpcの機能はIpcNetworkで管理します。
* Node関連APIで時々ConcurrentModificationExceptionが発生する問題を修正しました。
* サーバーを長時間実行した際に時々overflowが発生する問題を修正しました。
* 内部ポートに不明なパケットが流入した際にクラッシュする可能性があった問題を修正しました。


### 2.0.0 (2024.12.04)

#### New
##### Java 21
* GameAnvil 2.0はJava 21またはそれ以上のバージョンでのみ動作します。
* エンジンチームでは現在LTSであるJava 21の使用を推奨します。

##### Virtual Thread導入
* Quasarの依存関係が削除され、throws SuspendExecutionを使用せずにコードを作成できます。
* Java 21の動作に合わせて、GameAnvilの全てのユーザー呼び出しコードで非同期作業を使用できます。
* Java 21使用時、**synchronized**コードを使用しないよう注意してください。Pinning問題が発生する可能性があります。
    * この制限はJava 24で解除される可能性があります。[https://openjdk.org/jeps/491](https://openjdk.org/jeps/491)
* GameAnvilでは、既存のNodeとコードを簡単に作成できるようカスタムしたVirtual Threadを使用します。このスレッディングモデルは1 Thread N Virtual Threadであり、既存のFiberと同様です。 
    * 以下のJVMオプションを実行時に追加してください。
    * `--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.invoke=ALL-UNNAMED`
* Virtual Threadの詳細な動作は[こちら](https://openjdk.org/jeps/444)を参照してください。

##### 多数のクライアントへメッセージを送信できるAPIを追加
* Room.sendToClients APIが追加されました。

##### GameAnvilConnector 2.0
* GameAnvil 2.0に対応するGameAnvilConnector 2.0をリリースしました。
* 新規コネクタをご確認ください。

##### MatchNodeのSafePause機能追加
* MatchNodeでSafePause機能を活用できます。
* MatchNodeのSafePause機能は、開始ノード(src)と移動ノード(dest)が1:1の場合のみ実行可能です。
* SafePauseノードの登録及び実行方法は、既存のGameNodeと同様です。
* MatchNode SafePauseに関連する次のManagement HTTP APIが追加されました。
1. POST /management/matching-group-update
    * body: ``` {"nodeId": <Number ノードID>, "matchingGroup": "<String マッチンググループ>" }
    * 対象MatchNodeで引数として受け取ったmatchingGroupを制御するようにします。
2. GET /management/matching-queue-clear?nodeId=&lt;Number ノードID&gt;
    * 対象MatchNodeのマッチングキューを全て空にします。もしノードIDが0の場合、全てのノードのマッチングキューを空にします。

##### パケットパース中にエラー発生時、エラーログを出力

#### Remove
##### Quasar依存関係の削除
* Quasar及び関連コードが削除されました。 
* GameAnvil 1.4で提供していた非同期機能の代わりにJava 21のVirtual Threadを使用します。

##### MySql依存関係及び関連コードの削除 
* `com.github.jasync-sql:jasync-mysql`依存関係及びエンジンの非同期関連コードが削除されました。 
* GameAnvil 1.4で提供していた機能の代わりに上記ライブラリを直接使用しても非同期で動作します。

##### Redis依存関係及び関連コードの削除
* `io.lettuce:lettuce-core`依存関係及びエンジンの非同期関連コードが削除されました。 
* GameAnvil 1.4で提供していた機能の代わりに上記ライブラリを直接使用しても非同期で動作します。

##### HttpClient依存関係及び関連コードの削除
* `org.asynchttpclient:async-http-client`, `org.apache.httpcomponents.client5:httpclient5`依存関係及びエンジンの非同期関連コードが削除されました。 
* GameAnvil 1.4で提供していた機能の代わりに上記ライブラリを直接使用しても非同期で動作します。

##### MultiRequest削除
* MultiRequestの代わりにユーザー指定プロトコルバッファを使用できます。
* 複数の対象または複数のパケットを同時にリクエストする際、MultiRequestの代わりに新しく追加されたFuture方式のRequest APIを使用できます。


##### TimerHandlerのObject引数を削除
* 使用しないObject引数が削除されました。

##### addTopics(List&lt;T&gt;)、removeTopics(List&lt;T&gt;)削除  
* 代わりにaddTopic(String)、removeTopic(String)を使用します。


#### Change
#####  レスポンスを受け取るAPIの戻り値がFutureに変更 
* GameAnvilのAPIも、他の多くの非同期方式APIと同様にFutureを返す方式に変更されました。これで、より自由にコードフローを作成できます。 
* GameAnvilが提供するAsync APIを使用する必要がなくなりました。GameAnvilでVirtual Threadを公式サポートするため、ブロッキングコードを呼び出してもJava 21をサポートするコードであれば、Platform ThreadではなくVirtual Threadがブロックされます。今後、Async APIはサポートしません。
```java
// GameAnvil 2.0以前のバージョン 
Packet packetResponse = user.requestToNode(nodeId, myRequest1);
ListenableFuture<Response> httpFuture =  getHttpClient().executeRequest(myRequest2);
// Response httpResponse = httpFuture.get(); // !Java 11の場合、Platform Threadブロック発生！
Response httpResponse = Async.awaitFuture(httpFuture.toCompletableFuture());
```
```java
// GameAnvil 2.0 
Future<Packet> packetFuture = user.requestToNode(nodeId, myRequest1);
Future<Response> httpFuture = getHttpClient().executeRequest(myRequest2);
Packet packetResponse = packetFuture.get();  
Response httpResponse = httpFuture.get();  // Java 21ではVirtual Threadのみブロック
```
* これで、既存のGameAnvilでは不便だった以下の機能を改善できるようになりました。
    * 同時に複数の場所へリクエストを送信する機能の改善
    * リクエスト送信後、受信するまでに必要な作業の実行を改善
* 既存バージョンとの互換性を維持する必要があるコードであれば、返されたFutureに対して即座にgetを呼び出してください
> Platform Thread：従来のJavaで使用していたThread。OSスレッドまたはJavaスレッドです
> Virtual Thread：Java 21で追加された新しいThreadです。旧バージョンのGameAnvilのFiberと同様の動作をします。詳細な動作は[こちら](https://openjdk.org/jeps/444)を参照してください

##### Handler実行がContextベースに変更 
* Handler実行を各対象に合わせた`DispatchContext`ベースに変更しました。
* DispatchContextはリクエストごとに新しく生成します。
* 今後、`reply`関数はDispatchContextで提供します。
* 例えば、UserでEchoReqを処理するメッセージハンドラは次の通りです。

```java
public class _EchoReq implements IMessageHandler<IUserDispatchContext, EchoReq> {
    @Override
    public void execute(IUserDispatchContext ctx, EchoReq request) {
        MyUser user = ctx.getUser();
        ctx.reply(レスポンス_プロトコル_バッファ);
    }
}
```

* DispatchContextはリクエスト自体の情報を持つ値です。既存のGameAnvilは、Handlerコードでreplyを送信する際、必ず現在のフローでのみ送信可能でした。
    * この制限を無視して非同期作業でreplyを送信した場合、予期しない動作をする可能性がありました。
    * リクエスト情報が対象(ここではUser)に保存されることで発生する問題ですが、サーバーに他のリクエストが入ってくると以前のリクエスト情報を上書きするためです。
* 問題が発生する可能性のある簡単なコードを以下の例に作成しました。
```java 
// 旧バージョンのGameAnvil

// クラス宣言省略。ハンドラ
public void execute(MyUser user, EchoReq request) throws SuspendExecution {
    user.postJob((obj) -> {
        // 何らかの非同期作業実行後にレスポンスを送信する際、正常に動作しない可能性があります。
        // ハンドラのexecuteメソッド終了後、他のハンドラがreply情報を変更させる可能性があるためです。
        user.reply(..レスポンス_パケット..);
    }, null);
}
```

```java 
// 新しいGameAnvil 2.0

// クラス宣言省略。ハンドラ
public void execute(IUserContext ctx, EchoReq request) throws SuspendExecution {
    IUserContext userContext = ctx.getUserContext();
    userContext.runOnNextMsgLoop(() -> {
        // これで正常に動作します。
        ctx.reply(..レスポンス_パケット..);
    });
}
```

##### Baseクラスの代わりにインターフェースへ変更
* ユーザー実装Baseクラスがインターフェースに変更されました。
* 既存のBaseクラスで提供していたメソッド実装体は、クラスContextで提供されます。
* 例えばBaseUserで使用していたAPIは、次のようにマイグレーションできます。
* 既存のPacketDispatcher、Baseクラス定義Annotationは削除されました。
```java
public class MyUser implements IUser {
    private IUserContext userContext;
    public void onCreate(IUserContext ctx) {
        this.userContext = ctx;
        
        userContext.send(..パケット..);  // このようにIUserContextを活用して 
                                     // 既存のBaseクラスで提供していた機能をそのまま活用できます。
    }
}
```


| 既存 |変更 | 備考 |
| --- | --- | --- |
| BaseConnection | IConnection, IConnectionContext | |
| BaseSession|  ISession, ISessionContext|  |
| BaseGatewayNode | IGatewayNode, IGatewayNodeContext |  |
| BaseGameNode | IGameNode, IGameNodeContext|  |
| BaseUser | IUser, IUserContext|  |
| BaseRoom | IRoom, IRoomContext |  |
| BaseSupportNode | ISupportNode, ISupportNodeContext |  |

##### ユーザー指定クラス / メッセージ処理者(ハンドラ)の登録方法変更
* ユーザー指定クラスとメッセージ処理者の登録方法がそれぞれ別々だったものを、一箇所で登録するように変更しました。
* 簡単な使用方法は次の通りです。

```java
var gameAnvilServer = GameAnvilServer.getInstance();
var builder = gameAnvilServer.getServerTemplateBuilder();

builder.connection(MyConnection::new, config -> {
     config.protoBufferHandler(MyProto.HelloProto.class, new _HelloMessageHandler());
});

var gameServiceBuilder = builder.createGameService("MyGame"); // ゲームサービス定義
gameServiceBuilder.user("MyUserType", MyGameUser::new, config -> {
    config.protoBufferHandler(MyProto.UserHello.class, new _UserHelloMessageHandler());
});
```



##### ServiceIdの代わりにServiceNameを使用
* エンジンユーザーが認識しづらいServiceIdを受け取るAPIが、ユーザーが入力したServiceNameをそのまま受け取るように変更されました。

##### ProtoBuffer 4.28.3使用
* ProtoBufferを3.xバージョンにダウングレードした場合、正常に動作しない可能性があります。
* ProtoBufferの依存関係、protoc、すでにビルドされたProtoBufferファイルの交換が必要です。


##### 1 ThreadでN個のGameNodeを実行可能、ゲームノードのChannelID設定変更 
* 複数のGameNodeを実行できるように、ゲームノードのChannelID設定が変更されました。 
```
 "channelIDs": [
        ["ch1"]
        ["ch2"],
]
```
* 上記のように作成時、従来通り1 Thread = 1 GameNodeです。
```
 "channelIDs": [
        ["ch1", "ch2"]
]
```
* 上記のように作成時、1 Thread = 2 GameNodeです。


##### AutoIp失敗時、サーバーが強制終了されるように変更
* 初期に正常に動作しない状態で停止する代わりに、ユーザーが容易に認識できるよう強制終了します。

##### Timerインターフェース修正
* 理解しづらかった既存のタイマーインターフェースの代わりに、標準ライブラリのScheduledExecutorServiceで提供する関数と類似するようにTimerインターフェースを修正しました。

```
scheduleTimer - 1回
scheduleTimerWithFixedDelay - N回、直近の駆動以降ディレイ
scheduleTimerAtFixedRate - N回、固定ディレイ
```

##### Replyを2回以上送信できないように変更
* 従来Replyを2回以上送信した際に不明な問題が発生していた現象が解決されました。

##### User、RoomからNodeへアクセスする方法の変更
* 今後、getGameNode呼び出し時にNodeを直接返す代わりにINodeViewを返します。
* INodeViewから従来のようにNodeへ直接アクセスできますが、メソッドを渡してNodeの動作を実行することを推奨します。
    * NodeとUser、RoomのPlatform Threadは同一ですが、Virtual Threadはそれぞれ異なります。そのため、UserからNodeに定義された非同期作業を実行するなど、他領域の非同期作業を呼び出す際、意図通りに動作しない問題があり、使用時に注意が必要でした。INodeViewではNodeへの直接アクセスを防ぎ、代わりにNodeで実行するメソッドを渡すようにしてこれらの問題を解決します。
* GameAnvilの上級ユーザーには、INodeViewのこのような制約が不便な場合があります。単純な同期呼び出しメソッド(get/setなど)の場合は、前述した同期の問題点がないため、`INodeView.getUnsafeNode`メソッドを活用してNodeへ直接アクセスできます。このようにgetUnsafeNodeを通じてノードへ直接アクセスする際は、同期問題が発生しないよう注意して使用してください。

##### RoomにてUserのリスト提供
* 今後、Roomが保持しているUserのリスト提供を受けられます。

##### コールバックの意味がより明確になるようコールバック名を修正
* 次のコールバックが追加/変更されました。

| 対象 |既存 |変更 | 備考 |
| --- | --- | --- | -- |
| Session | onPreLogin |onBeforeLogin  |
| Session  | onPostLogin | onAfterLogin |
| Session  |  onPostLogout| onAftterLogout |
| User | onPostLogin | onAfterLogin |
| User  | onPostLeaveRoom |  onAfterLeaveRoom |
| User  |  canTransfer | canTransferOut  |
| User  |  onTransferInTimerHandler | onTransferIn | 既存のonTransferInメソッドに統合 | 
| User  | onPostTransferIn  | onAfterTransferIn  |
|  Room | onPostLeaveRoom | onAfterLeaveRoom |
| Room  | onLeaveRoom | canLeaveRoom | LeaveRoom条件検査|
| Room  |  なし | onMatchPartyCancel | マッチングキャンセル時に呼び出し|
| RoomMatchMaker | onPreMatch | 削除 | onMatchへ統合 |
| RoomMatchMaker | onPostMatch | 削除 |  onMatchへ統合 |

#### Fix

* Request API使用中に失敗時、エラーログにパケット情報を追加
* Request API使用中に対象が見つからない場合、Timeoutの代わりに即時失敗応答
* 対象ノードタイプ別にあったSend、Request APIを、対象ノードのIdを受け取る単一APIに統合
* ノードまたはサーバー終了時、データ整理プロセスを強化
* エンジンで定期的に残すログのマーカー追加
* ノード間の接続が切れた際、対象のログを残すように修正
* 特定の状況でのConcurrentModificationException発生を修正
* エンジン内部データサイズの最適化
* エンジン内部のエラーログが正常に出力されない場合がある問題を修正 
* 内部タイマーの最適化
* 内部Location動作の改善
* Gateway Bind失敗の状況で、サーバーが時々ゾンビ状態になる場合がある問題を修正 
* Timer引数がlongに変更され、引数がInteger値以上の場合に正常に動作しない問題を修正
* CPUコア数よりNode数が多い場合、サーバー起動時に警告が発生するよう修正
* ログアウトでルームから退出する際、時々正常に処理されない問題を修正
* 特定の状況でGameAnvilサーバー内部通信が正常に動作しない可能性がある問題を修正
* 内部メモリの最適化


---

### 1.4.2 (2024.02.26)

#### New
* Safe-Pause機能を改善しました。 
  * すでにSafe-Pauseが進行中の場合でも、進行中でないノードを出発地/到着地ノードに指定して新しいSafe-Pauseを実行できるように改善しました。 
  * GameNodeと共にMatchNodeもSafe-Pauseをサポートします。 

#### Fix
* エンジンのログ内容を補強し、可読性を改善しました。 
  * request失敗時に送信するパケットの詳細情報を追加しました。
  * システム情報ログに対するマーカーを、SYSTEM_INFO1つからSYSTEM_INFOとSYSTEM_WARNに分離しました。
  * MultiRequestで個別のリクエストが失敗した場合、どのリクエストが失敗したかログを残すようにしました。
  * マシン間接続状態に関連するログ内容を補強しました。 

#### Change
* GameAnvilConfig.jsonでmanagementIpと共にmanagementPortも設定できるように変更されました。

------

### 1.4.1 (2023.12.13)

#### New
###### エンジンProtobufバージョンを3.24.1にアップデート
###### Protobuf関連の便利機能を追加
  * send、replyでProtobufを直接使用できます。
  * パケットを直接Protobufに変更できます。(`MyProto.MyMessage resMsg = resPacket.toProtoBufferMessage();`)
  * 大部分の状況でretain releaseを呼び出さなくても正常に動作します。
###### Protocol登録時、indexを指定しなくてもよいように改善 
  * 既存のnumberはエンジンで自動的に割り当てます。

#### Fix
* サーバーでRequest呼び出し時、対象がない場合は即座に失敗レスポンスをするよう改善しました。
    * エンジンのrequestToGameUserのようなリクエスト(request) APIを呼び出した際、対象が見つからない場合、Timeoutまで待つ代わりに即座に失敗を返すよう改善しました。
* エンジンのログ内容を補強し、可読性を改善
    * エンジンログのうち、DEBUGとTRACEレベルを全て体系的に分離し、内容を理解しやすく整理しました。
    * ユーザーに関連するエンジンのログ内容を補強しました。
    * 一部のエンジンエラーメッセージを理解しやすいように改善しました。
* マネジメントノードは、ロケーションノードが構成される時のみ一緒に駆動するように改善しました。
    * マネジメントノードがロケーションノード構成位置で自動的に駆動されるため、これ以上ユーザーが直接設定して管理する必要はありません。また、駆動順序も気にする必要がありません。
* ログイン過程で誤ったチャンネルIDを入力した場合に対するレスポンスを改善しました。
    * 漠然としたSystemErrorレスポンスの代わりにログイン失敗レスポンスを返すよう修正し、より明確になりました。
* 登録されていないプロトコルがペイロードに含まれてゲームサーバーへ送信された場合、サーバーでエラーを処理し、クライアントへレスポンスを送信するよう改善
    * 従来のサーバーにWARNログのみ残す方式の場合、デバッグ過程でログを見逃す可能性がある問題があったため、クライアントへレスポンスを送信するように改善しました。
* Apache HttpComponentベースの新しいHttpClient2を提供します。
    * 既存のAsyncHttpClientオープンソースがこれ以上アップデートされないため、選択的に使用できるよう新しいライブラリベースのHttpClient2を提供します。
* ペイロードで圧縮パケットをサポートします。
    * 今後、ペイロードでも圧縮パケットを使用できるよう改善しました。
* マッチメイキング失敗時にクライアントへ通知を送信します。
    * マッチメイキングが失敗した際にも、クライアントへ適切な結果を送信するように改善しました。

#### Change
## 動的ログレベル設定
    * GameAnvilConfig.jsonでip項目の代わりにipcIpとmanagementIpを使用します。
    * BaseChannelInfoで正常にシリアライズしない問題を修正しました。BaseChannelInfoの代わりにGameAnvilServerでsetObjectSerializerを呼び出してシリアライズできます。
* matchUser、matchParty呼び出し時、Payloadを入れません。独自に保管します。
 * 全てのロケーションノード(マスターノード)のクラスタリングが完了した後にのみ、ロケーションノードへリクエストを送信できるよう修正しました。全てのロケーションノードのクラスタリングが完了したかを確認し、一定時間内に全クラスタリングが完了しない場合はエラーログを残す機能を追加しました。
 * ユーザーマッチング時、ランダムチャンネルでマッチングされたルームが生成されていた従来の方式から、マッチングをリクエストしたチャンネルの中からマッチングされたルームが生成される方式に変更しました。
  
------

### 1.3.1 (2023.04.20)

#### New

#### Fix

* 新しく起動したノードのチャンネル情報が更新されない問題を修正

#### Change

- Consoleで確認するSupportノードの有効状態チェックAPIを変更
- Locationノードが設定されたサーバーでのみManagementノードが起動するように変更

------

### 1.3.0 (2022.12.27)

#### New

###### Console 1.3との連携 
GameAnvil 1.3は、完全に新しくなったConsole 1.3と完璧に連携します。コンソールを通じて各ノードの状態を直感的に管理し、命令を下すことができます。Safe Pauseを行ったり、特定のノードを生成及び削除するなどの機能をコンソールを通じて便利に試すことができます。
</br> ※ 以前のバージョンのGameAnvilはConsole 1.3で使用できません。

###### 使いやすい同期機能
同期コンポーネントの提供により、Unityエンジンとの連携の利便性が大幅に強化されました。サーバーの複雑なロジック実装なしでも、リモートクライアント間でゲームオブジェクトの複数の要素が自動的に同期されます。

* GameObject生成、破壊の自動同期
* Animation同期
* Transform同期
* RigidBody及びRigidBody2D同期
* カスタム値同期機能

###### 性能最適化
GameAnvil 1.3は、以前のバージョンに比べてパケット処理性能が5%以上向上しました。内部動作コードを最適化し、より速い速度で動作します。

#### Fix

- Safe Pause関連バグ修正
  - Safe Pause進行中、出発地ノードにあるルームへマッチメイキングされる問題を修正しました。
  - 負荷テストとSafe Pauseを同時に実行する場合、MatchRoomExceptionが発生する問題を修正
  - Safe Pause過程でルーム転送を行う際、TimerObjectによりNPEが発生する問題を解決
  - Safe Pause進行時、ルームマッチメイキングが失敗する問題を解決
  - Safe Pauseを通じて転送されたルームの到着地ノードで、該当ルームと同一IDのルームが生成される問題を修正
    - ルームID生成はロケーションノードが担当するように修正
  - その他、ユーザー転送及びルーム転送に関連するバグ修正
    - ルーム転送失敗時、ロールバックされない問題を修正 
    - ルーム転送失敗時、結果パケットに失敗エラーコードを適用して送信するように修正
    - ルーム転送失敗時、生成したルームとユーザーをノードから照会して整理せず、直接整理するように修正
  - ルーム転送過程でクライアントが送信した一部のメッセージが消失する可能性を修正
    - ルームが他のノードへの転送完了時、タイミングの問題で既存ノードにルームが残り、ユーザーが入室できる状況を修正
      - 今後、転送中のルームはマッチメーカーのルームリストから除外される
      - 転送が完了し削除されるべき既存ルームや、転送が失敗したルームに入室しないように修正
    - ユーザー転送完了前に再接続時、目的地ノードでユーザーオブジェクトが見つからない問題を修正
    - ユーザー転送時、対象ノードに対して状態値を確認しないように修正
    - ルーム転送時、ユーザーの転送イン関連コールバック関数が呼び出されるように変更

- Safe Pause進行時、ルームマッチメイキングに失敗する現象を修正
  - 転送中のルーム状態の種類にTRANSFER_IN_PROGRESSを追加
  - 転送完了したルームがPAUSE状態に変わらないように修正
  - ルーム入室がルームファイバーで処理されようとする際、ルームが転送中の場合、ルーム入室処理を中止し転送を進行するように修正
	
* Roomとマッチメイキング関連バグ修正
  - CreateRoomメソッドで作成したルームをルームマッチメイキングに登録した後、該当ルームが終了すると例外が発生する問題を修正
    - ルームマッチ時、マッチルームが見つからない場合、ゲームユーザーにマッチユーザーカテゴリーを設定しないように修正

  - 全ユーザーが退出したルームに、タイミングの問題で新しいユーザーが入室できる問題を修正
    - ユーザー入室時に空きルームかどうかを確認するようにする
    - 最後のユーザーがルームから退出した直後にチャンネル情報整理及びMatchRefillキャンセル

  - クライアントでLeaveRoom呼び出し時、サーバーエラーが発生する問題を修正
    - UserPostLeaveRoomパケット送信主体をゲームユーザーファイバーに変更
  
  - ルームが終了した状態でルームを使用するAPI呼び出し時の例外であるClosedRoomExceptionを追加

* ゲートウェイ及びセッション関連バグ修正
  * ゲートウェイノードのセッションオブジェクトが転送(Transfer)状態でリークする問題を修正

  - セッション削除過程で例外が発生する場合、コネクションオブジェクトが削除されずに残る可能性がある問題を修正

* ノード関連バグ修正
  - 任意のノードを終了する場合、残りのサーバーでエラーまたは警告ログが残る問題を修正
    - 終了したノード情報をサーバー間で共有するように修正
    - 終了したノード情報を削除できるAPI追加

  - Shutdown状態のノードに、他のノードからパケットを送信する場合が発生する可能性がある問題を修正
    - 今後、終了告知後、実際の終了まで1秒間のディレイが発生するようになる

* その他
   * クライアントでリクエストに対するレスポンスを受け取れない問題を修正

  - 誤ったサービス名でサーバーを駆動する場合でも、全てのノードがReadyとして処理される問題を修正
    - ノード数を増加させるロジックのタイミングを修正
    - 誤ったサービス名を使用してノードを実行しようとする場合、ノードマネージャーですぐに終了するように修正


  - 重複ログイン中に強制接続終了処理される可能性があるバグを修正
    - DeviceIdが異なり、コネクションID、ゲートウェイノードIDが同じ場合に既存ユーザーをログアウトさせるケースについて、重複してセッションを閉じないように修正

  - ghostTimeout値が正しく適用されるように修正

  - ghostTimeout値がdemandClientStateCheck値より少なくとも3秒以上大きくなるよう制約事項を追加

#### Change

- Safe Pause高度化
  - 出発地ノードと到着地ノードの両方を選択するように変更
  - Safe Pause開始時に出発地ノードと到着地ノードを選択しないように変更
  - レポート機能強化：HTML形式以外にjson形式でレポートを生成する機能を追加

- Safe Pause関連プロトコル名変更及びReportSenderの動作をリクエスト方式に修正
  - 変更前：PubNonStopPatchReportStartCommand / PubNonStopPatchReportStartNode
  - 変更後： NonStopPatchReportStartCommandReq / NonStopPatchReportStartCommandRes / PubNonStopPatchReportStartNode		

- 同一ファイバーでリクエストを送信すると例外処理されるように変更

- BaseRoom使用方法変更
  - BaseRoomにonLeaveRoomメソッドのオーバーロードを追加
  - BaseRoomのonPostLeaveRoomのパラメータ削除によるユーザビリティ変更

- BaseUser使用方法変更
  - ユーザーがルームから完全に退出した後に呼び出されるonPostLeaveRoomメソッドを追加

- Disconnected状態値をisDetectedDisconnected,isCompleteDisconnectedに細分化
  - isDetectedDisconnectedは、サーバーでクライアントの接続が切れたと判断した際の状態値
  - isCompleteDisconnectedは、サーバーで接続を切る処理が完了した場合の状態値

- 固有HostId生成ロジック補強
  - ユーザーが入力したサブネットマスクに従ってHostIdを生成するように修正

- エンジンで使用するライブラリをアップデート

- UserTimer,RoomTimerに対する参照を返すAPI getUserTimer,getRoomTimerを追加

- DNS関連設定削除

---

### 1.2.0 (2021.07.13)

より詳細な情報は[リリースノート](https://nhnent.dooray.com/share/posts/sGAj_STlTEWDr5LPgKgIhg)を参照

#### New

* ユーザーライセンス適用
  
* [GameAnvilユーザーライセンス](https://gameplatform.toast.com/kr/services/gameanvil/license)
  
* ノード単位駆動及びスケーリング対応

  * ランタイムでも新しいノードを追加できる機能を追加
  * 従来に比べてより安定的にクラスタリングを結ぶように改善
  * ノード状態によるノード動作改善
  * ランタイムにノード単位で駆動または終了可能

* 簡素化されたブートストラッピング
  * GameAnvilBootstrap Deprecateに設定。今後のバージョンで削除される予定。
  * GameAnvilBootstrapの代わりにGameAnvilServerインスタンスを使用してGameAnvilサーバー実行。
  * 提供されるアノテーションを使用してBaseClassをGameAnvilから読み込むようユーザビリティが改善。

```java
// GameNode登録
@ServiceName("GameNode")
public class GameNode extends BaseGameNode {
}

// GameUser登録
@ServiceName("GameNode")
@UserType("GameUserType1")
@UseChannelInfo
public class GameUser extends BaseUser {
}
```


* 非同期MySQLクエリ対応
  * 非同期sql処理のためにJasync-sql / X Dev APIクラスを追加
  * ユーザーが簡単に非同期sqlを処理できるようサポート
  
* ユーザーガイドドキュメント補強
  * 1つのドキュメントに混在していた複数の重要なトピックをモジュール化し、ドキュメントの質と量をアップグレードしました。
  * チュートリアルを通じて、簡単ながらも素晴らしいジグソーパズルゲーム(サーバー＆クライアント)を直接開発してみることができます。 
  * JavaDoc APIリファレンスで内容が不足または漏れていた部分を全て補充しました。また説明が曖昧または誤解される可能性のある文章を全て修正しました。



#### Fix

* チャンネル情報管理及び同期機能リファクタリング
  * チャンネル情報を効率的に管理するよう改善
  * チャンネル情報をクライアントへ送信する機能を改善
  * チャンネルのルームとユーザーの個数を取得する機能を追加
  * チャンネルIDを設定しない場合(つまり""を使用する場合)、これ以上チャンネル機能を使用しないように改善。チャンネルを使用したい場合は、最小1以上の有効な文字列をチャンネルIDとして使用する必要あり。

* ルーム生成時、outPayloadがリリースされないバグを修正
* 同一アカウントのクライアントで再接続する前まで、ルームマッチメイキングが継続して失敗するバグを修正
* ユーザーマッチメイキング時にエラーが発生した場合、クライアントへレスポンスを送信しなかったバグを修正
* 発行(publish) API修正
  * ユーザビリティ改善
  * ユーザーが意図しないノードへpublishメッセージが送信されるバグを修正しました。あわせてpublish APIのユーザビリティを改善しました。
  * チャンネルIDにハングルを使用する際、publishメッセージが対象ノードを見つけられないバグを修正しました。



#### Change

* クラウドACL環境に合わせて基本ポートが変更されました。

| 名前 | ポート | 以前のポート |
| --- | --- | --- |
| ipcPort | 18000 | 16000 |
| publisherPort | 18100 | 13300 |
| gateway tcp | 18200 | 11200 |
| gateway web | 18300 | 11400 |
| management | 18400 | 25150 |
| support | 18600～18999 | ユーザー定義 |
| GameAnvil Agent | 19080 |  |
| GameAnvil Admin | 19090 |  |

* 今後、Adminはサポートしません。そのためGameAnvil内部ではこれ以上独自のDBを使用しません。

* 今後、Dynamic Moduleはサポートしません。

* ZMQライブラリ交換
	
	* libzmqとjzmqの組み合わせからJeromqへ交換
	
* ルームマッチメイキングリファクタリング及びユーザビリティ改善
  * ルームマッチメイキングフロー改善
  * 負荷テスト進行時にルームが見つからない問題を解決し、負荷テストの進行が可能に
  * ルーム人数情報管理機能を追加し、ユーザーがルーム人数管理機能を実装しなくてもよいように
  
* 認証とログイン、そして接続終了コードの最適化

    * Connectionで呼び出していたコールバックをSessionへ移動
    * DisconnectAlarmの原因把握を容易にするよう改善

* ユーザー提供トピック整理

| 名前 | 説明 | 基本登録位置 |
| --- | --- | --- |
| GameAnvilTopic.GAME\_NODE | 全ゲームノード | GameNode |
| GameAnvilTopic.GATEWAY\_NODE | 全ゲートウェイノード | GatewayNode |
| GameAnvilTopic.SUPPORT\_NODE | 全サポートノード | SupportNode |
| GameAnvilTopic.ALL\_CLIENT | 接続中の全てのクライアント | Session |
| GameAnvilTopic.ALL\_GAME\_USER | ゲームノードにある全ゲームユーザーオブジェクト | GameUser |


* 例外処理及びメソッドシグネチャ最適化


  * 無分別に使用された例外処理とメソッドシグネチャを整理し、ユーザーに必要な情報のみ渡すように修正

* HttpRequestにPATCHメソッド追加
* 証明書なしでテスト用にSSLを使用できる機能設定名がuseSelfからuseSelfSignedCertに変更されました。

---

### 1.1.12 (2021.06.07)
#### Change

* SupportNodeにGateway接続情報を受け取れるAPI追加
    * BaseSupportNode.getGatewayConnectionInfoList関数を使用して、Gateway接続情報リストを提供

```json
// Jsonに変換した情報リスト
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": [
    {
      "ip": "127.0.0.1",
      "dns": "",
      "connectionInfo": {
        "TCP_SOCKET": 11200,    // 該当ConnectionTypeによるポート情報を提供
        "WEB_SOCKET": 11400
      }
    }
  ]
}
```
---

### 1.1.11 (2021.05.06)

#### Change

* HttpRequestにPATCHメソッドを使用できるAPI追加
    * httpRequest.PATCH()
    * httpRequest.PATCH(AsyncHttpCompletionHandler asyncHttpCompletionHandler)
    * httpRequest.PATCHAsync()

---

### 1.1.10 (2021-04-21)

#### Fix

* RedisSingleに漏れていたpassword対応API追加

---

### 1.1.9 (2021-04-16)

#### Fix

* クライアントのPauseClientStateCheckを受信した際、idleClientTimeoutチェックも同時に停止するように修正。
* クライアントのPauseClientStateCheckから受け取ったpauseTimeの単位を秒で計算するように修正。

---

### 1.1.8 (2021-04-15)

#### New

* クライアントのPauseClientStateCheckを受け取り、入力された時間分、該当クライアントの状態チェックを行わない機能を追加。
* クライアントのResumeClientStateCheckを受け取り、状態チェックを再開する機能を追加。
* GameAnvilConfig.jsonのGatewayにPauseClientStateCheck関連の許容範囲を制限する設定値を追加
    * pauseClientStateCheckMaxDuration = クライアントが送信した時間値が該当数値より大きい場合、該当数値に制限
    * pauseClientStateCheckMinDuration = クライアントが送信した時間値が該当数値より小さい場合、該当数値に制限

---

### 1.1.7 (2021-04-02)

#### Fix

* /game-data/get使用時、GameData値がJsonObjectではなくStringで送信される問題を修正

---

### 1.1.6 (2021-03-30)

#### Change

* Dynamic Module機能がGameAnvilから仕様外となり削除されました。
  * 既存ユーザーのためにDynamic Moduleソース公開と使用ガイドを作成して共有します。該当ガイドを参考にして、GameAnvilではなくユーザープロジェクトにDynamic Moduleソースを追加して使用できます。
  * [GameAnvil-Guide/81 Dynamic Module仕様外によるソース提供及び使用法](https://nhnent.dooray.com/share/posts/xtwdO6FJTXmr7l28cZSM-w)

---

### 1.1.5 (2021-03-19)

#### Change

* GameAnvil DB & Admin機能(GameData、Dynamic Module除く)削除

    * GameAnvilエンジン側でこれ以上DBを使用しません。
    * 参考: [GameAnvil-Guide/76 Admin 1.1.0 -&gt; 1.1.1マイグレーションドキュメント](https://nhnent.dooray.com/share/posts/GIHoRQ8kTjSi7D4wQjUcWw)
    * 従来DBを使用していたAdmin機能は全てGameAnvil Adminへ移動されました。

```json
"management": {
  "nodeCnt": 2,
  "restIp": "127.0.0.1",
  "restPort": 25150,

  // 削除
  //"db": {
  //  "user": "root",
  //  "password": "1234",
  //  "url": "jdbc:h2:mem:gameanvil_admin;DB_CLOSE_DELAY=-1"
  //}
}
```

---

### 1.1.4 (2021-03-18)

#### Fix

* ルームマッチングにMatchingGroup適用時、断続的に生成されたルームにJoinできない問題を修正

---

### 1.1.3 (2021-02-05)

#### Fix

* インスタンス再起動時、Disable状態から復旧しない問題を修正
    * 低い確率で再現される可能性があるが、再起動する場合復旧される

* ManagementNodeでnodeInfoByHostId使用時、Double.infinityエラーが発生する問題を修正

---

### 1.1.2 (2021-01-07)

#### Fix

* DynamicModuleでSuspendExecution例外が発生する可能性のあるコード呼び出し時の例外処理が漏れており、DynamicModuleのメソッドが正常に呼び出されない問題を修正

---

### 1.1.1 (2021-01-05)

#### Fix

* GatewayNodeでgetNodeId()を呼び出すとNPEが発生する問題を修正

* 同一アカウントのクライアントで再接続する前まで、該当GameUserでルームマッチングが継続して失敗する問題を修正

---

### 1.1.0 (2020-12-17)

より詳細な情報は[リリースノート](https://nhnent.dooray.com/share/posts/bWby9jGjQri2cFNR_KYbnw)を参照

#### New

* Jdk11サポート

  * Jdk8サポートバージョンに加え、Jdk11サポートバージョンを配布しました。
  * GameAnvil version名が変更されました。

```xml
// jdk8
<dependency>
    <groupId>com.nhn.gameanvil</groupId>
    <artifactId>gameanvil</artifactId>
    <version>1.1.0-jdk8</version>
</dependency>
```

```xml
// jdk11
<dependency>
    <groupId>com.nhn.gameanvil</groupId>
    <artifactId>gameanvil</artifactId>
    <version>1.1.0-jdk11</version>
</dependency>
```

* Alarm

    * サーバー内で発生した状況を指定したURLで受け取れる機能を追加しました。
    * "-Dalarm.url" VMオプションを使用して、Alarmを受け取るURLを指定できます。
    * [GameAnvil-Guide/69 Alarm使用方法](https://nhnent.dooray.com/share/posts/Ap6DJT9KSaGv916_tj-xAA)

#### Change

* BaseObjectのfindAllUserLocsOfAccount戻り値をUserLocへ変更

* Node情報をRest APIで取得できる機能を追加しました。
    * /management/ipcNetworkNodeInfo
    * /management/managementNetworkNodeInfo
    * /management/locationLookupNodeInfo
    * /management/matchNodeInfo

#### Fix

* 同一AccountIdで異なるSubIdを使用する場合、ログインは成功するが、その後レスポンスパケットを受け取れない問題を修正しました。
* GatewayNode、MatchNode、SupportNode、ManagementNodeのNodeNumが1から始まる問題を修正しました。
* Nettyのsend、recv bufferサイズ最適化とそれに適合したwatermarkが適切に適用されるよう修正

---

### 1.0.7 (2021.01.05)

#### Fix
- GatewayNodeでgetNodeId()を呼び出すとNPEが発生する問題を修正

---

### 1.0.6 (2020.12.28)

#### Fix
- `RoomMatchMaking failure. The matched-room({}) does not exist in the game node.` ログが残りながらルームマッチングが失敗する場合、該当ユーザーは再接続するまで継続してルームマッチングが失敗する問題を修正

---

### 1.0.5 (2020.11.17)

#### Fix

- RoomMatchReqでエラーレスポンス時、パケットヘッダ復元(restore)ができない問題を修正

---

### 1.0.4 (2020.10.29)

#### Fix

- 任意のSessionにまだ何のパケットも送信されたことがない場合に、このSessionへsend呼び出し時NPE(Null Pointer Exception)が発生する問題を修正
- publishToClient使用時、問題発生(Notice含む)
- Roomで使用する繰り返しTimerにより、ルーム破棄が遅延する問題を修正
- protobuf構造体を変数として持つClassをdeserializeする際に例外が発生する問題を修正
- java.lang.UnsupportedOperationExceptionが発生

---

### 1.0.3 (2020.10.26)

#### Change

- 以前仕様から除外されていたお知らせ機能APIを再度有効化

---

### 1.0.2 (2020.10.12)

#### New

- Logに表示されるNode情報をユーザーが自由に設定できる機能を追加
- HostIdを直接入力できる機能を追加
- nodeInfoByHostIdでHostId別に情報を送信する機能を追加

#### Fix

- IP 0.0.0.0をバインディングする際にエラーが発生する問題を修正
- ConfigModuleの/config/get使用時、成否の値が失敗でレスポンスする問題を修正
- GameNodeが駆動された状態で他のGameNodeが駆動される場合、Publish Packet処理によりエラーが発生する可能性のある問題に例外処理を追加

#### Change

- UserId,RoomId生成時、最後の桁に使用するShardIndex値の範囲を指定できるように修正
- RestObjectのgetRemoteAddress値を追加
- NodeKeyリクエスト数を指定できるように修正

---

### 1.0.1 (2020.09.07)

#### Fix

- onLogin()でfalseを返す場合、Payloadがクライアントへ送信されない問題を修正
- DebugとTraceログ文の前のログレベル確認条件文が漏れていた部分を追加

---

### 1.0.0 (2020.08.31)

より詳細な情報は[配布ノート](https://nhnent.dooray.com/share/posts/5Fvh0aszQ5u6d_ZZxRWPvA)を参照

#### New

- 性能と安定性が大幅に向上(0.9バージョン対比約7倍)
- 性能に直接的な影響を与える多数のロジックを最適化し問題点を修正
- 完全に新しくなったGameNodeロードバランシング
- クラウドVMの状態を反映し、迅速かつ正確に最適なGameNodeへ負荷を分散
- Integer型ID導入
- 従来Stringとなっていた核心IDを全てIntegerへ交換し最適化を進行
- サーバーLogレベルをランタイムに変更できるAPIを追加
- Node単位でサービス中にLogレベルを変更可能

#### Fix

- ユーザー転送問題を修正、使用法改善及び最適化
- GameAnvilユーザーはより簡単にユーザー転送を使用可能
- 無停止パッチの完全修正
- これまでルーム転送と無停止パッチが正常に動作しなかった問題を全て修正
- サーバー駆動時、不明なタイミングの問題で一部Nodeが正常に起動しなかった問題を修正
- Nodeの一部のTimerオブジェクトが解除されず持続的に累積していたMemory Leakを修正
- クライアント接続チェック過程で新しく接続するユーザーが意図せず切断されていた問題を修正

#### Change

- Node名を以下の表のように、より直感的に変更

| v1.0       | ～ v0.10       | 必須 | 機能                                      | コンテンツ実装 | ネットワークアクセス   |
| ---------- | ------------- | ---- | ------------------------------------------- | ----------- | ----------------- |
| Gateway    | Session       | 必須 | クライアント接続と認証を処理             | 可能      | public            |
| Game       | Space         | 必須 | 実際のゲームサーバーとしてコンテンツを処理             | 可能        | private           |
| Support    | Service       | 任意 | 必要に応じて独立したサービスとして実装するよう支援 | 可能        | private or public |
| Match      | Match         | 任意 | マッチメイキングを実行                           | 可能        | private           |
| Location   | Location      | 必須 | ユーザーやルームなどの位置情報を保存及び管理      | 不可      | private           |
| Management | Management    | 必須 | サーバー情報集約及びAdmin/Agentと通信        | 不可      | private           |
| Ipc        | Communication | 必須 | GameflexサーバーのInter-process通信処理   | 不可    | private           |

- GameAnvilコード可読性とユーザビリティ向上
- パッケージ構造を効率的に改善
- クラス継承/包含構造の改善
- ネーミングを直感的かつ一貫性のあるものに修正
- Nodeの有効状態はInvalidとDisableへ細分化
- Invalidは一時的に反応しない状態
- Disableは永久的に無効になっている状態
- 今後Loopbackアドレスは自動的にバインディング
- GameAnvilで使用する全てのライブラリを最新バージョンへアップデート
- 全体ErrorCodeを1つに統合
- 不要なReConnectとMoveService機能を削除
- 登録時点ではなく駆動時点から次の時間を測定する新しいTimer追加

---

### 0.10.2 (2020.04.08)

#### New

- GameAnvil APIリファレンス(JavaDoc)サイトオープン

- AOT Instrumentationサポート

- エンジンをランタイムに毎回JIT Instrumentationする必要なく、コンパイルタイムにAOTで進行

- 公式GCとしてG1GCを使用するようガイド開始

- 最も基本的なVMオプション

  `java -javaagent:QUASAR_PATH\quasar-core-0.7.10-jdk8.jar=bm -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication`

#### Fix

- Quasarスタックバグを修正
- ルームマッチングユーザーが別々のルームへマッチングされるバグを修正
- 圧縮されたパケットが送信されない問題を修正

---

### 0.10.1 (2020.02.11)

#### New

- ユーザーへ提供するツールをまとめたUtilクラスを提供
- 従来はエンジンのあちこちに散らばっていたツールを該当クラス一箇所へ整理

#### Change

- Topic処理コード性能改善


---

### 0.10.0 (2020.02.06)

#### New

- 従来問題が多かった非同期サポートAPIを新しくリファクタリング
- Redis Clusterサポート及びLettuce公式サポートによる新しい非同期API追加
- 非同期HttpRequestとHttpResponse API追加
- ByteStringベースのカスタムプロトコル機能サポート
- Google protobuf以外にユーザーが希望する方式でメッセージをシリアライズ可能

#### Fix

- 以前のバージョンで最も深刻な問題だったファイバーの状態が壊れる問題を全て修正

#### Change

- エンジンで使用するパケットとヘッダサイズの最適化
- 0.9バージョン対比、性能が約2.5倍向上
- Embedded Redis Spec-out
- インフラで提供するサービスを使用するようにガイド

---

### 0.9.9 (2019.10.25)

#### New

- プロセス単位でWatchDogを連携できるようポートを追加で開放

#### Fix

- ゴーストユーザーバグ修正
- SpaceNodeで発生するConcurrent Modificationエラー修正
- NodeInfoManagerのComparatorエラー修正
- ログアウトしたユーザーのセッションオブジェクトが整理されない問題を修正
- マッチンググループが別々のチャンネルのユーザーへ正常に適用されなかった問題を修正
- パーティーマッチメイキング時にTimeoutに対するコールバックを正常に受け取れなかった問題を修正

#### Change

- ルームマッチメイキングをすでに進行中の状態で重複リクエストが来た場合のエラーコードを追加
- MATCH_ROOM_FAIL_IN_PROGRESS

---

### 0.9.8 (2019.09.05)

#### New

- SessionにClientTopicを追加及び削除できるaddTopic(),removeTopic() APIを追加
- Packet TTL機能追加
- Dangling Locationを定期的にチェックして整理するロジック追加

#### Fix

- AdminでユーザーをKickする場合、クライアントへForceLogoutNotiを送信しないように修正
- LocationNodeのSpot誤動作修正

#### Change

- onAuthenticate()コールバック失敗時に該当ソケットを閉じる前にForceCloseNotiをクライアントへ送信するように変更


---

### 0.9.7 (2019.07.27)

#### New

- 異常に残っているRoomを定期的にチェックして整理するロジック追加
- ユーザーマッチメイキングはリクエスト後に再ログインしても以前のマッチリクエストがキャンセルされずに進行中の場合があるため、このような以前のユーザーマッチメイキング処理状態を知らせるために再ログインレスポンスに新しいフラグを追加
- Packet Expire機能追加(デフォルト30秒)

#### Fix

- Connection ID(CID)が重複する問題を修正
- Disconnectionされたユーザーによりルームファイバーがcloseされるバグを修正
- ログインが失敗の場合、レスポンスメッセージにserviceIdフィールドが正常にセットされない問題を修正
- コンテンツコールバック処理部のうちtry-catchが漏れていた部分を修正
- クライアントReconnectに約3秒以上かかっていた問題を即時処理されるよう修正
- ゴーストユーザーチェック時に発生するスレッド同時性問題を修正
- ルームマッチメイキング重複処理バグ修正
- ルームに接続が切れたユーザーが残る問題を修正

#### Change

- Session Disconnectの全てのケースについてログを追加
- CreateRoomは現在のノードに直接ルームを生成するように変更


---

### 0.9.6 (2019.06.23)

#### New

- 転送可能な時点をコンテンツで調整できるようUserとRoomにcanTransfer()インターフェースを追加

#### Fix

- エンジン内部コードに存在していたメモリリーク修正
- ZMQ Router,Publisherソケットの接続バグ修正
- Loginが無限ループする問題を修正
- isLogined() APIが正常に動作しない問題を修正
- NodeInfoManager同期問題を修正
- ユーザーオブジェクトが正常に整理されない問題を修正

#### Change

- 既存のAsyncAwaitHttpRequestをFiberHttpRequestに名前変更
- FiberHttpRequestとHttpRequestに非同期関数を追加
- Location Index関連ログ追加
- SpaceNode分配方式をidleからroundRobinに変更
- NodeStarterでLocationNodeがある場合のみsleepするように変更


---

### 0.9.5 (2019.06.21)

#### Fix

- 無停止パッチ時に新しいユーザーがログインしたりユーザー転送中の場合、Pauseされたノードで該当ユーザーが整理されないエラー修正

#### Change

- 文字列形式のserviceIdを整数型に変更
- プロトコルのservice_codeをservice_idに変更し関連コード修正
- RESTfulサポートAPIの内部実装と使用方法変更


---

### 0.9.4 (2019.05.24)

#### New

- コンテンツで使用するプロトコルをファイル単位で登録できる機能追加
- 内部パケット処理を効率化するため
- MoveService機能追加

#### Fix

- ルームマッチメイキングで使用する比較演算子(Comparator)のエラー修正

#### Change

- Authenticationレスポンスに直近のログイン情報を追加し次回ログイン時に活用可能
- setReady()を明示的に呼び出してonReady状態へ移行するように変更
- LocationNodeがCommunicationNodeのonPrepare時点で初期化されるように変更


---

### 0.9.3 (2019.04.10)

#### Fix

- AsyncAwait.call() APIでtimeoutが発生するとNPE(Null Pointer Exception)も発生する問題を修正
- Epoll有効化時に停止する問題を修正
- ログイン時に発生するInvalid System Target Locationエラー修正

#### Change

- ManagementNodeはこれ以上エンジンユーザーが拡張して使用不可
- 代わりにServiceNodeを使用
- AdminでMachine情報の登録、修正、削除がリクエストした瞬間に即時適用されるように変更


---

### 0.9.2 (2019.02.11)

#### New

- Adminを利用した予約お知らせ、予約メンテナンス機能追加
- Adminを利用したWhite IP,White User,White Device機能追加

#### Fix

- AsyncAwait.run() APIで誤った方式で例外が処理されていたコード修正
- ManagementNodeが他のNodeより先に初期化されるように修正
- チャンネルIDを使用しない場合、一部のチャンネル関連publishが漏れる問題を修正
- CreateRoom,JoinRoomについてコンテンツが失敗を返す場合に誤ったエラーコードがヘッダに含まれる問題を修正
- パケットヘッダ生成時に発生していたメモリリーク修正
- ルームマッチメイキングで発生していたメモリリーク修正

#### Change

- onLogout()コールバックにpayload追加
- ResultCode修正に伴い、いくつかのプロトコルで重複フィールドを削除
- Dynamic Moduleユーザビリティ改善


---

### 0.9.1 (2018.11.12)

#### New

- マッチメイキングを専任するMatchNode追加
- ユーザーマッチメイキングにrefill機能追加
- ユーザーマッチメイキングの新しいタイプであるパーティーマッチメイキングを追加
- AdminのMachine情報でノード種類別に追加情報を追加
- ユーザーが同一DeviceIdとUserIdで再ログインする際に呼び出されるonLoginByOtherConnection()コールバック追加

#### Fix

- Dynamic Moduleバグ修正
- Admin接続ができない問題を修正
- Userのreplyバグ修正
- Timer追加/削除エラー修正
- AsyncAwait.run()に漏れていたtimeout適用
- ノードShutdown時に発生していたエラー修正
- ルームマッチメイキングを使用しないにも関わらずマッチング情報を削除する際にNPE(Null Pointer Exception)が発生する問題を修正

#### Change

- 一部インターフェースの位置変更
- 異なるDeviceIdでログイン時に重複ログイン処理の代わりに再ログイン処理になるように変更
- ルームマッチメイキングのonMatchRoom()コールバックで追加したpayloadが以降のフローに含まれるように修正


---

### 0.9.0 (2018.06.27)

#### New

- ユーザーマッチメイキング機能追加
- チャンネル別ユーザーとルームリスト機能追加
- Embedded Redis追加
- Reconnect機能追加
- (旧) Test Agent追加

#### Fix

- Account単位の重複ログイン処理問題を修正

#### Change

- OracleJDKからAdoptOpenJDKに変更
- Node(スレッド)単位でZMQ通信していたものをプロセス単位に変更

---

### 0.8.6 (2018.06.28)

#### New

- Send from Session to Management機能追加
- Request from Session to Management機能追加
- User MatchMaker追加
- Channel User List機能追加
- Custom Serializer機能追加
- RoomFinderにCustom Serializerをplug-in可能

#### Fix

- チャンネルユーザーマネージャーでチャンネルがtopicとして使用されるpublishに対する空文字列例外処理追加
- チャンネルがtopicとして使用されるpublishに対する空文字列例外処理追加
- AutoJoinRoomを使用しない環境でdelNamedRoomInfo()とdelRoomInfo()呼び出し時にDefaultRoomFinderにより発生する問題を修正
- RoomFinderでnullチェック漏れにより発生するNullPointerException修正
- ユーザーマッチメイキングでtimeoutされたリクエストが削除されない問題を修正
- redisにpasswordが設定されている場合に対する処理追加
- cfgManagementのsetRestIp()エラー修正
- ServiceノードでHttpハンドリングができない問題を修正。
- ブランチ: 0.8.6.2_fix_service_node_rest_handling
- タグ: tardis-0.8.6.2.1
- AutoJoinRoom処理時にgameIdが設定された場合のエラー修正
- MatchMaker修正及び例の補強
- ISession::onDisconnect()漏れ修正
- Communication Node間Node情報交換エラー修正
- FindRoomList性能問題解決
- Shutdownバグ修正
- BaseNodeがpublishToNodeメッセージを受信できない問題を修正
- FindRoomList性能問題解決
- Shutdownバグ修正
- BaseNodeがpublishToNodeメッセージを受信できない問題を修正
- NamedAutoJoinRoomバグ修正
- UpdateRoomInfo/DelRoomInfoが該当Nodeで即時反映されるように修正
- ファイバー内でblocking callを呼び出す部分により出力されていたError修正

#### Change

- FindRoomListは全ルームリストcontainerを一度にserialize/deserializeするように修正
- 既存のLocationException,TardisExceptionなどをRuntimeExceptionに変更、Exception整理
- forceLeaveRoomはkickOutRoomにrename
- forceLogoutはkickOutにrename
- tardisConfig構成変更

---

### 0.8.5 (2017.12.18)

#### New

- JWT認証トークンを使用したHTTPセッション認証及び管理機能追加
- 同一IDを使用し、複数Service重複ログインできる機能追加
- 1つのGameNodeに複数のRoomTypeを使用できる機能追加
- Serverの全情報をWebで閲覧できる機能追加 (NodeInfoPage)

#### Fix

- タイマー削除ができないバグ修正
- onPostLeaveRoomが2回呼び出されるバグ修正

#### Change

- ゴーストユーザー処理改善
- ゴーストユーザーロギング追加
- Packet compressロジック改善
- RestObject改善、使用関数追加
- パケットのエラー処理変更、プロトコルヘッダにエラーコードを含めて処理

---

### 0.8.4 (2018.01.10)

#### New

- Bootstrap適用
- NGB Bootstrapと同様の形態で実装
- 実装部それぞれのクラスのclassインスタンスのみ明示してコードを単純化
- 複数のメッセージを送受信できるMulti Dispatch機能追加
- ServiceNodeSender(以前のCustomServiceSender) publishToLobbyメソッド追加

#### Change

- AsyncAwait, RAsyncAwait名前変更
- Bootstrap RoomFinder設定をspaceへ移動
- Cache,Custom Exception名前変更 (Location, Service)
- PayloadクラスでgetFirstメソッドを通じて最初のPacketを取得できるように修正
- ServiceNodeSenderのrequestToLobbyメソッド削除(同一機能のrequestToNodeメソッドと重複する問題)
- PacketクラスのmakeDecompressメソッド呼び出し時asyncを使用しないように修正
- UserSenderのreplyメソッドオーバーロード(Packetリスト処理)
- 一部クラス名変更
- AsyncCall -> AsyncWaitingCall
- AsyncRun -> AsyncWaitingRun
- ReverseAsyncCall -> RAsyncWaitingCall
- ReverseAsyncRun -> RAsyncWaitingRun

#### Fix

- Session IPエラー修正

---

### 0.8.3 (2017.10.26)

#### New

- RESTハンドリング機能追加
- TardisConfig.jsonにpermissionGroupsとしてRESTハンドリングに対し、Path & IPベースのACL機能実装
- Channel List (チャンネル名リスト),Channel Information (特定チャンネルの詳細情報)
- Space LobbySenderにrequestToCustom,sendToCustomメソッド追加
- SpaceノードManagement情報に"State"項目追加
- Ilobby,ISessionLobby,IServiceインターフェースに以下void onShutdown() throws SuspendExecution追加
- Nodeがshutdownされる際、contentsでresourceを解放できるようonShutdown()インターフェースを追加
- ServiceNodeSenderにpublishToNode関数追加

#### Fix

- ManagementのJMX Management API修正及び追加(SessionGateway, CustomGateway)
- Channel List(チャンネル名リスト)、Channel Information(特定チャンネルの詳細情報)機能追加
- Session,CustomノードのRESTハンドリングロジック修正
- Gatewayノード(Session, Custom) isConnectableTypeメソッドの戻り値修正
- SessionGateway及びCustomGatewayラウンドロビン問題及びJMX戻り値情報修正
- キャッシュノードtargetMapキー値使用コード修正
- SessionGatewayノードREST RoundRobinコード修正
- ログインプロセス改善 (Invalid Target Locationエラーログ関連修正)
- ルームを削除するdelRoomInfo呼び出し時、該当ノード状態がREADYでなければリターン処理

#### Change

- SessionノードとクライアントのEnd-Pointを1つに統合 (Session Gateway)
- CustomノードのREST End-Pointを1つに統合 (Custom Gateway)
- ファイバー直接使用コードをReverseAsyncCallに変更
- SpaceSingular Dispatchタイムスタンプ追加
- onTimer登録し、定期的にチェック (10分)
- タイムスタンプが超過したユーザーはinternalLogout処理
- Asyncタイムアウト適用
- ゴーストユーザー処理ロジック追加及び改善、TardisConfig.jsonに必要な値を追加
- 各Responseプロトコルのresult_code形式を既存integer値からenumに変更し、種類を簡素化。is_payloadフィールドを削除
- クラス、パッケージ、関数とcfg内容がrename
- Request,Responseプロトコルで複数のpayloadを処理できるようclass PayloadデータタイプがPacketに変更、class OutPayload削除
- ユーザーTransfer-Inインターフェースのパラメータタイプをbyte配列に変更

---

### 0.8.2 (2017.09.21)

#### New

- CustomServiceにRestObject機能追加
- CustomServiceにRestHandler追加

#### Fix

- Cacheでユーザー情報が削除されないバグ修正
- エンジン内部で発生していたNPE(Null Pointer Exception)に対する例外処理追加
- パケット送信にCID(Connection ID)を正常に適用できなかった問題を修正

#### Change

- Customモジュールネーミングを全て新しく変更
- 例) CustomLobby -> CustomService
- Customモジュールインターフェース一部変更
- CustomServiceSenderのsendToUser()とrequestToUser() APIにserviceIdパラメータ追加
