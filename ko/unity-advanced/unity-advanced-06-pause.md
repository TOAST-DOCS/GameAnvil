## Game > GameAnvil > Unity 심화 개발 가이드 > 백그라운드 접속 끊김 방지

## 백그라운드 접속 끊김 방지

서버와 클라이언트의 연결 상태를 확인하기 위해 서버에서는 주기적으로 클라이언트의 상태를 체크하는 메시지를 보내고, 클라이언트에서는 이에 응답하는 메시지를 보냅니다. 그런데 모바일 기기에서 게임이 백그라운드로 전환될 경우 Unity 애플리케이션이 멈추게 되며, 애플리케이션이 멈추면 게임 서버와 패킷을 주고받지 못하게 됩니다. 이 상태가 에서는 연결 상태를 확인을 위한 메시지도 주고받지 못하게 되므로 결국 서버와의 연결이 끊기게 됩니다.

### 연결 확인 기능 일시정지 및 재개

연결 상태를 확인하지 못해 서버와의 연결이 끊어지는것을 방지하기 위해서는 백그라운드로 전환되기 전에 서버로 연결 확인을 위한 기능의 일시 정지를 요청해야 합니다.
애플리케이션이 백그라운드나 포그라운드로 전환될 때 Unity의 MonoBehaviour 에있는 OnApplicationPause() 콜백이 호출됩니다. 백그라운드로 전환 될 때 PauseClientStateCheck()를 호출하여 연결 확인 기능을 일시 정지하고, 포그라운드로 전환 될 때는 ResumeClientStateCheck()를 호출하여 연결 확인 기능을 재개 합니다. 

```c#
public class GameAnvilManager : MonoBehaviour
{
    ...
    
    private void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            connector.PauseClientStateCheck(pauseClientStateCheckTime);
        }
        else
        {
            connector.ResumeClientStateCheck();
        }
    }
    
    ...
}
```

PauseClientStateCheck()은 다음과 같은 1개의 매개변수를 가지고 있습니다.

| 타입      | 이름             | 설명                                           |
|---------|----------------|----------------------------------------------|
| int     | pauseTime       | 일시 정지할 시간. <br/>최소값 : 10초, 최대값 : 15분, 단위 밀리초 |

최소값 보다 작은 값을 입력하더라도 자동으로 최소값으로 적용되며, 최대값 보다 큰 값이 입력하더라도 최대값에 해당하는 시간이 지나면 자동으로 연결 확인 기능이 재개됩니다.

