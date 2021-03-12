## Game > GameAnvil > 릴리스 노트



### 1.0.5 (2020.11.17)

#### Fix

- RoomMatchReq에서 에러 응답 시 패킷 헤더 복원(restore)이 안 되는 문제 수정



### 1.0.4 (2020.10.29)

#### Fix

- 임의의 Session에 아직 아무런 패킷이 전달된 적이 없을 경우에 이 Session으로 send 호출 시 NPE(Null Pointer Exception) 발생하는 문제 수정
- publishToClient 사용 시 문제 발생(Notice 포함)
- Room에서 사용하는 반복 Timer로 인해 방 파괴가 지연되는 문제 수정
- protobuf 구조체를 변수로 가지고 있는 Class를 deserialize할 때 예외 발생하는 이슈 수정
- java.lang.UnsupportedOperationException이 발생



### 1.0.3 (2020.10.26)

#### Change

- 이전에 스펙에서 제외되었던 공지 기능 API를 다시 활성화



### 1.0.2 (2020.10.12)

#### New

- Log에 표시되는 Node 정보를 사용자가 원하는 대로 설정할 수 있는 기능 추가
- HostId를 직접 입력할 수 있는 기능 추가
- nodeInfoByHostId에서 HostId별로 정보를 보내주는 기능 추가

#### Fix

- IP 0.0.0.0을 바인딩할 때 에러가 발생하는 이슈 수정
- ConfigModule의 /config/get 사용 시 성공 여부 값이 실패로 응답하는 문제 수정
- GameNode가 구동된 상태에서 다른 GameNode가 구동될 경우, Publish Packet 처리로 인해 에러가 발생할 수 있는 문제 예외 처리 추가

#### Change

- UserId, RoomId 생성 시 마지막 자리에 사용하는 ShardIndex값 범위를 지정할 수 있도록 수정
- RestObject의 getRemoteAddress 값 추가
- NodeKey 요청 개수를 지정할 수 있도록 수정



### 1.0.1 (2020.09.07)

### Fix

- onLogin()에서 false를 리턴할 경우 Payload가 클라이언트로 전달되지 안는 이슈 수정
- Debug와 Trace 로그문 앞의 로그 레벨 확인 조건문 누락된 부분 추가



### 1.0.0 (2020.08.31)

#### New

- 성능과 안정성 대폭 향상 (0.9버전 대비 약 7배)
- 성능에 직접적인 영향을 주는 다수의 로직을 최적화하고 문제점 수정
- 완전히 새로워진 GameNode 로드밸런싱
- 클라우드 VM의 상태를 반영하여 빠르고 정확하게 최적의 GameNode로 부하를 분산
- Integer형 ID 도입
- 기존에 String으로 되어 있던 핵심 ID들을 모두 Integer로 교체하여 최적화 진행
- 서버 Log 레벨을 런타임에 변경할 수 있는 API 추가
- Node 단위로 서비스 중에 Log 레벨을 변경 가능

#### Fix

- 유저 전송 문제 수정, 사용법 개선 그리고 최적화
- GameAnvil 사용자는 더욱 쉽게 유저 전송을 사용 가능
- 무점검 패치 완벽 수정
- 그동안 방 전송과 무점검 패치가 제대로 동작하지 않던 문제 모두 수정
- 서버 구동 시에 알 수 없는 타이밍 이슈로 일부 Node가 제대로 뜨지 않던 문제 수정
- Node의 일부 Timer 객체가 해제되지 않고 지속적으로 누적되던 Memory Leak 수정
- 클라이언트 접속 체크 과정에서 새롭게 접속하는 유저가 의도하지 않게 끊기던 문제 수정

#### Change

- Node 이름을 아래의 표와 같이 더욱 직관적으로 변경

| v1.0       | ~ v0.10       | 필수 | 기능                                        | 콘텐츠 구현 | 네트워크 접근     |
| ---------- | ------------- | ---- | ------------------------------------------- | ----------- | ----------------- |
| Gateway    | Session       | 필수 | 클라이언트 접속과 인증을 처리               | 가능        | public            |
| Game       | Space         | 필수 | 실제 게임 서버로서 콘텐츠를 처리            | 가능        | private           |
| Support    | Service       | 선택 | 필요에 따라 독립된 서비스로 구현하도록 지원 | 가능        | private or public |
| Match      | Match         | 선택 | 매치메이킹을 수행                           | 가능        | private           |
| Location   | Location      | 필수 | 유저와 방 등의 위치 정보를 저장 및 관리     | 불가능      | private           |
| Management | Management    | 필수 | 서버 정보 취합 및 Admin/Agent와 통신        | 불가능      | private           |
| Ipc        | Communication | 필수 | Gameflex 서버의 Inter-process 통신 처리     | 불가능      | private           |

- GameAnvil 코드 가독성과 사용성 향상
- 패키지 구조를 효율적으로 개선
- 클래스 상속/포함 구조 개선
- 네이밍을 직관적이고 일관성 있게 수정
- Node의 비활성화 상태는 Invalid와 Disable로 세분화
- Invalid는 일시적으로 반응을 하지 않는 상태
- Disable은 영구적으로 비활성화된 상태
- 이제 Loopback 주소는 자동으로 바인딩
- GameAnvil에서 사용하는 모든 라이브러리를 최신 버전으로 업데이트
- 전체 ErrorCode를 하나로 통합
- 불필요한 ReConnect와 MoveService 기능 제거
- 등록 시점이 아닌 구동 시점부터 다음 시간을 측정하는 새로운 Timer 추가



### 0.10.2 (2020.04.08)

#### New

- GameAnvil API 레퍼런스 (JavaDoc) 사이트 오픈

- AOT Instrumentation 지원

- 엔진을 런타임에 매번 JIT Instrumentation할 필요 없이 컴파일 타임에 AOT로 진행

- 공식 GC로 G1GC를 사용하도록 가이드 시작

- 가장 기본적인 VM 옵션

  `java -javaagent:QUASAR_PATH\quasar-core-0.7.10-jdk8.jar=bm -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication`

#### Fix

- Quasar 스택 버그를 수정
- 룸 매칭 유저들이 서로 다른 방으로 매칭되는 버그 수정
- 압축된 패킷이 전송 안 되는 문제 수정



### 0.10.1 (2020.02.11)

#### New

- 사용자에게 제공하는 도구를 모아둔 Util 클래스를 제공
- 기존에는 엔진 여러 곳에 흩어져있던 도구들을 해당 클래스 한 군데로 정리

#### Change

- Topic 처리 코드 성능 개선



### 0.10.0 (2020.02.06)

#### New

- 기존에 문제가 많던 비동기 지원 API를 새롭게 리팩토링
- Redis Cluster 지원 및 Lettuce 공식 지원에 따른 새로운 비동기 API 추가
- 비동기 HttpRequest와 HttpResponse API 추가
- ByteString 기반의 커스텀 프로토콜 기능 지원
- Google protobuf 외의 사용자가 원하는 방식으로 메시지를 직렬화 가능

#### Fix

- 이전 버전에서 가장 심각한 문제였던 파이버의 상태가 망가지는 문제 모두 수정

#### Change

- 엔진에서 사용하는 패킷과 헤더 크기 최적화
- 0.9버전 대비 성능이 약 2.5배 향상
- Embedded Redis Spec-out
- 인프라에서 제공하는 서비스를 사용하도록 가이드



### 0.9.9 (2019.10.25)

#### New

- 프로세스 단위로 WatchDog을 연동할 수 있도록 포트를 추가로 개방

#### Fix

- 고스트 유저 버그 수정
- SpaceNode에서 발생하는 Concurrent Modification 에러 수정
- NodeInfoManager의 Comparator 오류 수정
- 로그아웃한 유저의 세션 객체가 정리되지 않는 문제 수정
- 매칭 그룹이 서로 다른 채널의 유저들에게 제대로 적용되지 않던 문제 수정
- 파티 매치 메이킹 시에 Timeout에 대한 콜백을 제대로 받지 못하던 문제 수정

#### Change

- 룸 매치 메이킹을 이미 진행 중인 상태에서 중복 요청이 왔을 경우에 대한 에러 코드 추가
- MATCH_ROOM_FAIL_IN_PROGRESS



### 0.9.8 (2019.09.05)

#### New

- Session에 ClientTopic을 추가하고 삭제할 수 있는 addTopic(), removeTopic() API 추가
- Packet TTL 기능 추가
- Dangling Location을 주기적으로 체크해서 정리하는 로직 추가

#### Fix

- Admin에서 유저를 Kick할 경우 클라이언트로 ForceLogoutNoti를 전송하지 않도록 수정
- LocationNode의 Spot 오동작 수정

#### Change

- onAuthenticate() 콜백 실패 시에 해당 소켓을 닫기 전에 ForceCloseNoti를 클라이언트로 전송하도록 변경



### 0.9.7 (2019.07.27)

#### New

- 비정상적으로 남아있는 Room을 주기적으로 체크하고 정리하는 로직 추가
- 유저 매치 메이킹은 요청을 한 후에 재로그인을 하더라도 이전의 매치 요청이 취소되지 않고 진행 중일 수 있는데, 이러한 이전의 유저 매치 메이킹 처리 상태를 알려주기 위해 재로그인 응답에 새로운 플래그를 추가
- Packet Expire 가능 추가(기본값 30초)

#### Fix

- Connection ID(CID)가 중복되는 문제 수정
- Disconnection된 유저의 의해 방 파이버가 close되는 버그 수정
- 로그인이 실패인 경우 응답 메시지에 serviceId 필드가 제대로 세팅되지 않는 문제 수정
- 콘텐츠 콜백 처리부 중 try-catch가 누락된 부분 수정
- 클라이언트 Reconnect가 약 3초 이상 걸리던 문제를 바로 처리되도록 수정
- 고스트 유저 체크 시 발생하는 스레드 동시성 문제 수정
- 룸 매치 메이킹 중복 처리 버그 수정
- 룸에 연결이 끊긴 유저가 남는 문제 수정

#### Change

- Session Disconnect의 모든 경우에 대해 로그 추가
- CreateRoom은 현재의 노드에 바로 방을 생성하도록 변경



### 0.9.6 (2019.06.23)

#### New

- 전송 가능한 시점을 콘텐츠에서 조율할 수 있도록 User와 Room에 canTransfer() 인터페이스를 추가

#### Fix

- 엔진 내부 코드에 존재하던 메모리 릭 수정
- ZMQ Router, Publiser 소켓의 연결 버그 수정
- Login이 무한 반복되는 문제 수정
- isLogined() API가 제대로 동작하지 않는 문제 수정
- NodeInfoManager 동기화 문제 수정
- 유저 객체가 제대로 정리되지 않는 문제 수정

#### Change

- 기존의 AsyncAwaitHttpRequest를 FiberHttpRequest로 이름 변경
- FiberHttpRequest와 HttpRequest에 비동기 함수 추가
- Location Index 관련 로그 추가
- SpaceNode 분배 방식을 idle에서 roundRobin으로 변경
- NodeStarter에서 LocationNode가 있을 경우에만 sleep하도록 변경



### 0.9.5 (2019.06.21)

#### Fix

- 무점검 패치 시에 새로운 유저가 로그인하거나 유저 전송 중일 경우 Pause된 노드에서 해당 유저가 정리되지 않는 오류 수정

#### Change

- 문자열 형식의 serviceId를 정수형으로 변경
- 프로토콜의 service_code를 service_id로 변경하고 관련 코드 수정
- RESTful 지원 API의 내부 구현과 사용법 변경



### 0.9.4 (2019.05.24)

#### New

- 콘텐츠에서 사용하는 프로토콜을 파일 단위로 등록할 수 있는 기능 추가
- 내부 패킷 처리를 효율적으로 하기 위함
- MoveService 기능 추가

#### Fix

- 룸 매치 메이킹에서 사용하는 비교 연산자(Comparator)의 오류 수정

#### Change

- Authentication 응답에 최근 로그인 정보를 추가하여 다음 로그인 시에 활용 가능
- setReady()를 명시적으로 호출해서 onReady 상태로 넘어가도록 변경
- LocationNode가 CommunicationNode의 onPrepare 시점에 초기화 되도록 변경



### 0.9.3 (2019.04.10)

#### Fix

- AsyncAwait.call() API에서 timeout이 발생하면 NPE(Null Pointer Exception)도 발생하는 문제 수정
- Epoll이 활성화되면서 멈추는 문제 수정
- 로그인 시 발생하는 Invalid System Target Location 오류 수정

#### Change

- ManagementNode는 이제 더 이상 엔진 사용자가 확장해서 사용할 수 없음
- 대신 ServiceNode를 사용
- Admin에서 Machine 정보의 등록, 수정, 삭제가 요청하는 순간 바로 적용되도록 변경



### 0.9.2 (2019.02.11)

#### New

- Admin을 이용한 예약 공지, 예약 점검 기능 추가
- Admin을 이용한 White IP, White User, White Device 기능 추가

#### Fix

- AsyncAwait.run() API에서 잘못된 방식으로 예외가 처리되던 코드 수정
- ManagementNode가 다른 Node들보다 먼저 초기화가 되도록 수정
- 채널 ID를 사용하지 않는 경우, 일부 채널 관련 publish가 누락되는 문제 수정
- CreateRoom, JoinRoom에 대해 콘텐츠가 실패를 반환할 경우에 잘못된 에러코드가 헤더에 실려가던 문제 수정
- 패킷 헤더 생성 시에 발생하던 메모리 릭 수정
- 룸 매치 메이킹에서 발생하던 메모리 릭 수정

#### Change

- onLogout() 콜백에 payload 추가
- ResultCode를 수정함에 따라 몇몇 프로토콜에서 중복 필드를 제거
- Dynamic Module 사용성 개선



### 0.9.1 (2018.11.12)

#### New

- 매치 메이킹을 전담하는 MatchNode 추가
- 유저 매치 메이킹에 refill 기능 추가
- 유저 매치 메이킹의 새로운 타입인 파티 매치 메이킹을 추가
- Admin의 Machine 정보에서 노드 종류별로 부가정보를 추가
- 유저가 동일한 DeviceId와 UserId로 재로그인 할 때 호출되는 onLoginByOtherConnection() 콜백 추가

#### Fix

- Dynamic Module 버그 수정
- Admin 접속이 안되는 문제 수정
- User의 reply 버그 수정
- Timer 추가/삭제 오류 수정
- AsyncAwait.run()에 누락된 timeout 적용
- 노드 Shutdown 시에 발생하던 오류 수정
- 룸 매치 메이킹을 사용하지 않음에도 불구하고 매칭 정보를 삭제하면서 NPE(Null Pointer Exception)가 발생하는 문제 수정

#### Change

- 일부 인터페이스의 위치 변경
- 서로 다른 DeviceId로 로그인 시에 중복 로그인 처리 대신 재로그인 처리가 되도록 변경
- 룸 매치 메이킹의 onMatchRoom() 콜백에서 추가한 payload가 이후의 흐름에 포함되도록 수정



### 0.9.0 (2018.06.27)

#### New

- 유저 매치 메이킹 기능 추가
- 채널별 유저와 방 목록 기능 추가
- Embedded Redis 추가
- Reconnect 기능 추가
- (구) Test Agent 추가

#### Fix

- Account 단위의 중복 로그인 처리 이슈 수정

#### Change

- OracleJDK에서 AdpotOpenJDK로 변경
- Node(스레드) 단위로 ZMQ 통신하던 것을 프로세스 단위로 변경



### 0.8.6 (2018.06.28)

#### New

- Send from Session to Management 기능 추가
- Request from Session to Management 기능 추가
- User MatchMaker 추가
- Channel User List 기능 추가
- Custom Serializer 기능 추가
- RoomFinder에 Custom Serializer를 plug-in 가능

#### Fix

- 채널 유저 매니저에서 채널이 topic으로 사용되는 publish에 대한 빈 문자열 예외 처리 추가
- 채널이 topic으로 사용되는 publish에 대한 빈 문자열 예외 처리 추가
- AutoJoinRoom을 사용하지 않는 환경에서 delNamedRoomInfo() 와 delRoomInfo() 호출 시에 DefaultRoomFinder로 인해 발생하는 문제 수정
- RoomFinder에서 null 체크 누락으로 발생하는 NullPointerException 수정
- 유저 매치메이킹에서 timeout된 요청이 제거되지 않는 문제 수정
- redis에 password가 설정되어 있는 경우에 대한 처리 추가
- cfgManagement의 setRestIp() 오류 수정
- Service 노드에서 Http 핸들링이 안되는 이슈 수정.
- 브랜치: 0.8.6.2_fix_service_node_rest_handling
- 태그: tardis-0.8.6.2.1
- AutoJoinRoom 처리 시에 gameId가 설정된 경우의 오류 수정
- MatchMaker 수정 및 예제 보강
- ISession::onDisconnect() 누락 수정
- Communication Node 간 Node 정보 교환 오류 수정
- FindRoomList 성능 이슈 해결
- Shutdown 버그 수정
- BaseNode가 publishToNode 메시지를 못 받는 문제 수정
- FindRoomList 성능 이슈 해결
- Shutdown 버그 수정
- BaseNode가 publishToNode 메시지를 못 받는 문제 수정
- NamedAutoJoinRoom 버그 수정
- UpdateRoomInfo/DelRoomInfo가 해당 Node에서 즉시 반영되도록 수정
- 파이버 내에서 blocking call을 호출하는 부분에 의해 출력되던 Error 수정

#### Change

- FindRoomList는 전체 방 목록 container를 한번에 serialize/deserialize하도록 수정
- 기존의 LocationException, TardisException 등을 RuntimeException으로 변경, Exception 정리
- forceLeaveRoom은 kickOutRoom으로 rename
- forceLogout은 kickOut으로 rename
- tardisConfig 구성 변경



### 0.8.5 (2017.12.18)

#### New

- JWT 인증토큰을 사용한 HTTP 세션 인증 및 관리 기능 추가
- 동일 ID 사용하여, 여러 Service 중복 로그인할 수 있는 기능 추가
- 하나의 GameNode에 여러 개의 RoomType을 사용할 수 있느 기능 추가
- Server의 모든 정보를 웹으로 볼 수 있는 기능 추가 (NodeInfoPage)

#### Fix

- 타이머 제거가 안되는 버그 수정
- onPostLeaveRoom 이 2번 호출되는 버그 수정

#### Change

- 고스트 유저 처리 개선
- 고스트 유저 로깅 추가
- Packet compress 로직 개선
- RestObject 개선, 사용 함수 추가
- 패킷의 에러 처리 변경, 프로토콜 헤더에 에러코드를 담아서 처리



### 0.8.4 (2018.01.10)

#### New

- Bootstrap 적용
- NGB Bootstrap 과 같은 형태로 구현
- 구현부 각각의 클래스의 class 인스턴스만 명시하여 코드를 단순화
- 여러 개의 메시지를 주고받을 수 있는 Multi Dispatch 기능 추가
- ServiceNodeSender(이전 CustomServiceSender) publishToLobby 메서드 추가

#### Change

- AsyncAwait, RAsyncAwait 이름 변경
- Bootstrap RoomFinder 설정을 space로 옮김
- Cache, Custom Exception 이름 변경 (Location, Service)
- Payload 클래스에서 getFirst 메서드를 통해 첫 번째 Packet 을 가져올 수 있도록 수정
- ServiceNodeSender 의 requestToLobby 메서드 제거(동일 기능의 requestToNode 메서드와 중복 되는 이슈)
- Packet 클래스의 makeDecompress 메서드 호출 시 async 사용하지 않도록 수정
- UserSender의 reply 메서드 오버로딩(Packet 리스트 처리)
- 일부 클래스 이름 변경
- AsyncCall -> AsyncWaitingCall
- AsyncRun -> AsyncWaitingRun
- ReverseAsyncCall -> RAsyncWaitingCall
- ReverseAsyncRun -> RAsyncWaitingRun

#### Fix

- Session IP 오류 수정



### 0.8.3 (2017.10.26)

#### New

- REST 핸들링 기능 추가
- TardisConfig.json 에 permissionGroups 으로 REST 핸들링에 대해, Path & IP 기반의 ACL 기능 구현
- Channel List (채널 이름 목록), Channel Information (특정 채널의 상세 정보)
- Space LobbySender 에 requestToCustom, sendToCustom 메서드 추가
- Space 노드 Management 정보에 "State" 항목 추가
- Ilobby , ISessionLobby, IService 인터페이스에 아래 void onShutdown() throws SuspendExecution 추가
- Node 가 shutdown될 때 contents에서 resource를 해지할 수 있도록 onShutdown() 인터페이스가 추가
- ServiceNodeSender에 publishToNode 함수 추가

#### Fix

- Management의 JMX Management API 수정 및 추가(SessionGateway, CustomGateway)
- Channel List(채널 이름 목록), Channel Information(특정 채널의 상세 정보) 기능 추가
- Session, Custom 노드의 REST 핸들링 로직 수정
- Gateway 노드(Session, Custom) isConnectableType 메서드의 리턴값 수정
- SessionGateway 및 CustomGateway 라운드 로빈 이슈 및 JMX 리턴 정보 수정
- 캐시 노드 targetMap 키값 사용하는 코드 수정
- SessionGateway 노드 REST RoundRobin 코드 수정
- 로그인 프로세스 개선 (Invalid Target Location 에러 로그 관련 수정)
- 룸을 삭제하는 delRoomInfo 호출 시, 해당 노드 상태가 READY 가 아니라면 리턴 처리

#### Change

- Session 노드와 클라이언트의 End-Point를 하나로 통합 (Session Gateway)
- Custom 노드의 REST End-Point를 하나로 통합 (Custom Gateway)
- 파이버 직접 사용 코드를 ReverseAsyncCall로 변경
- SpaceSingular Dispatch 타임스탬프 추가
- onTimer 등록하고, 주기적으로 체크 (10분)
- 타임스탬프가 초과된 유저는 internalLogout 처리
- Async 타임아웃 적용
- 고스트 유저 처리 로직 추가 및 개선, TardisConfig.json에 필요한 값들 추가
- 각 Response 프로토콜의 result_code 형식을 기존 integer 값에서 enum으로 변경하고, 종류를 간소화. is_payload 필드를 제거
- 클래스, 패키지, 함수들과 cfg 내용이 rename
- Request, Response 프로토콜에서 복수의 payload를 처리할 수 있도록 class Payload 데이터 타입이 Packet 으로 변경, class OutPayload 제거
- 유저 Transfer-In 인터페이스의 파라미터 타입을 byte 배열로 변경



### 0.8.2 (2017.09.21)

#### New

- CustomService에 RestObject 기능 추가
- CustomService에 RestHandler 추가

#### Fix

- Cache에서 유저 정보가 지워지지 않는 버그 수정
- 엔진 내부에서 발생하던 NPE(Null Pointer Exception)에 대한 예외 처리 추가
- 패킷 전송에 CID(Connection ID)를 제대로 적용하지 못하던 문제 수정

#### Change

- Custom 모듈 네이밍을 모두 새롭게 변경
- 예) CustomLobby -> CustomService
- Custom 모듈 인터페이스 일부 변경
- CustomServiceSender의 sendToUser()와 requestToUser() API에 serviceId 매개변수 추가