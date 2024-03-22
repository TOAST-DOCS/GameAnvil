## Game > GameAnvil > 콘솔 사용 가이드 > 서버 생성하기

## GameAnvil 메뉴와 탭

GameAnvil 콘솔은 상위의 메뉴로 큰 기능이 나뉘며 각 메뉴는 하위의 탭들로 구성됩니다. 예를 들어 **모니터링**, **서버**, **운영**은 메뉴입니다. 그리고 모니터링 메뉴는 **대시보드**, **서버 현황**, **유저 분포**, **오토스케일 그룹** 탭으로 구성됩니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/menu-server-manage-1.png)

**서버** 메뉴는 **서버**, **오토스케일 그룹**, **노드**, **배포 파일**, **Config** 탭으로 구성됩니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/menu-server-manage-2.png)

**운영** 메뉴는 **실행**, **이력** 탭으로 구성됩니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/menu-server-manage-3.png)

이 문서는 이러한 메뉴와 탭의 기능을 중심으로 설명합니다.


## 서버 생성을 위한 준비

서버를 생성하려면 최소한 서버 바이너리와 관련 설정 및 데이터 등이 필요합니다. 콘솔에서는 이를 위해 서버 메뉴의 배포 파일과 Config 탭을 제공합니다. 

배포 파일 탭에 게임 서버 바이너리를 업로드할 수 있습니다. 업로드한 바이너리는 아래 화면과 같이 내역이 관리됩니다. 서버를 생성할 때 이와 같은 배포 파일 중 하나를 선택해 각 인스턴스로 자동 배포하게 됩니다.  
![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/deploy.png)

배포 파일과 마찬가지로 서버를 구동하기 위한 구성 정보(Config)를 등록해야 합니다. 이를 통해 서버 바이너리를 어떤 설정에 기반하여 구동할 것인지 결정할 수 있습니다. 이러한 구성 파일 또한 다음 화면과 같이 그 내역이 별도로 관리됩니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/config.png)

구성 정보(Config)를 등록하는 방식은 크게 두 가지입니다. 첫 번째는 아래 화면과 같이 미리 생성된 JSON 파일을 업로드하는 방식입니다. 이때 최초로 구성 정보(Config) 파일을 작성하는 경우 Config 템플릿 다운로드를 클릭해 템플릿을 내려받아 사용할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/config-new.png)

두 번째는 아래 화면과 같이 편집창을 이용해 직접 구성 정보를 입력하는 방식입니다. 등록 방식 외에는 차이가 없으므로 원하는 방법을 선택해 구성 정보를 등록합니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/config-edit.png)

등록한 배포 파일과 구성 파일은 목록뿐만 아니라 이력 탭에서 그동안의 모든 이력을 조회할 수도 있습니다.


## 서버 생성하기

앞선 과정에서 배포 파일과 Config 파일이 정상적으로 등록되었다면 이제 서버를 생성할 수 있습니다. 서버는 실제 서비스뿐만 아니라 테스트나 개발 등 원하는 용도로 사용할 수 있습니다.

서버를 생성하려면 콘솔의 서버 메뉴에서 서버 생성을 클릭하십시오.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/create-01.png)


아래와 같이 서버 생성 페이지가 표시됩니다. 이 문서의 서버 기본 정보, 노드 구성의 내용을 참고하여 서버를 생성하십시오.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/create-new.png)


## 서버 기본 정보

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/create-03.png)

* 서버 이름: 현재 구성할 서버를 나타내는 고유한 식별자를 지정합니다. 동일한 식별자를 가진 서버를 여러 대 시작할 경우에는 해당 식별자 뒤에 1부터 1씩 증가하는 고유한 넘버링이 추가됩니다.

* 서버 수량: 시작할 서버의 수량을 설정합니다. 

* 인스턴스 타입: 생성할 서버 인스턴스의 타입을 선택합니다.

* 배포 파일: 서버를 시작하기 위해 업로드된 서버 바이너리를 선택합니다. 선택 전 반드시 최소 하나 이상의 서버 바이너리가 업로드되어 있어야 합니다. 배포 파일 탭에서 서버 바이너리를 관리할 수 있습니다.

* 태그: 서버 구성에 대해 임의의 태그를 작성해 둘 수 있습니다.

* 메모: 해당 서버에 대한 간단한 메모를 작성할 수 있습니다. 이 메모는 서버 구성이나 운영에 영향을 주는 값이 아닙니다.

## 노드 구성

GameAnvil의 노드는 게임 서버의 기능 단위입니다. 개발이 완료된 서버 바이너리는 게임에서 사용할 모든 종류의 노드 구현을 포함하고 있습니다. 이러한 바이너리 로그로 구동되는 서버 프로세스는 설정에 따라 임의의 선택된 노드로만 구성할 수 있으며, 서버 생성 전 등록해 둔 구성 정보(Config)를 바탕으로 설정합니다. 즉, 구성 정보(Config)에는 노드 구성 정보가 포함되어 있습니다. 여러 개의 구성 정보(Config)가 등록되어 있을 경우 적용할 구성을 목록에서 선택할 수 있습니다. 또한, 구성 정보에 포함된 노드 설정을 노드의 종류별로 활성화하거나 비활성화할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/create-04.png)


위의 이미지는 다음과 같이 총 4개의 게임 노드들로 서버를 구성합니다.

| 노드 종류   | 서비스명    | 구동 개수 | 설명                                            |
|---------|---------|-------|-----------------------------------------------|
| GAME    | RPSGame | 4     | 게임 콘텐츠가 구현된 게임 노드이며 각각 ch1~ch4까지의 채널을 담당한다. |



이때 구성 정보(Config)에 입력한 서비스명은 서버 개발에 사용된 정보와 동일해야 합니다. 서버 구현에 사용된 정보와 다른 값을 입력할 경우 서버가 정상적으로 구동되지 않습니다. 채널값 또한 구성 정보(Config)에 설정해 둔 값이 그대로 노출됩니다.

서버 생성과 동시에 자동으로 서버를 시작하려면 체크 박스를 클릭해 체크합니다. 기본값(생성 즉시 시작)을 사용할 것을 권장합니다.

## 생성된 서버 확인
서버를 구성한 뒤 실행하면 설정한 수량만큼 서버가 생성됩니다. 생성된 서버는 **TRANSIT** 상태로 시작되며 노란색으로 표시됩니다. 서버가 **RUNNING** 상태로 진행되기까지 NHN Cloud 인프라 상태에 따라 수 분(1~60분 이상)이 소요될 수 있습니다. 특히 GameAnvil 서비스가 최초의 서버를 구동할 때에는 기본 인프라 설정들을 동시에 진행하므로 조금 더 긴 시간이 소요될 수 있습니다. 사용자는 이 시간 동안 다른 종류의 서버들을 추가로 구성하거나 다른 메뉴를 사용할 수 있습니다. 

서버가 정상적으로 구동되면 이미지와 같이 **RUNNING** 상태로 바뀝니다. 이렇게 생성된 서버를 클릭하면 다음과 같이 서버 정보를 확인할 수 있습니다. 이 때, 사용자는 서버 제어 명령을 통해 해당 서버를 종료시키고 삭제하거나 재부팅 등을 할 수 있습니다. 혹은 다중 선택 버튼을 통해 한 번에 여러 개의 서버를 동시에 선택하여 명령을 수행할 수도 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/created.png)

서버 목록에서 임의의 항목을 클릭하면 서버 상세 정보를 확인할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/detail.png)


## 생성된 노드 정보 확인

생성된 서버를 구성하는 모든 노드 정보는 노드 탭에서 확인할 수 있습니다.

**상태**, **노드 타입**, **서버 타입**뿐만 아니라 **노드 ID** 등 좀 더 상세하고 명확한 정보를 기반으로 임의의 노드 정보만 필터링할 수도 있습니다. 
![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/node.png)

## 로드 밸런서

GameAnvil 서비스는 클라이언트의 접속을 효율적으로 처리하기 위해 로드 밸런서를 연동합니다. GameAnvil의 노드 중 클라이언트가 접속할 수 있는 포인트는 게이트웨이 노드와 서포트 노드입니다. 그리고 이들은 모두 여러 대의 인스턴스로 구성될 수 있습니다. 이때, 여러 대의 접속 포인트는 적절한 부하 분산을 위해 로드 밸런서와 연동되어야 합니다. 또한 해당 로드 밸런서는 외부에서 접속 가능한 플로팅 IP에 연결되어야 합니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/new-server/lb.png)

이와 같은 일련의 과정은 GameAnvil 서비스에서 자동으로 처리하며, **로드 밸런서 정보**를 클릭해 접속 정보를 확인할 수 있습니다. 해당 정보는 클라이언트의 접속 주소/포트 쌍이므로 불필요하게 외부에 전달되지 않도록 주의해야 합니다.


