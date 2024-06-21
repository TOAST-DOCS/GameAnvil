##  Game  GameAnvil  Server Development Guide  Getting Started



## Before Getting Started

This document describes the basic elements required to implement a server using GameAnvil and how to implement it. It is recommended that you refer to the reference project [GameAnvil Reference Server](../reference/reference-1-server) that comes with this document. Also refer to the description of each API and UML diagram through [JavaDoc document](https://gameplatform.toast.com/docs/api/).

GameAnvil servers are basically configured on a node basis. Among them, there are a total of four nodes where the user's code runs, as shown in the image below. This section describes how to develop a server, focusing on how to implement these four nodes.

![Nodes on Network.png](https://static.toastoven.net/prod_gameanvil/images/user_nodes_on_network_.png)



## Callback Override

By default, most features in GameAnvil come in the form of callbacks. In other words, engine users inherit the Base Classes provided by GameAnvil and then override these callback methods, so most features used as such. This process requires engine users to implement only the callback methods they require, so some callback methods can be ignored. All of these base classes have names that begin with "Base" and are available in the com.n.gameAnvil package or its sub-packages.

![callback-1.png](https://static.toastoven.net/prod_gameanvil/images/callback-1.png)

## Start of All Implementations, Nodes

For example, all nodes must override the following callback methods in common. And each node may require the implementation of additional callback methods for its role. For example, SampleGatewayNode inherits BaseGatewayNode, the default class of GatewayNode.

All BaseNodes, including BaseGatewayNodes, offer the following callback methods in common. When a user implements this callback method, the engine calls the callback at a certain point in time. This is the most basic use of GameAnvil. This usage is similar throughout the document, so you will be able to understand each chapter without much sense of heterogeneity (difficulty).

```java
@GatewayNode  
public class SampleGatewayNode extends BaseGatewayNode {  
  
    /**  
     * Call at initialization time 
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended.  
     */  
    @Override  
    public void onInit() throws SuspendExecution {    
    }  
  
    /**  
     * Call for what to do before you're ready 
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended 
 
     */  
    @Override  
    public void onPrepare() throws SuspendExecution {  
    }  
  
    /**  
     * Call when Ready  
     *  
     * @throws SuspendExecution This method means that the fiber can be suspended 
 
     */  
    @Override  
    public void onReady() throws SuspendExecution {    
    }  
  
    /**  
     * Call when paused  
     *  
     * @param payload Additional information that contents want to convey 
     * @throws SuspendExecution This method means that the fiber can be suspended 
     */  
    @Override  
    public void onPause(Payload payload) throws SuspendExecution {    
    }  
  
 
 
 
 
    /**  
     * Call when resuming  
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
}
```

Please refer to the table below for the meaning and use of these node common callbacks.


| Callback name  | Description               | Description                                                                                                                                          |
| ------------ | -------------------- |---------------------------------------------------------------------------------------------------------------------------------------------|
| onInit     | Initialize             | It is called when the node is performing initialization. This callback is appropriate if there is an initialization task required before the node runs. At this time, the node is not processing the message yet.                                                   |
| onPrepare  | Preparation               | It is called after the node initialization is complete. Users can process any tasks here before the node is ready. At this time, node can process message.                                                  |
| onReady    | Preparation ready           | After the node is all ready, it is drive completion stage. At this time, the node is in ready state, so the user can use all features from this point on.                                                              |
| onPause    | Pause          | It is called when a node is paused. Users can implement additional code they want to process here when the node is paused.                                                                        |
| onResume   | Resume               | It is called when the node resumes running from a pause. Users can implement the code they want to process here in the resume state.                                                                 |
| onShutdown | Node shutdown          | It is called when the node stops completely. the shutdown node cannot resume.                                                                                            |
| getMessageDispatcher | Has hackets to process | It returns a message when the node has message to process. Users can use the dispatchers they declare. For more information, refer to [Message Processing](./server-impl-07-message-handling#13-getMessageDispatcher). |



## throws SuspendExecution

In the previous example code, we looked at the following exception signatures of all callback methods.

```java
throws SuspendExecution
```

This is not an actual exception. It is a kind of bypass method used by [Quasar](https://docs.paralleluniverse.co/quasar/) to control the flow of fiber. This exception signature means that the method can suspend the fiber at any time.

Therefore, this exception should never be handled directly in the following way. Refer to [Suspendable](../server-basic/server-basic-03-suspendable) for more information.

```java
try {  
    ...  
} catch (SuspendExecution e)   
{  
    // Never catch SuspendExecution exceptions. 
}
```



## Methods and variables starting with \_(underscore)

While using the engine, you can see methods or variables that begin within the user-inherited Base Class. This means that it is only for use inside the engine. This means that the user should not access variables or methods that begin with \_\_. This is because, unlike C++, Java's scope control is inflexible, allowing some exposure, so you need to be careful.



## Gameanvil package and gameanvilcore package

The engine is largely composed of two parent packages. One of them, the gameanvil package, is for the user. Any class or API within this package is free to use.

On the other hand, the gamevilcore package contains engine core logic, so we do not allow users to access it directly. Nevertheless, the exposure to users is due to the scope control restrictions of the Java version currently supported by GameAnvil. Special caution should be taken to ensure that the contents of gamevilcore package are not included in the process of writing user code.



## Configure projects with InteliJ templates

Configuring GameAnvil projects one by one from the scratch requires a number of complex processes. In addition to importing the engine library, you need to create a configuration file, and you need a script to write protocol specifications and a compiler to compile them. To avoid unnecessary waste of development time due to this series of processes, GameAnvil provides a template for InteliJ that includes all of these. Download the template file from the link below and follow these steps.



[GameAnvil Template download\](https://static.toastoven.net/prod_gameanvil/files/GameAnvil Template.zip?disposition=attachment)



After running IntelliJ, search for **ImportSettings...** by displaying the entire search window with `ShiftShift` shortcut. Or select File > Manage IDESettings > **ImportSettings...**.

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/search_import_settings.png" />

On Finder screen, select the template file you downloaded. When a window appears to select elements to import, as shown in the image below, check both the file template and the project template, and click on OK. When the import is complete

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/select_import.png" />

When Create a New Project window opens, select GameAnvil Template from User-Defined tab on the left menu.

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/new_project_project_template.png" />

Once completed, a template project will be created to start GameAnvil server development immediately, as shown in the following image.

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/project_template_view.png" />

We also provide file templates that allow you to immediately create different classes provided by GameAnvil. Select the package you want in the project and right-click to open the context menu. At this time, file templates for each class are available, as shown in the following image.

<img src="https://static.toastoven.net/prod_gameanvil/images/server-impl/file_template.png" />

The following is a list of file templates provided. Each template implements one class template.

| File template     | Description               |
| -------------- |------------------|
| GatewayNode    | Basic Implementation of gateway nodes  |
| GameNode       | Basic implementation of game nodes     |
| SupportNode    | Basic implementation of support nodes    |
| Connection     | Basic implementation of connection class   |
| Session        | Basic implementation of session class    |
| GameUser       | Basic implementation of game user     |
| GameRoom       | Basic implementation of game room      |
| UserMatchMaker | Basic implementation of user match maker |
| UserMatchInfo  | Basic implementation of user match information  |
| RoomMatchMaker | Basic implementation of room matching information  |
| RoomMatchForm  | Basic implementation of room matching application form    |
| RoomMatchInfo  | Basic implementation of room matching information    |

