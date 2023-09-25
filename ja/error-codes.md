## Game > GameAnvil > エラーコード

## Client

| ResultCode | Name | Value | Description |
| ---- | ---- | ---- | ---- |
| ErrorCode                 | SUCCESS                                     |     0 | 成功                                                        |
|                           | SYSTEM_ERROR                                |     1 | システムエラー                                                 |
|                           | TIMEOUT                                     |    -1 | タイムアウト                                                   |
| ResultCodeConnect         | CONNECT_SUCCESS                             |     0 | 接続成功                                                   |
|                           | CONNECT_FAIL                                |     1 | 接続失敗                                                   |
|                           | CONNECT_ALREADY_CONNECTED                   |     2 | 接続済み                                           |
|                           | CONNECT_ALREADY_REQUEST                     |     3 | 接続試行中                                           |
| ResultCodeAuth            | AUTH_SUCCESS                                |     0 | 認証成功                                                   |
|                           | AUTH_FAIL_CONTENT                           |   201 | 認証失敗。コンテンツに拒否された。                                |
|                           | AUTH_FAIL_MAINTENANCE                       |   202 | 認証失敗。点検中。                                           |
|                           | AUTH_FAIL_INVALID_ACCOUNT_ID                |   203 | 認証失敗。間違ったAccountId。                               |
| ResultCodeChannelList     | CHANNEL_LIST_SUCCESS                        |     0 | チャンネルリストの取得成功                                     |
|                           | CHANNEL_LIST_FAIL_INVALID_SERVICEID         |  1801 | チャンネルリストの取得失敗。間違ったサービスID。               |
| ResultCodeChannelInfo     | CHANNEL_INFO_SUCCESS                        |     0 | チャンネル情報の取得成功                                     |
|                           | CHANNEL_INFO_FAIL_INVALID_SERVICEID         |  1901 | チャンネル情報の取得失敗。間違ったサービスID                |
| ResultCodeLogin           | LOGIN_SUCCESS                               |     0 | ログイン成功。 |
|                           | LOGIN_FAIL_CONTENT                          |   301 | ログイン失敗。コンテンツに拒否された。                                  |
|                           | LOGIN_FAIL_NOT_EXIST_NODE                   |   302 | ログイン失敗。ノードが存在しない。                              |
|                           | LOGIN_FAIL_MAINTENANCE                      |   303 | ログイン失敗。点検中。                                             |
|                           | LOGIN_FAIL_TIMEOUT_GAME_SERVER              |   304 | ログイン失敗。ゲームサーバーからレスポンスがない。                           |
|                           | LOGIN_FAIL_INVALID_SERVICEID                |   310 | ログイン失敗。間違ったサービスID。                               |
|                           | LOGIN_FAIL_INVALID_USERTYPE                 |   311 | ログイン失敗。間違ったユーザータイプ。                                   |
|                           | LOGIN_FAIL_INVALID_USERID                   |   312 | ログイン失敗。間違ったユーザーID。                                 |
|                           | LOGIN_FAIL_INVALID_SUB_ID                   |   313 | ログイン失敗。間違ったSubID。                                       |
|                           | LOGIN_FAIL_LOGINED_OTHER_SERVICE            |   320 | ログイン失敗。他のサービスにログインしている。                      |
|                           | LOGIN_FAIL_LOGINED_OTHER_CHANNEL            |   321 | ログイン失敗。他のチャンネルにログインしている。                        |
|                           | LOGIN_FAIL_LOGINED_OTHER_USER_TYPE          |   322 | ログイン失敗。同じAccount IDで他のUserTypeがログインしている。 |
|                           | LOGIN_FAIL_LOGINED_OTHER_DEVICE             |   323 | ログイン失敗。同じAccount IDで他のDeviceIdがログインしています。 |
| ResultCodeLogout          | LOGOUT_SUCCESS                              |     0 | ログアウト成功。                                                   |
|                           | LOGOUT_FAIL_CONTENT                         |   401 | ログアウト失敗。コンテンツに拒否された。                                 |
| ResultCodeCreateRoom | CREATE_ROOM_SUCCESS                  |     0 | ルーム作成成功。                   |
|                      | CREATE_ROOM_FAIL_CONTENT             |   601 | ルーム作成失敗。コンテンツに拒否された。 |
|                      | CREATE_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |   602 | ルーム作成失敗。すでにルームに入っている。 |
|                      | CREATE_ROOM_FAIL_CREATE_ROOM_ID      |   603 | ルーム作成失敗。ルームIDの発行に失敗 |
| ResultCodeJoinRoom | JOIN_ROOM_SUCCESS                  |     0 | ルームへの入場に成功。                   |
|                    | JOIN_ROOM_FAIL_CONTENT             |   701 | ルームへの入場に失敗。コンテンツに拒否された。 |
|                    | JOIN_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST |   702 | ルームへの入場に失敗。ルームが存在しない。 |
|                    | JOIN_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |   703 | ルームへの入場に失敗。すでにルームに入っている。 |
| ResultCodeLeaveRoom | LEAVE_ROOM_SUCCESS      |     0 | ルームからの退場に成功。              |
|                     | LEAVE_ROOM_FAIL_CONTENT |   801 | ルームからの退場に失敗。コンテンツに拒否された。 |
| ResultCodeMatchRoom | MATCH_ROOM_SUCCESS                          |     0 | ルームマッチ成功。                                                   |
|                     | MATCH_ROOM_FAIL_CONTENT                     |   901 | ルームマッチ失敗。コンテンツに拒否された。                                |
|                     | MATCH_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST    |   902 | ルームマッチ失敗。ルームが存在しない。                               |
|                     | MATCH_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM    |   903 | ルームマッチ失敗。すでにルームに入っている。                            |
|                     | MATCH_ROOM_FAIL_LEAVE_ROOM                  |   904 | ルームマッチ失敗。ルーム移動機能でマッチングさせる際、既存のルームからの退場に失敗した場合。 |
|                     | MATCH_ROOM_FAIL_IN_PROGRESS                 |   905 | ルームマッチ失敗。すでにマッチメイキングが進行中の場合。                 |
|                     | MATCH_ROOM_FAIL_MATCHED_ROOM_<br/>DOES_NOT_EXIST |   906 | ルームマッチ失敗。条件に合ったルームを探し、ルームに参加する途中でルームが消えた。 |
| ResultCodeNamedRoom | NAMED_ROOM_SUCCESS                  |     0 | 指定された名前のルームに入場成功。                          |
|                     | NAMED_ROOM_FAIL_CONTENT             |  1001 | 指定された名前のルームへの入場に失敗。コンテンツに拒否された。       |
|                     | NAMED_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST |  1002 | 指定された名前のルームへの入場に失敗。ルームの作成に失敗し、ルームが見つからない。 |
|                     | NAMED_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |  1003 | 指定された名前のルームへの入場に失敗。すでにルームに入っている。   |
|                     | NAMED_ROOM_FAIL_INVALID_<br/>ROOM_NAME |  1004 | 指定された名前のルームへの入場に失敗。間違ったルーム名を送った場合。 |
| ResultCodeMatchUserStart  | MATCH_USER_START_SUCCESS                   |     0 | ユーザーマッチの開始に成功               |
|                           | MATCH_USER_START_FAIL_CONTENT              |  1101 | ユーザーマッチの開始に失敗。コンテンツに拒否された。 |
|                           | MATCH_USER_START_FAIL_ALREADY_<br/>JOINED_ROOM |  1102 | ユーザーマッチの開始に失敗。すでにルームに入っている。 |
| ResultCodeMatchUserCancel | MATCH_USER_CANCEL_SUCCESS                  |     0 | ユーザーマッチのキャンセル成功。               |
|                           | MATCH_USER_CANCEL_FAIL_CONTENT             |  1201 | ユーザーマッチのキャンセル失敗。コンテンツに拒否された。 |
|                           | MATCH_USER_CANCEL_FAIL_ALREADY_<br/>JOINED_ROOM |  1202 | ユーザーマッチのキャンセル失敗。すでにマッチングが成立している。 |
| ResultCodeMatchPartyStart | MATCH_PARTY_START_SUCCESS                 |     0 | パーティマッチの開始成功。                                             |
|                           | MATCH_PARTY_START_FAIL_CONTENT            |  1301 | パーティマッチの開始に失敗。コンテンツに拒否された。                          |
|                           | MATCH_PARTY_START_FAIL_PARTY_<br/>MATCH_WEIRD |  1302 | パーティマッチの開始に失敗。パーティマッチングをリクエストした際、ルームがパーティマッチング用ルームではない場合。 |
| ResultCodeMatchPartyCancel | MATCH_PARTY_CANCEL_SUCCESS                |     0 | パーティマッチのキャンセル成功。                                           |
|                            | MATCH_PARTY_CANCEL_FAIL_CONTENT           |  1401 | パーティマッチのキャンセル失敗。コンテンツに拒否された。                           |
|                            | MATCH_PARTY_CANCEL_FAIL_PARTY_<br/>MATCH_WEIRD |  1402 | パーティマッチのキャンセル失敗。パーティマッチングをキャンセルした際、ルームがパーティマッチング用ルームではない場合。 |
| ResultCodeMatchUserDone | MATCH_USER_DONE_SUCCESS                  |     0 | ユーザーマッチ(パーティマッチ)成功。                                           |
|                         | MATCH_USER_DONE_FAIL_CONTENT             |  1501 | ユーザーマッチ(パーティマッチ)失敗。コンテンツに拒否された。(条件に合うルームが見つからなかった) |
|                         | MATCH_USER_DONE_FAIL_ROOM_<br/>DOES_NOT_EXIST |  1502 | ユーザーマッチ(パーティマッチ)失敗。条件に合うルームを探し、ルームに入場する途中にルームが消えた |
| ResultCodeMoveChannel | MOVE_CHANNEL_SUCCESS                     |     0 | チャンネルの移動成功。                                      |
|                       | MOVE_CHANNEL_FAIL_CONTENT                |  1601 | チャンネルの移動失敗。コンテンツに拒否された。                   |
|                       | MOVE_CHANNEL_FAIL_NODE_<br/>NOT_FOUND    |  1602 | チャンネルの移動失敗。チャンネルノードが見つからない。            |
|                       | MOVE_CHANNEL_FAIL_ALREADY_<br/>JOINED_CHANNEL |  1603 | チャンネルの移動失敗。すでにリクエストしたチャンネルに入っている。              |
|                       | MOVE_CHANNEL_FAIL_ALREADY_<br/>JOINED_ROOM |  1604 | チャンネルの移動失敗。すでにルームに入場しているため、チャンネルの移動ができない。 |
| ResultCodeDisconnect | FORCE_CLOSE_SYSTEM_ERROR            |  2000 | システムエラーによる強制終了。<br />通常、クライアントからこのコードを受け取ることはありません。このコードを受け取った場合はGameAnvil開発チームにお問い合わせください。 |
|                      | FORCE_CLOSE_BASE_CONNECTION         |  2010 | サーバーからBaseConnectionのclose()呼び出し                      |
|                      | FORCE_CLOSE_BASE_USER               |  2011 | サーバーからBaseUserのcloseConnection()呼び出し                  |
|                      | FORCE_CLOSE_ADMIN_KICK              |  2012 | Adminで強制終了。                                         |
|                      | FORCE_CLOSE_MAINTENANCE             |  2020 | サーバー点検による強制終了。                                |
|                      | FORCE_CLOSE_INVALID_NODE            |  2021 | GameNodeがinvalid状態に変更された場合。                       |
|                      | FORCE_CLOSE_USER_TRANSFER_FAIL      |  2022 | ユーザートランスファーが失敗した場合。                                 |
|                      | FORCE_CLOSE_USER_TRANSFER_ERROR     |  2023 | ユーザートランスファー中にシステムエラーが発生した場合。                   |
|                      | FORCE_CLOSE_AUTHENTICATION_FAIL     |  2030 | 認証失敗による強制終了。                                  |
|                      | FORCE_CLOSE_DUPLICATE_LOGIN         |  2031 | 重複接続による強制終了。                                 |
|                      | FORCE_CLOSE_BY_NEW_CONNECTION       |  2040 | 同じアカウント情報で新たなログインリクエストが入った場合。<br />ネットワーク切断などにより再接続すると、以前の接続を終了しながらこのコードを使用します。<br />通常、クライアントからこのコードを受け取ることはありません。このコードを受け取った場合はGameAnvil開発チームにお問い合わせください。 |
|                      | FORCE_CLOSE_DISCONNECT_ALARM        |  2041 | セッション情報が見つからない場合。<br />通常、クライアントからこのコードを受け取ることはありません。このコードを受け取った場合はGameAnvil開発チームにお問い合わせください。 |
|                      | FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL |  2042 | クライアントがサーバーの状態チェックに応答しなかった場合。<br />通常、クライアントからこのコードを受け取ることはありません。このコードを受け取った場合はGameAnvil開発チームにお問い合わせください。 |
|                      | FORCE_CLOSE_GHOST_USER              |  2043 | ゴーストユーザーの場合。<br />通常、クライアントからこのコードを受け取ることはありません。このコードを受け取った場合はGameAnvil開発チームにお問い合わせください。 |
|                      | SOCKET_DISCONNECT                   |  2100 | ネットワーク接続が切断<br />通常、クライアントからこのコードを受け取ることはありません。このコードを受け取った場合はGameAnvil開発チームにお問い合わせください。 |
|                      | SOCKET_TIME_OUT                     |  2101 | タイムアウトが発生、コネクタからの接続を切断<br />通常、クライアントからこのコードを受け取ることはありません。このコードを受け取った場合はGameAnvil開発チームにお問い合わせください。 |
|                      | SOCKET_ERROR                        |  2102 | ソケットエラーが発生し接続が切断<br />通常、クライアントからこのコードを受け取ることはありません。このコードを受け取った場合はGameAnvil開発チームにお問い合わせください。 |

## Server

| Category   | Name                                            | Value | Description |
| ---------- | ----------------------------------------------- | ----: | ----------- |
| Common     | NODE_NOT_FOUND                                  | 10010 |             |
|            | NODE_INFO_INVALID_ID                            | 10020 |             |
|            | NODE_INFO_NOT_FOUND                             | 10021 |             |
|            | NODE_INFO_INVALID_STATE                         | 10022 |             |
| Location   | LOCATION_ERROR                                  | 10100 |             |
|            | LOCATION_CONNECTION_LOC_ERROR                   | 10110 |             |
|            | LOCATION_CONNECTION_LOC_EMPTY_ACCOUNT_ID        | 10111 |             |
|            | LOCATION_CONNECTION_LOC_NOT_EXIST               | 10112 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_ERROR            | 10120 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_EMPTY_ACCOUNT_ID | 10121 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_INVALID_NODE_ID  | 10122 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_INVALID_CID      | 10123 |             |
|            | LOCATION_ROOM_LOC_ERROR                         | 10130 |             |
|            | LOCATION_ROOM_LOC_INVALID_ROOM_ID               | 10131 |             |
|            | LOCATION_INDEX_ERROR                            | 10140 |             |
|            | LOCATION_INDEX_INVALID_SENDER_NODE_INFO         | 10141 |             |
|            | LOCATION_INDEX_INVALID_CLUSTER_SIZE             | 10142 |             |
|            | LOCATION_INDEX_INVALID_REPLICA_SIZE             | 10143 |             |
|            | LOCATION_INDEX_INVALID_SHARD_FACTOR             | 10144 |             |
| Match User | MATCH_USER_ERROR                                | 10200 |             |
|            | MATCH_USER_REGISTER_ERROR                       | 10210 |             |
|            | MATCH_USER_REGISTER_INFO_NULL                   | 10211 |             |
|            | MATCH_USER_REGISTER_FAIL                        | 10212 |             |
|            | MATCH_USER_REGISTER_REFILL_FAIL                 | 10213 |             |
|            | MATCH_USER_REFILL_ERROR                         | 10220 |             |
|            | MATCH_USER_REFILL_INFO_NULL                     | 10221 |             |
|            | MATCH_USER_REFILL_REGISTER_FAIL                 | 10222 |             |
|            | MATCH_USER_UNREGISTER_ERROR                     | 10230 |             |
|            | MATCH_USER_UNREGISTER_INFO_NULL                 | 10231 |             |
|            | MATCH_USER_UNREGISTER_INVALID_USER_ID           | 10232 |             |
|            | MATCH_USER_UNREGISTER_FAIL                      | 10233 |             |
|            | MATCH_USER_MADE                                 | 10240 |             |
|            | MATCH_USER_MADE_CREATE_ROOM_ID_FAIL             | 10241 |             |
|            | MATCH_USER_MADE_FAIL_USER_TRANSFER              | 10242 |             |
| Match Room | MATCH_ROOM_ERROR                                | 10300 |             |
|            | MATCH_ROOM_FIND_ERROR                           | 10310 |             |
|            | MATCH_ROOM_FIND_MATCH_INVALID_ROOM_ID           | 10311 |             |
| Transfer   | TRANSFER_USER_ERROR                             | 10400 |             |
|            | TRANSFER_USER_TRANSFER_OUT                      | 10410 |             |
|            | TRANSFER_USER_TRANSFER_OUT_TIMER                | 10411 |             |
|            | TRANSFER_USER_TRANSFER_GET_INFO                 | 10412 |             |
|            | TRANSFER_USER_TRANSFER_SET_INFO                 | 10420 |             |
|            | TRANSFER_USER_TRANSFER_IN                       | 10421 |             |
|            | TRANSFER_USER_TRANSFER_IN_TIMER                 | 10422 |             |
|            | TRANSFER_USER_UPDATE_LOCATION_FAIL              | 10423 |             |
|            | TRANSFER_USER_TRANSFER_END_TO_SESSION           | 10424 |             |
|            | TRANSFER_USER_POST_TRANSFER_IN                  | 10425 |             |
|            | TRANSFER_ROOM_ERROR                             | 10500 |             |
|            | TRANSFER_ROOM_TRANSFER_OUT_DATA                 | 10510 |             |
|            | TRANSFER_ROOM_TRANSFER_OUT_DATA_USERS           | 10511 |             |
|            | TRANSFER_ROOM_TRANSFER_OUT_TIMER                | 10512 |             |
|            | TRANSFER_ROOM_UPDATE_LOCATION_FAIL              | 10513 |             |
|            | TRANSFER_ROOM_CREATE_USER_FAIL                  | 10521 |             |
|            | TRANSFER_ROOM_CREATE_ROOM_FAIL                  | 10522 |             |
|            | TRANSFER_ROOM_TRANSFER_IN_DATA                  | 10523 |             |
|            | TRANSFER_ROOM_TRANSFER_IN_TIMER                 | 10524 |             |
|            | TRANSFER_ROOM_DST_UPDATE_LOCATION_FAIL          | 10525 |             |
|            | TRANSFER_ROOM_UPDATE_SESSION_LOGIN_LOC_FAIL     | 10526 |             |
|            | TRANSFER_ROOM_GET_PACKETS                       | 10527 |             |
|            | TRANSFER_ROOM_FROM_SRC_ERROR                    | 10530 |             |
|            | TRANSFER_ROOM_FROM_SRC_NOT_EXIST                | 10531 |             |
|            | TRANSFER_ROOM_GET_PACKETS_ERROR                 | 10540 |             |
|            | TRANSFER_ROOM_GET_PACKETS_NOT_EXIST             | 10541 |             |
|            | TRANSFER_ROOM_GET_PACKETS_NOT_EXIST_ROOM        | 10542 |             |
|            | TRANSFER_ROOM_GET_PACKETS_INVALID_STATE         | 10543 |             |
|            | TRANSFER_ROOM_GET_PACKETS_SET_DATA_FAILED       | 10544 |             |
