## Game > GameAnvil > 릴리스 노트



### 1.0.0 (2020.08.31)

#### New

* 성능과 안정성 대폭 향상 (0.9버전 대비 약 7배)
  * 성능에 직접적인 영향을 주는 다수의 로직을 최적화하고 문제점 수정
* 완전히 새로워진 GameNode 로드밸런싱
  * 클라우드 VM의 상태를 반영하여 빠르고 정확하게 최적의 GameNode로 부하를 분산
* Integer형 ID 도입
  * 기존에 String으로 되어 있던 핵심 ID들을 모두 Integer로 교체하여 최적화 진행
* 서버 Log 레벨을 런타임에 변경할 수 있는 API 추가
  * Node 단위로 서비스 중에 Log 레벨을 변경 가능

#### Fix

* 유저 전송 문제 수정, 사용법 개선 그리고 최적화
  * GameAnvil 사용자는 더욱 쉽게 유저 전송을 사용 가능
* 무점검 패치 완벽 수정
  * 그 동안 방 전송과 무점검 패치가 제대로 동작하지 않던 문제 모두 수정
* 서버 구동 시에 알 수 없는 타이밍 이슈로 일부 Node가 제대로 뜨지 않던 문제 수정
* Node의 일부 Timer 객체가 해제되지 않고 지속적으로 누적되던 Memory Leak 수정
* 클라이언트 접속 체크 과정에서 새롭게 접속하는 유저가 의도하지 않게 끊기던 문제 수정

#### Change

* Node 이름을 아래의 표와 같이 더욱 직관적으로 변경

| v1.0       | ~ v0.10       | 필수 | 기능                                        | 컨텐츠 구현 | 네트워크 접근     |
| ---------- | ------------- | ---- | ------------------------------------------- | ----------- | ----------------- |
| Gateway    | Session       | 필수 | 클라이언트 접속과 인증을 처리               | 가능        | public            |
| Game       | Space         | 필수 | 실제 게임 서버로서 컨텐츠를 처리            | 가능        | private           |
| Support    | Service       | 선택 | 필요에 따라 독립된 서비스로 구현하도록 지원 | 가능        | private or public |
| Match      | Match         | 선택 | 매치메이킹을 수행                           | 가능        | private           |
| Location   | Location      | 필수 | 유저와 방 등의 위치 정보를 저장 및 관리     | 불가능      | private           |
| Management | Management    | 필수 | 서버 정보 취합 및 Admin/Agent와 통신        | 불가능      | private           |
| Ipc        | Communication | 필수 | Gameflex 서버의 Inter-process 통신 처리     | 불가능      | private           |

* GameAnvil 코드 가독성과 사용성 향상
  * 패키지 구조를 효율적으로 개선
  * 클래스 상속/포함 구조 개선
  * 네이밍을 직관적이고 일관성 있게 수정
* Node의 비활성화 상태는 Invalid와 Disable로 세분화
  * Invalid는 일시적으로 반응을 하지 않는 상태
  * Disable은 영구적으로 비활성화된 상태
* 이제 Loopback 주소는 자동으로 바인딩
* GameAnvil에서 사용하는 모든 라이브러리를 최신 버전으로 업데이트
* 전체 ErrorCode를 하나로 통합
* 불필요한 ReConnect와 MoveService 기능 제거
* 등록 시점이 아닌 구동 시점부터 다음 시간을 측정하는 새로운 Timer 추가

<br>

### 0.10.2 (2020.04.08)

#### New

* GameAnvil API 레퍼런스 (JavaDoc) 사이트 오픈

* AOT Instrumentation 지원

  * 엔진을 런타임에 매번 JIT Instrumentation할 필요없이 컴파일 타임에 AOT로 진행

* 공식 GC로 G1GC를 사용하도록 가이드 시작

  * 가장 기본적인 VM 옵션

    ```java
    -javaagent:QUASAR_PATH\quasar-core-0.7.10-jdk8.jar=bm
    -Xms6g
    -Xmx6g
    -XX:+UseG1GC
    -XX:MaxGCPauseMillis=100
    -XX:+UseStringDeduplication
    ```

    

#### Fix

* Quasar 스택 버그를 수정
* 룸매칭 유저들이 서로 다른 방으로 매칭되는 버그 수정
* 압축된 패킷이 전송안되는 문제 수정

<br>

### 0.10.1 (2020.02.11)

#### New

* 사용자에게 제공하는 도구를 모아둔 Util 클래스를 제공
  * 기존에는 엔진 여러 곳에 흩어져있던 도구들을 해당 클래스 한 군데로 정리

#### Change

* Topic 처리 코드 성능 개선

<br>

### 0.10.0 (2020.02.06)

#### New

* 기존에 문제가 많던 비동기 지원 API를 새롭게 리팩토링
  * Redis Cluster 지원 및 Lettuce 공식 지원에 따른 새로운 비동기 API 추가
  * 비동기 HttpRequest와 HttpResponse API 추가
* ByteString 기반의 커스텀 프로토콜 기능 지원
  * Google protobuf 외의 사용자가 원하는 방식으로 메시지를 직렬화 가능

#### Fix

* 이전 버전에서 가장 심각한 문제였던 Fiber의 상태가 망가지는 문제 모두 수정

#### Change

* 엔진에서 사용하는 패킷과 헤더 크기 최적화

* 0.9버전 대비 성능이 약 2.5배 향상

  







