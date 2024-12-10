## Game > GameAnvil > Unity 기초 개발 가이드 > 접속 종료

## 접속 종료

게임 플레이 종료 전 GameAnvilConnector.Disconnect() 함수를 호출해 연결을 종료할 것을 권장합니다. 종료하지 않으면 서버에서 클라이언트의 종료를 인지하지 못할 수 있으며, 이 경우 불필요한 동작을 계속할 수 있습니다.

```c#
public class GameAnvilManager : MonoBehaviour
{
    ...
    
    private void OnDestroy()
    {
        if (connector.IsConnected())
        {
            Disconnect();
        }
    }
    
    ...
}
```
