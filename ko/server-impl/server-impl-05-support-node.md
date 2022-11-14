## Game > GameAnvil > 서버 개발 가이드 > 서포트 노드 구현



## 1. Support Node 란?

![SupportNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_supportnode_on_network.png)

SupportNode는 이름 그대로 보조적인 기능을 수행하기 위한 노드입니다. 게임 유저나 방 객체와 상관없이 임의의 기능을 구현할 수 있습니다. 또한 SupportNode는 GatewayNode를 통한 접속을 요구하지 않기 때문에 별도의 커넥션이나 세션 관리가 필요 없습니다. 이러한 특징을 바탕으로 SupportNode는 주로 독자적인 기능을 담당하곤 합니다. 예를 들어 로그를 취합해서 전송하거나 빌링 서버와 통신을 전담하는 등의 역할로 사용할 수 있습니다. 또한 엔진의 RESTful 기능을 사용하여 간단한 웹 서버 대용으로 사용하는 것도 가능합니다. 이 때, SupportNode는 위의 그림과 같이 내부망 뿐만 아니라 외부망(Public)에 노출시킬 수도 있으므로 GatewayNode로 접속하기 전/후에 필요한 기능을 담당할 수도 있습니다. 이렇듯 임의의 보조적인 기능을 유연하게 구현하고 배치할 수 있는 것이 SupportNode의 장점입니다.



## 2. SupportNode 구현

이러한 SupportNode는 기본적으로 BaseSupportNode 추상 클래스를 상속 구현합니다. 노드 공통 콜백 메서드를 제외하면 추가로 RESTful 처리를 위한 onDispatch() 콜백이 유일합니다. 즉, SupportNode는 앞서 살펴본 다른 노드들과 바로 이 부분에서 차별화됩니다. 사용자가 정의한 RESTful 처리를 할 수 있다는 것이죠. 다시 말하면, SupportNode는 사용자가 정의한 일반적인 패킷 처리와 더불어 RESTful 메시지를 처리할 수 있는 유일한 사용자 노드입니다.

모든 노드는 사용자 정의 메시지를 처리하기 위한 디스패처 생성과 메시지 핸들러 등록 과정이 필요합니다. SupportNode는 GameNode와 마찬가지로 사용자가 임의의 컨텐츠를 구현할 수 있는 노드 중 하나입니다.  우선, (1) 정적 패킷 디스패처를 하나 생성합니다. 이 때, 반드시 메모리나 성능 측면에서 이점을 취할 수 있도록 정적(static)으로 생성합니다. (2) 그리고 처리하고 싶은 메시지를 구현해둔 [핸들러](server-07-message-handling#11)와 연결합니다. (3) 마지막으로 onDispatch 콜백에서 (1)에서 생성한 디스패처를 이용하여 패킷을 처리합니다. 이 때, 디스패칭 뿐만 아니라 패킷에 대한 추가적인 처리를 할 수도 있습니다. 이 때, 예제 코드에서 사용한 패킷 디스패처는 RestPacketDispatcher임에 주의합니다. 당연히 일반 패킷 디스패처를 사용하는 것도 가능하지만 이 예제에서는 RESTful 요청을 처리하기 위해 RestPacketDispatcher를 선택했습니다. 둘 사이의 사용법은 거의 동일합니다.

이러한 일련의 과정을 아래의 예제 코드에서 (1)~(3)에 해당하는 주석 바로 아래의 코드를 통해 살펴볼 수 있습니다.

```java
public class SampleSupportNode extends BaseSupportNode {

    // 1.REST 디스패처 생성
    private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher<>();

    // 2.SampleSupportNode에서 처리하고 싶은 URL과 핸들러를 매핑
    static {
        // path 와 method(GET, POST, ...) 조합으로 등록.
        restMsgHandler.registerMsg("/auth", RestObject.GET, _RestAuthReq.class);
        restMsgHandler.registerMsg("/echo", RestObject.GET, _RestEchoReq.class);
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onPrepare() throws SuspendExecution {
    }

    @Override
    public void onReady() throws SuspendExecution {
    }

    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {
    }

    /**
     * 처리할 RESTful 요청이 있으면 호출
     *
     * @param restObject 전달된 RESTful 요청
     * @return 메시지를 직접 처리하였으면 true를 반환하고 그렇지 않으면 false를 반환
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onDispatch(RestObject restObject) throws SuspendExecution {
        // 3.SampleSupportNode의 REST 요청 처리    
        if (!restMsgHandler.isRegisteredMessage(restObject))
            return false;

        restMsgHandler.dispatch(this, restObject);
        return true;
    }

    @Override
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {}

    @Override
    public void onResume(Payload outPayload) throws SuspendExecution {}

    @Override
    public void onShutdown() throws SuspendExecution {}
}
```

SupportNode 고유의 콜백은 다음과 같습니다.


| 콜백 이름  | 의미                       | 설명                                                                                                                                                                                                             |
| ------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| onDispatch | 처리할 RESTful 요청이 있음 | 노드에 처리할 RESTful 요청이 도달하면 호출됩니다. 사용자는 자신이 선언한 디스패처를 사용하거나 패킷에 대한 추가 작업을 진행할 수 있습니다. 자세한 내용은 [메시지 처리](server-07-message-handling#13-ondispatch)를 참고하세요. |
