## Game > GameAnvil > Unity 기초 개발 가이드 > 네트워크 연결 확인

## NetworkChecker

LTE, Wifi 전환 등의 이유로 인터넷 연결이 끊겼을 경우 이를 인식하여 서버와의 접속을 해제해주는 역할을 합니다.
해당 기능을 사용하려는 씬에 NetworkChecker가 존재해야 합니다.

### NetworkChecker 생성

GameObject를 생성하고 NetworkChecker 컴포넌트를 추가합니다.
Add Component > GameAnvil > NetworkChecker로 선택해서 컴포넌트로 추가할 수 있습니다.

더욱 간편한 방법으로는, Unity Hierarchy에서 우클릭 후 GameAnvil > NetworkChecker를 선택해서 바로 생성할 수 있습니다.
