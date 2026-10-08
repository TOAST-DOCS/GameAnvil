<!-- machine_translated: true -->

<!-- pre-align:aligned sig=4435e036d920 -->

<a id="game-gameanvil-server-development-guide-implement-gateway-node"></a>
## Game > GameAnvil > Server Development Guide > Implement Gateway Node { #game-gameanvil-server-development-guide-implement-gateway-node }

<a id="gateway-node"></a>
## Gateway Node { #gateway-node }

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gatewaynode_on_network.png)

GatewayNode is a gateway accessed by the client. The service manages sessions for client connections and game services. At this time, the client must log in to GatewayNode to proceed with the game and complete the authentication. The relationship between them is as in the following image:

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

Typically, a client establishes one connection to a GatewayNode. At this time, the service allows you to proceed with the authentication procedure for the connection. If it's successful, you can create one or more sessions in one year. Each session is the logical unit of connection between the client and the user. The image above shows a session created by the client with the Game service and the Chat service through one connection. This structure allows simple [session recovery](#session-recovery) even if the client's connection is accidentally lost.

<a id="implement-gatewaynode"></a>
### Implement GatewayNode { #implement-gatewaynode }

For such GatewayNode, @GameAnvilGatewayNode annotation can be declared and registered in the engine, and the BaseGatewayNode class can be implemented to redefine only the callback method. These common callback methods are clearly explained with their name.
```java
@GameAnvilGatewayNode // Register this class as Gateway to the engine
public class SampleGatewayNode extends BaseGatewayNode {
 
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
     * Call when you receive the Shutdown command
     */
    @Override
    public void onShuttingdown() {

    }
}
```


<a id="implement-connection"></a>
### Implement Connection { #implement-connection }

Connection designates the physical access itself to the client. The client can proceed with the authentication procedure on the connection using its unique AccountId. If authentication succeeds, the accountId is mapped to the created connection.

After implementing BaseConnection, these connections redefine the callback methods as follows. At this time, you can use the user's key value obtained after authenticating from any platform as AccountId. For example, if you acquire a UserId after authenticating through Gamebase, this value can be used as an AccountId in the GameAnvil authentication process. 

```java
@GameAnvilGatewayConnection // Register this class as Connection to the engine
public class SampleConnection extends BaseConnection {

    /**
     * Called when an authentication request is received
     *
     * @param accountId  Account ID
     * @param password   Account password
     * @param deviceId   Device ID of the client
     * @param payload    {@link IPayload} sent by client
     * @param outPayload {@link IPayload} to be sent to the client
     * @return If the return value is true, the authentication succeeded, and if false, the connection to the client ended.
     */
    @Override
    public boolean onAuthenticate(String accountId, String password, String deviceId, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }
}
```


For the meaning and usage of these callbacks, see the table below:

| Callback Name | Meaning | Description |
|----------------|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onAuthenticate | Authentication | Calls are called when the client requests authentication for the connection using the Authentication() API. The user can proceed with authentication processing based on the credentials sent by the client here. If authentication succeeds, you must return true and false if the authentication fails. |

<a id="perform-session"></a>
### Perform Session { #perform-session }

Clients who are successfully connected can enter a logical session for GameNode, one per service, between those connections. GameAnvil internally combines the accountId of the connection and the subId of the session to allow unique sessions to be distinguished across the server.

In this case, SubId can be allocated to any unique value within that connection according to random rules the user sets. In other words, different connections may have the same SubId. But it is possible to distinguish because they have different AccountId.

```java
@GameAnvilGatewaySession  // Register this class as Session in the engine
public class SampleSession extends BaseSession {

    /**
     * Called before login
     *
     * @param outPayload {@link IPayload} to be sent to the client
     */
    @Override
    public void onBeforeLogin(IPayload payload) {

    }

    /**
     * Called after a successful login
     */
    @Override
    public void onAfterLogin(boolean isReLogined) {

    }

    /**
     * Called after logout
     */
    @Override
    public void onAfterLogout() {

    }
}
```

For the meaning and usage of these callbacks, see the table below:


| Callback name | Meaning | Description |
|----------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| onBeforeLogin | Pre-Login Process | GameNode will be called just before you request login. At this time, the user can enter a value for the outPayload delivered as a parameter and send it to the login request. This payload is delivered as it is when processing the login callback from the game node. |
| onAfterLogin | Login After Processing | After logging in to GameNode, the payload is called. If there is any code to be processed by the session after logging in, implement it here. |
| onAfterLogout | Logout After Processing | You will be called after logout processing is completed. If there is any code to be processed in the session after logout, implement it here. |

<a id="connection-and-session"></a>
## Connection and Session { #connection-and-session }

The client connects to the gateway node. Create a connection, for example. This connection allows you to authenticate and log in based on your account and user information. Once logged in, the user object is created in the random game node. It means that a logical session has been created between the gateway node and the game node. Once the connection and session creation are complete, the user can proceed with the game. We'll come back to this later when we discuss game nodes.

<a id="session-recovery"></a>
### Session Recovery { #session-recovery }

If a re-connection occurs between the client and the gateway node, Session Recovery proceeds as shown in the image below. In the process of reconnecting, the client may try to connect to any of the multiple gateway nodes. In this case, a new session is restored based on the location information of the game node where the user object exists. Therefore, even if the user resumes during the game, the user can continue to play the previous game status.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

<a id="location-node"></a>
### Location Node { #location-node }

The location node is shown in the connection recovery image you looked at earlier. A location node is a system node that GameAnvil internally manages location information, such as users and rooms. The user cannot directly implement or use the location node. However, to understand the role of the location node in managing location information, it is easy to understand the flow of the overall GameAnvil system.

Take the connection recovery above as an example. As the client attempts to log in to the game node with its first access, all related sessions and user location information are stored in the location node. Therefore, if you proceed with a re-access, you can view this location information stored during the previous access process. This location information is used internally by GameAnvil for retrieving the location information of the user or room and for delivering messages based on it.