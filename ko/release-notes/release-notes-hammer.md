## Game > GameAnvil > 릴리스 노트 > GameHammer

### 1.3.0 (2022.12.27)
#### New
* vmOption을 통해 설정을 로드할 수 있는 기능 추가

---
### 1.2.1 (2021.11.30)

#### New 

* SecureSocket지원 기능 추가.
	* RemoteInfo class에  useSecureSocket 옵션 추가. (default : false)
    * GameHammerConfig의 targetServerList의 항목에 useSecureSocket 필드 추가.(default : false)
	* Tester.Bulder.addRemoteInfo()에 useSecureSocket을 입력 받는 오버로딩 추가.

---

### 1.2.0(2021.07.13)
#### Change
* 패키지 구조 정리
	* 내부용 패키지는 gameanvilcore로 묶음.
* ResultCode
  * ResultCodeAuth
    * AUTH_FAIL_MAINTENANCE 제거 
  * ResultCodeCreateRoom
    * CREATE_ROOM_FAIL_CREATE_ROOM_ID 추가
    * CREATE_ROOM_FAIL_CREATE_ROOM 추가
  * ResultCodeChannelInfo
    * CHANNEL_INFO_FAIL_NO_CHANNEL_INFO 추가
    * CHANNEL_INFO_FAIL_INVALID_SERVICE_ID 추가
    * CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND 추가
  * ResultCodeAllChannelInfo 추가
  * ResultCodeChannelCountInfo 추가
  * ResultCodeAllChannelCountInfo 추가
  * ResultCodeChannelList
    * CHANNEL_LIST_FAIL_INVALID_SERVICEID 제거
    * CHANNEL_LIST_FAIL_NO_CHANNEL_LIST 추가
  * ResultCodeJoinRoom
    * JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM 추가
    * JOIN_ROOM_FAIL_ALREADY_FULL 추가
    * JOIN_ROOM_FAIL_ROOM_MATCH 추가
  * ResultCodeLogin
    * LOGIN_FAIL_MAINTENANCE 제거
  * ResultCodeMatchUserCancel
    * MATCH_USER_CANCEL_FAIL_CONTENT -> MATCH_USER_CANCEL_FAIL 이름 변경
    * MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS 추가
  * ResultCodeMatchRoom
    * MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID 추가
    * MATCH_ROOM_FAIL_CREATE_FAILED_ROOM  추가
    * MATCH_ROOM_FAIL_INVALID_ROOM_ID  추가
    * MATCH_ROOM_FAIL_INVALID_NODE_ID  추가
    * MATCH_ROOM_FAIL_INVALID_USER_ID  추가
    * MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND  추가
    * MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY 추가
    * MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY 추가
    * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL  추가
    * MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL 추가
  * ResultCodeMatchUserDone
    * MATCH_USER_DONE_FAIL_TRANSFER 추가
    * MATCH_USER_DONE_FAIL_CREATE_ROOM 추가
  * ResultCodeNamedRoom
    * NAMED_ROOM_FAIL_CREATE_ROOM 추가
  * ResultCodeDisconnect
    * FORCE_CLOSE_MAINTENANCE 제거
    * FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID 추가.
    * FORCE_CLOSE_DISCONNECT_ALARM 제거
    * FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT 추가
    * FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION 추가
  * ResultCodeSessionClose 추가

### 1.1.2 (2021.11.30)

#### New 

* SecureSocket지원 기능 추가.
	* RemoteInfo class에  useSecureSocket 옵션 추가. (default : false)
    * GameHammerConfig의 targetServerList의 항목에 useSecureSocket 필드 추가.(default : false)
	* Tester.Bulder.addRemoteInfo()에 useSecureSocket을 입력 받는 오버로딩 추가.

---

### 1.1.1 (2021.04.16)

#### New 

* ping 기능 온오프 가능하도록 `Connection.setSendPingPaused()`추가.

#### Fix

* config의 pingIngerval이 적용되지 않는 버그 수정
* config의 pingIngerval이 0 이면 ping을 안 보내도록 수정

---

### 1.1.0 (2021.04.15)

#### Change

* 서버의 버전과 맞추기위해 1.1.0으로 올림.

#### New

* sendPauseClientStateCheck() 추가.
* sendResumeClientStateCheck() 추가.
* 서버에서 오는 상태체크 응답을 켜고 끌 수 있도록 기능 추가

---

### 1.0.2 (2020.02.10)

#### Fix
* 클라이언트에서 게임 노드로 지정된 시간(default 10초)동안 아무런 패킷을 보내지 않을 경우 서버에서 클라이언트로 상태 확인 요청을 보내게 되는데, GameHammer에서 이 상태 확인 요청에 잘못된 응답을 하여 접속이 끊어지는 문제 수정
* 시나리오 테스트를 장시간 유지하여 요청한 패킷의 수가 아주 많아질 경우 packetSeq 가 overflow되어 서버에서 응답을 주지 않는 문제 수정  
* send시에도 packetSeq를 증가시키는 문제 수정

#### Change
* 로그 내용 강화.
    * accountId, userId 추가.
    * 마지막으로 받은 패킷 추가.
    * Ping/Pong 마지막 패킷에서 제외
* 패킷의 크기를 줄이기 위해 
    * packetSeq 최대값 16383으로 제한
    * subId 최대값 127, 최소값 1로 제한

---

### 1.0.1 (2020.12.28)

#### Fix
* 같은 Message에 대해 waitFor를 중첩하여 사용할 경우 첫번째 응답시에 모든 중첩된 대기가 풀리는 버그 수정
* ResultAuthentication의 getPayloads()가 null을 리턴하는 버그 수정
* 테스트 종료시 간헐적으로 HandlerPing.onPingTime()에서 NullPointerException발생하는 이슈 수정
* 테스트중 간헐적으로 Statistics.record()에서 ConcurrentModificationException발생하는 이슈 수정

#### Change
* GameHammerConfig.json파일이 없을 경우 출력되는 로그를 error에서 warn으로 변경.

---

### 1.0.0 (2020.12.18)

#### Fix
* EA버전에서 장시간 테스트 실패하는 이슈 수정.
* EA버전 대비 TPS 성능 대폭 개선(약 2배)
* 누락된 기능 추가.
    * Connector
        * getChannelInfo
        * addListenerAdminKickoutNoti
        * addListenerForceCloseNoti
        * addListenerDisconnect
    * User
        * moveChannel
        * snapShot
        * addListenerMatchPartyStartNoti
        * addListenerMatchPartyCancelNoti
        * addListenerForceLogoutNoti
        * addListenerForceLeaveRoomNoti
        * addListenerMoveChannelNoti
        * addListenerNotice

#### Change
* Tester
    * 모든 요청 방식 기능에 Sync/Async 방식을 지원
        * Sync : 요청시 Future를 리턴값으로 받아 Fureture.get()으로 완료될때까지 대기하고 결과를 받아 처리하는 방식.
        * Async : 요청시 callback을 인자로 넘겨주고, callback에서 완료 결과를 받아 처리하는 방식.
* Scenario
    * TRANSACTION과 EVENT 개념 제거
        * 대신 각 State에서 changeState()를 사용해 원하는 State로 직접 이동

#### New
* 서버에서 보내는 noti를 기다려 처리할 수 있도록 waitForXXX 기능 추가.

---

### 1.0.0-EA (2020.08.03)

#### New
* Tester - GameAnvil Connector를 대신하여 서버와의 연동 기능 테스트를 지원
    * Connection - GameAnvil Connector의 ConnectionAgent가 담당하는 기능 지원
        * connect
        * authenticate
        * getChannelList
        * send
        * request
        * createUser
    * User - GameAnvil Connector의 UserAgent가 담당하는 기능 지원
        * login
        * logout
        * createRoom
        * namedRoom
        * joinRoom
        * leaveRoom
        * matchRoom
        * matchUser
        * matchPartyStart/Cancel
        * moveChannelStart/Cancel
        * send
        * request
        * addListenerMatchUserTimeout
        * addListenerMatchUserDone
* ScenarioTest - Tester를 사용한 시나리오 테스트를 지원. 
    * ScenarioMachine - 시나리오를 구성하는 여러 상태의 모음
    * State - 사용자가 정의하는 전체 시나리오 중 특정 상태를 표현
    * ScenarioActor - 시나리오를 수행하는 하나의 가상 유저