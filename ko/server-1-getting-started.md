## Game > GameAnvil > 서버 개발 가이드 > 시작하기
GameAnvil을 이용하여 게임 서버를 개발하는 방법에 대해서 쉽고 자세하게 설명합니다.

## 시스템 요구사항

| 개발 언어 | 개발 IDE | 지원 O/S | 프로젝트 관리 | 사용 가능한 네트워크 프로토콜 | 보안 |
| ---- | ---- | ---- | ---- | ---- | ---- |
| Java 8 | IntelliJ IDEA | Linux, Windows, MacOS | Maven | TCP/IP, WebSocket, HTTP/HTTPS  | SSL |

<br>

## 1분 만에 채팅 서버 만들기

GameAnvil은 기본적인 골격 작업을 빠르게 완료하기 위해 자체 템플릿을 제공합니다. 템플릿을 사용하면 몇 번의 클릭만으로 기본적인  게임 서버가 완성됩니다. 이렇게 생성된 서버는 간단한 채팅 기능을 포함하고 있습니다. 자세한 사항은 아래의 서버 템플릿 사용법을 참고하세요.

[서버 템플릿 사용법](server-link-gameanvil-template)

<br>

## 서버 구동

템플릿을 이용한 기본적인 서버 구성이 완료되었으면 이제 실행해볼 수 있습니다. 서버가 무사히 구동되면 아래와 같이 각 노드별로 **onReady** 로그가 출력됩니다. 이제 준비한 서버로 클라이언트를 이용해서 접속해 볼 수도 있습니다. 이 부분에 대해서는 [클라이언트 쪽 설명](client-1-getting-started)을 참고해주세요.

```
[2020-03-13 17:36:16,925] [INFO ] [MANAGEMENT@1005@ThinkServer@1] [c.n.t.m.ManagementNode] onReady
[2020-03-13 17:36:20,008] [INFO ] [LOCATION@30122@ThinkServer@2] [c.n.t.l.LocationNode] onReady
[2020-03-13 17:36:18,316] [INFO ] [SPACE@1@ThinkServer@0] [c.n.t.s.i.SpaceNode] onReady
[2020-03-13 17:36:19,008] [INFO ] [SESSION@1001@ThinkServer@2] [c.n.t.s.i.SessionNode] onReady
```

<br>

## 레퍼런스

GameAnvil은 템플릿 뿐만 아니라 레퍼런스 프로젝트를 제공하고 있습니다. GameAnvil 사용자가 참고할 수 있도록 템플릿으로 구성한 기본 골격에 다양한 기능이 구현해 두었습니다. 서버와 클라이언트에 대한 레퍼런스 프로젝트는 아래의 링크를 통해 확인할 수 있습니다. 그와 더불어 GameAnvil API와 UML 다이어그램을 JavaDoc 문서로 제공하고 있으니 함께 참고하시면 도움이 될 것입니다.

| 서버 | 클라이언트 | GameAnvil API Reference |
| --- | ---- | --- |
| [참고](server-link-reference-server) | [참고](server-link-reference-client) | [GameAnvil API Reference](http://10.162.4.61:9090/gameanvil) |

