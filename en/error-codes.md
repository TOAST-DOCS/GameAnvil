## Game > GameAnvil > Error Codes

## Client

| ResultCode | Name | Value | Description |
| ---- | ---- | ---- | ---- |
| ErrorCode                 | SUCCESS                                     |     0 | Successful                                                         |
|                           | SYSTEM_ERROR                                |     1 | System error                                                  |
|                           | TIMEOUT                                     |    -1 | Timeout                                                    |
| ResultCodeConnect         | CONNECT_SUCCESS                             |     0 | Connection succeeded                                                    |
|                           | CONNECT_FAIL                                |     1 | Connection failed                                                    |
|                           | CONNECT_ALREADY_CONNECTED                   |     2 | Already connected.                                           |
|                           | CONNECT_ALREADY_REQUEST                     |     3 | Already trying to connect.                                           |
| ResultCodeAuth            | AUTH_SUCCESS                                |     0 | Authentication succeeded                                                    |
|                           | AUTH_FAIL_CONTENT                           |   201 | Authentication failed. Rejected from content.                                |
|                           | AUTH_FAIL_MAINTENANCE                       |   202 | Authentication failed. Checking.                                           |
|                           | AUTH_FAIL_INVALID_ACCOUNT_ID                |   203 | Authentication failed. Invalid AccountId.                               |
| ResultCodeChannelList     | CHANNEL_LIST_SUCCESS                        |     0 | Importing the channel list succeeded                                      |
|                           | CHANNEL_LIST_FAIL_INVALID_SERVICEID         |  1801 | Failed to import chanel list. Invalid service id.               |
| ResultCodeChannelInfo     | CHANNEL_INFO_SUCCESS                        |     0 | Importing the channel information succeeded                                      |
|                           | CHANNEL_INFO_FAIL_INVALID_SERVICEID         |  1901 | Failed to get channel information. Invalid service ID                |
| ResultCodeLogin           | LOGIN_SUCCESS                               |     0 | Login succeeded |
|                           | LOGIN_FAIL_CONTENT                          |   301 | Login failed. Rejected from content.                                  |
|                           | LOGIN_FAIL_NOT_EXIST_NODE                   |   302 | Login failed. Node does not exist.                               |
|                           | LOGIN_FAIL_MAINTENANCE                      |   303 | Login failed. Checking.                                             |
|                           | LOGIN_FAIL_TIMEOUT_GAME_SERVER              |   304 | Login failed. Game server not responding.                           |
|                           | LOGIN_FAIL_INVALID_SERVICEID                |   310 | Login failed. Invalid service ID.                               |
|                           | LOGIN_FAIL_INVALID_USERTYPE                 |   311 | Login failed. Invalid user type.                                   |
|                           | LOGIN_FAIL_INVALID_USERID                   |   312 | Login failed. Invalid username.                                 |
|                           | LOGIN_FAIL_INVALID_SUB_ID                   |   313 | Login Failed. Invalid SubId.                                       |
|                           | LOGIN_FAIL_LOGINED_OTHER_SERVICE            |   320 | Login failed. You're signed in to another service.                      |
|                           | LOGIN_FAIL_LOGINED_OTHER_CHANNEL            |   321 | Login failed. You're signed in to a different channel.                        |
|                           | LOGIN_FAIL_LOGINED_OTHER_USER_TYPE          |   322 | Login failed. A different UserType is logged in with the same Account ID. |
|                           | LOGIN_FAIL_LOGINED_OTHER_DEVICE             |   323 | Login failed. Another DeviceId is logged in with the same Account ID. |
| ResultCodeLogout          | LOGOUT_SUCCESS                              |     0 | Logout succeeded.                                                   |
|                           | LOGOUT_FAIL_CONTENT                         |   Appkey is invalid | Logout failed. Rejected from content.                                 |
| ResultCodeCreateRoom | CREATE_ROOM_SUCCESS                  |     0 | Room creation succeeded.                   |
|                      | CREATE_ROOM_FAIL_CONTENT             |   601 | Room creation failed. Rejected from content. |
|                      | CREATE_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |   602 | Room creation failed. Already in a room. |
|                      | CREATE_ROOM_FAIL_CREATE_ROOM_ID      |   603 | Room creation failed. Failed to issue room ID |
| ResultCodeJoinRoom | JOIN_ROOM_SUCCESS                  |     0 | Joined the room.                   |
|                    | JOIN_ROOM_FAIL_CONTENT             |   701 | Failed to join the room. Rejected from content. |
|                    | JOIN_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST |   702 | Failed to join the room. The room does not exist. |
|                    | JOIN_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |   703 | Failed to join a room. Already in a room. |
| ResultCodeLeaveRoom | LEAVE_ROOM_SUCCESS      |     0 | Left the room.              |
|                     | LEAVE_ROOM_FAIL_CONTENT |   801 | Failed to leave the room. Rejected from content. |
| ResultCodeMatchRoom | MATCH_ROOM_SUCCESS                          |     0 | Room matched.                                                   |
|                     | MATCH_ROOM_FAIL_CONTENT                     |   901 | Failed to match a room. Rejected from content.                                |
|                     | MATCH_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST    |   902 | Failed to match a room. The room does not exist.                               |
|                     | MATCH_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM    |   903 | Failed to match a room. Already in a room.                            |
|                     | MATCH_ROOM_FAIL_LEAVE_ROOM                  |   904 | Failed to match a room. When the user failed to leave the existing room while they are matched using the room move feature. |
|                     | MATCH_ROOM_FAIL_IN_PROGRESS                 |   905 | Failed to match a room. Matchmaking is already in progress.                 |
|                     | MATCH_ROOM_FAIL_MATCHED_ROOM_<br/>DOES_NOT_EXIST |   906 | Failed to match a room. The room disappeared while joining the room to find one that satisfies the conditions. |
| ResultCodeNamedRoom | NAMED_ROOM_SUCCESS                  |     0 | Joined the room with the specified name.                          |
|                     | NAMED_ROOM_FAIL_CONTENT             |  1001 | Failed to join the room with the specified name. Rejected from content.       |
|                     | NAMED_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST |  1002 | Failed to join the room with the specified name. Cannot find the room as it is failed to be created. |
|                     | NAMED_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |  1003 | Failed to join the room with the specified name. Already in a room.   |
|                     | NAMED_ROOM_FAIL_INVALID_<br/>ROOM_NAME |  1004 | Failed to join the room with the specified name. Sent a wrong room name. |
| ResultCodeMatchUserStart  | MATCH_USER_START_SUCCESS                   |     0 | User matching started                |
|                           | MATCH_USER_START_FAIL_CONTENT              |  1101 | Failed to start user matching. Rejected from content. |
|                           | MATCH_USER_START_FAIL_ALREADY_<br/>JOINED_ROOM |  1102 | Failed to start user matching. Already in a room. |
| ResultCodeMatchUserCancel | MATCH_USER_CANCEL_SUCCESS                  |     0 | Canceled user matching.               |
|                           | MATCH_USER_CANCEL_FAIL_CONTENT             |  1201 | Failed to cancel user matching. Rejected from content. |
|                           | MATCH_USER_CANCEL_FAIL_ALREADY_<br/>JOINED_ROOM |  1202 | Failed to cancel user matching. Already matched. |
| ResultCodeMatchPartyStart | MATCH_PARTY_START_SUCCESS                 |     0 | Started party matching.                                             |
|                           | MATCH_PARTY_START_FAIL_CONTENT            |  1301 | Failed to start party matching. Rejected from content.                          |
|                           | MATCH_PARTY_START_FAIL_PARTY_<br/>MATCH_WEIRD |  1302 | Failed to start party matching. The room is not for party matching when party matching is requested. |
| ResultCodeMatchPartyCancel | MATCH_PARTY_CANCEL_SUCCESS                |     0 | Canceled party matching.                                           |
|                            | MATCH_PARTY_CANCEL_FAIL_CONTENT           |  1401 | Failed to cancel party matching. Rejected from content.                           |
|                            | MATCH_PARTY_CANCEL_FAIL_PARTY_<br/>MATCH_WEIRD |  1402 | Failed to cancel party matching. When the room is not for party matching when party matching is canceled. |
| ResultCodeMatchUserDone | MATCH_USER_DONE_SUCCESS                  |     0 | Successful user match (party match).                                           |
|                         | MATCH_USER_DONE_FAIL_CONTENT             |  1501 | User match (party match) failed. Rejected by content (no rooms found for the sword). |
|                         | MATCH_USER_DONE_FAIL_ROOM_<br/>DOES_NOT_EXIST |  1502 | Failed to match users (party matching). The room that meets the condition disappears while entering it. |
| ResultCodeMoveChannel | MOVE_CHANNEL_SUCCESS                     |     0 | Moved a channel.                                      |
|                       | MOVE_CHANNEL_FAIL_CONTENT                |  1601 | Failed to move a channel. Rejected from content.                   |
|                       | MOVE_CHANNEL_FAIL_NODE_<br/>NOT_FOUND    |  1602 | Failed to move a channel. Cannot find the channel node.            |
|                       | MOVE_CHANNEL_FAIL_ALREADY_<br/>JOINED_CHANNEL |  1603 | Failed to move a channel. Already in the requested channel.             |
|                       | MOVE_CHANNEL_FAIL_ALREADY_<br/>JOINED_ROOM |  1604 | Failed to move a channel. Cannot move a channel as the user already joined a channel. |
| ResultCodeDisconnect | FORCE_CLOSE_SYSTEM_ERROR            |  2000 | Forced shutdown due to a system error.<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |
|                      | FORCE_CLOSE_BASE_CONNECTION         |  2010 | Calls the close() of BaseConnection from the server                       |
|                      | FORCE_CLOSE_BASE_USER               |  2011 | Calls the closeConnection() of BaseUser from the server                   |
|                      | FORCE_CLOSE_ADMIN_KICK              |  2012 | Force shutdown from Admin.                                         |
|                      | FORCE_CLOSE_MAINTENANCE             |  2020 | Forced shutdown due to server maintenance.                                |
|                      | FORCE_CLOSE_INVALID_NODE            |  2021 | When the status of GameNode has changed to invalid.                       |
|                      | FORCE_CLOSE_USER_TRANSFER_FAIL      |  2022 | When user transfer failed.                                 |
|                      | FORCE_CLOSE_USER_TRANSFER_ERROR     |  2023 | When a system error occurs while transferring user.                   |
|                      | FORCE_CLOSE_AUTHENTICATION_FAIL     |  2030 | Forcibly shut down due to authentication failure.                                  |
|                      | FORCE_CLOSE_DUPLICATE_LOGIN         |  2031 | Forcibly shut down due to duplicate login.                                 |
|                      | FORCE_CLOSE_BY_NEW_CONNECTION       |  2040 | A new login request comes in with the same account information. <br />When you reconnect, for example, to a network drop, use this code to terminate the previous connection.<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |
|                      | FORCE_CLOSE_DISCONNECT_ALARM        |  2041 | Can't find session information.<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |
|                      | FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL |  2042 | The client has not responded to a health check from the server.<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |
|                      | FORCE_CLOSE_GHOST_USER              |  2043 | For a ghost user,<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |
|                      | SOCKET_DISCONNECT                   |  2100 | Disconnected from the network<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |
|                      | SOCKET_TIME_OUT                     |  2101 | Timeout occurred, disconnecting from the connector<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |
|                      | SOCKET_ERROR                        |  2102 | A socket error occurred and disconnected<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |

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


