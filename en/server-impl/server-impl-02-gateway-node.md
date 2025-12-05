## Game > GameAnvil > Server Development Guide > Implement Gateway Node

## Gateway Node

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gatewaynode_on_network.png)

GatewayNode is a gateway accessed by the client. The service manages sessions for client connections and game services. At this time, the client must log in to GatewayNode to proceed with the game and complete the authentication. The relationship between them is as in the following image:

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

Typically, a client establishes one connection to a GatewayNode. At this time, the service allows you to proceed with the authentication procedure for the connection. If it’s successful, you can create one or more sessions in one year. Each session is the logical unit of connection between the client and the user. The image above shows a session created by the client with the Game service and the Chat service through one connection. This structure allows simple [session recovery](#session-recovery) even if the client's connection is accidentally lost.

### Implement GatewayNode

For such GatewayNode, @GameAnvilGatewayNode annotation can be declared and registered in the engine, and the IGatewayNode interface can be implemented to redefine only the callback method. These common callback methods are clearly explained with their name.
```java
@GameAnvilGatewayNode // Register this class as Gateway in the engine
public class SampleGatewayNode implements IGatewayNode {
    private IGatewayNodeContext gatewayNodeContext;

    /**
     * Call to send gateway node context
     * <p/>
     * Call once after the object is created
     *
     * @param gatewayNodeContext Gateway Node Context
     */
    @Override
    public void onCreate(IGatewayNodeContext gatewayNodeContext) {
        this.gatewayNodeContext = gatewayNodeContext;
    }

    /**
     * Call when the node is initialized
     */
    @Override
    public void onInit() {

    }

    /**
     * Call for what to handle before you get Ready
     */
    @Override
    public void onPrepare() {

    }

    /**
     * Call when you are Ready
     */
    @Override
    public void onReady() {

    }

    /**
     * Call when Pause
     *
     * Additional information to send from the @param payload content
     */
    @Override
    public void onPause(IPayload payload) {

    }

    /**
     * Call when you receive the Shutdown command
     */
    @Override
    public void onShuttingdown() {

    }

    /**
     * Call when Resume
     *
     * Additional information to send from the @param payload content
     */
    @Override
    public void onResume(IPayload payload) {

    }
}
```


### Implement Connection

Connection designates the physical access itself to the client. The client can proceed with the authentication procedure on the connection using its unique AccountId. If authentication succeeds, the accountId is mapped to the created connection.

After implementing IConnection, these connections redefine the callback methods as follows. At this time, you can use the user's key value obtained after authenticating from any platform as AccountId. For example, if you acquire a UserId after authenticating through Gamebase, this value can be used as an AccountId in the GameAnvil authentication process. 

```java
@GameAnvilGatewayConnection // Register this class as a Connection in the engine 
public class SampleConnection implements IConnection {
    private IConnectionContext connectionContext;
    
    /**
     * Call to send connection context
     * <p/>
     * Call once after the object is created
     *
     * @param connectionContext connection context
     */
    @Override
    public void onCreate(IConnectionContext connectionContext) {
        this.connectionContext = connectionContext;
    }

    /**
     * Call for authentication requests
     *
     * @param accountId  account ID
     * @param password  account password
     * @param deviceId   device ID of client
     * @param payload    {@link IPayload} sent by client
     * @param outPayload  {@link IPayload} to be sent to the client
     * If the @return return value is true, the authentication succeeded, and if false, the connection to the client ended.
     */
    @Override
    public boolean onAuthenticate(String accountId, String password, String deviceId, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    /**
     * Call when the node belonging to the connection is Pause
     */
    @Override
    public void onPause() {

    }

    /**
     * Call when the node belonging to the connection is Resume
     */
    @Override
    public void onResume() {

    }

    /**
     * Call when connection to the client is lost
     */
    @Override
    public void onDisconnect() {

    }
}
```


For the meaning and usage of these callbacks, see the table below:

| Callback Name | Meaning | Description |
|----------------|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate | Create object | Called when the object is created. You receive the context where the API available for the created type can be used. If needed in the content, you can save and use it. |
| onAuthenticate | Authentication | Calls are called when the client requests authentication for the connection using the Authentication() API. The user can proceed with authentication processing based on the credentials sent by the client here. If authentication succeeds, you must return true and false if the authentication fails. |
| onPause | Pause | Pause of GatewayNode through the console, all connections to the GatewayNode will be called. Here, when the node is temporarily stopped, the user can implement the code he wants to process additionally in the connection. |
| onResume | Resume | When GatewayNode runs again during a temporary stop, all connections in the GatewayNode are called. Here, the user can implement the code he wants to process for the connection in a restarted state. |
| onDisconnect | Access Ended | Called when the connection is lost from the client. At this time, the code to be processed is implemented here. |

### Perform Session

Clients who are successfully connected can enter a logical session for GameNode, one per service, between those connections. GameAnvil internally combines the accountId of the connection and the subId of the session to allow unique sessions to be distinguished across the server.

In this case, SubId can be allocated to any unique value within that connection according to random rules the user sets. In other words, different connections may have the same SubId. But it is possible to distinguish because they have different AccountId.

```java
@GameAnvilGatewaySession  // Register this class as a session 
public class SampleSession implements ISession {
    private ISessionContext sessionContext;

    /**
     * Call to send the session context
     * <p/>
     * Call once after the object is created
     *
     * @param sessionContext session context
     */
    @Override
    public void onCreate(ISessionContext sessionContext) {
        this.sessionContext = sessionContext;
    }

    /**
     * Call before login calls
     *
     * @param outPayload  {@link IPayload} to be sent to the client
     */
    @Override
    public void onBeforeLogin(IPayload payload) {

    }

    /**
     * Call after login succeeded
     */
    @Override
    public void onAfterLogin(boolean isReLogined) {

    }

    /**
     * Call after logout
     */
    @Override
    public void onAfterLogout() {

    }
}
```

For the meaning and usage of these callbacks, see the table below:


| Callback name | Meaning | Description |
|----------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| onCreate | Create object | Call when the object is created. You receive the context where the API available for the created type can be used. If needed from the content, you can save and use it. |
| onBeforeLogin | Pre-Login Process | GameNode will be called just before you request login. At this time, the user can enter a value for the outPayload delivered as a parameter and send it to the login request. This payload is delivered as it is when processing the login callback from the game node. |
| onAfterLogin | Login After Processing | After logging in to GameNode, the payload is called. If there is any code to be processed by the session after logging in, implement it here. |
| onAfterLogout | Logout After Processing | You will be called after logout processing is completed. If there is any code to be processed in the session after logout, implement it here. |

## Connection and Session

The client connects to the gateway node. Create a connection, for example. This connection allows you to authenticate and log in based on your account and user information. Once logged in, the user object is created in the random game node. It means that a logical session has been created between the gateway node and the game node. Once the connection and session creation are complete, the user can proceed with the game. We'll come back to this later when we discuss game nodes.

### Session Recovery

If a re-connection occurs between the client and the gateway node, Session Recovery proceeds as shown in the image below. In the process of reconnecting, the client may try to connect to any of the multiple gateway nodes. In this case, a new session is restored based on the location information of the game node where the user object exists. Therefore, even if the user resumes during the game, the user can continue to play the previous game status.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

### Location Node

The location node is shown in the connection recovery image you looked at earlier. A location node is a system node that GameAnvil internally manages location information, such as users and rooms. The user cannot directly implement or use the location node. However, to understand the role of the location node in managing location information, it is easy to understand the flow of the overall GameAnvil system.

Take the connection recovery above as an example. As the client attempts to log in to the game node with its first access, all related sessions and user location information are stored in the location node. Therefore, if you proceed with a re-access, you can view this location information stored during the previous access process. This location information is used internally by GameAnvil for retrieving the location information of the user or room and for delivering messages based on it.
