## Game > GameAnvil > リリースノート > Unity Connector

### 2.1.0 (2025.06.30)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector.unitypackage)
#### GameAnvil 2.1.0以上
#### New
###### GameAnvil 2.1.0 Connector
* GameAnvil 2.1.0サーバーのリリースに合わせて、Connectorも2.1.0バージョンをリリースします。
  * 2.0.0と比較して機能上の大きな変更点はなく、一部のバグ修正、ResultCodeの名称変更、誤字脱字や不適切な説明などの修正があります。

#### Change
* GameAnvil 2.1サーバーに合わせてエンジンプロトコルをアップデート
  * GameAnvil 2.1以前のバージョンのサーバーは今後サポートされません
* GameAnvilConnectorにPacketサポート機能を追加
  * Request()のパラメータとしてPacketを受け取れる機能を追加
  * コールバックパラメータとしてPacketを受け取れるSetPacketCallback()を追加
    * ユーザー定義パケットをサポート
* Payloadにユーザー定義パケットのサポート機能を追加
* GameAnvilConnectorがErrorResultの代わりにResultを返すように修正
  * 非同期呼び出しの結果を返すために使用していたクラスErrorResultの名前をResultに変更
* 一部のResultCodeを変更

    | 変更前 | 変更後 |
    | ---- | ---- |
    | LOGIN\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | LOGIN_FAIL_INVALID_SERVICE_NAME<br>失敗。不正なサービス名 |
    | LOGIN\_FAIL\_INVALID\_USERTYPE<br>失敗。不正なユーザータイプ | LOGIN_FAIL_INVALID_USER_TYPE<br>失敗。不正なユーザータイプ |
    | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>失敗。不正なサービス名 |
    | ALL\_CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | ALL\_CHANNEL\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>失敗。不正なサービス名 |
    | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。不正なサービスID | CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>失敗。不正なサービス名 |
    | ALL\_CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_ID<br>失敗。チャネル情報が見つかりません | ALL\_CHANNEL\_COUNT\_INFO\_FAIL\_INVALID\_SERVICE\_NAME<br>失敗。不正なサービス名 |
    | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_FORM\_NULL<br>失敗。マッチング申込書がnullの場合 | MATCH\_ROOM\_FAIL\_MATCH\_FORM\_NULL<br>失敗。マッチング申込書がnullの場合 |
    | MATCH\_ROOM\_FAIL\_BASE\_ROOM\_MATCH\_INFO\_NULL<br> 失敗。マッチング情報がnullの場合 | MATCH\_ROOM\_FAIL\_MATCH\_INFO\_NULL<br> 失敗。マッチング情報がnullの場合 |
    | FORCE\_CLOSE\_BASE\_CONNECTION<br>サーバーでBaseConnectionのclose()を呼び出し | FORCE\_CLOSE\_CONNECTION<br>サーバーでIConnectionのclose()を呼び出し |
    | FORCE\_CLOSE\_BASE\_USER<br>サーバーでBaseUserのcloseConnection()を呼び出し | FORCE\_CLOSE\_USER<br>サーバーでIUserのcloseConnection()を呼び出し |

#### Fix
* サーバーで強制終了された場合、onDisconnectコールバックが呼び出された後にUserの状態が変更されていたのを、Userの状態が変更された後にonDisconnectコールバックが呼び出されるように修正
* サーバーとHammerのプロトコルバッファをそれぞれ異なる環境で生成する際、生成環境によって互換性がなくなる可能性があった問題を修正
* Request時に内部で使用されるseq値が最大値を超えた場合、レスポンスを受信できない可能性があった問題を修正

---


### 2.0.0 (2024.12.4)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-2.0.0.unitypackage)
#### GameAnvil 2.0.0以上
#### <span style="color: #e11d21">New</span>
###### GameAnvil 2.0 Connector
* async awaitを使用した新規コネクタがリリースされました。
* 今後、完了時に処理する関数を登録せず、戻り値で完了を確認します。
* GameAnvil接続 + 認証コードは次のとおりです。

```c#
var (err, res) = await connector.ConnectAndAuthentication(
                                             /*host*/"127.0.0.1", 
                                             /*port*/18200, 
                                             "device_id", 
                                             "account_id", 
                                             "password");
```
 
#### <span style="color: #e11d21">Remove</span>
###### ConnectionAgent削除
* ConnectionAgentはGameAnvilConnectorと区分が曖昧でした。
* 今後、GameAnvilConnectorで既存のConnectionAgentの動作を含みます。
* この変更に合わせて他のクラスの名前も変更されました。
    * UserAgentはGameAnvilUserに名前が変更されました。
    * ProtocolManagerはGameAnvilProtocolManagerに名前が変更されました。
    * Unityの利便性のために提供されていたコンポーネントの名前もGameAnvilManagerに変更されました。
###### Connector.Update削除
* 今後、Connector内部で自動的に動作します

###### リクエストのレスポンスを受け取る目的のデリゲート削除
* 今後、リクエストのレスポンスは戻り値として受け取ります。
* サーバーからリクエストなしに受信する種類のメッセージは、従来通りデリゲートを使用します。
    * SetMessageCallbackメソッド

#### <span style="color: #e11d21">Change</span>

###### ProtoBuffer 4.28.3使用
* 内部プロトバッファの依存関係が4.28.3に変更されました

###### UserAgentのインスタンス直接管理
* 以下の例のように、UserAgentのインスタンスを直接生成して使用するように変更されました。
* UserAgent.Dispose時に自動的にログアウトしないため注意してください。UserAgent.Disposeは内部で管理するオブジェクトのみを解放します。
```csharp
using var myUser = new GameAnvilUser(connector, "ServiceName", subId);
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

###### 煩雑なオーバーロード整理
* 過剰に提供されていたオーバーロードを整理し、機能一つにつき最小限のAPIに統合されました。

    ``` c#
    // 既存API例
    public void Login(string userType)
    public void Login(string userType, Interface.DelUserOnLogin onLogin)
    public void Login(string userType, Interface.DelUserOnLogin onLogin, Interface.DelUserOnError onError)
    public void Login(string userType, string channelId, Payload payload)
    public void Login(string userType, string channelId, Payload payload, Interface.DelUserOnLogin onLogin)
    public void Login(string userType, string channelId, Payload payload, Interface.DelUserOnLogin onLogin, Interface.DelUserOnError onError)
    
    // 変更されたAPI例
    public async Task<ErrorResult<ResultCodeLogin, LoginResult>> Login(string userType, string channelId, Payload? requestPayload = null)
    ```

#### <span style="color: #e11d21">Fix</span>
* インターネットが高速な環境で時々正常に接続されない問題が修正されました。

### 1.4.0 (2023.12.13)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.4.0.unitypackage)

#### New

Payloadで圧縮パケットをサポートします。protobuf 3.24.1にアップデートされ、Protocol登録時にindexを指定しなくてもよいように改善されました。

#### Change

ログイン時に誤ったChannelIdを入力した場合、SystemErrorレスポンスの代わりにLogin失敗レスポンスを返すように修正されました。

#### Fix

CONNECT_ALREADY_REQUEST状態でDisconnectができない問題が修正されました。

### 1.3.0 (2022.12.27)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.3.0.unitypackage)

#### GameAnvil 1.3.0以上

#### New

高速接続、ログレベル変更、同期機能などが新たに追加されました。GameAnvilコネクタコンポーネントを通じて新しい機能を利用できます。

###### 高速接続

高速接続機能は、GameAnvilエンジンベースのサーバーへの接続、認証、ログインの3つの過程を一度のメソッド呼び出しで行えるようにします。

###### ログレベル変更機能

専用コンポーネントのインスペクターウィンドウを通じてログレベルをinfo、debug、warn、errorなどに設定し、提供されるログの頻度を調整できます。

###### 同期コンポーネント提供

コンポーネントの提供により、Unityエンジンとの連携の利便性が大幅に強化されました。サーバーの複雑なロジック実装なしでも、リモートクライアント間でゲームオブジェクトの複数の要素が自動的に同期されます。

* GameObject生成、破壊の自動同期
* Animation同期
    * Animationパラメータ同期により、ゲームオブジェクトのAnimationを同期できます。
* Transform同期
    * Position、Transform、ScaleなどゲームオブジェクトのTransform要素を同期します。
    * コードによる変形以外に、どのような経路による変形もすべて検知します。
* RigidBody及びRigidBody2D同期
    * ローカルクライアントで計算されたリジッドボディの状態をリモートクライアントへ送信して同期します。
    * リジッドボディの状態に変化がない場合にはパケットを送信しないように最適化されています。
* カスタム値同期機能
    * ルーム単位でkey-valueペアとしてカスタム値を設定して使用できます。
    * CAS方式の値設定をサポートするため、タイミングの問題を容易に解決できます。

#### Fix

* マッチング成功後にマッチングリクエストを再度送信した際、ルーム入室可否が誤って記録される問題を修正
* packetTimeoutオプションを0に設定時、クライアントが自動的に接続終了しないように修正
* update()メソッド呼び出し時にガベージが発生する問題を解決
* エラーに対するリスナーを登録していない状態でエラーが発生した場合、例外が発生する現象を修正

#### Change

* API変更：名前変更および引数としてErrorCodeを受け取るように修正

| 変更前                   | 変更後                            |
|--------------------------|-----------------------------------|
| OnTimeout(msgId)         | OnError(msgId, ErrorCode)         |
| OnCustomTimeout(command) | OnCustomError(command, ErrorCode) | 

* ErrorCode追加

| 名称                  | 説明                |
|-----------------------|---------------------|
ErrorCode.PARSE_ERROR | プロトコルパースに失敗したことを示す |

* ユーザー定義プロトコルコールバックにResultCode追加

| 名称                    | 説明              |
|-------------------------|-------------------|
ResultCode.PARSE_ERROR  | パケットパースに失敗したことを示す |
| ResultCode.SYSTEM_ERROR | サーバーシステムエラー       |
| ResultCode.TIMEOUT      | タイムアウト            |
| ResultCode.SUCCESS      | 成功              |

* Exception追加

| 名称                 | 説明                 |
|----------------------|----------------------|
| ParseInvalidProtocol | プロトコルの解析に失敗した場合に発生 |

* ロギングシステムの改善
    * Ping-Pongログの出力をオプションで設定できる機能を追加
    * ログでメッセージ情報を出力する際に、メッセージIDの代わりに名前を出力するように修正
    * 出力するログレベルを調整できる機能を追加


* Request時にパケットを即時送信できるオプション requestDirect を追加
* setArgumentDelegateOrListenersOnError オプション追加

---

### 1.2.3 (2022.01.28)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.3.unitypackage)

#### GameAnvil 1.2.0以上

#### Fix

* サーバーから送信された圧縮パケットを処理できずエラーが発生する問題を修正

------

### 1.2.2 (2021.11.30)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.2.unitypackage)

#### GameAnvil 1.2.0以上

#### Fix

* ルームに入室した状態でMatchRoomを呼び出して失敗した場合、IsJoinedRoom()がfalseに変わる問題を修正

------

### 1.2.1 (2021.08.10)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.1.unitypackage)

#### GameAnvil 1.2.0以上

#### Fix

* SocketException発生時にOnDisconnectが2回呼び出されるバグを修正

------

### 1.2.0 (2021.07.13)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.2.0.unitypackage)

#### GameAnvil 1.2.0以上

#### Change

* Send()の戻り値の型をvoidに変更
* Config.defaultRequestTimeoutのデフォルト値を3秒に変更
* Request()または他のAPI呼び出し時にCallbackを引数として一緒に渡すと、引数として渡したコールバックにのみレスポンスが届くように変更
    * Config.useArgumentDelegateOnly 追加 (デフォルト値 true)
    * Config.useArgumentDelegateOnlyがfalseの場合、Request()または他のAPI呼び出し時にCallbackを引数として一緒に渡しても、引数で渡したコールバックと事前に登録したリスナーが同時に呼び出されます。（旧バージョンの動作方式）
* Exceptions
    * NotConnected 追加
    * NotAuthenticated 追加
    * NotLoggedIn 追加
    * MatchUserInProgress 追加
    * MatchPartyInProgress 追加
    * MatchUserNotInProgress 追加
    * MatchPartyNotInProgress 追加
* ConnectionAgent
    * サーバーに接続していない状態でリクエストを送信した場合、NotConnected例外が発生
    * Authenticatedされていない状態でリクエストを送信した場合、NotAuthenticated例外が発生
    * PauseClientStateCheck()、ResumeClientStateCheck()は例外。
    * LoginedUserInfo.payload削除
    * GetAllChannelInfo()追加
    * GetChannelCountInfo()追加
    * GetAllChannelCountInfo()追加
* UserAgent
    * Loginしていない状態でリクエストを送信した場合、NotLoggedIn例外が発生
    * LoginInfo.Message 削除
    * GetChannelInfo()追加
    * GetAllChannelInfo()追加
    * GetChannelCountInfo()追加
    * GetAllChannelCountInfo()追加
    * CreateRoom()にmatchingGroupパラメータ追加
    * JoinRoom()にmatchingUserCategoryパラメータ追加
    * MatchRoom()にmatchingGroup、matchingUserCategoryパラメータ追加。
    * MatchUserStart()にmatchingGroupパラメータを追加
    * MatchPartyStart()にmatchingGroupパラメータを追加
    * IsUserMatchInProgress()、IsPartyMatchInProgress()、IsMatchInProgress()追加
    * UserMatch中にUserMatchCancelを除くAPI呼び出し時、UserMatchInProgress例外が発生
        * UserMatch中でない時にUserMatchCancel呼び出し時、UserMatchNotInProgress例外が発生
        * PartyMatch中にPartyMatchCancelを除くAPI呼び出し時、UserMatchInProgress例外が発生
        * PartyMatch中でない時にUserMatchCancel呼び出し時、PartyMatchNotInProgress例外が発生
* ResultCode
    * ResultCodeConnect
        * CONNECT_FAIL_INVALID_IP 追加
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
        * MATCH_ROOM_FAIL_CREATE_FAILED_ROOM 追加
        * MATCH_ROOM_FAIL_INVALID_ROOM_ID 追加
        * MATCH_ROOM_FAIL_INVALID_NODE_ID 追加
        * MATCH_ROOM_FAIL_INVALID_USER_ID 追加
        * MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND 追加
        * MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY追加
        * MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY追加
        * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL 追加
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

### 1.1.6 (2023.01.20)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.6.unitypackage)

#### GameAnvil 1.1.0以上

#### Fix

* useIpv6オプションが有効な状態でconnect()呼び出し時にブロックされる可能性のある問題を修正

------

### 1.1.5 (2022.01.28)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.5.unitypackage)

#### GameAnvil 1.1.0以上

#### Fix

* サーバーから送信された圧縮パケットを処理できずエラーが発生する問題を修正

------

### 1.1.4 (2021.11.30)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.4.unitypackage)

#### GameAnvil 1.1.0以上

#### Fix

* ルームに入室した状態でMatchRoomを呼び出して失敗した場合、IsJoinedRoom()がfalseに変わる問題を修正

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

* GameAnvil ServerのClientStateCheck機能を一時停止させるConnectionAgent.PauseClientStateCheck()を追加。アプリがバックグラウンドに移行するなど、メッセージループが動作しなくなる場合に呼び出す。
* 一時停止していたClientStateCheck機能を再開させるConnectionAgent.ResumeClientStateCheck()を追加。アプリがバックグラウンドから復帰するなど、メッセージループが再び動作するようになった場合に呼び出す。
* Singleserver.SetOnPauseClientStateCheck() 追加。
* Singleserver.SetOnResumeClientStateCheck() 追加。

#### Fix

* OnDisconnectでforceがfalseの時、ResultCodeDisconnectの値が0で返ってくるバグを修正

------

### 1.1.1 (2021.04.07)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.1.unitypackage)

#### GameAnvil 1.1.0以上

#### Change

* listenerの個別登録有無を確認できるContainsListenerオーバーロードを追加。
* メッセージごとの全てのlistenerを削除できるRemoveAllListenersForMsg、RemoveAllListenersForMsgIdを追加
* ContainsUserListener、ContainsUserNotificationListener、ContainsUserErrorListener 追加。
* ContainsConnectionListener、ContainsConnectionNotificationListener、ContainsConnectionErrorListener、ContainsConnectionErrorListener 追加。

#### Fix

* 強制終了、ログアウト、ログイン失敗などの状況でUserAgentの一部情報が初期化されないバグを修正

------

### 1.1.0 (2020.12.18)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.1.0.unitypackage)

#### GameAnvil 1.1.0以上

#### Change

* .Net 4.5以上サポートへの完全移行
    * 使用ライブラリを全て.Net 4.5用最新バージョンに交換
* ConnectionAgent
    * IsConnected() : 接続確認API追加。
    * IsAuthenticated() : 認証済みか確認API追加。
* SingleServerのGameAnvil.Action -> GameAnvil.Handlerに名前変更。

------

### 1.0.0 (2020.08.31)

#### [ダウンロード](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-1.0.0.unitypackage)

#### GameAnvil 1.0.0以上

#### Change

* MoveService削除
* Reconnect、Retry機能削除
* PacketHelperのGetMessageにTypeをパラメータとして受け取るAPIを追加。
* 各ResultCodeに固有の数値を適用

* ResultCode追加および名前変更

    * <span style="color:#eb6420">現在FORCE_CLOSE_DUPLICATE_LOGINケースにFORCE_CLOSE_BY_NEW_CONNECTIONが渡される問題があります。次回リリース時に修正される予定です。</span>

#### Fix

* Disconnect時にもUserAgentのisLoginがtrueを返す問題を修正

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
        * <span style="color:#eb6420">現在FORCE_CLOSE_DUPLICATE_LOGINケースにこのコードが渡される問題があります。
          今後修正される予定。</span>
        * FORCE_CLOSE_DISCONNECT_ALARM：クライアントがサーバーの状態チェックに応答しない場合。
        * FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL：クライアントがサーバーの状態チェックに応答しない場合。
        * FORCE_CLOSE_GHOST_USER：ゴーストユーザーの場合。

-----

### 1.0.0-EA2 (2020.07.07)

#### C-Sharp

##### Change

* 名前変更：Gameflex -> GameAnvil
* RemoveAllListeners()
    * boolパラメータ追加。
    * falseの場合、userListener、connectonListenerは削除しない。
    * defalult = true,
* RemoveAllUserListeners() 追加。
* RemoveAllConnectionListeners() 追加。

-----

### 1.0.0-EA (2020.06.29)

#### C-Sharp

##### Change

* 名前変更：Tardis -> Gameflex
    * TardisSocket -> Socket
    * SessionAgent -> ConnectionAgent
* Type変更
    * UserIdのtypeをstringからintに変更
    * SubIdのtypeをstringからintに変更
    * RoomIdのtypeをstringからintに変更
* CreateUserAgent(), GetUserAgent()
    * パラメータ `string SubId` -> `int SubId` に変更
    * SubId > 0 でなければならない。
* LoginedUserInfo
    * UserId項目追加。
    * Payload項目追加。
* LoginInfo
    * userId 項目追加。
    * userType項目追加。
    * roomName項目追加。
* UserAgent
    * パラメータ `string roomId` -> `int roomId` に変更。
    * RequestToSessionActor() ->RequestToGatewaySession()
    * SendToSessionActor() -> SendToGatewaySession()
    * RequestToSessionActor() -> RequestToGatewaySession()
    * CreateRoom() - パラメータ `string roomName` 追加。
    * onCreateRoom() - パラメータ `string roomName` 追加。
    * onJoinRoom() - パラメータ `string roomName` 追加。
    * onMatchRoom()
        * パラメータ `string roomName` 追加。
        * パラメータ `bool created` 追加。
    * onNamedRoom()
        * パラメータ `string roomName` 追加。
        * パラメータ `bool created` 追加。

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

* ProtocolManger.unregister()使用時にexceptionが発生するバグを修正

-----

### 0.12.1 (2020.04.06)

#### C-Sharp

##### Change

* XMLドキュメントコメント適用
* Agent.LvNetLogger、Logger削除。
* Agent.ContainsListener(int customMsgId) -> AgentHelperへ移動
* SessionAgent.AddCustomMsgListener、RemoveCustomMsgListener -> AgentHelper.RemoveListener() に代替
* UserAgent.AddCustomMsgListener、RemoveCustomMsgListener -> AgentHelper.AddListener、RemoveListenerに代替
* Agent.FlushQueue 削除
* Connector.FlushNetworkQueue 削除
* Packet.GetMsgId、GetRetryCount、SetRetryCount、setUncompressSize、GetServiceId、GetSubId -> 削除
* ProtocolManager.RegisterProtocol()でprotocolIdの最大値チェック (最大510)
* 誤ったProtocolId使用時にInvalidProtocolId Exceptionが発生するように修正
* OnNamedRoomでRoomName -> RoomId 名前変更

##### Fix

* MatchUserDoneの時にisJoinRoomが設定されなかったバグを修正

-----

### 0.12.0 (2020.02.14)

#### C-Sharp

##### Change

* パラメータ名変更 : msgIndex -> customMsgId
    * Agent
        * public bool ContainsListener(int customMsgId)
    * SessionAgent
        * public void AddCustomMsgListener(int custonMsgId, Action\<SessionAgent, Packet> listener)
        * public void RemoveCustomMsgListener(int customMsgId, Action\<SessionAgent, Packet> listener)
    * UserAgent
        * public void AddCustomMsgListener(int custonMsgId, Action\<UserAgent, Packet> listener)
        * public void RemoveCustomMsgListener(int customMsgId, Action\<UserAgent, Packet> listener)
    * Payload
        * public Packet getPacket(int customMsgId)

* Packet
    * public Packet(int descId, int msgIndex, byte[] buffer) 削除
    * public Packet(int msgIndex, byte[] buffer) 削除
    * public string GetFileDescriptorName() 削除
    * public void SetDescId(int descId) 削除
    * public int GetDescId() 削除
    * public int GetMsgIndex() 削除
    * public Packet setTimeout(int timeout) 削除
    * public int GetTimeout() 削除

##### New

* ResultCodeMatchRoom
    * MATCH\_ROOM\_FAIL\_UNKNOWN\_ERROR = 6,
    * MATCH\_ROOM\_FAIL\_MATCHED\_ROOM\_DOES\_NOT\_EXIST = 7
* Packet
    * public static Packet CreateWithCustomMsg(int customMsgId, byte[] buffer)
    * public int GetMsgId()
