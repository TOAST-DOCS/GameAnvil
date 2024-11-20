## Game > GameAnvil > 테스트 개발 가이드 > 시작하기

## GameHammer

GameHammer는 GameAnvil 엔진을 이용한 게임 서버 개발 도구로 강력하고 편리한 테스트 도구입니다. 실제 커넥터에서 제공하는 모든 기능을 사용할 수 있으며, 다양한 테스트 케이스를 만들 수 있는 API를 제공하고 있습니다. 또한, 스트레스 테스트를 위해 다수의 GameHammer를 동시에 실행하고, 그 결과를 취합해 바로 확인할 수 있습니다.

### 시스템 요구 사항

GameHammer를 사용하려면 다음과 같은 사항이 필요합니다. 

- 지원하는 언어
    - Java
- 타깃 개발 환경
    - IntelliJ
- 지원하는 네트워크 프로토콜
    - TCP/IP
    - SSL over TCP/IP
- 사용 가능한 응용 프로토콜 형식
    - Google Protocol Buffers
    - Google FlatBuffers(예정)
    - 커스텀 바이트 스트림
    - HTTP/HTTPS(특정한 용도로 한정)

### 특장점

GameHammer는 다음과 같은 기능을 지원합니다.

- 커넥터와 대응하는 모든 기능 지원
- Sync/Async 방식 모두 지원
    - Async 방식의 API 제공
    - Sync 방식을 위한 future 제공
- 수천 개 또는 그 이상의 커넥션 동시에 사용 가능
- 상태 기반의 시나리오 관리 기능 지원