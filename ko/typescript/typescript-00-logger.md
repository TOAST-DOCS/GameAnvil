## Game > GameAnvil > TypeScript 개발 가이드 > 연결 에이전트

### GameAnvilConnector

GameAnvilConnector는 서버 연결과 통신을 담당하는 클래스로, 이 객체를 통해 서버에 요청하거나 서버로부터 오는 메시지에 대한 핸들러를 등록하여 관리할 수 있습니다. 이전 설치 챕터에서 GameAnvilConnector의 생성과 connect 기능 사용에 대해서 다루었습니다. 이번 문서에서는 더 자세한 사용법과, GameAnvilConnector의 다른 기능들을 알아보겠습니다.

#### 생성

아래와 같이 GameAnvilConnector 객체를 생성합니다. 보통은 하나의 프로세스에서 하나의 커넥터 객체를 사용하는 것이 일반적입니다.

```typescript
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();
```

###