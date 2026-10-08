<!-- machine_translated: true -->

<!-- pre-align:aligned sig=b2564be6b47a -->

<a id="game-gameanvil-server-development-guide-implement-game-node"></a>

## Game > GameAnvil > Server Development Guide > Implement Game Node { #game-gameanvil-server-development-guide-implement-game-node }

<a id="game-node"></a>

## Game Node { #game-node }

![GameNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gamenode_on_network.png)

GameNode is a node where real game objects are created and game content is implemented. After the client has authenticated to GatewayNode, the client must log in to GameNode to get started with such game content.

As shown in the image below, the client can create multiple logic sessions based on connections that have been authenticated with a unique AccountId. At this time, the session is created between the client and the user object. In the image below, a red arrow indicates these sessions:

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

Each session can be separated by a unique value within the connection, and we call this value SubId. It means that with a combination of AccountId and SubId, users can create as many sessions as they want. At this time, you can also create as many sessions for the same service as you want. The connection with the AccountId 1 in the image can create any additional sessions for the "Game" service or "Chat" service.

This session is directed to the user object. GameNode manages these user objects and room objects with their groups. This chapter covers these GameNodes, GameUser, and GameRoom.

<a id="implement-gamenode"></a>

## Implement GameNode { #implement-gamenode }

GameNode implements the BaseGameNode class. The example code below shows the callback methods that can be basically redefined in GameNode. A callback exists for channel management, along with a node common callback.

```java
@GameAnvilGameNode(gameServiceName = "MyGame") // (1) Register in Engine with GameNode for a service called "MyGame"
public class SampleGameNode extends BaseGameNode {

    /**
     * Call when user changes occur to other nodes in the same channel
     * <p>
     * Occurs when updateChannelUser() is called.
     *
     * @param type            {@link ChannelUpdateType}, which is the channel information change type (update/delete)
     * @param channelUserInfo {@link IChannelUserInfo}, the user information to be changed
     * @param userId          User ID of the target to be changed
     * @param accountId       Account ID of the target to be changed
     */
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {

    }

    /**
     * Call when room status changes occur in other nodes in the same channel
     * <p>
     * Occurs when updateChannelRoomInfo() is called
     *
     * @param type            Channel information change type (updated/deleted) {@link ChannelUpdateType}
     * @param channelRoomInfo {@link IChannelRoomInfo}, the room information to be changed
     * @param roomId          Room ID of the target to be changed
     */
    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {

    }

    /**
     * Called when the client requests channel information (Base.GetChannelInfoReq)
     *
     * @param outPayload Channel information to be delivered to the client
     */
    @Override
    public void onChannelInfo(IPayload payload) {

    }

    /**
     * Called when the node is initialized
     */
    @Override
    public void onInit() {

    }

    /**
     * Called to handle tasks before the node becomes Ready
     */
    @Override
    public void onPrepare() {

    }

    /**
     * Called when the node becomes Ready
     */
    @Override
    public void onReady() {

    }

    /**
     * Called when the node is Paused
     *
     * @param payload Additional information to send from the content
     */
    @Override
    public void onPause(IPayload payload) {

    }

    /**
     * Called when a Shutdown command is received
     */
    @Override
    public void onShuttingdown() {

    }

    /**
     * Called when the node is Resumed
     *
     * @param payload Additional information to send from the content
     */
    @Override
    public void onResume(IPayload payload) {

    }
}
```

```java
// A message processing class that executes when input from the protobuffer MyGame.GameNodeTest is received
@GameAnvilController
public class _GameNodeTest {
    // (2) Mapping the protocol and handler we want to handle in SampleGameNode
    @GameNodeMapping(
        value = MyGame.GameNodeTest.class, // Protobuffer to process
        loadClass = SampleGameNode.class   // The message receiver (SampleGameNode)
    )
    public void execute(IGameNodeDispatchContext ctx) {
        // Write the task to be performed here
    }
}
```

For the meaning and usage of callbacks, see the table below:

| Callback Name | Meaning | Description |
|-------------------------|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onChannelUserInfoUpdate | Update channel user information | Some of the GameNodes associated with the same channel are called for synchronization by all others in the same channel when channel user information is changed in one GameNode. At this time, the user can update the current channel information in GameNode based on the information you have received. |
| onChannelRoomInfoUpdate | Update channel room information | Many of the GameNodes associated with the same channel are called for synchronization by all the others in the same channel when channel room information is changed in one GameNode. At this time, the user can update the current channel information in GameNode based on the information you have received.  |
| onChannelInfo | Request Channel Information | Calls are made when the client requests channel information. In this callback, the user can configure channel information to pass to the client as they want. |
| onInit | Initialize | Calls when the node proceeds with the first initialization. If there is any initialization task required before the node runs, this callback is suitable. At this time, the node is still not being processing the message. |
| onPrepare | Prepare | The node will be called after the node is initialized. The user can proceed with any task here before the node is ready. At this time, the node can process messages. |
| onReady | Prepared | After all preparations are completed, the node is running. At this time, the node is in Ready state, so the user will be able to use all features from this time. |
| onPause | Pause | Pause the node will call the call. Here you can implement the code you want to process additionally when the node is temporarily stopped. |
| onShuttingdown | Node stopped | Calls are sent when the node receives a Shutdown command. Stopped nodes cannot be Resume. |
| onResume | Resume | Calls are called when the node restarts the run while it is temporarily stopped. Here, the user can implement the code he wants to process while in a restart. |

All nodes need a message handler registration process to process custom messages. In particular, this process is required for GameNode game content. Proceed with the setting up in the main class before the server runs. (1) Create the game service name set for GameAnvilConfig. The game service name must use the name defined in GameAnvilConfig. (2) And connect to the [translator](server-impl-07-message-handling.md#implement-message-handler-and-connect-messages-to-handlers) that has implemented the message you want to process. One GameNode class can only be registered for one service.

The primary purpose of these GameNodes is to perform processing for all GameUser and GameRoom objects connected to the node, which we'll discuss in a moment.

<a id="implement-user"></a>

## Implement User { #implement-user }

User objects are created in GameNode through the login process. User-based content must be implemented with this class in the center. As with all the examples discussed earlier, users can also connect their own message and manipulator to handle. From the example code below, you can see that the user offers a lot of callback methods. Some of these features are provided with default implementation, so you should not redefine them unless they are especially necessary. This corresponds to most classes provided not only by users but also by engines.

```java
@GameAnvilUser(
    gameServiceName = "MyGame", // Node to which the user belongs (service name such as SampleGameNode above)
    gameType = "BasicUser",     // Register a user of a unique user type, "BasicUser" in the engine
    useChannelInfo = true       // Setting up synchronization of information between channels
)
public class SampleGameUser extends BaseGameUser {

    /**
     * Called when logging in
     *
     * @param payload        {@link IPayload} received from the client
     * @param sessionPayload {@link IPayload} passed from onBeforeLogin
     * @param outPayload     {@link IPayload} to be sent to the client
     * @return If the return value is true, login succeeds; if false, login fails
     */
    @Override
    public boolean onLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        return false;
    }

    /**
     * Called for post-processing required after a successful login
     * <p></p>
     * (That is, called after onLogin or onReLogin succeeds)
     *
     * @param isRelogined Whether this is a re-login
     */
    @Override
    public void onAfterLogin(boolean isRelogined) {

    }

    /**
     * Called when a login is attempted again while already logged in
     * <p></p>
     * Even if the connection is lost while logged in, the user's game user object remains valid on the game node for a certain period of time
     *
     * @param payload        Arbitrary {@link IPayload} passed from the client
     * @param sessionPayload {@link IPayload} passed from onBeforeLogin
     * @param outPayload     Arbitrary {@link IPayload} to be sent to the client
     * @return If the return value is true, ReLogin succeeds; if false, ReLogin fails
     */
    @Override
    public boolean onReLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        return false;
    }

    /**
     * Callback called when the connection with the client is lost
     */
    @Override
    public void onDisconnect() {

    }

    /**
     * Called when the node to which the user belongs is paused, causing the user to be paused as well
     */
    @Override
    public void onPause() {

    }

    /**
     * Called when the node to which the user belongs is resumed, causing the user to be resumed as well
     */
    @Override
    public void onResume() {

    }

    /**
     * Called when the user logs out
     *
     * @param payload    {@link IPayload} received from the client
     * @param outPayload {@link IPayload} to be sent to the client
     */
    @Override
    public void onLogout(IPayload payload, IPayload outPayload) {

    }

    /**
     * Called to check whether the user can log out
     * <p></p>
     * Engine users can decide in this callback whether it is safe for the current game user to log out
     *
     * @return If the return value is false, the logout process stops and the callback is called again periodically thereafter. If the return value is true, logout proceeds
     */
    @Override
    public boolean canLogout() {
        return false;
    }

    /**
     * Called after onLeavingRoom of the room is executed and the user has completely left the room
     * <p></p>
     * Performs tasks that the user who has left the room needs to handle
     */
    @Override
    public void onAfterLeaveRoom() {

    }

    /**
     * Called to check whether the user can be transferred (moved) to another node
     *
     * @return If the return value is true, transfer is possible; if false, transfer is not possible. In the unavailable state, if SafePause is in progress, continue to call to send the user until SafePause ends.
     */
    @Override
    public boolean canTransfer() {
        return false;
    }

    /**
     * Called when the same user logs in from a different device while already logged in
     *
     * @param newDeviceId           Device ID of the newly connected user
     * @param outPayloadForKickUser {@link IPayload} to be sent to the client. Includes kickOut or LoginRes information
     * @return If the return value is true, the existing user will be forced to log out after the newly connected user is logged in. If false, the newly connected user will fail to log in.
     */
    @Override
    public boolean onLoginByOtherDevice(String newDeviceId, IPayload outPayloadForKickUser) {
        return false;
    }

    /**
     * Called when a login is attempted with a different user type while already logged in with an arbitrary user type
     *
     * @param userType   The type of the user attempting the new login
     * @param outPayload {@link IPayload} to be sent to the client
     * @return If the return value is true, the new login succeeds; if false, it fails
     */
    @Override
    public boolean onLoginByOtherUserType(String userType, IPayload outPayload) {
        return false;
    }

    /**
     * Called when a login is attempted through another connection (e.g., due to reconnection) while already logged in
     *
     * @param outPayload {@link IPayload} to be sent to the client
     */
    @Override
    public void onLoginByOtherConnection(IPayload outPayload) {

    }

    /**
     * Called when a room matchmaking request is received
     *
     * @param roomType             An arbitrary value that distinguishes the room type predefined between the client and the server
     * @param matchingGroup        The room matching group to be matched
     * @param matchingUserCategory The matching user category to be matched
     * @param payload              {@link IPayload} received from the client
     * @return Return the information of the room matched with the {@link RoomMatchResult} type. If you return null, new rooms are created according to the client request option or the request fails
     */
    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        return null;
    }

    /**
     * Called when processing the client's room matchmaking request fails
     *
     * @param matchRoomFailCode The reason why room matchmaking failed
     */
    @Override
    public void onMatchRoomFail(MatchRoomFailCode matchRoomFailCode) {

    }

    /**
     * Called when processing the client's user matchmaking request fails
     *
     * @param matchUserFailCode The reason why user matchmaking failed
     */
    @Override
    public void onMatchUserFail(MatchUserFailCode matchUserFailCode) {

    }

    /**
     * Callback called when the client requests user matchmaking
     *
     * @param roomType      An arbitrary value that distinguishes the room type predefined between the client and the server
     * @param matchingGroup The matching group
     * @param payload       {@link IPayload} received from the client
     * @param outPayload    {@link IPayload} to be sent to the client
     * @return If the return value is true, the user matchmaking request succeeds; if false, it fails
     */
    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        return false;
    }

    /**
     * Called when user matchmaking is canceled
     *
     * @param reason The reason for cancellation: timeout (TIMEOUT), canceled by the user's request (CANCEL), or canceled due to the match node shutting down (SHUTDOWN)
     */
    @Override
    public void onMatchUserCancel(MatchCancelReason reason) {

    }

    /**
     * Called to package the data to be sent from the source node when the user is transferred (moved) to another node
     *
     * @param transferPack A data package for storing data to be taken to another node
     */
    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    /**
     * When the user has been moved (transferred) from another node, calls to transfer data to the destination node, and the timer key to be re-registered
     * <p>
     * Use TimerHandlerTransferPack to check the list of timerHandlerKeys registered to the user
     * <p></p>
     * Use reRegister() of TimerHandlerTransferPack to re-register the timerHandler to be used
     *
     * @param transferPack             A data package for delivering data brought from another node
     * @param timerHandlerTransferPack A data package for delivering timer information brought from another node
     */
    @Override
    public void onTransferIn(IReadOnlyTransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    /**
     * Called after the transfer is complete when the user has been transferred (moved) from another node
     */
    @Override
    public void onAfterTransferIn() {

    }

    /**
     * Called when the client requests a snapshot
     * <p>
     * Used to synchronize client and server status information when the connection is mainly disconnected and the server status is likely to change.
     *
     * @param payload    {@link IPayload} with defines set, passed from the client to the server
     * @param outPayload {@link IPayload} with defines set, passed from the server to the client
     */
    @Override
    public void onSnapshot(IPayload payload, IPayload outPayload) {

    }

    /**
     * When making a request to move from the client to another channel, called to confirm whether the user is currently in a channel-moveable state
     * <p>
     * Caution! If the user explicitly calls the moveChannel() API to move a channel, canMoveOutChannel() is not called
     * <p></p>
     * It is called only when an implicit channel move occurs by the engine
     *
     * @param destinationChannelId The ID of the target channel to move to
     * @param payload              {@link IPayload} received from the client
     * @param errorPayload         Error {@link IPayload} sent from the server to the client if the channel move fails. Not sent if successful
     * @return If the return value is false, the channel move is not possible and the request fails; if true, it succeeds
     */
    @Override
    public boolean canMoveOutChannel(String destinationChannelId, IPayload payload, IPayload errorPayload) {
        return false;
    }

    /**
     * Called from the source node when moving to another channel
     *
     * @param destinationChannelId The ID of the target channel to move to
     * @param outPayload           {@link IPayload} to be sent to the target channel
     */
    @Override
    public void onMoveOutChannel(String destinationChannelId, IPayload outPayload) {

    }

    /**
     * Called from the source node after the channel move to another node is complete
     */
    @Override
    public void onAfterMoveOutChannel() {

    }

    /**
     * Called upon entering the target node when moving to another channel
     *
     * @param sourceChannelId The ID of the channel before the move
     * @param payload         {@link IPayload} received from the client
     * @param outPayload      {@link IPayload} to be sent to the client
     * @throws GameAnvilException Throws a GameAnvilException wrapping IOException, ExecutionException, or InterruptedException if any of these occur
     */
    @Override
    public void onMoveInChannel(String sourceChannelId, IPayload payload, IPayload outPayload) throws GameAnvilException {

    }
}
```

```java
@GameAnvilController
public class _GameUserTest {
   // Mapping the protocols and handlers you want to process in SampleGameUser
    @GameUserMapping(
        value = MyGame.GameUserTest.class, // Protobuffer to process
        loadClass = SampleGameUser.class   // The message receiver (SampleGameUser)
    )
    public void execute(IUserDispatchContext ctx) {
       // Write the task to be performed here
    }
}
```

In particular, users have to be set to register for the engine compared to other classes. First, after registering the user identity for which game service, register the user type. This user type is used to distinguish between the types of users by name and for calling APIs in the same way as the client. This specifies which server type the API is called for. Therefore, these user types must be predefined with a random string between the server and the client. This example code uses a user type called "BasicUser". For more detailed description of this, see a separate chapter. Finally, you can determine whether this user information is used to [synchronize user information](server-impl-09-channel.md#channel-user-information) between channels. If you don't need to synchronize information between channels, you can omit it.

Callbacks starting with onLogin in the example code above are all called as related to login. For example, for the first login request, an onLogin callback is called, and when processing re-login while you are already logged in, an onReLogin callback is called. In the same way, when processing logouts, an onLogout callback is called. Thus, GameAnvil's callback is often clearly explained through its name and JavaDoc commas.

The user can move between channels at any time. What is related to these channels will be explained in [specific chapters](server-impl-09-channel) below, so let me pass by here first. In addition, a user is an object that can be transferred between multiple GameNodes. This feature is also explained in more detail in a [separate chapter](server-impl-08-object-transfer).

GameAnvil offers two types of matchmaking - room matchmaking and user matchmaking. When these matchmaking requests reach the user, an onMatchRoom or onMatchUser callback is called. For this callback, the user may use a matchmaker provided by GameAnvil or implement it directly or use another matchmaker provided by a third party. For more detailed description, let me re-examine MatchNode in [the next chapter](server-impl-04-match-node).

The callbacks for these users are summarized in the table below:

| Callback Name | Meaning | Description |
|--------------------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| onLogin | Log in | Called when you first log in. Usually, the user performs tasks to reset the game user object by taking user information from repositories such as DB from this callback. |
| onAfterLogin | Successful Login After Processing | OnLogin is called after success. You can perform the post-processing task for logging in to this callback. |
| onReLogin | Re-logged in | If you log in again while you are already logged in, onReLogin will be called instead of onLogin. The task of re-logging is processed in this callback. |
| onDisconnect | Access Ended | Called when the client has lost access. At this time, the code to be processed is implemented here. |
| onPause | Pause | Pause of GameNode through the console will call all the users of the GameNode. When the node is temporarily stopped, the user can implement the code that the user wants to process additionally. |
| onResume | Resume | If GameNode runs again while the console is temporarily stopped, all users in the GameNode will be called. Here, the user can implement the code he wants to process for the user in a reset state. |
| onLogout | Logout | Called when the user logs out. This can be a logout explicitly requested by the user, or it may be automatically logged out by the engine if the set time is exceeded while the user is disconnected. |
| canLogout | Check logout availability | The user is called to check if the user is currently in a logout availability state. If you are in the game or should not lose your information, you can delay your log out by returning false. If false is returned, the engine will continue to call this callback after a random time. |
| onAfterLeaveRoom | Post-processing of the room has left | Run onLeavingRoom of the room and the user will be called after the room has left. Proceed with the task that must be performed by the user in the room. |
| canTransfer | Confirm that user transmission is available | The user will be called to check whether the user can be transferred to another node. If you are in the game or are not yet prepared, you can return false and delay the sending. If you return false, the engine will continue to call this callback after a random time. |
| onLoginByOtherDevice | Trying to log in to another device | If you have an additional login request to another device and you are already logged in, it will be called. The user can determine which of the existing and new users to log in with the return value. |
| onLoginByOtherUserType | Trying to log in with a different user type | It will be called when the login request comes with another user type while already logged in. You can determine whether to proceed with logging for the new user type with a return value. |
| onLoginByOtherConnection | Trying to log in with another connection | If you are already logged in (due to re-access), you will be called if you try to log in with another connection. If you need to do more about re-access, you can do it in this callback. |
| onMatchRoom | Request room matchmaking | If the user requests room matchmaking, the call will be called. At this time, the user can randomly use the room matchmaking API provided by the engine from this callback. |
| onMatchRoomFail | Room matchmaking request failed | If the user requests room matchmaking, the call will be called. |
| onMatchUserFail | User matchmaking request failed | If the user requests user matchmaking, the call will be called. In this callback, the user can use a user matchmaking API provided by the engine or a third-party matchmaking solution at random. |
| onMatchUser | Request User Matchmaking | If the user requests user matchmaking, the call will be sent. In this callback, the user can use a user matchmaking API provided by the engine or a third-party matchmaking solution at random. |
| onMatchUserCancel | Cancel User Matchmaking | If the user previously applied for a user matchmaking, the call will be canceled. If the cancelation is unavailable because the match is already completed, it may fail. |
| onTransferOut | Sending and ready to be sent from an existing node | When the user is sent to another GameNode, it will be called when the sending starts from the source node. In this callback, the user can create a data package to be sent with the user object. |
| onTransferIn | Transfer to the new node completed. | When the user is sent to another GameNode, the message is called while the sending is completed from the target node. The user can release the data package they brought with them and restore it to the original user state. |
| onAfterTransferIn | Post-processing has been completed | If the user transfer succeeds, the target node will be called for post-processing. |
| onSnapshot | Request a snapshot from the client | Calls when the client requests a snapshot. When the connection is mainly disconnected and the server status is likely to change, the call is used to synchronize the client and server status information. |
| canMoveOutChannel | Check whether the channel can be moved | Calls are used to check whether the user can be moved to another channel. If the user explicitly calls the moveChannel() API to move a channel, it will not be called. Only called when a silent channel movement occurs by the engine. |
| onMoveOutChannel | Prepare to move from an existing channel to another channel | When the user moves to another channel, it is called from the source node. The user can store the desired information in outPayload and take it to the target channel. |
| onAfterMoveOutChannel | If onMoveOutChannel succeeds, the user will be called for post-processing. |
| onMoveInChannel | Move to the new channel | When the user moves to another channel, the user will be called from the target node. The user can transfer any random information to the client by keeping it in outPayload. |

<a id="what-is-a-login"></a>

### What is a Login? { #what-is-a-login }

The information described above and the example code often display information about the login. These logs can also be defined as the process of creating its own user objects on GameNode after the client connects to the server. Some of the callback methods call onLogin() while trying to log in to create a user first. At this time, the user can obtain information from the DB, etc. to configure user objects. When this onLogin() callback succeeds, the user object is created on GameNode. Once logging is complete, the client can receive messages from other objects through its user object based on a directly defined protocol and implement various content.

<a id="logout"></a>

### Logout { #logout }

Logout is the opposite concept of login. This is the process of removing its user objects from GameNode. Once logout is started, the user object can keep its final status in DB, etc before being deleted from memory by calling onLogout() callback. Such logouts may be explicitly requested by the client, or are automatically processed by the engine after a certain amount of time has elapsed while the client's connection is lost. Therefore, if frequent access disconnections are expected, such as mobile games, you can make appropriate [setups](server-impl-16-config-vm.md#game) to avoid immediate logout. 

<a id="implement-room"></a>

## Implement Room { #implement-room }

Two or more users can create a synchronized message flow through the room. It means that users' requests are guaranteed to be orderly all inside the room. Creating rooms for one user, of course, can have meaning depending on the content. How to use a room is up to the engine user. These rooms, like the users, implement the default class IRoom interface to redefine multiple callback methods and can also process messages on their own. The example code below is a SampleRoom class for SampleUser.

```java
@GameAnvilRoom(
    gameServiceName = "MyGame", // Nodes (service names such as SampleGameNode above)
    gameType = "BasicRoom",     // Register a room of the type "BasicRoom" in the engine
    useChannelInfo = true       // Settings for synchronizing information between channels
)
public class SampleGameRoom extends BaseGameRoom<SampleGameUser> {

/**
     * Called when the room is initialized
     */
    @Override
    public void onInit() {

}

/**
     * Called when the room is destroyed
     */
    @Override
    public void onDestroy() {

}

/**
     * Called when a new room is created
     * <p/>
     * Whether to create the room is determined by the return value
     *
     * @param user       The user object that made the request
     * @param inPayload  {@link IPayload} received from the client
     * @param outPayload {@link IPayload} to be sent to the client
     * @return true if room creation succeeds, false if it fails
     */
    @Override
    public boolean onCreateRoom(SampleGameUser user, IPayload inPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

/**
     * Called when joining an arbitrary room
     * <p/>
     * Whether to join the room is determined by the return value
     *
     * @param user       The user object that made the request
     * @param inPayload  {@link IPayload} received from the client
     * @param outPayload {@link IPayload} to be sent to the client
     * @return true if entry succeeds, false if it fails
     */
    @Override
    public boolean onJoinRoom(SampleGameUser user, IPayload inPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

/**
     * Called when leaving a room
     * <p/>
     * Whether to leave the room is determined by the return value
     *
     * @param user       The user object that made the request
     * @param inPayload  {@link IPayload} received from the client
     * @param outPayload {@link IPayload} to be sent to the client
     * @return true if leaving succeeds, false if it fails
     */
    @Override
    public boolean canLeaveRoom(SampleGameUser user, IPayload inPayload, IPayload outPayload) {
        return true;
    }

/**
     * Called when a user leaves the room
     * <p/>
     * Processes the last task before the user leaves the room
     *
     * @param user The user leaving the room
     */
    @Override
    public void onLeaveRoom(SampleGameUser user) {

}

/**
     * Called after the user has completely left the room
     * <p/>
     * Processes tasks related to the room and the users remaining in the room
     */
    @Override
    public void onAfterLeaveRoom() {

}

/**
     * Called when automatically re-entering the room upon re-login
     *
     * @param user       The user object entering the room
     * @param outPayload {@link IPayload} to be sent to the client
     */
    @Override
    public void onRejoinRoom(SampleGameUser user, IPayload outPayload) {

}

/**
     * When the room moves (sends) to another node, calls to get data to pass from the departure node
     *
     * @param transferPack {@link ITransferPack} for storing the data package to be carried to another node
     */
    @Override
    public void onTransferOut(ITransferPack transferPack) {

}

/**
     * When the room moves to another node (sending), the newly created room objects from the target node are restored to their original state and called to register the time for the operator to process.
     * <p>
     * Check the list of timerKeys registered in the room through TimerHandlerTransferPack
     * <p/>
     * Use reRegister() of TimerHandlerTransferPack to re-register the timerHandler to use
     * <p/>
     * At this point, the room is not fully restored, so requested message to other places (node, user, etc.) is restricted
     *
     * @param userList                 The list of users to move
     * @param transferPack             {@link ITransferPack} containing the data package brought from the other node
     * @param timerHandlerTransferPack {@link ITimerHandlerTransferPack} for re-registering timer handlers
     */
    @Override
    public void onTransferIn(List<SampleGameUser> userList, IReadOnlyTransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

}

/**
     * Called after the room has completed moving (sending) to another node
     */
    @Override
    public void onAfterTransferIn() {

}

/**
     * Called when the node that the room belongs to is paused, causing the room to also be paused
     */
    @Override
    public void onPause() {

}

/**
     * Called when the node that the room belongs to is resumed, causing the room to also be resumed
     */
    @Override
    public void onResume() {

}

/**
     * Called when the client requests party matchmaking
     * <p/>
     * For party matchmaking, two or more users must enter a Named Room of the party type
     *
     * @param roomType      An arbitrary value that distinguishes between predefined room types between the client and the server
     * @param matchingGroup The matching group
     * @param user          The user (room leader) who requested party matchmaking
     * @param payload       {@link IPayload} received from the client
     * @param outPayload    {@link IPayload} to be sent to the client
     * @return true if the party matchmaking request succeeds, false if it fails
     */
    @Override
    public boolean onMatchParty(String roomType, String matchingGroup, SampleGameUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

/**
     * Called when party matchmaking is canceled
     *
     * @param reason The reason for cancellation: timeout (TIMEOUT), canceled by user request (CANCEL), canceled due to Match Node shutdown (SHUTDOWN)
     */
    @Override
    public void onMatchPartyCancel(MatchCancelReason reason) {

}

/**
     * Called when room matchmaking is canceled
     *
     * @param reason The reason for cancellation. (SHUTDOWN: canceled due to Match Node shutdown)
     */
    @Override
    public void onForceMatchRoomUnregistered(MatchCancelReason reason) {

}

/**
     * Called to check whether the room is in a state that allows it to move (send) to another node
     *
     * @return true if transfer is possible, false if not
     * <p/>
     * In case of unavailability, if SafePause is in progress, continue to call to transfer to the room until SafePause ends.
     */
    @Override
    public boolean canTransfer() {
        return true;
    }
}
```

```java
// Proto Buffer MyGame.GameRoomTest Running Message Processing Class
@GameAnvilController
public class _GameRoomTest {
    // Map the protocol and handler to be processed in SampleRoom
    @GameRoomMapping(
        value = MyGame.GameRoomTest.class, // Proto Buffer to process
        loadClass = SampleRoom.class   // Recipient (SampleRoom)
    )
    public void execute(IRoomDispatchContext ctx) {
        // Write up the task
    }
}
```

There are also several settings required to register the room as an engine. First, after registering a profile for any game service, register the room type. This room type is used to identify the type of room by its name and is used for calling APIs in the same way as the client. This specifies which server type the API is called for. Therefore, these room types between the server and the client must be predefined with a random string. This example code uses a room type called "BasicRoom". For more detailed description of this, see a separate chapter. Finally, you can determine whether this room information will be used to [synchronize room information](server-impl-09-channel.md#channel-room-information) between channels. If you don't need to synchronize information between channels, you can omit it.

The room features the most basic three onCreateRoom, onJoinRoom and onLeaveRoom. Each room will be called for creation, participation and departure. The user can implement the related feature directly from that callback. For example, you can create a room to direction and reset basic data structures. For any room participation request, you can also update the list of users in the room, or perform status synchronization between each user, and more.

This creation and participation in rooms is automatically processed by engines as well as by client-express requests. For example, in a game with two gardens, if user A and user B are matched, one person automatically creates a room and the other person will join the room. This process is to be automatically processed from the users A and B, not the users B's requests. In this process, onCreateRoom and onJoinRoom callbacks are also called.

If you clean up the callback of these rooms, you can see the table below.

| Callback Name | Meaning | Description |
|---------------------------- |-------------------- |------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onInit | Initialize | When a room is created, it will be called for initialization. You can write an init code for the room, such as registering topics. |
| onDestroy | Room Disappearance | If the last user leaves the room and there is no message to deal with, the room will disappear. This is the callback called. |
| onCreateRoom | Create Room | When the client requests to create a room, the call will be called. Create a material structure for the user list to be used in the content or write the code to be processed with other room creation. |
| onJoinRoom | Room Participation | If the client requests to participate as a random room on the server, the call will be sent. You can update the list of users to use in the content or process the information to synchronize between the users in the room. |
| canLeaveRoom | Check if you leave the room | You will be called when you leave the room. Whether to leave the room is determined by the return value. |
| onLeaveRoom | Leave the room | When the client requests to leave the room, the call will be called. Write the code that needs to be processed alongside updating the material structure for the user list to be used in the content or coming out of another room. |
| onAfterLeaveRoom | Afterprocessing after leaving the room | When onLeaveRoom succeeds, you will be called for postprocessing. Write the code to be processed just after you leave the room. |
| onRejoinRoom | Rejoin the room | When a user re-register (ReLogin) due to disconnection while they are inside the room, the engine will automatically re-register (ReJoin) the room. This is the callback being called. You can process the information required to synchronize, etc. during the re-input process. |
| onTransferOut | sent and ready to be sent from an existing node | When a room is sent to another GameNode, it will be called when the sending starts from the source node. In this callback, the user can create a data package to send with the room object. Note that room transfer includes user transmission for each user in the room. |
| onTransferIn | Transfer to the new node completed. | When the room is transferred to another GameNode, the message is called while the sending is completed from the target node. The user can release the data package they brought with them and restore it to the original room state. Note that room transfer includes user transmission for each user in the room. |
| onAfterTransferIn | After transfer to the new node is completed, process | When the room is transferred to another GameNode, it is called while the transmission is completed from the target node. The user can release the data package they brought with them and restore it to the original room state. Note that room transmission includes user transmission for each user in the room. |
| onPause | Pause | When you stop GameNode through the console, all rooms on that GameNode will be called. Here, when the node is temporarily stopped, the user can implement the code he wants to process additionally in the room. |
| onResume | Resume | If GameNode runs again while the node is temporarily stopped, all rooms in the GameNode will be called. Here, the user can implement the code he wants to process for the room while he is on a restart. |
| onMatchParty | Request Party Matchmaking | If the user requests Party Matchmaking, the message will be called. Party Matchmaking is a feature to request a match with one party from all the users in the room after creating a random NamedRoom for party use. In this callback, the user can use a party matchmaking API provided by the engine or a third-party matchmaking solution at random. |
| onMatchPartyCancel | Cancel party matchmaking request | If the user requests a party matchmaking, the user will be called. Party Matchmaking is a feature to request a match with one party from all the users in the room after creating a random NamedRoom for party use. For this callback, the user can use a partial matchmaking API provided by the engine or a third-party matchmaking solution at random. |
| onForceMatchRoomUnregistered | Room matchmaking is canceled | When room matchmaking is canceled, the user will be called. |
| canTransfer | Check whether the room is available for transmission | Calls to check whether the room is available for transmission to another node. If the game is still in play or unprepared in the room, you can return false and delay the sending. If false is returned, the engine will continue to call this callback after a random amount of time. Note that room transmission is only used when performing a non-stop check patch. |

<a id="room-type"></a>

## Room Type { #room-type }

The method of implementation of the room discussed earlier and the type of room provided by the engine separately are largely two. These two room types are used in four different ways in total.

| Room type | Description |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Normal Room | CreateRoom is created by the client's explicit request. Other clients can also use the room's ID to explicitly participate with JoinRoom. Therefore, the user that will participate must be previously shared with the ID of the room. |
| Named Room  | NamedRoom performs the action for a room named based on the only room name as it is. The client requests namedRoom as the only room name within the server group. In this case, if the room name does not exist on the server, the requestant will create a room directly. Conversely, if the room name already exists on the server, it will automatically enter the room. For example, it's easy to understand if you think of any kind of argument, such as "3:3 Hunter First!" that appears in our custom game list. |

These two room types are used in four different ways in total.

| Room Type | Usage |
|------------- |------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Normal Room | 1. Clients are created and participated through CreateRoom / JoinRoom requests.<br>2. You can create or participate in NormalRoom through room matchmaking. You can also register for preventive room matches created with CreateRoom. At this time, the room ID is automatically shared between the room and the user that is managed and matched by the engine. |
| Named Room  | 3. Clients are created and involved via the NamedRoom request.<br>4. You can create or participate in NamedRoom through user matchmaking. At this time, the room names of the created NamedRoom are created and managed uniquely in the engine. Unlike room matchmaking, rooms created with the normal NamedRoom cannot be targeted for user matchmaking. However, after creating a party room with NamedRoom for party matchmaking, multiple users can request matchmaking as a party. |

<a id="channel"></a>

## Channel { #channel }

GameNodes can be logically grouped according to their use. These logical groups are called [channels](server-impl-09-channel). For example, you can bind GameNode 1 and 2 to a "Beginner" channel, and GameNode 3 and 4 to an "Expert" channel. For more detailed description, we will cover this again in a [separate chapter](server-impl-09-channel).