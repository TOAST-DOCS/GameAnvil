## Game > GameAnvil > Client Development Guide > Reference Client

## Download

https://github.nhnent.com/game-server-engine/sample-game-client-unity.git

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-1.png)

## Sample development environment

- Unity3d: 2018.4.1f1
  - It is created using Unity Standalone. This sample is for developers' reference and works only in the editor.
- GameAnvil Connector: 1.1.0.0

## Connector API doc - C

- [Connector API C#](http://10.162.4.61:9090/csharp)

## Setting run-time environment with Unity3d

- Execute the project cloned from the Git storage using Unity3d. 
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-2.png)
- Check the sample_game_client_unity settings
  - GGameAnvil Library -  Check the necessary library
    - Assets/GameAnvil
      - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-3.png)
      - GameAnvil
      - Google.Protobuf
      - K4os.Compression.LZ4
      - log4net
      - SingleServer
      - SuperSocket.ClientEngine
      - System.Buffers
      - System.Memory
      - System.Runtime.CompilerServices.Unsafe

## Run the client using Unity3d

- Run Unity3d
  - Select a (scene) by selecting StartScene as in the screen at the bottom left. 
    ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-4.png)
  - Run the client by clicking the Play button located in the middle of the Unity project.
- Check if the client runs normally
  - As it is linked to Gamebase, when the client starts, the information below will be displayed after the initialization and guest login.
  - It runs normally if no error occurs from log console

## Exploring the sample_game_client_unity Folder Structure

- It is a project created using the client for linking to GameAnvil sample server, which is created for reference purposes while developing a game.
  - As it focuses on functionality, it may contain a number of game errors or bugs.
  - If you have any questions, contact our [customer center](https://alpha.toast.com/kr/support/inquiry).

### Assets

#### GameAnvil: The library location used by GameAnvil



#### GameAnvilSample: The GameAnvil Sample folder

- StartScene: The initial screen. It branches out to default Gamebase initialization, guest login process,, platform test, or game test.
- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-6.png)
- LoadingScene: The loading screen displayed when the screen is changed
- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-7.png)

##### Data: Data object folder

##### GameTest: The game test folder

###### Scenes: The game test screen folder

- GameLoginScene: The screen in which ID is entered and overall login is processed
- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-8.png)
- GameLobbyScene: The screen displayed after the user is logged in. Can check the TapBird (single player), Multiplayer TapBird (4-player), Snake (2-player) game, rankings, user information and nickname change from here.
- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-9.png)

##### PlatformTest: The GameAnvil API Test folder

###### Scenes: The GameAnvil API screen folder

- AuthScene; processing screen for launching(rest), connect, authentication, and login
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-10.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-11.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-12.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-13.png)
- LobbyScene: The game lobby screen after login, single game, room match multi (4 player), user match(2), rankings, shuffle deck screen 
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-14.png)
- MultiSnakeGameScene: User match 2-player game screen 
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-15.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-16.png)
- MultiTapBirdGameScene: Room match 4-player game screen 
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-17.png)
- SingleGameScene: Single room game screen 
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-18.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-19.png)

##### Protocols: The protocol folder to communicate with server

##### Snake: Snake game folder, user match game, displays the food sent by the server on the screen for the 2 players, displays the movement value of the user, process when the player eats food, checks game end conditions

###### Scenes: The Snake game screen folder

![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-20.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-21.png)

##### TapBird: The TapBird game folder, single game & records up to 4 players' high score, displays the scores of all the players in the game

###### Scenes: The TapBird game screen folder

![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-22.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-23.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-24.png)
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-25.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-26.png)



#### Gamebase: A folder for Gamebase



#### Plugins: A folder for IOS/Android



#### StreamingAssets: The folder that uses Gamebase



#### TOAST: The folder that uses Gamebase



## Sample Code

### ConnectHandler : Assets/GameAnvilSample/

- Initializes the GameAnvil connector and registers protocol Assets/GameAnvilSample/ConnectHandler.cs

```
connector = new GameAnvil.Connector(config);
            // Adding Connector Log
            connector.Logger += (level, log) =>
            {
                Debug.Log(string.Format("Log[{0}]:{1}", level, log));
            };
            connector.LvNetLogger += (level, log) =>
            {
                Debug.Log(string.Format("Net[{0}]:{1}", level, log));
            };

            // Registers protocol in the same order as servers
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(0, Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReflection.Descriptor);
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(1, Com.Nhn.Gameanvil.Sample.Protocol.GameMultiReflection.Descriptor);
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(2, Com.Nhn.Gameanvil.Sample.Protocol.GameSingleReflection.Descriptor);
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(3, Com.Nhn.Gameanvil.Sample.Protocol.ResultReflection.Descriptor);
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(4, Com.Nhn.Gameanvil.Sample.Protocol.UserReflection.Descriptor);
```

### Registers the GameAnvil Connector listener

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```
        // Registers the listener that is used to process the disconnected parts
        ConnectHandler.Instance.GetConnectionAgent().onDisconnectListeners += (SessionAgent sessionAgent, ResultCodeDisconnect result, bool force, Payload payload) =>
        {
            Debug.LogFormat("onDisconnect - {0}", result);
            UserInfo.Instance.MoveScene(Constants.SCENE_AUTH);
        };
```

### Connect

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```
        // Tries to connect to server.
        ConnectHandler.Instance.GetConnectionAgent().Connect(textIP.text, int.Parse(textPort.text),
            (SessionAgent sessionAgent, ResultCodeConnect result) =>
            {
                Debug.Log("Connect " + textID.text + ":" + textPort.text + " " + result);

                if (result == ResultCodeConnect.CONNECT_SUCCESS)
                {
                    // Handles input UI when successful
                    buttonConnect.gameObject.SetActive(false);
                    buttonAuth.gameObject.SetActive(true);
                    textID.gameObject.SetActive(true);
                }
                else
                {
                    // Handles server connection failure
                }
                buttonConnect.interactable = true;
            }
        );
```

### Auth

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```
            // Sets the protocol data needed to authenticate
            var authenticationReq = new Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReq
            {
                AccessToken = Constants.AUTH_ACCESS_TOKEN
            };
            Debug.Log("authenticationReq " + authenticationReq);

            // Attempts server authentication. Currently, device ID, ID, password are passed as the UUID value
            ConnectHandler.Instance.GetConnectionAgent().Authenticate(inputFieldUUID.text, inputFieldID.text, inputFieldID.text, new Payload().add(new Packet(authenticationReq)),
                (SessionAgent sessionAgent, ResultCodeAuth result, List<SessionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) =>
                {
                    Debug.Log("Auth " + result);

                    // Moves to the next step if successful.
                    if (result == ResultCodeAuth.AUTH_SUCCESS)
                    {
                        // Handles input UI when successful
                    }
                    else
                    {
                        // Handles failure
                    }
                }
            );
```

### Login

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```
        // Sets the protocol data needed to log in
        var loginReq = new Com.Nhn.Gameanvil.Sample.Protocol.LoginReq
        {
            // Sets arbitrarily required data
            Uuid = inputFieldUUID.text,
            LoginType = Com.Nhn.Gameanvil.Sample.Protocol.LoginType.LoginGuest,
            AppVersion = Application.version,
            AppStore = "None",
            DeviceModel = SystemInfo.deviceModel,
            DeviceCountry = "KR",
            DeviceLanguage = "ko"

        };
        Debug.Log("loginReq " + loginReq);

        // Logs into the server
        ConnectHandler.Instance.CreateUserAgent(Constants.GAME_SPACE_NAME, string.Empty).Login(Constants.SPACE_USER_TYPE, string.Empty, new Payload().add(new Packet(loginReq)),
            (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) =>
            {
                Debug.Log("Login " + result + ", " + loginInfo);

                // Moves to the next step if successful.
                if (result == ResultCodeLogin.LOGIN_SUCCESS)
                {
                    if (loginInfo.Payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>())
                    {
                        // Handles login response protocol
                        Com.Nhn.Gameanvil.Sample.Protocol.LoginRes loginRes = Com.Nhn.Gameanvil.Sample.Protocol.LoginRes.Parser.ParseFrom(loginInfo.Payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>().GetBytes());
                        Debug.Log("LoginRes " + loginRes);

                        // Sets the game data received from server
                        UserInfo.Instance.Uuid = inputFieldUUID.text;
                        UserInfo.Instance.Nickname = loginRes.Userdata.Nickname;
                        UserInfo.Instance.Heart = loginRes.Userdata.Heart;
                        UserInfo.Instance.Coin = loginRes.Userdata.Coin;
                        UserInfo.Instance.Ruby = loginRes.Userdata.Ruby;
                        UserInfo.Instance.Level = loginRes.Userdata.Level;
                        UserInfo.Instance.Exp = loginRes.Userdata.Exp;
                        UserInfo.Instance.HighScore = loginRes.Userdata.HighScore;
                        UserInfo.Instance.CurrentDeck = loginRes.Userdata.CurrentDeck;

                        // Clears all button listeners when switching scenes
                    }
                }
                else
                {
                    // Handles failure
                }
            }
       );
```

### GetUserAgent

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```
        // Stores the user object from connector
        gameUser = ConnectHandler.Instance.GetUserAgent(Constants.GAME_SPACE_NAME, string.Empty);
```

### CreateRoom : Create and enter a single-player room

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```
        // Create a single-player room
        gameUser.CreateRoom(Constants.SPACE_ROOM_TYPE_SINGLE, new Payload().add(new Packet(startGameReq)), (UserAgent userAgent, ResultCodeCreateRoom result, string roomId, Payload payload) =>
        {
            Debug.Log("CreateRoom " + result);

            if (result == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
            {
                // Switches to the game scene if successful, removes registered listeners
            }
            else
            {
                // Handles failure
            }
        });
```

### MatchRoom : Multiplayer room matchmaking

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```
        // A match request to enter an existing room - Can be played alone. Players enter until the room is full
        gameUser.MatchRoom(Constants.SPACE_ROOM_TYPE_MULTI_ROOM_MATCH, true, false, (UserAgent userAgent, ResultCodeMatchRoom result, string roomId, Payload payload) =>
        {
            Debug.Log("MatchRoom " + result);
            if (result == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                // Switches to the game scene if successful, removes registered listeners
            }
            else
            {
                // Handles failure
            }
        });
```

### MatchUserStart : Multiplayer matchmaking

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```
        // Matches 2 players, creates a room, makes them enter the room at the same time, and starts the game
        gameUser.MatchUserStart(Constants.SPACE_ROOM_TYPE_MULTI_USER_MATCH, (UserAgent userAgent, ResultCodeMatchUserStart result, Payload payload) =>
        {
            Debug.Log("MatchUser " + result);
            if (result == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
            {
                // Switches to the game scene if successful, removes registered listeners
            }
            else
            {
                // Handles failure
            }
        });
```

- onMatchUserDone is called from server when users are matched.

```
        // Due to timing issues, listener must be pre-registered and the game ready flag is set when users enter a game room
        gameUser.onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone result, bool created, string roomId, Payload payload) =>
        {
            Debug.Log("onMatchUserDoneListeners!!!!!! " + userAgent.GetUserId());

            if (result == ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS)
            {
                UserInfo.Instance.gameState = UserInfo.GameState.Wait;
            }
        };
```

- onMatchUserTimeout is called when user match is timed out.  Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs

```
        // User match request timeout listener
        snakeGameUser.onMatchUserTimeoutListeners += (UserAgent userAgent) =>
        {
            Debug.Log("onMatchUserTimeoutListeners!!!!!! " + userAgent.GetUserId());

            // moves to the lobby scene
        };
```

### LeaveRoom: Called when the user leaves the room

- Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/MultiTapBirdGameUI.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs

```
        // Defines the game over protocol
        var endGameReq = new Com.Nhn.Gameanvil.Sample.Protocol.EndGameReq
        {
            EndType = gameEndType
        };

        // A request to leave the room
        tapBirdUser.LeaveRoom(new Payload().add(new Packet(endGameReq)), (UserAgent userAgent, ResultCodeLeaveRoom result, bool force, string roomId, Payload payload) =>
        {
            Debug.Log("LeaveRoom " + result);

            if (result == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
            {
                if (payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes>())
                {
                    Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes endGameRes = Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes.Parser.ParseFrom(payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes>().GetBytes());

                    UserInfo.Instance.Heart = endGameRes.UserData.Heart;
                    UserInfo.Instance.TotalScore = endGameRes.TotalScore;

                    UserInfo.Instance.Coin = endGameRes.UserData.Coin;
                    UserInfo.Instance.CurrentDeck = endGameRes.UserData.CurrentDeck;
                    UserInfo.Instance.Exp = endGameRes.UserData.Exp;
                    UserInfo.Instance.HighScore = endGameRes.UserData.HighScore;
                    UserInfo.Instance.Level = endGameRes.UserData.Level;
                    UserInfo.Instance.Nickname = endGameRes.UserData.Nickname;
                    UserInfo.Instance.Ruby = endGameRes.UserData.Ruby;

                    Debug.Log("GameResult " + endGameRes);
                }

                // moves to the lobby scene
            }
            else
            {
                // Handles failure
            }
        });
```



### Handles the protocol defined by game

- Send: Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs : Sends packet to server and ends, without waiting for a response.

```
        // A packet to be transferred
        var tapMsg = new Com.Nhn.Gameanvil.Sample.Protocol.TapMsg
        {
            Combo = tapCount,
            SelectCardName = UserInfo.Instance.CurrentDeck + "_0" + 1,
            TapScore = 100
        };

        // A packet that is used to data to the server without a response
        tapBirdUser.Send(new Packet(tapMsg));
```

- Request: Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs : Sends a request to server and waits until it receives a response.

```
        // The game user receives response from server as a request and handles it.
        gameUser.Request<Com.Nhn.Gameanvil.Sample.Protocol.ShuffleDeckRes>(shuffleDeckReq, (userAgent, shuffleDeckRes) =>
        {
            Debug.Log("shuffleDeckRes" + shuffleDeckRes);

            if (shuffleDeckRes.ResultCode == Com.Nhn.Gameanvil.Sample.Protocol.ErrorCode.None)
            {
                // Server successfully shuffles and updates the response value
                UserInfo.Instance.CurrentDeck = shuffleDeckRes.Deck;
                UserInfo.Instance.Coin = shuffleDeckRes.BalanceCoin;
                UserInfo.Instance.Ruby = shuffleDeckRes.BalanceRuby;

                // Updates user information UI
            }
            else
            {
                // Handles failure
            }
        });
```

- Register listener: Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs : Registers the packet listener coming to server -> client.

```
        snakeGameUser.AddListener((UserAgent userAgent, Com.Nhn.Gameanvil.Sample.Protocol.SnakeFoodMsg msg) =>
        {
            if (msg != null)
            {
                Debug.Log("<<<< SnakeFoodMsg!!!!!! : " + msg);

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

######  
