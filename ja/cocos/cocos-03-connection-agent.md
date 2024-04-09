## Game > GameAnvil > CocosCreator開発ガイド > ConnectionAgent

## ConnectionAgent

ConnectionAgentはGameAnvilサーバーのコネクションノードに関する作業を担当します。接続(Connect())、認証(Authentication())などの基本的なセッション管理機能やチャンネルリストなどを提供し、直接定義したプロトコルに基づいて様々なコンテンツを実装することができます。ConnectionAgentはコネクタが初期化される時に自動的に作成され、Connector.GetConnectionAgent() 関数を利用して取得できます。

```typescript
let connectionAgent = connector.GetConnectionAgent();
```

### サーバー接続

ConnectionAgentのConnect関数を使ってサーバーに接続します。 

```typescript
// GameAnvilサーバーに接続
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void;
//  ipAddress :接続するサーバーアドレス。 
// callback : 結果を受け取って処理するコールバック。
connectionAgent.Connect(ipAddress,(agent: ConnectionAgent, resultCode: ResultCodeConnect) => {
    //  agent : Connect()を呼び出したConnectionAgentオブジェクト。
	  //  resultCode : Connect結果。
    if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

### 認証

ConnectionAgentのAuthenticate関数を使って認証手続きを行います。引数でdeviceId、accountId、password、payload、レスポンスを処理するコールバックを渡します。 deviceIdは重複接続に対する処理に活用され、accountIdとpasswordを活用してサーバーで認証を処理できます。認証のためにdeviceId、accountId、password以外の追加情報が必要な場合、payloadに入れて送ることができます。Authenticate関数を呼び出すと、サーバーではBaseConnectionのonAuthentication()コールバックが呼び出され、このコールバックの処理結果で認証の成功、失敗が決定されます。

```typescript
/**
 * GameAnvilサーバーに認証をリクエスト
 * 
 * 認証に成功すると、GameAnvilコネクタの他の機能を使用可能
 * @param deviceId ユーザーのデバイスを識別できる固有のID。同じユーザーの重複接続をチェックするために使用
 * @param accountId ユーザーアカウントを識別できる固有のID
 * @param passwordユーザーアカウントのパスワード
 * @param payload サーバーに渡す追加情報
 * @param callback 結果を受け取って処理するコールバック
 */
connectionAgent.Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    /**
     * Authentication()の結果
     * @param connection Authentication()をリクエストしたコネクションエージェント
     * @param resultCode認証の結果コード
     * @param loginedUserInfoListサーバーに残っているログイン情報リスト
     * @param message サーバーから受け取ったメッセージ、認証失敗理由など
     * @param payload サーバーから受け取った追加情報
     */
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
		// 成功
    } else {
		// 失敗
    }
});
```

### チャンネル情報

GameAnvilは設定で自由にチャンネルを構成することができます。このようなチャンネル構成はサーバーとクライアントの間であらかじめ約束して固定された形で使うこともできますが、状況によって自由に変更して使うこともできます。ConnectionAgentではこのように変更されたチャンネル情報を取得できるようにいくつかの関数を提供します。

GetChannelList()は特定のサービスのチャンネルIDリストをリクエストして受け取ることができます。

```typescript
/**
 * 特定のサービスで使用可能なチャンネルリストをリクエスト
 * @param serviceName チャネル情報を要求するサービスの名前
 * @param callback 結果を渡して処理するコールバック
 */
connectionAgent.GetChannelList(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeChannelList, channelIdList: Array<string>)=>{
    /**
     * GetChannelList()の結果
     * @param connection GetChannelList()をリクエストした コネクションエージェント
     * @param resultCode GetChannelList()の結果コード
     * @param channelIdListチャンネルリスト
     */
    if (ResultCodeChannelList.CHANNEL_LIST_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

GetChannelCountInfo()は特定のチャンネルのカウント情報(ユーザーとルーム数)をリクエストして取得できます。

```typescript
/**
 * 特定チャンネルのユーザーとルーム数をリクエスト
 * 
 * サーバーでサポートしている場合、使用可能
 * @param serviceName チャネル情報を要求するサービスの名前
 * @param channelId チャンネル情報をリクエストするチャンネルのID
 * @param callback 結果を受け取って処理するコールバック
 */
connectionAgent.GetChannelCountInfo(serviceName, channelId, (agent: ConnectionAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo)=>{
    /**
     * GetChannelCountInfo()の結果
     * @param connection GetChannelCountInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetChannelCountInfo()の結果コード
     * @param channelCountInfoチャンネルのユーザーとルーム数
     */
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

GetChannelInfo()は特定チャンネルの情報(ユーザー定義)をリクエストして取得できます。

```typescript
/**
 * 特定チャンネルの情報をリクエスト
 * 
 * サーバーでサポートしている場合、使用可能
 * @param serviceName チャネル情報を要求するサービスの名前
 * @param channelId チャンネル情報をリクエストするチャンネルのID
 * @param callback 結果を受け取って処理するコールバック
 */
connectionAgent.GetChannelInfo(serviceName, channelId, (agent: ConnectionAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload)=>{
    /**
     * GetChannelInfo()の結果
     * @param connection GetChannelInfo()をリクエストした コネクションエージェント
     * @param resultCode GetChannelInfo()の結果コード
     * @param channelInfoチャンネル情報
     */
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

GetAllChannelCountInfo()は特定のサービスの全てのチャンネルに対するカウント情報(ユーザーとルーム数)をリクエストして取得できます。

```typescript
/**
 * 特定のサービスにある全てのチャンネルのユーザーとルームの数をリクエスト
 * 
 * サーバーでサポートしている場合、使用可能
 * @param serviceName チャネル情報を要求するサービスの名前
 * @param callback 結果を受け取って処理するコールバック
 */
connectionAgent.GetAllChannelCountInfo(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo)=>{
    /**
     * GetAllChannelCountInfo()の結果
     * @param connection GetAllChannelCountInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetAllChannelCountInfo()の結果コード
     * @param channelInfoListチャンネルのユーザーとルーム数
     */
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

GetAllChannelInfo()は特定サービスのすべてのチャンネルの情報(ユーザー定義)をリクエストして取得できます。

```typescript
/**
 * 特定のサービスにある全てのチャンネルのユーザーとルームの数をリクエスト
 * 
 * サーバーでサポートしている場合、使用可能
 * @param serviceName チャネル情報を要求するサービスの名前
 * @param callback 結果を受け取って処理するコールバック
 */
connectionAgent.GetAllChannelInfo(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo)=>{
    /**
     * GetAllChannelInfo()の結果
     * @param connection GetAllChannelInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetAllChannelInfoの結果コード
     * @param allChannelInfoチャンネル情報
     */
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

### 接続終了

ConnectionAgentのDisconnect関数を使ってサーバーとの接続を終了します。

```typescript
/**
 * GameAnvilサーバーとの接続を解除
 * @param reason コールバックに渡す接続解除の理由。
 * @param callback 接続終了時に呼び出されるコールバック
 */
connectionAgent.Disconnect(reason, (agent: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string) =>{
    /**
     * Disconnect()の結果またはまたは強制接続終了通知
     * @param connection Disconnect()をリクエストしたコネクションエージェント
     * @param resultCode Disconnect()の結果コード
     * @param reason終了理由。 ConnectionAgent.Disconnect()に引数で渡した理由。
     */
    if (ResultCodeDisconnect.SOCKET_DISCONNECT == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

### Listener

IConnectionListenerはConnectionAgentの全ての動作の結果や通知を定義したインターフェイスです。 このインターフェイスを実装したリスナーをConnectionAgent.AddConnectionListener()で登録すると、登録したリスナーでレスポンスを受けることができます。

```typescript
class ConnectionListener implements IConnectionListener{
    /**
     * Connect()の結果
     * @param connection Connect()をリクエストしたコネクションエージェント
     * @param resultCode接続の結果コード
     */
    OnConnect(connection: ConnectionAgent, resultCode: ResultCodeConnect): void { }
    /**
     * Authentication()の結果
     * @param connection Authentication()をリクエストしたコネクションエージェント
     * @param resultCode認証の結果コード
     * @param loginedUserInfoListサーバーに残っているログイン情報リスト
     * @param message サーバーから受け取ったメッセージ、認証失敗理由など
     * @param payload サーバーから受け取った追加情報
     */
    OnAuthentication(connection: ConnectionAgent, resultCode: ResultCodeAuth, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload): void { }
    /**
     * GetChannelList()の結果
     * @param connection GetChannelList()をリクエストした コネクションエージェント
     * @param resultCode GetChannelList()の結果コード
     * @param channelIdListチャンネルリスト
     */
    OnChannelList(connection: ConnectionAgent, resultCode: ResultCodeChannelList, channelIdList: Array<string>): void { }
    /**
     * GetChannelInfo()の結果
     * @param connection GetChannelInfo()をリクエストした コネクションエージェント
     * @param resultCode GetChannelInfo()の結果コード
     * @param channelInfoチャンネル情報
     */
    OnChannelInfo(connection: ConnectionAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload): void { }
    /**
     * GetChannelCountInfo()の結果
     * @param connection GetChannelCountInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetChannelCountInfo()の結果コード
     * @param channelCountInfoチャンネルのユーザーとルーム数
     */
    OnChannelCountInfo(connection: ConnectionAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo): void { }
    /**
     * GetAllChannelInfo()の結果
     * @param connection GetAllChannelInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetAllChannelInfoの結果コード
     * @param allChannelInfoチャンネル情報
     */
    OnAllChannelInfo(connection: ConnectionAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo): void { }
    /**
     * GetAllChannelCountInfo()の結果
     * @param connection GetAllChannelCountInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetAllChannelCountInfo()の結果コード
     * @param channelInfoListチャンネルのユーザーとルーム数
     */
    OnAllChannelCountInfo(connection: ConnectionAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo): void { }
    /**
     * Disconnect()の結果またはまたは強制接続終了通知
     * @param connection Disconnect()をリクエストしたコネクションエージェント
     * @param resultCode Disconnect()の結果コード
     * @param reason終了理由。 ConnectionAgent.Disconnect()に引数で渡した理由。
     * @param force 強制終了するかどうか
     * @param payload サーバーから受け取った追加情報
     */
    OnDisconnect(connection: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload): void { }
    /**
     * エラー発生
     * @param connection エラーが発生したコネクションエージェント
     * @param errorCode エラーコード
     * @param msgName エラーが発生したメッセージの名前
     */
    OnErrorCommand(connection: ConnectionAgent, errorCode: ErrorCode, msgName: string): void { }
};

connectionAgent.AddListener(new ConnectionListener());
```
