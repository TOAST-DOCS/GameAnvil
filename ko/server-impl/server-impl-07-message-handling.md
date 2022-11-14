## Game > GameAnvil > 서버 개발 가이드 > 메시지 핸들링

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/three_steps_for_message_process.png)



## 1. 일반 메시지 처리

GameAnvil의 메시지 처리는 위의 그림과 같이 크게 세 부분으로 나뉩니다. 이 세 부분이 서로 맞물려 메시지 처리 흐름을 만들게 됩니다.



### 1.1. 패킷 디스패처 생성 및 메시지와 핸들러 연결

우선 메시지 처리를 하기 위한 패킷 디스패처를 생성합니다. 이 디스패처는 해당 클래스에 대해 사용되므로 불필요한 리소스 낭비를 막기 위해 반드시 static으로 생성합니다.

```java
// (패킷 디스패처 생성    
private static PacketDispatcher packetDispatcher = new PacketDispatcher();   
```

이렇게 생성한 디스패처에 원하는 메시지와 핸들러를 연결합니다.

```java
// 처리하고 싶은 메시지와 핸들러를 매핑
static {
    packetDispatcher.registerMsg(MyProto.MyMsg.getDescriptor(), _MyMsgHandler.class);
}
```



### 1.2. 메시지 핸들러 구현

이제 해당 메시지 핸들러를 직접 구현합니다. 이 때, GameAnvil은 엔진 내부는 물론이고 모든 샘플 코드에서 메시지 핸들러에 대해 _로 시작하는 네이밍을 사용합니다. 앞으로 등장하는 모든 예제 코드에서도 _로 시작하는 클래스는 모두 메시지 핸들러입니다.  가장 기본적인 형태의 메시지 핸들러는 아래와 같습니다. RECEIVER_CLASS는 해당 메시지를 받는 클래스를 의미합니다.

```java
public class _MyMsgHandler implements GameAnvilPacketHandler<RECEIVER_CLASS> {
    private static final Logger logger = getLogger(_MyMsgHandler.class);

    @Override
    public final void execute(RECEIVER_CLASS receiver, GameAnvilPacket gameAnvilPacket) throws SuspendExecution {

		...
    }
}
```

예를 들어, 패킷 수신자가 GameUser라면 그 메시지 핸들러는 아래와 같습니다.

```java
public class _MyGameMsg implements GameAnvilPacketHandler<GameUser> {
    private static final Logger logger = getLogger(_MyGameMsg.class);

	@Override
    public final void execute(final GameUser gameUser, final GameAnvilPacket gameAnvilPacket) throws SuspendExecution {
        ...
    }
}
```



### 1.3. onDispatch 콜백 구현

패킷 디스패처와 메시지 핸들러를 모두 구현하였습니다. 남은건 이제 엔진에서 처리할 패킷이 있을 때 해당 패킷 디스패처로 넘겨주기면 하면 됩니다. 사용자가 상속 구현하는 엔진의 객체 중 패킷을 처리할 수 있는 모든 것들은 onDispatch 콜백 메서드를 제공합니다. 이 콜백 메서드는 해당 객체에서 처리할 메시지가 있을 때 엔진에 의해 호출됩니다. 인자로 넘어온 처리할 패킷을 앞 서 생성한 패킷 디스패처로 넘기면 됩니다. 이 때, 해당 패킷에 대한 추가 처리를 하고 싶다면 그 역시 이 콜백에 구현하면 됩니다.

```java
/**
* 패킷이 전달될 때 호출
*
* @param packet 처리할 패킷
* @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
*/
@Override
public void onDispatch(Packet packet) throws SuspendExecution {
// 메시지 처리. 필요할 경우 패킷에 대한 추가 처리도 여기에서 할 수도 있습니다.
    if (packetDispatcher.isRegisteredMessage(packet))
	    packetDispatcher.dispatch(this, packet);
}
```

지금까지 일반적인 패킷 처리에 대한 흐름을 살펴보았습니다. 그와 더불어 RESTful 요청에 대한 처리를 바로 달아서 살펴보도록 하겠습니다.



## 2. RESTful 요청 처리

패킷 디스패처는 두 가지가 존재합니다. 하나는 앞서 살펴본 일반적인 패킷 디스패처이고 다른 하나는 RESTful 요청을 처리하기 위한 디스패처입니다. 전체적인 사용법은 두 가지가 거의 동일합니다.  단, 이러한 RESTful 메시지 처리는 오직 SupportNode만 지원합니다. 



### 2.1. REST 패킷 디스패처 생성 및 메시지와 핸들러 연결

다음의 예제 코드는 RESTful 요청을 처리하기 위해 이전과 다른 RestPacketDispatcher를 생성하고 있습니다. 또한 메시지 클래스가 아닌 URL 형태의 API를 핸들러에 연결하고 있습니다.

```java
private static RestPacketDispatcher restPacketDispatcher = new RestPacketDispatcher<>();

static {
    // path 와 method(GET, POST, ...) 조합으로 등록.
    restPacketDispatcher.registerMsg("/auth", RestObject.GET, _RestAuthReq.class);
    restPacketDispatcher.registerMsg("/echo", RestObject.GET, _RestEchoReq.class);
    restPacketDispatcher.registerMsg("/testGET", RestObject.GET, _RestTestGET.class);
    restPacketDispatcher.registerMsg("/testPOST", RestObject.POST, _RestTestPOST.class);
}
```

예제 코드와 같이 RestPacketDispatcher를 사용하여 사용자가 원하는 RESTful API와 메시지 핸들러를 연결하고 있습니다.



### 2.2 핸들러 구현

RESTful 메시지 핸들러는 Packet 대신 RestObject가 인자로 전달되는 차이점 외에는 동일한 모습입니다. 

```java
public class _RestAuthReq implements RestPacketHandler<WebSupportNode> {
    private static final Logger logger = getLogger(_RestAuthReq.class);

    @Override
    public void execute(WebSupportNode node, RestObject restObject) throws SuspendExecution {
		...
    }  
}
```



### 2.3. onDispatch 콜백 구현

RESTful 메시지 처리를 위한 onDispatch 콜백은 SupportNode만 지원합니다. 아래의 예제 코드와 같이 앞서 생성한 REST 패킷 디스패처리를 이용하여 전달된 RestObject를 처리할 수 있습니다. 해당 RestObject에 대한 추가 처리를 하고자 할 경우에도 이 콜백 메서드에 구현할 수 있습니다.

```java
/**
* 처리할 RESTful 요청이 있으면 호출
*
* @param restObject 전달된 RESTful 요청
* @return 메시지를 직접 처리하였으면 true를 반환하고 그렇지 않으면 false를 반환
* @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
*/
@Override
public boolean onDispatch(RestObject restObject) throws SuspendExecution {
	// REST 요청 처리        
    if (!restMsgHandler.isRegisteredMessage(restObject))
	    return false;

    restPacketDispatcher.dispatch(this, restObject);
    return true;
}
```

