## Game > GameAnvil > Server Development Guide > SupportNode Implementation



## Support Node

![SupportNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_supportnode_on_network.png)

SupportNode, as its name implies, is a node to perform secondary feature. It can implement any feature regardless of game user or room object. SupportNode also does not require access through GatewayNode, so no additional connection or session management is required. Based on these characteristics, SupportNode is often responsible for its own features. For example, you can collect and send logs, or you can use them as a role dedicated to communicating with the billing server. It is also possible to use the engine's RESTful feature as a simple Web server substitute. At this time, SupportNode may be exposed to the external network as well as the internal network as shown in the image above, so it may be responsible for the necessary features before/after connecting to the GatewayNode. The advantage of SupportNode is that it can flexibly implement and deploy arbitrary auxiliary features.



## Implement SupportNode

By default, SupportNode inherits and implement the BaseSupportNode abstract class. With the exception of the node common callback method, it is the only additional getRestMessageDispatcher method for RESTful processing. In other words, SupportNode is differentiated in this very area from the other nodes that we've seen before. That is, it can perform a user-defined RESTful processing. In other words, SupportNode is the only user node that can handle RESTful messages along with typical user-defined packet processing.

All nodes require a dispatcher generation and message handler registration process to handle custom messages. SupportNode, like GameNode, is one of the nodes where users can implement any random content.  First, (1) create a static packet dispatcher. At this time, it has to be statically generated so that it can take advantage of memory or performance. (2) And connect it to [Handler](server-impl-07-message-handling.md#process-general-messages) which has implemented the message you want to process. (3) Lastly, the RestMessageDispatcher processes packets using the dispatcher generated in (1). Note that the packet dispatcher used in this example code is RestMessageDispatcher. Naturally, it is also possible to use regular packet dispatcher, but in this example, RestMessageDispatcher was selected to handle RESTful requests. The usage between the two is almost the same.

You can take a look at this set of processes in the example code below, just below the annotations corresponding to (1) ~ (3).

```java
public class SampleSupportNode extends BaseSupportNode { public class SampleSupportNode extends BaseSupportNode {

    // 1.Create a REST dispatcher
    private static RestMessageDispatcher<SampleSupportNode> restMessageDispatcher = new RestMessageDispatcher<>();

    // map the URL and handler we want to handle on the 2.SampleSupportNode
    static {
        // Register with a combination of path and method(GET, POST, ...).
        restMessageDispatcher.registerMsg("/auth", RestObject.GET, _RestAuthReq.class);
        restMessageDispatcher.registerMsg("/echo", RestObject.GET, _RestEchoReq.class);
    }
    
    @Override
    public MessageDispatcher<PublicSupportNode> getMessageDispatcher() {
        return null;
    }


    @Override
    public RestMessageDispatcher<PublicSupportNode> getRestMessageDispatcher() {
        return restMessageDispatcher;
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
    public void onPause(PauseType type, Payload payload) throws SuspendExecution {}

    @Override
    public void onResume(Payload outPayload) throws SuspendExecution {}

    @Override
    public void onShutdown() throws SuspendExecution {}
}
```

The SupportNode-specific callbacks are as follows


| Callback name  | Description                       | Description                                                                                                                                                                                                             |
| ------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| getRestMessageDispatcher | There is RESTful request to process | It returns a message when the node has message to process. Users can use the dispatchers they declare. For more information, refer to [Message Processing](./server-impl-07-message-handling#13-getMessageDispatcher). |
