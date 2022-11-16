## Game > GameAnvil > Unity 기초 개발 가이드 > GameAnvilConnector

## GameAnvilConnector

GameAnvilConnector는 기본 설정과 에이전트 관리를 담당하며, 내부 동작과 관련된 로그를 볼 수 있도록 옵션을 설정하거나 콜백을 등록할 수 있습니다. GameAnvilConnector를 사용하려면 먼저 GameAnvilConnector를 생성해야 합니다.

### 생성

GameObject를 생성하고 GameAnvilConnector 컴포넌트를 추가합니다. Add Component > GameAnvil > GameAnvilConnector로 선택해서 컴포넌트로 추가할 수 있습니다.

더욱 간편한 방법으로는, Unity Hierarchy에서 우클릭 후 GameAnvil > GameAnvilConnector를 선택해서 바로 생성할 수 있습니다.

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/02-connector/01-component.png)

### 설정

GameAnvilConnector 동작에 사용되는 설정값이 있습니다. GameAnvilConnector 생성 시 기본값으로 설정되지만 필요하다면 인스펙터 창에서 직접 값을 변경할 수 있습니다.  

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/02-connector/02-config.png)

설정의 종류는 다음과 같습니다.

| 이름                         | 설명                                                           | 기본값 |
| --------------------------- | ------------------------------------------------------------- | :----: |
| defaultReqestTimeout        | TimeOut 기본 대기 시간                                           | 3(sec) |
| packetTimeout               | 패킷이 지정된 시간 안에 업데이트 되지 않으면, disconnect 되었다고 판단한다. pingInterval보다 큰 값으로 설정되어야 한다. | 5(sec) |
| pingInterval                | 서버와의 연결을 확인하기 위한 ping 주기. 사용하지 않을 경우 0으로 설정.      | 3(sec) |
| useIPv6                     | IPv6 환경에서 IPv4 주소를 이용하여 서버에 접속할 때 IPv4 주소를 Ipv6 주소로 변환하여 사용 여부  | false  |
| useSocketNoDelay            | 소켓의 Nodelay 사용 여부                                          | true   |
| useArgumentDelegateOnly     | 리스너에 등록한 콜백 말고 매개변수로 넘긴 콜백만 호출되도록 하는 옵션          | true   |
| Request Direct              | Request 호출시 응답 대기 중이 아닐 경우 queue에 넣지 않고 바로 서버로 전송  | true   |


### 로그

GameAnvilConnector는 직접 로그를 남기지 않고 콜백을 통해 로그를 전달합니다. 다음과 같이 OnLogMessage 콜백에 리스너를 등록해서 전달된 로그 메세지를 출력할 수 있습니다.

```c#
GameAnvilConnector.getInstance().OnLogMesseage.AddListener(log => Debug.Log("OnLogMesseage test : " + log));
```

GameAnvilConnector 로그에 사용되는 설정값이 있습니다.

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/02-connector/03-log.png)

설정의 종류는 다음과 같습니다.

| 이름 | 설명 | 기본값 | 
| --- | --- | :----: |
| Use Debug Log | GameAnvilConnector 내부 로그 출력 여부 | true |
| Enable Ping Pong Logs | PingPong에 의해 발생하는 로그를 출력 여부 | false |
| Log Level | GameAnvilConnector 로그 레벨 | Info |
| on Log Message (String) | GameAnvilConnector 내부 로그가 string 형태로 전달되는 콜백 | - |

이제 GameAnvilConnector의 사용 준비가 완료되었습니다.