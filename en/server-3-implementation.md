## Game > GameAnvil > Server Development Guide > Implementation Guide

engine that are needed to implement a basic server. For code level materials, refer to [GameAnvil reference server](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-reference-server). In addition, JavaDoc documents can be used through [GameAnvil API Reference](http://10.162.4.61:9090/gameanvil). If possible, it is recommended to refer to reference server projects and this guide document.



### Note

*GameAnvil uses a naming convention that begins with _ to the message handlers in the engine and for all sample code. In other words, all the classes that starts with _ in example code are message handlers. The implementation for these message handlers is not covered in this article. The easiest and the most correct way is checking code through the previous reference server project or JavaDoc.*



The most of GameAnvil's features are provided in callback. In other words, the engine users inherit **GameAnvil's base classes and redefine** the callback method to access the most of the features. Because the engine users can implement only the needed callback methods, some callback methods can be ignored. This document does not explain actual implementation of these callback methods. For the code level, it is best refer to the reference server project mentioned earlier.



## 1. Node common

All kinds of nodes must redefine the callback methods below. In addition, each node implements additional callback methods fit for their purposes. As the GatewayNode of the code below does not have additional callback, it is fit to show the common callback methods. Especially, the setReady() API called by onPrepare() is a command for actually running the engine as the nodes are fully prepared. Generally, it is called by onPrepare(). However, if users want to communicate with other nodes or send a ready command based on an arbitrary asynchronous result, calling it from another code block is fine as well.

```
public class SampleGatewayNode extends BaseGatewayNode {

    /**
     * Occurs while initializing.
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onInit() throws SuspendExecution {        
    }

    /**
     * Called while Preparing.
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPrepare() throws SuspendExecution {
        setReady(); // caution!
    }

    /**
     * Called in the Ready step.
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onReady() throws SuspendExecution {        
    }

    /**
     * Called when packet is passed.
     *
     * @param packet A packet to be processed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {        
    }

    /**
     * Called when paused.
     *
     * The additional information that @param payload contents wants to pass
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPause(Payload payload) throws SuspendExecution {        
    }

    /**
     * Called when resumed.
     *
     * The additional information that @param payload contents wants to pass
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onResume(Payload payload) throws SuspendExecution {        
    }

    /**
     * Called when the shutdown command is issued to end NonNetworkNode.
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onShutdown() throws SuspendExecution {        
    }
}
```



## 2. Gateway Node

GatewayNode is the node that the client is connected to. The BaseGatewayNode class is inherited and redefine the common callback method. There is nothing else to be implemented to GatewayNode itself. GatewayNode manages the connections and sessions from the client. The relationship among them are explained in the picture below:

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

Generally, the client establishes a single connection to GatewayNode. At this time, additional sessions can be created only when the connection is successfully authenticated. Each session is a logical connection unit for different game services. The picture above depicts the client creating sessions for Game service and Chat service.



### 2-1. Connection

Connection means the physical connection to the client itself. The client can authenticate from the connection using a unique AccountId. If authentication is successful, AccountId is mapped to the created connection. Connection redefines callback method as below. At this time, AccountId is authenticated on an arbitrary platform and the acquired key value of the user can be used. For example, when authenticated through Gamebase and acquire UserId, this value can be used as AccountId in authenticating GameAnvil.

```
public class SampleConnection extends BaseConnection<SampleGameSession> {

    /**
     * Called when authenticating.
     *
     * @param accountId  The ID of the account
     * @param password   The password of the account
     * @param deviceId   The device ID of the client
     * @param payload    The payload received from the client
     * @param outPayload The payload to pass to the client
     * The connection to the client ends when @return is false
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onAuthenticate(final String accountId, final String password, final String deviceId, final Payload payload, final Payload outPayload) throws SuspendExecution {        
    }

    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {        
    }

    /**
     * Called before login is called.
     *
     * @param outPayload The payload to pass to the client
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPreLogin(Payload outPayload) throws SuspendExecution {        
    }

    /**
     * Called when successfully logged in.
     *
     * @param session The session object used while logging in
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPostLogin(U session) throws SuspendExecution {        
    }

    /**
     * Called after logged out.
     *
     * @param session The session object that is connected to the game node's user
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPostLogout(U session) throws SuspendExecution {
    }

    @Override
    public void onPause() throws SuspendExecution {
    }

    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * Called when the connection to the client is lost.
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    public void onDisconnect() throws SuspendExecution {        
    }
}
```



### 2-2. Session

The client that successfully established connection can form a logical session with GameNode per service on the connection. GameAnvil internally combine the AccountId of connection and the SubId of session in the form of "AccoundId:SubId" to distinguish unique sessions and the users using them. These sessions can then register messages and handlers in the session and dispatch them using the callback method. At this time, a unique value needs to be assigned to subId in the connection according to the rules arbitrarily defined by users.

```
public class SampleSession extends BaseSession {

    private static PacketDispatcher dispatcher = new PacketDispatcher();

    static {
        dispatcher.registerMsg(SampleGame.MsgToSessionReq.getDescriptor(), _MsgToSessionHandler.class);
    }

    /**
     * Called when a packet is passed to session.
     *
     * @param packet A packet to be processed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        dispatcher.dispatch(this, packet);
    }
}
```



### 2-3. Bootstrap

The GatewayNode, connection, and session created in this way can be registered in the following method in the Bootstrap step.

```
public class Main {

    public static void main(String[] args) {

        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();
        // Register
        bootstrap.setGateway()
            .connection(SampleConnection.class)
            .session(SampleSession.class)
            .node(SampleGatewayNode.class)
            .enableWhiteModules();

        bootstrap.run();
    }
}
```



## 3. Game Node

GameNode is a node that is used to create actual game object and implement game content. GameNode basically inherits and implements the BaseGameNode abstract class. GameUser and GameRoom are some of the major objects that are managed by GameNode. Let's take a look at this in depth. The code below shows the callback method that can be redefined in GameNode. The callback that is used to synchronize channels to the common node callback. In addition, the protocol that users want to process in GameNode can be mapped to an arbitrary handler to process the packet using the onDispatch() callback.



```
public class SampleGameNode extends BaseGameNode {

    // 1.Create packet dispatcher    
    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    // 2.Map the protocol and handler that need to be processed by SampleGameNode
    static {
        packetDispatcher.registerMsg(SampleGame.GameNodeTest.getDescriptor(), _GameNodeTest.class);
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onPrepare() throws SuspendExecution {
        setReady(); // caution
    }

    /**
     * Called when packet is passed.
     *
     * @param packet A packet to be processed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {
        // 3.Process the message of SampleGameNode
        if (packetDispatcher.isRegisteredMessage(packet))
            packetDispatcher.dispatch(this, packet);
    }

    @Override
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {
    }

    @Override
    public void onResume(Payload payload) throws SuspendExecution {
    }

    @Override
    public void onShutdown() throws SuspendExecution {
    }

    /**
     * A callback that is called when users are changed in different nodes in the same channel
     * <p>
     * updateChannelUser() Occurs when called.
     *
     * @param type            Channel information change type(update/delete)
     * @param channelUserInfo The user information to be changed
     * @param userId          The user ID of the item to be changed
     * @param accountId       The account ID of the item to be changed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onChannelUserUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {        
    }

    /**
     * A callback that is called when the room status is changed in a different node of the same channel
     * <p>
     * updateChannelRoomInfo() Occurs when called.
     *
     * @param type            Channel information change type(update/delete)
     * @param channelRoomInfo The room information to be changed
     * @param roomId          The Room ID of the item to be changed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onChannelRoomUpdate(ChannelUpdateType type, RoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {        
    }

    /**
     * A callback that is called when the client requests channel information (Base.GetChannelInfoReq)
     *
     * @param outPayload The channel information to be passed to the client
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {        
    }
}
```



### 3-1. GameUser

The GameUser object, which represents the users in the game, is created in GameNode during login. All the messages in the user level can be processed here. Therefore, implementing user-related content centered around this class is recommended. Like all the previous examples, GameUser can map the protocols and handlers to be processed.

The example code below shows the entire list of callback methods of GameUser. Some of them provides basic implementation, users do not have to redefine them unless it is necessary. This is applied to GameUser and the most of the callback methods provided by the engine. For this part, refer to the JavaDoc document for each base class (e.g. BaseUser) in [GameAnvil API Reference ](http://10.162.4.61:9090/gameanvil) or [reference server project](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-reference-server).

Among the callback methods below, onLogin() is called while logging into the game for the first time to create a game user. When this onLogin() callback succeeds, a game user object is created on GameNode. Once login is complete, many different contents can be processed by transferring messages with the client based on the user-created protocol.

```
public class SampleGameUser extends BaseUser {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameUserTest.getDescriptor(), _GameUserTest.class);
    }

    /**
     * A callback that occurs when logging in
     *
     * @param payload        The arbitrary payload passed by the client
     * The payload passed by @param sessionPayload onPreLogin
     * @param outPayload     An arbitrary payload to pass to the client
     * true with @return boolean type: login succeeds. false: Returns login fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onLogin(final Payload payload,
                           final Payload sessionPayload,
                           Payload outPayload) throws SuspendExecution {        
    }

    /**
     * A callback that occurs after the login process
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPostLogin() throws SuspendExecution {

    }

    /**
     * A callback that is called when a user logs into a different device with a duplicate ID
     *
     * @param newDeviceId           The deviceId value of the newly connected user
     * @param outPayloadForKickUser The payload to pass to the client
     * true with @return boolean type: Newly connected user login succeeds, the current user is forceLogout. false: Newly connected user returns login fails.
     * @throws SuspendExecution This method cannot suspend Fiber/
     */
    @Override
    public boolean onLoginByOtherDevice(final String newDeviceId,
                                        Payload outPayloadForKickUser) throws SuspendExecution {
        return true;
    }

    /**
     * A callback that is called when another type of user is logging in when a different user is already logged in while using the MultiType user
     *
     * @param userType   The type of the user who tries a new login
     * @param outPayload The payload to pass to the client
     * true with @return boolean type: New login successful. false: Returns new login failure.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onLoginByOtherUserType(final String userType,
                                          Payload outPayload) throws SuspendExecution {
        return true;
    }

    /**
     *  A callback that is called when a duplicate ID is logged on the same device
     *
     * @param outPayload The payload to pass to the client
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {        
    }

    /**
     * A callback that is called when login is called while user object is active in the server
     *
     * @param payload        The arbitrary payload passed by the client
     * The payload passed by @param sessionPayload onPreLogin
     * @param outPayload     An arbitrary payload to pass to the client
     * true with @return boolean type: ReLogin succeeds, false: Returns ReLogin fails.
     * @throws SuspendExecution: This method cannot suspend Fiber.
     */
    @Override
    public boolean onReLogin(final Payload payload,
                             final Payload sessionPayload,
                             Payload outPayload) throws SuspendExecution {
    }

    /**
     * A callback that is called when the connection with the client is severed
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onDisconnect() throws SuspendExecution {        
    }

    /**
     * A callback that is called when there is a packet to be passed to a user
     *
     * @param packet A packet to be processed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onDispatch(final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, packet);
    }

    /**
     * A callback that is called to the user when node is paused
     *
     * @throws SuspendExecution This method can suspend Fiber
     */
    @Override
    public void onPause() throws SuspendExecution {        
    }

    /**
     * A callback that is called to the user when node is resumed
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onResume() throws SuspendExecution {        
    }

    /**
     * A callback that is called when the user logs out
     *
     * @param payload    The payload received from the client
     * @param outPayload The payload to pass to the client
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onLogout(final Payload payload, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * A callback that is used to check if the user can log out.
     *
     * Returns to @return boolean type: If false, logout fails and it is checked later at an interval: If true, the user is logged out.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean canLogout() throws SuspendExecution {        
    }

    /**
     * A callback that occurs when room matchmaking is requested from the client
     *
     * @param roomType The type of matched room
     * @param payload  The payload received from the client
     * @return Returns the information of the matched room, if it returns null, a new room can be created or processed as failure according to the client's request option
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public MatchRoomResult onMatchRoom(final String roomType,
                                       final Payload payload) throws SuspendExecution {
        return null;
    }

    /**
     * A callback that is called when user matchmaking is requested by the client
     *
     * @param roomType   The type of the matched room
     * @param payload    The payload received from the client
     * @param outPayload The payload to pass to the client
     * @return true: User matchmaking request succeeds, false: User matchmaking request fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onMatchUser(final String roomType,
                               final Payload payload,
                               Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * A callback that is called when userMatch is canceled from the client
     *
     * @param reason Reason of cancel (TIMEOUT/CANCEL)
     * Returns to @return boolean type. true: User matchmaking cancel successful, false: User matchmaking cancel fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
        return false;
    }

    /**
     * A callback that is called after timer handler is registered
     */
    @Override
    public void onRegisterTimerHandler() {        
    }

    /**     
	 * A callback that is called to pass arbitrary data when the user moves to another node
     *
     * @param transferPack The object that is used to configure the required data
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onTransferOut(final TransferPack transferPack) throws SuspendExecution {        
    }

    /**
     * A callback that is called to recover user define after the user moved to another node
     * <p>
     * Request calls to a different object is limited as the user is not completely recovered at this point.
     *
     * @param transferPack The object passed before moving. Retrieves the set value from the object.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onTransferIn(final TransferPack transferPack) throws SuspendExecution {        
    }

    /**
     * A callback that is called after the user's inter-node transfer
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPostTransferIn() throws SuspendExecution {        
    }

    /**
     * A callback that is called when the client requests snapshot
     * <p>
     * It is used to the status information of the client and server by calling it when the connection is lost  and there is a possibility of the server status changing.
     *
     * @param payload    The payload received from the client
     * @param outPayload The payload to pass to the client
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onSnapshot(final Payload payload, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * A callback that is called when the client requests the user to move to another channel
     *
     * @param destinationChannelId The target channel ID to switch
     * @param payload              The payload received from the client
     * @param errorPayload         The payload that is passed from the server to the client when channel switching fails. It is not passed when it succeeds.
     * @return false: Channel switch request fails, true: Channel switch request successful.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onCheckMoveOutChannel(final String destinationChannelId,
                                         final Payload payload,
                                         Payload errorPayload) throws SuspendExecution {
        return false;
    }

    /**
     * A callback that is called if   onCheckMoveOutChannel() returns true when the client receives a channel switch request
     * <p>
     * When switching channels, onCheckMoveOutChannel() is not called and the moveChannel() of the called server is immediately called.
     *
     * @param destinationChannelId The channel ID to switch
     * @param outPayload           The payload that needs to be passed to the channel to switch
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onMoveOutChannel(final String destinationChannelId,
                                 Payload outPayload) throws SuspendExecution {
    }

    /**
     * onMoveOutChannel() A callback that is called after the call
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPostMoveOutChannel() throws SuspendExecution {
    }

    /**
     * A callback that occurs when entering the switched channel
     *
     * @param sourceChannelId The channel ID before switching
     * @param payload         The payload received from the client
     * @param outPayload      The payload to be passed to the client
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onMoveInChannel(final String sourceChannelId,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * onMoveInChannel() A callback that is called after execution
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPostMoveInChannel() throws SuspendExecution {
    }

    /**
     * Check if User Transfer is possible.
     *
     * Returns to @return boolean type. true: Transfer possible, false: Transfer impossible, It is called continuously before NonStopPatch is ended if transfer is impossible.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean canTransfer() throws SuspendExecution;
}
```



### 3-2. GameRoom

Two or more game users can create a message flow through GameRoom. In order words, the requests from game users are processed in GameRoom as they are received. Of course, creation of GameRoom for one game user may have a meaning, depending on content. Using GameRoom is entirely dependent on the user. Like GameUser, GameRoom can redefine a variety of callback methods based on the base class (BaseRoom) and can process protocols on its own. The example code below is the SampleGameRoom class for SampleGameUser.

```
public class SampleGameRoom extends BaseRoom<SampleGameUser> {

    private static PacketDispatcher packetDispatcher = new PacketDispatcher();

    static {
        packetDispatcher.registerMsg(SampleGame.GameRoomTest.getDescriptor(), _GameRoomTest.class);
    }

    /**
     * A callback that occurs when Room is initialized
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onInit() throws SuspendExecution {        
    }

    /**
     * A callback that occurs when Room is deleted
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onDestroy() throws SuspendExecution {        
    }

    /**
     * A callback that is called when {@link Packet} is passed to Room
     *
     * @param user   he user packet that will process the packet. null if there is no user to process the packet.
     * @param packet A packet to be processed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onDispatch(SampleGameUser user, final Packet packet) throws SuspendExecution {
        packetDispatcher.dispatch(this, user, packet);
    }

    /**
     * A callback that occurs when the client requests room creation
     *
     * @param user       The user object that requested room creation
     * @param inPayload  The payload received from the client
     * @param outPayload The payload to pass to the client
     * @return true: Room creation succeeds, false: Room creation fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onCreateRoom(SampleGameUser user,
                                final Payload inPayload,
                                Payload outPayload) throws SuspendExecution {
    }

    /**
     * A callback that occurs when the client requests to join Room
     *
     * @param user       The user object that requests to join Room
     * @param inPayload  The payload received from the client
     * @param outPayload The payload to pass to the client
     * @return true: Room join successful, false: Room join fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onJoinRoom(SampleGameUser user,
                              final Payload inPayload,
                              Payload outPayload) throws SuspendExecution {
    }

    /**
     * A callback that occurs when the client requests to leave Room
     *
     * @param user       The user object that requested to leave Room
     * @param inPayload  The payload received from the client
     * @param outPayload The payload to pass to the client
     * Returns to @return boolean type. true: Leave Room succeeds, false: Leave Room fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onLeaveRoom(SampleGameUser user,
                               final Payload inPayload,
                               Payload outPayload) throws SuspendExecution {
    }

    /**
     * A callback that is called when the return value is true after onLeaveRoom is called
     *
     * @param user onLeaveRoom The user object that is processed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPostLeaveRoom(SampleGameUser user) throws SuspendExecution {
    }

    /**
     * A callback that is called when ReJoin is executed while users are in Room
     *
     * @param user       The user object that executed ReJoin
     * @param outPayload The payload to pass to the client
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onRejoinRoom(SampleGameUser user, Payload outPayload) throws SuspendExecution {        
    }

    /**
     * A callback that is called after timer handler is registered
     */
    @Override
    public void onRegisterTimerHandler() {
    }

    /**
     *  A callback that is called to serialize the define of room when room transfer occurs
     *
     * @param transferPack The information package that was passed before switching. Retrieves information from the object.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * A callback that is called to recover the define of room when room transfer occurs
     *
     * @param userList     The list of user objects that are in the Room.
     * @param transferPack The Room information package to be recovered
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onTransferIn(List<SampleGameUser> userList,
                             final TransferPack transferPack) throws SuspendExecution {
    }

    /**
     * A callback that is called by each room object after the node is paused
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onPause() throws SuspendExecution {
    }

    /**
     * A callback that is called by each room after the node is resumed
     *
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public void onResume() throws SuspendExecution {
    }

    /**
     * A callback that is called when the client requests partyMatch
     *
     * @param roomType   The type of room
     * @param user       The room host who requested party matching
     * @param payload    The payload received from the client
     * @param outPayload The payload to pass to the client
     * Returns to @return boolean type. true: user matching request succeeds, false: user matching request fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onMatchParty(final String roomType,
                                final SampleGameUser user,
                                final Payload payload,
                                Payload outPayload) throws SuspendExecution {
        return false;
    }

    /**
     * Check if room transfer is possible.
     * <p>
     * Check continuously until the status of NonNetworkNode becomes Ready after the initial callback.
     *
     * Returns to @return boolean type. true: room transfer possible, false: room transfer impossible.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean canTransfer() throws SuspendExecution {        
    }
}
```



### 3-3. Bootstrap

The GameNode, GameUser, and GameRoom created in this way can be registered as below in the Bootstrap step.

```
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGateway()
            .connection(SampleGameConnection.class)
            .session(SampleGameSession.class)
            .node(SampleGameGatewayNode.class)
            .enableWhiteModules();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)
            .room("SampleGameRoomType", SampleGameRoom.class);

        bootstrap.run();
    }
}
```



## 4. 유저 전송 (UserTransfer)

User objects are created in game node. This game user object can be transferred at any time among game nodes. For example, user objects are transferred among game nodes while the user object of #1 game node is entering the room created by the user object of #2 game node. This technology is the core of GameAnvil and it works as intended only when user object is correctly serialized/de-serialized. The target of serialize/de-serialize can be directly implemented by GameAnvil users. In order words, they decide what values of the game user object should be transferred.

This type of user transfers can be categorized in three types.

- First, user objects can be transferred to a different game node by room creation or room join logic including matchmaking. At this time, the objects can enter the room in a different channel, according to implementation.
- Second, it occurs when switching channels. A game node can belong to a channel. Therefore, switching channels means moving through game nodes.
- When entering the room in a different channel using matchmaking as in the first case, it is processed implicitly inside the engine.
- The client can request channel switch explicitly.
- Third, when uninterrupted maintenance for arbitrary game nodes (NonStopPatch) is executed, the user objects of the node are distributed to other valid game nodes and transferred. In this case, explicit command is issued on GameAnvil Console by admins.



### 4-1. User Transfer Implementation

Actual user transfer is internally processed by GameAnvil. At this time, the client does not recognize that its own game user object is being transferred among servers. In other words, even when it enters the room of a different game node, the client enters an arbitrary room in the sole GameAnvil server family.
However, when user objects are being transferred to a different game node, the user can determine which data to be transferred with them. Simply specifying the data allows it to be serialized with the user object as part of it. If done, the objects can easily be accessed and used without concerning de-serialization from the game node to which transfer the data.

The following are the callback methods related to user transfer. First to do is 'specifying the data to be transferred at the same time.' To do this, inherit the BaseUser abstract class and implement the callback below in the game user class. The user simply has to place the data to be transferred in the transferPack passed as the key-value pair using the key value that the user wants.

If the data to be transferred is not specified at this time, be careful as the data of the user object that is transferred to the target game node is reset to default.

```
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("TotalMoney", myTotalMoney);
    transferPack.put("Message", myMessage);
}
```

Now, let's take a look at the callback method to be processed in the target game node after user transfer is complete. The desired object can be accessed by using the key specified before the transfer. Restore the data of the user object that is transferred in this way.

```
@Override
public void onTransferIn(TransferPack transferPack) throws SuspendExecution {
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setTotalMoney(transferPack.getToLong("TotalMoney"));
    setTransferMessage(transferPack.getToString("Message"));
}
```

The two methods above are automatically called by GameAnvil. The user only has to implement them.



### 4-2. Transferable User Timer

When transferring user, the timer registered to the user can also be transferred. To use the transferable timer, it must be pre-registered using the desired key from the callback methods below. Map the timer handler to the desired key.

```
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferUserTimerHandler1", transferUserTimerHandler1());
     registerTimerHandler("transferUserTimerHandler2", (timerObject, object) -> {
         GameUser user = (GameUser) object;
         logger.warn("GameUser::transferUserTimerHandler2() : userId({})", user.getId());
     });
 }
```

The registered timer object can be used in the game user implementation part using the key. The timer that only registered does not take effect, for actual use, the timer must be added to the user object below for it to trigger.

```
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

The timer added to the user object is automatically transferred without having to do anything with the transfer. In other words, the user object does not have to go through the timer adding process from the target game node.



## 5. ID

GameAnvil uses many different IDs. Some of them are issued by the server and the rest are configured by the user in GameAnvilConfig. The account information needed to connect passes the information received from the client to the server. Following is the description of the popular ID used by GameAnvil.

| Name      | Description                                                         | Material | Range      |
| --------- | ------------------------------------------------------------ | ------ | --------- |
| ServiceId | - The ID that is used to distinguish each configured service - Single service ID can be configured to several nodes | int    | 0<id< 100 |
| HostId    | - The unique ID of host - When running several GameAnvil processes in a single host, configure a separate vmId in GameAnvilConfig| long   | -         |
| NodeId    | - The unique ID of node - Consists of HostId + ServiceId + internal Counter value | long   | -         |
| AccountId | - Map each input - Connection to a single account ID when the client is connecting | string | -         |
| UserId    | - The unique ID of game user object - Issued by server when game user object is created - Issue a new ID if a new gamer user object is created when the same user is trying to reconnect | int    | -         |
| RoomId    | - The unique ID of room - Issued by server when room object is created         | int    | -         |
| SubId     | - Map AccountId + SubId to each unique secondary ID Session in a single account (accountId) When using several Sessions in the - Connection unit, enter it while the ID - client used to distinguish each session user in a single connection | int    | 0 < id    |

### 5-1.ID Support API

Some of the features related to the ID mentioned above are provided to engine users as shown in the list below. To obtain or confirm the ID, the API below must be used.

- ServiceId class

| Method                                | Description                                    |
| ------------------------------------- | --------------------------------------- |
| boolean isValid(int serviceId)        | Checks if serviceId is valid               |
| int findServiceId(String serviceName) | Obtains the ServiceId corresponding to ServiceName |
| String findServiceName(int serviceId) | Acquires the ServiceName corresponding to ServiceId |

- HostId class

| Method                       | Description                     |
| ---------------------------- | ------------------------ |
| boolean isValid(long hostId) | Checks if hostId is valid   |
| long get()                   | Obtains the hostId of a process |

- NodeId class

| Method                       | Name                          |
| ----------------------------- | ----------------------------- |
| boolean isValid(long nodeId)  | Checks if nodeId is valid        |
| long getHostId(long nodeId)   | Obtains hostId from nodeId    |
| int getServiceId(long nodeId) | Obtains serviceId from nodeId |
| int getNodeNum(long nodeId)   | Obtains nodeNum from nodeId   |

- UserId class

| Method                      | Description               |
| --------------------------- | ---------------------- |
| boolean isValid(int userId) | Checks if userId is valid |

- RoomId class

| Method                      | Description               |
| --------------------------- | ---------------------- |
| boolean isValid(int roomId) | Checks if roomId is valid |

- SubId class

| Method                     | Description              |
| -------------------------- | --------------------- |
| boolean isValid(int subId) | Checks if subId is valid |



## 6. Channel

Channel is one of the ways used to logically divide a single server family. GameAnvil can configure a channel if it includes one or more game nodes. Basically, configure a channel in game node as below using GameAnvilConfig. In this example, configure ch1, ch1, ch2, and ch2 for each four game nodes.

```
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

At this time, the game nodes in the same channel share channel-related information. For example, the GameAnvil engine calls the pre-implemented callback when a new user enters or leaves the channel. Use GameUserInfo and GameRoomInfo as below to synchronize the user and room information among channels. As mentioned earlier, SampleChannelUserInfo and SampleChannelRoomInfo are additionally registered to the content of Bootstrap for GameNode.

```
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

Let's take a look at how to implement ChannelUserInfo and ChannelRoomInfo. First, channel user information implements the ChannelUserInfo and Serializable interface provided by GameAnvil. As they need to be transferred among the GameNodes in the same channel, Serializable is a must. The rest can be filled with the information desired by engine users.

```
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
     * Serializes with {@link KryoSerializer}
     *
     * Returns the object serialized by @return ByteBuffer.
     */
    @Override
    public ByteBuffer serialize() {
        return KryoSerializer.write(this);
    }

    /**
     * De-serializes with {@link KryoSerializer}.
     *
     * @param inputStream A serialized stream
     * Returns an object that is de-serialized by @return ChannelUserInfo.
     */
    @Override
    public ChannelUserInfo deserialize(InputStream inputStream) {
        return (GameChannelUserInfo) KryoSerializer.read(inputStream);
    }

    /**
     * The User Id of the Channel User information to be changed.
     *
     * Returns UserId to @return int type.
     */
    @Override
    public int getUserId() {
        return userId;
    }

    /**
     * The Account Id of the Channel User information to be changed.
     *
     * Returns AccountId to @return String type.
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

The channel room information is implemented in the same way. Implement the RoomInfo interface and Serializable provided by GameAnvil. Note that the name of the interface is not ChannelRoomInfo but RoomInfo. This class can be implemented based on the information that will be used by the engine users to synchronize channels like channel user information.

```
public class SampleChannelRoomInfo implements Serializable, RoomInfo {

    public static final int MAX_ENTRY_USER = 4;
    private int roomId = 0;

    private int userCnt;
    private int gameState;
    private long createTime;

    public SampleChannelRoomInfo() {
    }

    //... Implement class using the information needed by content

    /**
     * The Room Id of the Room information.
     *
     * Returns RoomId to @return int type.
     */
    @Override
    public int getRoomId() {
        return roomId;
    }

    /**
     * Serializes using the Room information {@link KryoSerializer}.
     *
     * Returns the serialized content to @return ByteBuffer type.
     */
    @Override
    public ByteBuffer serialize() {
        return KryoSerializer.write(this);
    }

    /**
     * De-serializes using the received information {@link KryoSerializer}.
     *
     * @param inputStream The data to be de-serialized
     * Returns the Room information to @return RoomInfo.
     */
    @Override
    public RoomInfo deserialize(InputStream inputStream) {
        return (RoomInfo) KryoSerializer.read(inputStream);
    }

    /**
     * Copies the Room information.
     *
     * Returns the copied Room information to @return RoomInfo.
     * When @throws CloneNotSupportedException cannot be copied.
     */
    @Override
    public RoomInfo copy() throws CloneNotSupportedException {
        GameRoomInfo roomInfo = (GameRoomInfo) super.clone();
        roomInfo.testListInfo = new ArrayList<>(testListInfo);

        return roomInfo;
    }
}
```

The user and room information of the channel mentioned earlier is distributed to the GameNodes in the same channel whenever the actual user or room needs to be updated. At this time, the related GameNode calls the two callback methods below with each channel user information and channel room information update.

```
    /**
     * A callback that is called when users are changed in different nodes in the same channel
     * <p>
     * updateChannelUser() Occurs when called.
     *
     * @param type            Channel information change type(update/delete)
     * @param channelUserInfo The user information to be changed
     * @param userId          The User Id of the target to be changed
     * @param accountId       The account ID of the item to be changed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    Engine users can implement them based on the channel user and room information received as parameters. {        
    }
```

Engine users can implement them based on the channel user and room information received as parameters.

```
    /**
     * A callback that is called when the room status is changed in a different node of the same channel
     * <p>
     * updateChannelRoomInfo() Occurs when called.
     *
     * @param type            Channel information change type(update/delete)
     * @param channelRoomInfo The room information to be changed
     * @param roomId          The Room ID of the item to be changed
     * @throws SuspendExecution This method can suspend Fiber.
     */
    public void onChannelRoomUpdate(ChannelUpdateType type, RoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {        
    }
```

Lastly, the client can request channel information to the server. At this time, the callback method below is called. Engine users can also implement this as they want.

```
    /**
     * A callback that is called when the client requests the Channel information (Base.GetChannelInfoReq)
     *
     * @param outPayload The channel information to be passed to the client
     * @throws SuspendExecution This method can suspend Fiber.
     */
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {        
    }
```



## 7. Matching Group

Like channels, matching group is one of the ways to logically separate a single server family. However, matching group is not explicitly predefined like channels do. In addition, while channel is a way to logically divide GameNode, matching group is a way to logically divide matchmaking. Let's take a look at the user matchmaking callback method again. The matching group configured in this example is "Newbie." This value of "Newbie" can be passed from the client to the server using payload or it can be a pre-stored value for the user by engine users on the server. Users can use the hardcoded value as below, but it is not recommended to use it to actual game server code. Anyway, a single matchmaker is created for the matching group named "Newbie" and all the "Newbie" requests afterward are guaranteed to be stacked in this matchmaker.

```
    /**
     * A callback that is called when the client requests userMatch
     *
     * @param roomType   The type of matched room
     * @param payload    The payload received from the client
     * @param outPayload The payload to pass to the client
     * Returns to @return boolean type. true: user matching request succeeds, false: user matching request fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    public boolean onMatchUser(final String roomType,
                               final Payload payload, Payload outPayload) throws SuspendExecution {

        String matchingGroup = "Newbie";
        SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId());
        sampleUserMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, term, payload);
    }
```

In every protocol related to matchmaking, there is the matching group field. Therefore, it is ideal to predefine the matching groups to use between the client and the server. For example, users can define matching groups based on skill level such as "Beginner", "Intermediate", or "Expert" or define matching groups based on country such as "Korea", "Japan", or "United States." This is entirely up to the engine users.



## 8. MatchNode & MatchMaker

Engine users can apply matchmaking by simply implementing matching logic. Matchmaker allows users to enter the same room based on logic. GameAnvil provides two matchmakers: RoomMatchMaker and UserMatchMaker. These matchmakers are separately run on MatchNode. These MatchNodes cannot implement additional content except matchmaking. Therefore, users can focus on the matchmakers instead of MatchNode.

### Note

*Matchmaker is created in the unit of matching group. In other words, matchmaking is done with same matching groups.*



### 6-1. UserMatchMaker

User matchmaking places game users' matching requests on the queue. It compares and analyzes the content of this request queue at a specific interval and allows arbitrary users to enter a room based on the standard that is desired by the user. Here, engine users can focus on the logic that determines how to compare and analyze the content of the request queue and how to match users. For reference, the most popular user matchmaking game is "*League of Legends*."

The basic of user matchmaking is the matching request itself. Implement these matching requests by inheriting the UserMatchInfo abstract class provided by the engine. The getId() method must be implemented to provide the game user ID that can be used to identify requester. The example below additionally implement the Comparable interface to compare matching requests.

```
public class SampleUserMatchInfo extends UserMatchInfo implements Serializable, Comparable<SampleUserMatchInfo> {

    private int userId;
    private int rating;

    public SampleUserMatchInfo(int userId, int rating) {
        this.userId = id;
        this.rating = rating;
    }

    public int getRating() {
        return rating;
    }

    /**
     * Returns RoomId if it is a party matching request, returns UserId if it is a user matching request.
     *
     * If it is a party matching request to @return int type, returns RoomId, if it is a user matching request, returns UserId.
     */
    @Override
    public int getId() {
        return userId;
    }

    /**
     * If it is a party matching request, returns the party size (number of people), if it is a user matching request, returns 0.
     *
     * If it is a party matching request to @return int type, returns the party size (number of people), if it is a user matching request, returns 0
     */
    @Override
    public int getPartySize() {
        return 0;
    }

    // If SampleUserMatchInfo objects need to be compared, implement the Comparable interface.
    @Override
    public int compareTo(SampleUserMatchInfo o) {
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
```



Now it is time to create the user matchmaker. The user matchmaker inherits the UserMatchMaker abstract class provided by the engine. Especially the match() method needs to be monitored as it is a callback that is called to execute actual matching. The refill() method is a callback that is called when a reinforce request is received for the completed matchmaking. For example, when one of the four users who are matched ends the game, it can be used to add one more user. The example code below shows how to implement this UserMatchMaker.

```
public class SampleUserMatchMaker extends UserMatchMaker<SampleUserMatchInfo> {

    private static final Logger logger = getLogger(SampleUserMatchMaker.class);

    public SampleUserMatchMaker() {
        super(2, 5000);
    }

    private Multiset<SampleUserMatchInfo> ratingSet = TreeMultiset.create();
    private final int matchMultiple = 1; // match How many people will you gather in the multiple of the maximum number before matching them by rating?
    private int currentMultiple = matchMultiple;
    private long lastMatchTime = System.currentTimeMillis();
    private int totalMatchMakings = 0;

    @Override
    public void match() {
        List<SampleUserMatchInfo> matchRequests = getMatchRequests(matchSize * currentMultiple);

        // Not loaded to the minimum number (minAmount)
        if (matchRequests == null) {
            if (System.currentTimeMillis() - lastMatchTime >= 10000)
                currentMultiple = Math.max(--currentMultiple, 1);

            return;
        }

        // The items that are not matched may remain in ratingSet, they do not need to be stored separately.
        // These items are received from the next getMatchRequests().
        ratingSet.clear();
        ratingSet.addAll(matchRequests);

        if (ratingSet.size() >= matchSize) {

            // Consume matchingAmount*matchSize in the order of ratingSets API
            int matchingAmount = matchSingles(ratingSet);

            if (matchingAmount > 0) {
                totalMatchMakings += matchingAmount;
                logger.info("{} match(s) made (total: {}) - {}", matchingAmount, totalMatchMakings, this.getMatchingGroup());

                lastMatchTime = System.currentTimeMillis();
                currentMultiple = matchMultiple;
            }
        }
    }

    @Override
    public boolean refill(SampleUserMatchInfo matchReq) {
        try {
            List<SampleUserMatchInfo> refillRequests = getRefillRequests();

            if (refillRequests.isEmpty()) {
                return false;
            }

            for (SampleUserMatchInfo refillInfo : refillRequests) {
                // Refills if the gap is less than 100 points
                if (Math.abs(matchReq.getRating() - refillInfo.getRating()) < 100) {
                    if (refillRoom(matchReq, refillInfo)) { // Matches the matching request to the room that needs to be refilled
                        return true;
                    }
                }
            }
        } catch (Exception e) {
            logger.error("SampleUserMatchMaker::refill()", e);
        }

        return false;
    }
}
```



The UserMatchMaker and UserMatchInfo that are created in this way must be registered in the Bootstrap step. Note that matchmaker is in the line of game node configuration. In other words, in addition to configuring the GameUser and GameRoom used by GameNode and configure to determine which UserMatchMaker to use to match them.

```
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)
            .room("SampleUserMatchType", SampleGameRoom.class)

            // Register
            .userMatchMaker("MyUserMatchRoomType", SampleUserMatchMaker.class, SampleUserMatchInfo.class);

        bootstrap.run();
    }
}
```



Now the client can request user matchmaking to the server. This request is passed to GameUser and calls the callback method below by the engine. This callback method provides engine users with an opportunity to use not only the user matchmaker provided by GameAnvil but also a third solution. The example below shows calling the matchUser() API to use the user matchmaker provided by the engine. Engine users either create their own matchmaker or use a different library. In this case, implement the callback method below to fit the situation. The thing to pay attention in this example code is matching group. Matching group is a concept that is used to logically divide matchmaking and it will be discussed in detail later in the intermediate concepts. The "Newbie" matching group used in the example shares the same matching queue among the same "Newbie" matching groups.

```
    /**
     * A callback that is called when the client requests userMatch
     *
     * @param roomType   The type of the matched room
     * @param payload    The payload received from the client
     * @param outPayload The payload to pass to the client
     * Returns to @return boolean type. true: user matching request succeeds, false: user matching request fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    public boolean onMatchUser(final String roomType,
                               final Payload payload, Payload outPayload) throws SuspendExecution {

        String matchingGroup = "Newbie";
        SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId());
        sampleUserMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, term, payload);
    }
```



Lastly, the client can cancel the previously requested matchmaking at any time. At this time, the engine calls the callback method of GameUser below if the cancel process succeeds. Engine users can implement what they want to process at the cancel timing.

```
    /**
     * A callback that is called when the client cancels userMatch
     *
     * @param reason Reason of cancel (TIMEOUT/CANCEL)
     * Returns to @return boolean type. true: user matching cancel succeeds false: user matching cancel fails.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
    }
```



### 6-2. RoomMatchMaker

Room matchmaking automatically sends game users to appropriate rooms. How the game users are sent to rooms depends on the implementation of engine users. They can be sent to a room with the most users, or a room with the least users. Or they can be sent to a room with the highest average score. Engine users can focus only on the matching logic. Well-known games that use room matchmaking include “*Hangame Poker*" and "*KartRider*."

- Note> RoomMatchMaker and UserMatchMaker are run independently. In other words, even when the same matching group requests the user matching and room matching separately, these requests are never matched to each other.

The foundation of this room matchmaking is the matching request itself. Implement the RoomMatchInfo interface provided by the engine for these matching requests. The getId() method must be implemented to provide the game user ID that can be used to identify requester.

```
public class SampleRoomMatchInfo implements Serializable, RoomMatchInfo {

    private int roomId;
    private int userCurrentCount = 0;
    private int maxUserCount = 4;

    public SampleRoomMatchInfo() {
    }

    public void setRoomId(int roomId) {
        this.roomId = roomId;
    }

    public int getUserCurrentCount() {
        return userCurrentCount;
    }

    public void setUserCurrentCount(int userCurrentCount) {
        this.userCurrentCount = userCurrentCount;
    }

    public int getMaxUserCount() {
        return maxUserCount;
    }

    /**
     * Obtains RoomId.
     * <p>
     * It is used to mean different things in different situations.
     * <p>
     * If RoomMatchInfo means the information of a matchable Room or a matched Room, it means the RoomId of the Room.
     * <p>
     * If RoomMatchInfo means a matching condition, it returns the RoomId specified by content.
     * <p>
     * It can be used in a way desired by content.
     * <p>
     * ex) The RoomId to be excluded from matching.
     *
     * Returns RoomId to @return int type.
     */
    int getRoomId();
    @Override
    public int getRoomId() {
        return roomId;
    }
}
```



Now it is time to create the room matchmaker. The room matchmaker inherits the RoomMatchMaker abstract class provided by the engine. Especially the match() method needs to be monitored as it is a callback that is called to execute actual matching. The example code below shows how to implement RoomMatchMaker. While UserMatchMaker implements the UserMatchInfo and Comparable interface, RoomMatchMaker provides Comparator. This is to simply show engine users that they can implement it as they desire.

```
public class SampleGameRoomMatchMaker extends RoomMatchMaker<SampleGameRoomMatchRoomInfo> {

    /**
     * Processes the MatchRoom request.
     * <p>
     * Finds the Room that meets the matching condition and returns the matching result.
     * <p>
     * If {@link BaseUser#matchRoom(String, String, RoomMatchInfo)} is called {@link BaseUser#onMatchRoom(String, Payload)}, it is called to process matching.
     * <p>
     * Passes RoomMatchInfo, the matching condition, to terms.
     * <p>
     * The list of RoomMatchInfos obtained using {@link #getRooms()} is the information of matchable Room.
     * <p>
     * Finds matchable Room by comparing this information and terms.
     * <p>
     * If a matchable Room is found, returns the RoomMatchInfo of the Room and returns null if not found.
     *
     * @param terms Matching condition
     * @param args  The data additionally passed
     * Returns the Room information matched with @return {@link RoomMatchInfo}. Returns null if there is no matched items.
     */
    @Override
    public SampleGameRoomMatchRoomInfo match(SampleGameRoomMatchRoomInfo terms, Object... args) {

        int bypassRoomId = terms.getRoomId();
        List<SampleGameRoomMatchRoomInfo> rooms = getRooms();

        for (SampleGameRoomMatchRoomInfo info : rooms) {
            // Excludes the room the user is participating if the moveRoom option is true
            if (info.getRoomId() == bypassRoomId)
                continue;

            if (info.getMaxUserCount() != terms.getMaxUserCount())
                continue;

            if (info.getMaxUserCount() == info.getUserCurrentCount())
                continue;

            return info;
        }

        return null;
    }

    @Override
    public Comparator<SampleGameRoomMatchRoomInfo> getComparator() {
        return new Comparator<SampleGameRoomMatchRoomInfo>() {
            @Override
            public int compare(SampleGameRoomMatchRoomInfo o1, SampleGameRoomMatchRoomInfo o2) {
                return o1.getMaxUserCount() - o2.getMaxUserCount();
            }
        };
    }
}
```



The RoomMatchMaker and RoomMatchInfo created in this way must be registered during the Bootstrap step. Note that they are similar to the user matchmaker. Note that room matchmaker is in the line of game node configuration. In other words, in addition to configuring the GameUser and GameRoom used by GameNode and configure to determine which RoomMatchMaker to use to match them. The user matchmaker and the room matchmaker can be used simultaneously. The example code below shows it by registering the two matchmakers. However, note that the Room and MatchInfo classes used by the two matchmakers are separate.

```
public class Main {

    public static void main(String[] args) {
        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
            .user("SampleGameUserType", SampleGameUser.class)

            .room("SampleGameUserMatchRoomType", SampleUserMatchRoom.class)
            .userMatchMaker("SampleGameUserMatchRoomType", SampleUserMatchMaker.class, SampleUserMatchInfo.class)

            // Register
            .room"SampleGameRoomMatchRoomType", SampleRoomMatchRoom.class)
            .roomMatchMaker("SampleGameRoomMatchRoomType", SampleRoomMatchMaker.class, SampleRoomMatchInfo.class);

        bootstrap.run();
    }
}
```



Now the client can request room matchmaking to the server. This request is passed to GameUser and the callback method is called by the engine. This callback method provides engine users with an opportunity to use a third solution as well as the room matchmaker provided by GameAnvil. The example below shows calling the matchRoom() API to use the room matchmaker provided by the engine. Engine users can create and use matchmaker or use a different library. In this case, implement the callback method below appropriately. The thing to pay attention in this example code is matching group. Matching group is a concept that is used to logically divide matchmaking and it will be discussed in detail later in the intermediate concepts. The "Newbie" matching group used in the example shares the same matching queue among the same "Newbie" matching groups.

```
    /**
     * A callback that occurs when the client requests roomMatch
     *
     * @param roomType The type of matched room
     * @param payload  The payload received from the client
     * Returns the room information matched with @return {@link MatchRoomResult}, if null is returned,   a new Room is created or the request fails according to the client request option.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    public MatchRoomResult onMatchRoom(final String roomType,
                                       final Payload payload) throws SuspendExecution {

        String matchingGroup = "Newbie";

        SampleRoomMatchInfo sampleRoomMatchInfo = new SampleRoomMatchInfo(getChannelId());
        sampleRoomMatchInfo.setMatchingGroup(matchingGroup);
        sampleRoomMatchInfo.setRoomMode(RoomMode.NORMAL);

        return matchRoom(matchingGroup, roomType, terms);
    }
```



## 7. Support Node

SupportNode is a node for executing secondary features. An arbitrary feature can be implemented regardless of game user or room object. For example, logs can be collected and transferred or communicate with the billing server. In addition, it is possible to use the RESTful feature as a simple web server.

This SupportNode basically inherits and implements the BaseSupportNode abstract class. The onDispatch() callback for processing RESTful is provided for the node common callback method.

```
public class SampleSupportNode extends BaseSupportNode {

    // 1.Create REST dispatcher
    private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher<>();

    // 2.Map the URL and handler that the user wants to process in SampleSupportNode
    static {
        // Register a combination of path and method (GET, POST, etc.)
        restMsgHandler.registerMsg("/auth", RestObject.GET, _RestAuthReq.class);
        restMsgHandler.registerMsg("/echo", RestObject.GET, _RestEchoReq.class);
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onPrepare() throws SuspendExecution {
    }

    @Override
    public void onReady() throws SuspendExecution {
    }

    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {
    }

    public abstract boolean onDispatch(RestObject restObject) throws SuspendExecution;


    /**
     * Called when rest call is passed.
     *
     * @param restObject The passed {@link RestObject}
     * Returns to @return boolean type.
     * @throws SuspendExecution This method can suspend Fiber.
     */
    @Override
    public boolean onDispatch(RestObject restObject) throws SuspendExecution {
        // 3.Handle the REST request of SampleSupportNode        
        if (!restMsgHandler.isRegisteredMessage(restObject))
            return false;

        restMsgHandler.dispatch(this, restObject);
        return true;
    }

    @Override
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {}

    @Override
    public void onResume(Payload outPayload) throws SuspendExecution {}

    @Override
    public void onShutdown() throws SuspendExecution {}
}
```



The SupportNode created in this way can be registered during the Bootstrap step as below:

```
public class Main {

    public static void main(String[] args) {

        GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

        // 등록
        bootstrap.setSupport("SampleWebSupport")
            .node(SampleSupportNode.class);

        bootstrap.run();
    }
}
```



## 8. Room Transfer (RoomTransfer)

Room transfer is a feature similar to the user transfer that is explained in the intermediate concepts. It is different only in that room, a bigger concept than user, is transferred. In game node, a room object with one or more game users can be created. This room object can be transferred among game nodes just like user object. For example, the room object of #1 game node can be transferred to the #2 game node. When the room is being transferred, the users in that room are also transferred. For this reason, the game can be played without interruption before and after the room is transferred. As the room is transferred quickly, the users in the room can continue playing the game without noticing the transfer. This technology is the core of GameAnvil's NonStopPatch and it only works when the room object is serialized/de-serialized well. The target of serialize/de-serialize can be directly implemented by GameAnvil users. In other words, the users determine which value will be transferred to the room object.

Only the NonStopPatch command can trigger this type of room transfer. This command is explicitly transferred by the game operator through GameAnvil Console.



### 8-1. Implementing Room Transfer

The actual room transfer is internally processed by GameAnvil. At this time, the client is not likely to recognize the room object being transferred among the servers with game users. Unless a special problem occurs, the overall flow is quickly progressed and the game flow before the transfer can be continued after the transfer.

At this time, when transferring room object to a different game node, the user can specify which data will be transferred as well. Simply specifying the data allows it to be serialized as part of the room object. In the game node to be transferred, the user can easily access those objects without considering de-serialization.

The following are the callback methods related to user transfer. First is "specifying the data to be transferred with." To do this, implement the callback below in the game room class that inherited the BaseRoom abstract class. Simply place the data to be transferred to the passed transferPack as a key-value pair using the key value desired by the user.

If the data to be transferred is not specified at this time, be careful as the data of the room object that is transferred to the target game node is reset to default.

**Note**> *As explained earlier, when room is transferred, the users in the room are transferred as well. As user transfer is explained in the intermediate concepts, it will not covered here.*

```
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("RoomInfo", myRoomInfo);
}
```

Let's take a look at the callback methods that need to be processed by the target game node after the transfer. Users can access the desired object before it is transferred using the specified key. Restore the data of the room object that is transferred in this way. Especially, users can see that the list of the user objects in the room transferred is passed as a parameter. Except this, the overall flow is very similar to that of user transfer.

```
@Override
public void onTransferIn(List<GameUser> userList, TransferPack transferPack) throws SuspendExecution {
    this.users.clear();
    for (GameUser user : userList)
        this.users.put(user.getUserId(), user);

    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setRoomInfo(transferPack.getToLong("RoomInfo"));
}
```



### 8-2. Transferable Room Timer

When transferring room, the registered timer can be transferred as well. To use a transferable timer, it must be separately pre-registered among the callback methods below using a desired key. Map the Timer handler to a desired key. The process is the same as that of user transfer.

```
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferRoomTimerHandler1", transferRoomTimerHandler1());
     registerTimerHandler("transferRoomTimerHandler2", (timerObject, object) -> {
         GameRoom room = (GameRoom) object;
         logger.warn("GameRoom::transferRoomTimerHandler2() : roomId({})", room.getId());
     });
 }
```

The registered timer object can be used in the game user implementation part using the key. The timer that only registered does not take effect, for actual use, the timer must be added to the user object below for it to trigger.

```
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

The timer added to a room object in this way is automatically transferred without processing for a separate transfer. In other words, the same timer adding process does not need to be done to the corresponding room object in the target game node.



## 9. Uninterrupted Maintenance (NonStopPatch)

During maintenance in general, explicitly apply maintenance to entire game service and stop users from playing the game. The needed patch is done afterward. However, if GameAnvil is used, patch can be done without interruption as long as it does not break the compatibility of server binary. This is called the uninterrupted patch and can be done using the GameAnvil Console management tool.

The key of uninterrupted patch is the technology used to transfer users and rooms. Transfer the users and rooms of the game server to be patched to a valid, different game server using these two features and apply the patch when the game server is empty. As this uninterrupted patch needs to be done during service, it must be tested before servicing. In addition, the binary of the server to be patched and the binary of the existing server need to be checked to see if the compatibility between them is broken.

To use uninterrupted patch, the user needs to re-define the related callback methods in the GameNode class.

```
    /**
     * A callback that is called if the node is originating node when starting uninterrupted maintenance
     *
     * Returns whether it is successful to @return boolean type.
     * @throws SuspendExecution This method can suspend Fiber.
     */
@Override
public boolean onNonStopPatchSrcStart() throws SuspendExecution {
    return true;
}

    /**
     * A callback that is called if the node is originating node when ending uninterrupted maintenance
     *
     * Returns whether it is successful to @return boolean type.
     * @throws SuspendExecution This method can suspend Fiber.
     */
@Override
public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}

    /**
     * A callback that is called to determine whether or not to end uninterrupted maintenance if the node is originating node when uninterrupted maintenance is started
     *
     * Returns whether it is successful to @return boolean type.
     * @throws SuspendExecution This method can suspend Fiber.
     */
@Override
public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}

    /**
     * A callback that is called if the node is destination node when starting uninterrupted maintenance
     *
     * Returns whether it is successful to @return boolean type.
     * @throws SuspendExecution This method can suspend Fiber.
     */
@Override
public boolean onNonStopPatchDstStart() throws SuspendExecution {
    return true;
}

    /**
     * A callback that is called if the node is destination node when uninterrupted maintenance ended
     *
     * Returns whether it is successful to @return boolean type.
     * @throws SuspendExecution This method can suspend Fiber.
     */
@Override
public boolean onNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}

    /**
     * A callback that is called to determine whether or not to end uninterrupted maintenance if the node is destination node when uninterrupted maintenance is started
     *
     * Returns whether it is successful to @return boolean type.
     * @throws SuspendExecution This method can suspend Fiber.
     */
@Override
public boolean canNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
```



### 3-1. Uninterrupted Patch Guide

Uninterrupted patch is successful only when the various elements of GameAnvil are working in harmony. Therefore, use it after reading and understanding the guide document below.

#### [Uninterrupted Patch Guide](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-link-nonstop-patch)

## 10. Protocol Definition and Compile

GameAnvil defines and builds protocol using [Google Protocol Buffers](https://developers.google.com/protocol-buffers). The example below explains how to define and build this type of protocol. First, create the SampleGame.proto file using a text editor and define a desired protocol. The detailed syntax of protocol buffer can be referred from [Official Protocol Buffers Guide](https://developers.google.com/protocol-buffers/docs/overview). 

```
package [package name];

// Imports if there is another proto file to be referenced
import [proto file name]

enum SampleUserTypeEnum {
    USER_TYPE_NONE = 0;
    USER_TYPE_ONE = 1;
}

message SampleUserData {
    int32 data = 1;
}

enum SampleRoomTypeEnum {
    ROOM_TYPE_NONE = 0;
    ROOM_TYPE_ONE = 1;
}

message SampleRoomData {
    int32 data = 1;
}

message SampleUserMessage {
    SampleUserTypeEnum sampleUserTypeEnum = 1;
    int32 id = 2;
    int64 money = 3;
    string nickname = 4;
    double score = 5;
    repeated SampleUserData sampleUserData = 6; // list
}

message SampleRoomMessage {
    SampleRoomTypeEnum sampleRoomTypeEnum = 1;
    int32 id = 2;
    int64 potMoney = 3;
    string roomName = 4;
    repeated SampleRoomData sampleRoomData = 5; // list
}
```



The protocol script written with the command line below can be compiled. At this time, it is convenient if a batch file such as build.bat is created. If the project is configured using the GameAnvil template, protocol scripts and batch files can be referenced inside the proto folder in a project. Once compiling is finished, all the Java classes required to process the protocol are automatically created.

```
protoc  ./MyGame.proto --java_out=../java
```



## 11. Configurations

GameAnvil can change the server configuration through the GameAnvilConfig.json file. This configuration file is categorized in 7 items. Except common, each node is configured. At this time, some of the values being configured must be identical to the values used during the Bootstrap step. For example, we configured GameNode as below. "SampleGameService," the ServiceName used here must be configured in GameAnvilConfig.json. If not, the service cannot be found and an error occurs while running server.

```
bootstrap.setGame("SampleGameService")
            .node(SampleGameNode.class)
```



### common

- Essential Common Settings

| Name                             | Description                                                  | Default     |
| ------------------------------- | ------------------------------------------------------------ | ----------- |
| vmId                            | A unique ID that is used when multiple JVMs are run on a single machine (0<=vmId<100) | 0       |
| ip                              | The IP that is commonly used by nodes. Automatically specifies private ip if there is no value for (specifying machine IP) |             |
| meetEndPoints                   | Registers the common IP and communicatePort of the target node, can include the endpoint of the server, can register multiple items using a list |             |
| keepAliveTime                   | communicateNode connection duration (ms)                     | 60000   |
| nodeTimeout                     | Node status check timeout (ms) not checked if 0.             | 180000   |
| msgExpiredTime                  | Message expiration time                                      | 30000   |
| defaultReqTimeout               | Inter node message routing timeout (ms) not checked if 0.    | 30000   |
| defaultAsyncAwaitTimeout        | Timeout (ms)                                   when blocking is called | 30000   |
| ipcPort                         | A port used when communicating with ipc node                 | 11100   |
| publisherPort                   | A port for publish socket                                    | 11101   |
| debugMode                       | An option that is used to prevent various timeouts from occurring while debugging. it must be false in actual implementation. | false       |
| nodeInfoUpdateTime              | An interval for transferring the node information of JVM to a different JVM | 500 (ms) |
| nodeInfoUpdateBusyTime          | The node status is determined as BUSY if node information update lasts for more than this time. | 510 (ms) |
| nodeInfoManagerRefreshTime      | An interval for updating NodeInfoManager using the passed node information. | 100 (ms) |
| loopInfoSamplingCount           | The number of loops to be sampled when checking the status of Message Loop | 100 (times) |
| enableLoopInfoMonitor           | Whether or not to update the loop information from NodeInfoPage | true        |
| maxUserCount                    | The maximum number of Users to be considered as idle when determining IdleNode | 2500   |
| addLocWeight                    | The multiplier that is used to compensate information while assigning UserLoc. Ex) If there are 8 game nodes, it arbitrarily assigns 8 users even though only 1 user is assigned to them. | 1       |
| allocatorType                   | bytebuf type 1 to be used (POOLED_DIRECT_BUFFER) 2 (POOLED_HEAP_BUFFER) 3 (UNPOOLED_DIRECT_BUFFER) 4 (UNPOOLED_HEAP_BUFFER) | 4       |
| httpClientOption                | The httpClientOption used by Gameflex                        |         |
| httpClientOption-keepAlive      | Whether or not to maintain the httpClientOption connection   | true    |
| httpClientOption-connTimeout    | httpClientOption connection timeout                          | 5000    |
| httpClientOption-requestTimeout | httpClientOption request timeout                             | 10000    |
| httpClientOption-maxConnPerHost | The maximum number of connections per httpClientOption host  | 10000    |
| httpClientOption-maxConn        | httpClientOption connection maximum number                   | 12000    |
| moduleURL                       | The information of the URL to read the config and dynamicModule information, the module information is called using the URL if the option is configured. | ""       |
| configPath                      | The path that is used to read the Json file of config when it is read as a file, ignored when moduleURL is configured | /config |

- Usage example

```
"common": {
    "ip": "127.0.0.1",
    "meetEndPoints": ["127.0.0.1:16000"],
    "keepAliveTime": 10000,
    "nodeTimeout": 180000,
    "msgExpiredTime:":30000,
    "defaultReqTimeout": 30000,
    "defaultAsyncAwaitTimeout": 30000,
    "communicatePort": 16000,
    "publisherPort" : 13300,
    "debugMode": false,
    "allocatorType": 1,
    "nodeInfoUpdateTime": 10,
    "nodeInfoUpdateBusyTime": 12,
    "nodeInfoManagerRefreshTime": 4,
    "loopInfoSamplingCount": 5,
    "maxLoopTime" : 10,
    "enableLoopInfoMonitor" : true,
    "maxUserCount" : 2500,
    "addLocWeight": 16,
    "httpClientOption": {
        "keepAlive": true,
        "connTimeout": 5000,
        "requestTimeout": 10000,
        "maxConnPerHost" : 10000,
        "maxConn": 12000
    }
},
```



### location

- Does not create LocationNode if there is no configuration value or clusterSize is 0.

| Name        | Description                                                         | Default |
| ----------- | ------------------------------------------------------------ | ------ |
| clusterSize | The number of configured servers (VM)                                       |        |
| replicaSize | The size of the duplicated group, the number of masters + slaves                       |        |
| shardFactor | The factor for sharding - the total number of shards = clusterSize x replicaSize x shardFactor - The number of shards to be run in a single machine (VM) = replicaSize x shardFactor - the total number of unique shards (the number of master shards) = clusterSize x shardFactor |        |

- Usage example

```
"location": {
    "clusterSize": 1,
    "replicaSize": 3,
    "shardFactor": 3
},
```



### match

- Does not create MatchNode if there is no configuration value or nodeCnt is 0.

| Name    | Description            | Default |
| ------- | --------------- | ------ |
| nodeCnt | The number of match nodes |        |

- Usage example

```
"match": {
    "nodeCnt": 3
},
```



### gateway

- Does not create GatewayNode if there is no configuration value or nodeCnt is 0.

| Name                                     | Description                                                  | Default |
| ---------------------------------------- | ------------------------------------------------------------ | ------- |
| nodeCnt                                  | Number of nodes                                              |         |
| ip                                       | Sets automatically as private ip if there is no IP value that is linked to the client |         |
| dns                                      | The domain address linked to the client                      |         |
| maintenance                              | Whether or not to check when starting server                 | false   |
| checkWhiteListBeforeOnAuth               | Whether or not to check WhiteList before calling the onAuthenticate callback | true    |
| connectGroup                             | TCP_SOCKET WEB_SOCKET                                        |         |
| connectGroup - port                      | The port linked to the client                                |         |
| connectGroup - idleClientTimeout         | The timeout after no data transfer, not used if 0            | 4000    |
| connectGroup - secure                    | Not used if there is no security setting value.              |         |
| connectGroup - secure - useSelf          | Whether or not to use security settings regardless of the keyCertChainPath and privateKeyPath value | false   |
| connectGroup - secure - keyCertChainPath | Certificate path - Started from the Dsecure setting path     |         |
| connectGroup - secure - privateKeyPath   | Private key path - Started from the Dsecure setting path     |         |
| duplicateLoginServices                   | Specify the service that allows multiple logins              |         |

- Usage example

```
"gateway": {
    "nodeCnt": 4,
    "ip": "127.0.0.1",
    "dns": "",
    "maintenance": false,
    "checkWhiteListBeforeOnAuth": true,
    "connectGroup": {
        "TCP_SOCKET": {
            "port": 11200,
            "idleClientTimeout": 0,
            "secure": {
                "useSelf": true,
                "keyCertChainPath": "gameflex.crt",
                "privateKeyPath": "privatekey.pem"
            }
        },
        "WEB_SOCKET": {
            "port": 11400,
            "idleClientTimeout": 0
        }
    },
    "duplicateLoginServices": {
        "RPSGame": ["GroupChatting"],
        "GroupChatting": ["SampleGameService"],
        "ForTestCase": []
    }
},
```



### game

- Does not create GameNode if there is no configuration value or nodeCnt is 0. As explained earlier, the ServiceName used during the Bootstrap step can be configured here.

| Name        | Description                                                  | Default |
| ----------- | ------------------------------------------------------------ | ------- |
| nodeCnt     | Number of nodes                                              |         |
| serviceId   | Service ID                                                   |         |
| serviceName | Service name                                                 |         |
| channelIDs  | The channel ID to be assigned to each node. They do not have to be unique. Can be used duplicate without distinguishing channels with the "" character string |         |
| userTimeout | User object removal timeout after disconnect                 | 10000   |

- Usage example

```
"game": [
    {
        "nodeCnt": 8,
        "serviceId": 0,
        "serviceName": "SampleGameService",
        "channelIDs": ["ch1","ch1","ch2","ch2","ch3","ch3","ch4","ch4"],
        "userTimeout": 3000
    },
    {
        "nodeCnt": 4,
        "serviceId": 1,
        "serviceName": "GroupChatting",
        "channelIDs": ["", "", "", ""],
        "userTimeout": 4000
    }
],
```



### support

- Does not create SupportNode if there is no configuration value or nodeCnt is 0. Like GameNode, SupportNode must have the ServiceName used during the Bootstrap step configured.

| Name                             | Description                                                  | Default |
| -------------------------------- | ------------------------------------------------------------ | ------- |
| nodeCnt                          | Number of nodes                                              |         |
| serviceId                        | service ID                                                   |         |
| serviceName                      | service name                                                 |         |
| restIp                           | Automatically configures to private IP if there is no set rest IP value |         |
| restPort                         | rest port                                                    |         |
| restPermissionGroups             | Registers a specific Path and IP to allow the ACL setting. Allows all the connections if it is empty. |         |
| restSecure                       | It is not used if there is no set value                      |         |
| restSecure - keyCertChainPath    | Certificate path - Started from the Dsecure setting path     |         |
| restSecure - privateKeyPath      | Private key path - Started from the Dsecure setting path     |         |
| restSecure - authorizationSecret | Required only when authentication token (JWT) is used, encryption signature key |         |
| restSecure - authorizationPath   | Required only when authentication token (JWT) is used, the path on which authentication will be performed |         |
| restCorsAllowDomains             | CORS allowed domain Ex -127.0.0.1:1234                       |         |

- Usage example

```
"support": [
    {
        "nodeCnt": 2,
        "serviceId": 3,
        "serviceName": "PLACE",
        "restIp": "127.0.0.1",
        "restPort": 17260,
        "restPermissionGroups": {
            "/": []
        },
        "restSecure": {
            "keyCertChainPath": "",
            "privateKeyPath": "",
            "authorizationSecret": "authSecret",
            "authorizationPath": "/auth"
        },
        "restCorsAllowDomains":[
        ]
    },
    {
        "nodeCnt": 2,
        "serviceId": 4,
        "serviceName": "DB",
        "restIp": "127.0.0.1",
        "restPort": 17251,
        "restPermissionGroups": {
            "/": []
        }
    },
    {
        "nodeCnt": 2,
        "serviceId": 5,
        "serviceName": "SampleWebSupport",
        "restIp": "127.0.0.1",
        "restPort": 17250
    }
],
```



### management

- Does not create ManagementNode if there is no configuration value or nodeCnt is 0.

| Name          | Description                                                  | Default                         |
| ------------- | ------------------------------------------------------------ | ------------------------------- |
| nodeCnt       | Number of nodes                                              |                                 |
| restIp        | Automatically configures to private IP if there is no set rest IP value |                                 |
| restPort      | rest port                                                    |                                 |
| db            | The DB setting that is used by management                    |                                 |
| db - user     | db connection ID                                             |                                 |
| db - password | db connection password                                       |                                 |
| db - url      | db connection url - h2db : jdbc:h2:mem:gameflex_admin;DB_CLOSE_DELAY=-1 - mysql : jdbc:mysql://localhost:3306/gameflex_admin?useSSL=false - mariadb : jdbc:mariadb://localhost:3306/gameflex_admin?useSSL=false |                                 |
| db - driver   | db connection driver- h2db : org.h2.Driver- mysql : com.mysql.jdbc.Driver- mariadb : org.hibernate.dialect.MySQLDialect | org.h2.Driver                   |
| db - dialect  | db connection dialect- h2db : org.hibernate.dialect.H2Dialect- mysql : org.hibernate.dialect.MySQLDialect- mariadb : org.hibernate.dialect.MySQLDialect | org.hibernate.dialect.H2Dialect |
| db - ddlAuto  | ddl option - update: Automatically creates Table - none: Does not automatically create Table. | update                          |

- Usage example

```
"management": {
    "nodeCnt": 2,
    "restIp": "127.0.0.1",
    "restPort": 25150,

    "db": {
        "user": "root",
        "password": "1234",
        "url": "jdbc:h2:mem:gameflex_admin;DB_CLOSE_DELAY=-1"
    }
}
```



## 12. Asynchronous Support API

GameAnvil provides the Async class for the asynchronous process on Fiber as explained earlier. General blocking/non-blocking calls can be Fibered if the Async class through the import command.

```
import com.nhn.gameanvil.async.Async;
```

*The description of all APIs can be found at [GameAnvil API Reference](http://10.162.4.61:9090/gameanvil) in a JavaDoc document.*

The API for calling can be categorized as call and run, and they are used for when there is no return value or there is a return value. In addition, the thread-based future can be converted so that they can use Fiber. The usage for each purpose will be explained in the next part of this document.

### **12-1.Blocking Call Process**

Blocking call in general means thread blocking. In other words, it blocks not only the Fiber executed by code but also the thread used to schedule this Fiber. This means that the node will soon stop, never directly use the blocking calls. GameAnvil provides Async API that is used to convert these thread-blocking calls into Fiber blocking. This API processes the blocking call using external executor and converts the code flow after the completion into Fiber. Use either runBlocking() or callBlocking() depending on the existence of return value. As explained in the basic concepts, these Fiber-blocking APIs can be suspended, the API calling methods must specify the SuspendExecution exception signature.

```
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() throws SuspendExecution {

    Async.runBlocking(executor, runnable); // thread blocking calls into blocking calls

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() throws SuspendExecution {

    return Async.callBlocking(executor, callable);  // thread blocking calls into blocking calls

}
```

### **12-2.Future Handling**

The wait on Future triggers thread blocking. For example, the code below blocks call threads.

```
Future<SomeObject> future = someAsyncJob();

SomeObject ret = future.get(); // Triggers thread blocking
```

GameAnvil provides API that is used to convert this wait on Future from thread blocking to Fiber blocking. However, these APIs support the ListenableFuture of Java's CompletableFuture and Guava. Fortunately, most libraries support asynchronization based on these two futures, there won't be a problem applying them. The code below is an example of processing the wait on future to Fiber blocking using these Async APIs.

```
Future<SomeObject> future = someAsyncJob();

SomeObject ret = Async.awaitFuture(future); // Blocks only the corresponding Fiber
```

### **12-3.Delegation of Blocking Handling**

The process for blocking calls is covered earlier. The runBlocking() or callBlocking() of Async API is used only when continuing the execution flow of the corresponding Fiber after blocking process is finished. However, if the user does not need to concern after delegating blocking calls to an external thread, the execution flow can be continued. In this case, use the API below. This API does not wait for the result of blocking call and cannot be suspended.

```
import com.nhn.gameanvil.async.Async;

void runningBlockingMethod() { // NOT suspendable

    Async.exec(executor, runnable); // As blocking calls are delegated to the external thread, this Fiber is not blocked.

}
import com.nhn.gameanvil.async.Async;

int callingBlockingMethod() {  // NOT suspendable

    return Async.exec(executor, callable);  // As blocking calls are delegated to the external thread, this Fiber is not blocked.

}
```



## 13. Asynchronous Redis Support

Previously, it was entirely GameAnvil user's responsibility to determine which type of Redis client to be used. However, it is learned that the type and complexity of the issues related to Redis increase proportionate to the difference in the type and usage of the Redis client selected by the user. To prevent this, we decided to include the information on how to use the Redis client to the basic guideline of GameAnvil. This guideline aims to consolidate the Redis clients supported by GameAnvil by turning the Redis clients and the basic usage into API. Of course, it is possible to use a Redis client other than the provided API, this should be avoided if possible.

### **[Note]**

*To distinguish the Lettuce class provided by GameAnvil and the product name "Lettuce," the former is designated as "Lettuce class," but it can be designated as plain "Lettuce" if necessary. To distinguish it from the other Lettuce, the product name is written as ***LETTUCE***. The LETTUCE used in this document is focused on the usage on GameAnvil, if more information is needed, refer to [LETTUCE Official Page](https://github.com/lettuce-io/lettuce-core).*

GameAnvil recommends using LETTUCE as the Redis client. The Redis wrapping API provided by GameAnvil uses LETTUCE as well. For reference, LETTUCE is as an asynchronous Redis client and the most of asynchronous APIs are based on CompletableFuture. This means that GameAnvil's Async API can be converted into Fiber-based asynchronization.

The Redis wrapping API provided by GameAnvil can be categorized into the Lettuce, RedisCluster, and RedisSingle classes. Lettuce provides the most common usage and it is a static class that does not internally manage the LETTUCE object. Therefore, if the user wants to use LETTUCE in the most common form, this Lettuce class is the most proper. RedisCluster and RedisSingle are the classes for responding to the Redis cluster and standalone and they internally manage the LETTUCE objects.

### **13-1.Lettuce**

The Lettuce class provides the most essential static APIs used to process Fiber-units. Internally, no status related to Redis are stored, it can be used without having to create a separate object. If the user is familiar with the Lettuce library, it is best to use the Lettuce class directly.

```
import com.nhn.gameanvil.async.redis.Lettuce;
```

The basic Lettuce usage can be maintained except the following three cautions.

- First, connect must use the Lettuce.connect() or Lettuce.connectAsync() of GameAnvil. As connection basically blocks threads, it includes the Fiber process.
- Second, shutdown must use Lettuce.shutdown() as well for the same reason with connection.
- Third, the wait for RedisFuture must use Lettuce.awaitFuture() to block Fiber.

The code that is used to connect to Redis using the Lettuce class.

```
RedisURI clusterURI = RedisURI.Builder.redis(IP_ADDRESS, 7500).build();
clusterClient = RedisClusterClient.create(Arrays.asList(clusterURI));
clusterConnection = Lettuce.connect(RpsConfig.DB_THREAD_POOL, clusterClient);

if (clusterConnection.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### **13-2.RedisCluster**

```
import com.nhn.gameanvil.async.redis.RedisCluster;
```

Wraps the API for Redis Cluster. The usage is basically not different from the usage for Lettuce. However, this class manages the objects related to Lettuce (e.g. RedisClusterClient, StatefulRedisClusterConnection etc.) on its own. Consider using this API if the user wants to manage these Lettuce objects through RedisCluster.

The cautions to be observed are the same with Lettuce. Below is the code that is used to connect to Redis using RedisCluster.

```
redisClient = RedisSingle.create("redis://IP_ADDRESS:6379");
redisAsyncCommands = Lettuce.connect(RpsConfig.DB_THREAD_POOL, redisClient).async();

if (redisClient.isOpen()) {
    logger.info("============= Connected to Redis using Lettuce =============");
}
```

### **13-3.RedisSingle**

```
import com.nhn.gameanvil.async.redis.RedisSingle;
```

When compared to RedisCluster, the only difference is that the target Redis is standalone.

The cautions to be observed are the same with Lettuce. Below is the code that is used to connect to Redis using RedisCluster.

```
redisCluster = new RedisCluster<>(IP_ADDRESS, 7500);
redisCluster.connect(RpsConfig.DB_THREAD_POOL, StringCodec.UTF8);

if (redisCluster.isConnected()) {
    logger.warn("============= Connected to Redis using Lettuce =============");
}
```

### **13-4.Using RedisFuture in Fiber**

Lettuce, RedisCluster, and RedisSingle provide the API that can make the RedisFuture supported by the Lettuce library wait on Fiber. As the internal implementation uses the Async.awaitFuture() provided by the engine, they can be interchangeably used. The four code below are the same code. The get() of the RedisFuture on the Fiber of GameAnvil must use one of the four methods.

- Async.awaitFuture()

```
try {
    Async.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- Lettuce.awaitFuture()

```
try {
    Lettuce.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- RedisCluster.awaitFuture()

```
try {
    redisCluster.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- RedisSingle.awaitFuture()

```
try {
    redisSingle.awaitFuture(clusterAsyncCommands.mget("testKey", getUserId()));
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

- **Incorrect usage**: If directly wait for Future, the Node(Thread) is blocked, so never use the code below.

```
try {
    RedisFuture future = clusterAsyncCommands.mget("testKey", getUserId()));
    future.get(); // 스레드 블로킹을 유발
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin() - timeout", e);
}
```

### **13-5.set/get**

Set and get, the most basic components, are provided by RedisCluster and RedisSingle by default.

- An example of set/get using RedisCluster

```
String setResult = redisCluster.set(key, value);
String getResult = redisCluster.get(key);
```

- An example of set/get using RedisSingle

```
String setResult = redisSingle.set(key, value);
String getResult = redisSingle.get(key);
```

- An example of directly using the RedisAsyncCommands of LETTUCE

```
RedisFuture<String> setFuture = redisAsyncCommands.set(key, value);
RedisFuture<String> getFuture = redisAsyncCommands.get(key);

String setResult = Async.awaitFuture(setFuture);
String getResult = Async.awaitFuture(getFuture);
```

### **13-6.Asynchronous Process of LETTUCE**

The various commands provided by Redis can be used through the Commands object of LETTUCE. Basically, LETTUCE provides the Commands object in Sync and the Commands object in Async. GameAnvil recommends using the Async method. Basically, AsyncCommands for Redis Cluster and StandAlone are as below:

- RedisAdvancedClusterAsyncCommands
- RedisAsyncCommands

The examples below use this AsyncCommands object to perform mget. The asynchronous process of LETTUCE uses RedisFuture by default. This RedisFuture is CompletableFuture. The detailed information of CompletableFuture can be found at [Java Official Reference](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html). For your information, the examples below show only a part of the asynchronous process for LETTUCE, adjust it to fit the code in development rather than using it as it is. For complete control of asynchronous code, understand the information on [LETTUCE](https://github.com/lettuce-io/lettuce-core) and [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html).

### **[Note]**

thenApply(), thenAccept() and others are called by arbitrary external thread, do not access the internal resources managed by Node or use Lock on such resources.

- Example 1> Asynchronously obtains the value for key1 and key2

```
Lettuce.awaitFuture(asyncCommands.mget("key1", "key2"));
```

- Example 2> Delegates the process to an external thread with future chain if it does not affect the code flow afterward (in other words, when it does not have to wait until obtaining the value using mget)

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

future.thenApplyAsync(r -> {
    Map<String, String> map = new HashMap<>();
    for (KeyValue<String, String> kv : r)
        map.put(kv.getKey(), kv.getValue());
    return map;
}).thenAccept(r -> {
    for (Entry<String, String> entry : r.entrySet())
        logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
});
```

- Example 3> Waits for the future and processes if it does not affect the code flow afterward

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

CompletionStage<Map<String, String>> cs = future.thenApplyAsync(r -> {
    Map<String, String> map = new HashMap<>();
    for (KeyValue<String, String> kv : r)
        map.put(kv.getKey(), kv.getValue());
    return map;
});

// do something here

try {
    // Remember that Lettuce.awaitFuture() must be used to have the future wait on Fiber
    Map<String, String> map = Lettuce.awaitFuture(cs);

    for (Entry<String, String> entry : map.entrySet())
        logger.warn("CompletableFuture Test =====> Key: {}, Value: {}", entry.getKey(), entry.getValue());
} catch (TimeoutException e) {
    logger.error("GameUser::onLogin()", e);
}
```



## 14.  How to Use Asynchronous HttpRequest & HttpResponse

Like Redis, GameAnvil provides basic API and guideline for processing HTTP. Of course, the user can choose a different HTTP usage, it is not recommended unless there is a special reason. GameAnvil internally uses [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client) to use async-based HTTP. It is recommended to directly use [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client) rather than using the API provided by us if the API that will be explained later and the range of usage exceed. As AsyncHttpClient internally uses CompletableFuture like LETTUCE, use Async.awaitFuture() to Fiber for future wait. The rest are the same as using it on a thread.

```
Async.awaitFuture(future.get()); // Waits the corresponding future on Fiber.
```

The HTTP API provided by GameAnvil consists of the HttpRequest and HttpResponse class for requests and responses and the HttpResultTemplate class for common process for the result. HTTP requests and responses can be intuitively processed if these classes are used and the result can be taken in any form. And because all code are asynchronous, there is nothing to do anything with it. The following are examples of using this.

- Example 1> The most basic usage. It is the most intuitive method as it automatically takes care of the future in Fiber unit. It is enough only using this unless there is a special reason.

```
HttpRequest request = new HttpRequest(URL);
HttpResponse response = request.GET();
```

- Example 2> The asynchronous method based on future. If another task needs to be done between HTTP request and response, the user can directly use future as below:

```
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

// Do something here

HttpResponse response = new HttpResponse(Async.awaitFuture(future, 10000, TimeUnit.MILLISECONDS));
```

- Example 3> HTTP request header configuration. AsyncHttpClient provides a variety of APIs as shown in the example. For detailed information on the usage of AsyncHttpClient, visit the [official page](https://github.com/AsyncHttpClient/async-http-client).

```
HttpRequest request = new HttpRequest(url);
request.getBuilder()
    .addHeader("if-none-check-node", "false")
    .setRequestTimeout(timeout);
    .addQueryParam("serviceId", serviceId);

HttpResponse httpResponse = request.GET();
```

- Example 4> Delegates the external thread to future chain if it does not affect the code flow afterward (The same method as Lettuce)

```
HttpRequest request = new HttpRequest("abc");
CompletableFuture<Response> future = request.GETAsync();

future.thenApplyAsync(r -> {
    try {
        HttpResponse response = new HttpResponse(r);
        String body = response.getContents(String.class);
        HttpResultTemplate<JsonObject> result = GameAnvilUtil.Gson().fromJson(body, new RestResponseParamType(JsonObject.class));
        if (!result.getHeader().getIsSuccessful()) {
            logger.warn("GET failed : resultCode {}, resultMessage {}",
                result.getHeader().getResultCode(),
                result.getHeader().getResultMessage());
        }
        return result.getContents();
    } catch (IOException e) {
        logger.error("Exception occurred: ", e);
        return null;
    }
}).thenAccept(r -> {
    if (r != null) {
        JsonElement element = r.get(ELEMENT_NAME);
        if (element != null) {
            logger.info("The response code: {}", element.getAsString());
        }
    }
});
```

- Example 5> Waits for the future and processes if it does not affect the code flow afterward

```
RedisFuture<List<KeyValue<String, String>>> future = asyncCommands.mget("key1", "key2");

CompletionStage<JsonObject> cs = future.thenApplyAsync(r -> {
    try {
        HttpResponse response = new HttpResponse(r);
        String body = response.getContents(String.class);
        HttpResultTemplate<JsonObject> result = GameAnvilUtil.Gson().fromJson(body, new RestResponseParamType(JsonObject.class));
        if (!result.getHeader().getIsSuccessful()) {
            logger.warn("GET failed : resultCode {}, resultMessage {}",
                result.getHeader().getResultCode(),
                result.getHeader().getResultMessage());
        }
        return result.getContents();
    } catch (IOException e) {
        logger.error("GameUser::onLogin()", e);
        return null;
    }
});

// do something here

try {
    // Remember that to make the future wait on Fiber, use Async.awaitFuture().
    JsonObject jsonObject = Async.awaitFuture(cs);
    if (jsonObject != null) {
        JsonElement element = jsonObject.get(ELEMENT_NAME);
        if (element != null) {
            logger.info("The response code: {}", element.getAsString());
        }
    }
} catch (TimeoutException e) {
    logger.error("Exception occurred: ", e);
}
```



## 15. RDBMS Asynchronous Process

The query for RDBMS is generally blocking. The way to process this blocking query on GameAnvil is not that different from the usage of other Asyncs. Regardless of the type of RDBMS used, the code for the SQL query can be implemented in the same way. And engine users can freely choose from SQL Mapper, ORM, or others to access DB.



### Note

*The most important thing while implementing the query for DB but often overlooked is the size of CP (ConnectionPool) for DB, the number of TPs (ThreadPool) to asynchronously process them, and understanding of the relationship between them. These two values are usually set the same, considering the amount of queries to be processed or set TP slightly larger than CP. For your information, in a large scale performance test using GameAnvil, setting TP for 6000~8000 people and CP for 250 per server process showed the optimal result. It is important to find the optimal value by running as many tests as possible considering the complex elements such as the complexity and frequency of query.*



### 15-1.Async Query

The asynchronous process for query can be divided in two cases: Case in which the query result is needed and the case in which the query result is not needed. In both cases, the corresponding Fiber waits until the query process is finished; the only difference is whether or not there is the result for such a query. In other words, as the code is processed after the query requested asynchronously is finished, engine users can implement it as if they are writing general blocking code.



First, when the user wants to obtain the query result, they need to use the callBlocking API of the Async class as shown in the example below. callBlocking calls an arbitrary blocking and returns the result.

```
try {
    return Async.callBlocking("MyThreadPool", new Callable<List<T>>() {
        @Override
        public List<T> call() throws Exception {
            return myQueryCode();
        }
    });
} catch (TimeoutException e) {
    logger.error("TimeoutException occured: ", e);
}

logger.info("Query has finished.");
```

At this time, the thread pool for processing asynchronously can be created during the Bootstrap step.

```
bootstrap.createExecutorService("MyThreadPool", 250);
```

Or if it is necessary, engine users may use the external thread pool directly created.

```
bootstrap.createExecutorService(myExecutorService, 250);
```



Second, when the user does not need to obtain the query result, they need to use the runBlocking API of the Async class as shown in the example below. runBlocking calls arbitrary blocking on Fiber.

```
try {
    Async.runBlocking("MyThreadPool", new Runnable() {
        @Override
        public void run() {
            try {
                myQueryCode();
            } catch (Exception e) {
                logger.error("Exception occured during query code: ", e);
            }
        }
    });
} catch (TimeoutException e) {
    logger.error("TimeoutException occured: ", e);
}

logger.info("Query has finished.");
```

In this case, the arbitrary thread pool can be passed to runBlocking API as a parameter.
