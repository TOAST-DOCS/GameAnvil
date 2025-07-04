## Game > GameAnvil > Advanced Tutorial

### Easily Create game servers with GameAnvil

GameAnvil is a real-time multiplayer game server creation platform. GameAnvil makes it easy to develop and operate game servers and clients. To use GameAnvil, you must start with developing game servers and clients. This document covers the process of developing multiplayer game servers using the basic features of GameAnvil. This document will help you easily learn the basic concepts of GameAnvil, how to organize and implement development projects and more.

GameAnvil not only provides server engines, but also provides connections for connecting clients to servers. This document also covers server development and client development using connectors. As you complete a sample that shows how servers and clients interact, you can get used to the overall flow of developing games using GameAnvil.

Instead of simply listing the server's concepts and APIs, this document will explain them through more specific examples and explain the process of developing a real-world playable multiplayer jigsaw puzzle game in order to help you understand. You can naturally improve your understanding of GameAnvil and multiplayer game development by following the contents of the document one by one.

## Project Configuration

Creating multiplayer games requires a server program that corresponds to the client. In this example, we use Unity and GameAnvil connectors to create client programs, and GameAnvil, the server engine that we introduced prior to creating server programs. First, we create a server program project with GameAnvil and then we create a client program project with Unity and GameAnvil connectors.

### Configure GameAnvil Projects

To apply GameAnvil to the project, you will need to download the GameAnvil library from the Maven repository and create a setup file that is essential to running GameAnvil. Finally, if you write a little boilerplate code, the initial setup of the development will be finished. In this chapter, we aim to complete the initial setup to start the development. Running the actual process and running the server will be covered in the next chapter.

GameAnvil offers IntelliJ templates that replace this set of processes, making it easier to complete initial tasks. You can download the project file template for IntelliJ from the following link. Do not decompress the templates you download.

[Download project templates](https://static.toastoven.net/prod_gameanvil/files/GameAnvil%20Template.zip?disposition=attachment)

Run IntelliJ to apply the downloaded template. Select **Customize** from the left menu on the **Welcome to IntelliJ IDEA** screen and click **Import Settings...** or search **Import Settings...** in the full search bar.

![](https://nhnent.dooray.com/files/3345932278160406768)

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/search_import_settings.png)

In the Finder or File Explorer, navigate to the path from which you downloaded the template and select the compressed file. When the **Select Components to Import** window opens, check and select both **File templates** and **Project templates**. Click **OK** and restart IntelliJ when the import is completed to finish the template application.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/select_import.png)

You can also configure projects using templates, but you can download and use projects that have already been pre-configured. Click the link below to download the project for tutorials, decompress it, and run it with IntelliJ.

[Download Project for Tutorial](https://static.toastoven.net/prod_gameanvil/files/GameAnvil%20Tutorial%20Project_1213.zip?disposition=attachment)

When you first open a project, you may be prompted to allow the script to run through Gradle, as shown below. Select **TrustProject** to open a complete project.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/trust_project.png)

The server project frame is now configured in IntelliJ. You can see that the code and configuration files have been created by looking at the Project panel.
- protocol package: a package containing a protocol buffer file compiled with java.
- Main: Class that contains the entry point Main function of the program.
- BasicProtocol.proto: A protocol file created using a protocol buffer.
- GameAnvilConfig.json: A file that records the server configuration information required to drive GameAnvil. You can modify it to fit your server implementation.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/gameanvil_projectview_init.png)

First, check the JDK settings. Click Select **File>Project Structure** from the top left menu. For Mac users, the `command+;` shortcuts are available.

Check the SDK settings on the Project tab. If there is no SDK set, download and set the desired version of JDK through `Add SDK>Download JDK`. The Language level is set to SDK default. Next, on the Modules tab, set the Language level to Project default. On the **Project** tab, check the SDK settings. If there is no SDK set, click **Add SDK>Download JDK** to download and set the desired version of JDK. **Language level**Set to **SDK default**. Next, on the **Modules** tab, set **Language level**to **Project default**.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/project_structure_1213.png)

**In the Settings menu, check the JVM used by the gradle. Set it to the same gradle version as the project SDK.**

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gradle_sdk_config_1213.png)

Project preparation is almost complete, but several settings are required for execution. In this case, we first create a client project, then complete the server setup and run it.
<br>

### Unity Project Configuration

Run the Unity Hub. Click **NEW** at the top right to open the Create a New Project window

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unityhub.png)

Select **2D** from the template list, verify the project name and location, and click **CREATE** to complete the project creation.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unityhub_new_project.png)

Download the GameAnvil connector from the link below. Connector is a package that provides the client API needed for communication with the GameAnvil server to help you implement the client with simple code.

[[ gameanvil-connector.unitypackage ]](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector.unitypackage)

Just like creating a server project, download and use boilerplate code created for quick progress. Download the package from the link below that includes the code for tutorials, image sources and more.

[[ GameAnvilTutorial.unitypacakge ]\](https://static.toastoven.net/prod_gameanvil/files/GameAnvil Tutorial.unitypackage)

Drag the downloaded package file to the Unity project or open the **Asset > Import Package > Custom Package...** menu to select a package file in the Finder or File Explorer.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_import_custom_package.png)

In the Import Unity Package dialog box, select all the check boxes in the list and click **Import**.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_import_custom_package_window.png)

Finally, for convenient testing, select the Player tab in the Project Settings window and set the project default window width and height to 1920 and 1080, respectively, in the Resolution and Presentation items.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_project_setting_resolution.png)

Client project setup is complete. 

## Start and Connect Servers

Go back to IntelliJ and run the server project

### Run GameAnvil Servers

When the execution setting is complete, click the green triangle icon to the left of the main() function of the main class to select **Main.main() Execute**. Once this has been initially done, clicking on the green triangle Run icon in the upper right corner of the IntelliJ will also launch the server.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run1_1213.png)

In build.gradle, JVM options are preconfigured for your convenience. To leverage these settings to run the server, right-click **Task > others > runMain** in the Gradle window of IntelliJ and click on **GameAnvilTutorial Execute **

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic-tutorial/gameanvil_run2_1213.png)


If the server is running normally, a number of logs related to the server's driving status will be output. The GameAnvil server consists of nodes that are event loops that divide the roles to perform into several. Each node needs time to prepare to execute code. When each node is ready, it outputs an onReady log. If a GatewayNode is ready during which the client plays a direct role in accessing the server and the onReady log is output from that node, the GameAnvil server is now accessible at any time.

<br>

### Create a Connect Handler

Now we go to the Unity project and write the code to get to the GameAnvil server. You need to create a connector object before you can connect to the server.

Prepare the scene by double-clicking Connect.scene in the Scene folder in the asset panel. Select the ConnectHandler game object in the hierarchy panel to further modify the implementation of the script ConnectHandler.cs file for the applied component.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using GameAnvil.User;
using UnityEngine.SceneManagement;
using Protocol;

public class ConnectHandler : MonoBehaviour
{
    [SerializeField]
    GameAnvil.Connector.Config config;

    private GameAnvil.Connector connector;
    private static ConnectHandler instance;

    [Header("Object Reference")]
    public Text serverInfoText;
    public Text clientInfoText;
    public Text logText;

    public GameObject popupCanvas;
    public InputField roomIdInput;

    public GameAnvil.Connector GetConnector()
    { getConnector()
        return connector;
    }

    public static ConnectHandler GetInstance()
    { return instance
        return instance;
    }

    private void Awake()
    { return instance
        if (instance != null)
        }
            DestroyImmediate(this.gameObject);
        }
        instance = this;
        DontDestroyOnLoad(this.gameObject);
        connector = new GameAnvil.Connector(config);
    }

    } private void OnDisable()
    { }
        if (connector.IsConnected())
        { }
            connector.CloseSocket();
        }
    }

    private void OnApplicationPause(bool pause)
    { }
        if (pause)
        { }
            // Suspend the server's clientStateCheck function for the given amount of time.
            // After this time, the clientStateCheck function may be activated and the connection may be disconnected.
            connector.GetConnectionAgent().PauseClientStateCheck(10 * 60);

            connector.Update();
        }
        else
        }
            connector.Update();

            // Reactivate the server's clientStateCheck functionality.
            connector.GetConnectionAgent().ResumeClientStateCheck();
        }
    }
}
```

Create a new Connector object in the Awake() function and store it in the connector field. Note that you must periodically call the connector's Update() method to process messages to and from the server. Write the Update() function, a function called by Unity, to call the connector's Update() every frame.

The GameAnvil server and the client should periodically send and receive status check packets. Use the PauseClientStateCheck() method to stop sending and receiving status check packets when the client is paused and etc. To resume status check, call the ResumeClientStateCheck() method. The OnApplicationPause() function detects the client's pause and calls the appropriate method.
<br>

### Add Server Connection Code

The window used to utilize multiple functions in the connector is called the Agent. Agents are divided into ConnectionAgent and UserAgent, depending on the functions they perform. Server connection and authentication features are accessible using the ConnectionAgent. Add the Start() method and the Connect() method to the ConnectionHandler code, as shown below, so that the connection is made when Unity starts.

```c#
// Client-side

[SerializeField]
GameAnvil.Connector.Config config;

public GameAnvil.Connector connector = null;
private ConnectionAgent connectionAgent;

public string ip = "127.0.0.1";
public int port = 11200;


[Header("Object Reference")]
public Text serverInfoText;
public Text clientInfoText;
public Text logText;

void Start() {
    // Attempt to connect directly at startup time.
    Connect();
  
    // Print connection information to the screen.
    serverInfoText.text = "Server information:" + ip + ":" + port;
}

public void Connect(){
  connectionAgent = connector.GetConnectionAgent();
	connectionAgent.Connect(ip, port,
            (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
                Debug.Log(result);
            
              	// Display the result on the game screen.
                if (result == ResultCodeConnect.CONNECT_FAIL) {
                    logText.text = "Connection information: connection failed";
                } else {
                    logText.text = "Connection Info: Connection Success";
                }
            }
        );
}
```

The Connect() function calls the Connect method by referring to the ConnectionAgent object through the connector. This attempts to connect to the server with the ip and port information given by the factor. The third factor is to register the callback that receives the connection attempt result.

The callback receives the ConnectionAgent object that made the connection and the connection attempt result code as parameters. It outputs the connection attempt result code to the console and sets it to display different phrases depending on the result so that it can also be seen on the screen. In the example code, `Connection information: Connection failure` is displayed on the screen if the connection fails.

Just as the server is ready to accept the connection, the client is fully ready to connect to the server.

<br>

### Verify Server Connection

You can now enter play mode from the Unity client and check that the result code is properly output on the console. You can check the connection success message along with the connection information of the IP and Port on the text of the game screen. Clients who have accessed the game server can now send and receive messages through the server.

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_connection_test.png)

<br>

## Create Room and User

The client that connects to the server is called **game user**. Clients that connect to the server can log in as one or more game users on the server (this example deals with logging in as one user). Game users can communicate with other users in the same room by belonging to one **game room **. In other words, if users want to exchange game-related messages with different users, they must be in the same room.

GameAnvil has prepared basic implementations of game users and game rooms in advance, so you can expand the engine's class and easily complete the structure of the game users and rooms using the API on the connector. The engine will cover how to define game users and rooms, and the connector will cover examples of using APIs that request room creation or participation.

### Server Operations

Servers define the functions of game users and game rooms as classes. When you attach an annotation provided by GameAnvil to a defined class, it automatically scans the class at the time of game execution and creates game users and room objects at the right time. First, let's define game users. Open the Create New File context menu with `Ctrl + N` or `Cmd + N` and select GameUser. (If the file template was not installed properly earlier, the GameUser entry will not exist. Please install the file template and proceed.)

When prompted in the dialog box, fill in the fields as shown above. Each value should be specified as a pre-consulted value between the client and the server. If you have entered another value you want in the field below, remember to enter the same value when you log in to create users on the client. Since we are creating a game server that uses a single service and a single user type here, GameAnvil's concept of multiple services and user types is not covered in detail in this document.

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gameuser.png)



Inheriting the BaseUser class provided by GameAnvil creates a file with the default code that implements the game user. To implement the desired features of GameAnvil, you can inherit the BaseUser class and set it to execute the desired code by overriding several callback functions that are called according to the situation. Here is part of the list of supported callback functions.

- onLogin: a callback that runs on a login request. You must forward the acceptance of the login request as a return value. If false is returned, the client will fail to log in. Parameters allow you to refer to the payloads received from the client, and references to the payloads that will be returned to the client allow you to pass additional information to the client.
- onPostLogin: Callback that runs after login.
- onReLogin: if you receive another login request for a user who has already logged in, this callback can be handled separately.
- onDisconnect: a callback that is called when the user information registered on the server is disconnected due to a client request logout or forced logout by the server.

Other callback functions are not involved in the login process, so you don't need to know exactly what all callbacks mean immediately. 

The onLogin callback method implements actions that must be performed during the login process. In this example, without any implementation of the login, return true to ensure that the login is successful.

```java

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.annotation.UserType;
import com.nhn.gameanvil.node.game.BaseUser;
import com.nhn.gameanvil.node.game.data.RoomMatchResult;
import com.nhn.gameanvil.packet.Payload;
import com.nhn.gameanvil.packet.message.MessageDispatcher;
import com.nhn.gameanvil.serializer.TransferPack;

@ServiceName("BASIC_SERVICE")
@UserType("BASIC_USER")
public class BasicUser extends BaseUser {

    private static final MessageDispatcher<BasicUser> messageDispatcher = new MessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
    }

    @Override
    public MessageDispatcher<BasicUser> getMessageDispatcher() {
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
    public boolean onReLogin(Payload payload, Payload sessionPayload, Payload outPayload) throws SuspendExecution {
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

You have completed the user implementation that able to login. Now you will implement the game room. Just like how you create users, you will use file templates to create subclasses of the BaseRoom class. At this time, you must enter the same service name as the service name you entered when creating the game user. Remember the room type information because it is necessary when the client requests to create a room in the future. In the User Class field, enter the class name of the BaseUser implementation class you created in the previous step.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gameroom.png"/>

Clicking **OK** automatically creates the corresponding callback method. Here we briefly explain the API for the client to explain the callbacks that GameAnvil supports. The client can call the room-related API to communicate with other users after logging in with the connector. It supports actions such as creating a room, joining a room created by another user or leaving the room.

- OnCreateRoom: runs when a user requests to create a room. It provides additional information delivered by the client when creating a room as a parameter along with the user information that requested to create a room, so based on this information, we decide whether or not to allow room creation and then return to implement it.
- OnJoinRoom: runs when you request admission to a room created by another user. Like other callbacks, it provides additional information delivered by the client when entering the room along with the user information that requested admission, so based on this information, we decide whether or not to allow admission to the room, and then return to implement it.
- OnLeaveRoom: runs when a user requests to leave a room. Like other callbacks, it provides additional information delivered by the client when entering the room along with the user information that requested admission, so based on this information, we decide whether to allow admission to the room and then return to implement it.

As the timing of GameAnvil callback's call and the meaning of its return value is consistent, it is easy to infer the behavior of other callbacks. GameAnvil callback is called immediately before or immediately after any action is requested and processed and the return value determines whether to allow the server to process the request. Whether or not the server has fully processed the request is forwarded to the client via the engine in packet form.

Here, we use these callbacks to implement a feature of storing information about users entering the room. When a user creates a room or enters the room, the user who delivered the request is stored in the field users map of the room object. When a user leaves the room, delete the user from the users’ map. Complete the game room class implementation by referring to the code below.

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.BaseRoom;
import com.nhn.gameanvil.node.game.BaseUser;
import com.nhn.gameanvil.node.game.RoomMessageDispatcher;
import com.nhn.gameanvil.packet.Payload;

import java.util.HashMap;
import java.util.Map;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoom extends BaseRoom<BasicUser> {
    private static RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>();
    private Map<Integer, BaseUser> users = new HashMap<>();

    @Override
    public final RoomMessageDispatcher<BasicRoom, BasicUser> getMessageDispatcher() {
        return dispatcher;
    }

    @Override
    public boolean onCreateRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(user.getUserId(), user);
        return true;
    }

    @Override
    public boolean onJoinRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(user.getUserId(), user);
        return true;
    }

    @Override
    public void onInit() throws SuspendExecution {

    }

    @Override
    public void onDestroy() throws SuspendExecution {

    }

    @Override
    public boolean onLeaveRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.remove(user);
        return true;
    }

    @Override
    public void onLeaveRoom(BasicUser user) throws SuspendExecution {

    }

    @Override
    public void onPostLeaveRoom() throws SuspendExecution {

    }

    @Override
    public void onRejoinRoom(BasicUser user, Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean canTransfer() throws SuspendExecution {
        return false;
    }
}
```

Game users and game rooms are now ready. However, there is no node yet to handle requests for creation and deletion of game users/game rooms. The node responsible for managing game users and game rooms is the GameNode. This is a node that typically performs most of the game logic processing roles that a game server expects to do. It's natural and simple to add nodes to GameAnvil. Just like you defined game users and game rooms, you can inherit a pre-defined class, create a class, and then implement additional functions you want. When you attach an annotation provided by GameAnvil, a node instance will be created and run automatically at the right time for runtime.

When you enter the service name, you must enter the same service name as the previously created game user and the service name used in the game room. If you were writing the tutorial as it was, enter it as follows and click on **OK** to complete.


<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gamenode.png"/>

In order for a node to perform its role, it must first execute a loop. When a node runs, it goes through a series of processes, which require some time. An indicator of whether or not the node is running or at which stage in the execution process is called the state of the node. The state of the node usually changes sequentially in the following order to reach the READY state.

- INIT
- PREPARE
- READY

Nodes that have reached the READY state are now ready to execute user-written logics. If you want to execute a specific code when each preparation step is reached, you can implement the callback method and set it to execute when the engine calls the callback method. There is no code to execute specifically at this time, so use the generated code as it is.


```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.BaseGameNode;
import com.nhn.gameanvil.node.game.data.BaseChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.BaseChannelUserInfo;
import com.nhn.gameanvil.node.game.data.ChannelUpdateType;
import com.nhn.gameanvil.packet.Packet;
import com.nhn.gameanvil.packet.message.MessageDispatcher;
import com.nhn.gameanvil.packet.Payload;

@ServiceName("BASIC_SERVICE")
public class BasicGameNode extends BaseGameNode {

    private static final MessageDispatcher<BasicGameNode> messageDispatcher = new MessageDispatcher<>();

    static {
        // messageDispatcher.registerMsg();
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
    public MessageDispatcher<BasicGameNode> getMessageDispatcher() {
        return messageDispatcher;
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
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, BaseChannelUserInfo baseChannelUserInfo, int i, String s) throws SuspendExecution {

    }

    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, BaseChannelRoomInfo baseChannelRoomInfo, int i) throws SuspendExecution {

    }

    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {

    }

}
```

It is also registered in the settings file to link the last created game user and game room to the GameNode. Add the following to the 'game' entry in the GameAnvilConfig.json file.


```json
// A node that acts as a game lobby. (Includes Gameroom,  users)  
"game": [ 
    { 
      "nodeCnt": 8, 
      "serviceId": 1, 
      "serviceName": "BASIC_SERVICE", 
      "channelIDs": [ 
        "", 
        "", 
        "", 
        "", 
        "", 
        "", 
        "", 
        "" 
      ], 
      "userTimeout": 5000,                // Channel ID to be assigned to each node. (It does not have to be unique.  "" String can be used in duplicate without channel distinction)
      "safeCreateTime": 1000,             // Set for test No normal setting required Default 60 seconds
 
	  "checkClientStateCycle": 10000, 		// Check routine call cycle
	  "demandClientStateTimeout": 10000,	// 
If demandClientStateTimeout elapses 
after receiving the last message, p
erform demandClientState.

	  "clientStateOkDeadline": 10000		// Deadline for responding ClientState request to ClientStateOK (time)
    } 
  ],
```

The features to access the server, log in as a game user, and create a game room is now complete. However, accessing the server does not immediately mean that you can request game-related features (such as creating game users and creating game rooms). Even if you run the server and the client as it is, the client will not be able to use the functions of the game server. Requesting these things from the server requires client authentication after connecting to the server. The next chapter deals with how servers and clients handle authentication.

<br>

## Access authentication

After the client connects to the server, it is necessary to verify the user's identity and go through an authentication process before logging into the game. 
If the game node is responsible for creating game users and game rooms, the gateway node is responsible for the user's access and authentication functions. Just as game nodes are implemented by creating classes, gateway nodes can be implemented in a consistent manner. GameAnvil automatically creates the default gateway, so we will skip the server-side implementation in this chapter.

### Add connection authentication code

Now we go to Unity project and implement it on the client side. The client can request an authentication request through the Connector Connection Agent API, just like a connection request. Since the authentication request can only be established after the connection request, we will modify the code to make an authentication request immediately after the connection is successful. When requesting authentication, we will deliver a callback according to the result of the authentication request to receive the result and output it through the text on the screen and the console so that the connection information can be known.

```c#
// Client-side

[SerializeField]
GameAnvil.Connector.Config config;

public GameAnvil.Connector connector = null;
private ConnectionAgent connectionAgent;

public string ip = "127.0.0.1";
public int port = 11200;

public string deviceId = "";
public string password = "";
public string accountId = "";

public Text connectInfoText;
public Text accountIdText;

void Start() {
    Connect();
  
    serverInfoText.text = "Server Information : " + ip + ":" + port;
}

public void Connect(){
    connectionAgent = connector.GetConnectionAgent();
    connectionAgent.Connect(ip, port,
        (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
            Debug.Log(result);

           if (result == ResultCodeConnect.CONNECT_FAIL) {
                logText.text = "Connection information: connection failed";
            } else {
                logText.text = "Connection Information: Connection Success";
           
             	// If the connection succeeds, try to authenticate immediately.
                Auth();
            }
        }
    );
}
public void Auth(){
    accountId = Random.Range(1000,9999) + "";
  	clientInfoText.text = "Client information: " + accountId;
		connectionAgent.Authenticate(deviceId, accountId, password,
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
                Debug.Log(result);
         
                if (result == ResultCodeAuth.AUTH_SUCCESS) {
                    logText.text = "Authentication Information: Authentication Success";
                } else {
                    logText.text = "Credentials: Authentication failed";
                }
            }
			);
}
```

The client is now set up to not only access the server, but also request the authentication process.

<br>

### Verify Access Authentication

Enter play mode on the Unity client. Make sure that logs are output sequentially on the console in the order of access and authentication. You can check that authentication information and authentication success or failure appear on the game screen. In other words, user authentication is completed at the gateway node of the server at the client's Auth request and the server is now able to log in.

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_authentication_test.png)

<br>

## Login

Now, the last step to do is to log in to the game node and create a game user. Game users are required to communicate with other clients who have accessed the server and if you want to communicate between clients, each client creates a game user and sends and receives messages through the game room.

### Add a Login Code

Clients who have been connected and authenticated can log in to the game node. When you log in to the server, a game user object is created for the client at the game node. Clients can send and receive messages to and from servers or other users through their server-side game user objects. If the authentication is successful, modify the authentication code created earlier as follows to log in immediately.

```c#
// Client-side 
 
public void Auth(){ 
    accountId = Random.Range(1000,9999) + ""; 
    connectionAgent.Authenticate(deviceId, accountId, password, 
     (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => { 
        Debug.Log(result); 
  
        if (result == ResultCodeAuth.AUTH_SUCCESS) { 
            logText.text = "Authentication Information: Authentication Success"; 
               
            Login(); // If the authentication  succeeds, try to login immediately.
        } else { 
            logText.text = "Authentication Information: Authentication Failure"; 
        } 
    }); 
} 
 
public void Login() 
{ 
    connector.CreateUserAgent("BASIC_SERVICE", 1).Login("BASIC_USER", "", null, (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => { 
        Debug.Log(result); 
 
        if (result == ResultCodeLogin.LOGIN_SUCCESS) { 
            logText.text = " Login Information : Login Success "; 
        } else { 
            logText.text = " Login Information : Login Failure"; 
        } 
    }); 
}
```

When you request a login, the callback method that you forwarded together should output the results of the login request as a log.

<br>

### Check out Login

Check that you are logged in successfully through Unity test mode.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_login_test.png"/>

<br>

## Create and Join a Room

### Client Tasks

Unity projects add a method to the ConnectHandler code that requests the creation of a room; note that RoomType, the first factor in the userAgent.CreateRoom method, must be the same as the value specified by the server. In general, such protocols as RoomType are used by server and client developers after defining values in advance. If you specified the same value as the tutorial at the server development stage, you can use the service name in the code below. Register the callback that receives the room creation results and output the result code to the console. Also, if the room creation is successful, save the roomId on the client side and write the code to navigate to the game scene.

```c#
using UnityEngine.SceneManagement;

public int roomId;
public UserAgent userAgent;

public void CreateRoom(){
    connector.GetUserAgent("BASIC_SERVICE", 1).CreateRoom("BASIC_ROOM", (UserAgent ua, ResultCodeCreateRoom resultCode, int roomId, string roomName, Payload payload) => {
        Debug.Log(resultCode); 
        if (resultCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
            this.roomId = roomId;
       
            SceneManager.LoadScene("GameScene");
        }
    });
}
```

After you have written the code, click **CreateRoom** in the hierarchy panel. Drag the ConnectHandler component from the **Button** component of the inspector to the **OnClick** listener entry to register the reference, and select **CreateRoom** method from the drop-down menu to enable the method to be executed with on-screen buttons.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_create_room_on_click.png)

If you've done that, you've implemented the ability to create a room through a client. Before we actually run it, we'll test it after implementing the room participation feature. This time, we'll add the room participation code to ConnectHandler.

Verify that the first factor in the userAgent.JoinRoom method matches the predefined RoomType string used in server development. Check the server's response to the request to join the room via callback, and if you successfully join the room, write it to do the same as creating a room. Once you've finished writing the method, link it to the OnClick event on the JoinRoom button.

```c#
private void Update() {
    connector.Update();

    if (popupCanvas && popupCanvas.activeInHierarchy && Input.GetKeyDown(KeyCode.Return)){
        JoinRoom();
    }
}

public void JoinRoom(){
    if (!string.IsNullOrEmpty(roomIdInput.text)){
        connector.GetUserAgent("BASIC_SERVICE", 1).JoinRoom("BASIC_ROOM", int.Parse(roomIdInput.text), null, (UserAgent userAgent, ResultCodeJoinRoom resultCode, int roomId, string roomName, Payload payload)=>{
            Debug.Log(resultCode);
            if ( resultCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS){
                this.roomId = roomId;
            
                SceneManager.LoadScene("GameScene");
            }
        });
    }
}
```

Now, let's go to GameScene and implement what kind of action we will do after participating in the room. First, modify the GameManager code as below to make it possible to display the room ID on the game.

```c#
public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }
}
```

<br>

Now, not only server access, authentication and login features to clients, but also room participation and creation features are all implemented.

### Create Room and Test Participation

Select **File>BuildSetting** from the top toolbar of the Unity project. Add the required scenes to the list of scenes to be built, in turn, as shown below. If the scene order is incorrect, drag the items on the list to adjust the order so that the Connect scene comes to the top.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/add_scene.png)

Build and play `cmd+b` or `ctrl+b` in Unity. The Connect scene loads as a new window opens. Make sure the connection, authentication, and login process is complete. If there is a failure log left in the middle of the process, the failure code can determine the cause. If that is not enough, you should check the server's log and analyze the cause. If the login is successful, press the Create Room button to request the creation of a room. The scene is moved and the ID of the created room is output together. The room ID may differ from the example image below.

In that state, run Unity Play mode and check that login is complete. Click the Join Room button and directly enter the RoomId of the room you created earlier to participate in the room. If you successfully participate in a room, you will be taken to the game scene, and you will be able to see that you have the same room ID on both game screens. The test screen is a success if it appears similar to below.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_room_test.png)

<br>

If it doesn't go well, please check the following again.
- Did you restart the server process after modifying the server implementation?
- Is the service name implemented on the server/client in the same way as set in GameAnvilConfig.json?

## Implement In-game Chat

You now have an environment where clients can communicate through servers. Here we provide a simple example of how a remote client can receive client-generated data. Example Learn how to send and receive chat history using a pre-implemented protocol within a project. Use the MessageRequest class to transfer communication data from a client to a server for this example limitation. Use the Message Response or Message Broadcast classes to send communication data from the server to the client.

(Message classes are not available if you created a project yourself using the project template. Download the project that was created for tutorials and set the internal BasicProtocol.java to be included in the project. You also need to register the protocol when you bootstrap. Protocol-related settings on the server side have been pre-completed with a pre-prepared tutorial project, so if you have followed the tutorial, you do not need to proceed.)

<br>

### Implement Client-side Transfer

First, let's write a code that sends a message from the client to the server. Once connected to the server as a logged-in user, the server's functionality can be utilized through the user agent. Here, packets are sent to the room using the Send feature of the user agent. In order to send a message, the packet containing the contents to be transmitted must be handed over to a factor.

Add a message transfer script to the Unity Project's GameManager script. When the SendMessage method is called, the user enters a message in the on-screen input field and packets containing his or her user ID are sent to the server via userAgent. And add an implementation to the Update function so that the SendMessage method runs each time Enter is pressed. The example code below shows the implementation of the SendMessage method.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using Protocol;


public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }

    void Update(){
	 	 if ( Input.GetKeyDown(KeyCode.Return) && !string.IsNullOrEmpty(ChatInputText.text)){
	 	 		SendMessage();
	 	 }
	}

	public void SendMessage(){
        MessageRequest messageRequest = new MessageRequest();
        messageRequest.Message = "\n[" + connectHandler.accountId +  "]:" + ChatInputText.text;
        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).Send(new Packet(messageRequest));
        ChatInputText.text = string.Empty;
	}
}
```

In this way, the feature to send packets from the client to the server was implemented. However, even if you send packets to the server now, the server will not respond. This is because the server did not analyze the contents and define what behavior to do when it received the packet. In the next chapter, we will cover server-side implementation.

<br>

### Implement Server-side Response

Here, we create a feature that allows the server to receive messages sent by game users and send them to users in the room. Handlers are used to process messages sent by clients in the game room of the server. Handlers are bundles of codes to handle specific protocols. Depending on the type of protocol, there can be multiple handlers and multiple handlers can be registered in a room. Therefore, a room can handle multiple protocols.

Navigate to the server project and double-check the BasicRoom class. Previously, you created RoomMessageDispatcher from the BasicRoom class that inherited the BaseRoom class. The packet dispatcher is responsible for allowing the message to find and execute the appropriate (intended by the programmer) handler when the message is received. The handler for the incoming message that you write this time is also registered in this packet dispatcher.

Handlers are also created through class creation. Create a new class file under the name BasicHandler. After you inherit the RoomPacketHandler, you override the execute method. After that, when you register for a packet dispatcher, the contents of this method are executed when a MessageRequest is received.

```Java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import org.slf4j.Logger;
import protocol.BasicProtocol;

import static org.slf4j.LoggerFactory.getLogger;

public class BasicHandler implements RoomMessageHandler<BasicRoom, BasicUser, BasicProtocol.MessageRequest> {
    private static final Logger logger = getLogger(BasicHandler.class);

    @Override
    public void execute(BasicRoom room, BasicUser requester, BasicProtocol.MessageRequest request) throws SuspendExecution {

    }
}
```

Before implementing the inside of the method, implement the broadcast method used by the handler in the Basic Room class. It is a method that duplicates and sends packets that have entered the method to all users in the room. When a MessageRequest is received, the protocol's descriptor and handler class are registered through the registerMsg method in order to register the previously generated handler to run. At this time, message handler registration must be implemented in a static block.

```Java
public class BasicRoom extends BaseRoom<BasicUser> { 
    private static RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>(); 
    private Map<Integer, BaseUser> users = new HashMap<>(); 
 
    static { 
        dispatcher.registerMsg(BasicProtocol.MessageRequest.class, BasicHandler.class); // Register handlers for messages to be processed 
    } 
 
    @Override 
    public final RoomMessageDispatcher<BasicRoom, BasicUser> getMessageDispatcher() { 
        return dispatcher; 
    } 
 
    ...Omit... 
 
    public void broadcast(com.google.protobuf.GeneratedMessageV3 message) { 
        users.values().stream().forEach(user -> user.send(message)); 
    } 
 
}
```

What will be executed, i.e. the implementation inside the execute method, is to be written as follows. The handler example implementation below sends a response message to the sender for the received message, as well as additional messages for room-by-room broadcasts to all users. At this time, the client is implemented to synchronize games based on room-by-room broadcast messages.

```java
import co.paralleluniverse.fibers.SuspendExecution; 
import com.nhn.gameanvil.node.game.RoomMessageHandler; 
import org.slf4j.Logger; 
import protocol.BasicProtocol; 
 
import static org.slf4j.LoggerFactory.getLogger; 
 
public class BasicHandler implements RoomMessageHandler<BasicRoom, BasicUser, BasicProtocol.MessageRequest> { 
    private static final Logger logger = getLogger(BasicHandler.class); 
   
  	@Override 
    public void execute(BasicRoom room, BasicUser requester, BasicProtocol.MessageRequest request) throws SuspendExecution { 
        BasicProtocol.MessageResponse response = BasicProtocol.MessageResponse.newBuilder().setMessage(request.getMessage()).build(); 
        BasicProtocol.MessageBroadcast broadcast = BasicProtocol.MessageBroadcast.newBuilder().setMessage(request.getMessage()).build(); 
 
        requester.reply(response); // Respond to sender
        room.broadcast(broadcast); // Send a message to all users in the room    } 
}

```

In the above code, first, a MessageRequest object is created by parsing the stream of received packets. Then, using the Message value of the MessageRequest object, a new MessageResponse and MessageBroadcast objects are created, respectively. Objects of the MessageResponse type send packets to user objects that have sent them to the room. The MessageBroadcast object is sent through the room to everyone in the room.

In this way, the server has added the ability to receive packets sent by clients, do some processing, and then return them. At this time, the client must also specify how to handle packets sent from the server.
<br>

### Implement Client-side Receipt

The client also needs to register a handler in advance to process packets on the server side. Otherwise, when the packet is received, it will discard the contents as it considers that it is a protocol whose processing method is unknown. When you receive the content from the server, you can register the handler of the protocol type of the packet sent from the server to detect it and process the content. In other words, write and register a handler registration code that handles MessageBroadcast-type messages.

Go back to the Unity project and add an implementation to GameManager's Start method. First, use the Protocol Manager to register the agreed protocols between the server and the client. Here, we registered Basic Protocol with number 0. Numbers are discussed in the following.

After that, register the listener through the user agent. As the method type, we use the protocol type. The protocol that the server sets to send is Message Broadcast, so you can use it. The handler implementation creates a simple logic that outputs the received message as it is in the text prepared on the screen. It is the most basic feature of chat.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using Protocol;

public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        ProtocolManager.GetInstance().RegisterProtocol(BasicProtocolReflection.Descriptor);
        connectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).AddListener<MessageBroadcast>((sendUserAgent, messageBroadcast) => {
            ChatLogText.text += messageBroadcast.Message;
        });
    }
  
		...omit...
}

```

<br>

### Confirm Message Forwarding

After modifying the server, check if it's newly run, build with `cmd+b` or `ctrl+b` in Unity and play. Create a room in the built game and check the server-side log. In that state, enter play mode in Unity Editor and enter RoomId of the room you created earlier to join that room.

In any game process, type text in the text box, and then press Enter. Then the same text will appear in other game processes.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_chat_test.png)

We learned how to handle messages through a simple chat server implementation. Next, we will look at the implementation of more practical examples.

<br>

## Implement Puzzle Game

In the game scene, a single-play puzzle game is pre-implemented. Enter play mode, drag the puzzle piece, and place it near the right position to adjust it to the correct position on the grid. In this chapter, we'll try to make it a multiplayer game.

Messages exchanged between users in the same room can represent any type of value other than MessageRequest, MessageResponse and MessageBroadcast. In this chapter, we will learn how to define protocols yourself and register with servers and clients.

Messages can be sent and received between servers and clients as long as they are defined based on pre-defined protocols. There may be a number of expressions, including XML and json, but GameAnvil uses Google Protocol Buffers. This is one of the best solutions in terms of speed and stability.

<br>

### Serialize/reverse-serialize messages with Google Protocol Buffers

To use a protocol buffer, you first need to write a specification about how you want to define a message. For example, MessageRequest specification contains a single string. After that, the protocol specification is compiled and converted into a file in the desired language. After that, you can use it as a message protocol for packets, similar to how you used MessageRequest.

We will start writing a message protocol for synchronizing puzzle locations. After adding the Puzzle.proto file to the src/main/proto path of the server project, write the protocol specification as shown below.

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto.png)

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

Puzzle Position protocol consists of the unique number information (Index) of each puzzle piece and the location information (PositionX and PositionY) of the puzzle piece to indicate the latest location of the puzzle piece. Now the protocol definition is done. Next, we will compile these protocol definitions into classes that fit each development language.

The protoc executable file is on the top path of the project. Go to the project and use the following command line to compile for Java and C#. If you run the command line and no result message appears, it's a success.

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

You can now check that a new Puzzle.java class has been created in the src/main/java path.

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto_java.png)

C# class files are moved to the Asset/Protocol path of the Unity project using programs such as Finder and File Explorer.

[](https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto_csharp.png)

You have finished writing a message protocol for synchronizing puzzle locations.

<br>

### Register Protocols

If you have defined the protocol and compiled it safely, you must register the protocol class on both the server and the client. For your information, chat protocols (such as MessageRequest) were registered in the template by default, so there was no need to register additionally. However, the Puzzle protocol that was implemented additionally must be registered directly. In the GameAnvil server's Main method, register the protocol as follows.

```C#
public class Main {

    public static void main(String[] args) {
        GameAnvilServer server = GameAnvilServer.getInstance();

        server.addProtoBufClass(BasicProtocol.class);
        server.addProtoBufClass(Puzzle.class);

        server.run();
    }

}
```

The Unity project adds a protocol to the ConnectHandler Start method as follows. At this time, it registers as 1, the same index as the server.

```c#
using Protocol;

public class ConnectHandler : MonoBehaviour {
    void Start()
    { }
        // Register the protocol
        ProtocolManager.GetInstance().RegisterProtocol(PuzzleReflection.Descriptor);
    }
  
  	...omit...
}
```

<br>

### Implement Client-side Transfer

Now, we've defined and registered protocols for the game. From now on, we're going to implement the function of actually sending messages based on these protocols. First of all, we're going to implement the part where the data is transmitted from the client side. While dragging the puzzle piece, we'll try to send that location to the server.

Go to the Unity project, open the Puzzle.cs file and add code as below. Create a message object containing the unique location information of the puzzle while being dragged. And send it to the server through userAgent. Only the protocol of the message is different, and you can see that it is the same way as the MessageRequest we practiced earlier.

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;
using Protocol;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using GameAnvil.User;

public class Puzzle : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public int index;

    public Transform puzzleHolder;
    public float tolerance;

  
    private ConnectHandler connectHandler;

    void Start(){
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

    public void FixPosition(){
        if (Vector2.Distance(transform.position, puzzleHolder.position) < tolerance){
            transform.position = puzzleHolder.position;
        }
    }

    public void SendPuzzlePosition(Vector2 puzzlePosition, bool onEndDrag){
		PuzzlePosition position = new PuzzlePosition();
		position.Index = index;
        position.PositionX = (int)puzzlePosition.x;
        position.PositionY = (int)puzzlePosition.y;
        position.OnEndDrag = onEndDrag;
      
        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).Send(position);
    }
}
```



<br>

### Implement Server-side Response

The client is constantly sending the location of the puzzle to the server. Now you have to write down how the puzzle location will be handled by the server. Let's try to process the MessageRequest and get the puzzle location back to everyone in the game room, just as we did with MessageResponse, MessageBroadcast.

Return to the server project and create a PuzzlePositionHandler.java class file and use the broadcast method to forward the received packets to every user in the room.

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import protocol.Puzzle;

public class PuzzlePositionHandler implements RoomMessageHandler<BasicRoom, BasicUser, Puzzle.PuzzlePosition> {

    @Override
    public void execute(BasicRoom room, BasicUser user, Puzzle.PuzzlePosition position) throws SuspendExecution {
        room.broadcast(position);
    }
}
```

Make sure to register the handler that you implemented earlier with RoomDispatcher in BasicRoom. Even if you have created a handler class file, if you do not register with RoomDispatcher, it will not be able to run when the message arrives. BasicRoom can now handle PuzzlePosition protocols in addition to BasicProtocol.

```java
public class BasicRoom extends BaseRoom<BasicUser> { 
    private static final Logger logger = getLogger(BasicRoom.class); 
    private static RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>(); 
    private Map<Integer, BaseUser> users = new HashMap<>(); 
 
    static { 
        dispatcher.registerMsg(BasicProtocol.MessageRequest.class, BasicHandler.class); 
        dispatcher.registerMsg(Puzzle.PuzzlePosition.class, PuzzlePositionHandler.class); 
    } 
   
    ...omit... 
   
}
```

<br>

### Implement Client-side Receipt

Previously, we had to register a handler in advance to process the MessageBroadcast sent from the server. Again, to create a handler that handles puzzle positions, we write a handler registration code that handles Puzzle Position messages in GameManager's Start method. When the handler receives the message, it finds the puzzle object and moves it to the location it received on the server.

```c#
public class GameManager : MonoBehaviour{
  
    void Start()
    { connectHandler
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        ProtocolManager.GetInstance().RegisterProtocol(0, BasicProtocolReflection.Descriptor);

        // Chat message listener
        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).AddListener<MessageBroadcast>((sendUserAgent, messageBroadcast) => {
            ChatLogText.text += messageBroadcast.Message;
        });
        // puzzle location listener
        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).AddListener<PuzzlePosition>((sendUserAgent, puzzlePosition) => {
            Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);

            if (puzzlePosition.OnEndDrag)
            { }
                puzzle.FixPosition();
            }
        });
    }
    ... omit ...
}
```

<br>

### Check puzzle position synchronization

Build and play with `cmd+b` or `ctrl+b` in Unity. After you create a room in the built game, you run Unity play mode and enter RoomId of the room to join the room. You can now drag and move the location to check that the location of the puzzle piece synchronizes and reflects it to the remote client.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_puzzle_position_playmode.gif)

<br>

### Process users enter in the middle

Think of a case where a new user enters a room after a random puzzle piece position is changed during the game. At this time, the puzzle state between the existing user and the new user is different. New users have the initial puzzle state, so that they must synchronize the puzzle state with the existing user. To solve this problem, I will modify the server logic. Now, the server modifies the logic to keep all the location information of the puzzle and to synchronize it when a new user enters.

Add a puzzlePositions map to the BasicRoom, which manages location information for each puzzle piece.

```java
public class BasicRoom extends BaseRoom<BasicUser> { 
 
    private static final RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>(); 
   
    private Map<Integer, BaseUser> users = new HashMap<>(); 
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>(); 
   
  	...omit... 
   
}
```

Now modify the PuzzlePositionHandler code to save the puzzle location in each room.

```Java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import protocol.Puzzle;

public class PuzzlePositionHandler implements RoomMessageHandler<BasicRoom, BasicUser, Puzzle.PuzzlePosition> {

    @Override
    public void execute(BasicRoom room, BasicUser user, Puzzle.PuzzlePosition position) throws SuspendExecution {
        room.puzzlePositions.put(position.getIndex(), position); // Save the position information to the map
        room.broadcast(position);
    }
}

```

Modify the BasicRoom's onJoinRoom to send the location information of the puzzle stored on the server so that new users can receive the stored puzzle location information when they enter the room.

```java
public class BasicRoom extends BaseRoom {
    @Override
    public boolean onJoinRoom(BasicUser basicUser, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(user.getUserId(), user);
        puzzlePositions.values().stream().forEach(this::broadcast); // synchronize puzzle positions
        return true;
    }
}
```

After this modification, test again. First fix the puzzle location on one client, then join the game room on another client to see if the puzzle location syncs properly. You can see that it still doesn't work contrary to your intentions.

This is because the client experiences a scene move after the onJoinRoom call. This means that a synchronization message was sent before the listener registration was completed, causing a problem. To resolve this issue, the client needs to synchronize the location information of the puzzle after the scene move is finished and the listener registration is completed.

We won't solve this problem right away, but we will cover it in a later. This problem is also evident in the puzzle mixing that will be explained right away. So let's look at the puzzle mixing first and then we'll deal with what we've modified to resolve it again.

<br>

## Implement Puzzle Mixing

Let's implement the logic of randomly mixing puzzle locations. When a client asks you to mix puzzle locations, the server determines a new location after mixing puzzles. And the basic idea is to return the changed location information back to the client.

### Register Protocols

First, let's create a protocol for making puzzle-mixing requests. Go to the server project and add a protocol specification to the Puzzle.proto file. This protocol doesn't have any information to send from the client to the server, so none of the fields are there. This is also meaningful enough as a protocol.

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
 
message ScatterPuzzle { } // Puzzle Mixing Request Protocol
```

If you modify a protocol buffer file, you must also compile it again.

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

Move the created C# class back to the Unity project's Asset/Protocol folder using programs such as Finder or File Explorer. At this time, the server is automatically replaced by the newly compiled class thanks to the output path, so you don't have to work separately.

<br>

### Implement Client side

Go to the Unity project and write the code for the mixing request on GameManager.cs as below. When the Scatter method is called, a new ScatterPuzzle type message is sent through the user agent to the game room.

```csharp
public class GameManager : Monobehaviour {
	public void Scatter() {
        	ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).Send(new ScatterPuzzle());
    }
}
```

Click **Scatter PuzzleButton** in the **Hierarchy** panel. In the **Button** component of **Inspector**, add an item to the **OnClick** listener, drag the GameManager component to register and select the Scatter method from the drop-down.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_scatter_puzzle_on_click.png"/>

### Implement Server side

When a mixing request is received, we use the handler as previously used. Create a new Java file and create the handler ScatterPuzzleHandler. Set the location of each of the 16 puzzles at random and send a PuzzlePositon type message. It also updates the map of the puzzle positions of the server with the new location information.


<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/scatter_puzzle_handler.png"/>

```Java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import org.slf4j.Logger;
import protocol.Puzzle;

import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

import static org.slf4j.LoggerFactory.getLogger;

public class ScatterPuzzleHandler implements RoomMessageHandler<BasicRoom, BasicUser, Puzzle.ScatterPuzzle> {
    private static final Logger logger = getLogger(ScatterPuzzleHandler.class);
    private static final int mapSize = 400;

    private class Point {
        public int x;
        public int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    @Override
    public void execute(BasicRoom room, BasicUser user, Puzzle.ScatterPuzzle scatterPuzzle) throws SuspendExecution {
        room.puzzlePositions = new HashMap<>();
        
        List<Point> random = IntStream.rangeClosed(-4, 4).boxed()
            .map(i -> new Point(i * mapSize / 4, i%2==0 ? mapSize : -mapSize))
            .collect(Collectors.toList());
        random = Stream.concat(random.stream(), random.stream().map(p -> new Point(p.y, p.x))).collect(Collectors.toList());
        Collections.shuffle(random);

        for (int i = 0; i < 16; i++) {
            Point point = random.get(i);
            int x = 1300 + point.x;
            int y = 550 + point.y;

            Puzzle.PuzzlePosition puzzlePosition = Puzzle.PuzzlePosition.newBuilder().setIndex(i).setPositionX(x).setPositionY(y).build();

            room.puzzlePositions.put(i, puzzlePosition);

            room.broadcast(puzzlePosition);
        }
    }
}

```

Register the handler implemented in this way in the Room.

```java
public class BasicRoom extends BaseRoom<BasicUser> { 
    private static final Logger logger = getLogger(BasicRoom.class); 
    private static final RoomMessageDispatcher<BasicRoom, BasicUser> dispatcher = new RoomMessageDispatcher<>(); 
     
    private Map<Integer, BaseUser> users = new HashMap<>(); 
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>(); 
 
 
    static { 
        dispatcher.registerMsg(BasicProtocol.MessageRequest.class, BasicHandler.class); // Register handlers for messages to be processed
        dispatcher.registerMsg(Puzzle.PuzzlePosition.class, PuzzlePositionHandler.class); 
        dispatcher.registerMsg(Puzzle.ScatterPuzzle.class, ScatterPuzzleHandler.class); 
    } 
    ...omit...   
}
```

Now, both the client-side transfer feature and the server-side response features are complete.
<br>

### Check puzzle mixing feature

Enter play mode in Unity Editor. Click the Scatter Puzzle button to see if the puzzle mixing function works well.

<br>

## Better handling of users entered in the middle

There was a synchronization problem in the process of processing the intermediate user earlier. This is because we thought that the timing of the user entering the room was appropriate as the timing of synchronizing the location of the puzzle piece. However, the puzzle game we are implementing causes scene movement when the user enters the room.

Therefore, we have to think about it divided into two points of view: registering a listener and calling an onJoinRoom callback. This may or may not be a problem, depending on the implementation of the game client. Since the scene movement starts after the onJoinRoom callback call, the client would like to request a puzzle location synchronization directly to the server immediately after the scene movement is completed.

<br>

### Register Protocols

Navigate to the server project and add a protocol to Puzzle.proto for requesting puzzle location synchronization.

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

message PuzzlePositionReq {}
```

After recompiling the protocol, move the generated C# class to the Unity project.

<br>

### Implement Client side

To ask the server for the location of the puzzle immediately after the scene movement, you need to modify the Start function that runs immediately after the scene movement. Add the code that sends the newly created PuzzlePositionReq protocol message to GameManager's Start method, as follows.

```c#
public class GameManager : MonoBehaviour
{ {
    void Start(){
        ...omit...

        ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).Send(new PuzzlePositionReq());
    }
  
  	...omit...
}
```

<br>

### Implement Server side

Move the puzzle location send code that you mis-implemented in onJoinRoom to the correct location and create a new one in PuzzlePositionReqHandler.

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomMessageHandler;
import protocol.Puzzle;

public class PuzzlePositionReqHandler implements RoomMessageHandler<BasicRoom, BasicUser, Puzzle.PuzzlePositionReq> {
    @Override
    public void execute(BasicRoom room, BasicUser user, Puzzle.PuzzlePositionReq req) throws SuspendExecution {
        room.puzzlePositions.values().stream().forEach(room::broadcast);
    }
}
```

And register the created handler in Basic Room.

```java
public class BasicRoom extends BaseRoom<BasicUser> { 
 
    static { 
        dispatcher.registerMsg(BasicProtocol.MessageRequest.class, BasicHandler.class); // Register handlers for messages to be processed
        dispatcher.registerMsg(Puzzle.PuzzlePosition.class, PuzzlePositionHandler.class); 
        dispatcher.registerMsg(Puzzle.ScatterPuzzle.class, ScatterPuzzleHandler.class); 
        dispatcher.registerMsg(Puzzle.PuzzlePositionReq.class, PuzzlePositionReqHandler.class); 
    } 
   
    ...omit... 
   
}
```

<br>

### Check the process of users entered in the middle

Build and play with `cmd+b` or `ctrl+b` in Unity. Now create a room in the built game and run puzzle shuffling. In that state, enter the play mode of Unity Editor, join this room, and verify that the puzzle position syncs.

<br>

## Implement user matchmaking

### Implement Server side

User matchmaking combines users' matchmaking requests so that similar levels of users can start games in the same room according to the appropriate criteria. Users can properly distinguish and match users by implementing various factors, such as points and scores. Here, we implement a logic that matches two users into one game.

Create a UserMatchInfo class. File name is Basic UserMatchInfo.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_user_match_info.png)

This class will contain the information of the users that will be used for matching. If there are any elements that will be used for matchmaking, you can add them here. In this example, we will only create and use methods that must be implemented without adding any other elements. One thing to note is that the getId() method must be implemented to return the user's ID requested. And since the party matchmaking feature is not used, set it to return 0.

```java
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.match.BaseUserMatchInfo;

import java.io.Serializable;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicUserMatchInfo extends BaseUserMatchInfo implements Serializable {

    private int id;

    public BasicUserMatchInfo(int id) {
        this.id = id;
    }

    @Override
    public int getId() {
        return id;
    }

    @Override
    public int getPartySize() {
        return 0;
    }
}
```

These User MatchInfo are created and used in the process of implementing an onMatchUser callback from the game user on the server when the client requests user matchmaking. GameAnvil provides a basic user matchmaker. The onMatchUser below uses the basic user matchmaking of these engines through the matchUser API.

```java

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.exceptions.NodeNotFoundException;
import match.BasicUserMatchInfo;
import org.slf4j.Logger;

import java.util.concurrent.TimeoutException;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@UserType("BASIC_USER")
public class BasicUser extends BaseUser {
    private static final Logger logger = getLogger(BasicUser.class);

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, Payload payload, Payload outPayload) throws SuspendExecution {
   try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(getUserId());
            return matchUser(matchingGroup, roomType, term); // Request user matchmaking
        } catch (Exception e) {
            logger.error("BasicUser::onMatchUser()", e);
        }
        } return false;
    }
}

```

Once you're basically ready to use user matchmaking, you'll now create a matchmaker that performs actual matchmaking. Add a Basic User Matchmaker class as shown below.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_user_match_maker.png)

The creator calls the creator of the parent class and delivers the number of matches and the effective time of the match application as a factor. After the effective time passed, the match request is automatically canceled. And the match method that performs actual matchmaking is called once a second by the engine.

getMatchRequests receives the minimum number of people for matching as a factor and queries the entire current matching pool to create as much optimal matching as possible. In other words, if there are a lot of matching requests, getMatchRequests can create 100 or 1,000 matches at a time. At this time, if the matching is successful, the UserMatchInfo list of matched users is returned. If you pass this list to the matchSingles API provided by the engine, you can create and move rooms for users in the list on your own.

Returns null if there are not enough requests to match, or if there are no targets that meet the requirements. In this case, the same match search is performed again on the next match call after 1 second.

```Java
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.match.BaseUserMatchMaker;
import org.slf4j.Logger;

import java.util.List;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicUserMatchMaker extends BaseUserMatchMaker<BasicUserMatchInfo> {
    private static final Logger logger = getLogger(BasicUserMatchMaker.class);
    private static final int NUM_USER = 2;
    private static final int TIMEOUT = 10000;

    public BasicUserMatchMaker() {
        super(NUM_USER, TIMEOUT);
    }

    @Override
    public void onMatch() {
        List<BasicUserMatchInfo> matchRequests = getMatchRequests(2);

        if (matchRequests == null) {
            return;
        }

        matchSingles(matchRequests);
    }

    @Override
    public boolean onRefill(BasicUserMatchInfo roomInfo) {
        return false;
    }
}
```

<br>

### Implement Client side

All matchmaking logic is implemented on the server, so the client just needs to send a request when matchmaking is needed. Add the UserMatchMaking method to ConnectHandler. And add a handler to move the scene when matchmaking is over.

```csharp
public class ConnectHandler : MonoBehaviour 
{ 
	...omit... 
	 
    void Start () { 
   
        ... omit... 
      
            connector.GetUserAgent("BASIC_SERVICE", 1).onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone resultCode, bool created, int roomId, Payload payload) =>{ 
            this.roomId = roomId; 
            SceneManager.LoadScene("GameScene"); 
        }; 
    } 
	 
	... omit... 
 
    public void UserMatchMaking(){ 
            connector.GetUserAgent("BASIC_SERVICE", 1).MatchUserStart("BASIC_ROOM", "BASIC_MATCHING_GROUP",(UserAgent userAgent, ResultCodeMatchUserStart resultCode, Payload payload) =>{ 
            Debug.Log(resultCode); 
        }); 
    } 
}

```

In the scene, drag the ConnectHandler component to the OnClick Listener on the User Match Making button and select the User Match Making method from the drop-down.

<br>

### User Match Making Test

Build with `cmd+b` or `ctrl+b` in Unity and then play. In Unity Editor, you enter play mode with that status. Press the User Match Making button on both sides to make sure the match is done and tied to the same room number.

<br>

## Implement Room Matchmaking

Room matchmaking is a feature that allows you to automatically enter the room that best meets the user's requirements among the rooms managed by the matchmaker. In other words, if user matchmaking is a feature that matches users and users, room matchmaking is a feature that matches users and rooms. At this time, users can be matched to a room under various conditions depending on the implementation method. Here, matchmaking is implemented to enter the room with the least number of people among rooms that have not yet been filled.

First of all, you need a class that actually performs matchmaking. And there should be one instance in each room that contains information to manage the room in the room matchmaker. Finally, every time a user applies for matchmaking, an application instance containing the user's requirements is required. Let's create these three new classes.

Additionally, some modifications are made to the existing logic. Room matchmaking is done only for rooms requested for room matchmaking, not all rooms. Therefore, at the time of room creation, add the code you apply for room matchmaking.
<br>

### Implement Server side

First, implement a class that will represent matchmaking requests. Create a BasicRoomMatchForm class. 

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_maker_form.png)

Whenever a user makes a matchmaking request, a BasicRoomMatchForm object is created and used. "BASIC_MATCHING_USER_CATEGORY" passed from generator to parent generator is not a concern in this document.

```java
import com.nhn.gameanvil.node.match.BaseRoomMatchForm;
import java.io.Serializable;

public class BasicRoomMatchForm extends BaseRoomMatchForm implements Serializable {
    public BasicRoomMatchForm() {
        super("BASIC_MATCHING_USER_CATEGORY");
    }
}
```



The following implements a class that represents the information of the room to be matched. Create a BasicRoomMatchInfo class as follows.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_info.png)

At this time, the maximum capacity of the room is four people specified in the static field. These maximum capacity and user types must be passed on as factors to the creator of the inherited BaseRoomMatchInfo.

```java

import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.match.BaseRoomMatchInfo;

import java.io.Serializable;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoomMatchInfo extends BaseRoomMatchInfo implements Serializable {
    private static final int MAX_USER = 4;

    public BasicRoomMatchInfo(int roomId) {
        super(roomId, "BASIC_USER", MAX_USER);
    }
}
```

Next, we create a room matchmaker that will actually handle room matchmaking.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_maker.png)

Create BasicRoomMatchMaker as follows.

```Java
import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.data.RoomMatchResultCode;
import com.nhn.gameanvil.node.match.BaseRoomMatchMaker;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoomMatchMaker extends BaseRoomMatchMaker<BasicRoomMatchForm, BasicRoomMatchInfo> {
    @Override
    public RoomMatchResultCode onPreMatch(BasicRoomMatchForm basicRoomMatchForm, Object... objects) {
        return RoomMatchResultCode.SUCCESS;
    }

    @Override
    public boolean canMatch(BasicRoomMatchForm basicRoomMatchForm, BasicRoomMatchInfo basicRoomMatchInfo, Object... objects) {
        return true;
    }

    @Override
    public void onPostMatch(BasicRoomMatchForm basicRoomMatchForm, BasicRoomMatchInfo basicRoomMatchInfo, Object... objects) {

    }

    @Override
    public int compare(BasicRoomMatchInfo o1, BasicRoomMatchInfo o2) {
        int o1UserCount = getUserCount(o1.getRoomId());
        int o2UserCount = getUserCount(o2.getRoomId());
        if (o1UserCount > o2UserCount) {
            return -1;
        } else if (o1UserCount < o2UserCount) {
            return 1;
        } else {
            return 0;
        }
    }

    @Override
    public void onIncreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {

    }

    @Override
    public void onDecreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {

    }
}
```

The compare method implements the condition of sorting the rooms in a matching pool. Example implements sorting the rooms according to the number of people.

Now, we're almost ready for room matchmaking. When the client sends a room match request, the BasicUser on the server calls an onMatchRoom callback. In this callback, we create the BasicRoom MatchForm object that we discussed earlier and forward it as a factor to the matchRoom API to call it. That is, we forwarded the room match request sent by the client to the matchmaker here.

```java
import co.paralleluniverse.fibers.SuspendExecution; 
import com.nhn.gameanvil.exceptions.NodeNotFoundException; 
import match.BasicUserMatchInfo; 
import org.slf4j.Logger; 
 
import java.util.concurrent.TimeoutException; 
 
import static org.slf4j.LoggerFactory.getLogger; 
 
@ServiceName("BASIC_SERVICE") 
@UserType("BASIC_USER") 
public class BasicUser extends BaseUser { 
 
	@Override 
	public final RoomMatchResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final Payload payload) throws SuspendExecution { 
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm(); 
 
        try { 
            return matchRoom(matchingGroup, roomType, gameRoomMatchForm); // Forward request to matchmaker
        } catch (Exception e) { 
            logger.error("BasicUser::onMatchRoom()", e); 
        } 
        return RoomMatchResult.FAILED; 
    } 
}
```

Call the registerRoomMatch API from the onCreateRoom callback in BasicRoom to make the room matchmaking target at the time the room is created, as shown below.

```java
public class BasicRoom extends BaseRoom<BasicUser> { 
    ..omit... 
 
    @Override 
    public boolean onCreateRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution { 
        users.put(user.getUserId(), user); 
 
        BasicRoomMatchInfo gameRoomMatchInfo = new BasicRoomMatchInfo(getId()); 
        try { 
            registerRoomMatch(gameRoomMatchInfo, "BASIC_MATCHING_USER_CATEGORY", user.getUserId()); // Register a room as a room matchmaking target
        } catch (Exception e) { 
            logger.error("BasicRoom::onCreateRoom():registerRoomMatch", e); 
        } 
 
        logger.debug("onCreateRoom(userId:{})," user); 
        return true; 
    } 
   
}
```

Also, the room match making information should be updated whenever the room information changes. FYI, the room match maker automatically synchronizes the number of people in the room. Therefore, we only need to update the additional information except the number of people.

The following code modifies the onJoinRoom callback to update the matchmaking information when users join the room.

```java
public class BasicRoom extends BaseRoom<BasicUser> { 
    ...omit... 
 
    @Override 
    public boolean onJoinRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution { 
        users.put(user.getUserId(), user); 
 
        BasicRoomMatchInfo gameRoomMatchInfo = new BasicRoomMatchInfo(getId()); 
        try { 
            updateRoomMatch(gameRoomMatchInfo); 
        } catch (Exception e) { 
            logger.error("BasicRoom::onCreateRoom():registerRoomMatch", e); 
        } 
 
        logger.debug("onJoinRoom(userId:{})," user); 
        return true; 
    } 
}
```

<br>

### Implement Client

Just like user matchmaking, the client just needs to send a request when matchmaking is needed. Add the RoomMatchMaking method to ConnectHandler.

```csharp
public class ConnectHandler : MonoBehaviour {
    public void RoomMatchMaking(){
            connector.GetUserAgent("BASIC_SERVICE", 1).MatchRoom("BASIC_ROOM", "BASIC_MATCHING_GROUP", "BASIC_MATCHING_USER_CATEGORY", true, 
            (UserAgent userAgent, ResultCodeMatchRoom resultCode, int integer, int roomId, string roomName, bool created, Payload payload) =>{
            Debug.Log(resultCode);
            if (resultCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS){
            	this.roomId = roomId;

            	SceneManager.LoadScene("GameScene");
            }
        });
    }
}

```

In the scene, drag to register the ConnectHandler component to the OnClick Listener on the Room Match Making button, and select the RoomMatchMaking method from the drop-down menu.

To create a room with matching groups, modify the room creation code. In this case, the matching group is a random string predefined between the server and the client to logically divide the room to be matched. Here, we use the string of BASIC_MATCHING_GROUP.

```c#
public class ConnectHandler : MonoBehaviour {
	public void CreateRoom(){
            connector.GetUserAgent("BASIC_SERVICE", 1).CreateRoom("", "BASIC_ROOM", "BASIC_MATCHING_GROUP", null (UserAgent ua, ResultCodeCreateRoom resultCode, int roomId, string roomName, Payload payload) => {
            Debug.Log(resultCode); 
            if (resultCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
                this.roomId = roomId;
              
                SceneManager.LoadScene("GameScene");
            }
        });
    }
}
```

<br>

### Room Match Making Test

Build with `cmd+b` or `ctrl+b` in Unity and then create a room in the play state. In Unity Editor, enter play mode. In play mode, press the Room Match Making button to see if you are moving to the room you created in build mode.

<br>

## Implement Room Leave

Finally, for the implementation of the feature to leave the room, add the method below to the Unity client's GameManager.

```c#
public void LeaveRoom() {
		ConnectHandler.GetInstance().GetConnector().GetUserAgent("BASIC_SERVICE", 1).LeaveRoom((userAgent, resultCode, force, roomId, payload) =>{
      	if(resultCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS){
      		Destroy(connectHandler.gameObject);
    			SceneManager.LoadScene("ConnectScene");
    		}
    });
}
```

In the scene, drag the GameManager component to the OnClick Listener on the Leave Room button to register, and select the LeaveRoom method from the drop-down menu.

## Wrap up Project

So far, we've used GameAnvil and Unity to create a puzzle game that enables real-time multiplayer functionality. In the process, we've used many of GameAnvil's core features. However, GameAnvil supports more rich and diverse features that are not included in this tutorial. Refer to the following documents for these features. Also, the accompanying reference sample project and JavaDoc will help you understand GameAnvil a lot.
