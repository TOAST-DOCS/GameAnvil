## Game > GameAnvil > Unity深層開発ガイド > 再接続

## 再接続

ゲーム中に様々な理由でサーバーとの接続が切断されることがあります。接続が切断された時、既存のプレイを継続できるように再接続機能をサポートします。使用するAPIは一般的な接続方法と同じですが、結果として受け取る情報に違いがあります。

### サーバー接続

ConnectionAgentのConnect関数を利用してサーバーに接続します。一般的な場合と同じです。

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
    } 
    else {
      // 接続失敗
    }
});
```

### 認証

ConnectionAgentのAuthenticate関数を利用して認証手続きを行います。入力値は一般的な場合と同じです。ただ、認証の結果として受け取った値のうちloginedUserInfoListに以前プレイしていたユーザー情報が入ってきます。

```c#
/// <summary>
/// GameAnvilサーバーに認証リクエスト<para></para>
/// 認証成功後、コネクタ使用可能
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
      foreach(ConnectionAgent.LoginedUserInfo loginedUserInfo in loginedUserInfoList)
      {
        // プレイユーザー情報
      }
    } 
});
```

### ログイン

認証結果で受け取ったユーザー情報を使ってログインをします。この時、userTypeやchannelIdなど以前のユーザー情報と同じ値を使ってログインする必要があります。そうしないと、ログインが失敗する可能性があります。

```c#
/// <summary>
/// ユーザーエージェントを探して返す
/// </summary>
/// <param name="serviceName">ユーザーエージェントが使用するサービス名</param>
/// <param name="subId">サービス別ユーザーエージェントを識別できる固有ID</param>
/// <returns>該当ユーザーエージェント、なければnull</returns>
UserAgent userAgent = Connector.GetUserAgent(loginedUserInfo.ServiceName, loginedUserInfo.SubId);

/// <summary>
/// サービスにログイン
/// </summary>
/// <param name="userType">ユーザーのタイプ </param>
/// <param name="payload">サーバーに伝達する追加情報</param>
/// <param name="channelId">ログインするチャンネルのID</param>
/// <param name="onLogin">結果を受け取るデリゲート</param>
userAgent.Login(loginedUserInfo.UserType, loginedUserInfo.ChannelId, null,
                 (UserAgent agent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => {
    /// <param name="userAgent">Login()をリクエストしたユーザーエージェント</param>
    /// <param name="result">Login()リクエスト結果</param>
    /// <param name="loginInfo">ログイン情報</param>
	if (result == ResultCodeLogin.LOGIN_SUCCESS) {
        if(loginInfo.IsRelogined){
            // 再接続
        }else{
            // 最初の接続
        }
	}
});
```

ログイン成功時、UserAgent.LoginInfoのIsReloginedがtrueなら再接続ログインに成功したことになります。UserAgent.LoginInfoに含まれている情報を基に再接続処理を行ってください。この時、一番重要なことは再接続時間の間、サーバーの変更されたデータをクライアントに同期することです。この同期に必要なデータをUserAgent.LoginInfoのPayloadに入れて処理できます。

再接続ログインをすると、サーバーではBaseUser.onRelogin()コールバックが呼び出されます。この時、同期に必要なメッセージを定義してoutPayloadパラメータに追加すると、UserAgent.LoginInfoのPayloadで受け取って処理できます。

もし、IsReloginedがfalseであれば、時間が過ぎて以前のユーザー情報が削除された後、ログインされたことになります。この場合は、初めてログインする手順を実行してください。
