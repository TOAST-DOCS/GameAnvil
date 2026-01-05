## Game > GameAnvil > Unity 基礎開発ガイド > 簡易ログイン

## 簡単ログイン

GameAnvilManagerが提供する簡易ログイン機能は、GameAnvilサーバーへの接続、認証、ログインの手続きを一度に処理できるようにします。接続、認証、ログインの手続きに関するより詳細な内容は、[Unity 応用開発ガイド > コネクタ](../unity-advanced/unity-advanced-02-connection.md)またはサーバー開発ガイドを参照してください。

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
        if (retult.loginResultCode == GameAnvilManager.LoginResultCode.SUCCESS){
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
| AUTH_FAIL_PARSE_ERROR              | 認証失敗。追加情報のためのメッセージ解析失敗。                    | サーバーとクライアントのプロトコル定義が異なる場合に発生する可能性がある。                           |
| LOGIN_FAIL_CONTENT | ログイン失敗。ユーザーコードでログインが拒否されました。 | |
| LOGIN_FAIL_NOT_EXIST_NODE          | ログイン失敗。ログイン対象ノードが存在しない。                    | サーバー構成時に対象ノードを含むサーバーを起動していない場合に発生する可能性がある。                      |
| LOGIN_FAIL_INVALID_SERVICE_NAME | ログイン失敗。ログイン対象のサービスが存在しません。 | リクエスト時にサービス名を誤って入力したり、サーバー設定のサービス名設定が誤っている場合に発生することがあります。 |
| LOGIN_FAIL_TIMEOUT_GAME_SERVER     | ログイン失敗。リクエストに対するレスポンスが指定された時間内に来ない。             |                                                              |
| LOGIN_FAIL_INVALID_USER_TYPE       | ログイン失敗。ログイン対象UserTypeが存在しない。              | リクエスト時にUserTypeを誤って入力したか、サーバーに実装されていない場合に発生する可能性がある。           |
| LOGIN_FAIL_INVALID_USER_ID         | ログイン失敗。サーバーの不明な理由でUserId生成失敗。             |                                                              |
| LOGIN_FAIL_INVALID_CHANNEL_ID      | ログイン失敗。ログイン対象チャンネルが存在しない。                    | リクエスト時にChannelIdを誤って入力したか、サーバー設定にChannelIdがない場合に発生する可能性がある。 |
| LOGIN_FAIL_INVALID_SUB_ID | ログイン失敗。不正なSubIdです。 | |
| LOGIN_FAIL_LOGINED_OTHER_SERVICE   | ログイン失敗。同じAccountIdですでに他のサービスにログインされている。     | サーバー構成時に重複ログインを許可しない場合に発生。                       |
| LOGIN_FAIL_LOGINED_OTHER_CHANNEL | ログイン失敗。同じAccountIdで他のチャネルにログインしています。 | |
| LOGIN_FAIL_LOGINED_OTHER_USER_TYPE | ログイン失敗。同じAccountId、異なるUserTypeでログインしています。 | サーバーの実装時に、既存のユーザーと新しくログインしたユーザーのどちらかを選択できます。 |
| LOGIN_FAIL_LOGINED_OTHER_DEVICE | ログイン失敗。同じAccountIdで他のデバイスからログインしています。 | サーバーの実装時に、既存のユーザーと新しくログインしたユーザーのどちらかを選択できます。 |
| LOGIN_FAIL_SYSTEM_ERROR | ログイン失敗。サーバーの不明なエラーにより失敗しました。 | |
| LOGIN_FAIL_PARSE_ERROR | ログイン失敗。追加情報のためのメッセージパースに失敗しました。 | |
| LOGIN_FAIL_TIMEOUT               | ログイン失敗。リクエストに対するレスポンスが指定された時間内に来ない。             |                                                              |
| UNKNOWN_ERROR                    | サーバーの不明なエラーにより失敗。                             |                                                              |

より詳細な失敗理由は、LoginResultのauthenticationResultやloginResultを利用して確認できます。

## Disconnect

GameAnvilManagerのLogout()メソッドを利用してサーバーとの接続を解除できます。

Logout()呼び出し時、ログアウト、サーバー接続終了、接続解除後に処理するコールバック呼び出しの過程を一度に処理できます。サーバーの実装によっては、ログアウト過程で追加情報が必要になる場合があります。このような場合にはlogoutPayloadで追加情報を伝達できます。

```c#
public async void ManagerLogout()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    var logoutPayload = new Payload(new Protocol.LogoutData());
    await gameAnvilManager.Logout(logoutPayload);
}
```

## 状態変更通知

Logout()を呼び出さなくても、ネットワークに問題があったり、サーバーから強制的にログアウトさせられるなど、GameAnvilManagerの状態が変更される可能性があり、これに対する通知を受け取ることができます。 
```c#
public void addStateChangeListener()
{
    gameAnvilManager.onStateChange.AddListener((GameAnvilManager.LoginState oldState, GameAnvilManager.LoginState newState) =>
    {
        // 状態変更通知
    });
}
```
oldStateで変更前の状態を、newStateで変更後の状態を知ることができます。  
