## Game > GameAnvil > Unity深層開発ガイド > UserAgent

## UserAgent

UserAgentはGameAnvilサーバーのGameNodeに関する作業を担当します。ログイン(Login())、ログアウト(Logout())、ルーム管理などの基本機能を提供し、直接定義したプロトコルに基づいて、クライアントは自分のユーザーオブジェクトを通じて他のオブジェクトとメッセージをやり取りし、様々なコンテンツを実装できます。
UserAgentを使うためにはConnector.CreateUserAgent()関数を使って新しいUserAgentを作成する必要があります。ServiceNameとSubIdで区分される複数のUserAgentを作成できます。作成されたUserAgentはConnectorで内部的に管理され、Connector.GetUserAgent()関数を使って再使用できます。

```c#
UserAgent userAgent = ConnectHandler.getInstance().GetUserAgent(serviceName, subID);
if (userAgent == null) {
    userAgent = ConnectHandler.getInstance().CreateUserAgent(serviceName, subID);
}
```

GameAnvilサーバーは複数のサービスを同時に運営することができ、1つのUserAgentは1つのサービスにログインして互いに独立して動作することになります。つまり、複数のUserAgentを作成して違うサービスにログインして同時に使用できます。SubIdを変えれば、同じサービスに複数のUserAgentを同時にログインして使うことも可能です。

### ログイン/ログアウト

ログインはクライアントがサーバーに接続した後、GameNodeに自分のユーザーオブジェクトを作成するプロセスと定義できます。ログアウトはログインの反対の概念です。つまり、GameNode上で自分のユーザーオブジェクトを削除するプロセスです。

ログイン時、どのUserTypeでどのチャンネルにログインするかを入力する必要があります。追加情報が必要な場合は、Payloadに含めることができます。

```c#
/// <summary>
/// サービスにログイン
/// </summary>
/// <param name="userType">ユーザーのタイプ </param>
/// <param name="payload">サーバーに伝達する追加情報</param>
/// <param name="channelId">ログインするチャンネルのID</param>
/// <param name="onLogin">結果を受け取るデリゲート</param>
userAgent.Login(userType, channelId, payload, (UserAgent user, Defines.ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => {
    /// <param name="userAgent">Login()をリクエストしたユーザーエージェント</param>
    /// <param name="result">Login()リクエスト結果</param>
    /// <param name="loginInfo">ログイン情報</param>
    if(result == Defines.ResultCodeLogin.LOGIN_SUCCESS){
        // 成功
    } else {
        // 失敗
    }
});

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
...
```

### ルームの作成、入室、退室

[Unity基礎開発ガイド > UserAgent](../unity-basic/unity-basic-04-user-agent.md)のルーム作成、入室、退室の内容と同じです。

### マッチメイキング

GameAnvilは2種類のマッチメイキングを提供しています。1つはルーム単位のマッチングを行うルームマッチメイキング、もう1つはユーザー単位のマッチングを行うユーザーマッチメイキングです。詳細は[Unity基礎開発ガイド > UserAgent](../unity-basic/unity-basic-04-user-agent.md)のマッチメイキングの部分を参照してください。

#### パーティーマッチメイキング

パーティーマッチメイキングは、ユーザーマッチメイキングの特殊な形で、2人以上のユーザーが1つのパーティーとしてユーザープールに登録され、条件に合う他のユーザーを探し、新しく作成したルームに一緒に入室させる方式です。パーティーを組んだユーザーは、常に同じルームに入室します。パーティー以外で一緒にマッチングするユーザーは、サーバーのマッチメーカーの実装により、別のパーティーになることもあれば、個人になることもあります。

パーティーマッチメイキングを行うためには、まずNamedRoom()を呼び出す必要があります。NamedRoom()呼び出し時にisPartyパラメータをtrueで渡すと、そのNamedRoomがパーティーの役割をします。NamedRoomにパーティーユーザーを全て集めた後、パーティーマッチメイキングを始めます。

```c#
/// <summary>
/// 指定した名前のルームに入室<para></para>
/// 指定した名前のルームがない場合、指定した名前のルームを作成し、そのルームに入室
/// </summary>
/// <param name="roomType">入室するルームのタイプ</param>
/// <param name="roomName">入室するルームの名前</param>
/// <param name="isParty">パーティーかどうか</param>
/// <param name="payload">サーバーに伝達する追加情報</param>
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
<br>

MatchPartyStart()を呼び出してパーティーマッチメイキングを開始できます。すでにルームに入室している場合など、サーバーの条件によってリクエストが失敗することがあります。

```c#
/// <summary>
/// パーティーマッチングをリクエスト<para></para>
/// 既にルームに入室している場合など、サーバーの条件によりリクエストが失敗<para></para>
/// マッチングが成功した場合、OnMatchUserDone()を通じて通知
/// </summary>
/// <param name="roomType">マッチングをリクエストするルームのタイプ</param>
/// <param name="matchingGroup">ルーム作成時に使用するマッチンググループ</param>
/// <param name="payload">サーバーに伝達する追加情報</param>
/// <param name="onMatchPartyStart">結果を受け取るデリゲート</param>
userAgent.MatchPartyStart(Constants.RoomType, Constants.ChannelId1, customPayload, (UserAgent user, Defines.ResultCodeMatchPartyStart result, Payload payload) => {
    /// <param name="userAgent">MatchPartyStart()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchPartyStart()リクエスト結果</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    if(result == Defines.ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS){
        // 成功
    } else {
        // 失敗
    }
});
```
<br>

パーティーマッチングが成功した場合、ユーザーマッチメイキングで使ったonMatchUserDoneListenersまたはIUserListener.OnMatchUserDoneを通じて通知を受けることができ、時間内にマッチングが成功しなかった場合、onMatchUserTimeoutListenersまたはIUserListener.OnMatchUserTimeoutを通じて通知を受けることができます。

```c#
/// <summary>
/// ユーザーマッチング結果を受け取るデリゲート
/// </summary>
userAgent.onMatchUserDoneListeners += (UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload) => {
    /// <param name="userAgent">MatchUserStart()またMatchPartyStart()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchUserStart()またはMatchPartyStart()のリクエスト結果</param>
    /// <param name="created">ルームの新設の有無</param>
    /// <param name="roomId">マッチングされたルームのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
};

/// <summary>
/// ユーザーマッチングタイムアウト通知デリゲート
/// </summary>
userAgent.onMatchUserTimeoutListeners  += (UserAgent userAgent) => {
    /// <param name="userAgent">MatchUserStart()またMatchPartyStart()をリクエストしたユーザーエージェント</param>
};
```
<br>

MatchPartyCancel()を呼び出すことで、パーティーマッチメイキングをキャンセルできます。パーティーマッチリクエスト中でない場合や、すでにパーティーマッチメイキングが成功している場合、またはタイムアウトが発生した場合、キャンセルリクエストは失敗する可能性があります。

```c#
/// <summary>
/// パーティーマッチングリクエストをキャンセル<para></para>
/// マッチリクエスト中でない場合や、すでにマッチングが成功した場合、またはタイムアウトが発生した場合は失敗します。
/// </summary>
/// <param name="roomType">マッチングをリクエストしたルームのタイプ</param>
/// <param name="onMatchPartyCancel">結果を受け取るデリゲート</param>
userAgent.MatchPartyCancel(Constants.RoomType, (UserAgent user, Defines.ResultCodeMatchPartyCancel result) => {
    /// <param name="userAgent">MatchPartyCancel()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchPartyCancel()リクエスト結果</param>
    if(result == Defines.ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS){
        // 成功
    } else {
        // 失敗
    }
});
```

#### チャンネル移動

場合によっては、マッチメイキングの結果、チャンネル移動が発生することがあります。チャンネル移動が行われた場合、onMoveChannelListenersまたはIUserListener.OnMoveChannelを通じて通知を受けることができます。

```c#
/// <summary>
/// チャネル移動リクエストの結果、またはサーバーが強制的に実行したチャンネル移動の通知を受けるデリゲート
/// </summary>
userAgent.onMoveChannelListeners  += (UserAgent userAgent, GameAnvil.Defines.ResultCodeMoveChannel result, bool force, string channelId, Payload payload) => {
    /// <param name="userAgent">MoveChannel()したユーザーエージェント</param>
    /// <param name="result">MoveChannel()結果コード</param>
    /// <param name="force">サーバーが強制的にチャンネルを移動したかどうか</param>
    /// <param name="channelId">移動したチャンネルのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
};
```
<br>

MoveChannel()を呼び出してサービス内の他のチャンネルに移動できます。

```c#
/// <summary>
/// 指定したチャンネルに移動
/// </summary>
/// <param name="channelId">移動するチャンネルのID</param>
/// <param name="payload">サーバーに伝達する追加情報</param>
/// <param name="onMoveChannel">結果を受け取るデリゲート</param>
userAgent.MoveChannel(channelId, usePayload ? customPayload : null, (UserAgent user, Defines.ResultCodeMoveChannel result, bool force, string channelID, Payload payload) => {
    /// <param name="userAgent">MoveChannel()したユーザーエージェント</param>
    /// <param name="result">MoveChannel()結果コード</param>
    /// <param name="force">サーバーが強制的にチャンネルを移動したかどうか</param>
    /// <param name="channelId">移動したチャンネルのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
	if(result == ResultCodeMoveChannel.ALL_CHANNEL_INFO_SUCCESS){
		// 全てのチャンネル情報リクエスト成功
	} else {
		// 全てのチャンネル情報リクエスト失敗
	}
});
```

### チャンネル情報

GameAnvilは設定で自由にチャンネル構成を変更できます。このようなチャンネル構成はサーバーとクライアントの間であらかじめ約束して固定された形で使うこともできますが、状況によって柔軟に変更して使うこともできます。UserAgentでは、このように変更されたチャンネル情報を取得したり、チャンネルを移動できるようにいくつかの関数を提供しています。

| 関数 | 説明 |
| --- | --- |
| GetChannelCountInfo() | 特定チャンネルのカウント情報(ユーザーとルーム数)リクエスト | 
| GetChannelInfo() | 特定チャンネルの情報(ユーザー定義)リクエスト |
| GetAllChannelCountInfo() | 特定サービスの全てのチャンネルのカウント情報(ユーザーとルーム数)リクエスト |
| GetAllChannelInfo() | 特定サービスの全てのチャンネルの情報(ユーザー定義)リクエスト |

下記のコードでさらに詳しく説明します。

GetChannelCountInfo()は特定チャンネルのカウント情報(ユーザーとルーム数)をリクエストして取得できます。

```c#
/// <summary>
/// 接続中のチャンネルのユーザーとルームの数をリクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="onChannelCountInfo">結果を受け取るデリゲート</param>
userAgent.GetChannelCountInfo((ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="userAgent">GetChannelCountInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから受け取ったチャンネルのユーザー数とルーム数情報</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// チャンネルカウント情報リクエスト成功
	} else {
		// チャンネルカウント情報リクエスト失敗
	}
});

/// <summary>
/// 特定チャンネルのユーザーとルームの数をリクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="serviceName">チャンネル情報をリクエストするサービスの名前</param>
/// <param name="channelId">チャンネル情報をリクエストするチャンネルのID</param>
/// <param name="onChannelCountInfo">結果を受け取るデリゲート</param>
userAgent.GetChannelCountInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="userAgent">GetChannelCountInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから受け取ったチャンネルのユーザー数とルーム数情報</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// チャンネルカウント情報リクエスト成功
	} else {
		// チャンネルカウント情報リクエスト失敗
	}
});
```
<br>

GetChannelInfo()はチャンネルの情報(ユーザー定義)をリクエストして取得できます。

```c#
/// <summary>
/// 接続中のチャンネル情報をリクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="onChannelInfo">結果を受け取るデリゲート</param>
userAgent.GetChannelInfo((ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="userAgent">GetChannelInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetChannelInfo()リクエスト結果</param>
    /// <param name="channelInfo">サーバーから受け取ったチャンネル情報</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// チャンネル情報リクエスト成功
	} else {
		// チャンネル情報リクエスト失敗
	}
});

/// <summary>
/// 特定チャンネルの情報をリクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="serviceName">チャンネル情報をリクエストするサービスの名前</param>
/// <param name="channelId">チャンネル情報をリクエストするチャンネルのID</param>
/// <param name="onChannelInfo">結果を受け取るデリゲート</param>
userAgent.GetChannelInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="userAgent">GetChannelInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetChannelInfo()リクエスト結果</param>
    /// <param name="channelInfo">サーバーから受け取ったチャンネル情報</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// チャンネル情報リクエスト成功
	} else {
		// チャンネル情報リクエスト失敗
	}
});
```
<br>

GetAllChannelCountInfo()はサービスの全てのチャンネルのカウント情報(ユーザーとルーム数)をリクエストして取得できます。パラメータでサービス名とレスポンスを処理するコールバックを渡します。

```c#
/// <summary>
/// 接続しているサービスの全てのチャンネルのユーザーとルームの数をリクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="onAllChannelCountInfo">結果を受け取るデリゲート</param>
userAgent.GetAllChannelCountInfo((ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="userAgent">GetAllChannelCountInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetAllChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから受け取ったチャンネルのユーザー数とルーム数情報リスト</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// 全てのチャンネルカウント情報リクエスト成功
	} else {
		// 全てのチャンネルカウント情報リクエスト失敗
	}
});

/// <summary>
/// 特定サービスに存在する全てのチャンネルのユーザーとルームの数をリクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="serviceName">チャンネル情報をリクエストするサービスの名前</param>
/// <param name="onAllChannelCountInfo">結果を受け取るデリゲート</param>
userAgent.GetAllChannelCountInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="userAgent">GetAllChannelCountInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetAllChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから受け取ったチャンネルのユーザー数とルーム数情報リスト</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// 全てのチャンネルカウント情報リクエスト成功
	} else {
		// 全てのチャンネルカウント情報リクエスト失敗
	}
});
```
<br>

GetAllChannelInfo()はサービスの全てのチャンネルの情報(ユーザー定義)をリクエストして取得できます。パラメータでサービス名とレスポンスを処理するコールバックを渡します。

```c#
/// <summary>
/// 特定サービスの全てのチャンネル情報リクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="serviceName">チャンネル情報をリクエストするサービスの名前</param>
/// <param name="onAllChannelInfo">結果を受け取るデリゲート</param>
userAgent.GetAllChannelInfo((ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="userAgent">GeAllChannelInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GeAllChannelInfo()情報リクエスト結果</param>
    /// <param name="channelInfo">サーバーから受け取ったチャンネル情報リスト</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// 全てのチャンネル情報リクエスト成功
	} else {
		// 全てのチャンネル情報リクエスト失敗
	}
});

/// <summary>
/// 特定サービスの全てのチャンネル情報リクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="serviceName">チャンネル情報をリクエストするサービスの名前</param>
/// <param name="onAllChannelInfo">結果を受け取るデリゲート</param>
userAgent.GetAllChannelInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="userAgent">GeAllChannelInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GeAllChannelInfo()情報リクエスト結果</param>
    /// <param name="channelInfo">サーバーから受け取ったチャンネル情報リスト</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// 全てのチャンネル情報リクエスト成功
	} else {
		// 全てのチャンネル情報リクエスト失敗
	}
});
```

### Listener

UserAgentで全てのリクエストに対する結果やサーバーからの通知を受け取る方法は大きく2つです。
1つはUserAgentで定義されているdelegateに関数を追加する方法です。もう1つはIUserListenerインターフェイスを実装したリスナーを登録する方法です。

UserAgentは全ての動作の結果や通知を受けることができるようにそれぞれのdelegateをメンバーとして持っています。このdelegateに関数を登録すると、先に説明したAPIにコールバックパラメータを省略して呼び出した場合、またはサーバーから通知を送った場合、登録した関数でレスポンスを受けることができます。

```c#
/// <summary>
/// ログイン結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">Login()をリクエストしたユーザーエージェント</param>
/// <param name="result">Login()リクエスト結果</param>
/// <param name="loginInfo">ログイン情報</param>
public Interface.DelUserOnLogin onLoginListeners;

/// <summary>
/// ルームマッチリクエストの結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">MatchRoom()をリクエストしたユーザーエージェント</param>
/// <param name="result">MatchRoom()リクエスト結果</param>
/// <param name="resultCode">ユーザー結果コード</param>
/// <param name="roomId">マッチしたルームのID</param>
/// <param name="roomName">マッチしたルームの名前</param>
/// <param name="created">マッチしたルームの新設有無</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnMatchRoom onMatchRoomListeners;

/// <summary>
/// ルーム作成リクエスト結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">CreateRoom()をリクエストしたユーザーエージェント</param>
/// <param name="result"> CreateRoom()リクエスト結果</param>
/// <param name="roomName">作成されたルームの名前</param>
/// <param name="roomId">作成されたルームのID</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnCreateRoom onCreateRoomListeners;

/// <summary>
/// ルーム入室リクエスト結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">JoinRoom()をリクエストしたユーザーエージェント</param>
/// <param name="result">JoinRoom()リクエスト結果</param>
/// <param name="roomId">入室したルームのID</param>
/// <param name="roomName">入室したルームの名前</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnJoinRoom onJoinRoomListeners;

/// <summary>
/// 名前のあるルームリクエスト結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">NameRoom()をリクエストしたユーザーエージェント</param>
/// <param name="result">NameRoom()リクエスト結果</param>
/// <param name="roomName">ルームの名前</param>
/// <param name="roomId">入室したルームのID</param>
/// <param name="created">入室したルームの新設の有無</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnNamedRoom onNamedRoomListeners;

/// <summary>
/// パーティーマッチングリクエスト結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">MatchPartyStart()をリクエストしたユーザーエージェント</param>
/// <param name="result">MatchPartyStart()リクエスト結果</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnMatchPartyStart onMatchPartyStartListeners;

/// <summary>
/// パーティーマッチングリクエストキャンセル結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">MatchPartyCancel()をリクエストしたユーザーエージェント</param>
/// <param name="result">MatchPartyCancel()リクエスト結果</param>
public Interface.DelUserOnMatchPartyCancel onMatchPartyCancelListeners;

/// <summary>
/// ユーザーマッチングリクエスト結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">MatchUserStart()をリクエストしたユーザーエージェント</param>
/// <param name="result">MatchUserStart()リクエスト結果</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnMatchUserStart onMatchUserStartListeners;

/// <summary>
/// ユーザーマッチングリクエスト結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">MatchUserStart()またはMatchPartyStart()をリクエストしたユーザーエージェント</param>
/// <param name="result">MatchUserStart()またはMatchPartyStart()リクエストの結果</param>
/// <param name="created">ルームの新設の有無</param>
/// <param name="roomId">マッチングされたルームのID</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnMatchUserDone onMatchUserDoneListeners;

/// <summary>
/// ユーザーマッチングタイムアウト通知デリゲート
/// </summary>
/// <param name="userAgent">MatchUserStart()またはMatchPartyStart()をリクエストしたユーザーエージェント</param>
public Interface.DelUserOnMatchUserTimeout onMatchUserTimeoutListeners;

/// <summary>
/// ユーザーマッチングリクエストのキャンセル結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">MatchUserCancel()をリクエストしたユーザーエージェント</param>
/// <param name="result">MatchUserCancel()リクエスト結果</param>
public Interface.DelUserOnMatchUserCancel onMatchUserCancelListeners;

/// <summary>
/// ルーム退室または強制退室通知を受け取るデリゲート
/// </summary>
/// <param name="userAgent">LeaveRoom()をリクエストしたユーザーエージェント</param>
/// <param name="result">LeaveRoom()リクエスト結果</param>
/// <param name="force">強制退室かどうか</param>
/// <param name="roomId">退室したルームのID</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnLeaveRoom onLeaveRoomListeners;

/// <summary>
/// チャネル情報リクエストの結果、またはサーバーが強制的に行ったチャネルの移動について通知を受けるデリゲート
/// </summary>
/// <param name="userAgent">GetChannelInfo()をリクエストしたユーザーエージェント</param>
/// <param name="result">GetChannelInfo()リクエスト結果</param>
/// <param name="channelInfo">サーバーから受け取ったチャンネル情報</param>
public Interface.DelUserOnChannelInfo onChannelInfoListeners;

/// <summary>
/// 全てのチャンネル情報リストのリクエスト結果や、サーバーで強制的に行ったチャンネル移動の通知を受け取るデリゲート
/// </summary>
/// <param name="userAgent">GeAllChannelInfo()をリクエストしたユーザーエージェント</param>
/// <param name="result">GeAllChannelInfo()情報リクエスト結果</param>
/// <param name="channelInfo">サーバーから受け取ったチャンネル情報リスト</param>
public Interface.DelUserOnAllChannelInfo onAllChannelInfoListeners;

/// <summary>
/// チャネル情報リクエストの結果、またはサーバーが強制的に行ったチャネルの移動について通知を受けるデリゲート
/// </summary>
/// <param name="userAgent">GetChannelCountInfo()をリクエストしたユーザーエージェント</param>
/// <param name="result">GetChannelCountInfo()リクエスト結果</param>
/// <param name="channelCountInfo">サーバーから受け取ったチャンネルのユーザー数とルーム数情報</param>
public Interface.DelUserOnChannelCountInfo onChannelCountInfoListeners;

/// <summary>
/// 全てのチャンネル情報要求の結果、またはサーバーで強制的に行ったチャンネル移動の通知を受けるデリゲート
/// </summary>
/// <param name="userAgent">GetAllChannelCountInfo()をリクエストしたユーザーエージェント</param>
/// <param name="result">GetAllChannelCountInfo()リクエスト結果</param>
/// <param name="channelCountInfo">サーバーから受け取ったチャンネルのユーザー数とルーム数情報リスト</param>
public Interface.DelUserOnAllChannelCountInfo onAllChannelCountInfoListeners;

/// <summary>
/// チャネル移動リクエストの結果、またはサーバーが強制的に実行したチャンネル移動の通知を受けるデリゲート
/// </summary>
/// <param name="userAgent">MoveChannel()したユーザーエージェント</param>
/// <param name="result">MoveChannel()結果コード</param>
/// <param name="force">サーバーで強制的にチャンネルを移動したかどうか</param>
/// <param name="channelId">移動したチャンネルのID</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnMoveChannel onMoveChannelListeners;

/// <summary>
/// スナップショットリクエスト結果を受け取るデリゲート
/// </summary>
/// <param name="userAgent">Snapshot()したユーザーエージェント</param>
/// <param name="result">Snapshot()リクエスト結果</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnSnapshot onSnapshotListeners;

/// <summary>
/// 基本機能使用中にエラーが発生した時、通知を受け取るデリゲート
/// </summary>
/// <param name="userAgent">エラーが発生したユーザーエージェント</param>
/// <param name="errorCode">エラーコード</param>
/// <param name="command">エラー発生機能</param>
public Interface.DelUserOnErrorCommand onErrorCommandListeners;

/// <summary>
/// パケット送信エラーが発生した時に通知を受けるデリゲート
/// </summary>
/// <param name="userAgent">エラーが発生したユーザーエージェント</param>
/// <param name="errorCode">エラーコード</param>
/// <param name="command">エラーが発生したメッセージ</param>
public Interface.DelUserOnErrorCustomCommand onErrorCustomCommandListeners;

/// <summary>
/// ログアウト要求の結果または強制ログアウトの通知を受け取るデリゲート
/// </summary>
/// <param name="userAgent">Logout()をリクエストしたユーザーエージェント</param>
/// <param name="result">Logout()結果</param>
/// <param name="force">サーバーによる強制かどうか</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnLogout onLogoutListeners;

/// <summary>
/// 告知通知を受け取るデリゲート
/// </summary>
/// <param name="userAgent">Notice()を受け取ったユーザーエージェント</param>
/// <param name="message">告知メッセージ</param>
public Interface.DelUserOnNotice onNoticeListeners;

/// <summary>
/// キックアウト通知を受け取るデリゲート
/// </summary>
/// <param name="userAgent">キックアウトされたユーザーエージェント</param>
/// <param name="message">Adminから受け取ったメッセージ</param>
public Interface.DelUserOnAdminKickout onAdminKickoutListeners;

/// <summary>
/// サーバーのセッションが閉じられた場合、通知を受け取るデリゲート
/// </summary>
/// <param name="userAgent">セッションが閉じられたユーザーエージェント</param>
/// <param name="result">セッションが閉じられた理由</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelUserOnSessionClose onSessionCloseListeners;
```
<br>

IUserListenerはUserAgentの全ての動作の結果や通知を定義したインターフェイスです。このインターフェイスを実装したリスナーをUserAgent.AddUserListener()で登録すると、登録したリスナーでレスポンスを受けることができます。

```c#
class UserListener : IUserListener{
    /// <summary>
    /// Login()リクエスト結果
    /// </summary>
    /// <param name="userAgent">Login()をリクエストしたユーザーエージェント</param>
    /// <param name="result">Login()リクエスト結果</param>
    /// <param name="loginInfo">ログイン情報</param>
    void OnLogin(UserAgent userAgent, GameAnvil.Defines.ResultCodeLogin result, UserAgent.LoginInfo loginInfo);
    
    /// <summary>
    /// MatchRoom()リクエスト結果
    /// </summary>
    /// <param name="userAgent">MatchRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchRoom()リクエスト結果</param>
    /// <param name="resultCode">ユーザー結果コード</param>
    /// <param name="roomId">マッチしたルームのID</param>
    /// <param name="roomName">マッチしたルームの名前</param>
    /// <param name="created">マッチしたルームの新設有無</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnMatchRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload);
    
    /// <summary>
    /// CreateRoom()リクエスト結果
    /// </summary>
    /// <param name="userAgent">CreateRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result">CreateRoom()リクエスト結果</param>
    /// <param name="roomId">作成されたルームのID</param>
    /// <param name="roomName">作成されたルームの名前</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnCreateRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeCreateRoom result, int roomId, string roomName, Payload payload);
    
    /// <summary>
    /// JoinRoom()リクエスト結果
    /// </summary>
    /// <param name="userAgent">JoinRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result">JoinRoom()リクエスト結果</param>
    /// <param name="roomId">入室したルームのID</param>
    /// <param name="roomName">入室したルームの名前</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnJoinRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeJoinRoom result, int roomId, string roomName, Payload payload);
    
    /// <summary>
    /// NameRoom()リクエスト結果
    /// </summary>
    /// <param name="userAgent">NameRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result">NameRoom()リクエスト結果</param>
    /// <param name="roomName">ルーム名</param>
    /// <param name="roomId">入室したルームのID</param>
    /// <param name="created">入室したルームの新設の有無</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnNamedRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeNamedRoom result, int roomId, string roomName, bool created, Payload payload);
    
    /// <summary>
    /// MatchUserStart()結果
    /// </summary>
    /// <param name="userAgent">MatchUserStart()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchUserStart()リクエスト結果</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnMatchUserStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserStart result, Payload payload);
    
    /// <summary>
    /// MatchUserStart()またはMatchPartyStart()リクエストの結果
    /// </summary>
    /// <param name="userAgent">MatchUserStart()またMatchPartyStart()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchUserStart()またはMatchPartyStart()のリクエスト結果</param>
    /// <param name="created">ルームの新設の有無</param>
    /// <param name="roomId">マッチングされたルームのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnMatchUserDone(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload);
    
    /// <summary>
    /// MatchUserStart()またはMatchPartyStart()のタイムアウト
    /// </summary>
    /// <param name="userAgent">MatchUserStart()またMatchPartyStart()をリクエストしたユーザーエージェント</param>
    void OnMatchUserTimeout(UserAgent userAgent);
    
    /// <summary>
    /// MatchUserCancel()の結果
    /// </summary>
    /// <param name="userAgent">MatchUserCancel()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchUserCancel()リクエスト結果</param>
    void OnMatchUserCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserCancel result);
    
    /// <summary>
    /// MatchPartyStart()リクエスト結果
    /// </summary>
    /// <param name="userAgent">MatchPartyStart()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchPartyStart()リクエスト結果</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnMatchPartyStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchPartyStart result, Payload payload);
    
    /// <summary>
    /// MatchPartyCancel()リクエスト結果
    /// </summary>
    /// <param name="userAgent">MatchPartyCancel()をリクエストしたユーザーエージェント</param>
    /// <param name="result">MatchPartyCancel()リクエスト結果</param>
    void OnMatchPartyCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchPartyCancel result);
    
    /// <summary>
    /// LeaveRoom()リクエスト結果
    /// </summary>
    /// <param name="userAgent">LeaveRoom()をリクエストしたユーザーエージェント</param>
    /// <param name="result">LeaveRoom()リクエスト結果</param>
    /// <param name="force">強制退室するかどうか</param>
    /// <param name="roomId">退室したルームのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnLeaveRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeLeaveRoom result, bool force, int roomId, Payload payload);
    
    /// <summary>
    /// GetChannelInfo()リクエスト結果
    /// </summary>
    /// <param name="userAgent">GetChannelInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetChannelInfo()リクエスト結果</param>
    /// <param name="channelInfo">サーバーから受け取ったチャンネル情報</param>
    void OnChannelInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeChannelInfo result, Payload channelInfo);
    
    /// <summary>
    /// GeAllChannelInfo()リクエスト結果
    /// </summary>
    /// <param name="userAgent">GeAllChannelInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GeAllChannelInfo()情報リクエスト結果</param>
    /// <param name="channelInfo">サーバーから受け取ったチャンネル情報リスト</param>
    void OnAllChannelInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeAllChannelInfo result, Dictionary<string, Payload> channelInfo);
    
    /// <summary>
    /// GetChannelCountInfo()リクエスト結果
    /// </summary>
    /// <param name="userAgent">GetChannelCountInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから受け取ったチャンネルのユーザー数とルーム数情報</param>
    void OnChannelCountInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo);
    
    /// <summary>
    /// GetAllChannelCountInfo()リクエスト結果
    /// </summary>
    /// <param name="userAgent">GetAllChannelCountInfo()をリクエストしたユーザーエージェント</param>
    /// <param name="result">GetAllChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから受け取ったチャンネルのユーザー数とルーム数情報リスト</param>
    void OnAllChannelCountInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo);
    
    /// <summary>
    /// MoveChannel()結果
    /// </summary>
    /// <param name="userAgent">MoveChannel()したユーザーエージェント</param>
    /// <param name="result">MoveChannel()結果コード</param>
    /// <param name="force">サーバーが強制的にチャンネルを移動したかどうか</param>
    /// <param name="channelId">移動したチャンネルのID</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnMoveChannel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMoveChannel result, bool force, string channelId, Payload payload);
    
    /// <summary>
    /// Snapshot()リクエスト結果
    /// </summary>
    /// <param name="userAgent">Snapshot()リクエストしたユーザーエージェント</param>
    /// <param name="result">Snapshot()リクエスト結果</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnSnapshot(UserAgent userAgent, GameAnvil.Defines.ResultCodeSnapshot result, Payload payload);
    
    /// <summary>
    /// 基本機能エラー発生
    /// </summary>
    /// <param name="userAgent">エラーが発生したユーザーエージェント</param>
    /// <param name="errorCode">エラーコード</param>
    /// <param name="command">エラー発生機能</param>
    void OnError(UserAgent userAgent, GameAnvil.Defines.ErrorCode errorCode, User.Defines.Commands command);
    
    /// <summary>
    /// パケット送信エラー発生
    /// </summary>
    /// <param name="userAgent">エラーが発生したユーザーエージェント</param>
    /// <param name="errorCode">エラーコード</param>
    /// <param name="command">エラーが発生したメッセージ</param>
    void OnError(UserAgent userAgent, GameAnvil.Defines.ErrorCode errorCode, string command);
    
    /// <summary>
    /// Logout()結果
    /// </summary>
    /// <param name="userAgent">Logout()をリクエストしたユーザーエージェント</param>
    /// <param name="result">Logout()結果</param>
    /// <param name="force">サーバーによる強制かどうか</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnLogout(UserAgent userAgent, GameAnvil.Defines.ResultCodeLogout result, bool force, Payload payload);
    
    /// <summary>
    /// Notice()通知
    /// </summary>
    /// <param name="userAgent">Notice()を受け取ったユーザーエージェント</param>
    /// <param name="message">告知メッセージ</param>
    void OnNotice(UserAgent userAgent, string message);
    
    /// <summary>
    /// キックアウトされた場合の通知
    /// </summary>
    /// <param name="userAgent">キックアウトされたユーザーエージェント</param>
    /// <param name="message">Adminから受け取ったメッセージ</param>
    void OnAdminKickout(UserAgent userAgent, string message);
    
    /// <summary>
    /// ユーザーリスナーでサーバーのセッションが閉じられた場合の通知<para></para>
    /// この通知を受け取った場合、再ログインして再起動します。
    /// </summary>
    /// <param name="userAgent">セッションが閉じられたユーザーエージェント</param>
    /// <param name="result">セッションが閉じられた理由</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    void OnSessionClose(UserAgent userAgent, GameAnvil.Defines.ResultCodeSessionClose result, Payload payload);
}

UserAgent userAgent = connector.GetUserAgent(serviceId, subId);
userAgent.AddUserListener(new UserListener());
```
