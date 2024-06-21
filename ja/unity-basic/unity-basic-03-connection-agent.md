## Game > GameAnvil > Unity基礎開発ガイド > ConnectionAgent

## QuickConnect

GameAnvilConnectorではQuickConnect機能を提供し、サーバーに接続、認証、ログインする手順を一度に処理できるようにします。接続、認証、ログインする手順の詳細については、[Unity深層開発ガイド > ConnectionAgent](../unity-advanced/unity-advanced-02-connection-agent.md)またはサーバー開発ガイドを参照してください。

### QuickConnect設定

QuickConnectに使用される設定値があります。GameAnvilConnector作成時にデフォルトで設定されますが、必要に応じてインスペクタウィンドウで直接値を変更できます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/03-connection-agent/01-config.png)

設定の種類は次のとおりです。

| 名前 | 説明 |
| --- | --- |
| ip | 対象IPアドレス |
| port | 対象ポート番号 |
| accountId | ユーザーアカウントを識別できる固有のID |
| deviceId | ユーザーデバイスを識別するための固有のID。サーバーの実装により、使用しない場合は空の文字列を伝達 |
| password | ユーザーアカウントのパスワード。サーバーの実装により、使用しない場合は空の文字列を伝達 |
| userType | ユーザーのタイプ |
| channelId | 対象チャンネルのID |
| serviceName | 対象サービスの名前 |

### QuickConnectの使用

QuickConnect()を使ってサーバーに接続、認証、ログインします。引数で認証ペイロード、ログインペイロード、DelOnQuickConnectコールバックを渡します。認証ペイロードとログインペイロードは省略できます。

```c#
/// <summary>
/// GameAnvilサーバーに接続試行、接続成功時、認証リクエスト、認証成功時、ログインリクエスト
/// 接続/認証/ログインが全て成功した場QUICK_CONNECT_SUCCESS
/// </summary>
/// <param name="authenticatePayload">認証のために追加で渡す情報</param>
/// <param name="loginPayload">ログインのために追加で渡す情報</param>
/// <param name="onQuickConnect">QuickConnectリクエスト結果を受け取るデリゲート</param>
public void QuickConnect(Payload authenticatePayload, Payload loginPayload, DelOnQuickConnect onQuickConnect){
    if (quickConnect != null)
    {
        CallQuickConnectCallback(onQuickConnect, ResultCodeQuickConnect.QUICK_CONNECT_ALREADY_REQUESTED, null, null);
        return;
    }

    quickConnect = StartCoroutine(QuickConnectCoroutines(authenticatePayload, loginPayload, onQuickConnect, new QuickConnectResult()));
}
```

<br>

DelOnQuickConnectコールバックを使ってQuickConnectの結果を受け取ることができます。

```c#
/// <summary>
/// QuickConnectリクエスト結果を受け取るデリゲート
/// </summary>
/// <param name="resultCode">QuickConnectリクエスト結果</param>
/// <param name="userAgent">ルームの作成、入室などユーザーに関する機能を使用できるようにするエージェント</param>
/// <param name="quickConnectResult">サーバーから受け取った認証、ログインに関する追加情報。接続リクエスト結果、認証リクエスト結果、ログインリクエスト結果の集合</param>
public delegate void DelOnQuickConnect(ResultCodeQuickConnect resultCode, UserAgent userAgent, QuickConnectResult quickConnectResult);
```

## Disconnect

GameAnvilConnectorのDisconnect(), QuickDisconnect()関数を利用してサーバーとの接続を解除できます。

QuickDisconnect()呼び出し時、ログアウト、サーバー接続終了、接続解除後処理するコールバック呼び出しの過程を一度に処理できます。

```c#
/// <summary>
/// ログアウト、 GameAnvilサーバーとの接続解除リクエスト、コールバックの呼び出し
/// </summary>
/// <param name="onQuickDisconnect">リクエスト結果を受け取るデリゲート</param>
public void QuickDisconnect(DelOnQuickDisconnect onQuickDisconnect);
```

Disconnect()呼び出し時、サーバー接続が終了します。

```c#
/// <summary>
/// GameAnvilサーバーとの接続解除リクエスト
/// </summary>
public void Disconnect();
```
