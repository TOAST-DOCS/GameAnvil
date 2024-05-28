## Game > GameAnvil > Unity基礎開発ガイド > UserAgent

## UserAgent

UserAgentはGameAnvilサーバーのGameNodeと関連した作業を担当します。ログイン(Login())、ログアウト(Logout())、ルーム管理などの基本機能を提供し、直接定義したプロトコルに基づいて、クライアントは自分のユーザーオブジェクトを通じて他のオブジェクトとメッセージをやり取りし、様々なコンテンツを実装できます。

GameAnvilConnectorは基本的にQuickConnectの過程で作成された1つのUserAgentを提供します。GameAnvilConnector.getUserAgent()を通じて取得できます。

### ログイン

ログインはクライアントがサーバーに接続した後、GameNodeに自分のユーザーオブジェクトを作成するプロセスと定義できます。

ログインはQuickConnectで一度に処理されるので説明を省略します。詳細は[Unity深層開発ガイド > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md)を参照してください。

### ログアウト

GameAnvilConnectorでは、ログアウトすると自動的に接続終了まで処理されます。ログアウトの詳細については、[Unity深層開発ガイド > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md)を参照してください。

```c#
/// <summary>
/// 現在のサービスからログアウト 
/// </summary>
/// <param name="onLogout">ログアウトデリゲート</param>
userAgent.Logout((UserAgent user, Defines.ResultCodeLogout result, bool force, Payload payload) => {
    /// <param name="userAgent">Logout()をリクエストしたユーザーエージェント</param>
    /// <param name="result">Logout()結果</param>
    /// <param name="force">サーバーによる強制かどうか</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    if(result == Defines.ResultCodeLogout.LOGOUT_SUCCESS){
        // 成功
    } else {
        // 失敗
    }
});
```

### ルームの作成、入室、退室

2人以上のユーザーはルームを通じて同期されたメッセージフローを作ることができます。つまり、ユーザーのリクエストはルーム内で全て順番が保証されます。もちろん、1人のユーザーのためのルームの作成もコンテンツによって意味を持つことができます。ルームをどのように使うかはあくまでエンジンユーザー次第です。

CreateRoom()を呼び出してルームを作成し、そのルームに入室します。

```c#
/// <summary>
/// 任意のルームを作成し、そのルームに入室
/// </summary>
/// <param name="roomName">作成するルームの名前</param>
/// <param name="roomType">作成するルームのタイプ</param>
/// <param name="matchingGroup">マッチングするルームのマッチンググループ</param>
/// <param name="payload">サーバーに渡す追加情報</param>
/// <param name="onCreateRoom">結果を受け取るデリゲート</param>
userAgent.CreateRoom(roomName, roomType, matchingGroup, payload, (UserAgent user, Defines.ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) => {
    /// <param name="userAgent">CreateRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result"> CreateRoom()リクエスト結果</param>
    /// <param name="roomName">作成されたルームの名前</param>
    /// <param name="roomId">作成されたルームのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
	if(result == Defines.ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
        // 成功
    } else {
        // 失敗
    }
});
```
<br>

JoinRoom()を呼び出して既に作成されたルームに入ります。

``` c#
/// <summary>
/// 指定したIDのルームに入室<para></para>
/// 指定したIDのルームがない場合は失敗
/// </summary>
/// <param name="roomType">入室するルームのタイプ</param>
/// <param name="roomId">入室するルームのID</param>
/// <param name="matchingUserCategory">ルームマッチングを使用中のルームに入室する場合に使用</param>
/// <param name="payload">サーバーに渡す追加情報</param>
/// <param name="onJoinRoom">結果を受け取るデリゲート</param>
userAgent.JoinRoom(roomType, roomId, matchingUserCategory, payload, (UserAgent user, Defines.ResultCodeJoinRoom result, int id, string roomName, Payload payload) => {
    /// <param name="result">JoinRoom()リクエスト結果</param>
    /// <param name="roomId">入室したルームのID</param>
    /// <param name="roomName">入室したルームの名前</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
	if(result == Defines.ResultCodeJoinRoom.JOIN_ROOM_SUCCESS){
        // 成功
    } else {
        // 失敗
    }
});
```
<br>

LeaveRoom()を呼び出して入室したルームから退室できます。

``` c#
/// <summary>
/// ユーザーエージェントが所属しているルームから退室
/// </summary>
/// <param name="payload">サーバーに渡す追加情報</param>
/// <param name="onLeaveRoom">結果を受け取るデリゲート</param>
userAgent.LeaveRoom(payload, (UserAgent user, Defines.ResultCodeLeaveRoom result, bool force, int roomId, Payload payload) => {
    /// <param name="userAgent">LeaveRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result">LeaveRoom()リクエスト結果</param>
    /// <param name="force">強制退室するかどうか</param>
    /// <param name="roomId">退室したルームのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
	if(result == Defines.ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS){
        // 成功
    } else {
        // 失敗
    }
});
```
<br>

NamedRoom()を呼び出して指定した名前のルームに入室できます。指定した名前のルームがない場合は、ルームを作成した後、そのルームに入室します。

```c#
/// <summary>
/// 指定した名前のルームに入室<para></para>
/// 指定した名前のルームがない場合、指定した名前のルームを作成し、そのルームに入室
/// </summary>
/// <param name="roomType">入室するルームのタイプ</param>
/// <param name="roomName">入室するルームの名前</param>
/// <param name="isParty">パーティーかどうか</param>
/// <param name="payload">サーバーに渡す追加情報</param>
/// <param name="onNamedRoom">結果を受け取るデリゲート</param>
userAgent.NamedRoom(roomType, roomName, isParty, payload, (UserAgent user, Defines.ResultCodeNamedRoom result, int roomId, string roomName, bool created, Payload payload) => {
    /// <param name="userAgent">NameRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result">NameRoom()リクエスト結果</param>
    /// <param name="roomName">ルーム名</param>
    /// <param name="roomId">入室したルームのID</param>
    /// <param name="created">入室したルームの新設の有無</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    if(result == Defines.ResultCodeNamedRoom.NAMED_ROOM_SUCCESS){
        // 成功
    } else {
        // 失敗
    }
});
```

### マッチメイキング

GameAnvilは大きく2つのマッチメイキングを提供しています。1つはルーム単位のマッチングを行うルームマッチメイキング、もう1つはユーザー単位のマッチングを行うユーザーマッチメイキングです。

#### ルームマッチメイキング

ルームマッチメイキングは、条件に合ったルームにユーザーを入室させる方法です。ルームマッチメイキングのリクエスト時、条件に合うルームがあればそのルームに直接入室させて、条件に合うルームがなければ新しいルームを作成して入室させます。

MatchRoom()を呼び出してルームマッチメイキングをリクエストできます。

```c#
/// <summary>
/// ルームマッチングリクエスト<para></para>
/// ルームがない場合、任意のルームを作成してそのルームに入室するか、マッチング失敗処理
/// </summary>
/// <param name="roomType">入室するルームのタイプ</param>
/// <param name="matchingGroup">マッチングするルームのマッチンググループ</param>
/// <param name="matchingUserCategory">マッチング時に使用するマッチングユーザーカテゴリー</param>
/// <param name="isCreateRoomIfNotJoinRoom">ルームの作成を許可</param>
/// <param name="payload">サーバーに渡す追加情報</param>
/// <param name="onMatchRoom">結果を受け取るデリゲート</param>
userAgent.MatchRoom(roomType, matchingGroup, matchingUserCategory, isCreateRoomIfNotJoinRoom, isMoveRoomIfJoinedRoom, payload, leaveRoomPayload, (UserAgent user, Defines.ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload) => {
    /// <param name="userAgent">MatchRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchRoom()リクエスト結果</param>
    /// <param name="resultCode">ユーザー結果コード</param>
    /// <param name="roomId">マッチしたルームのID</param>
    /// <param name="roomName">マッチしたルームの名前</param>
    /// <param name="created">マッチしたルームの新設有無</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    if(result == Defines.ResultCodeMatchRoom.MATCH_ROOM_SUCCESS){
        // マッチメイキングキャンセル成功
    } else {
        // マッチメイキングキャンセル失敗
    }
});
```

#### ユーザーマッチメイキング

ユーザーマッチメイキングは、ユーザープールを作成し、その中で条件に合うユーザーを探し、新しく作成したルームに入室させる方式です。ユーザープールに条件に合うユーザーの数が足りない場合、マッチメイキングが完了するまで時間がかかることがあり、時間内にマッチメイキングが完了しない場合、タイムアウトしてマッチングがキャンセルされることがあります。

MatchUserStart()を呼び出すことで、ユーザーのマッチメイキングを開始できます。すでにルームに入室している場合など、サーバーの条件によってリクエストが失敗する場合があります。

```c#
/// <summary>
/// ユーザーマッチングをリクエスト<para></para>
/// 既にルームに入室している場合など、サーバーの条件によりリクエストが失敗<para></para>
/// マッチングが成功した場合、OnMatchUserDone()で通知<para></para>
/// </summary>
/// <param name="roomType">マッチをリクエストするルームのタイプ</param>
/// <param name="matchingGroup">ルーム作成時に使用するマッチンググループ</param>
/// <param name="payload">サーバーに渡す追加情報</param>
/// <param name="onMatchUserStart">結果を受け取るデリゲート</param>
userAgent.MatchUserStart(roomType, matchingGroup, payload, (UserAgent user, Defines.ResultCodeMatchUserStart result, Payload payload) => {
    /// <param name="userAgent">MatchUserStart()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchUserStart()リクエスト結果</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
	if(result == Defines.ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS){
        // マッチメイキングリクエスト成功
    } else {
        // マッチメイキングリクエスト失敗
    }
});
```
<br>

マッチングが成功した場合、onMatchUserDoneListenersまたはIUserListener.OnMatchUserDoneを通じて通知を受けることができ、時間内にマッチングが成功しなかった場合、onMatchUserTimeoutListenersまたはIUserListener.OnMatchUserTimeoutを通じて通知を受けることができます。

```c#
/// <summary>
/// ユーザーマッチング結果を受け取るデリゲート
/// </summary>
userAgent.onMatchUserDoneListeners += (UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload) => {
    /// <param name="userAgent">MatchUserStart()またはMatchPartyStart()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchUserStart()またはMatchPartyStart()のリクエスト結果</param>
    /// <param name="created">ルームの新設の有無</param>
    /// <param name="roomId">マッチングしたルームのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
};

/// <summary>
/// ユーザーマッチングタイムアウト通知デリゲート
/// </summary>
userAgent.onMatchUserTimeoutListeners  += (UserAgent userAgent) => {
    /// <param name="userAgent">MatchUserStart()またはMatchPartyStart()をリクエストしたユーザーエージェント</param>
};
```
<br>

MatchUserCancel()を呼び出すことで、ユーザーマッチメイキングをキャンセルすることができます。ユーザーマッチリクエスト中でない場合や、すでにユーザーマッチメイキングが成功している場合、またはTimeoutが発生した場合、キャンセルリクエストは失敗する可能性があります。

```c#
/// <summary>
/// ユーザーマッチングリクエストをキャンセル<para></para>
/// マッチリクエスト中でない場合や、すでにマッチングが成功した場合、またはタイムアウトが発生した場合は、失敗<para></para>
/// </summary>
/// <param name="roomType">マッチをリクエストしたルームのタイプ</param>
/// <param name="onMatchUserCancel">結果を受け取るデリゲート</param>
userAgent.MatchUserCancel(Constants.RoomType, (UserAgent user, Defines.ResultCodeMatchUserCancel result) => {
    /// <param name="userAgent">MatchUserCancel()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchUserCancel()リクエスト結果</param>
	if(result == Defines.ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS){
        // マッチメイキングキャンセル成功
    } else {
        // マッチメイキングキャンセル失敗
    }
});
```

その他、パーティーマッチメイキング、チャンネル、リスナーなどユーザーエージェントの様々な機能の詳細は、[Unity深層開発ガイド > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md)を参照してください。
