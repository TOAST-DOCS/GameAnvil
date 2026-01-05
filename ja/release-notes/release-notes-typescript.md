## Game > GameAnvil > リリースノート > Typescript Connector

### 2.1.0 (2025.06.30)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector-typescript.zip)
#### GameAnvil 2.0.0以上


#### New
* GameAnvil 2.1.0サーバーのリリースに合わせて、コネクタも2.1.0バージョンをリリースします。
* 2.0.0と比較して機能上の大きな変更点はなく、一部のバグ修正、ResultCodeの名称変更、誤字脱字や不適切な説明などの修正があります。

#### Changed
* GameAnvil 2.1サーバーに合わせてエンジンプロトコルをアップデート
    * GameAnvil 2.1以前のバージョンのサーバーは今後サポートされません
* 一部のResultCodeを変更

  | 変更前                                                                       | 変更後                                                                     |
  |-----------------------------------------------------------------------------|---------------------------------------------------------------------------|
  | LOGIN\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | LOGIN_FAIL_INVALID_SERVICE_NAME<br>失敗。不正なサービス名 |
  | LOGIN\_FAIL\_INVALID\_USERTYPE<br>失敗。不正なユーザータイプ | LOGIN_FAIL_INVALID_USER_TYPE<br>失敗。不正なユーザータイプ |
  | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>失敗。不正なサービス名 |
  | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>失敗。不正なサービス名 |
  | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_FORM\_NULL<br>失敗。マッチング申込書がnullの場合 | MATCH\_ROOM\_FAIL\_MATCH\_FORM\_NULL<br>失敗。マッチング申込書がnullの場合 |
  | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_INFO\_NULL<br> 失敗。マッチング情報がnullの場合 | MATCH\_ROOM\_FAIL\_MATCH\_INFO\_NULL<br> 失敗。マッチング情報がnullの場合 |
  | FORCE\_CLOSE\_BASE\_CONNECTION<br>サーバーでBaseConnectionのclose()を呼び出し | FORCE\_CLOSE\_CONNECTION<br>サーバーでIConnectionのclose()を呼び出し |
  | FORCE\_CLOSE\_BASE\_USER<br>サーバーでBaseUserのcloseConnection()を呼び出し | FORCE\_CLOSE\_USER<br>サーバーでIUserのcloseConnection()を呼び出し |

#### Fix
* Request時に内部で使用されるseq値が最大値を超えた場合、レスポンスを受信できない可能性があった問題を修正

---

### 2.0.0 (2024.12.05)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-2.0.0.zip)

#### GameAnvil 2.0.0以上

#### New

* async awaitを使用した新規コネクタがリリースされました。
* 今後、完了時に処理する関数を登録せず、戻り値で完了を確認します。
* GameAnvil接続 + 認証コードは次のとおりです。

```typescript
const deviceId, accountId, password;

connector.host = "127.0.0.1";
connector.port = 18300;

const authResult = await connector.connectAndAuthenticateion(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);
```

#### Remove

###### ConnectionAgent削除
* ConnectionAgentはGameAnvilConnectorと区分が曖昧でした。
* 今後、GameAnvilConnectorで既存のConnectionAgentの動作を含みます。
* この変更に合わせて他のクラスの名前も変更されました。
    * UserAgentはGameAnvilUserに名前が変更されました。
    * ProtocolManagerはGameAnvilProtocolManagerに名前が変更されました。

###### Connector.Update削除
* 今後、Connector内部で自動的に動作します。

###### リクエストのレスポンスを受け取る目的のデリゲート削除
* 今後、リクエストのレスポンスは戻り値として受け取ります。
* サーバーからリクエストなしに受信する種類のメッセージは、従来通りデリゲートを使用します。
    * SetMessageCallbackメソッド

#### Change

###### UserAgentのインスタンス直接管理
* 以下の例のように、UserAgentのインスタンスを直接生成して使用するように変更されました。
* UserAgent.Dispose時に自動的にログアウトしないため注意してください。UserAgent.Disposeは内部で管理するオブジェクトのみを解放します。

```typescript
const myUser = new GameAnvilUser(connector, "ServiceName", subId);
```

###### Packetクラスのユーザビリティ変更
* 今後、常にbyte[]への変換を試みることはしません。
* 今後、パケットの圧縮は生成時に追加できます。
* 今後、パケットの解凍は自動的に動作します。

###### GameAnvilConnector同時性
* 複数のリクエストを送受信する際、一度に1つしか送信しなかった問題を修正しました。
* 今後、複数のリクエストを送受信できます。

###### MultiRequest削除
* サーバーからMultiRequestの機能が削除されたことに伴い、GameAnvilConnectorからもこれに対応する機能が削除されました。
    * `public void Request(List<Packet> packetList)`
    * `public bool Send(List<Packet> packetList)`

------

### 1.2.1 (2021.11.30)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.2.1.zip)

#### GameAnvil 1.2.0以上

#### Fix
* ルームに入室した状態でMatchRoomを呼び出して失敗した場合、IsJoinedRoom()がfalseに変わる問題を修正

------

### 1.2.0 (2021.07.13)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.2.0.zip)

#### GameAnvil 1.2.0以上

#### Change

* Request()または他のAPI呼び出し時にCallbackを引数として一緒に渡すと、引数として渡したコールバックにのみレスポンスが届くように変更
    * Config.useArgumentCallbackOnly追加（デフォルト値true）
    * Config.useArgumentCallbackOnlyがfalseの場合、Request()または他のAPI呼び出し時にCallbackを引数として一緒に渡しても、引数として渡したコールバックと事前に登録したリスナーが同時に呼び出されます。（旧バージョンの動作方式）
* ConnectionAgent
    * サーバーに接続していない状態でリクエストを送信した場合、"Not connected. Connect before call XXX." エラー発生
    * 認証されていない状態でリクエストを送信した場合、"Not authenticated. Authenticate before call XXX." というエラーが発生します。
        * PauseClientStateCheck()、ResumeClientStateCheck()は例外。
    * LoginedUserInfo.payload削除
    * GetAllChannelInfo()追加
    * GetChannelCountInfo()追加
    * GetAllChannelCountInfo()追加
* UserAgent
    * Loginしていない状態でリクエストを送信した場合、"Not loggedIn. Login before call XXX()." エラー発生
    * LoginInfo.message削除
    * GetChannelInfo()追加
    * GetAllChannelInfo()追加
    * GetChannelCountInfo()追加
    * GetAllChannelCountInfo()追加
    * CreateRoom()にmatchingGroupパラメータ追加
    * JoinRoom()にmatchingUserCategoryパラメータ追加
    * MatchRoom()にmatchingGroup、matchingUserCategoryパラメータ追加。
    * MatchUserStart()にmatchingGroupパラメータ追加
    * MatchPartyStart()にmatchingGroupパラメータ追加
    * IsUserMatchInProgress()、IsPartyMatchInProgress()、IsMatchInProgress()追加
    * UserMatch中にUserMatchCancelを除くAPI呼び出し時、"MatchUser is in progress. MatchUserCancel before call XXX." エラー発生
    * UserMatch中でない時にUserMatchCancel呼び出し時、"MatchUser is not in progress." エラー発生
    * PartyMatch中にPartyMatchCancelを除くAPI呼び出し時、"MatchParty is in progress. MatchPartyCancel before call XXX." エラー発生
    * PartyMatch中でない時にUserMatchCancel呼び出し時、"MatchParty is not in progress."エラー発生
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

------
### 1.1.4 (2021.11.30)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.4.zip)

#### GameAnvil 1.1.0以上

#### Fix
* ルームに入室した状態でMatchRoomを呼び出して失敗した場合、IsJoinedRoom()がfalseに変わる問題を修正

------

### 1.1.3 (2021.04.07)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.3.zip)

#### GameAnvil 1.1.0以上
#### Change
* ContainsCallback、ContainsUndefinedProtocolCallback、ContainsListener追加。
* RemoveCallback、RemoveUndefinedProtocolCallbackにcallbackを個別に削除できるようにcallbackオプショナルパラメータ追加。
* RemoveListenerのlistenerをオプショナルパラメータに変更。
* ContainsOnError、ContainsOnDisconnect追加。
* RemoveOnError、RemoveOnDisconnectにcallbackを個別に削除できるようにcallbackオプショナルパラメータ追加。
#### Fix
* ログアウト、ログイン失敗などの状況でUserAgentの一部情報が初期化されないバグを修正
* AddOnErrorで登録されたコールバックの引数に誤った値が渡されるバグを修正
* RequestPB呼び出し時にcallbackを入れない場合、例外が発生する問題を修正

------



### 1.1.2 (2021.03.16)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.2.zip)

#### GameAnvil 1.1.0以上
#### Change
* Connector.configのdefault値変更。
    * PingInterval : 10 -> 3
    * PacketTimeout : 12 -> 5
#### Fix
* CodeInserterで.protoのメッセージ内部に入れ子メッセージ宣言がある場合、次のメッセージからindex値がずれるバグを修正

------



### 1.1.1 (2021.02.10)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.1.zip)

#### GameAnvil 1.1.0以上
#### Change
* SingleServer
    * onMatchRoom使用時、payloadにnullが渡される問題を修正
    * OnLoginのid: stringパラメータをsubId: numberに変更
    * OnCreateRoom、OnJoinRoom、OnMatchRoom、OnNamedRoomに漏れていたroomNameパラメータ追加。
    * OnMatchRoom、OnNamedRoomに漏れていたcreatedパラメータ追加。
    * OnXXX登録時、既存の登録済みコールバックを削除。
    * OnLogin()のレスポンスとしてloginInfoにnullを渡す場合、serviceIdにリクエストされたserviceIdを入れてレスポンスするよう例外処理を追加。
    * 登録されたコールバックでエラーが発生した場合、callstack情報がユーザーに伝わるように再throw

------



### 1.1.0 (2020.12.18)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.1.0.zip)

#### GameAnvil 1.1.0以上
#### Change
* 互換性の問題で修正してGitEnterprizeにアップロードして使用していたprotobufjsを、問題が修正された公式最新バージョンに交換

------



### 1.0.1 (2020.10.08)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.0.1.zip)

#### GameAnvil 1.0.0以上
#### FIX

* Requestを同時に複数回呼び出す場合、呼び出し順序の逆順でパケットを送信するバグを修正

------



### 1.0.0 (2020.08.31)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript-1.0.0.zip)

#### GameAnvil 1.0.0以上
#### Change
* MoveService削除
* Reconnect、Retry機能削除
* 各ResultCodeに固有の数値を適用
* ResultCode追加および名前変更
    * <span style="color:#eb6420">現在FORCE_CLOSE_DUPLICATE_LOGINケースにFORCE_CLOSE_BY_NEW_CONNECTIONが渡される問題があります。
      次回リリース時に修正される予定です。</span>



#### ResultCode詳細変更事項

* ResultCodeAuth
    * 追加
        * AUTH_FAIL_INVALID_ACCOUNT_ID
* ResultCodeLogin
    * 名前変更
        * LOGIN_FAIL_EMPTY_SUB_ID -> LOGIN_FAIL_INVALID_SUB_ID
    * 削除
        * LOGIN_FAIL_LOGINED_SAME_SERVICE
    * 追加。
        * LOGIN_FAIL_INVALID_USERID：誤ったユーザーID。
        * LOGIN_FAIL_LOGINED_OTHER_USER_TYPE：同一Account IDで別のUserTypeがログインしている。
        * LOGIN_FAIL_LOGINED_OTHER_DEVICE：同一Account IDで別のDeviceIdがログインしている。
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
        * NAMED_ROOM_FAIL_INVALID_ROOM_NAME：誤ったルーム名を送信した場合。
* ResultCodeDisconnect
    * 名前変更
        * FORCE_CLOSE_SYSTEM -> FORCE_CLOSE_SYSTEM_ERROR
        * FORCE_CLOSE_CONTENT ->
          FORCE_CLOSE_BASE_CONNECTION：サーバーでBaseConnectionのclose()呼び出し
          FORCE_CLOSE_BASE_USER：サーバーでBaseUserのcloseConnection()呼び出し
    * 追加。
        * FORCE_CLOSE_INVALID_NODE：GameNodeがinvalid状態に変更された場合。
        * FORCE_CLOSE_USER_TRANSFER_FAIL：ユーザートランスファーが失敗した場合。
        * FORCE_CLOSE_USER_TRANSFER_ERROR：ユーザートランスファー中にシステムエラーが発生した場合。
    * 追加されたがクライアントで受け取れない場合。
      サーバーではクライアントの接続がすでに切れていると予想し、接続を強制終了する際に以下のコードを使用する。
      クライアントの接続が切れているため、以下のコードは受け取れないはずである。
      このコードを受け取った場合はGameAnvil開発チームへ報告してほしい。
        * FORCE_CLOSE_BY_NEW_CONNECTION：同じアカウント情報で新しいログインリクエストが入った場合。
        * <span style="color:#eb6420">現在FORCE_CLOSE_DUPLICATE_LOGINケースにこのコードが渡される問題がある。
          次回リリース時に修正される予定です。</span>
        * FORCE_CLOSE_DISCONNECT_ALARM：クライアントがサーバーの状態チェックに応答しない場合。
        * FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL：クライアントがサーバーの状態チェックに応答しない場合。
        * FORCE_CLOSE_GHOST_USER：ゴーストユーザーの場合。


-----



### 1.0.0-EA3 (2020.07.27)
#### Change
* ConnectionAgentにIsReconnecting()追加。

#### FIX
* UserAgentがログイン後にdisconnectされた際、isLogin()がtrueを返す問題を修正
* UserAgentを複数使用する際、リクエストに対するレスポンスを受け取れずにdisconnectされた場合、リクエストに一緒に渡したcallbackが削除されずに残り、次のリクエスト時にcallbackが2回呼び出される問題を修正

-----



### 1.0.0-EA2 (2020.07.07)
#### Change

* 名前変更：Gameflex -> GameAnvil
* RemoveAllCallback ()追加。

-----




### 1.0.0-EA (2020.06.29)
#### Change
* 名前変更：Tardis -> Gameflex
    * SessionAgent -> ConnectionAgent
* Type変更
    * UserIdのtypeをstringからnumberに変更
    * SubIdのtypeをstringからnumberに変更
    * RoomIdのtypeをstringからnumberに変更
* CreateUserAgent(), GetUserAgent()
    * パラメータ `SubId: string` -> `SubId: number` に変更
    * SubId > 0 でなければならない。
* LoginedUserInfo
    * UserId項目追加。
    * Payload項目追加。
* LoginInfo
    * userId 項目追加。
    * userType項目追加。
    * roomName項目追加。
* UserAgent
    * パラメータ `roomId: string` -> `roomId: number` に変更。
    * MultiSendToSessionActor()->MultiSendToGatewaySession()
    * SendToSessionActor() -> SendToGatewaySession()
    * SendPbToSessionActor() -> SendPbToGatewaySession()
    * RequestToSessionActor() -> RequestToGatewaySession()
    * MultiRequestToSessionActor() -> MultiRequestToGatewaySession()
    * RequestToSessionActor() -> RequestToGatewaySession()
    * RequestPbToSessionActor() -> RequestPbToGatewaySession()
    * RequestPbAsyncToSesstionActor() -> RequestPbAsyncToGatewaySession()
    * RequestAsyncToSessionActor() -> RequestAsyncToGatewaySession()
    * CreateRoom() - パラメータ `roomName: string` 追加。
    * onCreateRoom() - パラメータ `roomName: string` 追加。
    * onJoinRoom() - パラメータ `roomName: string` 追加。
    * onMatchRoom()
        * パラメータ `roomName: string` 追加。
        * パラメータ `created: boolean` 追加。
    * onNamedRoom()
        * パラメータ `roomName: string` 追加。
        * パラメータ `created: boolean` 追加。

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

* onDisconnect()コールバックですぐにconnect()を呼び出した場合にエラーが発生する問題を修正

-----



### 0.12.1 (2020.04.06)
#### Change

* JSDoc適用
* UserAgent.isJoinRoom追加。
* IUserListener.OnMatchPartyCancel()からPayload削除。
* IUserListener.OnNamedRoom()でroomName-> RoomIdに名前変更
* Payload.createDefault()除去。 CreateEmpty()に置換
* SessionAgent.addOnDisconnect()
    * AddOnDisconnect(callback: (agent: SessionAgent, message: string) => void): void; =>
      AddOnDisconnect(callback: (session: SessionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload) => void): void;

-----



### 0.12.0 (2020.02.14)
#### Change

* Packet
    * GetDescId(): number;削除
    * GetMsgIndex(): number;削除
    * GetTimeOut(): number;削除
    * SetTimeOut(timeOut: number): void;削除

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
