## Game > GameAnvil > Unity深層開発ガイド > ConnectionAgent

## ConnectionAgent

ConnectionAgentはGameAnvilサーバーのコネクションノードに関する作業を担当します。接続(Connect())、認証(Authentication())などの基本的なセッション管理機能やチャンネルリストなどを提供し、直接定義したプロトコルに基づいて様々なコンテンツを実装できます。ConnectionAgentはコネクタが初期化される時に自動的に作成され、Connector.GetConnectionAgent()関数を利用して取得できます。

```c#
ConnectionAgent connectionAgent = connector.GetConnectionAgent();
```

### サーバー接続

ConnectionAgentのConnect関数を利用してサーバーに接続します。 

```c#
/// <summary>
/// GameAnvilサーバーに接続試行
/// </summary>
/// <param name="ip">対象IPアドレス</param>
/// <param name="port">対象ポート</param>
/// <param name="onConnect">接続試行結果を受け取るデリゲート</param>
connector.GetConnectionAgent().Connect(ip, port, (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
    /// <param name="connectionAgent">Connect()をリクエストしたコネクションエージェント</param>
    /// <param name="result">Connect()リクエスト結果コード</param>
    if (result == ResultCodeConnect.CONNECT_SUCCESS) {
		// 接続成功
    } else {
	    // 接続失敗
    }
});
```

### 認証

ConnectionAgentのAuthenticate関数を使って認証手続きを行います。パラメータでdeviceId、accountId、password、payload、レスポンスを処理するコールバックを渡します。 deviceIdは重複接続に対する処理に活用され、accountIdとpasswordを活用してサーバーで認証処理を行うことができます。認証のためにdeviceId、accountId、password以外の追加情報が必要な場合、payloadに入れて送ることができ、使わない場合はpayloadパラメータは省略できます。
Authenticate関数を呼び出すとサーバーではBaseConnectionのonAuthentication()コールバックが呼び出され、このコールバックの処理結果で認証の成功、失敗が決定されます。

```c#
/// <summary>
/// GameAnvilサーバーに認証リクエスト<para></para>
/// 認証成功後、コネクタを使用可能
/// </summary>
/// <param name="deviceId">ユーザーのデバイスを識別するための固有のID。サーバーの実装により、使用しない場合は空の文字列を伝達</param>
/// <param name="accountId">ユーザーアカウントを識別できる固有のID</param>
/// <param name="passwd">ユーザーアカウントのパスワード。サーバーの実装により、使用しない場合は空の文字列を伝達</param>
/// <param name="onAuthentication">リクエスト結果を受け取るデリゲート</param>
connector.GetConnectionAgent().Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    /// <param name="connectionAgent">Authentication()をリクエストしたコネクションエージェント</param>
    /// <param name="result">Authentication()リクエスト結果</param>
    /// <param name="loginedUserInfoList">サーバーに残っているログイン情報リスト</param>
    /// <param name="message">サーバーから受け取ったメッセージ</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
		// 認証成功
    } else {
		// 認証失敗
    }
});
```

### チャンネル情報

GameAnvilは設定で自由にチャンネルを構成できます。このようなチャンネル構成はサーバーとクライアントの間であらかじめ約束して固定された形で使うこともできますが、状況によって多様に変更して使うこともできます。ConnectionAgentでは、このように変更されたチャンネル情報を取得できるようにいくつかの関数を提供しています。

| 関数 | 説明 |
| --- | --- |
| GetChannelList() | 特定サービスのチャンネルIDリストをリクエスト | 
| GetChannelCountInfo() | 特定チャンネルのカウント情報(ユーザーとルーム数)をリクエスト | 
| GetChannelInfo() | 特定チャンネルの情報(ユーザー定義)をリクエスト |
| GetAllChannelCountInfo() | 特定サービスの全てのチャンネルのカウント情報(ユーザーとルーム数)をリクエスト |

<br>

下記のコードでさらに詳しく説明します。


GetChannelList()は特定サービスのチャンネルIDリストをリクエストして取得できます。

```c#
/// <summary>
/// サービスの使用可能なチャンネルリストをリクエスト 
/// </summary>
/// <param name="serviceName">対象サービスの名前</param>
/// <param name="onChannelList">リクエスト結果を受け取るデリゲート</param>
connector.GetConnectionAgent().GetChannelList(serviceName, (ConnectionAgent connection, ResultCodeChannelList result, List<string> channelIdList) => {
    /// <param name="connectionAgent">GetChannelList()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetChannelList()リクエスト結果</param>
    /// <param name="channelIdList">サーバーから取得したチャンネルリスト</param>
	if(result == ResultCodeChannelList.CHANNEL_LIST_SUCCESS){
		// チャンネルリストリクエスト成功
	} else {
		// チャンネルリストリクエスト失敗
	}
});
```
<br>

GetChannelCountInfo()は特定チャンネルのカウント情報(ユーザーとルーム数)をリクエストして取得できます。

```c#
/// <summary>
/// チャンネルのユーザーとルーム数リクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="serviceName">対象チャンネルが属するサービスの名前</param>
/// <param name="channelId">対象チャンネルのID</param>
/// <param name="onChannelCountInfo">リクエスト結果を受け取るデリゲート</param>
connector.GetConnectionAgent().GetChannelCountInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="connectionAgent">GetChannelCountInfo()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから取得したチャンネルのユーザー数とルーム数情報</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// チャンネルカウント情報リクエスト成功
	} else {
		// チャンネルカウント情報リクエスト失敗
	}
});
```
<br>

GetChannelInfo()は特定チャンネルの情報(ユーザー定義)をリクエストして取得できます。

```c#
/// <summary>
/// チャンネル情報リクエスト<para></para>
/// サーバーがサポートしている場合使用可能
/// </summary>
/// <param name="serviceName">対象チャンネルが属するサービスの名前</param>
/// <param name="channelId">対象チャンネルのID</param>
/// <param name="onChannelInfo">リクエスト結果を受け取るデリゲート</param>
connector.GetConnectionAgent().GetChannelInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="connectionAgent">GetChannelInfo()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetChannelInfo()リクエスト結果</param>
    /// <param name="channelInfo">サーバーから取得したチャンネル情報</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// チャンネル情報リクエスト成功
	} else {
		// チャンネル情報リクエスト失敗
	}
});
```
<br>

GetAllChannelCountInfo()は特定サービスの全てのチャンネルのカウント情報(ユーザーとルーム数)をリクエストして取得できます。

```c#
/// <summary>
/// サービス内の全てのチャンネルのユーザーとルーム数をリクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="serviceName">対象サービスの名前</param>
/// <param name="onAllChannelCountInfo">リクエスト結果を受け取るデリゲート</param>
connector.GeConnectionAgent().GetAllChannelCountInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="connectionAgent">GetAllChannelCountInfo()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetAllChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから取得したチャンネルのユーザー数とルーム数情報など</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// 全てのチャンネルカウント情報リクエスト成功
	} else {
		// 全てのチャンネルカウント情報リクエスト失敗
	}
});
```
<br>

GetAllChannelInfo()は特定のサービスの全てのチャンネルに関する情報(ユーザー定義)をリクエストして取得できます。

```c#
/// <summary>
/// サービス内の全てのチャンネルの情報をリクエスト<para></para>
/// サーバーがサポートしている場合、使用可能
/// </summary>
/// <param name="serviceName">対象サービスの名前</param>
/// <param name="onAllChannelInfo">リクエスト結果を受け取るデリゲート</param>
connector.GetConnectionAgent().GetAllChannelInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="connectionAgent"> GetAllChannelInfo()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetAllChannelInfo()リクエストの結果</param>
    /// <param name="channelInfo">サーバーから取得したチャンネル情報リスト</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// 全てのチャンネル情報リクエスト成功
	} else {
		// 全てのチャンネル情報リクエスト失敗
	}
});
```

### 接続終了

ConnectionAgentのDisconnect関数を使ってサーバーとの接続を終了します。

```c#
/// <summary>
/// GameAnvilサーバーとの接続解除リクエスト
/// </summary>
/// <param name="onDisconnect">リクエスト結果を受け取るデリゲート</param>
connector.GetConnectionAgent().Disconnect((ConnectionAgent connectionAgent, ResultCodeDisconnect result) => {
    /// <param name="connectionAgent">Disconnect()発生したコネクションエージェント</param>
    /// <param name="result">>Disconnect()結果</param>
    if (result == ResultCodeDisconnect.SOCKET_DISCONNECT) {
		// 正常終了
    } else {
	    // 異常終了
    }
});
```

### Listener

ConnectionAgentで全てのリクエストに対する結果やサーバーからの通知を受け取る方法は大きく2つです。
1つはConnectionAgentで定義されているdelegateに関数を追加する方法です。もう1つはIConnectionListenerインターフェイスを実装したリスナーを登録する方法です。

まず、最初の方法です。ConnectionAgentは全ての動作の結果や通知を受けることができるようにそれぞれのdelegateをメンバーとして持っています。先に説明したAPIにコールバックパラメータを省略して呼び出したり、サーバーから通知を送った場合、delegateに登録された関数でレスポンスを受けることができます。 

```c#
/// <summary>
/// Connect()の結果
/// </summary>
/// <param name="connectionAgent">Connect()をリクエストしたコネクションエージェント</param>
/// <param name="result">Connect()の結果</param>
public Interface.DelConnectionOnConnect onConnectListeners;

/// <summary>
/// Authentication()結果
/// </summary>
/// <param name="connectionAgent">Authentication()をリクエストしたコネクションエージェント</param>
/// <param name="result">Authentication()リクエスト結果</param>
/// <param name="loginedUserInfoList">サーバーに残っているログイン情報リスト</param>
/// <param name="message">サーバーから受け取ったメッセージ</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelConnectionOnAuthentication onAuthenticationListeners;

/// <summary>
/// GetChannelList()リクエスト結果
/// </summary>
/// <param name="connectionAgent">GetChannelList()をリクエストしたコネクションエージェント</param>
/// <param name="result">GetChannelList()リクエスト結果</param>
/// <param name="channelIdList">サーバーから取得したチャンネルリスト</param>
public Interface.DelConnectionOnChannelList onChannelListListeners;

/// <summary>
/// GetChannelInfo()リクエスト結果
/// </summary>
/// <param name="connectionAgent">GetChannelInfo()をリクエストしたコネクションエージェント</param>
/// <param name="result">GetChannelInfo()リクエスト結果</param>
/// <param name="channelInfo">サーバーから取得したチャンネル情報</param>
public Interface.DelConnectionOnChannelInfo onChannelInfoListeners;

/// <summary>
/// GetAllChannelInfo()リクエスト結果
/// </summary>
/// <param name="connectionAgent"> GetAllChannelInfo()をリクエストしたコネクションエージェント</param>
/// <param name="result">GetAllChannelInfo()リクエストの結果</param>
/// <param name="channelInfo">サーバーから取得したチャンネル情報リスト</param>
public Interface.DelConnectionOnAllChannelInfo onAllChannelInfoListeners;

/// <summary>
/// GetChannelCountInfo()リクエスト結果
/// </summary>
/// <param name="connectionAgent">GetChannelCountInfo()をリクエストしたコネクションエージェント</param>
/// <param name="result">GetChannelCountInfo()リクエスト結果</param>
/// <param name="channelCountInfo">サーバーから取得したチャンネルのユーザー数とルーム数情報</param>
public Interface.DelConnectionOnChannelCountInfo onChannelCountInfoListeners;

/// <summary>
/// GetAllChannelCountInfo()リクエスト結果
/// </summary>
/// <param name="connectionAgent">GetAllChannelCountInfo()をリクエストしたコネクションエージェント</param>
/// <param name="result">GetAllChannelCountInfo()リクエスト結果</param>
/// <param name="channelCountInfo">サーバーから取得したチャンネルのユーザー数とルーム数情報など</param>
public Interface.DelConnectionOnAllChannelCountInfo onAllChannelCountInfoListeners;

/// <summary>
/// Disconnect()通知
/// </summary>
/// <param name="connectionAgent">Disconnect()発生したコネクションエージェント</param>
/// <param name="result">>Disconnect()理由</param>
/// <param name="force">強制終了するかどうか</param>
/// <param name="payload">サーバーから受け取った追加情報</param>
public Interface.DelConnectionOnDisconnect onDisconnectListeners;

/// <summary>
/// コネクションの基本機能使用中エラー発生
/// </summary>
/// <param name="connectionAgent">エラー発生したコネクションエージェント</param>
/// <param name="errorCode">エラーコード</param>
/// <param name="commands">エラー発生機能</param>
public Interface.DelConnectionOnErrorCommand onErrorCommandListeners;

/// <summary>
/// パケット送信後にエラー発生
/// </summary>
/// <param name="connectionAgent">処理をリクエストするコネクションエージェント</param>
/// <param name="errorCode">エラーコード</param>
/// <param name="command">パケットのメッセージ名前</param>
public Interface.DelConnectionOnErrorCustomCommand onErrorCustomCommandListeners;
```
<br>

次に、2つ目の方法です。IConnectionListenerはConnectionAgentの全ての動作の結果や通知を定義したインターフェイスです。このインターフェイスを実装したリスナーをConnectionAgent.AddConnectionListener()で登録すると、登録したリスナーでレスポンスを受けることができます。

```c#
public class ConnectionListener : IConnectionListener
{
    /// <summary>
    /// Connect()の結果
    /// </summary>
    /// <param name="connectionAgent">Connect()をリクエストしたコネクションエージェント</param>
    /// <param name="result">Connect()の結果</param>
    public void OnConnect(ConnectionAgent connectionAgent, ResultCodeConnect result) { }
    
    /// <summary>
    /// Authentication()結果
    /// </summary>
    /// <param name="connectionAgent">Authentication()をリクエストしたコネクションエージェント</param>
    /// <param name="result">Authentication()リクエスト結果</param>
    /// <param name="loginedUserInfoList">サーバーに残っているログイン情報リスト</param>
    /// <param name="message">サーバーから受け取ったメッセージ</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    public void OnAuthentication(ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) { }

    /// <summary>
    /// GetChannelList()リクエスト結果 
    /// </summary>
    /// <param name="connectionAgent">GetChannelList()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetChannelList()リクエスト結果</param>
    /// <param name="channelIdList">サーバーから取得したチャンネルリスト</param>
    public void OnChannelList(ConnectionAgent connectionAgent, ResultCodeChannelList result, List<string> channelIdList) { }

    /// <summary>
    /// GetChannelInfoの結果
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="result">GetChannelInfoの結果</param>
    /// <param name="channelInfo">チャンネル情報</param>
    public void OnChannelInfo(ConnectionAgent connectionAgent, ResultCodeChannelInfo result, Payload channelInfo) { }

    /// <summary>
    /// 全てのチャンネル情報リクエスト結果
    /// </summary>
    /// <param name="connectionAgent">GetAllChannelInfo()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetAllChannelInfo()リクエストの結果</param>
    /// <param name="channelInfo">サーバーから取得したチャンネル情報</param>
    public void OnAllChannelInfo(ConnectionAgent connectionAgent, ResultCodeAllChannelInfo result, Dictionary<string, Payload> channelInfo) { }

    /// <summary>
    /// GetChannelCountInfo()リクエスト結果
    /// </summary>
    /// <param name="connectionAgent">GetChannelCountInfo()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから取得したチャンネルのユーザー数とルーム数情報</param>
    public void OnChannelCountInfo(ConnectionAgent connectionAgent, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) { }

    /// <summary>
    /// GetAllChannelCountInfo()情報リクエスト結果
    /// </summary>
    /// <param name="connectionAgent">GetAllChannelCountInfo()をリクエストしたコネクションエージェント</param>
    /// <param name="result">GetAllChannelCountInfo()リクエスト結果</param>
    /// <param name="channelCountInfo">サーバーから取得したチャンネルのユーザー数とルーム数情報など</param>
    public void OnAllChannelCountInfo(ConnectionAgent connectionAgent, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) { }
    
    /// <summary>
    /// Disconnect()または強制接続終了結果通知
    /// </summary>
    /// <param name="connectionAgent">Disconnect()発生したコネクションエージェント</param>
    /// <param name="result">Disconnect()結果または強制接続終了理由</param>
    /// <param name="force">終了強制するかどうか</param>
    /// <param name="payload">サーバーから受け取った追加情報</param>
    public void OnDisconnect(ConnectionAgent connectionAgent, ResultCodeDisconnect result, bool force, Payload payload) { }
    
    /// <summary>
    /// コネクションの基本機能使用中エラー発生
    /// </summary>
    /// <param name="connectionAgent">エラー発生したコネクションエージェント</param>
    /// <param name="errorCode">エラーコード</param>
    /// <param name="commands">エラー発生機能</param>
    public void OnError(ConnectionAgent connectionAgent, ErrorCode errorCode, Commands commands) { }
    
    /// <summary>
    /// パケット送信後にエラー発生
    /// </summary>
    /// <param name="connectionAgent">処理をリクエストするコネクションエージェント</param>
    /// <param name="errorCode">エラーコード</param>
    /// <param name="command">パケットのメッセージ名前</param>
    public void OnError(ConnectionAgent connectionAgent, ErrorCode errorCode, string command) { }
}

connector.GetConnectionAgent().AddConnectionListener(new ConnectionListener);
```
