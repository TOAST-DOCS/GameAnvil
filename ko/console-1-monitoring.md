## Game > GameAnvil > 모니터링

## Game > GameAnvil > 모니터링 > 대시보드

### 대시보드

관리 페이지에서 등록한 머신과 인스턴스, 각각의 노드들에 대한 상태를 확인할 수 있습니다.

![monitoring_dashboard_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_dashboard_main.png)

- 머신
    - 관리 페이지에 등록한 전체 머신 수와 머신에 설치한 Agent와 정상적으로 연결된 전체 머신 수를 나타냅니다. 
        - 연결된 머신 수 / 전체 머신 수
    - Agent와 연결에 문제가 있으면, 빨간색 상태 표시가 됩니다.
    - 모니터링 버튼을 통하여 '[시스템 모니터링](game-gameanvil_3)' 페이지로 이동됩니다.
- 인스턴스
    - 관리 페이지에 등록한 전체 인스턴스 수와 동작(Running) 중인 인스턴스 수를 표시합니다.
        - 동작 중인 인스턴스 수 / 모든 인스턴스 수
    - 에러인 인스턴가 하나라도 있으면 빨간색 상태 표시가 됩니다.
    - 모니터링 버튼을 통하여 '[인스턴스 모니터링](game-gameanvil_4)' 페이지로 이동됩니다.
- 유저
    - GAME 노드에 접속한 유저 수
- 룸
    - GAME 노드에 생성된 룸 수
- 세션
    - GATEWAY 노드에서 확인 된 세션 수
- GATEWAY
    - 전체 GATEWAY 노드 수와 READY 상태인 GATEWAY 노드 수를 표시합니다. 
        - READY 상태인 GATEWAY 노드 수 / 전체 GATEWAY 노드 수
    - DISABLE 상태인 GATEWAY 노드가 하나라도 있으면 빨간색 상태 표시가 됩니다.
    - 모니터링 버튼을 통하여 'GATEWAY 노드 모니터링' 페이지로 이동됩니다.
- GAME
    - 전체 GAME 노드 수와 READY 상태인 GAME 노드 수를 표시합니다. 
        - READY 상태인 GAME 노드 수 / 전체 GAME 노드 수
    - DISABLE 상태인 GAME 노드가 하나라도 있으면 빨간색 상태 표시가 됩니다.
    - 모니터링 버튼을 통하여 'GAME 노드 모니터링' 페이지로 이동됩니다.
- SUPPORT
    - 전체 SUPPORT 노드 수와 READY 상태인 SUPPORT 노드 수를 표시합니다.
        - READY 상태인 SUPPORT 노드 수 / 전체 SUPPORT 노드 수
    - DISABLE 상태인 SUPPORT 노드가 하나라도 있으면 빨간색 상태 표시가 됩니다.
    - 모니터링 버튼을 통하여 'SUPPORT 노드 모니터링' 페이지로 이동됩니다.
- MATCH
    - 전체 MATCH 노드 수와 READY 상태인 MATCH 노드 수를 표시합니다.
        - READY 상태인 MATCH 노드 수 / 전체 MATCH 노드 수
    - DISABLE 상태인 MATCH 노드가 하나라도 있으면 빨간색 상태 표시가 됩니다.
    - 모니터링 버튼을 통하여 'MATCH 노드 모니터링' 페이지로 이동됩니다.

### 동시 접속자 변화 그래프

GAME 노드에 접속한 사용자의 수를 그래프를 통하여 한 눈에 확인할 수 있습니다.

![monitoring_dashboard_graph](https://static.toastoven.net/prod_gameanvil/images/monitoring_dashboard_graph.png)

- 그래프는 1분마다 자동 갱신되며, 오늘과 어제 그리고 지난 주의 그래프를 확인할 수 있습니다.
- 어제 또는 지난 주 데이터가 없으면 그래프는 표시되지 않습니다.
- 필터를 통하여 특정 GAME 노드 서비스의 동시 접속자 그래프를 확인할 수 있습니다.



## Game > GameAnvil > 모니터링 > 유저분포 모니터링

### 유저분포 모니터링

GAME 노드의 접속한 동시접속자, 분포 평균 그리고 머신, 인스턴스, 노드의 수를 확인할 수 있습니다.

![monitoring_userdistribution_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_userdistribution_main.png)

- 동시접속자
    - 동작 중(READY)인 GAME 노드에서 집계된 동시접속자 수를 의미합니다.
    - 필터에 따라서 동시접속자 수는 달라집니다.

- 머신
    - GAME 노드가 동작 중(READY)인 머신의 수를 의미합니다.
    - 필터에 따라서 머신 수는 달라집니다.

- 인스턴스
    - GAME 노드가 동작 중(READY)인 인스턴스 수를 의미합니다.
    - 필터에 따라서 인스턴스 수는 달라집니다.

- GAME 노드
    - GAME 노드가 동작 중(READY)인 노드 수를 의미합니다.
    - 필터에 따라서 노드 수는 달라집니다.

- 노드별 유저 분포 평균
    - 동시접속자를 전체 구동 중인 GAME 노드 수로 나는 평균입니다.
    - 필터에 따라서 평균은 달라집니다.
        - 동시접속자 / 전체 구동 중인 GAME 노드의 수

※ 유저 모니터링 페이지는 자동 새로고침을 지원하지 않습니다.<br>
데이터 갱신이 필요할 때에는 새로고침 버튼을 통하여 갱신하면 됩니다.

### 유저분포 그래프

그래프는 탭<span style='color: #EB4927'>①</span>을 통하여 머신, 인스턴스, 노드별로 유저분포 상태를 확인할 수 있습니다.

- 머신
    - GAME 노드가 동작 중(READY)인 머신들만 나타납니다.
    - 세로 축 레이블은 `호스트명`으로 표시됩니다.
- 인스턴스
    - GAME 노드가 동작 중(READY)인 인스턴스만 나타납니다.
    - 세로 축 레이블은 `인스턴스 이름@호스트명`으로 표시됩니다.
- 노드
    - 동작 중(READY)인 GAME 노드만 나타납니다.
    - 세로 축 레이블은 `노드 ID@호스트명`으로 표시됩니다.

그래프의 데이터는 기본적으로 50개만 보여지며, 50개 이상의 정보는 '더보기'<span style='color: #EB4927'>②</span>으로 확인이 가능합니다.

## Game > GameAnvil > 모니터링 > 시스템 모니터링

시스템 모니터링에서는 머신과 인스턴스들의 상태와 간략한 정보들을 쉽게 확인할 수가 있습니다.

![monitoring_system_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_main.png)

### 트리뷰

관리 페이지에서 등록한 머신과 인스턴스들이 계층구조를 트리 형태로 표시합니다.

트리의 구조는 아래와 같이 구성되어 있습니다.

![monitoring_system_tree](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_tree.png)

머신과 각각의 머신에 속하는 인스턴스들이 표시됩니다.
호스트 이름 또는 인스턴스 이름를 선택하면 각각의 머신 또는 인스턴스 정보 페이지가 보여집니다.

또한 호스트명 검색을 통하여 찾고자하는 머신을 쉽게 찾을 수 있습니다.

#### 머신(호스트명)을 선택했을 떄

선택한 머신과 속하는 인스턴스들의 정보를 나타냅니다.

![monitoring_system_machine](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_machine.png)

- 머신과 인스턴스에 대한 정보

1. 머신
    - 호스트 이름: 머신의 호스트 명
    - 머신 상태: Agent와의 연결 상태
    - IP 주소: 머신의 IP 주소
    - GameAnvil Agent Port: Agent의 포트번호

2. 인스턴스
    - 인스턴스 이름
    - 인스턴스 상태
    - 총 연결 노두 개수: READY 상태인 노드 수 / 모든 노드 수
    - 유저 수: 인스턴스에 속한 GAME 노드에 접속한 사용자 수
        - GAME 노드가 동작 중(READY)일 때만 표시
        - 그 외에는 '-'으로 표시
    - 룸 수: 인스턴스에 속한 GAME 노드에 생성된 방 수
        - GAME 노드가 동작 중(READY)일 때만 표시
        - 그 외에는 '-'으로 표시

#### 인스턴스를 선택했을 떄

선택한 인스턴스에 상세한 정보를 표시합니다.

![monitoring_system_instance](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_instance.png)

- 인스턴스에 대한 정보
    - 인스턴스 이름
    - 인스턴스 상태
    - 인스턴스 설정: 인스턴스 설정 정보를 팝업창으로 확인
    - GATEWAY
        - 동작 중인 GATEWAY 노드 수 / 전체 GATEWAY 노드 수
        - 해당 노드가 없는 경우에는 '-/-'로 표기
    - GAME
        - 동작 중인 GAME 노드 수 / 전체 GAME 노드 수
        - 해당 노드가 없는 경우에는 '-/-'로 표기
    - SUPPORT
        - 동작 중인 SUPPORT 노드 수 / 전체 MASUPPORTTCH 노드 수
        - 해당 노드가 없는 경우에는 '-/-'로 표기
    - MATCH
        - 동작 중인 MATCH 노드 수 / 전체 MATCH 노드 수
        - 해당 노드가 없는 경우에는 '-/-'로 표기됩니다.
    - 유저 수: 인스턴스에 속한 GAME 노드에 접속한 사용자 수
        - GAME 노드가 동작 중(READY)일 때만 표시
        - 그 외에는 '-'으로 표시
    - 룸 수: 인스턴스에 속한 GAME 노드에 생성된 방 수
        - GAME 노드가 동작 중(READY)일 때만 표시
        - 그 외에는 '-'으로 표시

## Game > GameAnvil > 모니터링 > 인스턴스 모니터링

관리 페이지에서 등록한 인스턴스의 정보와 시작/중지/배포를 하실 수 있습니다.

![monitoring_instance_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_main.png)

1. 검색
    - 호스트 이름이나 인스턴스 이름으로 특정 인스턴스를 찾을 수 있습니다.

2. 필터
    - 인스턴스 및 배포 상태와 인스턴스 설정 타입에 따라서 특정 인스턴스를 필터할 수 있습니다.

    ![monitoring_instance_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_filter.png)

### 인스턴스 테이블

![monitoring_instance_table](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_table.png)

#### 액션<span style='color: #EB4927'>①</span>

테이블의 체크박스에 선택된 인스턴스들에 대해서 액션을 수행할 수 있습니다.

- 시작
- 중지
- 배포


#### 배포파일 업로드<span style='color: #EB4927'>②</span>

관리 > 배포파일 페이지와 동일하게 배포파일을 업로드 할 수 있습니다.

![monitoring_instance_upload](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_upload.png)


#### 테이블

인스턴스 테이블에서 각 인스턴스에 대한 상태와 시작/중지/배포에 대한 액션을 수행할 수 있습니다.

- 인스턴스 이름
- 호스트 이름
- 인스턴스 설정
- GATEWAY
    - 동작 중인 GATEWAY 노드 수 / 전체 GATEWAY 노드 수
    - 해당 노드가 없는 경우에는 '-'로 표기
- GAME
    - 동작 중인 GAME 노드 수 / 전체 GAME 노드 수
    - 해당 노드가 없는 경우에는 '-'로 표기
- SUPPORT
    - 동작 중인 SUPPORT 노드 수 / 전체 SUPPORT 노드 수
    - 해당 노드가 없는 경우에는 '-'로 표기
- MATCH
    - 동작 중인 MATCH 노드 수 / 전체 MATCH 노드 수
    - 해당 노드가 없는 경우에는 '-'로 표기
- 인스턴스 상태
    - `에러` 상태에서는 `에러`<span style='color: #EB4927'>③</span>를 클릭하면 상세 오류 메시지를 확인 할 수 있습니다.
- 배포 상태
    - `에러` 상태에서는 `에러`<span style='color: #EB4927'>③</span>를 클릭하면 상세 오류 메시지를 확인 할 수 있습니다.
- 액션
    - 하나의 인스턴스에 액션(시작/중지/배포)를 수행할 수 있습니다.

## Game > GameAnvil > 모니터링 > 노드 모니터링

### ALL

동작 중(READY)인 모든 노드들에 대해서 모니터링 할 수 있습니다.

![monitoring_node_all](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_all.png)

- 노드 ID
- 서비스 ID
    - GAME, SUPPORT 노드에 한해서만 표시됩니다.
- 서비스 이름
- 노드 타입
- 인스턴스 이름
- 호스트 이름
- 노드 상태
- 액션
    - Resume
        - Pause 상태인 노드를 시작할 수 있습니다.
    - Pause
        - Ready 상태인 노드를 정지시킵니다.
    - ※ MATCH 노드는 Resume, Pause 액션을 수행할 수 없습니다.

#### 필터

![monitoring_node_all_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_all_filter.png)

동작 중인 전체 노드들 중에 아래의 필터를 통하여 특정 노드를 필터할 수 있습니다.
- 노드 타입
- 노드 상태
- 서비스 이름

### GATEWAY

동작 중(READY)인 GATEWAY 노드들에 대해서 모니터링 할 수 있습니다.

![monitoring_node_gateway](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_gateway.png)

- 노드 ID
- 노드 타입
- 인스턴스 이름
- 호스트 이름
- 세션 수
    - 각 GATEWAY 노드에 연결된 세션 수를 표시합니다.
- 노드 상태
- 액션
    - Resume
        - Pause 상태인 노드를 시작할 수 있습니다.
    - Pause
        - Ready 상태인 노드를 정지시킵니다.

#### 필터

![monitoring_node_gateway_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_gateway_filter.png)

동작 중인 GATEWAY 노드들 중에 아래의 필터를 통하여 특정 노드를 필터할 수 있습니다.
- 노드 상태

### GAME

동작 중(READY)인 GAME 노드들에 대해서 모니터링 할 수 있습니다.

![monitoring_node_game](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_game.png)

- 노드 ID
- 노드 타입
- 서비스 ID
- 채널 ID
- 인스턴스 이름
- 호스트 이름
- 유저 수
    - 각 GAME 노드에 접속한 사용자 수를 표시합니다.
- 룸 수
    - 각 GAME 노드에 생성된 방 개수를 표시합니다.
- 노드 상태
- 액션
    - Resume
        - Pause 상태인 노드를 시작할 수 있습니다.
    - Pause
        - Ready 상태인 노드를 정지시킵니다.

#### 필터

![monitoring_node_game_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_game_filter.png)

동작 중인 GAME 노드들 중에 아래의 필터를 통하여 특정 노드를 필터할 수 있습니다.
- 노드 상태
- 서비스 이름

### SUPPORT

동작 중(READY)인 SUPPORT 노드들에 대해서 모니터링 할 수 있습니다.

![monitoring_node_support](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_support.png)

- 노드 ID
- 노드 타입
- 서비스 ID
- 서비스 이름
- 인스턴스 이름
- 호스트 이름
- 노드 상태
- 액션
    - Resume
        - Pause 상태인 노드를 시작할 수 있습니다.
    - Pause
        - Ready 상태인 노드를 정지시킵니다.

#### 필터

![monitoring_node_support_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_support_filter.png)

동작 중인 SUPPORT 노드들 중에 아래의 필터를 통하여 특정 노드를 필터할 수 있습니다.
- 노드 상태
- 서비스 이름

### MATCH

동작 중(READY)인 MATCH 노드들에 대해서 모니터링 할 수 있습니다.

![monitoring_node_match](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_match.png)

- 노드 ID
- 노드 타입
- 인스턴스 이름
- 호스트 이름
- Room Match Room Count
- User Match Queue Size
- 노드 상태

#### 필터

![monitoring_node_match_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_match_filter.png)

동작 중인 MATCH 노드들 중에 아래의 필터를 통하여 특정 노드를 필터할 수 있습니다.
- 노드 상태