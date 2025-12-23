## Game > GameAnvil > Unity 基礎開発ガイド > ユーザーコントローラー

## GameAnvilUser

は、GameAnvilサーバーのGameNodeに関連するタスクを担当します。ログイン(Login())、ログアウト(Logout())、ルーム管理などの基本機能を提供し、直接定義したプロトコルに基づいて、クライアントは自身のユーザーオブジェクトを通じて他のオブジェクトとメッセージをやり取りし、様々なコンテンツを実装できます。

GameAnvilManagerは、基本的に簡単ログインの過程で作成された一つのGameAnvilUserを管理するGameAnvilUserControllerを提供します。LoginResultのUserControllerを通じて取得できます。

### ログイン

ログインは、クライアントがサーバーに接続した後、GameNodeに自身のユーザーオブジェクトを作成するプロセスと定義できます。

ログインは簡易ログインで一度に処理されるため、説明を省略します。詳細は[Unity 応用開発ガイド > ユーザー](../unity-advanced/unity-advanced-03-user.md)を参照してください。

### ログアウト

GameAnvilManagerではゲームサーバーからログアウトし、自動的に接続終了まで処理されます。ログアウトに関するより詳細な説明は[Unity 応用開発ガイド > ユーザー](../unity-advanced/unity-advanced-03-user.md)を参照してください。

### ルームの作成、入室、退室

ルームを利用することで、複数のユーザーからの操作（メッセージ）を、全員で共有された一つの時系列（同期化された流れ）に沿って処理する仕組みを構築できます。つまり、ルーム内では、全てのユーザーのリクエストが、サーバーによって厳密な順序で処理されることが保証されます。もちろん、1人のユーザーのためのルーム作成も、コンテンツによっては意味を持つ場合があります。ルームをどのように使用するかは、完全にエンジンユーザー次第です。

#### CreateRoom

CreateRoom()を呼び出してルームを作成し、そのルームに入室します。

```c#
public async void ManagerCreateRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    
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

レスポンスとしてErrorResult<ResultCodeCreateRoom, CreatedRoomResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。CreateRoomが成功すればErrorCodeフィールドの値がResultCodeCreateRoom.CREATE_ROOM_SUCCESSになり、そうでない場合はルーム生成に失敗したことになります。Dataフィールドを通じてリクエスト結果CreatedRoomResultを取得できます。これを通じて生成されたルームの情報を取得でき、サーバー実装によっては追加情報を取得することもできます。

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
public async void ManagerJoinRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        Payload joinRoomPayload = new Payload(new Protocol.JoinRoomData());
        ErrorResult<ResultCodeJoinRoom, JoinRoomResult> result = await userController.JoinRoom("RoomType", roomId, "MatchingUserCategory", joinRoomPayload);
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
| JOIN_ROOM_FAIL_ROOM_MATCH          | 705 | 失敗。ルームマッチメイキングで問題が発生しました。                          |

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
    GameAnvilUserController userController = gameAnvilManager.UserController;
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
    GameAnvilUserController userController = gameAnvilManager.UserController;

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

レスポンスとしてErrorResult<ResultCodeNamedRoom, NamedRoomResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。NamedRoomが成功すればErrorCodeフィールドの値がResultCodeNamedRoom.NAMED_ROOM_SUCCESSになり、そうでない場合は入室または生成に失敗したことになります。Dataフィールドを通じてリクエスト結果NamedRoomResultを取得できます。これを通じて入室または生成したルームの情報を取得でき、サーバー実装によっては追加情報を取得することもできます。

ResultCodeNamedRoomの詳細は以下の通りです。

| 名前                                 | 値  | 説明                                                              |
|-------------------------------------|-----|------------------------------------------------------------------|
| PARSE_ERROR | -2 | パケットパースエラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。 |
| TIMEOUT | -1 | タイムアウト。リクエストに対するレスポンスが指定時間内にありませんでした。 |
| SYSTEM_ERROR            | 1   | サーバーシステムエラー。サーバーの不明なエラーにより失敗。                                                    |
| INVALID_PROTOCOL | 2 | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| NAMED_ROOM_SUCCESS                  | 0   | 成功。                                                              |
| NAMED_ROOM_FAIL_CONTENT | 701 | 失敗。ユーザーコードで拒否されました。                                                                |
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

GameAnvilは大きく2種類のマッチメイキングを提供します。1つはルーム単位のマッチングを行うルームマッチメイキングで、もう1つはユーザー単位のマッチングを行うユーザーマッチメイキングです。

#### ルームマッチメイキング

ルームマッチメイキングは、条件に合うルームへユーザーを入室させる方式です。ルームマッチメイキングリクエスト時に条件に合うルームがあれば該当ルームへ即時入室させ、条件に合うルームがなければ新しいルームを生成して入室させます。

MatchRoom()を呼び出してルームマッチメイキングをリクエストできます。

```c#
public async void ManagerMatchRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        var matchRoomPayload = new Payload(new Protocol.MatchRoomData());
        ErrorResult<ResultCodeMatchRoom, MatchResult> result = await userController.MatchRoom(true, true, "RoomType", "MatchingGroup", "MatchingUserCategory", matchRoomPayload);
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
| Payload | payload                   | マッチメイキングリクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null)                                                                                                                                                                                                                |
| Payload | leaveRoomPayload | 他のルームに移動する場合、ルームから退出する際に処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeMatchRoom, MatchResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。MatchRoomが成功すればErrorCodeフィールドの値がResultCodeMatchRoom.NAMED_ROOM_SUCCESSになり、そうでない場合は入室または生成に失敗したことになります。Dataフィールドを通じてリクエスト結果MatchResultを取得できます。これを通じて入室または生成したルームの情報を取得でき、サーバー実装によっては追加情報を取得することもできます。

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
| MATCH_ROOM_FAIL_CONTENT                        | 901 | 失敗。ユーザーコードで拒否されました。                                                                 |
| MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 902 | 失敗。ルームが存在しません。 |
| MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM            | 903 | 失敗。すでにルームに入室しています。                                                                 |
| MATCH_ROOM_FAIL_LEAVE_ROOM                     | 904 | 失敗。他のルームへ移動する時、既存のルームからの退出に失敗した場合。                                                 |
| MATCH_ROOM_FAIL_IN_PROGRESS                    | 905 | 失敗。すでにマッチメイキングが進行中の場合。                                                             |
| MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST | 906 | 失敗。条件に合うルームを見つけ、そのルームに参加する途中で、ルームが削除されました。<br/>ルームへの入室処理中、ルーム内の全てのユーザーが退出した場合に発生することがあります。 |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID          | 907 | 失敗。ルームID生成に失敗した場合。                                                             |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM             | 908 | 失敗。ルーム生成に失敗した場合。                                                                |
| MATCH_ROOM_FAIL_INVALID_ROOM_ID                | 909 | 失敗。無効なルームIDが使用された場合。                                                             |
| MATCH_ROOM_FAIL_INVALID_NODE_ID                | 910 | 失敗。無効なノードIDが使用された場合。                                                            |
| MATCH_ROOM_FAIL_INVALID_USER_ID                | 911 | 失敗。無効なユーザーIDが使用された場合。                                                            |
| MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND         | 912 | 失敗。マッチングを進行したが、ルームが見つからなかった場合。                                                         |
| MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY | 913 | 失敗。無効なマッチングユーザーカテゴリーを使用した場合。                                                         |
| MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY   | 914 | 失敗。マッチングエンティティでユーザーカテゴリーサイズが0の場合。                                                         |
| MATCH_ROOM_FAIL_MATCH_FORM_NULL                | 915 | 失敗。マッチング申請書がNULLの場合。                                                              |
| MATCH_ROOM_FAIL_MATCH_INFO_NULL                | 916 | 失敗。マッチング情報がNULLの場合。                                                               |

MatchResultの詳細は次のとおりです。

| タイプ      | 名前      | 説明                |
|----------|----------|--------------------|
| bool     | IsCancel | リクエストがキャンセルされたかどうか。      |
| int | RoomId | 入室したルームのID。 |
| bool | Created | 新しいルームが作成されたかどうか。 |
| String   | RoomName | 入室したルームの名前。         |
| Payload? | payload | クライアントで必要な追加情報。 |

#### ユーザーマッチメイキング

ユーザーマッチメイキングは、ユーザープールを作成し、その中で条件に合うユーザーを探して新しく生成したルームへ入室させる方式です。ユーザープールに条件に合うユーザーの数が不足している場合、マッチメイキングが完了するまで時間がかかることがあり、時間内にマッチメイキングが完了しない場合はタイムアウトとなりマッチングが失敗する可能性があります。

MatchUserStart()を呼び出してユーザーマッチメイキングを開始できます。このリクエストの結果が成功であっても、ユーザーマッチメイキングが完了したわけではありません。単に開始リクエストが成功しただけであり、マッチメイキングの結果は別途のコールバックを通じて伝達されます。

```c#
public async void ManagerMatchUserStart()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    userController.OnMatchUserDone += (GameAnvilUserController userController, ResultCodeMatchUserDone resultCode, MatchResult matchResult) => {
        // マッチング成功
    };
    userController.OnMatchUserTimeout += (GameAnvilUserController userController) =>
    {
        // マッチング失敗
    };
    try
    {
        Payload matchUserPayload = new Payload(new Protocol.MatchUserData());
        ErrorResult<ResultCodeMatchUserStart, Payload> result = await gameAnvilManager.UserController.MatchUserStart("RoomType", "MatchingGroup", matchUserPayload);
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

MatchUserStart()は次のような3つのパラメータを持っています。

| タイプ    | 名前          | 説明                                                                   |
|---------|---------------|------------------------------------------------------------------------|
| String  | roomType      | 生成するルームのタイプ。                                                               |
| String  | matchingGroup | マッチンググループ。同じグループのユーザープールから条件に合うユーザーを探す。使用しない場合はstring.Empty(空文字)を入力 |
| Payload | payload       | ユーザーマッチメイキングリクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null)               |

レスポンスとしてErrorResult<ResultCodeMatchUserStart, Payload>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。MatchUserStartが成功すればErrorCodeフィールドの値がResultCodeMatchUserStart.MATCH_USER_START_SUCCESSになり、そうでない場合はユーザーマッチメイキングに失敗したことになります。サーバー実装によってはDataフィールドのPayloadを通じて追加情報を取得することもできます。

ResultCodeMatchUserStartの詳細は次のとおりです。

| 名前                                      | 値  | 説明                                       |
|-------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                               | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                                   | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                              | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗。               |
| INVALID_PROTOCOL                          | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| MATCH_USER_START_SUCCESS                  | 0    | 成功。                                        |
| MATCH_USER_START_FAIL_CONTENT             | 1101 | 失敗。ユーザーコードで拒否されました。                           |
| MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM | 1102 | 失敗。すでにルームに入室しています。                           |

<br>

ユーザーマッチメイキングが成功した場合、onMatchUserDoneを通じて通知を受け取ることができます。パラメータResultCodeMatchUserDone resultCodeを通じて結果コードを知ることができ、パラメータMatchResult matchResultを通じてマッチングされたルームの情報を取得できます。サーバー実装によってはMatchResultのpayloadを通じて追加情報を取得することもできます。
時間内にマッチングが成功しなかった場合、onMatchUserTimeoutを通じて通知を受け取ることができます。

ResultCodeMatchUserDoneの詳細は次のとおりです。

| 名前                                     | 値  | 説明                                                            |
|------------------------------------------|------|-----------------------------------------------------------------|
| PARSE_ERROR                              | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。                                 |
| TIMEOUT                                  | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。                                         |
| SYSTEM_ERROR                             | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗。                                             |
| INVALID_PROTOCOL                         | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。                                   |
| MATCH_USER_DONE_SUCCESS                  | 0    | 成功。                                                             |
| MATCH_USER_DONE_FAIL_CONTENT             | 1501 | 失敗。ユーザーコードで拒否されました。                                                            |
| MATCH_USER_DONE_FAIL_ROOM_DOES_NOT_EXIST | 1502 | 失敗。条件に合うルームを探してルームに参加させる途中、ルームが消滅しました。                                       |
| MATCH_USER_DONE_FAIL_TRANSFER            | 1503 | 失敗。条件に合うルームを探してルームに参加させる途中、ルームに参加するためのtransfer過程で失敗しました。 |
| MATCH_USER_DONE_FAIL_CREATE_ROOM         | 1504 | 失敗。ルーム生成に失敗しました。                                                                    |

<br>

MatchUserCancel()を呼び出して進行中のユーザーマッチメイキングをキャンセルできます。

```c#
public async void ManagerMatchUserCancel()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        ResultCodeMatchUserCancel result = await userController.MatchUserCancel("RoomType");
        if (result == ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS)
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

レスポンスとしてResultCodeMatchUserCancelを返し、キャンセルが成功すればResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESSになり、そうでない場合はキャンセルが失敗したことになります。ユーザーマッチメイキングが進行中でない場合、すでにユーザーマッチメイキングが成功したか、Timeoutが発生した場合はリクエストが失敗する可能性があります。

ResultCodeMatchUserCancelの詳細は次のとおりです。

| 名前                                       | 値  | 説明                                       |
|--------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                                | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                                    | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                               | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗。               |
| INVALID_PROTOCOL                           | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| MATCH_USER_CANCEL_SUCCESS                  | 0    | 成功。                                        |
| MATCH_USER_CANCEL_FAIL                     | 1201 | 失敗。ユーザーコードで拒否されました。                           |
| MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM | 1202 | 失敗。すでにルームに入室しています。                           |
| MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS     | 1203 | 失敗。ユーザーマッチメイキングが進行中ではありません。                          |

その他、パーティーマッチメイキング、チャンネル、リスナーなどユーザーエージェントの多様な機能に関する詳細は[Unity 応用開発ガイド > ユーザー](../unity-advanced/unity-advanced-03-user.md)を参照してください。
