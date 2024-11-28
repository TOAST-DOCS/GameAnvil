## Game > GameAnvil > 서버 개발 가이드 > 비동기 지원

## 비동기 지원

GameAnvil은 다음과 같은 목적을 위해 비동기 처리를 지원합니다.

![async-goal.png](https://static.toastoven.net/prod_gameanvil/images/async-goal.png)

GameAnvil의 API도 다른 많은 비동기 방식 API들처럼 Future를 리턴하는 방식으로 동작 합니다. 더욱 자유롭게 코드 흐름을 만들 수 있습니다. GameAnvil에서 Virtual Thread를 공식 지원하기 때문에, 블로킹 코드를 호출하더라도 Java 21을 지원하는 코드라면 Platform Thread 가 아닌 Virtual Thread가 블록이 됩니다. 

```java
Future<Packet> packetFuture = user.requestToNode(nodeId, myRequest1);
Future<Response> httpFuture = getHttpClient().executeRequest(myRequest2);
Packet packetResponse = packetFuture.get();  
Response httpResponse = httpFuture.get();  // Java 21 에서는 Virtual Thread 만 블락
```

> [참고]
> 
> Platform Thread : 기존 Java에서 사용한 Thread. OS 스레드 혹은 자바 스레드입니다.
> 
> Virtual Thread: Java 21에서 추가된 새로운 Thread입니다 이전 버전 GameAnvil의 Fiber 와 유사한 동작을 합니다 자세한 동작은 [여기](https://openjdk.org/jeps/444)를 참고하십시오.

