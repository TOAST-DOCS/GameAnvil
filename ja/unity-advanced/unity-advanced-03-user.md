## Game > GameAnvil > Unity 応用開発ガイド > ユーザー

## ユーザー

GameAnvilUserは、GameAnvilサーバーのユーザーに関連する作業を担当します。ログイン(Login())、ログアウト(Logout())及びルーム管理などの基本機能を提供します。
GameAnvilサーバーは複数のサービスを同時に運営でき、1つのGameAnvilUserは1つのサービスにログインして互いに独立して動作します。つまり、複数のGameAnvilUserを作成してそれぞれ異なるサービスにログインし、同時に使用することが可能です。SubIdを異なるものにすれば、同じサービスに複数のGameAnvilUserを同時にログインさせて使用することも可能です。

### 生成

GameAnvilUserを使用するためには、まずGameAnvilUserオブジェクトを生成する必要があります。

```c#
public void CreateUser()
{
    try
    {
        user = new GameAnvilUser(connector, serviceName, subId);
        // 成功
    }
    catch(Exception e)
    {
        // 失敗
    }
}
```

ServiceNameとSubIdで区別される複数のGameAnvilUserを生成できます。

```c#
public void CreateUsers()
{
    try
    {
        gameUser1 = new GameAnvilUser(connector, "GameService", 1);
        gameUser2 = new GameAnvilUser(connector, "GameService", 1);
        chatUser = new GameAnvilUser(connector, "ChatService", 1);
        // 成功
    } catch (Exception e)
    {
        // 失敗
    }
}
```

### 解除

使用を完了したGameAnvilUserは、Dispose()を呼び出して解除する必要があります。

```c#
public void DisposeUser()
{
    try
    {
        user.Dispose();
    } catch (Exception e)
    {
        // 失敗
    }
}
```

### ログイン/ログアウト

ログインは、クライアントがサーバーに接続した後、GameNodeに自身のユーザーオブジェクトを作成する過程と定義できます。ログアウトはログインの反対の概念です。つまり、GameNode上で自身のユーザーオブジェクトを削除する過程です。

#### ログイン

Login()を呼び出してサービスにログインします。ログイン時、どのUserTypeでどのチャンネルにログインするかを入力する必要があります。追加情報が必要な場合はPayloadに入れて送ることができます。

```c#
public async void Login()
{
    try
    {
        Payload loginPayload = new Payload(new Protocol.LoginData());
        ErrorResult<ResultCodeLogin, LoginResult> result = await user.Login("UserType", "ChannelId", loginPayload);
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

Login()は次のような4つのパラメータを持っています。

| タイプ    | 名前           | 説明                                                 |
|---------|----------------|------------------------------------------------------|
| String  | userType       | ログインして生成するユーザーのタイプ。                                 |
| String  | channelId      | ログインするチャンネルのID。                                    |
| Payload | requestPayload | ログインリクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeLogin, LoginResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。Loginが成功すればErrorCodeフィールドの値がResultCodeLogin.LOGIN_SUCCESSになり、そうでない場合はログインに失敗したことになります。Dataフィールドを通じてリクエスト結果LoginResultを取得できます。これを通じてログインされたユーザー情報を取得でき、サーバー実装によっては追加情報を取得することもできます。

ResultCodeLoginの詳細は次のとおりです。

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

LoginResultの詳細は次のとおりです。

| タイプ    | 名前         | 説明            |
|---------|--------------|-----------------|
| int     | UserId       | ログインしたユーザーのID    |
| string  | UserType     | ログインしたユーザーのタイプ     |
| string  | ServiceName  | ログインしたサービスの名前    |
| string  | ChannelId    | ログインしたチャンネルのID    |
| Payload | Payload      | サーバーから受け取った追加情報   |
| bool    | IsRelogined  | 再ログイン有無         |
| bool    | IsJoinedRoom | ルームに属しているかどうか     |
| int     | RoomId       | ユーザーが属するルームのID   |
| string  | RoomName     | ユーザーが属するルームの名前    |
| Payload | RoomPayload  | ユーザーが属するルームの追加情報  |
| bool    | IsMatching   | マッチングをリクエストした状態かどうか |

### ログアウト

Logout()を呼び出してサービスからログアウトします。

```c#
public async void Logout()
{
    try
    {
        Payload logoutPayload = new Payload(new Protocol.LogoutData());
        ErrorResult<ResultCodeLogout, LogoutResult> result = await user.Logout(logoutPayload);
        if (result.ErrorCode == ResultCodeLogout.LOGOUT_SUCCESS)
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
Logout()は次のような1つのパラメータを持っています。

| タイプ     | 名前           | 説明                                                    |
|----------|----------------|---------------------------------------------------------|
| Payload? | payload | ログアウトリクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeLogout, LogoutResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。Logoutが成功すればErrorCodeフィールドの値がResultCodeLogout.LOGOUT_SUCCESSになり、そうでない場合はログアウトに失敗したことになります。Dataフィールドを通じてリクエスト結果LogoutResultを取得できます。サーバー実装によってはLogoutResultのPayloadフィールドを通じて追加情報を取得することもできます。

ResultCodeLogoutの詳細は次のとおりです。

| 名前                | 値 | 説明                                       |
|---------------------|-----|--------------------------------------------|
| PARSE_ERROR         | -2  | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT             | -1  | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR        | 1   | サーバーシステムエラー。サーバーの不明なエラーにより失敗。               |
| INVALID_PROTOCOL    | 2   | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| LOGOUT_SUCCESS      | 0   | 成功。                                        |
| LOGOUT_FAIL_CONTENT | 401 | 失敗。ユーザーコードで拒否されました。                              |

#### 強制ログアウト通知
Logout()を呼び出さなくても、サーバーからユーザーを強制的にログアウトさせることができます。この場合、OnForceLogoutを通じてこれに対する通知を受け取ることができます。
```c#
public void AddOnLogout()
{
    user.OnForceLogout += (GameAnvilUser, Payload) =>
    {
        // 強制ログアウト通知
    };
}
```
サーバーの実装によっては、パラメータPayload payloadを通じて追加情報を取得することもできます。

### ルームの作成、入室、退室

ルームを利用することで、複数のユーザーからの操作（メッセージ）を、全員で共有された一つの時系列（同期化された流れ）に沿って処理する仕組みを構築できます。つまり、ルーム内では、全てのユーザーのリクエストが、サーバーによって厳密な順序で処理されることが保証されます。もちろん、1人のユーザーのためのルーム作成も、コンテンツによっては意味を持つ場合があります。ルームをどのように使用するかは、完全にエンジンユーザー次第です。

#### ルーム生成

CreateRoom()を呼び出してルームを作成し、そのルームに入室します。

```c#
public async void CreateRoom()
{
    try
    {
        Payload createRoomPayload = new Payload(new Protocol.CreateRoomData());
        ErrorResult<ResultCodeCreateRoom, CreatedRoomResult> result = await user.CreateRoom("RoomName", "RoomType", "MatchingGroup", createRoomPayload);
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

#### ルーム入室

JoinRoom()を呼び出して、既に作成されているルームに入室します。

``` c#
public async void JoinRoom()
{
    try
    {
        Payload joinRoomPayload = new Payload(new Protocol.JoinRoomData());
        ErrorResult<ResultCodeJoinRoom, JoinRoomResult> result = await user.JoinRoom("RoomType", roomId, "MatchingUserCategory", joinRoomPayload);
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

レスポンスとしてErrorResult<ResultCodeJoinRoom, JoinRoomResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。JoinRoomが成功すればErrorCodeフィールドの値がResultCodeJoinRoom.JOIN_ROOM_SUCCESSになり、そうでない場合はルーム入室に失敗したことになります。Dataフィールドを通じてリクエスト結果JoinRoomResultを取得できます。これを通じて入室したルームの情報を取得でき、サーバー実装によっては追加情報を取得することもできます。

ResultCodeJoinRoomの詳細は次のとおりです。

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

#### ルーム退場

LeaveRoom()を呼び出して、入室中のルームから退室できます。

``` c#
public async void LeaveRoom()
{
    try
    {
        Payload leaveRoomPayload = new Payload(new Protocol.LeaveRoomData());
        ErrorResult<ResultCodeLeaveRoom, Payload> result = await user.LeaveRoom(leaveRoomPayload);
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

レスポンスとしてErrorResult<ResultCodeLeaveRoom, Payload>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。LeaveRoomが成功すればErrorCodeフィールドの値がResultCodeLeaveRoom.LEAVE_ROOM_SUCCESSになり、そうでない場合はルーム退場に失敗したことになります。サーバー実装によってはDataフィールドのPayloadを通じて追加情報を取得することもできます。

ResultCodeLeaveRoomの詳細は次のとおりです。

| 名前                     | 値  | 説明                                        |
|-------------------------|-----|--------------------------------------------|
| PARSE_ERROR | -2 | パケットパースエラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。 |
| TIMEOUT | -1 | タイムアウト。リクエストに対するレスポンスが指定時間内にありませんでした。 |
| SYSTEM_ERROR | 1 | サーバーシステムエラー。サーバーの不明なエラーにより失敗しました。 |
| INVALID_PROTOCOL | 2 | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| LEAVE_ROOM_SUCCESS      | 0   | 成功。                                        |
| LEAVE_ROOM_FAIL_CONTENT | 801 | 失敗。ユーザーコードで拒否されました。 |

#### ルーム強制退場通知
LeaveRoom()を呼び出さなくても、サーバーから強制的にルームから退場させることができます。この場合、OnForceLeaveRoomを通じてこれに対する通知を受け取ることができます。
```c#
public void AddOnLeaveRoom()
{
    user.OnForceLeaveRoom += (GameAnvilUser gameAnvilUser, int roomId, Payload payload) =>
    {
        // ルーム強制退場通知
    };
}
```
パラメータint roomIdを通じてどのルームから強制退場させられたかを知ることができ、サーバーの実装によってはパラメータPayload payloadを通じて追加情報を取得することもできます。 

#### 指定した名前のルームに入室

NamedRoom()を呼び出して、指定した名前のルームに入室できます。指定した名前のルームがない場合は、ルームを作成してからそのルームに入室します。

```c#
public async void NamedRoom()
{
    try
    {
        bool isParty = false;
        Payload namedRoomPayload = new Payload(new Protocol.NamedRoomData());
        ErrorResult<ResultCodeNamedRoom, NamedRoomResult> result = await user.NamedRoom("RoomName", "RoomType", isParty, namedRoomPayload);
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

ResultCodeNamedRoomの詳細は次のとおりです。

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

NamedRoomResultの詳細は次のとおりです。

| タイプ     | 名前      | 説明                |
|---------|----------|--------------------|
| bool | Created | 新しいルームが作成されたかどうか |
| int | RoomId | 入室したルームのID |
| String? | RoomName | 入室したルームの名前 |
| Payload | payload | クライアントで必要な追加情報。 |

### マッチメイキング

GameAnvilは2種類のマッチメイキングを提供します。1つはルーム単位のマッチングを行うルームマッチメイキングで、もう1つはユーザー単位のマッチングを行うユーザーマッチメイキングです。

#### ルームマッチメイキング

ルームマッチメイキングは、条件に合うルームへユーザーを入室させる方式です。ルームマッチメイキングリクエスト時に条件に合うルームがあれば該当ルームへ即時入室させ、条件に合うルームがなければ新しいルームを生成して入室させます。

MatchRoom()を呼び出してルームマッチメイキングをリクエストできます。

```c#
public async void MatchRoom()
{
    try
    {
        var matchRoomPayload = new Payload(new Protocol.MatchRoomData());
        ErrorResult<ResultCodeMatchRoom, MatchResult> result = await user.MatchRoom(true, true, "RoomType", "MatchingGroup", "MatchingUserCategory", matchRoomPayload);
        if (result.ErrorCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
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

MatchRoom()は、以下の7つのパラメータを持っています。

| タイプ    | 名前                      | 説明                                                                                                                            |
|---------|---------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| bool    | isCreateRoomIfNotJoinRoom | 条件に合うルームが見つからなかった時、ルームを生成して入室するかどうか。<br/>true : ルームがなければ生成する。<br/>false : ルームがなければ失敗処理する。                                                                                                                                                            |
| bool    | isMoveRoomIfJoinedRoom    | すでにルームに入室している状態の場合、他のルームへ移動するかどうか。<br/>true : ルームに入室している状態であればルームを移動する。<br/>false : ルームに入室している状態でマッチングをリクエストした場合は失敗処理する。                                                                                                                                                 |
| string  | roomType                  | ルームタイプ。同じタイプのルームを探す。                                                                                                                                                                                                                                                         |
| string  | matchingGroup             | マッチンググループ。同じグループで生成されたルームを探す。                                                                                                                                                                                                                                                        |
| string  | matchingUserCategory      | マッチングされたルームで使用するユーザーカテゴリー。<br/>各ルームではルームに属するユーザーをカテゴリーに分け、カテゴリーごとに人数制限を適用できる。<br/>指定したmatchingUserCategoryの現在人数が最大ではないルームを探す。 |
| Payload | payload                   | マッチメイキングリクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null)                                                                                                                                                                                                                |
| Payload | leaveRoomPayload          | 他のルームへ移動する場合、ルームを出る時に処理するサーバーのユーザーコードで必要な追加情報。(default = null)                                                                                                                                                                                               |

レスポンスとしてErrorResult<ResultCodeMatchRoom, MatchResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。MatchRoomが成功すればErrorCodeフィールドの値がResultCodeMatchRoom.MATCH_ROOM_SUCCESSになり、そうでない場合はルームマッチメイキングに失敗したことになります。Dataフィールドを通じてリクエスト結果MatchResultを取得できます。これを通じてマッチングされたルームの情報を取得でき、サーバー実装によっては追加情報を取得することも
できます。

ResultCodeMatchRoomの詳細は次のとおりです。

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
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL      | 915 | 失敗。マッチング申請書がNULLの場合。                                                              |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL      | 916 | 失敗。マッチング情報がNULLの場合。                                                               |

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
public async void MatchUserStart()
{
    user.OnMatchUserDone +=(GameAnvilUser user, ResultCodeMatchUserDone resultCode, MatchResult matchResult) => {
        // マッチング成功
    };
    user.onMatchUserTimeOut +=(GameAnvilUser user, ResultCodeMatchUserTimeOut resultCode) =>
    {
        // マッチング失敗
    };
    try
    {
        Payload matchUserPayload = new Payload(new Protocol.MatchUserData());
        ErrorResult<ResultCodeMatchUserStart, Payload> result = await user.MatchUserStart("RoomType", "MatchingGroup", matchUserPayload);
        if (result.ErrorCode == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
        {
            // リクエスト成功
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

MatchUserStart()は次のような3つのパラメータを持っています。

| タイプ    | 名前          | 説明                                                                   |
|---------|---------------|------------------------------------------------------------------------|
| String  | roomType      | 生成するルームのタイプ。                                                               |
| String  | matchingGroup | マッチンググループ。同じグループのユーザープールから条件に合うユーザーを探す。 |
| Payload | payload       | ユーザーマッチメイキングリクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null)               |

レスポンスとしてErrorResult<ResultCodeMatchUserStart, Payload>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。MatchUserStartが成功すればErrorCodeフィールドの値がResultCodeMatchUserStart.MATCH_USER_START_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。サーバー実装によってはDataフィールドのPayloadを通じて追加情報を取得することもできます。

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
public async void MatchUserCancel()
{
    try
    {
        ResultCodeMatchUserCancel result = await user.MatchUserCancel("RoomType");
        if (result == ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS)
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

#### パーティーマッチメイキング

パーティーマッチメイキングはユーザーマッチメイキングの特殊な形態で、2人以上のユーザーが1つのパーティーとしてまとめられてユーザープールに登録され、条件に合う他のユーザーを探して新しく生成したルームへ一緒に入室させる方式です。パーティーとしてまとめられたユーザーは常に同じルームに入室します。パーティー以外に一緒にマッチングされたユーザーは、サーバーのマッチメーカー実装によって別のパーティーの場合もあれば、個人の場合もあります。

パーティーマッチメイキングを行うためには、まずNamedRoom()を呼び出す必要があります。NamedRoom()呼び出し時にisPartyパラメータをtrueとして渡すと、該当NamedRoomがパーティーの役割を果たします。NamedRoomにパーティーユーザーを全て集めてからパーティーマッチメイキングを開始します。

```c#
public async void PartyRoom()
{
    try
    {
        bool isParty = true;
        Payload partyRoomPayload = new Payload(new Protocol.PartyRoomData());
        ErrorResult<ResultCodeNamedRoom, NamedRoomResult> result = await user.NamedRoom("RoomName", "RoomType", isParty, partyRoomPayload);
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

<br>

MatchPartyStart()を呼び出してパーティーマッチメイキングを開始できます。このリクエストの結果が成功であっても、パーティーマッチメイキングが完了したわけではありません。単に開始リクエストが成功しただけであり、マッチメイキングの結果は別途のコールバックを通じて伝達されます。

```c#
public async void MatchPartyStart()
{
    try
    {
        Payload partyRoomPayload = new Payload(new Protocol.PartyRoomData());
        ErrorResult<ResultCodeMatchPartyStart, Payload> result = await user.MatchPartyStart("RoomType", "MatchingGroup", partyRoomPayload);
        if (result.ErrorCode == ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS)
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

MatchPartyStart()は次のような3つのパラメータを持っています。

| タイプ    | 名前          | 説明                                                                   |
|---------|---------------|------------------------------------------------------------------------|
| String  | roomType      | 生成するルームのタイプ。                                                               |
| String  | matchingGroup | マッチンググループ。同じグループのユーザープールから条件に合うユーザーを探す。使用しない場合はstring.Empty(空文字)を入力 |
| Payload | payload       | パーティーマッチメイキングリクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null)               |

レスポンスとしてErrorResult<ResultCodeMatchPartyStart, Payload>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。MatchPartyStartが成功すればErrorCodeフィールドの値がResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESSになり、そうでない場合はパーティーマッチメイキングが失敗したことになります。サーバー実装によってはDataフィールドのPayloadを通じて追加情報を取得することもできます。

ResultCodeMatchPartyStartの詳細は次のとおりです。

| 名前                                     | 値  | 説明                                       |
|------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                            | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                                | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                             | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗。               |
| INVALID_PROTOCOL                       | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| MATCH_PARTY_START_SUCCESS                | 0    | 成功。                                        |
| MATCH_PARTY_START_FAIL_CONTENT           | 1301 | 失敗。ユーザーコードで拒否されました。                           |
| MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD | 1302 | 失敗。パーティーマッチングをリクエストした際、ルームがパーティーマッチング用ルームではない場合           |

<br>
パーティーマッチングが成功した場合、ユーザーマッチメイキングで使用したonMatchUserDoneを通じて通知を受け取ることができます。そしてMatchResultパラメータを通じてマッチングされたルームの情報を取得でき、サーバー実装によっては追加情報を取得することもできます。
時間内にマッチングが成功しなかった場合、onMatchUserTimeoutを通じて通知を受け取ることができます。

<br>

MatchPartyCancel()を呼び出して進行中のパーティーマッチメイキングをキャンセルできます。

```c#
public async void MatchPartyCancel()
{
    try
    {
        ResultCodeMatchPartyCancel result = await user.MatchPartyCancel("RoomType");
        if (result == ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS)
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

MatchPartyCancel()は次のような1つのパラメータを持っています。

| タイプ   | 名前     | 説明             |
|--------|----------|------------------|
| String | roomType | キャンセルするマッチメイキングのルームタイプ |

レスポンスとしてResultCodeMatchPartyCancelを返します。MatchPartyCancelが成功すればResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESSが返され、そうでない場合はキャンセルリクエストが失敗したことになります。

ResultCodeMatchPartyStartの詳細は次のとおりです。

| 名前                                        | 値  | 説明                                       |
|---------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                               | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                                   | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                              | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗。               |
| INVALID_PROTOCOL                          | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| MATCH_PARTY_CANCEL_SUCCESS                  | 0    | 成功。                                        |
| MATCH_PARTY_CANCEL_FAIL_CONTENT           | 1401 | 失敗。ユーザーコードで拒否されました。                           |
| MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD | 1402 | 失敗。パーティーマッチングをキャンセルする際、ルームがパーティーマッチング用ルームではない場合           |
| MATCH_PARTY_CANCEL_FAIL_ALREADY_JOINED_ROOM | 1403 | 失敗。パーティーマッチングをキャンセルする際、すでにルームに入室している場合            |
| MATCH_PARTY_CANCEL_FAIL_NOT_IN_PROGRESS     | 1404 | 失敗。パーティーマッチング進行中ではないのにキャンセルしようとした場合                 | 

### チャンネル

#### チャンネル移動通知

場合によっては、マッチメイキングの結果としてチャンネル移動が発生することがあります。チャンネル移動が行われた場合、OnMoveChannelを通じて通知を受け取ることができます。そしてMoveChannelResultパラメータから移動したチャンネルの情報を取得でき、サーバー実装によっては追加情報を取得することもできます。

```c#
public void AddOnMoveChannel()
{
    user.OnMoveChannel += (GameAnvilUser user, MoveChannelResult result) =>
    {
        // チャンネル移動時に呼び出し 
    };
}
```

#### チャネル移動

MoveChannel()を呼び出してサービス内の他のチャンネルへ移動できます。

```c#
public async void MoveChannel()
{
    try
    {
        Payload moveChannelPayload = new Payload(new Protocol.MoveChannelData());
        ErrorResult<ResultCodeMoveChannel, MoveChannelResult> result = await user.MoveChannel("ChannelId", moveChannelPayload);
        if (result.ErrorCode == ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS)
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

MoveChannel()は次のような2つのパラメータを持っています。

| タイプ    | 名前      | 説明                                                   |
|---------|-----------|--------------------------------------------------------|
| string  | channelId | 移動するチャンネルのID                                       |
| Payload | payload   | チャンネル移動リクエストを処理するサーバーのユーザーコードで必要な追加情報。(default = null) |

レスポンスとしてErrorResult<ResultCodeMoveChannel, MoveChannelResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。MoveChannelが成功すればErrorCodeフィールドの値がResultCodeMoveChannel.MOVE_CHANNEL_SUCCESSになり、そうでない場合はチャンネル移動に失敗したことになります。Dataフィールドを通じてリクエスト結果MoveChannelResultを取得できます。これを通じて移動したチャンネルの情報を取得でき、サーバー実装に
よっては追加情報を取得できます。
できます。

ResultCodeMoveChannelの詳細は次のとおりです。

| 名前                                     | 値  | 説明                                       |
|------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                            | -2   | パケット解析エラー。サーバーとクライアントのバージョンが異なる場合に発生する可能性があります。   |
| TIMEOUT                                | -1   | タイムアウト。リクエストに対するレスポンスが指定された時間内に来ません。           |
| SYSTEM_ERROR                             | 1    | サーバーシステムエラー。サーバーの不明なエラーにより失敗。               |
| INVALID_PROTOCOL                       | 2    | サーバーに登録されていないプロトコル。追加情報に登録されていないプロトコルが使用されました。 |
| MOVE_CHANNEL_SUCCESS                     | 0    | 成功。                                        |
| MOVE_CHANNEL_FAIL_CONTENT                 | 1601 | 失敗。ユーザーコードで拒否されました。                           |
| MOVE_CHANNEL_FAIL_NODE_NOT_FOUND          | 1602 | 失敗。チャンネルノードが見つかりません。                         |
| MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL  | 1603 | 失敗。すでにリクエストしたチャンネルにいます。                          |
| MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM     | 1604 | 失敗。すでにルームに入室しているためチャンネル移動ができません。               |

MoveChannelResultの詳細は次のとおりです。

| タイプ     | 名前      | 説明                                                                                |
|----------|-----------|-------------------------------------------------------------------------------------|
| bool     | Force     | サーバーによる強制チャンネル移動かどうか。<br/>値がtrue : サーバーでチャンネルを移動。<br/>false : クライアントのリクエストによりチャンネルを移動。 |
| string   | ChannelId | 入室したルームのID。                                                                    |
| Payload? | payload   | クライアントで必要な追加情報。                                                             |

### チャンネル情報

GameAnvilは設定で自由にチャンネルを構成できます。このようなチャンネル構成は、サーバーとクライアント間であらかじめ約束して固定された形態で使用することもできますが、状況に応じて多様に変更して使用することもできます。GameAnvilUserでは、このように変更されたチャンネル情報を取得できるようにいくつかのメソッドを提供します。

| 名前                     | 説明                                  |
|--------------------------|---------------------------------------|
| GetChannelCountInfo()    | 特定チャンネルのカウント情報(ユーザーとルーム数)リクエスト             | 
| GetChannelInfo()         | 特定チャンネルの情報(ユーザー定義)リクエスト                  |
| GetAllChannelCountInfo() | 特定サービスの全てのチャンネルに対するカウント情報(ユーザーとルーム数)リクエスト |
| GetAllChannelInfo()      | 特定サービスの全てのチャンネルに対する情報(ユーザー定義)リクエスト        |

#### GetChannelCountInfo

GetChannelCountInfo()は、特定チャンネルのカウント情報(ユーザーとルーム数)をリクエストして受け取ることができます。

```c#
public async void ChannelCountInfo()
{
    try
    {
        ErrorResult<ResultCodeChannelCountInfo, ChannelCountResult> result = await user.GetChannelCountInfo("ServiceName", "ChannelId");
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

レスポンスとしてErrorResult<ResultCodeChannelCountInfo, ChannelCountResult>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。GetChannelCountInfoが成功すればErrorCodeフィールドの値がResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。Dataフィールドを通じてリクエスト結果であるChannelCountResultを取得できます。

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
        ErrorResult<ResultCodeChannelInfo, Payload> result = await user.GetChannelInfo("ServiceName", "ChannenId");
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

レスポンスとしてErrorResult<ResultCodeChannelInfo, Payload>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。GetChannelInfo()が成功すればErrorCodeフィールドの値がResultCodeChannelInfo.CHANNEL_INFO_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。成功時、DataフィールドのPayloadからユーザーが定義したチャンネル情報を取得することもできます。

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
        ErrorResult<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>> result = await user.GetAllChannelCountInfo("ServiceName");
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

レスポンスとしてErrorResult<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>>を返し、ErrorCodeフィールドの値を確認して成否を確認できます。GetAllChannelCountInfoが成功すればErrorCodeフィールドの値がResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESSになり、そうでない場合はリクエストが失敗したことになります。Dataフィールドを通じてリクエスト結果であるDictionary<string, ChannelCountResult>を取得できます。このDictionaryはチャンネルIDをキーに、ChannelCountResultを値として持っています。

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
        ErrorResult<ResultCodeAllChannelInfo, ChannelInfoResult> result = await user.GetAllChannelInfo("ServiceName");
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
