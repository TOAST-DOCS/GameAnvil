## Game > GameAnvil > Unity 応用開発ガイド > メッセージハンドリング

## メッセージハンドリング

GameAnvilConnector、GameAnvilUserの基本機能以外にも、ユーザーが定義したメッセージをサーバーへ送信できます。

### メッセージ作成及び登録
メッセージを作成して登録する方法は、GameAnvilManagerを使用する場合と同じです。[Unity 基礎開発ガイド > メッセージハンドリング](../unity-basic/unity-basic-06-message-handling.md)で紹介した説明を参照してください。 

### メッセージ送信

#### Request

GameAnvilConnectorではRequest()、GameAnvilUserではRequestUser()を呼び出してメッセージを送信し、レスポンスを受け取ることができます。 

```c#
public async void RequestPacket()
{
    try
    {
        Packet packet = Packet.MakePacket(new Protocol.SampleRequest());
        ErrorResult<ResultCode, Protocol.SampleResponse> result = await connector.Request<Protocol.SampleResponse>(packet);
        if (result.ErrorCode == ResultCode.SUCCESS)
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

Request<\TResponse\>(), RequestUser\<TResponse\>()は、次のように1つの型パラメータと1つのパラメータを持っています。

| タイプ     | 名前         | 説明           |
|----------|--------------|----------------|
| 型パラメータ  | TResponse | レスポンスとして受け取るメッセージタイプ |
| IMessage | message   | サーバーへ送るメッセージ     |

レスポンスとしてErrorResult<ResultCode, TResponse>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。RequestUser()が成功すればErrorCodeフィールドの値がResultCode.SUCCESSになり、そうでない場合はメッセージ送信に失敗したことになります。Dataフィールドを通じて応答メッセージを取得できます。

ResultCodeの詳細は次のとおりです。

| 名前              | 値 | 説明                                       |
|-------------------|----|--------------------------------------------|
| PARSE_ERROR       | -2 | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT           | -1 | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR      | 1  | サーバーシステムエラー。サーバーの不明なエラーにより失敗。               |
| INVALID_PROTOCOL  | 2  | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| HANDLER_NOT_EXIST | 10 | 失敗。サーバーにハンドラがありません。                          |
| HANDLER_ERROR     | 11 | 失敗。サーバーのハンドラで例外が発生しました。                       |
| SUCCESS           | 0  | 成功                                       |

#### SendUser

GameAnvilConnectorではSend()、GameAnvilUserではSendUser()を呼び出してサーバーへメッセージを送信し、別途のレスポンスは待ちません。

```c#
public async void SendMessage()
{
    try
    {
        connector.Send(new Protocol.SampleSend());
    } catch (Exception e)
    {
        // 例外
    }
}
```

Send()、SendUser()は次のように1つのパラメータを持っています。

| タイプ     | 名前    | 説明       |
|----------|---------|------------|
| IMessage | message | サーバーへ送るメッセージ |

#### MessageCallback

Send()、SendUser()で送るメッセージとは関係なく、サーバーから送られるメッセージを受信するためには、SetMessageCallback\<TProtoBuffer\>() を利用してコールバックを登録できます。登録されたコールバックを解除する時は、RemoveMessageCallback\<TProtoBuffer\>()を利用すればよいです。

```c#
public async void MessageCallback()
{
    connector.SetMessageCallback((GameAnvilConnector connector, ResultCode resultCode, Protocol.SampleReceive receive) =>
    {
        return Task.CompletedTask;
    });

    connector.RemoveMessageCallback<Protocol.SampleReceive>();
}
```

SetMessageCallback\<TProtoBuffer\>()は次のように1つの型パラメータと1つのパラメータを持っています。

| タイプ                                                          | 名前         | 説明                       |
|---------------------------------------------------------------|--------------|----------------------------|
| 型パラメータ                                                         | TProtoBuffer | サーバーから受け取るメッセージタイプ           |
| Func<GameAnvilUserController, ResultCode, TProtoBuffer, Task> | callback     | サーバーからメッセージが送られた時に呼び出されるコールバックメソッド |

RemoveMessageCallback\<TProtoBuffer\>()は次のように1つの型パラメータを持っています。

| タイプ    | 名前         | 説明              |
|---------|--------------|-------------------|
| 型パラメータ | TProtoBuffer | 登録したコールバックが受け取るメッセージタイプ    |
