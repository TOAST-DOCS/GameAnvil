## Game > GameAnvil > Unity 심화 개발 가이드 > 백그라운드 접속 끊김 방지

[Unity 기초 개발 가이드 > 백그라운드 접속 끊김 방지](../unity-basic/unity-basic-08-background-connection.md) 문서의 내용처럼 모바일 기기에서 게임이 백그라운드로 전환될 경우 서버 접속이 끊길 수 있는데, 이를 방지하기 위해 서버와의 연결 확인 기능을 일시정지시킬 수 있습니다.

이를 구현한 코드 예시는 아래와 같습니다.

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

            // 장시간 pause되었다가 resume될 경우 연결이 끊어질 수 있으므로 상태를 체크한다.
            if (connector.IsConnected())
            {
                // 앱이 resume 된 후 해야할 작업이 있다면 여기에서 처리한다.
            }
        }
    }
    ...
}
```