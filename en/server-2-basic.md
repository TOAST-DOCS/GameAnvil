## Game > GameAnvil > Server Development Guide > Basic Concepts

## 1. Node

The basic building block of GameAnvil is node. Each node performs the features that are fit for its purpose. The number of nodes and what they will do can be configured separately. Before diving deep into the nodes, let's categorize nodes with their purpose.

| Node       | Required | Feature                                        | Implement content | Network access       |
| ---------- | ---- | ------------------------------------------- | ----------- | ------------------- |
| Gateway    | Required | Handles client connection and authentication               | Possible        | public              |
| Game       | Required | Handles content as actual game server            | Possible        | private             |
| Support    | Optional | Supports independent services as necessary | Possible        | private or public |
| Match      | Optional | Performs matchmaking                          | Possible        | private             |
| Location   | Required | Stores and manages the location information of users and rooms     | Impossible      | private             |
| Management | Required | Collection of server information and communication with Console/Agent      | Impossible      | private             |
| Ipc        | Required | Process of the Inter-process communication of the GameAnvil server    | Impossible      | private             |

GameAnvil users only need to focus on (Gateway, Game, Support, and Match), which are the nodes that are used to implement content. The rest of the nodes are used internally by GameAnvil and users do not have to implement anything else. In addition, at least one required node must be present to run it.

This node hierarchy is described in the picture below:

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



## 2. Single-Threaded

In GameAnvil, a single node is processed by a single thread. This is important. Each node basically process everything asynchronously and this node thread must be guaranteed to be run uninterrupted. This model is similar to that of Vert.x or Node.js.

The biggest advantage of single thread is lock-free. Users do not have to explicitly use lock except for special cases, and they should not use lock. As explained earlier, the moment lock is used, the node thread may be stopped. This means the entire logic of the corresponding node would stop, users should not use lock on node threads while developing on GameAnvil. Therefore, no game-related objects should be shared between node threads and external threads. However, when the user creates external threads and assigns tasks to them, those external threads may share lock. If tasks are assigned to external threads or the user needs to acquire the result of the assigned task, use the [asynchronous support API ](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api) provided by GameAnvil.



## 3. Fiber

Fiber is a sort of lightweight user thread and the basic flow unit of the GameAnvil server code. The code flow of the single thread of the node explained earlier is further divided by several Fiber in order to effectively process a large number of sessions, users, and rooms at the same time. In other words, GameAnvil supports the Fiber-based Continuation.

Several Fibers are scheduled on the thread pool (Executor). At this time, if the size of thread pool is fixed to 1, it directly becomes the model for the GameAnvil nodes. In order words, the node is an annotation scheduler that is used to process multiple Fibers. The picture below depicts this.

![image.png](http://static.toastoven.net/prod_gameanvil/images/FiberConcept.png)

If Fibers are used like this, code can be written sequentially. Server code becomes similar to writing common blocking code. Users do not have to concern with separate callback handling or completion notifications. In addition to this, GameAnvil users do not have to concern with this Fiber unit. As GameAnvil engine takes care of all Fibers, users can develop projects as if they are writing single threading code.

![image.png](http://static.toastoven.net/prod_gameanvil/images/FiberConcept2.png)

The GameAnvil server code is based on asynchronous process. For this, it provides [asynchronous support API ](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api). If blocking is called on an arbitrary Fiber using asynchronous API, the status of only the corresponding Fiber becomes suspend. This will be explained in detail a later section.



### **[Caution]**

Below is a couple of things to be avoided when writing Fiber-based code. Users do not have to concern with the Fiber unit, the things below must be avoided.

- **Suspend** means that Fiber code yielded code execution to another Fiber and enter into the waiting status. This can be expressed as Fiber blocking.
- Methods that can be Suspended must have the **throws SuspendExecution** exception signature added to them. This is a promise with the engine that the method contains blocking code used to stop Fiber.

```
void someSuspendableMethod() throws SuspendExecution {

// some fiber-blocking call can be here

}
```

- If the throws SuspendExecution exception signature cannot be specified for a special reason, **@Suspendable** annotation can be used. In all other cases, users must use the throws SuspendExecution exception signature.

```
@Suspendable
void someSuspendableMethod() {

// some blocking call can be here

}
```

- Naturally, the methods that call these Suspendable methods also become Suspendable. For example, if there is the someCaller() method that calls someSuspendableMethod(), someCaller() must be declared as Suspendable.

```
void someCaller() throws SuspendExecution {

    someSuspendableMethod();

}
```

- SuspendExecution is a special technique that is used for the Quasar library used by the engine to process Fiber. This is not an **actual exception** and as Fiber is not yet Java standard, it can be considered as some kind of detour technique used by the Quasar library. **Therefore, this SuspendExecution exception should not be caught!** Although this is a common error, It is very important part.

```
// !! Wrong usage !!

void someCaller() {

    try {

        someSuspendableMethod();

    } catch (SuspendExecution e) {
        // Never explicitly catch SuspendExecution.!
    }
}
```

- However, catching entire exceptions is no problem. It is handled automatically.

```
void someCaller() throws SuspendExeuction {

    try {

        someSuspendableMethod();

    } catch (Exception e) {
        // No problem
    }
}
```



## 4. Asynchronous process based on Fibers

Code must be able to processed asynchronously on Fiber. In order words, When Fiber calls I/O that takes an arbitrary time, that Fiber may delegate its execution permission to another Fiber until the call is complete. Even thread blocking calls provide API that can convert this into a Fiber blocking call and desynchronize. That means a thread blocking call must be processed using [asynchronous support API ](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api). If it is directly called, the corresponding thread as well as all the Fibers on the thread are blocked and cause all the nodes to stop.

```
    void someProblematicMethod() {

        someThreadBlockingCall(); // the corresponding Fiber, and the entire thread are blocked.!

    }
```

This will be covered in detail in the asynchronous support of the implementation method](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api)에서 자세히 다룹니다.



## 5. Distribution server

In the previous basic concepts, we showed the picture of the GameAnvil node configuration. In order words, a single process can freely configure many different nodes and run them. However, all GameAnvil processes must include a single IPC (Inter-Process Communication) node. This IPC node is responsible for the communication between GameAnvil processes. There are in fact low level nodes in charge of network processing, users may understand the whole thing as IPC node.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)

### 5-1. Communication between nodes

The picture below depicts two or more GameAnvil processes communicating each other through these IPC nodes. In the picture, two different GameAnvil processes run the nodes in different configurations. At this point, each node can communicate each other.

In order to communicate with many different process nodes, each node passes messages through IPC nodes. On the other hand, the nodes on the same process use queue to communicate with each other. Therefore, in this case, the communication does not go through IPC nodes.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/IPC.png)

### 5-2. Meetpoint

Then, how processes interconnect with each other for this IPC? The answer is Meetpoint. GameAnvil can set Meetpoint address. This looks like below and one or more IP address pairs can be set. The GameAnvil process tries connecting to one of the Meetpoint addresses set on its initial run and synchronize the entire server family information.

```
"common": {

    "meetEndPoints": [
      "10.1.2.1:16000",
      "10.1.2.2:16000",
    ],

}
```



## 6. Connection and session

The client creates connection using GATEWAY node. Through this connection, authentication and login can be processed based on the account and user information. Once the login is complete, a user object is created in an arbitrary Game node. It means a logic session is created between Gateway node and the corresponding Game node. Once the connection and session creation is complete, the user can play the game.

### 6-1. Connection Recovery

If the connection between the client and Gateway node is severed, Connection Recovery is executed as in the picture below. While reconnecting, the client can try connecting to one of the several Gateway nodes that hasn't been connected before. In this case, the session is newly recovered based on the location information on the Game node that has a user object. Therefore, users can maintain their game progression even when they are reconnected while playing.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

### 6-2. Location Node

Location node can be seen from the previous Connection Recovery picture. This Location node is used for GameAnvil to internally manage the location of users and rooms. Users cannot directly implement or use Location node. However, the overall GameAnvil system flow can be understood only when the existence of the Location node that is used to manage location information is known, so it will be briefly mentioned.

The previous Connection Recovery for example, the location information on the related session and user is stored in Location node while the client is initially connected and trying to connect to Game node. Therefore, when reconnecting, the location information stored here during the previous connection process can be looked up. This location information is used in important tasks such as looking up the location information on users or rooms and delivering messages based on these information.



## 7. Key library

The following are the four key libraries used by GameAnvil. As Quasar, ZeroMQ, and Netty are internally used by the engine, GameAnvil users are unlikely to use these libraries directly. Protocol Buffers is used during parallelize/serialize messages. Regardless of whether they are directly used, understanding the following four libraries can be helpful when using the engine.

| Library       | Purpose                           |
| ---------------- | ------------------------------ |
| Quasar           | Supports Fiber-based Continuation |
| ZeroMQ           | Server's IPC                     |
| Netty            | Communication between server and client           |
| Protocol Buffers | Parallelization of messages between server and client  |



## 8. ByteCode Instrumentation

GameAnvil is a Fiber-based server engine. For this, GameAnvil uses Quasar library. To process Fiber-based de-synchronization, a predefined special exception is used. Alternatively, @Suspendable annotation can be used.

```
throws SuspendExecution
```

To interpret the predefined code, GameAnvil server code must use Quasar library to process ByteCode Instrumentation. ByteCode Instrumentation can be processed by using one of the following two methods.

### 8-1. Runtime Instrumentation

Add Quasar binary as a javaagent in front of the server run VM option. If this is done, proceed with ByteCode Instrumentation at run-time.

```
-javaagent:MY_PATH\quasar-core-0.7.10-jdk8.jar=bm
```

### Note

This item must be added in front of the VM option. At this time, set the quasar-core path to the one to which quasar-core is copied.

### 8-2. AOT Instrumentation

If AOT (ahead-of-time) Instrumentation needs to be processed, add the following content to the project object management file (pom.xml) and package, install, or deploy the server binary through Maven. Process Instrumentation when compile is finished. In this case, like the first case, the VM option does not require javaagent.

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>

    <executions>
        <execution>
            <id>instrument-classes</id>
            <phase>compile</phase>

            <configuration>
                <tasks>
                    <taskdef name="instrumentationTask" classname="co.paralleluniverse.fibers.instrument.InstrumentationTask" classpathref="maven.dependency.classpath"/>
                    <instrumentationTask>
                        <fileset dir="${project.build.directory}/classes/" includes="**/*.class"/>
                    </instrumentationTask>
                </tasks>
            </configuration>

            <goals>
                <goal>run</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
