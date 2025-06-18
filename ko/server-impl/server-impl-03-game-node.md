## Game > GameAnvil > 서버 개발 가이드 > 게임 노드 구현

## Game Node

![GameNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gamenode_on_network.png)

GameNode는 실제 게임 객체가 생성되고 게임 콘텐츠를 구현하는 노드입니다. 클라이언트는 GatewayNode에서 인증을 완료한 후, GameNode에 로그인을 완료해야 비로소 이러한 게임 콘텐츠를 시작할 수 있습니다.

아래의 이미지에서 보듯이, 클라이언트는 고유한 AccountId로 인증을 완료한 커넥션을 기반으로 여러 개의 논리 세션을 생성할 수 있습니다. 이때, 세션은 클라이언트와 유저 객체 사이에 만들어집니다. 아래의 그림에서 빨간색 점선 화살표가 이러한 세션을 나타냅니다.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

각각의 세션은 해당 커넥션 내에서 고유한 값으로 구분할 수 있으며 우리는 이 값을 SubId라고 부릅니다. 즉, AccountId와 SubId의 조합으로 사용자는 원하는 만큼 세션을 생성할 수 있습니다. 이때, 동일한 서비스에 대한 세션도 마찬가지로 원하는 만큼 생성할 수 있습니다. 즉, 이미지에서 AccountId가 1인 커넥션은  "Game" 서비스 혹은 "Chat" 서비스에 대한 세션을 얼마든지 추가로 생성할 수 있는 것입니다.

이러한 세션이 향하는 곳은 바로 유저 객체입니다. GameNode는 이러한 유저 객체와 그들의 그룹인 방 객체를 관리합니다. 이번 챕터는 이러한 GameNode와 GameUser 그리고 GameRoom에 대해 설명합니다.

## GameNode 구현

GameNode는 IGameNode 인터페이스를 구현합니다. 아래의 예제 코드는 GameNode에서 기본적으로 재정의할 수 있는 콜백 메서드를 보여줍니다. 노드 공통 콜백과 더불어 채널 관리를 위한 콜백이 존재합니다.

```java
@GameAnvilGameNode(gameServiceName = "MyGame") // (1) "MyGame"이라는 서비스를 위한 GameNode로 엔진에 등록
public class SampleGameNode implements IGameNode {
    private IGameNodeContext gameNodeContext;

    /**
     * 게임 노드 컨텍스트를 전달하기 위해 호출
     * <p/>
     * 객체가 생성된후 한번 호출된다
     *
     * @param gameNodeContext 게임 노드 컨텍스트
     */
    @Override
    public void onCreate(IGameNodeContext gameNodeContext) {
        this.gameNodeContext = gameNodeContext;
    }

    /**
     * 같은 채널의 다른 노드에 유저 변화가 있을때 호출
     * <p>
     * updateChannelUser() 호출시 발생.
     *
     * @param type            채널 정보 변경 타입(갱신/삭제) 인 {@link ChannelUpdateType}
     * @param channelUserInfo 변경될 유저 정보인 {@link IChannelUserInfo}
     * @param userId          변경 대상의 유저 아이디
     * @param accountId       변경 대상의 어카운트 아이디
     */
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {

    }

    /**
     * 같은 채널의 다른 노드에 룸 상태 변화가 있을때 호출
     * <p>
     * updateChannelRoomInfo() 호출시 발생
     *
     * @param type            채널 정보 변경 타입(갱신/삭제) {@link ChannelUpdateType}
     * @param channelRoomInfo 변경될 룸 정보인 {@link IChannelRoomInfo}
     * @param roomId          변경 대상의 룸 아이디
     */
    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {

    }

    /**
     * 클라이언트에서 채널 정보를 요청시 호출   (Base.GetChannelInfoReq)
     *
     * @param outPayload 클라이언트로 전달될 채널 정보
     */
    @Override
    public void onChannelInfo(IPayload payload) {

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

```java
// 프로토 버퍼 MyGame.GameNodeTest 입력이 들어왔을때 동작하는 메세지 처리 클래스
@GameAnvilController
public class _GameNodeTest {
    // (2) SampleGameNode에서 처리하고 싶은 프로토콜과 핸들러를 매핑
    @GameNodeMapping(
        value = MyGame.GameNodeTest.class, // 처리할 프로토 버퍼
        loadClass = SampleGameNode.class   // 메세지를 받는 대상 (SampleGameNode)
    )
    public void execute(IGameNodeDispatchContext ctx) {
        // 여기서 할 작업을 작성
    }
}
```

콜백의 의미와 용도는 아래의 표를 참고하십시오.

| 콜백 이름                   | 의미           | 설명                                                                                                                                                              |
|-------------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate                | 객체 생성        | 객체가 생성되었을 때 호출됩니다. 생성된 타입에서 사용 가능한 API를 사용할 수 있는 컨텍스트를 전달받습니다. 컨텐츠에서 필요하다면 저장해서 사용할 수 있습니다.                                                                     |
| onChannelUserInfoUpdate | 채널의 유저 정보 갱신 | 같은 채널로 묶인 여러 개의 GameNode 중, 하나의 GameNode에서 채널의 유저 정보가 변경되었을 때 같은 채널 내의 나머지 모든 GameNode에서 동기화를 위하여 호출됩니다. 이때, 사용자는 전달받은 정보를 바탕으로 현재 GameNode의 채널 정보를 갱신할 수 있습니다. |
| onChannelRoomInfoUpdate | 채널의 방 정보 갱신  | 같은 채널로 묶인 여러 개의 GameNode 중, 하나의 GameNode에서 채널의 방 정보가 변경되었을 때 같은 채널 내의 나머지 모든 GameNode에서 동기화를 위하여 호출됩니다. 이때, 사용자는 전달받은 정보를 바탕으로 현재 GameNode의 채널 정보를 갱신할 수 있습니다.  |
| onChannelInfo           | 채널 정보 요청     | 클라이언트가 채널 정보를 요청할 때 호출됩니다. 사용자는 이 콜백에서 원하는 대로 채널 정보를 구성하여 클라이언트로 전달할 수 있습니다.                                                                                    |
| onInit                  | 초기화          | 노드가 최초 초기화를 진행할 때 호출합니다. 노드 구동 전에 필요한 초기화 작업이 있다면 이 콜백이 적합합니다. 이때, 노드는 아직 메시지를 처리하지 않습니다.                                                                       |
| onPrepare               | 준비           | 노드 초기화가 완료된 후 호출됩니다. 사용자는 노드가 준비 완료되기 전에 임의의 작업을 이곳에서 처리할 수 있습니다. 이때, 노드는 메시지를 처리할 수 있습니다.                                                                      |
| onReady                 | 준비 완료        | 노드가 모든 준비를 마친 후, 구동 완료 단계입니다. 이때, 노드는 Ready 상태이므로 사용자는 이때부터 모든 기능을 사용할 수 있습니다.                                                                                  |
| onPause                 | 일시 정지        | 노드를 일시 정지하면 호출됩니다. 사용자는 노드가 일시 정지될 때 추가로 처리하고 싶은 코드를 이곳에 구현할 수 있습니다.                                                                                            |
| onShuttingdown          | 노드 정지        | 노드가 Shutdown 명령을 받을 때 호출됩니다. 중지된 노드는 재개(Resume) 할 수 없습니다.                                                                                                       |
| onResume                | 재개           | 노드가 일시 정지 상태에서 다시 구동을 재개할 때 호출됩니다. 사용자는 재개 상태에서 처리하고 싶은 코드를 이곳에 구현할 수 있습니다.                                                                                     |

모든 노드는 사용자 정의 메시지를 처리하기 위한 메시지 핸들러 등록 과정이 필요합니다. 특히, GameNode는 게임 콘텐츠를 위해 이러한 과정이 필수입니다. 서버 실행전 메인 클래스에서 설정을 진행합니다. (1)GameAnvilConfig에 설정되어 있는 게임 서비스 이름을 생성합니다. 게임 서비스 이름은 반드시 GameAnvilConfig에 정의되어 있는 이름을 사용해야 합니다. (2) 그리고 처리하고 싶은 메시지를 구현해둔 [핸들러](server-impl-07-message-handling.md#_2)와 연결합니다. 하나의 GameNode 클래스는 오직 하나의 서비스에 대해서만 등록할 수 있습니다.

이러한 GameNode의 주 목적은 노드에 접속된 모든 GameUser와 GameRoom 객체들에 대한 처리를 수행하는 것입니다. 이에 대해서는 바로 달아서 설명하도록 하겠습니다.

## 유저 구현

유저 객체는 로그인 과정을 거쳐 GameNode에 생성됩니다. 유저 기반의 콘텐츠는 이 클래스를 중심으로 구현해야합니다. 앞서 살펴본 모든 예제와 마찬가지로 유저 또한 처리할 고유의 메시지와 핸들러를 연결할 수 있습니다. 아래의 예제 코드를 보면 유저는 꽤 많은 콜백 메서드를 제공하는 것을 볼 수 있습니다. 이 중 일부는 기본 구현이 제공되므로 특별히 필요한 상황이 아니라면 재정의하지 않아도 됩니다. 이는 유저뿐만 아니라 엔진에서 제공하는 대부분의 클래스에 해당합니다.

```java
@GameAnvilUser(
    gameServiceName = "MyGame", // 유저가 소속될 노드 (위 SampleGameNode 와 같은 서비스 이름)
    gameType = "BasicUser",     // 유저의 고유 타입, "BasicUser"라는 유저 타입의 유저를 엔진에 등록
    useChannelInfo = true       // 채널간의 정보 동기화 설정
)
public class SampleGameUser implements IUser {
    private IUserContext userContext;

    /**
     * 유저 컨텍스트를 전달하기 위해 호출
     * <p/>
     * 객체가 생성된후 한번 호출된다
     *
     * @param userContext 유저 컨텍스트
     */
    @Override
    public void onCreate(IUserContext userContext) {
        this.userContext = userContext;
    }

    /**
     * 로그인할 때 호출
     *
     * @param payload        클라이언트에서 전달받은 {@link IPayload}
     * @param sessionPayload onBeforeLogin 에서 전달된 {@link IPayload}
     * @param outPayload     클라이언트로 전달할 {@link IPayload}
     * @return 반환값이 true 이면 로그인 성공, false 이면 로그인 실패
     */
    @Override
    public boolean onLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 로그인 성공 이후에 필요한 후처리를 위해 호출
     * <p/>
     * (즉, onLogin 혹은 onReLoin이 성공한 후 호출)
     *
     * @param isRelogined 재로그인 여부
     */
    @Override
    public void onAfterLogin(boolean isRelogined) {

    }

    /**
     * 이미 로그인 된 상태에서, 다시 로그인을 시도할 때 호출
     * <p/>
     * 로그인 된 상태에서는 접속이 끊어지더라도 사용자의 게임유저 객체가 게임노드에 일정 기간 유효한 상태로 남아있다
     *
     * @param payload        클라이언트에서 전달한 임의의 {@link IPayload}
     * @param sessionPayload onBeforeLogin 에서 전달된 {@link IPayload}
     * @param outPayload     클라이언트로 전달할 임의의 {@link IPayload}
     * @return 반환값이 true 이면 ReLogin 성공, false 이면 ReLogin 실패
     */
    @Override
    public boolean onReLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 클라이언트와의 연결이 끊어졌을때 호출되는 콜백
     */
    @Override
    public void onDisconnect() {

    }

    /**
     * 유저가 속한 노드가 Pause 될 때, 해당 유저도 Pause 되면서 호출
     */
    @Override
    public void onPause() {

    }

    /**
     * 유저가 속한 노드가 Resume 될 때, 해당 유저도 Resume 되면서 호출
     */
    @Override
    public void onResume() {

    }

    /**
     * 유저가 로그아웃할 때 호출
     *
     * @param payload    클라이언트로부터 전달받은 {@link IPayload}
     * @param outPayload 클라이언트로 전달할 {@link IPayload}
     */
    @Override
    public void onLogout(IPayload payload, IPayload outPayload) {

    }

    /**
     * 해당 유저가 로그아웃 가능한지 확인하기 위해 호출
     * <p/>
     * 엔진 사용자는 이 콜백에서 현재 게임 유저가 로그아웃을 해도 문제가 없을지 결정할 수 있다
     *
     * @return 반환값이 false 이면 로그아웃 진행이 멈추고, 이 후에 주기적으로 다시 콜백을 호출. 반환값이 true 이면 로그 아웃을 진행
     */
    @Override
    public boolean canLogout() {
        return true;
    }

    /**
     * 룸의 onLeavingRoom 실행되고 룸에서 유저가 완전히 나간 후 호출
     * <p/>
     * 룸에서 나간 유저가 처리해야 하는 작업을 진행한다
     */
    @Override
    public void onAfterLeaveRoom() {

    }

    /**
     * 유저가 다른 노드로 이동(전송) 가능한 상태인지 확인하기 위해 호출
     *
     * @return 반환값이 true 이면 전송 가능한 상태, false 이면 불가능한 상태. 불가능한 상태의 경우, 만일 SafePause가 진행 중이라면 SafePause가 종료되기 전까진 해당 유저를 전송하기 위해 지속적으로 호출
     */
    @Override
    public boolean canTransfer() {
        return true;
    }

    /**
     * 이미 로그인된 상황에서 다른 디바이스로 같은 유저가 로그인할 때 호출
     *
     * @param newDeviceId           새로 접속한 유저의 디바이스 아이디 값
     * @param outPayloadForKickUser 클라이언트로 전달할 {@link IPayload}. kickOut 혹은 LoginRes 정보 포함
     * @return 반환값이 true 이면 새로 접속한 유저가 로그인 된 후 기존 유저는 강제 로그아웃 처리. false 이면 새로 접속한 유저가 로그인 실패
     */
    @Override
    public boolean onLoginByOtherDevice(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 임의의 유저 타입으로 이미 로그인 한 상태에서 다른 유저 타입으로 로그인을 시도할 때 호출
     *
     * @param userType   새로 로그인을 시도하는 유저의 타입
     * @param outPayload 클라이언트로 전달할 {@link IPayload}
     * @return 반환값이 true 이면 새로운 로그인이 성공, false 이면 실패
     */
    @Override
    public boolean onLoginByOtherUserType(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 이미 로그인 된 상태에서 (재접속 등의 이유로) 다른 커넥션을 통해 로그인을 시도할 경우 호출
     *
     * @param outPayload 클라이언트로 전달할 {@link IPayload}
     */
    @Override
    public void onLoginByOtherConnection(IPayload payload) {

    }

    /**
     * 룸 매치메이킹 요청을 받으면 호출
     *
     * @param roomType             클라이언트와 서버 사이에 사전 정의한 룸 종류를 구분하는 임의의 값
     * @param matchingGroup        매칭되는 룸 매칭 그룹 전달
     * @param matchingUserCategory 매칭되는 매칭 유저 카테고리 전달
     * @param payload              클라이언트로부터 전달받은 {@link IPayload}
     * @return {@link RoomMatchResult} 타입으로 매칭된 룸의 정보 반환. null 을 반환 할 경우 클라이언트 요청 옵션에 따라서 새로운 룸이 생성되거나 요청 실패 처리
     */
    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        return RoomMatchResult.FAILED;
    }

    /**
     * 클라이언트 의 룸 매치메이킹 요청을 처리할 때 실패하는 경우 호출
     *
     * @param matchRoomFailCode 룸 매치메이킹이 실패한 이유
     */
    @Override
    public void onMatchRoomFail(MatchRoomFailCode matchRoomFailCode) {

    }

    /**
     * 클라이언트의 유저 매치메이킹  요청을 처리할 때 실패하는 경우 호출
     *
     * @param matchUserFailCode 유저 매치메이킹이 실패한 이유
     */
    @Override
    public void onMatchUserFail(MatchUserFailCode matchUserFailCode) {

    }

    /**
     * 클라이언트에서 유저 매치메이킹을 요청했을 경우 호출되는 콜백
     *
     * @param roomType      클라이언트와 서버 사이에 사전 정의한 룸 종류를 구분하는 임의의 값
     * @param matchingGroup 매칭 그룹
     * @param payload       클라이언트로부터 전달받은 {@link IPayload}
     * @param outPayload    클라이언트로 전달할 {@link IPayload}
     * @return 반환값이 true 이면 유저 매치메이킹 요청 성공, false 이면 실패
     */
    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 유저 매치메이킹이 취소될 때 호출
     *
     * @param reason 취소된 이유. 타임아웃(TIMEOUT), 사용자의 요청에 의한 취소(CANCEL), 매치 노드의 종료에 의한 취소(SHUTDOWN)
     */
    @Override
    public void onMatchUserCancel(MatchCancelReason matchCancelReason) {

    }

    /**
     * 유저가 다른 노드로 이동(전송)할 때, 출발 노드에서 전달할 데이터를 꾸리기 위해서 호출
     *
     * @param transferPack 다른 노드로 가지고 갈 데이터를 저장하기 위한 꾸러미
     */
    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    /**
     * 유저가 다른 노드로부터 이동(전송)되어 왔을 때 도착지 노드로 데이터, 다시 등록해야할 타이머 키를 전달하기 위해 호출
     * <p>
     * TimerHandlerTransferPack을 통해서 user에 등록되어 있던 timerHandlerKey 목록을 확인한다
     * <p/>
     * TimerHandlerTransferPack의 reRegister()를 활용해서 사용할 timerHandler를 다시 등록한다
     *
     * @param transferPack             다른 노드에서 가지고 온 데이터를 전달하기 위한 꾸러미
     * @param timerHandlerTransferPack
     */
    @Override
    public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    /**
     * 유저가 다른 노드로부터 이동(전송)되어 왔을 때 전송이 완료된 후 호출
     */
    @Override
    public void onAfterTransferIn() {

    }

    /**
     * 클라이언트에서 스냅샷 요청시 호출
     * <p>
     * 주로 접속이 끊어 지고 서버 상태가 변할 확률이 있을 경우 호출해서 클라이언트와 서버 상태정보를 동기화 하는데 사용된다
     *
     * @param payload    클라이언트에서 서버로 전달되는 디파인이 세팅된 {@link IPayload} 전달
     * @param outPayload 서버에서 클라이언트로 전달되는 디파인이 세팅된 {@link IPayload} 전달
     */
    @Override
    public void onSnapshot(IPayload payload, IPayload outPayload) {

    }

    /**
     * 클라이언트에서 다른 채널로 이동 요청을 할 때, 현재 유저가 채널 이동이 가능한 상태인지 확인하기 위해 호출
     * <p>
     * 주의! 만일, 사용자가 명시적으로 moveChannel() API를 호출하여 채널을 이동할 경우에는 canMoveOutChannel()가 호출되지 않는다
     * <p/>
     * 오직 엔진에 의해 암묵적인 채널 이동이 발생할 때 호출된다
     *
     * @param destinationChannelId 이동 대상 채널의 아이디
     * @param payload              클라이언트로부터 전달받은 {@link IPayload}
     * @param errorPayload         채널 이동을 실패할 경우, 서버에서 클라이언트로 전달하는 에러 {@link IPayload}. 성공일 경우에는 전달 안된다
     * @return 반환값이 false 이면 채널 이동이 불가능하므로 요청은 실패이고 true 이면 성공
     */
    @Override
    public boolean canMoveOutChannel(String channelId, IPayload payload, IPayload outPayload) {
        return false;
    }

    /**
     * 다른 노드로 채널 이동을 할 때, 출발 노드에서 호출
     *
     * @param destinationChannelId 이동 대상 채널의 아이디
     * @param outPayload           이동할 채널에 전달할 {@link IPayload}
     */
    @Override
    public void onMoveOutChannel(String channelId, IPayload payload) {

    }

    /**
     * 다른 노드로 채널 이동이 완료된 후 출발 노드에서 호출
     */
    @Override
    public void onAfterMoveOutChannel() {

    }

    /**
     * 다른 노드로 채널 이동을 할 때, 대상 노드로 진입하면서 호출
     *
     * @param sourceChannelId 이동하기 전의 채널 아이디
     * @param payload         클라이언트로부터 전달받은 {@link IPayload}
     * @param outPayload      클라이언트로 전달할 {@link IPayload}
     * @throws GameAnvilException IOException, ExecutionException, InterruptedException 발생시 GameAnvilException 으로 묶어서 throw
     */
    @Override
    public void onMoveInChannel(String channelId, IPayload payload, IPayload outPayload) {

    }
}

```

```java
@GameAnvilController
public class _GameUserTest {
   // SampleGameUser에서 처리하고 싶은 프로토콜과 핸들러를 매핑
    @GameUserMapping(
        value = MyGame.GameUserTest.class, // 처리할 프로토 버퍼
        loadClass = SampleGameUser.class   // 메세지를 받는 대상 (SampleGameUser)
    )
    public void execute(IUserDispatchContext ctx) {
       // 여기서 할 작업을 작성
    }
}
```

특히, 유저는 엔진에 등록하기 위한 설정이 다른 클래스에 비해 많이 요구합니다. 우선, 어떤 게임 서비스를 위한 유저인지 등록 후, 유저 타입을 등록합니다. 이 유저 타입은 이름 그대로 유저의 종류를 구별하기
위한 용도로서, 클라이언트에서도 마찬가지로 API를 호출할 때 사용됩니다. 즉, 해당 API가 서버의 어떤 유저 타입에 대한 호출인지 명시하는 것입니다. 그러므로 반드시 서버와 클라이언트 사이에 이러한 유저 타입을
임의의 문자열로 사전 정의해두어야 합니다. 이 예제 코드에서는 "BasicUser"라는 유저 타입을 사용합니다. 이에 대한 더 자세한 설명은 별도의 챕터에서 다시 다루도록 합니다. 마지막으로 이 유저
정보가 [채널 간 유저 정보 동기화](server-impl-09-channel.md#_4)에 사용될지 여부를 결정할 수 있습니다. 만일 채널 간 정보 동기화가 필요 없다면 생략할 수 있습니다.

위의 예제 코드에서 onLogin으로 시작하는 콜백은 모두 로그인과 관련되어 호출됩니다. 예를 들어 최초의 로그인 요청에 대해서는 onLogin 콜백이 호출되며, 이미 로그인이 되어 있는 상태에서 재로그인을 처리하는 경우는 onReLogin 콜백이 호출됩니다. 마찬가지로 로그아웃을 처리할 때에는 onLogout 콜백이 호출됩니다. 이처럼 GameAnvil의 콜백은 그 이름과 JavaDoc 주석을 통해 그 용도가 대부분 명확하게 설명이 됩니다.

유저는 언제든 채널 사이에서 이동이 가능합니다. 이러한 채널에 관련된 내용은 뒤에서 [별도의 챕터](server-impl-09-channel)로 따로 설명할 것이므로 여기에서는 우선 넘어가도록 하겠습니다. 뿐만 아니라 유저는 여러 개의 GameNode 사이에서 전송 가능한 객체입니다. 이와 관련한 기능 또한 [별도의 챕터](server-impl-08-object-transfer)에서 더 자세하게 설명하도록 합니다. 

GameAnvil은 두 종류의 매치메이킹 기능, 룸 매치메이킹과 유저 매치메이킹을 제공합니다. 이러한 매치메이킹 요청이 유저로 도달하면 onMatchRoom 혹은 onMatchUser 콜백이 호출됩니다. 사용자는 이 콜백에서 GameAnvil이 제공하는 매치메이커를 사용할 수도 있고 직접 구현하거나 3rd 파티로 제공받은 다른 매치메이커를 사용할 수도 있습니다. 이에 대한 더 자세한 설명은 바로 [다음 챕터](server-impl-04-match-node)에서 MatchNode를 다루면서 다시 하도록 하겠습니다.

이러한 유저의 콜백을 정리하면 아래의 표와 같습니다.

| 콜백 이름                    | 의미                      | 설명                                                                                                                                                      |
|--------------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate                 | 객체 생성                   | 객체가 생성되었을 때 호출됩니다. 생성된 타입에서 사용 가능한 API를 사용할 수 있는 컨텍스트를 전달받습니다. 컨텐츠에서 필요하다면 저장해서 사용할 수 있습니다.                                                             |
| onLogin                  | 로그인                     | 처음 로그인을 할 때 호출됩니다. 일반적으로 사용자는 이 콜백에서 DB 등의 저장소로부터 유저 정보를 가지고 와서 게임 유저 객체를 초기화하는 작업을 수행합니다.                                                              |
| onAfterLogin             | 로그인 성공 후처리              | onLogin이 성공한 후에 호출됩니다. 로그인에 대한 후처리 작업을 이 콜백에서 할 수 있습니다.                                                                                                 |
| onReLogin                | 재로그인                    | 이미 로그인이 되어 있는 상태에서 다시 로그인을 할 경우에는 onLogin이 아닌 onReLogin이 호출됩니다. 즉, 재로그인에 대한 작업은 이 콜백에서 처리합니다.                                                           |
| onDisconnect             | 접속 종료                   | 클라이언트로부터 접속이 끊겼을 때 호출됩니다. 이때, 추가로 처리할 코드를 이곳에 구현합니다.                                                                                                    |
| onPause                  | 일시 정지                   | 콘솔을 통해 GameNode를 일시 정지하면 해당 GameNode의 모든 유저에 대해 호출됩니다. 사용자는 노드가 일시 정지될 때 유저에서 추가로 처리하고 싶은 코드를 이곳에 구현할 수 있습니다.                                           |
| onResume                 | 재개                      | 콘솔을 통해 GameNode가 일시 정지 상태에서 다시 구동을 재개하면, 해당 GameNode의 모든 유저에 대해 호출됩니다. 사용자는 재개 상태에서 유저에 대해 처리하고 싶은 코드를 이곳에 구현할 수 있습니다.                                  |
| onLogout                 | 로그아웃                    | 유저가 로그아웃할 때 호출됩니다. 이는 사용자가 명시적으로 요청한 로그아웃일 수도 있고, 접속 끊김 상태에서 설정된 시간을 초과할 경우에 엔진에 의해 자동으로 로그아웃될 수도 있습니다.                                                 |
| canLogout                | 로그아웃 가능성 확인             | 해당 유저가 현재 로그아웃이 가능한 상태인지 체크하기 위해 호출됩니다. 만일 게임 플레이 중이거나 정보를 잃어서는 안 되는 상황에는 false를 반환하여 로그아웃을 미룰 수 있습니다. 만일 false를 반환하면 엔진은 임의의 시간 이후에 지속적으로 이 콜백을 호출합니다. |
| onAfterLeaveRoom         | 방 떠난 후처리                | 룸의 onLeavingRoom 실행되고 룸에서 유저가 완전히 나간 후 호출됩니다. 룸에서 나간 유저가 처리해야 하는 작업을 진행합니다.                                                                             |
| canTransfer              | 유저 전송이 가능한 상태인지 확인      | 해당 유저가 다른 노드로 전송될 수 있는 상태인지 체크하기 위해 호출됩니다.  만일 게임 플레이 중이거나 아직 준비가 안 된 경우에는 false를 반환하여 전송을 미룰 수 있습니다. false를 반환한 경우에는 엔진이 임의의 시간 이후에 지속적으로 이 콜백을 호출합니다. |
| onLoginByOtherDevice     | 다른 디바이스로 로그인 시도         | 이미 로그인이 되어 있는 상태에서 다른 기기로 추가 로그인 요청이 왔을 때 호출됩니다. 사용자는 기존의 유저와 새로운 유저 중 어느 쪽을 로그인시킬지 반환값으로 결정할 수 있습니다.                                                   |
| onLoginByOtherUserType   | 다른 유저 타입으로 로그인 시도       | 이미 로그인이 되어 있는 상태에서 다른 유저 타입으로 로그인 요청이 왔을 때 호출됩니다. 새로운 유저 타입에 대해 로그인을 진행할지 여부를 반환값으로 결정할 수 있습니다.                                                         |
| onLoginByOtherConnection | 다른 커넥션으로 로그인 시도         | 이미 로그인이 되어 있는 상태에서 (재접속으로 인해) 이전과 다른 커넥션으로 로그인 시도를 할 경우에 호출됩니다. 만일, 재접속에 대한 추가 작업이 필요하다면 이 콜백에서 할 수 있습니다.                                               |
| onMatchRoom              | 룸 매치메이킹 요청              | 사용자가 룸 매치메이킹을 요청하면 호출됩니다. 이때, 사용자는 이 콜백에서 엔진이 제공하는 룸 매치메이킹 API 혹은 제3의 매치메이킹 솔루션을 임의로 사용할 수 있습니다.                                                        |
| onMatchRoomFail          | 룸 매치메이킹 요청 실패           | 사용자가 룸 매치메이킹을 요청하면 호출됩니다.                                                                                                                               |
| onMatchUserFail          | 유저 매치메이킹 요청 실패          | 사용자가 유저 매치메이킹을 요청하면 호출됩니다. 사용자는 이 콜백에서 엔진이 제공하는 유저 매치메이킹 API 혹은 제3의 매치메이킹 솔루션을 임의로 사용할 수 있습니다.                                                          |
| onMatchUser              | 유저 매치메이킹 요청             | 사용자가 유저 매치메이킹을 요청하면 호출됩니다. 사용자는 이 콜백에서 엔진이 제공하는 유저 매치메이킹 API 혹은 제3의 매치메이킹 솔루션을 임의로 사용할 수 있습니다.                                                          |
| onMatchUserCancel        | 유저 매치메이킹 취소             | 사용자가 이전에 신청한 유저 매치메이킹을 취소하면 호출됩니다. 이미 매칭이 완료된 상황처럼 취소를 할 수 없는 경우에는 실패할 수도 있습니다.                                                                         |
| onTransferOut            | 기존 노드에서 전송되어 나갈 준비      | 유저가 다른 GameNode로 전송될 때, 소스 노드에서 전송을 시작할 때 호출됩니다. 사용자는 이 콜백에서 유저 객체와 함께 전송할 데이터 꾸러미를 꾸릴 수 있습니다.                                                          |
| onTransferIn             | 새로운 노드로 전송 완료 처리        | 유저가 다른 GameNode로 전송될 때, 대상 노드에서 전송 완료하면서 호출됩니다. 사용자는 함께 가지고 온 데이터 꾸러미를 풀어서 원래의 유저 상태로 복구할 수 있습니다.                                                       |
| onAfterTransferIn        | 전송 완료 후처리               | 유저 전송이 성공한 경우, 대상 노드에서 후처리를 위해 호출됩니다.                                                                                                                   |
| onSnapshot               | 클라이언트에서 스냅샷 요청          | 클라이언트에서 스냅샷 요청 시 호출됩니다. 주로 접속이 끊어지고 서버 상태가 변할 확률이 있을 경우 호출해서 클라이언트와 서버 상태 정보를 동기화하는데 사용됩니다.                                                             |
| canMoveOutChannel        | 채널 이동이 가능한 상태인지 확인      | 유저가 다른 채널로 이동할 수 있는 상태인지 체크하기 위해 호출됩니다. 만일, 사용자가 명시적으로 moveChannel() API를 호출하여 채널을 이동하는 경우에는 호출되지 않습니다. 오직 엔진에 의해 암묵적인 채널 이동이 발생할 때만 호출됩니다.             |
| onMoveOutChannel         | 기존 채널에서 다른 채널로 이동 준비    | 유저가 다른 채널로 이동할 때, 소스 노드에서 호출됩니다. 사용자는 원하는 정보를 outPayload에 담아서 대상 채널로 가지고 갈 수 있습니다.                                                                      |
| onAfterMoveOutChannel    | 기존 채널에서 다른 채널로 이동 준비 완료 | onMoveOutChannel이 성공하면 후처리를 위해 호출됩니다.                                                                                                                   |
| onMoveInChannel          | 새로운 채널로 이동 처리           | 유저가 다른 채널로 이동할 때, 대상 노드에서 호출됩니다. 사용자는 임의의 정보를 outPayload에 담아서 클라이언트로 전달할 수 있습니다.                                                                        |

### 로그인이란?

앞서 설명한 내용과 예제 코드에서 로그인에 관한 내용이 자주 등장합니다. 또한 이러한 로그인은 클라이언트가 서버에 접속한 후 GameNode에 자신의 유저 객체를 만드는 과정이라고 정의할 수 있습니다. 콜백 메서드 중 onLogin()은 최초에 유저를 생성하기 위해 로그인을 시도하는 과정에서 호출됩니다. 이때, 사용자는 유저 객체를 구성하기 위한 정보를 DB 등으로부터 획득할 수 있습니다. 이러한 onLogin() 콜백이 성공하면 GameNode 상에 해당 유저 객체가 생성됩니다. 이렇게 로그인이 완료되면, 직접 정의한 프로토콜을 기반으로 클라이언트는 자신의 유저 객체를 통해 다른 객체들과 메시지를 주고받으며 여러 가지 콘텐츠를 구현할 수 있습니다.

### 로그아웃

로그아웃은 로그인의 반대 개념입니다. 즉, GameNode 상에서 자신의 유저 객체를 제거하는 과정입니다. 로그아웃을 시작하면 해당 유저 객체는 onLogout() 콜백을 호출하여 메모리상에서 삭제되기 전에 DB 등으로 자신의 최종 상태를 보관할 수 있습니다. 이러한 로그아웃은 클라이언트가 명시적으로 요청할 수도 있고, 클라이언트의 접속이 끊긴 상태에서 임의의 시간이 경과한 후 엔진에 의해 자동으로 처리되기도 합니다. 그러므로 만일 모바일 게임과 같이 잦은 접속 끊김이 예상될 경우에는 바로 로그아웃이 진행되지 않도록 적절한 [설정](server-impl-16-config-vm.md#game)을 해둘 수 있습니다. 

## 방 구현

2명 이상의 유저는 방을 통해 동기화된 메시지 흐름을 만들 수 있습니다. 즉, 유저들의 요청은 방 안에서 모두 순서가 보장됩니다. 물론 1명의 유저를 위한 방 생성도 콘텐츠에 따라서 의미를 가질 수도 있습니다. 방을 어떻게 사용할지는 어디까지나 엔진 사용자의 몫입니다. 이러한 방은 유저와 마찬가지로 기본 클래스인 IRoom 인터페이스를 구현하여 여러 가지 콜백 메서드를 재정의할 수 있으며 자체적으로 메시지를 처리할 수도 있습니다. 아래의 예제 코드는 SampleUser를 위한 SampleRoom 클래스입니다.

 ```java
 @GameAnvilRoom(
    gameServiceName = "MyGame", // 방이 소속될 노드 (위 SampleGameNode 와 같은 서비스 이름)
    gameType = "BasicRoom",     // 방이 고유 타입, "BasicRoom"라는 타입의 방을 엔진에 등록
    useChannelInfo = true       // 채널간의 정보 동기화 설정
)
public class SampleRoom implements IRoom<SampleUser> {
    private IRoomContext roomContext;

    /**
     * 룸 컨텍스트를 전달하기 위해 호출
     * <p/>
     * 객체가 생성된후 한번 호출된다
     *
     * @param roomContext 룸 컨텍스트
     */
    @Override
    public void onCreate(IRoomContext<SampleUser> roomContext) {
        this.roomContext = roomContext;
    }

    /**
     * 룸이 초기화 될때 호출
     */
    @Override
    public void onInit() {

    }

    /**
     * 룸이 삭제 될때 호출
     */
    @Override
    public void onDestroy() {

    }

    /**
     * 새로운 룸을 생성할 때 호출
     * <p/>
     * 반환값에 의해 룸을 생성할지 여부가 결정
     *
     * @param user       요청한 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 {@link IPayload}
     * @param outPayload 클라이언트로 전달할 {@link IPayload}
     * @return 반환값이 true 이면 룸 생성이 성공, false 이면 실패
     */
    @Override
    public boolean onCreateRoom(SampleUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 임의의 룸에 참여할 때 호출
     * <p/>
     * 반환값에 의해 룸에 참여할지 여부가 결정
     *
     * @param user       요청한 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 {@link IPayload}
     * @param outPayload 클라이언트로 전달할 {@link IPayload}
     * @return 반환값이 true 이면 입장 성공, false 이면 실패
     */
    @Override
    public boolean onJoinRoom(SampleUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 룸에서 나갈 때 호출
     * <p/>
     * 반환값에 의해 룸에서 나갈지 여부가 결정
     *
     * @param user       요청을 한 유저 객체
     * @param inPayload  클라이언트로부터 전달받은 {@link IPayload}
     * @param outPayload 클라이언트로 전달할 {@link IPayload}
     * @return 반환값이 true 이면 나가기 성공, false 이면 실패
     */
    @Override
    public boolean canLeaveRoom(SampleUser user, IPayload payload, IPayload outPayload) {
        return true;
    }

    /**
     * 유저가 룸에서 나갈 때 호출
     * <p/>
     * 유저가 룸에서 나가기 전 마지막 적업을 처리한다
     *
     * @param user 룸에서 나가는 유저
     */
    @Override
    public void onLeaveRoom(SampleUser user) {

    }

    /**
     * 유저가 룸에서 완전히 나간 후 호출
     * <p/>
     * 룸과 룸에 남아있는 유저들에 관한 작업을 처리한다
     */
    @Override
    public void onAfterLeaveRoom() {

    }

    /**
     * 재로그인 시 해당 룸으로 자동으로 재진입 하면서 호출
     *
     * @param user       룸으로 들어가는 유저 객체
     * @param outPayload 클라이언트로 전달할 {@link IPayload}
     */
    @Override
    public void onRejoinRoom(SampleUser user, IPayload payload) {

    }

    /**
     * 해당 룸이 다른 노드로 이동(전송)할 때, 출발 노드에서 전달할 데이터를 꾸리기 위해서 호출
     *
     * @param transferPack 다른 노드로 가지고 갈 데이터를 저장하기 위한 꾸러미인 {@link ITransferPack}
     */
    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    /**
     * 해당 룸이 다른 노드로 이동(전송)할 때, 대상 노드에서 새롭게 생성된 룸 객체를 원래의 상태로 복원, 처리할 타이머 핸들러를 등록하기 위해 호출
     * <p>
     * TimerHandlerTransferPack을 통해서 룸에 등록되어 있던 timerKey 목록을 확인한다
     * <p/>
     * TimerHandlerTransferPack의 reRegister()를 활용해서 사용할 timerHandler를 다시 등록한다
     * <p/>
     * 이 시점에는 룸은 완전히 복구되지 않았으므로, 다른 곳(노드, 유저 등)으로의 메시지 요청이 제한된다
     *
     * @param userList                 이동할 유저 리스트
     * @param transferPack             다른 노드에서 가지고 온 데이터 꾸러미인 {@link ITransferPack}
     * @param timerHandlerTransferPack
     */
    @Override
    public void onTransferIn(List<SampleUser> list, ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    /**
     * 해당 룸이 다른 노드로 이동(전송)이 완료된 후 호출
     */
    @Override
    public void onAfterTransferIn() {

    }

    /**
     * 룸이 속한 노드가 Pause 될때, 해당 룸도 Pause 되면서 호출
     */
    @Override
    public void onPause() {

    }

    /**
     * 룸이 속한 노드가 Resume 될때, 해당 룸도 Resume 되면서 호출
     */
    @Override
    public void onResume() {

    }

    /**
     * 클라이언트에서 파티 매치메이킹을 요청했을 경우 호출
     * <p/>
     * 파티 매치메이킹을 위해서는 2명 이상의 유저가 파타 타입의 네임드룸에 입장해야 한다
     *
     * @param roomType      클라이언트와 서버 사이에 사전 정의한 룸 종류를 구분하는 임의의 값
     * @param matchingGroup 매칭 그룹
     * @param user          파티 매치메이킹을 요청한 유저(방장)
     * @param payload       클라이언트로부터 전달받은 {@link IPayload}
     * @param outPayload    클라이언트로 전달할 {@link IPayload}
     * @return 반환값이 true 이면 파티 매치메이킹 요청 성공, false 이면 실패
     */
    @Override
    public boolean onMatchParty(String roomType, String matchingGroup, SampleUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * 파티 매치메이킹이 취소될 때 호출
     *
     * @param reason 취소된 이유. 타임아웃(TIMEOUT), 사용자의 요청에 의한 취소(CANCEL), 매치 노드의 종료에 의한 취소(SHUTDOWN)
     */
    @Override
    public void onMatchPartyCancel(MatchCancelReason matchCancelReason) {

    }

    /**
     * 룸 매치메이킹이 취소될 때 호출
     *
     * @param reason 취소된 이유. (SHUTDOWN: 매치 노드의 종료에 의한 취소)
     */
    @Override
    public void onForceMatchRoomUnregistered(MatchCancelReason matchCancelReason) {

    }

    /**
     * 룸이 다른 노드로 이동(전송) 가능한 상태인지 확인하기 위해 호출
     *
     * @return 반환값이 true 이면 전송 가능한 상태, false 이면 불가능한 상태
     * <p/>
     * 불가능한 상태의 경우, 만일 SafePause가 진행 중이라면 SafePause가 종료되기 전까진 해당 룸을 전송하기 위해 지속적으로 호출
     */
    @Override
    public boolean canTransfer() {
        return true;
    }
}
```

```java
// 프로토 버퍼 MyGame.GameRoomTest 입력이 들어왔을때 동작하는 메세지 처리 클래스
@GameAnvilController
public class _GameRoomTest {
    // SampleRoom에서 처리하고 싶은 프로토콜과 핸들러를 매핑
    @GameRoomMapping(
        value = MyGame.GameRoomTest.class, // 처리할 프로토 버퍼
        loadClass = SampleRoom.class   // 메세지를 받는 대상 (SampleRoom) 
    )
    public void execute(IRoomDispatchContext ctx) {
        // 여기서 할 작업을 작성
    }
}
```

방 역시 엔진에 등록하기 위해 여러 가지의 설정이 필요합니다. 우선, 어떤 게임 서비스를 위한 방인지 등록 후, 방 타입을 등록합니다. 이 방 타입은 이름 그대로 방의 종류를 구별하기 위한 용도로서, 클라이언트에서도 마찬가지로 API를 호출할 때 사용됩니다. 즉, 해당 API가 서버의 어떤 방 타입에 대한 호출인지 명시하는 것입니다. 그러므로 반드시 서버와 클라이언트 사이에 이러한 방 타입을 임의의 문자열로 사전 정의해두어야 합니다. 이 예제 코드에서는 "BasicRoom"이라는 방 타입을 사용합니다. 이에 대한 더 자세한 설명은 별도의 챕터에서 다시 다루도록 합니다. 마지막으로 이 방 정보가 [채널 간 방 정보 동기화](server-impl-09-channel.md#_4)에 사용될지 여부를 결정할 수 있습니다. 만일 채널 간 정보 동기화가 필요 없다면 생략할 수 있습니다.

방은 가장 기본적인 3가지 콜백인 onCreateRoom, onJoinRoom, onLeaveRoom을 제공합니다. 각각 방의 생성, 참여 그리고 떠나기에 대해 호출됩니다. 사용자는 해당 콜백에서 관련한 기능을 직접 구현할 수 있습니다. 예를 들어, 방을 생성하면서 방장을 지정하고 기본적인 자료 구조들을 초기화할 수 있습니다. 또한 임의의 방 참여 요청에 대해서는 방의 유저 목록을 갱신하거나 각 유저 사이에 상태 동기화 등을 수행할 수 있습니다.

이러한 방의 생성과 참여는 클라이언트의 명시적 요청뿐만 아니라 매치 메이킹에 의해 엔진이 자동으로 처리하기도 합니다. 예를 들어 정원이 2명인 게임에서 유저 A와 유저 B가 매칭되었다면 한 사람은 자동으로 방을 생성하고, 다른 한 사람은 해당 방에 참여하게 됩니다. 이 과정은 유저 A와 유저 B의 요청이 아닌 엔진에서 자동으로 처리하는 것입니다. 이 과정에서도 물론 onCreateRoom과 onJoinRoom 콜백이 호출됩니다.

이러한 방의 콜백을 정리하면 아래의 표와 같습니다.

| 콜백 이름                        | 의미                 | 설명                                                                                                                                                                                              |
|------------------------------|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate                     | 객체 생성              | 객체가 생성되었을 때 호출됩니다. 생성된 타입에서 사용 가능한 API를 사용할 수 있는 컨텍스트를 전달받습니다. 컨텐츠에서 필요하다면 저장해서 사용할 수 있습니다.                                                                                                     |
| onInit                       | 초기화                | 방이 생성될 때 초기화를 위해 호출됩니다. 토픽 등록 등의 해당 방에 대한 초기화 코드를 작성할 수 있습니다.                                                                                                                                   |
| onDestroy                    | 방 사라짐              | 방에서 마지막 유저가 나가고 더 이상 처리할 메시지가 없으면 해당 방은 사라집니다. 이때 호출되는 콜백입니다.                                                                                                                                   |
| onCreateRoom                 | 방 생성               | 클라이언트가 방 생성을 요청하면 호출됩니다. 콘텐츠에서 사용할 유저 목록을 위한 자료 구조를 생성하거나 기타 방 생성과 함께 처리되어야 할 코드를 작성합니다.                                                                                                        |
| onJoinRoom                   | 방 참여               | 클라이언트가 서버상에 존재하는 임의의 방으로 참여를 요청하면 호출됩니다. 콘텐츠에서 사용할 유저 목록을 갱신하거나 방 안의 유저들 사이에 동기화할 정보를 처리할 수 있습니다.                                                                                               |
| canLeaveRoom                 | 방 떠남 확인            | 방에서 나갈 때 호출됩니다. 반환값에 의해 방에서 나갈지 여부가 결정됩니다.                                                                                                                                                      |
| onLeaveRoom                  | 방 떠남               | 클라이언트가 방 떠나기 요청을 하면 호출됩니다. 콘텐츠에서 사용할 유저 목록을 위한 자료 구조를 갱신하거나 기타 방에서 나오면서 함께 처리되어야 할 코드를 작성합니다.                                                                                                   |
| onAfterLeaveRoom             | 방 떠남 후처리           | onLeaveRoom이 성공하면 후처리를 위해 호출됩니다. 방을 나온 직후에 처리할 코드를 작성합니다.                                                                                                                                       |
| onRejoinRoom                 | 방 다시 참여            | 유저가 방에 들어가 있는 상태에서 접속 끊김 등으로 인해 재로그인(ReLogin)을 하는 경우 엔진에 의해 자동으로 해당 방에 재진입(ReJoin) 합니다. 이때, 호출되는 콜백입니다. 재진입 과정에서 동기화에 필요한 정보 등을 처리할 수 있습니다.                                                     |
| onTransferOut                | 기존 노드에서 전송되어 나갈 준비 | 방이 다른 GameNode로 전송될 때, 소스 노드에서 전송을 시작할 때 호출됩니다. 사용자는 이 콜백에서 방 객체와 함께 전송할 데이터 꾸러미를 꾸릴 수 있습니다. 참고로 방 전송은 방 안의 유저들 각각에 대한 유저 전송을 포함합니다.                                                            |
| onTransferIn                 | 새로운 노드로 전송 완료 처리   | 방이 다른 GameNode로 전송될 때, 대상 노드에서 전송 완료하면서 호출됩니다. 사용자는 함께 가지고 온 데이터 꾸러미를 풀어서 원래의 방 상태로 복구할 수 있습니다. 참고로 방 전송은 방 안의 유저들 각각에 대한 유저 전송을 포함합니다.                                                         |
| onAfterTransferIn            | 새로운 노드로 전송 완료 후 처리 | 방이 다른 GameNode로 전송될 때, 대상 노드에서 전송 완료하면서 호출됩니다. 사용자는 함께 가지고 온 데이터 꾸러미를 풀어서 원래의 방 상태로 복구할 수 있습니다. 참고로 방 전송은 방 안의 유저들 각각에 대한 유저 전송을 포함합니다.                                                         |
| onPause                      | 일시 정지              | 콘솔을 통해 GameNode를 일시 정지하면 해당 GameNode의 모든 방에 대해 호출됩니다. 사용자는 노드가 일시 정지될 때 방에서 추가로 처리하고 싶은 코드를 이곳에 구현할 수 있습니다.                                                                                     |
| onResume                     | 재개                 | 콘솔을 통해 GameNode가 일시 정지 상태에서 다시 구동을 재개하면, 해당 GameNode의 모든 방에 대해 호출됩니다. 사용자는 재개 상태에서 방에 대해 처리하고 싶은 코드를 이곳에 구현할 수 있습니다.                                                                            |
| onMatchParty                 | 파티 매치메이킹 요청        | 사용자가 파티 매치메이킹을 요청하면 호출됩니다. 파티 매치 메이킹은 임의의 NamedRoom을 파티 용도로 생성한 후, 방 안의 모든 유저가 하나의 파티로 매칭을 요청하는 기능입니다. 사용자는 이 콜백에서 엔진이 제공하는 파티 매치메이킹 API 혹은 제3의 매치메이킹 솔루션을 임의로 사용할 수 있습니다.                      |
| onMatchPartyCancel           | 파티 매치메이킹 요청 취소     | 사용자가 파티 매치메이킹을 요청하면 호출됩니다. 파티 매치 메이킹은 임의의 NamedRoom을 파티 용도로 생성한 후, 방 안의 모든 유저가 하나의 파티로 매칭을 요청하는 기능입니다. 사용자는 이 콜백에서 엔진이 제공하는 파티 매치메이킹 API 혹은 제3의 매치메이킹 솔루션을 임의로 사용할 수 있습니다.                      |
| onForceMatchRoomUnregistered | 방 매치메이킹이 취소        | 방 매치메이킹이 취소될 때 호출됩니다.                                                                                                                                                                           |
| canTransfer                  | 방 전송이 가능한 상태인지 확인  | 해당 방이 다른 노드로 전송될 수 있는 상태인지 체크하기 위해 호출됩니다.  만일 방에서 아직 게임이 플레이 중이거나 준비가 안 된 경우에는 false를 반환하여 전송을 미룰 수 있습니다. false를 반환한 경우에는 엔진이 임의의 시간 이후에 지속적으로 이 콜백을 호출합니다. 참고로 방 전송은 오직 무점검 패치를 진행할 때에만 사용됩니다. |

## 방 종류

앞서 살펴본 방의 구현 법과 별개로 엔진에서 제공하는 방의 종류는 크게 두 가지입니다. 이 두 가지의 방을 총 네 가지의 방법으로 사용합니다.

| 방 종류        | 설명                                                                                                                                                                                                                                                                                         |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Normal Room | 클라이언트가 명시적으로 CreateRoom을 요청해서 생성합니다. 다른 클라이언트도 해당 방의 아이디를 이용해 명시적으로 JoinRoom을 하여 참여할 수 있습니다. 그러므로 참여할 유저는 미리 해당 방의 아이디를 공유 받아야 합니다.                                                                                                                                                        |
| Named Room  | NamedRoom은 그 이름처럼 유일한 방 이름을 중심으로 명명된 방(NamedRoom)에 대해 동작을 수행합니다. 클라이언트는 서버 군 내에서 유일한 방 이름으로 NamedRoom을 요청합니다. 이때, 만일 서버에 해당 방 이름이 존재하지 않으면 요청자는 직접 방을 생성하게 됩니다. 반대로 만일 서버에 해당 방 이름이 이미 존재할 경우에는 그 방으로 자동 진입하게 됩니다. 예를 들어, 스타크래프트의 커스텀 게임 목록에 나타나는 "3:3 헌터 초보!"와 같은 온갖 방제목을 생각하면 이해하기 쉽습니다. |

이러한 두 가지 방 종류는 총 네 가지의 방법으로 사용됩니다.

| 방 종류        | 사용법                                                                                                                                                                                                                                                                                  |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Normal Room | 1. 클라이언트는 CreateRoom / JoinRoom 요청을 통해 생성 및 참여합니다.<br>2. 룸 매치 메이킹을 통해 NormalRoom을 생성하거나 참여할 수 있습니다. 또한 CreateRoom으로 만든 방도 룸 매치 메이킹 대상으로 등록이 가능합니다. 이때, 방 아이디는 엔진에 의해 관리되고 매칭되는 방과 유저 사이에서 자동으로 공유됩니다.                                                                                |
| Named Room  | 3. 클라이언트는 NamedRoom 요청을 통해 생성 및 참여합니다.<br>4. 유저 매치 메이킹을 통해 NamedRoom을 생성하거나 참여할 수 있습니다. 이때, 생성되는 NamedRoom의 방 이름은 엔진에서 고유하게 생성하고 관리합니다. 룸 매치 메이킹과 달리 일반적인 NamedRoom으로 생성한 방은 유저 매치 메이킹 대상이 될 수 없습니다. 단, 파티 매치 메이킹을 위해 NamedRoom으로 파티 룸을 생성한 후 여러 명의 유저들이 하나의 파티로 매치 메이킹을 요청할 수 있습니다. |

## 채널

GameNode들은 그 용도에 맞춰 논리적으로 그룹화할 수 있습니다. 이러한 논리 그룹을 [채널](server-impl-09-channel)이라고 합니다. 예를 들어, GameNode 1과 2를 "Beginner" 채널로 묶고, GameNode 3과 4를 "Expert" 채널로 묶을 수 있습니다. 이에 대한 자세한 설명은 [별도의 챕터](server-impl-09-channel)에서 더 자세하게 다루도록 합니다.
