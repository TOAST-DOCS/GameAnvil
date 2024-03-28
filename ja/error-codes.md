## Game > GameAnvil > 오류 코드

## Client

| ResultCode | Name | Value | Description |
| ---- | ---- | ---- | ---- |
| ErrorCode                 | SUCCESS                                     |     0 | 성공                                                         |
|                           | SYSTEM_ERROR                                |     1 | 시스템 에러                                                  |
|                           | TIMEOUT                                     |    -1 | 타임 아웃                                                    |
| ResultCodeConnect         | CONNECT_SUCCESS                             |     0 | 연결 성공                                                    |
|                           | CONNECT_FAIL                                |     1 | 연결 실패                                                    |
|                           | CONNECT_ALREADY_CONNECTED                   |     2 | 이미 연결되어있음.                                           |
|                           | CONNECT_ALREADY_REQUEST                     |     3 | 이미 연결시도중임.                                           |
| ResultCodeAuth            | AUTH_SUCCESS                                |     0 | 인증 성공                                                    |
|                           | AUTH_FAIL_CONTENT                           |   201 | 인증 실패. 컨텐츠에서 거부됨.                                |
|                           | AUTH_FAIL_MAINTENANCE                       |   202 | 인증 실패. 점검중.                                           |
|                           | AUTH_FAIL_INVALID_ACCOUNT_ID                |   203 | 인증 실패. 잘못된 AccountId.                               |
| ResultCodeChannelList     | CHANNEL_LIST_SUCCESS                        |     0 | 체널 목록 가져오기 성공                                      |
|                           | CHANNEL_LIST_FAIL_INVALID_SERVICEID         |  1801 | 체널 목록 가져오기 실패. 잘못된 서비스 아이디.               |
| ResultCodeChannelInfo     | CHANNEL_INFO_SUCCESS                        |     0 | 체널 정보 가져오기 성공                                      |
|                           | CHANNEL_INFO_FAIL_INVALID_SERVICEID         |  1901 | 체널 정보 가져오기 실패. 잘못된 서비스 아이디                |
| ResultCodeLogin           | LOGIN_SUCCESS                               |     0 | 로그인 성공. |
|                           | LOGIN_FAIL_CONTENT                          |   301 | 로그인실패. 컨텐츠에서 거부됨.                                  |
|                           | LOGIN_FAIL_NOT_EXIST_NODE                   |   302 | 로그인실패. 노드가 존재하지 않음.                               |
|                           | LOGIN_FAIL_MAINTENANCE                      |   303 | 로그인실패. 점검중.                                             |
|                           | LOGIN_FAIL_TIMEOUT_GAME_SERVER              |   304 | 로그인실패. 게임서버가 응답하지 않음.                           |
|                           | LOGIN_FAIL_INVALID_SERVICEID                |   310 | 로그인실패. 잘못된 서비스 아이디.                               |
|                           | LOGIN_FAIL_INVALID_USERTYPE                 |   311 | 로그인실패. 잘못된 유저 타입.                                   |
|                           | LOGIN_FAIL_INVALID_USERID                   |   312 | 로그인실패. 잘못된 유저 아이디.                                 |
|                           | LOGIN_FAIL_INVALID_SUB_ID                   |   313 | 로그인실패. 잘못된 SubId.                                       |
|                           | LOGIN_FAIL_LOGINED_OTHER_SERVICE            |   320 | 로그인실패. 다른 서비스에 로그인 되어있음.                      |
|                           | LOGIN_FAIL_LOGINED_OTHER_CHANNEL            |   321 | 로그인실패. 다른 채널에 로그인 되어있음.                        |
|                           | LOGIN_FAIL_LOGINED_OTHER_USER_TYPE          |   322 | 로그인실패. 동일 Account 아이디로 다른 UserType이 로그인 되어있음. |
|                           | LOGIN_FAIL_LOGINED_OTHER_DEVICE             |   323 | 로그인실패. 동일 Account 아이디로 다른 DeviceId가 로그인 되어있음. |
| ResultCodeLogout          | LOGOUT_SUCCESS                              |     0 | 로그아웃 성공.                                                   |
|                           | LOGOUT_FAIL_CONTENT                         |   401 | 로그아웃실패. 컨텐츠에서 거부됨.                                 |
| ResultCodeCreateRoom | CREATE_ROOM_SUCCESS                  |     0 | 룸 생성 성공.                   |
|                      | CREATE_ROOM_FAIL_CONTENT             |   601 | 룸 생성 실패. 컨텐츠에서 거부됨. |
|                      | CREATE_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |   602 | 룸 생성 실패. 이미 룸에 들어가 있음. |
|                      | CREATE_ROOM_FAIL_CREATE_ROOM_ID      |   603 | 룸 생성 실패. 룸 아이디 발급 실패 |
| ResultCodeJoinRoom | JOIN_ROOM_SUCCESS                  |     0 | 룸 입장 성공.                   |
|                    | JOIN_ROOM_FAIL_CONTENT             |   701 | 룸 입장 실패. 컨텐츠에서 거부됨. |
|                    | JOIN_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST |   702 | 룸 입장 실패. 룸이 존재하지 않음. |
|                    | JOIN_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |   703 | 룸 입장 실패. 이미 룸에 들어가 있음. |
| ResultCodeLeaveRoom | LEAVE_ROOM_SUCCESS      |     0 | 룸 나가기 성공.              |
|                     | LEAVE_ROOM_FAIL_CONTENT |   801 | 룸 나가기 실패. 컨텐츠에서 거부됨. |
| ResultCodeMatchRoom | MATCH_ROOM_SUCCESS                          |     0 | 룸 매치 성공.                                                   |
|                     | MATCH_ROOM_FAIL_CONTENT                     |   901 | 룸 매치 실패. 컨텐츠에서 거부됨.                                |
|                     | MATCH_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST    |   902 | 룸 매치 실패. 룸이 존재하지 않음.                               |
|                     | MATCH_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM    |   903 | 룸 매치 실패. 이미 룸에 들어가 있음.                            |
|                     | MATCH_ROOM_FAIL_LEAVE_ROOM                  |   904 | 룸 매치 실패. 룸 이동 기능으로 매칭시킬 때, 기존룸에서 나가기가 실패한 경우. |
|                     | MATCH_ROOM_FAIL_IN_PROGRESS                 |   905 | 룸 매치 실패. 이미 매치 매이킹이 진행중인 경우.                 |
|                     | MATCH_ROOM_FAIL_MATCHED_ROOM_<br/>DOES_NOT_EXIST |   906 | 룸 매치 실패. 조건에 맞는 룸을 찾아 룸에 참가 시키는 도중, 룸이 사라짐. |
| ResultCodeNamedRoom | NAMED_ROOM_SUCCESS                  |     0 | 지정된 이름의 룸 입장 성공.                          |
|                     | NAMED_ROOM_FAIL_CONTENT             |  1001 | 지정된 이름의 룸 입장 실패. 컨텐츠에서 거부됨.       |
|                     | NAMED_ROOM_FAIL_ROOM_<br/>DOES_NOT_EXIST |  1002 | 지정된 이름의 룸 입장 실패. 룸 생성이 실패하여 룸을 찾을 수 없음. |
|                     | NAMED_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM |  1003 | 지정된 이름의 룸 입장 실패. 이미 룸에 들어가 있음.   |
|                     | NAMED_ROOM_FAIL_INVALID_<br/>ROOM_NAME |  1004 | 지정된 이름의 룸 입장 실패. 잘못된 룸 이름을 보냈을 경우. |
| ResultCodeMatchUserStart  | MATCH_USER_START_SUCCESS                   |     0 | 유저 매치 시작 성공                |
|                           | MATCH_USER_START_FAIL_CONTENT              |  1101 | 유저 매치 시작 실패. 컨텐츠에서 거부됨 |
|                           | MATCH_USER_START_FAIL_ALREADY_<br/>JOINED_ROOM |  1102 | 유저 매치 시작 실패. 이미 룸에 들어가 있음. |
| ResultCodeMatchUserCancel | MATCH_USER_CANCEL_SUCCESS                  |     0 | 유저 매치 취소 성공.               |
|                           | MATCH_USER_CANCEL_FAIL_CONTENT             |  1201 | 유저 매치 취소 실패. 컨텐츠에서 거부됨. |
|                           | MATCH_USER_CANCEL_FAIL_ALREADY_<br/>JOINED_ROOM |  1202 | 유저 매치 취소 실패. 이미 매칭이 이루어짐. |
| ResultCodeMatchPartyStart | MATCH_PARTY_START_SUCCESS                 |     0 | 파티 매치 시작 성공.                                             |
|                           | MATCH_PARTY_START_FAIL_CONTENT            |  1301 | 파티 매치 시작 실패. 컨텐츠에서 거부됨.                          |
|                           | MATCH_PARTY_START_FAIL_PARTY_<br/>MATCH_WEIRD |  1302 | 파티 매치 시작 실패. 파티매칭을 요청할 때, 룸이 파티매칭용 룸이 아닌경우. |
| ResultCodeMatchPartyCancel | MATCH_PARTY_CANCEL_SUCCESS                |     0 | 파티 매치 취소 성공.                                           |
|                            | MATCH_PARTY_CANCEL_FAIL_CONTENT           |  1401 | 파티 매치 취소 실패. 컨텐츠에서 거부됨                           |
|                            | MATCH_PARTY_CANCEL_FAIL_PARTY_<br/>MATCH_WEIRD |  1402 | 파티 매치 취소 실패. 파티매칭을 취소할 때, 룸이 파티매칭용 룸이 아닌경우. |
| ResultCodeMatchUserDone | MATCH_USER_DONE_SUCCESS                  |     0 | 유저 매치(파티 매치) 성공.                                           |
|                         | MATCH_USER_DONE_FAIL_CONTENT             |  1501 | 유저 매치(파티 매치) 실패. 컨텐츠에서 거부됨(조검에 맞는 룸을 찾지 못함). |
|                         | MATCH_USER_DONE_FAIL_ROOM_<br/>DOES_NOT_EXIST |  1502 | 유저 매치(파티 매치) 실패. 조건에 맞는 룸을 찾아 룸에 입장하는 도중, 룸이 사라짐. |
| ResultCodeMoveChannel | MOVE_CHANNEL_SUCCESS                     |     0 | 체널 이동 성공.                                      |
|                       | MOVE_CHANNEL_FAIL_CONTENT                |  1601 | 체널 이동 실패. 컨텐츠에서 거부됨.                   |
|                       | MOVE_CHANNEL_FAIL_NODE_<br/>NOT_FOUND    |  1602 | 체널 이동 실패. 체널 노드를 찾을 수 없음.            |
|                       | MOVE_CHANNEL_FAIL_ALREADY_<br/>JOINED_CHANNEL |  1603 | 체널 이동 실패. 이미 요청한 체널에 있음.             |
|                       | MOVE_CHANNEL_FAIL_ALREADY_<br/>JOINED_ROOM |  1604 | 체널 이동 실패. 이미 룸에 입장하여 체널 이동을 할 수 없음. |
| ResultCodeDisconnect | FORCE_CLOSE_SYSTEM_ERROR            |  2000 | 시스템 오류로 인한 강제 종료.<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람. |
|                      | FORCE_CLOSE_BASE_CONNECTION         |  2010 | 서버에서 BaseConnection의 close() 호출                       |
|                      | FORCE_CLOSE_BASE_USER               |  2011 | 서버에서 BaseUser의 closeConnection() 호출                   |
|                      | FORCE_CLOSE_ADMIN_KICK              |  2012 | Admin에서 강제 종료.                                         |
|                      | FORCE_CLOSE_MAINTENANCE             |  2020 | 서버 점검으로 인한 강제 종료.                                |
|                      | FORCE_CLOSE_INVALID_NODE            |  2021 | GameNode가 invalid 상태로 변경된 경우.                       |
|                      | FORCE_CLOSE_USER_TRANSFER_FAIL      |  2022 | 유저 트렌스퍼가 실패한 경우.                                 |
|                      | FORCE_CLOSE_USER_TRANSFER_ERROR     |  2023 | 유저 트렌스퍼중 시스템 에러가 발생한 경우.                   |
|                      | FORCE_CLOSE_AUTHENTICATION_FAIL     |  2030 | 인증 실패로 인한 강제 종료.                                  |
|                      | FORCE_CLOSE_DUPLICATE_LOGIN         |  2031 | 중복접속으로 인한 강제 종료.                                 |
|                      | FORCE_CLOSE_BY_NEW_CONNECTION       |  2040 | 같은 계정 정보로 새로운 로그인 요청이 들어온 경우. <br />네트워크 순단 등으로 재접속을 하면 이전의 접속을 종료하면서 이 코드를 사용한다.<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람. |
|                      | FORCE_CLOSE_DISCONNECT_ALARM        |  2041 | 세션 정보를 찾을 수 없는 경우.<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람. |
|                      | FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL |  2042 | 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우.<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람. |
|                      | FORCE_CLOSE_GHOST_USER              |  2043 | 고스트 유저인 경우.<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람. |
|                      | SOCKET_DISCONNECT                   |  2100 | 네트워크 연결이 끊어짐<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람. |
|                      | SOCKET_TIME_OUT                     |  2101 | 타임아웃이 발생, 컨넥터에서 연결을 끊음<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람. |
|                      | SOCKET_ERROR                        |  2102 | 소켓 에러가 발생하여 연결을 끊음<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 바람. |

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


