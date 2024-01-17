## Game > GameAnvil > Unity 기초 개발 가이드 > GameAnvil 커넥터 종료

## GameAnvil 커넥터 종료

게임 플레이 종료 전 GameAnvilConnector.Disconnect() 함수를 호출해 연결을 종료하는 것이 좋습니다. 종료하지 않으면 서버에서 클라이언트의 종료를 인지하지 못할 수 있으며, 그럴 경우 불필요한 동작을 계속할 수 있습니다. 
GameAnvilConnector의 OnDestroy()에 해당 기능이 내장되어 있습니다.

```c#
private void OnDestroy()
{
    if (connector.IsConnected())
    {
        Disconnect();
    }
}
```
