## Game > GameAnvil > リリースノート > GameAnvil
### 1.4.2 (2024.02.26)

#### New
* Safe-Pause機能を改善しました。 
  * 既にSafe-Pauseが進行中であっても、進行中でないノードを出発/到着ノードとして指定することで、新しいSafe-pauseを実行できるように改善しました。
  * GameNode に加え、MatchNode もSafe-Paueをサポートします。 

#### Fix
* エンジンのログの内容を補強し、読みやすさを改善しました。
  * request失敗時に送信するパケットの詳細情報を追加しました。
  * システム情報ログのマーカーをSYSTEM_INFOからSYSTEM_INFOとSYSTEM_WARNに分離しました。
  * MultiRequestで個々のリクエストが失敗した場合、どのリクエストが失敗したかログを残すようにしました。
  * マシン間の接続状態に関連するログ内容を補強しました。

#### Change
* GameAnvilConfig.jsonでmanagementIpに加えてmanagementPortも設定できるように変更しました。

------

### 1.4.1 (2023.12.13)

#### New
###### エンジンのProtobufバージョンを3.24.1にアップデート。
###### Protobuf関連の便利機能を追加
  * send, replyでProtobufを直接使用できます。
  * パケットを直接Protobufに変更できます。 (`MyProto.MyMessage resMsg = resPacket.toProtoBufferMessage();`)
  * ほとんどの状況でretain releaseを呼び出さなくても正常に動作します。
###### Protocol登録時にindexを指定しなくてもよくなりました。
  * 既存のnumberはエンジンが自動的に割り当てます。

#### Fix
* サーバーからRequestを呼び出す際に対象がなければ、直ちに失敗レスポンスを送信するように改善しました。
    * エンジンのrequestToGameUserのようなリクエスト(request)APIを呼び出した時、ターゲットが見つからない場合、Timeoutまで待つのではなく、すぐに失敗を返すように改善しました。
* エンジンのログ内容を補強し、可読性を改善
    * エンジンログのうち、DEBUGとTRACEレベルをすべて体系的に分離し、内容を理解しやすく整理しました。
    * ユーザーに関連するエンジンのログ内容を補強しました。
    * 一部のエンジンのエラーメッセージを分かりやすく改善しました。
* マネジメントノードは、ロケーションノードが構成されている場合にのみ、一緒に駆動するように改善しました。
    * 管理ノードがロケーションノード構成位置で自動的に駆動されるため、ユーザーが直接設定し、管理する必要がなくなりました。 また、駆動順序も気にする必要がなくなりました。
* ログインプロセスで誤ったチャンネルIDを入力した場合のレスポンスを改善しました。
    * 漠然としたSystemErrorレスポンスの代わりにログイン失敗レスポンスを送信するように修正し、より明確性が増しました。
* 登録されていないプロトコルがペイロードに含まれてゲームサーバーに転送される場合、サーバーでエラーを処理し、クライアントにレスポンスを転送するように改善しました。
    * 既存のサーバーにWARNログだけを残す方式の場合、デバッグの過程でログを見逃す問題があり、クライアントにレスポンスを転送するように改善しました。
* Apache HttpComponentベースの新しいHttpClient2を提供します。
    * 既存のAsyncHttpClientオープンソースが更新されなくなったため、選択的に使用できるように新しいライブラリベースのHttpClient2を提供します。
* ペイロードで圧縮パケットをサポートします。
    * ペイロードにも圧縮パケットを使用できるように改善しました。
* マッチメイキング失敗時にクライアントに通知します。
    * マッチメイキングが失敗したときにも、クライアントに適切な結果を伝えるように改善しました。

#### Change
* 設定
    * GameAnvilConfig.jsonでip項目の代わりにipcIpとmanagementIpを使用します。
    * BaseChannelInfoで正常にシリアル化しない問題を修正しました。 BaseChannelInfoの代わりにGameAnvilServerでsetObjectSerializerを呼び出してシリアル化できます。
* matchUser、matchParty呼び出し時にPayloadを入れないようにしました。 独自に保管します。
 * すべてのロケーションノード(マスターノード)がクラスタリングが完了した後にのみロケーションノードにリクエストを送信できるように修正しました。 すべてのロケーションノードがクラスタリングが完了したことを確認し、一定時間内に全体クラスタリングが完了しない場合、エラーログを残す機能を追加しました。
 * ユーザーマッチングの際、ランダムチャンネルからマッチングされたルームが作成される従来の方式から、マッチングを要求したチャンネルの中からマッチングされた部屋が作成される方式に変更しました。
  
------

### 1.3.1 (2023.04.20)

#### New

#### Fix

* 新しく起動したノードのチャンネル情報が更新されない問題を修正

#### Change

- Consoleで確認するSupportノードの有効化状態チェックAPIを変更
- Locationノードが設定されたサーバーでのみManagementノードが起動するように変更

------

### 1.3.0 (2022.12.27)

#### New

###### Console 1.3との連動 
GameAnvil 1.3は完全に生まれ変わったConsole 1.3と完璧に連動します。コンソールを通じて各ノードの状態を直感的に管理し、コマンドを発行できます。Safe Pauseの進行や、特定のノードを作成して削除するなどの機能をコンソールを通じてより快適に試すことができます。
</br> ※以前のバージョンのGameAnvilはConsole 1.3で使用できません。

###### 便利な同期機能
同期コンポーネントの提供により、Unityエンジンとの連動利便性が大幅に強化されました。サーバーの複雑なロジックを実装しなくても、遠隔のクライアント間でゲームオブジェクトのさまざまな要素が自動的に同期されます。

* GameObject作成、破壊の自動同期
* Animation同期
* Transform同期
* RigidBodyおよびRigidBody2D同期
* カスタム値同期機能

###### 性能の最適化
GameAnvil 1.3は、以前のバージョンに比べてパケット処理性能が5%以上向上しました。内部動作コードを最適化して、より高速で動作します。

#### Fix

- Safe Pause関連のバグを修正
  - Safe Pauseの進行中に出発地ノードにあるルームにマッチメイキングされる問題を修正しました。
  - ストレステストとSafe Pauseを同時に実行した場合にMatchRoomExceptionが発生する問題を修正
  - Safe Pauseプロセスでルームの転送をする際に、TimerObjectによりNPEが発生する問題を解決
  - Safe Pauseの進行時にルームマッチメイキングが失敗する問題を解決
  - Safe Pauseを通じて転送されたルームの到着地ノードで該当のルームと同じIDのルームが作成される問題を修正
    - ルームIDの作成はロケーションノードで担当するように修正
  - その他、ユーザー転送およびルームの転送関連のバグを修正
    - ルームの転送に失敗した際にロールバックされない問題を修正 
    - ルームの転送に失敗した場合、結果パケットに失敗エラーコードを適用して送信するように修正
    - ルームの転送に失敗した際に作成したルームとユーザーをノードで照会して整理せず、直接整理するように修正
  	- ルーム転送プロセスでクライアントが送信したメッセージの一部が失われる可能性を修正
    - 他のノードへのルーム転送が完了した時に、タイミングイシューにより既存のノードにルームが残り、ユーザーが入場できる問題を修正
      - 現在、トランスファー中のルームはマッチメイカーのルームリストから除外されています。
      - トランスファーが完了し、削除されるはずの既存のルームや、トランスファーに失敗したルームに入場しないように修正
    - ユーザートランスファーが完了する前に再接続した場合、目的地ノードでユーザーオブジェクトが見つからない問題を修正
    - ユーザートランスファー時に対象ノードの状態値を確認しないように修正
    - ルームトランスファー時に、ユーザーのトランスファー関連コールバック関数が呼び出されるように変更

- Safe Pauseの進行時にルームマッチメイキングに失敗する現象を修正
  - トランスファー中であるルームの状態の種類にTRANSFER_IN_PROGRESSを追加
  - トランスファーが完了したルームがPAUSE状態に変わらないように修正
  - ルームへの入場がルームファイバーで処理されようとした時に、ルームがトランスファー中の場合は、ルームの入場処理を中止し、トランスファーを進行するように修正
	
* Roomとマッチメイキング関連バグの修正
  - CreateRoomメソッドで作成したルームをルームマッチメイキングに登録した後に該当のルームが終了すると、例外が発生する問題を修正
    - ルームマッチングの際にマッチルームが見つからなかった場合、ゲームユーザーにマッチユーザーカテゴリーを設定しないように修正

  - すべてのユーザーが退場したルームにタイミングイシューにより、新しくユーザーが入場できる問題を修正
    - ユーザーが入場する際に空室かどうかを確認するように修正
    - 最後のユーザーがルームから退場した直後にチャンネル情報の整理およびMatchRefillキャンセルを実行

  - クライアントからLeaveRoomを呼び出した際にサーバーエラーが発生する問題を修正
    - UserPostLeaveRoomのパケット転送の主体をゲームユーザーファイバーに変更
  
  - ルームが終了した状態でルームを使用するAPIを呼び出した際に例外であるClosedRoomExceptionを追加

* ゲートウェイおよびセッション関連のバグを修正
  * ゲートウェイノードのセッションオブジェクトが転送(Transfer)状態でleakされる問題を修正

  - セッションが削除される過程で例外が発生した場合、コネクションオブジェクトが削除されず残ることがある問題を修正

* ノードの関連バグを修正
  - 任意のノードを終了した場合、残りのサーバーでエラーまたは警告ログが残る問題を修正
    - 終了したノード情報をサーバー間で共有するように修正
    - 終了したノード情報を削除できるAPIを追加

  - Shutdown状態のノードに他のノードからパケットを送信することがある問題を修正
    - 終了告知後に実際の終了まで1秒間のディレイを設定

* その他
   * クライアントからリクエストに対するレスポンスを得られない問題を修正

  - 誤ったサービス名でサーバーを駆動した場合も、すべてのノードがReadyとして処理される問題を修正
    - ノード数を増加させるロジックを正しい時点に変更
    - 誤ったサービス名を使用してノードを実行しようとした場合、ノードマネージャですぐに終了するように修正


  - 重複ログイン中に強制接続終了処理されるバグを修正
    - DeviceIdが異なり、コネクションID、ゲートウェイノードIDが同じ場合、既存ユーザーをログアウトさせる際に、重複してセッションを閉じないように修正

  - ghostTimeout値が正しく適用されるように修正

  - ghotsTimeout値がdemandClientStateCheck値より少なくとも3秒以上大きくなるように制約事項を追加

#### Change

- Safe Pauseの高度化
  - 出発地ノードと到着地ノードの両方を選択するように変更
  - Safe Pause開始時に出発地ノードと到着地ノードを選択しないように変更
  - レポート機能強化: HTML形式の他、json形式でレポートを作成する機能を追加

- Safe Pause関連のプロトコル名の変更およびReportSenderの動作をリクエストする方式に修正
  - 変更前: PubNonStopPatchReportStartCommand / PubNonStopPatchReportStartNode
  - 変更後: NonStopPatchReportStartCommandReq / NonStopPatchReportStartCommandRes / PubNonStopPatchReportStartNode		

- 同じファイバーでリクエストを送信すると例外処理されるように変更

- BaseRoomの使用方法を変更
  - BaseRoomにonLeaveRoomメソッドのオーバーローディングを追加
  - BaseRoomのonPostLeaveRoomのパラメータを削除してユーザビリティを変更

- BaseUser使用方法変更
  - ユーザーがルームから完全に退場した後に呼び出されるonPostLeaveRoomメソッドを追加

- Disconnectedの状態値をisDetectedDisconnected、isCompleteDisconnectedに細分化
  - isDetectedDisconnectedは、サーバーからクライアントの接続が切断されたと判断した時の状態値
  - isCompleteDisconnectedは、サーバーから接続を切断する処理が完了した場合の状態値

- 固有のHostId作成ロジックを補強
  - ユーザーが入力したサブネットマスクに従ってHostIdを作成するように修正

- エンジンで使用するライブラリを最新化

- UserTimer、RoomTimerへの参照を返すAPI getUserTimer、getRoomTimerを追加

- DNS関連の設定を削除

---

### 1.2.0 (2021.07.13)

詳細については、[配布ノート](https://nhnent.dooray.com/share/posts/sGAj_STlTEWDr5LPgKgIhg)を参照

#### New

* ユーザーライセンスを適用
  
* [GameAnvilユーザーライセンス](https://gameplatform.toast.com/kr/services/gameanvil/license)
  
* ノード単位の駆動およびスケーリングをサポート

  * ランタイムでも新しいノードをアップロードできる機能を追加
  * 従来に比べてより安定的にクラスタリングできるように改善
  * ノード状態に応じたノード動作を改善
  * ランタイムにノード単位で駆動または終了できるように改善

* 簡素化されたブートストラップ
  * GameAnvilBootstrap Deprecateに設定。今後のバージョンで削除される予定。
  * GameAnvilBootstrapの代わりにGameAnvilServerインスタンスを使用してGameAnvilサーバーを実行。
  * 提供されるアノテーションを使用して、BaseClassをGameAnvilから読み込むようにユーザビリティを改善

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


* 非同期MySQLクエリをサポート
  * 非同期sql処理用のJasync-sql / X Dev APIクラスを追加
  * ユーザーが簡単に非同期sqlを処理できるようにサポート
  
* ユーザーガイド文書の補強
  * 1つの文書に含意的に記載されていたいくつかの重要なトピックをモジュール化し、文書の質と量をアップグレードしました。
  * チュートリアルを通じて、簡単かつ素晴らしいジグソーパズルゲーム(サーバー&クライアント)を直接開発できます。
  * JavaDoc APIリファレンスで内容が不足していた部分をすべて補充しました。また、説明が曖昧で誤解を招く可能性のある文章をすべて修正しました。



#### Fix

* チャンネル情報の管理および同期機能のリファクタリング
  * チャンネル情報を効率的に管理できるように改善
  * チャンネル情報をクライアントに送信する機能を改善
  * チャンネルのルームとユーザー数を取得する機能を追加
  * チャンネルIDを設定しなければ(すなわち""を使用すると)、それ以降チャンネル機能を使用しないように改善。チャンネルを使用したい場合は、少なくとも1つ以上の有効な文字列をチャンネルIDとして使用しなければならない。

* ルームの作成時にoutPayloadがリリースされないバグを修正
* 同じアカウントのクライアントから再接続するまで、ルームマッチメイキングが失敗し続けるバグを修正
* ユーザーマッチメイキング時にエラーが発生した場合、クライアントにレスポンスを送信しなかったバグを修正
* 発行(publish)APIを修正
  * ユーザビリティを改善
  * ユーザーが意図しないノードにpublishメッセージが送信されるバグを修正しました。また、publish APIのユーザビリティを改善しました。
  * チャンネルIDでハングルを使用した場合、pushlishメッセージが対象ノードを見つけられないバグを修正しました。



#### Change

* クラウドACL環境に合わせて基本ポートが変更されました。

| 名前 | ポート | 以前のポート |
| --- | --- | --- |
| ipcPort | 18000 | 16000 |
| publisherPort | 18100 | 13300 |
| gateway tcp | 18200 | 11200 |
| gateway web | 18300 | 11400 |
| management | 18400 | 25150 |
| support | 18600\～18999 | ユーザー定義 |
| GameAnvil Agent | 19080 |  |
| GameAnvil Admin | 19090 |  |

* Adminのサポートを終了します。これにより、GameAnvil内部では独自DBを使用しません。

* Dynamic Moduleのサポートを終了します。

* ZMQライブラリに変更
	
	* libzmqとjzmqの組み合わせからJeromqに変更
	
* ルームマッチメイキングのリファクタリングおよびユーザビリティを改善
  * ルームマッチメイキングのフローを改善
  * ストレステストを実行する際に、ルームが見つからない問題を修正
  * ユーザーがルームの人数管理機能を実装しなくてもいいように、ルームの人数情報管理機能を追加
  
* 認証とログイン、接続終了コードの最適化

    * Connectionから呼び出していたコールバックをSessionに移動
    * DisconnectAlarmの原因把握を簡単にできるように改善

* ユーザー提供トピックを整理

| 名前 | 説明 | 基本登録位置 |
| --- | --- | --- |
| GameAnvilTopic.GAME\_NODE | すべてのゲームノード | GameNode |
| GameAnvilTopic.GATEWAY\_NODE | すべてのゲートウェイノード | GatewayNode |
| GameAnvilTopic.SUPPORT\_NODE | すべてのサポートノード | SupportNode |
| GameAnvilTopic.ALL\_CLIENT | 接続中のすべてのクライアント | Session |
| GameAnvilTopic.ALL\_GAME\_USER | ゲームノードにあるすべてのゲームユーザーオブジェクト | GameUser |


* 例外処理およびメソッドシグネチャの最適化


  * 無分別に使用された例外処理とメソッドシグネチャを整理して、ユーザーに必要な情報だけを渡すように修正

* HttpRequestにPATCHメソッドを追加
* 証明書がなくてもテスト用でSSLを使用できる機能の設定名がuseSelfからuseSelfSignedCertに変更されました。

---

### 1.1.12 (2021.06.07)
#### Change

* SupportNodeにGatewayの接続情報を取得できるAPIを追加
    * BaseSupportNode.getGatewayConnectionInfoList関数を使用して、Gatewayの接続情報リストを提供

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
        "TCP_SOCKET": 11200,    // 該当ConnectionTypeに応じたポート情報を提供
        "WEB_SOCKET": 11400
      }
    }
  ]
}
```
---

### 1.1.11 (2021.05.06)

#### Change

* HttpReqestにPATCHメソッドを使用できるAPIを追加
    * httpRequest.PATCH()
    * httpRequest.PATCH(AsyncHttpCompletionHandler asyncHttpCompletionHandler)
    * httpRequest.PATCHAsync()

---

### 1.1.10 (2021-04-21)

#### Fix

* RedisSingleに不足していたpasswordサポートAPIを追加

---

### 1.1.9 (2021-04-16)

#### Fix

* クライアントのPauseClientStateCheckを受け取った時にidleClientTimeoutチェックも一緒に停止しないように修正。
* クライアントのPauseClientStateCheckで受け取ったpauseTimeの単位を秒数で計算するように修正。

---

### 1.1.8 (2021-04-15)

#### New

* クライアントのPauseClientStateCheckを受け取り、入力された時間だけ該当のクライアントの状態をチェックしないようにする機能を追加。
* クライアントのResumeClientStateCheckを受け取り、状態チェックを再開する機能を追加。
* GameAnvilConfig.jsonのGatewayにPauseClientStateCheck関連の許容範囲を制限する設定値を追加
    * pauseClientStateCheckMaxDuration =クライアントが送信した時間値が該当の数値より大きい場合、該当の数値に制限
    * pauseClientStateCheckMinDuration =クライアントが送信した時間値が該当の数値より小さい場合、該当の数値に制限

---

### 1.1.7 (2021-04-02)

#### Fix

* /game-data/get使用時にGameData値がJsonObjectではなく、Stringで送信される問題を修正

---

### 1.1.6 (2021-03-30)

#### Change

* Dynamic Module機能がGameAnvilからスペックアウトされ、削除されました。
  * 既存の使用者のために、Dynamic Moduleソースの公開と使用ガイドを作成して共有します。ガイドを参照して、GameAnvilではなくユーザープロジェクトにDynamic Moduleソースを追加して使用できます。
  * [GameAnvil-Guide/81 Dynamic Moduleスペックアウトによるソース提供および使用方法](https://nhnent.dooray.com/share/posts/xtwdO6FJTXmr7l28cZSM-w)

---

### 1.1.5 (2021-03-19)

#### Change

* GameAnvil DB & Admin機能(GameData、Dynamic Moduleを除く)を削除

    * GameAnvilエンジンでは、これ以上DBを使用しません。
    * 参考: [GameAnvil-Guide/76 Admin 1.1.0 -&gt; 1.1.1マイグレーション文書](https://nhnent.dooray.com/share/posts/GIHoRQ8kTjSi7D4wQjUcWw)
    * 従来のDBを使用していたAdmin機能はすべてGameAnvil Adminに移動しました。

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

* ルームマッチングにMatchingGroupを適用すると、断続的に作成されたルームにJoinできない問題を修正

---

### 1.1.3 (2021-02-05)

#### Fix

* インスタンスの再起動時にDisable状態で復旧されない問題を修正
    * 低確率で再発する可能性があるが、再起動すると復旧する

* ManagementNodeでnodeInfoByHostId使用時にDouble.infinityエラーが表示される問題を修正

---

### 1.1.2 (2021-01-07)

#### Fix

* DynamicModuleでSuspendExcution例外発生する可能性のあるコードを呼び出した際の例外処理が抜けていて、DynamicModuledml methodが正しく呼び出されない問題を修正

---

### 1.1.1 (2021-01-05)

#### Fix

* GatewayNodeでgetNodeId()を呼び出すと、NPEが発生する問題を修正

* 同じアカウントのクライアントで再接続するまで、該当のGameUserでルームマッチングに失敗し続ける問題を修正

---

### 1.1.0 (2020-12-17)

詳細については、[配布ノート](https://nhnent.dooray.com/share/posts/bWby9jGjQri2cFNR_KYbnw)を参照

#### New

* Jdk11サポート

  * Jdk8サポートバージョンとともにJdk11サポートバージョンを配布しました。
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

    * サーバー内で発生した状況を指定したURLで取得できる機能を追加しました。
    * "-Dalarm.url" VMオプションを使用して、Alarmを受け取るURLを指定できます。
    * [GameAnvil-Guide/69 Alarmの使用方法](https://nhnent.dooray.com/share/posts/Ap6DJT9KSaGv916_tj-xAA)

#### Change

* BaseObjectのfindAllUserLocsOfAccountの戻り値をUserLocに変更

* Node情報をRest APIで取得できる機能を追加しました。
    * /management/ipcNetworkNodeInfo
    * /management/managementNetworkNodeInfo
    * /management/locationLookupNodeInfo
    * /management/matchNodeInfo

#### Fix

* 同じAccountIdで他のSubIdを使用した場合、ログインには成功するが、その後、レスポンスパケットを受信できない問題を修正しました。
* GatewayNode、MatchNode、SupportNode、ManagementNodeのNodeNumが1からスタートする問題を修正しました。
* Nettyのsend,recv bufferサイズの最適化とそれに適したwatermarkが適切に適用されるように修正

---

### 1.0.7 (2021.01.05)

#### Fix
- GatewayNodeでgetNodeId()を呼び出すとNPEが発生する問題を修正

---

### 1.0.6 (2020.12.28)

#### Fix
- `RoomMatchMaking failure. The matched-room({}) does not exist in the game node.` というログが残り、ルームマッチングが失敗した場合、そのユーザーは再接続するまで継続してルームマッチングが失敗する問題を修正しました。

---

### 1.0.5 (2020.11.17)

#### Fix

- RoomMatchReqでエラーレスポンス時に、パケットヘッダの復元(restore)ができない問題を修正

---

### 1.0.4 (2020.10.29)

#### Fix

- 任意のSessionにまだ何のパケットも送信していない場合、このSessionでsendを呼び出すと、NPE(Null Pointer Exception)が発生する問題を修正
- publishToClient使用時に問題が発生(Noticeを含む)
- Roomで使用する繰り返しTimerによって、ルームの破壊が遅れる問題を修正
- protobuf構造体を変数として持っているClassをdeserializeする際に例外発生する問題を修正
- java.lang.UnsupportedOperationExceptionが発生

---

### 1.0.3 (2020.10.26)

#### Change

- スペックから除外されていた告知機能APIを再び有効化

---

### 1.0.2 (2020.10.12)

#### New

- Logに表示されるNode情報をユーザーの希望通りに設定できる機能を追加
- HostIdを直接入力できる機能を追加
- nodeInfoByHostIdでHostIdごとに情報を送信する機能を追加

#### Fix

- IP 0.0.0.0をバインディングした際にエラーが発生する問題を修正
- ConfigModuleの/config/getを使用した際に、成否値が失敗としてレスポンスする問題を修正
- GameNodeが駆動された状態で他のGameNodeが駆動された場合、Publish Packet処理によって、エラーが発生することのある問題に例外処理を追加

#### Change

- UserId、RoomIdを作成する際に、最後に使用するShardIndex値の範囲を指定できるように修正
- RestObjectのgetRemoteAddress値を追加
- NodeKeyのリクエスト数を指定できるように修正

---

### 1.0.1 (2020.09.07)

#### Fix

- onLogin()でfalseを返す場合、Payloadがクライアントに送信されない問題を修正
- DebugとTraceログ文の前のログレベル確認条件文が抜けている部分を追加

---

### 1.0.0 (2020.08.31)

詳細については、[配布ノート](https://nhnent.dooray.com/share/posts/5Fvh0aszQ5u6d_ZZxRWPvA)を参照

#### New

- 性能と安定性の大幅向上(0.9バージョンに比べ約7倍)
- 性能に直接的な影響を与える多数のロジックを最適化し、問題点を修正
- 完全に生まれ変わったGameNodeロードバランシング
- クラウドVMの状態を反映し、迅速かつ正確に最適なGameNodeで負荷を分散
- Integer型IDを導入
- Stringになっていた主要IDをすべてIntegerに変更し、最適化を進行
- サーバーLogレベルをランタイムに変更できるAPIを追加
- Node単位でサービス中にLogレベルを変更可能

#### Fix

- ユーザー転送問題を修正、使用方法の改善と最適化
- GameAnvilユーザーはより簡単にユーザー転送を使用可能
- 無点検パッチを完璧に修正
- これまでルームの転送と無点検パッチが正しく動作しなかった問題をすべて修正
- サーバー駆動時に不明なタイミングイシューにより、一部のNodeが正しく表示されなかった問題を修正
- Nodeの一部のTimerオブジェクトが解除されず、継続的に累積されていたMemory Leakを修正
- クライアントの接続チェックプロセスで、新しく接続するユーザーが意図せず切断されていた問題を修正

#### Change

- Node名を次の表のようにより直感的なものに変更

| v1.0       | ～v0.10       | 必須 | 機能                                       | コンテンツ実装 | ネットワークアクセス    |
| ---------- | ------------- | ---- | ------------------------------------------- | ----------- | ----------------- |
| Gateway    | Session       | 必須 | クライアント接続と認証を処理              | 可能       | public            |
| Game       | Space         | 必須 | 実際のゲームサーバーとしてコンテンツを処理           | 可能       | private           |
| Support    | Service       | 任意 | 必要に応じて独立したサービスとして実装できるようにサポート | 可能       | private or public |
| Match      | Match         | 任意 | マッチメイキングを実行                          | 可能       | private           |
| Location   | Location      | 必須 | ユーザーとルームなどの位置情報を保存および管理    | 不可     | private           |
| Management | Management    | 必須 | サーバー情報の取得およびAdmin/Agentと通信       | 不可     | private           |
| Ipc        | Communication | 必須 | GameflexサーバーのInter-process通信処理    | 不可     | private           |

- GameAnvilコードの可読性とユーザビリティを向上
- パッケージ構造を効率的に改善
- クラス継承/包含構造を改善
- ネーミングを直感的で一貫性のあるものに修正
- Nodeの無効化状態はInvalidとDisableに細分化
- Invalidは一時的に反応しない状態
- Disableは永久的に無効化された状態
- Loopbackアドレスは自動的にバインディング
- GameAnvilで使用するすべてのライブラリを最新バージョンにアップデート
- ErrorCode全体を1つに統合
- 不要なReConnectとMoveService機能を削除
- 登録時点ではなく、駆動時点から次の時間を測定する新しいTimerを追加

---

### 0.10.2 (2020.04.08)

#### New

- GameAnvil APIリファレンス(JavaDoc)サイトをオープン

- AOT Instrumentationをサポート

- エンジンをランタイムに毎回JIT Instrumentationする必要なくコンパイルタイムにAOTで進行

- 公式GCとしてG1GCを使用するようガイドを開始

- 最も基本的なVMオプション

  `java -javaagent:QUASAR_PATH\quasar-core-0.7.10-jdk8.jar=bm -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication`

#### Fix

- Quasarスタックバグを修正
- ルームマッチングユーザーが異なるルームにマッチングされるバグを修正
- 圧縮されたパケットが送信されない問題を修正

---

### 0.10.1 (2020.02.11)

#### New

- ユーザーに提供するツールをすべてまとめたUtilクラスを提供
- エンジン内の様々な場所に散らばっていたツールを、該当クラスに一箇所に整理

#### Change

- Topic処理コードの性能を改善


---

### 0.10.0 (2020.02.06)

#### New

- 問題が多かった非同期サポートAPIを新しくリファクタリング
- Redis ClusterサポートおよびLettuce公式サポートによる新たな非同期APIを追加
- 非同期HttpRequestとHttpResponse APIを追加
- ByteStringベースのカスタムプロトコル機能をサポート
- Google protobuf以外のユーザーが希望する方法でメッセージをシリアライズ可能

#### Fix

- 以前のバージョンで最も深刻な問題だったファイバーの状態が破損する問題をすべて修正

#### Change

- エンジンで使用するパケットとヘッダサイズを最適化
- 0.9バージョンに比べて性能が約2.5倍向上
- Embedded Redis Spec-out
- インフラが提供するサービスを使用するようにガイド

---

### 0.9.9 (2019.10.25)

#### New

- プロセス単位でWatchDogを連動できるようにポートを追加で開放

#### Fix

- ゴーストユーザーバグを修正
- SpaceNodeで発生するConcurrent Modificationエラーを修正
- NodeInfoManagerのComparatorエラーを修正
- ログアウトしたユーザーのセッションオブジェクトが整理されない問題を修正
- マッチンググループが異なるチャンネルのユーザーに正しく適用されなかった問題を修正
- パーティーマッチメイキング時にTimeoutに対するコールバックを正しく得られなかった問題を修正

#### Change

- ルームマッチメイキングがすでに進行中の状態で重複リクエストを受信した場合のエラーコードを追加
- MATCH_ROOM_FAIL_IN_PROGRESS

---

### 0.9.8 (2019.09.05)

#### New

- SessionにClientTopicを追加して削除できるaddTopic()、removeTopic()APIを追加
- Packet TTL機能を追加
- Dangling Locationを定期的にチェックして整理するロジックを追加

#### Fix

- AdminでユーザーをKickした場合、クライアントにForceLogoutNotiを送信しないように修正
- LocationNodeのSpot誤作動を修正

#### Change

- onAuthenticate()コールバック失敗時に該当ソケットを閉じる前にForceCloseNotiをクライアントに転送するように変更


---

### 0.9.7 (2019.07.27)

#### New

- 意図せず残っているRoomを定期的にチェックして整理するロジックを追加
- ユーザーマッチメイキングは、リクエストした後に再ログインしても以前のマッチリクエストがキャンセルされず、進行中である場合があります。このような以前のユーザーマッチメイキングの処理状態を知らせるために、再ログインレスポンスに新しいフラグを追加
- Packet Expire可能を追加(デフォルト値は30秒)

#### Fix

- Connection ID(CID)が重複する問題を修正
- Disconnectionされたユーザーによってルームのファイバーがcloseされるバグを修正
- ログインに失敗した場合、レスポンスメッセージにserviceIdフィールドが正しくセッティングされない問題を修正
- コンテンツコールバック処理部分のうち、try-catchが抜けていた部分を修正
- クライアントReconnectが約3秒以上かかっていた問題をすぐに処理できるように修正
- ゴーストユーザーをチェックすると発生するスレッド同時性問題を修正
- ルームマッチメイキングの重複処理バグを修正
- ルームへの接続が切断されたユーザーが残る問題を修正

#### Change

- Session Disconnectのすべてのケースにログを追加
- CreateRoomは現在のノードにすぐにルームを作成するように変更


---

### 0.9.6 (2019.06.23)

#### New

- 転送可能な時点をコンテンツで調整できるようにUserとRoomにcanTransfer()インターフェイスを追加

#### Fix

- エンジン内部のコードに存在していたメモリリークを修正
- ZMQ Router、Publiserソケットの接続バグを修正
- Loginが無限に繰り返される問題を修正
- isLogined() APIが正しく動作しない問題を修正
- NodeInfoManagerの同期問題を修正
- ユーザーオブジェクトが正しく整理されない問題を修正

#### Change

- 既存のAsyncAwaitHttpRequestをFiberHttpRequestに名前を変更
- FiberHttpRequestとHttpRequestに非同期関数を追加
- Location Index関連のログを追加
- SpaceNode分配方式をidleからroundRobinに変更
- NodeStarterでLocationNodeが存在する場合のみsleepするように変更


---

### 0.9.5 (2019.06.21)

#### Fix

- 無点検パッチ時に新しいユーザーがログインした場合やユーザー転送中の場合、Pauseされたノードで該当ユーザーが整理されないエラーを修正

#### Change

- 文字列形式のserviceIdを整数型に変更
- プロトコルのservice_codeをservice_idに変更して関連コードを修正
- RESTfulサポートAPIの内部実装と使用方法を変更


---

### 0.9.4 (2019.05.24)

#### New

- コンテンツで使用するプロトコルをプロトコル単位で登録できる機能を追加
- 内部パケット処理を効率的にするため
- MoveService機能を追加

#### Fix

- ルームマッチメイキングで使用する比較演算子(Comparator)のエラーを修正

#### Change

- Authenticationレスポンスに直近のログイン情報を追加して、次のログイン時に活用可能
- setReady()を明示的に呼び出してonReady状態に移行するように変更
- LocationNodeがCommunicationNodeのonPrepare時点で初期化されるように変更


---

### 0.9.3 (2019.04.10)

#### Fix

- AsyncAwait.call() APIでtimeoutが発生すると、NPE(Null Pointer Exception)も発生する問題を修正
- Epollが有効になると停止する問題を修正
- ログイン時に発生するInvalid System Target Locationエラーを修正

#### Change

- ManagementNodeをエンジンユーザーが拡張して使用できないように変更
- 代わりにServiceNodeを使用
- AdminでMachine情報の登録、修正、削除がリクエストされた瞬間、すぐに適用されるように変更


---

### 0.9.2 (2019.02.11)

#### New

- Adminを利用した予約告知、予約点検機能を追加
- Adminを利用したWhite IP、White User、White Device機能を追加

#### Fix

- AsyncAwait.run() APIで誤った方式で例外が処理されていたコードを修正
- ManagementNodeが他のNodeより先に初期化されるように修正
- チャンネルIDを使用しない場合、一部のチャンネル関連publishが抜け落ちる問題を修正
- CreateRoom、JoinRoomにコンテンツが失敗を返した場合、誤ったエラーコードがヘッダに載る問題を修正
- パケットヘッダを作成すると発生していたメモリリークを修正
- ルームマッチメイキングで発生していたメモリリークを修正

#### Change

- onLogout()コールバックにpayloadを追加
- ResultCodeを修正するに伴い、いくつかのプロトコルで重複フィールドを削除
- Dynamic Moduleユーザビリティを改善


---

### 0.9.1 (2018.11.12)

#### New

- マッチメイキングを専任するMatchNodeを追加
- ユーザーマッチメイキングにrefill機能を追加
- ユーザーマッチメイキングの新しいタイプであるパーティーマッチメイキングを追加
- AdminのMachine情報でノードの種類ごとに付加情報を追加
- ユーザーが同じDeviceIdとUserIdで再ログインする際に呼び出されるonLoginByOtherConnection()コールバックを追加

#### Fix

- Dynamic Moduleバグを修正
- Admin接続ができない問題を修正
- Userのreplyバグを修正
- Timer追加/削除エラーを修正
- AsyncAwait.run()に抜けているtimeoutを適用
- ノードShutdown時に発生していたエラーを修正
- ルームマッチメイキングを使用していないにもかかわらず、マッチング情報を削除してNPE(Null Pointer Exception)が発生する問題を修正

#### Change

- 一部のインターフェイスの位置を変更
- 異なるDeviceIdでログインすると、重複ログイン処理の代わりに再ログイン処理が行われるように変更
- ルームマッチメイキングのonMatchRoom()コールバックで追加したpayloadがそれ以降のフローに含まれるように修正


---

### 0.9.0 (2018.06.27)

#### New

- ユーザーマッチメイキング機能を追加
- チャンネル別のユーザーとルームリスト機能を追加
- Embedded Redisを追加
- Reconnect機能を追加
- (旧)Test Agentを追加

#### Fix

- Account単位の重複ログイン処理問題を修正

#### Change

- OracleJDKからAdpotOpenJDKに変更
- Node(スレッド)単位でZMQ通信していたものをプロセス単位に変更

---

### 0.8.6 (2018.06.28)

#### New

- Send from Session to Management機能を追加
- Request from Session to Management機能を追加
- User MatchMakerを追加
- Channel User List機能を追加
- Custom Serializer機能を追加
- RoomFinderにCustom Serializerをplug-in可能

#### Fix

- チャンネルユーザーマネージャーでチャンネルがtopicとして使用されるpublishに対する空文字列例外処理を追加
- チャンネルがtopicで使用されるpublishに空文字列の例外処理を追加
- AutoJoinRoomを使用しない環境でdelNamedRoomInfo()とdelRoomInfo()を呼び出すと、DefaultRoomFinderによって発生する問題を修正
- RoomFinderでnullチェックの欠落により発生するNullPointerExceptionを修正
- ユーザーマッチメイキングでtimeoutされたリクエストが削除されない問題を修正
- redisにpasswordが設定されている場合の処理を追加
- cfgManagementのsetRestIp()エラーを修正
- ServiceノードでHttpハンドリングができない問題を修正。
- ブランチ: 0.8.6.2_fix_service_node_rest_handling
- タグ: tardis-0.8.6.2.1
- AutoJoinRoom処理時にgameIdが設定された場合のエラーを修正
- MatchMaker修正および例題を補強
- ISession::onDisconnect()の抜けを修正
- Communication Node間のNode情報交換エラーを修正
- FindRoomListの性能問題を解決
- Shutdownバグを修正
- BaseNodeがpublishToNodeメッセージを受け取れない問題を修正
- FindRoomListの性能問題を解決
- Shutdownバグを修正
- BaseNodeがpublishToNodeメッセージを受け取れない問題を修正
- NamedAutoJoinRoomバグを修正
- UpdateRoomInfo/DelRoomInfoが該当Nodeで即時反映されるように修正
- ファイバー内でblocking callを呼び出す部分によって出力されていたErrorを修正

#### Change

- FindRoomListはルーム全体のリストcontainerを一度にserialize/deserializeするように修正
- 既存のLocationException、TardisExceptionなどをRuntimeExceptionに変更して、Exceptionを整理
- forceLeaveRoomはkickOutRoomにrename
- forceLogoutはkickOutにrename
- tardisConfig構成を変更

---

### 0.8.5 (2017.12.18)

#### New

- JWT認証トークンを使用したHTTPセッション認証および管理機能を追加
- 同じIDを使用して、複数のServiceに重複ログインできる機能を追加
- 1つのGameNodeに複数のRoomTypeを使用できる機能を追加
- Serverのすべての情報をWebで閲覧できる機能を追加(NodeInfoPage)

#### Fix

- タイマーが削除されないバグを修正
- onPostLeaveRoomが2回呼び出されるバグを修正

#### Change

- ゴーストユーザーの処理を改善
- ゴーストユーザーロギングを追加
- Packet compressロジックを改善
- RestObjectを改善、使用関数を追加
- パケットのエラー処理を変更、プロトコルヘッダにエラーコードを入れて処理

---

### 0.8.4 (2018.01.10)

#### New

- Bootstrapを適用
- NGB Bootstrapと同じ形態で実装
- 実装部分の各クラスのclassインスタンスのみ明示してコードを単純化
- 複数のメッセージをやり取りできるMulti Dispatch機能を追加
- ServiceNodeSender(以前のCustomServiceSender) publishToLobbyメソッドを追加

#### Change

- AsyncAwait、RAsyncAwaitの名前を変更
- Bootstrap RoomFinder設定をspaceに移行
- Cache、Custom Exceptionの名前を変更(Location、Service)
- PayloadクラスのgetFirstメソッドで最初のPacketを所得できるように修正
- ServiceNodeSenderのrequestToLobbyメソッドを削除(同じ機能のrequestToNodeメソッドと重複する問題)
- PacketクラスのmakeDecompressメソッドを呼び出した際にasyncを使用しないように修正
- UserSenderのreplyメソッドオーバーローディング(Packetリストを処理)
- 一部のクラス名を変更
- AsyncCall → AsyncWaitingCall
- AsyncRun → AsyncWaitingRun
- ReverseAsyncCall → RAsyncWaitingCall
- ReverseAsyncRun → RAsyncWaitingRun

#### Fix

- Session IPエラーを修正

---

### 0.8.3 (2017.10.26)

#### New

- RESTハンドリング機能を追加
- TardisConfig.jsonにpermissionGroupsでRESTハンドリングに対して、Path & IPベースのACL機能を実装
- Channel List(チャンネル名リスト)、Channel Information(特定チャンネルの詳細情報)
- Space LobbySenderにrequestToCustom、sendToCustomメソッドを追加
- SpaceノードManagement情報に"State"項目を追加
- Ilobby、ISessionLobby、IServiceインターフェイスに以下のvoid onShutdown() throws SuspendExecutionを追加
- Nodeがshutdownする際にcontentsでresourceを解除できるようにonShutdown()インターフェイスが追加
- ServiceNodeSenderにpublishToNode関数を追加

#### Fix

- ManagementのJMX Management APIを修正および追加(SessionGateway、CustomGateway)
- Channel List(チャンネル名のリスト)、Channel Information(特定チャンネルの詳細情報)機能を追加
- Session、CustomノードのRESTハンドリングロジックを修正
- Gatewayノード(Session、Custom)isConnectableTypeメソッドの戻り値を修正
- SessionGatewayおよびCustomGatewayのラウンドロビン問題およびJMXリターン情報を修正
- キャッシュノードtargetMapキー値を使用するコードを修正
- SessionGatewayノードのREST RoundRobinコードを修正
- ログインプロセスを改善(Invalid Target Locationエラーログ関連を修正)
- ルームを削除するdelRoomInfoを呼び出した時、該当ノードの状態がREADYでない場合はリターン処理

#### Change

- SessionノードとクライアントのEnd-Pointを1つに統合(Session Gateway)
- CustomノードのREST End-Pointを1つに統合(Custom Gateway)
- ファイバーの直接使用コードをReverseAsyncCallに変更
- SpaceSingular Dispatchタイムスタンプを追加
- onTimer登録して、定期的にチェック(10分)
- タイムスタンプが超過したユーザーはinternalLogout処理
- Asyncタイムアウトを適用
- ゴーストユーザーの処理ロジックを追加および改善、TardisConfig.jsonに必要な値を追加
- 各Responseプロトコルのresult_code形式を従来のinteger値からenumに変更し、種類を簡素化。is_payloadフィールドを削除
- クラス、パッケージ、関数とcfg内容がrename
- Request、Responseプロトコルで複数のpayloadを処理できるようにclass Payloadデータ型がPacketに変更、class OutPayloadを削除
- ユーザーTransfer-Inインターフェイスのパラメータタイプをbyte配列に変更

---

### 0.8.2 (2017.09.21)

#### New

- CustomServiceにRestObject機能を追加
- CustomServiceにRestHandlerを追加

#### Fix

- Cacheでユーザー情報が消えないバグを修正
- エンジン内部で発生していたNPE(Null Pointer Exception)に例外処理を追加
- パケット転送にCID(Connection ID)を正しく適用できていなかった問題を修正

#### Change

- Customモジュールのネーミングをすべて新しく変更
- 例) CustomLobby → CustomService
- Customモジュールインターフェイスの一部を変更
- CustomServiceSenderのsendToUser()とrequestToUser() APIにserviceId引数を追加
