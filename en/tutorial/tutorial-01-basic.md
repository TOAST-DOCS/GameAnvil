## Game > GameAnvil > Basic Tutorial

### How to create multiplayer games easily with GameAnvil

GameAnvil is a real-time multiplayer game server creation platform. 
GameAnvil makes it easy to develop and operate game servers and clients. 

This document covers the development of a multiplayer sync game that can be played in real life using GameAnvil's basic features. 
Instead of simply listing the server's concepts and APIs, we have developed our own multiplayer game servers and sample clients so that you can naturally learn GameAnvil's basic concepts and project configuration and implementation methods.

Not only does GameAnvil provide server engines, but it also provides connectors that help connect clients to servers. As you complete a sample that allows to see servers and clients interact, you can get used to the overall flow of developing games using GameAnvil.

## Prepare a hands-on experience environment - Server Project

To create a multiplayer game, you need a server program that corresponds to the client. After building the game server, tutorials are conducted by the way implementing the client. 

In this example, we use Unity and GameAnvil connectors to create client programs, and use the server engine GameAnvil that we introduced prior to creating server programs. First, let's create a server program project with GameAnvil.

Follow these steps to download the final server sample project from the link below. You can download the project and refer to it to see in advance what it will be like to implement the server features in multiple stages from the initial template.

[Download server sample project](https://static.toastoven.net/prod_gameanvil/files/tutorial/basic-tutorial/GameAnvilServerTutorial_1213.zip?disposition=attachment)

### Project Configuration

In this chapter, we aim to complete the initial setup to begin development. Running the actual process and running the server will be covered in the next chapter.

The example uses the server project IDE as JetBrain's IntelliJ. The version of IntelliJ used in the example is IDEA Ultimate 2023.1.2. If you have not purchased a license, you may use the IntelliJ IDEA Community Edition. It is expected to work fine with other versions of IntelliJ, but we recommend that you proceed with the same view as the sample run version because we have not tested every case.

To apply GameAnvil to your project, you must download the GameAnvil library to Maven repository and create a setup file that is essential to running GameAnvil. Finally, when you write a little boilerplate code, the initial setup of the development is complete.

GameAnvil offers IntelliJ templates that replace this set of processes, making it easier to complete initial tasks. You can download the project file template for IntelliJ from the following link. Do not decompress the templates you download.

[Template download](https://static.toastoven.net/prod_gameanvil/files/tutorial/basic-tutorial/GameAnvilTemplate.zip?disposition=attachment)

Run IntelliJ to apply the downloaded template. Select **Customize** from the left menu on the **Welcome to IntelliJ IDEA** screen and click **Import Settings...**. Or search **Import Settings...** in the search bar.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/import_gameanvil_template.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/search_import_settings.png)

<br>

In Finder or File Explorer window, navigate to the path from which you downloaded the template and select the compressed file. When the **Select Components to Import** window opens, check and select both **File templates** and **Project templates**. Click **OK** and **Import and Restart** to restart IntelliJ and complete the template application.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/select_import.png)

Click **New Project** in the group of buttons in the upper right corner of IntelliJ, scroll through the list to the left, and select **GameAnvilTemplate** in the bottom under **Templates**. Name the project **SynchronizeTutorial**. The name must not contain spaces. Create the project after checking the project location and base package name.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/imported_gameanvil_template.png)

The server project frame is now configured in IntelliJ. You can see that the code and configuration files have been created by looking at the Project panel.

* Main: class that contains the entry point Main function of the program.
* Protocol package: a package containing a protocol buffer file compiled with java.
* Proto Package: a protocol file created using the Google Protobuf library.
* build.sh /build.bat: an executable file that compiles a protocol file into java to create a protocol buffer file.
* GameAnvilConfig.json: A file that records the server configuration information required to drive GameAnvil. You can modify it to fit your server implementation.
* logback.xml: A file used to configure logging in Java projects. As a configuration file for the Logback framework, you specify how the logging system works, the format of the log, where it is stored, and so on. You can use this file to set the logging level, log format, path and name of the log file, and log rolling policy.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_project_view_init_1213.png)

## Modify GameAnvil Server Settings Files

You can change the settings of the GameAnvil server through the GameAnvilConfig.json file found under the resources package in the Project panel.

* common: part that covers the overall server setup
* location: part that addresses location node-related settings
* match: part that deals with match node related settings
* gateway: part that deals with gateway node related settings
* game: part that deals with game node related settings

Because you have configured the project through the template, you can see that the GameAnvilConfig.json file has the default setting information required for server operation. In this example, there are three main points to watch carefully.

1. The nodeCnt value of the game
2. The serviceName value of the game
3. The channelIDs value of the game

Game nodes can be run in multiple VMs, depending on the amount you need, or depending on the server's performance. Once you set how many game nodes you want to run, it automatically read the configuration file at the time of the server's execution and display a set number of nodes. The template setting is set to float one game node, and you can use it as it is. In addition, the parts to be modified are the serviceName and channelIDs in the game part.

If you look at the last part of the game side of GameAnvilConfig.json file, there is a part marked as, Todo. Let's modify it and set up the service name and channel information.

```json
  "game": [ 
    { 
      "nodeCnt": 1, 
      "serviceId": 1, 
      "serviceName": "Todo - Input My Service Name", 
      "channelIDs": ["ToDo - Input My ChannelName","ToDo - Input My ChannelName"], // Channel ID to be assigned to each node. (It does not have to be unique.  "" means it doesn’t use channel) 
      "userTimeout": 5000 // Set how long to manage user objects without removing them from the server after the client is disconnected    
    } 
  ]
```

### About Service

When a server provides multiple games, a service is a separate name for each game service. A service name is a promised string between the server and the client that represents a specific service. It will be used to enter the service name in the subsequent process, so please remember this.

Here, we use a service named Sync. In the game part, modify the serviceName as follows.

```
"serviceName" : "Sync",
```

### About channel

A channel is one of the ways to logically divide a single server group. We will skip the detailed description in this document because we do not use channels in this example. As we do not use channels, modify the channelIDs in the game as follows.

```
"channelIDs" : [""],
```

The contents of the GameAnvil server configuration file that you created in this way are as follows.

```json
"game": [ 
    { 
      "nodeCnt": 1, 
      "serviceId": 1, 
      "serviceName": "Sync", 
      "channelIDs": [""], // Channel ID to be assigned to each node. (It does not have to be unique.  "" means it doesn’t use channel) 
      "userTimeout": 5000 // Set how long to manage user objects without removing them from the server after the client is disconnected    
   } 
  ]
```

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_config_json_1213.png)

For your information, if you look at the gateway setting, you can see that the TCP_SOCKET connection is set to use the 18200 ports. This is the port that connects to the client, which will then be used by the client project to fill in the server connection information.

## Run GameAnvil Server

### Setting the Java Version

GameAnvil supports Java 8 and 11 versions. Some settings may vary depending on the version and we used Java 11 versions here.

First, let's check the jdk settings. Select **File>ProjectStructure** from the top left menu to open the **ProjectStructure** window. For Mac users, the **command+;** shortcuts are available. 

On the **Project** tab, check the SDK settings. If there is no SDK, download and set the desired version of JDK through **Add SDK>Download JDK**. Set **Language level** to **SDK default**. Next, on the **Modules** tab, set **Language level** to **Project default**.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/project_structure_1213.png)

From the **Settings** menu, check the JVM used by **gradle**. Set it to the same version as **Project****SDK**.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gradle_sdk_config_1213.png)

### Run Server

When the execution setting is complete, click on the green triangle icon to the left of the main() function of the main class to select Run Main. Main(). Once this has been done, clicking the green triangle Run icon in the upper right corner of the IntelliJ will also launch the server.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run1_1213.png)

In build.gradle, JVM options are pre-configured for your convenience. To leverage these settings to run the server, right-click on Task > others > runMain in the Gradle window of IntelliJ and click on Run GameAnvilTutorial.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run2_1213.png)

If the server is running normally, a number of logs related to the server's driving status will be output. 

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run_log.png)

GameAnvil servers are made up of multiple nodes. These nodes divide the functions that the server will perform into multiple roles. We've only checked the initial drive of the server, but we're not fully prepared because we haven't written the code for the node or other server to run.

Each node needs time to prepare to execute code, and when each node is ready, it outputs an onReady log. The node that the client plays a direct role in connecting to the server is the gateway node. If the gateway node is ready and the onReady log of the gateway node is output, the GameAnvil server is always accessible.

In the next chapter, we will implement the BasicGameNode required for sample game behavior among GameAnvil's multiple nodes.

## Implement GameAnvil Server Features

### Implement Game Node

GameAnvil provides multiple node classes prefixed with `Base-`. Basic node features are already implemented inside the engine, and users can inherit these Base classes and use various callback features. In this example, we are going to create and use a game node class that inherits the BaseGameNode class.

In the Project panel, right-click on the path where the Main class is located and select **New>Package** to create a new package named **node**. Right-click the node package again and select **New>BaseGameNode**. When the Create File dialog box opens, type **SyncGameNode** in **Filename** and **Sync** in **ServiceName** and click **OK**.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/create_sync_game_node.png)

This feature is available because you applied File templates (schemes) together when you installed the template earlier. If **New>BaseGameNode** is not visible, select **New>Java Class** to create an empty class.

The automatically generated code is as follows.

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.BaseGameNode;
import com.nhn.gameanvil.node.game.data.BaseChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.BaseChannelUserInfo;
import com.nhn.gameanvil.node.game.data.ChannelUpdateType;
import com.nhn.gameanvil.packet.Payload;
import com.nhn.gameanvil.packet.message.MessageDispatcher;

@ServiceName("Sync")
public final class SyncGameNode extends BaseGameNode {

    private static final MessageDispatcher<SyncGameNode> messageDispatcher = new MessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public MessageDispatcher<SyncGameNode> getMessageDispatcher() {
        return messageDispatcher;
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
    public void onPause(Payload payload) throws SuspendExecution {

    }

    @Override
    public void onResume(Payload payload) throws SuspendExecution {

    }

    @Override
    public void onShutdown() throws SuspendExecution {

    }

    @Override
    public boolean onNonStopPatchSrcStart() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchDstStart() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchDstEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canNonStopPatchDstEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, BaseChannelUserInfo baseChannelUserInfo, int userId, String accountId) throws SuspendExecution {

    }

    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, BaseChannelRoomInfo baseChannelRoomInfo, int userId) throws SuspendExecution {

    }

    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {

    }
}
```

### About node

Every node has a state depending on whether or not a loop has started that allows something to start processing. Below are some of the states that a node can have.

* INIT
* PREPARE
* READY
* SHUTDOWN

The node reaches the READY state, starting with the INIT state and changing the state in the order described. The READY state indicates that the node is in a state capable of processing and performing a given task.

The auto-generated code contains a code that overrides a callback hooked to the state of each node. For example, if you write a particular logic to the onInit() method, that callback is inserted and called just before the node starts initializing (Init).

GameAnvil has most of the code pre-prepared, so there's no more code to write at this stage. Just use the game node as you create it.

### About User Types

The user participates in the room at each game node and sends and receives packets, which is a promised character string that distinguishes each user implementation.

To use the room-based implementation offered by GameAnvil, you need **game users** and **game room** classes in addition to the nodes implemented above. Let us explain how to easily implement it with just class inheritance and annotation attachments.

### Implement Game User

When a client logs in to a server, the server creates the client information as an object called **game user** and stores it in memory and retains it. What information the game user will express can be implemented freely as needed by the user. Game user implementation can also be implemented consistently through class inheritance and callback overriding.

In the Project panel, right-click the path where the Main class is located and select **New>Package** to create a new package named **user**. Right-click on the **user** package again and select **New>BaseUser**. When the File Creation dialog box opens, type **SyncGameUser** in **FileName**, **Sync** in **ServiceName**, and **UserType****USER_TYPE_SYNC** and click on **OK**.

The user type is a promised string between the server and the client that distinguishes each user implementation, and the user type must be used in subsequent client project implementations, so please remember it.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/new-game-user-server.png)

The automatically generated code is as follows.

```java

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.annotation.UserType;
import com.nhn.gameanvil.node.game.BaseUser;
import com.nhn.gameanvil.node.game.data.RoomMatchResult;
import com.nhn.gameanvil.packet.Payload;
import com.nhn.gameanvil.packet.message.MessageDispatcher;
import com.nhn.gameanvil.serializer.TransferPack;

@ServiceName("Sync")
@UserType("USER_TYPE_SYNC")
public final class SyncGameUser extends BaseUser {

    private static final MessageDispatcher<SyncGameUser> messageDispatcher = new MessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public MessageDispatcher<SyncGameUser> getMessageDispatcher() {
        return messageDispatcher;
    }

    @Override
    public boolean onLogin(Payload payload, Payload sessionPayload, Payload outPayload) throws SuspendExecution {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onPostLogin() throws SuspendExecution {

    }

    @Override
    public boolean onLoginByOtherDevice(String newDeviceId, Payload outPayloadForKickUser) throws SuspendExecution {
        return true;
    }

    @Override
    public boolean onLoginByOtherUserType(String userType, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public void onLoginByOtherConnection(Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean onReLogin(Payload payload, final Payload sessionPayload, Payload outPayload) throws SuspendExecution {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onDisconnect() throws SuspendExecution {
    }


    @Override
    public void onPause() throws SuspendExecution {

    }

    @Override
    public void onResume() throws SuspendExecution {

    }

    @Override
    public void onLogout(Payload payload, Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean canLogout() throws SuspendExecution {
        boolean canLogout = true;
        return canLogout;
    }

    @Override
    public void onPostLeaveRoom() throws SuspendExecution {

    }

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, Payload payload) throws SuspendExecution {
        return null;
    }

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, Payload payload, Payload outPayload) throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canTransfer() throws SuspendExecution {
        return true;
    }

    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {

    }

    @Override
    public void onTransferIn(TransferPack transferPack) throws SuspendExecution {

    }

    @Override
    public void onPostTransferIn() throws SuspendExecution {

    }

    @Override
    public boolean onCheckMoveOutChannel(String destinationChannelId, Payload payload, Payload errorPayload) throws SuspendExecution {
        boolean canMoveOut = false;
        return canMoveOut;
    }

    @Override
    public void onMoveOutChannel(String destinationChannelId, Payload outPayload) throws SuspendExecution {
    }

    @Override
    public void onPostMoveOutChannel() throws SuspendExecution {
    }

    @Override
    public void onMoveInChannel(String sourceChannelId, Payload payload, Payload outPayload) throws SuspendExecution {
    }

    @Override
    public void onPostMoveInChannel() throws SuspendExecution {

    }
}

```

Game users are created by a client requesting a login to the server. The server can determine whether to allow login or not and export it as a return value through a payload transmitted from the client. Only the engine user creates the main logic and the engine is responsible for the success or failure of login.

In this tutorial, to allow logins without any special verification process, the onLogin function always returns true. This will always create a user object and give a success response when a client requests login.

### Implement Game Room

If you successfully access the game node as a game user, you can now send and receive packets to and from other users through the game room. A game room is a group that logically binds users who send and receive packets. Game room implementations can also be implemented through class inheritance and callback overriding.

In the Project panel, right-click on the path where the Main class is located and select **New>Package** to create a new package named **room**. Right-click on the **room** package again and select **New>BaseRoom**. When the File Creation dialog box opens, type **SyncGameRoom** in **FileName**, **Sync** in **ServiceName**, **RoomType** in **ROOM_TYPE_SYNC** and **UserClass** in **SyncGameUser** and click on **OK**.

The room type is a promised string between the server and the client that distinguishes each room implementation, and there are parts where the room type must be entered when implementing the client project, so remember to enter it.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/new_game_room_server.png)

The automatically generated code is as follows.

```java

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.BaseRoom;
import com.nhn.gameanvil.node.game.RoomMessageDispatcher;
import com.nhn.gameanvil.packet.Payload;
import com.nhn.gameanvil.serializer.TransferPack;

import java.util.List;

@ServiceName("Sync")
@RoomType("ROOM_TYPE_SYNC")
public final class SyncGameRoom extends BaseRoom<SyncGameUser> {

    private static final RoomMessageDispatcher<SyncGameRoom, SyncGameUser> messageDispatcher = new RoomMessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public RoomMessageDispatcher<SyncGameRoom, SyncGameUser> getMessageDispatcher() {
        return messageDispatcher;
    }

    @Override
    public void onInit() throws SuspendExecution {
    }

    @Override
    public void onDestroy() throws SuspendExecution {
    }

    @Override
    public boolean onCreateRoom(SyncGameUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public boolean onJoinRoom(SyncGameUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public boolean onLeaveRoom(SyncGameUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public void onLeaveRoom(SyncGameUser sampleUser) throws SuspendExecution {

    }

    @Override
    public void onPostLeaveRoom() throws SuspendExecution {

    }

    @Override
    public void onRejoinRoom(SyncGameUser user, Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean canTransfer() throws SuspendExecution {
        return true;
    }

    @Override
    public void onTransferOut(TransferPack transferPack) throws SuspendExecution {

    }

    @Override
    public void onTransferIn(List<SyncGameUser> userList, TransferPack transferPack) throws SuspendExecution {
    }

    @Override
    public void onPause() throws SuspendExecution {

    }

    @Override
    public void onResume() throws SuspendExecution {

    }
}
```

Game rooms are created when a game user requests a server to create a room. On the client side, you can simply create a room and enter an existing room with a method call. If you want to insert a custom code at the time the user enters the room or the room is created, you can easily insert the code by overriding the appropriate callback.

## Wrap up the server implementation

So far, you have completed building the server for running the basic tutorial sample. When you run the server again, you will see the phrase `{"message:" "All nodes are ready!!"}` in the log. This log appears, which means that the GameAnvil server was running successfully.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/all_nodes_are_ready.png)

The server is now ready to accept the client's request. The next step is to implement the client using the GameAnvil connector and the Unity sample project.

## Prepare a hands-on experience environment - Client Project

You can download the final client sample project that you'll complete the fix by following the steps below. If you want to take several steps from the initial Unity project you downloaded and configured to implement the client feature you can download it and refer to it if you want to see in advance what it's going to be like.

[Download final client sample project](https://static.toastoven.net/prod_gameanvil/files/tutorial/basic-tutorial/BasicSyncTutorial.zip?disposition=attachment)

### Download GameAnvilConnector

Download the file below for the use of GameAnvil connector dll.

[GameAnvil-Connector.unitypackage](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector.unitypackage)

### Download Unity Package

Download the Unity package from the link below to practice GameAnvil connector.

[GameAnvil-Tutorial-Sample.unitypackage](https://static.toastoven.net/prod_gameanvil/files/tutorial/basic-tutorial/GameAnvil-Tutorial-Sapmple.unitypackage)

### Create Unity Project

Run the Unity Hub and click the New Project button at the top right. The version of the Unity Hub is irrelevant.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/unity-hub.png)

Select **2D** as the template, verify the project name and storage location, and click **Create project**. The Unity version used in this example is 2020.3.37f1, and we recommend that you do it in the same environment as the sample run version because we have not tested every case, although it is acceptable to use a different version during the practice.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/new-unity-project.png)

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/new-unity-project-done.png)

### Import GameAnvilConnector and Unity Package

Right-click the project view, select **Import Package > Custom Package...** and select the Unity package that you downloaded in the previous step when the Finder or File Explorer opens. Run the import in the sequence of GameAnvilConnector and Tutorial-Sample.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/import-unity-package-connector.png)

Open the IntroScene in the Scene folder in the GameAnvilSample folder to see the screen below.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/unity-after-import-package.png)

In **File>Build Settings**, click **Add OpenScene** to set it to be included when building.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/intro_scene_to_build_settings.png)

## Add GameAnvilConnector

Right-click in the Hierarchy view and click **GameAnvil>GameAnvilConnector**. The GameAnvilConnector game object is created and you can modify the settings on the Inspector of the GameAnvilConnector game object as follows.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/add-gameanvilconnector-done.png)

* QuickConnect: displays the quick connection progress.
* GameAnvil Connector Configuration: a bundle of connector-related settings.
* Connect Configuration: Can modify quick connection information.
* Authentication Configuration—can modify quick connection credentials.
* Login Configuration—can modify the quick connection login information.
* LogListener: manage log output from inside the GameAnvil connector.

For now, it's okay if you don't know the details of the settings. You can check the description of each item as you go through the tutorial.

## Quick Connect

The GameAnvil client must go through three steps to connect to the GameAnvil server: Connect, Authentication, and Login.

* Connect: create and connect sockets to communicate between the server and the client.
* Auth: the server determines whether to allow the client to send and receive data through the server.
* Login: create an object that represents the client's information in the server's memory, i.e. a game user.

Each step is sequentially performed, and if the previous step does not complete successfully, the next step cannot proceed. The success of each step can be determined by the parameters passed to the callback.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/connect-auth-login.gif)

Here, we open the QuickConnectUIMANAGER script, which is added as a component to the Canvas game object on the Hierarchy view, in the source code editor to add implementations, and practice each course directly.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/quickconnect_ui_manager.png)

### Configure Connect-Related Fields

Note down the server information to connect to. If you are floating the server directly locally, the ip used is `127.0.1`. The port used is `18200`, which is the default port of the gateway node. You can use the default values of GameAnvilConnector without having to set them separately. You can see that the ip and port information has a code that connects to Unity's InputField so that you can modify it in play mode if necessary.

```c#
void Start() 
{ 
    ipInputField.text = GameAnvilConnector.getInstance().ip; 
    portInputField.text = GameAnvilConnector.getInstance().port.ToString(); 
 
    ...omit... 
 
    ipInputField.onValueChanged.AddListener(delegate { ipChanged(); }); 
    portInputField.onValueChanged.AddListener(delegate { portChanged(); }); 
} 
 
void ipChanged() 
{ 
    GameAnvilConnector.getInstance().ip = ipInputField.text; 
} 
 
void portChanged() 
{ 
    if (!int.TryParse(portInputField.text, out GameAnvilConnector.getInstance().port)) 
    { 
        GameAnvilConnector.getInstance().port = 11200; 
    } 
}
```

### Set up Authentication related Fields

Fill in the information required for authentication. There are three types of information required for authentication: accountId, deviceId and password. Currently, the server is implemented to pass the authentication stage unconditionally, so there will be no problem with setting it to any value. You can use the default value of the GameAnvilConnector, `test` and, if necessary, you can see that the code is written so that the value entered through Unity's InputField is available in play mode.

```c#
void Start()
{ {
    ...omit...
    
    accountIdInputField.text = GameAnvilConnector.getInstance().accountId;
    deviceIdInputField.text = GameAnvilConnector.getInstance().deviceId;
    passwordInputField.text = GameAnvilConnector.getInstance().password;

    ...omit...

    accountIdInputField.onValueChanged.AddListener(delegate { accountIdChanged(); });
    deviceIdInputField.onValueChanged.AddListener(delegate { deviceIdChanged(); });
    passwordInputField.onValueChanged.AddListener(delegate { passwordChanged(); });

    ...omit...
}

void accountIdChanged()
{
    GameAnvilConnector.getInstance().accountId = accountIdInputField.text;
}

} void deviceIdChanged()
{ GameAnvilConnector.getInstance()
    GameAnvilConnector.getInstance().deviceId = deviceIdInputField.text;
}

} void passwordChanged()
{ GameAnvilConnector.getInstance()
    GameAnvilConnector.getInstance().password = passwordInputField.text;
}
```

### Set up Login related field

Fill in the information you need to log in. The information you need to log in includes user type, channel ID, and service name. You must use the user type and service name that you created when you implemented the server. By default, write to apply the values that you specified on the server when it runs. It is set up to modify the values through Unity's InputField, if necessary.

```c#
void Start()
{ {
    ...omit...

    userTypeInputField.text = GameAnvilConnector.getInstance().userType;
    channelIdInputField.text = GameAnvilConnector.getInstance().channelId;
    serviceNameInputField.text = GameAnvilConnector.getInstance().serviceName;

    ...omit...

    userTypeInputField.onValueChanged.AddListener(delegate { userTypeChanged(); });
    channelIdInputField.onValueChanged.AddListener(delegate { channelIdChanged(); });
    serviceNameInputField.onValueChanged.AddListener(delegate { serviceNameChanged(); });

    ...omit...
}

void userTypeChanged()
{
    GameAnvilConnector.getInstance().userType = userTypeInputField.text;
}

} void channelIdChanged()
{ channelIdChanged
    GameAnvilConnector.getInstance().channelId = channelIdInputField.text;
}

void serviceNameChanged()
{ GameAnvilConnector.getInstance()
    GameAnvilConnector.getInstance().serviceName = serviceNameInputField.text;
}
```

### Quick connection request API call

Quick connection requests can be made through GameAnvilConnector as follows.

```c#
GameAnvilConnector.getInstance().QuickConnect(DelOnQuickConnect);
```

After a quick connection, you can find out the results through the agent you delivered upon request.

```c#
void DelOnQuickConnect(GameAnvilConnector.ResultCodeQuickConnect resultCode, UserAgent userAgent, GameAnvilConnector.QuickConnectResult quickConnectResult)
{
    Debug.Log(resultCode);
}
```

To make quick connection requests by clicking a button, we implement the QuickConnect method as below. In the example, since the contents are written by default, just disable annotation.

```c#
public void QuickConnect()
{
    GameAnvilConnector.getInstance().QuickConnect(DelOnQuickConnect);
    state = UIState.QUICK_CONNECTING;
}
```

After a quick connection, modify the DelOnQuickConnect method as shown below to switch the UI state. In the example, since the contents are written by default, just disable annotation.

```c#
void DelOnQuickConnect(GameAnvilConnector.ResultCodeQuickConnect resultCode, UserAgent userAgent, GameAnvilConnector.QuickConnectResult quickConnectResult)
{
    if (quickConnectResult.resultCodeQuickConnect.Equals(GameAnvilConnector.ResultCodeQuickConnect.QUICK_CONNECT_SUCCESS))
    {
        state = UIState.QUICK_CONNECT_COMPLETE;
    }
    else
    {
        state = UIState.QUICK_CONNECT_FAIL;
    }
}
```

### Quick Connection Shutdown API call

Quick connection shutdown can also be requested as below, just like the connection request API.

```c#
GameAnvilConnector.getInstance().QuickDisconnect();
```

To make quick connection shutdown requests by clicking a button, we implement the QuickDisconnect method as below. In the example, since the contents are written by default, just disable annotation.

```c#
public void QuickDisconnect()
{
    GameAnvilConnector.getInstance().QuickDisconnect();
    state = UIState.NOT_QUICK_CONNECTED;
}
```

### Quick connection progress output

Quick connection progress can be read as follows.

```c#
GameAnvilConnector.getInstance().GetQuickConnectState().ToString();
```

The code displayed on the screen is written in the Update function as shown below so that you can always know the progress of the quick connection.

```c#
void Update()
{
    quickConnectResultText.text = GameAnvilConnector.getInstance().GetQuickConnectState().ToString();
}
```

### Enter the value to be used for the quick connection directly into the GameAnvilConnector

The same service name or user type string value used in the server implementation phase must be used on the client. 
In the Inspector window of the GameAnvilConnector, in the Login Configuration, set the User Type and Service Name to the same value as the server, as shown below.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil-login-configuration.png)

### Quick Connection Request/Connection Shutdown Test

Verify that the server is running and enter play mode in Unity Editor. Click **QuickConnect** to verify that the connection is successful.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/quick-connect.gif)

If you attempt a quick connection, Connect, Authenticate and Login will proceed in the following order in the Quick Connection Status window.

* NOT_CONNECTED
* CONNECT_IN_PROGRESS
* CONNECT_COMPLETE
* AUTHENTICATE_IN_PROGRESS
* AUTHENTICATE_COMPLETE
* LOGIN_IN_PROGRESS
* LOGIN_COMPLETE
* READY 

If the connection is complete, it will be shown as READY. Press the End Connection button in this state to verify that the connection is successfully shutdown.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/quick-disconnect.gif)

## Create and enter a game room

### Use Game Room Creation Request API

By calling the GameAnvil connector's room creation request API, the client can easily ask the server to create a game room. When calling the game room creation request method, you have to hand over the room type as a parameter, and you have to pass the room time value that was agreed with the server in advance.

```c#
GameAnvilConnector.getInstance().getUserAgent().CreateRoom("ROOM_TYPE_SYNC", DelOnCreateRoom);
```

You can receive the result of the room creation request through the agent you delivered together. You can also receive the generated game room information (room ID, room name, etc.).

```c#
public void DelOnCreateRoom(UserAgent userAgent, ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) {
    Debug.Log(result);
}
```

To make room creation requests by clicking a button, we implement the CreateRoom method as below. The selected room type is enabled by drop-down. In the example, since the contents are written by default, just disable annotation.

```c#
public void CreateRoom()
{ 
    GameAnvilConnector.getInstance().getUserAgent().CreateRoom(roomTypeDropdown.options[roomTypeDropdown.value].text, DelOnCreateRoom);
}
```

You can also modify the function that receives the result of the room creation request to change the UI status. Set the ID of the created game room to display on the screen. In the example, since the contents are written by default, just disable annotation.

```c#
public void DelOnCreateRoom(UserAgent userAgent, ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) {
    Debug.Log(result);
    if (result.Equals(GameAnvil.Defines.ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)){
        state = UIState.JOIN_ROOM_COMPLETE;
        RoomIdText.text =  roomId.ToString();
    }
    else
    {
        state = UIState.JOIN_ROOM_FAIL;
    }
}
```

The game room creation function is completed. We will postpone the test for a while, and implement the game room entry function first.

### Use the Game Room Entry Request API

Suppose a game room was created on the server. To access the room, you can call the game room entry request method from the GameAnvil connector. At this time, the game room ID delivered at the time of room creation is delivered.

```c#
GameAnvilConnector.getInstance().getUserAgent().JoinRoom("ROOM_TYPE_SYNC", {Enter room ID});
```

Check whether the user is currently in the room using the IsJoinedRoom() method as shown below.

```c#
GameAnvilConnector.getInstance().getUserAgent().IsJoinedRoom()
```

To make JoinRoom requests by clicking a button, we add the below implementation to the existing JoinRoom method. The room ID was made to use the value entered in the input field. In the example, since the contents are written by default, just disable annotation.

```c#
public void JoinRoom()
{
    int roomId = 0;
    int.TryParse(joinRoomIdInputField.text, out roomId);
    GameAnvilConnector.getInstance().getUserAgent().JoinRoom(roomTypeDropdown.options[roomTypeDropdown.value].text, roomId);
}
```

Now, both the game room creation function and the entrance function are complete.

### Game room test

In the Unity editor, press the shortcut `CMD + b` or `Ctrl + b` to build. In the window from the build results, click the button to confirm that a game room is created. Once a game room is created, the ID of the game room will be displayed on the screen.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/create-room.gif)

After that, enter play mode in Unity Editor. Enter the ID of the previously displayed game room in the entry field, and click **Join Room** to see if you are participating in the game room.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/join-room.gif)

## Introduction to Synchronization Controllers

You can now send and receive packets between game users who are in the same game room. These packets allow you to write code to synchronize the information you need between client processes. In a simpler way, synchronization can be achieved simply by attaching a synchronization component to the game object you want to synchronize.

### Add Synchronization Controllers

Right-click in the Hierarchy view and select **GameAnvil>SyncController**.

As scene is moved in this example, in order to manually create a synchronization object after the scene movement, uncheck Instant Sync Object Immediatly on the inspector of the Sync Controller object.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/add-syn-controller-done.png)

All of GameAnvil's synchronization features are now available. Here's a look at how to attach and use synchronization components with the simplest example.

### Create synchronization objects

In the Project view, navigate to the inside of the Resources folder and double-click Anvil prefab to switch to the prefab modification screen. In the Inspector window, click the AddComponent button and click `GameAnvil > GameAnvil Sync > TransformSync` so that the prefab is ready to synchronize the Transform information of the game object between the game users.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/anvil-add-sync-component-done.png)

To use the finished sync game object prefab in Unity Play mode, you can create a game object through the Game Object Creation API provided by Sync Controller and add it to the scene. You must forward the prefab name as the first factor.

Note that synchronization game objects with synchronization components provided by GameAnvil must be synchronized using Sync Controller's Instantiate API provided by GameAnvil rather than the GameObject.Instantate() method provided by Unity.

```c#
SyncController.Instance.Instantiate("Anvil", new Vector3(0, 0, 0), Quaternion.identity);
```

### Create a Game Scene

For specific usage examples, open the SpawnAnvil scene in the Project **View > GameAnvilSample>Scene** folder. Then, in the **File>Build Settings** menu, click **Add OpenScene** to be included when building.

We will add the implementation by modifying the SpawnAnvilSample script of the component assigned to the SpawnAnvilSample game object. In the example, since the contents are written by default, just disable annotation.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace GameAnvil
{
    public class SpawnAnvilSample : MonoBehaviour
    {
        void Start()
        {
            SyncController.Instance.InstantiateSyncObject();
        }

        void Update()
        {
            if (Input.GetMouseButtonDown(0))
            {
                Vector3 mousePos = Input.mousePosition;
                mousePos.z = Camera.main.nearClipPlane;
                Vector3 worldPosition = Camera.main.ScreenToWorldPoint(mousePos);

                SyncController.Instance.Instantiate("Anvil", worldPosition, Quaternion.identity);
            }

            if (GameAnvilConnector.getInstance() == null || GameAnvilConnector.getInstance().getUserAgent() == null || !GameAnvilConnector.getInstance().getUserAgent().IsJoinedRoom())
            {
                SceneManager.LoadScene("IntroScene");
            }
        }
    }

}
```

The Start function runs InstantSyncObject() to start synchronization immediately after scene movement.

In the Update function, each click was made to obtain the mouse's coordinates and create a prefab that was modified in the previous step. In order to be able to reconnect in case of disconnection, we moved to the original scene. 

### Synchronization test

In Unity Editor, press the `CMD + b` or `Ctrl + b` shortcuts to build. Create a game room in the window that results from the build and press the Spawn Anvil button to move the scene from the IntroScene to the SpawnAnvil Scene.

Afterwards, enter play mode in Unity Editor. After accessing the same game room as the build result, press the Spawn Anvil button to move the scene from the IntroScene to the SpawnAnvil Scene. After that, click anywhere on the screen to create a new sync game object. Once you create a game object on one client, make sure it appears the same on the screen of another client entering the same room.

<video src="https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/synchronize-create-done.mov" controls="controls" autoplay style="max-width: 640px;
display: block;
margin: auto;">
</video>

## Advanced synchronization controller

In the previous step, we checked object creation synchronization. Here we deal with a more complex example, Rigidbody synchronization of game objects. Follow the same steps as in the previous example to complete the implementation.

### Create synchronization objects

In the Project view, navigate to the inside of the Resources folder and double-click Player Prefab to switch to the Prefab modification screen. In the Inspector window, click **AddComponent** and click **GameAnvil > GameAnvil Sync > TransformSync**. You are ready to synchronize Transform information for game objects between game users in that prefab.

Double-click **Resources> Player** in the Project view to switch to the prefab modification screen. In the Inspector window, click **AddComponent** and click **GameAnvil Sync >GameAnvil Sync > RigidBodySync**. You are ready to synchronize RigidBody information for game objects between game users in that prefab.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/add-rigidbody-sync-done.png)

### Create a Game Scene

In the Project view, in the Scene folder inside the GameAnvilSample folder, open the KeyboardToMove scene, and in the **File>Build Settings** menu, click on **Add OpenScene** to set it to be included at the time of building.

We will add the implementation by modifying the KeyboardToMoveSample script of the component assigned to the KeyboardToMoveSample game object. In the example, since the contents are written by default, just disable annotation.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Animations;
using GameAnvil;
using UnityEngine.SceneManagement;

public class KeyboardToMoveSample : MonoBehaviour
{
    private static KeyboardToMove player;
    private float moveForce = 10;
    public Transform playerPointer;

    // Start is called before the first frame update
    void Start()
    {
        SyncController.Instance.InstantiateSyncObject();
    }

    // Update is called once per frame
    void Update()
    {
        if (GameAnvilConnector.getInstance() == null || GameAnvilConnector.getInstance().getUserAgent() == null || !GameAnvilConnector.getInstance().getUserAgent().IsJoinedRoom())
        {
            SceneManager.LoadScene("IntroScene");
        }

        if (player != null)
        {
            playerPointer.position = player.transform.position;
            player.GetComponent<Rigidbody>().AddForce(Vector3.right * Input.GetAxis("Horizontal") * moveForce, ForceMode.Force);
            player.GetComponent<Rigidbody>().AddForce(Vector3.up * Input.GetAxis("Vertical") * moveForce, ForceMode.Force);
        }
    }

    public static void SpawnPlayer()
    {
        Vector3 mousePos = Input.mousePosition;
        Vector3 worldPosition = Camera.main.ScreenToWorldPoint(mousePos);
        worldPosition.z = 0;

        player = SyncController.Instance.Instantiate("player", worldPosition, Quaternion.identity).GetComponent<KeyboardToMove>();
    }

    public static void SetSelected(KeyboardToMove selected)
    {
        if ( player == selected)
        {
            player = null;
            GameObject.Destroy(selected.gameObject);
        }
        else
        {
            player = selected;
        }
    }
}
```
The Start function runs InstantSyncObject() to start synchronization immediately after scene movement.

The SpawnPlayer function creates a new Player object at the mouse location each time it is called.

The Update function checks that the room entry and login are maintained, and allows you to control the last object that you created with a keyboard. Follow the keystroke by performing the AddForce() function on the rigid body to prompt you to update the location.

### Synchronization test

In Unity Editor, press the shortcut `CMD + b` or `Ctrl + b` to build. Create a game room in the window from the build results window, and click on **KeyboardToMove** to navigate from the IntroScene to the KeyboardToMoveScene.

Afterwards, you will enter play mode in Unity Editor. After accessing the same game room as the build result, click **KeyboardToMove** to navigate from the IntroScene to KeyboardToMoveScene. After that, click anywhere on the screen to create a new sync game object and move the game object to the keyboard to see if this is reflected in the other client as well.

<video src="https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/synchronize-rigidbody-done.mov" controls="controls" autoplay style="max-width: 640px;
display: block;
margin: auto;">
</video>

## Wrap up tutorial

In this document, we learned about the convenience features of GameAnvil connectors such as simple connection and synchronization. As we introduced at the beginning of the tutorial, GameAnvil has all the features you need to create a game server and the tutorial only briefly covers some of them. You can learn more about it in the following documents.
