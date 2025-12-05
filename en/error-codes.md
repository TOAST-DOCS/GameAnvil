## Game > GameAnvil > Error Codes

## Client

| ResultCode | Name | Value | Description |
| ---- | ---- | ---- | ---- |
| ErrorCode | PARSE_ERROR | -2 | Packet parsing error. |
| | TIMEOUT | -1 | Timeout |
| | SUCCESS | 0 | Success |
| | SYSTEM_ERROR | 1 | Server system error |
| | INVALID_PROTOCOL | 2 | Protocol not registered on the server |
| | HANDLER_NOT_EXIST | 10 | No handler on the server |
| | HANDLER_ERROR | 11 | An exception occurred in the server's handler. |
| ResultCodeConnect | CONNECT_SUCCESS | 0 | Success |
| | CONNECT_FAIL | 1 | Failed |
| | CONNECT_ALREADY_CONNECTED | 2 | Failed. Already connected |
| | CONNECT_ALREADY_REQUEST | 3 | Failed. Already attempting to connect |
| | CONNECT_FAIL_INVALID_IP | 4 | Failed. Invalid IP address |
| ResultCodeAuth | AUTH_SUCCESS | 0 | Success |
| | AUTH_FAIL_CONTENT | 201 | Failed. Content rejected |
| | AUTH_FAIL_MAINTENANCE | 202 | Failed. Invalid authentication account ID |
| ResultCodeLogin | LOGIN_SUCCESS | 0 | Success |
|                           | LOGIN_FAIL_CONTENT                          |   301 | Login failed. Rejected from content.                                  |
|                           | LOGIN_FAIL_NOT_EXIST_NODE                   |   302 | Login failed. Node does not exist.                               |
|                           | LOGIN_FAIL_MAINTENANCE                      |   303 | Login failed. Checking.                                             |
|                           | LOGIN_FAIL_INVALID_SERVICE_NAME                |   310 | Login failed. Invalid service name.                               |
|                           | LOGIN_FAIL_INVALID_USER_TYPE                 |   311 | Login failed. Invalid user type.                                   |
|                           | LOGIN_FAIL_INVALID_USER_ID                   |   312 | Login failed. Invalid username.                                 |
|                           | LOGIN_FAIL_INVALID_SUB_ID                   |   313 | Login Failed. Invalid Sub Id.                                       |
|                               | LOGIN_FAIL_INVALID_CHANNEL_ID                    | 314   | Login failed. Invalid channel ID                                                                                                                                      |
|                           | LOGIN_FAIL_LOGINED_OTHER_SERVICE            |   320 | Login failed. You're signed in to another service.                      |
|                           | LOGIN_FAIL_LOGINED_OTHER_CHANNEL            |   321 | Login failed. You're signed in to a different channel.                        |
|                           | LOGIN_FAIL_LOGINED_OTHER_USER_TYPE          |   322 | Login failed. A different UserType is logged in with the same Account ID. |
|                           | LOGIN_FAIL_LOGINED_OTHER_DEVICE             |   323 | Login failed. Another DeviceId is logged in with the same Account ID. |
| ResultCodeLogout | LOGOUT_SUCCESS | 0 | Success |
| | LOGOUT_FAIL_CONTENT | 401 | Failed. Content rejected |
| ResultCodeSnapshot | SNAPSHOT_SUCCESS | 0 | Success |
| | SNAPSHOT_FAIL_CONTENT | 501 | Failed. Content rejected |
| ResultCodeCreateRoom | CREATE_ROOM_SUCCESS | 0 | Success |
|                      | CREATE_ROOM_FAIL_CONTENT             |   601 | Room creation failed. Rejected from content. |
|                      | CREATE_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |   602 | Room creation failed. Already in a room. |
|                      | CREATE_ROOM_FAIL_CREATE_ROOM_ID      |   603 | Room creation failed. Failed to issue room ID |
| | CREATE_ROOM_FAIL_CREATE_ROOM | 604 | Failed. Failed to create room |
| ResultCodeJoinRoom | JOIN_ROOM_SUCCESS                  |     0 | Joined the room.                   |
|                    | JOIN_ROOM_FAIL_CONTENT             |   701 | Failed to join the room. Rejected from content. |
|                    | JOIN_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST |   702 | Failed to join the room. The room does not exist. |
|                    | JOIN_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |   703 | Failed to join a room. Already in a room. |
| | JOIN_ROOM_FAIL_ALREADY_FULL | 704 | Failed. If the room is already full |
| | JOIN_ROOM_FAIL_ROOM_MATCH | 705 | Failed. If a problem occurs during room matchmaking |
| ResultCodeLeaveRoom | LEAVE_ROOM_SUCCESS      |     0 | Left the room.              |
|                     | LEAVE_ROOM_FAIL_CONTENT |   801 | Failed to leave the room. Rejected from content. |
| ResultCodeMatchRoom | MATCH_ROOM_SUCCESS                          |     0 | Room matched.                                                   |
|                     | MATCH_ROOM_FAIL_CONTENT                     |   901 | Failed to match a room. Rejected from content.                                |
|                     | MATCH_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST    |   902 | Failed to match a room. The room does not exist.                               |
|                     | MATCH_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM    |   903 | Failed to match a room. Already in a room.                            |
|                     | MATCH_ROOM_FAIL_LEAVE_ROOM                  |   904 | Failed to match a room. When the user failed to leave the existing room while they are matched using the room move feature. |
|                     | MATCH_ROOM_FAIL_IN_PROGRESS                 |   905 | Failed to match a room. Matchmaking is already in progress.                 |
|                     | MATCH_ROOM_FAIL_MATCHED_ROOM_<br/>DOES_NOT_EXIST |   906 | Failed to match a room. The room disappeared while joining the room to find one that satisfies the conditions. |
| | MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID | 907 | Failed. If RoomId creation failed. |
| | MATCH_ROOM_FAIL_CREATE_FAILED_ROOM | 908 | Failed. If Room creation failed. |
| | MATCH_ROOM_FAIL_INVALID_ROOM_ID | 909 | Failed. If an invalid Room ID was used. |
| | MATCH_ROOM_FAIL_INVALID_NODE_ID | 910 | Failed. If an invalid Node ID was used. |
| | MATCH_ROOM_FAIL_INVALID_USER_ID | 911 | Failed. If an invalid User ID was used. |
| | MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND | 912 | Failed. Matching was successful, but no room was found |
| | MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY | 913 | Failed. If an incorrect matching user category was used |
| | MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY | 914 | Failed. If the user category size in the matching room is 0. |
| | MATCH_ROOM_FAIL_MATCH_FORM_NULL | 915 | Failed. If the matching request is null. |
| | MATCH_ROOM_FAIL_MATCH_INFO_NULL | 916 | Failed. If the matching information is null. |
| ResultCodeNamedRoom | NAMED_ROOM_SUCCESS                  |     0 | Joined the room with the specified name.                          |
|                     | NAMED_ROOM_FAIL_CONTENT             |  1001 | Failed to join the room with the specified name. Rejected from content.       |
|                     | NAMED_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST |  1002 | Failed to join the room with the specified name. Cannot find the room as it is failed to be created. |
|                     | NAMED_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |  1003 | Failed to join the room with the specified name. Already in a room.   |
|                     | NAMED_ROOM_FAIL_INVALID_<br/>ROOM_NAME |  1004 | Failed to join the room with the specified name. Sent a wrong room name. |
| | NAMED_ROOM_FAIL_CREATE_ROOM | 1005 | Failed. Failed to create room |
| ResultCodeMatchUserStart  | MATCH_USER_START_SUCCESS                   |     0 | User matching started                |
|                           | MATCH_USER_START_FAIL_CONTENT              |  1101 | Failed to start user matching. Rejected from content. |
| | MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM | 1102 | Failed. Already in the room |
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
| | MATCH_PARTY_CANCEL_FAIL_ALREADY_JOINED_ROOM | 1403 | Failed, when canceling a party match, if you are already in the room |
| | MATCH_PARTY_CANCEL_FAIL_NOT_IN_PROGRESS | 1404 | Failed, when trying to cancel a party match that is not in progress |
| ResultCodeMatchUserDone | MATCH_USER_DONE_SUCCESS                  |     0 | Successful user match (party match).                                           |
|                         | MATCH_USER_DONE_FAIL_CONTENT             |  1501 | User match (party match) failed. Rejected by content (no rooms found for the sword). |
|                         | MATCH_USER_DONE_FAIL_ROOM_<br/>DOES_NOT_EXIST |  1502 | Failed to match users (party matching). The room that meets the condition disappears while entering it. |
| | MATCH_USER_DONE_FAIL_TRANSFER | 1503 | Failed. While searching for a room that meets the conditions and joining the room, the transfer process to join the room failed. |
| | MATCH_USER_DONE_FAIL_CREATE_ROOM | 1504 | Failed. Room creation failed. |
| ResultCodeMoveChannel | MOVE_CHANNEL_SUCCESS                     |     0 | Moved a channel.                                      |
|                       | MOVE_CHANNEL_FAIL_CONTENT                |  1601 | Failed to move a channel. Rejected from content.                   |
|                       | MOVE_CHANNEL_FAIL_NODE_<br/>NOT_FOUND    |  1602 | Failed to move a channel. Cannot find the channel node.            |
|                       | MOVE_CHANNEL_FAIL_ALREADY_<br/>JOINED_CHANNEL |  1603 | Failed to move a channel. Already in the requested channel.             |
|                       | MOVE_CHANNEL_FAIL_ALREADY_<br/>JOINED_ROOM |  1604 | Failed to move a channel. Cannot move a channel as the user already joined a channel. |
| ResultCodeChannelList | CHANNEL_LIST_SUCCESS | 0 | Success |
| | CHANNEL_LIST_FAIL_NO_CHANNEL_LIST | 1801 | Failed. Channel list not found |
| ResultCodeChannelInfo | CHANNEL_INFO_SUCCESS | 0 | Success |
| | CHANNEL_INFO_FAIL_NO_CHANNEL_INFO | 1901 | Failed. Channel information not found |
| | CHANNEL_INFO_FAIL_INVALID_SERVICE_NAME | 1902 | Failed. Invalid service name |
| | CHANNEL_INFO_FAIL_INVALID_CHANNEL_ID | 1903 | Failed. Invalid channel ID |
| | CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND | 1904 | Failed. Channel not found |
| ResultCodeAllChannelInfo | ALL_CHANNEL_INFO_SUCCESS | 0 | Success |
| | ALL_CHANNEL_INFO_FAIL_NO_CHANNEL_INFO | 1911 | Failed. Channel information not found |
| | ALL_CHANNEL_INFO_FAIL_INVALID_SERVICE_NAME | 1912 | Failed. Invalid service name |
| | ALL_CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND | 1913 | Failed. Channel not found |
| ResultCodeChannelCountInfo | CHANNEL_COUNT_INFO_SUCCESS | 0 | Success |
| | CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO | 1921 | Failed. Channel information not found |
| | CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_NAME | 1922 | Failed. Invalid service name |
| | CHANNEL_COUNT_INFO_FAIL_INVALID_CHANNEL_ID | 1923 | Failed. Invalid channel ID |
| | CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND | 1924 | Failed. Channel not found |
| ResultCodeAllChannelCountInfo | ALL_CHANNEL_COUNT_INFO_SUCCESS | 0 | Success |
| | ALL_CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO | 1931 | Failed. Channel information not found |
| | ALL_CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_NAME | 1932 | Failed. Invalid service name |
| | ALL_CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND | 1933 | Failed. Channel not found |
| ResultCodeDisconnect | FORCE_CLOSE_SYSTEM_ERROR            |  2000 | Forced shutdown due to a system error.<br />It is rare for clients to receive this code, and if you do, please contact the GameAnvil development team. |
|                      | FORCE_CLOSE_CONNECTION         |  2010 | Calls the close() of BaseConnection from the server                       |
|                      | FORCE_CLOSE_USER               |  2011 | Calls the closeConnection() of BaseUser from the server                   |
|                      | FORCE_CLOSE_ADMIN_KICK              |  2012 | Force shutdown from Admin.                                         |
|                      | FORCE_CLOSE_MAINTENANCE             |  2020 | Forced shutdown due to server maintenance.                                |
| | FORCE_CLOSE_USER_TRANSFER_FAIL | 2021 | If a user transfer fails |
| | FORCE_CLOSE_USER_TRANSFER_ERROR | 2022 | If a system error occurs during a user transfer |
| | FORCE_CLOSE_AUTHENTICATION_FAIL | 2030 | Forced close due to authentication Failed |
| | FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID | 2031 | Forced close due to authentication failure - If the account ID does not exist |
| | FORCE_CLOSE_DUPLICATE_LOGIN | 2032 | Forced close due to duplicate login |
| | FORCE_CLOSE_BY_NEW_CONNECTION | 2040 | When a new login request comes in with the same account information<br />When reconnecting due to network interruption, etc., this code is used to terminate the previous connection<br />Generally, this code will rarely be received from the client, and if so, please contact the GameAnvil development team. |
| | FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT | 2041 | When a disconnection from the client is detected<br />Generally, this code will rarely be received from the client, and if so, please contact the GameAnvil development team. |
| | FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION | 2042 | When the session cannot be found<br />Generally, this code will rarely be received from the client, and if so, please contact the GameAnvil development team. |
| | FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL | 2043 | If the client did not respond to the server's status check<br />This code is rarely received by the client, and if so, you should contact the GameAnvil development team. |
| | FORCE_CLOSE_GHOST_USER | 2044 | If you are a ghost user<br />This code is rarely received by the client, and if so, you should contact the GameAnvil development team. |
| | SOCKET_DISCONNECT | 2100 | The network connection was lost. |
| | SOCKET_TIME_OUT | 2101 | A timeout occurred, and the connector closed the connection. |
| | SOCKET_ERROR | 2102 | A socket error occurred, and the connection was closed. |

## Server

| Category   | Name                                             | Value | Description |
|------------|--------------------------------------------------|------:| ----------- |
| Common     | INVALID_USER_ID                                  | 10000 |             |
|            | NODE_NOT_FOUND                                   | 10010 |             |
|            | TARGET_NOT_FOUND                                 | 10011 |             |
|            | NODE_INFO_INVALID_ID                             | 10020 |             |
|            | NODE_INFO_NOT_FOUND                              | 10021 |             |
|            | NODE_INFO_INVALID_STATE                          | 10022 |             |
|            | REQ_TIME_OUT                                     | 10030 |             |
| Location   | LOCATION_ERROR                                   | 10100 |             |
|            | LOCATION_CONNECTION_LOC_ERROR                    | 10110 |             |
|            | LOCATION_CONNECTION_LOC_EMPTY_ACCOUNT_ID         | 10111 |             |
|            | LOCATION_CONNECTION_LOC_NOT_EXIST                | 10112 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_ERROR             | 10120 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_EMPTY_ACCOUNT_ID  | 10121 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_INVALID_NODE_ID   | 10122 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_INVALID_CID       | 10123 |             |
|            | LOCATION_UPDATE_CONNECTION_LOC_INVALID_SUB_ID    | 10124 |             |
|            | LOCATION_ROOM_LOC_ERROR                          | 10130 |             |
|            | LOCATION_ROOM_LOC_INVALID_ROOM_ID                | 10131 |             |
|            | LOCATION_INDEX_ERROR                             | 10140 |             |
|            | LOCATION_INDEX_INVALID_SENDER_NODE_INFO          | 10141 |             |
|            | LOCATION_INDEX_INVALID_CLUSTER_SIZE              | 10142 |             |
|            | LOCATION_INDEX_INVALID_REPLICA_SIZE              | 10143 |             |
|            | LOCATION_INDEX_INVALID_SHARD_FACTOR              | 10144 |             |
|            | LOCATION_USER_LOC_ERROR                          | 10150 |             |
| Match User | MATCH_USER_ERROR                                 | 10200 |             |
|            | MATCH_USER_REGISTER_ERROR                        | 10210 |             |
|            | MATCH_USER_REGISTER_INFO_NULL                    | 10211 |             |
|            | MATCH_USER_REGISTER_FAIL                         | 10212 |             |
|            | MATCH_USER_REGISTER_REFILL_FAIL                  | 10213 |             |
|            | MATCH_USER_REFILL_ERROR                          | 10220 |             |
|            | MATCH_USER_REFILL_INFO_NULL                      | 10221 |             |
|            | MATCH_USER_REFILL_REGISTER_FAIL                  | 10222 |             |
|            | MATCH_USER_UNREGISTER_ERROR                      | 10230 |             |
|            | MATCH_USER_UNREGISTER_INFO_NULL                  | 10231 |             |
|            | MATCH_USER_UNREGISTER_INVALID_USER_ID            | 10232 |             |
|            | MATCH_USER_UNREGISTER_FAIL                       | 10233 |             |
|            | MATCH_USER_MADE                                  | 10240 |             |
|            | MATCH_USER_MADE_CREATE_ROOM_ID_FAIL              | 10241 |             |
|            | MATCH_USER_MADE_FAIL_USER_TRANSFER               | 10242 |             |
| Match Room | MATCH_ROOM_ERROR                                 | 10300 |             |
|            | MATCH_ROOM_NOT_EXIST_MATCH_MANAGER               | 10301 |             |
|            | MATCH_ROOM_NOT_EXIST_MATCH_INFO                  | 10302 |             |
|            | MATCH_ROOM_ALREADY_REGISTER_ROOM                 | 10310 |             |
|            | MATCH_ROOM_MATCHING_USER_CATEGORY_EMPTY          | 10320 |             |
|            | MATCH_ROOM_INCREASE_USER_COUNT_FAIL_FULL         | 10321 |             |
| Transfer   | TRANSFER_USER_ERROR                              | 10400 |             |
|            | TRANSFER_USER_TRANSFER_OUT                       | 10410 |             |
|            | TRANSFER_USER_TRANSFER_OUT_TIMER                 | 10411 |             |
|            | TRANSFER_USER_TRANSFER_GET_INFO                  | 10412 |             |