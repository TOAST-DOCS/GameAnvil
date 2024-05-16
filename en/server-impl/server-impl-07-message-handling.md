## Game > GameAnvil > Server Development Guide > Message Handling

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/three_steps_for_message_process_1213.png)

## Process general messages

* GameAnvil's message processing is largely divided into three parts, as shown in the image above. These three parts interlock together to create a message processing flow.
* Among these, onDispatch callback implementation is done by engine itself, and the engine user implements the Dispatcher's declaration and message handler to handle the packet.

### Create packet dispatchers and connect messages to handlers

First, create a packet dispatcher for message processing. This dispatcher is used for the corresponding class, so be sure to generate static to prevent unnecessary waste of resources.

```java
// (create packet dispatcher      
private static final MessageDispatcher<MY_USER_CLASS> messageDispatcher = new MessageDispatcher<>();
```

Connect the desired message and handler to the dispatcher that you created.

```java
// Mapping Handlers and Messages you want to process 
static {  
    messageDispatcher.registerMsg(MyProto.MyMsg.class, _MyMsgHandler.class);
 
} 
```


### Implementing message handler

We now implement that message handler directly. At this time, GameAnvil uses naming that starts with \__ for the message handler in all sample codes as well as inside the engine. In all of the example codes that appear in the future, all classes that begin with _ are message handlers. The most basic forms of message handlers are as follows. RECEVER_CLASS means the class that receives the message.

```java
public class _MyGameMsg implements MessageHandler<RECEIVER_CLASS, MyProto.MyMsg> {

    private static final Logger logger = getLogger(_MyMsgHandler.class);

    @Override
    public void execute(RECEIVER_CLASS receiver, MyProto.MyMsg myMsg) throws SuspendExecution {
    }

}
```

For example, if the recipient of the packet is a GameUser, the message handler is as follows.

```java
public class _MyGameMsg implements MessageHandler<GameUser, MyProto.MyMsg> {

    private static final Logger logger = getLogger(_MyMsgHandler.class);

    @Override
    public void execute(GameUser receiver, MyProto.MyMsg myMsg) throws SuspendExecution {
    }

}
```


### Implementing getMessageDispatcher callback

We have implemented both packet dispatchers and message handlers. Now we just need to hand over the dispatcher when the engine has a packet to process.

```java
public class GameUser {  
    // .. omitted  
    @Override  
    public final MessageDispatcher<GameUser> getMessageDispatcher() {  
        return packetDispatcher;  
    }  
    // .. omitted  
}
```

So far, we've looked at the flow of general packet processing. We'll look at the processing of RESTful requests right away.



## Processing RESTful request

There are two types of packet dispatchers. One is a typical packet dispatcher we've shared earlier, and the other is a dispatcher for processing RESTful requests. Overall usage is about the same. However, this RESTful message processing supports only the SupportNode. 



### Create REST packet dispatchers and connect messages to handlers

The following example code is creating a different RestPacketDispatcher to handle RESTful requests. It is also connecting an API in the form of a URL to the handler rather than a message class.

```java
private static final RestMessageDispatcher<RECEIVER_CLASS> restDispatcher = new RestMessageDispatcher<>();  
  
static {  
    //Restered with  path and method(GET, POST, ...) .  
    restDispatcher.registerMsg("/auth", RestObject.GET, _RestAuthReq.class);  
    restDispatcher.registerMsg("/echo," RestObject.GET, _RestEchoReq.class);  
    restDispatcher.registerMsg("/testGET," RestObject.GET, _RestTestGET.class);  
    restDispatcher.registerMsg("/testPOST," RestObject.POST, _RestTestPOST.class);  
}
```

As in the example code, RestMessageDispatcher is used to connect the RESTful API and the message handler that the user wants.



### Implement handler

RESTful message handlers look the same except for the difference that RestObject is passed to the factor instead of Packet. 

```java
public class _RestAuthReq implements RestMessageHandler<WebSupportNode> {
    private static final Logger logger = getLogger(_RestAuthReq.class);

    @Override
    public void execute(WebSupportNode node, RestObject restObject) throws SuspendExecution {
		...
    }  
}
```



### Implement RestMessageDispatcher

getRestMessageDispatcher for RESTful message processing supports only SupportNode. It can be easily implemented to pass the restMessageDispatcher.
```java
@Override
public final RestMessageDispatcher<WebSupportNode> getRestMessageDispatcher() {
    return restMessageDispatcher;
}
```
