## Game > GameAnvil > Unity 応用開発ガイド > コネクタ

## コネクタ

GameAnvilConnectorは、GameAnvilサーバーとの接続を管理する作業を担当します。接続(Connect())、認証(Authentication())など、基本的なセッション管理機能及びチャンネルリストなどを提供します。

### サーバー接続

Connect()を呼び出してサーバーに接続します。

```c#
public async void Connect()
{
    try
    {
        // connectorはGameAnvilConnectorインスタンスを保存した変数です。
        await connector.Connect(host, port);
        // 成功
    }
    catch (Exception e)
    {
        // 失敗
    }
}
```

Connect()は次のような2つのパラメータを持っています。

| タイプ   | 名前 | 説明             |
|--------|------|------------------|
| String | host | 接続するサーバーのipまたはアドレス |
| int    | port | 接続するサーバーのport     |

別途の戻り値はなく、成功した場合は次のコードを実行し、失敗した場合は例外が発生します。

### 認証

サーバーに接続した後、Authentication()を呼び出して認証手続きを進めます。
Authentication()メソッドを呼び出すと、サーバーではConnectionオブジェクトのonAuthenticate()コールバックが呼び出され、このコールバックの処理結果で認証の成功、失敗が決定されます。

```c#
public async void Authenticate()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        ErrorResult<ResultCodeAuth, AuthenticationResult> result = await connector.Authentication("DeviceId", "AccountId", "Password", authenticationPayload);
        if(result.ErrorCode == ResultCodeAuth.AUTH_SUCCESS)
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

Authentication()は次のような4つのパラメータを持っています。

| タイプ    | 名前      | 説明                                                                                                            |
|---------|-----------|-----------------------------------------------------------------------------------------------------------------|
| String  | deviceId  | 認証に使用するデバイスID。<br/>異なるデバイスから認証リクエストを行う場合、この値を利用してデバイスからのリクエストであることを識別できます。使用しない場合はstring.Empty(空文字)を入力 |
| String  | accountId | 認証に使用するアカウント名。                                                                                                                                                                                                                                                                              |
| String  | password  | 認証に使用するパスワード。使用しない場合はstring.Empty(空文字)を入力                                                                                                                                                                                                                             |
| Payload | payload   | 認証リクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null)                                                                                                                                                                                                                        |

レスポンスとしてErrorResult<ResultCodeAuth, AuthenticationResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。Authenticationが成功すればErrorCodeフィールドの値がResultCodeAuth.AUTH_SUCCESSになり、そうでない場合は認証に失敗したことになります。Dataフィールドを通じてリクエスト結果AuthenticationResultを取得できます。これを通じて認証結果情報を取得でき、サーバー実装によっては追加情報を取得することも
できます。

ResultCodeAuthの詳細は次のとおりです。

| 名前                         | 値 | 説明                                       |
|------------------------------|-----|--------------------------------------------|
| PARSE_ERROR                  | -2  | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                      | -1  | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                 | 1   | サーバーシステムエラー。サーバーの不明なエラーにより失敗                |
| INVALID_PROTOCOL             | 2   | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| AUTH_SUCCESS                 | 0   | 成功                                       |
| AUTH_FAIL_CONTENT            | 201 | 失敗。ユーザーコードで拒否されました                            |
| AUTH_FAIL_INVALID_ACCOUNT_ID | 602 | 失敗。無効なアカウントID                             |                                |

AuthenticationResultの詳細は次のとおりです。

| タイプ                         | 名前              | 説明                            |
|------------------------------|-------------------|---------------------------------|
| List<AlreadyLoginedUserInfo> | LoginUserInfoList | 認証成功時点でサーバーに残っているログイン済みユーザー情報リスト |
| String                       | Message           | 認証メッセージ                        |
| Payload                      | payload           | クライアントで必要な追加情報                |

### 接続と認証

接続または認証時に必要な値をGameAnvilConnectorの属性として保存して使用することもできます。

```c#
public  void initializeConnector()
{
    connector.Host = "127.0.0.1";
    connector.Port = 18200;
    connector.DeviceId = "DeviceId";
    connector.AccountId = "AccountId";
    connector.Password = "Password";
}

public async void ConnectAndAuthentication1()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        var (resultCode, result) = await connector.ConnectAndAuthentication(authenticationPayload);
        if (resultCode == ResultCodeAuth.AUTH_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    } catch (Exception e)
    {
        // 例外
    }
}

public async void ConnectAndAuthentication2()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        await connector.Connect();
        var (resultCode, result) = await connector.Authentication(authenticationPayload);
        if (resultCode == ResultCodeAuth.AUTH_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    } catch (Exception e)
    {
        // 例外
    }
}
```

### セキュア接続
GameAnvilサーバーにセキュリティ設定を行った場合は、ConnectSecure()を呼び出してセキュア接続を行う必要があります。 
```c#
public async void ConnectSecure()
{
    try
    {
        await connector.ConnectSecure(host, port);
        // 成功
    }
    catch (Exception e)
    {
        // 失敗
    }
}
```
ConnectSecure()は次のような2つのパラメータを持っています。

| タイプ   | 名前 | 説明             |
|--------|------|------------------|
| String | host | 接続するサーバーのipまたはアドレス |
| int    | port | 接続するサーバーのport     |

別途の戻り値はなく、成功した場合は次のコードを実行し、失敗した場合は例外が発生します。

### チャンネル情報

GameAnvilは設定で自由にチャンネルを構成できます。このようなチャンネル構成は、サーバーとクライアント間であらかじめ約束して固定された形態で使用することもできますが、状況に応じて多様に変更して使用することもできます。GameAnvilConnectorでは、このように変更されたチャンネル情報を取得できるようにいくつかのメソッドを提供します。 

| 名前                     | 説明                                  |
|--------------------------|---------------------------------------|
| GetChannelList()         | 特定サービスのチャンネルIDリストリクエスト                  | 
| GetChannelCountInfo()    | 特定チャンネルのカウント情報(ユーザーとルーム数)リクエスト             | 
| GetChannelInfo()         | 特定チャンネルの情報(ユーザー定義)リクエスト                  |
| GetAllChannelCountInfo() | 特定サービスの全てのチャンネルに対するカウント情報(ユーザーとルーム数)リクエスト |
| GetAllChannelInfo()      | 特定サービスの全てのチャンネルに対する情報(ユーザー定義)リクエスト        |

#### GetChannelList

GetChannelList()は、特定サービスのチャンネルIDリストをリクエストして受け取ることができます。

```c#
public async void ChannelList()
{
    try
    {
        Payload channelInfoPayload = new Payload(new Protocol.ChannelInfoData());
        ErrorResult<ResultCodeChannelList, List<string>> result = await connector.GetChannelList("ServiceName");
        if (result.ErrorCode == ResultCodeChannelList.CHANNEL_LIST_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    } catch (Exception e)
    {
        // 例外
    }
}
```

GetChannelList()は次のような1つのパラメータを持っています。

| タイプ   | 名前        | 説明           |
|--------|-------------|----------------|
| String | ServiceName | チャンネル情報をリクエストするサービス |

レスポンスとしてErrorResult<ResultCodeChannelList, List<string>>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。GetChannelListが成功すればErrorCodeフィールドの値がResultCodeChannelList.CHANNEL_LIST_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。成功時、Dataフィールドを通じてリクエスト結果であるチャンネルIDリストを取得できます。

ResultCodeChannelListの詳細は次のとおりです。

| 名前                              | 値  | 説明                                       |
|-----------------------------------|------|--------------------------------------------|
| PARSE_ERROR                       | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                           | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                      | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗                |
| INVALID_PROTOCOL                  | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| CHANNEL_LIST_SUCCESS              | 0    | 成功                                       |
| CHANNEL_LIST_FAIL_NO_CHANNEL_LIST | 1801 | 失敗。チャンネルリストが見つかりません                          |                                |                          

#### GetChannelCountInfo

GetChannelCountInfo()は、特定チャンネルのカウント情報(ユーザーとルーム数)をリクエストして受け取ることができます。

```c#
public async void ChannelCountInfo()
{
    try
    {
        ErrorResult<ResultCodeChannelCountInfo, ChannelCountResult> result = await connector.GetChannelCountInfo("ServiceName", "ChannelId");
        if (result.ErrorCode == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    } catch (Exception e)
    {
        // 例外
    }
}
```

GetChannelCountInfo()は次のような2つのパラメータを持っています。

| タイプ   | 名前        | 説明               |
|--------|-------------|--------------------|
| String | ServiceName | チャンネル情報をリクエストするサービス      |
| String | channelId   | チャンネル情報をリクエストするチャンネルのID |

レスポンスとしてErrorResult<ResultCodeChannelCountInfo, ChannelCountResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。GetChannelCountInfoが成功すればErrorCodeフィールドの値がResultCodeChannelCountInfo.CHANNEL_LIST_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。Dataフィールドを通じてリクエスト結果であるChannelCountResultを取得できます。

ResultCodeChannelCountInfoの詳細は次のとおりです。

| 名前                                       | 値  | 説明                                       |
|--------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                                | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                                    | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                               | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗                |
| INVALID_PROTOCOL                           | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| CHANNEL_COUNT_INFO_SUCCESS                 | 0    | 成功                                       |
| CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1921 | 失敗。チャンネル情報が見つかりません                          |
| CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1922 | 失敗。無効なサービスID                           |
| CHANNEL_COUNT_INFO_FAIL_INVALID_CHANNEL_ID | 1923 | 失敗。無効なチャンネルID                            |
| CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924 | 失敗。チャンネルが見つかりません                            |

ChannelCountResultの詳細は次のとおりです。

| タイプ   | 名前      | 説明         |
|--------|-----------|--------------|
| string | ChannelId | チャンネルID返却。    |
| int    | UserCount | チャンネルのユーザー数返却。 |
| int    | RoomCount | チャンネルのルーム数返却。  |

<br>

#### GetChannelInfo

GetChannelInfo()は特定チャンネルの情報(ユーザー定義)をリクエストして受け取ることができます。

```c#
public async void ChannelInfo()
{
    try
    {
        ErrorResult<ResultCodeChannelInfo, Payload> result = await connector.GetChannelInfo("ServiceName", "ChannenId");
        if (result.ErrorCode == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    } catch (Exception e)
    {
        // 例外
    }
}
```

GetChannelInfo()は次のような2つのパラメータを持っています。

| タイプ   | 名前        | 説明               |
|--------|-------------|--------------------|
| String | ServiceName | チャンネル情報をリクエストするサービス      |
| String | channelId   | チャンネル情報をリクエストするチャンネルのID |

レスポンスとしてErrorResult<ResultCodeChannelInfo, Payload>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。GetChannelInfo()が成功すればErrorCodeフィールドの値がResultCodeChannelInfo.CHANNEL_LIST_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。成功時、DataフィールドのPayloadでユーザーが定義したチャンネル情報を取得することもできます。

ResultCodeChannelInfoの詳細は次のとおりです。

| 名前                                 | 値  | 説明                                       |
|--------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                          | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                              | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                         | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗                |
| INVALID_PROTOCOL                     | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| CHANNEL_INFO_SUCCESS                 | 0    | 成功                                       |
| CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1921 | 失敗。チャンネル情報が見つかりません                          |
| CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1922 | 失敗。無効なサービスID                           |
| CHANNEL_INFO_FAIL_INVALID_CHANNEL_ID | 1923 | 失敗。無効なチャンネルID                            |
| CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924 | 失敗。チャンネルが見つかりません                            |

#### GetAllChannelCountInfo

GetAllChannelCountInfo()は、特定サービスの全てのチャンネルに対するカウント情報(ユーザーとルーム数)をリクエストして受け取ることができます。

```c#
public async void AllChannelCountInfo()
{
    try
    {
        ErrorResult<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>> result = await connector.GetAllChannelCountInfo("ServiceName");
        if (result.ErrorCode == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    } catch (Exception e)
    {
        // 例外
    }
}
```

GetAllChannelCountInfo()は次のような1つのパラメータを持っています。

| タイプ   | 名前        | 説明           |
|--------|-------------|----------------|
| String | ServiceName | チャンネル情報をリクエストするサービス |

レスポンスとしてErrorResult<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。GetAllChannelCountInfoが成功すればErrorCodeフィールドの値がResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。Dataフィールドを通じてリクエスト結果であるDictionary<
string, ChannelCountResult>を取得できます。このDictionaryはチャンネルIDをキーに、ChannelCountResultを値として持っています。

ResultCodeAllChannelCountInfoの詳細は次のとおりです。

| 名前                                           | 値  | 説明                                       |
|------------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                                  | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                                      | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                                 | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗                |
| INVALID_PROTOCOL                             | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| ALL_CHANNEL_COUNT_INFO_SUCCESS                 | 0    | 成功                                       |
| ALL_CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO  | 1931 | 失敗。チャンネル情報が見つかりません                          |
| ALL_CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1932 | 失敗。無効なサービスID                           |
| ALL_CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1933 | 失敗。チャンネルが見つかりません                            |

#### GetAllChannelInfo

GetAllChannelInfo()は、特定サービスの全てのチャンネルに対する情報(ユーザー定義)をリクエストして受け取ることができます。

```c#
public async void AllChannelInfo()
{
    try
    {
        ErrorResult<ResultCodeAllChannelInfo, ChannelInfoResult> result = await connector.GetAllChannelInfo("ServiceName");
        if (result.ErrorCode == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    } catch (Exception e)
    {
        // 例外
    }
}
```

GetAllChannelInfo()は次のような1つのパラメータを持っています。

| タイプ   | 名前        | 説明           |
|--------|-------------|----------------|
| String | ServiceName | チャンネル情報をリクエストするサービス |

レスポンスとしてErrorResult<ResultCodeAllChannelInfo, ChannelInfoResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。GetAllChannelInfoが成功すればErrorCodeフィールドの値がResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。Dataフィールドを通じてリクエスト結果であるChannelInfoResultを取得できます。

ResultCodeAllChannelInfoの詳細は次のとおりです。

| 名前                                     | 値  | 説明                                       |
|------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                            | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                                | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                           | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗                |
| INVALID_PROTOCOL                       | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| ALL_CHANNEL_INFO_SUCCESS                 | 0    | 成功                                       |
| ALL_CHANNEL_INFO_FAIL_NO_CHANNEL_INFO  | 1911 | 失敗。チャンネル情報が見つかりません                          |
| ALL_CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1912 | 失敗。無効なサービスID                           |
| ALL_CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1913 | 失敗。チャンネルが見つかりません                            |

ChannelInfoResultのchannelInfoフィールドは、チャンネルIDをキーに、ユーザー定義チャンネル情報を含むPayloadを値として持つDictionary<string, Payload>です。これを利用してチャンネルごとのユーザー定義情報を取得できます。

### 接続終了

Disconnect()メソッドを利用してサーバーとの接続を終了します。

```c#
public async void Disconnect()
{
    try
    {
        await connector.Disconnect();
        // 成功
    } catch (Exception e)
    {
        // 失敗
    }
}
```

別途の戻り値はなく、成功した場合は次のコードを実行し、失敗した場合は例外が発生します。

#### 接続終了通知
Disconnect()を呼び出さなくても、サーバーから強制的に接続を終了したり、ネットワークに問題が発生すると接続が切れることがあり、これに対する通知を受け取ることができます。 
```c#
private void addOnDisconnect()
{
    connector.OnDisconnect += (ResultCodeDisconnect resultCode, Payload payload)=>{
        // 接続切れ通知
    };
}
```

ResultCodeDisconnectパラメータを通じて接続が切れた原因に関する情報を取得でき、サーバーから強制終了された場合はサーバーの実装によって追加情報を取得することもできます。
