## Game > GameAnvil > Server Development Guide > Message Handling

![GatewayNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/three_steps_for_message_process_1213.png)

## Handle General Messages

* GameAnvil's message processing is largely divided into three parts, as shown in the image above. These three parts interlock together to create a message processing flow.
* Among these, onDispatch callback implementation is done by engine itself, and the engine user implements the Dispatcher's declaration and message handler to handle the packet.

### Implement Message Handler and Connect Messages to Handlers

We now implement that message handler directly. At this time, GameAnvil uses naming that starts with __ for the message handler in all sample codes as well as inside the engine. In all of the example codes that appear in the future, all classes that begin with _ are message handlers. The most basic forms of message handlers are as follows. RECEVER_CLASS means the class that receives the message.

```java
@GameAnvilController // Use this class as a message handler.
public class _MyEchoHandler {

   @MAPPING_CLASS // Declare which messages this method will handle.
    public void execute(CONTEXT_CLASS ctx, EchoSend request) {
        System.out.println("Receive EchoSend Message!");
    }
}
```

For example, if the recipient of the packet is a GameUser, the message handler is as follows.

```java
@GameAnvilController // Use this class as a message handler.
public class _MyEchoHandler {

    @GameUserMapping(
        value = EchoSend.class,             // Proto buffer to process
        loadClass = SampleGameUser.class    // Recipient of the message
    )
    public void execute(IUserDispatchContext ctx, EchoSend request) {
        System.out.println("Receive EchoSend Message!");
    }
}
```

## Handle http Request

There are two types of packet dispatchers: the general packet dispatcher discussed earlier and a dispatcher for handling HTTP requests. The overall usage of both is nearly identical. However, only SupportNode supports HTTP message processing.

### Create http Packet Dispatcher and Connect Messages to Handlers

The following example code declares a different @SupportNodeHttpMapping than before to handle RESTful requests. It also associates the handler with an API in the form of a URL, rather than a message class. The difference is that the HTTP message handler passes a RestObject as an argument.

```java 
@GameAnvilController
public class _RestAuthReq {

    @SupportNodeHttpMapping(path = "/hello", method = "GET", loadClass = WebSupportNode.class)
    public void execute(WebSupportNode node, RestObject restObject) {
		// ...
    }  
}
```