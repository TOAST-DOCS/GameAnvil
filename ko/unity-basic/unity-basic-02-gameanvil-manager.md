## Game > GameAnvil > Unity 기초 개발 가이드 > 매니저

## GameAnvilManager

GameAnvilManager는 기본 설정과 에이전트 관리를 담당하며, 내부 동작과 관련된 로그를 볼 수 있도록 옵션을 설정하거나 콜백을 등록할 수 있습니다. GameAnvilManager를 사용하려면 먼저 씬에 GameAnvilManager를 추가해야 합니다.

### 생성

Unity Hierarchy 창에서 마우스 오른쪽 버튼을 클릭한 뒤 **GameAnvil > GameAnvilManager**를 선택해 바로 생성할 수 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/02-gameanvil-manager/01-add-gameanvil-manager.png)

또는 빈 GameObject를 생성하고 GameAnvilManager 컴포넌트를 추가할 수도 있습니다.

### 설정

GameAnvilManager 에는 여러가지 설정값이 있습니다. GameAnvilManager 생성 시 기본값으로 설정되지만 필요하다면 Inspector 창에서 직접 값을 변경할 수 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/02-gameanvil-manager/02-gameanvil-manager-inspector.png)

설정의 종류는 다음과 같습니다.

| 구분             | 이름                            | 설명                                                                                                                      |    기본값    |
|----------------|-------------------------------|-------------------------------------------------------------------------------------------------------------------------|:---------:|
| Connect        | Ip                            | 간편 로그인 에서 연결할 서버의 ip                                                                                                    | 127.0.0.1 |
|                | Port                          | 간편 로그인 에서 연결할 서버의 port                                                                                                  |   18200   |
| Authentication | AccountId                     | 간편 로그인 에서 사용할 사용자 식별 ID                                                                                                 |   test    |
|                | DeviceId                      | 간편 로그인 에서 사용할 기기 고유값                                                                                                    |   test    |
|                | Password                      | 간편 로그인 에서 사용할 패스워드                                                                                                      |   test    |
| Login          | User Type                     | 간편 로그인 에서 사용할 유저 타입                                                                                                     |           |
|                | Channel Id                    | 간편 로그인 에서 사용할 채널 아이디                                                                                                    |           |
|                | Service Name                  | 간편 로그인 에서 사용할 서비스 이름                                                                                                    |           |
| Client Check   | Pause Client State Check Time | 클라이언트 상태 체크 확인 중지 시간 설정. <br/>백그라운드로 전환시 클라이언트의 연결상태 체크를 일시 중지한다. 이 시간동안은 앱이 백그라운드에 머물다가 포그라운드로 올라와도 서버에서 강제 종료시키지 않는다. |    600    |
| Log            | Threshold                     | 로그 레벨을 설정합니다.                                                                                                           |   Info    |

이제 GameAnvilManager의 사용 준비가 완료되었습니다.