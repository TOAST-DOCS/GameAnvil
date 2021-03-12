## Game > GameAnvil > 서버 개발 가이드 > 무정지 패치 방법

이 문서는 무정지 패치를 진행하는 방법을 설명합니다. 그와 더불어 무정지 패치 과정에서 발생할 수 있는 이슈에 관해 정리합니다.

### Note

- 무정지 점검을 사용하기 전에 반드시 알파에서 먼저 진행하여 문제가 없는지 확인하세요.
- 이 가이드 문서를 반드시 읽어보고 모든 내용을 이해한 후에 사용하세요.



## 1. NonStopPatch를 사용하면 안 되는 상황

- 패치할 서버 파일과 이미 배포된 서버 파일의 GameAnvil 버전이 다를 경우
- 패킷이 수정되거나 추가되었을 경우
- Transfer 간에 전달되는 Data Class가 변경되었을 경우



## 2. NonStopPatch를 사용하기 위해 해야 하는 작업

### 2-1. Match 기능을 사용 중이라면

- 구버전 서버와 신버전 서버의 버전 정보를 확인하는 로직을 넣어야 합니다.

### 2-2. TransferData

- 구버전 서버와 신버전 서버의 transferData가 다를 경우, 신버전의 GameUser/GameRoom의 onTranferIn 함수에서 삭제되었거나, 추가되는 transferData에 대한 처리를 해야 합니다.
- proto 객체로 데이터를 전달할 경우 kryo에 해당 class를 등록해야 합니다.



## 3. NonStopPatch 사용 시 알아야 하는 것들

- NonStopPatch Report 정보 제공
  - Rest API
    - http://127.0.0.1:25150/managementEndPoint/non-stop-patch/report-page
    - NonStopPatch 진행 사항 및 결과 제공
  - File
    - log/nonStopPatch/report-2020-08-10 205510.log
    - NonStopPatch 종료 시 Report를 Json 형식으로 File 저장
- NonStopPatch 강제 종료 기능
  - NonStopPatch가 진행 중일 경우 다른 Node의 상태를 변경할 수 없습니다.
  - NonStopPatch 진행 중 문제가 발생하여, 종료가 되지 않을 수 있습니다.
  - NonStopPatch Report와 로그를 통해 문제점을 파악하고, 강제로 NonStopPatch를 종료할 것인지 결정합니다.
  -
    - NonStopPatch 강제 종료 버튼을 누르면 NonStopPatch가 강제로 종료됩니다.
  - Pause된 Node를 처리합니다.

## 4. NonStopPatch 시작

- NonStopPatch 진행은 Admin으로 합니다.

### 4-1. NonStopPatch를 진행할 Node 선택

- 출발지 선택
- ![NonStopPatch-2.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-2.png)
- 도착지 선택
- ![NonStopPatch-3.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-3.png)

### 4-2. NonStopPatch Report Page를 열어두기

- http://10.162.3.241:25150/non-stop-patch/report-page?refreshTime=1000

### 4-3. NonStopPatch 버튼 클릭

- NonStopPatch 버튼을 클릭해서 NonStopPatch 진행
- ![NonStopPatch-4.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-4.png)
- ![NonStopPatch-5.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-5.png)
- 출발지 Node는 NonStopPatch Src 상태로 변함
  - NonStopPatch가 종료되면 Pause 상태가 됨
- 도착지 Node는 NonStopPatch Dst 상태로 변함
  - NonStopPatch가 종료되면 Ready 상태가 됨

### 4-4. NonStopPatch 결과 확인

- NonStopPatch Report 확인
- ![NonStopPatch-6.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-6.png)
- 현재 GameNode 상태 확인
- ![NonStopPatch-7.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-7.png)

### 4-5. 서버 종료

- Pause 중인 서버 종료
- ![NonStopPatch-8.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-8.png)
- 종료된 서버 확인
- ![NonStopPatch-9.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-9.png)

### 4-6. 서버 패치

- 배포된 파일을 지우고 패치할 서버 파일을 배포
  - 직접 서버 파일 이름을 지정할 경우 지우지 않아도 됨
- ![NonStopPatch-10.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-10.png)

### 4-7. 서버 다시 시작

- 서버 다시 시작
- ![NonStopPatch-11.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-11.png)

### 4-8. 반대쪽 Node도 NonStopPatch 진행

- 위와 동일하게 반대편 Node도 진행

### 4-9. 서버 종료

- 위와 동일하게 반대편 Node도 진행

### 4-10. 서버 패치

- 위와 동일하게 반대편 Node도 진행

### 4-11. 서버 다시 시작

- 위와 동일하게 반대편 Node도 진행

### 4-12. NonStopPatch가 잘되었는 지 확인

- 서버 패치 이후 이슈가 없는지 확인

## 5. NonStopPatch 시 함수 호출 순서

### 5-1. BaseGameRoom

- 생성 시 호출되는 Callback 함수
  - 경우에 맞춰 Callback 함수 내용을 채웁니다.
  - GameRoom 생성
    - onInit -> onCreate
  - NonStopPatch 시 GameRoom 생성
    - onInit -> onTransferIn
- NonStopPatch 종료 후 호출되는 Callback 함수
  - NonStopPatch 종료 후 GameNode의 State가 변경되면서 GameRoom에서 Callback 함수가 호출됩니다.
  - 출발지 Node
    - onPause
    - 출발지 Node는 NonStopPatch가 종료되고 pause 상태가 되기 때문에,
  - 도착지 Node
    - X

### 5-2. BaseUserRoom

- 생성 시 호출되는 Callback 함수
  - 경우에 맞춰 Callback 함수 내용을 채워야 합니다.
  - GameUser 생성
    - onLogin
  - NonStopPatch 시 GameUser 생성
    - onTransferIn
- NonStopPatch 종료 후 호출되는 Callback 함수
  - NonStopPatch 종료 후 GameNode의 State가 변경되면서 GameUser에서 Callback 함수가 호출됩니다.
  - 출발지 Node
    - onPause
    - 출발지 Node는 NonStopPatch가 종료되고 pause 상태가 되기 때문에,
  - 도착지 Node
    - X

### 5-3. BaseGameNode

- NonStopPatch 시 호출되는 Callback 함수
- 출발지 Node일 경우

```
// NonStopPatch가 시작될 경우 호출
@Override
public boolean onNonStopPatchSrcStart() throws SuspendExecution {
    return true;
}
// NonStopPatch가 종료될 경우 호출
@Override
public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}
// NonStopPatch 종료 체크 함수
@Override
public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}
```

- 도착지 Node일 경우

```
// NonStopPatch가 시작될 경우 호출
@Override
public boolean onNonStopPatchDstStart() throws SuspendExecution {
    return true;
}
// NonStopPatch가 종료될 경우 호출
@Override
public boolean onNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
// NonStopPatch 종료 체크 함수
@Override
public boolean canNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
```
