## Game > GameAnvil > 릴리스 노트 > GameAnvil
### 2.1.0 (2025.06.30)
#### New
##### 의존성 관리와 spring boot 추가 
* GameAnvil 의존성 관리에 spring boot 를 사용하여 Bean 의 추가와 주입을 할 수 있습니다.
* 기존 인터페이스를 사용하여 엔진에 등록하였던 코드도 이제 어노테이션을 선언하여 엔진에 등록합니다.
#### Remove
##### Gateway, Connection에서 메세지 처리 Deprecated 
* 사용성이 낮고 직관적이지 않은 API 로 이후 릴리즈에서 제거될 예정입니다.

##### Gateway, Connection에서 Timer, Topic 삭제
* 사용성이 낮고 직관적이지 않은 API 들이 제거되었습니다. 
* Room, User등 일반적으로 사용하는 게임노드에서는 그대로 사용할 수 있습니다.
#### FIX
* ChannelCountInfo API 요청 시 예상과 다른 오류 메세지가 나올 수 있는 문제를 수정하였습니다.
* 서버 종료 시 까지 기본 대기시간이 감소하였습니다.
* 프로토 버퍼 사용 시 전체 경로가 아닌 파일 이름을 사용하도록 수정하여 같은 파일이면 충돌이 발생하지 않도록 수정하였습니다.
* 내부적으로 더이상 사용하지 않는 프로토 버퍼 패킷이 삭제되었습니다.
* 발생하지 않지만 선언한 Checked Exception 선언이 제거되었습니다.
* 가끔씩 전체 클러스터링을 하지 않았는데 All nodes are Ready 출력이 되는 문제를 수정했습니다.
* 토픽 추가 시 내부적으로 디버그 로그가 추가되었습니다.
* 내부적으로 사용하고 있는 Ipc 노드가 삭제되었습니다. 기존 Ipc 의 기능은 IpcNetwork 에서 관리합니다 
* Node 관련 API에서 가끔 ConcurrentModificateException 발생하는 문제가 수정되었습니다.
* 서버를 오래 실행 시 가끔 overflow가 발생할 수 있는 문제가 수정되었습니다.
* 내부 포트로 알 수 없는 패킷이 들어올 때 크래시가 발생할 수 있는 문제가 수정되었습니다.


### 2.0.0 (2024.12.04)

#### New
##### Java 21
* GameAnvil 2.0 은 Java 21 혹은 그 이상 버전에서만 동작합니다.
* 엔진팀에서는 현재 LTS 인 Java 21 사용을 권장합니다.

##### Virtual Thread 도입
* Quasar 의존성이 제거되어 throws SuspendExecution을 사용하지 않고 코드를 작성할 수 있습니다.
* Java 21 의 동작에 맞게 GameAnvil의 모든 사용자 호출 코드에서 비동기 작업을 사용할 수 있습니다.
* Java 21 사용 시 **synchronized** 코드를 사용하지 않도록 주의하십시오 Pinning 이슈가 발생할 수 있습니다.
    * 이 제한은 Java 24에서 해제될 수 있습니다. [https://openjdk.org/jeps/491](https://openjdk.org/jeps/491)
* GameAnvil 에서는 기존 Node 와 코드를 쉽게 작성할 수 있도록 커스텀한 Virtual Thread를 사용합니다. 이 스레딩 모델은 1 Thread N Virtual Thread로 기존 Fiber와 동일합니다. 
    * 다음의 JVM 옵션을 실행 시 추가 하십시오.
    * `--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.invoke=ALL-UNNAMED`
* Virtual Thread의 자세한 동작은 [여기](https://openjdk.org/jeps/444)를 참고하십시오.

##### 다수의 클라이언트에게 메세지를 보낼 수 있는 API 추가
* Room.sendToClients API가 추가되었습니다.

##### GameAnvilConnector 2.0
* GameAnvil 2.0에 맞는 GameAnvilConnector 2.0을 출시 하였습니다.
* 신규 커넥터를 확인 부탁드립니다.

##### MatchNode 의 SafePause 기능 추가
* MatchNode 에서 SafePause 기능을 활용할 수 있습니다.
* MatchNode 의 SafePause 기능은 시작 노드(src)와  이동 노드(dest)가 1:1일때만 실행 가능합니다.
* SafePause 노드의 등록 및 실행 방법은 기존 GameNode와 동일합니다.
* MatchNode SafePause와 관련된 다음 Management HTTP API가 추가되었습니다.
1. POST /management/matching-group-update
    *  body: ``` {"nodeId": <Number 노드 아이디>, "matchingGroup": "<String 매칭그룹>" }```
    *  대상 MatchNode에서 인자로 받은 matchingGroup을 제어하도록 합니다.
2. GET /management/matching-queue-clear?nodeId=&lt;Number 노드 아이디&gt;
    * 대상 MatchNode의 매칭 큐를 모두 비웁니다. 만약 노드 아이디가 0이라면 모든 노드의 매칭 큐를 비웁니다.

##### 패킷 파싱 중 오류 발생시 오류 로그를 출력

#### Remove
##### Quasar 의존성 제거
* Quasar와 관련 코드가 제거되었습니다. 
* GameAnvil 1.4 에서 제공한 비동기 기능 대신 Java 21 의 Virtual Thread를 사용합니다.

##### MySql 의존성과 관련 코드 제거 
* `com.github.jasync-sql:jasync-mysql` 의존성과 엔진의 비동기 관련 코드가 제거되었습니다. 
* GameAnvil 1.4 에서 제공한 기능 대신 위 라이브러리를 직접 사용하여도 비동기로 동작합니다.

##### Redis 의존성과 관련 코드 제거
* `io.lettuce:lettuce-core` 의존성과 엔진의 비동기 관련 코드가 제거되었습니다. 
* GameAnvil 1.4 에서 제공한 기능 대신 위 라이브러리를 직접 사용하여도 비동기로 동작합니다.

##### HttpClient 의존성과 관련 코드 제거
* `org.asynchttpclient:async-http-client`, `org.apache.httpcomponents.client5:httpclient5` 의존성과 엔진의 비동기 관련 코드가 제거되었습니다. 
* GameAnvil 1.4 에서 제공한 기능 대신 위 라이브러리를 직접 사용하여도 비동기로 동작합니다.

##### MultiRequest 제거
* MultiRequest 대신 사용자 지정 프로토 버퍼를 사용할 수 있습니다.
* 여러 대상 또는 여러 패킷을 동시에 요청할 때 MultiRequest 대신 새로 추가된 Future 방식의 Request API 를 사용할 수 있습니다.


##### TimerHandler 의 Object 인자 제거
* 활용하지 않는 Object 인자가 삭제되었습니다.

##### addTopics(List&lt;T&gt;), removeTopics(List&lt;T&gt;) 삭제  
* 대신 addTopic(String), removeTopic(String) 을 사용합니다.


#### Change
#####  응답을 받는 API 의 반환 값이 Future 로 변경 
* GameAnvil의 API도 다른 많은 비동기 방식 API들처럼 Future를 리턴하는 방식으로 변경되었습니다. 이제 더욱 자유롭게 코드 흐름을 만들 수 있습니다. 
* GameAnvil에서 제공하는 Async API를 사용할 필요가 없어졌습니다. GameAnvil에서 Virtual Thread를 공식 지원하기 때문에, 블로킹 코드를 호출하더라도 Java 21을 지원하는 코드라면 Platform Thread가 아닌 Virtual Thread가 블록이 됩니다. 이제 Async API는 지원하지 않습니다.
```java
// GameAnvil 2.0 이전 버전 
Packet packetResponse = user.requestToNode(nodeId, myRequest1);
ListenableFuture<Response> httpFuture =  getHttpClient().executeRequest(myRequest2);
// Response httpResponse = httpFuture.get(); // !Java 11 일 때 Platform Thread 블락 발생!
Response httpResponse = Async.awaitFuture(httpFuture.toCompletableFuture());
```
```java
// GameAnvil 2.0 
Future<Packet> packetFuture = user.requestToNode(nodeId, myRequest1);
Future<Response> httpFuture = getHttpClient().executeRequest(myRequest2);
Packet packetResponse = packetFuture.get();  
Response httpResponse = httpFuture.get();  // Java 21 에서는 Virtual Thread 만 블락
```
* 이제 기존 GameAnvil에서는 불편한 점이 있었던 다음 기능들을 개선할 수 있게 되었습니다.
    * 동시에 여러 곳으로 요청을 보내는 기능 개선
    * 요청을 보낸 후 받기 전까지 필요한 작업 실행 개선
* 기존 버전과 활용성을 유지해야 하는 코드라면 반환받은 Future에 즉시 get 을 호출하십시오
> Platform Thread : 기존 Java에서 사용한 Thread. OS 스레드 혹은 자바 스레드입니다
> Virtual Thread: Java 21에서 추가된 새로운 Thread입니다 이전 버전 GameAnvil의 Fiber 와 유사한 동작을 합니다 자세한 동작은 [여기](https://openjdk.org/jeps/444)를 참고하십시오

##### Handler 실행이 Context 기반으로 변경 
* Handler 실행을 각 대상에 맞는 `DispatchContext` 기반으로 변경하였습니다.
* DispatchContext 는 각 요청마다 새로 생성합니다.
* 이제 `reply` 함수는 DispatchContext에서 제공합니다.
* 예를 들어 User 에서 EchoReq를 처리하는 메세지 핸들러는 다음과 같습니다.

```java
public class _EchoReq implements IMessageHandler<IUserDispatchContext, EchoReq> {
    @Override
    public void execute(IUserDispatchContext ctx, EchoReq request) {
        MyUser user = ctx.getUser();
        ctx.reply(응답_프로토_버퍼);
    }
}
```

* DispatchContext 는 요청 자체의 정보를 가지고 있는 값입니다. 기존 GameAnvil 은 Handler 코드 에서 reply 를 보낼 때 반드시 현재 흐름에서만 보낼 수 있었습니다.
    * 이 제한을 무시하고 비동기 작업에서 reply 를 전송 시 알 수 없는 행동을 할 수 있었습니다.
    * 요청 정보가 대상(여기서는 User)에 저장하여 발생한 문제인데 서버에 다른 요청이 들어오면 이전 요청 정보를 덮어 쓰기 때문입니다.
* 문제가 발생할 수 있는 간단한 코드를 아래 예제에 작성하였습니다.
```java 
// 이전 버전의 GameAnvil

// 클래스 선언 생략. 핸들러
public void execute(MyUser user, EchoReq request) throws SuspendExecution {
    user.postJob((obj) -> {
        // 어떠한 비동기 작업 수행 후 응답을 보낼 때 정상적으로 동작하지 않을 수 있습니다.
        // 핸들러 execute 메서드가 끝난 후 다른 핸들러가 reply 정보를 변경 시킬 수 있기 때문입니다.
        user.reply(..응답_패킷..);
    }, null);
}
```

```java 
// 새로운 GameAnvil 2.0

// 클래스 선언 생략. 핸들러
public void execute(IUserContext ctx, EchoReq request) throws SuspendExecution {
    IUserContext userContext = ctx.getUserContext();
    userContext.runOnNextMsgLoop(() -> {
        // 이제 정상적으로 작동합니다.
        ctx.reply(..응답_패킷..);
    });
}
```

##### Base 클래스 대신 인터페이스로 변경
* 사용자 구현 Base 클래스가 인터페이스로 변경되었습니다.
* 기존 Base 클래스에서 제공하던 메서드 구현체는 클래스 Context에서 제공됩니다.
* 예를 들어 BaseUser에서 사용하던 API 는 다음과 같이 마이그레이션 할 수 있습니다.
* 기존 PacketDispatcher, Base 클래스 정의 Annotation은 삭제되었습니다.
```java
public class MyUser implements IUser {
    private IUserContext userContext;
    public void onCreate(IUserContext ctx) {
        this.userContext = ctx;
        
        userContext.send(..패킷..);  // 이렇게 IUserContext 를 활용하여 
                                     // 기존 Base 클래스에서 제공하던 기능을 그대로 활용할 수 있습니다.
    }
}
```


| 기존 |변경  | 비고 |
| --- | --- | --- |
| BaseConnection | IConnection, IConnectionContext | |
| BaseSession|  ISession, ISessionContext|  |
| BaseGatewayNode | IGatewayNode, IGatewayNodeContext |  |
| BaseGameNode | IGameNode, IGameNodeContext|  |
| BaseUser | IUser, IUserContext|  |
| BaseRoom | IRoom, IRoomContext |  |
| BaseSupportNode | ISupportNode, ISupportNodeContext |  |

##### 사용자 지정 클래스 / 메세지 처리자 등록 방법 변경
* 사용자 지정 클래스와 메세지 처리자의 등록 방법이 각자 따로 있었던 것을 한 곳에서 등록하도록 변경하였습니다.
* 간단한 사용 방법은 다음과 같습니다.

```java
var gameAnvilServer = GameAnvilServer.getInstance();
var builder = gameAnvilServer.getServerTemplateBuilder();

builder.connection(MyConnection::new, config -> {
     config.protoBufferHandler(MyProto.HelloProto.class, new _HelloMessageHandler());
});

var gameServiceBuilder = builder.createGameService("MyGame"); // 게임 서비스 정의
gameServiceBuilder.user("MyUserType", MyGameUser::new, config -> {
    config.protoBufferHandler(MyProto.UserHello.class, new _UserHelloMessageHandler());
});
```



##### ServiceId 대신 ServiceName 사용
* 엔진 사용자가 인지하기 어려운 ServiceId 를 받는 API 가 사용자가 입력한 ServiceName 을 그대로 받도록 변경되었습니다.

##### ProtoBuffer 4.28.3 사용
* ProtoBuffer 를 3.x 버전으로 다운그레이드 시 정상적으로 동작하지 않을 수 있습니다.
* ProtoBuffer 의존성, protoc, 이미 빌드 된 ProtoBuffer 파일의 교체가 필요합니다.


##### 1 Thread 에서 N 개의 GameNode 를 실행 가능, 게임 노드의 ChannelID 설정 변경 
* 여러 GameNode를 실행 시킬 수 있도록 게임 노드의 ChannelID 설정이 변경되었습니다. 
```
 "channelIDs": [
        ["ch1"]
        ["ch2"],
]
```
* 위과 같이 작성 시 기존처럼 1 Thread = 1 GameNode 입니다.
```
 "channelIDs": [
        ["ch1", "ch2"]
]
```
* 위과 같이 작성 시 1 Thread = 2 GameNode 입니다.


##### AutoIp 실패 시 서버가 강제 종료되도록 변경
* 초기 정상적으로 동작하지 않는 상태로 멈추는 대신 사용자가 쉽게 인지할 수 있도록 강제 종료합니다.

##### Timer 인터페이스 수정
* 이해하기 어려운 기존 타이머 인터페이스 대신 표준 라이브러리의 ScheduledExecutorService 에서 제공하는 함수와 유사하게 Timer 인터페이스를 수정했습니다.

```
scheduleTimer - 1회
scheduleTimerWithFixedDelay - N회, 최근 구동 이후 딜레이
scheduleTimerAtFixedRate - N회, 고정 딜레이
```

##### Reply 를 2번 이상 보낼 수 없도록 변경
* 기존 Reply 를 2회 이상 보낼 시 알 수 없는 문제가 발생하던 현상이 해결되었습니다.

##### User, Room 에서 Node 를 접근하는 방법 변경
* 이제 getGameNode 호출 시 Node 를 직접 반환하는 대신 INodeView 를 반환합니다.
* INodeView 에서 기존처럼 Node 를 직접 접근할 수 있지만 메서드를 넘겨 Node 동작을 하는 것을 권장합니다.
    * Node 와 User, Room 의 Platform Thread 는 동일하지만 Virtual Thread 는 각자 다릅니다. 그러므로 User 에서 Node 에 정의된 비동기 작업을 수행 하는 등 다른 영역의 비동기 작업을 호출 시 의도한 대로 동작하지 않는 문제가 있어 사용 시 주의가 필요했습니다. INodeVIew 에서는 Node 의 직접 접근을 막고 대신 Node 에서 실행할 메서드를 넘기도록 하여 이러한 문제점을 해결합니다.
* GameAnvil의 고급 사용자에게는 INodeView의 이러한 제약이 불편할 수 있습니다. 단순한 동기 호출 메서드(get/set 등)일 때는 앞서 설명한 동기화의 문제점이 없기 때문에 `INodeVIew.getUnsafeNode` 메서드를 활용하여 Node 를 직접 접근할 수 있습니다. 이렇게 getUnsafeNode 를 통해 노드를 직접 접근 시 동기화 문제가 발생하지 않도록 주의하여 사용하십시오.

##### Room 에서 User 의 목록 제공
* 이제 Room 이 가지고 있는 User 의 목록을 제공 받을 수 있습니다.

##### 콜백의 의미가 좀더 명확하게 나타나도록 콜백 이름 수정
* 다음 콜백이 추가/변경되었습니다.

| 대상 |기존  |변경  | 비고 |
| --- | --- | --- | -- |
| Session | onPreLogin |onBeforeLogin  |
| Session  | onPostLogin | onAfterLogin |
| Session  |  onPostLogout| onAftterLogout |
| User | onPostLogin | onAfterLogin |
| User  | onPostLeaveRoom |  onAfterLeaveRoom |
| User  |  canTransfer | canTransferOut  |
| User  |  onTransferInTimerHandler | onTransferIn | 기존 onTransferIn 메서드로 병합 | 
| User  | onPostTransferIn  | onAfterTransferIn  |
|  Room | onPostLeaveRoom | onAfterLeaveRoom |
| Room  | onLeaveRoom | canLeaveRoom | LeaveRoom 조건 검사|
| Room  |  없음 | onMatchPartyCancel | 매칭 취소 시 호출|
| RoomMatchMaker | onPreMatch | 삭제 | onMatch 로 병합 |
| RoomMatchMaker | onPostMatch | 삭제 |  onMatch 로 병합 |

#### Fix

* Request API 사용 중 실패 시 오류 로그에 패킷 정보 추가
* Request API 사용 중 대상을 찾을 수 없을 때 Timeout 대신 즉시 실패 응답
* 대상 노드 타입별로 있던 Send, Request API를 대상 노드의 Id를 받는 단일 API로 통합
* 노드 또는 서버 종료 시 데이터 정리 프로세스 강화
* 엔진에서 주기적으로 남기는 로그의 마커 추가
* 노드 간 연결이 끊어졌을 때 대상의 로그를 남기도록 수정
* 특정 상황에서 ConcurrentModificationException 발생 수정
* 엔진 내부 데이터 크기 최적화
* 가끔 엔진 내부의 오류 로그가 정상적으로 나오지 않는 문제 수정 
* 내부 타이머 최적화
* 내부 Location 동작 개선
* Gateway Bind 실패 상황에서 서버가 가끔 좀비 상태가 될 수 있는 문제 수정 
* Timer 인자가 long 으로 변경되어 인자가 Integer 값 이상일 시 정상적으로 동작하지 않는 문제가 수정
* Cpu 코어 수 보다 Node 수가 많을 때 서버 시작 시 경고 발생하도록 수정
* 로그아웃으로 방에서 나갈 때 가끔 정상적으로 처리 되지 않는 문제 수정
* 특정 상황에서 GameAnvil 서버 내부 통신이 정상적으로 동작하지 않을 수 있는 문제가 수정
* 내부 메모리 최적화


---

### 1.4.2 (2024.02.26)

#### New
* Safe-Pause 기능을 개선했습니다. 
  * 이미 Safe-Pause가 진행중인 경우라도, 진행중이 아닌 노드를 출발지/도착지 노드로 지정하여 새로운 Safe-pause를 실행할 수 있도록 개선했습니다. 
  * GameNode 와 더불어 MatchNode 도 Safe-Paue를 지원합니다. 

#### Fix
* 엔진의 로그 내용을 보강하고 가독성을 개선했습니다. 
  * request 실패시 보내는 패킷의 자세한 정보 추가했습니다.
  * 시스템 정보 로그에 대한 마커를 SYSTEM_INFO 하나에서 SYSTEM_INFO와 SYSTEM_WARN으로 분리하였습니다.
  * MultiRequest에서 개별 요청이 실패할 경우 어떤 요청이 실패했는지 로그를 남기도록 했습니다.
  * 머신간 연결 상태와 관련된 로그 내용을 보강했습니다. 

#### Change
* GameAnvilConfig.json 에서 managementIp와 더불어 managementPort도 설정할 수 있도록 변경되었습니다.

------

### 1.4.1 (2023.12.13)

#### New
###### 엔진 Protobuf 버전을 3.24.1로 업데이트
###### Protobuf 관련 편의 기능 추가
  * send, reply에서 Protobuf를 직접 사용할 수 있습니다.
  * 패킷을 직접 Protobuf로 변경할 수 있습니다. (`MyProto.MyMessage resMsg = resPacket.toProtoBufferMessage();`)
  * 대부분의 상황에서 retain release를 호출하지 않아도 정상적으로 동작합니다.
###### Protocol 등록 시 index를 지정하지 않아도 되도록 개선 
  * 기존 number는 엔진에서 자동으로 할당합니다.

#### Fix
* 서버에서 Request 호출 시 대상이 없으면 즉각 실패 응답을 하도록 개선했습니다.
    * 엔진의 requestToGameUser와 같은 요청(request) API를 호출했을 때 대상을 찾지 못하면 Timeout까지 기다리는 대신 즉시 실패를 반환하도록 개선했습니다.
* 엔진의 로그 내용을 보강하고 가독성 개선
    * 엔진 로그 중 DEBUG와 TRACE 레벨을 모두 체계적으로 분리하고 내용을 이해하기 쉽게 정리했습니다.
    * 유저와 관련된 엔진의 로그 내용을 보강했습니다.
    * 일부 엔진 오류 메시지를 이해하기 쉽도록 개선했습니다.
* 매니지먼트 노드는 로케이션 노드가 구성될 때만 함께 구동되도록 개선했습니다.
    * 매니지먼트 노드가 로케이션 노드 구성 위치에서 자동으로 구동되므로 더 이상 사용자가 직접 설정하고 관리할 필요가 없습니다. 또한 구동 순서도 신경쓸 필요가 없습니다.
* 로그인 과정에서 잘못된 채널ID를 입력할 경우에 대한 응답을 개선했습니다.
    * 이제 막연한 SystemError 응답 대신 로그인 실패 응답을 주도록 수정하여 더욱 명확해졌습니다.
* 등록되지 않은 프로토콜이 페이로드에 담겨 게임 서버로 전달될 경우 서버에서 오류를 처리하고 클라이언트로 응답을 전달하도록 개선
    * 기존 서버에 WARN 로그만 남기던 방식의 경우 디버깅 과정에서 로그를 놓칠 수 있는 문제가 있어 클라이언트로 응답을 전달하도록 개선했습니다.
* Apache HttpComponent 기반의 새로운 HttpClient2를 제공합니다.
    * 기존의 AsyncHttpClient 오픈소스가 더 이상 업데이트되지 않으므로 선택적으로 사용할 수 있도록 새로운 라이브러리 기반의 HttpClient2를 제공합니다.
* 페이로드에서 압축 패킷을 지원합니다.
    * 이제 페이로드에도 압축 패킷을 사용할 수 있도록 개선했습니다.
* 매치 메이킹 실패 시에 클라이언트로 알림을 전달합니다.
    * 매치 메이킹이 실패했을 때에도 클라이언트로 적절한 결과를 전달하도록 개선했습니다.

#### Change
* 설정
    * GameAnvilConfig.json에서 ip 항목 대신 ipcIp와 managementIp를 사용합니다.
    * BaseChannelInfo에서 정상적으로 직렬화하지 않는 문제를 수정했습니다. BaseChannelInfo 대신 GameAnvilServer에서 setObjectSerializer를 호출하여 직렬화할 수 있습니다.
* matchUser, matchParty 호출 시 Payload를 넣지 않습니다. 자체적으로 보관합니다.
 * 모든 로케이션 노드(마스터 노드)가 클러스터링이 완료된 이후에만 로케이션 노드로 요청을 보낼 수 있도록 수정했습니다. 모든 로케이션 노드가 클러스터링이 완료되었는지 확인하고, 일정 시간 내에 전체 클러스터링이 완료되지 않을 경우 오류 로그를 남기는 기능을 추가했습니다.
 * 유저 매칭 시 랜덤 채널에서 매칭된 룸이 생성되던 기존 방식에서 매칭을 요청한 채널 중에서 매칭된 룸이 생성되는 방식으로 변경했습니다.
  
------

### 1.3.1 (2023.04.20)

#### New

#### Fix

* 새로기동된 노드의 채널정보가 갱신되지않는 이슈 수정

#### Change

- Console에서 확인 하는 Support 노드의 활성화 상태체크 API 변경
- Location 노드가 설정된 서버에서만 Management 노드가 기동 하도록 변경

------

### 1.3.0 (2022.12.27)

#### New

###### Console 1.3과의 연동 
GameAnvil 1.3은 완전히 새로워진 Console 1.3과 완벽하게 연동됩니다. 콘솔을 통해 각 노드의 상태를 직관적으로 관리하고, 명령을 내릴 수 있습니다. Safe Pause를 진행하거나 특정 노드를 생성하고 제거하는 등의 기능을 콘솔을 통해 편리하게 시도할 수 있습니다.
</br> ※ 이전 버전의 GameAnvil은 Console 1.3에서 사용할 수 없습니다.

###### 사용하기 편리한 동기화 기능
동기화 컴포넌트 제공으로 Unity 엔진과 연동 편의성이 대폭 강화되었습니다. 서버의 복잡한 로직 구현 없이도 원격의 클라이언트 간에 게임오브젝트의 여러 요소들이 자동으로 동기화 됩니다.

* GameObject 생성, 파괴 자동 동기화
* Animation 동기화
* Transform 동기화
* RigidBody 및 RigidBody2D 동기화
* 커스텀 값 동기화 기능

###### 성능 최적화
게임엔빌 1.3은 이전 버전에 비해 패킷 처리 성능이 5% 이상 향상되었습니다. 내부 동작 코드를 최적화하여 더 빠른 속도로 동작합니다.

#### Fix

- Safe Pause 관련 버그 수정
  - Safe Pause 진행 중 출발지 노드에 있는 방으로 매치메이킹이 되는 문제를 수정하였습니다.
  - 스트레스 테스트와 Safe Pause를 동시에 수행할 경우 MatchRoomException이 발생하는 문제 수정
  - Safe Pause 과정에서 방 전송을 할 때 TimerObject로 인해 NPE가 발생하는 문제 해결
  - Safe Pause 진행 시 룸매치메이킹이 실패하는 이슈 해결
  - Safe Pause를 통해 전송이 된 방의 도착지 노드에서 해당 방과 동일한 아이디의 방이 생성되는 이슈 수정
    - 룸 아이디 생성은 로케이션 노드에서 담당하도록 수정
  - 그 외, 유저 전송 및 방 전송과 관련된 버그 수정
    - 방 전송 실패 시 롤백이 되지 않는 문제 수정 
    - 방 전송 실패시 결과 패킷에 실패 에러코드를 적용해서 전달하도록 수정
    - 방 전송 실패시 생성한 룸과 유저를 노드에서 조회해서 정리하지 않고 직접 정리하도록 수정
  	- 방 전송 과정에서 클라이언트가 보낸 일부 메시지가 유실될 가능성 수정
    - 방이 다른 노드로 전송이 완료되었을 때, 타이밍 이슈로 기존 노드에 방이 남아 유저가 입장할 수 있는 상황을 수정
      - 이제 트랜스퍼 중인 방은 매치메이커의 방 목록에서 제외됨
      - 트랜스퍼가 완료되어서 제거되어야 하는 기존 방이나, 트랜스퍼가 실패한 방으로 입장하지 않도록 수정
    - 유저 트랜스퍼가 완료 되기 전에 재접속시 목적지 노드에서 유저 객체를 찾을 수 없는 문제 수정
    - 유저 트랜스퍼시 대상 노드에 대해 상태값을 확인하지 않도록 수정
    - 룸 트랜스퍼시에, 유저의 트랜스퍼 인 관련 콜백 함수가 호출되도록 변경

- Safe Pause 진행시 룸 매치메이킹에 실패하는 현상 수정
  - 트랜스퍼 중인 방 상태 종류에 TRANSFER_IN_PROGRESS 추가
  - 트랜스퍼 완료된 방이 PAUSE 상태로 바뀌지 않도록 수정
  - 방 입장이 룸 파이버에서 처리되려고 할 때 방이 트랜스퍼 중인 경우 방 입장 처리를 중지하고 트랜스퍼를 진행하도록 수정
	
* Room과 매치메이킹 관련 버그 수정
  - CreateRoom 메서드로 만든 방을 룸 매치메이킹에 등록한 후에 해당 방이 종료하면 예외가 발생하는 이슈 수정
    - 룸 매치시에 매치 룸을 찾지 못하였을 경우, 게임 유저에 매치 유저 카테고리를 설정하지 않도록 수정

  - 모든 유저가 나간 방에 타이밍 이슈로 새로운 유저가 입장할 수 있는 문제 수정
    - 유저 입장시에 빈 방인지 여부를 확인하도록 함
    - 마지막 유저가 방에서 나간 직후에 채널 정보 정리 및 MatchRefill 취소

  - 클라이언트에서 LeaveRoom 호출 시에 서버 에러가 발생하는 문제 수정
    - UserPostLeaveRoom 패킷 전송 주체를 게임 유저 파이버로 변경
  
  - 방이 종료된 상태에서 방을 사용하는 API 호출시의 예외인 ClosedRoomException 추가

* 게이트웨이 및 세션 관련 버그 수정
  * 게이트웨이 노드의 세션 객체가 전송(Transfer) 상태로 leak되는 이슈 수정

  - 세션 삭제 과정에서 예외가 발생할 경우 커넥션 객체가 제거되지 않고 남아 있을 수 있는 문제 수정

* 노드 관련 버그 수정
  - 임의의 노드를 종료할 경우 나머지 서버에서 에러 또는 경고 로그가 남는 문제 수정
    - 종료된 노드 정보를 서버간 공유하도록 수정
    - 종료된 노드 정보를 지울 수 있는 API 추가

  - Shutdown 상태인 노드에 다른 노드에서 패킷을 보내는 경우가 발생할 수 있는 문제 수정
    - 이제 종료 고지 후 실제 종료 까지 1초간 딜레이가 있게 됨

* 그 외
   * 클라이언트에서 요청에 대한 응답을 받지 못하는 이슈 수정

  - 잘못된 서비스명으로 서버를 구동할 경우에도 모든 노드가 Ready로 처리되는 문제 수정
    - 노드 갯수를 증가시키는 로직을 올바른 시점으로 변경
    - 잘못된 서비스명을 사용하여 노드를 실행 시키려고 하는 경우 노드매니저에서 바로 종료하도록 수정


  - 중복 로그인 중 강제 접속 종료 처리될 수 있는 버그 수정
    - DeviceId가 다르면서 커넥션 아이디, 게이트웨이 노드 아이디가 같을 때 기존 유저를 로그아웃 시키는 경우에 대해, 중복해서 세션을 닫지 않도록 수정

  - ghostTimeout 값이 제대로 적용되도록 수정

  - ghotsTimeout 값이 demandClientStateCheck 값 보다 적어도 3초 이상 크도록 제약 사항 추가

#### Change

- Safe Pause 고도화
  - 출발지 노드와 도착지 노드를 모두 선택하도록 변경
  - Safe Pause 시작시에 출발지 노드와 도착지 노드를 선택하지 않도록 변경
  - 리포트 기능 강화 : HTML 형식 외 json 형식으로 리포트를 생성하는 기능 추가

- Safe Pause 관련 프로토콜 명 변경 및 ReportSender의 동작을 리퀘스트 방식으로 수정
  - 변경 전 : PubNonStopPatchReportStartCommand / PubNonStopPatchReportStartNode
  - 변경 후 : NonStopPatchReportStartCommandReq / NonStopPatchReportStartCommandRes / PubNonStopPatchReportStartNode		

- 동일한 파이버로 리퀘스트를 보내면 예외 처리 되도록 변경

- BaseRoom 사용법 변경
  - BaseRoom에 onLeaveRoom 메서드의 오버로딩 추가
  - BaseRoom의 onPostLeaveRoom의 파라미터 제거로 사용성 변경

- BaseUser 사용법 변경
  - 유저가 룸에서 완전히 나간 후 호출되는 onPostLeaveRoom 메서드 추가

- Disconnected 상태값을 isDetectedDisconnected, isCompleteDisconnected로 세분화
  - isDetectedDisconnected는 서버에서 클라이언트의 연결이 끊겼다고 판단 하였을 때의 상태값
  - isCompleteDisconnected는 서버에서 연결을 끊는 처리가 완료되었을 경우의 상태값

- 고유한 HostId 생성 로직 보강
  - 사용자가 입력한 서브넷 마스크에 따라 HostId를 생성하도록 수정

- 엔진에서 사용하는 라이브러리 최신화

- UserTimer, RoomTimer에 대한 참조를 반환하는 API getUserTimer, getRoomTimer 추가

- DNS 관련 설정 삭제

---

### 1.2.0 (2021.07.13)

더 자세한 정보는 [배포 노트](https://nhnent.dooray.com/share/posts/sGAj_STlTEWDr5LPgKgIhg)를 참고

#### New

* 사용자 라이센스 적용
  
* [GameAnvil 사용자 라이센스](https://gameplatform.toast.com/kr/services/gameanvil/license)
  
* 노드 단위 구동 및 스케일링 지원

  * 런타임에서도 새로운 노드를 올릴 수 있는 기능 추가
  * 기존에 비해 좀더 안정적으로 클러스터링을 맺도록 개선
  * 노드 상태에 따른 노드 동작 개선
  * 런타임에 노드 단위로 구동하거나 종료할 수 있음

* 간소화된 부트스트래핑
  * GameAnvilBootstrap Deprecate 로 설정. 차후 버전에서 삭제될 예정.
  * GameAnvilBootstrap 대신 GameAnvilServer 인스턴스를 사용하여 GameAnvil 서버 실행.
  * 제공되는 어노테이션을 사용하여 BaseClass를 GameAnvil에서 읽어오도록 사용성이 개선됨.

```java
// GameNode 등록
@ServiceName("GameNode")
public class GameNode extends BaseGameNode {
}

// GameUser 등록
@ServiceName("GameNode")
@UserType("GameUserType1")
@UseChannelInfo
public class GameUser extends BaseUser {
}
```


* 비동기 MySQL 쿼리 지원
  * 비동기 sql 처리를 위해 Jasync-sql / X Dev API 클래스를 추가
  * 사용자가 쉽게 비동기 sql를 처리하도록 지원
  
* 사용자 가이드 문서 보강
  * 하나의 문서에 함축적으로 나열되어 있던 여러 중요한 주제들을 모듈화하여 문서의 질과 양을 업그레이드하였습니다.
  * 튜토리얼을 통해 쉽고 간단하지만 꽤 멋진 직소 퍼즐 게임(서버&클라이언트)을 직접 개발해 볼 수 있습니다. 
  * JavaDoc API 레퍼런스에서 내용이 부족하거나 누락된 부분을 모두 보충하였습니다. 또한 설명이 애매하거나 잘못 이해될 수 있는 문장들을 모두 다듬었습니다.



#### Fix

* 채널 정보 관리 및 동기화 기능 리팩토링
  * 채널 정보를 효율적으로 관리하도로 개선
  * 채널 정보를 클라이언트에 보내주는 기능 개선
  * 채널의 방과 유저의 개수를 얻어오는 기능 추가
  * 채널 ID를 설정하지 않으면(즉 ""를 사용하면), 더이상 채널 기능을 사용하지 않는 것으로 개선. 채널을 사용하고자 할 경우에는 최소 1이상의 유효한 문자열을 채널 아이디로 사용해야 함.

* 방생성시 outPayload가 릴리즈되지 않는 버그 수정
* 같은 계정의 클라이언트에서 재접속 하기 전까지 룸 매치메이킹이 계속하여 실패하는 버그 수정
* 유저 매치메이킹 시에 에러가 발생할 경우 클라이언트로 응답을 보내지 않던 버그 수정
* 발행(publish) API 수정
  * 사용성 개선
  * 사용자가 의도하지 않은 노드로 publish 메시지가 전달되는 버그를 수정하였습니다. 그와 더불어 publish API의 사용성을 개선하였습니다.
  * 채널 아이디로 한글을 사용할 때 pushlish 메시지가 대상 노드를 찾지 못하는 버그를 수정하였습니다.



#### Change

* 클라우드 ACL 환경에 맞춰 기본 포트가 변경되었습니다.

| 이름 | 포트 | 이전포트 |
| --- | --- | --- |
| ipcPort | 18000 | 16000 |
| publisherPort | 18100 | 13300 |
| gateway tcp | 18200 | 11200 |
| gateway web | 18300 | 11400 |
| management | 18400 | 25150 |
| support | 18600\~18999 | 사용자 정의 |
| GameAnvil Agent | 19080 |  |
| GameAnvil Admin | 19090 |  |

* 더 이상 Admin을 지원하지 않습니다. 그로인해 GameAnvil 내부에서는 더 이상 자체 DB를 사용하지 않습니다.

* 더 이상 Dynamic Module을 지원하지 않습니다.

* ZMQ 라이브러리 교체
	
	* libzmq 와 jzmq 조합에서 Jeromq로 교체
	
* 룸 매치메이킹 리팩토링 및 사용성 개선
  * 룸매치 메이킹 흐름 개선
  * 스트레스 테스트 진행 시 방을 찾지 못하는 문제 해결하여 스트레스 테스트 진행이 가능
  * 방 인원수 정보 관리 기능 추가하여 사용자가 방 인원수 관리 기능을 구현하지 않아도 됨
  
* 인증과 로그인 그리고 접속종료 코드 최적화

    * Connection 에서 호출하던 콜백을 Session으로 이동
    * DisconnectAlarm 원인 파악을 쉽게 하도록 개선

* 사용자 제공 토픽 정리

| 이름 | 설명 | 기본 등록 위치 |
| --- | --- | --- |
| GameAnvilTopic.GAME\_NODE | 모든 게임 노드 | GameNode |
| GameAnvilTopic.GATEWAY\_NODE | 모든 게이트웨이 노드 | GatewayNode |
| GameAnvilTopic.SUPPORT\_NODE | 모든 서포트 노드 | SupportNode |
| GameAnvilTopic.ALL\_CLIENT | 접속중인 모든 클라이언트 | Session |
| GameAnvilTopic.ALL\_GAME\_USER | 게임 노드에 있는 모든 게임 유저 객체 | GameUser |


* 예외 처리 및 메서드 시그니처 최적화


  * 무분별하게 사용된 예외 처리와 메서드 시그니처를 정리하고 사용자에게 필요한 정보만 넘겨주도록 수정

* HttpRequest에 PATCH 메서드 추가
* 인증서 없이 테스트용으로 SSL를 사용할 수 있는 기능 설정명이 useSelf에서 useSelfSignedCert으로 변경되었습니다.

---

### 1.1.12 (2021.06.07)
#### Change

* SupportNode에 Gateway 접속정보를 받을 수 있는 API 추가
    * BaseSupportNode.getGatewayConnectionInfoList 함수를 사용하여, Gateway 접속정보 목록을 제공

```json
// Json으로 변환한 정보 목록
{
  "header": {
    "isSuccessful": true,
    "resultCode": 0,
    "resultMessage": "SUCCESS"
  },
  "body": [
    {
      "ip": "127.0.0.1",
      "dns": "",
      "connectionInfo": {
        "TCP_SOCKET": 11200,    // 해당 ConnectionType에 따른 포트 정보 제공
        "WEB_SOCKET": 11400
      }
    }
  ]
}
```
---

### 1.1.11 (2021.05.06)

#### Change

* HttpReqest에 PATCH 메서드를 사용할 수 있는 API 추가
    * httpRequest.PATCH()
    * httpRequest.PATCH(AsyncHttpCompletionHandler asyncHttpCompletionHandler)
    * httpRequest.PATCHAsync()

---

### 1.1.10 (2021-04-21)

#### Fix

* RedisSingle에 누락된 password 지원 API 추가

---

### 1.1.9 (2021-04-16)

#### Fix

* 클라이언트의 PauseClientStateCheck를 받았을 때 idleClientTimeout 체크도 같이 멈추도록 수정.
* 클라이언트의 PauseClientStateCheck에서 받은 pauseTime의 단위를 초로 계산하도록 수정.

---

### 1.1.8 (2021-04-15)

#### New

* 클라이언트의 PauseClientStateCheck를 받아 입력받은 시간만큼 해당 클라이언트의 상태체크를 하지 않도록 기능추가.
* 클라이언트의 ResumeClientStateCheck를 받아 상태 체크를 재개하는 기능 추가.
* GameAnvilConfig.json 의 Gateway 에 PauseClientStateCheck 관련 허용 범위 제한하는 설정값 추가
    * pauseClientStateCheckMaxDuration = 클라이언트가 보낸 시간값이 해당 수치보다 클경우 해당 수치로 제한
    * pauseClientStateCheckMinDuration = 클라이언트가 보낸 시간값이 해당 수치보다 작을경우 해당 수치로 제한

---

### 1.1.7 (2021-04-02)

#### Fix

* /game-data/get 사용시 GameData 값이 JsonObject 가 아니라 String으로 전달되는 문제 수정

---

### 1.1.6 (2021-03-30)

#### Change

* Dynamic Module 기능이 GameAnvil에서 스펙아웃되어 삭제 되었습니다.
  * 기존에 사용하시는 분들을 위해 Dynamic Module 소스 공개와 사용 가이드를 작성하여 공유 드립니다. 해당 가이드를 참고하셔서 GameAnvil이 아닌 사용자 프로젝트에 Dynamic Module 소스을 추가하여 사용할 수 있습니다.
  * [GameAnvil-Guide/81 Dynamic Module 스펙아웃으로 인한 소스 제공 및 사용법](https://nhnent.dooray.com/share/posts/xtwdO6FJTXmr7l28cZSM-w)

---

### 1.1.5 (2021-03-19)

#### Change

* GameAnvil DB & Admin 기능(GameData, Dynamic Module 제외) 삭제

    * GameAnvil 엔진단에서 더이상 DB를 사용하지 않습니다.
    * 참고: [GameAnvil-Guide/76 Admin 1.1.0 -&gt; 1.1.1 마이그레이션 문서](https://nhnent.dooray.com/share/posts/GIHoRQ8kTjSi7D4wQjUcWw)
    * 기존에 DB를 사용하던 Admin 기능들은 모두 GameAnvil Admin으로 이동되었습니다.

```json
"management": {
  "nodeCnt": 2,
  "restIp": "127.0.0.1",
  "restPort": 25150,

  // 삭제
  //"db": {
  //  "user": "root",
  //  "password": "1234",
  //  "url": "jdbc:h2:mem:gameanvil_admin;DB_CLOSE_DELAY=-1"
  //}
}
```

---

### 1.1.4 (2021-03-18)

#### Fix

* 룸매칭에 MatchingGroup 적용시 간헐적으로 생성된 방에 Join이 안되는 문제 수정

---

### 1.1.3 (2021-02-05)

#### Fix

* 인스턴스 재시작시 Disable상태에서 복구되지 않는 이슈 수정
    * 낮은 확율로 재현될 수 있으나 다시 재시작 할 경우 복구 됨

* ManagementNode에서 nodeInfoByHostId 사용 시 Double.infinity 에러가 뜨는 문제 수정

---

### 1.1.2 (2021-01-07)

#### Fix

* DynamicModule에서 SuspendExcution 예외 발생할수 있는 코드 호출시의 예외처리가 누락되어 DynamicModuledml method가 제대로 호출이 안되는 이슈 수정

---

### 1.1.1 (2021-01-05)

#### Fix

* GatewayNode 에서 getNodeId()를 호출하면 NPE 발생하는 이슈 수정

* 같은 계정의 클라이언트에서 재접속 하기 전까지 해당 GameUser로 룸 매칭이 계속하여 실패하는 이슈 수정

---

### 1.1.0 (2020-12-17)

더 자세한 정보는 [배포 노트](https://nhnent.dooray.com/share/posts/bWby9jGjQri2cFNR_KYbnw)를 참고

#### New

* Jdk11 지원

  * Jdk8 지원 버전과 더불어 Jdk11 지원 버전을 배포하였습니다.
  * GameAnvil version 명이 변경되었습니다.

```xml
// jdk8
<dependency>
    <groupId>com.nhn.gameanvil</groupId>
    <artifactId>gameanvil</artifactId>
    <version>1.1.0-jdk8</version>
</dependency>
```

```xml
// jdk11
<dependency>
    <groupId>com.nhn.gameanvil</groupId>
    <artifactId>gameanvil</artifactId>
    <version>1.1.0-jdk11</version>
</dependency>
```

* Alarm

    * 서버 내에서 발생한 상황을 지정한 URL로 받을 수 있는 기능 추가했습니다.
    * "-Dalarm.url" VM 옵션을 사용하여, Alarm을 받을 URL을 지정할 수 있습니다.
    * [GameAnvil-Guide/69 Alarm 사용법](https://nhnent.dooray.com/share/posts/Ap6DJT9KSaGv916_tj-xAA)

#### Change

* BaseObject의 findAllUserLocsOfAccount 리턴값 UserLoc 로 변경

* Node정보를 Rest API로 얻어올 수 있는 기능을 추가하였습니다.
    * /management/ipcNetworkNodeInfo
    * /management/managementNetworkNodeInfo
    * /management/locationLookupNodeInfo
    * /management/matchNodeInfo

#### Fix

* 동일한 AccountId로 다른 SubId를 사용할 경우 로그인은 성공하지만, 그 이후 응답패킷을 받을 수 없는 문제를 수정하였습니다.
* GatewayNode, MatchNode, SupportNode, ManagementNode의 NodeNum가 1부터 시작하는 문제를 수정하였습니다.
* Netty의 send,recv buffer 사이즈 최적화와 그에 알맞은 watermark가 적절하게 적용 되도록 수정

---

### 1.0.7 (2021.01.05)

#### Fix
- GatewayNode 에서 getNodeId()를 호출하면 NPE 발생하는 이슈 수정

---

### 1.0.6 (2020.12.28)

#### Fix
- `RoomMatchMaking failure. The matched-room({}) does not exist in the game node.` 로그가 남으면서 룸매칭이 실패할 경우 해당 유저는 재접속 학시 전까지 계속하여 룸매칭이 실패하는 이슈 수정

---

### 1.0.5 (2020.11.17)

#### Fix

- RoomMatchReq에서 에러 응답 시 패킷 헤더 복원(restore)이 안 되는 문제 수정

---

### 1.0.4 (2020.10.29)

#### Fix

- 임의의 Session에 아직 아무런 패킷이 전달된 적이 없을 경우에 이 Session으로 send 호출 시 NPE(Null Pointer Exception) 발생하는 문제 수정
- publishToClient 사용 시 문제 발생(Notice 포함)
- Room에서 사용하는 반복 Timer로 인해 방 파괴가 지연되는 문제 수정
- protobuf 구조체를 변수로 가지고 있는 Class를 deserialize할 때 예외 발생하는 이슈 수정
- java.lang.UnsupportedOperationException이 발생

---

### 1.0.3 (2020.10.26)

#### Change

- 이전에 스펙에서 제외되었던 공지 기능 API를 다시 활성화

---

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

---

### 1.0.1 (2020.09.07)

#### Fix

- onLogin()에서 false를 리턴할 경우 Payload가 클라이언트로 전달되지 안는 이슈 수정
- Debug와 Trace 로그문 앞의 로그 레벨 확인 조건문 누락된 부분 추가

---

### 1.0.0 (2020.08.31)

더 자세한 정보는 [배포 노트](https://nhnent.dooray.com/share/posts/5Fvh0aszQ5u6d_ZZxRWPvA)를 참고

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

---

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

---

### 0.10.1 (2020.02.11)

#### New

- 사용자에게 제공하는 도구를 모아둔 Util 클래스를 제공
- 기존에는 엔진 여러 곳에 흩어져있던 도구들을 해당 클래스 한 군데로 정리

#### Change

- Topic 처리 코드 성능 개선


---

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

---

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

---

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


---

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


---

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


---

### 0.9.5 (2019.06.21)

#### Fix

- 무점검 패치 시에 새로운 유저가 로그인하거나 유저 전송 중일 경우 Pause된 노드에서 해당 유저가 정리되지 않는 오류 수정

#### Change

- 문자열 형식의 serviceId를 정수형으로 변경
- 프로토콜의 service_code를 service_id로 변경하고 관련 코드 수정
- RESTful 지원 API의 내부 구현과 사용법 변경


---

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


---

### 0.9.3 (2019.04.10)

#### Fix

- AsyncAwait.call() API에서 timeout이 발생하면 NPE(Null Pointer Exception)도 발생하는 문제 수정
- Epoll이 활성화되면서 멈추는 문제 수정
- 로그인 시 발생하는 Invalid System Target Location 오류 수정

#### Change

- ManagementNode는 이제 더 이상 엔진 사용자가 확장해서 사용할 수 없음
- 대신 ServiceNode를 사용
- Admin에서 Machine 정보의 등록, 수정, 삭제가 요청하는 순간 바로 적용되도록 변경


---

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


---

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


---

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

---

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

---

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

---

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

---

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

---

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