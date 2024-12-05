## Game > GameAnvil > Server Development Guide > GameNode Implementation



## Game Node

![GameNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gamenode_on_network.png)

GameNode is a node where real game objects are created and game content is implemented. Clients must complete authentication on GatewayNode and then log in to GameNode before they can start such game content.

As shown in the image below, the client can create multiple logical sessions based on connections that have been authenticated with its own AccountId. A session is created between the client and the user object. In the image below, a red dotted arrow indicates such a session.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

Each session can be separated by a unique value within that connection, which we call SubId. In other words, it is a combination of AccountId and SubId, allowing users to create as many sessions as they want. In this case, you can create as many sessions for the same service as you want. That is to say, a connection with AccountId of 1 in the image can create as many additional sessions for the "Game" service or the "Chat" service as you want.

These sessions are directed to user objects. GameNode manages these user objects and their group, room objects. This chapter describes these GameNode, GameUser and GameRoom.


## GameNode

GameNode inherits and implements the BaseGameNode class. The example code below shows a callback method that can be overridden by default by GameNode. In addition to the node common callback, there is a callback for channel management.

All nodes require a dispatcher generation and message handler registration process to handle custom messages. Especially for GameNode, this process is mandatory for game content.  First, (1) create a static packet dispatcher. At this time, it has to be statically generated so that it can take advantage of memory or performance. (2) And connect it to [Handler](server-impl-07-message-handling.md#process-general-messages) which has implemented the message you want to process. (3) Lastly, MessageDispatcher processes packets using the dispatcher generated in (1).

You can take a look at this set of processes in the example code below, just below the annotations corresponding to (1) ~ (3).

It also registers these implemented classes with engine for specific services using the @ServiceName annotation. Only one GameNode class can be registered for one service.

```java
@ServiceName("MyGame") // Register with engine as GameNode for a service called "MyGame" 
public class SampleGameNode extends BaseGameNode {  
  
    // (1) Create packet dispatcher    
    private static final MessageDispatcher<SampleGameNode> messageDispatcher = new MessageDispatcher<>();  
  
    // (2) Map handlers and protocols that SampleGameNode wants to handle 
    static {  
        messageDispatcher.registerMsg(SampleGame.GameNodeTest.class, _GameNodeTest.class);  
    }  
  
    @Override  
    public MessageDispatcher<SampleGameNode> getMessageDispatcher() {  
        return messageDispatcher;  
    }  
  
    @Override  
    public void onInit() throws SuspendExecution {  
    }  
  
    @Override  
    public void onPrepare() throws SuspendExecution {  
    }  
  
    /**  
     * Called when paused  
     *  
     * @param payload Additional information that contents want to convey 
     * @throws SuspendExecution This method means that the fiber can be suspended  
     */  
    @Override  
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {  
    }  
  
    /**  
     * Called when resumed  
     *  
     * @param payload Additional information that contents want to convey 
     * @throws SuspendExecution This method means that the fiber can be suspended  
     */  
    @Override  
    public void onResume(Payload payload) throws SuspendExecution {  
    }  
    
    /**  
     * Called when a shutdown command is received 
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onShutdown() throws SuspendExecution {  
    }  
  
    /**  
     * Call when a user change occurs on another node on the same channel  
     * i.e , updateChannelUser() Occurs when calling API   
     *  
     * @param type            Forward Channel Information Change Type (Renew/Delete).  
     * @param channelUserInfo Forward User information to be changed.  
     * @param userId          Forward UserId of Change target.  
     * @param accountId       Forward Account Id of Change Target.  
     * @throws SuspendExecution This method means that the fiber can be suspended 
 
     */  
    @Override  
    public void onChannelUserInfoUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {    
    }  
  
    /**  
     * Call when room state changes occur on other nodes on the same channel 
     * i.e , updateChannelRoomInfo() Occurrs when calling API   
     *  
     * @param type            Forward Channel Information Change Type (Renew/Delete). 
     * @param channelRoomInfo Forward Room information to be changed.  
     * @param roomId          Forward RoomId of Change target.  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onChannelRoomInfoUpdate(ChannelUpdateType type, ChannelRoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {    
    }  
  
    /**  
     * Called when client requests channel information 
     *  
     * @param outPayload Forward the Channel information to be forwarded Client.      
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {    
    }  
}
```

The primary purpose of these GameNodes is to perform processing for all GameUser and GameRoom objects connected to the node, which we'll discuss in a moment.

Refer to the table below for the meaning and purpose of callbacks, excluding common callbacks.


| Callback name               | Description                  | Description                                                                                                                                                               |
| ------------------------- | ----------------------- |------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onChannelUserInfoUpdate | Update the channel's user information | When the user information of channel is changed in one of several GameNodes grouped in the same channel, it is called for synchronization in all the remaining GameNodes in the same channel. At this time, the user may update the channel information of the current GameNode based on the received information. |
| onChannelRoomInfoUpdate | Update the room information of the channel   | Among several GameNodes grouped in the same channel, when the room information of the channel is changed in one GameNode, all the remaining GameNodes in the same channel are called for synchronization. At this time, the user may update the channel information of the current GameNode based on the received information.  |
| onChannelInfo           | Request for channel information        | It is called when the client requests channel information. The user can configure the channel information as desired from this callback and forward it to the client.                                                                                     |



## User Implementation

User objects are created in GameNode after login process. It is recommended that user-based content be implemented centered around this class. As in all of the examples explained above, users can also associate their own messages with handlers to be processed. If you look at the example code below, you'll see that the user offers quite a few callback methods. Some of which come with base implementations, so you don't have to override them unless you're in a situation that's especially necessary. This is true for most classes offered by engine as well as the user.

In particular, the user also requires more annotations to register with engine than other classes. First, register the user type after registering the user for which game service it is used. This user type, as its name implies, is used to call the API on the client as well. This is to specify which user type of the server the API is calling. Therefore, you must pre-define these user types between the server and the client as random string. This example code uses the user type "BasicUser". We will explain this further in a separate chapter. Finally, you can decide whether this user information will be used for [Synchronizing user information between channels](server-impl-09-channel#3-채널-정보-동기화). If synchronizing information between channels is not required, you may skip the @UseChannelInfo annotation.

```java
@ServiceName("MyGame") 
@UserType("BasicUser") 
@UseChannelInfo 
public class SampleGameUser extends BaseUser { 
 
    private static final MessageDispatcher<SampleGameUser> messageDispatcher = new MessageDispatcher<>(); 
 
    static { 
        messageDispatcher.registerMsg(SampleGame.GameUserTest.class, _GameUserTest.class); 
    } 
 
    @Override 
    public MessageDispatcher<SampleGameUser> getMessageDispatcher() { 
        return messageDispatcher; 
    } 
 
    /** 
     * Called when login 
     * 
     * @param payload        Information received from the client
     * @param sessionPayload Payload receivedf from onPreLogin 
     * @param outPayload     Information to be forwarded to client 
     * @return If successful login return true, if not return false 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public boolean onLogin(final Payload payload, 
                           final Payload sessionPayload, 
                           Payload outPayload) throws SuspendExecution {     
    } 
 
    /** 
     * Call for post-processing required after successful login (i.e., call after successful onLogin or onReLoin)     * 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 

    @Override 
    public void onPostLogin() throws SuspendExecution { 
 
    } 
 
    /** 
     * Call when the same user already logged in logs in to another device  
     * 
     * @param newDeviceId           deviceId value of newly connected user
     * @param outPayloadForKickUser Payload to be forwarded to clients
     * @return If the return value is true, the existing user is forced to log out after the newly connected user is logged in. If it is false, the newly connected user fails to log in
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */ 
    @Override 
    public boolean onLoginByOtherDevice(final String newDeviceId, 
                                        Payload outPayloadForKickUser) throws SuspendExecution { 
        return true; 
    } 
 
    /** 
     * Call when attempting to log in with another user type while already logged in with any user type
     * 
     * @param userType   User type attempting to newly log in
     * @param outPayload Payload to be forwarded to clients
     * @return If the return value is true, the new login is successful, and if it is false, it fails
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */ 

    @Override 
    public boolean onLoginByOtherUserType(final String userType, 
                                          Payload outPayload) throws SuspendExecution { 
        return true; 
    } 
 
    /** 
     * Called if you are already logged in and attempt to log in through another connection (for reasons such as reconnection)
     * 
     * @param outPayload Payload to be forwarded to clients 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {     
    } 
 
    /** 
     * Called when you try to log in again, while you are already logged in 
     * (When logged in, the user's GameUser object remains valid on the GameNode) 
     * 
     * @param payload        Any payload delivered by the client 
     * @param sessionPayload Payload to be forwarded  onPreLogin 
     * @param outPayload    Any payload to be forwarded to clients 
     * @return If return value is true, successful ReLogin, if fals, failed ReLogin 
     * @throws SuspendExecution: This method means that the fiber can be suspended 
     */ 

    @Override 
    public boolean onReLogin(final Payload payload, 
                             final Payload sessionPayload, 
                             Payload outPayload) throws SuspendExecution { 
    } 
 
    /** 
     * Callback called when you lose connection with the client 
     * 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onDisconnect() throws SuspendExecution {     
    } 
 
    /** 
     * When the node to which the user belongs is paused, the user is paused and called
     * 
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */ 
    @Override 
    public void onPause() throws SuspendExecution {     
    } 
 
    /** 
     * When the node to which the user belongs is resumed, the user is resumed and called
     * 
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */ 

    @Override 
    public void onResume() throws SuspendExecution {     
    } 
 
    /** 
     * Called when user logout 
     * 
     * @param payload    Payload delivered from clients 
     * @param outPayload Payload to be forwarded to clients
 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onLogout(final Payload payload, Payload outPayload) throws SuspendExecution { 
    } 
 
    /** 
     * Called to see if the user is able to log out
     * Engine users can decide if the current GameUser logs out of this callback is fine
     * 
     * @return If the return value is false, the logout process stops, after which the callback is called back periodically. If the return value is true, proceed with logout
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 

    @Override 
    public boolean canLogout() throws SuspendExecution { 
    } 
 
    /** 
     * Call when you receive a room matchmaking request
     * 
     * @param roomType Any value that separates the pre-defined room types between client and server
     * @param payload  Payload delivered from clients 
     * @return Returns the information return of a matched room; if null, a new room may be created or failed depending on the client's request option
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public MatchRoomResult onMatchRoom(final String roomType, 
                                       final Payload payload) throws SuspendExecution { 
        return null; 
    } 

    /** 
     * Callback called when client requests user matchmaking
     * 
     * @param roomType   Any value that separates the pre-defined room types between client and server
     * @param payload    Payload delivered from clients 
     * @param outPayload Payload to be forwarded to clients
     * @return If the return value is true, the user match making request is successful, if false it fails
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 

    @Override 
    public boolean onMatchUser(final String roomType, 
                               final Payload payload, 
                               Payload outPayload) throws SuspendExecution { 
        return false; 
    } 
 
    /** 
     * Call when user matching is cancelled
     * 
     * @param reason Reasons for cancellation. Usually either TIMEOUT or CANCEL at the request of the user.
     * @return If the return value is true, the matching cancellation process is successful; if false, the cancellation fails
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution { 
        return false; 
    } 

    /** 
     * Callback for registering timer handlers to process
     * When this callback is called, the user registers as many timer handlers as they want to process
     * (Refer to "Timer" chapter for more information.) 
     */ 
    @Override 
    public void onRegisterTimerHandler() {     
    } 

 
    /** 
     * Call to see if the user is able to move (transfer) to another node
     * 
     * @return If the return value is true, it is a state that can be transferred, and if it is false, it is impossible. In the case of an impossible state, if NonStopPatch is in progress, it is called continuously to send the user until the patch is over     
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public boolean canTransfer() throws SuspendExecution { 
    } 
     
    /**
     * When a user moves (sends) to another node, a call is made to collect data to be transferred from the source node
     * (Refer to " Transferrable objects" chapter for more information.) 
     * 
     * @param transferPack Packages for storing data to be taken to other nodes 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onTransferOut(final TransferPack transferPack) throws SuspendExecution {     
    } 
 
    /** 
     * When a user moves (transfers) to another node, a call is made to restore a newly created user object to its original state on the target node
     * (Refer to " Transferrable objects" chapter for more information.) 
     *  
     * At this point, the user has not been fully recovered, so message requests to other places (nodes, users, etc.) are restricted.
     * 
     * @param transferPack   Packages of data from other nodes 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onTransferIn(final TransferPack transferPack) throws SuspendExecution {     
    } 
 
    /** 
     * Call after completion of user movement (transfer) between nodes
     * 
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */ 
    @Override 
    public void onPostTransferIn() throws SuspendExecution {     
    } 

 
    /** 
     * When a client requests a move to another channel, the call is made to verify that the current user is able to move the channel
     *
     * Caution> If the user explicitly calls the moveChannel() API to move the channel, the onCheckMoveOutChannel() is not called;it is called only when an implicit channel movement occurs by engine.
     * 
     * @param destinationChannelId      ID of the channel to be moved 
     * @param payload              Payload delivered from clients 
     * @param errorPayload         Payload from server to client if channel movement fails (not delivered if successful)
     * @return If the return value is false, the channel cannot be moved and the request is failed, if true, it is successful
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public boolean onCheckMoveOutChannel(final String destinationChannelId, 
                                         final Payload payload, 
                                         Payload errorPayload) throws SuspendExecution { 
        return false; 
    } 
 
    /** 
     * Called from source node when move channel to another node
     * 
     * @param destinationChannelId  channel ID to be transferred 
     * @param outPayload           Payloads to be forwarded to the channel to be transferred
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onMoveOutChannel(final String destinationChannelId, 
                                 Payload outPayload) throws SuspendExecution { 
    } 
 
    /** 
     * Called from source node after channel transfer to another node is complete
     * 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onPostMoveOutChannel() throws SuspendExecution { 
    } 
 
    /** 
     * Called when channeling to another node as it enters the destination node
     * 
     * @param sourceChannelId    Channel ID before transfer 
     * @param payload         Payload delivered from clients 
     * @param outPayload      Payload to be forwarded to clients
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onMoveInChannel(final String sourceChannelId, 
                                final Payload payload, 
                                Payload outPayload) throws SuspendExecution { 
    } 
 
    /** 
     * Called from destination node after channel movement to another node is complete
     * 
     * @throws SuspendExecution This method means that the fiber can be suspended
     */ 
    @Override 
    public void onPostMoveInChannel() throws SuspendExecution { 
    } 
}

```

In the example code above, any callback that starts with onLogin is called in relation to login. For example, onLogin callback is called for the first login request, and onReLogin callback is called if you are processing the relogin while you are already logged in. Similarly, onLogout callback is called when processing logout. As such, GameAnvil's callback is mostly clearly described by its name and JavaDoc annotations.

Users can move between channels at any time. Contents related to these channels will be explained separately in a [ separate chapter ](server-impl-09-channel) later, so we will move on for now. In addition, users are objects that can be transferred between multiple GameNodes. Features related to this will also be described in more detail in [Chapter separately.](server-impl-08-object-transfer)

GameAnvil provides two types of matchmaking features, room matchmaking and user matchmaking. When these matchmaking requests are reached by the user, onMatchRoom or onMatchUser callback is called. Users can use the matchmaker provided by GameAnvil in this callback, implement it themselves or use another matchmaker provided by a 3rd party. We'll desribe more about this in [Next Chapter](server-impl-04-match-node) as we deal with MatchNode.

Callbacks for these users are summarized in the table below.


| Callback name                | Description                                    | Description                                                                                                                                                      |
| -------------------------- | ----------------------------------------- |---------------------------------------------------------------------------------------------------------------------------------------------------------|
| onLogin                  | Login                                  | It is called the first time you log in. Normally, the user takes user information from a repository such as DB from this callback and performs the task of initializing the game user object.                                                              |
| onPostLogin              | Process after successful login                      | It is called after onLogin is successful. Post-processing operations for logins can be done on this callback.                                                                                                 |
| onLoginByOtherDevice     | Request for channel information                          | It is called when an additional login request is made to another device while you are already logged in. Users can decide on a return value to log in to the existing or new user.                                                   |
| onLoginByOtherUserType   | Attempt to log in with another user type           | It is called when you are already logged in and you receive a login request for another user type. You can decide whether to log in for the new user type as a return value.                                                         |
| onLoginByOtherConnection | Attempt to log in with another connection              | It is called if you are already logged in and attempt to log in with a different connection than before (due to reconnection). If you need additional work on reconnection, you can do it with this callback.                                               |
| onReLogin                | Relogin                                | If you log in again while you are already logged in, onReLogin is called instead of onLogin. In other words, this callback handles the relogin operation.                                                           |
| onDisconnect             |  Disconnect                               | It is called when the connection is lost from the client. At this time, the code to be further processed is implemented here.                                                                                                   |
| onPause                  | Pause                               | Pause GameNode through the console and it will be called for all users of that GameNode. Users can implement additional code they want to process here when the node is paused.                                           |
| onResume                 | Resume                                    | When GameNode resumes operation from pause through console, it is called for all users of that GameNode. Users can implement the code they want to process for users here in the resume state.                                  |
| onLogout                 | Log out                               | It is called when the user logs out. This can be an explicit logout requested by the user, or it can be automatically logged out by engine if the set time is exceeded while disconnected.                                                 |
| canLogout                | Verify logout availability                    | It is called to verify if the user is currently available for logout. If you're playing or you shouldn't lose information, you can delay logout by returning false. If you return false, the engine will continually call this callback after a certain period of time. |
| onMatchRoom              | Room Matchmaking Request                      | It is called when the user requests room matchmaking. At this time, the user can randomly use the room matchmaking API or a third matchmaking solution provided by engine in this callback.                                                        |
| onMatchUser              | User Matchmaking Request                    | It is called when a user requests user matchmaking. The user can randomly use the room matchmaking API or third matchmaking solution provided by engine in this callback.                                                          |
| onMatchUserCancel        | Cancel User Matchmaking                    | It is called when the user cancels the previously requested user matchmaking. If the cancellation is not possible, such as when matching has already been completed, it may fail.                                                                         |
| canTransfer              | Verify that user transfer is possible         | It is called to check if the user is in a state where he/she can be transferred to another node. If you are playing the game or are not ready yet, you can delay the transfer by returning false. If returning false, the engine will continue to call this callback after certain period of time. |
| onTransferOut            | Ready to be transferred from existing node         | It is called when a user is transferred to another GameNode and when the source node initiates a transfer. The user can collect data packets to be sent with the user object in this callback.                                                          |
| onTransferIn             | Processing transfer completion to new node             | When a user is transferred to another GameNode, it is called upon completion of the transfer at the destination node. Users can unpack the data packets they brought together and restore them to their original state.                                                       |
| onPostTransferIn         | Processing after transfer completion                        | It is called for post-processing when the user transfer is successful.                                                                                                                   |
| onCheckMoveOutChannel    | Verify that channel transfer is possible         | It is called to check that the user can transfer to another channel. If the user explicitly calls the moveChannel() API to move the channel, it is not called. It is called only when an implicit channel movement occurs by engine.             |
| onMoveOutChannel         | Prepare to move from existing channel to another channel      | When the user moves to another channel, it is called from the source node. The user can put the desired information in outPayload and take it to the destination channel.                                                                      |
| onPostMoveOutChannel     | Ready to move out from existing channel to other channel | If onMoveOutChannel is successful, it will be called for post-processing.                                                                                                                   |
| onMoveInChannel          | Process moving to new channel                  | When a user moves to another channel, it is called from the destination node. The user can put any information in outPayload and forward it to the client.                                                                        |
| onPostMoveInChannel      | Complete the move to the new channel                  | If the onMoveInChannel is successful, it will be called for post-processing.                                                                                                                    |
| getMessageDispatcher | Has hackets to process | It returns messages to process. Users can use the dispatchers they declare. For more information, refer to [Process messages](./server-impl-07-message-handling#13-getMessageDispatcher).            |

### What is log in?

The above description and example code frequently describe login. You can also define this login as the process by which a client connects to a server and then creates its own user object in GameNode. Among callback methods, onLogin() is called during initial attempt to login to create a user. In this case, the user may obtain information for configuring the user object from DB, etc. If this onLogin() callback is successful, a corresponding user object is created on the GameNode. When login is completed in this way, the client can implement various contents by exchanging messages with other objects through his or her user object based on the directly defined protocol.

### Log out
Logout is the opposite concept of login, <i>i.e., </i>the process of removing one's own user object from GameNode. When you start logout, the user object can call onLogout() callback and keep its final state in a DB, etc. before it is deleted from memory. These logouts can be requested explicitly by client or automatically processed by engine after a certain amount of time has elapsed while the client is disconnected. Therefore, if you expect frequent disconnections, such as in mobile games, you can set the appropriate [set](server-impl-16-config-vm#game) to prevent logouts from happening immediately.



## Room implementation

Two or more users can create a synchronized message flow through a room. In other words, the sequence of all users' requests is guaranteed in the room. Of course, creating a room for one user can also have meaning depending on the content. It is up to the engine user to decide how to use the room. Like the user, these rooms inherit the base class, BaseRoom, and can override several callback methods and handle their own messages. The example code below is the SampleGameRoom class for SampleGameUser.

Rooms also require multiple annotations to register with engine. First, register what kind of game service it is for, and then register the room type. This room type is literally used to distinguish the type of room and is also used by clients to call the API. In other words, it specifies what room type of server the API is calling. Therefore, you must pre-define these room types between the server and the client as random string. This example code uses a room type called "BasicRoom." We will explain this in more detail later in separate chapters. Finally, you can decide whether or not this room information will be used for [Synchronizing Room Information between Channels](server-impl-09-channel#synchronize-channel-information). If you do not need to synchronize information between channels, you can skip the @UseChannelInfo annotation.

```java
@ServiceName("MyGame")  
@RoomType("BasicRoom")  
@UseChannelInfo  
public class SampleGameRoom extends BaseRoom<SampleGameUser> {  
  
    private static final RoomMessageDispatcher<SampleGameRoom, SampleGameUser> messageDispatcher = new RoomMessageDispatcher<>();  
  
    static {  
        messageDispatcher.registerMsg(SampleGame.GameRoomTest.class, _GameRoomTest.class);  
    }  
  
    @Override  
    public RoomMessageDispatcher<SampleGameRoom, SampleGameUser> getMessageDispatcher() {  
        return messageDispatcher;  
    }  
  
    /**  
     * Called when room is initiated  
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended  
     */  
    @Override  
    public void onInit() throws SuspendExecution {      
    }  
  
    /**  
     * Called when the room destroys from the server 
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onDestroy() throws SuspendExecution {      
    }  
 
    /**  
     * Called when creating a new room 
     *  
     * @param user       Requested user object  
     * @param inPayload  Payload delivered from clients 
     * @param outPayload Payload to be forwarded to clients
     * @return If the return value is true, room creation is successful, if false, it fails 
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public boolean onCreateRoom(SampleGameUser user,  
                                final Payload inPayload,  
                                Payload outPayload) throws SuspendExecution {  
    }  
  
    /**  
     * Called when you join a room 
     *  
     * @param user       Requested user object 
     * @param inPayload  Payload delivered from clients  
     * @param outPayload Payload to be forwarded to clients 
     * @return If the return value is true, the entry is successful; if false, it fails  
     * @throws SuspendExecution This method means that the fiber can be suspended 
    
  */  
    @Override  
    public boolean onJoinRoom(SampleGameUser user,  
                              final Payload inPayload,  
                              Payload outPayload) throws SuspendExecution {  
    }  
  
    /**  
     * Called when leave a room  
     *  
     * @param user       Requested user object 
     * @param inPayload  Payload delivered from clients  
     * @param outPayload Payload to be forwarded to clients  
     * @return If the return value is true, exit is successful; if false, it fails 
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public boolean onLeaveRoom(SampleGameUser user,  
                               final Payload inPayload,  
                               Payload outPayload) throws SuspendExecution {  
    }  
   
    /**  
     * onLeaveRoom   If the callback is successful, call for post-processing after exit is complete     *  
     * @param user User object left a room  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onPostLeaveRoom(SampleGameUser user) throws SuspendExecution {  
    }  
  
    /**  
     * If a user reconnects and re-logins while participating in a room due to a disconnected connection, etc., automatically re-enters the room and calls it 
     *  
     * @param user       User object go into a room  
     * @param outPayload Payload to be forwarded to clients  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onRejoinRoom(SampleGameUser user, Payload outPayload) throws SuspendExecution {      
    }  
  
   /**  
     * Callback for registering timer handlers to process 
     * When this callback is called, the user registers as many timer handlers as they want to process. 
     * (Refer to "Timer" chapter for more information.)   
     */  
    @Override  
    public void onRegisterTimerHandler() {  
    }  
  
  
    /**  
     * Call to see if the room is in a state where it can be moved (transferred) to another node 
     *  
     * @return   If the return value is true, it is a state that can be transferred, and if it is false, it is impossible. In the case of an impossible state, if NonStopPatch is in progress, it is called continuously to send the user until the patch is over 
     * @throws SuspendExecution This method means that the fiber can be suspended  
     */  
    @Override  
    public boolean canTransfer() throws SuspendExecution {      
    }  
    
    /**
     * It is called when the room is forwarded (transferred) to another node, to collect data to be sent from source node.
     * (Refer to "Transferable Objects" chapter for more information.)  
     *  
     * @param transferPack   Package for storing data to be taken to other nodes  
     * @throws SuspendExecution This method means that the fiber can be suspended  
     */  
    @Override  
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {  
    }  
 
    /**  
     * Call to restore a newly created room object to its original state on the destination node when that room moves (transfers) to another node 
     * (Refer to "Transferable Objects" chapter for more information.)  
     *   
     * At this point, the room has not been fully restored, so message requests to other places (nodes, users, etc.) are restricted. 
     *  
     * @param transferPack   Packages of data from other nodes 
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onTransferIn(List<SampleGameUser> userList,  
                             final TransferPack transferPack) throws SuspendExecution {  
    }  
  
    /**  
     * When the node to which the room belongs is paused, the room is also paused and called 
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onPause() throws SuspendExecution {  
    }  
  
    /**  
     * When the node to which the room belongs is resumed, the room is also resumed and called 
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended  
     */  
    @Override  
    public void onResume() throws SuspendExecution {  
    }  
  
    /**  
     * Callback called when client requests party matchmaking 
     * (Two or more users must be in the NamedRoom for party matchmaking. At this time, this callback is called in the NamedRoom.)  
     *  
     * @param roomType   Any value that separates the pre-defined room types between client and server 
     * @param user       User requested party matching (room manager)  
     * @param payload    Payload received from clients  
     * @param outPayload Payload to be forwarded to clients 
     * @return If the return value is true, the party matchmaking request is successful and false is failed     
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */    
    @Override  
    public boolean onMatchParty(final String roomType,  
                                final SampleGameUser user,  
                                final Payload payload,  
                                Payload outPayload) throws SuspendExecution {  
        return false;  
    }  
} 

```

Rooms provide three of the most basic callbacks, onCreateRoom, onJoinRoom and onLeaveRoom. They are called for creating, joining and leaving rooms, respectively. Users can implement related features directly from those callbacks. For example, while creating a room, you can designate a room manager and initialize basic data structures. You can also update the list of users in the room for any room joining requests,or synchronize the status between users.

These rooms are created and joined automatically by engine through matchmaking as well as explicit requests from the client. For example, if User A and User B are matched in a two-person game, one person automatically creates a room and the other person joins the room. This process is automatically handled by engine, not by User A and User B's requests. This process also, of course, calls callback to onCreateRoom and onJoinRoom.

Callbacks in these rooms are summarized in the table below.


| Callback name              | Description                             | Description                                                                                                                                                                                              |
| ------------------------ | ---------------------------------- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onInit                 | Initialize                           | When a room is created, it is called for initialization. You can write an initialization code for that room, such as topic registration.                                                                                                                                   |
| onDestroy              | Destroy Room                        | If last user leaves the room and there are no more messages to process, the room will be destroyed. This is the callback that is called.                                                                                                                                   |
| onCreateRoom           | Create Room                           | It is called when a client requests a room creation. Create a material structure for the list of users that the content will use, or write a code that needs to be processed with other room creations.                                                                                                        |
| onJoinRoom             | Join Room                          | It is called when the client asks you to join any room on the server. You can update the list of users you want to use in the content or process information to synchronize between users in the room.                                                                                               |
| onLeaveRoom            | Leave Room                           | It is called when the client requests to leave the room. Update the structure of the material for the list of users that the content will use, or write the code that needs to be processed as you leave the room.                                                                                                   |
| onPostLeaveRoom        | Post processing after leaving room                   | If onLeaveRoom is successful, it will be called for post-processing. Write the code to process immediately after leaving the room.                                                                                                                                       |
| onRejoinRoom           | Rejoin room                     | If the user is in the room and Relogin is performed due to disconnection, etc., the engine automatically re-joins the room. This is the callback that is called. During the re-entry process, information required for synchronization can be processed.                                                      |
| canTransfer            | Verify if room transfer is possible   | It is called to check if the room is in a state where it can be transferred to another node. If the room is still playing or not ready, you can delay the transfer by returning false. If returns false, the engine continually calls this callback after any time. For your information, the room transfer is only used when performing an unchecked patch. |
| onTransferOut          | Ready to be transferred from existing node | It is called when a room is sent to another GameNode and when the source node starts transfer. Users can collect data packets to transfer with room objects in this callback. For reference, a room transfer involves a user transfer to each of the users in the room.                                                            |
| onTransferIn           | Processing transfer completion to new node     | When a room is sent to another GameNode, it is called upon completion of the transfer at the destination node. Users can unpack the data packets they brought together and restore them to their original state. For your information, a room transfer involves a user transfer to each of the users in the room.                                                         |
| onPause                | Pause                        | When you pause GameNode via console, it will be called for all rooms in that GameNode. Users can implement code here that they want to process additionally in the room when the node is paused.                                                                                     |
| onResume               | Resume                             | When GameNode resumes running from pause through the console, it is called for all rooms in that GameNode. Users can implement the code they want to process for the room here in the resume state.                                                                            |
| onMatchParty           | Request party matchmaking             | It is called when a user requests party matchmaking. Party matchmaking is a feature that creates a randomly named room for a party and then all users in a room request a match for one party. Users can use the party matchmaking API or third-party matchmaking solution provided by the engine in this callback.                      |
| getMessageDispatcher | Has hackets to process | It returns a message when the node has message to process. Users can use the dispatchers they declare. For more information, refer to [Message Processing](./server-impl-07-message-handling#13-getMessageDispatcher).                                                     |

## Room type

Apart from the implementation of the room we described earlier, there are two main types of rooms provided by engine. These two rooms are used in a total of four ways.

| Room type     | Description                                                         |
| ----------- | ------------------------------------------------------------ |
| Normal Room | The client explicitly requests CreateRoom to create it. Other clients can also join by explicitly joinRoom using the room's ID. Therefore, users who want to join must share the room's ID in advance. |
| Named Room  | NamedRoom performs an action against a named room, which, like its name, centers on a unique room name. The client requests NamedRoom with the only room name in the server group. In this case, if that room name does not exist on the server, the requester will create the room himself or herself. On the other hand, if a room name already exists on the server, it will automatically enter it. It's easy to understand, for example, by thinking of all kinds of control trees like "3:3 Hunter Beginner!" in StarCraft's custom game list. |

These two rooms are used in a total of four ways.

| Room type     | How to use                                                       |
| ----------- | ------------------------------------------------------------ |
| Normal Room | 1. Through CreateRoom/JoinRoom requests, clients create and join.<br>2. Room Matchmaking allows you to create or join in NormalRoom. Also, rooms made with CreateRoom can be registered as room matchmaking targets. At this time, room IDs are automatically shared between rooms and users that are managed and matched by engine. |
| Named Room  | 3. Through NamedRoom requests, clients create and join.<br>4. You can create or join NamedRoom through user matchmaking. At this time, the named room of the namedRoom is uniquely created and managed by engine. Unlike room matchmaking, rooms created with regular NamedRoom cannot be targeted for user matchmaking. However, after creating a party room with NamedRoom for party matchmaking, multiple users can request matchmaking with one party. |



## Channel

GameNodes can be grouped logically according to their intended purpose. These logical groups are called [channel](server-impl-09-channel.md). For example, you could group GameNodes 1 and 2 into "Beginner" channel and group GameNodes 3 and 4 into "Expert" channel. This will be explained in more detail in a [ separate chapter ](server-impl-09-channel).
