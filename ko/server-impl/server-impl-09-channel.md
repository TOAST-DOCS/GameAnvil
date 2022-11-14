## Game > GameAnvil > 서버 개발 가이드 > 채널



## 1. 채널 (Channel)

![channel-sync2.png](https://static.toastoven.net/prod_gameanvil/images/channel-sync2.png)

채널은 단일 서버군을 논리적으로 나눌 수 있는 방법 중 하나입니다. GameAnvil은 한 개 이상의 게임 노드를 포함할 경우에 채널을 설정할 수 있습니다. 기본적으로 GameAnvilConfig을 통해 게임 노드에 아래의 예제처럼 채널을 설정할 수 있습니다. 이 예제에서 4개의 게임 노드에 대해 각각 ch1,ch1,ch2,ch2를 설정합니다.

```json
"game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "MyGame",
      "channelIDs": [
        "ch1",
        "ch1",
        "ch2",
        "ch2"
      ],
    }
```

채널을 사용하고 싶지 않을 경우에는 채널 ID를 모두 다음과 같이 ""로 설정하면 됩니다. 이 경우 모든 채널 기능도 사용할 수 없습니다. 즉, 아래의 게임 노드 4개는 모두 채널과 관계없이 독립적으로 동작합니다.

```json
"game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "MyGame",
      "channelIDs": [
        "",
        "",
        "",
        ""
      ],
    }
```

만일 전체 게임 노드를 하나의 채널로 관리하고 싶다면 다음과 같이 유효한 문자열로 채널 아이디를 입력해줍니다.

```json
"game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "MyGame",
      "channelIDs": [
        "OnlyOneChannel",
        "OnlyOneChannel",
        "OnlyOneChannel",
        "OnlyOneChannel"
      ],
    }
```

이러한 채널을 통해 다음과 같은 기능을 사용할 수 있습니다.

* 채널 정보를 관리할 수 있습니다. 해당 채널에 속한 유저와 방 정보를 사용할 수 있게 됩니다.
* 채널 별 유저와 방의 개수를 조회할 수 있습니다.
* 채널 단위로 메시지를 전송할 수 있습니다. publishToChannel API를 사용하면 대상 채널에 속한 모든 게임 노드로 메시지를 전달합니다.



## 2. 채널 정보 관리

사용자는 채널에서 관리할 정보를 직접 구현할 수 있습니다. 이러한 정보는 같은 채널 안에서 자동으로 동기화가 됩니다.

### 2.1. 채널 유저 정보

우선 채널에서 유저 정보를 관리하기 위해서는 다음과 같이 유저 클래스를 구현할 때 어노테이션을 통해 채널 유저 정보를 활성화시켜야 합니다.

```java
@ServiceName("MyGame")
@UserType("BasicUser")
@UseChannelInfo // 채널 유저 정보 활성화
public class SampleGameUser extends BaseUser implements TimerHandler {
	...
}
```

그리고 추가로 BaseChannelUserInfo를 구현합니다. 이 때, 사용자가 채널에서 관리하고 싶은 정보를 모두 포함하면 됩니다.

```java
public class GameChannelUserInfo implements Serializable, BaseChannelUserInfo {
	// 채널에서 보여줄 유저 정보
	private int userId = 0;
	private String accountId = "";
	private int level = 0;

    ...
  
    /**
     * 변경될 Channel User 정보의 User Id.
     *
     * @return int type 으로 UserId 반환.
     */
    @Override
    int getUserId() {
        return userId;
    }

    /**
     * 변경될 Channel User 정보의 Account Id.
     *
     * @return String type 으로 AccountId 반환.
     */
    @Override
    String getAccountId() {
        return accountId;
    }
}
```

이렇게 작성한 채널 유저 정보는 다음과 같이 유저 객체에서 추가하거나 갱신할 수 있습니다. 이 때, updateChannelUserInfo API를 사용합니다. 만일 해당 유저 객체가 서버에서 로그아웃되면 해당 채널 유저 정보도 자동으로 함께 제거됩니다. 다음은 이에 대한 pseudo 코드입니다.

```java
public class SampleGameUser extends BaseUser {
	GameChannelUserInfo channelUserInfo = new GameChannelUserInfo();
  
	...

	// 유저가 로그인 시 채널 유저 정보를 추가합니다.
	onLogin(...) throws SuspendExecution {
		...
		channelUserInfo.setUserId(getId())
		channelUserInfo.setAccountId(getAccountId());
		channelUserInfo.setLevel(getLevel());
      
		updateChannelUserInfo(channelUserInfo);
		...
	}
  
	// 채널 이동 시 대상 채널에 새로운 채널 유저 정보를 추가합니다.
	onMoveInChannel(...) throws SuspendExecution {
		...
		channelUserInfo.setUserId(getId());
		channelUserInfo.setAccountId(getAccountId());
		channelUserInfo.setLevel(getLeve());
      
		updateChannelUserInfo(channelUserInfo);
		...
	}
  
	// 사용자 컨텐츠에서 변경된 유저 정보를 언제든 갱신할 수 있습니다.
	updateLevel(int level) {
		channelUserInfo.setLevel(getLevel());
      
		updateChannelUserInfo(channelUserInfo);
	}
}
```

참고로 채널 이동 시에 이전 채널에서는 자동으로 해당 채널 유저 정보를 삭제하므로 사용자는 대상 채널에서 새롭게 추가할 정보만 신경쓰면 됩니다.

### 2.2. 채널 방 정보

채널에서 방 정보를 관리하기 위해서는 앞서 살펴본 게임 유저와 마찬가지로 방  클래스를 구현할 때 어노테이션을 통해 채널 방 정보를 활성화시켜야 합니다.

```java
@ServiceName("MyGame")
@RoomType("BasicRoom")
@UseChannelInfo // 채널 정보 활성화
public class GameRoom extends BaseUser implements TimerHandler {
    ...
}
```

그리고 추가로 BaseChannelRoomInfo를 구현합니다. 이 때, 사용자가 채널에서 관리하고 싶은 방 관련 정보를 모두 포함하면 됩니다.

```java
public class GameChannelRoomInfo implements Serializable, BaseChannelRoomInfo {
	private int roomId = 0;
	private String roomName = "";
	private int userCount = 0;

    ...
      
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
     * Room 정보를 복사 한다.
     *
     * @return RoomInfo 로 복사된 Room 정보를 반환.
     * @throws CloneNotSupportedException 복사가 안되는 경우.
     */
    @Override
    public BaseChannelRoomInfo copy() throws CloneNotSupportedException {
        GameChannelRoomInfo channelRoomInfo = (GameChannelRoomInfo)super.clone();
      
        // 데이터 복사가 필요할 경우 여기에서 진행합니다.
      
        return channelRoomInfo;
    }   
}
```

이렇게 작성한 채널 방 정보는 다음과 같이 방 객체에서 추가하거나 갱신할 수 있습니다. 이 때, updateChannelRoomInfo API를 사용합니다. 만일 해당 방 객체가 서버에서 사라지면 해당 채널 방 정보도 자동으로 함께 제거됩니다. 다음은 이에 대한 pseudo 코드입니다.

```java
public class GameRoom extends BaseRoom<GameUser> {
	GameChannelRoomInfo channelRoomInfo = new GameChannelRoomInfo();
  
	...
      
	// 방 생성 시 updateChannelRoomInfo 함수를 통해서 채널 방 정보를 추가합니다.
	onCreateRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setRoomId(getId());
		channelRoomInfo.setRoomName(getRoomName());
		channelRoomInfo.setUserCount(0);
      
		updateChannelRoomInfo(channelUserInfo);
	}
  
	// 방에 새로운 유저가 들어오면 updateChannelRoomInfo 함수를 통해서 변경된 방정보를 적용합니다.
	onJoinRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setUserCount(++userCount);
      
		updateChannelRoomInfo(channelUserInfo);
	}
  
    // 방에 유저가 나가면 updateChannelRoomInfo 함수를 통해서 변경된 방정보를 적용합니다.
	onPostLeaveRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setUserCount(--userCount);
      
		updateChannelRoomInfo(channelUserInfo);
	}
}
```

## 3. 채널 정보 동기화

동일한 채널의 게임 노드는 서로 채널 관련 정보를 공유합니다. 예를 들면 같은 채널에 속한 하나의 게임 노드에서 앞서 살펴본 방식으로 유저나 방 정보가 변경되면 해당 채널의 나머지 게임 노드는 다음과 같은 콜백 메서드가 호출됩니다. 이러한 콜백을 이용하여 동일한 채널 내의 모든 게임 노드가 정보를 동기화활 수 있습니다. 다음은 게임 노드에서 이러한 채널 동기화를 위해 사용되는 콜백 메서드입니다.

```java
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
```

### 3.1. 클라이언트로 채널 정보 동기화

클라이언트는 서버로 언제든 채널 정보를 요청할 수 있습니다. 이 때, 앞서 살펴본 게임 노드의 콜백 메서드 중 onChannelInfo가 호출됩니다. 단, 클라이언트의 잘못된 구현 혹은 악의적인 사용을 막고자 이 콜백 메서드 호출은 최소한의 재호출 주기(기본값 1초)를 가집니다.  예를 들어 클라이언트가 1초 동안 10번의 채널 정보 요청을 하더라도 서버는 단 1회의 onChannelInfo 콜백 메서드를 호출합니다. 나머지 9번의 요청은 이전에 캐싱해둔 정보를 전달합니다. 다음은 이러한 onChannelInfo를 구현한 pseudo 코드입니다.

```java
public void onChannelInfo(Payload outPayload) throws SuspendExecution {

    // 클라이언트에 보낼 사용자 정의 채널 정보 생성
	Game.GameChannelInfo.Builder channelInfoBuilder = Game.GameChannelInfo.newBuilder();

	channelInfoBuilder.setChannelId(this.getChannelId());

	// getChannelUserInfo API를 통해서 채널 유저 정보를 가져옵니다.
	for (BaseChannelUserInfo channelUserInfo : getChannelUserInfo(UserType_1)) {
		GameChannelUserInfo gameChannelUserInfo = (GameChannelUserInfo) channelUserInfo;

        Game.GameChannelUserInfo.Builder channelUserInfoBuilder = Game.GameChannelUserInfo.newBuilder();
        channelUserInfoBuilder.setUserName(gameChannelUserInfo.getUserName());
        channelUserInfoBuilder.setLevel(gameChannelUserInfo.getlevel());

    	channelInfoBuilder.addChannelUserInfos(channelUserInfoBuilder.build());
    }
  
	// getChannelRoomInfo API를 통해서 채널 방 정보를 가져옵니다.
    for (BaseChannelRoomInfo channelRoomInfo : getChannelRoomInfo(RoomType_2)) {  
	   	GameChannelRoomInfo gameChannelRoomInfo = (GameChannelRoomInfo) channelRoomInfo;
  
    	Game.GameChannelRoomInfo.Builder channelRoomInfoBuilder = Game.GameChannelRoomInfo.newBuilder();
        channelRoomInfoBuilder.setRoomId(gameChannelRoomInfo.getRoomId());
        channelRoomInfoBuilder.setRoomName(gameChannelRoomInfo.getRoomName());
        channelRoomInfoBuilder.setUserCount(gameChannelRoomInfo.getUserCnt());

        channelInfoBuilder.addChannelRoomInfos(channelRoomInfoBuilder.build());
	}

    // outPayload에 클라이언트에게 보낼 채널 정보를 추가합니다.
	outPayload.add(new Packet(channelInfoBuilder));
}
```

### 3.2. 클라이언트로 채널에 속한 유저와 방의 개수 전달하기

GameAnvil 커넥터는 이러한 정보를 요청하기 위해 GetChannelCountInfo API를 제공합니다. 엔진에서 항상 채널 단위의 유저/방 개수를 관리하고 있으므로 사용자는 별도의 구현을 할 필요가 없습니다.
