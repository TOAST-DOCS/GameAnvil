## Game > GameAnvil > 서버 개발 가이드 > 매치 노드 구현



## 1. MatchNode와 MatchMaker



![MatchNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_matchnode_on_network.png)


사용자는 간단하게 매칭 로직만 구현한 후 MatchNode를 활성하여 매치메이킹을 적용할 수 있습니다. 매치메이커는 매칭 로직에 기반하여 유저들을 임의의 방으로 입장시켜줍니다. GameAnvil은 두 가지 매치메이커를 제공합니다. 각각 유저 단위의 매칭을 수행하는  UserMatchMaker와 방 단위의 매칭을 수행하는 RoomMatchMaker입니다.

이러한 매치메이커는 MatchNode에서 독립적으로 구동됩니다. 이 때, MatchNode는 매치메이킹을 수행하는 용도 외에 추가적인 콘텐츠를 구현할 수 없습니다. 그러므로 사용자는 MatchNode가 아닌 매치메이커에만 집중하면 됩니다. 단, 매치메이커를 수행하기 위해서는 최소 한 개 이상의 MatchNode를 활성화해야 합니다.

### Note

* *매치메이커는 매칭 그룹 단위로 생성됩니다. 즉, 동일한 매칭 그룹끼리 매치메이킹을 수행합니다.*
* *MatchNode는 필수 노드가 아니므로, 매치메이킹을 사용하지 않을 경우에는 구동할 필요가 없습니다.*



## 2. 매칭그룹 (MatchingGroup)

매칭 그룹도 채널과 마찬가지로 단일 서버군을 논리적으로 나눌 수 있는 방법 중 하나입니다. 단, 매칭 그룹은 채널과 달리 명확하게 미리 설정해두고 사용하는 값이 아닙니다. 또한 채널은 GameNode를 논리적으로 나누기 위한 방법인 반면에 매칭 그룹은 매치메이킹을 논리적으로 나누기 위한 방법입니다. 앞서 게임 유저에서 살펴본 유저 매치메이킹 콜백 메서드인 onMatchUser를 다시 한번 살펴보겠습니다. 

콜백 메서드의 문자열 인자 matchingGroup은 클라이언트가 유저 매치메이킹을 요청할 때 함께 전달한 값입니다. 즉, 매칭그룹은 클라이언트와 서버 사이에서 사전에 약속된 임의의 문자열입니다. 그 종류나 개수는 사용자가 원하는 만큼 사용할 수 있습니다. 모든 매치메이킹 요청은 각각의 매칭그룹 단위로 큐가 관리되며, 해당 큐 안에서만 서로 매칭이 가능합니다. 예를 들어 아래의 예제에서 인자 matchingGroup으로 "초보"라는 값이 전달될 경우, 이 요청은 다른 "초보" 요청들과 매칭되는 것이 보장됩니다.

```java
    /**
     * 클라이언트에서 userMatch를 요청했을 경우 호출되는 콜백
     *
     * @param roomType   클라이언트와 서버 사이에 사전 정의한 방 종류를 구분하는 임의의 값
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 유저 매치메이킹 요청 성공이고 false 실패
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    @Override
    public final boolean onMatchUser(final String roomType, final String matchingGroup, final Payload payload, Payload outPayload) throws SuspendExecution {

        UserMatchInfo userMatchInfo = new UserMatchInfo(getUserId());
        userMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, userMatchInfo, payload);
    }
```

매칭그룹은 "초보", "중수", "고수"처럼 실력을 기반으로 정의할 수도 있고 "한국", "일본", "미국"처럼 국가별로 정의할 수도 있습니다. 즉, 사용자가 원하는 어떤 값도 매칭그룹이 될 수 있습니다.



## 3. 유저 매치메이커 구현

유저 매치메이킹은 게임 유저들의 매칭 요청을 큐에 적재합니다. 특정 시간 주기로 이 요청 큐의 내용을 비교, 분석하여 사용자가 원하는 기준으로 임의의 유저들을 하나의 방으로 입장시켜줍니다. 여기에서 사용자는 요청 큐의 내용을 어떻게 비교하고 분석해서 어떤 기준으로 유저들을 매칭할지에 대한 로직에만 집중하면 됩니다. 참고로 가장 대표적인 유저 매치메이킹 게임은 "*리그 오브 레전드*"가 있습니다.



### 3.1. 유저 매치 요청 구현

이러한 유저 매치메이킹의 가장 기본은 바로 매칭 요청 그 자체입니다. 이러한 매치 요청을 아래와 같이 엔진에서 제공하는 BaseUserMatchInfo 추상 클래스를 상속하여 구현합니다.  이 때, 요청자를 구분할 수 있는 게임 유저의 ID를 제공할 수 있도록 getId() 메서드는 반드시 구현해야 합니다. 또한 요청은 언제든 직렬화할 수 있어야 하므로 Serializable 인터페이스를 구현해야 합니다. 아래의 예제는 매칭 요청 사이의 비교를 위해 Comparable 인터페이스를 추가로 구현하고 있습니다.

```java
public class UserMatchInfo extends BaseUserMatchInfo implements Serializable, Comparable<UserMatchInfo> {

    private int userId;
    private int rating;

    public UserMatchInfo(int userId, int rating) {
        this.userId = userId;
        this.rating = rating;
    }

    public int getRating() {
        return rating;
    }

    /**
     * 해당 유저 매치 요청이 어느 유저로부터 온 것인지 판단하기 위해 사용됩니다.
     *
     * @return 파티매칭 요청인경우 RoomId를 유저매칭 요청인경우 UserId 를 반환
     */
    @Override
    public int getId() {
        return userId;
    }

    /**
     * 요청 파티의 크기를 반환합니다.
     *
     * @return 파티매칭 요청인경우 파티의 크기(인원수), 유저매칭 요청인경우 0을 반환
     */
    @Override
    public int getPartySize() {
        return 0;
    }

    // 만일 UserMatchInfo 객체 사이에 비교가 필요하다면 Comparable 인터페이스를 구현합니다.
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



유저 매치 요청에서 재정의해야할 메소드를 표로 정리하면 다음과 같습니다.

| 콜백 이름    | 의미                | 설명                                                         |
| ------------ | ------------------- | ------------------------------------------------------------ |
| getId        | 매치 요청자 정보    | 해당 유저 매치 요청이 어느 유저로부터 온 것인지 판단하기 위해 사용됩니다. 그러므로 반드시 요청자의 ID를 반환하도록 재정의 해주어야 합니다. |
| getPartySize | 매치 요청 파티 규모 | 요청 파티의 크기를 반환합니다. 이 값으로 파티 매치메이킹 요청인지를 판단하므로 유저 매치메이킹 요청인 경우에는 반드시 0을 반환하도록 재정의합니다. 파티 매치 요청의 경우에는 해당 파티원읜 인원수를 반환하도록 합니다. |



### 3.2. 유저 매치 메이커

이제 유저 매치 요청을 실제로 처리할 매치메이커를 만들 차례입니다. 유저 매치메이커는 엔진에서 제공하는 BaseUserMatchMaker 추상 클래스를 상속 구현합니다. 특히, onMatch() 메서드는 실제 매칭을 수행하기 위해 호출되는 콜백이므로 주의 깊게 살펴보세요.  onRefill() 메서드는 이미 완료된 매치 메이킹에 대해 충원 요청을 처리하는 콜백입니다. 예를 들어 4명이 매치메이킹 된 상태에서 1명이 게임을 종료해버렸을 때 1명을 더 충원하기 위해 사용할 수 있습니다. 아래의 예제 코드는 이러한 유저 매치메이커를 구현하는 방법을 보여줍니다.



특히 매치메이커를 구현한 클래스는 @ServiceName 어노테이션을 사용하여 특정 서비스에 대한 용도로 엔진에 등록합니다. 또한 매치메이커에 의해 생성될 방의 타입을 @RoomType 어노테이션으로 미리 정의합니다. 이 때, 하나의 매치메이커 클래스는 오직 하나의 서비스에 대해서만 등록할 수 있습니다.

```java
@ServiceName("MyGame") // "MyGame"이라는 서비스에 대한 매치 메이커를 엔진에 등록
@RoomType("1vs1") // 이 매치메이커에 의해 생성되는 방의 종류를 의미하는 문자열
public class UserMatchMaker extends BaseUserMatchMaker<UserMatchInfo> {

    private static final Logger logger = getLogger(UserMatchMaker.class);

    public UserMatchMaker() {
        super(2, 5000); // 매치메이킹은 2명 정원이며 요청이 5초 내에 매칭되지 않으면 Timeout 처리되도록 인자를 설정
    }

    private Multiset<UserMatchInfo> ratingSet = TreeMultiset.create();
    private final int matchMultiple = 1; // match 정원의 몇 배수까지 인원을 모은 후에 rating 별로 정렬해서 매칭할 것인가?
    private int currentMultiple = matchMultiple;
    private long lastMatchTime = System.currentTimeMillis();
    private int totalMatchMakings = 0;

    /**
     * 유저 매치메이킹과 파티 매치메이킹 요청을 처리. 첫번째 호출 이후 주기적으로 호출됩니다.
     * 
     * getMatchRequests(int)를 이용해 현재까지 누적된 전체 매치 요청 목록을 얻어올 수 있습니다.
     * 사용자는 이 목록을 이용해 조건에 맞는 요청들을 별도의 Collection에 순서대로 적재한 후 matchSingles(Collection)이나 assignRoom(BaseUserMatchInfo) 또는 assignRoom(Collection) API를 이용해 매치 완성을 할 수 있습니다.
     * 만일, 최소한의 요청 수가 모이지 않았으면 다음 호출 시에 처리합니다.
     */
    @Override
    public void onMatch() {
        List<UserMatchInfo> matchRequests = getMatchRequests(matchSize * currentMultiple);

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

    /**
     * 유저/파티 매치메이킹 과정에서 임의의 유저가 나갔을 경우에 새로운 유저를 채우기 위한 처리
     * 
     * 일반적으로 매치메이킹 된 방에 대해 onLeaveRoom이 호출될 때 matchRefill을 호출하여 연동할 수 있습니다. 즉, 매칭된 방에서 누군가 나갈 때 리필을 요청하는 것입니다. 리필은 큐에 쌓여있는 매치 요청은 사용하지 않습니다. 리필 요청 이 후에 들어오는 새로운 매치 요청만 그 대상으로 합니다.
     * 
     * getRefillRequests API를 이용하여 리필 요청들을 획득할 수 있습니다. 게임에 따라 리필을 사용하기 보다는 임의의 유저가 나간 상태로 게임을 진행하거나, 매칭된 게임을 취소한 후 다시 매치메이킹을 요청하는 것이 더 나은 방법일 수 있습니다.
     *
     * @param req 리필에 사용할 신규 매치 요청 정보
     * @return 리필이 성공하면 true를 실패하면 false를 반환합니다.
     */
    @Override
    public boolean onRefill(UserMatchInfo matchReq) {
        try {
            List<UserMatchInfo> refillRequests = getRefillRequests();

            if (refillRequests.isEmpty()) {
                return false;
            }

            for (UserMatchInfo refillInfo : refillRequests) {
                // 100점 이상 차이나지 않으면 리필
                if (Math.abs(matchReq.getRating() - refillInfo.getRating()) < 100) {
                    if (refillRoom(matchReq, refillInfo)) { // 해당 매칭 요청을 리필이 필요한 방으로 매칭
                        return true;
                    }
                }
            }
        } catch (Exception e) {
            logger.error("UserMatchMaker::refill()", e);
        }

        return false;
    }
}
```



이러한 유저 매치메이커의 콜백 메소드를 정리하면 다음의 표와 같습니다.

| 콜백 이름 | 의미                | 설명                                                         |
| --------- | ------------------- | ------------------------------------------------------------ |
| onMatch   | 매치 요청들을 처리  | 사용자는 BaseUserMatchMaker가 제공하는 API를 사용하여 직접 매치 요청을 처리할 수 있습니다. 이 때, 사용자가 원하는대로 로직을 직접 구현할 수 있습니다. 예제 코드의 처리 흐름이 가장 기본적인 방식입니다. <br> 즉, getMatchRequests API를 이용하여 최소한의 매치 요청들을 획득한 후 사용자가 원하는 대로 인원수에 맞춰 요청들을 조합한 후 임의의 Collection에 순서대로 담습니다. 이 Collection을 matchSingles API에 인자로 전달하면 정원수에 맞춰 매치가 이루어집니다. 예제 코드의 경우 정원이 2명인 유저 매치메이킹이므로 Collection을 순회하며 순서대로 2명씩 추출하여 하나의 게임으로 매칭시킵니다. |
| onRefill  | 매치 리필 요청 처리 | 유저/파티 매치메이킹 과정에서 임의의 유저가 나갔을 경우에 새로운 유저를 채우기 위한 처리를 합니다.<br>일반적으로 매치메이킹 된 방에 대해 onLeaveRoom이 호출될 때, matchRefill을 호출하여 연동할 수 있습니다. 즉, 매칭된 방에서 누군가 나갈 때 리필을 요청하는 것입니다. 리필은 큐에 쌓여있는 매치 요청은 사용하지 않습니다. 리필 요청 이 후에 들어오는 새로운 매치 요청만 그 대상으로 합니다. |



### 3.3. GameUser에서 매치 메이커로 요청 전달하기

이제 클라이언트는 서버로 유저 매치메이킹을 요청할 수 있습니다. 이 요청은 GameUser에 전달된 후 엔진에 의해 onMatchUser 콜백 메서드를 호출합니다. 이에 대해서는 앞 서 GameNode와 GameUser를 설명하면서 한 번 살펴보았습니다. 사용자는 이 콜백 메서드에서 GameAnvil이 제공하는 유저 매치메이커를 사용해도 되고 직접 구현한 별도의 매치메이커나 다른 솔루션을 사용해도 됩니다. 하지만 특별한 이유가 없다면 GameAnvil의 유저 매치메이킹을 사용하길 권합니다.

아래의 예제는 엔진에서 제공하는 유저 매치메이커를 사용하기 위해 앞 서 살펴본 매치 요청을 생성하여 matchUser API를 호출하는 모습입니다. 특히, 매개 변수로 전달된 매칭 그룹은 매치메이킹을 논리적으로 나눌 수 있는 개념이라고 앞서 설명했습니다. 만일 "Newbie"라는 매칭 그룹을 전달했다면 동일한 "Newbie" 매칭 그룹끼리 같은 매칭 큐를 공유하게 됩니다.

```java
    /**
     * 클라이언트에서 userMatch 를 요청했을 경우 호출되는 콜백
     *
     * @param roomType   클라이언트와 서버 사이에 사전 정의한 방 종류를 구분하는 임의의 값
     * @param matchingGroup 클라이언트가 요청한 매칭그룹
     * @param payload    클라이언트로부터 전달받은 페이로드
     * @param outPayload 클라이언트로 전달할 페이로드
     * @return 반환값이 true이면 유저 매치메이킹 요청 성공이고 false 실패
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public boolean onMatchUser(final String roomType, final String matchingGroup,
                               final Payload payload, Payload outPayload) throws SuspendExecution {

        UserMatchInfo userMatchInfo = new UserMatchInfo(getUserId()); // 매치 요청 생성
        userMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, userMatchInfo, payload);
    }
```



마지막으로 클라이언트는 언제든 앞서 요청한 유저 매치메이킹에 대해 취소를 할 수 있습니다. 이때, 엔진은 취소 처리가 성공적으로 진행되면 다음과 같이 GameUser의 onMatchUserCancel 콜백 메서드를 호출합니다. 사용자는 이 콜백에서 취소 타이밍에 처리하고 싶은 부분을 구현하면 됩니다.

```java
    /**
     * 클라이언트에서 userMatch 가 취소되었을 때 호출되는 콜백
     *
     * @param reason 취소된 이유(TIMEOUT/CANCEL)
     * @return 성공적으로 취소가 되었으면 true를 반환하고 취소할 수 없는 경우에는 false를 반환합니다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
    }
```



## 4. 룸 매치메이커 구현

룸 매치메이킹은 유저를 가장 적합한 방으로 자동 입장시켜주는 기능입니다. 룸 매치메이킹을 요청한 유저를 어떤 기준으로 어느 방에 입장시킬지는 사용자가 구현하기에 달렸습니다. 가장 유저수가 많은 방으로 입장시킬 수도 있고, 가장 한산한 방으로 입장시킬 수도 있습니다. 혹은 평균 점수가 가장 높은 방으로 입장시킬 수도 있습니다. 사용자는 이러한 매칭 로직에만 집중하면 됩니다. 참고로 가장 대표적인 룸 매치메이킹 게임은 "*한게임 포커*"나 "*카트라이더*" 등이 있습니다.



### Note

- *매치메이커는 매칭 그룹 단위로 생성됩니다. 즉, 동일한 매칭 그룹끼리 매치메이킹을 수행합니다.*
- *룸 매치메이커와 유저 매치메이커는 서로 독립적으로 운영됩니다. 즉, 동일한 매칭 그룹으로 유저 매칭과 룸 매칭을 각각 요청하더라도 이 두 요청이 함께 매칭되는 일은 없습니다.*



### 4.1. 룸 매치 요청 구현

이러한 룸 매치 메이킹의 가장 기본은 바로 매칭 요청 그 자체입니다. 매칭 요청은 한 명의 유저가 보낸 요청을 의미하며 아래와 같이 엔진에서 제공하는 BaseRoomMatchForm 추상 클래스를 상속 구현합니다. 요청은 언제든 직렬화할 수 있어야 하므로 Serializable 인터페이스를 추가로 구현해야 합니다. 다음은 이러한 매치 요청을 구현한 예제입니다.

```java
public class GameRoomMatchForm extends BaseRoomMatchForm implements Serializable {
	
	// 사용자가 매칭에서 사용할 조건 추가
	private int money;
	private int level;

	public GameRoomMatchForm(int money, int level) {
        this.money = money;
        this.level = level;
    }
    
    public GameRoomMatchForm(String matchingUserCategory) {
        super(matchingUserCategory);
    }
    
    ...
}
```

룸 매치 요청은 기본적으로 매칭 로직에서 사용할 정보를 포함합니다. 이는 사용자가 매치 메이킹 로직을 직접 구현할 때 사용하게 됩니다. 룸 매치 요청에서 한 가지 중요한 정보는 매칭 유저 카테고리입니다. 매칭 유저 카테고리는 하나의 방에서 유저들이 속한 그룹을 구분하기 위한 임의의 문자열입니다. 예를 들어 4인 정원인 방에서 두 개의 팀으로 나눠서 2 vs 2의 게임을 할 경우, 각각의 유저가 속할 팀을 지정하기 위한 용도로 사용될 수 있습니다. 만일 아무런 값도 지정하지 않으면 엔진의 기본 값이 사용됩니다.



### 4.2. 룸 매치 정보 구현

매칭 대상이 되는 방은 룸 매치메이커에 의해 매치 정보가 관리됩니다. 즉, 하나의 룸 매치 정보는 하나의 매칭 가능한 방 정보를 의미한다고 생각할 수 있습니다. 이 때, 해당 방의 여러가지 정보와 상태값들을 포함할 수 있습니다. BaseRoomMatchInfo를 상속 구현하며 필수로 방의 ID와 매칭 유저 카테고리 그리고 각 매칭 카테고리별 최대 정원 수를 설정해야 합니다. 그리고 마지막으로 룸 매치 정보도 직렬화를 위해 반드시 Serializable 인터페스를 구현해야 합니다. 아래는 예제 코드를 보여줍니다. 

```java
public class GameRoomMatchInfo extends BaseRoomMatchInfo implements Serializable {
	private static final int MAX_USER = 3;
	
	public GameRoomMatchInfo(int roomId) {
		// 별도의 매칭 유저 카테고리가 필요 없을 경우에는 기본값을 사용할 수 있습니다.
    	super(roomId, MAX_USER);
	}
    
    ...
}
```


아래의 예제는 사용자가 임의의 매칭 유저 카테고리를 설정하는 경우입니다. 하나의 게임 방에 3명의 레드팀과 3명의 블루팀으로 구성하고 있습니다. 이렇게 카테고리를 나누어 둘 경우 룸 매치메이커가 각각의 매칭 카테고리별 유저 수를 알아서 관리해줍니다. 그와 더불어 매치메이킹에 사용할 추가 정보를 원하는대로 구성해둘 수 있습니다.

```java
public class GameRoomMatchInfo extends BaseRoomMatchInfo implements Serializable {    
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



### 4.3. 룸 매치 정보 등록/갱신

이러한 룸 매치 정보는 사용자가 직접 등록/갱신할 수 있습니다. 즉, 사용자가 원하지 않을 경우에는 특정 방은 룸 매치 메이킹 대상으로 등록하지 않을 수도 있습니다. 이런 등록 절차는 일반적으로 다음과 같이 방이 생성되는 onCreateRoom 콜백 메서드에서 주로 진행합니다.

```java
@Override
public final boolean onCreateRoom(final GameUser user, final Payload payload, Payload outPayload) throws SuspendExecution {
    
    ...
        
	// 이제부터 이 방은 룸 매치메이킹에 사용됩니다.
	registerRoomMatch(gameRoomMatchInfo, user.getUserId());
}
```

임의의 방에 대해 룸 매치메이킹 등록 여부를 확인하기 위해 BaseRoom 클래스는 다음과 같은 API를 제공합니다.

```java
/**
 * 룸매치에 등록된 방인지 확인합니다.
 *
 * @return 등록된 방이면 true를 반환합니다.
 */
final public boolean isRegisteredRoomMatch()
```

이렇게 등록해둔 룸 매치 정보는 언제든 필요에 따라 사용자가 그 정보를 갱신할 수도 있습니다.  갱신을 위해 새로운 룸 매치 정보를 만든 후에 updateRoomMatch API를 호출하면 됩니다.

```java
GameRoomMatchInfo gameRoomMatchInfo = new GameRoomMatchInfo(getId());
gameRoomMatchInfo.setMemberMoney(1000);

updateRoomMatch(gameRoomMatchInfo); // 이 룸 매치 정보를 갱신합니다.
```



마지막으로 해당 방이 사라질 때, 룸 매치 정보는 자동으로 엔진에서 삭제됩니다. 그러므로 삭제는 사용자가 따로 신경쓸 필요가 없습니다.



### 4.4. 룸 매치메이커

이제 룸 매치메이커를 만들 차례입니다. 룸 매치메이커는 엔진에서 제공하는 BaseRoomMatchMaker 추상 클래스를 상속 구현 합니다. 룸 매치메이킹은 가장 적합한 방을 찾는 과정이므로 실제 매칭 전/후를 위한 특별한 콜백 메서드들이 제공됩니다. 사용자는 이 콜백 메서드들을 재정의하여 원하는 대로 매칭을 수행할 수 있습니다. 아래의 예제 코드는 이러한 룸 매치메이커를 어떤식으로 구현할 수 있는지 보여줍니다.  



특히 매치메이커를 구현한 클래스는 @ServiceName 어노테이션을 사용하여 특정 서비스에 대한 용도로 엔진에 등록합니다. 또한 매치메이커에 의해 생성될 방의 타입을 @RoomType 어노테이션으로 미리 정의합니다. 이 때, 하나의 매치메이커 클래스는 오직 하나의 서비스에 대해서만 등록할 수 있습니다.

```java
@ServiceName("MyGame") // "MyGame"이라는 서비스에 대한 매치 메이커를 엔진에 등록
@RoomType("REDvsBLUE") // 이 매치메이커에 의해 생성되는 방의 종류를 의미하는 문자열
public class GameRoomMatchMaker extends BaseRoomMatchMaker<GameRoomMatchForm, GameRoomMatchInfo> {
    
    /**
     * 룸 매치를 시작하기 전에 요청 정보를 기반으로 필요한 사전 처리를 구현합니다.
     *
     * @param baseRoomMatchForm 전달받은 룸 매칭 요청
     * @param args  추가로 전달 받은 데이터
     * @return 매칭 결과 코드
     */
    @Override
    public RoomMatchResultCode onPreMatch(GameRoomMatchForm roomMatchForm, Object... args) {
        // 매칭 요청서의 Money가 100보다 작을 경우 매칭을 시작하지 않고, 매칭을 신청한 클라이언트에게 실패 결과 코드(1000)를 전달
        if (roomMatchForm.getMoney() < 100)
            return RoomMatchResultCode.FAIL(1000);

        return RoomMatchResultCode.SUCCESS;
    }
    
    /**
     * 룸 매치 요청과 임의의 룸 매치 정보가 매칭 가능한 상태인지 확인합니다.
     * 엔진은 룸 매치메이킹이 성공할 때까지 전체 방 목록에 대해 한번씩 이 콜백 메서드를 호출합니다.
     *
     * @param baseRoomMatchForm 전달받은 룸 매치 요청
     * @param baseRoomMatchInfo 매칭 풀에 등록되어 있는 룸 매치 정보(매칭이 가능한 방 정보)
     * @param args  추가로 전달 받은 데이터
     * @return 매칭 성공 여부
     */
    @Override
    public boolean canMatch(GameRoomMatchForm roomMatchForm, GameRoomMatchInfo roomMatchInfo, Object... args) {
        if (roomMatchForm.getLevel() > roomMatchInfo.getAvgLevel())
            return false;
        return true;
    }
    
    /**
     * 룸 매치가 성공한 후 필요한 처리를 구현합니다.
     *
     * @param baseRoomMatchForm 룸 매치 요청
     * @param baseRoomMatchInfo 매칭된 룸 매치 정보
     * @param args  추가로 전달 받은 데이터
     */
    @Override
    public void onPostMatch(GameRoomMatchForm baseRoomMatchForm, GameRoomMatchInfo baseRoomMatchInfo, Object... args) {
        // 매칭이 정상적으로 진행되고 있는 지 확인한다.
        logger.debug("GameRoomMatchMaker::onPostMatch() matching success, roomId({})", baseRoomMatchInfo.getRoomId());
    }
    
    /**
     * 룸 매치 정보를 정렬하기 위해 구현합니다.
     *
     * @return 비교 결과를 반환
     */
    @Override
    public int compare(GameRoomMatchInfo o1, GameRoomMatchInfo o2) {
        if (o1.getCreateTime() < o2.getCreateTime()) {
            return -1;
        } else if (o1.getCreateTime() > o2.getCreateTime()) {
            return 1;
        } else {
            return 0;
        }
    }
    
    /**
     * 룸 매치메이킹 대상인 방에서 유저수가 증가할 때 호출됩니다.
     *
     * @param roomId 유저수가 증가한 방의 아이디
     * @param matchingUserCategory 증가한 유저의 매칭 유저 카테고리
     * @param currentUserCount 해당 매칭 유저 카테고리에 속하는 유저수
     */
    @Override
    public void onIncreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {
        // 유저수가 변경되었으므로 매칭 풀을 다시 정렬하여 유저가 적은 방이 먼저 매칭되도록 하기 위함
        sort();
    }
    
    /**
     * 룸 매치메이킹 대상인 방에서 유저수가 감소할 때 호출됩니다.
     *
     * @param roomId 유저수가 감소한 방의 아이디
     * @param matchingUserCategory 감소한 매칭 유저 카테고리
     * @param currentUserCount 해당 매칭 유저 카테고리에 속하는 유저수
     */
    @Override
    public void onDecreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {
        // 유저수가 변경되었으므로 매칭 풀을 다시 정렬하여 유저가 적은 방이 먼저 매칭되도록 하기 위함
        sort();
    }
    
    /**
     * 임의의 룸 매치 정보 목록을 반환합니다. 
     * 인자로 넘겨받은 현재 매칭풀(전체 매칭 가능한 방의 목록)을 원하는대로 가공하여 원하는 목록을 만들어 반환할 수 있습니다.
     * 만일 이 메서드를 재정의하지 않으면 전체 방 목록을 반환합니다.
     *
     * @return 가공한 룸 매치 정보의 목록
     */
    @Override
    public List<GameRoomMatchInfo> getRooms(List<GameRoomMatchInfo> rooms){
        // 매칭풀에서 방10개를 가져온다.
        List<GameRoomMatchInfo> gameRoomMatchInfoList = rooms.stream().limit(10).collect(Collectors.toList());
        // 가져온 방 10개를 랜덤으로 순서를 섞는다.
        Collections.shuffle(gameRoomMatchInfo);

        return gameRoomMatchInfoList;
    }    
}
```



앞서 살펴본 룸 매치메이커의 콜백 메서드를 정리하면 아래의 표와 같습니다.

| 콜백 이름           | 의미                              | 설명                                                         |
| ------------------- | --------------------------------- | ------------------------------------------------------------ |
| onPreMatch          | 룸 매치 시작 전 처리              | 룸 매치 요청에 저장된 각종 정보를 사용하여, 현재 룸 매치메이킹을 수행할 수 있는 상태인지 미리 체크하는 용도로 사용합니다. 예를 들어, 한 번의 룸 매치메이킹에 100 코인이 필요하다면 여기에서 해당 유저가 100코인 이상을 보유하고 있는지 확인할 수 있습니다. |
| canMatch            | 매치 확인                         | 인자로 전달받은 룸 매치 요청과 룸 매치 정보 사이에 매칭이 가능한지 확인하기 위해 호출합니다. 즉,  전체 룸 매치 풀을 순회하면서 해당 룸 매치 요청이 들어갈 가장 적합한 방을 찾기 위한 로직을 여기에 구현할 수 있습니다. |
| onPostMatch         | 룸 매치 성공 후 처리              | 룸 매치메이킹이 성공한 후에 호출됩니다. 룸 매치가 완료된 후 처리할 로직은 여기에 구현할 수 있습니다. |
| onIncreaseUserCount | 임의의 매칭 대상 방에서 유저 증가 | 임의의 방에서 유저가 증가할 경우 호출됩니다. 이 때, 증가한 유저가 어떤 매칭 유저 카테고리인지 그리고 해당 매칭 유저 카테고리의 총원이 몇 명인지도 인자로 전달됩니다. |
| onDecreaseUserCount | 임의의 매칭 대상 방에서 유저 감소 | 임의의 방에서 유저가 감소할 경우 호출됩니다.  이 때, 감소한 유저가 어떤 매칭 유저 카테고리인지 그리고 해당 매칭 유저 카테고리의 총원이 몇 명인지도 인자로 전달됩니다. |
| getRooms            | 임의의 방 목록을 획득             | 엔진의 기본 구현은 전체 룸 매치메이킹이 가능한 방의 목록을 반환합니다. 사용자는 이를 재정의하여 원하는 특정 방 목록만 반환할 수 있습니다. |



### 4.5. GameUser에서 매치 메이커로 요청 전달하기

이제 클라이언트는 서버로 룸 매치메이킹을 요청할 수 있습니다. 이 요청은 GameUser에 전달된 후 엔진에 의해 onMatchRoom 콜백 메서드를 호출합니다. 이에 대해서는 앞 서 GameNode와 GameUser를 설명하면서 한 번 살펴보았습니다. 사용자는 이 콜백 메서드에서 GameAnvil이 제공하는 룸 매치메이커를 사용해도 되고 직접 구현한 별도의 매치메이커나 다른 솔루션을 사용해도 됩니다. 하지만 특별한 이유가 없다면 GameAnvil의 룸 매치메이킹을 사용하길 권합니다.

아래의 예제는 엔진에서 제공하는 룸 매치메이커를 사용하기 위해 앞 서 살펴본 매치 요청을 생성하여 matchRoom API를 호출하는 모습입니다. 특히, 매개 변수로 전달된 매칭 그룹은 매치메이킹을 논리적으로 나눌 수 있는 개념이라고 앞서 설명했습니다. 만일 "Newbie"라는 매칭 그룹을 전달했다면 동일한 "Newbie" 매칭 그룹끼리 같은 매칭 큐를 공유하게 됩니다.

```java
    /**
     * 클라이언트에서 roomMatch 를 요청했을 경우 발생하는 콜백
     *
     * @param roomType 클라이언트와 서버 사이에 사전 정의한 방 종류를 구분하는 임의의 값
     * @param matchingGroup 클라이언트가 요청한 매칭그룹
     * @param matchingUserCategory 해당 유저가 속한 방 안에서의 카테고리
     * @param payload  클라이언트로부터 전달받은 페이로드
     * @return {@link MatchRoomResult} 으로 matching 된 room 의 정보 반환, null 을 반환 할시  클라이언트 요청 옵션에 따라서 새로운 Room 이 생성되거나,요청 실패 처리 된다.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
    public MatchRoomResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final Payload payload) throws SuspendExecution {
        
        GameRoomMatchForm gameRoomMatchForm = new GameRoomMatchForm(RoomMode.NORMAL, innerPayload.getOption(), 0);
        
        return matchRoom(matchingGroup, roomType, gameRoomMatchForm);
    }
```

