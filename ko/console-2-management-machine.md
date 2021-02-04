GameAnvil 프로세스가 실행 될 장비를 등록하는 메뉴입니다. 

머신 등록 전 해당 머신에 GameAnvil Agent 를 설치해야 합니다. [ [GameAnvil Download](http://static.toastoven.net/prod_gameanvil/files/GameAnvil-Template.zip) ] 

![management_machine_1.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_1.png)


1.머신 등록

![management_machine_2.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_2.png)
* 입력 타입 : 콘솔 창에 직접 머신 정보를 입력하여 등록 할 수 있고 .csv 형식 파일을 업로드 하여 다수의 머신을 등록 할 수 있습니다.
* 호스트 이름 : 등록 할 장비의 호스트명을 입력 합니다.
* IP 주소 : 등록 할 장비의 Public IP 주소를 입력합니다. 해당 주소를 통해 콘솔과 통신하므로 정확한 주소 입력이 필요합니다. 
* GameAnvil Agent Port : 해당 장비에 설치 된 GameAnvil Agent Port 정보를 입력합니다. ( 기본 값은 19080 입니다. )
    * 파일 업로드 : 템플릿 예시 파일을 다운로드 받아 등록 할 머신 정보를 입력 후 파일을 업로드 합니다. [ [Template File]()]


2.머신 설정 

머신 등록 후 마스터 머신과 로케이션 관리 머신을 설정합니다. 

등록 한 머신중 한개의 마스터머신과 한개 이상의 로케이션 관리 머신을 선택합니다. 

![management_machine_3.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_3.png)
* 마스터 머신 : Management Node 가 실행 될 머신을 선택합니다. 마스터 머신이 등록 되어야 인스턴스 / 노드 제어가 가능합니다. 
* 로케이션 관리 머신 : Location Node 가 실행 될 머신을 선택합니다. 
* Java 버전 : GameAnvil 서버 빌드  사용 된 Java 버전을 선택합니다. 

Management Node 와 Location Node 는 GameAnvil 을 구성하는 필수 노드이지만 콘텐츠 구현 및 제어가 불가한 노드입니다. 
머신 설정 / 머신 초기화 기능을 통해 Management Node 와 Location Node 를 관리할 수 있습니다. 

3.머신 설정 초기화

머신 설정 기능을 통해 등록 한 마스터 머신과 로케이션 관리 머신 설정을 초기화 해주는 기능입니다.

기존에 설정 된 머신 설정값이 초기화 되면서 Management Node 와 Location Node 도 종료됩니다. 

등록 된 모든 인스턴스 상태가 동작 중이 아닌 경우에만 머신 설정 초기화가 가능합니다.

![management_machine_4.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_4.png)

![management_machine_5.png](http://static.toastoven.net/prod_gameanvil/images/management_machine_5.png)

