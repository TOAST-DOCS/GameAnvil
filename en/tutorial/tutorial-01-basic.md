<!-- machine_translated: true -->

<a id="game-gameanvil-basic-tutorial"></a>

## Game > GameAnvil > Basic Tutorial { #game-gameanvil-basic-tutorial }

<a id="create-multiplayer-games-easily-with-gameanvil"></a>
### Create Multiplayer Games Easily with GameAnvil { #create-multiplayer-games-easily-with-gameanvil }

GameAnvil is a real-time multiplayer game server creation platform. It makes it easy to develop and operate your game servers and clients.

This article covers the process of using the basic features of GameAnvil to develop a multiplayer synchronous game that can actually be played. Instead of just listing the concepts and APIs for the server, we developed a direct multiplayer game server and sample client to naturally acquire the basic concepts and project configuration and implementation methods for GameAnvil.

GameAnvil provides not only server engines, but also connectors that help connect clients to the server. Completing a sample to see how the server and client interact, you get familiar with the overall flow of using GameAnvil to develop your game.

<a id="prepare-practice-environment---server-project"></a>

## Prepare Practice Environment - Server Project { #prepare-practice-environment---server-project }

To create multiplayer games, you need a server program that corresponds to the client. After building up a game server, the tutorial proceeds in a way to implement the client.

This example uses Unity and the GameAnvil connector to create the client program, and the previously introduced server engine GameAnvil to create the server program. First, create a server program project that uses GameAnvil.

By performing the steps below, the final server sample project created can be downloaded from the link below. If you implement the server feature through several steps in the initial template, you can download the project to pre-check what structure it will have.

[Download Server Sample Project](https://static.toastoven.net/prod_gameanvil/files/v2_2/gameanvil_tutorial.zip?disposition=attachment)

<a id="configure-project"></a>
### Configure Project { #configure-project }

This chapter aims to complete the initial configuration to get started with the development. The next chapter covers running the server by actually executing the process.

In the example, the server project IDE is used as the IntelliJ of JetBrain. The version of the IntelliJ used in the example is IDEA Ultimate 2024.2.1. If you have not purchased a license, you will not be able to use IntelliJ IDEA Community Edition. While using a different version of the IntelliJ, it is expected to operate without any problems. However, since not all cases have been tested, it is recommended to proceed from the same horizon as the sample running version.

To apply GameAnvil to your project, you must download the GameAnvil Library to the Maven repository and write the setting file required to run GameAnvil. At last, if you enter a little bit of the boiler plate code, the initial development settings will be completed.

GameAnvil provides an IntelliJ template to replace such a series of processes, making it easier to complete initial tasks. You can download a project file template for IntelliJ from the following link. The template you downloaded should not be unzipped.

[Download Template](https://static.toastoven.net/prod_gameanvil/files/v2_2/GameAnvil%20Template.zip?disposition=attachment)

Run the IntelliJ to apply the template you downloaded. From the left menu of the **Welcome to InteliJ IDEA**, select **Customize** and click **Import Settings...**. Or from the entire search box, search for **Import Settings...**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/1_import_gameanvil_template.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/2_search_import_settings.png)

<br>

Select a zipped file by navigating to the path where you downloaded the template from the Finder or File Explorer window. When the **Select Components to Import** window opens, both **File templates** and **Project Templates** are checked and selected. Click **OK** and click **Import and Restart** to restart the IntelliJ, and the template will be applied.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/3_select_import.png)

In the IntelliJ button group on the top right, click **New Project** and scroll to the left to select the **GameAnvil 2.2.0 Template** in the **Templates** at the bottom. Set the project name. The name must not have space. Check the project location and base package name before creating a project.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/4_imported_gameanvil_template.png)

The server project skeleton has now been configured on IntelliJ. You can check the Project panel to see that the code and settings files are created.

- Main: a class that includes the main function of the program's entry point.
- GameAnvilConfig.json: a file contains the server settings information required to run GameAnvil. It can be modified according to the server implementation.
- logback.xml: a file used to configure logging in Java projects. As a setting file for the Logback framework, it specifies the operation method of the logging system, the format of the log, the storage location, and more. You can use this file to set logging level, log format, path and name of log file, log rolling policy, and more.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/5_gameanvil_project_view_init.png)

<a id="java-version-settings"></a>
### Java Version Settings { #java-version-settings }

GameAnvil supports Java 21 and 25 versions. Some configuration methods may vary depending on the version, and Java 21 version was used here.

First, let's check the jdk settings. Select **File > Project Structure** from the left upper menu to open the **Project Structure** window. For a Mac user, you can use **Command + ;** shortcut.

Check the SDK settings in the **Project** tab. If you don't have the SDK set, download and set the version you want of JDK via **Add SDK > Download JDK**. Set the **Language level** to **SDK default**. Then set the **Language level** to **Project default** in the **Modules** tab.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/7_project_structure.png)

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/8_module_language_level.png)

Check the **gradle** settings in the **Set** menu.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/9_gradle_config.png)

<a id="modify-gameanvil-server-setting-file"></a>

## Modify GameAnvil Server Setting File { #modify-gameanvil-server-setting-file }

You can change the GameAnvil server setting through the GameAnvilConfig.json file found in the resources package subpart of the project panel.

- common: Sections that cover the server overall settings
- location: Sections that cover the settings related to location nodes
- match: Sections that cover the settings related to match nodes
- gateway: Sections that cover the settings related to gateway nodes
- game: Sections that cover the settings related to game nodes

Since you have configured the project with a template, you can confirm that the GameAnvilConfig.json file has set the default settings required for server operation. There are three main things to note in this example:

1. NodeCnt value for game
2. serviceName value for game
3. channelIDs value for game

Game nodes can be configured and executed with multiple VMs depending on your needs or depending on the server performance. When you set how many game nodes you want to run, the server will automatically read the settings file to enable a certain number of nodes when running. The template settings are set to not enable any game nodes, so modify the setting to enable one. Additional modifications are serviceName and channelIDs in the game section.

If you look at the last part of the game page of the GameAnvilConfig.json file, there is a part displayed as Todo. Modify this to set the service name and channel information.

```json
  "game": [
    {
        "nodeCnt": 0,
        "serviceId": 1,
        "serviceName": "Todo - Input My Service Name",
        "channelIDs": [["ToDo - Input My ChannelName","ToDo - Input My ChannelName"]], // Channel ID to be assigned to each node. (It does not have to be unique.  "" means it doesn't use channel)
        "userTimeout": 5000 // Set how long to manage user objects without removing them from the server after the client is disconnected
    }
]
```

<a id="about-service"></a>
### About Service { #about-service }

A service is a name used to separate each game service when a single server provides multiple games. The service name is an agreed string between the server and the client that denotes a specific service. Don't forget it as it will be used when entering a service name in a later process.

Here it uses a service with the name of Sync. Modify the contents below in serviceName in the game section:

```
"serviceName" : "Sync",
```

<a id="about-channel"></a>
### About Channel { #about-channel }

Channel is one of the methods to logically split a single server group. As a channel is not used in the example, we will omit the detailed description in this article. The content is modified as follows in the channelIDs in the game section because it does not use a channel:

```
"channelIDs" : [[""]],
```

The content of the GameAnvil server setting file that has been written thus is as follows:

```json
"game": [
    {
      "nodeCnt": 0,
      "serviceId": 1,
      "serviceName": "Sync",
      "channelIDs": [[""]], // Channel ID to be assigned to each node. (It does not have to be unique.  "" means it doesn't use channel)
      "userTimeout": 5000 // Set how long to manage user objects without removing them from the server after the client is disconnected
    }
  ]
```

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/basic-tutorial/6_gameanvil_config_json.png)

With the gateway settings, you can see that the TCP_SOCKET connection is set to use the 18200 port. This is the port connected to the client, which will be used in future client projects to enter the server access information.

<a id="implement-gameanvil-server-feature"></a>

## Implement GameAnvil Server Feature { #implement-gameanvil-server-feature }

<a id="perform-game-node"></a>

### Implement Game Node { #perform-game-node }
In this chapter, we will implement the BasicGameNode required for the sample game operation among the various nodes of GameAnvil. GameAnvil provides multiple base classes with the prefix `Base`. The feature of the basic node is already implemented inside the base class, and users can use various callback features by inheriting from them. In this example, we try to create a game node class that inherits from the BaseGameNode base class.

In the project panel, right-click the path to where the Main class is located, and then select **New > Package** to create a new package named **node**. Then right-click the node package again and select **New > GameAnvil GameNode**. When the File Creation dialog box opens, enter **SyncGameNode** in **File name**, and **Sync** in **Service name**, and click **OK**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/13_select_game_node_file_template.png)

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/14_create_sync_game_node.png)

This feature is available because you have previously applied File templates (schemes) when installing a template. If the entry **New > GameAnvil GameNode** is not visible, select **New > Java Class** to create an empty class.

The automatically written code is as follows:

```java
package com.tutorial.gameanvil.node;

import com.nhn.gameanvil.game.GameAnvilGameNode;
import com.nhn.gameanvil.node.game.BaseGameNode;
import com.nhn.gameanvil.node.game.ChannelUpdateType;
import com.nhn.gameanvil.node.game.data.IChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.IChannelUserInfo;
import com.nhn.gameanvil.packet.IPayload;

@GameAnvilGameNode(gameServiceName = "Sync")
public class SyncGameNode extends BaseGameNode {
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {

    }

    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {

    }

    @Override
    public void onChannelInfo(IPayload payload) {

    }

    @Override
    public void onInit() {

    }

    @Override
    public void onPrepare() {

    }

    @Override
    public void onReady() {

    }

    @Override
    public void onPause(IPayload payload) {

    }

    @Override
    public void onShuttingdown() {

    }

    @Override
    public void onResume(IPayload payload) {

    }
}

```

<a id="about-node"></a>

### About Node { #about-node }

All nodes have status depending on whether a loop to start processing something has started. The following is some of the states that a node can have:

- INIT
- PREPARE
- READY
- SHUTDOWN

The node starts from the INIT state and turns the state to the specified order to reach the READY state. The READY state indicates that the node is in a state where the task can be processed and performed.

Auto-generated codes include codes that override the callbacks that are hooked to the state of each node. For example, if you write a specific logic to the onInit() method, the callback is inserted and called just before the node starts initializing (Init).

GameAnvil has most code prepared so no more code to write in this step. You can use the game node as it has been created.

<a id="about-user-type"></a>

### About User Type { #about-user-type }

The user is the object that receives the packet to join the room from each game node, a promised string that separates each user implementation.

In addition to the nodes implemented above, you need a **game user** and **game room** class to use the room-based implementation provided by GameAnvil. I will describe how to easily implement with only interface implementation.

<a id="implement-game-user"></a>

### Implement Game User { #implement-game-user }

When a client logs in to the server, the server stores and retains the client information as an object called the **game user**. What kind of information the game user has to express can be implemented freely by the user's needs. Game user implementation can also be consistently implemented through the inheritance of the class and callback overriding.

In the project panel, right-click on the path to where the Main class is located and then select **New > Package** to create a new package named **user**. Then right-click the **user** package again and select **New > GameAnvil GameUser**. When the File Creation dialog box opens, enter **SyncGameUser** in **File name**, **Sync** in **Service name**, and **USER_TYPE_SYNC** in **User type**, and click **OK**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/15_create_sync_game_user.png)

The auto-generated code is as follows:

```java
package com.tutorial.gameanvil.user;

import com.nhn.gameanvil.game.GameAnvilUser;
import com.nhn.gameanvil.node.game.BaseGameUser;
import com.nhn.gameanvil.node.game.data.MatchCancelReason;
import com.nhn.gameanvil.node.game.data.MatchRoomFailCode;
import com.nhn.gameanvil.node.game.data.MatchUserFailCode;
import com.nhn.gameanvil.node.game.data.RoomMatchResult;
import com.nhn.gameanvil.packet.IPayload;
import com.nhn.gameanvil.serializer.IReadOnlyTransferPack;
import com.nhn.gameanvil.serializer.ITimerHandlerTransferPack;
import com.nhn.gameanvil.serializer.ITransferPack;

@GameAnvilUser(
    gameServiceName = "Sync",
    gameType = "USER_TYPE_SYNC",
    useChannelInfo = false
)
public class SyncGameUser extends BaseGameUser {
    @Override
    public boolean onLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onAfterLogin(boolean isRelogined) {

    }

    @Override
    public boolean onReLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onDisconnect() {

    }

    @Override
    public void onPause() {

    }

    @Override
    public void onResume() {

    }

    @Override
    public void onLogout(IPayload payload, IPayload outPayload) {

    }

    @Override
    public boolean canLogout() {
        return true;
    }

    @Override
    public void onAfterLeaveRoom() {

    }

    @Override
    public boolean canTransfer() {
        return true;
    }

    @Override
    public boolean onLoginByOtherDevice(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean onLoginByOtherUserType(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onLoginByOtherConnection(IPayload payload) {

    }

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        return RoomMatchResult.FAILED;
    }

    @Override
    public void onMatchRoomFail(MatchRoomFailCode matchRoomFailCode) {

    }

    @Override
    public void onMatchUserFail(MatchUserFailCode matchUserFailCode) {

    }

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = false;
        return isSuccess;
    }

    @Override
    public void onMatchUserCancel(MatchCancelReason matchCancelReason) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(IReadOnlyTransferPack iReadOnlyTransferPack, ITimerHandlerTransferPack iTimerHandlerTransferPack) {

    }

    @Override
    public void onAfterTransferIn() {

    }

    @Override
    public void onSnapshot(IPayload payload, IPayload outPayload) {

    }

    @Override
    public boolean canMoveOutChannel(String channelId, IPayload payload, IPayload outPayload) {
        return false;
    }

    @Override
    public void onMoveOutChannel(String channelId, IPayload payload) {

    }

    @Override
    public void onAfterMoveOutChannel() {

    }

    @Override
    public void onMoveInChannel(String channelId, IPayload payload, IPayload outPayload) {

    }
}

```

Game users are created by requiring the client to log in to the server. On the server, you can decide whether to allow logging into the payloads sent from the client and exporting them as a return value. The main logic is only written by the engine user, and the engine is responsible for handling login success or failure.

In this tutorial, to allow login without special validation, we have enabled the onLogin function to always return true. It will always create user objects when the client has a login request and responds successfully.

<a id="implement-game-room"></a>

### Implement Game Room { #implement-game-room }

Once you successfully access the game node as a game user, you can now receive packets from other users and through the game room. Game Room is a group that logically binds the users who receive the packet. Game rooms can also be created through interface implementation.

In the project panel, right-click on the path to where the Main class is located and then select **New > Package** to create a new package named **room**. Then right-click again on the **room** package and select **New > GameAnvil GameRoom**. When the File Creation dialog box opens, enter **SyncGameRoom** in **File name**, **Sync** in **Service name**, **ROOM_TYPE_SYNC** in **Room type**, and **SyncGameUser** in **User**, and click **OK**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/16_create_sync_game_room.png)

The auto-generated code is as follows. Note that SyncGameUser is not imported in the auto-generated code, which causes a compile error — you can fix this by adding the import manually.

```java
package com.tutorial.gameanvil.room;

import com.nhn.gameanvil.game.GameAnvilRoom;
import com.nhn.gameanvil.node.game.BaseGameRoom;
import com.nhn.gameanvil.node.game.data.MatchCancelReason;
import com.nhn.gameanvil.packet.IPayload;
import com.nhn.gameanvil.serializer.IReadOnlyTransferPack;
import com.nhn.gameanvil.serializer.ITimerHandlerTransferPack;
import com.nhn.gameanvil.serializer.ITransferPack;
import java.util.List;

@GameAnvilRoom(
    gameServiceName = "Sync",
    gameType = "ROOM_TYPE_SYNC",
    useChannelInfo = false
)
public class SyncGameRoom extends BaseGameRoom<SyncGameUser> {
    @Override
    public void onInit() {

    }

    @Override
    public void onDestroy() {

    }

    @Override
    public boolean onCreateRoom(SyncGameUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean onJoinRoom(SyncGameUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean canLeaveRoom(SyncGameUser user, IPayload payload, IPayload outPayload) {
        return true;
    }

    @Override
    public void onLeaveRoom(SyncGameUser user) {

    }

    @Override
    public void onAfterLeaveRoom() {

    }

    @Override
    public void onRejoinRoom(SyncGameUser user, IPayload payload) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(List<SyncGameUser> list, IReadOnlyTransferPack iReadOnlyTransferPack, ITimerHandlerTransferPack iTimerHandlerTransferPack) {

    }

    @Override
    public void onAfterTransferIn() {

    }

    @Override
    public void onPause() {

    }

    @Override
    public void onResume() {

    }

    @Override
    public boolean onMatchParty(String roomType, String matchingGroup, SyncGameUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onMatchPartyCancel(MatchCancelReason matchCancelReason) {

    }

    @Override
    public void onForceMatchRoomUnregistered(MatchCancelReason matchCancelReason) {

    }

    @Override
    public boolean canTransfer() {
        return true;
    }
}

```

Game rooms are created when the game user requests room creation on the server. On the client side, you can simply call the method to create a room and enter the room that exists. When the user enters the room or when the room is created, you can easily paste the code by overriding appropriate callbacks.

<a id="run-gameanvil-server"></a>

## Run GameAnvil Server { #run-gameanvil-server }

The server setup required to run the basic tutorial sample is now complete. Let's go ahead and start the server.

<a id="run-server"></a>
### Run Server { #run-server }

Double-click Tasks > other > `runMain` from the gradle menu on the right. Once executed, the server runs even if you click the green Run triangle icon on the top right of the IntelliJ.

You must run it with `runMain` to apply the VM option required to run the GameAnvil server. If you are just running the main() function of the Main class, you must add the required VM options below in **Edit Configurations...**.

```
"--add-opens", "java.base/java.lang=ALL-UNNAMED",
"--add-opens", "java.base/java.lang.invoke=ALL-UNNAMED",
"--add-opens", "java.base/jdk.internal.misc=ALL-UNNAMED", 
"--add-opens", "java.base/java.nio=ALL-UNNAMED",
"-Dio.netty.tryReflectionSetAccessible=true",
"-Dio.netty.noUnsafe=false"
```

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/10_gameanvil_run.png)

When the server runs normally, a number of logs related to the server running status are output. Each node needs time to prepare to execute the code, and when each node is ready, it will output an onReady log. A gateway node is to perform the direct role for the client to connect to the server. If the gateway node is ready and the onReady logs for GatewayNode are output, the GameAnvil server will be connectable at any time.
When you start the server, you can see the message `All nodes are ready!!` among the logs. This means that the GameAnvil server has started successfully.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/17_all_nodes_are_ready.png)

The server is now ready to receive client requests. In the next step, we will implement the client by using the GameAnvil connector and the Unity sample project.

<a id="prepare-practice-environment---client-project"></a>

## Prepare Practice Environment - Client Project { #prepare-practice-environment---client-project }

<a id="download-gameanvilconnector"></a>
### Download GameAnvilConnector { #download-gameanvilconnector }

Download the file below to use GameAnvil connector dll:

[gameanvil_connector_2.2.0.unitypackage](https://static.toastoven.net/prod_gameanvil/files/v2_2/gameanvil-connector.unitypackage)

<a id="download-unity-package"></a>
### Download Unity Package { #download-unity-package }

For hands-on use of GameAnvil connector, download the Unity package from the link below:

[gameanvil_tutorial_basic.unitypackage](https://static.toastoven.net/prod_gameanvil/files/v2_2/gameanvil_tutorial.unitypackage)

<a id="create-unity-project"></a>
### Create Unity Project { #create-unity-project }

After launching the Unity hub, click the New Project button on the top. The version of the Unity hub does not matter.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/18_unity_hub.png)

Select **Universal 2D** as your template, check the project name and where to save it, and click **Create project**.
The Unity version used in this example is 6000.0.59f2. You are free to use a different version when doing the exercise, but the description may differ from this example.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/19_new_unity_project.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/20_new_unity_project_done.png)

<a id="import-gameanvilconnector-and-unity-package"></a>
### Import GameAnvilConnector and Unity Package { #import-gameanvilconnector-and-unity-package }

Right-click on the project view and select **Import Package > Custom Package...** and open the Finder or File Explorer to select the Unity package you downloaded from the previous step. Run Import in the order gameanvil_connector, gameanvil_tutorial_basic.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/21_import_unity_package_gameanvil_connector.png)

From the Scene folder in the GameAnvilSample folder, open IntroScene and see the screen below:

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/22_unity_after_import_package.png)

You can check the Unity UI components used in the example through the UI Manager added to Canvas.

In **File > Build Profiles**, add IntroScene to the **Scene List** to set it to be included when building.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/26_intro_scene_to_build_settings.png)

<a id="gameanvilmanager"></a>

## GameAnvilManager { #gameanvilmanager }

In the Hierarchy view, right-click on the mouse button and click **GameAnvil > GameAnvilManager**. GameAnvilManager game objects are created and you can modify the settings as shown below on the inspector of the GameAnvilManager game objects:

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/27_gameanvil_manager.png)

- Connect Configuration: You can modify access information.
- Authentication Configuration: You can modify the credentials.
- Login Configuration: You can modify the login information.
- Pause Client Check: You can adjust the amount of time to pause the automatic connection status check while the client is in the background. This prevents the connection from being lost when the app switches to the background during this period and then returns to the game.

It is okay that you don't know the details of the settings right now. You can see the description for each item as you proceed with the tutorial.

<a id="connect-server-and-client"></a>

## Connect Server and Client { #connect-server-and-client }

To allow the GameAnvil client to connect to the GameAnvil server, the GameAnvil client must pass the three steps of Connect, Authentication, and Login.

- Connect: Create and connect sockets to allow communication between the server and the client.
- Auth: Determine whether to allow the client to transfer data through the server.
- Login: Create an object that expresses client information in the server's memory, i.e., a game user.

Each step proceeds sequentially, and if the previous step is not completed normally, you cannot proceed with the next step.

You can use the Login API provided by GameAnvilManager on the connector to integrate the Connect, Authentication, and Login processes at once.

Here, you will add the GameAnvilManagerTester script that is added as a component to the Tester game object in the Hierarchy view to open the source code editor and run each process directly.

<a id="connect-related-field-settings"></a>
### Connect Related Field Settings { #connect-related-field-settings }

Enter the server information to connect to. If the server is pulled directly from a local server, ip will use `127.0.0.1`. The port will use `18200`, which is the default port of the gateway node. IP and port information can be verified that the code to connect to the InputField of the Unity is written so that it can be modified in play mode.

```c#
public string managerIp
{
    get => UI.managerIpInputField.text;
    set => UI.managerIpInputField.text = value;
}

public int managerPort
{
    get => int.Parse(UI.managerPortInputField.text);
    set => UI.managerPortInputField.text = value.ToString();
}
```

<a id="authentication-related-field-settings"></a>
### Authentication Related Field Settings { #authentication-related-field-settings }

Enter the information required for authentication. There are three types of authentication required: accountId, deviceId, and password. We are now in a server implementation state to pass the authentication step unconditionally, so no action will be set to any value. You can check that the code is written so you can use the entered values through the InputField of the Unity in play mode.

```c#
public string managerAccountId
{
    get => UI.managerAccountIdInputField.text;
    set => UI.managerAccountIdInputField.text = value;
}

public string managerPassword
{
    get => UI.managerPasswordInputField.text;
    set => UI.managerPasswordInputField.text = value;
}

public string managerDeviceId
{
    get => UI.managerDeviceIdInputField.text;
    set => UI.managerDeviceIdInputField.text = value;
}
```

<a id="login-related-field-settings"></a>
### Login Related Field Settings { #login-related-field-settings }

List the information required to log in. The information required to log in includes user type, channel ID, and service name. Must use the user type and service name written at the time of server deployment. It is set so that the value can be modified through the InputField of the Unity in play mode.

```c#
public string managerServiceName
{
    get => UI.managerServiceNameInputField.text;
    set => UI.managerServiceNameInputField.text = value;
}

public string managerUserType
{
    get => UI.managerUserTypeInputField.text;
    set => UI.managerUserTypeInputField.text = value;
}

public string managerChannelId
{
    get => UI.managerChannelIdInputField.text;
    set => UI.managerChannelIdInputField.text = string.IsNullOrEmpty(value) ? "" : value;
}
```

<a id="set-field-auto-input"></a>
### Set Field Auto Input { #set-field-auto-input }

In order to reduce the amount of work you need to enter each input in play mode, ConstantManager can pre-enter the value to use. The input values used in the example are as follows.

```c#
public class ConstantManager : MonoBehaviour
{
    private GameAnvilManagerTester managerTester;

    public static string ip = "127.0.0.1";
    public static int port = 18200;
    [Space]
    public static string accountId = "test";
    public static string password = "test";
    public static string deviceId = "test";
    [Space]
    public static string serviceName = "Sync";
    public static int subId = 1;
    [Space]
    public static string userType = "USER_TYPE_SYNC";
    public static string channelId = "";
    [Space]
    public static string roomType = "ROOM_TYPE_SYNC";

    ...(omitted)...
}
```

<a id="unity-ui-settings"></a>
### Unity UI Settings { #unity-ui-settings }

```c#
void Start()
{
    UI = GameObject.FindWithTag("UIManager").GetComponent<UIManager>();

    gameAnvilManager = GameAnvilManager.Instance;
    gameAnvilManager.onStateChange.AddListener(OnManagerStateChanged);

    UI.managerLoginButton.onClick.AddListener(ManagerLogin);
    UI.managerLogoutButton.onClick.AddListener(ManagerLogout);
    UI.managerCreateRoomButton.onClick.AddListener(ManagerCreateRoom);
    UI.managerJoinRoomButton.onClick.AddListener(ManagerJoinRoom);
    UI.managerLeaveRoomButton.onClick.AddListener(ManagerLeaveRoom);
    UI.managerUserMatchMakingButton.onClick.AddListener(ManagerMatchUserStart);
    UI.managerRoomMatchMakingButton.onClick.AddListener(ManagerMatchRoom);
    UI.managerCancelMatchMakingButton.onClick.AddListener(ManagerCancelMatchMaking);
}

void Update()
{
    UI.managerLoginState.Set(gameAnvilManager.State == GameAnvilManager.LoginState.LOGIN_COMPLETE);
    UI.managerJoinedRoomState.Set(gameAnvilManager.UserController != null && gameAnvilManager.UserController.IsJoinedRoom);
}

public void OnManagerStateChanged(GameAnvilManager.LoginState oldState, GameAnvilManager.LoginState newState)
{
    UI.consoleInputField.text += newState.ToString() + '\n';
}
```

<a id="call-login-api"></a>
### Call Login API { #call-login-api }

GameAnvilManager's Login API, which integrates the Connect, Authentication, and Login processes, uses the following. You can check the call results through the result value.

```c#
public async void ManagerLogin()
{
    gameAnvilManager.ip = managerIp;
    gameAnvilManager.port = managerPort;
    gameAnvilManager.accountId = managerAccountId;
    gameAnvilManager.deviceId = managerDeviceId;
    gameAnvilManager.password = managerPassword;
    gameAnvilManager.userType = managerUserType;
    gameAnvilManager.channelId = managerChannelId;
    gameAnvilManager.serviceName = managerServiceName;

    try
    {
        var result = await gameAnvilManager.Login(null);
        UI.managerResultCodeInputField.text = result.loginResultCode.ToString();
        UI.managerExceptionInputField.text = null;
        if (result.loginResultCode != GameAnvilManager.LoginResultCode.SUCCESS)
            UI.consoleInputField.text += result.loginResultCode.ToString() + '\n';
        UI.consoleInputField.text += result.loginResultCode.ToString() + '\n';
    }
    catch (Exception e)
    {
        UI.managerResultCodeInputField.text = null;
        UI.managerExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

<a id="call-logout-api"></a>
### Call Logout API { #call-logout-api }

```c#
public async void ManagerLogout()
{
    await gameAnvilManager.Logout();
}
```

<a id="test-server-connection-and-login"></a>
### Test Server Connection and Login { #test-server-connection-and-login }

After confirming that the server is running, the Unity editor enters play mode. Click **Login** to confirm that the server connection and login is working properly.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/28_login_success.png)

When the login is complete, it will appear in the LOGIN status with a green light display in the status display window at the top. Press the **Logout** button to see if you log out normally with this status.

When logout is complete, it will appear in the LOGOUT status with a red light display in the status display window at the top.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/29_logout_success.png)

<a id="create-and-enter-game-room"></a>

## Create and Enter Game Room { #create-and-enter-game-room }

<a id="room-creation-and-stage-related-fields-settings"></a>
### Room Creation and Stage Related Fields Settings { #room-creation-and-stage-related-fields-settings }

```c#
public string managerRoomType
{
    get => UI.managerRoomTypeInputField.text;
    set => UI.managerRoomTypeInputField.text = value;
}

public int managerRoomId
{
    get => int.Parse(UI.managerRoomIdInputField.text);
    set => UI.managerRoomIdInputField.text = value.ToString();
}
```

<a id="use-game-room-creation-request-api"></a>
### Use Game Room Creation Request API { #use-game-room-creation-request-api }

By calling the GameAnvil connector's room creation request API, the client can easily request the server to create a game room. When calling the game room creation request method, you must pass the room type to the parameter and send a previously agreed room time value to the server.

```c#
public async void ManagerCreateRoom()
{
    try
    {
        var result = await gameAnvilManager.UserController.CreateRoom(string.Empty, managerRoomType, ConstantManager.channelId);
        UI.managerRoomResultInputField.text = result.ErrorCode.ToString();
        UI.managerRoomIdResultInputField.text = result.Data.RoomId.ToString();
        UI.managerRoomExceptionInputField.text = null;
        UI.consoleInputField.text += result.ErrorCode.ToString() + '\n';
        UI.consoleInputField.text += "Room Id : " + result.Data.RoomId + '\n';
    }
    catch(Exception e)
    {
        UI.managerRoomResultInputField.text = null;
        UI.managerRoomIdResultInputField.text = null;
        UI.managerRoomExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

The game room creation feature has been implemented. The tests are delayed a while and the game room entry feature will be implemented first.

<a id="use-game-room-input-request-api"></a>
### Use Game Room Input Request API { #use-game-room-input-request-api }

Suppose a game room has been created on the server. To access the room, you can call the GameAnvil room entry request method from the GameAnvil connector. At this time, the game room ID received at the time of room creation is forwarded. Enter the room ID to enter through the InputField of your unit in play mode.

```c#
public async void ManagerJoinRoom()
{
    try
    {
        var result = await gameAnvilManager.UserController.JoinRoom(managerRoomType, managerRoomId, string.Empty);
        UI.managerRoomResultInputField.text = result.ErrorCode.ToString();
        UI.managerRoomIdResultInputField.text = result.Data.RoomId.ToString();
        UI.managerRoomExceptionInputField.text = null;
        UI.consoleInputField.text += result.ErrorCode.ToString() + '\n';
        UI.consoleInputField.text += "Room Id : " + result.Data.RoomId + '\n';
    }
    catch (Exception e)
    {
        UI.managerRoomResultInputField.text = null;
        UI.managerRoomIdResultInputField.text = null;
        UI.managerRoomExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

The game room creation and entry feature are now completed.

<a id="game-room-test"></a>
### Game Room Test { #game-room-test }

Press `CMD + b` or `Ctrl + b` to build from the Unity editor. Click the buttons in the build result window to confirm that a game room is created. When a game room is created, the ID of the game room is displayed on the screen.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/30_create_room_success.png)

Enter play mode from the Unity editor. Set the screen to 1080 X 1920 if needed. Enter the ID of the previously displayed game room in the input field, and click **Join Room** to see if you are involved in the game room.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/31_join_room_success.png)

<a id="introduction-to-synchronization-controller"></a>

## Introduction to Synchronization Controller { #introduction-to-synchronization-controller }

You can now give and receive packets between game users who have accessed the same game room. This packet allows you to write code to synchronize the required information between client processes. In a simpler way, you can only synchronize by attaching a synchronization component to the game objects you want to synchronize.

<a id="synchronization-controller"></a>
### Synchronization Controller { #synchronization-controller }

Search the SyncController game object in the Hierarchy view to see if the SyncController is added as a component.

If the game object does not exist at any time, right-click the mouse button in the Hierarchy view and select **GameAnvil > SyncController** to add it.

You now have access to all the synchronization features of GameAnvil. Here is how to attach and use synchronous components through the most simple example.

<a id="synchronization-object"></a>
### Synchronization Object { #synchronization-object }

Go inside the Resources folder in the project view and double-click the Character Prefab to switch to the Modify Prefab screen. Verify that the Sync, TransformSync, RigidBody2DSync, and AnimatorSync components are added in the inspector window.

These components are for the synchronization feature provided by the GameAnvil connector. By adding only components, the prefab is ready to synchronize the Transform, Rigidbody2D, and Animation information of the game object between the game users.

![](https://static.toastoven.net/prod_gameanvil/images/v2_2/basic-tutorial/32_character_sync_prefab.png)

To use the completed synchronized game object prefab in Unity Play mode, you can create a game object through the Game Object Creation API provided by SyncController and add it to the tone. You must pass the prefab name as the first argument.

Please note that synchronized game objects with the synchronization component provided by GameAnvil must use the SyncController's Instantiate API provided by GameAnvil instead of the GameObject.Instantiate() method provided by the unit.

When you enter the room, write the code to create a synchronization game object.

```c#
public async void ManagerCreateRoom()
{
    try
    {
        var result = await gameAnvilManager.UserController.CreateRoom(string.Empty, managerRoomType, ConstantManager.channelId);
        UI.managerRoomResultInputField.text = result.ErrorCode.ToString();
        UI.managerRoomIdResultInputField.text = result.Data.RoomId.ToString();
        UI.managerRoomExceptionInputField.text = null;
        UI.consoleInputField.text += result.ErrorCode.ToString() + '\n';
        UI.consoleInputField.text += "Room Id : " + result.Data.RoomId + '\n';

        // Add Code
        if (result.ErrorCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
        {
            OnEnterRoom();
        }
    }
    catch(Exception e)
    {
        UI.managerRoomResultInputField.text = null;
        UI.managerRoomIdResultInputField.text = null;
        UI.managerRoomExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

```c#
public async void ManagerJoinRoom()
{
    try
    {
        var result = await gameAnvilManager.UserController.JoinRoom(managerRoomType, managerRoomId, string.Empty);
        UI.managerRoomResultInputField.text = result.ErrorCode.ToString();
        UI.managerRoomIdResultInputField.text = result.Data.RoomId.ToString();
        UI.managerRoomExceptionInputField.text = null;
        UI.consoleInputField.text += result.ErrorCode.ToString() + '\n';
        UI.consoleInputField.text += "Room Id : " + result.Data.RoomId + '\n';

        // Add Code
        if (result.ErrorCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS)
        {
            OnEnterRoom();
        }
    }
    catch (Exception e)
    {
        UI.managerRoomResultInputField.text = null;
        UI.managerRoomIdResultInputField.text = null;
        UI.managerRoomExceptionInputField.text = e.ToString();
        UI.consoleInputField.text += e.ToString() + '\n';
    }
}
```

```c#
private void OnEnterRoom()
{
    Debug.Log("User enter the room.");
    SyncController.Instance.InstantiateSyncObject();
    SyncController.Instance.Instantiate("Character", Vector3.zero, Quaternion.identity);
}
```

<a id="synchronization-test"></a>
### Synchronization Test { #synchronization-test }

Build by pressing the shortcuts `CMD + b` or `Ctrl + b` in the Unity editor. After logging in from the build result window, create a game room and press the **Toggle Hide** button to display the screen testing the synchronization game objects.

Enter play mode from the Unity editor. After entering the same game room as the window that came from the build, press the **Toggle Hide** button to display the test screen of synchronization game object.

In this case, to be identified as a different user, you must use a different accountId than the one already used when logging in.

When you enter a room and create a game object from one client, make sure the same appears on the screen of another client in the same room.

Enter the W,S,A,D key or the top-added key on the keyboard to move the game object. Verify that the game object moves and the animation changes in the process. Verify that the same appears on the other client screen that is in the same room.

With the synchronization feature provided by GameAnvil, we have experienced synchronization of game object creation and Transform, Rigidbody, and Animation.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/basic-tutorial/33_sync_test.gif)

<a id="end-tutorial"></a>

## End Tutorial { #end-tutorial }

In this article, we have learned through practice about the quick login and synchronization features that integrate connection, authentication, and login procedures in GameAnvil connectors. As introduced at the beginning of the tutorial, GameAnvil has all the features required to create a game server, and the tutorial has only covered partly. More details of the use can be learned in the following article.