## Game > GameAnvil > 서버 개발 가이드 > 구현 가이드

기본적인 서버를 구현하기 위해 알아야 할 엔진의 핵심적인 부분을 중심으로, 기본 요소들과 이를 구현하는 방법에 대해 설명합니다. 코드 레벨의 자료는 [GameAnvil 레퍼런스 서버](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-reference-server)를 참고합니다. 또한, [GameAnvil API Reference](http://10.162.4.61:9090/gameanvil)를 통해 JavaDoc 문서도 이용 가능합니다. 가능하다면 레퍼런스 서버 프로젝트와 이 가이드 문서를 함께 살펴보시기를 추천드립니다.



### Note

*GameAnvil은 엔진 내부는 물론 모든 샘플 코드에서 메시지 핸들러에 대해 _로 시작하는 네이밍을 사용합니다. 즉, 앞으로 등장하는 예제 코드에서도 _로 시작하는 모든 클래스는 메시지 핸들러입니다. 이러한 메시지 핸들러에 대한 구현은 이 글에서 별도로 설명하지 않습니다. 앞서 알려드린 레퍼런스 서버 프로젝트나 JavaDoc을 통해 코드를 확인하는 것이 가장 쉽고 정확한 방법입니다.*



GameAnvil 대부분의 기능은 콜백 방식으로 제공합니다. 즉, 엔진 사용자는 **GameAnvil이 제공하는 기본 클래스(Base Classes)를 상속한 후 콜백 메서드를 재정의**하는 형태로 대부분의 기능을 사용하게 됩니다. 이 과정에서 엔진 사용자가 필요한 콜백 메서드만 구현하면 되므로 일부 콜백 메서드는 무시할 수도 있습니다. 이 문서는 이러한 콜백 메서드의 실제 구현 방법까지는 안내하지 않습니다. 코드 레벨의 참고는 앞서 설명한 레퍼런스 서버 프로젝트를 참고하는 것이 가장 좋은 방법입니다.



## 1. Node 공통

모든 종류의 노드는 공통적으로 아래의 콜백 메서드들을 재정의해야 합니다. 그와 더불어 각 노드는 용도에 맞춰 추가 콜백 메서드를 구현하게 됩니다. 아래 코드의 GatewayNode는 추가 콜백이 없으므로 공통 콜백 메서드를 보여주기에 적합합니다. 특히, onPrepare()에서 호출한 setReady() API는 엔진에게 노드가 모두 준비되었으니 실제 구동을 시작하라는 명령입니다. 일반적으로 onPrepare()에서 호출합니다. 하지만 다른 노드와의 통신을 통하거나 임의의 비동기 결과를 바탕으로 준비 명령을 내리고 싶을 경우에는 다른 코드 블록에서 호출해도 무방합니다.

```
public class SampleGatewayNode extends BaseGatewayNode {

    /**
     * 초기화할 때 발생.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onInit() throws SuspendExecution {        
    }

    /**
     * Prepare 시에 호출된다.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPrepare() throws SuspendExecution {
        setReady(); // 주의!
    }

    /**
     * Ready 단계에서 호출된다.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onReady() throws SuspendExecution {        
    }

    /**
     * 패킷이 전달될 때 호출된다.
     *
     * @param packet 처리할 패킷
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {        
    }

    /**
     * pause 될때에 호출된다.
     *
     * @param payload contents 에서 전달하고자 하는 추가 정보
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPause(Payload payload) throws SuspendExecution {        
    }

    /**
     * resume 될때에 호출된다.
     *
     * @param payload contents에서 전달하고자 하는 추가 정보
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onResume(Payload payload) throws SuspendExecution {        
    }

    /**
     * NonNetworkNode 종료를 위해 shutdown 명령시 호출된다.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onShutdown() throws SuspendExecution {        
    }
}
```



## 2. Gateway Node

GatewayNode는 클라이언트가 접속하는 노드입니다. BaseGatewayNode 클래스를 상속하여 공통 콜백 메서드만 재정의하면 됩니다. 그 외 GatewayNode 자체에 특별히 추가로 필수 구현할 사항은 없습니다.  GatewayNode는 클라이언트로부터의 커넥션과 세션들을 관리하는데 이들의 관계는 다음의 그림과 같습니다.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

일반적으로 클라이언트는 GatewayNode로 하나의 커넥션을 맺습니다. 이때, 해당 커넥션에 대해 인증 절차를 진행하여 성공한 경우에 한해 하나 이상의 세션을 생성할 수 있습니다. 각각의 세션은 서로 다른 게임 서비스에 대한 논리적 연결 단위입니다. 위의 그림은 클라이언트가 하나의 커넥션을 통해 Game 서비스와 Chat 서비스 각각에 대해 세션을 생성한 모습입니다.



### 2-1. 커넥션 (Connection)

커넥션은 클라이언트의 물리적 접속 자체를 의미합니다. 클라이언트는 고유한 AccountId를 이용하여 커넥션 상에서 인증 절차를 진행할 수 있습니다. 인증이 성공할 경우 해당 AccountId는 생성된 커넥션에 매핑됩니다. 커넥션은 아래와 같은 콜백 메서드를 재정의 합니다. 이때, AccountId는 임의의 플랫폼에서 인증한 후 획득하는 유저의 키값 등을 사용할 수 있습니다. 예를 들어 Gamebase를 통해 인증한 후 UserId를 획득하면 이 값을 GameAnvil의 인증 과정에서 AccountId로 사용할 수 있습니다.

```
public class SampleConnection extends BaseConnection<SampleGameSession> {

    /**
     * authenticate 시에 호출된다.
     *
     * @param accountId  계정의 아이디
     * @param password   계정 패스워드
     * @param deviceId   클라이언트의 기기 아이디
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return false 시 클라이언트와의 접속이 종료
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onAuthenticate(final String accountId, final String password, final String deviceId, final Payload payload, final Payload outPayload) throws SuspendExecution {        
    }

    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {        
    }

    /**
     * login 호출 이전에 호출된다.
     *
     * @param outPayload 클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPreLogin(Payload outPayload) throws SuspendExecution {        
    }

    /**
     * 로그인 성공 이후에 호출된다.
     *
     * @param session 로그인에 사용한 세션 객체
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPostLogin(U session) throws SuspendExecution {        
    }

    /**
     * 로그아웃 이후에 호출된다.
     *
     * @param session 게임 노드의 유저에 연결된 세션 객체
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
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
     * 클라이언트와 연결이 끊어졌을 때 호출된다.
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    public void onDisconnect() throws SuspendExecution {        
    }
}
```



### 2-2. 세션 (Session)

커넥션을 성공적으로 맺은 클라이언트는 해당 커넥션 상에서 서비스별로 한 개씩 GameNode로 논리적인 세션을 맺을 수 있습니다. GameAnvil은 내부적으로 커넥션의 AccountId와 세션의 SubId를 "AccoundId:SubId" 형태로 조합해서 고유한 세션과 해당 세션을 사용 중인 유저를 구분합니다. 이러한 세션은 아래와 같이 해당 세션에서 처리할 메시지와 핸들러를 등록한 후 콜백 메서드를 통해 디스패치할 수 있습니다. 이때, subId는 사용자가 임의로 정한 규칙에 맞추어 해당 커넥션 내에서 고유한 값으로 할당하면 됩니다.

```
public class SampleSession extends BaseSession {

    private static PacketDispatcher dispatcher = new PacketDispatcher();

    static {
        dispatcher.registerMsg(SampleGame.MsgToSessionReq.getDescriptor(), _MsgToSessionHandler.class);
    }

    /**
     * 세션으로 패킷이 전달될 때 호출된다.
     *
     * @param packet 처리할 패킷
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        dispatcher.dispatch(this, packet);
    }
}
```



### 2-3. Bootstrap

이렇게 생성한 GatewayNode와 커넥션 그리고 세션은 Bootstrap 단계에서 아래와 같이 등록할 수 있습니다.

```
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



## 3. Game Node

GameNode는 실제 게임 객체가 생성되고 게임 콘텐츠를 구현하는 노드입니다. GameNode는 기본적으로 BaseGameNode 추상 클래스를 상속 구현합니다. GameNode가 관리하는 대표적인 객체는 GameUser와 GameRoom입니다. 이에 관해서는 아래에서 더욱 자세하게 살펴봅니다. 아래 코드는 GameNode에서 기본적으로 재정의할 수 있는 콜백 메서드를 보여줍니다. 노드 공통 콜백에 채널 간 동기화를 위한 콜백이 추가되었습니다. 또한 GameNode에서 처리하고 싶은 프로토콜은 임의의 핸들러와 매핑한 후 onDispatch() 콜백을 통해 해당 패킷을 처리할 수 있습니다. 아래 코드에서 1~3에 해당하는 주석 아래 코드가 이를 위한 처리 흐름을 순서대로 보여줍니다.



```
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
     * 패킷이 전달될 때 호출된다.
     *
     * @param packet 처리할 패킷
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
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
     * 같은 채널의 다른 node 에 유저 변화가 있을때 호출되는 콜백
     * <p>
     * updateChannelUser() 호출시 발생.
     *
     * @param type            채널 정보 변경 타입(갱신/삭제)
     * @param channelUserInfo 변경될 유저 정보
     * @param userId          변경 대상의 유저 Id
     * @param accountId       변경 대상의 Account Id
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onChannelUserUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {        
    }

    /**
     * 같은 채널의 다른 node 에 room 상태 변화가 있을때 호출되는 콜백
     * <p>
     * updateChannelRoomInfo() 호출 시 발생.
     *
     * @param type            채널 정보 변경 타입(갱신/삭제)
     * @param channelRoomInfo 변경될 Room 정보
     * @param roomId          변경 대상의 Room Id
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onChannelRoomUpdate(ChannelUpdateType type, RoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {        
    }

    /**
     * 클라이언트에서 채널 정보를 요청 시 호출되는 콜백 (Base.GetChannelInfoReq)
     *
     * @param outPayload 클라이언트로 전달될 Channel 정보
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {        
    }
}
```



### 3-1. GameUser

게임상의 유저를 나타내는 GameUser 객체는 로그인 과정을 거쳐 GameNode에 생성됩니다. 유저 단위의 모든 메시지는 이곳에서 처리할 수 있습니다. 그러므로 유저 관련 콘텐츠는 이 클래스를 중심으로 구현하는 것이 바람직합니다. 앞서 살펴본 모든 예제와 마찬가지로 GameUser 또한 처리할 프로토콜과 핸들러를 매핑할 수 있습니다.

아래의 예제 코드는 GameUser의 전체 콜백 메서드 목록을 보여줍니다. 이 중 일부는 기본 구현이 제공되므로 필요한 상황이 아니면 재정의할 필요는 없습니다. 이는 GameUser뿐만 아니라 엔진에서 제공하는 대부분의 콜백 메서드에 해당하는 사항입니다. 이러한 부분은 상속받는 각각의 기본 클래스(e.g.BaseUser)에 대해  [GameAnvil API Reference](http://10.162.4.61:9090/gameanvil)에서 JavaDoc 문서를 확인하거나 [레퍼런스 서버 프로젝트](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-reference-server)를 참고하는 것이 좋습니다.

아래의 콜백 메서드 중 onLogin()은 최초에 게임 유저를 생성하기 위해 로그인을 시도하는 과정에서 호출됩니다. 이 onLogin() 콜백이 성공하면 GameNode 상에 해당 게임 유저 객체가 생성됩니다. 로그인이 완료된 후에는 사용자가 작성한 프로토콜을 기반으로 클라이언트와 직접 메시지를 주고 받으며 여러 가지 콘텐츠를 처리할 수 있습니다.

```
public class SampleGameUser extends BaseUser {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameUserTest.getDescriptor(), _GameUserTest.class);
    }

    /**
     * login 시 발생하는 콜백
     *
     * @param payload        클라이언트에서 전달한 임의의 페이로드
     * @param sessionPayload onPreLogin에서 전달된 페이로드
     * @param outPayload     클라이언트로 전달할 임의의 페이로드
     * @return boolean type으로 true: 로그인 성공. false: 로그인 실패 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onLogin(final Payload payload,
                           final Payload sessionPayload,
                           Payload outPayload) throws SuspendExecution {        
    }

    /**
     * Login 처리 이후에 발생하는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPostLogin() throws SuspendExecution {

    }

    /**
     * 한 아이디로 유저가 로그인된 상황에서 다른 디바이스에서 같은 유저가 로그인할 때 호출되는 콜백
     *
     * @param newDeviceId           새로 접속한 유저의 deviceId 값
     * @param outPayloadForKickUser 클라이언트로 전달할 페이로드
     * @return boolean type으로 true:새로 접속한 유저가 login success, 현재 유저는 forceLogout. false: 새로 접속한 유저가 login fail을 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다/
     */
    @Override
    public boolean onLoginByOtherDevice(final String newDeviceId,
                                        Payload outPayloadForKickUser) throws SuspendExecution {
        return true;
    }

    /**
     * MultiType 유저를 사용할 때 어떤 type의 유저가 login된 상황에서 다른 type의 유저가 login할 때 호출되는 콜백
     *
     * @param userType   새로 로그인을 시도하는 유저의 타입
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return boolean type으로 true: 새 로그인 성공. false: 새 로그인 실패를 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onLoginByOtherUserType(final String userType,
                                          Payload outPayload) throws SuspendExecution {
        return true;
    }

    /**
     * Login 한 상황에서 동일 device 동일 id로 login할 때 호출되는 콜백
     *
     * @param outPayload 클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {        
    }

    /**
     * 서버에 유저 객체가 살아 있는 상태에서 login 이 호출되었을 경우 호출되는 콜백
     *
     * @param payload        클라이언트에서 전달한 임의의 페이로드
     * @param sessionPayload onPreLogin에서 전달된 페이로드
     * @param outPayload     클라이언트로 전달할 임의의 페이로드
     * @return boolean type으로 true: ReLogin 성공, false: ReLogin 실패를 반환.
     * @throws SuspendExecution: 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onReLogin(final Payload payload,
                             final Payload sessionPayload,
                             Payload outPayload) throws SuspendExecution {
    }

    /**
     * 클라이언트와의 연결이 끊어졌을때 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onDisconnect() throws SuspendExecution {        
    }

    /**
     * 유저로 전달되는 패킷이 있을 때 호출되는 콜백
     *
     * @param packet 처리할 패킷
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, packet);
    }

    /**
     * node가 pause될 때 node가 관리하고 있는 유저에게 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPause() throws SuspendExecution {        
    }

    /**
     * node가 resume될 때 node가 관리하고 있는 유저에게 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onResume() throws SuspendExecution {        
    }

    /**
     * 유저가 logout 시 호출되는 콜백
     *
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onLogout(final Payload payload, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * 유저가 로그아웃 가능한지 확인하는 콜백.
     *
     * @return boolean type으로 반환: false일 경우 로그아웃이 안되며 이 후에 주기적으로 다시 확인을 한다. : true일 경우 유저는 로그아웃이 된다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean canLogout() throws SuspendExecution {        
    }

    /**
     * 클라이언트에서 룸 매이메이킹를 요청했을 경우 발생하는 콜백
     *
     * @param roomType 매칭되는 룸의 타입
     * @param payload  클라이언트로부터 전달받은 페이로드
     * @return 매칭된 룸의 정보 반환, null을 반환할시 클라이언트의 요청 옵션에 따라 새로운 룸이 생성될 수도 있고 실패 처리될 수도 있음
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public MatchRoomResult onMatchRoom(final String roomType,
                                       final Payload payload) throws SuspendExecution {
        return null;
    }

    /**
     * 클라이언트에서 유저 매치메이킹을 요청했을 경우 호출되는 콜백
     *
     * @param roomType   매칭되는 룸의 타입
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return true: 유저 매치메이킹 요청 성공, false: 유저 매치메이킹 요청 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onMatchUser(final String roomType,
                               final Payload payload,
                               Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * 클라이언트에서 userMatch가 취소되었을 때 호출되는 콜백
     *
     * @param reason 취소된 이유(TIMEOUT/CANCEL)
     * @return boolean type으로 반환. true: 유저 매치메이킹 취소 성공, false: 유저 매치메이킹 취소 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
        return false;
    }

    /**
     * 타이머 핸들러가 등록되고 호출되는 콜백
     */
    @Override
    public void onRegisterTimerHandler() {        
    }

    /**     
	 * 유저가 다른 노드로 이동할 때 임의의 데이터를 전달하기 위해서 호출되는 콜백
     *
     * @param transferPack 필요한 데이터를 설정할 객체
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onTransferOut(final TransferPack transferPack) throws SuspendExecution {        
    }

    /**
     * 유저가 다른 node로 이동한 후 user define을 복구하기 위해 호출되는 콜백
     * <p>
     * 이 시점에는 유저가 완전히 복구되기 전이기 때문에 다른 곳으로 request 호출이 제한된다.
     *
     * @param transferPack 이동 전 전달되었던 객체. 해당 객체에서 설정값을 가져온다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onTransferIn(final TransferPack transferPack) throws SuspendExecution {        
    }

    /**
     * 유저의 node 간 transfer가 완료된 후 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPostTransferIn() throws SuspendExecution {        
    }

    /**
     * 클라이언트에서 snapshot 요청 시 호출되는 콜백
     * <p>
     * 주로 접속이 끊어지고  서버 상태가 변할 확률이 있을 경우 호출해서 클라이언트와 서버의 상태정보를 동기화하는 데 사용된다.
     *
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onSnapshot(final Payload payload, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * 클라이언트에서 user에게 다른 채널로 이동 요청을 할 경우 호출되는 콜백
     *
     * @param destinationChannelId 이동하려고 하는 대상 channel ID
     * @param payload              클라이언트로부터 전달받은 페이로드
     * @param errorPayload         channel 이동 실패 시 서버에서 클라이언트로 전달하는 페이로드. 성공일 경우 전달 안 됨.
     * @return false: 채널 이동 요청 실패, true: 채널 이동 요청 성공.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onCheckMoveOutChannel(final String destinationChannelId,
                                         final Payload payload,
                                         Payload errorPayload) throws SuspendExecution {
        return false;
    }

    /**
     * 클라이언트에서 채널 이동 요청이 들어온 경우  onCheckMoveOutChannel()에서 true 반환한 경우 호출되는 콜백
     * <p>
     * 채널 이동 시 이동할 채널로 정보를 전달하기 위해서 호출 서버의 moveChannel() 호출 시 onCheckMoveOutChannel()가 호출이 안 되고 바로 호출이 된다.
     *
     * @param destinationChannelId 이동할 channel ID
     * @param outPayload           이동할 channel에 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onMoveOutChannel(final String destinationChannelId,
                                 Payload outPayload) throws SuspendExecution {
    }

    /**
     * onMoveOutChannel() 호출후 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPostMoveOutChannel() throws SuspendExecution {
    }

    /**
     * 이동한 채널로 입장할 때 발생하는 콜백
     *
     * @param sourceChannelId 이동하기 전 channel ID
     * @param payload         클라이언트로부터 전달받은 페이로드
     * @param outPayload      클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onMoveInChannel(final String sourceChannelId,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * onMoveInChannel() 수행 후 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPostMoveInChannel() throws SuspendExecution {
    }

    /**
     * User Transfe 가 가능한지 확인.
     *
     * @return boolean type으로 반환. true: Transfer 가능, false: Transfer 불가능, Transfer가 불가능일 경우 NonStopPatch가 종료되기 전까진 지속적으로 호출된다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean canTransfer() throws SuspendExecution;
}
```



### 3-2. GameRoom

2명 이상의 게임 유저는 GameRoom을 통해 동기화된 메시지 흐름을 만들 수 있습니다. 즉, 게임 유저들의 요청은 GameRoom 내에서 모두 순서가 보장됩니다. 물론 1명의 게임 유저를 위한 GameRoom 생성도 콘텐츠에 따라서 의미를 가질 수도 있습니다. GameRoom을 어떻게 사용할지는 어디까지나 엔진 사용자의 몫입니다. 이러한 GameRoom은 GameUser와 마찬가지로 기본 클래스(BaseRoom)을 바탕으로 여러 가지 자체 콜백 메서드를 재정의할 수 있으며 자체적으로 프로토콜을 처리할 수도 있습니다. 아래의 예제 코드는 SampleGameUser를 위한 SampleGameRoom 클래스입니다.

```
public class SampleGameRoom extends BaseRoom<SampleGameUser> {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameRoomTest.getDescriptor(), _GameRoomTest.class);
    }

    /**
     * Room이 초기화될 때 발생하는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onInit() throws SuspendExecution {        
    }

    /**
     * Room 이 삭제될 때 발생하는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onDestroy() throws SuspendExecution {        
    }

    /**
     * Room으로 {@link Packet}이 전달될 때 호출되는 콜백
     *
     * @param user   해당 패킷을 처리할 유저 객체. 해당 패킷을 처리할 유저가 없을 경우 null.
     * @param packet 처리할 패킷
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onDispatch(SampleGameUser user, final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, user, packet);
    }

    /**
     * 클라이언트에서 방 생성을 요청할 때 발생하는 콜백
     *
     * @param user       방 생성을 요청한 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return true: 방 생성 성공, false: 방 생성 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onCreateRoom(SampleGameUser user,
                                final Payload inPayload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * 클라이언트에서 Room 입장 요청을 할 때 발생하는 콜백
     *
     * @param user       Room 입장을 요청하는 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return true: Room 입장 성공, false: Room 입장 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onJoinRoom(SampleGameUser user,
                              final Payload inPayload,
                              Payload outPayload) throws SuspendExecution {
    }

    /**
     * 클라이언트에서 Room 나가기 요청을 할 때 발생하는 콜백
     *
     * @param user       나가기 요청을 한 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return boolean type으로 반환. true Room 나가기 성공, false Room 나기기 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean onLeaveRoom(SampleGameUser user,
                               final Payload inPayload,
                               Payload outPayload) throws SuspendExecution {
    }

    /**
     * onLeaveRoom 호출 후 반환값이 true인 경우 호출되는 콜백
     *
     * @param user onLeaveRoom 처리가 끝난 유저 객체
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPostLeaveRoom(SampleGameUser user) throws SuspendExecution {
    }

    /**
     * 유저가 Room에 있는 상태에서 ReJoin될 경우 호출되는 콜백
     *
     * @param user       ReJoin을 한 유저 객체
     * @param outPayload 클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onRejoinRoom(SampleGameUser user, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * 타이머 핸들러가 등록되고 호출되는 콜백
     */
    @Override
    public void onRegisterTimerHandler() {
    }

    /**
     * room transfer 발생 시 room의 define을 serialize하기 위해 호출되는 콜백
     *
     * @param transferPack 이동 전 전달 되었던 정보 꾸러미. 해당 객체에서 정보를 가져온다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * room transfer 발생 시 room의 define을 복원하기 위해서 호출되는 콜백
     *
     * @param userList     해당 Room에 있는 유저 객체 리스트.
     * @param transferPack 복원할 Room 정보 꾸러미
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onTransferIn(List<SampleGameUser> userList,
                             final TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * node가 pause되고 난 후 각 room 객체에서 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onPause() throws SuspendExecution {
    }

    /**
     * node가 resume되고 난 후 각 room에서 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * 클라이언트에서 partyMatch 를 요청했을 경우 호출되는 콜백
     *
     * @param roomType   룸의 타입
     * @param user       파티 매칭을 요청한 방장
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return boolean type 으로 반환. true: user matching 요청 성공,false: user matching 요청 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
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
     * 최초 콜백 호출 후 NonNetworkNode 상태가 Ready 가 되기 전까지 지속적으로 체크.
     *
     * @return boolean type 으로 반환. true: room transfer 가능, false: room transfer 불가능.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    @Override
    public boolean canTransfer() throws SuspendExecution {        
    }
}
```



### 3-3. 부트스트랩 (Bootstrap)

이렇게 생성한 GameNode와 GameUser 그리고 GameRoom은 Bootstrap 단계에서 아래와 같이 등록할 수 있습니다.

```
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



## 4. 유저 전송 (UserTransfer)

게임 노드에는 게임 유저 객체가 생성됩니다. 이 게임 유저 객체는 게임 노드 사이에서 언제든지 전송될 수 있습니다. 예를 들면 1번 게임 노드의 유저 객체가 2번 게임 노드에 생성된 방으로 들어가는 과정에서 게임 노드 간 유저 객체 전송이 일어납니다. 이 기술은 GameAnvil의 가장 핵심이자 근간이 되며 유저 객체를 잘 직렬화/역직렬화해야 의도한 대로 동작합니다. 이러한 직렬화/역직렬화 대상은 GameAnvil 사용자가 직접 구현할 수 있습니다. 즉, 해당 게임 유저 객체의 어떤 값을 전송할 것인지 결정하는 것이죠.

이러한 유저 전송이 발생하는 경우는 크게 세 가지로 나눌 수 있습니다.

- 첫째, 매치 메이킹을 포함한 방 생성이나 방 참여 로직에 의해 유저 객체는 다른 게임 노드로 전송될 수 있습니다. 이때, 구현에 따라 다른 채널의 방으로 진입할 수도 있습니다.
- 둘째, 다른 채널로 이동할 때 발생합니다. 하나의 게임 노드는 하나의 채널에 속할 수 있습니다. 그러므로 채널을 이동하는 것은 게임 노드 사이의 이동을 의미합니다.
- 첫 번째 경우처럼 매치 메이킹 등을 이용해서 다른 채널의 방으로 진입할 경우에는 엔진 내부에서 묵시적으로 처리합니다.
- 클라이언트는 명시적으로 채널 이동을 요청할 수도 있습니다.
- 셋째, 임의의 게임 노드에 대해 무정지 점검(NonStopPatch)을 진행하면 해당 노드의 유저 객체들은 다른 유효한 게임 노드들로 분산되어 전송됩니다. 이 경우는 운영 측면에서 GameAnvil Console을 통해 명시적으로 명령을 내린 경우입니다.



### 4-1. 유저 전송 구현

실제 유저 전송은 GameAnvil이 내부적으로 조용하게 처리합니다. 이때, 클라이언트는 자신의 게임 유저 객체가 서버 사이에서 전송되는지 인지하지 못합니다. 즉, 다른 게임 노드의 방으로 들어가더라도 클라이언트는 단지 하나의 GameAnvil 서버군에서 임의의 방으로 들어간 것뿐이죠.

단, 다른 게임 노드로 유저 객체를 전송할 때 어떤 데이터를 함께 전송할지는 사용자가 지정할 수 있습니다. 간단히 지정만 해두면 이 데이터는 유저 객체의 일부로 함께 직렬화됩니다. 전송할 대상 게임 노드에서는 역직렬화를 신경 쓸 필요 없이 쉽게 해당 객체들에 접근해서 사용할 수 있습니다.

다음은 이러한 유저 전송에 관련된 콜백 메서드들입니다. 우선 '함께 전송할 데이터 지정하기'입니다. 이를 위해 BaseUser 추상 클래스를 상속받은 게임 유저 클래스에 아래의 콜백을 구현합니다. 방법은 간단합니다. 아래의 예제와 같이 전송할 데이터를 사용자가 원하는 key 값을 이용하여 key-value 쌍으로 매개변수로 넘겨받은 transferPack에 넣기만 하면 됩니다.

만일 여기에서 전송할 데이터를 지정하지 않으면 대상 게임 노드로 전송된 유저 객체의 해당 데이터는 모두 기본값으로 초기화되므로 주의가 필요합니다.

```
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("TotalMoney", myTotalMoney);
    transferPack.put("Message", myMessage);
}
```

이제 유저 전송이 완료된 후 대상 게임 노드에서 처리할 콜백 메서드를 살펴보겠습니다. 아래와 같이 전송 전에 지정한 key를 이용해서 원하는 객체에 접근할 수 있습니다. 이와 같은 방법으로 전송 완료된 유저 객체의 해당 데이터를 원래 상태로 만들어 줍니다.

```
@Override
public void onTransferIn(TransferPack transferPack) throws SuspendExecution {
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setTotalMoney(transferPack.getToLong("TotalMoney"));
    setTransferMessage(transferPack.getToString("Message"));
}
```

위의 2가지 메서드는 GameAnvil이 알아서 호출합니다. 사용자는 그냥 구현만 하면 됩니다.



### 4-2. 전송 가능한 유저 타이머

유저가 전송될 때 유저에 등록해둔 타이머도 함께 전송할 수 있습니다. 전송 가능한 타이머를 사용하기 위해서는 별도로 아래의 콜백 메서드에서 원하는 key를 이용해서 미리 등록해두어야 합니다. Timer 핸들러를 다음과 같이 원하는 key로 매핑해둡니다.

```
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferUserTimerHandler1", transferUserTimerHandler1());
     registerTimerHandler("transferUserTimerHandler2", (timerObject, object) -> {
         GameUser user = (GameUser) object;
         logger.warn("GameUser::transferUserTimerHandler2() : userId({})", user.getId());
     });
 }
```

등록이 완료된 타이머 객체는 언제든 해당 key를 이용해서 아래와 같이 게임 유저 구현부에서 사용할 수 있습니다. 단순히 등록만 한 타이머는 효과가 없으므로 실제 사용을 위해서는 반드시 아래와 같이 유저 객체에 추가해야 타이머가 발동합니다.

```
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

이렇게 유저 객체에 추가해 둔 타이머는 별도로 전송에 대한 처리를 하지 않아도 모두 자동으로 전송됩니다. 즉, 대상 게임 노드에서 해당 유저 객체에 대해 동일한 타이머 추가 과정을 거칠 필요가 없습니다.



## 5. 아이디 (ID)

GameAnvil은 여러 종류의 아이디를 사용합니다. 그중 일부는 서버가 자체 발급하고 다른 일부는 사용자가 GameAnvilConfig에 직접 설정합니다. 접속에 필요한 계정 정보 등은 클라이언트에서 입력받은 정보를 서버로 전달합니다. 다음은 GameAnvil에서 사용하는 대표적인 아이디에 대한 설명입니다.

| 이름      | 설명                                                         | 자료형 | 범위      |
| --------- | ------------------------------------------------------------ | ------ | --------- |
| ServiceId | - 설정한 각각의 서비스를 구분하기 위한 아이디 - 하나의 서비스 아이디는 여러 개의 노드로 구성할 수 있음 | int    | 0<id< 100 |
| HostId    | - 호스트의 고유 아이디 - 하나의 호스트에 여러 개의 GameAnvil 프로세스를 구동할 경우에는 GameAnvilConfig에 별도의 vmId를 설정 | long   | -         |
| NodeId    | - 노드의 고유 아이디 - HostId + ServiceId + 내부Counter 값으로 구성 | long   | -         |
| AccountId | - 클라이언트가 접속할 때 입력 - 커넥션(Connection) 하나당 하나의 계정 아이디가 매핑 | string | -         |
| UserId    | - 게임 유저 객체의 고유 아이디 - 게임 유저 객체가 생성될 때 서버가 발급 - 동일한 유저가 재접속 할 경우라도 새로운 게임 유저 객체가 생성되면 새로운 아이디 발급 | int    | -         |
| RoomId    | - 방의 고유 아이디 - 방 객체가 생성될 때 서버가 발급         | int    | -         |
| SubId     | - 하나의 계정(accountId) 내에서 고유한 보조 아이디 - 세션(Session) 하나당 AccountId + SubId가 매핑 - 커넥션(Connection) 단위로 여러개의 세션(Session)을 사용할 때 하나의 접속 내에서 각각의 세션 유저를 구분하는 아이디 - 클라이언트가 접속할 때 입력 | int    | 0 < id    |

### 5-1.아이디 지원 API

앞서 살펴본 ID에 관한 일부 기능을 아래의 표와 같이 엔진 사용자에게 제공합니다. 해당 ID를 획득하거나 확인하기 위해서는 반드시 아래의 API를 사용해야 합니다.

- ServiceId 클래스

| 메서드                                | 설명                                    |
| ------------------------------------- | --------------------------------------- |
| boolean isValid(int serviceId)        | 유효한 serviceId인지 확인               |
| int findServiceId(String serviceName) | ServiceName에 해당하는 ServiceId을 획득 |
| String findServiceName(int serviceId) | ServiceId에 해당하는 ServiceName을 획득 |

- HostId 클래스

| 메서드                       | 설명                     |
| ---------------------------- | ------------------------ |
| boolean isValid(long hostId) | 유효한 hostId인지 확인   |
| long get()                   | 프로세스의 hostId를 획득 |

- NodeId 클래스

| 메서드                        | 이름                          |
| ----------------------------- | ----------------------------- |
| boolean isValid(long nodeId)  | 유효한 nodeId인지 확인        |
| long getHostId(long nodeId)   | nodeId로부터 hostId를 획득    |
| int getServiceId(long nodeId) | nodeId로부터 serviceId를 획득 |
| int getNodeNum(long nodeId)   | nodeId로부터 nodeNum을 획득   |

- UserId 클래스

| 메서드                      | 설명                   |
| --------------------------- | ---------------------- |
| boolean isValid(int userId) | 유효한 userId인지 확인 |

- RoomId 클래스

| 메서드                      | 설명                   |
| --------------------------- | ---------------------- |
| boolean isValid(int roomId) | 유효한 roomId인지 확인 |

- SubId 클래스

| 메서드                     | 설명                  |
| -------------------------- | --------------------- |
| boolean isValid(int subId) | 유효한 subId인지 확인 |



## 6. 채널 (Channel)

채널은 단일 서버군을 논리적으로 나눌 수 있는 방법 중 하나입니다. GameAnvil은 한 개 이상의 게임 노드를 포함할 경우에 채널을 설정할 수 있습니다. 기본적으로 GameAnvilConfig을 통해 게임 노드에 아래의 예제처럼 채널을 설정합니다. 이 예제에서 4개의 게임 노드에 대해 각각 ch1,ch1,ch2,ch2를 설정합니다.

```
"game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "Game",
      "channelIDs": [
        "ch1",
        "ch1",
        "ch2",
        "ch2"
      ],
    }
```

이때, 동일한 채널의 게임 노드는 서로 채널 관련 정보를 공유합니다. 예를 들면 채널에 새로운 유저가 진입하거나 떠날 때 GameAnvil 엔진이 미리 구현한 콜백을 호출합니다. 채널 간 유저와 방 정보 동기화를 위해 다음과 같이 GameUserInfo 그리고 GameRoomInfo를 사용합니다. 앞서 살펴본 GameNode에 대한 Bootstrap의 내용에 추가로 SampleChannelUserInfo와 SampleChannelRoomInfo가 등록되는 것을 확인할 수 있습니다.

```
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class, SampleChannelUserInfo.class)
            .room("SampleGameRoomType", SampleGameRoom.class, SampleChannelRoomInfo.class);

        bootstrap.run();
    }
}
```

그럼 이러한 ChannelUserInfo와 ChannelRoomInfo를 구현하는 방법에 대해 살펴보겠습니다. 우선 채널 유저 정보는 GameAnvil이 제공하는 ChannelUserInfo 인터페이스와 Serializable 인터페이스를 구현합니다. 동일한 채널에 속한 GameNode 사이에서 전송되어야 하므로 Serializable은 필수입니다. 나머지는 엔진 사용자가 원하는 정보로 채우면 됩니다.

```
public class SampleChannelUserInfo implements ChannelUserInfo, Serializable {

    private int userId = 0;
    private String userType = "";
    private String accountId = "";

    public SampleChannelUserInfo(String userType, int userId, String accountId) {
        this.userType = userType;
        this.userId = userId;
        this.accountId = accountId;
    }

    public SampleChannelUserInfo() {}

    public void setUserType(String userType) {
        this.userType = userType;
    }

    public void setUserId(int userId) {
        this.userId = userId;
    }

    public void setAccountId(String accountId) {
        this.accountId = accountId;
    }

    /**
     * {@link KryoSerializer} 를 가지고 직렬화 한다.
     *
     * @return ByteBuffer 로 직렬화 된 객체 반환.
     */
    @Override
    public ByteBuffer serialize() {
        return KryoSerializer.write(this);
    }

    /**
     * {@link KryoSerializer} 를 가지고 역직렬화 한다.
     *
     * @param inputStream 직렬화된 스트림
     * @return ChannelUserInfo 으로 역직렬화된 객체를 반환.
     */
    @Override
    public ChannelUserInfo deserialize(InputStream inputStream) {
        return (GameChannelUserInfo) KryoSerializer.read(inputStream);
    }

    /**
     * 변경될 Channel User 정보의 User Id.
     *
     * @return int type 으로 UserId 반환.
     */
    @Override
    public int getUserId() {
        return userId;
    }

    /**
     * 변경될 Channel User 정보의 Account Id.
     *
     * @return String type 으로 AccountId 반환.
     */
    @Override
    public String getAccountId() {
        return accountId;
    }

    @Override
    public int compareTo(GameChannelUserInfo o) {
        if (o.userId != this.userId)
            return -1;
        else
            return 0;
    }

}
```

채널 방 정보도 동일한 방법으로 구현합니다. GameAnvil이 제공하는 RoomInfo 인터페이스와 Serializable을 구현합니다. 인터페이스명이 ChannelRoomInfo가 아닌 RoomInfo임에 주의하세요. 이 클래스는 채널 유저 정보와 마찬가지로 엔진 사용자가 채널 간 동기화에 사용할 정보를 바탕으로 구현하면 됩니다.

```
public class SampleChannelRoomInfo implements Serializable, RoomInfo {

    public static final int MAX_ENTRY_USER = 4;
    private int roomId = 0;

    private int userCnt;
    private int gameState;
    private long createTime;

    public SampleChannelRoomInfo() {
    }

    //... 콘텐츠에서 필요한 정보로 클래스 구현

    /**
     * Room 정보의 Room Id.
     *
     * @return int type 으로 RoomId 반환.
     */
    @Override
    public int getRoomId() {
        return roomId;
    }

    /**
     * Room 정보를 {@link KryoSerializer} 이용해서 serialize 처리를 한다.
     *
     * @return ByteBuffer type 으로 serialize 된 내용을 반환.
     */
    @Override
    public ByteBuffer serialize() {
        return KryoSerializer.write(this);
    }

    /**
     * 전달받은 정보를 {@link KryoSerializer} 이용해서 deserialize 처리를 한다.
     *
     * @param inputStream deserialize 할 데이터
     * @return RoomInfo 로 Room 정보를 반환.
     */
    @Override
    public RoomInfo deserialize(InputStream inputStream) {
        return (RoomInfo) KryoSerializer.read(inputStream);
    }

    /**
     * Room 정보를 복사 한다.
     *
     * @return RoomInfo 로 복사된 Room 정보를 반환.
     * @throws CloneNotSupportedException 복사가 안되는 경우.
     */
    @Override
    public RoomInfo copy() throws CloneNotSupportedException {
        GameRoomInfo roomInfo = (GameRoomInfo) super.clone();
        roomInfo.testListInfo = new ArrayList<>(testListInfo);

        return roomInfo;
    }
}
```

앞서 살펴본 채널의 유저,방 정보는 실제 해당 유저나 방에 대한 갱신이 필요할 때마다 동일한 채널 내의 GameNode 사이에 전파됩니다. 이때, 관련 GameNode는 아래의 2가지 콜백 메서드를 각각 채널 유저 정보와 채널 방 정보 갱신에 대해 호출합니다.

```
    /**
     * 같은 채널의 다른 node 에 유저 변화가 있을때 호출되는 콜백
     * <p>
     * updateChannelUser() 호출시 발생.
     *
     * @param type            Channel 정보 변경 타입(갱신/삭제)
     * @param channelUserInfo 변경될 유저 정보
     * @param userId          변경 대상의 User Id
     * @param accountId       변경 대상의 Account Id
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    public void onChannelUserUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {        
    }
```

파라메터로 전달받은 채널 유저,방 정보를 바탕으로 엔진 사용자가 원하는 방식으로 구현하면 됩니다.

```
    /**
     * 같은 채널의 다른 node 에 room 상태 변화가 있을때 호출되는 콜백
     * <p>
     * updateChannelRoomInfo() 호출시 발생.
     *
     * @param type            Channel 정보 변경 타입(갱신/삭제)
     * @param channelRoomInfo 변경될 Room 정보
     * @param roomId          변경 대상의 Room Id
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    public void onChannelRoomUpdate(ChannelUpdateType type, RoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {        
    }
```

마지막으로 클라이언트는 서버로 채널 정보를 요청할 수 있습니다. 이때, 아래의 콜백 메서드가 호출됩니다. 이 또한 엔진 사용자가 원하는 방식으로 적절하게 구현합니다.

```
    /**
     * 클라이언트에서 Channel 정보를 요청 시 호출되는 콜백 (Base.GetChannelInfoReq)
     *
     * @param outPayload 클라이언트로 전달될 Channel 정보
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {        
    }
```



## 7. 매칭그룹 (MatchingGroup)

매칭 그룹도 채널과 마찬가지로 단일 서버군을 논리적으로 나눌 수 있는 방법 중 하나입니다. 단, 매칭 그룹은 채널과 달리 명시적으로 미리 설정하지 않습니다. 또한 채널은 GameNode를 논리적으로 나누기 위한 방법인 반면에 매칭 그룹은 매치메이킹을 논리적으로 나누기 위한 방법입니다. 앞서 살펴본 유저 매치메이킹 콜백 메서드를 다시 한번 살펴보겠습니다. 이 예제에서 설정한 매칭 그룹은 "Newbie"입니다. 이 "Newbie"라는 값은 클라이언트에서 서버로 payload를 통해 전달할 수도 있고 엔진 사용자가 서버 상에 해당 유저에 대해 미리 저장해둔 값일 수도 있습니다. 아래와 같이 하드코딩한 값을 쓸 수도 있으나 실제 게임 서버 코드에는 바람직하지 않습니다. 어쨌든  "Newbie"라는 매칭 그룹에 대해 하나의 매치메이커가 생성되고 이 후의 모든 "Newbie" 요청은 이 매치 메이커에 쌓여서 함께 처리되는 것이 보장됩니다.

```
    /**
     * 클라이언트에서 userMatch를 요청했을 경우 호출되는 콜백
     *
     * @param roomType   매칭되는 room의 타입
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return boolean type으로 반환. true: user matching 요청 성공,false: user matching 요청 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend될 수 있다.
     */
    public boolean onMatchUser(final String roomType,
                               final Payload payload, Payload outPayload) throws SuspendExecution {

        String matchingGroup = "Newbie";
        SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId());
        sampleUserMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, term, payload);
    }
```

매치 메이킹 관련한 모든 프로토콜에는 매칭 그룹 필드가 존재합니다. 그러므로 클라이언트와 서버 사이에 미리 매칭 그룹을 정의해두고 사용하는 것이 가장 이상적입니다. 예를 들어, "초보", "중수", "고수"처럼 실력을 기반으로 한 매칭 그룹을 정의할 수도 있고 "한국", "일본", "미국"처럼 국가별 매칭 그룹을 정의할 수도 있습니다. 이는 어디까지나 엔진 사용자가 원하는 대로 사용할 수 있는 부분입니다.



## 8. MatchNode & MatchMaker

엔진 사용자는 간단하게 매칭 로직만 구현함으로써 매치메이킹을 적용할 수 있습니다. 매치메이커는 로직에 기반하여 유저들을 동일한 방으로 입장시켜줍니다. GameAnvil은 RoomMatchMaker와 UserMatchMaker의 2가지의 매치메이커를 제공합니다. 이 매치메이커는 MatchNode에서 독자적으로 구동됩니다. 이러한 MatchNode는 매치메이킹을 수행하는 용도 외에 추가적인 콘텐츠를 구현할 수 없습니다. 그러므로 유저는 MatchNode가 아닌 매치메이커에만 집중하면 됩니다.

### Note

*매치메이커는 매칭 그룹 단위로 생성됩니다. 즉, 동일한 매칭 그룹끼리 매치메이킹을 수행합니다.*



### 6-1. UserMatchMaker

유저 매치메이킹은 게임 유저들의 매칭 요청을 큐에 적재합니다. 특정 시간 주기로 이 요청 큐의 내용을 비교, 분석하여 사용자가 원하는 기준으로 임의의 유저들을 하나의 방으로 입장시켜줍니다. 여기에서 엔진 사용자는 요청 큐의 내용을 어떻게 비교하고 분석해서 어떤 기준으로 유저들을 매칭할지에 대한 로직에만 집중하면 됩니다. 참고로 가장 대표적인 유저 매치메이킹 게임은 "*리그 오브 레전드*"가 있습니다.

이러한 유저 매치메이킹의 가장 기본은 바로 매칭 요청 그 자체입니다. 이러한 매칭 요청을 아래와 같이 엔진에서 제공하는 UserMatchInfo 추상 클래스를 상속하여 구현합니다.  요청자를 구분할 수 있는 게임 유저의 ID를 제공할 수 있도록 getId() 메서드는 반드시 구현해야 합니다. 아래의 예제는 매칭 요청 사이의 비교를 위해 Comparable 인터페이스를 추가로 구현하고 있습니다.

```
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



이제 유저 매치메이커를 만들 차례입니다. 유저 매치메이커는 엔진에서 제공하는 UserMatchMaker 추상 클래스를 상속 구현합니다. 특히, match() 메서드는 실제 매칭을 수행하기 위해 호출되는 콜백이므로 주의 깊게 살펴보세요.  refill() 메서드는 이미 완료된 매치 메이킹에 대해 충원 요청이 왔을 때 호출되는 콜백입니다. 예를 들어 4명이 매치메이킹 된 상태에서 1명이 게임을 종료해버렸을 때 1명을 더 충원하기 위해 사용할 수 있습니다. 아래의 예제 코드는 이러한 UserMatchMaker를 어떤 식으로 구현할 수 있는지 보여줍니다.

```
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



이렇게 생성한 UserMatchMaker와 UserMatchInfo는 Bootstrap 단계에서 등록해야 합니다. 한 가지 주의할 점은 매치메이커는 아래와 같이 게임 노드 구성의 연장선 상에 있다는 점입니다. 즉, GameNode에서 사용할 GameUser와 GameRoom을 구성하는 것과 더불어 이들을 매칭하기 위해 어떤 UserMatchMaker를 사용할지 구성하는 것입니다.

```
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



이제 클라이언트는 서버로 유저 매치메이킹을 요청할 수 있습니다. 이 요청은 GameUser에 전달된 후 엔진에 의해 아래의 콜백 메서드를 호출합니다. 이 콜백 메서드는 엔진 사용자로 하여금 GameAnvil이 제공하는 유저 매치메이커 뿐만 아니라 제3의 솔루션을 사용할 기회를 제공합니다. 아래의 예제는 엔진에서 제공하는 유저 매치메이커를 사용하기 위해 matchUser() API를 호출하는 모습입니다. 엔진 사용자는 자체적으로 매치메이커를 만들어서 사용하거나 다른 라이브러리 등을 사용할 수도 있습니다. 이 경우에는 그에 알맞게 아래의 콜백 메서드를 구현하도록 합니다. 특히, 이 예제 코드에서 유심히 봐야할 부분은 매칭 그룹입니다. 매칭 그룹은 매치메이킹을 논리적으로 나눌 수 있는 개념으로 이 문서의 중급 개념에서 좀 더 자세하게 설명합니다. 예제에서 사용한 "Newbie" 매칭 그룹은 동일한 "Newbie" 매칭 그룹끼리 같은 매칭 큐를 공유하게 됩니다.

```
    /**
     * 클라이언트에서 userMatch 를 요청했을 경우 호출되는 콜백
     *
     * @param roomType   매칭되는 룸의 타입
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
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



마지막으로 클라이언트는 언제든 앞서 요청한 매치메이킹에 대해 취소를 할 수 있습니다. 이때, 엔진은 취소 처리가 성공적으로 진행되면 아래와 같은 GameUser의 콜백 메서드를 호출합니다. 엔진 사용자는 이 콜백에서 취소 타이밍에 처리하고 싶은 부분을 구현하면 됩니다.

```
    /**
     * 클라이언트에서 userMatch 가 취소되었을 때 호출되는 콜백
     *
     * @param reason 취소된 이유(TIMEOUT/CANCEL)
     * @return boolean type 으로 반환. true: user matching 취소 성공, false: user matching 취소 실패.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
    }
```



### 6-2. RoomMatchMaker

룸 매치메이킹은 게임 유저들을 적합한 방으로 자동 입장시켜줍니다. 룸 매치메이킹을 요청한 게임 유저를 어떤 방으로 입장시킬지는 엔진 사용자가 구현하기에 달렸습니다. 가장 유저수가 많은 방으로 입장시킬 수도 있고, 가장 한산한 방으로 입장시킬 수도 있습니다. 혹은 평균 점수가 가장 높은 방으로 입장시킬 수도 있습니다. 엔진 사용자는 이러한 매칭 로직에만 집중하면 됩니다. 참고로 가장 대표적인 룸 매치메이킹 게임은 "*한게임 포커*"나 "*카트라이더*" 등이 있습니다.

- 주의> RoomMatchMaker와 UserMatchMaker는 서로 독립적으로 운영됩니다. 즉, 동일한 매칭 그룹으로 유저 매칭과 룸 매칭을 각각 요청하더라도 이 두 요청이 함께 매칭되는 일은 없습니다.

이러한 룸 매치 메이킹의 가장 기본은 바로 매칭 요청 그 자체입니다. 이러한 매칭 요청을 아래와 같이 엔진에서 제공하는 RoomMatchInfo 인테페이스를 구현합니다.  요청자를 구분할 수 있는 게임 유저의 ID를 제공할 수 있도록 getId() 메서드는 반드시 구현해야 합니다.

```
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
     * RoomMatchInfo가 매칭 가능한 Room 또는 매칭된 Room의 정보를 의미하는 경우 해당 Room의 RoomId를 의미한다.
     * <p>
     * RoomMatchInfo가 매칭 조건을 의미하는 경우 컨탠츠에서 지정한 RoomId를 반환한다.
     * <p>
     * 콘텐츠에서 원하는 방식으로 사용할 수 있다.
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



이제 룸 매치메이커를 만들 차례입니다. 룸 매치메이커는 엔진에서 제공하는 RoomMatchMaker 추상 클래스를 상속 구현 합니다. 특히, match() 메서드는 실제 매칭을 수행하기 위해 호출되는 콜백이므로 주의 깊게 살펴보세요. 아래의 예제 코드는 이러한 RoomMatchMaker를 어떤식으로 구현할 수 있는지 보여줍니다. UserMatchMaker는 UserMatchInfo가 Comparable 인터페이스를 구현한 반면에 RoomMatchMaker는 Comparator를 제공하고 있습니다. 이는 단순히 엔진 사용자가 유연하게 원하는 방식대로 구현할 수 있음을 부여주기 위함입니다.

```
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
     * @param terms 매칭 조건
     * @param args  추가로 전달받은 데이터
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



이렇게 생성한 RoomMatchMaker와 RoomMatchInfo는 Bootstrap 단계에서 등록해야 합니다. 주의할 점은 유저 매치메이커와 동일합니다. 룸 매치메이커 역시 아래와 같이 게임 노드 구성의 연장선 상에 있습니다. 즉, GameNode에서 사용할 GameUser와 GameRoom을 구성하는 것과 더불어 이들을 매칭하기 위해 어떤 RoomMatchMaker를 사용할지 구성하는 것입니다. 또한, 유저 매치메이커와 룸 매치메이커를 함께 사용할 수 있습니다. 아래의 예제 코드는 이를 보여주기 위해 2가지 매치메이커를 함께 등록하고 있습니다. 단, 두 매치메이커가 사용하는 Room과 MatchInfo 클래스가 나뉘어져 있음에 주의하십시요.

```
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



이제 클라이언트는 서버로 룸 매치메이킹을 요청할 수 있습니다. 이 요청은 GameUser에 전달된 후 엔진에 의해 아래의 콜백 메서드를 호출합니다. 이 콜백 메서드는 엔진 사용자로 하여금 GameAnvil이 제공하는 룸 매치메이커 뿐만 아니라 제3의 솔루션을 사용할 기회를 제공합니다. 아래의 예제는 엔진에서 제공하는 룸 매치메이커를 사용하기 위해 matchRoom() API를 호출하는 모습입니다. 엔진 사용자는 자체적으로 매치메이커를 만들어서 사용하거나 다른 라이브러리 등을 사용할 수도 있습니다. 이 경우에는 그에 알맞게 아래의 콜백 메서드를 구현하도록 합니다. 특히, 이 예제 코드에서 유심히 봐야할 부분은 매칭 그룹입니다. 매칭 그룹은 매치메이킹을 논리적으로 나눌 수 있는 개념으로 이 문서의 중급 개념에서 좀 더 자세하게 설명합니다. 예제에서 사용한 "Newbie" 매칭 그룹은 동일한 "Newbie" 매칭 그룹끼리 같은 매칭 큐를 공유하게 됩니다.

```
    /**
     * 클라이언트에서 roomMatch 를 요청했을 경우 발생하는 콜백
     *
     * @param roomType 매칭되는 룸의 타입
     * @param payload  클라이언트로부터 전달받은 페이로드
     * @return {@link MatchRoomResult} 으로 matching 된 room 의 정보 반환, null 을 반환 할시  클라이언트 요청 옵션에 따라서 새로운 Room 이 생성되거나,요청 실패 처리 된다.
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



## 7. Support Node

SupportNode는 보조적인 기능을 수행하기 위한 노드입니다. 게임 유저나 방 객체와 상관없이 임의의 기능을 구현할 수 있습니다. 예를 들어 로그를 취합해서 전송하거나 빌링 서버와 통신을 전담하는 등의 역할로 사용할 수 있습니다. 또한 엔진의 RESTful 기능을 사용하여 간단한 웹 서버 대용으로 사용하는 것도 가능합니다.

이러한 SupportNode는 기본적으로 BaseSupportNode 추상 클래스를 상속 구현합니다. 노드 공통 콜백 메서드에 추가로 RESTful 처리를 위한 onDispatch() 콜백이 제공됩니다. 아래 코드에서 1~3에 해당하는 주석 아래 코드가 이를 위한 처리 흐름을 순서대로 보여줍니다.

```
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
     * rest call 이 전달될 때 호출된다.
     *
     * @param restObject 전달된 {@link RestObject}
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



이렇게 생성한 SupportNode는 Bootstrap 단계에서 아래와 같이 등록할 수 있습니다.

```
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



## 8. 방 전송 (RoomTranfer)

방 전송은 중급 개념에서 설명한 유저 전송과 매우 비슷한 기능입니다. 유저 보다 큰 개념인 방이 전송되는 차이가 있을 뿐입니다. 게임 노드에는 한 명 이상의 게임 유저가 참여한 방 객체가 생성될 수 있습니다. 이 방 객체는 유저 객체와 마찬가지로 게임 노드 사이에서 전송될 수 있습니다. 예를 들면 1번 게임 노드의 방 객체가 2번 게임 노드로 전송이 되는 것이죠. 이렇게 방이 전송될 때 당연하게도 방 안에 존재하는 유저들도 함께 전송이 일어납니다. 그 덕분에 방이 전송되기 전/후의 게임 흐름은 연속되어 진행될 수 있습니다. 즉, 해당 방 객체가 전송되는 과정은 매우 빠르게 진행되므로 방 안의 유저들은 인지하지 못한 상태에서 게임을 지속할 수 있습니다.  이 기술은 GameAnvil 무점검 패치(NonStopPatch)의 핵심이자 근간이 되며 방 객체를 잘 직렬화/역직렬화해야 의도한대로 동작합니다. 이러한 직렬화/역직렬화 대상은 GameAnvil 사용자가 직접 구현할 수 있습니다. 즉, 해당 방 객체의 어떤 값을 전송할 것인지 결정하는 것이죠.

이러한 방 전송을 발생시키는 것은 오직 무점검 패치(NonStopPatch) 명령 뿐입니다. 이 명령은 일반적으로 GameAnvil Console을 통해 게임 운영자가 명시적으로 전달합니다.



### 8-1. 방 전송 구현

실제 방 전송은 GameAnvil이 내부적으로 조용하게 처리합니다. 이때, 클라이언트는 자신의 게임 유저와 더불어 자신이 속한 방 객체가 서버 사이에서 전송되는지 인지하지 못할 가능성이 높습니다. 특별한 문제가 발생하지 않는 한 전체 흐름이 매우 빠르게 진행되기 때문에 전송 전의 게임 흐름을 전송 후에 계속 이어감에 있어 무리가 없습니다.

이때, 다른 게임 노드로 방 객체를 전송할 때 어떤 데이터를 함께 전송할지는 사용자가 지정할 수 있습니다. 간단히 지정만 해두면 이 데이터는 방 객체의 일부로 함께 직렬화됩니다. 전송할 대상 게임 노드에서는 역직렬화를 신경 쓸 필요 없이 쉽게 해당 객체들에 접근해서 사용할 수 있습니다.

다음은 이러한 유저 전송에 관련된 콜백 메서드들입니다. 우선 "함께 전송할 데이터 지정하기"입니다. 이를 위해 BaseRoom 추상 클래스를 상속받은 게임 룸 클래스에 아래의 콜백을 구현합니다. 방법은 간단합니다. 아래의 예제와 같이 전송할 데이터를 사용자가 원하는 key값을 이용하여 key-value 쌍으로 매개변수로 넘겨받은 transferPack에 넣기만 하면 됩니다.

만일 여기에서 전송할 데이터를 지정하지 않으면 대상 게임 노드로 전송된 방 객체의 해당 데이터는 모두 기본값으로 초기화되므로 주의가 필요합니다.

**참고**> *앞서 설명하였듯이 방이 전송될 때는 당연하게도 방 안의 유저들이 함께 전송됩니다. 유저 전송에 관해서는 중급 개념에서 설명했으므로 여기에서는 따로 설명하지 않습니다.*

```
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("RoomInfo", myRoomInfo);
}
```

이제 방 전송이 완료된 후 대상 게임 노드에서 처리할 콜백 메서드를 살펴보겠습니다. 아래와 같이 전송 전에 지정한 key를 이용해서 원하는 객체에 접근할 수 있습니다. 이와 같은 방법으로 전송 완료된 방 객체의 해당 데이터를 원래 상태로 만들어 줍니다. 특히, 전송된 방 안의 유저 객체 리스트를 매개 변수로 전달받고 있음을 확인할 수 있습니다. 이 부분만 제외하면 전체적인 흐름은 유저 전송과 매우 흡사합니다.

```
@Override
public void onTransferIn(List<GameUser> userList, TransferPack transferPack) throws SuspendExecution {
    this.users.clear();
    for (GameUser user : userList)
        this.users.put(user.getUserId(), user);

    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setRoomInfo(transferPack.getToLong("RoomInfo"));
}
```



### 8-2. 전송 가능한 방 타이머

방이 전송될 때 방에 등록해둔 타이머도 함께 전송할 수 있습니다. 전송 가능한 타이머를 사용하기 위해서는 별도로 아래의 콜백 메서드에서 원하는 key를 이용해서 미리 등록해두어야 합니다. Timer 핸들러를 다음과 같이 원하는 key로 매핑해둡니다. 이는 유저 전송과 완전히 동일합니다.

```
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferRoomTimerHandler1", transferRoomTimerHandler1());
     registerTimerHandler("transferRoomTimerHandler2", (timerObject, object) -> {
         GameRoom room = (GameRoom) object;
         logger.warn("GameRoom::transferRoomTimerHandler2() : roomId({})", room.getId());
     });
 }
```

등록이 완료된 타이머 객체는 언제든 해당 key를 이용해서 아래와 같이 게임 룸 구현부에서 사용할 수 있습니다. 단순히 등록만 한 타이머는 효과가 없으므로 실제 사용을 위해서는 반드시 아래와 같이 유저 객체에 추가해야 타이머가 발동합니다.

```
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

이렇게 방 객체에 추가해 둔 타이머는 별도로 전송에 대한 처리를 하지 않아도 모두 자동으로 전송됩니다. 즉, 대상 게임 노드에서 해당 방 객체에 대해 동일한 타이머 추가 과정을 거칠 필요가 없습니다.



## 9. 무정지 점검 (NonStopPatch)

일반적으로 점검을 할 때에는 게임 서비스 전체에 대해 명시적으로 점검을 걸고 유저들의 게임 플레이를 차단합니다. 그 후 필요한 패치를 진행하죠. 하지만 GameAnvil을 사용하면 서버 바이너리의 호환성이 깨지는 경우가 아닌 이상 서비스를 멈출 필요 없이 패치를 진행할 수 있습니다. 이를 무점검 패치라고 하며 기본적으로 GameAnvil Console 운영 도구를 이용해서 진행할 수 있습니다.

무점검 패치의 핵심은 앞서 설명했던 유저 전송과 방 전송 기술입니다. 이 두 기능을 기반으로 패치를 진행할 게임 서버의 유저와 방을 모두 유효한 타 게임 서버로 전송시킨 후 게임 서버가 빈 상태가 되었을 때 패치를 진행하는 것이죠. 이러한 무점검 패치는 서비스 과정에서 진행할 부분이므로 반드시 서비스 전에 선 테스트를 진행하여 문제가 없는지 확인하는 과정을 거치시기 바랍니다. 또한 패치할 서버 바이너리와 이전 서버 바이너리 사이의 호환성이 깨지지 않는지 반드시 체크해보아야 합니다.

무점검 패치를 이용하기 위해서 사용자는 GameNode 클래스에 관련 콜백 메서드들을 재정의해야 합니다.

```
    /**
     * 무정지 점검 시작 시 해당 node 가 출발지 node 일 경우 호출되는 콜백
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean onNonStopPatchSrcStart() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검이 종료되면 해당 node 가 출발지 node 일 경우 호출되는 콜백
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검이 시작되고 해당 node 가 출발지 node 일 경우 무정지 점검을 끝낼지 확인하기 위해 호출되는 콜백
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검 시작 시 해당 node 가 도착지 node 일 경우 호출되는 콜백
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean onNonStopPatchDstStart() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검이 종료되면 해당 node 가 도착지 node 일 경우 호출되는 콜백
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean onNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검이 시작되고 해당 node 가 도착지 node 일 경우 무정지 점검을 끝낼지 확인하기 위해 호출되는 콜백
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean canNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
```



### 3-1. 무점검 패치 가이드

무점검 패치는 GameAnvil의 여러 요소가 복합적으로 잘 맞물려 돌아갔을 때 좋은 결과가 나옵니다. 그러므로 반드시 아래의 가이드 문서를 읽어보고 모든 내용을 이해한 후에 사용할 수 있도록 하세요.

#### [무점검 패치 가이드](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-nonstop-patch)

## 10. 프로토콜 정의와 컴파일

GameAnvil은 [Google Protocol Buffers](https://developers.google.com/protocol-buffers)를 사용하여 프로토콜을 정의하고 빌드합니다. 아래의 예제는 이러한 프로토콜을 정의하는 법과 빌드하는 법을 설명합니다. 우선 SampleGame.proto 파일을 텍스트 에디터로 생성한 후 원하는 프로토콜을 정의합니다. 프로토콜 버퍼의 자세한 문법은 [공식 Protocol Buffers 가이드](https://developers.google.com/protocol-buffers/docs/overview)를 참고할 수 있습니다.

```
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



그리고 아래와 같은 명령줄로 작성한 프로토콜 스크립트를 컴파일할 수 있습니다. 이때, build.bat와 같은 배치 파일을 생성해두면 편리합니다. 만일 GameAnvil 템플릿을 이용하여 프로젝트를 구성했다면 프로젝트 내의 proto 폴더에서 프로토콜 스크립트와 배치파일을 참고할 수 있습니다. 컴파일이 완료되면 하면 프로토콜 처리에 필요한 모든 Java 클래스가 자동으로 생성됩니다.

```
protoc  ./MyGame.proto --java_out=../java
```



## 11. Configurations

GameAnvil은 GameAnvilConfig.json 파일을 통해 서버 구성을 변경할 수 있습니다. 이러한 구성 파일은 총 7개의 항목으로 분류되며 common을 제외하면 각각 한 종류의 노드를 설정합니다. 이때, 설정하는 값 중 일부는 Bootstrap 단계에서 사용하는 값과 일치해야 합니다. 예를 들면 앞서 우리는 아래와 같이 GameNode를 구성했습니다. 여기에 사용된 ServiceName인 "SampleGameService"는 반드시 GameAnvilConfig.json에 설정되어 있어야 합니다. 그렇지 않으면 해당 서비스를 찾지 못하여 서버 구동 과정에서 오류가 발생합니다.

```
bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
```



### common

- 필수적인 공통 설정

| 이름                            | 설명                                                         | 기본값  |
| ------------------------------- | ------------------------------------------------------------ | ------- |
| vmId                            | 하나의 machine에서 여러 개의 JVM을 띄울 경우 사용하는 고유식별 ID ( 0 <= vmId < 100) | 0       |
| ip                              | 노드마다 공통으로 사용하는 IP. (머신의 IP를 지정) 값이 없을 경우 private ip로 자동 지정된다. |         |
| meetEndPoints                   | 대상 노드의 common IP와 communicatePort 등록 해당 서버 endpoint 포함 가능 리스트로 여러 개 등록 가능 |         |
| keepAliveTime                   | communicateNode 연결 지속 시간 (ms)                          | 60000   |
| nodeTimeout                     | 노드 상태 체크 타임아웃(ms) 0이면 체크하지 않음.             | 180000  |
| msgExpiredTime                  | 메시지 만료 시간                                             | 30000   |
| defaultReqTimeout               | 노드간 메시지 라우팅 타임아웃(ms)0이면 체크하지 않음.        | 30000   |
| defaultAsyncAwaitTimeout        | 블로킹 호출 시 타임아웃(ms)                                  | 30000   |
| ipcPort                         | 다른 ipc node 와 통신할때 사용되는 포트                      | 11100   |
| publisherPort                   | publish socket을 위한 포트                                   | 11101   |
| debugMode                       | 디버깅 시 각종 timeout 이 발생안하도록 하는 옵션리얼에서는 반드시 false 이어야 한다. | false   |
| nodeInfoUpdateTime              | JVM에 있는 노드 정보를 다른 JVM에게 전파하는 주기            | 500(ms) |
| nodeInfoUpdateBusyTime          | 노드 정보가 업데이트될 때 이 시간보다 오래 걸렸을 경우 해당 노드는 BUSY 상태라고 판단함. | 510(ms) |
| nodeInfoManagerRefreshTime      | 전달받은 노드 정보를 이용해 NodeInfoManager를 갱신하는 주기. | 100(ms) |
| loopInfoSamplingCount           | Message Loop의 상태를 체크할 때 Sampling할 loop 횟수         | 100(회) |
| enableLoopInfoMonitor           | NodeInfoPage에서 loop 정보 갱신 여부                         | true    |
| maxUserCount                    | IdleNode 판단 시 idle로 판단할 최대 User 수                   | 2500    |
| addLocWeight                    | UserLoc 할당 시 정보 보정을 위해 사용할 배수. 예) 게임 노드가 8개일 경우 1명이 할당되었지만 8명의 유저가 할당되었다고 임의로 지정. | 1       |
| allocatorType                   | 사용할 bytebuf type 1 (POOLED_DIRECT_BUFFER) 2 (POOLED_HEAP_BUFFER) 3 (UNPOOLED_DIRECT_BUFFER) 4 (UNPOOLED_HEAP_BUFFER) | 4       |
| httpClientOption                | Gameflex에서 사용하는 httpClientOption                       |         |
| httpClientOption-keepAlive      | httpClientOption 연결 지속 여부                              | true    |
| httpClientOption-connTimeout    | httpClientOption connection 타임아웃                         | 5000    |
| httpClientOption-requestTimeout | httpClientOption  request 타임아웃                           | 10000   |
| httpClientOption-maxConnPerHost | httpClientOption host당 최대 connection 개수                 | 10000   |
| httpClientOption-maxConn        | httpClientOption connection 최대 개수                        | 12000   |
| moduleURL                       | config, dynamicModule 정보를 읽어올 URL 정보 해당 옵션이 설정될 경우 URL을 사용해서 module 정보를 불러온다. | ""      |
| configPath                      | config를 File로 읽어올 시 config의 Json 파일을 읽어올 경로 moduleURL이 설정될 경우 무시 | /config |

- 사용 예시

```
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



### location

- 설정값이 없거나 clusterSize가 0이면 LocationNode를 생성하지 않습니다.

| 이름        | 설명                                                         | 기본값 |
| ----------- | ------------------------------------------------------------ | ------ |
| clusterSize | 구성되는 서버의 수(VM)                                       |        |
| replicaSize | 복제 그룹의 크기 master + slave의 개수                       |        |
| shardFactor | sharding을 위한 인수 <br />-전체 shard의 개수 = clusterSize x replicaSize x shardFactor <br />-하나의 머신(VM)에서 구동할 shard의 개수 = replicaSize x shardFactor <br />-고유한 shard의 총 개수 (master 샤드의 개수) = clusterSize x shardFactor |        |

- 사용 예시

```
"location": {
    "clusterSize": 1,
    "replicaSize": 3,
    "shardFactor": 3
},
```



### match

- 설정값이 없거나 nodeCnt가 0이면 MatchNode를 생성하지 않습니다.

| 이름    | 설명            | 기본값 |
| ------- | --------------- | ------ |
| nodeCnt | match node 개수 |        |

- 사용 예시

```
"match": {
    "nodeCnt": 3
},
```



### gateway

- 설정값이 없거나 nodeCnt가 0이면 GatewayNode를 생성하지 않습니다.

| 이름                                     | 설명                                                         | 기본값 |
| ---------------------------------------- | ------------------------------------------------------------ | ------ |
| nodeCnt                                  | 노드 개수                                                    |        |
| ip                                       | 클라이언트와 연결되는 IP 설정값이 없을 경우 private ip로 자동 설정 |        |
| dns                                      | 클라이언트와 연결되는 도메인 주소                            |        |
| maintenance                              | 서버 시작 시 점검 여부                                       | false  |
| checkWhiteListBeforeOnAuth               | onAuthenticate 콜백 호출 전 WhiteList 체크 여부              | true   |
| connectGroup                             | TCP_SOCKET WEB_SOCKET                                        |        |
| connectGroup - port                      | 클라이언트와 연결되는 포트                                   |        |
| connectGroup - idleClientTimeout         | 데이터 송수신이 없는 상태 이후의 타임아웃.0 이면 사용하지 않음 | 4000   |
| connectGroup - secure                    | 보안 설정 설정값이 없을 경우 사용하지 않음.                  |        |
| connectGroup - secure - useSelf          | keyCertChainPath, privateKeyPath 설정값과 상관없이 보안 설정 사용여부 | false  |
| connectGroup - secure - keyCertChainPath | 인증서 경로 <br />-Dsecure 설정 경로에서 시작됨              |        |
| connectGroup - secure - privateKeyPath   | 개인키 경로 <br />-Dsecure 설정 경로에서 시작됨              |        |
| duplicateLoginServices                   | 중복 로그인 가능 서비스 지정                                 |        |

- 사용 예시

```
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



### game

- 설정값이 없거나 nodeCnt가 0이면 GameNode를 생성하지 않습니다. 앞서 설명했듯이 엔진의 Bootstrap 단계에서 사용한 ServiceName을 이곳에 설정해야 합니다.

| 이름        | 설명                                                         | 기본값 |
| ----------- | ------------------------------------------------------------ | ------ |
| nodeCnt     | 노드 개수                                                    |        |
| serviceId   | Service ID                                                   |        |
| serviceName | Service 이름                                                 |        |
| channelIDs  | 노드마다 부여할 채널 ID. 유니크하지 않아도 됨, "" 문자열로 채널 구분없이 중복사용도 가능 |        |
| userTimeout | disconnect 이후의 유저객체 제거 타임아웃                     | 10000  |

- 사용 예시

```
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



### support

- 설정값이 없거나 nodeCnt가 0이면 SupportNode를 생성하지 않습니다. SupportNode도 GameNode와 마찬가지로 Bootstrap 단계에서 사용한 ServiceName이 반드시 설정되어야 합니다.

| 이름                             | 설명                                                         | 기본값 |
| -------------------------------- | ------------------------------------------------------------ | ------ |
| nodeCnt                          | 노드 개수                                                    |        |
| serviceId                        | service ID                                                   |        |
| serviceName                      | service 이름                                                 |        |
| restIp                           | rest ip 설정값이 없을 경우 private ip로 자동 설정            |        |
| restPort                         | rest port                                                    |        |
| restPermissionGroups             | ACL 설정 허용할 특정 Path와 IP를 등록. 비어 있을 경우 모든 접속 허용 |        |
| restSecure                       | 설정값이 없을 경우 사용하지 않음                             |        |
| restSecure - keyCertChainPath    | 인증서 경로 -Dsecure 설정 경로에서 시작됨                    |        |
| restSecure - privateKeyPath      | 개인키 경로 -Dsecure 설정 경로에서 시작됨                    |        |
| restSecure - authorizationSecret | 인증토큰(JWT) 사용하는 경우만 필요, 암호화 시그니처 키       |        |
| restSecure - authorizationPath   | 인증토큰(JWT) 사용하는 경우만 필요, 인증을 진행할 경로       |        |
| restCorsAllowDomains             | CORS 허용 domain 예-127.0.0.1:1234                           |        |

- 사용 예시

```
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



### management

- 설정값이 없거나 nodeCnt가 0이면 ManagementNode를 생성하지 않습니다.

| 이름          | 설명                                                         | 기본값                          |
| ------------- | ------------------------------------------------------------ | ------------------------------- |
| nodeCnt       | 노드 개수                                                    |                                 |
| restIp        | rest ip 설정값이 없을 경우 private ip로 자동 설정            |                                 |
| restPort      | rest port                                                    |                                 |
| db            | management에서 사용하는 db 설정                              |                                 |
| db - user     | db 접속 ID                                                   |                                 |
| db - password | db 접속 비밀번호                                             |                                 |
| db - url      | db 접속 url - h2db : jdbc:h2:mem:gameflex_admin;DB_CLOSE_DELAY=-1 - mysql : jdbc:mysql://localhost:3306/gameflex_admin?useSSL=false - mariadb : jdbc:mariadb://localhost:3306/gameflex_admin?useSSL=false |                                 |
| db - driver   | db 접속 driver- h2db : org.h2.Driver- mysql : com.mysql.jdbc.Driver- mariadb : org.hibernate.dialect.MySQLDialect | org.h2.Driver                   |
| db - dialect  | db 접속 dialect- h2db : org.hibernate.dialect.H2Dialect- mysql : org.hibernate.dialect.MySQLDialect- mariadb : org.hibernate.dialect.MySQLDialect | org.hibernate.dialect.H2Dialect |
| db - ddlAuto  | ddl 옵션 - update : 자동으로 Table 생성 - none : 자동으로 Table을 생성하지 않음. | update                          |

- 사용 예시

```
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



## 12. 비동기 지원 API

앞서 설명한 파이버 상에서의 비동기 처리를 위해 GameAnvil은 Async 클래스를 제공합니다. 아래와 같은 import문을 통해 Async 클래스를 이용하면 일반적인 블로킹/논블로킹 호출을 모두 파이버화 할 수 있습니다.

```
import com.nhn.gameanvil.async.Async;
```

*모든 API에 대한 설명은 [GameAnvil API Reference](http://10.162.4.61:9090/gameanvil)에서 JavaDoc으로 작성된 문서를 확인할 수 있습니다.*

호출용 API는 크게 call과 run으로 나뉘며 각각 반환값이 있는 경우와 그렇지 않은 경우에 사용합니다. 그 외 스레드 기반의 future를 파이버 기반으로 사용할 수 있도록 전환해줍니다. 각각의 용도에 따른 사용법은 문서의 다음 부분에서 더 자세하게 다루도록 합니다.

### **12-1.블로킹 호출 처리**

일반적인 블로킹 호출은 스레드 블로킹을 의미합니다. 즉, 현재 코드가 수행 중인 파이버 뿐만 아니라 이 파이버를 스케줄링하는 스레드까지 블로킹 시킨다는 뜻입니다. 이 말은 곧 노드가 멈춘다는 의미이므로 절대 블로킹 호출을 직접적으로 사용해서는 안 됩니다. GameAnvil은 이러한 스레드 블로킹 호출을 파이버 블로킹으로 전환해 주는 Async API를 제공합니다. 이 API는 외부 executor를 사용하여 해당 블로킹 호출을 처리한 후 완료 이후의 코드 흐름을 다시 파이버화 합니다. 반환값 유무에 따라 runBlocking()과 callBlocking() 중 하나를 사용하면 됩니다. 또한 기본 개념에서 설명했듯이 이러한 파이버 블로킹 API는 Suspendable하므로 API 호출 메서드는 반드시 SuspendExecution 예외 시그니처를 명시해야 합니다.

```
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() throws SuspendExecution {

    Async.runBlocking(executor, runnable); // 스레드 블로킹 호출을 파이버 블로킹 호출로 전환

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() throws SuspendExecution {

    return Async.callBlocking(executor, callable);  // 스레드 블로킹 호출을 파이버 블로킹 호출로 전환

}
```

### **12-2.Future 처리**

Future에 대한 대기는 스레드 블로킹을 유발합니다. 예를 들어 아래와 같은 코드는 호출 스레드를 블로킹합니다.

```
Future<SomeObject> future = someAsyncJob();

SomeObject ret = future.get(); // 스레드 블로킹을 유발
```

GameAnvil은 이런 future에 대한 대기를 스레드 블로킹에서 파이버 블로킹으로 전환해주는 API를 제공합니다. 단, 이 API들은 Java의 CompletableFuture와 Guava의 ListenableFuture만 지원합니다. 다행히도 대부분의 라이브러리는 이 2가지의 future를 기반으로 비동기를 지원하기 때문에 큰 무리 없이 적용 가능할 것입니다. 아래의 코드는 이러한 Async API를 이용해서 future에 대한 대기를 파이버 블로킹으로 처리하는 예입니다.

```
Future<SomeObject> future = someAsyncJob();

SomeObject ret = Async.awaitFuture(future); // 해당 파이버만 블로킹
```

### **12-3.블로킹 처리 위임**

앞서 블로킹 호출에 대한 처리를 살펴보았습니다. Async API의 runBlocking()이나 callBlocking()은 블로킹 처리를 완료한 이후에 다시 해당 파이버의 실행 흐름을 이어가는 경우에 사용합니다. 반면 외부 스레드로 블로킹 호출을 위임한 후 그 결과에 대해 신경 쓸 필요가 없다면 실행 흐름을 계속 이어갈 수 있을 것입니다. 이런 경우에는 아래의 API를 사용하면 됩니다. 이 API는 블로킹 호출의 결과를 대기하지 않으므로 Suspendable하지 않음에 주의하세요.

```
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() { // NOT suspendable

    Async.exec(executor, runnable); // 외부 스레드로 블로킹 호출을 위임했으므로 이 파이버는 블로킹되지 않는다.

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() {  // NOT suspendable

    return Async.exec(executor, callable);  // 외부 스레드로 블로킹 호출을 위임했으므로 이 파이버는 블로킹되지 않는다.

}
```



## 13. 비동기 Redis 지원

그동안 어떤 종류의 Redis 클라이언트를 사용할지는 전적으로 GameAnvil 사용자의 선택이었습니다. 하지만 이로 인해 Redis 관련 이슈의 종류와 복잡도가 사용자가 선택한 Redis 클라이언트 종류와 사용 방식의 차이에 비례해서 증가한다는 사실을 알게 되었습니다. 우리는 이를 방지하고자 GameAnvil의 기본적인 가이드라인에 Redis 클라이언트에 관한 사용법을 포함하기로 결정했습니다. 이 가이드라인은 GameAnvil에서 지원하는 Redis 클라이언트의 종류와 그 기본적인 사용법을 API화하여 통일시키는 것을 목표로 합니다. 물론 제공되는 API가 아닌 다른 종류의 Redis 클라이언트를 선택해서 별도로 사용하는 것도 가능하지만 틀별한 이유가 없다면 지양하길 권합니다.

### **[Note]**

*이후의 내용에서 GameAnvil에서 제공하는 Lettuce 클래스와 제품명인 "Lettuce"를 구분하기 위해 전자의 경우는 가능한 "Lettuce 클래스"라고 표기하고 일부 내용상 필요에 의해 그냥 "Lettuce"로 표기할 수도 있습니다. 이와 구분하기 위해 제품명은 전체 대문자 ***LETTUCE***로 표기합니다. 이 글에서 설명하는 LETTUCE는 GameAnvil에서의 사용 방법에 포커스를 두기 때문에 그 이상의 설명이 필요할 경우에는 [LETTUCE 공식 페이지](https://github.com/lettuce-io/lettuce-core)를 참고하세요.*

GameAnvil은 Redis 클라이언트로 LETTUCE의 사용을 권장합니다. GameAnvil에서 제공하는 Redis 래핑 API 또한 LETTUCE를 사용합니다. 참고로 LETTUCE는 비동기 Redis 클라이언트로서 대부분의 비동기 API는 CompletableFuture를 기반으로 합니다. 이는 곧 GameAnvil의 Async API를 이용해서 파이버 기반의 비동기화로 전환할 수 있음을 의미합니다.

GameAnvil에서 제공하는 Redis 래핑 API는 크게 3가지의 클래스인 Lettuce, RedisCluster 그리고 RedisSingle로 나뉩니다. Lettuce는 가장 일반적인 형태의 사용법을 제공하며 내부적으로 LETTUCE 객체를 관리하지 않는 static 클래스입니다. 그러므로 LETTUCE를 가장 일반적인 형태로 사용하고 싶을 경우에는 이 Lettuce 클래스가 가장 적합합니다. RedisCluster와 RedisSingle은 각각 Redis 클러스터와 스탠드얼론에 대응하기 위한 클래스로서 내부적으로 LETTUCE 객체들을 관리합니다.

### **13-1.Lettuce**

Lettuce 클래스는 파이버 단위의 처리를 위한 가장 핵심적인 static API들을 제공합니다. 내부적으로 Redis에 관한 그 어떤 상태도 보관하지 않으므로 별도의 객체를 만들 필요가 없이 바로 사용이 가능합니다. 만일 Lettuce 라이브러리에 대해 어느 정도 익숙하다면 Lettuce 클래스를 직접 사용하는 것이 가장 좋습니다.

```
import com.nhn.gameanvil.async.redis.Lettuce;
```

다음의 3가지 주의 사항 외에는 기본적인 Lettuce 사용법을 그대로 유지할 수 있습니다.

- 첫째, 반드시 connect는 GameAnvil의 Lettuce.connect() 혹은 Lettuce.connectAsync()를 사용한다. 커넥션은 기본적으로 스레드를 블로킹하므로 이에 대한 파이버화 처리를 포함합니다.
- 둘째, shutdown 또한 커넥션과 동일한 이유로 Lettuce.shutdown()을 사용해야 합니다.
- 셋째, RedisFuture에 대한 대기는 반드시 Lettuce.awaitFuture()를 사용해서 파이버 블로킹화해야 합니다.

이런 Lettuce 클래스를 사용하여 Redis에 접속하는 코드는 아래와 같습니다.

```
RedisURI clusterURI = RedisURI.Builder.redis(IP_ADDRESS, 7500).build();
clusterClient = RedisClusterClient.create(Arrays.asList(clusterURI));
clusterConnection = Lettuce.connect(RpsConfig.DB_THREAD_POOL, clusterClient);

if (clusterConnection.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### **13-2.RedisCluster**

```
import com.nhn.gameanvil.async.redis.RedisCluster;
```

Redis Cluster에 대한 API를 래핑합니다. 기본적으로 앞서 설명한 Lettuce 와 사용법은 크게 다르지 않습니다. 하지만 이 클래스는 Lettuce 관련 객체들(e.g.RedisClusterClient, StatefulRedisClusterConnection 등)을 자체적으로 관리합니다. 이러한 Lettuce 객체들을 직접 관리하기 보다 RedisCluster를 통해 관리하고 싶다면 사용을 고려해보세요.

주의 사항은 Lettuce의 경우와 완전히 동일합니다. 아래는 RedisCluster를 이용해서 Redis에 접속하는 코드입니다.

```
redisClient = RedisSingle.create("redis://IP_ADDRESS:6379");
redisAsyncCommands = Lettuce.connect(RpsConfig.DB_THREAD_POOL, redisClient).async();

if (redisClient.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### **13-3.RedisSingle**

```
import com.nhn.gameanvil.async.redis.RedisSingle;
```

RedisCluster와 비교했을 때, 대상 Redis가 스탠드얼론이라는 차이점 밖에 없습니다.

주의 사항은 Lettuce의 경우와 완전히 동일합니다. 아래는 RedisSingle을 이용해서 Redis에 접속하는 코드입니다.

```
redisCluster = new RedisCluster<>(IP_ADDRESS, 7500);
redisCluster.connect(RpsConfig.DB_THREAD_POOL, StringCodec.UTF8);

if (redisCluster.isConnected()) {
    logger.warn("============= Connected to Redis using Lettuce =============");
}
```

### **13-4.RedisFuture를 파이버에서 사용하기**

Lettuce, RedisCluster 그리고 RedisSingle은 모두 Lettuce 라이브러리가 지원하는 RedisFuture를 파이버 상에서 대기할 수 있는 API를 제공합니다. 내부 구현은 모두 엔진에서 제공하는 Async.awaitFuture()를 동일하게 사용하므로 혼용해도 무방합니다. 아래의 4가지 코드는 모두 동일한 코드입니다. GameAnvil의 파이버 상에서 RedisFuture에 대한 get()은 반드시 이 4가지 중 하나의 방법을 사용해야 합니다.

- Async.awaitFuture()

```
try {
    Async.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- Lettuce.awaitFuture()

```
try {
    Lettuce.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- RedisCluster.awaitFuture()

```
try {
    redisCluster.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- RedisSingle.awaitFuture()

```
try {
    redisSingle.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- **잘못된 사용법**: 직접 Future에 대한 대기를 할 경우 해당 Node(Thread)가 블로킹되므로 절대 아래와 같은 코드는 사용하면 안됩니다.

```
try {
    RedisFuture future = clusterAsyncCommands.mget("testKey", getUserId()));
    future.get(); // 스레드 블로킹을 유발
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

### **13-5.set/get**

가장 기본이 되는 set과 get은 RedisCluser와 RedisSingle에서 기본 제공합니다.

- RedisCluster를 이용한 set/get 예제

```
String setResult = redisCluster.set(key, value);
String getResult = redisCluster.get(key);
```

- RedisSingle을 이용한 set/get 예제

```
String setResult = redisSingle.set(key, value);
String getResult = redisSingle.get(key);
```

- 직접 LETTUCE의 RedisAsyncCommands 객체를 사용한 예제

```
RedisFuture<String> setFuture = redisAsyncCommands.set(key, value);
RedisFuture<String> getFuture = redisAsyncCommands.get(key);

String setResult = Async.awaitFuture(setFuture);
String getResult = Async.awaitFuture(getFuture);
```

### **13-6.본격적인 LETTUCE 비동기 처리**

Redis가 제공하는 다양한 커맨드들은 LETTUCE의 Commands 객체를 통해 사용 가능합니다. 기본적으로 LETTUCE는 Sync방식의 Commands 객체과 Async방식의 Commands 객체를 제공하는데 GameAnvil은 그중 Aync방식의 사용을 권장합니다. 기본적으로 AsyncCommands는 Redis Cluster인 경우와 StandAlone인 경우에 대해 각각 아래와 같습니다.

- RedisAdvancedClusterAsyncCommands
- RedisAsyncCommands

아래의 예제들은 이런 AsyncCommands 객체를 이용하여 mget을 수행하는 예제들입니다. LETTUCE의 비동기 처리는 기본적으로 RedisFuture를 사용하고 이 RedisFuture는 CompletableFuture입니다. CompletableFuture에 대한 자세한 내용은 [Java 공식 레퍼런스](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)에서 확인 가능합니다. 참고로 아래의 예제들은 LETTUCE에 대한 비동기 처리의 극히 일부 방식만을 보여주고 있으므로 그대로 사용하기 보다는 개발중인 코드에 알맞게 작성하세요. 완벽한 비동기 코드의 제어를 위해서는 반드시 [LETTUCE](https://github.com/lettuce-io/lettuce-core)와 [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)에 대한 내용을 숙지해야 합니다.

### **[Note]**

thenApply()와 thenAccept() 등은 임의의 외부 스레드에서 호출되므로 Node에서 관리하는 내부 리소스에 대한 접근을 하거나 리소스에 대한 Lock을 사용하면 안됩니다.

- 예제1> key1과 key2에 대한 값을 비동기로 획득

```
Lettuce.awaitFuture(asyncCommands.mget("key1", "key2"));
```

- 예제2> 이 후의 코드 흐름과 상관없는 경우 future chain으로 외부 스레드에 처리를 위임 (즉, mget으로 값 획득을 완료할 때까지 대기할 필요가 없을 경우)

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

future.thenApplyAsync(r -> {
    Map<String, String> map = new HashMap<>();
    for (KeyValue<String, String> kv : r)
        map.put(kv.getKey(), kv.getValue());
    return map;
}).thenAccept(r -> {
    for (Entry<String, String> entry : r.entrySet())
        logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
});
```

- 예제3> 이후의 코드 흐름과 상관있는 경우에 해당 future를 대기한 후 처리

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

CompletionStage<Map<String, String>> cs = future.thenApplyAsync(r -> {
    Map<String, String> map = new HashMap<>();
    for (KeyValue<String, String> kv : r)
        map.put(kv.getKey(), kv.getValue());
    return map;
});

// do something here

try {
    // 파이버 상에서 해당 future를 대기하기 위해 Lettuce.awaitFuture()를 사용해야 함을 명심하세요
    Map<String, String> map = Lettuce.awaitFuture(cs);

    for (Entry<String, String> entry : map.entrySet())
        logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin()", e);
}
```



## 14. 비동기 HttpReqeust & HttpResponse 사용법

Http 처리에 관한 부분도 Redis와 마찬가지로 GameAnvil에서 기본적인 API와 가이드라인을 제공합니다. 물론 다른 종류의 Http 사용법 역시 선택이 가능하지만 특별한 이유가 없다면 지양하길 권합니다. GameAnvil은 비동기 기반의 Http 사용을 위해 내부적으로 [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)를 사용합니다. 다음에서 설명할 API와 그 사용 범위를 넘는 경우에는 저희가 제공하는 API 보다 직접 [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client)를 사용하길 권합니다. LETTUCE와 마찬가지로 AsyncHttpClient도 내부적으로 CompletableFuture를 사용하므로 future에 대한 대기를 Async.awaitFuture()를 이용해서 파이버화 해주기만 하면 나머지는 일반 스레드 상에서의 사용법과 완전히 동일합니다.

```
Async.awaitFuture(future.get()); // 파이버 상에서 해당 future를 대기합니다.
```

GameAnvil에서 제공하는 Http API는 요청과 응답을 위한 HttpRequest, HttpResponse 클래스 그리고 결과에 대한 일반적인 처리를 위한 HttpResultTemplate 클래스로 이루어집니다. 이 클래스들을 이용하면 간단하고 직관적으로 Http 요청과 응답을 처리할 수 있으며 그 결과를 원하는 형태로 취할 수도 있습니다. 또한 모든 코드는 비동기이므로 특별한 처리가 필요없습니다. 다음은 이를 사용한 예제 코드들입니다.

- 예제1> 가장 기본적인 사용법 내부적으로 파이버 단위의 future 처리를 알아서 해주므로 가장 직관적인 방식입니다. 특별한 이유가 없다면 이러한 기본적인 사용법만으로도 충분합니다.

```
HttpRequest request = new HttpRequest(URL);
HttpResponse response = request.GET();
```

- 예제2> future 기반의 비동기 방식 HTTP 요청과 응답 대기 사이에 다른 작업을 하고 싶을 경우 아래와 같이 future를 직접 이용할 수 있습니다.

```
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

// Do something here

HttpResponse response = new HttpResponse(Async.awaitFuture(future, 10000, TimeUnit.MILLISECONDS));
```

- 예제3> HTTP 요청 header 구성 아래의 예제와 같이 AsyncHttpClient는 다양한 API를 제공합니다. AsyncHttpClient에 대한 자세한 사용법은 [공식 페이지](https://github.com/AsyncHttpClient/async-http-client)를 참고하세요.

```
HttpRequest request = new HttpRequest(url);
request.getBuilder()
    .addHeader("if-none-check-node", "false")
    .setRequestTimeout(timeout);
    .addQueryParam("serviceId", serviceId);

HttpResponse httpResponse = request.GET();
```

- 예제4> 이후의 코드 흐름과 상관없는 경우 future chain으로 외부 스레드에 처리를 위임 (Lettuce의 경우와 동일한 방식)

```
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

future.thenApplyAsync(r -> {
    try {
        HttpResponse response = new HttpResponse(r);
        String body = response.getContents(String.class);
        HttpResultTemplate<JsonObject> result = GameAnvilUtil.Gson().fromJson(body, new RestResponseParamType(JsonObject.class));
        if (!result.getHeader().getIsSuccessful()) {
            logger.warn("GET failed : resultCode {}, resultMessage {}",
                result.getHeader().getResultCode(),
                result.getHeader().getResultMessage());
        }
        return result.getContents();
    } catch (IOException e) {
        logger.error("Exception occurred: ", e);
        return null;
    }
}).thenAccept(r -> {
    if (r != null) {
        JsonElement element = r.get(ELEMENT_NAME);
        if (element != null) {
            logger.info("The response code: {}", element.getAsString());
        }
    }
});
```

- 예제5> 이후의 코드 흐름과 상관있는 경우에 해당 future를 대기한 후 처리

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

CompletionStage<JsonObject> cs = future.thenApplyAsync(r -> {
    try {
        HttpResponse response = new HttpResponse(r);
        String body = response.getContents(String.class);
        HttpResultTemplate<JsonObject> result = GameAnvilUtil.Gson().fromJson(body, new RestResponseParamType(JsonObject.class));
        if (!result.getHeader().getIsSuccessful()) {
            logger.warn("GET failed : resultCode {}, resultMessage {}",
                result.getHeader().getResultCode(),
                result.getHeader().getResultMessage());
        }
        return result.getContents();
    } catch (IOException e) {
        logger.error("GameUser::onLogin()", e);
        return null;
    }
});

// do something here

try {
    // 파이버 상에서 해당 future를 대기하기 위해 Async.awaitFuture()를 사용해야 함을 명심하자.
    JsonObject jsonObject = Async.awaitFuture(cs);
    if (jsonObject != null) {
        JsonElement element = jsonObject.get(ELEMENT_NAME);
        if (element != null) {
            logger.info("The response code: {}", element.getAsString());
        }
    }
} catch (TimeoutException e) {
    logger.error("Exception occurred: ", e);
}
```



## 15. RDBMS 비동기 처리

RDBMS에 대한 쿼리는 일반적으로 블로킹입니다. 이런 블로킹 쿼리를 GameAnvil 상에서 처리하는 방법은 앞서 살펴보았던 다른 Async 사용법과 크게 다르지 않습니다. 어떤 종류의 RDBMS를 사용하던 SQL 쿼리에 대한 코드는 동일한 방법으로 구현할 수 있습니다. 또한 엔진 사용자는 DB 접근을 위해 자유롭게 SQL Mapper나 ORM 등을 선택할 수 있습니다.



### Note

*DB에 대한 쿼리를 구현하는 과정에서 가장 중요하지만 흔히 놓치는 부분은 DB에 대한 CP(ConnectionPool) 크기와 이를 비동기로 처리할 TP(ThreadPool)의 개수에 대한 설정과 이들 사이의 관계에 대한 이해입니다. 일반적으로 이들 두 수치는 처리할 쿼리의 양을 고려하여 동일한 값으로 설정하거나 TP를 CP보다 조금 더 넉넉하게 설정하면 됩니다. 참고로 GameAnvil를 이용한 대규모 성능 테스트 결과, 서버 프로세스 하나 당 6000~8000명 처리 기준 TP와 CP 250개 설정이 가장 좋은 결과를 보여주었습니다. 이는 어디까지나 쿼리 복잡도와 빈도 등 복합적인 요소를 고려하여 가능한 많은 테스트를 거쳐 최적의 값을 찾는 것이 최선입니다.*



### 15-1.Async Query

우선 쿼리에 대한 비동기 처리는 크게 2가지로 나눌 수 있습니다. 쿼리의 결과가 필요한 경우와 그렇지 않은 경우입니다. 이 두 경우는 쿼리의 결과 유무 차이만 있을 뿐 전체 쿼리 수행이 완료될 때까지 해당 파이버가 대기하는 것은 동일합니다. 즉, 비동기로 요청한 쿼리가 완료된 후 다음 코드로 진행되므로 엔진 사용자는 일반적인 블로킹 코드를 작성하듯이 구현할 수 있습니다.



첫째, 쿼리의 결과를 획득하고자 할 경우에는 다음의 예제와 같이 Async 클래스의 callBlocking API를 사용합니다. callBlocking은 파이버 상에서 임의의 블로킹 호출을 수행한 후 결과를 반환합니다.

```
try {
    return Async.callBlocking("MyThreadPool", new Callable<List<T>>() {
        @Override
        public List<T> call() throws Exception {
            return myQueryCode();
        }
    });
} catch (TimeoutException e) {
    logger.error("TimeoutException occured: ", e);
}

logger.info("Query has finished.");
```

이때, 비동기 처리를 위한 스레드풀은 Bootstrap 단계에서 미리 생성해둘 수 있습니다.

```
bootstrap.createExecutorService("MyThreadPool", 250);
```

혹은 엔진 사용자가 필요에 따라 직접 생성한 외부 스레드풀을 사용할 수도 있습니다.

```
bootstrap.createExecutorService(myExecutorService, 250);
```



둘째, 쿼리의 결과가 필요 없는 경우에는 다음의 예제와 같이 Async 클래스의 runBlocking API를 사용합니다. runBlocking은 파이버 상에서 임의의 블로킹 호출을 수행합니다.

```
try {
    Async.runBlocking("MyThreadPool", new Runnable() {
        @Override
        public void run() {
            try {
                myQueryCode();
            } catch (Exception e) {
                logger.error("Exception occured during query code: ", e);
            }
        }
    });
} catch (TimeoutException e) {
    logger.error("TimeoutException occured: ", e);
}

logger.info("Query has finished.");
```

이 경우도 마찬가지로 임의의 스레드풀을 runBlocking API에 매개변수로 전달할 수 있습니다.
