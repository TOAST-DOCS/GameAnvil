## Game > GameAnvil > Server Development Guide > Gateway Node Implementation



## Gateway Node

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_gatewaynode_on_network.png)

Gateway Node is the gateway to which the client accesses. In other words, it manages the client's connections and sessions for game services. At this time, the client must access the Gateway Node and complete authentication in order to proceed with the game. Their relationship is shown in the following image.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionAndSession.png)

In general, a client has a connection with a Gateway Node. At this time, you can proceed with the authentication process for that connection and create one or more sessions only if it is successful. Each session is a logical unit of connection between the client and the user. The image above shows the client creating a session with Game and Chat services through a single connection. This structure makes it simple to [Session Recovery ](#21-session-recovery) even if the client is unintentionally disconnected.



### GatewayNode implementation

These GatewayNodes only need to inherit the BaseGatewayNode class and override the common callback method. These common callback methods clearly state the purpose of their names. They also use the @GatewayNode annotation to register the class they implemented with engine.

```java
@GatewayNode // Register this GatewayNode class with the engine
public class SampleGatewayNode extends BaseGatewayNode {

    /**]
     * Called at initialization time
     *]
     * @throws SuspendExecution This method means that the fiber can be suspended.
     */
    @Override
    public void onInit() throws SuspendExecution { 
    }

    /**]
     * Called for parts to be processed before they're ready
     *]
     * @throws SuspendExecution This method means we can suspend the fiber
     */
    @Override
    public void onPrepare() throws SuspendExecution {
    }

    /**
     * Called when ready
     *]
     * @throws SuspendExecution This method means that the fiber can be suspended.
     */
    @Override
    public void onReady() throws SuspendExecution { 
    }

    /**
     * Called when paused
     * * * @param payload contents
     * @param payload contents any additional information you wish to pass
     * @throws SuspendExecution This method means the fiber can be suspended
     */
    @Override
    public void onPause(Payload payload) throws SuspendExecution { 
    }

    /** */ /** */ @Override.
     * Called when resuming
     * * /** * Returns.
     * @param payload contents any additional information you wish to pass
     * * @throws SuspendExecution This method means that the fiber can be suspended
     */
    @Override
    public void onResume(Payload payload) throws SuspendExecution { 
    }

    /**
     * Called when a shutdown command is received
     *]
     * @throws SuspendExecution This method means that the fiber can be suspended.
     */
    @Override
    public void onShutdown() throws SuspendExecution { 
    }
}
```



### Connection implementation

Connection means the client's physical connection itself. Clients can use their own AccountId to proceed with the authentication process on the connection. If authentication is successful, the AccountId is mapped to the connection that was created.

These connections override the callback methods after inheriting BaseConnection as follows. At this time, you can use AccountId as a key value for the user that you obtain after authenticating on any platform. For example, if you obtain UserId after authenticating through Gamebase, you can use this value as AccountId during GameAnvil's authentication process. In addition, @Connection Annotation is used to register the class you implemented with engine, just like the GatewayNode implementation.

```java
@Connection // Register this connection class to the engine 
public class SampleConnection extends BaseConnection<SampleGameSession> {  
  
    /**  
     * Call when authenticating  
     *  
     * @param accountId  Account ID  
     * @param password   Account password  
     * @param deviceId   Client’s device ID  
     * @param payload    Payload delivered from clients 
     * @param outPayload Payload to be forwarded to clients 
     * @return Authentication successful or not: If false, the connection to the client is terminated 
     * @throws SuspendExecution This method means that the fiber can be suspended 
 
 
 
 
 
     */  
    @Override  
    public boolean onAuthenticate(final String accountId, final String password, final String deviceId, final Payload payload, final Payload outPayload) throws SuspendExecution {        
    }  
  
    @Override  
    public void onPause() throws SuspendExecution {  
    }  
  
    @Override  
    public void onResume() throws SuspendExecution {  
    }  
  
    /**  
     * Call when disconnected from the client 
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    public void onDisconnect() throws SuspendExecution {        
    }  
}
```

Refer to the table below for the meaning and purpose of these callbacks.


| Callback name      | Description                | Description                                                                                                                                                                                                                                                    |
| ---------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onAuthenticate | Verify                | It is called when the client requests authentication for the connection using the Authentication() API. Here, the user can proceed with the authentication process based on the authentication information sent by client. If the authentication is successful, it must return true, and if fails, it must return false. |
| onPause        | Pause           | When you pause a Gateway Node through the console, it is called for all connections in that Gateway Node. Users can implement additional code they want to process in the Connection here when the node is paused.                                                             |
| onResume       | Resume                | When the Gateway Node resumes operation from a pause through console, it is called for all connections in that Gateway Node. Users can implement the code they want to process for the connection here in the resume state.                                              |
| onDisconnect   |  Disconnect           | It is called when the connection is lost from the client. At this time, the code to be further processed is implemented here.                                                                                                                                                           |
| getMessageDispatcher | Has hackets to process | Returns a message when the node has a message to process. Users can use the dispatchers they declare. For more information, refer to [Message Processing](./server-impl-07-message-handling#13-getMessageDispatcher). |



### Session implementation

Clients who successfully establish a connection can have one logical session for GameNode on that connection, one for each service. GameAnvil can internally combine the AccountId of the connection with the SubId of the session to distinguish between unique sessions across the entire server.

At this time, SubId can be assigned to any unique value within that connection according to random rules set by the user. In other words, different connections can have the same SubId. However, since they have different AccountId, they can be distinguished. In addition, @Session annotation is used to register the implemented session class with engine.

```java
@Session // Register this session class to the engine 
public class SampleSession extends BaseSession {  
  
    private static MessageDispatcher<SampleSession> dispatcher = new MessageDispatcher<>();  
  
    static {  
        dispatcher.registerMsg(SampleGame.MsgToSessionReq.class, _MsgToSessionHandler.class);  
    }  
  
    @Override  
    public MessageDispatcher<SampleSession> getMessageDispatcher() {  
        return dispatcher;  
    }  
  
    /**  
     * Call before login call  
     *  
     * @param outPayload Payload to be forwarded to clients 
     * @throws SuspendExecution This method means that the fiber can be suspended  
     */  
    @Override  
    public void onPreLogin(Payload outPayload) throws SuspendExecution {        
    }  
  
    /**  
     * Call after successful login  
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended  
     */  
    @Override  
    public void onPostLogin() throws SuspendExecution {        
    }  
  
    /**  
     * Call after logout  
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onPostLogout() throws SuspendExecution {  
    }  
}
```

Refer to the table below for the meaning and purpose of these callbacks.


| Callback name    | Description                | Description                                                                                                                                                                                                                                          |
| -------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| onPreLogin   | Login pre-process       | It is called just before requesting a login to the GameNode. At this time, the user can put any value in the output payload delivered as a parameter and load it into the login request. This payload is delivered as it is when the game node processes a login callback. |
| onPostLogin  | Login post-process       | Called after completing login to GameNode. If there is a code to handle in the session after login, you can implement it here.                                                                                                                                   |
| onPostLogout | Logout post-process     | It is called after the logout processing is complete. If there is a code to be processed in the session after the logout, you can implement it here.                                                                                                                                       |
| getMessageDispatcher | Has hackets to process | It returns when the node has a message to process. Users can use the dispatchers they declare. For more information, refer to [Message Processing](./server-impl-07-message-handling#13-getMessageDispatcher). |



## Connection and Session

Client connects to the gateway node. That is, it creates a connection. This connection allows authentication and login based on account and user information. Once login is completed, a user object is created on any game node. This means that a logical session has been created between the gateway node and its game node. Once the connection and session creation are completed, the user will be able to proceed with the game. We'll revisit this when we explain the game node right behind us.

### Session Recovery

If a reconnection occurs between the client and the gateway node, session recovery proceeds as shown in the image below. During the reconnection process, the client can also attempt a connection to a different location among multiple gateway nodes than before. In this case, the session is newly restored based on the location information for the game node where the user object exists. Therefore, the user can continue the previous game state even if he reconnects during the game.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

### Location Node

The location node is visible in Connection Recovery image that we explained earlier. A location node is a system node where GameAnvil manages location information internally, such as users and rooms. Users cannot implement or use the location node directly. However, understanding the role of the location node in managing location information can help us understand the flow of the overall GameAnvil system, so I would like to briefly mention it here.

Let's take the above connection recovery as an example. As the client first connects and attempts to log in to the game node, all related session information and user location information are stored on the location node. Therefore, you can inquire about this location information that was saved during the previous connection process when reconnecting. This location information is very important for GameAnvil internally inquiring about the location of a user or room and delivering messages based on it.
