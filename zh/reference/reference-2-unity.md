## Game > GameAnvil > Reference Projects > Unity Samples

# Connector API doc - C#

[Connector API C#](https://gameplatform.toast.com/docs/api/)



# Download Client

[Sample Game Client](https://github.com/nhn/gameanvil.sample-game-client-unity.git)



# Configuration Environment

* Unity3d: 2020.3.37f1

  - It is created using Unity Standalone. This sample is for developers' reference and works only in the editor.

* GameAnvil Connector: 1.4.1


# Run Client

## Use Unity3d

Run the project you cloned or downloaded from the git repository into Unity.

![reference-2-unity-01](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-01.png) 



### Check Running environment

Check out the GameAnvil Connector C# library.

* Assets/GameAnvil

![reference-2-unity-02](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-02.png) 

	* GameAnvil
    * GameAnvilConnector
	* Google.ProtoBuf
	* K4os.Compression.LZ4
    * link
	* log4net
	* SuperSocket.ClientEngine
	* System.Buffers
	* System.Memory
	* System.Runtime.CompilerServices.Unsafe

Right-click the GameAnvil.dll file, and in Properties, verify that the version information is the correct version.



### Run Client

Click the Play button in the Unity Editor to run it. If there are no errors in the log console on the right, it started successfully.

![reference-2-unity-03](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-03.png) 



# Explore Client

## Configure Client Project

It is a project created using the client for linking to GameAnvil sample server, which is created for reference purposes while developing a game.

- As it focuses on functionality, it may contain a number of game errors or bugs.
- For more information, contact Help [Center](https://www.nhncloud.com/kr/support/inquiry).



* Assets
  * GameAnvil: The library location used by GameAnvil
  * GameAnvilSample: The GameAnvil Sample folder
    * StartScene: The initial screen. It branches out to default Gamebase initialization, guest login process,, platform test, or game test.

      ![reference-2-unity-04](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-04.png) 

    * LoadingScene: The loading screen displayed when the screen is changed

      ![reference-2-unity-05](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-05.png) 

    * Data: Data object folder
    * GameTest: The game test folder
      * Scenes: The game test screen folder
        * GameLoginScene: The screen in which ID is entered and overall login is processed

          ![reference-2-unity-06](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-06.png) 

        * GameLobbyScene: The screen after a user logs in, showing TapBird (1 player), MultiTapBird (4 players), and Snake (2 players) games, rankings, user information, and nickname changes.

          ![reference-2-unity-07](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-07.png) 

    * PlatformTest: GameAnvil API Test folder
      * Scenes: GameAnvil API Scenes folder
        * AuthScene ; launching(rest), screen that handles connect, authenticate, and login

          ![reference-2-unity-08](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-08.png)  ![reference-2-unity-09](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-09.png)  ![reference-2-unity-10](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-10.png)  ![reference-2-unity-11](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-11.png)

        * LobbyScene: Game lobby screen after login, single game, room match multiplayer (4 players), user match (2), ranked, and shuffle deck available screens

          ![reference-2-unity-12](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-12.png) 

        * MultiSnakeGameScene: User Match 2 Player Game Screen

          ![reference-2-unity-13](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-13.png) ![reference-2-unity-14](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-14.png)

        * MultiTapBirdGameScene: Room Match 4 Player Game Screen

          ![reference-2-unity-15](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-15.png) 

        * SingleGameScene: Single Room Game Scene

          ![reference-2-unity-16](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-16.png)  ![reference-2-unity-17](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-17.png) 

    * Protocols: The protocol folder to communicate with server
    * Snake: Snake game folder, user match game, displays the food sent by the server on the screen for the 2 players, displays the movement value of the user, process when the player eats food, checks game end conditions
      * Scenes: The Snake game screen folder
        
        ![reference-2-unity-18](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-18.png) ![reference-2-unity-19](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-19.png)

    * TapBird: The TapBird game folder, single game & records up to 4 players' high score, displays the scores of all the players in the game
      * Scenes: The TapBird game screen folder
        
        ![reference-2-unity-20](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-20.png) ![reference-2-unity-21](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-21.png) ![reference-2-unity-22](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-22.png)

  * Plugins: A folder for IOS/Android



## Client Behavior

### ConnectHandler : Assets/GameAnvilSample/

- Assets/GameAnvilSample/ConnectHandler.cs
- Initialize the GameAnvil connector and handle protocol registration.

```c#
connector = new GameAnvil.Connector(config);
// Add connector logs
connector.Logger += (level, log) =>
{ level
    Debug.Log(string.Format("Log[{0}]:{1}", level, log));
};
connector.LvNetLogger += (level, log) =>
{ }; }; }; }; }; }; }; }
    Debug.Log(string.Format("Net[{0}]:{1}", level, log));
};

// Register protocols in the same order as the server
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.GameMultiReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.GameSingleReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.ResultReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.UserReflection.Descriptor);
```

### Register GameAnvil Connector Listener

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
// Register listeners to handle the disconnect part
ConnectHandler.Instance.GetConnectionAgent().onDisconnectListeners += (ConnectionAgent connectionAgent, ResultCodeDisconnect result, bool force, Payload payload) =>
{ }
    Debug.LogFormat("onDisconnect - {0}", result);
    // what to do when the connection is lost
};
```

### Connect 

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
        // Attempt to connect to the server.
        ConnectHandler.Instance.GetConnectionAgent().Connect(textIP.text, int.Parse(textPort.text),
            (SessionAgent sessionAgent, ResultCodeConnect result) =>
            { }
                Debug.Log("Connect " + textID.text + ":" + textPort.text + " " + result);

                if (result == ResultCodeConnect.CONNECT_SUCCESS)
                { }
                    // Process on successful server connection
                }
                else
                { }
                    // Handle server connection failure
                }
            }
        );
```

### Auth 

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
            // Set protocol data required for authentication
            var authenticationReq = new Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReq
            { // Set the authentication data
                AccessToken = Constants.AUTH_ACCESS_TOKEN
            };

            // Attempt to authenticate to the server. Currently deviceid, id, and pw are all passed as uuid values
            ConnectHandler.Instance.GetConnectionAgent().Authenticate(inputFieldUUID.text, inputFieldID.text, inputFieldID.text, new Payload().add(new Packet(authenticationReq)),
                (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) =>
                { result
                    if (result == ResultCodeAuth.AUTH_SUCCESS)
                    {
                        // Process on success
                    }
                    else
                    {
                        // Handle on failure
                    }
                }
            );
```

### Login 

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
       // Set protocol data required for login
        var loginReq = new Com.Nhn.Gameanvil.Sample.Protocol.LoginReq
        { }
            // Set the required data
        };

        // Log in to the server
        ConnectHandler.Instance.CreateUserAgent(Constants.GAME_SPACE_NAME, Constants.userSubId).Login(Constants.SPACE_USER_TYPE, string.Empty, new Payload().add(new Packet(loginReq)),
            (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) =>
            { }
                if (result == ResultCodeLogin.LOGIN_SUCCESS)
                { }
                    if (loginInfo.Payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>())
                    {
                        // Handle the login response protocol
                        Com.Nhn.Gameanvil.Sample.Protocol.LoginRes loginRes = Com.Nhn.Gameanvil.Sample.Protocol.LoginRes.Parser.ParseFrom(loginInfo.Payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>().GetBytes());

                        // Set game data received from the server

                        // Handle the status of being in the room
                        if (loginInfo.isJoinedRoom)
                        { return true
                            if (loginInfo.RoomPayload.contains<Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg>())
                            { }
                                Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg roomInfoMsg = Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg.Parser.ParseFrom(loginInfo.RoomPayload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg>().GetBytes());
                                // Processing according to room type
                                if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomSingle)
                                { }
                                    // Single
                                }
                                else if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomSnake)
                                { }
                                    // User Match 
                                }
                                } else if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomTap)
                                {
                                    // Room match
                                }
                            }
                        }
                    }
                }
                }
                {
                    // Handle failure
                }
            }
       );
```

### GetUserAgent

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
// Save the user object from the connector
gameUser = ConnectHandler.Instance.GetUserAgent(Constants.GAME_SPACE_NAME, string.Empty);
```

### CreateRoom: Create and enter a single-player room

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // Create a room for a single player
        gameUser.CreateRoom(Constants.SPACE_ROOM_TYPE_SINGLE, new Payload().add(new Packet(startGameReq)), (UserAgent userAgent, ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) =>
        { result
            if (result == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
            {
                // Handle success
            }
            else
            {
                // Handle failure
            }
        });
```

### MatchRoom: Multi-room matchmaking

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // Request a match to enter a created room - you can play alone. Enter all players up to the maximum number
        gameUser.MatchRoom(Constants.SPACE_ROOM_TYPE_MULTI_ROOM_MATCH, "UNLIMITED_TAP", true, false, (UserAgent userAgent, ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload) =>
        { result
            if (result == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                // Process on success
            }
            else
            {
                // Handle on failure
            }
        });
```

### MatchUserStart: Multi-user matchmaking

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // Create a room by matching two players, enter at the same time, and start the game
        gameUser.MatchUserStart(Constants.SPACE_ROOM_TYPE_MULTI_USER_MATCH, "SNAKE", (UserAgent userAgent, ResultCodeMatchUserStart result, Payload payload) =>
        {
            if (result == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
            {
                // Process on success
            }
            else
            {
                // Handle on failure
            }
        });
```

- onMatchUserDone is called from server when users are matched.

```c#
        // Pre-register listeners and set the game-ready flag when the user enters the game room
        gameUser.onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone result, bool created, int roomId, Payload payload) =>
        { result
            if (result == ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS)
            {
				// Process on success
            }
        };
```

- onMatchUserTimeout is called when user match is timed out. Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs

```c#
        // Listeners for user match request timeout
        snakeGameUser.onMatchUserTimeoutListeners += (UserAgent userAgent) =>
        {
        	// Handle user match timeout
        };
```

### LeaveRoom: Called when the user leaves the room

- Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/MultiTapBirdGameUI.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs

```c#
     // Define the end of game protocol
        var endGameReq = new Com.Nhn.Gameanvil.Sample.Protocol.EndGameReq
        { EndType = gameEndType
            EndType = gameEndType
        };

        // Request to leave the game room
        tapBirdUser.LeaveRoom(new Payload().add(new Packet(endGameReq)), (UserAgent userAgent, ResultCodeLeaveRoom result, bool force, int roomId, Payload payload) =>
        {
            if (result == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
            { }
            	// Handle success, process message received
                if (payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes>())
                { }
                    Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes endGameRes = Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes.Parser.ParseFrom(payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes>().GetBytes());
                }
            }
            else
            {
                // Handle on failure
            }
        });
```



### Handle the protocol defined by game

- Send: Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs: Sends packet to server and ends, without waiting for a response.

```c#
        // Packets to send
        var tapMsg = new Com.Nhn.Gameanvil.Sample.Protocol.TapMsg
        { tapCount
            Combo = tapCount,
            SelectCardName = UserInfo.Instance.CurrentDeck + "_0" + 1,
            TapScore = 100
        };

        // Packet to pass to the server as a data castle without response
        tapBirdUser.Send(new Packet(tapMsg));
```

- Request: Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs : Sends a request to the server and waits for a response.

```c#
        // The game user sends a request to the server and receives a response.
        gameUser.Request<Com.Nhn.Gameanvil.Sample.Protocol.ShuffleDeckRes>(shuffleDeckReq, (userAgent, shuffleDeckRes) =>
        {
            if (shuffleDeckRes.ResultCode == Com.Nhn.Gameanvil.Sample.Protocol.ErrorCode.None)
            {
                // Process on success
            }
            else
            {
                // Handle on failure
            }
        });
```

- Register a listener: Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs: Register a listener for packets pushed from the server to the client

```c#
        snakeGameUser.AddListener((UserAgent userAgent, Com.Nhn.Gameanvil.Sample.Protocol.SnakeFoodMsg msg) =>
        {
            if (msg != null)
            {
                if (msg.IsDelete)
                {
                    foreach (var currentFood in SnakeGameInfo.Instance.FoodList)
                    {
                        if (msg.FoodData.Idx == currentFood.transform.position.z)
                        {
                            Debug.Log("Delete Food !!!!! : " + currentFood.transform.position.z + ":" + currentFood.transform.position.x + ", " + currentFood.transform.position.y);
                            Destroy(currentFood);
                            SnakeGameInfo.Instance.FoodList.Remove(currentFood);
                            break;
                        }
                    }
                }
                else
                {
                    textFoodList.text = msg.FoodData.ToString();
                }
            }
        });
```
