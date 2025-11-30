## Game > GameAnvil > In-depth Tutorial

### Create Game Server Easily with GameAnvil

GameAnvil is a real-time multiplayer game server creation platform. GameAnvil provides a connector to connect clients to servers, as well as server engines. It makes it easy to develop and operate your game servers and clients.

This article, along with the basic features of GameAnvil, covers the process of developing an in-game chat and real-playing multiplayer puzzle game using a user-defined protocol.

Completing a sample to see how the server and client interact, you get familiar with the overall flow of using GameAnvil to develop your game.

Use the templates provided to easily build a server, and with the features provided by the GameAnvil connector, you can implement chat, multiplayer synchronization, matching, and more on your Unity client.

Instead of just listing the concepts and APIs for the server, this article presents an orderly description of the process of developing a real-playing multiplayer puzzle game with a more concrete example to help you understand. Follow the content of the document one by one, and naturally increase your understanding of GameAnvil and multiplayer game development.

## Project Configuration

To create a multiplayer game, you need a server program that is adaptable to the client. In this example, Unity and GameAnvil connectors are used for client program creation, and the server engine GameAnvil introduced earlier for server program creation. First create a server program project using GameAnvil, then create a client program project using the Unity and GameAnvil connectors.

By performing the steps below, the final server sample project created can be downloaded from the link below. If you implement the server feature through several steps in the initial template, you can download the project to pre-check what structure it will have.

[Download Server Sample Project](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil_Tutorial_Advanced_Server.zip?disposition=attachment)

### GameAnvil Project Configuration

To apply GameAnvil to your project, you must download the GameAnvil Library from the Maven repository and write the setting file required to run GameAnvil. Finally, if you write a little bit of the boiler plate code, the development initial settings will end. This chapter aims to complete the initial configuration to get started with the development. Running a server by running a real process is covered in the next chapter.

GameAnvil provides an IntelliJ template to replace such a series of processes, making it easier to complete initial tasks. You can download a project file template for IntelliJ from the following link. The template you downloaded should not be unzipped.

[Download Template](https://static.toastoven.net/prod_gameanvil/files/v2_1/GameAnvil%20Template.zip?disposition=attachment)

Run the IntelliJ to apply the template you downloaded. From the left menu of the **Welcome to InteliJ IDEA**, select **Customize** and click **Import Settings...**. Or from the entire search box, search for **Import Settings...**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/1_import_gameanvil_template.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/2_search_import_settings.png)

<br>

Select a zipped file by navigating to the path where you downloaded the template from the Finder or File Explorer window. When the **Select Components to Import** window opens, both `File templates` and `Project Templates` are checked and selected. Click **OK** to restart the IntelliJ to complete applying the template after the import is complete.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/3_select_import.png)

In the IntelliJ button group on the top right, click **New Project** and scroll to the left to select the `GameAnvil 2.1.0 Template` in the **Templates** at the bottom. Set the project name. The name must not have space. Check the project location and create a project.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/4_imported_gameanvil_template.png)

The server project skeleton has now been configured on IntelliJ. You can check the Project panel to see that the code and settings files are created.

- Main: a class that includes the main function of the program's entry point.
- protocol package: a package that contains a protocol buffer file compiled with java.
- BasicProtocol.proto, Puzzle.proto: a protocol file created using a protocol buffer.
- GameAnvilConfig.json: a file contains the server settings information required to run GameAnvil. It can be modified according to the server implementation.
- logback.xml: a file used to configure logging in Java projects. As a setting file for the Logback framework, it specifies the operation method of the logging system, the format of the log, the storage location, and more. You can use this file to set logging level, log format, path and name of log file, log rolling policy, and more.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/5_gameanvil_project_view_init.png)

Check the JDK settings first. GameAnvil supports Java 21 version. Some configuration methods may vary depending on the version, and Java 21 version was used here.

In the top left menu, click Select **File > Project Structure**. For a Mac user, you can use `Command + ;` shortcuts.

Check the SDK settings in the Project tab. If you don't have the SDK set, download and set the version you want of JDK via `Add SDK > Download JDK`. Set the Language level to SDK default. Then set the Language level to Project default in the Modules tab.

Check the SDK settings in the **Project** tab. If you don't have the SDK set, download and set the version you want of JDK by clicking **Add SDK > Download JDK**. Set the Language level to `SDK default (Java 21)`. Then set the Language level to `Project default (Java 21)` in the **Modules** tab.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/6_project_structure.png)

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/7_module_language_level.png)

Check the **gradle** settings in the **Set** menu.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/8_gradle_config.png)

The project is almost ready, but several settings are required to run. Here, first create a client project, then complete the server settings and run it.

### Unity Project Configuration

Run the Unity Hub. Click **NEW** on the top right to open the Create New Project window

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/9_unity_hub.png)

Select **2D** from the template list, check the project name and location, and click **CREATE** to complete the project creation.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/10_new_unity_project.png)

Download the GameAnvil connector from the following link. The connector is a package that provides the client API required to communicate with the GameAnvil server and helps you implement the client with just a simple code.

[gameanvil-connector.unitypackage](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector.unitypackage)

From the link below, download the Unity package that includes tutorial code, image source, and more for creating client projects for hands-on practice.

[gameanvil\_tutorial\_advanced.unitypackage](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil_tutorial_advanced.unitypackage)

Drag and import the downloaded package files to the Unity project. Or open the **Asset > Import Package > Custom Package...** menu to select a package file from the Finder or File Explorer. Select all checkboxes in the Import Unity Package dialog box and click **Import**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/11_import_unity_package.png)

Finally, to provide convenience for testing, select the Player tab in the Project Settings window and set the project default window width and height to 1920, 1080, respectively, as shown below in the Resoultion and Presentation item.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/12_unity_project_setting_resolution.png)

Client project configuration has been completed.

## Server Run and Connect

### Run GameAnvil Server

When the execution is set, double-click Tasks > other > `runMain` from the gradle menu on the right. Once executed, the server runs even if you click the green Run triangle icon on the top right of the IntelliJ.

You must run it with `runMain` to apply the VM option required to run the GameAnvil server. If you are just running the main() function of the Main class, you must add the required VM options below in \*\*Edit Configurations...\*\*.

```
"--add-opens", "java.base/java.lang=ALL-UNNAMED",
"--add-opens", "java.base/java.lang.invoke=ALL-UNNAMED" 
"-XX:+UseG1GC"
```

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/13_gameanvil_run.png)

When the server runs normally, a number of logs related to the server running status are output.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/14_gameanvil_run_log.png)

GameAnvil server consists of multiple nodes. These nodes divide the functions that the server performs into several roles. We have only checked the server's initial run yet. As we have not set up the code for running a node or other server, we are not fully prepared.

Each node needs time to prepare to execute the code,. When each node is ready, it will output an onReady log. A gateway node is to perform the direct role for the client to connect to the server. If the gateway node is ready and the onReady logs for GatewayNode are output, the GameAnvil server will be connectable at any time.

### Write Connect Handler

Now go to the Unity project to write the code to allow you to connect to the GameAnvil server. You must first create a connector object to connect to the server.

Double-click ConnectScene in the Scene folder in the Asset panel to prepare the scene. The ConnectHandler script added as a component to the ConnectHandler game object in the Hierarchy view is opened in the source code editor to see what is implemented and run each process directly.

GameAnvil servers and clients must receive status check packets periodically. To stop receiving status check packets in situations where the client has been paused, use the PauseClientStateCheck() method. To resume status check, call the ResumeClientStateCheck() method. Detect the client's suspension in the OnApplicationPause() function to call the appropriate method.

```c#
using System;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using Protocol;
using UnityEngine.SceneManagement;
using Random = UnityEngine.Random;

public class ConnectHandler : MonoBehaviour
{
    private static ConnectHandler instance;

    [Header("Object Reference")]
    public Text serverInfoText;
    public InputField serverInfoInput;
    public Text clientInfoText;
    public Text logText;

    public string ip = "127.0.0.1";
    public int port = 18200;

    public string deviceId = "";
    public string password = "";
    public string accountId = "";

    public int roomId;

    public GameObject popupCanvas;
    public InputField roomIdInput;

    private static GameAnvilConnector connector;
    private static GameAnvilUser user;
    
    void Start()
    {
        // Enter the connection information on the screen.
        serverInfoText.text = "Server Information:";
        serverInfoInput.text = ip;
    }

    private void Awake()
    {
        DontDestroyOnLoad(this.gameObject);
    }

    private void OnApplicationPause(bool pause)
    {
        if(!getConnector().IsConnected)
            return;

        if (pause)
        {
            // Stops the clientStateCheck feature on the server for the entered time.
            // After this time, the clientStateCheck feature may be enabled and the connection may be lost.
            getConnector().PauseClientStateCheck(10 * 60);
        }
        else
        {
            // Restart the clientStateCheck feature on the server.
            getConnector().ResumeClientStateCheck();
        }
    }

    public void Quit()
    {
        if (getConnector().IsConnected)
        {
            getConnector().Disconnect();
        }
#if UNITY_EDITOR
        UnityEditor.EditorApplication.isPlaying = false;
#else
        Application.Quit();
#endif
    }
}
```

### Create Connector and User

You must create a Connector and a User to use multiple features in the connector. The Connector provides features such as server access and authentication, and the User provides features related to the user, such as login, room creation, and entry.

Create a new GameAnvilConnector object in the getConnector() function and saves it in the connector field. Then a new GameAnvilUser object is created in the getUser() function and stored in the user field. The service name used when creating a user is also used as a server project.

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();
    }

    return connector;
}

public GameAnvilUser getUser()
{
    if (user == null)
    {
        user = new GameAnvilUser(getConnector(), "BASIC_SERVICE", 1);
    }

    return user;
}
```

<br>

### Connect to Server

The Connect() method to connect to the server using the API provided by the connector is as follows:

```c#
// Client-side

public async void Connect()
{
    ip = serverInfoInput.text;

    getConnector().OnDisconnect += (resultCode, payload) =>
    {
        Debug.Log("Disconnected");
    };

    try
    {
        await getConnector().Connect(ip, port);
        Debug.Log("CONNECT_SUCCESS");
        logText.text = "CONNECT_SUCCESS";
    }
    catch (Exception e)
    {
        Debug.Log("CONNECT_FAIL");
        logText.text = "CONNECT_FAIL";
        throw;
    }
}
```

<br>

The Connect() function calls the Connect method of the GameAnvilConnector. This attempts to connect to the server with the ip and port information given by the argument.

The client is also prepared to connect to the server as the server is prepared to accept the connection.

### Confirm Server Connection

Now enter play mode from the Unity client to see if the result code is printed correctly on the console. On the game screen, you can see success messages for the connection, along with the access information for the IP and Port. Clients connected to the game server can now receive messages through the server.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/15_connect_success.png)

## Create Room and User

The client connected to the server is called the **game user**. Clients connected to the server can log in as more than one game user (in this example, when logging in as a single user) on the server. Game users can communicate with other users in the same room because they belong to one **game room**. It means that users must be in the same room to exchange messages related to the game with different users.

GameAnvil has pre-prepared the default implementation of the game user and the game room, allowing you to easily expand the engine class and complete the structure of the game user and the room using the connector's API. On the engine side, we discuss how to define the game user and the room, while on the connector side, we discuss examples of using APIs that request room creation or participation.

### User

On a server, the features of the game user and the game room are defined as classes. First, let me define the game user.

In the project panel, right-click on the path to where the Main class is located and then select **New > Package** to create a new package called **game**. Then right-click again on the **game** package and then select **New > GameAnvil User**. When the Create File dialog box opens, enter **BasicUser** in **File name**, and **USER\_TYPE\_BASIC** in **User type** in **Service name**, and click **OK**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/16_create_user.png)

By implementing the IUser interface provided by GameAnvil, a file with the default code to implement the game user is created. To implement the desired feature for the game user in GameAnvil, you can enable the IUser interface to override multiple callback functions called according to the situation to execute the desired code. This is part of a list of supported callback functions.

- onLogin: A callback that runs on the login request. You must forward whether to allow the login request with the return value. If you return false, the client will fail to log in. You can use parameters to reference the payloads received from the client, and you can transfer additional information to the client beyond the login permission information through a reference to the payloads you want to return to the client.
- onPostLogin: A callback that runs after logging in.
- onReLogin: If you receive a new login request for a user that is already logged in, you can process it separately in this callback.
- onDisconnect: Callback called when the connection to the server-registered user information has been lost by a client requesting logout or force logout of the server.

Other callback functions are not involved in the login process, so you don't need to know exactly what every callback means right now.

The onLogin callback method implements the actions that must be executed during the login process. In this example, return true to ensure unconditional login succeeds without any login implementation. And create a getter for userContext.

```java
@GameAnvilUser(
        gameServiceName = "BASIC_SERVICE",
        gameType = "USER_TYPE_BASIC",
        useChannelInfo = true
)
public class BasicUser implements IUser {

    private static final Logger logger = getLogger(BasicUser.class);

    private IUserContext userContext;

    public IUserContext getUserContext() {
        return userContext;
    }

    @Override
    public void onCreate(IUserContext userContext) {
        this.userContext = userContext;
    }

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
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();
        try {
            return userContext.matchRoom(matchingGroup, roomType, gameRoomMatchForm);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicUser::onMatchRoom", e);
            return RoomMatchResult.FAILED;
        }
    }

    @Override
    public void onMatchRoomFail(MatchRoomFailCode matchRoomFailCode) {

    }

    @Override
    public void onMatchUserFail(MatchUserFailCode matchUserFailCode) {

    }

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;

        try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(userContext.getUserId());
            return userContext.matchUser(matchingGroup, roomType, term);
        } catch (TimeoutException | NodeNotFoundException | GameAnvilException e) {
            logger.error("BasicUser::onMatchUser", e);
        }

        return isSuccess;
    }

    @Override
    public void onMatchUserCancel(MatchCancelReason matchCancelReason) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

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

<br>

### Room

The implementation of an loginable user has been completed. The game room is now implemented. Just like the user creation method, the IRoom interface implemented class is created using the **GameAnvil Room** file template. Enter **BasicRoom** in **File name**, **BASIC_SERVICE** in **Service name**, **ROOM_TYPE_BASIC** in Room type, and the class name **BasicUser** of the IUser implementation class created in the previous step in **User**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/17_create_room.png)

Click **OK** to automatically write the supported callback method. To explain the callback supported by GameAnvil, let me briefly explain the client's API. The client can call the room-related API to communicate with other users after logging in with the connector. Support actions such as creating a room, participating in the room created by another user, or leaving the room.

- onCreateRoom: runs when the user requests room creation. The additional information provided by the client when creating a room is provided in parameters, along with the user information requested to create a room, so we decide whether to allow the creation of a room based on it and return and implement it.
- onJoinRoom: runs if you request entry to a room created by another user. As with other callbacks, we provide additional information from the client when entering the room, along with the user information on which he requested the room entry, so we decide whether to allow the room entry based on it and implement it.
- onLeaveRoom: runs when a user requests to be excepted from the room. As with other callbacks, it provides the user information that requested the room exit along with additional information that was passed when the room exit request was made. Based on this information, it is implemented by returning the information after deciding whether to allow the room exit.

Information about the user belonging to the room can be obtained through the **roomContext.getAllUsers()** function. Utilize this and add the **broadcast()** function to send the same packet to all users in the room. Add also the field to store the room matching information and the data structure to synchronize the puzzle location.

Then create a room matching information object in **onInit** callback and add the code to register and update each room matching information in **onCreateRoom/onJoinRoom** callback. It allows the room to be added to the matching list, allowing other users to be matched with the room.

```java
package org.example.game;

import com.nhn.gameanvil.exceptions.GameAnvilException;
import com.nhn.gameanvil.exceptions.NodeNotFoundException;
import com.nhn.gameanvil.game.GameAnvilRoom;
import com.nhn.gameanvil.node.game.IRoom;
import com.nhn.gameanvil.node.game.context.IRoomContext;
import com.nhn.gameanvil.node.game.data.MatchCancelReason;
import com.nhn.gameanvil.packet.IPayload;
import com.nhn.gameanvil.packet.Packet;
import com.nhn.gameanvil.serializer.ITimerHandlerTransferPack;
import com.nhn.gameanvil.serializer.ITransferPack;
import org.example.StringValues;
import org.example.match.BasicRoomMatchInfo;
import protocol.Puzzle;
import org.slf4j.Logger;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.TimeoutException;

import static org.slf4j.LoggerFactory.getLogger;

@GameAnvilRoom(
        gameServiceName = "BASIC_SERVICE",
        gameType = "ROOM_TYPE_BASIC",
        useChannelInfo = true
)
public class BasicRoom implements IRoom<BasicUser> {

    private static final Logger logger = getLogger(BasicRoom.class);

    private IRoomContext roomContext;

    // Add Code
    private BasicRoomMatchInfo gameRoomMatchInfo;
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();

    public void broadcast(Packet packet) {
        final var allUsers = roomContext.getAllUsers();

        for (final var user : allUsers) {
            final var userContext = ((BasicUser)user).getUserContext();
            userContext.send(packet);
        }
    }

    @Override
    public void onCreate(IRoomContext<BasicUser> roomContext) {
        this.roomContext = roomContext;
    }

    @Override
    public void onInit() {
        gameRoomMatchInfo = new BasicRoomMatchInfo(roomContext.getId());
    }

    @Override
    public void onDestroy() {

    }

    @Override
    public boolean onCreateRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean canLeaveRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        return true;
    }

    @Override
    public void onLeaveRoom(BasicUser user) {

    }

    @Override
    public void onAfterLeaveRoom() {

    }

    @Override
    public void onRejoinRoom(BasicUser user, IPayload payload) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(List<BasicUser> list, ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

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
    public boolean onMatchParty(String roomType, String matchingGroup, BasicUser user, IPayload payload, IPayload outPayload) {
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

<br>

### GameNode

The game user and the game room are now prepared. But there are still no nodes to process the creation and deletion requests for game users/game rooms. GameNode is the node that serves to manage game users and game rooms. This node is the node that performs most game logic processing roles generally expected of a game server. Adding nodes to GameAnvil is natural and simple. Just like you have defined the game user and the game room, you can create a class by implementing a predefined interface and then implementing additional desired features. After selecting the **GameAnvil GameNode** template, create a game node class with the file name **BasicGameNode**, set the service name to **BASIC\_SERVICE** and press the **OK** button.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/18_create_game_node.png)

To be able to perform the role, the node must first run a loop. When the node runs, it will undergo a series of processes and takes some time. Metrics that indicate whether the node is running or in what stage of the execution process are called the status of the node. The status of the node generally changes sequentially according to the following order:

- INIT
- PREPARE
- READY

A node that has reached the READY state is now ready to execute the logic pre-written by the user. If you want to execute a specific code when each preparation phase is reached, you can implement the callback method and set the code to run when the callback method is called out from the engine. Now there is no code to be specially executed, so the created code will be used as it is.

```java
import com.nhn.gameanvil.game.GameAnvilGameNode;
import com.nhn.gameanvil.node.game.ChannelUpdateType;
import com.nhn.gameanvil.node.game.IGameNode;
import com.nhn.gameanvil.node.game.context.IGameNodeContext;
import com.nhn.gameanvil.node.game.data.IChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.IChannelUserInfo;
import com.nhn.gameanvil.packet.IPayload;

@GameAnvilGameNode(gameServiceName = "BASIC_SERVICE")
public class BasicGameNode implements IGameNode {
    private IGameNodeContext gameNodeContext;

    public IGameNodeContext getContext() {
        return gameNodeContext;
    }

    @Override
    public void onCreate(IGameNodeContext gameNodeContext) {
        this.gameNodeContext = gameNodeContext;
    }

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

Register it in the settings file to link the game user and game room you created with GameNode. Add the following to the 'game' item of the GameAnvilConfig.json file:

```json
// Nodes that play the role of game lobby. (including game room and user)
  "game": [
    {
      "nodeCnt": 1,
      "serviceId": 1,
      "serviceName": "BASIC_SERVICE",
      "channelIDs": [["ch1"]], // Channel ID to be granted for each node. (does not have to be unique. "" means not using the channel)
      "userTimeout": 5000 // Set how long a user object will be maintained on the server after the client disconnects
    }
  ],
```

<br>

### Game Node, User, and Room Settings

 You can also create a class by right-clicking and selecting **New > Java Class**. 

```java
@GameAnvilGameNode(gameServiceName = StringValues.serviceName)
public class BasicGameNode implements IGameNode {
    // ...
}

@GameAnvilRoom(
    gameServiceName = StringValues.serviceName,
    gameType = StringValues.roomType,
    useChannelInfo = false
)
public class BasicRoom implements IRoom<BasicUser> {
    // ...
}

@GameAnvilUser(
    gameServiceName = StringValues.serviceName,
    gameType = StringValues.userType,
    useChannelInfo = false)
public class BasicUser implements IUser {
    // ...
}
```

And the @GameAnvilGameNode annotation provided by the engine automatically register the setting related to the game node, the user via the @GameAnvilUser annotation, and the @GameAnvilRoom annotation. 

The user type is the agreed string between the server and client to identify each user implementation, and the room type is the agreed string between the server and client to identify each room implementation.

Remember, this type must be used when implementing a future client project. The user and room type used in the example are as follows:

```java
public class StringValues {
    public static final String serviceName = "BASIC_SERVICE";
    public static final String userType = "USER_TYPE_BASIC";
    public static final String roomType = "ROOM_TYPE_BASIC";
}
```

Enter the BasicGameNode, BasicUser, and BasicRoom creators to use in the example as parameters.

Finally, in the section registering the Config, you will proceed with tasks such as setting up channel information, registering protocols, and more. The part of registering protocols will be covered in the later part of implementing in-game chat and puzzle logic.

The feature allows the client to connect to the server, log in as a game user, and create a game room has now been implemented. But because you are connected to the server, you can't immediately request game-related features (create a game user, create a game room, etc.) Even if you run the server and client in the current state, the client will not be able to use the features of the game server. To request these to the server, a client authentication process is required after accessing the server. The next chapter covers how to proceed with authentication on the server and client.

## Server Connection

After the client connects to the server, the user must be verified and authenticated before logging in to the game. If the game node is responsible for the game user and the game room creation role, the gateway node is responsible for the user's access and authentication feature. Gateway nodes can also be implemented in a consistent manner like game nodes through class creation. After selecting the **GameAnvil GatewayNode** template, set the file name to **BasicGatewayNode** and press the **OK** button to create a gateway node class. For gateway node classes, additional codes are not required other than the default created code.

### Register Protocol

During the authentication process, the server and client verify the protocols to use each other. Therefore, you must register the protocol before performing the authentication process. You can only register a protocol once, so please add the protocol registration code inside the getConnector() code. After following the tutorial, pre-register the protocol to use in the game chat that will be implemented.

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();

        // RegisterProtocol must be before Auth.
        GameAnvilProtocolManager.RegisterProtocol(BasicProtocolReflection.Descriptor);
    }

    return connector;
}
```

<br>

### Add Authentication Code

Go to the unity project to implement the client-side. The client can request authentication through the connector GameAnvilConnector API, like the connection request. Requests for authentication can only be submitted after the connection request. We will enable you to check the authentication request result through the text on the screen and the console.

```c#
// Client-side

public async void Auth()
{
    accountId = Random.Range(1000, 9999) + "";
    clientInfoText.text = "Client Information : " + accountId;

    try
    {
        var result = await connector.Authentication(deviceId, accountId, password);
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
    }
    catch (Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
        throw;
    }
}
```

The client is now set not only to connect to the server, but also to request the authentication process.

### Verify Authentication

Enter play mode from the Unity client. Verify that logs on the console are accessed and issued in order of authentication. Once cleaned up, the user has been authenticated from the server's gateway node to be able to log in to the server according to the client's Auth request.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/19_auth_success.png)

## Login

The last step is to log in to the game node to create a game user. The game user is a concept required to communicate with other clients connected to the server. To communicate between clients, each client creates a game user to send messages through the game room.

### Add Login Code

Clients who have accessed and authenticated can log in to the game node. When you log in to the server, a game user object is created for the client in the game node. The client can send and receive messages from the server or other users through its server-side game user objects. Modify the authentication code you wrote previously as shown below to make sure you log in immediately when the authentication is successful:

```c#
// Client-side

public async void Login()
{
    try
    {
        var result = await getUser().Login("USER_TYPE_BASIC", "ch1");
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
    }
    catch (Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
        throw;
    }
}
```

Enable the callback methods you send together when you request an login to log out the results of the login request.

### Confirm Login

Verify that you logged in successfully through the Unity test mode.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/20_login_success.png)

## Create and Participate

### Client Task

Enter the method to request creation of room in the ConnectHandler code in the Unity project. Please note that in this case, the RoomType, the second element of the CreateRoom method, must be equal to the value specified on the server. Generally, protocols such as RoomType are predefined and used by servers and client developers. Output the room creation result code to the console. Also, once the creation succeeded, the roomId will be stored on the client side and the code will be written to move to the game thin.

```c#
public async void CreateRoom()
{
    try
    {
        var result = await getUser().CreateRoom(string.Empty, "ROOM_TYPE_BASIC", string.Empty);
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
        if (result.ErrorCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
        {
            Debug.Log("Room Id : " + result.Data.RoomId);
            this.roomId = result.Data.RoomId;
            SceneManager.LoadScene("GameScene");
        }
    }
    catch(Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
    }
}
```

If this is over, you will have implemented the feature to create rooms through the client. Connect the CreateRoom method to the OnClick event on the Create Room button. Before we actually run it, we will implement the room participation feature and then test it. Add a room participation code to ConnectHandler this time.

Verify that the first argument of the JoinRoom method matches the predefined RoomType string used in server development. Check the response of the server side to the room participation request with the result code and write it to do the same thing as creating a room if the room participation succeeds. When the method is created, associate the OnClick event on the Join Room button with the JoinRoom method. When the Enter button is entered in the Update() function of the unit, the code is written so that JoinRoom() is called.

```c#
private void Update()
{
    if (popupCanvas && popupCanvas.activeInHierarchy && Input.GetKeyDown(KeyCode.Return))
    {
        JoinRoom();
    }
}

public async void JoinRoom()
{
    if (!string.IsNullOrEmpty(roomIdInput.text))
    {
        try
        {
            var result = await getUser().JoinRoom("ROOM_TYPE_BASIC", int.Parse(roomIdInput.text), string.Empty);
            Debug.Log(result.ErrorCode.ToString());
            logText.text = result.ErrorCode.ToString();
            if (result.ErrorCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS)
            {
                Debug.Log("Room Id : " + result.Data.RoomId);
                this.roomId = result.Data.RoomId;
                SceneManager.LoadScene("GameScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
        }
    }
}
```

Now go to GameScene to realize what you want to do after attending the room. First, the initial GameManager code is as follows. Create a room ID that can be displayed on the game.

```c#
public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public InputField ChatInputText;

    private ConnectHandler connectHandler;


    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }
}
```

Now the server access, authentication, and login features, as well as room participation and creation features are implemented on the client.

### Create and Test Participation in Room

Select **File > Build Setting** in the top toolbar bar of the Unity project. Add the tones you need to build as shown below. If the thin order is incorrect, drag the items on the list to adjust the order so that ConnectScene appears at the top.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/21_add_scene.png)

Build and play with `cmd+b` or `ctrl+b in` Unity. The ConnectScene is loaded as a new window is opened. Enter the server's IP address and press the Connect button to try connecting to the server. Enter `127.0.0.1` because you are using a local server in the example. If the server connection is successful, you can check the connection status along with console logs on the game screen as `CONNECT_SUCCESS`. If a failure log remains in the middle of the process, you can find out about the cause through the failure code. If this is not enough, you need to check the server logs to analyze the source. If the login is successful, press the Create Room button to request the creation of a room. With GameScene, the tone is moved and the ID of the room created is also output. The room ID may be different than the example image below:

After running Unity play mode as it is, make sure to complete the login. Then click the Join Room button and enter the RoomId of the room you previously created to join the room. If you successfully join the room, you will be able to navigate to the game scene and check whether you have the same room ID on both game screens. If the test screen appears similarly as below, it will succeed:

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/22_join_room.png)

If the scalp is not working properly, please check the below again:

- Is the server process restarted after modifying the server implementation?
- Is the service name implemented on the server/client like the one set for GameAnvilConfig.json?

## Perform In-Game Chat

An environment that can communicate between clients via the server has now been configured. Here we will implement a simple example of how data generated by the client can be received by a remote client. Learn how to get chat history using a previously implemented protocol within the example project. With this example limit, the MessageRequest class is used when transferring communication data from client to server. When sending communication data from the server to the client, the MessageResponse or MessageBroadcast class is used.

(Message classes are not available when a project is created using a project template.) Download the project created for the tutorial to allow the internal BasicProtocol.java to be included in the project. A process of registering a protocol before the server runs is also required. Configuration for the protocol on the server side is pre-prepared for the tutorial project, so if the tutorial project is still in use, you don't have to proceed.)

### Perform Client Side Transfer

First, write the code to send the message from the client to the server. Once connected to the server as a logged-in user, you can use the server's features through the user agent. Here you use the Send feature of the user agent to send packets to the room. To send a message, the packet containing the content to be sent must be delivered as an argument.

Add a message sending script to the GameManager script in the Unity project. When the SendMessage method is called, the message entered by the user and the packet containing his user ID are sent to the server through userAgent in the input field on the screen. And add the Update function to the implementation so that each time you press Enter, the SendMessage method runs. The example code below shows the implementation of the SendMessage method:

```c#
using System;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil.Defines;
using Protocol;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public InputField ChatInputText;

    private ConnectHandler connectHandler;

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Return) && !string.IsNullOrEmpty(ChatInputText.text))
        {
            SendMessage();
        }
    }

    public void SendMessage()
    {
        MessageRequest messageRequest = new MessageRequest();
        messageRequest.Message = "\n[" + connectHandler.accountId + "]:" + ChatInputText.text;
        connectHandler.getUser().SendUser(messageRequest);
        ChatInputText.text = string.Empty;
    }
}
```

This has made it possible to send packets from the client to the server. But even if you send the packet to the server right now, there will be no response from the server. The reason for this is that the server side did not define how to analyze the content and what to do when the packet was received. The following chapter will cover implementation on the server side.

### Perform Server Side Response

First, register a protocol before running the server to allow the chat protocol to be used on the server.

```java
@GameAnvilApplication
public class Main {
    public static void main(String[] args) {
        final var server = GameAnvilServer.getInstance();

        server.addProtocol(BasicProtocol.class);

        server.run(Main.class, args);
    }
}
```

Here you will use the feature to send messages sent by the game user to users in the room that are received by the server. Handlers are used to process messages sent by clients from the server's game room. Handler designates the code bundle used to process a specific protocol. There can be multiple handlers depending on the protocol type, and multiple handlers can be registered in a room. Thus, a room can process multiple protocols.

Handlers are also created by implementing an interface. In the project panel, right-click on the path to where the Main class is located and then select **New > Package** to create a new package called **handler**. Then right-click the **handler** package again and then select **New > GameAnvil RoomMessageHandler**. When the Create File dialog box opens, enter **BasicHandler** in **File name**, **BasicProtocol.MessageRequest** in **Message**, and the class name **BasicRoom** of the IRoom implementation class created before **Room**, and click **OK**.

It creates a BasicHandler class that is registered as a handler used by BasicRoom with @GameAnvilController annotations and @GameRoomMapping annotations and can be used without any separate registration procedures. BasicRoom can now process MessageRequest messages from BasicProtocol through BasicHandler.

What will be executed, i.e., the internal implementation of the execute method, will be explained as follows. In the handling example implementation below, you will send a response message to the sender about the received message and will send additional messages to the user in the room for broadcasting on a room-unit basis. At this time, the client is implemented to synchronize the game based on the room unit broadcast message.

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.BasicProtocol;

@GameAnvilController
public class BasicHandler {

    @GameRoomMapping(value = BasicProtocol.MessageRequest.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, BasicProtocol.MessageRequest request) {
        BasicProtocol.MessageResponse response = BasicProtocol.MessageResponse.newBuilder().setMessage(request.getMessage()).build();
        BasicProtocol.MessageBroadcast broadcast = BasicProtocol.MessageBroadcast.newBuilder().setMessage(request.getMessage()).build();

        BasicRoom room = ctx.getRoom();
        room.broadcast(Packet.makePacket(broadcast));
        ctx.reply(response);
    }
}
```

In the code above, first use the Message value of the MessageRequest object to create new MessageResponse and MessageBroadcast objects. The object of the MessageResponse type sends the packet to the user object that sent it to the room. MesageBroadcast objects are sent to all users inside the room via room.

It allows the server to receive the packets sent by the client, and then process them a little bit and then return them back to the server. At this time, the client must also specify how to process the packets sent from the server.

### Perform Client Side Reception

You must register a pre-handler to process server-side packets on the client as well. Otherwise, when the packet is received, the processing method is judged to be unknown and the contents will be deleted. To detect it and process the content when you receive the send from the server, you can register a protocol-type operator for the packet sent from the server. Enter and register the dealers registration code to process messages in the MessageBroadcast type.

Go back to the Unity project to add the implementation to the Start method in GameManager. First, use the protocol manager to register agreed protocols between the server and the client. BasicProtocol registered here as 0. As for the number, it will be explained in the following.

Register a recipient through a later user agent. As a method type, the protocol type is used. You can use this because the protocol set to be sent by the server is MessageBroadcast. Handler implementation uses simple logic to write the received message to the text prepared on the screen. It's the most basic feature of chat.

```c#
void Start()
{
    connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

    roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

    // Add Code
    connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
    {
        ChatLogText.text += messageBroadcast.Message;
    });
}
```

<br>

### Confirm Message Delivery

After modifying the server, make sure you have a new run, build and play with `cmd+b` or `ctrl+b` in Unity. Create a room in the built game and check the server-side logs. Enter the RoomId of the previously created room after entering play mode in the Unity editor's state and join the room.

In any game process, enter text in the input box and press Enter. Then the same text will appear in other game processes.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/23_unity_chat_test.png)

You’ve learned the message processing process through a simple chat server implementation. Next, we will look at the implementation process of a more practical example.

## Perform Puzzle Game

A puzzle game with single play can be pre-implemented in the game scene. When you enter play mode and drag the puzzle piece to place it near the correct location, it will be balanced to the exact location on the grid. In this chapter, we will try to make this game to be a multiplayer game.

Messages sent between users in the same room can contain any value other than MessageRequest, MessageResponse, and MessageBroadcast. In this chapter, we will look at how to directly define a protocol and register it with the server and client.

If messages are only defined based on a predefined protocol, it is possible to send and receive messages between the server and the client. There are many different expressions, including XML, json, and more, but GameAnvil uses Google Protocol Buffers. This is one of the best solutions in terms of speed and reliability.

### Serialize and Deserialize Messages with Google Protocol Buffers

To use the protocol buffer, you must first specify how to define the message. For example, contain a single string in the specifications of MessageRequet. After that, compile the protocol specifications to convert them to a file in the desired language. Later, it can be used as a message protocol for the packet similar to the way you used MessageRequest.

If you use the tutorial sample project as it is, the Puzzle protocol creation process below has already been completed, so please see how to understand the concept. We will start creating a message protocol to synchronize the puzzle location. Add the Puzzle.proto file to the src/main/proto path of the server project and enter the protocol details as shown below:

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}
```

The PuzzlePosition protocol is configured with the unique number information (Index) and location information (PositionX and PositionY) of each puzzle piece to display the latest location of the puzzle piece. The protocol definition is now completed. In the following, we will compile these protocol definitions into the classes for each development language.

The protoc execution file is in the project highest level path. Go to the project and use the following command line to compile for Java and C#. When you run a command line, you will succeed if no results messages appear.

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

You can now check that a new Puzzle.java class has been created in the src/main/java path.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/24_puzzle_proto_java.png)

C# class files are transferred to the Asset/Protocol path of the Unity project using programs such as Finder and File Explorer.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/25_puzzle_proto_csharp.png)

Creating message protocols to synchronize puzzle positions has been completed.

### Register Protocol

If you have defined a protocol and compiled it successfully, then you must register the protocol class on both the server and the client. Register a protocol from the Main method of the GameAnvil server as shown below.

```java
@GameAnvilApplication
public class Main {
    public static void main(String[] args) {
        final var server = GameAnvilServer.getInstance();

        server.addProtocol(BasicProtocol.class);
        // Add Code
        server.addProtocol(Puzzle.class);

        server.run(Main.class, args);
    }
}
```

The Unity project adds a protocol as follows to the getConnector() method of the ConnectHandler script:

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();

        GameAnvilProtocolManager.RegisterProtocol(BasicProtocolReflection.Descriptor);
        // Add Code
        GameAnvilProtocolManager.RegisterProtocol(PuzzleReflection.Descriptor);
    }

    return connector;
}
```

<br>

### Perform Client Side Transfer

Now you've finished with both defining and registering protocols for the game. Now implement the feature to actually send messages based on these protocols. First implement the parts that transfer data from the client side. Transfer the location to the server while dragging a puzzle piece.

Go to the Unity project, open the Puzzle.cs file and add the code as shown below. Create a message object that contains unique location information for the puzzle while dragging. And it will be sent to the server via GameAnvilUser. Only the protocol for the message is different, but it is the same method as the previously tried MessageRequest.

```c#
using UnityEngine;
using UnityEngine.EventSystems;
using Protocol;

public class Puzzle : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public int index;

    public Transform puzzleHolder;
    public float tolerance;


    private ConnectHandler connectHandler;

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();
    }

    public void OnBeginDrag(PointerEventData eventData)
    {
        transform.position = eventData.position;
    }

    public void OnDrag(PointerEventData eventData)
    {
        transform.position = eventData.position;
        SendPuzzlePosition(transform.position, false);
    }

    public void OnEndDrag(PointerEventData eventData)
    {
        FixPosition();
        SendPuzzlePosition(transform.position, true);
    }

    public void FixPosition()
    {
        if (Vector2.Distance(transform.position, puzzleHolder.position) < tolerance)
        {
            transform.position = puzzleHolder.position;
        }
    }

    public void SendPuzzlePosition(Vector2 puzzlePosition, bool onEndDrag)
    {
        PuzzlePosition position = new PuzzlePosition();
        position.Index = index;
        position.PositionX = (int)puzzlePosition.x;
        position.PositionY = (int)puzzlePosition.y;
        position.OnEndDrag = onEndDrag;

        connectHandler.getUser().SendUser(position);
    }
}
```

<br>

### Perform Server Side Response

On the client side, the location of the puzzle has been constantly sent to the server. Now you need to write down how to locate the puzzle on the server. Process MessageRequest to get the puzzle location back to all the users in the game room, like you did with MessageResponse and MessageBroadcast.

Return to the server project and create a PuzzlePositionHandler class file using the **GameAnvil RoomMessageHandler** file template. And use the broadcast method to get the received packet to all the users in the room.

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionHandler {

    @GameRoomMapping(value = Puzzle.PuzzlePosition.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePosition request) {
        BasicRoom room = ctx.getRoom();
        room.broadcast(Packet.makePacket(request)); // 방 전체 유저에게 메시지를 전송
    }
}

```

PuzzlePositionHandler is also registered as a handler used by BasicRoom with @GameAnvilController annotations and @GameRoomMapping annotations. Now BasicRoom can process PuzzlePosition messages in the Puzzle protocol through the PuzzlePositionHandler.


<br>

### Perform Client Side Reception

You had to register a pre-handler to process MessageBroadcasts sent from the server earlier. In order to create a manipulator for handling puzzle positions in the same way this time, enter the manipulator registration code for handling PuzzlePosition messages in the Start method of GameManager. When the message is received, the dealers locate the puzzle object and move it to the received location on the server.

```c#
public class GameManager : MonoBehaviour{

    ...(omitted)...

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
        {
            ChatLogText.text += messageBroadcast.Message;
        });

        // Add
        connectHandler.getUser().SetMessageCallback<PuzzlePosition>((_, _, puzzlePosition) =>
        {
            Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);

            if (puzzlePosition.OnEndDrag)
            {
                puzzle.FixPosition();
            }
        });
    }

    ...(omitted)...
}
```

<br>

### Confirm Puzzle Location Synchronization

Build and play with `cmd+b` or `ctrl+b in` Unity. After creating a room in the built game, run Unity play mode and enter the RoomId of the created room to join the room. Now, if you drag a puzzle piece to the location, you can see that the location of the puzzle piece is synchronized to reflect to the remote client.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/26_unity_puzzle_position_playmode.gif)

<br>

### Handle Late Joiner

Let’s think if a new user enters the room after the random puzzle piece location changes during the game. At this time, the puzzle status between an existing user and a new user is different. The newly entered user has an initial puzzle status, so the existing user and puzzle status must be synchronized. To address this, please modify the server-side logic. The server now retains all the location information in the puzzle, and when a new user enters it, it fixes the logic to use that information to synchronize.

Add a puzzlePositions map to BasicRoom. This map manages the location information for each puzzle piece.

```java
public class BasicRoom implements IRoom<BasicUser> {

    private static final Logger logger = getLogger(BasicRoom.class);

    private IRoomContext roomContext;

    // Add Code
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();

    ...(omitted)...
}
```

Now modify the PuzzlePositionHandler code to store the puzzle location in each room.

```Java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionHandler {

    @GameRoomMapping(value = Puzzle.PuzzlePosition.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePosition request) {
        BasicRoom room = ctx.getRoom();

        // Add Code
        room.puzzlePositions.put(request.getIndex(), request);
        room.broadcast(Packet.makePacket(request)); // 방 전체 유저에게 메시지를 전송
    }
}

```

Modify the onJoinRoom of BasicRoom to send the location information of the puzzle stored on the server so that the new user can receive the location information of the puzzle stored when they enter the room.

```java
public class BasicRoom implements IRoom<BasicUser> {

    ...(omitted)...

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        // Add Code
        // With the code written temporarily, the problem will be resolved and the code will be removed in the following process:
        for(Puzzle.PuzzlePosition puzzlePosition : puzzlePositions.values()) {
            broadcast(Packet.makePacket(puzzlePosition));
        }
        return isSuccess;
    }

    ...(생략)...
}
```

Modify it and try again. Modify the puzzle location from one client first, then join the game room from the other client to see if the puzzle location is synchronized properly. You can see that it is still not working unlike intended.

This is because a thin move occurs in the client after the onJoinRoom call. The problem has occurred because the synchronization message is sent before the listener registration is completed. To resolve this problem, you must synchronize the location information of the puzzle after the thin move from the client to the listener registration.

This issue is not resolved right now and will be discussed in more detail. These problems will be reflected in the puzzle mix to be explained below. Therefore, start by mixing the puzzle and then retry the modifications to resolve it.

<br>

## Perform Puzzle Mix

We will implement a logic that blends puzzle positions with random positions. If the client requests to mix the puzzle location, the server determines the new location after mixing the puzzle. And the basic idea is to return the changed location information to the client.

### Register Protocol

First, we will create a protocol to request a puzzle mix. Go to the server project to add a protocol specification to the Puzzle.proto file. This protocol has no field because it has no specific information to send from client to server. This is also useful enough as a protocol.

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}

message ScatterPuzzle { } // Puzzle Mix Request Protocol
```

If you modify a protocol buffer file, you must also compile it again.

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

Move the created C# class back to the Asset/Protocol folder of the Unity project using programs such as Finder or File Explorer.

<br>

### Implement Client Side

Go to the unit project and write the code for the mix request in GameManager.cs as shown below. When the Scatter method is called, messages of the new ScatterPuzzle type are sent to the game room through GameAnvilUser.

```c#
public class GameManager : Monobehaviour {

    ...(omitted)...

    public void Scatter()
    {
        connectHandler.getUser().SendUser(new ScatterPuzzle());
    }

    ...(omitted)...
}
```

Click the **Scatter Puzzle Button** in the **Hierarchy** panel. Add an item to the **OnClick** listener from the **Button** component of **Inspector**, drag the GameManager component to register and select the Scatter method from the dropdown.

### Server Side Implementation

Handler uses the previously used method to process when a mix request is received. Create a ScatterPuzzleHandler class through the **GameAnvil RoomMessageHandler** file template. After randomly setting the positions of each of the 16 puzzles, send a message of the PuzzlePositon type. The puzzlePositions map of the server is also updated with new location information.

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

@GameAnvilController
public class ScatterPuzzleHandler {
    private static final int mapSize = 400;

    private static class Point {
        public int x;
        public int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    @GameRoomMapping(value = Puzzle.ScatterPuzzle.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.ScatterPuzzle request) {
        BasicRoom room = ctx.getRoom();
        room.puzzlePositions = new HashMap<>();
        List<ScatterPuzzleHandler.Point> random = IntStream.rangeClosed(-4, 4).boxed()
                .map(i -> new Point(i * mapSize / 4, i % 2 == 0 ? mapSize : -mapSize))
                .collect(Collectors.toList());
        random = Stream.concat(random.stream(), random.stream().map(p -> new Point(p.y, p.x))).collect(Collectors.toList());
        Collections.shuffle(random);

        for (int i = 0; i < 16; i++) {
            ScatterPuzzleHandler.Point point = random.get(i);
            int x = 1300 + point.x;
            int y = 550 + point.y;

            Puzzle.PuzzlePosition puzzlePosition = Puzzle.PuzzlePosition.newBuilder().setIndex(i).setPositionX(x).setPositionY(y).build();
            room.puzzlePositions.put(i, puzzlePosition);
            room.broadcast(Packet.makePacket(puzzlePosition));
        }
    }
}
```

Sending on the client side and responding on the server side are now completed.

### Confirm Puzzle Mix

Enter play mode from the Unity editor. Click the Scatter Puzzle button to see if the puzzle mixing feature works properly.

## Better Late Joiner Process

A synchronization problem occurred while handling late joiners in front of the user. The reason for this was that the user thought the time when they entered the room was appropriate as the time when the location of the puzzle fragment was synchronized. But the puzzle game we're implementing will have a fine move when the user enters the room.

Therefore, we have to think of it as two points in time when registering readers and calling for onJoinRoom callbacks. This may be without problems depending on the implementation of the game client, or may cause problems. Since the thin move begins after the onJoinRoom callback call, the client wishes to request a puzzle location synchronization directly to the server immediately after the thin move is complete.

### Register Protocol

Go to the server project and add the protocol to Puzzle.proto to request puzzle location synchronization.

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}

message ScatterPuzzle {}

message PuzzlePositionReq {} // Request Puzzle Synchronization
```

Transfer the created C# class to the Unity project after recompiling the protocol.

<br>

### Implement Client Side

To enable the server to request the puzzle location immediately after the thin move, you must modify the Start function running immediately after the thin move with GameScence. Add the code to send the newly created PuzzlePositionReq protocol message to the GameManager Start method as follows.

```c#
public class GameManager : MonoBehaviour
{
    ...(omitted)...

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
        {
            ChatLogText.text += messageBroadcast.Message;
        });

        connectHandler.getUser().SetMessageCallback<PuzzlePosition>((_, _, puzzlePosition) =>
        {
            Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);

            if (puzzlePosition.OnEndDrag)
            {
                puzzle.FixPosition();
            }
        });

        // Add Code
        connectHandler.getUser().SendUser(new PuzzlePositionReq());
    }

  	...(omitted)...
}
```

<br>

### Server Side Implementation

Remove the puzzle location sending code that has been incorrectly implemented to onJoinRoom, and create a new PuzzlePositionReqHandler class for the handling of the Puzzle.PuzzlePositionReq message.

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionReqHandler {
    @GameRoomMapping(value = Puzzle.PuzzlePositionReq.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePositionReq request) {
        BasicRoom room = ctx.getRoom();
        room.puzzlePositions.values().stream().forEach(puzzlePosition -> { room.broadcast(Packet.makePacket(puzzlePosition)); });
    }
}

```

<br>

### Confirm Late Joiner Processing

Build and play with `cmd+b` or `ctrl+b in` Unity. Now create a room in the built game and run Puzzle Mix. In that state, enter the Unity editor's play mode and join the room to confirm that the puzzle location is synchronized.

## Perform User MatchMaking

### Server Side Implementation

User matchmaking collects users' matchmaking requests to allow users of the same level to start the game in the same room according to the appropriate criteria. You can implement various elements, such as win points or scores, to adequately separate and match users. It implements a logic that matches two users in one game.

In the project panel, right-click on the path to where the Main class is located and then select **New > Package** to create a new package named **match**. Then right-click the **match** package again and then select **New > GameAnvil UserMatchInfo**. When the Create File dialog box opens, enter **as BasicUserMatchInfo** in **File name** and click **OK**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/27_create_user_match_info.png)

This class contains information about the user to be used for matching. If there are elements to be used for matches, you can add it here. In this example, we will use only the default methods without adding any elements. One caution is to ensure that the getId() method must be implemented to return the requested user ID. And because the party matchmaking feature is not enabled, set it to return 0.

```java
package org.example.match;

import com.nhn.gameanvil.node.match.usermatch.AbstractUserMatchInfo;

public class BasicUserMatchInfo extends AbstractUserMatchInfo implements Comparable<BasicUserMatchInfo> {

    private int userId;

    public BasicUserMatchInfo(int userId) {
        this.userId = userId;
    }

    @Override
    public int getId() {
        return userId;
    }

    @Override
    public int getPartySize() {
        return 0;
    }

    @Override
    public int compareTo(BasicUserMatchInfo o) {
        return 0;
    }
}

```

These UserMatchInfos are created and used by the client while implementing onMatchUser callbacks on the server's game user when the client requests user matchmaking. GameAnvil provides a basic user match maker. The onMatchUser below uses the default user matching of these engines through the matchUser API.

```java
public class BasicUser implements IUser {

    ...(omitted)...

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;

        try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(userContext.getUserId());
            return userContext.matchUser(matchingGroup, roomType, term);
        } catch (TimeoutException | NodeNotFoundException | GameAnvilException e) {
            logger.error("BasicUser::onMatchUser", e);
        }

        return isSuccess;
    }

    ...(omitted)...
}
```

Once you are basic prepared to use user matchmaking, now write a matchmaker to perform actual matchmaking.

Right-click on the **match** package in the project panel and then select **New > GameAnvil UserMatchMaker**. When the Create File dialog box opens, enter **BasicUserMatchMaker** in **File name**, **Room** in **BasicRoom**, and **User Match Info** in **BasicUserMatchInfo**, and click **OK**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/28_create_user_match_maker.png)

The creator calls the creator of the parent class and sends the number of matches and the match request validity time with an argument. The match request is automatically canceled after the valid period. And the match method that performs actual matches is called internally by the engine once every second.

getMatchRequests takes the minimum number of people for matching with the argument to view the entire current matching pool to create as optimal matches as possible. If you have a lot of matching requests, you can create 100 or 1,000 matches at a time by getMatchRequests. When the match is successful, the UserMatchInfo list of the matched users is returned. If you pass this list to the matchSingles API provided by the engine, you can create and move rooms for the users in that list.

Return null if there are not enough requests accumulating for the match, or if there are no targets matching the conditions. In this case, the same match search is performed again in the match call after 1 second.

```java
package org.example.match;

import com.nhn.gameanvil.game.GameAnvilUserMatchMaker;
import com.nhn.gameanvil.node.match.AbstractUserMatchMaker;
import org.example.game.BasicRoom;

import java.util.List;

@GameAnvilUserMatchMaker(loadClass = BasicRoom.class)
public class BasicUserMatchMaker extends AbstractUserMatchMaker<BasicUserMatchInfo> {

    public BasicUserMatchMaker() {
        super(2, 5000);
    }

    private int matchSize = 2;

    @Override
    public void onMatch() {
        List<BasicUserMatchInfo> matchRequests = getMatchRequests(matchSize);

        if (matchRequests == null) {
            return;
        }

        int matchingAmount = matchSingles(matchRequests);
    }

    @Override
    public boolean onRefill(BasicUserMatchInfo matchReq) {
        return false;
    }
}
```

<br>

### Implement Client Side

Since all matching logic is implemented on the server, the client can only send a request when matching is needed. Add the MatchUser method to the ConnectHandler. And add the code to move the scenes to the point when the matches are finished.

```c#
public class ConnectHandler : MonoBehaviour
{
	...(omitted)...

    public void MatchUser()
    {
        try
        {
            getUser().OnMatchUserDone += OnMatchUserDone;
            var result = getUser().MatchUserStart("ROOM_TYPE_BASIC", string.Empty);
            Debug.Log("MATCHING_USER...");
            logText.text = "MATCHING_USER...";
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
            throw;
        }
    }

    private void OnMatchUserDone(GameAnvilUser user, ResultCodeMatchUserDone resultCode, MatchResult matchResult)
    {
        getUser().OnMatchUserDone -= OnMatchUserDone;
        Debug.Log("Room Id : " + matchResult.RoomId);
        this.roomId = matchResult.RoomId;
        SceneManager.LoadScene("GameScene");
    }

	...(omitted)...
}
```

Drag the ConnectHandler component to the OnClick listener on the MatchUser button in the tone to register and select the MatchUser method in the dropdown.

### User Matchmaking Test

Build and play with `cmd+b` or `ctrl+b in` Unity. In that state, enters play mode in the Unity editor. Press the User Match Making button on both sides to confirm that the match is completed and attached to the same room number.

## Perform Room Matchmaking

Room matchmaking is a feature that lets you automatically fit any of the rooms managed by a matchmaker into the rooms that best meet the user’s needs. So where user matchmaking is a feature to match users and users, room matchmaking is a feature to match users and rooms. At this time, you can match a user with a room in various conditions depending on the implementation method. Here, matches are realized by entering the room with the least number of people that have not yet had a garden.

First, you need a class to actually perform the matches. And the room matchmaker must have one instance containing information to manage the room for each room. At last, each time a user applies for matches, an instance of the application contains the user's requirements. So let me write three new classes.

Modify some of the existing logic additionally. Room matchmaking is performed only for the rooms applied for room matchmaking and not for all rooms. At the time of creation, the code to apply for room matchmaking is therefore added.

### Server Side Implementation

First implement the class to specify the matchmaking request. Create a BasicRoomMatchForm class.

Right-click on the **match** package in the project panel and then select **New > GameAnvil RoomMatchForm**. When the Create File dialog box opens, enter **BasicRoomMatchForm** in **File name** and click **OK**.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/29_create_room_match_form.png)

Every time a user makes a matchmaking request, the BasicRoomMatchForm object is created and used.

```java
package org.example.match;

import com.nhn.gameanvil.node.match.roommatch.AbstractRoomMatchInfo;

public class BasicRoomMatchInfo extends AbstractRoomMatchInfo {
    private static final int MAX_ENTRY_USER = 4;

    public BasicRoomMatchInfo(int roomId) {
        super(roomId, MAX_ENTRY_USER);
    }
}
```

The following implements a class that expresses the information of the room to be matched. The BasicRoomMatchInfo class is created as follows.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/30_create_room_match_info.png)

At this time, the maximum garden of the room is four people specified in the MAX_ENTRY_USER field. You must pass these maximum gardens and roomId to the AbstractRoomMatchInfo creator you inherited.

```java
package org.example.match;

import com.nhn.gameanvil.game.GameAnvilRoomMatchMaker;
import com.nhn.gameanvil.node.match.AbstractRoomMatchMaker;
import org.example.game.BasicRoom;

@GameAnvilRoomMatchMaker(loadClass = BasicRoom.class)
public class BasicRoomMatchMaker extends AbstractRoomMatchMaker<BasicRoomMatchForm, BasicRoomMatchInfo> {

    @Override
    public boolean onMatch(BasicRoomMatchForm roomMatchForm, BasicRoomMatchInfo roomMatchInfo, Object... args) {
        return false;
    }

    @Override
    public int compare(BasicRoomMatchInfo o1, BasicRoomMatchInfo o2) {
        return 0;
    }
}
```

Next, create a room match maker to actually process the room matchmaking.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/31_create_room_match_maker.png)

The BasicRoomMatchMaker is created as follows:

```java
public class BasicUser implements IUser {

    ...(omitted)...

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();
        try {
            return userContext.matchRoom(matchingGroup, roomType, gameRoomMatchForm);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicUser::onMatchRoom", e);
            return RoomMatchResult.FAILED;
        }
    }

    ...(omitted)...
}
```

The compare method implements conditions for arranging the rooms in the matching pool. The example was implemented to sort rooms according to the number of people.

We are now almost preparing for room matches. When the client sends a room match request, the BasicUser on the server is called an onMatchRoom callback. Create the BasicRoomMatchForm object discussed earlier in this callback, then send it as an argument to the matchRoom API and call it. The request for room matching sent by the client has been forwarded from here to the matchmaker.

```java
public class BasicUser implements IUser {

    ...(omitted)...

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();
        try {
            return userContext.matchRoom(matchingGroup, roomType, gameRoomMatchForm);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicUser::onMatchRoom", e);
            return RoomMatchResult.FAILED;
        }
    }

    ...(omitted)...
}
```

When the room is created, call the registerRoomMatch API as shown below in the onCreateRoom callback for BasicRoom to make the room a matchmaking target.

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    ...(omitted)...

    @Override
    public boolean onCreateRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        try {
            roomContext.registerRoomMatch(gameRoomMatchInfo, user.getUserContext().getUserId());
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicRoom::onCreateRoom()", e);
        }
        return isSuccess;
    }

    ...(omitted)...
}
```

Room matchmaking information must also be updated each time room information is volatile. Note that the room match maker automatically synchronizes the number of people in the room. Therefore, you can only perform updates for additional information, excluding the number of people.

Here is the code to modify onJoinRoom callbacks to update the matchmaking information when the user participates in the room.

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    ...(omitted)...

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        try {
            roomContext.updateRoomMatch(gameRoomMatchInfo);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicRoom::onJoinRoom()", e);
        }
        return isSuccess;
    }

    ...(omitted)...
}
```

<br>

### Implement Client

Like user matchmaking, the client must only send a request when matching is required. Add the RoomMatchMaking method to the ConnectHandler.

```c#
public class ConnectHandler : MonoBehaviour {

    ...(omitted)...

    public async void MatchRoom()
    {
        try
        {
            var result = await user.MatchRoom(true, true, "ROOM_TYPE_BASIC", "", "");
            if (result.ErrorCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                Debug.Log("Room Id : " + result.Data.RoomId);
                this.roomId = result.Data.RoomId;
                SceneManager.LoadScene("GameScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
            throw;
        }
    }

    ...(omitted)...
}
```

Drag and register the ConnectHandler component to the OnClick listener on the MatchRoom button, and from the drop-down menu, select the MatchRoom method.

<br>

### Room Matchmaking Test

Create a room in the play status after building it with `cmd+b` or `ctrl+b` in Unity. In that state, enters play mode in the Unity editor. In play mode, press the Room Match Making button to see if you are moving to the room you created in build mode.

<br>

## Perform To Leave Room

At last, add the following methods to the GameManager of the Unity client to implement the feature to leave the room.

```c#
public class GameManager : MonoBehaviour
{
    ...(omitted)...

    public async void LeaveRoom()
    {
        try
        {
            var result = await connectHandler.getUser().LeaveRoom();
            Debug.Log(result.ErrorCode);
            if (result.ErrorCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
            {
                Destroy(connectHandler.gameObject);
                SceneManager.LoadScene("ConnectScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            throw;
        }
    }

    ...(omitted)...
}
```

Drag the GameManager component to the OnClick listener of the Leave Room button in the tone to register and select the LeaveRoom method from the drop-down menu.

## End Project

We have implemented a puzzle game with real-time multiplay using GameAnvil and Unity. Numerous key features of GameAnvil have been used in that process. However, GameAnvil supports more extensive and diverse features that are not included in this tutorial. For these features, see the following article. Also included reference sample projects and JavaDoc will help a lot to understand GameAnvil.
