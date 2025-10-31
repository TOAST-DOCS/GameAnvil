## Game > GameAnvil > Unity基礎開発ガイド > User Controller

## GameAnvilUser

は、GameAnvilサーバーのGameNodeに関連するタスクを担当します。ログイン(Login())、ログアウト(Logout())、ルーム管理などの基本機能を提供し、直接定義したプロトコルに基づいて、クライアントは自身のユーザーオブジェクトを通じて他のオブジェクトとメッセージをやり取りし、様々なコンテンツを実装できます。

GameAnvilManagerは、基本的に簡単ログインの過程で作成された一つのGameAnvilUserを管理するGameAnvilUserControllerを提供します。LoginResultのUserControllerを通じて取得できます。

### ログイン

ログインは、クライアントがサーバーに接続した後、GameNodeに自身のユーザーオブジェクトを作成するプロセスと定義できます。

ログインは簡単ログインで一度に処理されるため、説明は省略します。詳細は[Unity詳細開発ガイド > UserAgent](../unity-advanced/unity-advanced-03-user.md)を参照してください。

### ログアウト

GameAnvilManagerでは、ゲームサーバーからログアウトし、自動的に接続終了まで処理されます。ログアウトに関する詳細は、[Unity詳細開発ガイド > UserAgent](../unity-advanced/unity-advanced-03-user.md)を参照してください。

### ルームの作成、入室、退室

ルームを利用することで、複数のユーザーからの操作（メッセージ）を、全員で共有された一つの時系列（同期化された流れ）に沿って処理する仕組みを構築できます。つまり、ルーム内では、全てのユーザーのリクエストが、サーバーによって厳密な順序で処理されることが保証されます。もちろん、1人のユーザーのためのルーム作成も、コンテンツによっては意味を持つ場合があります。ルームをどのように使用するかは、完全にエンジンユーザー次第です。

#### CreateRoom

CreateRoom()を呼び出してルームを作成し、そのルームに入室します。

```c#
public async void ManagerCreateRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userControll = gameAnvilManager.UserController;
    
    try
    {
        Payload createRoomPayload = new Payload(new Protocol.CreateRoomData());
        ErrorResult<ResultCodeCreateRoom, CreatedRoomResult> result = await userControll.CreateRoom("RoomName", "RoomType", "MatchingGroup", createRoomPayload);
        if (result.ErrorCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
        {
            // 成功
        } else
        {
            // 失敗
        }
    }
    catch(Exception e)
    {
        // 例外
    }
}
```

CreateRoom()は、以下の4つのパラメータを持っています。

| タイプ     | 名前           | 説明                                                   |
|---------|---------------|-------------------------------------------------------|
| String | roomName | 作成するルームの名前。使用しない場合はstring.Empty(空文字列)を入力 |
| String | roomType | 作成するルームのタイプ。サーバーに登録されたルームタイプを入力 |
| String | matchingGroup | マッチング時に使用するマッチンググループ名。使用しない場合はstring.Empty(空文字列)を入力 |
| Payload | payload | ルーム作成リクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeCreateRoom, CreatedRoomResult>を返し、ErrorCodeフィールドの値を確認して成功したかどうかを確認できます。CreateRoomが成功すると、ErrorCodeフィールドの値はResultCodeCreateRoom.CREATE_ROOM_SUCCESSとなり、そうでない場合はルームの作成に失敗したことになります。Dataフィールドを通じてリクエスト結果のCreatedRoomResultを取得できます。これにより、作成されたルームの情報を取得でき、サーバーの実装によっては追加の
情報を得ることもできます。

ResultCodeCreateRoomの詳細は以下の通りです。

| 名前                                  | 値  | 説明                                        |
|--------------------------------------|-----|--------------------------------------------|
| PARSE_ERROR | -2 | パケットパースエラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。 |
| TIMEOUT | -1 | タイムアウト。リクエストに対するレスポンスが指定時間内にありませんでした。 |
| SYSTEM_ERROR | 1 | サーバーシステムエラー。サーバーの不明なエラーにより失敗しました。 |
| INVALID_PROTOCOL | 2 | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| CREATE_ROOM_SUCCESS                  | 0   | 成功                                        |
| CREATE_ROOM_FAIL_CONTENT | 601 | 失敗。ユーザーコードで拒否されました。 |
| CREATE_ROOM_FAIL_ALREADY_JOINED_ROOM | 602 | 失敗。既にルームに入室済みです。 |
| CREATE_ROOM_FAIL_CREATE_ROOM_ID | 603 | 失敗。ルームIDの作成に失敗した場合。 |
| CREATE_ROOM_FAIL_CREATE_ROOM | 604 | 失敗。ルームの作成に失敗しました。 |

CreatedRoomResultの詳細は以下の通りです。

| タイプ     | 名前      | 説明               |
|---------|----------|-------------------|
| int | RoomId | 作成したルームのID |
| String? | RoomName | 作成したルームの名前 |
| Payload | payload | クライアントで必要な追加情報 |

#### JoinRoom

JoinRoom()を呼び出して、既に作成されているルームに入室します。

``` c#
GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
GameAnvilUserController userControll = gameAnvilManager.UserController;
try
{
    Payload joinRoomPayload = new Payload(new Protocol.CreateRoomData());
    ErrorResult<ResultCodeJoinRoom, JoinRoomResult> result = await userControll.JoinRoom(managerRoomType, managerRoomId, string.Empty, joinRoomPayload);
    if(result.ErrorCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS)
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
```

JoinRoom()は、以下の4つのパラメータを持っています。

| タイプ     | 名前                  | 説明                                                                                                                                                                                          |
|---------|----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String | roomType | 入室するルームのタイプ。 |
| int | roomId | 入室するルームのID。 |
| String | matchingUserCategory | 入室するルームで使用するmatchingUserCategory。使用しない場合はstring.Empty(空文字列)を入力 <br/>各ルームでは、ルームに属するユーザーをカテゴリに分け、カテゴリごとに人数制限を適用できます。指定したmatchingUserCategoryの現在の人数が最大の場合、JoinRoomは失敗することがあります。 |
| Payload | payload | ルームへの入室リクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeJoinRoom, JoinRoomResult>を返し、ErrorCodeフィールドの値を確認して成功したかどうかを確認できます。JoinRoomが成功すると、ErrorCodeフィールドの値はResultCodeJoinRoom.JOIN_ROOM_SUCCESSとなり、そうでない場合はルームの作成に失敗したことになります。Dataフィールドを通じてリクエスト結果のJoinRoomResultを取得できます。これにより、入室したルームの情報を取得でき、サーバーの実装によっては追加情報を得ることもできます。

ResultCodeJoinRoomの詳細は以下の通りです。

| 名前                                | 値  | 説明                                        |
|------------------------------------|-----|--------------------------------------------|
| PARSE_ERROR | -2 | パケットパースエラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。 |
| TIMEOUT | -1 | タイムアウト。リクエストに対するレスポンスが指定時間内にありませんでした。 |
| SYSTEM_ERROR | 1 | サーバーシステムエラー。サーバーの不明なエラーにより失敗しました。 |
| INVALID_PROTOCOL | 2 | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| JOIN_ROOM_SUCCESS                  | 0   | 成功。                                        |
| JOIN_ROOM_FAIL_CONTENT | 701 | 失敗。ユーザーコードで拒否されました。 |
| JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702 | 失敗。入室をリクエストしたルームが存在しません。 |
| JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM | 703 | 失敗。既にルームに入室済みです。 |
| JOIN_ROOM_FAIL_ALREADY_FULL | 704 | 失敗。入室をリクエストしたルームは満員です。 |
| JOIN_ROOM_FAIL_ROOM_MATCH | 705 | 失敗。ルームマッチメイキングで問題が発生しました。 |

JoinRoomResultの詳細は以下の通りです。

| タイプ     | 名前      | 説明                |
|---------|----------|--------------------|
| int | RoomId | 入室したルームのID。 |
| String? | RoomName | 入室したルームの名前。 |
| Payload | payload | クライアントで必要な追加情報。 |

#### LeaveRoom

LeaveRoom()を呼び出して、入室中のルームから退室できます。

``` c#
public async void ManagerLeaveRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userControll = gameAnvilManager.UserController;
    try
    {
        Payload leaveRoomPayload = new Payload(new Protocol.LeaveRoomData());
        ErrorResult<ResultCodeLeaveRoom, Payload> result = await userControll.LeaveRoom(leaveRoomPayload);
        if (result.ErrorCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
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

LeaveRoom()は、以下の1つのパラメータを持っています。

| タイプ     | 名前     | 説明                                                   |
|---------|---------|-------------------------------------------------------|
| Payload | payload | ルーム退室リクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeLeaveRoom, Payload>を返し、ErrorCodeフィールドの値を確認して成功したかどうかを確認できます。JoinRoomが成功すると、ErrorCodeフィールドの値はResultCodeLeaveRoom.LEAVE_ROOM_SUCCESSとなり、そうでない場合はルームの作成に失敗したことになります。サーバーの実装によっては、DataフィールドのPayloadを通じて追加情報を得ることもできます。

ResultCodeLeaveRoomの詳細は以下の通りです。

| 名前                     | 値  | 説明                                        |
|-------------------------|-----|--------------------------------------------|
| PARSE_ERROR | -2 | パケットパースエラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。 |
| TIMEOUT | -1 | タイムアウト。リクエストに対するレスポンスが指定時間内にありませんでした。 |
| SYSTEM_ERROR | 1 | サーバーシステムエラー。サーバーの不明なエラーにより失敗しました。 |
| INVALID_PROTOCOL | 2 | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| LEAVE_ROOM_SUCCESS      | 0   | 成功。                                        |
| LEAVE_ROOM_FAIL_CONTENT | 801 | 失敗。ユーザーコードで拒否されました。 |

#### NamedRoom

NamedRoom()を呼び出して、指定した名前のルームに入室できます。指定した名前のルームがない場合は、ルームを作成してからそのルームに入室します。

```c#
public async void ManagerNamedRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userControll = gameAnvilManager.UserController;

    try
    {
        bool isParty = false;
        Payload namedRoomPayload = new Payload(new Protocol.NamedRoomData());
        ErrorResult<ResultCodeNamedRoom, NamedRoomResult> result = await userControll.NamedRoom("RoomName", "RoomType", isParty, namedRoomPayload);
        if (result.ErrorCode == ResultCodeNamedRoom.NAMED_ROOM_SUCCESS)
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

NamedRoom()は、以下の4つのパラメータを持っています。

| タイプ     | 名前      | 説明                                                                                      |
|---------|----------|------------------------------------------------------------------------------------------|
| String | roomType | 入室または作成するルームのタイプ |
| String | roomName | 入室または作成するルームの名前 |
| bool | isParty | パーティマッチメイキング用のルームかどうか。<br/>同じパーティに属するユーザーがパーティマッチメイキングが完了するまで一緒に待機するためのルームを作成する場合、trueと入力します。 |
| Payload | payload | 入室または作成リクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeNamedRoom, NamedRoomResult>を返し、ErrorCodeフィールドの値を確認して成功したかどうかを確認できます。NamedRoomが成功すると、ErrorCodeフィールドの値はResultCodeNamedRoom.NAMED_ROOM_SUCCESSとなり、そうでない場合は入室または作成に失敗したことになります。Dataフィールドを通じてリクエスト結果のNamedRoomResultを取得できます。これにより、入室または作成したルームの情報を取得でき、サーバーの実装によっては追加の
情報を得ることもできます。

ResultCodeNamedRoomの詳細は以下の通りです。

| 名前                                 | 値  | 説明                                                              |
|-------------------------------------|-----|------------------------------------------------------------------|
| PARSE_ERROR | -2 | パケットパースエラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。 |
| TIMEOUT | -1 | タイムアウト。リクエストに対するレスポンスが指定時間内にありませんでした。 |
| SYSTEM_ERROR | 1 | サーバーシステムエラー。サーバーの不明なエラーにより失敗しました。 |
| INVALID_PROTOCOL | 2 | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| NAMED_ROOM_SUCCESS                  | 0   | 成功                                                              |
| NAMED_ROOM_FAIL_CONTENT | 701 | 失敗。ユーザーコードで拒否されました。 |
| NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702 | 失敗。ルームが存在しません。<br/>ルームへの入室処理中、ルーム内の全てのユーザーが退出した場合に発生することがあります。 |
| NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM | 703 | 失敗。既にルームに入室済みです。 |
| NAMED_ROOM_FAIL_INVALID_ROOM_NAME | 704 | 失敗。不正なルーム名をリクエストしました。 |
| NAMED_ROOM_FAIL_CREATE_ROOM | 705 | 失敗。ルームの作成に失敗しました。 |

NamedRoomResultの詳細は以下の通りです。

| タイプ     | 名前      | 説明                |
|---------|----------|--------------------|
| bool | Created | 新しいルームが作成されたかどうか |
| int | RoomId | 入室したルームのID |
| String? | RoomName | 入室したルームの名前 |
| Payload | payload | クライアントで必要な追加情報。 |

### マッチメイキング

GameAnvilは、大きく2つのマッチメイキングを提供します。一つはルーム単位のマッチングを行うルームマッチメイキング、もう一つはユーザー単位のマッチングを行うユーザーマッチメイキングです。

#### ルームマッチメイキング

ルームマッチメイキングは、条件に合うルームにユーザーを入室させる方式です。ルームマッチメイキングをリクエストした際に、条件に合うルームがあればそのルームにすぐに入室させ、条件に合うルームがなければ新しいルームを作成して入室させます。

MatchRoom()を呼び出して、ルームマッチメイキングをリクエストできます。

```c#
public async void ManagerMatchRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userControll = gameAnvilManager.UserController;
    try
    {
        var matchRoomPayload = new Payload(new Protocol.MatchRoomData());
        ErrorResult<ResultCodeMatchRoom, MatchResult> result = await userControll.MatchRoom(false, false, managerRoomType, ConstantManager.channelId, string.Empty, matchRoomPayload);
        if (result.ErrorCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
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

MatchRoom()は、以下の7つのパラメータを持っています。

| タイプ     | 名前                       | 説明                                                                                                                              |
|---------|---------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| bool | isCreateRoomIfNotJoinRoom | 条件に合うルームが見つからなかった場合に、ルームを作成して入室するかどうか。<br/>true : ルームがなければ作成する。<br/>false : ルームがなければ失敗として処理する。 |
| bool | isMoveRoomIfJoinedRoom | 既にルームに入室している場合に、別のルームに移動するかどうか。<br/>true : ルームに入室している状態であればルームを移動する。<br/>false : ルームに入室している状態でマッチングをリクエストした場合は失敗として処理する。 |
| string | roomType | ルームタイプ。同じタイプのルームを探します。 |
| string | matchingGroup | マッチンググループ。同じグループで作成されたルームを探します。 |
| string | matchingUserCategory | マッチングされたルームで使用するユーザーカテゴリ。<br/>各ルームでは、ルームに属するユーザーをカテゴリに分け、カテゴリごとに人数制限を適用できます。指定したmatchingUserCategoryの現在の人数が最大でないルームを探します。 |
| Payload | payload | マッチメイキングリクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null) |
| Payload | leaveRoomPayload | 他のルームに移動する場合、ルームから退出する際に処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeMatchRoom, MatchResult>を返し、ErrorCodeフィールドの値を確認して成功したかどうかを確認できます。MatchRoomが成功すると、ErrorCodeフィールドの値はResultCodeMatchRoom.NAMED_ROOM_SUCCESSとなり、そうでない場合は入室または作成に失敗したことになります。Dataフィールドを通じてリクエスト結果のMatchResultを取得できます。これにより、入室または作成したルームの情報を取得でき、サーバーの実装によっては追加の
情報を得ることもできます。

ResultCodeMatchRoomの詳細は以下の通りです。

| 名前                                            | 値  | 説明                                                                                    |
|------------------------------------------------|-----|----------------------------------------------------------------------------------------|
| PARSE_ERROR | -2 | パケットパースエラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。 |
| TIMEOUT | -1 | タイムアウト。リクエストに対するレスポンスが指定時間内にありませんでした。 |
| SYSTEM_ERROR | 1 | サーバーシステムエラー。サーバーの不明なエラーにより失敗しました。 |
| INVALID_PROTOCOL | 2 | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| NAMED_ROOM_SUCCESS                             | 0   | 成功                                                                                    |
| NAMED_ROOM_FAIL_CONTENT | 701 | 失敗。ユーザーコードで拒否されました。 |
| NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702 | 失敗。ルームへの入室処理中に、ルームが削除されました。 |
| NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM | 703 | 失敗。既にルームに入室済みです。 |
| NAMED_ROOM_FAIL_INVALID_ROOM_NAME | 704 | 失敗。不正なルーム名をリクエストしました。 |
| NAMED_ROOM_FAIL_CREATE_ROOM | 705 | 失敗。ルームの作成に失敗しました。 |
| MATCH_ROOM_SUCCESS                             | 0   | 成功                                                                                    |
| MATCH_ROOM_FAIL_CONTENT | 901 | 失敗。ユーザーコードで拒否されました。 |
| MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 902 | 失敗。ルームが存在しません。 |
| MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM | 903 | 失敗。既にルームに入室済みです。 |
| MATCH_ROOM_FAIL_LEAVE_ROOM | 904 | 失敗。別のルームに移動する際、既存のルームからの退出に失敗した場合 |
| MATCH_ROOM_FAIL_IN_PROGRESS                    | 905 | 失敗。既にマッチメイキングが進行中の場合                                                                |
| MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST | 906 | 失敗。条件に合うルームを見つけ、そのルームに参加する途中で、ルームが削除されました。<br/>ルームへの入室処理中、ルーム内の全てのユーザーが退出した場合に発生することがあります。 |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID          | 907 | 失敗。 ルームIDの作成に失敗した場合                                                                |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM             | 908 | 失敗。 ルームの作成に失敗した場合                                                                    |
| MATCH_ROOM_FAIL_INVALID_ROOM_ID                | 909 | 失敗。 不正なルームIDが使用された場合                                                                |
| MATCH_ROOM_FAIL_INVALID_NODE_ID | 910 | 失敗。不正なノードIDが使用された場合 |
| MATCH_ROOM_FAIL_INVALID_USER_ID | 911 | 失敗。不正なユーザーIDが使用された場合 |
| MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND | 912 | 失敗。マッチングを行ったが、ルームが見つからなかった場合 |
| MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY | 913 | 失敗。不正なマッチングユーザーカテゴリが使用された場合。 |
| MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY | 914 | 失敗。マッチングルームでユーザーカテゴリのサイズが0の場合 |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL | 915 | 失敗。マッチング申込書がNULLの場合 |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL | 916 | 失敗。マッチング情報がNULLの場合 |

MatchResultの詳細は以下の通りです。

| タイプ      | 名前      | 説明                |
|----------|----------|--------------------|
| bool | IsCancel | リクエストがキャンセルされたかどうか。 |
| int | RoomId | 入室したルームのID。 |
| bool | Created | 新しいルームが作成されたかどうか。 |
| String   | RoomName | 入室したルームの名前。         |
| Payload? | payload | クライアントで必要な追加情報。 |

#### ユーザーマッチメイキング

ユーザーマッチメイキングは、ユーザープールを作成し、その中から条件に合うユーザーを探して新しく作成したルームに入室させる方式です。ユーザープールに条件に合うユーザーの数が足りない場合、マッチメイキングが完了するまでに時間がかかることがあり、時間内にマッチメイキングが完了しないと、タイムアウトでマッチングが失敗することがあります。

MatchUserStart()を呼び出して、ユーザーマッチメイキングを開始できます。既にルームに入室している場合など、サーバーの条件によってはリクエストが失敗することがあります。

```c#
public async void ManagerMatchUserStart()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userControll = gameAnvilManager.UserController;
    userControll.onMatchUserDoneSuccess.AddListener((MatchResult matchResult) => {
        // マッチング成功
    });
    userControll.onMatchUserTimeOut.AddListener((MatchResult matchResult) =>
    {
        // マッチング失敗
    });
    try
    {
        ErrorResult<ResultCodeMatchUserStart, Payload> result = await gameAnvilManager.UserController.MatchUserStart(managerRoomType, "", new Payload(roomOption));
        if(result.ErrorCode == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
        {  
            // リクエスト成功
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

<br>


You can be notified via onMatchUserDoneListeners or IUserListener.OnMatchUserDone if the match was successful, or via onMatchUserTimeoutListeners or IUserListener.OnMatchUserTimeout if the match was not successful in time.

```c#
/// <summary>
/// Representative to receive user matching results
/// </summary>
userAgent.onMatchUserDoneListeners += (UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload) => {
    /// <param name="userAgent">The user agent that requested MatchUserStart() or MatchPartyStart()</param>.
    /// <param name="result">Result of the MatchUserStart() or MatchPartyStart() request</param>
    /// <param name="created">Whether a room was created</param>
    /// <param name="roomId">Id of the matched room</param>
    /// <param name="payload">Additional information received from the server</param>
};

/// <summary>
/// User matching timeout notification proxy
/// </summary>
userAgent.onMatchUserTimeoutListeners += (UserAgent userAgent) => {
    /// <param name="userAgent">The user agent that requested MatchUserStart() or MatchPartyStart()</param>.
};
```
<br>

You can cancel user matchmaking by calling MatchUserCancel(). If you are not in the middle of requesting a user match, the cancel request might fail if user matching has already succeeded or a timeout has occurred.

```c#
/// <summary>
/// Cancel a user match request<para></para>
/// If not in the middle of a match request, fails if a match has already succeeded or a timeout has occurred<para></para>
/// </summary>
/// <param name="roomType">The type of bar requested to be matched</param>
/// <param name="onMatchUserCancel">The agent to cancel the result</param>.
userAgent.MatchUserCancel(Constants.RoomType, (UserAgent user, Defines.ResultCodeMatchUserCancel result) => {
    /// <param name="userAgent">The user agent that requested MatchUserCancel()</param>
    /// <param name="result">Result of the MatchUserCancel() request</param>
	if(result == Defines.ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS){
        // Matchmaking cancel success
    } else {
        // Failed to cancel matchmaking
    }
});
```

Other features of the User Agent, such as party matchmaking, channels, listeners, and more, are described in the [Unity Advanced Development Guide > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md).
