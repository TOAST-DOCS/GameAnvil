## Game > GameAnvil > Unity基礎開発ガイド > Simple Login

## 簡単ログイン

GameAnvilManagerで提供される簡単ログイン機能は、GameAnvilサーバーへの接続、認証、ログインの手続きを一度に処理できるようにします。接続、認証、ログインの手続きに関する詳細は、[Unity詳細開発ガイド > ConnectionAgent](../unity-advanced/unity-advanced-02-connection.md)またはサーバー開発ガイドを参照してください。

### 簡単ログインの設定

簡単ログインを使用するには、GameAnvilManagerの設定のうち、ログイン動作時に使用される値をあらかじめ設定しておく必要があります。簡単ログインで使用される設定の種類は以下の通りです。

| 区分            | 名前          | 説明                     |   デフォルト値   |
|----------------|--------------|-------------------------|:---------:|
| Connect | IP | 簡単ログインで接続するサーバーのIP | 127.0.0.1 |
| | Port | 簡単ログインで接続するサーバーのPort | 18200 |
| Authentication | AccountId | 簡単ログインで使用するユーザー識別ID | test |
|                | DeviceId     | 簡単ログインで使用するデバイス固有値 | test |
|                | Password     | 簡単ログインで使用するパスワード | test |
| Login          | User Type    | 簡単ログインで使用するユーザータイプ | |
|                | Channel Id   | 簡単ログインで使用するチャネルID | |
|                | Service Name | 簡単ログインで使用するサービス名 | |

これらの設定は、UnityエディタでGameAnvilManagerのInspectorウィンドウから設定するか、スクリプトコードで設定できます。

```c#
public void Start()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    gameAnvilManager.ip = managerIp;
    gameAnvilManager.port = managerPort;
    gameAnvilManager.accountId = managerAccountId;
    gameAnvilManager.deviceId = managerDeviceId;
    gameAnvilManager.password = managerPassword;
    gameAnvilManager.userType = managerUserType;
    gameAnvilManager.channelId = managerChannelId;
    gameAnvilManager.serviceName = managerServiceName;
}
```

### 簡単ログインの使用

簡単ログインを通じて、サーバーへの接続、認証、ログインを行います。戻り値のLoginResultを利用して結果を確認できます。このとき、サーバーとの接続に問題が発生した場合は、例外が発生することがあります。

```c#
public async void ManagerLogin()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    try
    {
        var result = await gameAnvilManager.Login();
        if (result.loginResultCode == GameAnvilManager.LoginResultCode.SUCCESS){
            // 成功
        } else {
            // 失敗
        }
    }
    catch (Exception e)
    {
        // サーバー接続に関連する問題が発生した場合、例外が発生
        // 例) 接続失敗、接続切断など
    }
}
```

<br>
サーバーの実装によっては、サーバーの認証プロセスやログインプロセスで追加情報が必要になる場合があります。そのような場合は、authenticatePayloadやloginPayloadで追加情報を渡すことができます。認証プロセスで必要な情報はauthenticatePayloadで、ログインプロセスで必要な情報はloginPayloadで渡します。

``` c#
public async void ManagerLogin()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    try
    {   
        var authenticatePayload = new Payload(new Protocol.AuthenticateData());
        var loginPayload = new Payload(new Protocol.LoginData());
        var result = await gameAnvilManager.Login(authenticatePayload, loginPayload);
        if (result.loginResultCode == GameAnvilManager.LoginResultCode.SUCCESS){
            // 成功
        } else {
            // 失敗
        }
    }
    catch (Exception e)
    {
        // サーバー接続に関連する問題が発生した場合、例外が発生
        // 例) 接続失敗、接続切断など
    }
}
```

簡単ログインに成功すると、LoginResultのUserControllerに、ログインが完了したユーザーのGameAnvilUserControllerインスタンスが割り当てられます。このインスタンスを利用して、サーバーのユーザーオブジェクトにアクセスできます。簡単ログインに失敗すると、UserControllerにはnullが割り当てられます。
<br>

簡単ログインに失敗した場合、その理由をLoginResultのloginResultCodeで確認できます。確認できる失敗理由は以下の通りです。

| 名前                                | 説明                                           |                                                             |
|------------------------------------|-----------------------------------------------|-------------------------------------------------------------|
| START_FAIL_LOGIN_IN_PROGRESS | 既にログインが進行中です。 | |
| START_FAIL_ALREADY_LOGGED_IN | 既にログイン済みです。 | |
| CONNECT_FAIL                       | サーバー接続失敗。                                     |                                                             |                    
| CONNECT_FAIL_INVALID_IP | サーバー接続失敗。不正なIPです。 | |
| CONNECT_FAIL_INVALID_PORT | サーバー接続失敗。不正なPortです。 | |
| AUTH_FAIL_CONTENT | 認証失敗。ユーザーコードで認証が拒否されました。 | |
| AUTH_FAIL_INVALID_ACCOUNT_ID | 認証失敗。不正なAccountIdです。 | 空文字列など |
| AUTH_FAIL_SYSTEM_ERROR | 認証失敗。サーバーの不明なエラーにより失敗しました。 | |
| AUTH_FAIL_TIMEOUT | 認証失敗。リクエストに対するレスポンスが指定時間内にありませんでした。 | |
| AUTH_FAIL_PARSE_ERROR | 認証失敗。追加情報のためのメッセージパースに失敗しました。 | サーバーとクライアントのプロトコル定義が異なる場合に発生することがあります。 |
| LOGIN_FAIL_CONTENT | ログイン失敗。ユーザーコードでログインが拒否されました。 | |
| LOGIN_FAIL_NOT_EXIST_NODE | ログイン失敗。ログイン対象のノードが存在しません。 | サーバー構成時に対象ノードを含むサーバーを起動しない場合に発生することがあります。 |
| LOGIN_FAIL_INVALID_SERVICEID | ログイン失敗。ログイン対象のサービスが存在しません。 | リクエスト時にサービス名を誤って入力したり、サーバー設定のサービス名設定が誤っている場合に発生することがあります。 |
| LOGIN_FAIL_INVALID_SERVICE_NAME | ログイン失敗。ログイン対象のサービスが存在しません。 | リクエスト時にサービス名を誤って入力したり、サーバー設定のサービス名設定が誤っている場合に発生することがあります。 |
| LOGIN_FAIL_TIMEOUT_GAME_SERVER | ログイン失敗。リクエストに対するレスポンスが指定時間内にありませんでした。 | |
| LOGIN_FAIL_INVALID_USERTYPE | ログイン失敗。ログイン対象のUserTypeが存在しません。 | リクエスト時にUserTypeを誤って入力したり、サーバーに実装されていない場合に発生することがあります。 |
| LOGIN_FAIL_INVALID_USERID | ログイン失敗。サーバーの不明な理由によりUserIdの作成に失敗しました。 | |
| LOGIN_FAIL_INVALID_CHANNEL_ID | ログイン失敗。ログイン対象のチャネルが存在しません。 | リクエスト時にChannelIdを誤って入力したり、サーバー設定にChannelIdがない場合に発生することがあります。 |
| LOGIN_FAIL_INVALID_SUB_ID | ログイン失敗。不正なSubIdです。 | |
| LOGIN_FAIL_LOGINED_OTHER_SERVICE | ログイン失敗。同じAccountIdで既に他のサービスにログインしています。 | サーバー構成時に重複ログインを許可しない場合に発生します。 |
| LOGIN_FAIL_LOGINED_OTHER_CHANNEL | ログイン失敗。同じAccountIdで他のチャネルにログインしています。 | |
| LOGIN_FAIL_LOGINED_OTHER_USER_TYPE | ログイン失敗。同じAccountId、異なるUserTypeでログインしています。 | サーバーの実装時に、既存のユーザーと新しくログインしたユーザーのどちらかを選択できます。 |
| LOGIN_FAIL_LOGINED_OTHER_DEVICE | ログイン失敗。同じAccountIdで他のデバイスからログインしています。 | サーバーの実装時に、既存のユーザーと新しくログインしたユーザーのどちらかを選択できます。 |
| LOGIN_FAIL_SYSTEM_ERROR | ログイン失敗。サーバーの不明なエラーにより失敗しました。 | |
| LOGIN_FAIL_PARSE_ERROR | ログイン失敗。追加情報のためのメッセージパースに失敗しました。 | |
| LOGIN_FAIL_TIMEOUT | ログイン失敗。リクエストに対するレスポンスが指定時間内にありませんでした。 | |
| UNKNOWN_ERROR | サーバーの不明なエラーにより失敗しました。 | |

より詳細な失敗理由を確認したい場合は、LoginResultのauthenticationResultやloginResultを利用して確認できます。

## Disconnect

GameAnvilManagerのLogout()関数を利用して、サーバーとの接続を解除できます。

Logout()を呼び出すと、ログアウト、サーバー接続終了、接続解除後の処理コールバック呼び出しまでを一度に処理できます。

```c#
public async void ManagerLogout()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    await gameAnvilManager.Logout();
}
```
