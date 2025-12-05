## Game > GameAnvil > Server Development Guide > SupportNode Implementation

## Support Node

![SupportNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_supportnode_on_network.png)

SupportNode, as its name implies, is a node to perform secondary feature. It can implement any feature regardless of game user or room object. SupportNode also does not require access through GatewayNode, so no additional connection or session management is required. Based on these characteristics, SupportNode is often responsible for its own features. For example, you can collect and send logs, or you can use them as a role dedicated to communicating with the billing server. It is also possible to use the engine's RESTful feature as a simple Web server substitute. At this time, SupportNode may be exposed to the external network as well as the internal network as shown in the image above, so it may be responsible for the necessary features before/after connecting to the GatewayNode. The advantage of SupportNode is that it can flexibly implement and deploy arbitrary auxiliary features.

## Implement SupportNode

By default, SupportNode inherits and implement the BaseSupportNode abstract class. With the exception of the node common callback method, it is the only additional getRestMessageDispatcher method for RESTful processing. In other words, SupportNode is differentiated in this very area from the other nodes that we've seen before. That is, it can perform a user-defined RESTful processing. In other words, SupportNode is the only user node that can handle RESTful messages along with typical user-defined packet processing.

```java
@GameAnvilSupportNode(gameServiceName = "MySupport") // (1) Register with the engine as a SupportNode for a service called "MySupport"
public class SampleSupportNode implements ISupportNode {
    private ISupportNodeContext supportNodeContext;

    /**
    * Call to pass the support node context.
    * <p/>
    * Call once after the object is created.
     *
     * @param supportNodeContext Support node context
     */
    @Override
    public void onCreate(ISupportNodeContext supportNodeContext) {
        this.supportNodeContext = supportNodeContext;
    }

    /**
     * Call when a node is rest
     */
    @Override
    public void onInit() {

    }

    /**
     * Call for processing before ready
     */
    @Override
    public void onPrepare() {

    }

    /**
     * Call when Ready
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
```java
// A message processing class that executes when the protobuffer MyGame.GameNodeTest input is received.
@GameAnvilController
public class _SupportNodeTest {
    // (2) Map protocol and handler that you want to handle in SampleSupportNode
    @GameNodeMapping(
        value = MyGame.SupportNodeTest.class,   // Proto buffer to be handled
        loadClass = SampleSupportNode.class     // Target of the message (SampleSupportNode)
    )
    public void runGameNodeTest(IGameNodeDispatchContext ctx) {
        // Write down what you want to do here
    }
}
```


All nodes need a message handler registration process to process custom messages. SupportNode, like GameNode, is a node where users can implement arbitrary content. First, (1) create a support node service name specified in GameAnvilConfig. The support node service name must be one defined in GameAnvilConfig. (2) Then, connect it to the [handler](server-impl-07-message-handling.md#_2) that implements the message you want to process.