## Game > GameAnvil > Unity 기초 개발 가이드 > 백그라운드 접속 끊김 방지

## 백그라운드 접속 끊김 방지

모바일 기기에서 게임이 백그라운드로 전환될 경우 Unity의 애플리케이션이 멈추게 됩니다. 애플리케이션이 멈추면 게임 서버와 패킷을 주고받지 못하게 되며 이 상태가 지속되면 연결 확인을 위한 패킷도 주고받지 못하게 되므로 결국 서버와의 연결이 끊길 수 있습니다.

GameAnvilManager 에는 게임이 백그라운드로 전환되더라도 연결이 끊어지는 것을 막기 위해서 앱이 백그라운드로 전환 될 때 서버의 연결 확인 기능을 일정 시간 동안 정지 하도록 되어있습니다. 이 시간은 pauseClientStateCheckTime 값을 이용해 조절할 수 있으며 기본값은 600(초) 입니다. 
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/07-pause/01-pause-client-check.png)

### 재접속

pauseClientStateCheckTime 설정한 시간이 지나면 서버의 연결 확인 기능이 다시 동작하게 되고, 접속이 끊어질 수 있습니다. 이럴 경우에는 다시 간편 로그인을 진행해야 합니다.
