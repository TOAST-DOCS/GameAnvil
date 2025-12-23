## Game > GameAnvil > Unity 基礎開発ガイド > メッセージハンドリング

## メッセージハンドリング

GameAnvilUserControllerのRequestUser()とSendUser()メソッドを利用して、ユーザーが定義したメッセージをサーバーへ送信できます。メッセージを送信するためには、メッセージを生成して登録する過程が必要です。

### メッセージ生成

GameAnvilは基本メッセージプロトコルとして[ProtocolBuffers](https://developers.google.com/protocol-buffers/docs/proto3)を使用します。.protoファイルにメッセージを定義し、protocコンパイラで実際のクラスソースコードを生成することになります。生成されたソースコードをプロジェクトに追加して使用できます。protocに関する詳細な説明は[こちら](https://developers.google.com/protocol-buffers/docs/proto3#generating)を参照してください。

次にメッセージを作成するために、Assetsフォルダの下にprotocolsフォルダを作成し、次のようにmessages.protoファイルを作成します。

```protobuf
// messages.proto
syntax = "proto3";

package Messages;

message SampleRequest
{
  string msg = 1;
}

message SampleResponse
{
  repeated string msgs = 1;
}

message SampleSend
{
  string msg = 1;
}

message SampleReceive
{
  repeated string msgs = 1;
}
```

そしてWindowsコマンドプロンプト(cmd)でprotocを利用してメッセージを生成します。

```
/protoc --csharp_out=./ messages.proto
```

### メッセージ登録

新しく生成したメッセージを使用するには、使用するメッセージをProtocolManagerにあらかじめ登録する必要があります。あらかじめ登録しないと、動作しなかったり誤作動したり、例外が発生する可能性があります。

```c#
GameAnvilProtocolManager.RegisterProtocol(Messages.MessagesReflection.Descriptor);
```

### メッセージ送信

#### RequestUser

RequestUser()でメッセージを送信し、レスポンスを受け取ることができます。

```c#
public async void ManagerRequest()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        var (resultCode, response) = await userController.RequestUser<Protocol.SampleResponse>(new Protocol.SampleRequest());
        if (resultCode == ResultCode.SUCCESS)
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

RequestUser\<TProtoBuffer\>()は次のように1つの型パラメータと1つのパラメータを持っています。

| タイプ     | 名前         | 説明           |
|----------|--------------|----------------|
| 型パラメータ | TProtoBuffer | レスポンスとして受け取るメッセージタイプ |
| IMessage | message   | サーバーへ送るメッセージ     |

レスポンスとしてErrorResult<ResultCode, TProtoBuffer>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。RequestUser()が成功すればErrorCodeフィールドの値がResultCode.SUCCESSになり、そうでない場合はメッセージ送信に失敗したことになります。Dataフィールドを通じて応答メッセージを取得できます。

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

SendUser()でメッセージを送信すると、SendUser()の呼び出し直後にサーバーへ送信され、別途のレスポンスは待ちません。

```c#
public async void ManagerSend()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        userController.SendUser(new Protocol.SampleSend());
    } catch (Exception e)
    {
        // 例外
    }
}
```

SendUser()は次のように1つのパラメータを持っています。

| タイプ     | 名前    | 説明       |
|----------|---------|------------|
| IMessage | message | サーバーへ送るメッセージ |

#### MessageCallback

SendUser()で送るメッセージとは関係なく、サーバーから送られるメッセージを受信するためには、SetMessageCallback<TProtoBuffer>()を利用してコールバックを登録できます。登録されたコールバックを解除する時は、RemoveMessageCallback<TProtoBuffer>()を利用すればよいです。

```c#
public async void ManagerMessageCallback()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    userController.SetMessageCallback((GameAnvilUserController user, ResultCode resultCode, Protocol.SampleReceive receive) =>
    {
        return Task.CompletedTask;
    });
    
    userController.RemoveMessageCallback<Protocol.SampleReceive>();
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
