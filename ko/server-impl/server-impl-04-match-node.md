## Game > GameAnvil > 서버 개발 가이드 > 매치 노드 구현

## MatchNode와 MatchMaker

![MatchNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_matchnode_on_network.png)

사용자는 간단하게 매칭 로직만 구현하고 MatchNode를 활성화해 매치 메이킹을 적용할 수 있습니다. 매치 메이커는 매칭 로직에 기반하여 유저들을 임의의 방으로 입장시켜 줍니다. GameAnvil은 두 가지 매치
메이커를 제공합니다. 각각 유저 단위의 매칭을 수행하는 UserMatchMaker와 방 단위의 매칭을 수행하는 RoomMatchMaker입니다.

이러한 매치 메이커는 MatchNode에서 독립적으로 구동됩니다. 이때, MatchNode는 매치 메이킹을 수행하는 용도 외에 추가적인 콘텐츠를 구현할 수 없습니다. 그러므로 사용자는 MatchNode가 아닌 매치
메이커에만 집중하면 됩니다. 단, 매치 메이커를 수행하기 위해서는 최소 한 개 이상의 MatchNode를 활성화해야 합니다.

> [참고]
>
> 매치 메이커는 매칭 그룹 단위로 생성됩니다. 즉, 동일한 매칭 그룹끼리 매치 메이킹을 수행합니다.
>
> MatchNode는 필수 노드가 아니므로 매치 메이킹을 사용하지 않을 경우에는 구동할 필요가 없습니다.

## 매칭 그룹(MatchingGroup)

매칭 그룹도 채널과 마찬가지로 단일 서버군을 논리적으로 나눌 수 있는 방법 중 하나입니다. 단, 매칭 그룹은 채널과 달리 명확하게 미리 설정해 두고 사용하는 값이 아닙니다. 또한 채널은 GameNode를 논리적으로
나누기 위한 방법인 반면에 매칭 그룹은 매치 메이킹을 논리적으로 나누기 위한 방법입니다. 같은 매칭 그룹에서 유저 매칭 또는 룸 매칭을 요청한 경우 매칭을 요청한 채널 내에서 매칭된 룸이 생성됩니다.

앞서 게임 유저에서 살펴본 유저 매치 메이킹 콜백 메서드인 onMatchUser를 다시 한번 살펴보겠습니다. 콜백 메서드의 문자열 인자 matchingGroup은 클라이언트가 유저 매치 메이킹을 요청할 때 함께
전달한 값입니다. 즉, 매칭 그룹은 클라이언트와 서버 사이에서 사전에 약속된 임의의 문자열입니다. 그 종류나 개수는 사용자가 원하는 만큼 사용할 수 있습니다. 모든 매치 메이킹 요청은 각각의 매칭 그룹 단위로 큐가
관리되며, 해당 큐 안에서만 서로 매칭이 가능합니다. 예를 들어 아래의 예제에서 인자 matchingGroup으로 "초보"라는 값이 전달될 경우 이 요청은 다른 "초보" 요청들과 매칭되는 것이 보장됩니다.

```java
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
public boolean onMatchUser(final String roomType, final String matchingGroup, final IPayload payload, IPayload outPayload) {

    SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId(), rating);
    
    return matchUser(matchingGroup, roomType, userMatchInfo, payload);
}
```

매칭 그룹은 "초보", "중수", "고수"처럼 실력을 기반으로 정의할 수도 있고 "한국", "일본", "미국"처럼 국가별로 정의할 수도 있습니다. 즉, 사용자가 원하는 어떤 값도 매칭그룹이 될 수 있습니다.

## 유저 매치 메이커 구현

유저 매치 메이킹은 게임 유저들의 매칭 요청을 큐에 적재합니다. 특정 시간 주기로 이 요청 큐의 내용을 비교, 분석하여 사용자가 원하는 기준으로 임의의 유저들을 하나의 방으로 입장시켜 줍니다. 여기에서 사용자는 요청
큐의 내용을 어떻게 비교하고 분석해서 어떤 기준으로 유저들을 매칭할지에 대한 로직에만 집중하면 됩니다. 참고로 가장 대표적인 유저 매치 메이킹 게임은 <리그 오브 레전드>가 있습니다.

### 유저 매치 요청 구현

이러한 유저 매치 메이킹의 가장 기본은 바로 매칭 요청 그 자체입니다. 이러한 매치 요청을 아래와 같이 엔진에서 제공하는 AbstractUserMatchInfo 추상 클래스를 상속하여 구현합니다. 이때, 요청자를 구분할 수
있는 게임 유저의 ID를 제공할 수 있도록 getId() 메서드는 반드시 구현해야 합니다. 또한 요청은 언제든 직렬화할 수 있어야 하므로 Serializable 인터페이스를 구현해야 합니다. 아래의 예제는 매칭 요청
사이의 비교를 위해 Comparable 인터페이스를 추가로 구현하고 있습니다.

```java
public class SampleUserMatchInfo extends AbstractUserMatchInfo implements Comparable<SampleUserMatchInfo> {
    private int userId;
    private int rating;

    public SampleUserMatchInfo(int userId, int rating) {
        this.userId = userId;
        this.rating = rating;
    }

    public int getRating() {
        return rating;
    }

    /**
     * 요청 주체의 아이디 반환
     * <p/>
     * 파티 매치메이킹 요청인 경우 룸 아이디, 유저 매치메이킹 요청인경우 유저 아이디를 반환
     *
     * @return 파티 매치메이킹 요청인경우 룸 아이디, 유저 매치메이킹 요청인경우 유저 아이디를 반환
     */
    @Override
    public int getId() {
        return userId;
    }

    /**
     * 파티 크기 반환
     * <p/>
     * 파티 매치메이킹 요청인경우 파티의 크기(인원수), 유저 매치메이킹 요청인경우 0을 반환
     *
     * @return 파티 매치메이킹 요청인경우 파티의 크기(인원수), 유저 매치메이킹 요청인경우 0 반환
     */
    @Override
    public int getPartySize() {
        return 0;
    }

    // 만일 SampleUserMatchInfo 객체 사이에 비교가 필요하다면 Comparable 인터페이스를 구현합니다.
    @Override
    public int compareTo(UserMatchInfo o) {
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
}
```

유저 매치 요청에서 재정의해야 할 메서드를 표로 정리하면 다음과 같습니다.

| 콜백 이름        | 의미          | 설명                                                                                                                             |
|--------------|-------------|--------------------------------------------------------------------------------------------------------------------------------|
| getId        | 매치 요청자 정보   | 해당 유저 매치 요청이 어느 유저로부터 온 것인지 판단하기 위해 사용됩니다. 그러므로 반드시 요청자의 ID를 반환하도록 재정의해 주어야 합니다.                                               |
| getPartySize | 매치 요청 파티 규모 | 요청 파티의 크기를 반환합니다. 이 값으로 파티 매치 메이킹 요청 여부를 판단하므로 유저 매치 메이킹 요청인 경우에는 반드시 0을 반환하도록 재정의합니다. 파티 매치 요청의 경우에는 해당 파티원의 인원 수를 반환하도록 합니다. |

### 유저 매치 메이커

유저 매치 메이커는 유저 매치 요청을 실제로 처리하며, 엔진에서 제공하는 AbstractUserMatchMaker 추상 클래스를 상속 구현합니다. 특히 onMatch() 메서드는 실제 매칭을 수행하기 위해 호출되는
콜백이므로 주의 깊게 살펴보십시오. onRefill() 메서드는 이미 완료된 매치 메이킹에 대해 충원 요청을 처리하는 콜백입니다. 예를 들어 4명이 매치 메이킹된 상태에서 1명이 게임을 종료했을 때 1명을 더
충원하기 위해 사용할 수 있습니다. 아래의 예제 코드는 이러한 유저 매치 메이커를 구현하는 방법을 보여줍니다.
```java
public class SampleUserMatchMaker extends AbstractUserMatchMaker<SampleUserMatchInfo> {

    public SampleUserMatchMaker() {
        super(2, 5000);
    }

    private int matchSize = 2;

    /**
     * 유저/파티 매치메이킹 요청을 처리
     * <p>
     * {@link IUser#onMatchUser(String, String, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)}에서 {@link IUserContext#matchUser(String, String, AbstractUserMatchInfo)}를 호출하거나
     * {@link IRoom#onMatchParty(String, String, IUser, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)}에서 {@link IRoomContext#matchParty(String, String, AbstractUserMatchInfo)}를 호출하면 매치메이킹을 처리할 수 있다
     * <p>
     * 첫번째 호출 이후 주기적으로 호출된다
     * <p>
     * {@link #getMatchRequests(int)}를 이용해 매치를 요청한 유저/파티 의 UserMatchInfo 목록을 가져온다
     * <p>
     * 이 목록을 이용해 조건에 맞는 요청을 모아 {@link #matchSingles(Collection)}, {@link #assignRoom(AbstractUserMatchInfo)}, {@link #assignRoom(Collection)}을 이용해 같은 룸으로 넣어준다
     * <p>
     * 조건에 맞는 요청의 수가 모자랄 경우 다음 호출시에 처리
     */
    @Override
    public void onMatch() {
        List<SampleUserMatchInfo> matchRequests = getMatchRequests(matchSize);

        if (matchRequests == null) {
            return;
        }

        int matchingAmount = matchSingles(matchRequests);
    }

    /**
     * 유저/파티 매칭으로 만들어진 룸에서 유저가 룸을 나갔을 경우 새로운 유저를 할당
     * <p>
     * {@link IRoom#canLeaveRoom(IUser, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)} 에서 {@link IRoomContext#matchRefill(AbstractUserMatchInfo)} 을 호출하면 호출되어 새 유저를 채울수 있다
     * <p>
     * 유저/파티 매치메이킹 요청이 있을 경우 먼저 한번 {@link #onRefill(AbstractUserMatchInfo)}가 호출되어 리필이 필요한 룸으로부터 유저를 채워 넣을 수 있다
     * <p>
     * 유저/파티 매치메이킹 요청마다 처음 한번만 {@link #onRefill(AbstractUserMatchInfo)}가 호출되고 주기적으로 호출되지 않는다
     * <p>
     * 리필로 사용되지 않은 요청은 {@link #getMatchRequests(int)}의 목록에 남아 {@link #onMatch()}에서 계속 사용된다
     * <p>
     * {@link #getRefillRequests()} 를 이용해 리필을 요청한 룸의 UserMatchInfo 목록을 가져온다
     * <p>
     * 이 목록에서 조건 req 에 맞는 요청을 찾아 {@link #refillRoom(AbstractUserMatchInfo, AbstractUserMatchInfo)}를 이용해 방으로 넣어준다
     *
     * @param req 유저/파티 매치를 요청한 유저 또는 파티의 UserMatchInfo
     * @return 반환값이 true 이면 리필 성공, false 이면 실패
     */
    @Override
    public boolean onRefill(SampleUserMatchInfo matchReq) {
        return false;
    }
}
```

특히 매치 메이커를 구현한 클래스는 특정 서비스에 엔진에 등록합니다. 또한 매치 메이커에 의해 생성될 방의 타입을 미리
정의합니다. 이때, 하나의 매치 메이커 클래스는 오직 하나의 서비스에 대해서만 등록할 수 있습니다.

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
            ...
        });

        // "BasicRoom"라는 룸 타입의 유저를 엔진에 등록
        gameServiceBuilder.room("BasicRoom", SampleGameRoom::new, config -> {
            ...
            
            // 유저 매치 메이커 등록
            config.matchMaker(SampleUserMatchMaker::new);

            ...
        });

        GameAnvilServer.getInstance().run();
    }
}
```

이러한 유저 매치 메이커의 콜백 메서드를 정리하면 다음의 표와 같습니다.

| 콜백 이름    | 의미          | 설명                                                                                                                                                                                                                                                                                                                                                                                        |
|----------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onMatch  | 매치 요청들을 처리  | 사용자는 AbstractUserMatchMaker가 제공하는 API를 사용하여 직접 매치 요청을 처리할 수 있습니다. 이때, 사용자가 원하는 대로 로직을 직접 구현할 수 있습니다. 예제 코드의 처리 흐름이 가장 기본적인 방식입니다. <br> 즉, getMatchRequests API를 이용하여 최소한의 매치 요청들을 획득한 후 사용자가 원하는 대로 인원수에 맞춰 요청들을 조합한 후 임의의 Collection에 순서대로 담습니다. 이 Collection을 matchSingles API에 인자로 전달하면 정원수에 맞춰 매치가 이루어집니다. 예제 코드의 경우 정원이 2명인 유저 매치 메이킹이므로 Collection을 순회하며 순서대로 2명씩 추출하여 하나의 게임으로 매칭시킵니다. |
| onRefill | 매치 리필 요청 처리 | 유저/파티 매치 메이킹 과정에서 임의의 유저가 나간 경우 새로운 유저를 채우기 위한 처리를 합니다. 일반적으로 매치 메이킹된 방에 대해 onLeaveRoom이 호출될 때 matchRefill을 호출하여 연동할 수 있습니다. 즉, 매칭된 방에서 누군가 나갈 때 리필을 요청하는 것입니다. 리필은 큐에 쌓여 있는 매치 요청은 사용하지 않습니다. 리필 요청 이후에 들어오는 새로운 매치 요청만 그 대상으로 합니다.                                                                                                                                                      |

### GameUser에서 매치 메이커로 요청 전달하기

이제 클라이언트는 서버로 유저 매치 메이킹을 요청할 수 있습니다. 이 요청은 GameUser에 전달된 후 엔진에 의해 onMatchUser 콜백 메서드를 호출합니다. 이에 대해서는 앞서 GameNode와
GameUser를 설명하면서 한 번 살펴보았습니다. 사용자는 이 콜백 메서드에서 GameAnvil이 제공하는 유저 매치 메이커를 사용해도 되고 직접 구현한 별도의 매치 메이커나 다른 솔루션을 사용해도 됩니다. 하지만
특별한 이유가 없다면 GameAnvil의 유저 매치 메이킹을 사용하길 권장합니다.

아래의 예제는 엔진에서 제공하는 유저 매치 메이커를 사용하기 위해 앞서 살펴본 매치 요청을 생성하여 matchUser API를 호출하는 모습입니다. 특히, 매개 변수로 전달된 매칭 그룹은 매치 메이킹을 논리적으로
나눌 수 있는 개념이라고 앞서 설명했습니다. 만일 "Newbie"라는 매칭 그룹을 전달했다면 동일한 "Newbie" 매칭 그룹끼리 같은 매칭 큐를 공유하게 됩니다.

```java
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
public boolean onMatchUser(final String roomType, final String matchingGroup, final IPayload payload, IPayload outPayload) {

    SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId(), rating);

    return matchUser(matchingGroup, roomType, userMatchInfo, payload);
}
```

마지막으로 클라이언트는 언제든 앞서 요청한 유저 매치 메이킹을 취소할 수 있습니다. 이때, 엔진은 취소 처리가 성공적으로 진행되면 다음과 같이 GameUser의 onMatchUserCancel 콜백 메서드를
호출합니다. 사용자는 이 콜백에서 취소 타이밍에 처리하고 싶은 부분을 구현하면 됩니다.

```java
/**
 * 유저 매치메이킹이 취소될 때 호출
 *
 * @param reason 취소된 이유. 타임아웃(TIMEOUT), 사용자의 요청에 의한 취소(CANCEL), 매치 노드의 종료에 의한 취소(SHUTDOWN)
 */
@Override
public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
    
}
```

## 룸 매치 메이커 구현

룸 매치 메이킹은 유저를 가장 적합한 방으로 자동 입장시켜 주는 기능입니다. 룸 매치 메이킹을 요청한 유저를 어떤 기준으로 어느 방에 입장시킬지는 사용자가 구현하기에 달렸습니다. 가장 유저 수가 많은 방으로 입장시킬
수도 있고, 가장 한산한 방으로 입장시킬 수도 있습니다. 혹은 평균 점수가 가장 높은 방으로 입장시킬 수도 있습니다. 사용자는 이러한 매칭 로직에만 집중하면 됩니다. 참고로 가장 대표적인 룸 매치 메이킹 게임은 <
한게임 포커>나 <카트라이더> 등이 있습니다.


> [참고]
>
> 매치 메이커는 매칭 그룹 단위로 생성됩니다. 즉, 동일한 매칭 그룹끼리 매치 메이킹을 수행합니다.
>
> 룸 매치 메이커와 유저 매치 메이커는 서로 독립적으로 운영됩니다. 즉, 동일한 매칭 그룹으로 유저 매칭과 룸 매칭을 각각 요청하더라도 이 두 요청이 함께 매칭되는 일은 없습니다.

### 룸 매치 요청 구현

이러한 룸 매치 메이킹의 가장 기본은 바로 매칭 요청 그 자체입니다. 매칭 요청은 한 명의 유저가 보낸 요청을 의미하며 아래와 같이 엔진에서 제공하는 AbstractRoomMatchForm 추상 클래스를 상속 구현합니다.
요청은 언제든 직렬화할 수 있어야 하므로 Serializable 인터페이스를 추가로 구현해야 합니다. 다음은 이러한 매치 요청을 구현한 예제입니다.

```java
public class SampleRoomMatchForm extends AbstractRoomMatchForm {
    public SampleRoomMatchForm() {
        super();
    }
}
```

룸 매치 요청은 기본적으로 매칭 로직에서 사용할 정보를 포함합니다. 이는 사용자가 매치 메이킹 로직을 직접 구현할 때 사용하게 됩니다. 룸 매치 요청에서 한 가지 중요한 정보는 매칭 유저 카테고리입니다. 매칭 유저
카테고리는 하나의 방에서 유저들이 속한 그룹을 구분하기 위한 임의의 문자열입니다. 예를 들어 4인 정원인 방에서 두 개의 팀으로 나눠서 2 vs 2의 게임을 할 경우, 각각의 유저가 속할 팀을 지정하기 위한 용도로
사용될 수 있습니다. 만일 아무런 값도 지정하지 않으면 엔진의 기본 값이 사용됩니다.

### 룸 매치 정보 구현

매칭 대상이 되는 방은 룸 매치 메이커에 의해 매치 정보가 관리됩니다. 즉, 하나의 룸 매치 정보는 하나의 매칭 가능한 방 정보를 의미한다고 생각할 수 있습니다. 이때, 해당 방의 여러 가지 정보와 상태값들을 포함할
수 있습니다. BaseRoomMatchInfo를 상속 구현하며 필수로 방의 ID와 매칭 유저 카테고리 그리고 각 매칭 카테고리별 최대 정원 수를 설정해야 합니다. 그리고 마지막으로 룸 매치 정보도 직렬화를 위해
반드시 Serializable 인터페이스를 구현해야 합니다. 아래는 예제 코드를 보여줍니다.

```java
public class SampleRoomMatchInfo extends AbstractRoomMatchInfo {
    private static final int MAX_ENTRY_USER = 4;

    public SampleRoomMatchInfo(int roomId) {
        super(roomId, MAX_ENTRY_USER);
    }
}
```

아래의 예제는 사용자가 임의의 매칭 유저 카테고리를 설정하는 경우입니다. 하나의 게임 방에 레드팀과 블루팀이 각각 3명씩으로 구성되어 있습니다. 이와 같이 카테고리를 나누어 둘 경우 룸 매치 메이커가 각각의 매칭
카테고리별 유저 수를 알아서 관리해 줍니다. 또한 매치 메이킹에 사용할 추가 정보를 원하는 대로 구성할 수 있습니다.

```java
public class GameRoomMatchInfo extends AbstractRoomMatchInfo {
    private static final int MAX_USER = 3;

    // 매칭 조건 정보
    int mapId;
    int maxLevel;
    int avgLevel = 0;

    public GameRoomMatchInfo(int roomId) {
        // RED_TEAM 최대 인원 3명으로 인원 정보를 추가합니다.
        super(roomId, "RED_TEAM", 3);

        // 필요할 경우 추가 매치 카테고리를 등록합니다.
        // BLUE_TEAM 최대 인원 3명으로 인원 정보를 추가합니다.
        addMatchingUserCategory("BLUE_TEAM", 3);
    }

    public void setAvgLevel(int avgLevel) {
        this.avgLevel = avgLevel;
    }
    
    ...
}
```

### 룸 매치 정보 등록/갱신

이러한 룸 매치 정보는 사용자가 직접 등록/갱신할 수 있습니다. 즉, 사용자가 원하지 않을 경우에는 특정 방은 룸 매치 메이킹 대상으로 등록하지 않을 수도 있습니다. 이런 등록 절차는 일반적으로 다음과 같이 방이
생성되는 onCreateRoom 콜백 메서드에서 주로 진행합니다.

```java
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
public final boolean onCreateRoom(final GameUser user, final IPayload payload, IPayload outPayload) {
    
    ...

    // 이제부터 이 방은 룸 매치 메이킹에 사용됩니다.
    registerRoomMatch(gameRoomMatchInfo, user.getUserId());
}
```

임의의 방에 대해 룸 매치 메이킹 등록 여부를 확인하기 위해 IRoomContext 인터페이스에 다음과 같은 API를 제공합니다.

```java
/**
 * 룸 매치 등록  여부 반환
 *
 * @return 반환값이 true 이면 룸 매치 등록, false 이면 미등록
 */
public boolean isRegisteredRoomMatch()
```

이렇게 등록해 둔 룸 매치 정보는 언제든 필요에 따라 사용자가 그 정보를 갱신할 수도 있습니다. 갱신을 위해 새로운 룸 매치 정보를 만든 후에 updateRoomMatch API를 호출하면 됩니다.

```java
GameRoomMatchInfo gameRoomMatchInfo = new GameRoomMatchInfo(getId());
gameRoomMatchInfo.setMemberMoney(1000);

roomContext.updateRoomMatch(gameRoomMatchInfo); // 이 룸 매치 정보를 갱신합니다.
```

해당 방이 사라질 때 룸 매치 정보는 엔진에서 자동으로 삭제되므로 사용자가 따로 삭제할 필요가 없습니다.

### 룸 매치 메이커

이제 룸 매치 메이커를 만들 차례입니다. 룸 매치 메이커는 엔진에서 제공하는 AbstractRoomMatchMaker 추상 클래스를 상속 구현 합니다. 룸 매치 메이킹은 가장 적합한 방을 찾는 과정이므로 실제 매칭 전/후를
위한 특별한 콜백 메서드들이 제공됩니다. 사용자는 이 콜백 메서드들을 재정의하여 원하는 대로 매칭을 수행할 수 있습니다. 아래의 예제 코드는 이러한 룸 매치 메이커를 어떤 식으로 구현할 수 있는지 보여줍니다.

```java
public class SampleRoomMatchMaker extends AbstractRoomMatchMaker<SampleRoomMatchForm, SampleRoomMatchInfo> {
    /**
     * 룸 매치메이킹 요청시 호출
     * <p>
     * 등록된 룸들 중에 매칭 조건과 맞는 룸을 찾아 매치 성공 여부를 결정한다
     *
     * @param baseRoomMatchForm 룸 매치메이킹 신청 조건인 {@link AbstractRoomMatchForm}
     * @param baseRoomMatchInfo 매칭풀의 룸 정보인 {@link AbstractRoomMatchInfo}
     * @param args              추가로 전달 받은 파라미터
     * @return 반환값이 true 이면 룸 매치메이킹 성공, false 이면 룸 매치메이킹 실패
     */
    @Override
    public boolean onMatch(SampleRoomMatchForm roomMatchForm, SampleRoomMatchInfo roomMatchInfo, Object... args) {
        return false;
    }

    /**
     * 정렬을 위한 매치메이킹 정보 비교
     * <p>
     * {@link com.nhn.gameanvil.node.game.context.IRoomContext#updateChannelRoomInfo(IChannelRoomInfo)} 메서드 호출, 룸 등록, 룸 삭제, 정렬 메서드 호출 시 구현한 compare 로 인해 매칭풀이 정렬된다
     *
     * @param o1 비교할 첫번째 파라미터
     * @param o2 비교할 두번째 파라미터
     * @return 비교 값 반환 (-1: 오름차순, 0: 변동없음, 1: 내림차순)
     */
    @Override
    public int compare(SampleRoomMatchInfo o1, SampleRoomMatchInfo o2) {
        return 0;
    }
}
```

특히 매치 메이커를 구현한 클래스는 특정 서비스의 엔진에 등록합니다. 또한 매치 메이커에 의해 생성될 방의 타입을 미리
정의합니다. 이때, 하나의 매치 메이커 클래스는 오직 하나의 서비스에 대해서만 등록할 수 있습니다.

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
            ...
        });

        // "BasicRoom"라는 룸 타입의 유저를 엔진에 등록
        gameServiceBuilder.room("BasicRoom", SampleGameRoom::new, config -> {
            ...
            
            // 유저 매치 메이커 등록
            config.matchMaker(SampleRoomMatchMaker::new);
            
            ...
        });

        GameAnvilServer.getInstance().run();
    }
}
```

앞서 살펴본 룸 매치 메이커의 콜백 메서드를 정리하면 아래의 표와 같습니다.

| 콜백 이름       | 의미                   | 설명                                                                                                                              |
|-------------|----------------------|---------------------------------------------------------------------------------------------------------------------------------|
| onMatch     | 매치 확인                | 인자로 전달 받은 룸 매치 요청과 룸 매치 정보 사이에 매칭이 가능한지 확인하기 위해 호출합니다. 즉,  전체 룸 매치 풀을 순회하면서 해당 룸 매치 요청이 들어갈 가장 적합한 방을 찾기 위한 로직을 여기에 구현할 수 있습니다. |
| compare     | 정렬을 위한 매치메이킹 정보 비교   | 정렬을 위한 매치메이킹 정보 비교합니다. 결과 값은 -1: 오름차순, 0: 변동없음, 1: 내림차순 입니다.                                                                    |

### GameUser에서 매치 메이커로 요청 전달하기

이제 클라이언트는 서버로 룸 매치 메이킹을 요청할 수 있습니다. 이 요청은 GameUser에 전달된 후 엔진에 의해 onMatchRoom 콜백 메서드를 호출합니다. 이에 대해서는 앞서 GameNode와
GameUser를 설명하면서 한 번 살펴보았습니다. 사용자는 이 콜백 메서드에서 GameAnvil이 제공하는 룸 매치 메이커를 사용해도 되고 직접 구현한 별도의 매치 메이커나 다른 솔루션을 사용해도 됩니다. 하지만
특별한 이유가 없다면 GameAnvil의 룸 매치 메이킹을 사용하길 권장합니다.

아래의 예제는 엔진에서 제공하는 룸 매치 메이커를 사용하기 위해 앞서 살펴본 매치 요청을 생성하여 matchRoom API를 호출하는 모습입니다. 특히, 매개 변수로 전달된 매칭 그룹은 매치 메이킹을 논리적으로 나눌
수 있는 개념이라고 앞서 설명했습니다. 만일 "Newbie"라는 매칭 그룹을 전달했다면 동일한 "Newbie" 매칭 그룹끼리 같은 매칭 큐를 공유하게 됩니다.

```java
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
public RoomMatchResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final IPayload payload) {

    SampleRoomMatchForm sampleRoomMatchForm = new SampleRoomMatchForm(RoomMode.NORMAL, innerPayload.getOption(), 0);

    return userContext.matchRoom(matchingGroup, roomType, sampleRoomMatchForm);
}
```

