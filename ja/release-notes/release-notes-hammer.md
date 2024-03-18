## Game > GameAnvil > リリースノート > GameHammer

### 1.4.0 (2023.12.13)

#### New

* シナリオテストのユーザビリティを改善
  - シナリオアクターを介してメソッドを直接渡す方式を追加
  - アノテーション付きでメソッドをハンドラとして登録する方式を追加
  - シナリオテストでサーバーにリクエストして、レスポンスを受け取るまでにどのくらいの時間がかかるかを測定して、結果ログに出力する機能を追加
  - 状態変化をステート内部で設定せず、シナリオを作成する際にまとめて作成するようにして、状態移動がどのように行われるかを1カ所で管理する機能を追加
  - シナリオテスト時にサーバーが実行中でない場合は、ログを通じて分かるように修正
  - シナリオテストログを転送および照会する機能を改善
* Payloadで圧縮パケットをサポートする機能を追加
* protobuf 3最新バージョンにアップデート
* Protocolを登録する際にindexを指定しなくてもいいように改善

#### Change

* ログイン時に誤ったChannelIdを入力した場合、SystemErrorレスポンスの代わりにLogin失敗レスポンスされるように修正
* ConfigLoaderにパラメータとして受け取ったストリングからCustomConfigをローディングできるように、オーバーローディングを追加
* ScenarioActor.connect()のパラメータ順序を変更。
    * 他のapiと統一されるようにcallbackを前に受け取るように修正
* パケットencode/decode性能を改善

#### Fix

* StateのonEnter、onExitでexceptionが発生した場合、ログが残らない問題を修正
* テストが終了した時点でrequestを送信し、レスポンスを待たない場合、warnログが残る問題を修正
* ConnectされていないConnectorでTimerが動作しない問題を修正。
* testTimeout時間が過ぎたが、startしなければならないscenarioActorが残っている場合、テストが終了しないバグを修正。

---

### 1.3.0 (2022.12.27)
#### New
* vmOptionで設定をロードできる機能を追加

---
### 1.2.1 (2021.11.30)

#### New 

* SecureSocketサポート機能を追加。
	* RemoteInfo classにuseSecureSocketオプションを追加。(default : false)
    * GameHammerConfigのtargetServerListの項目にuseSecureSocketフィールド追加。(default : false)
	* Tester.Bulder.addRemoteInfo()にuseSecureSocketを入力されるオーバーローディングを追加。

---

### 1.2.0(2021.07.13)
#### Change
* パッケージ構造を整理
	* 内部用のパッケージはgameanvilcoreでまとめる。
* ResultCode
  * ResultCodeAuth
    * AUTH_FAIL_MAINTENANCEを削除 
  * ResultCodeCreateRoom
    * CREATE_ROOM_FAIL_CREATE_ROOM_IDを追加
    * CREATE_ROOM_FAIL_CREATE_ROOMを追加
  * ResultCodeChannelInfo
    * CHANNEL_INFO_FAIL_NO_CHANNEL_INFOを追加
    * CHANNEL_INFO_FAIL_INVALID_SERVICE_IDを追加
    * CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUNDを追加
  * ResultCodeAllChannelInfoを追加
  * ResultCodeChannelCountInfoを追加
  * ResultCodeAllChannelCountInfoを追加
  * ResultCodeChannelList
    * CHANNEL_LIST_FAIL_INVALID_SERVICEIDを削除
    * CHANNEL_LIST_FAIL_NO_CHANNEL_LISTを追加
  * ResultCodeJoinRoom
    * JOIN_ROOM_FAIL_ALREADY_JOINED_ROOMを追加
    * JOIN_ROOM_FAIL_ALREADY_FULLを追加
    * JOIN_ROOM_FAIL_ROOM_MATCHを追加
  * ResultCodeLogin
    * LOGIN_FAIL_MAINTENANCEを削除
  * ResultCodeMatchUserCancel
    * MATCH_USER_CANCEL_FAIL_CONTENT → MATCH_USER_CANCEL_FAILに名前を変更
    * MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESSを追加
  * ResultCodeMatchRoom
    * MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_IDを追加
    * MATCH_ROOM_FAIL_CREATE_FAILED_ROOMを追加
    * MATCH_ROOM_FAIL_INVALID_ROOM_IDを追加
    * MATCH_ROOM_FAIL_INVALID_NODE_IDを追加
    * MATCH_ROOM_FAIL_INVALID_USER_IDを追加
    * MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUNDを追加
    * MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORYを追加
    * MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTYを追加
    * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULLを追加
    * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULLを追加
  * ResultCodeMatchUserDone
    * MATCH_USER_DONE_FAIL_TRANSFERを追加
    * MATCH_USER_DONE_FAIL_CREATE_ROOMを追加
  * ResultCodeNamedRoom
    * NAMED_ROOM_FAIL_CREATE_ROOMを追加
  * ResultCodeDisconnect
    * FORCE_CLOSE_MAINTENANCEを削除
    * FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_IDを追加
    * FORCE_CLOSE_DISCONNECT_ALARMを削除
    * FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENTを追加
    * FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSIONを追加
  * ResultCodeSessionCloseを追加

### 1.1.2 (2021.11.30)

#### New 

* SecureSocketサポート機能を追加。
	* RemoteInfo classにuseSecureSocketオプションを追加。(default : false)
    * GameHammerConfigのtargetServerListの項目にuseSecureSocketフィールド追加。(default : false)
	* Tester.Bulder.addRemoteInfo()にuseSecureSocketを入力されるオーバーローディングを追加。

---

### 1.1.1 (2021.04.16)

#### New 

* ping機能をオンオフできるように`Connection.setSendPingPaused()`を追加。

#### Fix

* configのpingIngervalが適用されないバグを修正
* configのpingIngervalが0の場合、pingを送信しないように修正

---

### 1.1.0 (2021.04.15)

#### Change

* サーバーのバージョンと合わせるために1.1.0にアップデート

#### New

* sendPauseClientStateCheck()を追加。
* sendResumeClientStateCheck()を追加。
* サーバーから送信される状態チェックのレスポンスをオンオフできる機能を追加

---

### 1.0.2 (2020.02.10)

#### Fix
* クライアントからゲームノードに指定された時間(default10秒)中に何のパケットも送信しなかった場合、サーバーからクライアントにヘルスチェックリクエストが送信されるが、GameHammerでこのヘルスチェックリクエストに誤ったレスポンスをして、接続が切断される問題を修正
* シナリオテストを長時間継続して、リクエストしたパケット数が大幅に増加した場合、packetSeqがoverflowされてサーバーからレスポンスを受け取れない問題を修正 
* send時にもpacketSeqを増加させる問題を修正

#### Change
* ログ内容を強化。
    * accountId、userIdを追加。
    * 最後に受け取ったパケットを追加。
    * Ping/Pongの最後のパケットから除外
* パケットのサイズを減らすために
    * packetSeqの最大値を16383に制限
    * subIdの最大値を127、最小値を1に制限

---

### 1.0.1 (2020.12.28)

#### Fix
* 同じMessageに対してwaitForを連続して使用した場合、最初のレスポンス時にすべての重畳待機が解除されるバグを修正
* ResultAuthenticationのgetPayloads()がnullをリターンするバグを修正
* テストが終了した際に、断続的にHandlerPing.onPingTime()でNullPointerExceptionが発生する問題を修正
* テスト中に断続的にStatistics.record()でConcurrentModificationExceptionが発生する問題を修正

#### Change
* GameHammerConfig.jsonファイルがない場合に出力されたログをerrorからwarnに変更。

---

### 1.0.0 (2020.12.18)

#### Fix
* EAバージョンで長時間テストに失敗する問題を修正。
* EAバージョンに比べTPS性能を大幅に改善(約2倍)
* 不足していた機能を追加。
    * Connector
        * getChannelInfo
        * addListenerAdminKickoutNoti
        * addListenerForceCloseNoti
        * addListenerDisconnect
    * User
        * moveChannel
        * snapShot
        * addListenerMatchPartyStartNoti
        * addListenerMatchPartyCancelNoti
        * addListenerForceLogoutNoti
        * addListenerForceLeaveRoomNoti
        * addListenerMoveChannelNoti
        * addListenerNotice

#### Change
* Tester
    * すべてのリクエスト方式機能にSync/Async方式をサポート
        * Sync :リクエスト時にFutureを戻り値として受け取り、Fureture.get()で完了するまで待機して結果を受け取り、処理する方式。
        * Async :リクエスト時にcallbackを引数として渡し、callbackから完了結果を受け取って処理する方式。
* Scenario
    * TRANSACTIONとEVENTの概念を削除
        * 代わりに各StateからchangeState()を使用して、希望するStateに直接移動

#### New
* サーバーから送信されるnotiを待って処理できるようにwaitForXXX機能を追加。

---

### 1.0.0-EA (2020.08.03)

#### New
* Tester - GameAnvil Connectorに代わってサーバーとの連動機能テストをサポート
    * Connection - GameAnvil ConnectorのConnectionAgentが担当する機能をサポート
        * connect
        * authenticate
        * getChannelList
        * send
        * request
        * createUser
    * User - GameAnvil ConnectorのUserAgentが担当する機能をサポート
        * login
        * logout
        * createRoom
        * namedRoom
        * joinRoom
        * leaveRoom
        * matchRoom
        * matchUser
        * matchPartyStart/Cancel
        * moveChannelStart/Cancel
        * send
        * request
        * addListenerMatchUserTimeout
        * addListenerMatchUserDone
* ScenarioTest - Testerを使用したシナリオテストをサポート。 
    * ScenarioMachine - シナリオを構成する複数の状態の集合
    * State - ユーザーが定義する全シナリオの中から特定の状態を表現
    * ScenarioActor - シナリオを実行する1つの仮想ユーザー
