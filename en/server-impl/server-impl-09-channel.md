## Game > GameAnvil > Server Development Guide > Channel

## Channel

![channel-sync2.png](https://static.toastoven.net/prod_gameanvil/images/channel-sync2.png)

Channels are a way to logically divide a single server cluster. GameAnvil can set up channels when it contains one or more game nodes. By default, GameAnvilConfig allows you to set channels on game nodes as shown in the example below. In this example, we set ch1, ch1, ch2, and ch2 for the four game nodes.

```json
"game": [
  {
    "serviceId": 1, // Must be unique within the server group [0 to 99]
    "serviceName": "MyGame", // Must be unique within the server group
    "nodeCnt": 4, // Number of nodes. If nodeCnt is 0, no game nodes will be created. It is recommended to enter the number of machine cores [0 to 99]
    "channelIDs": [
      ["Beginner"], 
      ["Beginner"],
      ["Expert"],
      ["Expert"]
    ], // Channel ID to be assigned to each node. (Does not have to be unique. Multiple "" strings can be used to distinguish channels)
    "userTimeout": 5000       // Timeout for user object removal after disconnect. [0 to 604800000 (7 days)]
  }
],
```

If you don't want to use channels, you can set all of the channel IDs to "", as shown below. In this case, all channel features are also disabled, meaning that all four Game Nodes below will work independently of the channel.

```json
"game": [
  {
    "serviceId": 1, // Must be unique within the server group [0 ~ 99]
    "serviceName": "MyGame", // Must be unique within the server group
    "nodeCnt": 4, // Number of nodes. If nodeCnt is 0, no Game Nodes will be created. It is recommended to use the number of machine cores [0 ~ 99]
    "channelIDs": [
    [""],
    [""],
    [""],
    [""]
    ], // Channel IDs to assign to each node. (Does not have to be unique. Multiple "" strings can be used regardless of channel.)
    "userTimeout": 5000 // Timeout for user object deletion after disconnect. [0 ~ 604800000 (7 days)]
  }
],
```
If you want to manage all game nodes as one channel, enter the channel ID as a valid string as follows:

```json
"game": [
  {
    "serviceId": 1, // Must be unique within the server group [0 ~ 99]
    "serviceName": "MyGame", // Must be unique within the server group
    "nodeCnt": 4, // Number of nodes. If nodeCnt is 0, no Game Nodes will be created. It is recommended to use the number of machine cores [0 ~ 99]
    "channelIDs": [
    ["Beginner"],
    ["Beginner"],
    ["Beginner"],
    ["Beginner"]
    ], // Channel IDs to assign to each node. (Does not have to be unique. Multiple "" strings can be used regardless of channel.)
    "userTimeout": 5000 // Timeout for user object deletion after disconnect. [0 ~ 604800000 (7 days)]
  }
],
```

By default, one game node is set per thread, but for efficient resource utilization, multiple game nodes can be set per thread as follows. The total number of game nodes is not 4 specified by nodeCnt, but actually 3 (ignoring the nodeCnt value) set in the channel information. There are 2 game node threads, divided into 2 groups. Thread 1 is set with game node 1 ("beginner") and game node 2 ("intermediate"), and thread 2 is set with game node 3 ("expert").
```json
"game": [
  {
    "serviceId": 1,          // Must be unique within the server group [0 ~ 99]
    "serviceName": "MyGame", // Must be unique within the server group
    "nodeCnt": 4,          // Number of nodes. If nodeCnt is 0, no game nodes will be created. It is recommended to use the number of machine cores [0 ~ 99]
        "channelIDs": [
    ["Beginner", "Intermediate"], // Thread 1
    ["Expert"]          // Thread 2
    ],          // Channel IDs to assign to each node. (Does not have to be unique. Multiple "" strings can be used regardless of channel.)
    "userTimeout": 5000          // Timeout for user object deletion after disconnect. [0 ~ 604800000 (7 days)]
  }
],
```

These channels allow you to:

* You can manage channel information. Information about users and rooms that belong to that channel becomes available.
* You can view the number of users and rooms per channel.
* You can send messages on a per-channel basis. Use the publishToChannel API to deliver messages to all game nodes that belong to the target channel.

## Manage Channel Information

Users can implement their own information to be managed in a channel. This information is automatically synchronized within the same channel.

### Channel User Information

First, to manage user information in a channel, you need to enable channel user information through an annotation when implementing the user class with useChannelInfo settings, as shown below:
```java
@GameAnvilUser(
    gameServiceName = "MyGame", // The node to which the user will belong (the same service name as SampleGameNode above)
    gameType = "BasicUser", // A unique user type. Registers a user of the "BasicUser" user type in the engine.
    useChannelInfo = true // Enables information synchronization between channels.
)
public class SampleGameUser implements IUser {
    // ... 
}
```

And further implement IChannelUserInfo. This can contain any information that the user wants to manage in the channel.

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
     * Return user ID
     *
     * @return Return user ID
     */
    @Override
    public int getUserId() {
        return userId;
    }

    /**
     * Return account ID
     *
     * @return Return account ID
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

The channel user information you created can then be added or updated in the user object as follows. To do so, use the userContext.updateChannelUserInfo() API. If the user object is logged out of the server, the corresponding channel user information is automatically removed as well. The following is the pseudo code for this:

```java
public class SampleGameUser implements IUser {
    GameChannelUserInfo channelUserInfo = new GameChannelUserInfo();

	...

    // Add channel user information when the user logs in.
    onLogin(...) {
		...
        channelUserInfo.setUserId(getId())
        channelUserInfo.setAccountId(getAccountId());
        channelUserInfo.setLevel(getLevel());

        userContext.updateChannelUserInfo(channelUserInfo);
		...
    }

    // When moving a channel, new channel user information is added to the target channel.
    onMoveInChannel(...) {
		...
        channelUserInfo.setUserId(getId());
        channelUserInfo.setAccountId(getAccountId());
        channelUserInfo.setLevel(getLeve());

        userContext.updateChannelUserInfo(channelUserInfo);
		...
    }

    // You can update your user information at any time by changing your user content.
    updateLevel(int level) {
        channelUserInfo.setLevel(getLevel());

        userContext.updateChannelUserInfo(channelUserInfo);
    }
}
```

Note that when you move channels, the old channel automatically deletes user information from that channel, so you only need to worry about adding new information on the destination channel.

### Channel Room Information

To manage room information in a channel, set useChannelInfo to true when implementing the room class, just like the game user we looked at earlier.
```java
 @GameAnvilRoom(
    gameServiceName = "MyGame", // The node to which the room will belong (the same service name as SampleGameNode above)
    gameType = "BasicRoom", // The room type is unique. Register a room of type "BasicRoom" with the engine.
    useChannelInfo = true // Sets up information synchronization between channels.
)
public class SampleRoom implements IRoom<SampleUser> {
    // ...
}
```

And further implement BaseChannelRoomInfo. This is where you include all the information about the rooms you want to manage in the channel.

```java
public class GameChannelRoomInfo implements IChannelRoomInfo, Comparable<GameChannelRoomInfo> {

    public static final int MAX_ENTRY_USER = 4;
    private int roomId = 0;

    private long allUserMoney; // for room sorting comparator.


    /**
     * Return room ID
     *
     * @return Return room ID
     */
    @Override
    public int getRoomId() {
        return roomId;
    }

    /**
     * Copy room information
     *
     * @return Return the copied room information
     * @throws CloneNotSupportedException Occurred if copying fails
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

The channel room information you created can then be added or updated in the room object as follows. To do so, use the updateChannelRoomInfo API. If the room object disappears from the server, the corresponding channel room information is automatically removed as well. The following is the pseudo code for this:

```java
public class SampleGameRoom implements IRoom<SampleGameUser> {
    GameChannelRoomInfo channelRoomInfo = new GameChannelRoomInfo();

	...

    // When creating a room, add channel room information through the updateChannelRoomInfo function.
    onCreateRoom(...) {
		...
        channelRoomInfo.setRoomId(getId());
        channelRoomInfo.setRoomName(getRoomName());
        channelRoomInfo.setUserCount(0);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }

    // When a new user enters the room, the changed room information is applied through the updateChannelRoomInfo function.
    onJoinRoom(...) {
		...
        channelRoomInfo.setUserCount(++userCount);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }

    // When a user leaves a room, the changed room information is applied through the updateChannelRoomInfo function.
    onPostLeaveRoom(...) {
		...
        channelRoomInfo.setUserCount(--userCount);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }
}
```

## Synchronize Channel Information

Game nodes in the same channel share channel-related information with each other. For example, when a user or room information changes on one Game Node in the same channel in the manner described above, the following callback methods are called on the remaining Game Nodes in that channel. These callbacks allow all Game Nodes within the same channel to synchronize their information. The following callback methods are used by Game Nodes to synchronize these channels.

```java
/**
 * Call when user changes occur to other nodes in the same channel
 * <p>
 * Call when calling updateChannelUser ().
 *
 * @param type            {@link ChannelUpdateType}, which is the channel information change type (update/delete)
 * @param channelUserInfo {@link IChannelUserInfo}, the user information to be changed
 * @param userId          User ID of the target for change
 * @param accountId       Account ID to be changed
 */
@Override
public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {
}

/**
 * Call when room status changes occur in other nodes in the same channel
 * <p>
 * Occurred by calling updateChannelRoomInfo()
 *
 * @param type            Channel information change type (update/delete) {@link ChannelUpdateType}
 * @param channelRoomInfo {@link IChannelRoomInfo}, the room information to be changed
 * @param roomId          Room ID to be changed
 */
@Override
public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {
}

/**
 * Call when the client requests channel information   (Base.GetChannelInfoReq)
 *
 * Channel information to be sent to the @param outPayload client
 */
@Override
public void onChannelInfo(IPayload payload) {
}
```

### Synchronize Channel Information with Client

The client can request channel information from the server at any time. To do so, onChannelInfo is called from the Game Node's callback method, which we discussed earlier. However, to prevent incorrect implementation or malicious use by clients, this callback method call has a minimal re-invocation period (1 second by default).  For example, if a client makes 10 requests for channel information in 1 second, the server will only call the onChannelInfo callback method once. The other 9 requests will pass information that it has previously cached. Here is some pseudo code that implements such an onChannelInfo.

```java
public void onChannelInfo(Payload outPayload) {

    // Create custom channel information to send to clients
    Game.GameChannelInfo.Builder channelInfoBuilder = Game.GameChannelInfo.newBuilder();

    channelInfoBuilder.setChannelId(gameNodeContext.getChannelId());

    // Get channel user information through the getChannelUserInfo API.
    for (IChannelUserInfo channelUserInfo : getContext().getChannelUserInfo(StringValues.userType)) {
        Game.GameChannelInfo.Builder channelUserInfoBuilder = Game.GameChannelInfo.newBuilder();
        channelUserInfoBuilder.setAccountId(channelUserInfo.getAccountId());
        channelUserInfoBuilder.setUserId(channelUserInfo.getUserId());

        channelInfoBuilder.addChannelUserInfo(channelUserInfoBuilder.build());
    }

    // Get channel room information through the getChannelRoomInfo API.
    for (IChannelRoomInfo channelRoomInfo : getContext().getChannelRoomInfo(StringValues.roomType_2_User_Match)) {
        Game.GameChannelRoomInfo.Builder channelRoomInfoBuilder = Game.GameChannelRoomInfo.newBuilder();
        channelRoomInfoBuilder.setRoomId(channelRoomInfo.getRoomId());
        channelRoomInfoBuilder.setUserCount(((GameChannelRoomInfo)channelRoomInfo).getUserCnt());

        channelInfoBuilder.addChannelRoomInfo(channelRoomInfoBuilder.build());
    }

    // Add channel information to be sent to the client in outPayload.
    outPayload.add(channelInfoBuilder);
}
```

### Pass the Number of Users and Rooms in the Channel to the Client

The GameAnvil connector provides the GetChannelCountInfo API to request this information. The engine always manages the number of users/rooms per channel, so you don't need to implement anything.