## Game > GameAnvil > 서버 개발 가이드 > 서버 구성의 기본 요소와 구현

기본적인 서버를 구현하기 위해 알아야할 기본 요소들과 이를 구현하는 방법에 대해 설명합니다. 엔진의 핵심적인 부분을 중심으로 가능한 이해하기 쉽게 가이드 합니다. 코드 레벨의 참고 자료가 필요한 경우에는 [GameAnvil 레퍼런스 서버](server-link-reference-server)를 참고할 수 있습니다. 또한 [GameAnvil API Reference](http://10.162.4.61:9090/gameanvil)를 통해 JavaDoc 문서도 이용 가능합니다. 가능하다면 레퍼런스 서버 프로젝트와 이 가이드 문서를 함께 살펴보기를 추천합니다.

<br>

### Note

*GameAnvil은 엔진 내부는 물론 모든 샘플 코드에서 메시지 핸들러에 대해 _로 시작하는 네이밍을 사용합니다. 즉, 앞으로 등장하는 예제 코드에서도 _로 시작하는 모든 클래스는 메시지 핸들러입니다. 이러한 메시지 핸들러에 대한 구현은 이 글에서 별도로 설명하지 않습니다. 왜냐하면 메시지 핸들러는 모두 동일한 방식으로 프로토콜마다 작성하는 클래스이므로 엔진의 핵심 요소를 설명하는 이 가이드의 집중도를 떨어뜨릴 수 있습니다. 그러므로 앞서 알려드린 레퍼런스 서버 프로젝트나 JavaDoc을 통해 코드를 확인하는 것이 가장 쉽고 정확한 방법입니다.*

<br>

GameAnvil 대부분의 기능은 콜백 방식으로 제공합니다. 즉, 엔진 사용자는 **GameAnvil이 제공하는 기본 클래스(Base Classes)를 상속한 후 콜백 메소드를 재정의**하는 형태로 대부분의 기능을 사용하게 됩니다. 이 과정에서 엔진 사용자가 필요한 콜백 메소드만 구현하면 되므로 일부 콜백 메소드는 무시할 수도 있습니다. 이 가이드 문서는 이러한 콜백 메소드의 실제 구현 방법까지는 설명하지 않습니다. 이러한 코드 레벨의 참고는 역시 앞서 설명한 레퍼런스 서버 프로젝트를 참고하는게 가장 좋은 방법입니다.

<br>

## 1.Node 공통

모든 종류의 노드는 공통적으로 아래의 콜백 메소드들을 재정의 해야 합니다. 그와 더불어 각 노드는 용도에 맞춰 추가 콜백 메소드를 구현하게 됩니다. 아래 코드의 GatewayNode는 추가 콜백이 없으므로 공통 콜백 메소드를 보여주기에 적합합니다. 특히, 주의해서 봐야할 부분은 onPrepare()에서 호출한 setReady() 입니다. 이 API는 엔진에게 노드가 모두 준비되었으니 실제 구동을 시작하라는 명령입니다. 일반적으로 onPrepare()에서 호출합니다. 하지만 다른 노드와의 통신을 통하거나 임의의 비동기 결과를 바탕으로 준비 명령을 내리고 싶을 경우에는 다른 코드 블럭에서 호출해도 무방합니다.

```java
public class SampleGatewayNode extends BaseGatewayNode {

    /**
     * 초기화 할때 발생.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onInit() throws SuspendExecution {        
    }

    /**
     * Prepare 시에 호출 된다.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPrepare() throws SuspendExecution {
        setReady(); // 주의!
    }

    /**
     * Ready 단계에서 호출 된다.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onReady() throws SuspendExecution {        
    }

    /**
     * 패킷이 전달될 때 호출 된다.
     *
     * @param packet 전달된 {@link Packet} 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {        
    }

    /**
     * pause 될때에 호출 된다.
     *
     * @param payload contents 에서 전달하고자 하는 추가 정보 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPause(Payload payload) throws SuspendExecution {        
    }

    /**
     * resume 될때에 호출 된다.
     *
     * @param payload contents 에서 전달하고자 하는 추가 정보 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onResume(Payload payload) throws SuspendExecution {        
    }

    /**
     * NonNetworkNode 종료를 위해 shutdown 명령시 호출 된다.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onShutdown() throws SuspendExecution {        
    }
}
```
<br>

## 2.Gateway Node

GatewayNode는 클라이언트가 접속하는 노드입니다. 앞서 살펴본 것처럼 BaseGatewayNode 클래스를 상속하여 공통 콜백 메소드만 재정의하면 됩니다. 그 외 GatewayNode 자체에 특별히 추가로 필수 구현할 사항은 없습니다.  GatewayNode는 클라이언트로부터의 Connection과 Session들을 관리하는데 이들의 관계는 다음의 그림과 같습니다.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

일반적으로 클라이언트는 GatewayNode로 하나의 Connection을 맺습니다. 이 때, 해당 Connection에 대해 인증 절차를 진행하여 성공한 경우에 하나 이상의 Session을 생성할 수 있습니다. 각각의 세션은 서로 다른 게임 서비스에 대한 논리적 연결 단위입니다. 위의 그림은 클라이언트가 하나의 Connection을 통해 Game 서비스와 Chat 서비스 각각에 대해 Session을 생성한 모습입니다.

<br>

### 2-1.Connection

Connection은 클라이언트의 물리적 접속 자체를 의미합니다. 클라이언트는 고유한 AccountId를 이용하여 Connection 상에서 인증 절차를 진행할 수 있습니다. 인증이 성공할 경우 해당 AccountId는 생성된 Connection에 매핑됩니다. Connection은 아래와 같은 콜백 메소드를 재정의 합니다.

```java
public class SampleConnection extends BaseConnection<SampleGameSession> {

    /**
     * authenticate 시에 호출 된다.
     *
     * @param accountId  계정 Id 전달.
     * @param password   계정 password 전달.
     * @param deviceId   client 의 deviceId 전달.
     * @param payload    전달 받은 {@link Payload} 전달.
     * @param outPayload client 로 전달 할 {@link Payload} 전달.
     * @return boolean type 으로 반환. false 시 client 와 의 접속이 종료.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onAuthenticate(final String accountId, final String password, final String deviceId, final Payload payload, final Payload outPayload) throws SuspendExecution {        
    }

    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {        
    }

    /**
     * login 호출 이전에 호출 된다.
     *
     * @param outPayload {@link Payload} 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPreLogin(Payload outPayload) throws SuspendExecution {        
    }

    /**
     * 로그인 성공 이후에 호출 된다.
     *
     * @param session session 에 생성되는 객체 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPostLogin(U session) throws SuspendExecution {        
    }

    /**
     * 로그아웃 이후에 호출 된다.
     *
     * @param session space node 의 user 에 연결된 Session 객체 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPostLogout(U session) throws SuspendExecution {        
    }

    @Override
    public void onPause() throws SuspendExecution {        
    }

    @Override
    public void onResume() throws SuspendExecution {        
    }

    /**
     * 클라이언트와 연결이 끊어졌을때 호출 된다.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public void onDisconnect() throws SuspendExecution {        
    }
}
```

<br>

### 2-2.Session

Connection을 성공적으로 맺은 클라이언트는 해당 Connection 상에서 서비스별로 한 개씩 GameNode에 논리적인 Session을 맺을 수 있습니다. 고유한 Session은 Connection의 AccountId와 Session의 SubId를 조합해서 나타낼 수 있습니다. Session은 아래와 같이 해당 세션에서 처리할 메시지와 핸들러를 등록한 후 콜백 메소드를 통해 디스패치할 수 있습니다.

```java
public class SampleSession extends BaseSession {
    
    private static PacketDispatcher dispatcher = new PacketDispatcher();

    static {
        dispatcher.registerMsg(SampleGame.MsgToSessionReq.getDescriptor(), _MsgToSessionHandler.class);
    }

    /**
     * Session으로 패킷이 전달될 때 호출 된다.
     *
     * @param packet 전달된 {@link Packet} 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        dispatcher.dispatch(this, packet);
    }
}
```

<br>

### 2-3.Bootstrap

이렇게 생성한 GatewayNode와 Connection 그리고 Session은 Bootstrap 단계에서 아래와 같이 등록할 수 있습니다.

```java
public class Main {

    public static void main(String[] args) {

        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();
        // 등록
        bootstrap.setGateway()
            .connection(SampleConnection.class)
            .session(SampleSession.class)
            .node(SampleGatewayNode.class)
            .enableWhiteModules();

        bootstrap.run();
    }
}
```

<br>

## 3. Game Node

GameNode는 실제 게임 객체가 생성되고 게임 컨텐츠를 구현하는 노드입니다. 이러한 GameNode는 기본적으로 BaseGameNode 추상 클래스를 상속 구현합니다. GameNode가 관리하는 대표적인 객체는 GameUser와 GameRoom 입니다. 이에 관해서는 이 글의 아래에서 더욱 자세하게 살펴봅니다. 아래의 코드는 GameNode에서 기본적으로 재정의할 수 있는 콜백 메소드를 보여줍니다. 노드 공통 콜백에 채널 간 동기화를 위한 콜백이 추가되었습니다. 또한 GameNode에서 처리하고 싶은 프로토콜은 임의의 핸들러와 매핑한 후 onDispatch() 콜백을 통해 해당 패킷을 처리할 수 있습니다. 아래 코드에서 1~3에 해당하는 주석 아래 코드가 이를 위한 처리 흐름을 순서대로 보여줍니다.

<br>

```java
public class SampleGameNode extends BaseGameNode {
    
    // 1.패킷 디스패처 생성    
    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    // 2.SampleGameNode에서 처리하고 싶은 프로토콜과 핸들러를 매핑
    static {
        packetDispatcher.registerMsg(SampleGame.GameNodeTest.getDescriptor(), _GameNodeTest.class);
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onPrepare() throws SuspendExecution {
        setReady(); // 주의
    }

    /**
     * 패킷이 전달될 때 호출 된다.
     *
     * @param packet 전달된 {@link Packet} 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {
		// 3.SampleGameNode의 메시지 처리
        if (packetDispatcher.isRegisteredMessage(packet))
            packetDispatcher.dispatch(this, packet);
    }

    @Override
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {
    }

    @Override
    public void onResume(Payload payload) throws SuspendExecution {
    }

    @Override
    public void onShutdown() throws SuspendExecution {
    }

    /**
     * 같은 채널의 다른 node 에 유저 변화가 있을때 올라오는 callback.
     * <p>
     * updateChannelUser() 호출시 발생.
     *
     * @param type            Channel 정보 변경 타입(갱신/삭제) 전달.
     * @param channelUserInfo 변경될 User 정보 전달.
     * @param userId          변경 대상의 User Id 전달.
     * @param accountId       변경 대상의 Account Id 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onChannelUserUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {        
    }

    /**
     * 같은 채널의 다른 node 에 room 상태 변화가 있을때 올라오는 callback.
     * <p>
     * updateChannelRoomInfo() 호출시 발생.
     *
     * @param type            Channel 정보 변경 타입(갱신/삭제) 전달.
     * @param channelRoomInfo 변경될 Room 정보 전달.
     * @param roomId          변경 대상의 Room Id 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onChannelRoomUpdate(ChannelUpdateType type, RoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {        
    }

    /**
     * Client 에서 Channel 정보를 요청 시 올라오는 callback. (Base.GetChannelInfoReq)
     *
     * @param outPayload Client 로 전달될 Channel 정보 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {        
    }
}
```

<br>

### 3-1.GameUser

게임 상의 유저를 나타내는 GameUser 객체는 로그인 과정을 거쳐 GameNode에 생성됩니다. 유저 단위의 모든 메시지는 이 곳에서 처리할 수 있습니다. 그러므로 유저 관련 컨텐츠는 이 클래스를 중심으로 구현하는 것이 바람직합니다. 앞서 살펴본 모든 예제와 마찬가지로 GameUser 또한 처리할 프로토콜과 핸들러를 매핑할 수 있습니다. 



아래의 예제 코드는 GameUser의 전체 콜백 메소드 목록을 보여줍니다. 이 중 일부는 기본 구현이 제공되므로 필요한 상황이 아니면 재정의할 필요는 없습니다. 이는 GameUser 뿐만 아니라 엔진에서 제공하는 대부분의 콜백 메소드에 해당하는 사항입니다. 이러한 부분은 상속받는 각각의 기본 클래스(e.g.BaseUser)에 대해  [GameAnvil API Reference](http://10.162.4.61:9090/gameanvil)에서 JavaDoc 문서를 확인하거나 [레퍼런스 서버 프로젝트](server-link-reference-server)를 참고하는 것이 좋습니다.



아래의 콜백 메소드 중 onLogin()은 최초에 게임 유저를 생성하기 위해 로그인을 시도하는 과정에서 호출됩니다. 이 onLogin() 콜백이 성공하면 GameNode 상에 해당 게임 유저 객체가 생성됩니다. 로그인이 완료된 후에는 사용자가 작성한 프로토콜을 기반으로 클라이언트와 직접 메시지를 주고 받으며 여러가지 컨텐츠를 처리할 수 있습니다.

```java
public class SampleGameUser extends BaseUser {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameUserTest.getDescriptor(), _GameUserTest.class);
    }
    
	/**
     * login 시 발생하는 callback.
     *
     * @param payload        client 에서 전달된 define 전달.
     * @param sessionPayload gateway 의 onPreLogin 에서 전달된 define 값 전달.
     * @param outPayload     client 로 전달되는 define 전달.
     * @return boolean type 으로 true : 로그인 성공. false : 로그인 실패 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onLogin(final Payload payload, 
                           final Payload sessionPayload, 
                           Payload outPayload) throws SuspendExecution {        
    }

    /**
     * Login 처리 이후에 발생하는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPostLogin() throws SuspendExecution {
        
    }

    /**
     * 한 이디로 유저가 login 된 상황에서 다른 디바이스에서 같은 유저가 login 할때 호출되는 callback.
     *
     * @param newDeviceId           새로 접속한 User의 deviceId 값 전달.
     * @param outPayloadForKickUser kickOut 혹은 LoginRes 를 통해서 client 에 전달.
     * @return boolean type 으로 true :새로 접속한 유저가 login success,현재 유저는 forceLogout. false : 새로 접속한 유저가 login fail 를 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다/
     */
    @Override
    public boolean onLoginByOtherDevice(final String newDeviceId, 
                                        Payload outPayloadForKickUser) throws SuspendExecution {
        return true;
    }

    /**
     * MultiType 유저를 사용할때 어떤 type 의 유저가 login 된 상황에서 다른 type 의 유저가 login 할때 호출되는 callback.
     *
     * @param userType   새로 로그인을 시도하는 user 의 type 전달.
     * @param outPayload client 로 전달되는 define 전달.
     * @return boolean type 으로 true : 새 로그인 성공. false : 새 로그인 실패 를 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onLoginByOtherUserType(final String userType, 
                                          Payload outPayload) throws SuspendExecution {
        return true;
    }

    /**
     * Login 한 상황에서 동일 device 동일 id 로 login 할때 호출되는 callback.
     *
     * @param outPayload client 로 전달되는 define 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {        
    }

    /**
     * server 에 user 객체가 살아 있는 상태에서 login 이 호출되었을 경우 발생하는 callback.
     *
     * @param payload        client 에서 전달된 define 전달.
     * @param sessionPayload gateway 의 onPreLogin 에서 전달된 define 값 전달.
     * @param outPayload     client 로 전달되는 define 전달.
     * @return boolean type 으로 true : ReLogin 성공, false : ReLogin 실패 를 반환.
     * @throws SuspendExecution : 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onReLogin(final Payload payload, 
                             final Payload sessionPayload, 
                             Payload outPayload) throws SuspendExecution {
    }

    /**
     * client 와의 연결이 끊어 졌을때 호출되는  callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onDisconnect() throws SuspendExecution {        
    }

    /**
     * user 로 전달되는 패킷이 있을때 호출되는 callback.
     *
     * @param packet Node 로부터 {@link Packet} 을 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, packet);
    }

    /**
     * node 가 pause 될때 node 가 관리하고 있는 user 에게 호출되는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPause() throws SuspendExecution {        
    }

    /**
     * node 가 resume 될때 node 가 관리하고 있는 user 에게 호출되는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onResume() throws SuspendExecution {        
    }

    /**
     * user 가 logout 시 호출되는 callback.
     *
     * @param payload    입력받은 추가 데이터 전달.
     * @param outPayload 전달할 추가 데이터 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onLogout(final Payload payload, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * user 가 logout 될수 있는 확인 하는 callback.
     *
     * @return boolean type 으로 반환 : false 일경우 logout 안되고 주기적으로 다시 확인 : true 일 경우 user 는 logout 이 된다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean canLogout() throws SuspendExecution {        
    }

    /**
     * client 에서 roomMatch 를 요청했을 경우 발생하는 callback.
     *
     * @param roomType 매칭되는 room 의 type 전달.
     * @param payload  client 의 요청시 추가적으로 전달되는 define 잔달.
     * @return {@link MatchRoomResult} 으로 matching 된 room 의 정보 반환, null 을 반환 할시  client 요청 옵션에 따라서 새로운 Room 이 생성되거나,요청 실패 처리 된다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public MatchRoomResult onMatchRoom(final String roomType, 
                                       final Payload payload) throws SuspendExecution {
        return null;
    }

    /**
     * client 에서 userMatch 를 요청했을 경우 호출되는 callback.
     *
     * @param roomType   매칭되는 room 의 type 전달.
     * @param payload    client 의 요청시 추가적으로 전달되는 define 전달.
     * @param outPayload 서버에서 client 로 전달되는 define 전달.
     * @return boolean type 으로 반환. true: user matching 요청 성공,false: user matching 요청 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onMatchUser(final String roomType, 
                               final Payload payload, 
                               Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * client 에서 userMatch 가 취소되었을 때 호출되는 callback.
     *
     * @param reason 취소된 이유(TIMEOUT/CANCEL) 전달.
     * @return boolean type 으로 반환. true: user matching 취소 성공, false: user matching 취소 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
        return false;
    }

    /**
     * 타이머 핸들러가 등록되고 호출되는 callback.
     */
    @Override
    public void onRegisterTimerHandler() {        
    }

    /**
     * user 가 다른 node 로 이동할때 user define 된 데이터를 전달 하기 위해서 호출되는 callback.
     *
     * @param transferPack 필요한 데이터를 설정할 객체를 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onTransferOut(final TransferPack transferPack) throws SuspendExecution {        
    }

    /**
     * user 가 다른 node 로 이동한후 user define 를 복구하기 위해 호출되는 callback.
     * <p>
     * 이 시점에는 user 가 완전히 복구되기 전이기 때문에 다른 곳으로 request 호출이 제한된다.
     *
     * @param transferPack 이동전 전달되었던 객체가 전달. 해당 객체에서 설정 값을 가져온다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onTransferIn(final TransferPack transferPack) throws SuspendExecution {        
    }

    /**
     * user 의 node 간 transfer 가 완료된후 호출되는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPostTransferIn() throws SuspendExecution {        
    }

    /**
     * client 에서 snapshot 요청시 호출되는 callback.
     * <p>
     * 주로 접속이 끊어 지고  서버 상태가 변할 확률이 있을 경우 호출해서 client 와 server 상태정보를 동기화 하는데 사용된다.
     *
     * @param payload    client 에서 서버로 전달되는 define 전달.
     * @param outPayload 서버에서 client 로 전달되는 define 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onSnapshot(final Payload payload, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * client 에서 user 에게 다른 채널로 이동요청을 할 경우 호출되는 callback.
     *
     * @param destinationChannelId 이동하려고 하는 대상 channel ID 전달.
     * @param payload              client 에서 서버로 전달하는 define 전달.
     * @param errorPayload         channel 이동 실패시 서버에서 client 로 전달하는 define 전달. 성공일 경우 전달 안됨.
     * @return boolean type 으로 반환. false : 채널 이동 요청 실패, true: 채널 이동 요청 성공.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onCheckMoveOutChannel(final String destinationChannelId, 
                                         final Payload payload,
                                         Payload errorPayload) throws SuspendExecution {
        return false;
    }

    /**
     * client 에서 채널 이동 요청이 들어 온 경우  onCheckMoveOutChannel() 에서 true 반환한 경우 호출되는 callback.
     * <p>
     * 채널 이동시 이동할 채널로 정보를 전달 하기 위해서 호출 서버의 moveChannel() 호출시 onCheckMoveOutChannel() 가 호출이 안되고 바로 호출이 된다.
     *
     * @param destinationChannelId 이동할 channel ID 전달.
     * @param outPayload           이동할 channel 에 전달할 define 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onMoveOutChannel(final String destinationChannelId,
                                 Payload outPayload) throws SuspendExecution {
    }

    /**
     * onMoveOutChannel() 호출후 호출되는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPostMoveOutChannel() throws SuspendExecution {
    }

    /**
     * 이동한 채널로 입장할때 발생하는 callback.
     *
     * @param sourceChannelId 이동하기전 channel ID 전달.
     * @param payload         이동하기 전 channel 에서 보낸 define 전달.
     * @param outPayload      채널 이동후 client 로 전달되는 define 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onMoveInChannel(final String sourceChannelId,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * onMoveInChannel() 수행후 호출되는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPostMoveInChannel() throws SuspendExecution {
    }

    /**
     * User Transfer 가 가능한 지 확인.
     *
     * @return boolean type 으로 반환. true: Transfer 가능, false : Transfer 불가능, Transfer 가 불가능일 경우 NonStopPatch 가 종료되기 전까진 지속적으로 호출된다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean canTransfer() throws SuspendExecution;
}
```

<br>

### 3-2.GameRoom

두 명 이상의 게임 유저는 GameRoom을 통해 동기화된 메시지 흐름을 만들 수 있습니다. 즉, 게임 유저들의 요청은 GameRoom 내에서 모두 순서가 보장됩니다. 물론 한 명의 게임 유저를 위한 GameRoom 생성도 컨텐츠에 따라서 의미를 가질 수도 있습니다. GameRoom을 어떻게 사용할지는 어디까지나 엔진 사용자의 몫입니다. 이러한 GameRoom은 GameUser와 마찬가지로 기본 클래스(BaseRoom)을 바탕으로 여러가지 자체 콜백 메소드를 재정의할 수 있으며 자체적으로 프로토콜을 처리할 수도 있습니다. 아래의 예제 코드는 SampleGameUser를 위한 SampleGameRoom 클래스입니다.

```java
public class SampleGameRoom extends BaseRoom<SampleGameUser> {
    
    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameRoomTest.getDescriptor(), _GameRoomTest.class);
    }

    /**
     * Room 이 초기화 될때 발생하는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onInit() throws SuspendExecution {        
    }

    /**
     * Room 이 삭제 될때 발생하는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onDestroy() throws SuspendExecution {        
    }

    /**
     * Room 으로 {@link Packet} 이 전달될때 호출되는 callback.
     *
     * @param user   해당 패킷을 처리 할 user 객체를 전달, 해당 패킷을 처리할  user 가 없을 경우 null.
     * @param packet 전달 되는 {@link Packet}.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onDispatch(SampleGameUser user, final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, user, packet);
    }

    /**
     * client 에서 Room 생성을 요청 할때 발생하는 callback.
     *
     * @param user       Room 생성을 요청한 user 객체 전달.
     * @param inPayload  Room 생성 요청시 client 에서 보낸 define 전달.
     * @param outPayload Room 생성 요청시 client 로 전달할 define 전달.
     * @return boolean type 으로 반환. true: Room 생성 성공, false: Room 생성 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onCreateRoom(SampleGameUser user, 
                                final Payload inPayload, 
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * client 에서 Room 입장 요청을 할때 발생하는 callback.
     *
     * @param user       Room 입장을 요청하는 user 객체 전달.
     * @param inPayload  Room 입장 요청시 client 에서 보낸 define 전달.
     * @param outPayload Room 생성 요청을 한  client 에게 전달할 define 전달.
     * @return boolean type 으로 반환. true : Room 입장 성공, false: Room 입장 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onJoinRoom(SampleGameUser user, 
                              final Payload inPayload, 
                              Payload outPayload) throws SuspendExecution {
    }

    /**
     * client 에서 Room 나가기 요청을 할때 발생하는 callback.
     *
     * @param user       나가기 요청을 한 user 객체 전달.
     * @param inPayload  Room 나가기 요청시 client 에서 보낸 define 전달.
     * @param outPayload Room 나가기 요청을 한 client 에게 보낼 define 전달.
     * @return boolean type 으로 반환. true Room 나가기 성공, false Room 나기기 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onLeaveRoom(SampleGameUser user, 
                               final Payload inPayload, 
                               Payload outPayload) throws SuspendExecution { 
    }

    /**
     * onLeaveRoom 호출 후 반환값이 true 인경우 호출되는 callback.
     *
     * @param user onLeaveRoom 처리가 끝난 User 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPostLeaveRoom(SampleGameUser user) throws SuspendExecution {
    }

    /**
     * user 가 Room 에 있는 상태에서 ReJoin 될 경우 호출되는 callback.
     *
     * @param user       ReJoin 을 한 user 객체 전달.
     * @param outPayload ReJoin 인시  client 에게 전달할 room define 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onRejoinRoom(SampleGameUser user, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * 타이머 핸들러가 등록되고 호출되는 callback.
     */
    @Override
    public void onRegisterTimerHandler() {
    }

    /**
     * room transfer 발생시 room 의 define 를 serialize 하기 위해 호출되는 callback.
     *
     * @param transferPack 이동전 전달되었던 객체가 전달. 해당 객체에서 설정 값을 가져온다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * room transfer 발생시 room 의 define 를 복원하기 위해서 호출되는 callback.
     *
     * @param userList     해당 Room 에 있는 User 객체 리스트 전달.
     * @param transferPack 복원할 Room 정보 define 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onTransferIn(List<SampleGameUser> userList, 
                             final TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * node 가 pause 되고 난후 각 room 객체에서 호출되는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onPause() throws SuspendExecution {
    }

    /**
     * node 가 resume 되고 난 후 각 room 에서 호출되는 callback.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * client 에서 partyMatch 를 요청했을 경우 호출되는 callback.
     *
     * @param roomType   RoomType 전달.
     * @param user       파티 매칭을 요청한 방장 전달.
     * @param payload    client 의 요청시 추가적으로 전달되는 define 전달.
     * @param outPayload 서버에서 client 로 전달되는 define 전달.
     * @return boolean type 으로 반환. true: user matching 요청 성공,false: user matching 요청 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean onMatchParty(final String roomType, 
                                final SampleGameUser user, 
                                final Payload payload, 
                                Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * room transfer 를 현재 할 수 있는 지 체크.
     * <p>
     * 최초 callback 호출 후 NonNetworkNode 상태가 Ready 가 되기 전까지 지속적으로 체크.
     *
     * @return boolean type 으로 반환. true: room transfer 가능, false: room transfer 불가능.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public boolean canTransfer() throws SuspendExecution {        
    }
}
```

<br>

### 3-3.Bootstrap

이렇게 생성한 GameNode와 GameUser 그리고 GameRoom은 Bootstrap 단계에서 아래와 같이 등록할 수 있습니다.

```java
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGateway()
            .connection(SampleGameConnection.class)
            .session(SampleGameSession.class)
            .node(SampleGameGatewayNode.class)
            .enableWhiteModules();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)
            .room("SampleGameRoomType", SampleGameRoom.class);

        bootstrap.run();
    }
}
```

<br>

## 6.MatchNode & MatchMaker

엔진 사용자는 간단하게 매칭 로직만 구현함으로써 매치메이킹을 적용할 수 있습니다. 매치메이커는 로직에 기반하여 유저들을 동일한 방으로 입장시켜줍니다. GameAnvil은 RoomMatchMaker와 UserMatchMaker의 두 가지의 매치메이커를 제공합니다. 이 매치메이커는 MatchNode에서 독자적으로 구동됩니다. 이러한 MatchNode는 매치메이킹을 수행하는 용도 외에 추가적인 컨텐츠를 구현할 수 없습니다. 그러므로 유저는 MatchNode가 아닌 매치메이커에만 집중하면 됩니다.



### Note

*매치메이커는 매칭그룹 단위로 생성됩니다. 즉, 동일한 매칭그룹끼리 매치메이킹을 수행합니다. 이러한 매칭그룹에 대해서는 중급 개념에서 좀 더 자세하게 설명합니다.*

<br>

### 6-1. UserMatchMaker

유저 매치메이킹은 게임 유저들의 매칭 요청을 큐에 적재합니다. 특정 시간 주기로 이 요청 큐의 내용을 비교,분석하여 사용자가 원하는 기준으로 임의의 유저들을 하나의 방으로 입장시켜줍니다. 여기에서 엔진 사용자는 요청 큐의 내용을 어떻게 비교하고 분석해서 어떤 기준으로 유저들을 매칭시킬지에 대한 로직에만 집중하면 됩니다. 참고로 가장 대표적인 유저 매치메이킹 게임은 "*리그 오브 레전드*"가 있습니다.



이러한 유저 매치메이킹의 가장 기본은 바로 매칭 요청 그 자체입니다. 이러한 매칭 요청을 아래와 같이 엔진에서 제공하는 UserMatchInfo 추상 클래스를 상속하여 구현합니다.  요청자를 구분할 수 있는 게임 유저의 ID를 제공할 수 있도록 getId() 메소드는 반드시 구현해야 합니다. 아래의 예제는 매칭 요청 사이의 비교를 위해 Comparable 인터페이스를 추가로 구현하고 있습니다.

```java
public class SampleUserMatchInfo extends UserMatchInfo implements Serializable, Comparable<SampleUserMatchInfo> {

    private int userId;
    private int rating;

    public SampleUserMatchInfo(int userId, int rating) {
        this.userId = id;
        this.rating = rating;
    }
    
    public int getRating() {
        return rating;
    }

    /**
     * 파티매칭 요청인경우 RoomId, 유저매칭 요청인경우 UserId를 반환 해준다.
     *
     * @return int type 으로 파티매칭 요청인경우 RoomId, 유저매칭 요청인경우 UserId 를 반환.
     */
    @Override
    public int getId() {
        return userId;
    }

    /**
     * 파티매칭 요청인경우 파티의 크기(인원수), 유저매칭 요청인경우 0을 반환 해준다.
     *
     * @return int type 으로 파티매칭 요청인경우 파티의 크기(인원수), 유저매칭 요청인경우 0을 반환.
     */
    @Override
    public int getPartySize() {
        return 0;
    }
    
    // 만일 SampleUserMatchInfo 객체 사이에 비교가 필요하다면 Comparable 인터페이스를 구현합니다.
    @Override
    public int compareTo(SampleUserMatchInfo o) {
        if (this.rating < o.getRating())
            return -1;
        else if (this.rating > o.getRating())
            return 1;

        if (this.id < o.getId())
            return -1;
        else if (this.id > o.getId())
            return 1;

        return 0;
}
```

<br>

이제 유저 매치메이커를 만들 차례입니다. 유저 매치메이커는 엔진에서 제공하는 UserMatchMaker 추상 클래스를 상속 구현 합니다. 특히, match() 메소드는 실제 매칭을 수행하기 위해 호출되는 콜백이므로 주의 깊게 살펴보세요.  refill() 메소드는 이미 완료된 매치메이킹에 대해 충원 요청이 왔을 때 호출되는 콜백입니다. 예를 들어 4명이 매치메이킹 된 상태에서 1명이 게임을 종료해버렸을 때 1명을 더 충원하기 위해 사용할 수 있습니다. 아래의 예제 코드는 이러한 UserMatchMaker를 어떤식으로 구현할 수 있는지 보여줍니다.

```java
public class SampleUserMatchMaker extends UserMatchMaker<SampleUserMatchInfo> {

    private static final Logger logger = getLogger(SampleUserMatchMaker.class);

    public SampleUserMatchMaker() {
        super(2, 5000);
    }

    private Multiset<SampleUserMatchInfo> ratingSet = TreeMultiset.create();
    private final int matchMultiple = 1; // match 정원의 몇 배수까지 인원을 모은 후에 rating 별로 정렬해서 매칭할 것인가?
    private int currentMultiple = matchMultiple;
    private long lastMatchTime = System.currentTimeMillis();
    private int totalMatchMakings = 0;

    @Override
    public void match() {
        List<SampleUserMatchInfo> matchRequests = getMatchRequests(matchSize * currentMultiple);

        // 최소 개수(minAmount)만큼 적재되지 않았음
        if (matchRequests == null) {
            if (System.currentTimeMillis() - lastMatchTime >= 10000)
                currentMultiple = Math.max(--currentMultiple, 1);

            return;
        }

        // matching이 성사되지 않은 항목들은 ratingSet에 그대로 남아있을 수 있으나 따로 보관할 필요는 없다.
        // 이 항목들은 다음 getMatchRequests()에서 다시 전달받는다.
        ratingSet.clear();
        ratingSet.addAll(matchRequests);

        if (ratingSet.size() >= matchSize) {

            // ratingSet의 순서대로 matchingAmount*matchSize 만큼 항목들을 소비API
            int matchingAmount = matchSingles(ratingSet);

            if (matchingAmount > 0) {
                totalMatchMakings += matchingAmount;
                logger.info("{} match(s) made (total: {}) - {}", matchingAmount, totalMatchMakings, this.getMatchingGroup());

                lastMatchTime = System.currentTimeMillis();
                currentMultiple = matchMultiple;
            }
        }
    }

    @Override
    public boolean refill(SampleUserMatchInfo matchReq) {
        try {
            List<SampleUserMatchInfo> refillRequests = getRefillRequests();

            if (refillRequests.isEmpty()) {
                return false;
            }

            for (SampleUserMatchInfo refillInfo : refillRequests) {
                // 100점 이상 차이나지 않으면 리필
                if (Math.abs(matchReq.getRating() - refillInfo.getRating()) < 100) {
                    if (refillRoom(matchReq, refillInfo)) { // 해당 매칭 요청을 리필이 필요한 방으로 매칭
                        return true;
                    }
                }
            }
        } catch (Exception e) {
            logger.error("SampleUserMatchMaker::refill()", e);
        }

        return false;
    }
}

```

<br>

이렇게 생성한 UserMatchMaker와 UserMatchInfo는 Bootstrap 단계에서 등록해야 합니다. 한 가지 주의할 점은 매치메이커는 아래와 같이 게임 노드 구성의 연장선 상에 있다는 점입니다. 즉, GameNode에서 사용할 GameUser와 GameRoom을 구성하는 것과 더불어 이들을 매칭하기 위해 어떤 UserMatchMaker를 사용할지 구성하는 것입니다.

```java
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)
            .room("SampleUserMatchType", SampleGameRoom.class)

            // 등록
            .userMatchMaker("MyUserMatchRoomType", SampleUserMatchMaker.class, SampleUserMatchInfo.class);

        bootstrap.run();
    }
}
```

<br>

이제 클라이언트는 서버로 유저 매치메이킹을 요청할 수 있습니다. 이 요청은 GameUser에 전달된 후 엔진에 의해 아래의 콜백 메소드를 호출합니다. 이 콜백 메소드는 엔진 사용자로 하여금 GameAnvil이 제공하는 유저 매치메이커 뿐만 아니라 제3의 솔루션을 사용할 기회를 제공합니다. 아래의 예제는 엔진에서 제공하는 유저 매치메이커를 사용하기 위해 matchUser() API를 호출하는 모습입니다. 엔진 사용자는 자체적으로 매치메이커를 만들어서 사용하거나 다른 라이브러리 등을 사용할 수도 있습니다. 이 경우에는 그에 알맞게 아래의 콜백 메소드를 구현하도록 합니다. 특히, 이 예제 코드에서 유심히 봐야할 부분은 매칭그룹입니다. 매칭 그룹은 매치메이킹을 논리적으로 나눌 수 있는 개념으로 이 문서의 중급 개념에서 좀 더 자세하게 설명합니다. 예제에서 사용한 "Newbie" 매칭 그룹은 동일한 "Newbie" 매칭 그룹끼리 같은 매칭 큐를 공유하게 됩니다.

```java
    /**
     * client 에서 userMatch 를 요청했을 경우 호출되는 callback.
     *
     * @param roomType   매칭되는 room 의 type 전달.
     * @param payload    client 의 요청시 추가적으로 전달되는 define 전달.
     * @param outPayload 서버에서 client 로 전달되는 define 전달.
     * @return boolean type 으로 반환. true: user matching 요청 성공,false: user matching 요청 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public boolean onMatchUser(final String roomType, 
                               final Payload payload, Payload outPayload) throws SuspendExecution {
        
        String matchingGroup = "Newbie";
        SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId());
        sampleUserMatchInfo.setRating(rating);
        
        return matchUser(matchingGroup, roomType, term, payload);
    }
```

<br>

마지막으로 클라이언트는 언제든 앞서 요청한 매치메이킹에 대해 취소를 할 수 있습니다. 이 때, 엔진은 취소 처리가 성공적으로 진행되면 아래와 같은 GameUser의 콜백 메소드를 호출합니다. 엔진 사용자는 이 콜백에서 취소 타이밍에 처리하고 싶은 부분을 구현하면 됩니다.

```java
    /**
     * client 에서 userMatch 가 취소되었을 때 호출되는 callback.
     *
     * @param reason 취소된 이유(TIMEOUT/CANCEL) 전달.
     * @return boolean type 으로 반환. true: user matching 취소 성공, false: user matching 취소 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
    }
```

<br>

### 6-2. RoomMatchMaker

룸 매치메이킹은 게임 유저들을 적합한 방으로 자동 입장시켜줍니다. 룸 매치메이킹을 요청한 게임 유저를 어떤 방으로 입장시킬지는 엔진 사용자가 구현하기에 달렸습니다. 가장 유저수가 많은 방으로 입장시킬 수도 있고, 가장 한산한 방으로 입장시킬 수도 있습니다. 혹은 평균 점수가 가장 높은 방으로 입장시킬 수도 있습니다. 엔진 사용자는 이러한 매칭 로직에만 집중하면 됩니다. 참고로 가장 대표적인 룸 매치메이킹 게임은 "*한게임 포커*"나 "*카트라이더*" 등이 있습니다.



이러한 룸 매치메이킹의 가장 기본은 바로 매칭 요청 그 자체입니다. 이러한 매칭 요청을 아래와 같이 엔진에서 제공하는 RoomMatchInfo 인테페이스를 구현합니다.  요청자를 구분할 수 있는 게임 유저의 ID를 제공할 수 있도록 getId() 메소드는 반드시 구현해야 합니다.

```java
public class SampleRoomMatchInfo implements Serializable, RoomMatchInfo {

    private int roomId;
    private int userCurrentCount = 0;
    private int maxUserCount = 4;

    public SampleRoomMatchInfo() {
    }

    public void setRoomId(int roomId) {
        this.roomId = roomId;
    }

    public int getUserCurrentCount() {
        return userCurrentCount;
    }

    public void setUserCurrentCount(int userCurrentCount) {
        this.userCurrentCount = userCurrentCount;
    }

    public int getMaxUserCount() {
        return maxUserCount;
    }
    
    /**
     * RoomId를 얻는다.
     * <p>
     * 상황에 따라 다른 의미로 사용된다.
     * <p>
     * RoomMatchInfo 가 매칭 가능한 Room 또는 매칭된 Room 의 정보를 의미하는 경우 해당 Room 의 RoomId를 의미한다.
     * <p>
     * RoomMatchInfo 가 매칭 조건을 의미하는 경우 컨탠츠에서 지정한 RoomId를 반환한다.
     * <p>
     * 컨텐츠에서 원하는 방식으로 사용할 수 있다.
     * <p>
     * ex) 매칭에서 제외할 RoomId.
     *
     * @return int type 으로 RoomId를 반환.
     */
    int getRoomId();
    @Override
    public int getRoomId() {
        return roomId;
    }
}
```

<br>

이제 룸 매치메이커를 만들 차례입니다. 룸 매치메이커는 엔진에서 제공하는 RoomMatchMaker 추상 클래스를 상속 구현 합니다. 특히, match() 메소드는 실제 매칭을 수행하기 위해 호출되는 콜백이므로 주의 깊게 살펴보세요. 아래의 예제 코드는 이러한 RoomMatchMaker를 어떤식으로 구현할 수 있는지 보여줍니다. UserMatchMaker는 UserMatchInfo가 Comparable 인터페이스를 구현한 반면에 RoomMatchMaker는 Comparator를 제공하고 있습니다. 이는 단순히 엔진 사용자가 유연하게 원하는 방식대로 구현할 수 있음을 부여주기 위함입니다.

```java
public class SampleGameRoomMatchMaker extends RoomMatchMaker<SampleGameRoomMatchRoomInfo> {

    /**
     * MatchRoom 요청을 처리한다.
     * <p>
     * 등록된 Room들 중에 매칭 조건과 맞는 Room을 찾아 매칭결과를 반환 한다.
     * <p>
     * {@link BaseUser#onMatchRoom(String, Payload)} 에서 {@link BaseUser#matchRoom(String, String, RoomMatchInfo)}을 호출하면 호출되어 매칭을 처리할 수 있다.
     * <p>
     * 매칭 조건 RoomMatchInfo 이 terms 로 전달 된다.
     * <p>
     * {@link #getRooms()}를 사용해 얻어온 RoomMatchInfo 의 목록이 매칭가능한 Room 의 정보이다.
     * <p>
     * 이 정보와 terms 를 비교하여 매칭 가능한 Room 을 찾는다.
     * <p>
     * 매칭 가능한 Room 을 찾았다면 그 Room 의 RoomMatchInfo 를 반환하고 못찾았을 경우 null 을 반환한다.
     *
     * @param terms 매칭 조건을 나타내는 {@link RoomMatchInfo} 전달.
     * @param args  추가로 전달 받은 데이터 전달.
     * @return {@link RoomMatchInfo} 으로 매칭된 Room 의 정보 반환. 없을 경우 null 반환.
     */
    @Override
    public SampleGameRoomMatchRoomInfo match(SampleGameRoomMatchRoomInfo terms, Object... args) {

        int bypassRoomId = terms.getRoomId();
        List<SampleGameRoomMatchRoomInfo> rooms = getRooms();
        
        for (SampleGameRoomMatchRoomInfo info : rooms) {
            // moveRoom 옵션이 true 일 경우 참여중인 방은 제외하기
            if (info.getRoomId() == bypassRoomId)
                continue;

            if (info.getMaxUserCount() != terms.getMaxUserCount())
                continue;

            if (info.getMaxUserCount() == info.getUserCurrentCount())
                continue;

            return info;
        }

        return null;
    }

    @Override
    public Comparator<SampleGameRoomMatchRoomInfo> getComparator() {
        return new Comparator<SampleGameRoomMatchRoomInfo>() {
            @Override
            public int compare(SampleGameRoomMatchRoomInfo o1, SampleGameRoomMatchRoomInfo o2) {
                return o1.getMaxUserCount() - o2.getMaxUserCount();
            }
        };
    }
}
```

<br>

이렇게 생성한 RoomMatchMaker와 RoomMatchInfo는 Bootstrap 단계에서 등록해야 합니다. 주의할 점은 유저 매치메이커와 동일합니다. 룸 매치메이커 역시 아래와 같이 게임 노드 구성의 연장선 상에 있습니다. 즉, GameNode에서 사용할 GameUser와 GameRoom을 구성하는 것과 더불어 이들을 매칭하기 위해 어떤 RoomMatchMaker를 사용할지 구성하는 것입니다. 또한, 유저 매치메이커와 룸 매치메이커를 함께 사용할 수 있습니다. 아래의 예제 코드는 이를 보여주기 위해 두 가지 매치메이커를 함께 등록하고 있습니다. 단, 두 매치메이커가 사용하는 Room과 MatchInfo 클래스가 나뉘어져 있음에 주의하세요.

```java
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)
            
            .room("SampleGameUserMatchRoomType", SampleUserMatchRoom.class)
            .userMatchMaker("SampleGameUserMatchRoomType", SampleUserMatchMaker.class, SampleUserMatchInfo.class)

            // 등록
            .room"SampleGameRoomMatchRoomType", SampleRoomMatchRoom.class)
            .roomMatchMaker("SampleGameRoomMatchRoomType", SampleRoomMatchMaker.class, SampleRoomMatchInfo.class);

        bootstrap.run();
    }
}
```

<br>

이제 클라이언트는 서버로 룸 매치메이킹을 요청할 수 있습니다. 이 요청은 GameUser에 전달된 후 엔진에 의해 아래의 콜백 메소드를 호출합니다. 이 콜백 메소드는 엔진 사용자로 하여금 GameAnvil이 제공하는 룸 매치메이커 뿐만 아니라 제3의 솔루션을 사용할 기회를 제공합니다. 아래의 예제는 엔진에서 제공하는 룸 매치메이커를 사용하기 위해 matchRoom() API를 호출하는 모습입니다. 엔진 사용자는 자체적으로 매치메이커를 만들어서 사용하거나 다른 라이브러리 등을 사용할 수도 있습니다. 이 경우에는 그에 알맞게 아래의 콜백 메소드를 구현하도록 합니다. 특히, 이 예제 코드에서 유심히 봐야할 부분은 매칭그룹입니다. 매칭 그룹은 매치메이킹을 논리적으로 나눌 수 있는 개념으로 이 문서의 중급 개념에서 좀 더 자세하게 설명합니다. 예제에서 사용한 "Newbie" 매칭 그룹은 동일한 "Newbie" 매칭 그룹끼리 같은 매칭 큐를 공유하게 됩니다.

```java
    /**
     * client 에서 roomMatch 를 요청했을 경우 발생하는 callback.
     *
     * @param roomType 매칭되는 room 의 type 전달.
     * @param payload  client 의 요청시 추가적으로 전달되는 define 잔달.
     * @return {@link MatchRoomResult} 으로 matching 된 room 의 정보 반환, null 을 반환 할시  client 요청 옵션에 따라서 새로운 Room 이 생성되거나,요청 실패 처리 된다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public MatchRoomResult onMatchRoom(final String roomType, 
                                       final Payload payload) throws SuspendExecution {

        String matchingGroup = "Newbie";
        
        SampleRoomMatchInfo sampleRoomMatchInfo = new SampleRoomMatchInfo(getChannelId());
        sampleRoomMatchInfo.setMatchingGroup(matchingGroup);
        sampleRoomMatchInfo.setRoomMode(RoomMode.NORMAL);

        return matchRoom(matchingGroup, roomType, terms);
    }
```

<br>

## 7. Support Node

SupportNode는 보조적인 기능을 수행하기 위한 노드입니다. 게임 유저나 방 객체와 상관없이 임의의 기능을 구현할 수 있습니다. 예를 들어 로그를 취합해서 전송하거나 빌링 서버와 통신을 전담하는 등의 역할로 사용할 수 있습니다. 또한 엔진의 RESTful 기능을 사용하여 간단한 웹서버 대용으로 사용하는 것도 가능합니다. 



이러한 SupportNode는 기본적으로 BaseSupportNode 추상 클래스를 상속 구현합니다. 노드 공통 콜백 메소드에 추가로 RESTful 처리를 위한 onDispatch() 콜백이 제공됩니다. 아래 코드에서 1~3에 해당하는 주석 아래 코드가 이를 위한 처리 흐름을 순서대로 보여줍니다.

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

    public abstract boolean onDispatch(RestObject restObject) throws SuspendExecution;

        
    /**
     * rest call 이 전달될 때 호출 된다.
     *
     * @param restObject 전달된 {@link RestObject} 전달.
     * @return boolean type 으로 반환.
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

<br>

이렇게 생성한 SupportNode는 Bootstrap 단계에서 아래와 같이 등록할 수 있습니다.

```java
public class Main {

    public static void main(String[] args) {

        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        // 등록
        bootstrap.setSupport("SampleWebSupport")
            .node(SampleSupportNode.class);

        bootstrap.run();
    }
}
```

<br>

### 8.프로토콜 정의와 컴파일

GameAnvil은 [Google Protocol Buffers](https://developers.google.com/protocol-buffers)를 사용하여 프로토콜을 정의하고 빌드합니다. 아래의 예제는 이러한 프로토콜을 정의하는 법과 빌드하는 법을 설명합니다. 우선 SampleGame.proto 파일을 텍스트 에디터로 생성한 후 원하는 프로토콜을 정의합니다. 프로토콜 버퍼의 자세한 문법은 [공식 Protocol Buffers 가이드](https://developers.google.com/protocol-buffers/docs/overview)를 참고할 수 있습니다.

```protobuf
package [패키지명];

// 참조해야할 다른 proto파일이 있다면 import
import [proto 파일 명]

enum SampleUserTypeEnum {
    USER_TYPE_NONE = 0;
    USER_TYPE_ONE = 1;
}

message SampleUserData {
    int32 data = 1;
}

enum SampleRoomTypeEnum {
    ROOM_TYPE_NONE = 0;
    ROOM_TYPE_ONE = 1;
}

message SampleRoomData {
    int32 data = 1;
}

message SampleUserMessage {
    SampleUserTypeEnum sampleUserTypeEnum = 1;
    int32 id = 2;
    int64 money = 3;
    string nickname = 4;
    double score = 5;
    repeated SampleUserData sampleUserData = 6; // list
}

message SampleRoomMessage {
    SampleRoomTypeEnum sampleRoomTypeEnum = 1;
    int32 id = 2;
    int64 potMoney = 3;
    string roomName = 4;
    repeated SampleRoomData sampleRoomData = 5; // list
}
```

<br>

그리고 아래와 같은 명령줄로 작성한 프로토콜 스크립트를 컴파일할 수 있습니다. 이 때, build.bat와 같은 배치 파일을 생성해두면 편리합니다. 만일 GameAnvil 템플릿을 이용하여 프로젝트를 구성했다면 프로젝트 내의 proto 폴더에서 프로토콜 스크립트와 배치파일을 참고할 수 있습니다. 컴파일이 완료되면 하면 프로토콜 처리에 필요한 모든 Java 클래스가 자동으로 생성됩니다.

```
protoc  ./MyGame.proto --java_out=../java
```

<br>

## 9.Configurations

GameAnvil은 GameAnvilConfig.json 파일을 통해 서버 구성을 변경할 수 있습니다. 이러한 구성 파일은 총 7개의 항목으로 분류되며 common을 제외하면 각각 한 종류의 노드를 설정합니다. 이 때, 설정하는 값 중 일부는 Bootstrap 단계에서 사용하는 값과 일치해야 합니다. 예를 들면 앞서 우리는 아래와 같이 GameNode를 구성했습니다. 여기에 사용된 ServiceName인 "SampleGameService"는 반드시 GameAnvilConfig.json에 설정되어 있어야 합니다. 그렇지 않으면 해당 서비스를 찾지 못하여 서버 구동 과정에서 오류가 발생합니다.

```java
bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
```

<br>

### common

* 필수적인 공통 설정

| 이름 | 설명 | 기본값 |
| --- | --- | --- |
| vmId | 하나의 machine에서 여러개의 JVM을 띄울 경우 사용하는 고유식별 ID ( 0 <= vmId < 100) | 0 |
| ip | 노드마다 공통으로 사용하는 IP. (머신의 IP를 지정)<br>값이 없을 경우 private ip로 자동 지정된다. |  |
| meetEndPoints | 대상 노드의 common IP와 communicatePort 등록<br>해당 서버 endpoint 포함가능<br>리스트로 여러개 등록 가능 |  |
| keepAliveTime | communicateNode 연결 지속 시간 (ms) | 60000 |
| nodeTimeout | 노드 상태 체크 타임아웃(ms)0이면 체크하지 않음. | 180000 |
| msgExpiredTime | 메세지 만료 시간 | 30000 |
| defaultReqTimeout | 노드간 메세지 라우팅 타임아웃(ms)0이면 체크하지 않음. | 30000 |
| defaultAsyncAwaitTimeout | Blocking 호출 시 타임아웃(ms) | 30000 |
| ipcPort | 다른 ipc node 와 통신할때 사용되는 port | 11100 |
| publisherPort | publish socket 을 위한 port | 11101 |
| debugMode | 디버깅시 각종 timeout 이 발생안하도록 하는 옵션리얼에서는 반드시 false 이어야 한다. | false |
| nodeInfoUpdateTime | JVM에 있는 노드 정보를 다른 JVM에게 전파하는 주기 | 500(ms) |
| nodeInfoUpdateBusyTime | 노드 정보가 업데이트 될 때 이 시간보다 오래 걸렸을 경우 해당 노드는 BUSY 상태라고 판단함. | 510(ms) |
| nodeInfoManagerRefreshTime | 전달 받은 노드정보를 이용해 NodeInfoManager를 갱신하는 주기. | 100(ms) |
| loopInfoSamplingCount | Message Loop의 상태를 체크할 때 Sampling할 loop 횟수 | 100(회) |
| enableLoopInfoMonitor | NodeInfoPage에서 loop 정보 갱신 여부 | true |
| maxUserCount | IdleNode 판단시 idle로 판단할 최대 User 수 | 2500 |
| addLocWeight | UserLoc 할당시 정보 보정을 위해 사용할 배수.<br>ex) 게임노드가 8개일 경우 1명이 할당 되었지만 8명의 유저가 할당되었다고 임의로 지정. | 1 |
| allocatorType | 사용할 bytebuf type<br>1 (POOLED\_DIRECT\_BUFFER)<br>2 (POOLED\_HEAP\_BUFFER)<br>3 (UNPOOLED\_DIRECT\_BUFFER)<br>4 (UNPOOLED\_HEAP\_BUFFER) | 4 |
| httpClientOption | Gameflex에서 사용하는 httpClientOption |  |
| httpClientOption-keepAlive | httpClientOption 연결 지속 여부 | true |
| httpClientOption-connTimeout | httpClientOption  connection 타임아웃 | 5000 |
| httpClientOption-requestTimeout | httpClientOption   request 타임아웃 | 10000 |
| httpClientOption-maxConnPerHost | httpClientOption host당 최대 connection 개수 | 10000 |
| httpClientOption-maxConn | httpClientOption connection 최대 개수 | 12000 |
| moduleURL | config, dynamicModule 정보를 읽어올 URL 정보<br>해당 옵션이 설정될 경우 URL을 사용해서 module 정보를 불러온다. | "" |
| configPath | config를 File로 읽어올 시 config의 Json 파일을 읽어올 경로<br>moduleURL이 설정될 경우 무시 | /config |

* 사용 예시

```json
"common": {
    "ip": "127.0.0.1",
    "meetEndPoints": ["127.0.0.1:16000"],
    "keepAliveTime": 10000,
    "nodeTimeout": 180000,
    "msgExpiredTime:":30000,
    "defaultReqTimeout": 30000,
    "defaultAsyncAwaitTimeout": 30000,
    "communicatePort": 16000,
    "publisherPort" : 13300,
    "debugMode": false,
    "allocatorType": 1,
    "nodeInfoUpdateTime": 10,
    "nodeInfoUpdateBusyTime": 12,
    "nodeInfoManagerRefreshTime": 4,
    "loopInfoSamplingCount": 5,
    "maxLoopTime" : 10,
    "enableLoopInfoMonitor" : true,
    "maxUserCount" : 2500,
    "addLocWeight": 16,
    "httpClientOption": {
        "keepAlive": true,
        "connTimeout": 5000,
        "requestTimeout": 10000,
        "maxConnPerHost" : 10000,
        "maxConn": 12000
    }
},
```

<br>

### location

* 설정값이 없거나 clusterSize가 0이면 LocationNode를 생성하지 않습니다.

| 이름 | 설명 | 기본값 |
| --- | --- | --- |
| clusterSize | 구성되는 서버의 수(VM) |  |
| replicaSize | 복제 그룹의 크기<br>master + slave의 개수 |  |
| shardFactor | sharding을 위한 인수<br>-전체 shard의 개수 = clusterSize x replicaSize x shardFactor<br>-하나의 머신(VM)에서 구동할 shard의 개수 = replicaSize x shardFactor<br>-고유한 shard의 총 개수 (master 샤드의 개수) = clusterSize x shardFactor |  |

* 사용 예시

```json
"location": {
    "clusterSize": 1,
    "replicaSize": 3,
    "shardFactor": 3
},
```

<br>

### match

* 설정값이 없거나 nodeCnt가 0이면 MatchNode를 생성하지 않습니다.

| 이름 | 설명 | 기본값 |
| --- | --- | --- |
| nodeCnt | match node 개수 |  |

* 사용 예시

```json
"match": {
    "nodeCnt": 3
},
```

<br>

### gateway

* 설정값이 없거나 nodeCnt가 0이면 GatewayNode를 생성하지 않습니다.

| 이름 | 설명 | 기본값 |
| --- | --- | --- |
| nodeCnt | 노드 개수 |  |
| ip | 클라이언트와 연결되는 IP<br>설정값이 없을 경우 private ip로 자동 설정 |  |
| dns | 클라이언트와 연결되는 도메인 주소 |  |
| maintenance | 서버 시작 시 점검 여부 | false |
| checkWhiteListBeforeOnAuth | onAuthenticate 함수 호출 전 WhiteList 체크 여부 | true |
| connectGroup | TCP\_SOCKET<br>WEB\_SOCKET |  |
| connectGroup - port | 클라이언트와 연결되는 포트 |  |
| connectGroup  - idleClientTimeout | 데이터 송수신이 없는 상태 이후의 타임아웃.0 이면 사용하지 않음 | <span style="color:  #6897bb;;">4000</span> |
| connectGroup  - secure | 보안 설정<br>설정값이 없을 경우 사용하지 않음. |  |
| connectGroup  - secure - useSelf | keyCertChainPath, privateKeyPath 설정값과 상관없이 보안 설정 사용여부 | false |
| connectGroup  - secure - keyCertChainPath | 인증서 경로<br>-Dsecure 설정 경로에서 시작됨 |  |
| connectGroup  - secure - privateKeyPath | 개인키 경로<br>-Dsecure 설정 경로에서 시작됨 |  |
| duplicateLoginServices | 중복 로그인 가능 서비스 지정 |  |

* 사용 예시

```json
"gateway": {
    "nodeCnt": 4,
    "ip": "127.0.0.1",
    "dns": "",
    "maintenance": false,
    "checkWhiteListBeforeOnAuth": true,
    "connectGroup": {
        "TCP_SOCKET": {
            "port": 11200,
            "idleClientTimeout": 0,
            "secure": {
                "useSelf": true,
                "keyCertChainPath": "gameflex.crt",
                "privateKeyPath": "privatekey.pem"
            }
        },
        "WEB_SOCKET": {
            "port": 11400,
            "idleClientTimeout": 0
        }
    },
    "duplicateLoginServices": {
        "RPSGame": ["GroupChatting"],
        "GroupChatting": ["SampleGameService"],
        "ForTestCase": []
    }
},
```

<br>

### game

* 설정값이 없거나 nodeCnt가 0이면 GameNode를 생성하지 않습니다. 앞서 설명했듯이 엔진의 Bootstrap 단계에서 사용한 ServiceName을 이 곳에 설정해야 합니다.

| 이름 | 설명 | 기본값 |
| --- | --- | --- |
| nodeCnt | 노드 개수 |  |
| serviceId | Service ID |  |
| serviceName | Service 이름 |  |
| channelIDs | 노드마다 부여할 채널 ID.<br>유니크하지 않아도 됨.<br>"" 문자열로 채널 구분없이 중복사용도 가능 |  |
| userTimeout | disconnect 이후의 유저객체 제거 타임아웃. | <span style="color:  #6897bb;;">10000</span> |

* 사용 예시

```json
"game": [
    {
        "nodeCnt": 8,
        "serviceId": 0,
        "serviceName": "SampleGameService",
        "channelIDs": ["ch1","ch1","ch2","ch2","ch3","ch3","ch4","ch4"],
        "userTimeout": 3000 
    },
    {
        "nodeCnt": 4,
        "serviceId": 1,
        "serviceName": "GroupChatting",
        "channelIDs": ["", "", "", ""],
        "userTimeout": 4000
    }
],
```

<br>

### support

* 설정값이 없거나 nodeCnt가 0이면 SupportNode를 생성하지 않습니다. SupportNode도 GameNode와 마찬가지로 Bootstrap 단계에서 사용한 ServiceName이 반드시 설정되어야 합니다.

| 이름 | 설명 | 기본값 |
| --- | --- | --- |
| nodeCnt | 노드 개수 |  |
| serviceId | service ID |  |
| serviceName | service 이름 |  |
| restIp | rest ip<br>설정값이 없을 경우 private ip로 자동 설정 |  |
| restPort | rest port |  |
| restPermissionGroups | ACL 설정<br>허용할 특정 Path와 IP를 등록.<br>비어있을 경우 모든 접속 허용 |  |
| restSecure | 설정값이 없을 경우 사용하지 않음. |  |
| restSecure - keyCertChainPath | 인증서 경로<br>-Dsecure 설정 경로에서 시작됨 |  |
| restSecure - privateKeyPath | 개인키 경로<br>-Dsecure 설정 경로에서 시작됨 |  |
| restSecure - authorizationSecret | 인증토큰(JWT) 사용하는 경우만 필요.<br>암호화 시그니쳐 키. |  |
| restSecure - authorizationPath | 인증토큰(JWT) 사용하는 경우만 필요.<br>인증을 진행할 경로. |  |
| restCorsAllowDomains | CORS 허용 domain<br>예-127.0.0.1:1234 |  |

* 사용 예시

```json
"support": [
    {
        "nodeCnt": 2,
        "serviceId": 3,
        "serviceName": "PLACE",
        "restIp": "127.0.0.1",
        "restPort": 17260,
        "restPermissionGroups": {
            "/": []
        },
        "restSecure": {
            "keyCertChainPath": "",
            "privateKeyPath": "",
            "authorizationSecret": "authSecret",
            "authorizationPath": "/auth"
        },
        "restCorsAllowDomains":[
        ]
    },
    {
        "nodeCnt": 2,
        "serviceId": 4,
        "serviceName": "DB",
        "restIp": "127.0.0.1",
        "restPort": 17251,
        "restPermissionGroups": {
            "/": []
        }
    },
    {
        "nodeCnt": 2,
        "serviceId": 5,
        "serviceName": "SampleWebSupport",
        "restIp": "127.0.0.1",
        "restPort": 17250
    }
],
```

<br>

### management

* 설정값이 없거나 nodeCnt가 0이면  ManagementNode를 생성하지 않습니다.

| 이름 | 설명 | 기본값 |
| --- | --- | --- |
| nodeCnt | 노드 개수 |  |
| restIp | rest ip<br>설정값이 없을 경우 private ip로 자동 설정 |  |
| restPort | rest port |  |
| db | management에서 사용하는 db 설정 |  |
| db - user | db 접속 ID |  |
| db - password | db 접속 비밀번호 |  |
| db - url | db 접속 url<br>\- h2db : jdbc:h2:mem:gameflex\_admin;DB\_CLOSE\_DELAY=\-1<br>\- mysql : jdbc:mysql://localhost:3306/gameflex\_admin?useSSL=false<br>\- mariadb : jdbc:mariadb://localhost:3306/gameflex\_admin?useSSL=false |  |
| db - driver | db 접속 driver- h2db : <span style="color:  #6a8759;;">org.h2.Driver</span>\- mysql : com\.mysql\.jdbc\.Driver\- mariadb : org\.hibernate\.dialect\.MySQLDialect | <span style="color:  #6a8759;;">org.h2.Driver</span> |
| db - dialect | db 접속 dialect- h2db : <span style="color:  #6a8759;;">org.hibernate.dialect.H2Dialect</span>\- mysql : org\.hibernate\.dialect\.MySQLDialect\- mariadb : org\.hibernate\.dialect\.MySQLDialect | <span style="color:  #6a8759;;">org.hibernate.dialect.H2Dialect</span> |
| db - ddlAuto | ddl 옵션<br>\- update : 자동으로 Table 생성<br>\- none : 자동으로 Table을 생성하지 않음\. | update |

* 사용 예시

```json
"management": {
    "nodeCnt": 2,
    "restIp": "127.0.0.1",
    "restPort": 25150,

    "db": {
        "user": "root",
        "password": "1234",
        "url": "jdbc:h2:mem:gameflex_admin;DB_CLOSE_DELAY=-1"
    }
}
```

