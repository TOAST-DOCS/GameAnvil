## Game > GameAnvil > Unity 기초 개발 가이드 > 네트워크 연결 확인

## NetworkChecker

LTE, Wi-Fi 전환 등의 이유로 인터넷 연결이 끊길 경우 이를 인식하여 서버와의 접속을 해제할 수 있습니다.
해당 기능을 사용하려는 씬에 NetworkChecker가 존재해야 합니다.

### NetworkChecker 생성

Unity Hierarchy 창에서 마우스 오른쪽 버튼을 클릭한 뒤 **GameAnvil > NetworkChecker**를 선택해 바로 생성할 수 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/08-network-checker/01-network-checker.png)

또는 빈 GameObject를 생성하고 NetworkChecker 컴포넌트를 추가할 수도 있습니다.