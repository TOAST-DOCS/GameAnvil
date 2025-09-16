## Game > GameAnvil > 서버 개발 가이드 > 메시지 핸들링

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/07-message-handling/three_steps_for_message_process.png)

## 일반 메시지 처리

* GameAnvil의 메시지 처리는 위의 그림과 같이 크게 세 부분으로 나뉩니다. 이 세 부분이 서로 맞물려 메시지 처리 흐름을 만들게 됩니다.
* 이 중 onDispatch 콜백 구현은 엔진에서 자체적으로 하고 있으며 엔진 사용자는 Dispatcher의 선언과 메시지 핸들러를 구현하여 패킷을 처리합니다.

### 메시지 핸들러 구현 및 메시지와 핸들러 연결

이제 해당 메시지 핸들러를 직접 구현합니다. 이때, GameAnvil은 엔진 내부는 물론이고 모든 샘플 코드에서 메시지 핸들러에 대해 _로 시작하는 네이밍을 사용합니다. 앞으로 등장하는 모든 예제 코드에서도 _로 시작하는 클래스는 모두 메시지 핸들러입니다.  가장 기본적인 형태의 메시지 핸들러는 아래와 같습니다. CONTEXT_CLASS는 해당 메시지의 흐름을 나타내는 클래스를 의미하고 MAPPING_CLASS는 처리할 메시지를 등록하는 클래스를 의미합니다. Context 클래스에서 대상 객체 획득, 메시지의 응답 등을 할 수 있습니다.

```java
@GameAnvilController // 이 클래스를 메시지 처리자로 사용합니다.
public class _MyEchoHandler {

    @MAPPING_CLASS // 이 메서드가 어떤 메시지를 처리할지 선언합니다.
    public void execute(CONTEXT_CLASS ctx, EchoSend request) {
        System.out.println("Receive EchoSend Message!");
    }
}
```

예를 들어, 패킷 수신자가 GameUser라면 그 메시지 핸들러는 아래와 같습니다.

```java
@GameAnvilController // 이 클래스를 메시지 처리자로 사용합니다.
public class _MyEchoHandler {

    @GameUserMapping(
        value = EchoSend.class,             // 처리할 프로토 버퍼
        loadClass = SampleGameUser.class    // 메시지를 받는 대상 
    )
    public void execute(IUserDispatchContext ctx, EchoSend request) {
        System.out.println("Receive EchoSend Message!");
    }
}
```

## http 요청 처리

패킷 디스패처는 두 가지가 존재합니다. 하나는 앞서 살펴본 일반적인 패킷 디스패처이고 다른 하나는 http 요청을 처리하기 위한 디스패처입니다. 전체적인 사용법은 두 가지가 거의 동일합니다.  단, 이러한 http 메시지 처리는 오직 SupportNode만 지원합니다. 

### http 패킷 디스패처 생성 및 메시지와 핸들러 연결

다음의 예제 코드는 RESTful 요청을 처리하기 위해 이전과 다른 @SupportNodeHttpMapping을 선언합니다. 또한 메시지 클래스가 아닌 URL 형태의 API를 핸들러에 연결하고 있습니다. http 메시지 핸들러는 RestObject가 인자로 전달되는 차이점이 있습니다.

```java 
@GameAnvilController
public class _RestAuthReq {

    @SupportNodeHttpMapping(path = "/hello", method = "GET", loadClass = WebSupportNode.class)
    public void execute(WebSupportNode node, RestObject restObject) {
		// ...
    }  
}
```
