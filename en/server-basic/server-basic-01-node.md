## Game > GameAnvil > Server Concept Description > Node



## Node

The most basic unit of GameAnvil server configuration is a node. Each node independently performs the features appropriate for that role. You can freely set up how many nodes you want to perform. Before describing the nodes in detail, the nodes are divided by roles as follows. 

| Node       | Required/Optional      | Features                                        | Implement content | Network access       |
| ---------- |------------| ------------------------------------------- | ----------- | ------------------- |
| Gateway    | tokenId         | Handles client connection and authentication               | Available        | public              |
| Game       | tokenId         | Handles content as actual game server            | Available        | private             |
| Support    | Optional         | Supports independent services as necessary | Available        | private or public |
| Match      | Optional         | Performs matchmaking                          | Available        | private             |
| Location   | Required (auto-configured) | Stores and manages the location information of users and rooms     | Unavailable      | private             |
| Management | Required (auto-configured) | Collection of server information and communication with Console/Agent      | Unavailable      | private             |
| Ipc        | Required (auto-configured) | Process of the Inter-process communication of the GameAnvil server    | Unavailable      | private             |

In particular, the user only needs to focus on the nodes (Gateway, Game, Support, Match) that can implement the content. The rest is used internally by engine and there is nothing additionally for the user to implement. In addition, there must be at least one required node to enable the entire server to operate normally. The location node needs to be explicitly declared in GameAnvilConfig.json.

This node hierarchy is described in the picture below:

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



## Single-Threaded

In GameAnvil, a single node is processed by a single thread. This is important. Each node basically process everything asynchronously and this node thread must be guaranteed to be run uninterrupted. This model is similar to that of Vert.x or Node.js.

The biggest advantage of single threads is definitely Lock-Free. Users do not need and should not explicitly use Lock, except in special cases. As soon as you use a lock, that node thread can stop processing. This means that logic stops across that node, so you should never use Lock for node threads when you develop it in GameAnvil. Therefore, game-related objects should never be shared between node threads and external threads. However, if you create an external thread separately and delegate tasks, the lock between external threads is fine. If you need to delegate tasks to external threads or obtain results for delegated tasks, use [Async Support API](../server-impl/server-impl-10-async.md) provided by GameAnvil.

