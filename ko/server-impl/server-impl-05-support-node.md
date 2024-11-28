## Game > GameAnvil > 서버 개발 가이드 > 서포트 노드 구현



## Support Node

![SupportNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_supportnode_on_network.png)

SupportNode는 이름 그대로 보조적인 기능을 수행하기 위한 노드입니다. 게임 유저나 방 객체와 상관없이 임의의 기능을 구현할 수 있습니다. 또한 SupportNode는 GatewayNode를 통한 접속을 요구하지 않기 때문에 별도의 커넥션이나 세션 관리가 필요 없습니다. 이러한 특징을 바탕으로 SupportNode는 주로 독자적인 기능을 담당하곤 합니다. 예를 들어 로그를 취합해서 전송하거나 빌링 서버와 통신을 전담하는 등의 역할로 사용할 수 있습니다. 또한 엔진의 RESTful 기능을 사용하여 간단한 웹 서버 대용으로 사용하는 것도 가능합니다. 이때, SupportNode는 위의 그림과 같이 내부망뿐만 아니라 외부망(Public)에 노출시킬 수도 있으므로 GatewayNode로 접속하기 전/후에 필요한 기능을 담당할 수도 있습니다. 이렇듯 임의의 보조적인 기능을 유연하게 구현하고 배치할 수 있는 것이 SupportNode의 장점입니다.



## SupportNode 구현

이러한 SupportNode는 기본적으로 ISupportNode 인터페이스를 구현합니다. 노드 공통 콜백 메서드민 가지고 있습니다. SupportNode는 앞서 살펴본 다른 노드들과 다르게  사용자가 정의한 RESTful 처리를 할 수 있다는 것이죠. 다시 말하면, SupportNode는 사용자가 정의한 일반적인 패킷 처리와 더불어 RESTful 메시지를 처리할 수 있는 유일한 사용자 노드입니다.

```java
public class SampleSupportNode implements ISupportNode {
    private ISupportNodeContext supportNodeContext;

    /**
     * 서포트 노드 컨텍스트를 전달하기 위해 호출
     * <p/>
     * 객체가 생성된후 한번 호출된다
     *
     * @param supportNodeContext 서포트 노드 컨텍스트
     */
    @Override
    public void onCreate(ISupportNodeContext supportNodeContext) {
        this.supportNodeContext = supportNodeContext;
    }

    /**
     * 노드가 초기화 될때 호출
     */
    @Override
    public void onInit() {

    }

    /**
     * Ready 되기 전에 처리할 부분을 위해 호출
     */
    @Override
    public void onPrepare() {

    }

    /**
     * Ready 될 때 호출
     */
    @Override
    public void onReady() {

    }

    /**
     * Pause 될 때 호출
     *
     * @param payload 컨텐츠에서 전달하고자 하는 추가 정보
     */
    @Override
    public void onPause(IPayload payload) {

    }

    /**
     * Shutdown 명령을 받으면 호출
     */
    @Override
    public void onShuttingdown() {

    }

    /**
     * Resume 될 때 호출
     *
     * @param payload 컨텐츠에서 전달하고자 하는 추가 정보
     */
    @Override
    public void onResume(IPayload payload) {

    }
}
```

모든 노드는 사용자 정의 메시지를 처리하기 위한 메시지 핸들러 등록 과정이 필요합니다. SupportNode는 GameNode와 마찬가지로 사용자가 임의의 콘텐츠를 구현할 수 있는 노드 중 하나입니다.  우선, (1) GameAnvilConfig에 설정되어 있는 서포트노드 서비스 이름을 생성합나다. 서포트노드 서비스 이름은 반듸 GameAnvilConfig에 정의되어 있는 이름을 사용 해야 합니다. (2) 그리고 처리 하고 싶은  메시지를 구현해둔 [핸들러](server-impl-07-message-handling.md#11)와 연결합니다. 
예제 코드에서 사용한 핸들러는 RESTful 요청을 처리 하시 위한 핸들러 입니다. 당연히 일반 패킷을 처리하는 핸들러도 사용 가능하지만 이 예제에서는 RESTful 요청을 처리하기 위해 등록을 했습니다. 둘 사이의 사용법은 거의 동일합니다.

```java
public class Main {
    public static void main(String[] args) {
        // 게임앤빌 서버 설정 빌더
        var gameAnvilServerBuilder = GameAnvilServer.getInstance().getServerTemplateBuilder();

        // (1) "MySupport"이라는 서비스를 위한 SupportNode로 엔진에 등록
        final var gameServiceBuilder = gameAnvilServerBuilder.createSupportService("MySupport");
        gameServiceBuilder.supportNode(SampleSupportNode::new, config -> {
            // (2) SampleSupportNode에서 처리하고 싶은 RESTful 핸들러를 매핑
            config.httpHandler("/echo", RestObject.GET, new _RestEchoReq());
            config.httpHandler("/echo", RestObject.POST, new _RestEchoReq());
        });

        GameAnvilServer.getInstance().run();
    }
}
```
