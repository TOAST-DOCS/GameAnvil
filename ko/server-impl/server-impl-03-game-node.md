## Game > GameAnvil > 서버 개발 가이드 > 게임 노드 구현



## 1. Game Node란?

![GameNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gamenode_on_network.png)

GameNode는 실제 게임 객체가 생성되고 게임 컨텐츠를 구현하는 노드입니다. 클라이언트는 GatewayNode에서 인증을 완료한 후, GameNode에 로그인을 완료해야 비로소 이러한 게임 컨텐츠를 시작할 수 있습니다.

아래의 이미지에서 보듯이, 클라이언트는 고유한 AccountId로 인증을 완료한 커넥션을 기반으로 여러개의 논리 세션을 생성할 수 있습니다. 이 때, 세션은 클라이언트와 유저 객체 사이에 만들어집니다. 아래의 그림에서 빨간색 점선 화살표가 이러한 세션을 나타냅니다.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

각각의 세션은 해당 커넥션 내에서 고유한 값으로 구분할 수 있으며 우리는 이 값을 SubId라고 부릅니다. 즉, AccountId와 SubId의 조합으로 사용자는 원하는 만큼 세션을 생성할 수 있습니다. 이 때, 동일한 서비스에 대한 세션도 마찬가지로 원하는 만큼 생성할 수 있습니다. 즉, 이미지의 Account 1 커넥션은  "Game" 서비스 혹은 "Chat" 서비스에 대한 세션을 얼마든지 추가로 생성할 수도 있습니다.

이러한 세션이 향하는 곳은 바로 유저 객체입니다. GameNode는 이러한 유저 객체와 그들의 그룹인 방 객체를 관리합니다. 이번 챕터는 이러한 GameNode와 GameUser 그리고 GameRoom에 대해 설명합니다.



## 2. GameNode 구현

GameNode는 BaseGameNode 클래스를 상속하여 구현합니다.  아래의 예제 코드는 GameNode에서 기본적으로 재정의할 수 있는 콜백 메서드를 보여줍니다. 노드 공통 콜백과 더불어 채널 관리를 위한 콜백이 존재합니다.

모든 노드는 사용자 정의 메시지를 처리하기 위한 디스패처 생성과 메시지 핸들러 등록 과정이 필요합니다. 특히, GameNode는 게임 컨텐츠를 위해 이러한 과정이 필수입니다.  우선, (1) 정적 패킷 디스패처를 하나 생성합니다. 이 때, 반드시 메모리나 성능 측면에서 이점을 취할 수 있도록 정적(static)으로 생성합니다. (2) 그리고 처리하고 싶은 메시지를 구현해둔 [핸들러](server-07-message-handling#11)와 연결합니다. (3) 마지막으로 onDispatch 콜백에서 (1)에서 생성한 디스패처를 이용하여 패킷을 처리합니다. 이 때, 디스패칭 뿐만 아니라 패킷에 대한 추가적인 처리를 할 수도 있습니다.

이러한 일련의 과정을 아래의 예제 코드에서 (1)~(3)에 해당하는 주석 바로 아래의 코드를 통해 살펴볼 수 있습니다.

또한 이렇게 구현한 클래스를 @ServiceName 어노테이션을 사용하여 특정 서비스에 대한 용도로 엔진에 등록합니다. 하나의 GameNode 클래스는 오직 하나의 서비스에 대해서만 등록할 수 있습니다.

```java
@ServiceName("MyGame") // "MyGame"이라는 서비스를 위한 GameNode로 엔진에 등록
public class SampleGameNode extends BaseGameNode {

    // (1) 패킷 디스패처 생성  
    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    // (2) SampleGameNode에서 처리하고 싶은 프로토콜과 핸들러를 매핑
    static {
        packetDispatcher.registerMsg(SampleGame.GameNodeTest.getDescriptor(), _GameNodeTest.class);
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onPrepare() throws SuspendExecution {
    }

    /**
     * 패킷이 전달될 때 호출
     *
     * @param packet 처리할 패킷
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {
        // (3) SampleGameNode의 메시지 처리. 필요할 경우 패킷에 대한 추가 처리도 여기에서 할 수도 있습니다.
        if (packetDispatcher.isRegisteredMessage(packet))
            packetDispatcher.dispatch(this, packet);
    }

    /**
     * pause 될 때 호출
     *
     * @param payload contents 에서 전달하고자 하는 추가 정보
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {
    }

    /**
     * resume 될 때 호출
     *
     * @param payload contents에서 전달하고자 하는 추가 정보
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onResume(Payload payload) throws SuspendExecution {
    }
  
    /**
     * shutdown 명령을 받으면 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onShutdown() throws SuspendExecution {
    }

    /**
     * 같은 채널의 다른 노드에서 유저 변화가 발생할 때 호출
     * 즉, updateChannelUser() API 호출시 발생
     *
     * @param type            Channel 정보 변경 타입(갱신/삭제) 전달.
     * @param channelUserInfo 변경될 User 정보 전달.
     * @param userId          변경 대상의 User Id 전달.
     * @param accountId       변경 대상의 Account Id 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {  
    }

    /**
     * 같은 채널의 다른 노드에서 방 상태 변화가 발생할 때 호출
     * 즉, updateChannelRoomInfo() API 호출시 발생
     *
     * @param type            Channel 정보 변경 타입(갱신/삭제) 전달.
     * @param channelRoomInfo 변경될 Room 정보 전달.
     * @param roomId          변경 대상의 Room Id 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType type, ChannelRoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {  
    }

    /**
     * 클라이언트에서 채널 정보 요청 시 호출
     *
     * @param outPayload Client 로 전달될 Channel 정보 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {  
    }
}
```

이러한 GameNode의 주 목적은 노드에 접속된 모든 GameUser와 GameRoom 객체들에 대한 처리를 수행하는 것입니다. 이에 대해서는 바로 달아서 설명하도록 하겠습니다.

공통 콜백을 제외한 콜백의 의미와 용도는 아래의 표를 참고하세요.


| 콜백 이름               | 의미                  | 설명                                                                                                                                                                                                                                                      |
| ------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onChannelUserInfoUpdate | 채널의 유저 정보 갱신 | 같은 채널로 묶인 여러 개의 GameNode 중, 하나의 GameNode에서 채널의 유저 정보가 변경되었을 때 같은 채널 내의 나머지 모든 GameNode에서 동기화를 위하여 호출됩니다. 이 때, 사용자는 전달받은 정보를 바탕으로 현재 GameNode의 채널 정보를 갱신할 수 있습니다. |
| onChannelRoomInfoUpdate | 채널의 방 정보 갱신   | 같은 채널로 묶인 여러 개의 GameNode 중, 하나의 GameNode에서 채널의 방 정보가 변경되었을 때 같은 채널 내의 나머지 모든 GameNode에서 동기화를 위하여 호출됩니다. 이 때, 사용자는 전달받은 정보를 바탕으로 현재 GameNode의 채널 정보를 갱신할 수 있습니다.   |
| onChannelInfo           | 채널 정보 요청        | 클라이언트가 채널 정보를 요청할 때 호출됩니다. 사용자는 이 콜백에서 원하는대로 채널 정보를 구성하여 클라이언트로 전달할 수 있습니다.                                                                                                                      |



## 3. 유저 구현

유저 객체는 로그인 과정을 거쳐 GameNode에 생성됩니다. 유저 기반의 컨텐츠는 이 클래스를 중심으로 구현하는 것이 바람직합니다. 앞서 살펴본 모든 예제와 마찬가지로 유저 또한 처리할 고유의 메시지와 핸들러를 연결할 수 있습니다. 아래의 예제 코드를 보면 유저는 꽤 많은 콜백 메서드를 제공하는 것을 볼 수 있습니다. 이 중 일부는 기본 구현이 제공되므로 특별히 필요한 상황이 아니라면 재정의하지 않아도 됩니다. 이는 유저 뿐만 아니라 엔진에서 제공하는 대부분의 클래스에 해당합니다.

특히, 유저는 엔진에 등록하기 위한 어노테이션도 다른 클래스에 비해 많이 요구합니다. 우선, 어떤 게임 서비스를 위한 유저인지 등록 후, 유저 타입을 등록합니다. 이 유저 타입은 이름 그대로 유저의 종류를 구별하기 위한 용도로서, 클라이언트에서도 마찬가지로 API를 호출할 때 사용됩니다. 즉, 해당 API가 서버의 어떤 유저 타입에 대한 호출인지 명시하는 것입니다. 그러므로 반드시 서버와 클라이언트 사이에 이러한 유저 타입을 임의의 문자열로 사전 정의해두어야 합니다. 이 예제 코드에서는 "BasicUser"라는 유저 타입을 사용합니다. 이에 대한 더 자세한 설명은 별도의 챕터에서 다시 다루도록 합니다. 마지막으로 이 유저 정보가 [채널간 유저 정보 동기화](server-09-channel#3)에 사용될지 여부를 결정할 수 있습니다. 만일 채널간 정보 동기화가 필요없다면 @UseChannelInfo 어노테이션은 생략할 수 있습니다.

```java
@ServiceName("MyGame")
@UserType("BasicUser")
@UseChannelInfo
public class SampleGameUser extends BaseUser {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameUserTest.getDescriptor(), _GameUserTest.class);
    }

    /**
     * 로그인할 때 호출
     *
     * @param payload        클라이언트에서 전달받은 정보
     * @param sessionPayload onPreLogin에서 전달된 페이로드
     * @param outPayload     클라이언트로 전달할 정보
     * @return 로그인이 성공이면 true를 반환 그렇지 않으면 false를 반환
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onLogin(final Payload payload,
                           final Payload sessionPayload,
                           Payload outPayload) throws SuspendExecution {    
    }

    /**
     * 로그인 성공 이후에 필요한 후처리를 위해 호출 (즉, onLogin 혹은 onReLoin이 성공한 후 호출)
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onPostLogin() throws SuspendExecution {

    }

    /**
     * 이미 로그인된 상황에서 다른 디바이스로 같은 유저가 로그인할 때 호출
     *
     * @param newDeviceId           새로 접속한 유저의 deviceId 값
     * @param outPayloadForKickUser 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 새로 접속한 유저가 로그인 된 후 기존 유저는 강제 로그아웃 처리. false면 새로 접속한 유저가 로그인 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onLoginByOtherDevice(final String newDeviceId,
                                        Payload outPayloadForKickUser) throws SuspendExecution {
        return true;
    }

    /**
     * 임의의 유저 타입으로 이미 로그인 한 상태에서 다른 유저 타입으로 로그인을 시도할 때 호출
     *
     * @param userType   새로 로그인을 시도하는 유저의 타입
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 새로운 로그인이 성공하고 false이면 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onLoginByOtherUserType(final String userType,
                                          Payload outPayload) throws SuspendExecution {
        return true;
    }

    /**
     * 이미 로그인 된 상태에서 (재접속 등의 이유로) 다른 커넥션을 통해 로그인을 시도할 경우 호출
     *
     * @param outPayload 클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {    
    }

    /**
     * 이미 로그인 된 상태에서, 다시 로그인을 시도할 때 호출
     * (로그인 된 상태에서는 사용자의 GameUser 객체가 GameNode에 여전히 유효한 상태로 남아있음)
     *
     * @param payload        클라이언트에서 전달한 임의의 페이로드
     * @param sessionPayload onPreLogin에서 전달된 페이로드
     * @param outPayload     클라이언트로 전달할 임의의 페이로드
     * @return 반환값이 true이면 ReLogin 성공, false이면 ReLogin 실패
     * @throws SuspendExecution: 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onReLogin(final Payload payload,
                             final Payload sessionPayload,
                             Payload outPayload) throws SuspendExecution {
    }

    /**
     * 클라이언트와의 연결이 끊어졌을때 호출되는 콜백
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onDisconnect() throws SuspendExecution {    
    }

    /**
     * GameUser에 처리할 패킷이 있을 때 호출
     *
     * @param packet 처리할 패킷
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, packet);
    }

    /**
     * 유저가 속한 노드가 pause될 때, 해당 유저도 pause 되면서 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onPause() throws SuspendExecution {    
    }

    /**
     * 유저가 속한 노드가 resume될 때, 해당 유저도 resume 되면서 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onResume() throws SuspendExecution {    
    }

    /**
     * 유저가 logout할 때 호출
     *
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onLogout(final Payload payload, Payload outPayload) throws SuspendExecution {
    }

    /**
     * 해당 유저가 로그아웃 가능한지 확인하기 위해 호출
     * 엔진 사용자는 이 콜백에서 현재 GameUser가 로그아웃을 해도 문제가 없을지 결정할 수 있음
     *
     * @return 반환값이 false이면 로그아웃 진행이 멈추고, 이 후에 주기적으로 다시 콜백을 호출. 반환값이 true이면 로그 아웃을 진행
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean canLogout() throws SuspendExecution {
    }

    /**
     * 룸 매이메이킹 요청을 받으면 호출
     *
     * @param roomType 클라이언트와 서버 사이에 사전 정의한 방 종류를 구분하는 임의의 값
     * @param payload  클라이언트로부터 전달받은 페이로드
     * @return 매칭된 룸의 정보 반환를 반환. 만일 null이면 클라이언트의 요청 옵션에 따라 새로운 방이 생성될 수도 있고, 실패 처리될 수도 있음
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public MatchRoomResult onMatchRoom(final String roomType,
                                       final Payload payload) throws SuspendExecution {
        return null;
    }

    /**
     * 클라이언트에서 유저 매치메이킹을 요청했을 경우 호출되는 콜백
     *
     * @param roomType   클라이언트와 서버 사이에 사전 정의한 방 종류를 구분하는 임의의 값
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 유저 매치메이킹 요청 성공이고 false 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onMatchUser(final String roomType,
                               final Payload payload,
                               Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * 유저 매칭이 취소될 때 호출
     *
     * @param reason 취소된 이유. 일반적으로 타임아웃(TIMEOUT) 이거나 사용자의 요청에 의한 취소(CANCEL) 중 한 가지이다.
     * @return 반환값이 true이면 매칭 취소 처리가 성공이고 false이면 취소가 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
        return false;
    }

    /**
     * 처리할 타이머 핸들러를 등록하기 위한 콜백
     * 이 콜백이 호출되었을 때, 사용자는 처리하고 싶은 타이머 핸들러들을 원하는만큼 등록하도록 합니다.
     * (좀 더 자세한 내용은 "타이머" 챕터를 참고하도록 하세요)
     */
    @Override
    public void onRegisterTimerHandler() {    
    }

    /**
     * 유저가 다른 노드로 이동(전송) 가능한 상태인지 확인하기 위해 호출
     *
     * @return 반환값이 true이면 전송 가능한 상태이고 false이면 불가능한 상태이다. 불가능한 상태의 경우, 만일 NonStopPatch가 진행 중이라면 패치가 종료되기 전까진 해당 유저를 전송하기 위해 지속적으로 호출
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean canTransfer() throws SuspendExecution {
    }
    
    /**
	 * 유저가 다른 노드로 이동(전송)할 때, 소스 노드에서 전달할 데이터를 꾸리기 위해서 호출
	 * (좀 더 자세한 내용은 "전송 가능 객체" 챕터를 참고하세요.)
     *
     * @param transferPack 다른 노드로 가지고 갈 데이터를 저장하기 위한 꾸러미
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onTransferOut(final TransferPack transferPack) throws SuspendExecution {    
    }

    /**
     * 유저가 다른 노드로 이동(전송)할 때, 대상 노드에서 새롭게 생성된 유저 객체를 원래의 상태로 복원하기 위해 호출
     * (좀 더 자세한 내용은 "전송 가능 객체" 챕터를 참고하세요.)
     * 
     * 이 시점에는 유저가 완전히 복구되지 않았으므로, 다른 곳(노드, 유저 등)으로의 메시지 요청이 제한된다.
     *
     * @param transferPack 다른 노드에서 가지고 온 데이터 꾸러미
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onTransferIn(final TransferPack transferPack) throws SuspendExecution {    
    }

    /**
     * 노드 간 유저 이동(전송)이 완료된 후 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onPostTransferIn() throws SuspendExecution {    
    }

    /**
     * 클라이언트에서 다른 채널로 이동 요청을 할 때, 현재 유저가 채널 이동이 가능한 상태인지 확인하기 위해 호출
	 *
	 * 주의> 만일, 사용자가 명시적으로 moveChannel() API를 호출하여 채널을 이동할 경우에는 onCheckMoveOutChannel()가 호출되지 않습니다. 오직 엔진에 의해 암묵적인 채널 이동이 발생할 때 호출됩니다.
     *
     * @param destinationChannelId 이동 대상 채널의 ID
     * @param payload              클라이언트로부터 전달받은 페이로드
     * @param errorPayload         채널 이동을 실패할 경우, 서버에서 클라이언트로 전달하는 페이로드. (성공일 경우에는 전달 안됨)
     * @return 반환값이 false이면 채널 이동이 불가능하므로 요청은 실패이고 true면 성공
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onCheckMoveOutChannel(final String destinationChannelId,
                                         final Payload payload,
                                         Payload errorPayload) throws SuspendExecution {
        return false;
    }

    /**
     * 다른 노드로 채널 이동을 할 때, 소스 노드에서 호출
     *
     * @param destinationChannelId 이동할 channel ID
     * @param outPayload           이동할 channel에 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onMoveOutChannel(final String destinationChannelId,
                                 Payload outPayload) throws SuspendExecution {
    }

    /**
     * 다른 노드로 채널 이동이 완료된 후 소스 노드에서 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onPostMoveOutChannel() throws SuspendExecution {
    }

    /**
     * 다른 노드로 채널 이동을 할 때, 대상 노드로 진입하면서 호출
     *
     * @param sourceChannelId 이동하기 전의 채널 ID
     * @param payload         클라이언트로부터 전달받은 페이로드
     * @param outPayload      클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onMoveInChannel(final String sourceChannelId,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * 다른 노드로 채널 이동이 완료된 후 대상 노드에서 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onPostMoveInChannel() throws SuspendExecution {
    }
}
```

위의 예제 코드에서 onLogin으로 시작하는 콜백은 모두 로그인과 관련되서 호출됩니다. 예를 들어 최초의 로그인 요청에 대해서는 onLogin 콜백이 호출되며, 이미 로그인이 되어 있는 상태에서 재로그인을 처리하는 경우는 onReLogin 콜백이 호출됩니다. 마찬가지로 로그아웃을 처리할 때에는 onLogout 콜백이 호출됩니다. 이처럼 GameAnvil의 콜백은 그 이름과 JavaDoc 주석을 통해 그 용도가 대부분 명확하게 설명이 됩니다.

유저는 언제든 채널 사이에서 이동이 가능합니다. 이러한 채널에 관련된 내용은 뒤에서 [별도의 챕터](server-09-channel)로 따로 설명할 것이므로 여기에서는 우선 넘어가도록 하겠습니다. 뿐만 아니라 유저는 여러 개의 GameNode 사이에서 전송 가능한 객체입니다. 이와 관련한 기능 또한 [별도의 챕터](server-08-object-transfer)에서 더 자세하게 설명하도록 합니다.

GameAnvil은 두 종류의 매치메이킹 기능, 룸 매치메이킹과 유저 매치메이킹,을 제공합니다. 이러한 매치메이킹 요청이 유저로 도달하면 onMatchRoom 혹은 onMatchUser 콜백이 호출됩니다. 사용자는 이 콜백에서 GameAnvil이 제공하는 매치메이커를 사용할 수도 있고 직접 구현하거나 3rd 파티로 제공받은 다른 매치메이커를 사용할 수도 있습니다. 이에 대한 더 자세한 설명은 바로 [다음 챕터](server-04-match-node)에서 MatchNode를 다루면서 다시 하도록 하겠습니다.

이러한 유저의 콜백을 정리하면 아래의 표와 같습니다.


| 콜백 이름                | 의미                                     | 설명                                                                                                                                                                                                                                                            |
| -------------------------- | ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onLogin                  | 로그인                                   | 처음 로그인을 할 때 호출됩니다. 일반적으로 사용자는 이 콜백에서 DB 등의 저장소로부터 유저 정보를 가지고 와서 게임 유저 객체를 초기화하는 작업을 수행합니다.                                                                                                     |
| onPostLogin              | 로그인 성공 후처리                       | onLogin이 성공한 후에 호출됩니다. 로그인에 대한 후처리 작업을 이 콜백에서 할 수 있습니다.                                                                                                                                                                       |
| onLoginByOtherDevice     | 채널 정보 요청                           | 이미 로그인이 되어 있는 상태에서 다른 기기로 추가 로그인 요청이 왔을 때 호출됩니다. 사용자는 기존의 유저와 새로운 유저 중 어느쪽을 로그인 시킬지 반환값으로 결정할 수 있습니다.                                                                                 |
| onLoginByOtherUserType   | 다른 유저 타입으로 로그인 시도           | 이미 로그인이 되어 있는 상태에서 다른 유저 타입으로 로그인 요청이 왔을 때 호출됩니다. 새로운 유저 타입에 대해 로그인을 진행할지 여부를 반환값으로 결정할 수 있습니다.                                                                                           |
| onLoginByOtherConnection | 다른 커넥션으로 로그인 시도              | 이미 로그인이 되어 있는 상태에서 (재접속으로 인해) 이전과 다른 커넥션으로 로그인 시도를 할 경우에 호출됩니다. 만일, 재접속에 대한 추가 작업이 필요하다면 이 콜백에서 할 수 있습니다.                                                                            |
| onReLogin                | 재로그인                                 | 이미 로그인이 되어 있는 상태에서 다시 로그인을 할 경우에는 onLogin이 아닌 onReLogin이 호출됩니다. 즉, 재로그인에 대한 작업은 이 콜백에서 처리합니다.                                                                                                            |
| onDisconnect             | 접속 종료                                | 클라이언트로부터 접속이 끊겼을 때 호출됩니다. 이 때, 추가로 처리할 코드를 이 곳에 구현합니다.                                                                                                                                                                   |
| onDispatch               | 처리할 패킷이 있음                       | 유저에 처리할 패킷이 도달하면 호출됩니다. 사용자는 자신이 선언한 디스패처를 사용하거나 패킷에 대한 추가 작업을 진행할 수 있습니다. 자세한 내용은 [메시지 처리](server-07-message-handling#13-ondispatch)를 참고하세요.                                         |
| onPause                  | 일시 정지                                | 콘솔을 통해 GameNode를 일시 정지하면 해당 GameNode의 모든 유저에 대해 호출됩니다. 사용자는 노드가 일시 정지될 때 유저에서 추가로 처리하고 싶은 코드를 이 곳에 구현할 수 있습니다.                                                                               |
| onResume                 | 재개                                     | 콘솔을 통해 GameNode가 일시 정지 상태에서 다시 구동을 재개하면, 해당 GameNode의 모든 유저에 대해 호출됩니다. 사용자는 재개 상태에서 유저에 대해 처리하고 싶은 코드를 이 곳에 구현할 수 있습니다                                                                 |
| onLogout                 | 로그 아웃                                | 유저가 로그아웃할 때 호출됩니다. 이는 사용자가 명시적으로 요청한 로그아웃일 수도 있고, 접속 끊김 상태에서 설정된 시간을 초과할 경우에 엔진에 의해 자동으로 로그아웃될 수도 있습니다.                                                                            |
| canLogout                | 로그 아웃 가능성 확인                    | 해당 유저가 현재 로그아웃이 가능한 상태인지 체크하기 위해 호출됩니다. 만일 게임 플레이 중이거나 정보를 잃어서는 안되는 상황에는 false를 반환하여 로그아웃을 미룰 수 있습니다. 만일 false를 반환하면 엔진은 임의의 시간 이후에 지속적으로 이 콜백을 호출 합니다. |
| onMatchRoom              | 룸 매치메이킹 요청                       | 사용자가 룸 매치메이킹을 요청하면 호출됩니다. 이 때, 사용자는 이 콜백에서 엔진이 제공하는 룸 매치메이킹 API 혹은 제3의 매치메이킹 솔루션을 임의로 사용할 수 있습니다.                                                                                           |
| onMatchUser              | 유저 매치메이킹 요청                     | 사용자가 유저 매치메이킹을 요청하면 호출됩니다. 사용자는 이 콜백에서 엔진이 제공하는 유저 매치메이킹 API 혹은 제3의 매치메이킹 솔루션을 임의로 사용할 수 있습니다.                                                                                              |
| onMatchUserCancel        | 유저 매치메이킹 취소                     | 사용자가 이전에 신청한 유저 매치메이킹을 취소하면 호출됩니다. 이미 매칭이 완료된 상황처럼 취소를 할 수 없는 경우에는 실패할 수도 있습니다.                                                                                                                      |
| onRegisterTimerHandler   | 타이머 등록                              | 엔진 구동 과정에서 이 콜백이 호출되었을 때, 사용자는 처리하고 싶은 타이머 핸들러들을 원하는만큼 등록할 수 있습니다. 이렇게 등록된 타이머는 엔진에 의해 항시 관리되며 특히 유저 전송 시에도 자동으로 함께 전송됩니다.                                            |
| canTransfer              | 유저 전송이 가능한 상태인지 확인         | 해당 유저가 다른 노드로 전송될 수 있는 상태인지 체크하기 위해 호출됩니다.  만일 게임 플레이 중이거나 아직 준비가 안 된 경우에는 false를 반환하여 전송을 미룰 수 있습니다. false를 반환한 경우에는 엔진이 임의의 시간 이후에 지속적으로 이 콜백을 호출합니다.    |
| onTransferOut            | 기존 노드에서 전송되어 나갈 준비         | 유저가 다른 GameNode로 전송될 때, 소스 노드에서 전송을 시작할 때 호출됩니다. 사용자는 이 콜백에서 유저 객체와 함께 전송할 데이터 꾸러미를 꾸릴 수 있습니다.                                                                                                     |
| onTransferIn             | 새로운 노드로 전송 완료 처리             | 유저가 다른 GameNode로 전송될 때, 대상 노드에서 전송 완료하면서 호출됩니다. 사용자는 함께 가지고 온 데이터 꾸러미를 풀어서 원래의 유저 상태로 복구할 수 있습니다.                                                                                               |
| onPostTransferIn         | 전송 완료 후처리                         | 유저 전송이 성공한 경우, 대상 노드에서 후처리를 위해 호출됩니다.                                                                                                                                                                                                |
| onCheckMoveOutChannel    | 채널 이동이 가능한 상태인지 확인         | 유저가 다른 채널로 이동할 수 있는 상태인지 체크하기 위해 호출됩니다. 만일, 사용자가 명시적으로 moveChannel() API를 호출하여 채널을 이동하는 경우에는 호출되지 않습니다. 오직 엔진에 의해 암묵적인 채널 이동이 발생할 때만 호출됩니다.                           |
| onMoveOutChannel         | 기존 채널에서 다른 채널로 이동 준비      | 유저가 다른 채널로 이동할 때, 소스 노드에서 호출됩니다. 사용자는 원하는 정보를 outPayload에 담아서 대상 채널로 가지고 갈 수 있습니다.                                                                                                                           |
| onPostMoveOutChannel     | 기존 채널에서 다른 채널로 이동 준비 완료 | onMoveOutChannel이 성공하면 후처리를 위해 호출됩니다.                                                                                                                                                                                                           |
| onMoveInChannel          | 새로운 채널로 이동 처리                  | 유저가 다른 채널로 이동할 때, 대상 노드에서 호출됩니다. 사용자는 임의의 정보를 outPayload에 담아서 클라이언트로 전달할 수 있습니다.                                                                                                                             |
| onPostMoveInChannel      | 새로운 채널로 이동 완료                  | onMoveInChannel이 성공하면 후처리를 위해 호출됩니다.                                                                                                                                                                                                            |

### 3.1. 로그인이란?

앞서 설명한 내용과 예제 코드에서 로그인에 관한 내용이 자주 등장합니다. 또한 이러한 로그인은 클라언트가 서버에 접속한 후 GameNode에 자신의 유저 객체를 만드는 과정이라고 정의할 수 있습니다. 콜백 메서드 중 onLogin()은 최초에 유저를 생성하기 위해 로그인을 시도하는 과정에서 호출됩니다. 이 때, 사용자는 유저 객체를 구성하기 위한 정보를 DB 등으로부터 획득할 수 있습니다. 이러한 onLogin() 콜백이 성공하면 GameNode 상에 해당 유저 객체가 생성됩니다. 이렇게 로그인이 완료되면, 직접 정의한 프로토콜을 기반으로 클라이언트는 자신의 유저 객체를 통해 다른 객체들과 메시지를 주고 받으며 여러 가지 컨텐츠를 구현할 수 있습니다.

### 3.2. 로그아웃이란?

로그아웃은 로그인의 반대 개념입니다. 즉, GameNode 상에서 자신의 유저 객체를 제거하는 과정입니다. 로그아웃을 시작하면 해당 유저 객체는 onLogout() 콜백을 호출하여 메모리 상에서 삭제되기 전에 DB 등으로 자신의 최종 상태를 보관할 수 있습니다. 이러한 로그아웃은 클라이언트가 명시적으로 요청할 수도 있고, 클라이언트의 접속이 끊긴 상태에서 임의의 시간이 경과한 후 엔진에 의해 자동으로 처리되기도 합니다. 그러므로 만일 모바일 게임과 같이 잦은 접속 끊김이 예상될 경우에는 바로 로그아웃이 진행되지 않도록 적절한 [설정](server-16-config-vm#25-game)을 해둘 수 있습니다.



## 4. 방 구현

2명 이상의 유저는 방을 통해 동기화된 메시지 흐름을 만들 수 있습니다. 즉, 유저들의 요청은 방 안에서 모두 순서가 보장됩니다. 물론 1명의 유저를 위한 방 생성도 컨텐츠에 따라서 의미를 가질 수도 있습니다. 방을 어떻게 사용할지는 어디까지나 엔진 사용자의 몫입니다. 이러한 방은 유저와 마찬가지로 기본 클래스인 BaseRoom을 상속하여 여러 가지 콜백 메서드를 재정의할 수 있으며 자체적으로 메시지를 처리할 수도 있습니다. 아래의 예제 코드는 SampleGameUser를 위한 SampleGameRoom 클래스입니다.

방 역시 엔진에 등록하기 위해 여러 가지의 어노테이션이 필요합니다. 우선, 어떤 게임 서비스를 위한 방인지 등록 후, 방 타입을 등록합니다. 이 방 타입은 이름 그대로 방의 종류를 구별하기 위한 용도로서, 클라이언트에서도 마찬가지로 API를 호출할 때 사용됩니다. 즉, 해당 API가 서버의 어떤 방 타입에 대한 호출인지 명시하는 것입니다. 그러므로 반드시 서버와 클라이언트 사이에 이러한 방 타입을 임의의 문자열로 사전 정의해두어야 합니다. 이 예제 코드에서는 "BasicRoom"이라는 방 타입을 사용합니다. 이에 대한 더 자세한 설명은 별도의 챕터에서 다시 다루도록 합니다. 마지막으로 이 방 정보가 [채널간 방 정보 동기화](server-09-channel#3)에 사용될지 여부를 결정할 수 있습니다. 만일 채널간 정보 동기화가 필요없다면 @UseChannelInfo 어노테이션은 생략할 수 있습니다.

```java
@ServiceName("MyGame")
@RoomType("BasicRoom")
@UseChannelInfo
public class SampleGameRoom extends BaseRoom<SampleGameUser> {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameRoomTest.getDescriptor(), _GameRoomTest.class);
    }

    /**
     * 방이 초기화될 때 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onInit() throws SuspendExecution {    
    }

    /**
     * 방이 서버에서 사라질 때 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onDestroy() throws SuspendExecution {    
    }

    /**
     * GameRoom에 처리할 패킷이 있을 때 호출
     *
     * @param user   해당 패킷을 처리할 유저 객체. 해당 패킷을 처리할 유저가 없을 경우 null.
     * @param packet 처리할 패킷
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onDispatch(SampleGameUser user, final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, user, packet);
    }

    /**
     * 새로운 방을 생성할 때 호출
     *
     * @param user       요청한 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 방 생성이 성공이고 false이면 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onCreateRoom(SampleGameUser user,
                                final Payload inPayload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * 임의의 방에 참여할 때 호출
     *
     * @param user       요청한 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 입장 성공이고 false이면 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onJoinRoom(SampleGameUser user,
                              final Payload inPayload,
                              Payload outPayload) throws SuspendExecution {
    }

    /**
     * 방에서 나갈 때 호출
     *
     * @param user       요청을 한 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 나가기 성공이고 false이면 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean onLeaveRoom(SampleGameUser user,
                               final Payload inPayload,
                               Payload outPayload) throws SuspendExecution {
    }

    /**
     * onLeaveRoom 콜백이 성공한 경우, 나가기가 완료된 다음에 후처리를 위해 호출
     *
     * @param user 방에서 나간 유저 객체
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onPostLeaveRoom(SampleGameUser user) throws SuspendExecution {
    }

    /**
     * 유저가 방에 참여한 상태에서, 접속 끊김 등으로 인해 재접속 및 재로그인을 할 경우 해당 방으로 자동으로 재진입 하면서 호출
     *
     * @param user       방으로 들어가는 유저 객체
     * @param outPayload 클라이언트로 전달할 페이로드
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onRejoinRoom(SampleGameUser user, Payload outPayload) throws SuspendExecution {    
    }

   /**
     * 처리할 타이머 핸들러를 등록하기 위한 콜백
     * 이 콜백이 호출되었을 때, 사용자는 처리하고 싶은 타이머 핸들러들을 원하는만큼 등록하도록 합니다.
     * (좀 더 자세한 내용은 "타이머" 챕터를 참고하도록 하세요)
     */
    @Override
    public void onRegisterTimerHandler() {
    }


    /**
     * 방이 다른 노드로 이동(전송) 가능한 상태인지 확인하기 위해 호출
     *
     * @return 반환값이 true이면 전송 가능한 상태이고 false이면 불가능한 상태이다. 불가능한 상태의 경우, 만일 NonStopPatch가 진행 중이라면 패치가 종료되기 전까진 해당 방을 전송하기 위해 지속적으로 호출
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public boolean canTransfer() throws SuspendExecution {    
    }
  
    /**
	 * 해당 방이 다른 노드로 이동(전송)할 때, 소스 노드에서 전달할 데이터를 꾸리기 위해서 호출
	 * (좀 더 자세한 내용은 "전송 가능 객체" 챕터를 참고하세요.)
     *
     * @param transferPack 다른 노드로 가지고 갈 데이터를 저장하기 위한 꾸러미
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * 해당 방이 다른 노드로 이동(전송)할 때, 대상 노드에서 새롭게 생성된 방 객체를 원래의 상태로 복원하기 위해 호출
     * (좀 더 자세한 내용은 "전송 가능 객체" 챕터를 참고하세요.)
     * 
     * 이 시점에는 방은 완전히 복구되지 않았으므로, 다른 곳(노드, 유저 등)으로의 메시지 요청이 제한된다.
     *
     * @param transferPack 다른 노드에서 가지고 온 데이터 꾸러미
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onTransferIn(List<SampleGameUser> userList,
                             final TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * 방이 속한 노드가 pause될 때, 해당 방도 pause 되면서 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onPause() throws SuspendExecution {
    }

    /**
     * 방이 속한 노드가 resume될 때, 해당 방도 resume 되면서 호출
     *
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * 클라이언트에서 파티 매치메이킹을 요청했을 경우 호출되는 콜백
     * (파티 매치메이킹을 위해서는 2명 이상의 유저가 NamedRoom에 들어가 있어야 한다. 이 때, 해당 NamedRoom에서 이 콜백이 호출된다.)
     *
     * @param roomType   클라이언트와 서버 사이에 사전 정의한 방 종류를 구분하는 임의의 값
     * @param user       파티 매칭을 요청한 유저(방장)
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 파티 매치메이킹 요청 성공이고 false 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */  
    @Override
    public boolean onMatchParty(final String roomType,
                                final SampleGameUser user,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
        return false;
    }
}
```

방은 가장 기본적인 3가지 콜백인 onCreateRoom, onJoinRoom, onLeaveRoom을 제공합니다. 각각 방의 생성, 참여 그리고 떠나기에 대해 호출됩니다. 사용자는 해당 콜백에서 관련한 기능을 직접 구현할 수 있습니다. 예를 들어, 방을 생성하면서 방장을 지정하고 기본적인 자료 구조 들을 초기화할 수 있습니다. 또한 임의의 방 참여 요청에 대해서는 방의 유저 목록을 갱신하거나 각 유저 사이에 상태 동기화 등을 수행할 수 있습니다.

이러한 방의 생성과 참여는 클라이언트의 명시적인 요청 뿐만 아니라 매치메이킹에 의해 엔진이 자동으로 처리하기도 합니다. 예를 들어 2명 정원인 게임에 대해 A와 B 유저가 유저 매치 되었다면 두 명 중 한 명은 자동으로 방을 생성하고, 나머지 한 명이 해당 방으로 참여하게 됩니다. 이러한 과정은 A와 B 유저의 요청이 아닌 엔진에서 자동으로 처리하는 것입니다. 이 과정에서도 당연히 onCreateRoom과 onJoinRoom 콜백이 호출됩니다.

이러한 방의 콜백을 정리하면 아래의 표와 같습니다.


| 콜백 이름              | 의미                             | 설명                                                                                                                                                                                                                                                                                                                                                         |
| ------------------------ | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onInit                 | 초기화                           | 방이 생성될 때 초기화를 위해 호출됩니다. 토픽 등록 등의 해당 방에 대한 초기화 코드를 작성할 수 있습니다.                                                                                                                                                                                                                                                     |
| onDestroy              | 방 사라짐                        | 방에서 마지막 유저가 나가고 더 이상 처리할 메시지가 없으면 해당 방은 사라집니다. 이 때 호출되는 콜백입니다.                                                                                                                                                                                                                                                  |
| onDispatch             | 처리할 패킷이 있음               | 유저에 처리할 패킷이 도달하면 호출됩니다. 사용자는 자신이 선언한 디스패처를 사용하거나 패킷에 대한 추가 작업을 진행할 수 있습니다. 자세한 내용은[메시지 처리](server-07-message-handling#13-ondispatch)를 참고하세요.                                                                                                                                      |
| onCreateRoom           | 방 생성                          | 클라이언트가 방 생성을 요청하면 호출됩니다. 컨텐츠에서 사용할 유저 목록을 위한 자료 구조를 생성하거나 기타 방 생성과 함께 처리되어야할 코드를 작성합니다.                                                                                                                                                                                                    |
| onJoinRoom             | 방 참여                          | 클라이언트가 서버 상에 존재하는 임의의 방으로 참여를 요청하면 호출됩니다. 컨텐츠에서 사용할 유저 목록을 갱신하거나 방 안의 유저들 사이에 동기화할 정보를 처리할 수 있습니다.                                                                                                                                                                                 |
| onLeaveRoom            | 방 떠남                          | 클라이언트가 방 떠나기 요청을 하면 호출됩니다. 컨텐츠에서 사용할 유저 목록을 위한 자료 구조를 갱신하거나 기타 방에서 나오면서 함께 처리되어야할 코드를 작성합니다.                                                                                                                                                                                           |
| onPostLeaveRoom        | 방 떠남 후처리                   | onLeaveRoom이 성공하면 후처리를 위해 호출됩니다. 방을 나온 직후에 처리할 코드를 작성합니다.                                                                                                                                                                                                                                                                  |
| onRejoinRoom           | 방 다시 참여                     | 유저가 방에 들어가 있는 상태에서 접속 끊김 등으로 인해 재로그인(ReLogin)을 하는 경우 엔진에 의해 자동으로 해당 방에 재진입(ReJoin) 하게 됩니다. 이 때, 호출되는 콜백입니다. 재진입 과정에서 동기화에 필요한 정보 등을 처리할 수 있습니다.                                                                                                                    |
| onRegisterTimerHandler | 타이머 등록                      | 엔진 구동 과정에서 이 콜백이 호출되었을 때, 사용자는 처리하고 싶은 타이머 핸들러들을 원하는만큼 등록할 수 있습니다. 이렇게 등록된 타이머는 엔진에 의해 항시 관리되며 특히 유저 전송 시에도 자동으로 함께 전송됩니다.                                                                                                                                         |
| canTransfer            | 방 전송이 가능한 상태인지 확인   | 해당 방이 다른 노드로 전송될 수 있는 상태인지 체크하기 위해 호출됩니다.  만일 방에서 아직 게임이 플레이 중이거나 준비가 안 된 경우에는 false를 반환하여 전송을 미룰 수 있습니다. false를 반환한 경우에는 엔진이 임의의 시간 이후에 지속적으로 이 콜백을 호출합니다. 참고로 방 전송은 오직 무점검 패치를 진행할 때에만 사용됩니다. |
| onTransferOut          | 기존 노드에서 전송되어 나갈 준비 | 방이 다른 GameNode로 전송될 때, 소스 노드에서 전송을 시작할 때 호출됩니다. 사용자는 이 콜백에서 방 객체와 함께 전송할 데이터 꾸러미를 꾸릴 수 있습니다. 참고로 방 전송은 방 안의 유저들 각각에 대한 유저 전송을 포함합니다.                                                                                                                                  |
| onTransferIn           | 새로운 노드로 전송 완료 처리     | 방이 다른 GameNode로 전송될 때, 대상 노드에서 전송 완료하면서 호출됩니다. 사용자는 함께 가지고 온 데이터 꾸러미를 풀어서 원래의 방 상태로 복구할 수 있습니다. 참고로 방 전송은 방 안의 유저들 각각에 대한 유저 전송을 포함합니다.                                                                                                                            |
| onPause                | 일시 정지                        | 콘솔을 통해 GameNode를 일시 정지하면 해당 GameNode의 모든 방에 대해 호출됩니다. 사용자는 노드가 일시 정지될 때 방에서 추가로 처리하고 싶은 코드를 이 곳에 구현할 수 있습니다.                                                                                                                                                                                |
| onResume               | 재개                             | 콘솔을 통해 GameNode가 일시 정지 상태에서 다시 구동을 재개하면, 해당 GameNode의 모든 방에 대해 호출됩니다. 사용자는 재개 상태에서 방에 대해 처리하고 싶은 코드를 이 곳에 구현할 수 있습니다.                                                                                                                                                                 |
| onMatchParty           | 파티 매치메이킹 요청             | 사용자가 파티 매치메이킹을 요청하면 호출됩니다. 파티 매치메이킹은 임의의 NamedRoom을 파티 용도로 생성한 후, 방 안의 모든 유저가 하나의 파티로 매칭을 요청하는 기능입니다. 사용자는 이 콜백에서 엔진이 제공하는 파티 매치메이킹 API 혹은 제3의 매치메이킹 솔루션을 임의로 사용할 수 있습니다.                                                                 |

## 5. 방 종류

앞서 살펴본 방의 구현법과 별개로 엔진에서 제공하는 방의 종류는 크게 두 가지입니다. 이 두 가지의 방을 총 네 가지의 방법으로 사용합니다.

| 방 종류     | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| Normal Room | 클라이언트가 명시적으로 CreateRoom을 요청해서 생성합니다. 다른 클라이언트듣 해당 방의 아이디를 이용하여 명시적으로 JoinRoom을 하여 참여할 수 있습니다. 그러므로 참여할 유저는 미리 해당 방의 아이디를 공유 받아야 합니다. |
| Named Room  | NamedRoom은 그 이름처럼 유일한 방 이름을 중심으로 명명된 방(NamedRoom)에 대해 동작을 수행합니다. 클라이언트는 서버군 내에서 유일한 방 이름으로 NamedRoom을 요청합니다. 이 때, 만일 서버에 해당 방 이름이 존재하지 않으면 요청자는 직접 방을 생성하게 됩니다. 반대로 만일 서버에 해당 방 이름이 이미 존재할 경우에는 그 방으로 자동 진입하게 됩니다. 예를 들어, 스타크래프트의 커스텀 게임 목록에 나타나는 "3:3 헌터 초보!"와 같은 온갖 방제목을 생각하면 이해하기 쉽습니다. |

이러한 두 가지 방 종류는 총 네 가지의 방법으로 사용됩니다.

| 방 종류     | 사용법                                                       |
| ----------- | ------------------------------------------------------------ |
| Normal Room | 1. 클라이언트는 CreateRoom / JoinRoom 요청을 통해 생성 및 참여합니다.<br>2. 룸 매치메이킹을 통해 NormalRoom을 생성하거나 참여할 수 있습니다. 또한 CreateRoom으로 만든 방도 룸 매치메이킹 대상으로 등록이 가능합니다. 이 때, 방 아이디는 엔진에 의해 관리되고 매칭되는 방과 유저 사이에서 자동으로 공유됩니다. |
| Named Room  | 3. 클라이언트는 NamedRoom 요청을 통해 생성 및 참여합니다.<br>4. 유저 매치메이킹을 통해 NamedRoom을 생성하거나 참여할 수 있습니다. 이 때, 생성되는 NamedRoom의 방 이름은 엔진에서 고유하게 생성하고 관리합니다. 룸 매치메이킹과 달리 일반적인 NamedRoom으로 생성한 방은 유저 매치메이킹 대상이 될 수 없습니다. 단, 파티 매치메이킹을 위해 NamedRoom으로 파티 룸을 생성한 후 여러명의 유저들이 하나틔 파티로 매치메이킹을 요청할 수 있습니다. |



## 6. 채널

GameNode들은 그 용도에 맞춰 논리적으로 그룹화할 수 있습니다. 이러한 논리 그룹을 [채널](server-09-channel)이라고 합니다. 예를 들어, GameNode 1과 2를 "Beginner" 채널로 묶고, GameNode 3과 4를 "Expert" 채널로 묶을 수 있습니다. 이에 대한 자세한 설명은 [별도의 챕터](server-09-channel)에서 더 자세하게 다루도록 합니다.
