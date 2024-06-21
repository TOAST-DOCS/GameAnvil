## Game > GameAnvil > Server Development Guide > Configuring and Running a Server



## Configuration

GameAnvil can configure servers in two main ways. The most common method is to configure the server to run through the GUI via NHN Cloud's console. This is the method you will use for VM-based services or testing on the cloud. However, this method is cumbersome and inconvenient to use during development, so we provide the GameAnvilConfig.json file so that you can configure the server directly on your PC during development. The default path to this file is resources/ in your project. If you want to configure the server with multiple processes, each process will need its own GameAnvilConfig.json.

Some of the values you enter in this configuration file must match the values you use in your server code to register with the engine as annotations. For example, earlier we configured the GameNode as shown below. The service name "MyGame" used here must be set to the same in GameAnvilConfig.json, otherwise the service will not be found and the server will fail to start up.

```java
@ServiceName("MyGame") // Register with the engine as a GameNode for a service named "MyGame"
public class SampleGameNode extends BaseGameNode {
    ...
}
```

So, let's take a look at these.



## Modify GameAnvilConfig.json

GameAnvilConfig provides a very large number of settings to flexibly configure your server. However, the engine defaults are sufficient for most of them, so we will only describe the settings that you need to understand. They are divided into five main categories



### Common settings (common)

Set common information that is required regardless of node configuration. 

```json
"common": {
    "ipcIp": "127.0.0.1", // IP to use for server interprocess-communication (IPC) (specify the IP of the machine)
    "meetEndPoints": ["127.0.0.1:18000"], // Peer information to synchronize clustering information with (use 18000 if you haven't changed the port)
    "debugMode": false // option to prevent various timeouts from occurring when debugging, must be false in real world
}
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
    "clusterSize": 1, // How many machines (VMs) does it consist of in total?
    "replicaSize": 1, // Size of the replication group (number of masters + slaves)
    "shardFactor": 2 // Factor for sharding
}
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
    "nodeCnt": 1
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
    "nodeCnt": 4, // Number of nodes. (Nodes are numbered from 0)
    "ip": "127.0.0.1", // IP to connect to the client.
    "connectGroup": {
        "tcp_socket": {
            "port": 18200, // Port to connect to the client.
            "idleClientTimeout": 240000 // Timeout after no data is sent or received. (not used if 0)  
        },
        "web_socket": {
            "port": 18300,
            "idleClientTimeout": 240000
        }
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
    { "game
        "nodeCnt": 4, // The four game nodes will correspond to channels "beginner", "intermediate", "beginner", and "intermediate", respectively.
        "serviceId": 1,
        "serviceName": "Todo - Input My Service Name",
        "channelIDs": ["beginner","intermediate","intermediate","beginner","intermediate"], // Channel IDs to assign to each node (doesn't have to be unique, "" means don't use channel)
        "userTimeout": 5000 // Timeout to remove user objects after disconnect.
    }
]
```


> [Note 1]

> The service name that you used in your server code to register your game node class with the engine must be entered here in the settings. Here is an example of an annotation for a Game Node, where we used the service name "MyGame".*
> 
> ```java
> @ServiceName("MyGame") // Register with the engine as a GameNode for a service named "MyGame"
> public class SampleGameNode extends BaseGameNode {
>     ...
> }
> ```
>
> Therefore, you must set the "MyGame" service name in GameAnvilConfig as follows. The example below shows the "MyGame" service name mapped to the service ID "1". This service name and service ID must be the only values in the entire configuration.*
>
> ```json
> "game": [
>     {
>        "nodeCnt": 4, // The four game nodes will correspond to channels "beginner", "intermediate", "beginner", and "intermediate", respectively.
>        "serviceId": 1,
>        "serviceName": "MyGame", // Service name
>        "channelIDs": ["beginner","intermediate","beginner","intermediate"], // Channel IDs to give to each node (does not need to be unique, "" means don't use channel)
>        "userTimeout": 5000 // How long to manage the user object without removing it from the server after the client disconnects.
>     },
>     {
>         "nodeCnt": 1,
>         "serviceId": 2,
>        "serviceName": "MyChatting", // Service name
>        "channelIDs": [""], // Channel IDs to be given to each node (does not need to be unique, "" means channel not used)
>        "userTimeout": 5000 // How long to manage the user object without removing it from the server after the client disconnects.
>     }
> ]
> ```
>
> The configuration example above additionally shows the "MyChatting" service. You can see that the service ID is "2", which is different from "MyGame".*

> [Note 2]

> If the corresponding set("game") key-value is missing or nodeCnt is 0, the process will not create a game node.

Each configuration item is described below.

| Name        | Description                                                                         | Default value |
| ----------- |----------------------------------------------------------------------------| ------ |
| nodeCnt     | Number of nodes                                                                      |        |
| serviceId   | As the service ID, it must be the only numeric value in the entire server configuration, but can only be a value between 1 and 99.           |        |
| serviceName | This is the service name and should be the only string in the entire server configuration. This service name is used in the annotation to register the game node with the engine. |        |
| channelIDs  | This does not have to be the only value for the channel ID to be given to each node. <br>However, "" means that the channel is not used.          |        |
| userTimeout | Sets how long to manage user objects without removing them from the server after the client disconnects.                       | 10000  |


### Support

Support nodes are nodes that fulfill a secondary role. They can also communicate directly with clients, making them ideal for exchanging game-related information or delegating periodic tasks or tasks that require independent implementation outside of the game.

```json
"support": [
    {
        "nodeCnt": 2,
        "serviceId": 3,
        "serviceName": "DB",
        "restIp": "127.0.0.1",
        "restPort": 17251
    },
    {
        "nodeCnt": 2,
        "serviceId": 4,
        "serviceName": "SampleWebSupport",
        "restIp": "127.0.0.1",
        "restPort": 17250
    }
],
```


> [Note 1]
>
> As with the Game Node we saw earlier, the service name you used in your server code to register the Support Node class you created with the engine must be entered in the settings here.* 

> [Note 2]
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

The javaagent option on the top line is only required if you are using Fiber Code in a JIT fashion. If you are using it in an AOT fashion, you should clear this line; we will cover [Bytecode Instrumentation](../server-basic/server-basic-06-bytecode-instrument)in more detail later. Set the memory size according to your system. For reference, the development team uses 4-6GB on 8GB machines and 10-12GB on 16GB machines, and GameAnvil uses G1GC as its official GC, so you should use G1GC unless you have a specific reason not to. Finally, we strongly recommend adding a minimal option for GC logs, especially during development.

####  Java 8 

```
-javaagent:YOUR_PATH\quasar-core-0.7.10-jdk8.jar=bm
-Xms4g
-Xmx4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication

-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:gc.log
```

#### Java 11

```
-javaagent:YOUR_PATH\quasar-core-0.8.0-jdk11.jar=bm

--add-opens=java.base/java.lang=ALL-UNNAMED
--illegal-access=deny

-Xms4g
-Xmx4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication

-Xlog:gc*,safepoint:./log/gc.log:time,level,tags
```



### VM Options for GC Logs

Options for GC logs are essential for tracking memory licks and the like, so we recommend adding the following GC log-related options at least during development if for no other reason. However, in production, you may need to add only some optimized options as needed, as they may affect performance. In addition to the options recommended above, you can add the following options for each Java version.

#### Java 8
```
-XX:+PrintGCDetails
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-XX:+PrintHeapAtGC
-XX:+PrintReferenceGC
-Xloggc:YOUR_PATH/gc.log
```
You can also add file rotation options as follows
```
-XX:+PrintGCDetails
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-XX:+PrintHeapAtGC
-XX:+PrintReferenceGC
-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=100
-XX:GCLogFileSize=10M
-Xloggc:YOUR_PATH/gc.log
```

#### Java 11
```
-Xlog:gc*,safepoint:YOUR_PATH/gc.log:time,level,tags,uptime
```
You can also add file rotation options as follows
```
-Xlog:gc*,safepoint:YOUR_PATH/gc.log:time,level,tags,uptime:filecount=100,filesize=10M
```

