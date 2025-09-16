## Game > GameAnvil > 오류 코드

## Client

| ResultCode                    | Name                                             | Value | Description                                                                                                                                         |
|-------------------------------|--------------------------------------------------|-------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| ErrorCode                     | PARSE_ERROR                                      | -2    | 패킷 파싱 에러.                                                                                                                                           |
|                               | TIMEOUT                                          | -1    | 타임 아웃                                                                                                                                               |
|                               | SUCCESS                                          | 0     | 성공                                                                                                                                                  |
|                               | SYSTEM_ERROR                                     | 1     | 서버 시스템 에러                                                                                                                                           |
|                               | INVALID_PROTOCOL                                 | 2     | 서버에 등록되지 않은 프로토콜                                                                                                                                    |
|                               | HANDLER_NOT_EXIST                                | 10    | 서버에 핸들러 없음                                                                                                                                          |
|                               | HANDLER_ERROR                                    | 11    | 서버의 핸들러에서 예외 발생.                                                                                                                                    |
| ResultCodeConnect             | CONNECT_SUCCESS                                  | 0     | 성공                                                                                                                                                  |
|                               | CONNECT_FAIL                                     | 1     | 실패                                                                                                                                                  |
|                               | CONNECT_ALREADY_CONNECTED                        | 2     | 실패. 이미 연결되어있음                                                                                                                                       |
|                               | CONNECT_ALREADY_REQUEST                          | 3     | 실패. 이미 연결 시도중임                                                                                                                                      |
|                               | CONNECT_FAIL_INVALID_IP                          | 4     | 실패. 잘못된 IP 입력                                                                                                                                       |
| ResultCodeAuth                | AUTH_SUCCESS                                     | 0     | 성공                                                                                                                                                  |
|                               | AUTH_FAIL_CONTENT                                | 201   | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | AUTH_FAIL_MAINTENANCE                            | 202   | 실패. 잘못된 인증 계정 아이디                                                                                                                                   |
| ResultCodeLogin               | LOGIN_SUCCESS                                    | 0     | 성공                                                                                                                                                  |
|                               | LOGIN_FAIL_CONTENT                               | 301   | 실패. 콘텐츠에서 거부됨                                                                                                                                       |
|                               | LOGIN_FAIL_NOT_EXIST_NODE                        | 302   | 실패. 노드가 존재하지 않음                                                                                                                                     |
|                               | LOGIN_FAIL_TIMEOUT_GAME_SERVER                   | 303   | 실패. 게임 서버가 응답하지 않음                                                                                                                                   |
|                               | LOGIN_FAIL_INVALID_SERVICE_NAME                  | 310   | 실패. 잘못된 서비스 이름                                                                                                                                      |
|                               | LOGIN_FAIL_INVALID_USER_TYPE                     | 311   | 실패. 잘못된 유저 타입                                                                                                                                       |
|                               | LOGIN_FAIL_INVALID_USER_ID                       | 312   | 실패. 잘못된 유저 아이디                                                                                                                                      |
|                               | LOGIN_FAIL_INVALID_SUB_ID                        | 313   | 실패. 잘못된 서브아이디                                                                                                                                       |
|                               | LOGIN_FAIL_INVALID_CHANNEL_ID                    | 314   | 실패. 잘못된 채널 아이디                                                                                                                                      |
|                               | LOGIN_FAIL_LOGINED_OTHER_SERVICE                 | 320   | 실패. 다른 서비스에 로그인 되어있음                                                                                                                                |
|                               | LOGIN_FAIL_LOGINED_OTHER_CHANNEL                 | 321   | 실패. 다른 채널에 로그인 되어있음                                                                                                                                 |
|                               | LOGIN_FAIL_LOGINED_OTHER_USER_TYPE               | 322   | 실패. 동일 계정 아이디로 다른 유저타입이 로그인 되어있음                                                                                                                    |
|                               | LOGIN_FAIL_LOGINED_OTHER_DEVICE                  | 323   | 실패. 동일 계정 아이디로 다른 기기 식별용 고유 아이디가 로그인 되어있음                                                                                                           |
| ResultCodeLogout              | LOGOUT_SUCCESS                                   | 0     | 성공                                                                                                                                                  |
|                               | LOGOUT_FAIL_CONTENT                              | 401   | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
| ResultCodeSnapshot            | SNAPSHOT_SUCCESS                                 | 0     | 성공                                                                                                                                                  |
|                               | SNAPSHOT_FAIL_CONTENT                            | 501   | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
| ResultCodeCreateRoom          | CREATE_ROOM_SUCCESS                              | 0     | 성공                                                                                                                                                  |
|                               | CREATE_ROOM_FAIL_CONTENT                         | 601   | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | CREATE_ROOM_FAIL_ALREADY_<br/>JOINED_ROOM        | 602   | 실패. 이미 방에 들어가 있음                                                                                                                                    |
|                               | CREATE_ROOM_FAIL_CREATE_ROOM_ID                  | 603   | 실패. 방 아이디 발급 실패                                                                                                                                     |
|                               | CREATE_ROOM_FAIL_CREATE_ROOM                     | 604   | 실패. 방 생성 실패                                                                                                                                         |
| ResultCodeJoinRoom            | JOIN_ROOM_SUCCESS                                | 0     | 성공                                                                                                                                                  |
|                               | JOIN_ROOM_FAIL_CONTENT                           | 701   | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST               | 702   | 실패. 방이 존재하지 않음                                                                                                                                      |
|                               | JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM               | 703   | 실패. 이미 방에 들어가 있음                                                                                                                                    |
|                               | JOIN_ROOM_FAIL_ALREADY_FULL                      | 704   | 실패. 이미 방의 인원수가 차있을 경우                                                                                                                               |
|                               | JOIN_ROOM_FAIL_ROOM_MATCH                        | 705   | 실패. 룸매치 메이킹에서 문제가 발생할 경우                                                                                                                            |
| ResultCodeLeaveRoom           | LEAVE_ROOM_SUCCESS                               | 0     | 성공                                                                                                                                                  |
|                               | LEAVE_ROOM_FAIL_CONTENT                          | 801   | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
| ResultCodeMatchRoom           | MATCH_ROOM_SUCCESS                               | 0     | 성공                                                                                                                                                  |
|                               | MATCH_ROOM_FAIL_CONTENT                          | 901   | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST              | 902   | 실패. 방이 존재하지 않음                                                                                                                                      |
|                               | MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM              | 903   | 실패. 이미 방에 들어가 있음                                                                                                                                    |
|                               | MATCH_ROOM_FAIL_LEAVE_ROOM                       | 904   | 실패. 방 이동 기능으로 매칭시킬 때, 기존방에서 나가기가 실패한 경우                                                                                                             |
|                               | MATCH_ROOM_FAIL_IN_PROGRESS                      | 905   | 실패. 이미 매치 매이킹이 진행중인 경우                                                                                                                              |
|                               | MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST      | 906   | 실패. 조건에 맞는 방을 찾아 방에 참가 시키는 도중, 방이 사라짐                                                                                                               |
|                               | MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID            | 907   | 실패. RoomId 생성이 실패하였을 경우                                                                                                                             |
|                               | MATCH_ROOM_FAIL_CREATE_FAILED_ROOM               | 908   | 실패. 방 생성이 실패하였을 경우                                                                                                                                  |
|                               | MATCH_ROOM_FAIL_INVALID_ROOM_ID                  | 909   | 실패. 잘못된 룸아이디가 사용되었을 경우                                                                                                                              |
|                               | MATCH_ROOM_FAIL_INVALID_NODE_ID                  | 910   | 실패. 잘못된 노드아이디가 사용되었을 경우                                                                                                                             |
|                               | MATCH_ROOM_FAIL_INVALID_USER_ID                  | 911   | 실패. 잘못된 유저아이디가 사용되었을 경우                                                                                                                             |
|                               | MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND           | 912   | 실패. 매칭을 진행하였으나, 방을 찾지 못함                                                                                                                            |
|                               | MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY   | 913   | 실패. 잘못된 매칭 유저 카테고리를 사용하였을 경우                                                                                                                        |
|                               | MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY     | 914   | 실패. 매칭룸에서 유저 카테고리 사이즈가 0일 경우                                                                                                                        |
|                               | MATCH_ROOM_FAIL_MATCH_FORM_NULL                  | 915   | 실패. 매칭 신청서가 null일 경우                                                                                                                                  |
|                               | MATCH_ROOM_FAIL_MATCH_INFO_NULL                  | 916   | 실패. 매칭 정보가 null일 경우                                                                                                                                   |
| ResultCodeNamedRoom           | NAMED_ROOM_SUCCESS                               | 0     | 성공                                                                                                                                                  |
|                               | NAMED_ROOM_FAIL_CONTENT                          | 1001  | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST              | 1002  | 실패. 방 생성이 실패하여 방을 찾을 수 없음                                                                                                                           |
|                               | NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM              | 1003  | 실패. 이미 방에 들어가 있음                                                                                                                                    |
|                               | NAMED_ROOM_FAIL_INVALID_ROOM_NAME                | 1004  | 실패. 잘못된 방 이름을 보냈을 경우                                                                                                                                |
|                               | NAMED_ROOM_FAIL_CREATE_ROOM                      | 1005  | 실패. 방 생성 실패                                                                                                                                         |
| ResultCodeMatchUserStart      | MATCH_USER_START_SUCCESS                         | 0     | 성공                                                                                                                                                  |
|                               | MATCH_USER_START_FAIL_CONTENT                    | 1101  | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM        | 1102  | 실패. 이미 방에 들어가 있음                                                                                                                                    |
| ResultCodeMatchUserCancel     | MATCH_USER_CANCEL_SUCCESS                        | 0     | 성공                                                                                                                                                  |
|                               | MATCH_USER_CANCEL_FAIL                           | 1201  | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM       | 1202  | 실패. 이미 매칭이 이루어짐                                                                                                                                     |
|                               | MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS           | 1203  | 실패, 매칭이 진행중이지 않은 경우                                                                                                                                 |
| ResultCodeMatchPartyStart     | MATCH_PARTY_START_SUCCESS                        | 0     | 성공                                                                                                                                                  |
|                               | MATCH_PARTY_START_FAIL_CONTENT                   | 1301  | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD         | 1302  | 실패. 파티매칭을 요청할 때, 방이 파티매칭용 방이 아닌경우                                                                                                                   |
| ResultCodeMatchPartyCancel    | MATCH_PARTY_CANCEL_SUCCESS                       | 0     | 성공                                                                                                                                                  |
|                               | MATCH_PARTY_CANCEL_FAIL_CONTENT                  | 1401  | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD        | 1402  | 실패. 파티매칭을 취소할 때, 방이 파티매칭용 방이 아닌경우                                                                                                                   |
|                               | MATCH_PARTY_CANCEL_FAIL_ALREADY_JOINED_ROOM      | 1403  | 실패, 파티매칭을 취소할 때, 이미 방에 입장해 있는 경우                                                                                                                    |
|                               | MATCH_PARTY_CANCEL_FAIL_NOT_IN_PROGRESS          | 1404  | 실패, 파티매칭 진행 중이 아닌데 취소하려고 할 때                                                                                                                        |
| ResultCodeMatchUserDone       | MATCH_USER_DONE_SUCCESS                          | 0     | 성공                                                                                                                                                  |
|                               | MATCH_USER_DONE_FAIL_CONTENT                     | 1501  | 실패. 컨텐츠에서 거부됨(조건에 맞는 방을 찾지 못함)                                                                                                                      |
|                               | MATCH_USER_DONE_FAIL_ROOM_DOES_NOT_EXIST         | 1502  | 실패. 조건에 맞는 방을 찾아 방에 참가 시키는 도중, 방이 사라짐                                                                                                               |
|                               | MATCH_USER_DONE_FAIL_TRANSFER                    | 1503  | 실패. 조건에 맞는 방을 찾아 방에 참가 시키는 도중, 방에 참가하기 위해 transfer 하는 과정에서 실패함                                                                                      |
|                               | MATCH_USER_DONE_FAIL_CREATE_ROOM                 | 1504  | 실패. 방 생성 실패                                                                                                                                         |
| ResultCodeMoveChannel         | MOVE_CHANNEL_SUCCESS                             | 0     | 성공                                                                                                                                                  |
|                               | MOVE_CHANNEL_FAIL_CONTENT                        | 1601  | 실패. 컨텐츠에서 거부됨                                                                                                                                       |
|                               | MOVE_CHANNEL_FAIL_NODE_NOT_FOUND                 | 1602  | 실패. 채널 노드를 찾을 수 없음                                                                                                                                  |
|                               | MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL         | 1603  | 실패. 이미 요청한 채널에 있음                                                                                                                                   |
|                               | MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM            | 1604  | 실패. 이미 방에 입장하여 채널 이동을 할 수 없음                                                                                                                        |
| ResultCodeChannelList         | CHANNEL_LIST_SUCCESS                             | 0     | 성공                                                                                                                                                  |
|                               | CHANNEL_LIST_FAIL_NO_CHANNEL_LIST                | 1801  | 실패. 채널 목록을 찾을 수 없음                                                                                                                                  |
| ResultCodeChannelInfo         | CHANNEL_INFO_SUCCESS                             | 0     | 성공                                                                                                                                                  |
|                               | CHANNEL_INFO_FAIL_NO_CHANNEL_INFO                | 1901  | 실패. 채널 정보를 찾을 수 없음                                                                                                                                  |
|                               | CHANNEL_INFO_FAIL_INVALID_SERVICE_NAME           | 1902  | 실패. 잘못된 서비스 이름                                                                                                                                      |
|                               | CHANNEL_INFO_FAIL_INVALID_CHANNEL_ID             | 1903  | 실패. 잘못된 채널 아이디                                                                                                                                      |
|                               | CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND              | 1904  | 실패. 채널을 찾을 수 없음                                                                                                                                     |
| ResultCodeAllChannelInfo      | ALL_CHANNEL_INFO_SUCCESS                         | 0     | 성공                                                                                                                                                  |
|                               | ALL_CHANNEL_INFO_FAIL_NO_CHANNEL_INFO            | 1911  | 실패. 채널 정보를 찾을 수 없음                                                                                                                                  |
|                               | ALL_CHANNEL_INFO_FAIL_INVALID_SERVICE_NAME       | 1912  | 실패. 잘못된 서비스 이름                                                                                                                                      |
|                               | ALL_CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND          | 1913  | 실패. 채널을 찾을 수 없음                                                                                                                                     |
| ResultCodeChannelCountInfo    | CHANNEL_COUNT_INFO_SUCCESS                       | 0     | 성공                                                                                                                                                  |
|                               | CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO          | 1921  | 실패. 채널 정보를 찾을 수 없음                                                                                                                                  |
|                               | CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_NAME     | 1922  | 실패. 잘못된 서비스 이름                                                                                                                                      |
|                               | CHANNEL_COUNT_INFO_FAIL_INVALID_CHANNEL_ID       | 1923  | 실패. 잘못된 채널 아이디                                                                                                                                      |
|                               | CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND        | 1924  | 실패. 채널을 찾을 수 없음                                                                                                                                     |
| ResultCodeAllChannelCountInfo | ALL_CHANNEL_COUNT_INFO_SUCCESS                   | 0     | 성공                                                                                                                                                  |
|                               | ALL_CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO      | 1931  | 실패. 채널 정보를 찾을 수 없음                                                                                                                                  |
|                               | ALL_CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_NAME | 1932  | 실패. 잘못된 서비 이름                                                                                                                                       |
|                               | ALL_CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND    | 1933  | 실패. 채널을 찾을 수 없음                                                                                                                                     |
| ResultCodeDisconnect          | FORCE_CLOSE_SYSTEM_ERROR                         | 2000  | 시스템 오류로 인한 강제 종료<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 필요                                                             |
|                               | FORCE_CLOSE_CONNECTION                           | 2010  | 서버에서 BaseConnection의 close() 호출                                                                                                                     |
|                               | FORCE_CLOSE_USER                                 | 2011  | 서버에서 BaseUser의 closeConnection() 호출                                                                                                                 |
|                               | FORCE_CLOSE_ADMIN_KICK                           | 2012  | 어드민에서 강제 종료                                                                                                                                         |
|                               | FORCE_CLOSE_INVALID_NODE                         | 2020  | GameNode가 INVALID 상태로 변경된 경우                                                                                                                        |
|                               | FORCE_CLOSE_USER_TRANSFER_FAIL                   | 2021  | 유저 트렌스퍼가 실패한 경우                                                                                                                                     |
|                               | FORCE_CLOSE_USER_TRANSFER_ERROR                  | 2022  | 유저 트렌스퍼중 시스템 에러가 발생한 경우                                                                                                                             |
|                               | FORCE_CLOSE_AUTHENTICATION_FAIL                  | 2030  | 인증 실패로 인한 강제 종료                                                                                                                                     |
|                               | FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID | 2031  | 인증 실패로 인한 강제 종료 - 계정 아이디가 없을 경우                                                                                                                     |
|                               | FORCE_CLOSE_DUPLICATE_LOGIN                      | 2032  | 중복접속으로 인한 강제 종료                                                                                                                                     |
|                               | FORCE_CLOSE_BY_NEW_CONNECTION                    | 2040  | 같은 계정 정보로 새로운 로그인 요청이 들어온 경우<br />네트워크 순단 등으로 재접속을 하면 이전의 접속을 종료하면서 이 코드를 사용<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 필요 |
|                               | FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT         | 2041  | 클라이언트와의 연결 끊김을 감지할 경우<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 필요                                                        |
|                               | FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION    | 2042  | 세션을 찾을 수 없는 경우<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 필요                                                               |
|                               | FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL              | 2043  | 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 필요                                               |
|                               | FORCE_CLOSE_GHOST_USER                           | 2044  | 고스트 유저인 경우<br />일반적으로 클라이언트에서 이 코드를 받을 일은 거의 없으며, 이 코드를 받았다면 GameAnvil 개발팀에 문의 필요                                                                   |
|                               | SOCKET_DISCONNECT                                | 2100  | 네트워크 연결이 끊어짐                                                                                                                                        |
|                               | SOCKET_TIME_OUT                                  | 2101  | 타임아웃이 발생, 컨넥터에서 연결을 끊음                                                                                                                              |
|                               | SOCKET_ERROR                                     | 2102  | 소켓 에러가 발생하여 연결을 끊음                                                                                                                                  |

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
|            | TRANSFER_USER_TRANSFER_OUT_TIMER_HANDLER         | 10413 |             |
|            | TRANSFER_USER_TRANSFER_SET_INFO                  | 10420 |             |
|            | TRANSFER_USER_TRANSFER_IN                        | 10421 |             |
|            | TRANSFER_USER_TRANSFER_IN_TIMER                  | 10422 |             |
|            | TRANSFER_USER_UPDATE_LOCATION_FAIL               | 10423 |             |
|            | TRANSFER_USER_TRANSFER_END_TO_SESSION            | 10424 |             |
|            | TRANSFER_USER_POST_TRANSFER_IN                   | 10425 |             |
|            | TRANSFER_USER_TRANSFER_IN_TIMER_HANDLER          | 10426 |             |
|            | TRANSFER_ROOM_ERROR                              | 10500 |             |
|            | TRANSFER_ROOM_TRANSFER_OUT_DATA                  | 10510 |             |
|            | TRANSFER_ROOM_TRANSFER_OUT_DATA_USERS            | 10511 |             |
|            | TRANSFER_ROOM_TRANSFER_OUT_TIMER                 | 10512 |             |
|            | TRANSFER_ROOM_UPDATE_LOCATION_FAIL               | 10513 |             |
|            | TRANSFER_ROOM_TRANSFER_OUT_TIMER_HANDLER         | 10514 |             |
|            | TRANSFER_ROOM_CREATE_USER_FAIL                   | 10521 |             |
|            | TRANSFER_ROOM_CREATE_ROOM_FAIL                   | 10522 |             |
|            | TRANSFER_ROOM_TRANSFER_IN_DATA                   | 10523 |             |
|            | TRANSFER_ROOM_TRANSFER_IN_TIMER                  | 10524 |             |
|            | TRANSFER_ROOM_TRANSFER_CHANNEL_ROOM_INFO         | 10525 |             |
|            | TRANSFER_ROOM_DST_UPDATE_LOCATION_FAIL           | 10526 |             |
|            | TRANSFER_ROOM_DST_UPDATE_ROOM_MATCH_NODE_ID_FAIL | 10527 |             |
|            | TRANSFER_ROOM_UPDATE_SESSION_LOGIN_LOC_FAIL      | 10528 |             |
|            | TRANSFER_ROOM_GET_PACKETS                        | 10529 |             |
|            | TRANSFER_ROOM_FROM_SRC_ERROR                     | 10530 |             |
|            | TRANSFER_ROOM_FROM_SRC_NOT_EXIST                 | 10531 |             |
|            | TRANSFER_ROOM_TRANSFER_IN_TIMER_HANDLER          | 10532 |             |
|            | TRANSFER_ROOM_GET_PACKETS_ERROR                  | 10540 |             |
|            | TRANSFER_ROOM_GET_PACKETS_NOT_EXIST              | 10541 |             |
|            | TRANSFER_ROOM_GET_PACKETS_NOT_EXIST_ROOM         | 10542 |             |
|            | TRANSFER_ROOM_GET_PACKETS_INVALID_STATE          | 10543 |             |
|            | TRANSFER_ROOM_GET_PACKETS_SET_DATA_FAILED        | 10544 |             |
| Safe Pause | SAFE_PAUSE_START_FAIL_INVALID_TARGET_NODE_TYPE   | 10600 |             |
|            | SAFE_PAUSE_START_FAIL_ALREADY_STARTED            | 10601 |             |
|            | SAFE_PAUSE_START_FAIL_STATE_NOT_READY            | 10602 |             |


