## Game > GameAnvil > 콘솔 사용 가이드 > 서비스 관리

## 1. 서비스

GameAnvil의 서비스 관리 기능을 설명하기에 앞서 서비스가 무엇인지부터 살펴 보겠습니다. 하나의 서비스는 고유한 서비스ID와 서비스명을 가집니다. 그리고 GameNode와 SupportNode는 구현하기에 따라 여러 종류의 서비스로 등록이 가능합니다. 즉, 서비스는 GameNode 혹은 SupportNode의 역할을 정의한다고 볼 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/service/list.png)

예를 들어 위의 이미지에서 서비스ID 1인 "MyGame" 서비스는 해당 서버에 구현된 게임 컨텐츠에 대한 서비스를 의미합니다. 즉, 게임 컨텐츠를 제공하는 역할을 의미한다고 볼 수 있습니다. 마찬가지로 서비스ID 2인 "MyChat" 서비스는 해당 서버에 구현된 채팅 컨텐츠에 대한 서비스를 의미합니다. 

다시 말하면, 여러분의 게임 서버에는 두 가지의 GameNode가 다음의 의사 코드처럼 그 역할에 맞게 따로 구현되어 있다는 것을 의미합니다. 이 코드에서 임의의 서비스명을 해당 GameNode 구현에 연결하기 위해 사용한 어노테이션을 주의 깊게 살펴 보십시오.

```java
// 게임 컨텐츠를 구현한 GameNode
@ServiceName("MyGame")
MyGameNode extends gameanvil.GameNode {
    ...
}

// 채팅 컨텐츠를 구현한 GameNode
@ServiceName("MyChat")
MyChatNode extends gameanvil.GameNode {
    ...
}
```

눈치 채셨겠지만, @ServiceName 어노테이션에 등록한 서비스명이 바로 서버를 구성할 때 사용한 서비스명과 일치해야 합니다. 그렇지 않으면 서버 구성에 실패할 것입니다. 이러한 서비스명은 사람이 읽기에는 편하지만 서버 프로그램에 있어서는 불필요하게 길고 큰 값입니다. 그래서 우리는 서버 구성 단계에서 각 서비스명에 고유한 정수인 서비스ID를 설정해 주었습니다. 서비스ID는 반드시 0보다 큰 정수값이어야 합니다.


## 2. 채널

채널은 하나의 서비스를 논리적으로 나눌 수 있는 방법을 제공합니다. 단, GameNode에 한해서만 사용 가능하며 채널명은 어떤 문자열이든 사용 가능합니다. 더욱 상세한 설명은 [Game > GameAnvil > 서버 개발 가이드 > 채널](server-09-channel.md)의 내용을 참고 하십시오.


## 3. 서비스 관리

서비스 탭은 서버 구성 단계에서 설정한 서비스와 채널 정보를 보여줍니다. 테이블에서 하나의 행을 클릭하면 아래와 같이 해당 서비스에 대한 상세 정보를 확인할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/service/detail.png)

이 때, 서비스/채널 상세 정보 페이지가 제공하는 "수정"이나 "삭제" 버튼을 사용하여 해당 서비스와 채널 정보를 수정하거나 삭제할 수 있습니다.


또한 "등록" 버튼을 눌러 새로운 서비스와 채널 정보를 등록할 수도 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/service/new.png)

