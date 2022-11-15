## Game > GameAnvil > CocosCreator 개발 가이드 > Connector

## Connector

GameAnvil 커넥터를 사용하려면 먼저 Connector를 생성해야 합니다. 기본 설정과 에이전트 관리를 담당하며, 내부 동작과 관련된 로그를 볼 수 있도록 콜백을 등록할 수 있습니다.

### 생성

다음과 같이 Connecor를 생성할 수 있습니다.

``` typescript
import { Connector } from 'GameAnvil/gameanvil';
let connector = Connector.Create();
```

### 설정

Connector가 동작에 사용되는 설정이 있습니다. 이 설정들은 Connector 생성시 기본값으로 설정되지만 필요하다면 다음과 같이 직접 값을 바꿀 수 있습니다. 

``` typescript
import { Connector } from 'GameAnvil/gameanvil';
let connector = Connector.Create();
connector.config.PacketTimeout = 10;
```

설정의 종류는 다음과 같습니다.

| 이름                 | 설명                                                         | 기본값 |
| -------------------- | ------------------------------------------------------------ | ------ |
| defaultReqestTimeout | TimeOut 기본 대기시간 설정                                   | 3(sec) |
| packetTimeout        | 패킷이 지정된 시간안에 업데이트 되지 않으면, disconnect 되었다고 판단된다. pingInterval 보다 높게 설정되야 한다. | 5(sec) |
| pingInterval         | 서버와의 연결을 확인하기 위한 ping 주기 설정. 0일경우 사용안함 | 3(sec) |

### Update

서버와 주고받는 메시지 처리를 위해 Update() 함수를 주기적으로 호출해야 합니다. Update() 함수의 호출 주기는 자유롭게 설정해도 되지만 호출하지 않을 경우 서버로부터 메시지를 받더라도 이에 대한 알림을 받을 수 없습니다.

```typescript
setInterval(() => { connector.Update(); }, 10);
```

### ConnectHandler

다음과 같은 싱글 턴 클래스를 만들어 사용하면 편리합니다. 

```typescript
/*
    Internet Explorer 등 구버전 브라우저에서 커넥터 사용하기 위해서는 
    core-js, regenerator-runtime 등의 플러그인을 사용해야 한다.
*/
import 'core-js';
import 'regenerator-runtime';

import { Connector, ProtocolManager } from 'GameAnvil/gameanvil';

export default class GameAnvilManager {
    private static manager: GameAnvilManager;
    private connector: Connector;
    private constructor() {
        // 프로토콜 등록.
        ProtocolManager.RegisterProtocol(0, GameMessages);

        // 커넥터 생성.
        this.connector = Connector.Create();

        // 메세지 루프. 10ms 마다 호출.
        let updater = setInterval(() => { this.connector.Update(); }, 10);
    }

    static GetInstance() {
        if (this.manager == null) {
            this.manager = new GameAnvilManager();
        }
        return this.manager;
    }

    GetConnectionAgent() {
        return this.connector.GetConnectionAgent();
    }

    GetUserAgent(serviceName: string, subId: string) {
        return this.connector.GetUserAgent(serviceName, subId);
    }

    CreateUserAgent(serviceName: string, subId: string) {
        return this.connector.CreateUserAgent(serviceName, subId);
    }
}
```

이 코드는 예시이며 필요에 따라 적절하게 변경해 사용하면 됩니다. 이제 GameAnvil 커넥터의 사용 준비가 완료되었습니다.