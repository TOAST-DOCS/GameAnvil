## Game > GameAnvil > 서버 개발 가이드 > 시작하기

GameAnvil을 이용해 게임 서버를 개발하는 방법을 자세하게 설명합니다.

## 시스템 요구 사항

| 개발 언어  | 개발 IDE      | 지원 OS               | 프로젝트 관리 | 사용 가능한 네트워크 프로토콜 | 보안 |
| ---------- | ------------- | --------------------- | ------------- | ----------------------------- | ---- |
| Java 8, 11 | IntelliJ IDEA | Linux, Windows, MacOS | Maven         | TCP/IP, WebSocket, HTTP/HTTPS | SSL  |



## 1분 만에 채팅 서버 만들기

GameAnvil은 기본적인 골격 작업을 빠르게 완료하기 위해 자체 템플릿을 제공합니다. 템플릿을 사용하면 몇 번의 클릭만으로 기본적인  게임 서버가 완성됩니다. 이렇게 생성된 서버는 간단한 채팅 기능을 포함하고 있습니다. 자세한 사항은 아래의 서버 템플릿 사용법을 참고하세요.

- [템플릿 다운로드](http://static.toastoven.net/prod_gameanvil/files/GameAnvil-Template.zip)

- 따라 하기

- IntelliJ를 열어서 **File > Settings**를 선택합니다.

   ![GameAnvilTemplate-1](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-1.png)

- **Import Settings** 대화 상자에서 앞서 다운로드한 **GameAnvil-Template.zip** 파일을 선택해 가져옵니다.
  ![GameAnvilTemplate-2](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-2.png)

- 가져오기가 완료된 후 IntelliJ를 다시 시작합니다.

- 다시 시작되면 다음과 같은 대화 상자가 나타납니다. **Create New Project**를 선택합니다.    ![GameAnvilTemplate-3](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-3.png)

- 이제 앞서 등록한 템플릿을 선택할 수 있습니다. **Next** 버튼을 클릭하면 적용됩니다.    ![GameAnvilTemplate-4](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-4.png)

- 적용 과정에서 아래와 같이 **Enable Auto-Import**을 묻는 창이 나타나면 **Enable**을 선택합니다.    ![GameAnvilTemplate-5](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-5.png)

- 모두 완료되면 서버 템플릿 프로젝트의 소스 코드가 나타납니다. **Run**을 실행하면 아래와 같이 채팅 서버가 구동됩니다.    ![GameAnvilTemplate-6](http://static.toastoven.net/prod_gameanvil/images/GameAnvilTemplate-6.png)



## 서버 구동

템플릿을 이용한 기본적인 서버 구성이 완료되었으면 이제 실행해 볼 수 있습니다. 서버가 무사히 구동되면 아래와 같이 각 노드별로 **onReady** 로그가 출력됩니다. 이제 준비한 서버로 클라이언트를 이용해 접속해 봅니다.

```
[2020-03-13 17:36:16,925] [INFO ] [MANAGEMENT@1005@ThinkServer@1] [c.n.t.m.ManagementNode] onReady
[2020-03-13 17:36:20,008] [INFO ] [LOCATION@30122@ThinkServer@2] [c.n.t.l.LocationNode] onReady
[2020-03-13 17:36:18,316] [INFO ] [SPACE@1@ThinkServer@0] [c.n.t.s.i.SpaceNode] onReady
[2020-03-13 17:36:19,008] [INFO ] [SESSION@1001@ThinkServer@2] [c.n.t.s.i.SessionNode] onReady
```



## 레퍼런스

GameAnvil은 템플릿뿐 아니라 레퍼런스 프로젝트를 제공합니다. GameAnvil 사용자가 참고할 수 있도록 템플릿으로 구성한 기본 골격에 다양한 기능을 구현해 두었습니다. 이러한 서버와 클라이언트의 레퍼런스 프로젝트는 별도의 메뉴에서 확인할 수 있습니다. 더불어 GameAnvil API와 UML 다이어그램을 JavaDoc 문서로 제공하고 있으니 함께 참고하시기 바랍니다.

[GameAnvil JavaDoc API Reference Site](http://10.162.4.61:9090/gameanvil)(JavaDoc 사이트를 NHN Cloud 내에 포함시킬 경우 해당 도메인으로 교체할 예정)
