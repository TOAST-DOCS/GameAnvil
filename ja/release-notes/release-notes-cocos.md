## Game > GameAnvil > リリースノート > Connector-Typescript
### 1.2.1 (2021.11.30) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.2.1.zip)

#### GameAnvil 1.2.0以上

#### Fix
*ルームに入場した状態でMatchRoomの呼び出しに失敗した場合、IsJoinedRoom()がfalseに変わる問題を修正

------

### 1.2.0 (2021.07.13) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.2.0.zip)

#### GameAnvil 1.2.0以上

#### Change

* Request()または他のAPIを呼び出す時にCallbackを引数として一緒に渡すと、引数で渡されたcallbackでのみレスポンスするように変更
  * Config.useArgumentCallbackOnlyを追加(デフォルト値true)
  * Config.useArgumentCallbackOnlyがfalseの場合、Request()または他のAPIを呼び出す際に、Callbackを引数で一緒に渡しても引数で渡されたcallbackとあらかじめ登録しておいたlistenerが同時に呼び出される。(以前バージョンの動作方式)
* ConnectionAgent
  * サーバーに接続していない状態でリクエストを送信した場合、"Not connected. Connect before call XXX."エラーが発生
  * Authenticatedされていない状態でリクエストを送信した場合、Not authenticated. Authenticate before call XXX."エラーが発生
    * PauseClientStateCheck(), ResumeClientStateCheck()は例外。
  * LoginedUserInfo.payloadを削除
  * GetAllChannelInfo()を追加
  * GetChannelCountInfo()を追加
  * GetAllChannelCountInfo()を追加
* UserAgent
  * Loginしていない状態でリクエストを送信した場合、"Not loggedIn. Login before call XXX()."エラーが発生
  * LoginInfo.messageを削除
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
  * UserMatch中にUserMatchCancelを除外したapiを呼び出すと、"MatchUser is in progress. MatchUserCancel before call XXX."エラーが発生
   * UserMatch中ではない時にUserMatchCancelを呼び出すと、"MatchUser is not in progress."エラーが発生
   * PartyMatch中にPartyMatchCancelを除外したapiを呼び出すと、"MatchParty is in progress. MatchPartyCancel before call XXX."エラーが発生
   * PartyMatch中ではない時にUserMatchCancelを呼び出すと、"MatchParty is not in progress."エラーが発生
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

------
### 1.1.4 (2021.11.30) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.4.zip)

#### GameAnvil 1.1.0以上

#### Fix
* ルームに入場した状態でMatchRoomの呼び出しに失敗した場合、IsJoinedRoom()がfalseに変わる問題を修正

------

### 1.1.3 (2021.04.07) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.3.zip)

#### GameAnvil 1.1.0以上
#### Change
* ContainsCallback、ContainsUndefinedProtocolCallback、ContainsListenerを追加。
* RemoveCallback、RemoveUndefinedProtocolCallbackにcallbackごとに削除可能なcallbackオプショナルパラメータを追加。
* RemoveListenerのlistenerをオプショナルパラメータに変更。
* ContainsOnError、ContainsOnDisconnectを追加。
* RemoveOnError、RemoveOnDisconnectにcallbackごとに削除可能なcallbackオプショナルパラメータを追加。
#### Fix
* ログアウトやログインに失敗した時にUserAgentの一部情報が初期化されないバグを修正
* AddOnErrorで登録されたコールバックの引数に誤った値が入るバグを修正
* RequestPBを呼び出した時にcallbackを入れない場合、例外が発生する問題を修正

------



### 1.1.2 (2021.03.16) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.2.zip)

#### GameAnvil 1.1.0以上
#### Change
* Connector.configのdefault値を変更。
	* PingInterval : 10 → 3
	* PacketTimeout : 12 → 5
#### Fix
* CodeInserterで.protoのメッセージ内部に重複接続メッセージ宣言がある場合、その次のメッセージからindex値が滞るバグを修正

------



### 1.1.1 (2021.02.10) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.1.zip)

#### GameAnvil 1.1.0以上
#### Change
* SingleServer 
	* onMatchRoomを使用している時にpayloadにnullが来る問題を修正
	* OnLoginにid: stringパラメータをsubId: numberに変更
	* OnCreateRoom、OnJoinRoom、OnMatchRoom、OnNamedRoomに脱落したroomNameパラメータを追加。
	* OnMatchRoom、OnNamedRoomに脱落したcreatedパラメータを追加。 
	* OnXXXを登録する時にすでに登録されたコールバックを削除。
	* OnLogin()レスポンスでloginInfoにnullを渡す場合、serviceIdにリクエストされたserviceIdを入れて、レスポンスするように例外処理を追加。
	* 登録されたコールバックでエラーが発生した場合、callstack情報がユーザーにアップロードできるように再びthrow

------



### 1.1.0 (2020.12.18) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.0.zip)

#### GameAnvil 1.1.0以上
#### Change
* 互換性のあるイシューに修正して、GitEnterprizeに載せて使用していたprotobufjsを問題が修正された公式最新バージョンに交換

------



### 1.0.1 (2020.10.08) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.0.1.zip)

#### GameAnvil 1.0.0以上
#### FIX

* Requestを同時に何度も呼び出した場合に呼び出し順序の逆にパケットを転送するバグを修正

------



### 1.0.0 (2020.08.31) 

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.0.0.zip)

#### GameAnvil 1.0.0以上
#### Change
* MoveServiceを削除
* Reconnect、Retry機能を削除
* 各ResultCodeに固有数字を適用
* ResultCodeを追加および名前を変更
	* <span style="color:#eb6420">現在、FORCE_CLOSE_DUPLICATE_LOGINケースにFORCE_CLOSE_BY_NEW_CONNECTIONが移行する問題がある。
        次のリリースで修正予定。</span>



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
		* LOGIN_FAIL_INVALID_USERID : 誤ったユーザーID
		* LOGIN_FAIL_LOGINED_OTHER_USER_TYPE : 同じAccount IDで異なるUserTypeがログインしている。
		* LOGIN_FAIL_LOGINED_OTHER_DEVICE : 同じAccount IDで異なるDeviceIdがログインしている。
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
		FORCE_CLOSE_BASE_CONNECTION :サーバーでBaseConnectionのclose()を呼び出し
		FORCE_CLOSE_BASE_USER :サーバーでBaseUserのcloseConnection()を呼び出し
	* 追加。
		* FORCE_CLOSE_INVALID_NODE : GameNodeがinvalid状態で変更された場合。
		* FORCE_CLOSE_USER_TRANSFER_FAIL :ユーザートランスファーが失敗した場合。
		* FORCE_CLOSE_USER_TRANSFER_ERROR :ユーザートランスファー中にシステムエラーが発生した場合。
	* 追加されたがクライアントから受け取れない場合。
		サーバーでは、クライアントの接続がすでに切断されたと予想して接続を強制終了し、以下のコードを使用する。
		クライアントの接続が切断されているため、以下のコードを受け取ってはならない。
		このコードを受け取った場合、GameAnvil開発チームに情報を提供してください。
		* FORCE_CLOSE_BY_NEW_CONNECTION : 同じアカウント情報で新たなログインリクエストが入ってきた場合。 
		* <span style="color:#eb6420">現在、FORCE_CLOSE_DUPLICATE_LOGINケースに、このコードが移行する問題がある。
          次のリリースで修正予定。</span>
		* FORCE_CLOSE_DISCONNECT_ALARM :クライアントがサーバーの状態チェックにレスポンスしていない場合。
		* FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL :クライアントがサーバーの状態チェックにレスポンスしていない場合。
		* FORCE_CLOSE_GHOST_USER : ゴーストユーザーである場合。


-----



### 1.0.0-EA3 (2020.07.27)
#### Change
* ConnectionAgentにIsReconnecting()を追加。

#### FIX
* UserAgentがログインした後、disconnectされた時にisLogin()がtrueをリターンする問題を修正
* UserAgentを複数使用した時にrequestへのレスポンスを受け取れず、disconnectされた場合、requestに一緒に渡したcallbackが消えずに残り、次のrequest時にcallbackが2回呼び出される問題を修正

-----



### 1.0.0-EA2 (2020.07.07)
#### Change

* 名前を変更: Gameflex → GameAnvil
* RemoveAllCallback()を追加。

-----




### 1.0.0-EA (2020.06.29)
#### Change
* 名前を変更: Tardis → Gameflex
	* SessionAgent → ConnectionAgent
* Typeを変更
	* UserIdのtypeがstringからnumberに変更
	* SubIdのtypeがstringからnumberに変更
	* RoomIdのtypeがstringからnumberに変更
* CreateUserAgent()、GetUserAgent()
	* パラメータ`SubId: string` → `SubId: number`に変更
	* SubId > 0でなければならない。
* LoginedUserInfo
	* UserId項目を追加。
	* Payload項目を追加。
* LoginInfo
	* userId項目を追加。
	* userType項目を追加。
	* roomName項目を追加。
* UserAgent
	* パラメータ`roomId: string` → `roomId: number`に変更。
	* MultiSendToSessionActor()→MultiSendToGatewaySession()
	* SendToSessionActor() → SendToGatewaySession()
	* SendPbToSessionActor() → SendPbToGatewaySession()
	* RequestToSessionActor() → RequestToGatewaySession()
	* MultiRequestToSessionActor() → MultiRequestToGatewaySession()
	* RequestToSessionActor() → RequestToGatewaySession()
	* RequestPbToSessionActor() → RequestPbToGatewaySession()
	* RequestPbAsyncToSesstionActor() → RequestPbAsyncToGatewaySession()
	* RequestAsyncToSessionActor() → RequestAsyncToGatewaySession()
	* CreateRoom() - パラメータ`roomName: string`を追加。
	* onCreateRoom() - パラメータ`roomName: string`を追加。
	* onJoinRoom() - パラメータ`roomName: string`を追加。
	* onMatchRoom() 
		* パラメータ`roomName: string`を追加。
		* パラメータ`created: boolean`を追加。
	* onNamedRoom()
		* パラメータ`roomName: string`を追加。
		* パラメータ`created: boolean`を追加。

#### New
* エラーコード
	* ResultCodeLogin
		* LOGIN_FAIL_EMPTY_SUB_ID
		* LOGIN_FAIL_TIMEOUT_GAME_SERVER
	* ResultCodeMoveChannel
		* MOVE_CHANNEL_FAIL_NODE_NOT_FOUND
		* MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL
		* MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM

-----



### 0.12.1.1 (2020.06.23)
#### Change

* onDisconnect()コールバックから直接connect()を呼び出した場合にエラーが発生する問題を修正

-----



### 0.12.1 (2020.04.06)
#### Change

* JSDocを適用
* UserAgent.isJoinRoomを追加。
* IUserListener.OnMatchPartyCancel()からPayloadを削除。
* IUserListener.OnNamedRoom()からroomName→ RoomIdに名前を変更
* Payload.createDefault()を削除。 CreateEmpty()に代替
* SessionAgent.addOnDisconnect()
	* AddOnDisconnect(callback: (agent: SessionAgent, message: string) => void): void; =>
	AddOnDisconnect(callback: (session: SessionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload) => void): void;

-----



### 0.12.0 (2020.02.14)
#### Change

* Packet
	* GetDescId(): number;を削除
	* GetMsgIndex(): number;を削除
	* GetTimeOut(): number;を削除
	* SetTimeOut(timeOut: number): void;を削除

#### New

* ResultCodeMatchRoom
	* MATCH\_ROOM\_FAIL\_UNKNOWN\_ERROR = 6,
	* MATCH\_ROOM\_FAIL\_MATCHED\_ROOM\_DOES\_NOT\_EXIST = 7
* Packet
	* GetMsgId(): number;
* ProtocolManager
	* static getMsgId(descId: number, msgIndex: number): number;
	* static getMsgIdFromPb\(msg: IMessage \| \(new \(\) =\> T\)\): any;
	* static getCustomMsgId(customMsgId: number): number;
