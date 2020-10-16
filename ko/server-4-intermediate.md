# 중급 주제

## 1. 분산 서버

앞서 기본 개념에서 GameAnvil의 노드 구성은 아래의 그림과 같다고 했습니다. 즉, 하나의 프로세스는 여러 가지의 노드를 자유롭게 구성해서 구동할 수 있습니다. 단, 모든 GameAnvil 프로세스는 반드시 하나의 IPC 노드가 필수적으로 포함됩니다. 이 IPC 노드는 GameAnvil 프로세스 간의 통신을 담당합니다. 사실 실제 네트워크 처리를 담당하는 로우 레벨 노드가 더 있지만 사용자는 이 부분을 통틀어 IPC 노드로 이해하면 됩니다. 

![Node Layer.png](./images/NodeLayer.png)



### 1-1. 노드간 통신

이러한 IPC 노드를 통해 두 개 이상의 GameAnvil 프로세스가 통신하는 모습은 아래의 그림과 같습니다. 이 그림에서 두 개의 GameAnvil 프로세스는 서로 다른 구성의 노드들을 구동합니다. 이 때, 각각의 노드들은 서로 통신이 가능합니다. 

서로 다른 프로세스의 노드들과 통신하기 위해 각 노드들은 IPC 노드를 통해 메시지를 전달합니다. 반면에 동일한 프로세스 상의 노드들은 큐를 이용해서 상호 통신합니다. 그러므로 이 경우에는 IPC 노드를 통하지 않습니다.

![Node Layer.png](./images/IPC.png)



### 1-2. Meetpoint

그럼, 이러한 IPC를 위해 프로세스는 상호 접속을 어떻게 하는걸까요? 그 답은 Meetpoint 입니다. GameAnvil은 Meetpoint 주소를 설정할 수 있습니다. 아래와 같은 형태인데 하나 이상의 IP 주소쌍을 설정할 수 있습니다. GameAnvil 프로세스는 최초 구동 시에 설정된 Meetpoint 주소 중 하나에 대해 접속을 시도하여 전체 서버군 정보를 동기화 합니다.

```json
"common": {
    
    "meetEndPoints": [
      "10.1.2.1:16000",
      "10.1.2.2:16000",
    ],
    
}
```

<br>

## 2. Connection과 Session

클라이언트는 Gateway 노드로 접속(Connection)을 합니다. 이 접속을 통해 계정과 유저 정보를 바탕으로 인증과 로그인을 진행할 수 있습니다. 로그인까지 완료되면 임의의 Game 노드에 유저 객체가 생성됩니다. 이는 Gateway 노드와 해당 Game 노드 사이에는 논리적인 세션이 생성되었음을 의미합니다. 이렇게 접속과 세션 생성이 완료되면 해당 유저는 게임 진행이 가능해집니다. 



### 2-1. Connection Recovery

만일, 클라이언트와 Gateway 노드 사이의 접속이 끊기면 아래의 그림과 같이 접속 복구(Connection Recovery)가 진행됩니다. 재접속을 하는 과정에서 클라이언트는 여러대의 Gateway 노드 중 이전과 다른 곳에 접속을 시도할 수도 있습니다. 이 경우 유저 객체가 존재하는 Game 노드에 대한 위치 정보를 바탕으로 새롭게 세션을 복구합니다. 그러므로 유저는 게임 진행 중에 재접속을 하더라도 이전의 게임 상태를 이어갈 수 있습니다.

![Node Layer.png](./images/ConnectionRecovery.png)



### 2-2. Location Node

앞서 살펴본 접속 복구(Connection Recovery) 그림에서 Location 노드가 보입니다. 이 Location 노드는 GameAnvil이 내부적으로 유저와 방 등의 위치 정보를 관리하는 용도로 사용합니다. 사용자는 Location 노드에 대해 직접 구현하거나 사용할 수 없습니다. 하지만 위치 정보를 관리하는 Location 노드의 존재를 알야아 전체적인 GameAnvil 시스템의 흐름을 이해할 수 있기 때문에 여기에서 간단하게 언급을 하고자 합니다.

위의 접속 복구(Connection Recovery)를 예로 들어 보겠습니다. 클라이언트가 최초 접속을 하여 Game 노드로 로그인을 시도하는 과정에서 이와 관련된 세션과 유저에 대한 위치 정보는 모두 Location 노드에 저장됩니다. 그러므로 재접속을 진행할 경우에는 이전 접속 과정에서 저장해 두었던 이 위치 정보를 조회할 수 있습니다. 이러한 위치 정보는 GameAnvil 내부적으로 유저나 방의 위치 정보를 조회하고 이를 바탕으로 메시지를 전달하는 용도 등으로 매우 중요하게 사용됩니다.

<br>

## 3. UserTransfer

게임 노드에는 게임 유저 객체가 생성됩니다. 이 게임 유저 객체는 게임 노드 사이에서 언제든지 전송될 수 있습니다. 예를 들면 1번 게임 노드의 유저 객체가 2번 게임 노드에 생성된 방으로 들어가는 과정에서 게임 노드간 유저 객체 전송이 일어납니다. 이 기술은 GameAnvil의 가장 핵심이자 근간이 되며 유저 객체를 잘 직렬화/역직렬화 해주어야 의도한대로 동작합니다. 이러한 직렬화/역직렬화 대상은 GameAnvil 사용자가 직접 구현할 수 있습니다. 즉, 해당 게임 유저 객체의 어떤 값을 전송할 것인지 결정하는 것이죠.

이러한 유저 전송이 발생하는 경우는 크게 세 가지로 나눌 수 있습니다.

* 첫 째, 매치메이킹을 포함한 방 생성이나 방 참여 로직에 의해 유저 객체는 다른 게임 노드로 전송될 수 있습니다. 이 때, 구현에 따라 다른 채널의 방으로 진입할 수도 있습니다.
* 둘 째, 다른 채널로 이동 할 때 발생합니다. 하나의 게임 노드는 하나의 채널에 속할 수 있습니다. 그러므로 채널을 이동하는 것은 게임 노드 사이의 이동을 의미합니다.
  * 첫 번째 경우처럼 매치메이킹 등을 이용해서 다른 채널의 방으로 진입할 경우에는 엔진 내부에서 묵시적으로 처리합니다.
  * 클라이언트는 명시적으로 채널 이동을 요청할 수도 있습니다.
* 셋 째, 임의의 게임 노드에 대해 무정지 점검(NonStopPatch)을 진행하면 해당 노드의 유저 객체들은 다른 유효한 게임 노드들로 분산되어 전송 됩니다. 이 경우는 운영 측면에서 GameAnvil Console을 통해 명시적으로 명령을 내린 경우입니다.

<br>

### 3-1.  유저 전송 구현

실제 유저 전송은 GameAnvil이 내부적으로 조용하게 처리합니다. 이 때, 클라이언트는 자신의 게임 유저 객체가 서버 사이에서 전송되는지 인지하지 못합니다. 즉, 다른 게임 노드의 방으로 들어가더라도 클라이언트는 단지 하나의 GameAnvil 서버군에서 임의의 방으로 들어간 것 뿐이죠. 

단, 다른 게임 노드로 유저 객체를 전송할 때 어떤 데이터를 함께 전송할지는 사용자가 지정할 수 있습니다. 간단히 지정만 해두면 이 데이터는 유저 객체의 일부로 함께 직렬화 됩니다. 전송할 대상 게임 노드에서는 역직렬화를 신경쓸 필요 없이 쉽게 해당 객체들에 접근해서 사용할 수 있습니다.

다음은 이러한 유저 전송에 관련된 콜백 메소드들입니다. 우선 "함께 전송할 데이터 지정하기" 입니다. 이를 위해 BaseUser 추상 클래스를 상속받은 게임 유저 클래스에 아래의 콜백을 구현합니다. 방법은 간단합니다. 아래의 예제와 같이 전송할 데이터를 사용자가 원하는 key값을 이용하여 key-value 쌍으로 매개변수로 넘겨받은 transferPack에 넣기만 하면 됩니다.

만일 여기에서 전송할 데이터를 지정하지 않으면 대상 게임 노드로 전송된 유저 객체의 해당 데이터는 모두 기본값으로 초기화 되므로 주의가 필요 합니다.

```java
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("TotalMoney", myTotalMoney);
    transferPack.put("Message", myMessage);
}
```

이제 유저 전송이 완료된 후 대상 게임 노드에서 처리할 콜백 메소드를 살펴보겠습니다. 아래와 같이 전송 전에 지정한 key를 이용해서 원하는 객체에 접근할 수 있습니다. 이와 같은 방법으로 전송 완료된 유저 객체의 해당 데이터를 원래 상태로 만들어 줍니다.


```java
@Override
public void onTransferIn(TransferPack transferPack) throws SuspendExecution {
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setTotalMoney(transferPack.getToLong("TotalMoney"));
    setTransferMessage(transferPack.getToString("Message"));
}
```

위의 두 가지 메소드는 GameAnvil이 알아서 호출합니다. 사용자는 그냥 구현만 하면 됩니다.

<br>

###  3-2.  전송 가능한 유저 타이머

유저가 전송될 때 유저에 등록해둔 타이머도 함께 전송할 수 있습니다. 전송 가능한 타이머를 사용하기 위해서는 별도로 아래의 콜백 메소드에서 원하는 key를 이용해서 미리 등록해두어야 합니다. Timer 핸들러를 다음과 같이 원하는 key로 매핑해둡니다.


```java
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferUserTimerHandler1", transferUserTimerHandler1());
     registerTimerHandler("transferUserTimerHandler2", (timerObject, object) -> {
         GameUser user = (GameUser) object;
         logger.warn("GameUser::transferUserTimerHandler2() : userId({})", user.getId());
     });
 }
```

등록이 완료된 타이머 객체는 언제든 해당 key를 이용해서 아래와 같이 게임 유저 구현부에서 사용할 수 있습니다. 단순히 등록만 한 타이머는 효과가 없으므로 실제 사용을 위해서는 반드시 아래와 같이 유저 객체에 추가해주어야 타이머가 발동합니다. 

```java
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

이렇게 유저 객체에 추가해 둔 타이머는 별도로 전송에 대한 처리를 하지 않아도 모두 자동으로 전송됩니다. 즉, 대상 게임 노드에서 해당 유저 객체에 대해 동일한 타이머 추가 과정을 거칠 필요가 없습니다.

<br>

## 4. 아이디

GameAnvil은 여러 종류의 아이디를 사용합니다. 그 중 일부는 서버가 자체 발급하고 다른 일부는 사용자가 GameAnvilConfig에 직접 설정합니다. 접속에 필요한 계정 정보 등은 클라이언트에서 입력받은 정보를 서버로 전달합니다. 다음은 GameAnvil에서 사용하는 대표적인 아이디에 대한 설명입니다.

| 이름      | 설명                                                         | 자료형 | 범위      |
| --------- | ------------------------------------------------------------ | ------ | --------- |
| ServiceId | - 설정한 각각의 서비스를 구분하기 위한 아이디<br>- 하나의 서비스 아이디는 여러 개의 노드로 구성할 수 있음 | int    | 0<id< 100 |
| HostId    | - 호스트의 고유 아이디<br>- 하나의 호스트에 여러 개의 GameAnvil 프로세스를 구동할 경우에는 GameAnvilConfig에 별도의 vmId를 설정 | long   | -         |
| NodeId    | - 노드의 고유 아이디<br>- HostId + ServiceId + 내부Counter 값으로 구성 | long   | -         |
| AccountId | - 클라이언트가 접속할 때 입력<br>- 접속(Connection) 하나당 하나의 계정 아이디가 매핑 | string | -         |
| UserId    | - 게임 유저 객체의 고유 아이디<br>- 게임 유저 객체가 생성될 때 서버가 발급<br>- 동일한 유저가 재접속 할 경우라도 새로운 게임 유저 객체가 생성되면 새로운 아이디 발급 | int    | -         |
| RoomId    | - 방의 고유 아이디<br>- 방 객체가 생성될 때 서버가 발급      | int    | -         |
| SubId     | - 하나의 계정(accountId) 내에서 고유한 보조 아이디<br>- 세션(Session) 하나당 AccountId + SubId가 매핑<br>- 접속(Connection) 단위로 여러개의 세션(Session)을 사용할 때 하나의 접속 내에서 각각의 세션 유저를 구분하는 아이디<br>- 클라이언트가 접속할 때 입력 | int    | 0 < id    |



### 4-1.아이디 지원 API

앞서 살펴본 ID에 관한 일부 기능을 아래의 표와 같이 엔진 사용자에게 제공합니다. 해당 ID를 획득하거나 체크하기 위해서는 반드시 아래의 API를 사용해야 합니다.

* ServiceId 클래스

| 함수명                                | 설명                                    |
| ------------------------------------- | --------------------------------------- |
| boolean isValid(int serviceId)        | 유효한 serviceId인지 체크               |
| int findServiceId(String serviceName) | ServiceName에 해당하는 ServiceId을 획득 |
| String findServiceName(int serviceId) | ServiceId에 해당하는 ServiceName을 획득 |

* HostId 클래스

| 함수명                       | 설명                     |
| ---------------------------- | ------------------------ |
| boolean isValid(long hostId) | 유효한 hostId인지 체크   |
| long get()                   | 프로세스의 hostId를 획득 |

* NodeId 클래스

| 함수명                        | 이름                          |
| ----------------------------- | ----------------------------- |
| boolean isValid(long nodeId)  | 유효한 nodeId인지 체크        |
| long getHostId(long nodeId)   | nodeId로부터 hostId를 획득    |
| int getServiceId(long nodeId) | nodeId로부터 serviceId를 획득 |
| int getNodeNum(long nodeId)   | nodeId로부터 nodeNum을 획득   |

* UserId 클래스

| 함수명                      | 설명                   |
| --------------------------- | ---------------------- |
| boolean isValid(int userId) | 유효한 userId인지 체크 |

* RoomId 클래스

| 함수명                      | 설명                   |
| --------------------------- | ---------------------- |
| boolean isValid(int roomId) | 유효한 roomId인지 체크 |

* SubId 클래스

| 함수명                     | 설명                  |
| -------------------------- | --------------------- |
| boolean isValid(int subId) | 유효한 subId인지 체크 |

<br>

## 5. Channel

채널은 단일 서버군을 논리적으로 나눌 수 있는 방법 중 하나입니다. GameAnvil은 한 개 이상의 Game 노드를 포함할 경우에 채널을 설정할 수 있습니다. 기본적으로 GameAnvilConfig을 통해 Game 노드에 아래의 예제처럼 채널을 설정합니다. 이 예제에서 총 4개의 Game노드에 대해 각각 ch1,ch1,ch2,ch2를 설정합니다.

```json
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



이 때, 동일한 채널의 Game 노드는 서로 채널 관련 정보를 공유합니다. 예를 들면 채널에 새로운 유저가 진입하거나 떠날 때 GameAnvil 엔진이 미리 구현한 콜백을 호출해줍니다. 채널간 유저와 방 정보 동기화를 위해 다음과 같이 GameUserInfo 그리고 GameRoomInfo를 사용합니다. 앞서 살펴본 GameNode에 대한 Bootstrap의 내용에 추가로 SampleChannelUserInfo와 SampleChannelRoomInfo가 등록되는 것을 확인할 수 있습니다.

```java

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

```java
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
     * @param inputStream 직렬화된 스트림을 전달.
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



채널 방 정보도 동일한 방법으로 구현합니다. GameAnvil이 제공하는 RoomInfo 인터페이스와 Serializable을 구현합니다. 인터페이스명이 ChannelRoomInfo가 아닌 RoomInfo임에 주의하세요. 이 클래스는 채널 유저 정보와 마찬가지로 엔진 사용자가 채널간 동기화에 사용할 정보를 바탕으로 구현하면 됩니다.

```java
public class SampleChannelRoomInfo implements Serializable, RoomInfo {

    public static final int MAX_ENTRY_USER = 4;
    private int roomId = 0;

    private int userCnt;
    private int gameState;
    private long createTime;
    
    public SampleChannelRoomInfo() {
    }
    
	//... 컨텐츠에서 필요한 정보로 클래스 구현

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
     * 전달 받은 정보를 {@link KryoSerializer} 이용해서 deserialize 처리를 한다.
     *
     * @param inputStream deserialize 할 데이터를 전달.
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



앞서 살펴본 채널의 유저,방 정보는 실제 해당 유저나 방에 대한 갱신이 필요할 때마다 동일한 채널 내의 GameNode 사이에 전파됩니다. 이 때, 관련 GameNode는 아래의 두 가지 콜백 메소드를 각각 채널 유저 정보와 채널 방 정보 갱신에 대해 호출합니다.

```java
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
    public void onChannelUserUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {        
    }
```

파라메터로 전달받은 채널 유저,방 정보를 바탕으로 엔진 사용자가 원하는 방식으로 구현하면 됩니다.
```java
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
    public void onChannelRoomUpdate(ChannelUpdateType type, RoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {        
    }
```
마지막으로 클라이언트는 서버로 채널 정보를 요청할 수 있습니다. 이 때, 아래의 콜백 메소드가 호출됩니다. 이 또한 엔진 사용자가 원하는 방식으로 적절하게 구현합니다.
```java
    /**
     * Client 에서 Channel 정보를 요청 시 올라오는 callback. (Base.GetChannelInfoReq)
     *
     * @param outPayload Client 로 전달될 Channel 정보 전달.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {        
    }
```

<br>

## 6. MatchingGroup

매칭 그룹도 채널과 마찬가지로 단일 서버군을 논리적으로 나눌 수 있는 방법 중 하나입니다. 단, 매칭그룹은 채널과 달리 명시적으로 미리 설정하지 않습니다. 또한 채널은 GameNode를 논리적으로 나누기 위한 방법인 반면에 매칭 그룹은 매치메이킹을 논리적으로 나누기 위한 방법입니다. 앞서 살펴본 유저 매치메이킹 콜백 메소드를 다시 한번 살펴보겠습니다. 이 예제에서 설정한 매칭그룹은 "Newbie"입니다. 이 "Newbie"라는 값은 클라이언트에서 서버로 payload를 통해 전달할 수도 있고 엔진 사용자가 서버 상에 해당 유저에 대해 미리 저장해둔 값일 수도 있습니다. 아래와 같이 하드코딩한 값을 쓸 수도 있으나 실제 게임 서버 코드에는 바람직하지 않습니다. 어쨌든  "Newbie"라는 매칭그룹에 대해 하나의 매치메이커가 생성되고 이 후의 모든 "Newbie" 요청은 이 매치메이커에 쌓여서 함께 처리되는 것이 보장됩니다.

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

매치 메이킹 관련한 모든 프로토콜에는 매칭 그룹 필드가 존재합니다. 그러므로 클라이언트와 서버 사이에 미리 매칭그룹을 정의해두고 사용하는 것이 가장 이상적입니다. 예를 들어, "초보", "중수", "고수" 처럼 실력을 기반으로 한 매칭 그룹을 정의할 수도 있고 "한국", "일본", "미국" 처럼 국가별 매칭 그룹을 정의할 수도 있습니다. 이는 어디까지나 엔진 사용자가 원하는대로 사용할 수 있는 부분입니다.