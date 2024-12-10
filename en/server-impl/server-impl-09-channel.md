## Game > GameAnvil > Server Development Guide > Channel



## Channel

![channel-sync2.png](https://static.toastoven.net/prod_gameanvil/images/channel-sync2.png)

Channels are a way to logically divide a single server cluster. GameAnvil can set up channels when it contains one or more game nodes. By default, GameAnvilConfig allows you to set channels on game nodes as shown in the example below. In this example, we set ch1, ch1, ch2, and ch2 for the four game nodes.

```json
"game": 
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

If you don't want to use channels, you can set all of the channel IDs to "", as shown below. In this case, all channel features are also disabled, meaning that all four Game Nodes below will work independently of the channel.

```json
"game": 
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

If you want to manage all of your game nodes as a single channel, enter the channel ID as a valid string as shown below.

```json
"game": 
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

The following features are available through these channels

* You can manage channel information. Information about users and rooms that belong to that channel becomes available.
* You can view the number of users and rooms per channel.
* You can send messages on a per-channel basis. Use the publishToChannel API to deliver messages to all game nodes that belong to the target channel.



## Manage Channel Information

Users can implement their own information to be managed in a channel. This information is automatically synchronized within the same channel.

### Channel User Information

First, to manage user information in a channel, you need to enable channel user information through an annotation when implementing the user class, like this

```java
@ServiceName("MyGame")
@UserType("BasicUser")
@UseChannelInfo // Enable channel user information
public class SampleGameUser extends BaseUser implements TimerHandler {
	...
}
```

And further implement BaseChannelUserInfo. This can contain any information that the user wants to manage in the channel.

```java
public class GameChannelUserInfo implements Serializable, BaseChannelUserInfo {
	// User information to display in the channel
	private int userId = 0;
	private String accountId = "";
	private int level = 0;

    ...
  
    /**
     * User Id of the channel user information to be changed.
     * ...
     * @return int type to return UserId.
     */
    @Override
    int getUserId() {
        return userId;
    }

    /**
     * Get the Account Id of the channel user information to be changed.
     * * @return
     * @return AccountId as String type.
     */
    @Override
    String getAccountId() {
        return accountId;
    }
}
```

The channel user information you created can then be added or updated in the user object as follows. To do so, use the updateChannelUserInfo API. If the user object is logged out of the server, the corresponding channel user information is automatically removed as well. The following is the pseudo code for this.

```java
public class SampleGameUser extends BaseUser {
	GameChannelUserInfo channelUserInfo = new GameChannelUserInfo();
  
	...

	// Add channel user information when the user logs in.
	onLogin(...) throws SuspendExecution {
		...
		channelUserInfo.setUserId(getId())
		channelUserInfo.setAccountId(getAccountId());
		channelUserInfo.setLevel(getLevel());
      
		updateChannelUserInfo(channelUserInfo);
		...
	}
  
	// Adds new channel user info to the target channel on channel move.
	onMoveInChannel(...) throws SuspendExecution {
		...
		channelUserInfo.setUserId(getId());
		channelUserInfo.setAccountId(getAccountId());
		channelUserInfo.setLevel(getLeve());
      
		updateChannelUserInfo(channelUserInfo);
		...
	}
  
	// Update user information that has changed in the user content at any time.
	updateLevel(int level) {
		channelUserInfo.setLevel(getLevel());
      
		updateChannelUserInfo(channelUserInfo);
	}
}
```

Note that when you move channels, the old channel automatically deletes user information from that channel, so you only need to worry about adding new information on the destination channel.

### Channel Room Information

To manage room information in a channel, you need to enable channel room information through an annotation when implementing the room class, just like the game user we saw earlier.

```java
@ServiceName("MyGame")
@RoomType("BasicRoom")
@UseChannelInfo // Enable channel information
public class GameRoom extends BaseUser implements TimerHandler {
    ...
}
```

And further implement BaseChannelRoomInfo. This is where you include all the information about the rooms you want to manage in the channel.

```java
public class GameChannelRoomInfo implements Serializable, BaseChannelRoomInfo {
	private int roomId = 0;
	private String roomName = "";
	private int userCount = 0;

    ...
      
    /**
     * Room Id for room information.
     * ...
     * @return int type to return RoomId.
     */
    @Override
    public int getRoomId() {
        return roomId;
    }

    /**
     * Copy the room information.
     * * * * * * RoomInfo
     * @return RoomInfo Returns the copied Room information.
     * * @throws CloneNotSupportedException If cloning is not supported.
     */
    @Override
    public BaseChannelRoomInfo copy() throws CloneNotSupportedException {
        GameChannelRoomInfo channelRoomInfo = (GameChannelRoomInfo)super.clone();
      
        // If you need to copy the data, do it here.
      
        return channelRoomInfo;
    } 
}
```

The channel room information you created can then be added or updated in the room object as follows. To do so, use the updateChannelRoomInfo API. If the room object disappears from the server, the corresponding channel room information is automatically removed as well. The following is the pseudo code for this.

```java
public class GameRoom extends BaseRoom<GameUser> {
	GameChannelRoomInfo channelRoomInfo = new GameChannelRoomInfo();
  
	...
      
	// Add the channel room information via the updateChannelRoomInfo function when the room is created.
	onCreateRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setRoomId(getId());
		channelRoomInfo.setRoomName(getRoomName());
		channelRoomInfo.setUserCount(0);
      
		updateChannelRoomInfo(channelUserInfo);
	}
  
	// When a new user joins the room, we apply the changed room information through the updateChannelRoomInfo function.
	onJoinRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setUserCount(++userCount);
      
		updateChannelRoomInfo(channelUserInfo);
	}
  
    // When a user leaves the room, apply the changed room information via the updateChannelRoomInfo function.
	onPostLeaveRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setUserCount(--userCount);
      
		updateChannelRoomInfo(channelUserInfo);
	}
}
```

## Synchronize Channel Information

Game nodes in the same channel share channel-related information with each other. For example, when a user or room information changes on one Game Node in the same channel in the manner described above, the following callback methods are called on the remaining Game Nodes in that channel. These callbacks allow all Game Nodes within the same channel to synchronize their information. The following callback methods are used by Game Nodes to synchronize these channels.

```java
 	/**
     * Called when a user change occurs on another node in the same channel.
     * i.e., when calling the updateChannelUser() API
     * * Parameters.
     * @param type Channel information to be changed type (update/delete).
     * @param channelUserInfo Passes the User information to be changed.
     * @param userId Passes the User Id to be changed.
     * @param accountId Account Id to be changed.
     * @throws SuspendExecution This method can suspend the fiber.
     */
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution { 
    }

    /**.
     * Called when a room state change occurs on another node in the same channel.
     * i.e., when the updateChannelRoomInfo() API is called.
     *]
     * @param type Passing the type of Channel information change (update/delete).
     * * @param channelRoomInfo Passing the Room information to be changed.
     * @param roomId Passes in the Room Id to be changed.
     * * @throws SuspendExecution This method allows the fiber to be suspended.
     */
    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType type, ChannelRoomInfo channelRoomInfo, final int roomId) throws SuspendExecution { 
    }

    /**.
     * Called when the client requests channel information.
     * * * @param outPayload
     * @param outPayload Channel information to be passed to the Client.
     * * @throws SuspendExecution This method can be used to suspend the fiber.
     */
    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution { 
    }
```

### Synchronize Channel Information with Client

The client can request channel information from the server at any time. To do so, onChannelInfo is called from the Game Node's callback method, which we discussed earlier. However, to prevent incorrect implementation or malicious use by clients, this callback method call has a minimal re-invocation period (1 second by default).  For example, if a client makes 10 requests for channel information in 1 second, the server will only call the onChannelInfo callback method once. The other 9 requests will pass information that it has previously cached. Here is some pseudo code that implements such an onChannelInfo.

```java
public void onChannelInfo(Payload outPayload) throws SuspendExecution {

    // Create custom channel information to send to the client
	Game.GameChannelInfo.Builder channelInfoBuilder = Game.GameChannelInfo.newBuilder();

	channelInfoBuilder.setChannelId(this.getChannelId());

	// Get the channel user information via the getChannelUserInfo API.
	for (BaseChannelUserInfo channelUserInfo : getChannelUserInfo(UserType_1)) {
		GameChannelUserInfo gameChannelUserInfo = (GameChannelUserInfo) channelUserInfo;

        Game.GameChannelUserInfo.Builder channelUserInfoBuilder = Game.GameChannelUserInfo.newBuilder();
        channelUserInfoBuilder.setUserName(gameChannelUserInfo.getUserName());
        channelUserInfoBuilder.setLevel(gameChannelUserInfo.getlevel());

    	channelInfoBuilder.addChannelUserInfos(channelUserInfoBuilder.build());
    }
  
	// Get the channel room information via the getChannelRoomInfo API.
    for (BaseChannelRoomInfo channelRoomInfo : getChannelRoomInfo(RoomType_2)) { 
	   	GameChannelRoomInfo gameChannelRoomInfo = (GameChannelRoomInfo) channelRoomInfo;
  
    	Game.GameChannelRoomInfo.Builder channelRoomInfoBuilder = Game.GameChannelRoomInfo.newBuilder();
        channelRoomInfoBuilder.setRoomId(gameChannelRoomInfo.getRoomId());
        channelRoomInfoBuilder.setRoomName(gameChannelRoomInfo.getRoomName());
        channelRoomInfoBuilder.setUserCount(gameChannelRoomInfo.getUserCnt());

        channelInfoBuilder.addChannelRoomInfos(channelRoomInfoBuilder.build());
	}

    // Add the channel information to be sent to the client to the outPayload.
	outPayload.add(channelInfoBuilder);
}
```

### Pass the Number of Users and Rooms in the Channel to the Client

The GameAnvil connector provides the GetChannelCountInfo API to request this information. The engine always manages the number of users/rooms per channel, so you don't need to implement anything.
