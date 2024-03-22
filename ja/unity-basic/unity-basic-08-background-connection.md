## Game > GameAnvil > Unity 기초 개발 가이드 > 백그라운드 접속 끊김 방지

## 백그라운드 접속 끊김 방지

모바일 기기에서 게임이 백그라운드로 전환될 경우 Unity의 애플리케이션이 멈추게 됩니다. 애플리케이션이 멈추면 GameAnvilConnector의 Update()를 호출하는 과정이 이루어지지 않아 게임 서버와 패킷을 주고받지 못하게 됩니다. 이 상태가 지속되면 연결 확인을 위한 패킷도 주고받지 못하게 되므로 결국 서버와의 연결이 끊길 수 있습니다.

특정 시간 동안 게임이 백그라운드로 전환되더라도 연결이 끊어지는 것을 원치 않을 경우 ConnectionAgent의 PauseClientStateCheck(), ResumeClientStateCheck()를 사용하여 서버의 연결 확인 기능을 일시 정지시킬 수 있습니다.

서버의 연결 확인 일시 정지 시간은 GameAnvilConnector의 pauseClientStateCheckTime 값을 이용해 조절할 수 있습니다.

더 자세한 내용은 [Unity 심화 개발 가이드 > 백그라운드 접속 끊김 방지](../unity-advanced/unity-advanced-06-background-connection.md)를 참고하십시오.

### 재접속

PauseClientStateCheck()에 입력한 시간이 지나면 서버의 연결 확인 기능이 다시 동작하게 되고, 접속이 끊어질 수 있습니다. 이럴 경우에는 재접속을 진행해야 합니다.

재접속과 관련된 자세한 내용은 [Unity 심화 개발 가이드 > 재접속](../unity-advanced/unity-advanced-07-reconnect.md)에서 확인할 수 있습니다.