## Game > GameAnvil > 관리 > 머신


### 머신

GameAnvil 프로세스가 실행 될 장비를 등록하는 메뉴입니다. 

머신 등록 전 해당 머신에 GameAnvil Agent 를 설치해야 합니다. [ [GameAnvil Download](http://static.toastoven.net/prod_gameanvil/files/GameAnvil-Template.zip) ] 

![management_machine_1.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_1.png)

#### 머신 등록

![management_machine_2.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_2.png)
* 입력 타입 : 콘솔 창에 직접 머신 정보를 입력하여 등록 할 수 있고 .csv 형식 파일을 업로드 하여 다수의 머신을 등록 할 수 있습니다.
* 호스트 이름 : 등록 할 장비의 호스트명을 입력 합니다.
* IP 주소 : 등록 할 장비의 Public IP 주소를 입력합니다. 해당 주소를 통해 콘솔과 통신하므로 정확한 주소 입력이 필요합니다. 
* GameAnvil Agent Port : 해당 장비에 설치 된 GameAnvil Agent Port 정보를 입력합니다. ( 기본 값은 19080 입니다. )
    * 파일 업로드 : 템플릿 예시 파일을 다운로드 받아 등록 할 머신 정보를 입력 후 파일을 업로드 합니다. [ [Template File]()]


#### 머신 설정 

머신 등록 후 마스터 머신과 로케이션 관리 머신을 설정합니다. 

등록 한 머신중 한개의 마스터머신과 한개 이상의 로케이션 관리 머신을 선택합니다. 

![management_machine_3.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_3.png)
* 마스터 머신 : Management Node 가 실행 될 머신을 선택합니다. 마스터 머신이 등록 되어야 인스턴스 / 노드 제어가 가능합니다. 
* 로케이션 관리 머신 : Location Node 가 실행 될 머신을 선택합니다. 
* Java 버전 : GameAnvil 서버 빌드  사용 된 Java 버전을 선택합니다. 

Management Node 와 Location Node 는 GameAnvil 을 구성하는 필수 노드이지만 콘텐츠 구현 및 제어가 불가한 노드입니다. 
머신 설정 / 머신 초기화 기능을 통해 Management Node 와 Location Node 를 관리할 수 있습니다. 

#### 머신 설정 초기화

머신 설정 기능을 통해 등록 한 마스터 머신과 로케이션 관리 머신 설정을 초기화 해주는 기능입니다.

기존에 설정 된 머신 설정값이 초기화 되면서 Management Node 와 Location Node 도 종료됩니다. 

등록 된 모든 인스턴스 상태가 동작 중이 아닌 경우에만 머신 설정 초기화가 가능합니다.

![management_machine_4.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_4.png)

![management_machine_5.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_5.png)


## Game > GameAnvil > 관리 > 인스턴스

### 인스턴스

머신 관리를 통해 등록한 머신에 띄울 인스턴스를 관리합니다.

인스턴스는 여러 [노드](server-2-basic) 들로 구성되며 게임 환경에 따라 인스턴스를 원하는 형태로 구성할 수 있습니다.

#### 목록

등록된 인스턴스의 목록을 확인할 수 있습니다.

인스턴스에 대한 기본적인 정보와 구성된 노드 정보들이 노출됩니다.

![ManagementInstance-0](https://static.toastoven.net/prod_gameanvil/images/management_instance_0.png)

노드타입별 필터 기능과 인스턴스 이름, 호스트 이름 검색기능을 제공합니다.

필터와 검색어는 AND 조건으로 동작됩니다.

![ManagementInstance-1](https://static.toastoven.net/prod_gameanvil/images/management_instance_1.png)

인스턴스에 구성된 노드들의 상세 설정을 **확인**버튼을 클릭하여 팝업을 통해 확인할 수 있습니다.

상단에 구성된 탭을 통해 노드별 확인이 가능합니다.

![ManagementInstance-2](https://static.toastoven.net/prod_gameanvil/images/management_instance_2.png)

#### 등록

인스턴스 등록 화면입니다.

먼저 인스턴스에 대한 기본 정보를 입력하고 원하는 구성의 노드들을 설정합니다.

노드 설정의 경우 템플릿 형태로만 입력이 가능하며, 템플릿에 대한 자세한 설명은 **인스턴스 > 설정** 가이드를 통해 확인하실 수 있습니다.

인스턴스의 기본 정보를 입력하는 등록 화면의 최상단입니다.

항목별 입력값은 실시간으로 확인하여 형식에 맞지 않는 값이 입력된 경우 안내 메세지를 보여줍니다.

**서버 빌드 업로드 경로**와 **머신**의 경우 한번 등록된 이후에는 수정이 불가하다는 점에 유의해야 합니다.

![ManagementInstance-3](https://static.toastoven.net/prod_gameanvil/images/management_instance_3.png)

인스턴스의 기본 정보(인스턴스 이름, 서버 빌드 업로드 경로)를 입력하면 인스턴스를 배포할 머신을 선택할 수 있습니다.

머신 관리에서 등록한 머신의 목록이 팝업에 보여집니다.

동일한 구성의 인스턴스를 다수의 머신을 선택하여 등록할 수 있습니다.

![ManagementInstance-12](https://static.toastoven.net/prod_gameanvil/images/management_instance_12.png)

기본 정보에 이어 각 노드의 설정을 진행합니다.

노드 타입 중 **COMMON, VM_OPTION**은 반드시 설정되어야 하며, **GATEWAY, GAME, SUPPORT, MATCH** 중 하나 이상의 노드를 반드시 설정해야 합니다.

노드의 설정은 템플릿을 통해서만 가능합니다.

![ManagementInstance-4](https://static.toastoven.net/prod_gameanvil/images/management_instance_4.png)

먼저 등록된 템플릿을 선택하거나 설정 선택 항목에서 **설정 템플릿 추가**를 통해 새로운 템플릿을 바로 추가해서 지정할 수 있습니다.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_14.png)

노드 설정을 하면 **COMMON, GATEWAY, SUPPORT**에 대한 Port 정보를 입력해야 합니다.

**GATEWAY, SUPPORT** 노드는 설정 여부에 따라 Port 입력을 요구합니다.

Port는 **18000 ~ 20000** 사이의 값이 사용되며, 동일 머신 내에서 중복되지 않게 사용되어야 합니다.

![ManagementInstance-5](http://static.toastoven.net/prod_gameanvil/images/management_instance_5.png)

**GATEWAY** 노드가 설정된 화면입니다.

**TCP/WEB SOCKET** 사용여부에 따라 Port 입력이 동적으로 구성됩니다.

![ManagementInstance-6](https://static.toastoven.net/prod_gameanvil/images/management_instance_6.png)

**SUPPORT** 노드가 설정된 호면입니다.

설정된 **서비스수** 에 따라 Port 입력이 동적으로 구성됩니다.

![ManagementInstance-7](https://static.toastoven.net/prod_gameanvil/images/management_instance_7.png)

**GATEWAY, SUPPORT** 설정에 따라 구성된 Port 입력 영역입니다.

구성된 Port 입력을 마친 후 **중복 확인**을 통해 먼저 등록된 인스턴스들과의 Port 중복 체크를 할 수 있습니다. 

![ManagementInstance-8](https://static.toastoven.net/prod_gameanvil/images/management_instance_8.png)

**중복 확인**에 의해 중복된 Port가 입력된 경우 안내 팝업을 통해 중복된 Port 정보를 확인할 수 있습니다.

이 정보를 토대로 Port를 재입력한 후 중복되지 않는 입력값이 채워질 때까지 **중복 확인** 과정을 반복해야 합니다.

![ManagementInstance-9](https://static.toastoven.net/prod_gameanvil/images/management_instance_9.png)

Port 중복 확인이 정상적으로 완료가 되면, 인스턴스 등록 화면 내 모든 입력필드가 **비활성화** 상태가 되며, 최종적으로 등록이 가능한 상태가 됩니다.

저장하기 전에 일부 설정을 변경하려면 **설정 다시 입력** 버튼을 클릭하여 **활성화** 상태로 전환한 후 변경할 수 있습니다.

**활성화** 상태가 되면 다시 **중복 확인** 과정을 거쳐야 등록 가능한 상태가 됩니다.

![ManagementInstance-11](https://static.toastoven.net/prod_gameanvil/images/management_instance_11.png)

![ManagementInstance-10](https://static.toastoven.net/prod_gameanvil/images/management_instance_10.png)

위에서 설명한 등록 방식 외에 인스턴스를 등록할 수 있는 다른 방법이 있습니다.

**설정 불러오기** 기능입니다.


![ManagementInstance-13](https://static.toastoven.net/prod_gameanvil/images/management_instance_13.png)

**설정 불러오기**는 먼저 등록된 인스턴스의 설정을 그대로 복사하는 기능입니다.

설정 불러오기 팝업을 통해 등록된 인스턴스들의 노드 설정을 확인할 수 있습니다.

인스턴스 목록 중 하나를 선택하면 기본 설정을 제외한 모든 설정이 등록 화면에 채워집니다.

동일한 구성의 인스턴스를 추가 등록할 때 유용하게 사용될 수 있는 기능입니다.

#### 수정 및 삭제

등록된 인스턴스를 수정하거나 삭제할 수 있습니다.

인스턴스를 수정, 삭제 하려면 인스턴스의 상태가 **시작 대기, 중지, 에러** 상태일 때만 가능합니다.

**인스턴스 모니터링**에서 인스턴스의 현재 상태를 확인하거나 변경할 수 있습니다.

![ManagementInstance-15](https://static.toastoven.net/prod_gameanvil/images/management_instance_15.png)

![ManagementInstance-16](https://static.toastoven.net/prod_gameanvil/images/management_instance_16.png)

### 설정

인스턴스의 각 타입별 노드들의 템플릿을 관리합니다.

인스턴스 등록시 각 노드들의 설정은 직접 입력하지 않고 템플릿 선택을 통해서만 설정이 가능합니다.

자주 사용하게 될 각 노드별 설정을 미리 등록하여 인스턴스 등록에 편리하게 사용하실 수 있습니다.

#### 목록

등록된 템플릿의 목록을 확인할 수 있습니다.

노드 타입별로 정렬되며 해당 템플릿을 사용중인 인스턴스 현황에 대해서도 확인할 수 있습니다.

템플릿 역시 인스턴스와 마찬가지로 노드타입별 필터 기능과 템플릿 이름 검색기능을 제공합니다.

필터와 검색어는 AND 조건으로 동작됩니다.

![ManagementTemplate-1](https://static.toastoven.net/prod_gameanvil/images/management_template_1.png)

![ManagementTemplate-2](https://static.toastoven.net/prod_gameanvil/images/management_template_2.png)

#### 등록

템플릿 등록 화면입니다.

![ManagementTemplate-3](https://static.toastoven.net/prod_gameanvil/images/management_template_3.png)


#### 수정 및 삭제

