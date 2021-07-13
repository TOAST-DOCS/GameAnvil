## Game > GameAnvil > Unity 개발 가이드 > 백그라운드 접속 끊김 방지

## 백그라운드 접속 끊김 방지

모바일 기기에서 게임이 백그라운드로 전환될 경우 유니티의 Application이 멈추게 됩니다. Application이 멈추면 Connecor.upate()를 호출하지 못하게 되고, 따라서 게임서버와 패킷을 주고받지 못하게 됩니다. 이 상태로 시간이 지나면 연결확인을 위한 ping/pong이나 clientStateCheck 패킷도 주고받지 못하게 되고, 결국 서버에서 연결을 끊어버리게 됩니다. 

경우에 따라서 특정 시간동안 게임이 백그라운드로 전환되더라도 연결이 끊어지는 것을 원치 않을 수 있습니다. 이때 ConnectionAgent의 PauseClientStateCheck(), ResumeClientStateCheck()를 사용하여 서버의 연결확인 기능을 일시정지 시킬 수 있습니다. 

```c#
public class ConnectHandler : MonoBehaviour
{
    ...
    private void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            // 앱이 pause 되기전에 해야할 작업이 있다면 여기에서 처리한다.

            // 입력한 시간(sec)동안 서버의 clientStateCheck 기능을 정지시킨다
            // 이시간이 지나면 clientStateCheck 기능이 동작하여 연결이 끊어질 수 있다. 
            connector.GetConnectionAgent().PauseClientStateCheck(600);

            // 앱이 pause 되기 직전 connector.Update() 를 호출하여 
            // Connector에 쌓은 메시지를 처리하고 상태를 업데이트 해준다. 
            connector.Update();
        } else
        {
            // 앱이 resume 된 직후 connector.Update() 를 호출하여 
            // Connector에 쌓은 메시지를 처리하고 상태를 업데이트 해준다. 
            connector.Update();

            // 서버의 clientStateCheck 기능을 다시 동작시킨다.
            connector.GetConnectionAgent().ResumeClientStateCheck();

            // 장시간 puase되었다가 resume될 경우 연결이 끊어질 수 있으므로 상태를 체크한다.
            if (connector.IsConnected())
            {
                // 앱이 resume 된 후 해야할 작업이 있다면 여기에서 처리한다.
            }
        }
    }
    ...
}
```

PauseClientStateCheck()에 입력한 시간이 지나면 서버의 연결확인 기능이 다시 동작하게 되고, 접속이 끊어질 수 있습니다. 이럴 경우에는 재접속을 진행해야합니다.
