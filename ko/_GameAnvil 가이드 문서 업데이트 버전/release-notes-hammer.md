## Game > GameAnvil > 릴리스 노트 > GameHammer



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

### 1.0.1 (2020.12.28)

#### Fix
* 같은 Message에 대해 waitFor를 중첩하여 사용할 경우 첫번째 응답시에 모든 중첩된 대기가 풀리는 버그 수정
* ResultAuthentication의 getPayloads()가 null을 리턴하는 버그 수정
* 테스트 종료시 간헐적으로 HandlerPing.onPingTime()에서 NullPointerException발생하는 이슈 수정
* 테스트중 간헐적으로 Statistics.record()에서 ConcurrentModificationException발생하는 이슈 수정

#### Change
* GameHammerConfig.json파일이 없을 경우 출력되는 로그를 error에서 warn으로 변경.

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