## Game > GameAnvil > Server Concept Description > Node



## Node

A node is the most basic unit of the GameAnvil server configuration. Each node performs the functions tailored to its role independently. You can freely set which role to perform with several nodes. Before detailing the node, you can divide the nodes by role as follows: 

| Node | Required/Optional | Function | Content Implementation | Network Access |
| ---------- |------------| ------------------------------------------- | ----------- | ------------------- |
| Gateway | Required | Handles client connection and authentication | Available | public |
| Game | Required | Handle content as an actual game server | Available | private |
| Support | Optional | Support implementation as a separate service as needed | Available | private or public |
| Match | Optional | Perform matchmaking | Available | private |
| Location | Required (automatic configuration) | Store and manage location information such as users and rooms | Unavailable | private |
| Management | Required (automatic configuration) | Gather server information and communicates with Console/Agent | Unavailable | private |
| Ipc | Required (automatic configuration) | Handle inter-process communication with the GameAnvil server | Unavailable | private |

In particular, users can focus on the nodes (Gateway, Game, Support, and Match) where content can be implemented. The rest is used internally by engines, and nothing can be implemented additionally by users. At least one required node must exist for the entire server to run normally. In this case, the Location node needs to be explicitly declared in GameAnvilConfig.json.

The hierarchical structure of these nodes is the same as the image below:

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



## Single Thread (Single-Threaded)

One node in GameAnvil is processed as a single thread. This is very important. By default, each node must be processed asynchronously, and ensure that these node threads run continuously without blocking. These run models are very compatible with those of Vert.x or Node.js.

The biggest advantage of single threads is also Lock-Free. The user does not need to use and cannot use a lock explicitly except in special cases. As soon as you use a block, the node thread might stop processing. It means that logic stops across the node, so you should never use lock for node threads when developing from GameAnvil. Therefore, objects related to the game must never be shared between the node thread and the external thread. However, when creating external threads separately and delegating tasks, the gap between the external threads does not matter. If you need to delegate tasks to an external thread or get results for tasks that you have delegated, use the [asynchronous support API](../server-impl/server-impl-10-async.md) provided by GameAnvil.

