## Game > GameAnvil > Server Development Guide > MatchNode Implementation

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
     * @param matchingGroup Matching group
     * @param payload    Payload received from clients  
     * @param outPayload Payload to be forwarded to clients
     * @return If the return value is true, the user matchmaking request is successful and false is failed  
     */  
    @Override
    public boolean onMatchUser(final String roomType, final String matchingGroup, final IPayload payload, IPayload outPayload) {

    SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId(), rating);

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
         * Return the ID of the requesting subject
         * <p/>
        * If it is a party matchmaking request, it returns the room ID, and if it is a user matchmaking request, it returns the user ID.
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
    public int compareTo(SampleUserMatchInfo  o) {  
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

```java
@GameAnvilUserMatchMaker(loadClass = SampleRoom.class) // MatchMaker class registered on SampleRoom
public class SampleUserMatchMaker extends AbstractUserMatchMaker<SampleUserMatchInfo> {

    public SampleUserMatchMaker() {
        super(2, 5000);
    }

    private int matchSize = 2;

    /**
     * Process user/party matchmaking requests
     * <p>
     * Call {@link IUserContext#matchUser(String, String, AbstractUserMatchInfo)} from {@link IUser#onMatchUser(String, String, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)}
     * Or call {@link IRoomContext#matchParty(String, String, AbstractUserMatchInfo)} from {@link IRoom#onMatchParty(String, String, IUser, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)} to perform matchmaking.
     * <p>
    * Called periodically after the first call.
    * <p>
    * Get the UserMatchInfo list of users/parties that requested a match using {@link #getMatchRequests(int)}.
    * <p>
    * Collect requests that meet the criteria using this list and place them in the same room using {@link #matchSingles(Collection)}, {@link #assignRoom(AbstractUserMatchInfo)}, and {@link #assignRoom(Collection)}.
    * <p>
    * If there are not enough requests that meet the criteria, they will be processed in the next call.
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
     * If a user leaves a room created through user/party matching, a new user is assigned.
     * <p>
     * Call {@link IRoomContext#matchRefill(AbstractUserMatchInfo)} from {@link IRoom#canLeaveRoom(IUser, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)} to fill in new users.
    * <p>
    * When a user/party matchmaking request is made, {@link #onRefill(AbstractUserMatchInfo)} is called first to fill a room that needs to be refilled.
    * <p>
    * {@link #onRefill(AbstractUserMatchInfo)} is called only once for each user/party matchmaking request and is not called periodically.
    * <p>
    * Unused requests remain in the list of {@link #getMatchRequests(int)} and are continuously used in {@link #onMatch()}.
    * <p>
    * Retrieve the UserMatchInfo list of the rooms that requested a refill using {@link #getRefillRequests()}.
    * <p>
    * Find requests that match the criteria req in this list and place them in a room using {@link #refillRoom(AbstractUserMatchInfo, AbstractUserMatchInfo)}.
    *
    * @param req UserMatchInfo of the user or party that requested the user/party match. * @return If the return value is true, the refill is successful, if false, it is unsuccessful.
     */
    @Override
    public boolean onRefill(SampleUserMatchInfo matchReq) {
        return false;
    }
}
```

Specifically, a class implementing a matchmaker registers itself with a specific service in the engine. It also predefines the type of room to be created by the matchmaker. Note that a matchmaker class can only be registered with one service.

Callback methods for these user matchmakers are summarized in the table below:

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
     */
    @Override
    public boolean onMatchUser(final String roomType, final String matchingGroup, final IPayload payload, IPayload outPayload) {

    SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId(), rating);

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
public class SampleRoomMatchForm extends AbstractRoomMatchForm {
    public SampleRoomMatchForm() {
        super();
    }
}
```

Room matching requests basically contain information for use in matching logic. This is used by users to implement matchmaking logic themselves. One important piece of information in room matching requests is the matching user category. A matching user category is random string to separate groups of users in a room. For example, if you're playing 2 vs. 2 teams in a four-person room, it can be used to designate which teams each user belongs to. If no value is specified, the default value of the engine is used.

### Implement Room matching information

For a room to be matched, match information is managed by a room match maker. In other words, one room matching information may be considered to mean one matchable room information. In this case, various pieces of information and state values of the room may be included. It inherits and implement BaseRoomMatchInfo and must set the ID of the room, the matching user category and maximum number of people for each matching category. And finally, the serializable interface must be implemented to serialize the room matching information. Below shows an example code. 

```java
public class SampleRoomMatchInfo extends AbstractRoomMatchInfo {
    private static final int MAX_ENTRY_USER = 4;

    public SampleRoomMatchInfo(int roomId) {
        super(roomId, MAX_ENTRY_USER);
    }
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
/**
* Call when creating a new room
* <p/>
* Determine whether to create a room based on the return value
*
* @param user The requested user object
* @param inPayload The {@link IPayload} passed from the client
* @param outPayload The {@link IPayload} passed to the client
* @return If the return value is true, the room creation was successful; if false, it failed.
*/
@Override
public final boolean onCreateRoom(final GameUser user, final IPayload payload, IPayload outPayload) {

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
gameRoomMatchInfo.setMemberMoney(1000);  
  
updateRoomMatch(gameRoomMatchInfo); // Update this room matching information. 
```

When the room disappears, the room matching information is automatically deleted from the engine, so you don't need to delete it separately.

### Room matchmaker

Now it's time to create a room matchmaker. The room matchmaker inherits BaseRoomMatchMaker abstract class provided by engine. Room matchmaking is the process of finding the most suitable room, so special callback methods are provided for before and after the actual match. Users can override these callback methods to perform any match they want. The example code below shows how these room matchmakers can be implemented.  

```java
@GameAnvilUserMatchMaker(loadClass = SampleRoom.class) // MatchMaker class registered on SampleRoom
public class SampleRoomMatchMaker extends AbstractRoomMatchMaker<SampleRoomMatchForm, SampleRoomMatchInfo> {
    /**
    * Called when requesting room matchmaking
    * <p>
    * Finds rooms that meet matching conditions among registered rooms and determines whether the match is successful.
    *
    * @param baseRoomMatchForm Room matchmaking request conditions {@link AbstractRoomMatchForm}
    * @param baseRoomMatchInfo Room information in the matching pool {@link AbstractRoomMatchInfo}
    * @param args Additional parameters passed
    * @return If the return value is true, room matchmaking is successful; if false, room matchmaking is unsuccessful.
     */
    @Override
    public boolean onMatch(SampleRoomMatchForm roomMatchForm, SampleRoomMatchInfo roomMatchInfo, Object... args) {
        return false;
    }

    /**
     * Compare matchmaking information for sorting
     * <p>
     * {@link com.nhn.gameanvil.node.game.context.IRoomContext#updateChannelRoomInfo(IChannelRoomInfo)} The matching pool is sorted by the implemented compare when calling methods, registering rooms, deleting rooms, or calling sorting methods.
    *
    * @param o1: The first parameter to compare.
    * @param o2: The second parameter to compare.
    * @return: The comparison value (-1: ascending, 0: no change, 1: descending).
     */
    @Override
    public int compare(SampleRoomMatchInfo o1, SampleRoomMatchInfo o2) {
        return 0;
    }
}
```

Specifically, a class implementing a matchmaker registers itself with a specific service's engine. It also predefines the type of room to be created by the matchmaker. Note that a matchmaker class can only be registered with one service.

The following table summarizes the room matchmaker's callback methods explained earlier:

| Callback name           | Meaning                              | Description                                                                                                                                                    |
| ------------------- | --------------------------------- |-------------------------------------------------------------------------------------------------------------------------------------------------------|
| onMatch | Check Match | Called to check if a match is possible between the room match request and the room match information passed as arguments. This function can implement logic here to iterate through the entire room match pool and find the most suitable room for the room match request. |
| compare | Compare matchmaking information for sorting | Compares matchmaking information for sorting. The result value is -1: ascending, 0: no change, or 1: descending. |


### Send requests from GameUser to matchmaker

Client can now request a room matchmaking from server. This request is forwarded to GameUser and then the engine calls onMatchRoom callback method. We have shared about this earlier while explaining GameNode and GameUser. Users can use the room matchmaker provided by GameAnvil in this callback method, or use a separate matchmaker or other solution they have implemented themselves. However, unless there is a specific reason, we recommend using GameAnvil's room matchmaking.

The example below shows a call to matchRoom API by creating a match request that was previously discussed in order to use the room matchmaker provided by engine. In particular, we explained that the matching group forwarded as parameters is a concept that can logically divide matchmaking. If you forwarded a matching group named "Newbie," the same "Newbie" matching group would share the same matching queue.

```java
    /**  
     * Call if you receive a room matchmaking request 
     *  
     * @param roomType Any value that separates the pre-defined room types between client and server 
     * @param matchingGroup Matching group requested by client  
     * @param matchingUserCategory categories within the room to which the user belongs to  
     * @param payload  Payload delivered from clients  
     * @return Return the information of the room matched with the @return {@link RoomMatchResult} type. If you return null, new rooms are created according to the client request option or the request fails
     */
    @Override
    public RoomMatchResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final IPayload payload) {

    SampleRoomMatchForm sampleRoomMatchForm = new SampleRoomMatchForm(RoomMode.NORMAL, innerPayload.getOption(), 0);

    return userContext.matchRoom(matchingGroup, roomType, sampleRoomMatchForm);
}
```

