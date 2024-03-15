## Game > GameAnvil > リリースノート > Connector-CSharp 

### 1.4.0 (2023.12.13)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.4.0.unitypackage)

#### New
Payloadで圧縮パケットをサポートします。protobuf 3.24.1でアップデートされ、Protocol登録時にindexを指定しなくてもいいように改善されました。

#### Change
ログイン時に誤ったChannelIdを入力した場合、SystemErrorレスポンスの代わりにLogin失敗レスポンスが送られるように修正されました。

#### Fix
CONNECT_ALREADY_REQUEST状態でDisconnectされない問題が修正されました。

### 1.3.0 (2022.12.27)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.3.0.unitypackage)

#### GameAnvil 1.3.0以上

#### New
高速接続、ログレベルの変更、同期機能などが新しく追加されました。GameAnvilコネクタコンポーネントを通じて、新たな機能を利用できます。
###### 高速接続
高速接続機能は、GameAnvilエンジンベースのサーバーに接続、認証、ログインする3つのプロセスを1つのメソッド呼び出しで実現させます。
###### ログレベルの変更機能
専用コンポーネントのインスペクターウィンドウを通じて、ログレベルをinfo、debug、warn、errorなどに設定し、提供されるログの頻度を調整できます。
###### 同期コンポーネントの提供
コンポーネントの提供により、Unityエンジンとの連動利便性が大幅に強化されました。サーバーの複雑なロジックを実装しなくても、遠隔のクライアント間でゲームオブジェクトのさまざまな要素が自動的に同期されます。
 
* GameObjectの作成、破壊自動同期
* Animationの同期
  * Animationパラメータの同期により、ゲームオブジェクトのAnimationを同期できます。
* Transformの同期
  * Position、Transform、ScaleなどのゲームオブジェクトのTransform要素を同期します。
  * コードによる変形を除く、どのような経路を介した変形もすべて検出します。
* RigidBodyおよびRigidBody2D同期
  * ローカルクライアントで計算された剛体状態を遠隔クライアントに転送して同期します。
  * 剛体状態に変化がない場合は、パケットを送信しないように最適化されています。
* カスタム値同期機能
  * ルーム単位でkey-valueペアでカスタム値を設定して使用できます。
  * CAS方式の値設定をサポートしているため、タイミングイシューも簡単に解決できます。

#### Fix

* マッチングに成功した後、マッチングリクエストを再送信した場合、ルームへの入場記録が誤って記録される問題を修正
* packetTimeoutオプションを0に設定した場合に、クライアントが自動的に接続を終了しないように修正
* update()メソッドを呼び出すと、ガベージが作成される問題を解決
* エラーにリスナーを登録していない状態でエラーが発生した場合、例外が発生する現象を修正
 
#### Change

* API変更:名前の変更および引数としてErrorCodeを受け取るように修正

| 変更前 | 変更後 |
|--|--|
| OnTimeout(msgId) | OnError(msgId、ErrorCode) |
| OnCustomTimeout(command) | OnCustomError(command、ErrorCode)| 

* ErrorCodeを追加

| 名称 | 説明 |
|--|--|
| ErrorCode.PARSE_ERROR | プロトコル解析に失敗したことを示す|

* ユーザー定義のプロトコルコールバックにResultCodeを追加

| 名称 | 説明 |
|--|--|
| ResultCode.PARSE_ERROR | パケット解析に失敗したことを示す |
| ResultCode.SYSTEM_ERROR| サーバーシステムエラー |
| ResultCode.TIMEOUT | タイムアウト |
| ResultCode.SUCCESS | 成功 |

* Exceptionを追加

| 名称 | 説明 |
|--|--|
| ParseInvalidProtocol | プロトコルの解析に失敗した場合に発生 |

* ロギングシステムを改善
  * ping-pongログ出力をオプションで決められる機能を追加
  * ログからメッセージ情報を出力する状況で、メッセージIDの代わりに名前を出力するように修正
  * 出力するログレベルを調整できる機能を追加


* Requestすると、すぐにパケット転送できるオプションrequestDirectを追加
* setArgumentDelegateOrListenersOnErrorオプションを追加

---

### 1.2.3 (2022.01.28)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.3.unitypackage)

#### GameAnvil 1.2.0以上

#### Fix
* サーバーから送信された圧縮パケットを処理できずにエラーが発生する問題を修正

------
### 1.2.2 (2021.11.30)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.2.unitypackage)

#### GameAnvil 1.2.0以上

#### Fix

* ルームに入場した状態でMatchRoomの呼び出しに失敗した場合、IsJoinedRoom()がfalseに変わる問題を修正

------
### 1.2.1 (2021.08.10) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.1.unitypackage)

#### GameAnvil 1.2.0以上

#### Fix

* SocketExceptionが発生すると、OnDisconnectが2回呼び出されるバグを修正

------

### 1.2.0 (2021.07.13) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.0.unitypackage)

#### GameAnvil 1.2.0以上

#### Change

* Send()のリターンタイプがvoidに変更
* Config.defaultReqestTimeoutのデフォルト値が3秒に変更
* Request()または他のAPIを呼び出す時にCallbackを引数として一緒に渡すと、引数で渡されたcallbackでのみレスポンスするように変更
    * Config.useArgumentDelegateOnlyを追加(デフォルト値true)
    * Config.useArgumentDelegateOnlyがfalseの場合、Request()または他のAPIを呼び出す際に、Callbackを引数で一緒に渡しても引数で渡されたcallbackとあらかじめ登録しておいたlistenerが同時に呼び出される。(以前のバージョンの動作方式)
* Exceptions
    * NotConnectedを追加
    * NotAuthenticatedを追加
    * NotLoggedInを追加
    * MatchUserInProgressを追加
    * MatchPartyInProgressを追加
    * MatchUserNotInProgressを追加
    * MatchPartyNotInProgressを追加
* ConnectionAgent
    * サーバーに接続していない状態でリクエストを送信した場合、NotConnected例外が発生
    * Authenticatedされていない状態でリクエストを送信した場合、NotAuthenticated例外が発生
    * PauseClientStateCheck()、ResumeClientStateCheck()は例外。
    * LoginedUserInfo.payloadを削除
    * GetAllChannelInfo()を追加
    * GetChannelCountInfo()を追加
    * GetAllChannelCountInfo()を追加
* UserAgent
    * Loginしていない状態でリクエストを送信した場合、NotLoggedIn例外が発生
    * LoginInfo.Messageを削除
    * GetChannelInfo()を追加
    * GetAllChannelInfo()を追加
    * GetChannelCountInfo()を追加
    * GetAllChannelCountInfo()を追加
    * CreateRoom()にmatchingGroupパラメータを追加
    * JoinRoom()にmatchingUserCategoryパラメータを追加
    * MatchRoom()にmatchingGroup, matchingUserCategoryパラメータを追加。
    * MatchUserStart()にmatchingGroupパラメータを追加
    * MatchPartyStart()にmatchingGroupパラメータを追加
    * IsUserMatchInProgress()、IsPartyMatchInProgress()、IsMatchInProgress()を追加
    * UserMatch中にUserMatchCancelを除外したapiを呼び出すと、UserMatchInProgress例外が発生
        * UserMatch中ではない時にUserMatchCancelを呼び出すと、UserMatchNotInProgress例外が発生
        * PartyMatch中にPartyMatchCancelを除外したapiを呼び出すと、UserMatchInProgress例外が発生
        * PartyMatch中ではない時にUserMatchCancelを呼び出すと、PartyMatchNotInProgress例外が発生
* ResultCode
	* ResultCodeConnect
    	* CONNECT_FAIL_INVALID_IPを追加
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

------
### 1.1.6 (2023.01.20)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.6.unitypackage)

#### GameAnvil 1.1.0以上

#### Fix
* useIpv6オプションを有効にした状態でconnect()を呼び出すと、ブロックされることのある問題を修正 

------

### 1.1.5 (2022.01.28)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.5.unitypackage)

#### GameAnvil 1.1.0以上

#### Fix
* サーバーから送信された圧縮パケットを処理できずにエラーが発生する問題を修正

------
### 1.1.4 (2021.11.30) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.4.unitypackage)

#### GameAnvil 1.1.0以上

#### Fix
* ルームに入場した状態でMatchRoomの呼び出しに失敗した場合、IsJoinedRoom()がfalseに変わる問題を修正

------
### 1.1.3 (2021.08.10) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.3.unitypackage)

#### GameAnvil 1.1.0以上
#### Fix
* SocketException発生時にOnDisconnectが2回呼び出されるバグを修正

------

### 1.1.2 (2021.04.15) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.2.unitypackage)

#### GameAnvil 1.1.0以上
#### New
* GameAnvil ServerのClientStateCheck機能を一時停止させるConnectionAgent.PauseClientStateCheck()を追加。アプリがバックグラウンドに移動するなど、メッセージループが動作しなくなる場合に呼び出してくれる。
* 一時停止していたClientStateCheck機能を再び動作させるConnectionAgent.ResumeClientStateCheck()を追加。アプリがバックグラウンドから復帰するなど、メッセージループが再び動作すると呼び出してくれる。
* Singleserver.SetOnPauseClientStateCheck()を追加。
* Singleserver.SetOnResumeClientStateCheck()を追加。
#### Fix
* OnDisconnectでforceがfalseの場合、ResultCodeDisconnect値が0になるバグを修正

------



### 1.1.1 (2021.04.07) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.1.unitypackage)

#### GameAnvil 1.1.0以上
#### Change
* listenerの個別登録可否を確認できるContainsListenerオーバーローディングを追加。 
* メッセージごとにすべてのlistenerを削除できるRemoveAllListenersForMsg、RemoveAllListenersForMsgIdを追加
* ContainsUserListener、ContainsUserNotificationListener、ContainsUserErrorListenerを追加。
* ContainsConnectionListener, ContainsConnectionNotificationListener, ContainsConnectionErrorListener, ContainsConnectionErrorListenerを追加。
#### Fix
* 強制終了、ログアウト、ログイン失敗などの状況でUserAgentの一部の情報が初期化されないバグを修正

------



### 1.1.0 (2020.12.18) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.0.unitypackage)

#### GameAnvil 1.1.0以上
#### Change
* .Net 4.5以上サポートに完全移行
	* 使用ライブラリをすべて.Net 4.5用の最新バージョンに変更
* ConnectionAgent 
	* IsConnected() :接続の可否を確認するAPIを追加。
	* IsAuthenticated() :認証されたかどうかを確認するAPIを追加。
* SingleServerのGameAnvil.Action → GameAnvil.Handlerに名前を変更。

------

### 1.0.0 (2020.08.31) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.0.0.unitypackage)

#### GameAnvil 1.0.0以上
#### Change

* MoveServiceを削除
* Reconnect、Retry機能を削除
* PacketHelperのGetMessageにTypeをパラメータで受け取るAPIを追加。
* 各ResultCodeに固有数字を適用

* ResultCodeの追加および名前を変更

	* <span style="color:#eb6420">現在、FORCE_CLOSE_DUPLICATE_LOGINケースにFORCE_CLOSE_BY_NEW_CONNECTIONが移行する問題がある。次のリリースで修正予定。</span>

#### Fix

* Disconnect時にもUserAgentのisLoginがtrueを返す問題を修正



#### ResultCodeの変更事項

* ResultCodeAuth
	* 追加
		* AUTH_FAIL_INVALID_ACCOUNT_ID
* ResultCodeLogin
	* 名前を変更
		* LOGIN_FAIL_EMPTY_SUB_ID → LOGIN_FAIL_INVALID_SUB_ID
	* 削除
		* LOGIN_FAIL_LOGINED_SAME_SERVICE
	* 追加。
		* LOGIN_FAIL_INVALID_USERID : 誤ったユーザーID。
		* LOGIN_FAIL_LOGINED_OTHER_USER_TYPE : 同じAccount IDで他のUserTypeがログインしている。
		* LOGIN_FAIL_LOGINED_OTHER_DEVICE : 同じAccount IDで他のDeviceIdがログイン している。
* ResultCodeMatchRoom
	* 削除
		* MATCH_ROOM_FAIL_UNKNOWN_ERROR
* ResultCodeMatchUserStart
	* 削除
		* MATCH_USER_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchUserCancel
	* 削除
		* MATCH_USER_CANCEL_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyStart
	* 削除
		* MATCH_PARTY_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyCancel
	* 削除
		* MATCH_PARTY_CANCEL_FAIL_MATCH_UNKNOWN_ERROR        
* ResultCodeNamedRoom
	* 追加。
		* NAMED_ROOM_FAIL_INVALID_ROOM_NAME : 誤ったルームの名前を送信した場合。
* ResultCodeDisconnect
	* 名前を変更
		* FORCE_CLOSE_SYSTEM → FORCE_CLOSE_SYSTEM_ERROR
		* FORCE_CLOSE_CONTENT →
		FORCE_CLOSE_BASE_CONNECTION :サーバーからBaseConnectionのclose()を呼び出す
		FORCE_CLOSE_BASE_USER :サーバーからBaseUserのcloseConnection()を呼び出す
	* 追加。
		* FORCE_CLOSE_INVALID_NODE : GameNodeがinvalid状態に変更された場合。
		* FORCE_CLOSE_USER_TRANSFER_FAIL :ユーザートランスファーが失敗した場合。
		* FORCE_CLOSE_USER_TRANSFER_ERROR :ユーザートランスファー中にシステムエラーが発生した場合。
	* 追加されたが、クライアントから受け取れない場合。
		サーバーでは、クライアントの接続がすでに切断されたと予想して接続を強制終了し、以下のコードを使用する。
		クライアントの接続が切断されているため、以下のコードを受け取ってはならない。
		このコードを受け取った場合、GameAnvil開発チームに情報を提供してください。
		* FORCE_CLOSE_BY_NEW_CONNECTION : 同じアカウント情報で新たなログインリクエストが入ってきた場合。 
		* <span style="color:#eb6420">現在、FORCE_CLOSE_DUPLICATE_LOGINケースにこのコードが移行する問題がある。
          次のリリースで修正予定。</span>
		* FORCE_CLOSE_DISCONNECT_ALARM :クライアントがサーバーの状態チェックにレスポンスしていない場合。
		* FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL :クライアントがサーバーの状態チェックにレスポンスしていない場合。
		* FORCE_CLOSE_GHOST_USER : ゴーストユーザーである場合。


-----



### 1.0.0-EA2 (2020.07.07)

#### C-Sharp
##### Change

* 名前変更: Gameflex → GameAnvil
* RemoveAllListeners() 
	* boolパラメータを追加。
	* falseの場合、userListener、connectonListenerは削除されない。
	* defalult = true,
* RemoveAllUserListeners()を追加。
* RemoveAllConnectionListeners()を追加。

-----

### 1.0.0-EA (2020.06.29)

#### C-Sharp
##### Change

* 名前変更: Tardis → Gameflex
	* TardisSocket → Socket
	* SessionAgent → ConnectionAgent
* Type変更
	* UserIdのtypeがstringからintに変更
	* SubIdのtypeがstringからintに変更
	* RoomIdのtypeがstringからintに変更
* CreateUserAgent(), GetUserAgent()
	* パラメータを`string SubId` → `int SubId`に変更
	* SubId > 0でなければならない。
* LoginedUserInfo
	* UserId項目を追加。
	* Payload項目を追加。
* LoginInfo
	* userId項目を追加。
	* userType項目を追加。
	* roomName項目を追加。
* UserAgent
	* パラメータを`string roomId` → `int roomId`に変更。
	* RequestToSessionActor() ->RequestToGatewaySession()
	* SendToSessionActor() → SendToGatewaySession()
	* RequestToSessionActor() → RequestToGatewaySession()
	* CreateRoom() - パラメータ`string roomName`を追加。
	* onCreateRoom() - パラメータ`string roomName`を追加。
	* onJoinRoom() - パラメータ`string roomName`を追加。
	* onMatchRoom() 
		* パラメータ`string roomName`を追加。
		* パラメータ`bool created`を追加。
	* onNamedRoom()
		* パラメータ`string roomName`を追加。
		* パラメータ`bool created`を追加。

##### New

* エラーコード
	* ResultCodeLogin
		* LOGIN_FAIL_EMPTY_SUB_ID
		* LOGIN_FAIL_TIMEOUT_GAME_SERVER
	* ResultCodeMoveChannel
		* MOVE_CHANNEL_FAIL_NODE_NOT_FOUND
		* MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL
		* MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM
* Exception
	* InvalidSubId

-----



### 0.12.1.1 (2020.06.23)

#### C-Sharp

##### Change

* ProtocolManger.unregister()を使用すると、exceptionが発生するバグを修正

-----



### 0.12.1 (2020.04.06)

#### C-Sharp

##### Change

* XML文書の注釈を適用
* Agent.LvNetLogger, Loggerを削除。
* Agent.ContainsListener(int customMsgId) → AgentHelperに移動
* SessionAgent.AddCustomMsgListener、RemoveCustomMsgListener → AgentHelper.RemoveListener()に代替
* UserAgent.AddCustomMsgListener、RemoveCustomMsgListener → AgentHelper.AddListener、RemoveListenerに代替
* Agent.FlushQueueを削除
* Connector.FlushNetworkQueueを削除
* Packet.GetMsgId、GetRetryCount、SetRetryCount、setUncompressSize、GetServiceId、GetSubId → 削除
* ProtocolManager.RegisterProtocol()でprotocolIdの最大値をチェック(最大510)
* 誤ったProtocolIdを使用すると、InvalidProtocolId Exceptionが発生するように修正
* OnNamedRoomからRoomName → RoomIdに名前を変更

##### Fix

* MatchUserDoneの場合、isJoinRoomセッティングができないバグを修正

-----



### 0.12.0 (2020.02.14)

#### C-Sharp

##### Change

* パラメータ名を変更: msgIndex → customMsgId
	* Agent
		* public bool ContainsListener(int customMsgId)
	* SessionAgent
		* public void AddCustomMsgListener(int custonMsgId、Action\<SessionAgent、Packet> listener)
		* public void RemoveCustomMsgListener(int customMsgId、Action\<SessionAgent、Packet> listener)
	* UserAgent
		* public void AddCustomMsgListener(int custonMsgId、Action\<UserAgent、Packet> listener)
		* public void RemoveCustomMsgListener(int customMsgId、Action\<UserAgent、Packet> listener)
	* Payload
		* public Packet getPacket(int customMsgId)

* Packet
	* public Packet(int descId、int msgIndex、byte[] buffer)を削除
	* public Packet(int msgIndex, byte[] buffer)を削除
	* public string GetFileDescriptorName()を削除
	* public void SetDescId(int descId)を削除
	* public int GetDescId()を削除
	* public int GetMsgIndex()を削除
	* public Packet setTimeout(int timeout)を削除
	* public int GetTimeout()を削除

##### New

* ResultCodeMatchRoom
	* MATCH\_ROOM\_FAIL\_UNKNOWN\_ERROR = 6,
	* MATCH\_ROOM\_FAIL\_MATCHED\_ROOM\_DOES\_NOT\_EXIST = 7
* Packet
	* public static Packet CreateWithCustomMsg(int customMsgId, byte[] buffer)
	* public int GetMsgId()
