## Game > GameAnvil > Server Development Guide > Configuring and Running a Server

## Configuration

GameAnvil can configure servers in two main ways. The most common method is to configure the server to run through the GUI via NHN Cloud's console. This is the method you will use for VM-based services or testing on the cloud. However, this method is cumbersome and inconvenient to use during development, so we provide the GameAnvilConfig.json file so that you can configure the server directly on your PC during development. The default path to this file is resources/ in your project. If you want to configure the server with multiple processes, each process will need its own GameAnvilConfig.json.

Some of the values you enter in this configuration file must match the values you use in your server code to register with the engine as annotations. For example, earlier we configured the GameNode as shown below. The service name "MyGame" used here must be set to the same in GameAnvilConfig.json, otherwise the service will not be found and the server will fail to start up.

```java

var gameServiceBuilder = gameAnvilServerBuilder.createGameService("MyGame");
gameServiceBuilder.gameNode(SampleGameNode::new, config -> {
    // Add handler in here 
});

```

So, let's take a look at these.



## Modify GameAnvilConfig.json

GameAnvilConfig provides a very large number of settings to flexibly configure your server. However, the engine defaults are sufficient for most of them, so we will only describe the settings that you need to understand. They are divided into five main categories



### Common settings (common)

Set common information that is required regardless of node configuration. 

```json
"common": {
    "ipcIp": "127.0.0.1", // IP address for internal network communication. (Specify the machine's IP address.)
    "meetEndPoints": ["127.0.0.1:18000"], // Register the target node's common IP and ipcPort. (The server endpoint can be included; multiple entries can be listed.)
    "debugMode": false // This option prevents various timeouts during debugging. This must be false in real-world use.
},
```



Each configuration item is described below.

| Name            | Description                                                         | Default value  |
|---------------| ------------------------------------------------------------ | ------- |
| icpIp         | (specifies the IP of the machine). If no value is provided, it is automatically assigned to private ip. | - |
| meetEndPoints | Register the target node's common IP and communicatePort with the corresponding server endpoint in the list of possible inclusions. |-|
| debugMode     | This must be false for optional realizations to prevent various timeouts from occurring when debugging. | false   |



### Location

Location nodes are actually system nodes that are responsible for user and room location information for the entire server. They are managed by the engine and are intended for direct use, so you don't need to implement anything additional. However, it's up to you to decide how many of these system nodes you want to configure, so we provide a separate configuration method. During development, you can use the example below as is. On the other hand, the configuration for the actual service must be applied appropriately depending on the content or volume of the game, so it is recommended to have a separate discussion with GameAnvil representatives. Each configuration item is shown below.

```json
"location": {
    "clusterSize": 1, // How many machines (VMs) does it consist of in total? [1 to 5]
    "replicaSize": 1, // Size of the replication group (number of masters + slaves) [2 to 5]
    "shardFactor": 2 // Factor for sharding [2 to 5]
},
```

> [Note]
>
> If the corresponding setting ("location") key-value is missing or clusterSize is 0, the process will not create a location node.*


Each configuration item is described below.

| Name        | Description                                                         | Default value |
| ----------- | ------------------------------------------------------------ | ------ |
| clusterSize | Number of servers configured (VMs)                                       | -      |
| replicaSize | Size of replication group Number of masters + slaves                       | -      |
| shardFactor | Arguments for sharding <br />-Count of all shards = clusterSize x replicaSize x shardFactor <br />-Number of shards to run on one machine (VM) = replicaSize x shardFactor <br />-Total number of unique shards (number of master shards) = clusterSize x shardFactor | -      |

### Location Cluster

You can make requests to the master location node to retrieve location information, such as users and rooms. However, you can only send requests after all location nodes have completed clustering. When you enable location nodes, the engine spins up the location nodes and checks to see if all location nodes have completed clustering. If full clustering is not complete within a certain amount of time, it leaves an error log.

### Location Fail-over

If you set replicaSize to 2 or more, there will be a master location node and a slave location node. Location fail-over is implemented so that if the master location node dies, the slave location node takes over the role of the master. If you ` restart`the server where the master location node was located, add `-DrestartedAfterDown=true`to the VmOption to distinguish between them. In this case, all restarted location nodes will run as slaves.

### Match

Match nodes are the nodes that perform matchmaking, that is, they drive the matchmaker that you implement. You only need to decide how many of these match nodes you want to drive. For normal development or small services, one match node is sufficient.

```json
"match": {
    "nodeCnt": 1 // [0 to 99]
},
```

> [Note]
>
> If the corresponding set("match") key-value is missing or nodeCnt is 0, the process will not create a match node.*



Each configuration item is described below.

| Name    | Description            | Default value |
| ------- | --------------- | ------ |
| nodeCnt | Number of match nodes | -      |




### gateway

Gateway nodes are the nodes that clients connect to, so you need to have the right number of nodes for the number of clients you want to connect to.

```json
"gateway": {
  "connectGroup": {   // Connection type.
    "TCP_SOCKET": {
      "port": 18200   // Port that will be connected to client. [1 to 66,535]
    }
  },
  "nodeCnt": 4,       // Number of nodes. (node numbers are assigned starting from 0) [0 to 99]
  "duplicateLoginServices": {
    "MyGame": ["MyChatting"]
  }
},
```


> [Note]
>
> If the corresponding setting ("gateway") key-value is missing or nodeCnt is 0, the process will not create a gateway node.


Each configuration item is described below.

| Name                             | Description                                                         | Default value |
| -------------------------------- | ------------------------------------------------------------ | ------ |
| nodeCnt                          | Number of nodes                                                    | -      |
| ip                               | IP address for the client to connect to<br> (automatically set to the machine's private IP if none exists) | -      |
| connectGroup                     | Gateway nodes support TCP sockets and WEB sockets           | -      |
| connectGroup : port              | The port the client will connect to                                     | -      |
| connectGroup : idleClientTimeout | Timeout between no data sent or received and connection cleanup (0 is not used) | 4000   |



### game

Game nodes are the nodes where actual game-related objects are created and content is played. You can configure the number of nodes, channels, and more to suit the nature of your game content.


```json
"game": [
  {
    "serviceId": 1, // Must be unique within the server group [0 to 99]
    "serviceName": "MyGame", // Must be unique within the server group
    "nodeCnt": 4, // Number of nodes. If nodeCnt is 0, no game nodes will be created. It is recommended to use the number of machine cores [0 to 99]
    "channelIDs": [
    ["Beginner"],
    ["Intermediate"],
    ["Beginner"],
    ["Intermediate"]
    ], // Channel IDs to assign to each node. (Does not have to be unique. Multiple "" strings can be used regardless of channel.)
    "userTimeout": 5000 // Timeout for user object deletion after disconnect. [0 to 604800000 (7 days)]
  }
],
```

> [Note]

> The service name that you used in your server code to register your game node class with the engine must be entered here in the settings. Here is an example of an annotation for a Game Node, where we used the service name "MyGame".*
> 
> ```java
> var gameServiceBuilder = gameAnvilServerBuilder.createGameService("MyGame");
> gameServiceBuilder.gameNode(SampleGameNode::new, config -> {
>   // Add handler in here
> });
> ```
>
> Therefore, you must set the "MyGame" service name in GameAnvilConfig as follows. The example below shows the "MyGame" service name mapped to the service ID "1". This service name and service ID must be the only values in the entire configuration.*
>
> ```json
> "game": [
>   {
>     "serviceId": 1,           // Must be unique across servers [0 to 99]
>     "serviceName": "MyGame",  // Must be unique across servers
>     "nodeCnt": 4,             // If the number of nodes, nodeCnt, is 0, no Game Node is created. It is recommended to enter the number of machine cores [0 to 99]
>     "channelIDs": [
>       ["Beginner"],
>       ["Intermediate"],
>       ["Beginner"],
>       ["Intermediate"]
>     ],                        // Channel ID to be assigned to each node. (Does not have to be unique. Multiple "" strings can be used to distinguish channels.)
>     "userTimeout": 5000       // Timeout for user object removal after disconnect. [0 to 604,800,000 (7 days)]
>   },
>   {
>     "serviceId": 2,             // Must be unique across servers [0 to 99]
>     "serviceName": "MyChatting",// Must be unique across servers
>     "nodeCnt": 1,               // If the number of nodes, nodeCnt, is 0, no Game Node is created. It is recommended to enter the number of machine cores [0 to 99]
>     "channelIDs": [[""]],       // Channel ID to be assigned to each node. (Does not have to be unique. Multiple "" strings can be used to distinguish channels.)
>     "userTimeout": 5000         // Timeout for user object removal after disconnect. [0 to 604,800,000 (7 days)]
>   }
> ],
> ```
>
> The configuration example above additionally shows the "MyChatting" service. You can see that the service ID is "2", which is different from "MyGame".*

> [Note]

As shown in the above settings, the default channel configuration is one game node per thread.

> If you use a lot of channels, but only a few channels are heavily populated, performance can degrade due to the lots of threads. Therefore, we support multiple game nodes per thread.
> 
> ```json
> "game": [
>   {
>     "serviceId": 1,           // Must be unique across servers [0 to 99]
>     "serviceName": "MyGame",  // Must be unique across servers
>     "nodeCnt": 4,             // If the number of nodes, nodeCnt, is 0, no Game Node is created. It is recommended to enter the number of machine cores [0 to 99]
>     "channelIDs": [
>       ["Beginner", "Intermediate"], // Thread1
>       ["Expert"]         // Thread2
>     ],                        // Channel ID to be assigned to each node. (Does not have to be unique. Multiple "" strings can be used to distinguish channels.)
>     "userTimeout": 5000       // Timeout for user object removal after disconnect. [0 to 604,800,000 (7 days)]
>   }
> ],
>```
> Game node: total 3 (igmore nodeCnt value)
> 
> Game node thread: total 2
> 
> Thread1: game node1 ("beginner"), game node2 ("intermediate")
> 
> Thread2: game node3 ("expert")

> [Note]
>
> If the key-value for the setting ("game") is missing or nodeCnt is 0, no game nodes will be created in the process.

Each configuration item is described below:

| Name | Description | Default |
|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------|
| serviceId | The service ID. This must be a unique numeric value across the entire server configuration. <br/>Only values ​​between 0 and 99 are allowed. | 0 |
| serviceName | The service name. This must be a unique string across the entire server configuration. <br/>This service name is used to register game nodes with the engine. | ​​|
| nodeCnt | Sets the number of game nodes. <br /> If 0, no game nodes will be created. | 0 |
| channelIDs | The channel IDs assigned to each node. These do not need to be unique. <br>However, "" indicates that no channels are used. | |
| userTimeout | Sets the timeout time (ms) for removing a user object after disconnecting. <br/>If the user is not reconnected before the specified time elapses after the user status is disconnected, the user object will be deleted when the logout process is completed. <br/>Sets the period for which the user object will be managed on the server without being removed after the client disconnects. <br/>If 0, the user object will not be maintained and will be deleted immediately. | 0 |

### Support

Support nodes are nodes that fulfill a secondary role. They can also communicate directly with clients, making them ideal for exchanging game-related information or delegating periodic tasks or tasks that require independent implementation outside of the game.

```json
"support": [
  {
    "serviceId": 3,       // [0 to 99]
    "serviceName": "DB",
    "nodeCnt": 2,         // [0 to 99]
    "restIp": "127.0.0.1",
    "restPort": 18611     // REST connection port. [ 0 to 65,535]
  },
  {
    "serviceId": 4,
    "serviceName": "SampleWebSupport",
    "nodeCnt": 2,
    "restPort": 18612
  }
},
```


> [Note]
>
> As with the Game Node we saw earlier, the service name you used in your server code to register the Support Node class you created with the engine must be entered in the settings here.

> [Note]
>
> If the corresponding setting ("support") key-value is missing or nodeCnt is 0, the process will not create a support node.*

Each configuration item is described below.


| Name        | Description                                              | Default value |
| ----------- | ------------------------------------------------- | ------ |
| nodeCnt     | Number of nodes                                         | - |
| serviceId   | As the service ID, it must be the only numeric value in the entire server configuration, but can only be a value between 1 and 99. | - |
| serviceName | This must be the only string in the entire server configuration as the service name. This service name is used in the annotation to register the support node with the engine. | - |
| restIp      | IP address for RESTful requests<br/> (automatically set to the machine's private IP if none exists) | - |
| restPort    | Ports for RESTful requests                         |-|



## VM Options

We're sharing the core of the VM options used by our development team to run GameAnvil servers. The VM options we recommend here have been validated over the course of many large-scale performance tests, and we encourage you to use them as a guide and make appropriate changes to suit your needs.



### Recommended VM Options

* Set the memory size according to your system. For reference, the development team uses 4-6GB on 8GB machines and 10-12GB on 16GB machines.
* GameAnvil uses G1GC as its official GC, so you should use G1GC unless you have a specific reason not to.
* We strongly recommend adding a minimal option for GC logs, especially during development.

#### Java 21

```
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/java.lang.invoke=ALL-UNNAMED 
-Xms6g
-Xmx6g
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication

-Xloggc:gc.log
```

* The `--add-opens java.base/java.lang=ALL-UNNAMED` statement in the first line is a JVM option required because GameAnvil uses a custom virtual thread.
Removing this setting may cause GameAnvil to malfunction.
* The `--add-opens java.base/java.lang.invoke=ALL-UNNAMED` statement in the second line is a statement for GameAnvil to optimize reflection performance. Removing this option will still launch the game. But performance may be reduced.

### VM Options for GC Logs

Options for GC logs are essential for tracking memory licks and the like, so we recommend adding the following GC log-related options at least during development if for no other reason. However, in production, you may need to add only some optimized options as needed, as they may affect performance. In addition to the options recommended above, you can add the following options for each Java version.

#### Java 21

```
-Xlog:gc*,safepoint:YOUR_PATH/gc.log:time,level,tags,uptime
```

You can also add file rotation options as follows:

```
-Xlog:gc*,safepoint:YOUR_PATH/gc.log:time,level,tags,uptime:filecount=100,filesize=10M
```
