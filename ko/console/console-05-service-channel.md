## Game > GameAnvil > 콘솔 사용 가이드 > 서비스와 채널

## 서비스

이 문서에서는 앞서 구성 정보(Config) 등록을 다루면서 언급한 '서비스'에 대해 설명합니다.

하나의 서비스는 고유한 서비스 ID와 서비스명을 가지며, GameNode와 SupportNode는 구현하기에 따라 여러 종류의 서비스로 등록할 수 있습니다. 즉 서비스는 GameNode 또는 SupportNode의 역할을 정의한다고 할 수 있습니다.

예를 들어 'RPSGame' 서비스는 해당 서버에 구현된 'RPS'라는 게임 콘텐츠에 대한 서비스 용도입니다. 즉 게임 콘텐츠를 제공하는 역할을 의미합니다. 마찬가지로 'RPSChat' 서비스를 추가하여 해당 서버에 구현된 채팅 콘텐츠에 대한 서비스를 정의할 수도 있습니다. 

이는 게임 서버에 두 개의 GameNode가 다음의 의사 코드처럼 그 역할에 맞게 따로 구현되어 있음을 의미합니다. 이 코드에서 임의의 서비스명을 해당 GameNode 구현에 연결하기 위해 사용한 어노테이션을 주의 깊게 살펴보십시오.

```java
// 게임 콘텐츠를 구현한 GameNode
@ServiceName("RpsGame")
MyGameNode extends gameanvil.GameNode {
    ...
}

// 채팅 콘텐츠를 구현한 GameNode
@ServiceName("RpsChat")
MyChatNode extends gameanvil.GameNode {
    ...
}
```

@ServiceName 어노테이션에 등록한 서비스명은 서버를 구성할 때 사용한 서비스명과 일치해야 합니다. 이러한 서비스명은 사람이 읽기에는 편하지만 서버 프로그램이 읽기에는 불필요하게 길고 큰 값이므로 구성 정보(Config)에서 각 서비스명에 고유한 정수인 서비스 ID를 매핑했습니다. 서비스 ID는 반드시 0보다 큰 정수값이어야 합니다. 자세한 정보는 서버 구현 문서에서 확인할 수 있습니다.


## 채널

채널은 하나의 서비스를 논리적으로 나눌 수 있는 방법을 제공합니다. 예를 들어 RPSGame 서비스를 '초보', '중수', '고수' 채널 등으로 나눌 수 있습니다. 단 GameNode에 한하여 사용 가능하며 채널명은 어떤 문자열이든 사용할 수 있습니다. 자세한 내용은 [Game > GameAnvil > 서버 개발 가이드 > 채널](../server-impl/server-impl-09-channel.md)을 참고하십시오.

