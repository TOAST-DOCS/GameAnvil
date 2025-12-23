## Game > GameAnvil > サーバー開発ガイド > コールバックフロー

## コールバックフロー
GameAnvilで動作に従い内部的に処理されるコールバック順序です。

## 認証
![callback-flow-1.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-1.png)

クライアントが認証リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト       | コールバックメソッド         | 順序 | 説明         |
|-------------|----------------|----|------------|
| Client      | Authentication | 1  | クライアントからリクエスト |
| IConnection | onAuthenticate | 2  |            |

## ログイン
![callback-flow-2.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-2.png)

クライアントログインリクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト    | コールバックメソッド                    | 順序  | 説明                    |
|----------|--------------------------|-----|----------------------|
| Client   | Login                    | 1   | クライアントからリクエスト            |
| ISession | onBeforeLogin            | 2   | ログイン処理前の設定状態確認      |
| - IUser  | onLoginByOtherDevice     | 2-1 | 異なるデバイスログイン時に呼び出し       |
| - IUser  | onLoginByOtherUserType   | 2-2 | 異なるユーザータイプログイン時に呼び出し      |
| - IUser  | onLoginByOtherConnection | 2-3 | 新しい接続でログイン時に呼び出し       |
| IUser    | onLogin                  | 3-1 | ログインしていない場合ログイン呼び出し    |
| IUser    | onReLogin                | 3-1 | ログインしている場合再ログイン呼び出し    |
| - IRoom  | onRejoinRoom             | 3-2 | 再ログイン時、ルームに入っていた場合に呼び出し |
| IUser    | onAfterLogin             | 4   |                      |
| ISession | onAfterLogin             | 5   |                      |

## ログアウト
![callback-flow-3.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-3.png)

クライアントログアウトリクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト    | コールバックメソッド         | 順序 | 説明                                          |
|------------|------------------|-----|-----------------------------------------------|
| Client      | Logout           | 1   | クライアントからリクエスト                                                    |
| IUser       | canLogout        | 2   | ログアウト処理が可能か確認                                             |
| - IRoom    | onLeaveRoom      | 2-1 |                                               |
| - IRoom     | onAfterLeaveRoom | 2-2 | 2-2、2.3番の動作はuserとroomで処理されるため順序保証がされない |
| - IUser    | onAfterLeaveRoom | 2-3 |                                               |
| IUser      | onLogout         | 3   |                                               |
| ISession   | onAfterLogout    | 4   |                                               |

## ルーム生成
![callback-flow-4.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-4.png)

クライアントルーム生成リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト | コールバックメソッド                | 順序 | 説明       |
|---------|-------------------------|-----|------------|
| Client  | CreateRoom or NamedRoom | 1   | クライアントからリクエスト |
| IRoom   | onCreateRoom            | 2   |            |

## ルーム入室
![callback-flow-5.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-5.png)

クライアントルーム入室リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト | コールバックメソッド              | 順序 | 説明       |
|---------|-----------------------|-----|------------|
| Client  | JoinRoom or NamedRoom | 1   | クライアントからリクエスト |
| IRoom   | onJoinRoom            | 2   |            |

## ルーム退出
![callback-flow-6.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-6.png)

クライアントルーム退出リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト | コールバックメソッド            | 順序 | 説明                                      |
|---------|---------------------|----|-------------------------------------------|
| Client  | LeaveRoom           | 1  | クライアントからリクエスト                                                    |
| IRoom   | canLeaveRoom        | 2  | ルーム退出処理が可能か確認                                            |
| IRoom   | onLeaveRoom         | 3  |                                           |
| IRoom   | onAfterLeaveRoom    | 4  |                                           |
| IUser   | onAfterLeaveRoom    | 5  | 4、5番の動作はuserとroomで処理されるため、順序は保証されない |
| IRoom   | onDestroy           | 6  |                                           |

## ルームマッチ
![callback-flow-7.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-7.png)

クライアントルームマッチリクエスト時に処理されるコールバックメソッド順序です。マッチング前にルームから出る処理を先に行う

| 呼び出しオブジェクト                | コールバックメソッド         | 順序 | 説明                                      |
|------------------------|------------------|----|-------------------------------------------|
| Client                 | MatchRoom        | 1  | クライアントからリクエスト                                                    |
| - IRoom                | canLeaveRoom     | 2  | ルーム退出処理が可能か確認                                            |
| - IRoom                | onLeaveRoom      | 3  |                                           |
| - IRoom                | onAfterLeaveRoom | 4  |                                           |
| - IUser                | onAfterLeaveRoom | 5  | 4、5番の動作はuserとroomで処理されるため順序保証がされない |
| IUser                  | onMatchRoom      | 6  |                                           |
| AbstractRoomMatchMaker | onMatch          | 7  |                                           |
| IRoom                  | onCreateRoom     | 8  |                                           |
| IRoom                  | onJoinRoom       | 8  |                                           |
| IUser                  | onMatchRoomFail  | 9  |                                           |

## ユーザーマッチ
![callback-flow-8.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-8.png)

クライアントユーザーマッチ開始リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト                  | コールバックメソッド          | 順序 | 説明       |
|--------------------------|-------------------|-----|------------|
| Client                   | MatchUserStart    | 1   | クライアントからリクエスト |
| IUser                    | onMatchUser       | 2   |            |
| - AbstractUserMatchMaker | onRefill          | 2-1 | ユーザーリフィル処理   |
| AbstractUserMatchMaker   | onMatch           | 3   |            |
| IRoom                    | onCreateRoom      | 4   |            |
| IRoom                    | onJoinRoom        | 4   |            |
| - IUser                  | onMatchUserCancel | 4-1 | ルーム入室キャンセル     |
| IUser                    | onMatchUserFail   | 5   | マッチ失敗    |

## ユーザーマッチキャンセル
![callback-flow-8.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-9.png)

クライアントユーザーマッチキャンセルリクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト                  | コールバックメソッド           | 順序 | 説明       |
|--------------------------|--------------------|-----|------------|
| Client                   | MatchUserCancel    | 1   | クライアントからリクエスト |
| IUser                    | onMatchUserCancel  | 2   |            |

## パーティーマッチ
![callback-flow-8.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-10.png)

クライアントユーザーマッチ開始リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト                  | コールバックメソッド          | 順序 | 説明       |
|--------------------------|-------------------|-----|------------|
| Client                   | MatchPartyStart   | 1   | クライアントからリクエスト |
| IRoom                    | onMatchParty      | 2   |            |
| - AbstractUserMatchMaker | onRefill          | 2-1 | ユーザーリフィル処理   |
| AbstractUserMatchMaker   | onMatch           | 3   |            |
| IRoom                    | onLeaveRoom       | 4   |            |
| IRoom                    | onAfterLeaveRoom  | 5   |            |
| IUser                    | onAfterLeaveRoom  | 6   |            |
| IRoom                    | onCreateRoom      | 7   |            |
| IRoom                    | onJoinRoom        | 7   |            |
| IUser                    | onMatchUserFail   | 8   | マッチ失敗    |

## パーティーマッチキャンセル
![callback-flow-8.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-11.png)

クライアントパーティーマッチキャンセルリクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト                  | コールバックメソッド           | 順序 | 説明       |
|--------------------------|--------------------|-----|------------|
| Client                   | MatchPartyCancel    | 1   | クライアントからリクエスト |
| IRoom                    | onMatchPartyCancel  | 2   |            |

## ユーザートランスファー
![callback-flow-9.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-12.png)

ユーザートランスファー開始リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト | コールバックメソッド            | 順序 | 説明             |
|---------|---------------------|----|------------------|
| IUser   | canTransferOut      | 1  | ユーザートランスファーが可能か確認 |
| IUser   | onTransferOut       | 2  |                  |
| IUser   | onTransferIn        | 3  |                  |
| IUser   | onAfterTransferIn   | 4  |                  |
| - IUser | onPause             | 5  |                  |
| - IUser | onResume            | 6  |                  |

## ルームトランスファー
![callback-flow-10.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-13.png)

ルームトランスファー開始リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト | コールバックメソッド          | 順序 | 説明              |
|---------|-------------------|-----|-------------------|
| IRoom   | canTransferOut    | 1   | ルームトランスファーが可能か確認   |
| IRoom   | onTransferOut     | 2   |                   |
| - IUser | onTransferOut     | 3-1 | ルームトランスファー後ユーザートランスファー  |
| - IUser | onTransferIn      | 3-2 |                   |
| IRoom   | onTransferIn      | 4   |                   |
| IRoom   | onAfterTransferIn | 5   |                   |
| - IRoom | onPause           | 6   |                   |
| - IRoom | onResume          | 7   |                   |

## チャンネル移動
![callback-flow-11.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-14.png)

クライアントチャンネル移動リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト  | コールバックメソッド             | 順序 | 説明       |
|----------|----------------------|----|------------|
| Client   | MoveChannel          | 1  | クライアントからリクエスト |
| IUser    | canMoveOutChannel    | 2  |            |
| IUser    | onMoveOutChannel     | 3  |            |
| IUser    | onTransferOut        | 4  |            |
| IUser    | onTransferIn         | 5  |            |
| IUser    | onAfterTransferIn    | 6  |            |
| - IUser  | onPause              | 7  |            |
| - IUser  | onResume             | 8  |            |
| IUser    | onMoveInChannel      | 9  |            |

## チャンネル情報確認
![callback-flow-12.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-15.png)

クライアントチャンネル情報リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト     | コールバックメソッド         | 順序 | 説明       |
|-------------|------------------|----|------------|
| Client      | GetChannelInfo   | 1  | クライアントからリクエスト |
| - IGameNode | onChannelInfo    | 2  |            |

## チャンネルユーザー情報更新
![callback-flow-13.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-16.png)

クライアントチャンネル情報更新リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト     | コールバックメソッド                 | 順序 | 説明 |
|-------------|--------------------------|----|----|
| IUser       | updateChannelUserInfo    | 1  |    |
| - IGameNode | onChannelRoomInfoUpdate  | 2  |    |

## チャンネルルーム情報更新
![callback-flow-14.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-17.png)

クライアントチャンネルルーム情報更新リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト     | コールバックメソッド                 | 順序 | 説明 |
|-------------|--------------------------|----|----|
| IRoom       | updateChannelRoomInfo    | 1  |    |
| - IGameNode | onChannelUserInfoUpdate  | 2  |    |

## ゲームデータ更新
![callback-flow-15.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-18.png)

マネジメントノードを通じてゲームデータ更新リクエスト時に処理されるコールバックメソッド順序です。

| 呼び出しオブジェクト        | コールバックメソッド                         | 順序 | 説明          |
|----------------|----------------------------------|----|---------------|
| ManagementNode | /game-data/publish-to-management | 1  | マネジメントノードへリクエスト  |
| - INode        | onUpdateGameData                 | 2  |               |
