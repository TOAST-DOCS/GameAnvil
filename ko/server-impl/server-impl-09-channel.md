## Game > GameAnvil > 서버 개발 가이드 > 채널

## 채널(Channel)

![channel-sync2.png](https://static.toastoven.net/prod_gameanvil/images/channel-sync2.png)

채널은 단일 서버 군을 논리적으로 나눌 수 있는 방법 중 하나입니다. GameAnvil은 한 개 이상의 게임 노드를 포함할 경우에 채널을 설정할 수 있습니다. 기본적으로 GameAnvilConfig을 통해 게임 노드에
아래의 예제처럼 채널을 설정할 수 있습니다. 이 예제에서 4개의 게임 노드에 대해 각각 '초보', '초보', '고수', '고수'를 설정합니다.

```json
"game": [
  {
    "serviceId": 1, // 서버군에서 유니크해야함 [0 ~ 99]
    "serviceName": "MyGame", // 서버군에서 유니크해야함
    "nodeCnt": 4, // 노드 개수, nodeCnt가 0일 경우 Game Node를 생성하지 않는다.머신 core 수로 넣는 것을 권고 [0 ~99]
    "channelIDs": [
      ["초보"], 
      ["초보"],
      ["고수"],
      ["고수"]
    ], // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. "" 문자열로 채널 구분없이 중복사용도 가능)
    "userTimeout": 5000       // disconnect 이후의 유저객체 제거 타임아웃. [0 ~ 604800000(7일)]
  }
],
```

채널을 사용하고 싶지 않을 경우에는 채널 ID를 모두 다음과 같이 ""로 설정하면 됩니다. 이 경우 모든 채널 기능도 사용할 수 없습니다. 즉, 아래의 게임 노드 4개는 모두 채널과 관계없이 독립적으로 동작합니다.

```json
"game": [
  {
    "serviceId": 1, // 서버군에서 유니크해야함 [0 ~ 99]
    "serviceName": "MyGame", // 서버군에서 유니크해야함
    "nodeCnt": 4, // 노드 개수, nodeCnt가 0일 경우 Game Node를 생성하지 않는다.머신 core 수로 넣는 것을 권고 [0 ~99]
    "channelIDs": [
      [""],
      [""],
      [""],
      [""]
    ], // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. "" 문자열로 채널 구분없이 중복사용도 가능)
    "userTimeout": 5000       // disconnect 이후의 유저객체 제거 타임아웃. [0 ~ 604800000(7일)]
  }
],
```
만일 전체 게임 노드를 하나의 채널로 관리하고 싶다면 다음과 같이 유효한 문자열로 채널 아이디를 입력합니다.

```json
"game": [
  {
    "serviceId": 1,           // 서버군에서 유니크해야함 [0 ~ 99]
    "serviceName": "MyGame",  // 서버군에서 유니크해야함
    "nodeCnt": 4,             // 노드 개수, nodeCnt가 0일 경우 Game Node를 생성하지 않는다.머신 core 수로 넣는 것을 권고 [0 ~99]
    "channelIDs": [
      ["초보"],
      ["초보"],
      ["초보"],
      ["초보"]
    ],                        // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. "" 문자열로 채널 구분없이 중복사용도 가능)
    "userTimeout": 5000       // disconnect 이후의 유저객체 제거 타임아웃. [0 ~ 604800000(7일)]
  }
],
```

기본적으로 하나의 스레드에 하나의 게임 노드가 설정이 되지만 효율적인 리소스 활용을 위해서 하나의 스레드에 여러 개의 게임 노드가 다음과 같이 설정이 가능합니다. 게임 노드의 총개수는 nodeCnt로 지정한 4개가 아니라 실제로 채널 정보에 설정한 3(nodeCnt 값 무시) 개가 됩니다. 게임 노드 스레드는 두 그룹으로 되어 2개가 됩니다. 스레드 1에는 게임 노드 1("초보"), 게임 노드 2("중수")가 스레드 2애는 게임 노드 3("고수")가 설정이 됩니다.
```json
"game": [
  {
    "serviceId": 1,           // 서버군에서 유니크해야함 [0 ~ 99]
    "serviceName": "MyGame",  // 서버군에서 유니크해야함
    "nodeCnt": 4,             // 노드 개수, nodeCnt가 0일 경우 Game Node를 생성하지 않는다.머신 core 수로 넣는 것을 권고 [0 ~99]
    "channelIDs": [
      ["초보", "중수"], // 스레드1
      ["고수"]         // 스레드2
    ],                        // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. "" 문자열로 채널 구분없이 중복사용도 가능)
    "userTimeout": 5000       // disconnect 이후의 유저객체 제거 타임아웃. [0 ~ 604800000(7일)]
  }
],
```

이러한 채널을 통해 다음과 같은 기능을 사용할 수 있습니다.

* 채널 정보를 관리할 수 있습니다. 해당 채널에 속한 유저와 방 정보를 사용할 수 있게 됩니다.
* 채널별 유저와 방의 개수를 조회할 수 있습니다.
* 채널 단위로 메시지를 전송할 수 있습니다. publishToChannel API를 사용하면 대상 채널에 속한 모든 게임 노드로 메시지를 전달합니다.

## 채널 정보 관리

사용자는 채널에서 관리할 정보를 직접 구현할 수 있습니다. 이러한 정보는 같은 채널 안에서 자동으로 동기화가 됩니다.

### 채널 유저 정보

우선 채널에서 유저 정보를 관리하기 위해서는 다음과 같이 유저 클래스를 구현할 때 enableChannelInfo() 메서드를 통해서 채널 유저 정보 관리를 활성화시켜야 합니다.

```java
public class Main {
    public static void main(String[] args) {
        // 게임앤빌 서버 설정 빌더
        var gameAnvilServerBuilder = GameAnvilServer.getInstance().getServerTemplateBuilder();

        // 컨텐츠 프로토콜 등록.
        gameAnvilServerBuilder.addProtocol(SampleGame.class);

        // "MyGame"이라는 서비스를 위한 GameNode로 엔진에 등록
        var gameServiceBuilder = gameAnvilServerBuilder.createGameService("MyGame");
        gameServiceBuilder.gameNode(SampleGameNode::new, config -> {
            config.protoBufferHandler(MyGame.GameNodeTest.class, new _GameNodeTest());
        });

        // "BasicUser"라는 유저 타입의 유저를 엔진에 등록
        gameServiceBuilder.user("BasicUser", SampleGameUser::new, config -> {
            // 채널간의 정보 동기화 설정
            config.enableChannelInfo();
            
            config.protoBufferHandler(MyGame.GameUserTest.class, new _GameUserTest());
        });

        GameAnvilServer.getInstance().run();
    }
}
```

그리고 추가로 IChannelUserInfo 인터페이스를 구현합니다. 이때, 사용자가 채널에서 관리하고 싶은 정보를 모두 포함하면 됩니다.

```java
public class GameChannelUserInfo implements IChannelUserInfo, Comparable<GameChannelUserInfo> {

    private String userType = "";
    private int userId = 0;
    private String accountId = "";

    public GameChannelUserInfo(String userType, int userId, String accountId) {
        this.userType = userType;
        this.userId = userId;
        this.accountId = accountId;
    }

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
     * 유저 아이디 반환
     *
     * @return 유저 아이디 반환
     */
    @Override
    public int getUserId() {
        return userId;
    }

    /**
     * 어카운트 아이디 반환
     *
     * @return 어카운트 아이디 반환
     */
    @Override
    public String getAccountId() {
        return accountId;
    }

    @Override
    public int compareTo(GameChannelUserInfo o) {
        if (o.userId != this.userId) {
            return -1;
        } else {
            return 0;
        }
    }
}
```

이렇게 작성한 채널 유저 정보는 다음과 같이 유저 객체에서 추가하거나 갱신할 수 있습니다. 이때, userContext.updateChannelUserInfo() API를 사용합니다. 만일 해당 유저 객체가 서버에서 로그아웃되면 해당 채널
유저 정보도 자동으로 함께 제거됩니다. 다음은 이에 대한 pseudo 코드입니다.

```java
public class SampleGameUser implements IUser {
    GameChannelUserInfo channelUserInfo = new GameChannelUserInfo();
  
	...

    // 유저가 로그인 시 채널 유저 정보를 추가합니다.
    onLogin(...) {
		...
        channelUserInfo.setUserId(getId())
        channelUserInfo.setAccountId(getAccountId());
        channelUserInfo.setLevel(getLevel());

        userContext.updateChannelUserInfo(channelUserInfo);
		...
    }

    // 채널 이동 시 대상 채널에 새로운 채널 유저 정보를 추가합니다.
    onMoveInChannel(...) {
		...
        channelUserInfo.setUserId(getId());
        channelUserInfo.setAccountId(getAccountId());
        channelUserInfo.setLevel(getLeve());

        userContext.updateChannelUserInfo(channelUserInfo);
		...
    }

    // 사용자 콘텐츠에서 변경된 유저 정보를 언제든 갱신할 수 있습니다.
    updateLevel(int level) {
        channelUserInfo.setLevel(getLevel());

        userContext.updateChannelUserInfo(channelUserInfo);
    }
}
```

참고로 채널 이동 시에 이전 채널에서는 자동으로 해당 채널 유저 정보를 삭제하므로 사용자는 대상 채널에서 새롭게 추가할 정보만 신경 쓰면 됩니다.

### 채널 방 정보

채널에서 방 정보를 관리하기 위해서는 앞서 살펴본 게임 유저와 마찬가지로 방 클래스를 구현할 때 방 정보를 설정합니다.

```java
public class Main {
    public static void main(String[] args) {
        // 게임앤빌 서버 설정 빌더
        var gameAnvilServerBuilder = GameAnvilServer.getInstance().getServerTemplateBuilder();

        // 컨텐츠 프로토콜 등록.
        gameAnvilServerBuilder.addProtocol(SampleGame.class);

        // "MyGame"이라는 서비스를 위한 GameNode로 엔진에 등록
        var gameServiceBuilder = gameAnvilServerBuilder.createGameService("MyGame");
        gameServiceBuilder.gameNode(SampleGameNode::new, config -> {
            config.protoBufferHandler(MyGame.GameNodeTest.class, new _GameNodeTest());
        });

        // "BasicRoom"라는 룸 타입의 유저를 엔진에 등록
        gameServiceBuilder.room("BasicRoom", SampleGameRoom::new, config -> {
            // 채널간의 정보 동기화 설정
            config.enableChannelInfo();

            config.protoBufferHandler(MyGame.GameRoomTest.class, new _GameRoomTest());
        });

        GameAnvilServer.getInstance().run();
    }
}
```

그리고 추가로 IChannelRoomInfo 인터페이스를 구현합니다. 이때, 사용자가 채널에서 관리하고 싶은 방 관련 정보를 모두 포함하면 됩니다.

```java
public class GameChannelRoomInfo implements IChannelRoomInfo, Comparable<GameChannelRoomInfo> {

    public static final int MAX_ENTRY_USER = 4;
    private int roomId = 0;

    private long allUserMoney; // for room sorting comparator.


    /**
     * 룸 아이디 반환
     *
     * @return 룸 아이디 반환
     */
    @Override
    public int getRoomId() {
        return roomId;
    }

    /**
     * 룸 정보 복사
     *
     * @return 복사된 룸 정보를 반환
     * @throws CloneNotSupportedException 복사가 안되는 경우 발생
     */
    @Override
    public IChannelRoomInfo copy() throws CloneNotSupportedException {
        GameChannelRoomInfo channelRoomInfo = (GameChannelRoomInfo)super.clone();
        channelRoomInfo.testListInfo = new ArrayList<>(testListInfo);

        return channelRoomInfo;
    }

    @Override
    public int compareTo(GameChannelRoomInfo o) {
        return (int)(o.getAllUserMoney() - this.allUserMoney); // descending.
    }

}
```

이렇게 작성한 채널 방 정보는 다음과 같이 방 객체에서 추가하거나 갱신할 수 있습니다. 이때, roomContext.updateChannelRoomInfo() API를 사용합니다. 만일 해당 방 객체가 서버에서 사라지면 해당 채널 방 정보도
자동으로 함께 제거됩니다. 다음은 이에 대한 pseudo 코드입니다.

```java
public class SampleGameRoom implements IRoom<SampleGameUser> {
    GameChannelRoomInfo channelRoomInfo = new GameChannelRoomInfo();
  
	...

    // 방 생성 시 updateChannelRoomInfo 함수를 통해서 채널 방 정보를 추가합니다.
    onCreateRoom(...) {
		...
        channelRoomInfo.setRoomId(getId());
        channelRoomInfo.setRoomName(getRoomName());
        channelRoomInfo.setUserCount(0);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }

    // 방에 새로운 유저가 들어오면 updateChannelRoomInfo 함수를 통해서 변경된 방 정보를 적용합니다.
    onJoinRoom(...) {
		...
        channelRoomInfo.setUserCount(++userCount);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }

    // 방에 유저가 나가면 updateChannelRoomInfo 함수를 통해서 변경된 방 정보를 적용합니다.
    onPostLeaveRoom(...) {
		...
        channelRoomInfo.setUserCount(--userCount);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }
}
```

## 채널 정보 동기화

동일한 채널의 게임 노드는 서로 채널 관련 정보를 공유합니다. 예를 들면 같은 채널에 속한 하나의 게임 노드에서 앞서 살펴본 방식으로 유저나 방 정보가 변경되면 해당 채널의 나머지 게임 노드는 다음과 같은 콜백
메서드가 호출됩니다. 이러한 콜백을 이용하여 동일한 채널 내의 모든 게임 노드가 정보를 동기화할 수 있습니다. 다음은 게임 노드에서 이러한 채널 동기화를 위해 사용되는 콜백 메서드입니다.

```java
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
```

### 클라이언트로 채널 정보 동기화

클라이언트는 서버로 언제든 채널 정보를 요청할 수 있습니다. 이때, 앞서 살펴본 게임 노드의 콜백 메서드 중 onChannelInfo가 호출됩니다. 단, 클라이언트의 잘못된 구현 혹은 악의적인 사용을 막고자 이 콜백
메서드 호출은 최소한의 재호출 주기(기본값 1초)를 가집니다. 예를 들어 클라이언트가 1초 동안 10번의 채널 정보 요청을 하더라도 서버는 단 1회의 onChannelInfo 콜백 메서드를 호출합니다. 나머지 9번의
요청은 이전에 캐싱 해 둔 정보를 전달합니다. 다음은 이러한 onChannelInfo를 구현한 pseudo 코드입니다.

```java
public void onChannelInfo(Payload outPayload) {

    // 클라이언트에 보낼 사용자 정의 채널 정보 생성
    Game.GameChannelInfo.Builder channelInfoBuilder = Game.GameChannelInfo.newBuilder();

    channelInfoBuilder.setChannelId(gameNodeContext.getChannelId());

    // getChannelUserInfo API를 통해서 채널 유저 정보를 가져옵니다.
    for (IChannelUserInfo channelUserInfo : getContext().getChannelUserInfo(StringValues.userType)) {
        Game.GameChannelInfo.Builder channelUserInfoBuilder = Game.GameChannelInfo.newBuilder();
        channelUserInfoBuilder.setAccountId(channelUserInfo.getAccountId());
        channelUserInfoBuilder.setUserId(channelUserInfo.getUserId());

        channelInfoBuilder.addChannelUserInfo(channelUserInfoBuilder.build());
    }

    // getChannelRoomInfo API를 통해서 채널 방 정보를 가져옵니다.
    for (IChannelRoomInfo channelRoomInfo : getContext().getChannelRoomInfo(StringValues.roomType_2_User_Match)) {
        Game.GameChannelRoomInfo.Builder channelRoomInfoBuilder = Game.GameChannelRoomInfo.newBuilder();
        channelRoomInfoBuilder.setRoomId(channelRoomInfo.getRoomId());
        channelRoomInfoBuilder.setUserCount(((GameChannelRoomInfo)channelRoomInfo).getUserCnt());

        channelInfoBuilder.addChannelRoomInfo(channelRoomInfoBuilder.build());
    }

    // outPayload에 클라이언트에게 보낼 채널 정보를 추가합니다.
    outPayload.add(channelInfoBuilder);
}
```

### 클라이언트로 채널에 속한 유저와 방의 개수 전달하기

GameAnvil 커넥터는 이러한 정보를 요청하기 위해 GetChannelCountInfo API를 제공합니다. 엔진에서 항상 채널 단위의 유저/방 개수를 관리하고 있으므로 사용자는 별도의 구현을 할 필요가
없습니다.
