## Game > GameAnvil > Server Development Guide > Getting Started

## Before Starting

This article discusses the basic elements and implementation methods required to implement a server using GameAnvil. We recommend that you refer to the [GameAnvil tutorial](../tutorial/tutorial-01-basic.md) provided along with this document.

GameAnvil servers are by default configured as nodes. The nodes where the user's code runs are four in total, as shown in the image below. Here we discuss the server development method based on the implementation methods for these four nodes.

![Nodes on Network.png](https://static.toastoven.net/prod_gameanvil/images/user_nodes_on_network_.png)

## Redefine Callback

By default, most GameAnvil features are provided in the form of callbacks. Engine users will use most features after implementing the default interface provided by GameAnvil (IGatewayNode, ISupportNode, and IGameNode) in a way that redefines these callback methods. In this process, only the callback methods required by the engine user are implemented, so some callback methods may be ignored. These default interfaces are each name that begins with "I" and are provided as com.nhn.gameanvil packages or subpackets.

![callback-1.png](https://static.toastoven.net/prod_gameanvil/images/callback-1.png)

## Start of All Implements, Nodes

For example, all nodes must redefine the callback methods below in common. And each node may require the implementation of additional callback methods for the role. For example, in the code below, SampleGatewayNode implements IGatewayNode, which is the default interface of GatewayNode.

All interface nodes, including IGatewayNode, offer the following callback methods in common. The context interface according to the type implemented by the onCreate() method is passed to the parameters. When a user implements these callback methods, the engine calls the callback at a certain point in time. This is the most basic use of GameAnvil. This usage is consistent throughout the document, so you should be able to understand each chapter without much confusion.

```java
@GameAnvilGatewayNode // Register this class as Gateway to the engine
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
     * Call when you are ready
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

For the meaning and usage of these node common callbacks, see the table below:

| Callback name | Meaning | Description | |---------------- |------- |--------------------------------------------------------------------------------------------- |
| onCreate | Create object | Calls when the object is created. You receive the context where the API available for the created type can be used. If needed from the content, it can be saved and used. |
| onInit | Initialize | Calls when the node proceeds with the first initialization. If there is any initialization task required before the node runs, this callback is suitable. At this time, the node is still not being processing the message. |
| onPrepare | Prepare | The node will be called after the node is initialized. The user can proceed with any task here before the node is ready. At this time, the node can process messages. |
| onReady | Prepared | After all preparations are completed, the node is running. At this time, the node is in Ready state, so the user will be able to use all features from this time. |
| onPause | Pause | Pause the node will call the call. Here you can implement the code you want to process additionally when the node is temporarily stopped. |
| onShuttingdown | Node stopped | Calls are sent when the node receives a Shutdown command. Stopped nodes cannot be Resume. |
| onResume | Resume | Calls are called when the node restarts the run while it is temporarily stopped. Here, the user can implement the code he wants to process while in a restart. |

## Methods and variables beginning with _(underscore)

Using the engine, users can see the method or variable beginning with _ in the implementation interface. It means that it should only be used internally in the engine. The user must not access variables or methods beginning with _. This requires caution if you allow some exposures as Java is not flexible for scope control.

## gameanvil package and gameanvilcore package

Engine consists largely of two top packets, and gameanvil package is for the user. All classes or APIs in this package are free to use. On the other hand, the gameanvilcore package contains engine cores logic, so the user is not allowed to access directly. Nevertheless, the users are exposed to the scope control limitations of the Java version currently supported by GameAnvil. Special care is required to avoid the contents of the gameanvilcore package during the user code creation process.

## Configure Project with IntelliJ Template

Configuring GameAnvil projects one by one from the beginning requires many complex processes. You must write a configuration file along with loading the engine library, and also a script to write protocol specifications and a compiler to compile them. In order to avoid unnecessary waste of development time due to this sequence of processes, GameAnvil offers a template for IntelliJ that includes all of these elements. From the link below, download a template file and follow the steps below:

[Download GameAnvil Template](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil%20Template.zip?disposition=attachment)

Run IntelliJ and click `Shift Shift` to view the entire search box and search for **Import Settings...**. Or select File > Manage IDE Settings > **Import Settings...**.

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/search_import_settings.png" />

Select the template file you downloaded from the Finder screen. When a window is displayed that selects the elements you want to import, check both the file template and project template, and click OK. 

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/select_import.png" />

When the import is complete, select GameAnvil Template from the User-defined tab of the left menu when the Create New Project window opens.

<img src="https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/01-getting-started/new_project_project_template.png" />

Once completed, a template project is created that can immediately start developing the GameAnvil server, as shown in the image below:

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/project_template_view.png" />

There is also a file template that allows you to immediately create various classes provided by GameAnvil. Select the desired package from the project and right-click to open the context menu. At this time, you can use the file template for each class, as shown in the image below:

<img src="https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/01-getting-started/file_template.png" />

The following is a list provided as a file template. Each file template implements one class template.

| File Template | Description |
|--------------------------------|-------------------------|
| Connection | Basic implementation of a connection |
| GameNode | Basic implementation of a game node |
| GatewayNode | Basic implementation of a gateway node |
| MessageHandler for GameNode | Basic implementation of a game node message handler |
| MessageHandler for GatewayNode | Basic implementation of a gateway node message handler |
| MessageHandler for Room | Basic implementation of a room message handler |
| MessageHandler for SupportNode | Basic implementation of a support node message handler |
| MessageHandler for User | Basic implementation of a user message handler |
| RestMessageHandler | Basic implementation of a rest message handler |
| Room | Basic implementation of a room |
| RoomMatchForm | Basic implementation of a room matching request |
| RoomMatchInfo | Basic implementation of room matching information |
| RoomMatchMaker | Basic implementation of a room matchmaker |
| Session | Basic implementation of a session |
| SupportNode | Basic implementation of the support node |
| User | Basic implementation of a game user |
| UserMatchInfo | Basic implementation of user match information |
| UserMatchMaker | Basic implementation of the user matchmaker |

 