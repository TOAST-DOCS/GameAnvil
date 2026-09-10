<!-- pre-align:aligned sig=39fc01e201f7 -->

<a id="game-gameanvil-release-notes-gamehammer"></a>
## Game > GameAnvil > リリースノート > GameHammer { #game-gameanvil-release-notes-gamehammer }

<a id="20-january-29-2026"></a>
### 2.2.0 (2026.01.29) { #20-january-29-2026 }

<!-- TODO: translate body -->

<a id="20-january-29-2026-new"></a>
#### New

<!-- TODO: translate body -->

<a id="20-january-29-2026-change"></a>
#### Change

<!-- TODO: translate body -->

<a id="10-june-30-2025"></a>
### 2.1.0 (2025.06.30) { #10-june-30-2025 }

<a id="10-june-30-2025-change"></a>
#### Change
* GameAnvil 2.1.0サーバーのリリースに合わせて、GameHammerも2.1.0バージョンをリリース
* GameAnvil 2.1サーバーに合わせてエンジンプロトコルをアップデート
  * GameAnvil 2.1以前のバージョンのサーバーは今後サポートされません
* 一部のResultCodeを変更

    | 変更前 | 変更後 |
    | ---- | ---- |
    | LOGIN\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | LOGIN_FAIL_INVALID_SERVICE_NAME<br>失敗。不正なサービス名 |
    | LOGIN\_FAIL\_INVALID\_USERTYPE<br>失敗。不正なユーザータイプ | LOGIN_FAIL_INVALID_USER_TYPE<br>失敗。不正なユーザータイプ |
    | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>失敗。不正なサービス名 |
    | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>失敗。不正なサービス名 |
    | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_FORM\_NULL<br>失敗。マッチング申込書がnullの場合 | MATCH\_ROOM\_FAIL\_MATCH\_FORM\_NULL<br>失敗。マッチング申込書がnullの場合 |
    | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_INFO\_NULL<br> 失敗。マッチング情報がnullの場合 | MATCH\_ROOM\_FAIL\_MATCH\_INFO\_NULL<br> 失敗。マッチング情報がnullの場合 |
    | FORCE\_CLOSE\_BASE\_CONNECTION<br>サーバーでBaseConnectionのclose()を呼び出し | FORCE\_CLOSE\_CONNECTION<br>サーバーでIConnectionのclose()を呼び出し |
    | FORCE\_CLOSE\_BASE\_USER<br>サーバーでBaseUserのcloseConnection()を呼び出し | FORCE\_CLOSE\_USER<br>サーバーでIUserのcloseConnection()を呼び出し |

<a id="10-june-30-2025-fix"></a>
#### Fix
* サーバーとHammerのプロトコルバッファをそれぞれ異なる環境で生成する際、生成環境によって互換性がなくなる可能性があった問題を修正

---

<a id="00-20241204"></a>
### 2.0.0 (2024.12.04) { #00-20241204 }

<a id="00-20241204-new"></a>
#### New

* ClientStateCheckOKログ出力有無を設定できる機能を追加
* より簡単なシナリオ作成方式を追加
    * ステートで頻繁に呼び出す機能を、事前に定義されたアクションと条件として登録できる機能を追加
    * 非実装型ステートを提供。クラスを生成せずステート名で生成可能に 
* ErrorCode追加
    * HANDLER_NOT_EXIST
    * HANDLER_ERROR
    * MATCH_PARTY_CANCEL_FAIL_ALREADY_JOINED_ROOM
    * MATCH_PARTY_CANCEL_FAIL_NOT_IN_PROGRESS
    
<a id="00-20241204-change"></a>
#### Change

* Hammerが接続状態をより正確に監視するよう修正
* ユーザーコードではサービスIDの代わりにサービス名のみ使用するように修正
* protobufの依存関係を4.28.3にアップデート

<a id="00-20241204-fix"></a>
#### Fix

* ユーザーの状態チェックレスポンスに対する設定が初期化されない問題を修正
* サーバーとプロトコルのペアを合わせた後、ログ省略メッセージリストを修正した場合に上書きされる可能性のある問題を修正
* サーバーとプロトコルのペアを合わせる際、時々ConcurrentModificationExceptionが発生する問題を修正
* ForceCloseConnectionNotiをAuthentication以前でも処理できるよう修正
* Ping再開時、遅延なくすぐにPingを実行するよう修正
* INVALID_PROTOCOLを処理してResultCodeとして受け取れるよう修正

---

<a id="40-20231213"></a>
### 1.4.0 (2023.12.13) { #40-20231213 }

<a id="40-20231213-new"></a>
#### New

* シナリオテストのユーザビリティ改善
  - シナリオアクターを通じてメソッドを直接渡す方式を追加
  - アノテーションを付与してメソッドをハンドラとして登録する方式を追加
  - シナリオテストでサーバーへリクエストし、レスポンスを受け取るまでにかかる時間を測定して結果ログに出力する機能を追加
  - 状態変化をステート内部で設定せず、シナリオを一度に作成するようにして状態遷移を一箇所で管理する機能を追加
  - シナリオテスト時、サーバーが実行中でない場合はログを通じて把握できるよう修正
  - シナリオテストログの送信及び照会機能の改善
* Payloadで圧縮パケットをサポートする機能を追加
* protobuf 3の最新バージョンにアップデート
* Protocol登録時、indexを指定しなくてもよいように改善

<a id="40-20231213-change"></a>
#### Change

* ログイン時、誤ったChannelIdを入力した場合、SystemErrorレスポンスの代わりにLogin失敗レスポンスを返すよう修正
* ConfigLoaderにパラメータとして受け取った文字列からCustomConfigをロードできるようオーバーロードを追加
* ScenarioActor.connect()のパラメータ順序を変更。
    * 他のAPIと統一されるよう、callbackを先に受け取るように修正
* パケットエンコード / デコード性能改善

<a id="40-20231213-fix"></a>
#### Fix

* StateのonEnter、onExitで例外が発生した場合、ログが残らない問題を修正
* テスト終了時点でrequestを送信してレスポンスを待たない場合、warnログが残る問題を修正
* 接続されていないConnectorではタイマーが動作しない問題を修正
* testTimeout時間が経過してもstartすべきScenarioActorが残っている場合、テストが終了しないバグを修正

---

<a id="30-20221227"></a>
### 1.3.0 (2022.12.27) { #30-20221227 }
<a id="30-20221227-new"></a>
#### New
* VM Optionを通じて設定をロードできる機能を追加

---
<a id="21-20211130"></a>
### 1.2.1 (2021.11.30) { #21-20211130 }

<a id="21-20211130-new"></a>
#### New 

* SecureSocketサポート機能を追加。
	* RemoteInfo classにuseSecureSocketオプションを追加。(default : false)
    * GameHammerConfigのtargetServerListの項目にuseSecureSocketフィールドを追加。(default : false)
	* Tester.Builder.addRemoteInfo()にuseSecureSocketの入力を受け付けるオーバーロードを追加。

---

<a id="game-gameanvil-release-notes-gamehammer-1"></a>
### 1.2.0(2021.07.13) { #game-gameanvil-release-notes-gamehammer-1 }
<a id="game-gameanvil-release-notes-gamehammer-1-change"></a>
#### Change
* パッケージ構造の整理
	* 内部用パッケージはgameanvilcoreに統合。
* ResultCode
  * ResultCodeAuth
    * AUTH_FAIL_MAINTENANCE削除 
  * ResultCodeCreateRoom
    * CREATE_ROOM_FAIL_CREATE_ROOM_ID追加
    * CREATE_ROOM_FAIL_CREATE_ROOM追加
  * ResultCodeChannelInfo
    * CHANNEL_INFO_FAIL_NO_CHANNEL_INFO追加
    * CHANNEL_INFO_FAIL_INVALID_SERVICE_ID追加
    * CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND追加
  * ResultCodeAllChannelInfo追加
  * ResultCodeChannelCountInfo追加
  * ResultCodeAllChannelCountInfo追加
  * ResultCodeChannelList
    * CHANNEL_LIST_FAIL_INVALID_SERVICEID削除
    * CHANNEL_LIST_FAIL_NO_CHANNEL_LIST追加
  * ResultCodeJoinRoom
    * JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM追加
    * JOIN_ROOM_FAIL_ALREADY_FULL追加
    * JOIN_ROOM_FAIL_ROOM_MATCH追加
  * ResultCodeLogin
    * LOGIN_FAIL_MAINTENANCE削除
  * ResultCodeMatchUserCancel
    * MATCH_USER_CANCEL_FAIL_CONTENT -> MATCH_USER_CANCEL_FAIL名前変更
    * MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS追加
  * ResultCodeMatchRoom
    * MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID追加
    * MATCH_ROOM_FAIL_CREATE_FAILED_ROOM追加
    * MATCH_ROOM_FAIL_INVALID_ROOM_ID追加
    * MATCH_ROOM_FAIL_INVALID_NODE_ID追加
    * MATCH_ROOM_FAIL_INVALID_USER_ID追加
    * MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND追加
    * MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY追加
    * MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY追加
    * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL追加
    * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL追加
  * ResultCodeMatchUserDone
    * MATCH_USER_DONE_FAIL_TRANSFER追加
    * MATCH_USER_DONE_FAIL_CREATE_ROOM追加
  * ResultCodeNamedRoom
    * NAMED_ROOM_FAIL_CREATE_ROOM追加
  * ResultCodeDisconnect
    * FORCE_CLOSE_MAINTENANCE削除
    * FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID追加。
    * FORCE_CLOSE_DISCONNECT_ALARM削除
    * FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT追加
    * FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION追加
  * ResultCodeSessionClose追加

<a id="12-20211130"></a>
### 1.1.2 (2021.11.30) { #12-20211130 }

<a id="12-20211130-new"></a>
#### New 

* SecureSocketサポート機能を追加。
	* RemoteInfo classにuseSecureSocketオプションを追加。(default : false)
    * GameHammerConfigのtargetServerListの項目にuseSecureSocketフィールドを追加。(default : false)
	* Tester.Builder.addRemoteInfo()にuseSecureSocketの入力を受け付けるオーバーロードを追加。

---

<a id="11-20210416"></a>
### 1.1.1 (2021.04.16) { #11-20210416 }

<a id="11-20210416-new"></a>
#### New 

* ping機能のオンオフが可能になるよう`Connection.setSendPingPaused()`追加。

<a id="11-20210416-fix"></a>
#### Fix

* configのpingIngervalが適用されないバグを修正
* configのpingIngervalが0の場合、pingを送信しないよう修正

---

<a id="10-20210415"></a>
### 1.1.0 (2021.04.15) { #10-20210415 }

<a id="10-20210415-change"></a>
#### Change

* サーバーのバージョンと合わせるため1.1.0へ引き上げ。

<a id="10-20210415-new"></a>
#### New

* sendPauseClientStateCheck() 追加。
* sendResumeClientStateCheck() 追加。
* サーバーから来る状態チェックレスポンスをオン/オフできるよう機能を追加

---

<a id="02-20200210"></a>
### 1.0.2 (2020.02.10) { #02-20200210 }

<a id="02-20200210-fix"></a>
#### Fix
* クライアントからゲームノードへ指定された時間(default 10秒)の間何もパケットを送信しない場合、サーバーからクライアントへ状態確認リクエストを送信することになりますが、GameHammerでこの状態確認リクエストに誤ったレスポンスを行い接続が切断される問題を修正
* シナリオテストを長時間維持し、リクエストしたパケット数が非常に多くなった場合、packetSeqがoverflowしてサーバーから応答が返ってこない問題を修正  
* send時にもpacketSeqを増加させる問題を修正

<a id="02-20200210-change"></a>
#### Change
* ログ内容の強化。
    * accountId、userId追加。
    * 最後に受信したパケット追加。
    * Ping/Pongを最後のパケットから除外
* パケットサイズを縮小するため 
    * packetSeq最大値を16383に制限
    * subId最大値を127、最小値を1に制限

---

<a id="01-20201228"></a>
### 1.0.1 (2020.12.28) { #01-20201228 }

<a id="01-20201228-fix"></a>
#### Fix
* 同じMessageに対してwaitForを重複して使用する場合、最初の応答時に全ての重複した待機が解除されるバグを修正
* ResultAuthenticationのgetPayloads()がnullを返すバグを修正
* テスト終了時に断続的にHandlerPing.onPingTime()でNullPointerExceptionが発生する問題を修正
* テスト中に断続的にStatistics.record()でConcurrentModificationExceptionが発生する問題を修正

<a id="01-20201228-change"></a>
#### Change
* GameHammerConfig.jsonファイルがない場合に出力されるログをerrorからwarnに変更。

---

<a id="00-20201218"></a>
### 1.0.0 (2020.12.18) { #00-20201218 }

<a id="00-20201218-fix"></a>
#### Fix
* EAバージョンで長時間テストが失敗する問題を修正。
* EAバージョン対比TPS性能を大幅に改善(約2倍)
* 漏れていた機能を追加。
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

<a id="00-20201218-change"></a>
#### Change
* Tester
    * 全てのリクエスト方式機能にSync/Async方式をサポート
        * Sync：リクエスト時にFutureを戻り値として受け取り、Future.get()で完了するまで待機して結果を受け取って処理する方式。
        * Async：リクエスト時にcallbackを引数として渡し、callbackで完了結果を受け取って処理する方式。
* Scenario
    * TRANSACTIONとEVENTの概念を削除
        * 代わりに各StateでchangeState()を使用して希望するStateへ直接移動

<a id="00-20201218-new"></a>
#### New
* サーバーから送信されるnotiを待って処理できるようwaitForXXX機能を追加。

---

<a id="00-ea-20200803"></a>
### 1.0.0-EA (2020.08.03) { #00-ea-20200803 }

<a id="00-ea-20200803-new"></a>
#### New
* Tester - GameAnvil Connectorの代わりにサーバーとの連携機能テストをサポート
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
    * State - ユーザーが定義する全体シナリオのうち特定の状態を表現
    * ScenarioActor - シナリオを実行する一人の仮想ユーザー
