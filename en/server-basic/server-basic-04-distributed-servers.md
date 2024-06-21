## Game > GameAnvil > Server Concept Description > Distribution Servers



## Distributed Server

Previously, in the basic concept, GameAnvil's node configuration is as shown in the image below. In other words, a process can be operated by freely configuring multiple nodes. However, all GameAnvil processes must have one inter-process communication (IPC) node. This IPC node is responsible for communication between GameAnvil processes. In fact, there are more low-level nodes that are responsible for actual network processing, but the user should understand this as an IPC node as a whole.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



### Node-to-Node Communication

The picture below depicts two or more GameAnvil processes communicating each other through these IPC nodes. In the picture, two different GameAnvil processes run the nodes in different configurations. At this point, each node can communicate each other.

In order to communicate with many different process nodes, each node passes messages through IPC nodes. On the other hand, the nodes on the same process use queue to communicate with each other. Therefore, in this case, the communication does not go through IPC nodes.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/IPC.png)



### Meetpoint

For these IPCs, processes interconnect through Meetpoint. GameAnvil can set one or more Meetpoint IP address pairs in the following form. GameAnvil process synchronizes the entire server group information by attempting to access one of the Meetpoint addresses set during initial drive.

```
"common": {

    "meetEndPoints": [
      "10.1.2.1:18000",
      "10.1.2.2:18000",
    ],

}
```

