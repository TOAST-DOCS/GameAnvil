## Game > GameAnvil > 콘솔 사용 가이드 > 시작하기

## 시작하기에 앞서

게임 서버 운영은 개발이 완료된 서버 바이너리를 원하는 규모의 물리 장비로 배포하고, 원하는 논리 구성으로 구동하는 것에서 시작합니다. 또한 모니터링과 바이너리 관리, 그리고 패치 지원 등이 필요합니다. GameAnvil 콘솔은 이러한 요구 사항을 모두 충족합니다. 이 문서에서는 GameAnvil을 이용하여 구현한 게임 서버를 콘솔상에서 배포하고 운영하는 방법을 설명합니다.


## 용어 정리

이 문서에서는 다음의 용어들을 자주 사용합니다. 대부분의 용어는 일반적인 의미와 다르지 않으나, '게임 서버'와 같은 일부 용어는 GameAnvil에서 특수하게 정의한 의미로 사용하므로 용어를 모두 숙지한 뒤 문서를 이용할 것을 권장합니다.

| 용어      | 의미                                                                                                                                                                                      |
|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 서버 바이너리 | 서버 인스턴스상에서 서버 프로세스로 실행 가능한 .jar 파일. 해당 바이너리는 사용자가 구현한 모든 노드의 코드를 포함하고 있습니다.                                                                                                             |
| 서버 프로세스 | 게임 서버를 구동 중인 프로세스. 즉, 서버 바이너리를 실행한 상태.                                                                                                                                                  |
| 서버 인스턴스 | 게임 서버를 구동하기 위한 NHN Cloud 인스턴스(VM)                                                                                                                                                      |
| 게임 서버   | 서버 바이너리가 배포된 서버 인스턴스 또는 이미 프로세스로 구동된 상태의 서버 인스턴스 전반을 아우르는 추상적인 용어. 즉, 게임 서비스가 가능한 물리 장비+바이너리 조합. GameAnvil은 서버 인스턴스와 서버 프로세스를 하나의 개념으로 관리합니다. 문서나 콘솔의 메뉴 등에서는 이를 줄여 '서버'라고 하기도 합니다. |
| 서버      | 게임 서버와 동일한 의미                                                                                                                                                                           |
| 노드      | 하나의 서버 프로세스를 구성하는 기능 단위. 하나의 서버 프로세스는 1개 이상의 노드로 구성 가능 (자세한 사항은 서버 개발 가이드 참고)                                                                                                           |
| 콘솔      | NHN Cloud에서 GameAnvil 서비스를 활성화하면 접근 가능한 GameAnvil Console을 줄여서 콘솔이라고 칭합니다.                                                                                                              |
| 배포      | 서버 바이너리를 서버 인스턴스로 업로드하는 과정                                                                                                                                                              |
| 배포 파일   | 서버 인스턴스로 업로드된/업로드할 서버 바이너리                                                                                                                                                              |
| 구성      | 서버를 원하는 형태로 구성(Config)하는 것을 의미. 서버 설정도 동일한 의미를 가집니다.                                                                                                                                    |


## GameAnvil 서비스 활성화

GameAnvil을 사용하려면 NHN Cloud 콘솔에서 GameAnvil 서비스를 활성화해야 합니다. **서비스 선택** 또는 **서비스 추가**를 클릭합니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/v2/getting-started/activation-1.png)

**Game** 카테고리에서 **GameAnvil**을 선택합니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/v2/getting-started/activation-2.png)

서비스 활성화를 최초로 진행할 경우 다음과 같은 결제 수단 등록 팝업 화면이 나타납니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/v2/getting-started/activation-3-1.png)

화면의 안내에 따라 결제 수단을 등록하면 다음과 같이 조직 생성 또는 선택 화면이 표시됩니다. 조직을 설정한 뒤 확인을 클릭해 서비스 활성화를 완료합니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/v2/getting-started/org-and-project.png)

서비스가 활성화되고 조직이 생성되면 사용자는 고유한 콘솔 접근 권한을 갖게 됩니다. 사용자는 콘솔에 접속하여 운영 단계에서 필요한 모든 기능을 설정하고 관리할 수 있습니다.



## GameAnvil 라이선스 동의

먼저 라이선스 스크롤을 맨 아래로 내린 후, 약관에 동의하기에 체크할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/v2/getting-started/license_agree.png)



## GameAnvil 상품 선택

서비스를 활성화하면 GameAnvil 상품 선택 화면이 표시됩니다. GameAnvil 콘솔을 사용하기 전 상품을 선택해야 합니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/v2/getting-started/choose_product.png)

GameAnvil은 게임 서버의 시스템 노드 규모와 기술 지원 범위가 다른 두 종류의 상품을 제공합니다. 시스템 노드는 GameAnvil 내부적으로 인스턴스와 노드, 그리고 유저 정보 등을 관리하는 데 사용되는 자원이며, 사용자에게는 노출되지 않습니다.
Standard 상품은 중·소규모 게임에 적절한 상품으로 이중화된 시스템 노드와 풍부한 기술 지원을 제공합니다. Premium 상품은 대규모 게임을 위해 최적화된 시스템 노드와 좀 더 넓은 범위의 기술 지원을 제공합니다. 서비스할 게임의 특성과 규모에 맞추어 상품을 선택하십시오.

## GameAnvil 서비스 대시보드 확인

위 과정을 완료하면 GameAnvil 서비스를 사용할 모든 준비가 완료되었습니다. 콘솔에서 GameAnvil 대시보드를 확인할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/v2/getting-started/dashboard_init.png)