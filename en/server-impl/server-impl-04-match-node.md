## Game > GameAnvil > Server Development Guide > Match Node Implementation



## MatchNode and MatchMaker



![MatchNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_matchnode_on_network.png)


Users can simply implement matching logic and apply matchmaking by activating MatchNode. Matchmaker allows users to enter any room based on matching logic. GameAnvil offers two matchmakers. UserMatchMaker, which performs user-specific matching, and RoomMatchMaker, which performs room-specific matching.

These matchmakers run independently on MatchNode. At this time, MatchNode cannot implement additional content other than the purpose of performing matchmaking. Therefore, the user only needs to focus on the matchmaker, not the MatchNode. However, at least one matchNode must be enabled in order to perform the matchmaker.

> [Note] 
> 
> Matchmakers are created on a match group basis. In other words, the same match groups perform matchmaking. 
> 
> MatchNode is not a required node, so you do not need to run it if you do not use matchmaking.

## MatchingGroup

Similar to channels, matching groups are one of the logical ways to divide a single server group. However, unlike channels, matching groups are not clearly pre-configured and used value. Also, a channel is a way to logically divide GameNodes, while a matching group is a way to logically divide matchmaking. If user matching or room matching is requested from the same matching group, a matched room is created within the channel that requested the matching. 

Let's take a look again at onMatchUser, the user matchmaking callback method described earlier in the game. The string factor matchingGroup of the callback method is the value delivered together when the client requests user matchmaking. That is, a matching group is any string that is pre-arranged between the client and the server. The type or number can be used as many times as the user wants. All matchmaking requests are queued on a matchgroup basis and can only be matched within that queue. For example, if the value "beginner" is passed to the factor matchingGroup in the example below, the request is guaranteed to match other "beginner" requests.

```java
    /**  
     * Callback called when client requests userMatch 
     *  
     * @param roomType   Any value that separates the pre-defined room types between client and server  
     * @param payload    Payload received from clients  
     * @param outPayload Payload to be forwarded to clients
     * @return If the return value is true, the user matchmaking request is successful and false is failed  
     * @throws SuspendExecution This method means that the fiber can be suspended 
 
     */  
    @Override  
    public final boolean onMatchUser(final String roomType, final String matchingGroup, final Payload payload, Payload outPayload) throws SuspendExecution {  
  
        UserMatchInfo userMatchInfo = new UserMatchInfo(getUserId());  
        userMatchInfo.setRating(rating);  
  
        return matchUser(matchingGroup, roomType, userMatchInfo, payload);  
    } 

```

Matching groups can be defined based on skills, such as “Beginners,” “Medians” and “Masters” or they can be defined by country, such as “Korea,,“Japan” and “the United States.” In other words, any value that the user wants can be a matching group.



## Implementing User Match Maker

User matchmaking places game users' matching requests on the queue. It compares and analyzes the content of this request queue at a specific interval and allows arbitrary users to enter a room based on the standard that is desired by the user. Here, engine users can focus on the logic that determines how to compare and analyze the content of the request queue and how to match users. For reference, the most popular user matchmaking game is "League of Legends."


### Implementing User Match Request

The most fundamental element of this user matchmaking is the match request itself. These match requests are implemented by inheriting BaseUser MatchInfo abstract class provided by engine as follows. At this point, the getId() method must be implemented to provide a game user's ID that can distinguish the requester. You must also implement a Serializable interface because requests must be serializable at any time. The example below further implements a Comparable interface for comparison between matching requests.

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
     * It is used to determine which user match request came from. 
     *  
     * @return Returns RoomId for party matching requests and UserId for user matching requests 
     */  
    @Override  
    public int getId() {  
        return userId;  
    }  
  
    /**  
     * Returns the size of the request party. 
     *  
     * @return Returns the size of the party (number of people) for a party matching request and 0 for a user matching request 
     */  
    @Override  
    public int getPartySize() {  
        return 0;  
    }  
  
    // Implement the Computable interface if comparison is required between UserMatchInfo objects. 
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



The methods that need to be overridden in a user match request are summarized in a table as follows.

| Callback name    | Description                | Description                                                                                                                         |
| ------------ | ------------------- |----------------------------------------------------------------------------------------------------------------------------|
| getId        | Match requester information    | It is used to determine which user the user match request is from. Therefore, it must be overridden to return the requester's ID.                                           |
| getPartySize | Match Request Party Size | Returns the size of the request party. This value determines whether a party matchmaking request is made, which overrides to return 0 for user matchmaking requests. For a party match request, return the number of party members. |



### User match maker

User matchmaker actually handles user match requests and inherits BaseUserMatchMaker abstract class provided by engine, especially since the onMatch() method is a callback that is called to perform a real match, so please take a look carefully. onRefill() method is a callback that handles fill requests for matchmaking that has already been completed. For example, you can use it to fill one more person when four people are matchmaking and one person exits the game. The example code below shows how to implement these user matchmakers.


In particular, classes that implement matchmakers use the @ServiceName annotation to register with engine for specific services. In addition, the type of room created by the matchmaker is pre-defined as the @RoomType annotation. At this time, only one matchmaker class can be registered for one service.

```java
@ServiceName("MyGame") // Register a matchmaker for a service called "MyGame" with engine 
@RoomType("1vs1") // String representing the type of room created by this matchmaker 
public class UserMatchMaker extends BaseUserMatchMaker<UserMatchInfo> {  
  
    private static final Logger logger = getLogger(UserMatchMaker.class);  
  
    public UserMatchMaker() {  
        super(2, 5000); // Set a factor so that if the matchmaking has a capacity of two and if the request does not match within 5 seconds, the timeout is processed    }  
  
    private Multiset<UserMatchInfo> ratingSet = TreeMultiset.create();  
    private final int matchMultiple = 1; // match  How many times the number of people will be gathered and then sorted by rating and matched?     
    private int currentMultiple = matchMultiple;  
    private long lastMatchTime = System.currentTimeMillis();  
    private int totalMatchMakings = 0;  
  
    /**  
     * Processing user matchmaking and party matchmaking requests. Called periodically after the first call. 
     *   
     * GetMatchRequests(int) can be used to get a complete list of match requests accumulated to date. 
     * User uses this list to load the requests that meet the requirements in order in a separate collection, and then can complete matching by using  matchSingles(Collection), or assignRoom(BaseUserMatchInfo) or assignRoom(Collection) API.  
     * If the minimum number of requests has not been collected, it will be processed on the next call. 
     */  
    @Override  
    public void onMatch() {  
        List<UserMatchInfo> matchRequests = getMatchRequests(matchSize * currentMultiple);  
  
        // Not collected by minimum number (minAmount)  
        if (matchRequests == null) {  
            if (System.currentTimeMillis() - lastMatchTime >= 10000)  
                currentMultiple = Math.max(--currentMultiple, 1);  
  
            return;  
        }  
  
        // Items that have not been matched may remain in ratingSet but need not be stored separately.  
        // These items are delivered again in the following getMatchRequests(). 
        ratingSet.clear();  
        ratingSet.addAll(matchRequests);  
  
        if (ratingSet.size() >= matchSize) {  
  
            // Consume items by matchingAmount * matchSize in the order of ratingSet 
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
     * Processing to fill new users if any user leaves during user/party matchmaking 
     *   
     * Generally, when onLeaveRoom is called for a matchmaking room, you can call matchRefill to sync it. In other words, it is to ask for a refill when someone leaves the matched room. Refill does not use match requests stacked in the queue; it only covers new match requests that come in after refill requests. 
     *   
     * Can obtain refill request by using getRefillRequests API. Depending on the game, rather than using refill, it may be better to proceed with the game with any user out, or cancel the matched game and ask for matchmaking again. 
     *  
     * @param req New match request information to be used for refills 
     * @return Returns true if the refill is successful and false if fails.  
     */  
    @Override  
    public boolean onRefill(UserMatchInfo matchReq) {  
        try {  
            List<UserMatchInfo> refillRequests = getRefillRequests();  
  
            if (refillRequests.isEmpty()) {  
                return false;  
            }  
  
            for (UserMatchInfo refillInfo : refillRequests) {  
                // Refill if there's no more than 100 points difference 
                if (Math.abs(matchReq.getRating() - refillInfo.getRating()) < 100) {  
                    if (refillRoom(matchReq, refillInfo)) { // Match the matching request to a room that requires refilling 
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



Callback methods for these user matchmakers are summarized in the table below.

| Callback name | Description                | Description                                                                                                                                                                                                                                                                                                                                                                                        |
| --------- | ------------------- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onMatch   | Process match request   | Users can directly process match requests using API provided by BaseUserMatchMaker. In other words, it allows users to implement logic directly as they wish. The processing flow of example code is the most basic method. <br> In other words, the minimum match requests are obtained using getMatchRequests API, the requests are combined according to the number of people the user wants, and then put in order in a random collection. If you pass this collection as a factor to the matchSingles API, it matches the number of people in the group. For an example code, it is two-person user matchmaking, so we move around the collection and extract two people in order to match them to one game. |
| onRefill  | Process Match refill request  | If any user leaves during the user/party matchmaking process, the process is performed to fill in the new user. In general, you can call matchRefill when onLeaveRoom is called for a match-making room. This means asking for a refill when someone leaves a matched room. Refill does not use match requests stacked in the queue. It only covers new match requests that come in after refill requests.                                                                                                                                                      |



### Send requests from GameUser to matchmaker

Client can now request a user matchmaking from server. This request is forwarded to GameUser and then the engine calls the onMatchUser callback method. We have looked at earlier while explaining GameNode and GameUser. Users can use the user matchmaker provided by GameAnvil in this callback method, or they can use a separate matchmaker or other solution they have implemented themselves. However, unless there is a specific reason, we recommend using GameAnvil's user matchmaking.

The example below shows calling the matchUser API by creating a match request that was previously described to use the user matchmaker provided by engine. In particular, we explained earlier that a matching group passed through parameters is a concept that can logically divide matchmaking. If a matching group called "Newbie" was passed, the same "Newbie" matching group would share the same matching queue.

```java
    /**  
     * Callback called when client requests userMatch 
     *  
     * @param roomType   Any value that separates the pre-defined room types between client and server   
     * @param matchingGroup Matching group requested by client 
     * @param payload    Payload delivered from clients  
     * @param outPayload Payload to be forwarded to clients   
     * @return If the return value is true, the user match making request is successful, if false it is failed 
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    public boolean onMatchUser(final String roomType, final String matchingGroup,  
                               final Payload payload, Payload outPayload) throws SuspendExecution {  
  
        UserMatchInfo userMatchInfo = new UserMatchInfo(getUserId()); // create matching request  
        userMatchInfo.setRating(rating);  
  
        return matchUser(matchingGroup, roomType, userMatchInfo, payload);  
    }
```



Finally, client can cancel the previously requested user matchmaking at any time. At this time, if the cancellation process is successful, the engine calls GameUser's onMatchUserCancel callback method as follows. Users can implement the part they want to process at the cancellation timing in this callback.

```java
    /**  
     * Callback called when client cancels userMatch 
     *  
     * @param reason Reason of cancelleation(TIMEOUT/CANCEL)  
     * @return Returns true if it is successfully canceled or false if it cannot be canceled.  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {  
    }
```



## Implementing Room Matchmaker

Room matchmaking is a feature that automatically allows users to enter the most suitable room. It is up to the user to implement which room to enter the user who requested room matchmaking. You can make to enter the room with the most users, or the quietest room. Also, you can enter the room with the highest average score. All users need to do is focus on this matching logic. For reference, the most representative room match making games include "Hangame Poker" and "Kartrider."


> [Note]
>
> Matchmakers are created on a match group basis. In other words, the same match groups perform matchmaking.
> 
> Room matchmakers and user matchmakers operate independently of each other. In other words, if you request user matching and room matching to the same matching group, the two requests will not match together.


### Implement Room matching request

The essence of such room matchmaking is the match request itself. A match request refers to a request sent by a single user and inherits the BaseRoomMatchForm abstract class provided by the engine, as shown below. Requests must be serializable at any time, so you need to implement an additional serializable interface. Here is an example of implementing such a match request.

```java
public class GameRoomMatchForm extends BaseRoomMatchForm implements Serializable {  
	  
	// Add conditions for users to use in matching 
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

Room matching requests basically contain information for use in matching logic. This is used by users to implement matchmaking logic themselves. One important piece of information in room matching requests is the matching user category. A matching user category is random string to separate groups of users in a room. For example, if you're playing 2 vs. 2 teams in a four-person room, it can be used to designate which teams each user belongs to. If no value is specified, the default value of the engine is used.



### Implement Room matching information

For a room to be matched, match information is managed by a room match maker. In other words, one room matching information may be considered to mean one matchable room information. In this case, various pieces of information and state values of the room may be included. It inherits and implement BaseRoomMatchInfo and must set the ID of the room, the matching user category and maximum number of people for each matching category. And finally, the serializable interface must be implemented to serialize the room matching information. Below shows an example code. 

```java
public class GameRoomMatchInfo extends BaseRoomMatchInfo implements Serializable {  
	private static final int MAX_USER = 3;  
	  
	public GameRoomMatchInfo(int roomId) {  
		// If you do not need a separate matching user category, you can use the default values.    	super(roomId, MAX_USER);  
	}  
      
    ...  
} 
```


The example below is when a user sets up a random matching user category. A game room consists of three players each, Red Team and Blue Team. When categories are divided in this way, room matchmaker manages the number of users for each matching category. You can also configure any additional information want to use for matchmaking.

```java
public class GameRoomMatchInfo extends BaseRoomMatchInfo implements Serializable { 
    private static final int MAX_USER = 3;

	// match condition information
    int mapId;
    int maxLevel;
    int avgLevel = 0;
    
    public GameRoomMatchInfo(int roomId) {
	// Adds the room information for RED_TEAM with a maximum of 3 players.
    	super(roomId, "RED_TEAM", 3);
        
        // Register additional match categories if needed.
        // BLUE_TEAM Adds people information with a maximum of 3 people.
        addMatchingUserCategory("BLUE_TEAM", 3);
    }
    
    public void setAvgLevel(int avgLevel) {
        this.avgLevel = avgLevel;
    }
    
    ...
}
```



### Register/Refresh Room matching Information

These room matching information can be registered/updated directly by user, which means that a particular room may not be registered as a room matchmaking target if user does not want to. This registration process is typically done in onCreateRoom callback method, where a room is created as follows.

```java
@Override  
public final boolean onCreateRoom(final GameUser user, final Payload payload, Payload outPayload) throws SuspendExecution {  
      
    ...  
          
	// From now on, this room will be used for room match making.  
	registerRoomMatch(gameRoomMatchInfo, user.getUserId());  
}
```

To verify room matchmaking registration for any room, BaseRoom class provides the following APIs.

```java
/** 
 * Verify that the room is registered for room match. 
 * 
 * @return Returns true if it is a registered room. 
 */ 
final public boolean isRegisteredRoomMatch() 

```

This registered room matching information can be updated by user as needed at any time. You can create new room matching information for update and call updateRoomMatch API.

```java
GameRoomMatchInfo gameRoomMatchInfo = new
GameRoomMatchInfo(getId());  
gameRoomMatchInfo.setMemberMoney(1000);  
  
updateRoomMatch(gameRoomMatchInfo); // Update this room matching information. 
```



When the room disappears, the room matching information is automatically deleted from the engine, so you don't need to delete it separately.



### Room matchmaker

Now it's time to create a room matchmaker. The room matchmaker inherits BaseRoomMatchMaker abstract class provided by engine. Room matchmaking is the process of finding the most suitable room, so special callback methods are provided for before and after the actual match. Users can override these callback methods to perform any match they want. The example code below shows how these room matchmakers can be implemented.  



In particular, classes that implement matchmakers use the @ServiceName annotation to register with engine for specific services. In addition, the type of room created by the matchmaker is pre-defined as the @RoomType annotation. At this time, only one matchmaker class can be registered for one service.

```java
@ServiceName("MyGame") // Register a matchmaker for a service called "MyGame" with the engine  
@RoomType("REDvsBLUE") // String representing the type of room created by this matchmaker 
public class GameRoomMatchMaker extends BaseRoomMatchMaker<GameRoomMatchForm, GameRoomMatchInfo> {  
      
    /**  
     * Implement the required preprocessing based on the request information before starting roommatch.     *  
     * @param baseRoomMatchForm Room matching request received 
     * @param args  Additional data received 
     * @return Matching Result Code 
     */  
    @Override  
    public RoomMatchResultCode onPreMatch(GameRoomMatchForm roomMatchForm, Object... args) {  
        // If the Money in the matching request is less than 100, matching is not started, and the failure result code (1000) is delivered to the client who applied for matching 
        if (roomMatchForm.getMoney() < 100)  
            return RoomMatchResultCode.FAIL(1000);  
  
        return RoomMatchResultCode.SUCCESS;  
    }        
   
  /**  
     * Verify that the room matching request and any room matching information are matchable. 
     * Engine calls this callback method once for the entire room list until the room matchmaking is successful. 
     *  
     * @param baseRoomMatchForm Room matching request received 
     * @param baseRoomMatchInfo Room matching information registered in the matching pool (room information that can be matched) 
     * @param args  Additional data received 
     * @return Whether matching is successful or not  
     */  
    @Override  
    public boolean canMatch(GameRoomMatchForm roomMatchForm, GameRoomMatchInfo roomMatchInfo, Object... args) {  
        if (roomMatchForm.getLevel() > roomMatchInfo.getAvgLevel())  
            return false;  
        return true;  
    }
 
    /**  
     * Implement the necessary processing after the room matching is successful. 
     *  
     * @param baseRoomMatchForm Room matching request  
     * @param baseRoomMatchInfo Room matching information that has matched  
     * @param args  Additional data received 
     */  
    @Override  
    public void onPostMatch(GameRoomMatchForm baseRoomMatchForm, GameRoomMatchInfo baseRoomMatchInfo, Object... args) {  
        // Verify that the matching normally working. 
        logger.debug("GameRoomMatchMaker::onPostMatch() matching success, roomId({})", baseRoomMatchInfo.getRoomId());  
    }  
      
    /**  
     * Implement to sort room matching information. 
     *  
     * @return return comparison result  
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
     * Called when the number of users increases in the room where the room matching is being made.  
     *  
     * @param roomId   ID of Room where the number of users increased      
     * @param matchingUserCategory Matching user categories for increased users 
     * @param currentUserCount  Number of users in that matching user category 
     */  
    @Override  
    public void onIncreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {  
        // Because the number of users has changed, it is to rearrange the matching pool so that rooms with fewer users match first 
        sort();  
    }  
      
    /**  
     * Called when the number of users decreases in the room where the room matching is being made.  
     *  
     * @param roomId  ID of Room where the number of users increased      
     * @param matchingUserCategory Matching user categories for decreased users 
     * @param currentUserCount Number of users in that matching user category 
     */  
    @Override  
    public void onDecreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {  
        // Because the number of users has changed, it is to rearrange the matching pool so that rooms with fewer users match first 
        sort();  
    }  
       
    /**  
     * Returns a list of random room matching information.   
     * You can process the current matching pool (the list of all matchable rooms) that you have taken over as a factor to create and return the list you want. 
     * If you do not override this method, return the full room list.  
     *  
     * @return List of processed room matching information 
     */  
    @Override  
    public List<GameRoomMatchInfo> getRooms(List<GameRoomMatchInfo> rooms){  
        // Get 10 rooms from matching pool. 
        List<GameRoomMatchInfo> gameRoomMatchInfoList = rooms.stream().limit(10).collect(Collectors.toList());  
        // Mix the order of the 10 rooms you brought at random. 
        Collections.shuffle(gameRoomMatchInfo);  
  
        return gameRoomMatchInfoList;  
    }      
}
```



The following table summarizes the room matchmaker's callback methods explained earlier.

| Callback name           | Description                              | Description                                                                                                                                                    |
| ------------------- | --------------------------------- |-------------------------------------------------------------------------------------------------------------------------------------------------------|
| onPreMatch          | Processing before starting a room matching              | It is used to check in advance whether you are currently able to perform room match making using various information stored in a room matching request. For example, if you need 100 coins for one room match making, you can check here to see if the user has more than 100 coins. |
| canMatch            | Verify match                         | It is called to see if a match can be made between the room matching request that was forwarded as a factor and the room matching information. In other words, you can implement logic here to find the best room for that room matching request while circulating the entire room matching pool.                       |
| onPostMatch         | Process after successful room match              | It is called after a successful room matchmaking. The logic to process after completed room matching can be implemented here.                                                                                           |
| onIncreaseUserCount | Increasing users in any matching destination room | It is called when there are more users in any room. At this time, the number of matching user categories and the total number of people in that matching user category are also transferred as factors.                                                        |
| onDecreaseUserCount | Decreasing users in any matching destination room | It is called when users decrease in any room. At this time, which matching user category is the reduced user and how many people are there in total for that matching user category is also transferred as a factor.                                                       |
| getRooms            | Obtain a list of random rooms             | Default implementation of engine returns a list of rooms with full room where able to do matchmaking. Users can override this and return only a list of specific rooms they want.                                                                       |



### Send requests from GameUser to matchmaker

Client can now request a room matchmaking from server. This request is forwarded to GameUser and then the engine calls onMatchRoom callback method. We have shared about this earlier while explaining GameNode and GameUser. Users can use the room matchmaker provided by GameAnvil in this callback method, or use a separate matchmaker or other solution they have implemented themselves. However, unless there is a specific reason, we recommend using GameAnvil's room matchmaking.

The example below shows a call to matchRoom API by creating a match request that was previously discussed in order to use the room matchmaker provided by engine. In particular, we explained that the matching group forwarded as parameters is a concept that can logically divide matchmaking. If you forwarded a matching group named "Newbie," the same "Newbie" matching group would share the same matching queue.

```java
    /**  
     * Callback that generated when a client requests a roomMatch 
     *  
     * @param roomType Any value that separates the pre-defined room types between client and server 
     * @param matchingGroup Matching group requested by client  
     * @param matchingUserCategory categories within the room to which the user belongs to  
     * @param payload  Payload delivered from clients  
     * @return   information return that matched with {@link MatchRoomResult}, When null is returned, a new Room is created or a request failure is processed according to the client request option.
     * @throws SuspendExecution This method means that the fiber can be suspended.  
     */  
    public MatchRoomResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final Payload payload) throws SuspendExecution {  
          
        GameRoomMatchForm gameRoomMatchForm = new GameRoomMatchForm(RoomMode.NORMAL, innerPayload.getOption(), 0);  
          
        return matchRoom(matchingGroup, roomType, gameRoomMatchForm);  
    }
```

