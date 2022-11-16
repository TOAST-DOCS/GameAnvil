## Game > GameAnvil > Unity 심화 개발 가이드 > Connector

## Connector

커넥터는 기본 설정과 에이전트 관리를 담당하며, 내부 동작과 관련된 로그를 볼 수 있도록 옵션을 설정하거나 콜백을 등록할 수 있습니다.

[Unity 기초 개발 가이드 > GameAnvilConnector](../unity-basic/unity-basic-02-connector.md) 문서에서 커넥터의 주요 기능을 간편하게 사용할 수 있는 GameAnvilConnector에 대해서 다루었습니다.

이번 문서에서는 커넥터를 더욱 확장성있게 사용할 수 있는 Connector에 대하여 설명합니다.
Connector를 사용하려면 먼저 Connector를 생성해야 합니다.

### 생성

다음과 같이 Connector를 생성할 수 있습니다.

```c#
using GameAnvil;
Connector connector = new Connector();
```

### 설정

Connector가 동작에 사용되는 설정이 있습니다. 이 설정들은 Connector 생성 시 기본값으로 설정되지만 필요하다면 다음과 같이 직접 값을 바꿀 수 있습니다. 

```c#
using GameAnvil;
Connector connector = new Connector();
connector.config.packetTimeout = 10;
```

`Connector.Config`를 미리 생성해놓고 이를 매개변수로 받아서 Connector 를 생성할 수도 있습니다. 또 이렇게 생성한`Connector.Config`를 MonoBehavior의 맴버로 만들어 놓을 경우 Unity의 Inspector 창에서 설정값을 바꿀 수도 있습니다. 

```c#
using GameAnvil;
var config = new Connector.Config();
config.packetTimeout = 10;
Connector connector = new Connector(config);
```

![](https://static.toastoven.net/prod_gameanvil/images/unity-advanced/01-connector/01-config.png)

설정의 종류는 다음과 같습니다.

| 이름                         | 설명                                                           | 기본값 |
| --------------------------- | ------------------------------------------------------------- | :----: |
| defaultReqestTimeout        | TimeOut 기본 대기 시간                                           | 3(sec) |
| packetTimeout               | 패킷이 지정된 시간 안에 업데이트 되지 않으면, disconnect 되었다고 판단한다. pingInterval보다 큰 값으로 설정되어야 한다. | 5(sec) |
| pingInterval                | 서버와의 연결을 확인하기 위한 ping 주기. 사용하지 않을 경우 0으로 설정.      | 3(sec) |
| useIPv6                     | IPv6 환경에서 IPv4 주소를 이용하여 서버에 접속할 때 IPv4 주소를 ipv6 주소로 변환하여 사용 여부  | false  |
| useSocketNoDelay            | 소켓의 Nodelay 사용 여부                                          | true   |
| useArgumentDelegateOnly     | 리스너에 등록한 콜백말고 매개변수로 넘긴 콜백만 호출되도록 하는 옵션           | true   |
| Request Direct              | Request 호출시 응답 대기 중이 아닐 경우 queue에 넣지 않고 바로 서버로 전송  | true   |


### 로그

Connector는 직접 로그를 남기지 않고 콜백을 통해 로그를 전달합니다. 다음과 같이 콜백을 등록해야 Connector에서 발생하는 로그를 받을 수 있습니다. 

| Connector 로그 관련 함수 | 설명 |
| --------------------- | --- |
| AddLogListener | 로깅 메세지를 처리할 콜백 등록 |
| RemoveLogListener | 로깅 메시지를 처리할 콜백 목록에서 특정 콜백 제거 |
| ContainsLogListener | 로깅 메시지를 처리할 콜백 목록이 특정 콜백을 포함하는지 검사 |
| EnablePingPongLogs | PingPong에 의해 발생하는 로그 출력 여부 설정 |
| GetEnablePingPongLogs | PingPong에 의해 발생하는 로그 출력 여부 조회 |
| SetLogLevel | 로그 레벨 설정 |
| GetLogLevel | 로그 레벨 조회 |

아래 코드를 통해서 더욱 자세하게 살펴보겠습니다.

Connector.AddLogListener()로 로깅 메세지를 처리할 콜백을 등록할 수 있습니다.

```c#
/// <summary>
/// 로깅 메시지를 처리할 콜백 등록
/// </summary>
/// <param name="logger">로깅 메시지를 처리할 콜백</param>
public void AddLogListener (Action<string> logListener);
```

Connector.RemoveLogListener()로 등록했던 로깅 메세지 처리 콜백을 삭제할 수 있습니다.

```c#
/// <summary>
/// 로깅 메시지를 처리할 콜백 목록에서 특정 콜백 제거
/// </summary>
/// <param name="logListener">제거할 콜백</param>
public void RemoveLogListener (Action<string> logListener);
```

Connector.ContainsLogListener()로 특정 콜백이 로깅 메세지를 처리할 콜백 목록에 포함되어 있는지 확인할 수 있습니다.

```c#
/// <summary>
/// 로깅 메시지를 처리할 콜백 목록이 특정 콜백을 포함하는지 검사
/// </summary>
/// <returns></returns>
public bool ContainsLogListener (Action<string> logListener);
```

Connector.EnablePingPongLogs()로 PingPong에 의해 발생하는 로그 출력 여부를 설정할 수 있습니다. PingPong은 서버와 연결이 잘 되어 있는지 확인하기 위한 패킷을 주고 받는 과정입니다.

```c#
/// <summary>
/// PingPong에 의해 발생하는 로그를 출력할지 여부를 설정한다. true이면 로그를 출력하고, false면 관련 로그는 출력하지 않는다.
/// </summary>
/// <param name="enable">PingPong로그를 출력할지 여부</param>
public void EnablePingPongLogs (bool enable);
```

Connector.GetEnablePingPongLogs()로 PingPong에 의해 발생하는 로그 출력 여부를 조회할 수 있습니다.

```c#
/// <summary>
/// PingPong에 의해 발생하는 로그를 출력할지 여부를 조회한다. true이면 로그를 출력하고, false면 관련 로그는 출력하지 않는다.
/// </summary>
/// <returns>ingPong로그를 출력할지 여부</returns>
public bool GetEnablePingPongLogs ();
```

Connector.SetLogLevel()로 로그 레벨을 설정할 수 있습니다. 로그 레벨에 따라 보여질 로그의 종류가 달라질 수 있습니다.

| 로그 레벨 | 설명 |
| :------: | --- |
| Debug | 서버와 클라이언트가 주고 받는 모든 패킷의 상세 로그 (개발용)|
| Info | 서버와 클라이언트가 주고 받는 모든 패킷의 간략 로그 |
| Warn | 커넥터의 사용 자체에는 문제가 없으나 간과할 경우 실제 서비스에서는 문제가 될 수 있는 경우, packetTimeout을 0으로 설정하고 서버에 접속한 경우, 서버에서 패킷을 받았으니 이를 처리할 리스너가 등록되지 않은 경우, 서버로부터 ClientStateCheck를 받은 경우의 로그 |
| Error | 소켓 에러, 서버로부터 받은 에러, 알 수 없는 에러 등의 로그 |

```c#
/// <summary>
/// 로그 레벨을 설정한다.
/// </summary>
/// <param name="level">설정할 로그 레벨</param>
public void SetLogLevel (LogLevel level);
```

Connector.GetLogLevel()로 설정한 로그 레벨을 조회할 수 있습니다.

```c#
/// <summary>
/// 설정한 로그 레벨을 조회한다.
/// </summary>
/// <returns>로그 레벨</returns>
public Loglevel GetLogLevel ();
```

Connector 내부 로그를 사용하는 예제입니다.

```c#
connector = new GameAnvil.Connector();

// LogListener 등록하여 커넥터 내부의 로그를 유니티에서 볼 수 있게 한다.
connector.AddLogListener(message =>{
    OnLogMesseage?.Invoke(message);
    Debug.Log(message);
});

connector.SetLogLevel(GameAnvil.Internal.LogLevel.Info);
connector.EnablePingPongLogs(true);
```


### Update

서버와 주고받는 메시지 처리를 위해 Update() 함수를 주기적으로 호출해야 합니다. MonoBehaviour의 Update() 에서 호출하도록 하는것이 제일 간편한 방법이지만, 필요에 따라 다른 방식으로 호출해도 무방합니다. 하지만 Thread safe하지 않으므로 별도의 Thread에서 호출하면 안됩니다. 
Update() 함수의 호출 주기는 자유롭게 설정해도 되지만 호출하지 않을 경우 서버로부터 메시지를 받더라도 이에 대한 알림을 받을 수 없습니다.

```c#
connector.Update();
```

### ConnectorHandler

MonoBehaviour를 상속받아 커넥터를 관리하는 ConnectorHandler 스크립트 예시입니다. 필요에 따라 새로 만들거나 적절하게 변경해 사용하면 됩니다.
ConnectorHandler를 씬의 GameObject에 컴포넌트로 추가하여 사용할 수 있습니다.

```c#
using GameAnvil.Connection;
using UnityEngine;

namespace GameAnvil
{
    public class ConnectorHandler : MonoBehaviour
    {

        private static ConnectorHandler instance;
        public static Connector connector = null;
        public ConnectionAgent connectionAgent;

        [Header("Configuration")]
        public Connector.Config config;
        public int PauseClientStateCheckMiliSecond = 10 * 60;

        private void Awake()
        {
            if (instance == null)
            {
                instance = this;
                DontDestroyOnLoad(gameObject);
                connector = new Connector(config);
                connectionAgent = connector.GetConnectionAgent();
            } 
            else
            {
                Debug.Log("ConnectorHandler instance already exist. Destroy new one.");
                Destroy(gameObject);
            }
        }

        private void Update()
        {
            connector.Update();
        }

        void OnDisable()
        {
            if (connector.IsConnected())
            {
                connectionAgent.Disconnect();
            }
        }

        private void OnApplicationPause(bool pause)
        {
            if (pause)
            {
                connector.GetConnectionAgent().PauseClientStateCheck(PauseClientStateCheckMiliSecond);
                connector.Update();
            } 
            else
            {
                connector.Update();
                connector.GetConnectionAgent().ResumeClientStateCheck();
            }

            if (connector.IsConnected())
            {
                // 앱이 resume 된 후 해야할 작업이 있다면 여기에서 처리한다.
            }
        }
    }
}
```

이제 GameAnvil 커넥터의 사용 준비가 완료되었습니다.