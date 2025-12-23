## Game > GameAnvil > Unity 応用開発ガイド > 再接続

## 再接続

ゲーム中、様々な理由でサーバーとの接続が切れることがあります。接続が切れた際、既存のプレイを継続できるように再接続機能をサポートします。使用するAPIは一般的な接続方法と同じですが、結果として受け取る情報に違いがあります。

### サーバー接続

GameAnvilConnectorのConnect関数を利用してサーバーに接続します。一般的な場合と同じです。

```c#
public async void Connect()
{
    try
    {
        await connector.Connect(host, port);
        // 成功
    }
    catch (Exception e)
    {
        // 失敗
    }
}
```

### 認証

GameAnvilConnectorのAuthenticate関数を利用して認証手続きを進めます。入力値は一般的な場合と同じです。ただし認証の結果として受け取るResultCodeAuth値のうち、loginedUserInfoListに以前プレイしていたユーザー情報が含まれて返ってきます。

```c#
public async void Authenticate()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        ErrorResult<ResultCodeAuth, AuthenticationResult> result = await connector.Authentication("DeviceId", "AccountId", "Password", );
        if(result.ErrorCode == ResultCodeAuth.AUTH_SUCCESS)
        {
            // 成功
            foreach(AlreadyLoginedUserInfo alreadyLoginedUserInfo in result.Data.LoginUserInfoList)
            {
                // 以前プレイしていたユーザー情報
            }
        } else
        {
            // 失敗
        }
    }
    catch (Exception e)
    {
        // 例外
    }
}
```

### ログイン

認証結果として受け取ったユーザー情報を利用してログインを進めます。この時、userTypeやchannelIdなど以前のユーザー情報と同じ値を利用してログインする必要があります。そうしないとログインに失敗する可能性があります。

```c#
public async void Login(AlreadyLoginedUserInfo userInfo)
{
    try
    {
        Payload loginPayload = new Payload(new Protocol.LoginData());
        GameAnvilUser user = new GameAnvilUser(connector, userInfo.ServiceName, userInfo.UserID);
        ErrorResult<ResultCodeLogin, LoginResult> result = await user.Login(userInfo.UserType, userInfo.ChannelId, loginPayload);
        if(result.ErrorCode == ResultCodeLogin.LOGIN_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    }
    catch (Exception e)
    {
        // 例外
    }
}
```

ログイン成功時、認証の結果として受け取るLoginResultのIsReloginedがtrueであれば、再接続ログインに成功したことになります。LoginResultに含まれる情報をもとに再接続処理を行えばよいです。この時最も重要なのが、再接続時間中にサーバーの変更されたデータをクライアントに同期することです。この同期に必要なデータをLoginResultのPayloadに入れて処理できます。

再接続ログインをすると、サーバーではUserオブジェクトのonRelogin()コールバックが呼び出されます。この時、同期に必要なメッセージを定義してoutPayloadパラメータに追加すれば、LoginResultのPayloadとして受け取り処理できます。

もしIsReloginedがfalseであれば、時間が経過して以前のユーザーがサーバーからログアウト処理された後にログインされたことになります。この場合には初めてログインする手順を実行すればよいです。 
