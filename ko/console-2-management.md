## Game > GameAnvil > 콘솔 사용 가이드 > 관리


## 머신

GameAnvil 프로세스가 실행 될 장비를 등록하고 등록 된 머신을 관리하는 메뉴 입니다. 

머신 등록 전 해당 머신에 GameAnvil Agent 를 설치해야 합니다. [ [GameAnvil-Agent Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-agent-1.1.4.tar) ] 

![management_machine_main.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_main_ko.png)

### 머신 등록

GameAnvil 프로세스가 실행 될 장비를 등록 합니다. 

GameAnvil 서비스를 활성화 시킨 후 가장 먼저 머신 등록 작업이 진행되어야 합니다.

![management_machine_register.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_register_ko.png)
* 입력 타입 : 콘솔 창에 직접 머신 정보를 입력하여 등록 할 수 있고 .csv 형식 파일을 업로드 하여 다수의 머신을 등록 할 수 있습니다.
* 호스트 이름 : 등록 할 장비의 호스트명을 입력 합니다.
* IP 주소 : 등록 할 장비의 Public IP 주소를 입력합니다. 해당 주소를 통해 콘솔과 통신하므로 정확한 주소 입력이 필요합니다. 
* GameAnvil Agent Port : 해당 장비에 설치 된 GameAnvil Agent Port 정보를 입력합니다. ( 기본 값은 19080 입니다. )
    * 파일 업로드 : 템플릿 예시 파일을 다운로드 받아 등록 할 머신 정보를 입력 후 파일을 업로드 합니다. [ [Template File](https://static.toastoven.net/prod_gameanvil/files/GameAnvil_Machine_Template.csv)]


### 머신 목록
등록 된 머신 리스트를 확인할 수 있으며 호스트 이름, IP 주소, 설명 검색 기능을 제공합니다. 
![management_machine_list.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_list_ko.png)

### 머신 설정 

머신 등록 후 마스터 머신과 로케이션 관리 머신을 설정합니다. 

등록 한 머신중 한개의 마스터머신과 한개 이상의 로케이션 관리 머신을 선택합니다. 

![management_machine_setup.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_setup_ko.png)
* 마스터 머신 : Management Node 가 실행 될 머신을 선택합니다. 마스터 머신이 등록 되어야 인스턴스 / 노드 제어가 가능합니다. 
* 로케이션 관리 머신 : Location Node 가 실행 될 머신을 선택합니다. 
* Java 버전 : GameAnvil 서버 빌드  사용 된 Java 버전을 선택합니다. 

Management Node 와 Location Node 는 GameAnvil 을 구성하는 필수 노드이지만 콘텐츠 구현 및 제어가 불가한 노드입니다. 
머신 설정 / 머신 초기화 기능을 통해 Management Node 와 Location Node 를 관리할 수 있습니다. 

### 머신 설정 초기화

머신 설정 기능을 통해 등록 한 마스터 머신과 로케이션 관리 머신 설정을 초기화 해주는 기능입니다.

기존에 설정 된 머신 설정값이 초기화 되면서 Management Node 와 Location Node 도 종료됩니다. 

기존에 등록 된 모든 인스턴스 상태가 동작 중이 아닌 경우에만 머신 설정 초기화가 가능합니다.

![management_machine_init.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_init_ko.png)

![management_machine_setup_none.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_setup_none_ko.png)



## 인스턴스

머신 관리를 통해 등록한 머신에 띄울 인스턴스를 관리합니다.

인스턴스는 여러 [노드](server-2-basic) 들로 구성되며 게임 환경에 따라 인스턴스를 원하는 형태로 구성할 수 있습니다.

### 목록

등록된 인스턴스의 목록을 확인할 수 있습니다.

인스턴스에 대한 기본적인 정보와 구성된 노드 정보들이 노출됩니다.

![ManagementInstance-0](https://static.toastoven.net/prod_gameanvil/images/management_instance_list_ko.png)

노드타입별 필터 기능과 인스턴스 이름, 호스트 이름 검색기능을 제공합니다.

필터와 검색어는 AND 조건으로 동작됩니다.

![ManagementInstance-1](https://static.toastoven.net/prod_gameanvil/images/management_instance_filter_ko.png)

인스턴스에 구성된 노드들의 상세 설정을 **확인** 버튼을 클릭하여 팝업을 통해 확인할 수 있습니다.

상단에 구성된 탭을 통해 노드별 확인이 가능합니다.

![ManagementInstance-2](https://static.toastoven.net/prod_gameanvil/images/management_instance_config_popup_ko.png)

### 등록

인스턴스 등록 화면입니다.

먼저 인스턴스에 대한 기본 정보를 입력하고 원하는 구성의 노드들을 설정합니다.

노드 설정의 경우 템플릿 형태로만 입력이 가능하며, 템플릿에 대한 자세한 설명은 **인스턴스 > 설정** 가이드를 통해 확인하실 수 있습니다.

인스턴스의 기본 정보를 입력하는 등록 화면의 최상단입니다.

항목별 입력값은 실시간으로 확인하여 형식에 맞지 않는 값이 입력된 경우 안내 메세지를 보여줍니다.

**서버 빌드 업로드 경로**와 **머신**의 경우 한번 등록된 이후에는 수정이 불가하다는 점에 유의해야 합니다.

![ManagementInstance-3](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_1_ko.png)

인스턴스의 기본 정보(인스턴스 이름, 서버 빌드 업로드 경로)를 입력하면 인스턴스를 배포할 머신을 선택할 수 있습니다.

머신 관리에서 등록한 머신의 목록이 팝업에 보여집니다.

동일한 구성의 인스턴스를 다수의 머신을 선택하여 등록할 수 있습니다.

![ManagementInstance-12](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_2_ko.png)

기본 정보에 이어 각 노드의 설정을 진행합니다.

노드 타입 중 **COMMON, VM_OPTION**은 반드시 설정되어야 하며, **GATEWAY, GAME, SUPPORT, MATCH** 중 하나 이상의 노드를 반드시 설정해야 합니다.

노드의 설정은 **설정 불러오기**를 이용해 미리 등록된 템플릿을 선택하는 방식으로만 가능합니다.

![ManagementInstance-4](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_3_ko.png)

설정 불러오기 팝업을 통해 등록된 템플릿들의 각 노드별 설정을 확인할 수 있습니다.

템플릿 목록 중 하나를 선택하면 기본 설정을 제외한 모든 설정이 등록 화면에 채워집니다.

동일한 구성의 인스턴스를 등록하거나 관리할 때 유용하게 사용될 수 있도록 제공하는 기능입니다.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_4_ko.png)

**설정 불러오기**를 하면 **COMMON, GATEWAY, SUPPORT**에 대한 Port 정보를 입력해야 합니다.

**GATEWAY, SUPPORT** 노드는 설정 여부에 따라 Port 입력을 요구합니다.

**GATEWAY** 노드가 설정된 화면입니다.

**TCP/WEB SOCKET** 사용여부에 따라 Port 입력이 동적으로 구성됩니다.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_5_ko.png)

**SUPPORT** 노드가 설정된 화면입니다.

설정된 **서비스수** 에 따라 Port 입력이 동적으로 구성됩니다.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_6_ko.png)

**GATEWAY, SUPPORT** 설정에 따라 구성된 Port 입력 영역입니다.

Port는 **18000 ~ 20000** 사이의 값이 사용되며, 동일 머신 내에서 중복되지 않게 사용되어야 합니다.

구성된 Port 입력을 마친 후 **포트 중복 확인**을 통해 먼저 등록된 인스턴스들과의 Port 중복 체크를 진행해야 합니다.  

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_7_ko.png)

**포트 중복 확인**에 의해 중복된 Port가 입력된 경우 안내 팝업을 통해 중복된 Port 정보를 확인할 수 있습니다.

이 정보를 토대로 Port를 재입력한 후 중복되지 않는 입력값이 채워질 때까지 **포트 중복 확인** 과정을 반복해야 합니다.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_8_ko.png)

**포트 중복 확인**이 정상적으로 완료가 되면, 인스턴스 등록 화면 내 모든 입력필드가 **비활성화** 상태가 되며, 최종적으로 등록이 가능한 상태가 됩니다.

저장하기 전에 일부 설정을 변경하려면 **설정 다시 입력** 버튼을 클릭하여 **활성화** 상태로 전환한 후 변경할 수 있습니다.

**활성화** 상태가 되면 다시 **포트 중복 확인** 과정을 거쳐야 등록 가능한 상태가 됩니다.

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_9_ko.png)

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_10_ko.png)

### 수정 및 삭제

등록된 인스턴스를 수정하거나 삭제할 수 있습니다.

인스턴스를 수정, 삭제 하려면 인스턴스의 상태가 **시작 대기, 중지, 에러** 상태일 때만 가능합니다.

**모니터링 > 인스턴스 모니터링**에서 인스턴스의 현재 상태를 확인하거나 변경할 수 있습니다.

![ManagementInstance-15](https://static.toastoven.net/prod_gameanvil/images/management_instance_modify_delete_ko.png)

![ManagementInstance-16](https://static.toastoven.net/prod_gameanvil/images/management_instance_start_ko.png)

인스턴스 수정에서 **서버 빌드 업로드 경로**와 **머신**설정은 변경할 수 없습니다.

인스턴스의 노드 설정은 등록 방법과 동일하게 **설정 불러오기**를 통해 변경할 수 있습니다.

![ManagementInstance-17](https://static.toastoven.net/prod_gameanvil/images/management_instance_modify_ko.png)

## 설정

인스턴스의 각 타입별 노드들의 템플릿을 관리합니다.

인스턴스 등록시 각 노드들의 설정은 직접 입력하지 않고 템플릿 선택을 통해서만 설정이 가능합니다.

자주 사용하게 될 각 노드별 설정을 미리 등록하여 인스턴스 등록에 편리하게 사용하실 수 있습니다.

### 목록

등록된 템플릿의 목록을 확인할 수 있습니다.

템플릿 이름으로 정렬되며 해당 템플릿을 사용중인 인스턴스 현황에 대해서도 확인할 수 있습니다.

템플릿 이름 검색기능을 제공합니다.

![ManagementTemplate-1](https://static.toastoven.net/prod_gameanvil/images/management_template_list_ko.png)

![ManagementTemplate-2](https://static.toastoven.net/prod_gameanvil/images/management_template_list_popup_ko.png)

### 등록

템플릿 등록 화면입니다.

인스턴스의 각 노드에 적용할 설정을 템플릿으로 등록합니다.

인스턴스 등록 화면에서는 각 노드의 설정을 직접 입력이 불가하고 템플릿으로만 설정이 가능하기 때문에 필요한 설정들을 미리 등록해야 합니다.

기본 설정값을 제공하니 이를 이용하여 템플릿 등록을 진행할 수 있습니다.

게임 특성에 맞게 템플릿들을 등록하고 활용해보세요.

먼저 기본 정보인 템플릿 이름과 설명을 입력합니다. 

각 노드별 설정은 직접 설정값을 입력해 등록할 수 있고, **설정 불러오기**를 통해 기존에 등록된 템플릿 설정을 불러와 등록할 수도 있습니다.

유사한 템플릿을 등록하고자 할 때 **설정 불러오기**로 설정을 불러온 후 변경이 필요한 일부 설정값만 직접 입력하여 등록하시면 됩니다.  

![ManagementTemplate-3](https://static.toastoven.net/prod_gameanvil/images/management_template_register_1_ko.png)

노드 타입 중 **COMMON, VM_OPTION**은 반드시 설정되어야 하며, **GATEWAY, GAME, SUPPORT, MATCH** 중 하나 이상의 노드를 반드시 설정해야 합니다.

**COMMON** 노드의 설정값 입니다.

![ManagementTemplate-4](https://static.toastoven.net/prod_gameanvil/images/management_template_register_2_ko.png)

**VM OPTION** 노드의 설정값 입니다.

![ManagementTemplate-5](https://static.toastoven.net/prod_gameanvil/images/management_template_register_3_ko.png)

**GATEWAY** 노드의 설정값 입니다.

TCP/WEB SOCKET, SSL 사용 여부에 따라 설정값이 달라집니다.

![ManagementTemplate-6](https://static.toastoven.net/prod_gameanvil/images/management_template_register_4_ko.png)

**GAME** 노드의 설정값 입니다.

![ManagementTemplate-7](https://static.toastoven.net/prod_gameanvil/images/management_template_register_5_ko.png)

GAME 노드는 **서비스**를 선택하여 설정해야 합니다.

**서비스 선택** 팝업을 통해 최대 99개의 서비스를 선택하여 설정할 수 있습니다. 

서비스는 **관리 > 인스턴스 > 서비스** 메뉴에서 관리할 수 있습니다. 

![ManagementTemplate-8](https://static.toastoven.net/prod_gameanvil/images/management_template_register_6_ko.png)

원하는 서비스를 선택한 후 확인하면 각 서비스별로 필요한 설정값을 추가로 입력하게 됩니다. 

![ManagementTemplate-9](https://static.toastoven.net/prod_gameanvil/images/management_template_register_7_ko.png)  

SUPPORT 노드도 **서비스**를 선택하여 설정해야 합니다.

GAME 노드와 동일하게 **서비스 선택** 팝업을 통해 최대 99개의 서비스를 선택한 후 설정값을 추가로 입력하게 됩니다. 

![ManagementTemplate-10](https://static.toastoven.net/prod_gameanvil/images/management_template_register_8_ko.png)

템플릿 등록 중 미리 등록해두지 못한 서비스가 있다면 **서비스 선택** 팝업에서 바로 등록할 수 있습니다.

**서비스 등록** 버튼을 클릭하면 등록 팝업이 나타납니다.
 
여기서 서비스 ID, 서비스 이름을 입력하여 서비스를 등록할 수 있습니다.

등록만 가능하며, 수정이나 삭제는 **관리 > 인스턴스 > 서비스** 메뉴에서 진행할 수 있습니다.   

![ManagementTemplate-11](https://static.toastoven.net/prod_gameanvil/images/management_template_register_9_ko.png)

![ManagementTemplate-12](https://static.toastoven.net/prod_gameanvil/images/management_template_register_10_ko.png)

마지막으로 **MATCH** 노드의 설정값 입니다.

![ManagementTemplate-13](https://static.toastoven.net/prod_gameanvil/images/management_template_register_11_ko.png)

템플릿은 입력 항목 별로 기본 값과 Min, Max값 제한이 적용된 항목들이 있습니다.

**설정값 초기화** 기능은 해당 노드의 설정을 기본 값으로 되돌립니다.

**GAME**, **SUPPORT** 노드는 **설정값 초기화** 기능을 제공하지 않습니다.

Min, Max 범위를 벗어나는 설정값이 입력된 경우 해당 필드가 FocusOut될 때 필드에 설정된 Min, Max 값으로 자동 변경됩니다.

### 수정 및 삭제

등록된 템플릿을 수정하거나 삭제할 수 있습니다.

템플릿을 수정, 삭제 하려면 해당 템플릿이 적용된 모든 인스턴스의 상태가 **시작 대기, 중지, 에러** 상태일 때만 가능합니다.

템플릿을 사용중인 인스턴스는 **설정 목록**에서 팝업을 통해 확인할 수 있으며, **인스턴스 모니터링**에서 인스턴스의 현재 상태를 확인하거나 변경할 수 있습니다.

![ManagementTemplate-15](https://static.toastoven.net/prod_gameanvil/images/management_template_modify_delete_ko.png)

템플릿 수정에서는 **설정 불러오기** 기능이 제공되지 않기 때문에 필요한 설정값을 직접 수정해야 합니다.

![ManagementTemplate-16](https://static.toastoven.net/prod_gameanvil/images/management_template_modify_ko.png)

그리고 인스턴스에 적용된 템플릿인 경우 인스턴스 상태와 관계없이 삭제가 불가합니다.

해당 인스턴스에서 템플릿을 변경하거나 인스턴스를 삭제한 후 템플릿을 삭제할 수 있습니다.

![ManagementTemplate-17](https://static.toastoven.net/prod_gameanvil/images/management_template_delete_ko.png)

## 배포파일

GameAnvil 인스턴스에 배포되는 프로세스 파일을 관리하는 메뉴 입니다. 

![ManagementDeployFile-1](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_ko.png)

### 파일 업로드

파일 업로드 메뉴를 통해 배포파일 업로드가 가능하며 한번에 1개의 파일만 업로드가 가능합니다.

확장자가 .jar 인 파일만 업로드를 지원하고 파일 크기는 최대 1GB 까지 허용됩니다. 

업로드 버튼 클릭시 파일 업로드만 수행되며 인스턴스 모니터링 메뉴에서 **배포하기** 기능을 실행하여야 배포가 완료됩니다.

![ManagementDeployFile-2](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_upload_ko.png)

### 배포 이력

배포 파일 목록에서 배포이력 확인 클릭 시 해당 배포파일의 배포 이력이 하단에 노출 됩니다.

배포가 수행 된 인스턴스, 머신, 배포 일시 등의 정보를 확인할 수 있습니다. 


![ManagementDeployFile-3](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_history_ko.png)