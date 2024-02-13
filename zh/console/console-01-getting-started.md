## Game > GameAnvil > 콘솔 사용 가이드 > 시작하기

## 1. 시작하기에 앞서

게임 서버 운영의 시작은 개발이 완료된 서버 바이너리를 원하는 규모의 물리 장비로 배포하고 원하는 논리 구성으로 구동하는 것입니다. 그와 더불어 모니터링과 바이너리 관리 그리고 패치 지원 등이 필요합니다. GameAnvil 콘솔은 이러한 요구 사항을 모두 충족합니다. 이 문서는 여러분이 GameAnvil을 이용하여 구현한 게임 서버를 콘솔 상에서 배포하고 운영하는 방법을 설명합니다.


## 2. 용어 정리

이 문서에서는 다음의 용어들을 자주 사용합니다. 대부분의 용어는 그 의미가 일반적인 뜻과 크게 차이가 없으나 "게임 서버"와 같은 일부 용어는 GameAnvil에서 특수하게 의미를 정의해 두었으므로 가능하면 이런 용어들을 모두 숙지한 후에 나머지 문서를 읽으시길 추천 드립니다.

| 용어      | 의미                                                                                                                                                                                      |
|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 서버 바이너리 | 서버 인스턴스 상에서 서버 프로세스로 실행 가능한 jar 파일. 해당 바이너리는 사용자가 구현한 모든 노드의 코드를 포함하고 있습니다.                                                                                                             |
| 서버 프로세스 | 게임 서버를 구동 중인 프로세스. 즉, 서버 바이너리를 실행한 상태.                                                                                                                                                  |
| 서버 인스턴스 | 게임 서버를 구동 하기 위한 NHN Cloud 인스턴스(VM)                                                                                                                                                      |
| 게임 서버   | 서버 바이너리가 배포된 서버 인스턴스 혹은 이미 프로세스로 구동된 상태의 서버 인스턴스 전반을 아우르는 추상적인 용어. 즉, 게임 서비스가 가능한 물리 장비 + 바이너리 조합. GameAnvil은 서버 인스턴스와 서버 프로세스를 하나의 개념으로 관리합니다. 문서나 콘솔의 메뉴 등에서는 이를 줄여서 그냥 서버라고 하기도 합니다. |
| 서버      | 게임 서버와 동일한 의미                                                                                                                                                                           |
| 노드      | 하나의 서버 프로세스를 구성하는 기능 단위. 하나의 서버 프로세스는 1개 이상의 노드로 구성 가능 (자세한 사항은 서버 개발 가이드 참고)                                                                                                           |
| 콘솔      | NHN Cloud에서 GameAnvil 서비스를 활성화하면 접근 가능한 GameAnvil Console을 줄여서 콘솔이라고 칭합니다.                                                                                                              |
| 배포      | 서버 바이너리를 서버 인스턴스로 업로드하는 과정                                                                                                                                                              |
| 배포 파일   | 서버 인스턴스로 업로드된/업로드할 서버 바이너리                                                                                                                                                              |
| 구성      | 서버를 원하는 형태로 구성(Config)하는 것을 의미. 서버 설정도 동일한 의미를 가집니다.                                                                                                                                    |


## 3. GameAnvil 서비스 활성화

GameAnvil을 사용하기 위해서는 NHN Cloud에서 GameAnvil 서비스를 활성화해야 합니다. 아래의 이미지에서 빨간색 박스로 표시한 "서비스 추가"를 통해 진행할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/getting-started/activation-1.png)

해당 버튼을 클릭하면 아래와 같은 서비스 선택 화면이 나타납니다. 이 중에서 빨간색 박스로 표시된 GameAnvil을 선택합니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/getting-started/activation-2.png)

최초로 상품 활성화를 진행 중이라면 아래와 같은 결제 수단 등록 팝업창이 나타납니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/getting-started/activation-3-1.png)

팝업창의 내용을 따라 결제 수단 등록을 완료하면 다음과 같이 조직 생성 및 선택 화면을 통해 활성화가 계속 진행됩니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/getting-started/org-and-project.png)

원하는 이름으로 조직을 생성하고 나면 사용자는 자신만의 고유한 콘솔 접근 권한을 가지게 됩니다. 바로 이 콘솔이 운영 단계에서 필요한 모든 기능을 제공합니다. 이 문서의 나머지 부분은 이 콘솔의 기능과 사용법을 중심으로 내용을 풀어 나가도록 하겠습니다.

## 4. GameAnvil 상품 선택

자, 이제 GameAnvil의 콘솔 사용에 앞서 상품 선택을 진행해야 합니다. 서비스를 활성화하면 아래와 같이 상품 선택 페이지가 시작됩니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/getting-started/activated-1-1.png)

GameAnvil은 현재 두 가지의 상품을 제공합니다. 각 상품은 게임 서버의 시스템 노드 규모와 기술 지원 범위가 다릅니다. Standard 상품은 중소규모 게임에 있어서 가장 적당한 상품으로 이중화된 시스템 노드와 풍부한 기술 지원이 제공됩니다. Premium 상품은 대규모 게임을 위해 최적화된 시스템 노드와 좀 더 넓은 범위의 기술 지원을 제공하는 상품입니다. 서비스할 게임의 특성과 규모에 맞춰 상품을 선택하세요.

자, 여기까지 진행했다면 콘솔을 사용할 모든 준비가 완료되었습니다. 참고로 시스템 노드는 사용자에게 노출되지 않으며 GameAnvil 내부적으로 인스턴스와 노드 그리고 유저 정보 등을 관리하는데 사용되는 자원입니다.

## 5. Welcome to GameAnvil Console

여기까지 모든 과정을 완료하면 콘솔 대시보드를 볼 수 있습니다. 축하합니다. 이제 GameAnvil 서비스를 사용할 모든 준비가 완료되었습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/getting-started/console-dashboard.png)