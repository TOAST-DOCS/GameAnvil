## Game > GameAnvil > 레퍼런스 프로젝트 > Unity 샘플

# 1. Connector API doc - C#

[Connector API C#](https://gameplatform.toast.com/docs/api/)



# 2. 클라이언트 다운로드

[Sample Game Client](https://github.com/nhn/gameanvil.sample-game-client-unity.git)



# 3. 구성 환경

* Unity3d: 2018.4.1f1

  - Unity Standalone으로 제작되었습니다. 본 샘플은 개발 참고용으로서 에디터환경에서만 동작이 확인되었습니다.

* GameAnvil 커넥터: 1.2.0.0


# 4. 클라이언트 구동하기

## Unity3d 사용

git 저장소에서 클론한거나 다운받은 프로젝트를 Unity3d로 실행합니다.

![reference-2-unity-01](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-01.png) 



### 실행 환경 확인

GameAnvil 커넥터 C# 라이브러리를 확인 합니다.

* Assets/GameAnvil

![reference-2-unity-02](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-02.png) 

	* GameAnvil
	* Google.ProtoBuf
	* K4os.Compression.LZ4
	* log4net
	* SuperSocket.ClientEngine
	* System.Buffers
	* System.Memory
	* System.Runtime.CompilerServices.Unsafe

GameAnvil.dll 파일을 파일 오른쪽 버튼으로 속성에서 버전정보가 원하는 버전인지 확인합니다.  



### 클라이언트 실행

유니티 에디터의 Play버튼으로 실행을 합니다. GameBase 와 연동이 되어 있어서 클라이언트 시작하자마자 초기화와 게스트 로그인 처리 되어 화면 아래쪽에 정보가 표시가 됩니다. 오른쪽에 로그 콘솔에 오류가 없다면 정상 시작된 상태입니다.

![reference-2-unity-03](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-03.png) 



# 5. 클라이언트 살펴 보기

## 클라이언트 프로젝트 구성

게임 개발에 참고할 수 있게 만든 GameAnvil 샘플 서버와 연동하기 위한 클라이언트로 제작된 프로젝트입니다.

- 기능 사용성에 목적을 두어서 게임 자체에 대한 에러나 버그가 많을 수 있습니다.
- 궁금한 점이 있다면 [고객센터](https://www.toast.com/kr/support/inquiry) 로 문의해 주시기 바랍니다.



### Assets
#### GameAnvil: GameAnvil에서 사용하는 Library 위치
#### GameAnvilSample: GameAnvil Sample 폴더
- StartScene: 처음 시작하는 화면으로 기본 Gamebase 초기화와 게스트 로그인 처리, 플랫폼 테스트와 게임 테스트 분기

  ![reference-2-unity-04](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-04.png) 

- LoadingScene: 화면이 변경될 때마다 보이는 로딩

  ![reference-2-unity-05](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-05.png) 

##### Data: 데이터성 객체 폴더
##### GameTest: 게임 테스트 폴더
###### Scenes: 게임 테스트 화면 폴더
- GameLoginScene : 아이디 입력 받아 전체적인 로그인 처리하는 화면

  ![reference-2-unity-06](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-06.png) 

- GameLobbyScene : 유저가 로그인하고나서의 화면, TapBird(1인), 멀티TapBird(4인), Snake(2인) 게임과, 랭킹, 유저 정보, 닉네임 변경를 확인

  ![reference-2-unity-07](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-07.png) 

##### PlatformTest : GameAnvil API Test 폴더
###### Scenes : GameAnvil API 화면 폴더
- AuthScene ; launching(rest), 커넥트, 인증, 로그인 처리하는 화면

  ![reference-2-unity-08](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-08.png)  ![reference-2-unity-09](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-09.png)  ![reference-2-unity-10](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-10.png)  ![reference-2-unity-11](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-11.png)

- LobbyScene: 로그인 이후 게임 로비 화면, 싱글 게임, 룸 매치 멀티(4인), 유저 매치(2), 랭킹, 셔플덱 가능 화면

  ![reference-2-unity-12](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-12.png) 

- MultiSnakeGameScene: 유저 매치 2인 게임 화면

  ![reference-2-unity-13](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-13.png) ![reference-2-unity-14](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-14.png)

- MultiTapBirdGameScene: 룸 매치 4명 게임 화면

  ![reference-2-unity-15](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-15.png) 

- SingleGameScene: 싱글 룸 게임 화면

  ![reference-2-unity-16](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-16.png)  ![reference-2-unity-17](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-17.png) 

##### Protocols: 서버와 통신할 프로토콜 폴더
##### Snake: Snake 게임 폴더, 유저 매치 게임, 2인 동시에 서버에서 보내준 food를 화면에보여주고, 유저의 이동값을 표시, food 먹었을 때의 처리, 게임 end 조건 판단
###### Scenes: Snake 게임 화면 폴더

​	![reference-2-unity-18](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-18.png) ![reference-2-unity-19](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-19.png)

##### TapBird: TapBird 게임 폴더, 싱글 게임 & 4명까지 최고 스코어를 기록하는 게임, 같이 게임하는 유저의 점수를 모두 표시
###### Scenes: TapBird 게임 화면 폴더

​	![reference-2-unity-20](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-20.png) ![reference-2-unity-21](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-21.png) ![reference-2-unity-22](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-22.png)

#### Gamebase: Gamebase용 폴더
#### Plugins: IOS / Android 용 폴더
#### StreamingAssets: Gamebase 사용 폴더
#### TOAST: Gamebase 사용 폴더



## 클라이언트 동작 내용

### ConnectHandler : Assets/GameAnvilSample/

- Assets/GameAnvilSample/ConnectHandler.cs
- GameAnvil 커넥터초기화 밎 프로토콜 등록 처리를 합니다.

```c#
connector = new GameAnvil.Connector(config);
// 커넥터 로그 추가
connector.Logger += (level, log) =>
{
    Debug.Log(string.Format("Log[{0}]:{1}", level, log));
};
connector.LvNetLogger += (level, log) =>
{
    Debug.Log(string.Format("Net[{0}]:{1}", level, log));
};

// 서버와 같은 순서로 프로토콜 등록
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(0, Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(1, Com.Nhn.Gameanvil.Sample.Protocol.GameMultiReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(2, Com.Nhn.Gameanvil.Sample.Protocol.GameSingleReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(3, Com.Nhn.Gameanvil.Sample.Protocol.ResultReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(4, Com.Nhn.Gameanvil.Sample.Protocol.UserReflection.Descriptor);
```

### GameAnvil Connector 리스너 등록

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
// 연결 끊기는 부분 처리 리스너 등록
ConnectHandler.Instance.GetConnectionAgent().onDisconnectListeners += (ConnectionAgent connectionAgent, ResultCodeDisconnect result, bool force, Payload payload) =>
{
    Debug.LogFormat("onDisconnect - {0}", result);
    // 연결이 끊어졌을때 필요한 처리
};
```

### Connect 

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
        // 서버에 접속 시도.
        ConnectHandler.Instance.GetConnectionAgent().Connect(textIP.text, int.Parse(textPort.text),
            (SessionAgent sessionAgent, ResultCodeConnect result) =>
            {
                Debug.Log("Connect " + textID.text + ":" + textPort.text + " " + result);

                if (result == ResultCodeConnect.CONNECT_SUCCESS)
                {
                    // 서버 접속 성공시 처리
                }
                else
                {
                    // 서버 접속 실패 처리
                }
            }
        );
```

### Auth 

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
            // 인증에 필요한 프로토콜 데이터 설정
            var authenticationReq = new Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReq
            {
                AccessToken = Constants.AUTH_ACCESS_TOKEN
            };

            // 서버에 인증 시도. 현재는 deviceid, id, pw 모두 uuid값으로 전달
            ConnectHandler.Instance.GetConnectionAgent().Authenticate(inputFieldUUID.text, inputFieldID.text, inputFieldID.text, new Payload().add(new Packet(authenticationReq)),
                (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) =>
                {
                    if (result == ResultCodeAuth.AUTH_SUCCESS)
                    {
                        // 성공시 처리
                    }
                    else
                    {
                        // 실패시 처리
                    }
                }
            );
```

### Login 

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
       // 로그인에 필요한 프로토콜 데이터 설정
        var loginReq = new Com.Nhn.Gameanvil.Sample.Protocol.LoginReq
        {
            // 필요한 데이터 설정
        };

        // 서버에 로그인
        ConnectHandler.Instance.CreateUserAgent(Constants.GAME_SPACE_NAME, Constants.userSubId).Login(Constants.SPACE_USER_TYPE, string.Empty, new Payload().add(new Packet(loginReq)),
            (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) =>
            {
                if (result == ResultCodeLogin.LOGIN_SUCCESS)
                {
                    if (loginInfo.Payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>())
                    {
                        // 로그인 응답 프로토콜 처리
                        Com.Nhn.Gameanvil.Sample.Protocol.LoginRes loginRes = Com.Nhn.Gameanvil.Sample.Protocol.LoginRes.Parser.ParseFrom(loginInfo.Payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>().GetBytes());

                        // 서버에서 받은 게임 데이터 설정

                        // 룸에 들어있는 상태 처리
                        if (loginInfo.isJoinedRoom)
                        {
                            if (loginInfo.RoomPayload.contains<Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg>())
                            {
                                Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg roomInfoMsg = Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg.Parser.ParseFrom(loginInfo.RoomPayload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg>().GetBytes());
                                // 룸 타입에 따른 처리
                                if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomSingle)
                                {
                                    // 싱글
                                }
                                else if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomSnake)
                                {
                                    // 유저매치 
                                }
                                else if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomTap)
                                {
                                    // 룸 매치
                                }
                            }
                        }
                    }
                }
                else
                {
                    // 실패 처리
                }
            }
       );
```

### GetUserAgent

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
// 커넥터로부터 유저 객체 저장
gameUser = ConnectHandler.Instance.GetUserAgent(Constants.GAME_SPACE_NAME, string.Empty);
```

### CreateRoom : 혼자 게임 하는 방 생성및 입장

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // 혼자 게임 하는 방 생성
        gameUser.CreateRoom(Constants.SPACE_ROOM_TYPE_SINGLE, new Payload().add(new Packet(startGameReq)), (UserAgent userAgent, ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) =>
        {
            if (result == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
            {
                // 성공처리
            }
            else
            {
                // 실패 처리
            }
        });
```

### MatchRoom : 멀티 룸 매치 메이킹

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // 만들어진 룸에 들어가는 매치 요청 - 혼자서도 플레이 가능. 최대 인원수까지 모두 입장
        gameUser.MatchRoom(Constants.SPACE_ROOM_TYPE_MULTI_ROOM_MATCH, "UNLIMITED_TAP", true, false, (UserAgent userAgent, ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload) =>
        {
            if (result == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                // 성공시 처리
            }
            else
            {
                // 실패 처리
            }
        });
```

### MatchUserStart : 멀티 유저 매치메이킹

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // 2명 단위로 매칭하여 룸을 만들고 동시에 입장한 후 게임 진행
        gameUser.MatchUserStart(Constants.SPACE_ROOM_TYPE_MULTI_USER_MATCH, "SNAKE", (UserAgent userAgent, ResultCodeMatchUserStart result, Payload payload) =>
        {
            if (result == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
            {
                // 성공시 처리
            }
            else
            {
                // 실패 처리
            }
        });
```

- 유저가 매치될 때 서버로부터 onMatchUserDone이 호출됩니다.

```c#
        // 리스너를 미리 등록, 유저가 게임 룸에 들어갔을 때 게임 레디 flag 설정
        gameUser.onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone result, bool created, int roomId, Payload payload) =>
        {
            if (result == ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS)
            {
				// 성공시 처리
            }
        };
```

- 유저 매치 타임아웃 시 onMatchUserTimeout이 호출됩니다.  Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs

```c#
        // 유저 매치 요청 타임 아웃 리스너
        snakeGameUser.onMatchUserTimeoutListeners += (UserAgent userAgent) =>
        {
        	// 유저매치 타임아웃 처리
        };
```

### LeaveRoom: 룸에서 나갈 때 호출

- Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/MultiTapBirdGameUI.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs

```c#
     // 게임종료 프로토콜 정의
        var endGameReq = new Com.Nhn.Gameanvil.Sample.Protocol.EndGameReq
        {
            EndType = gameEndType
        };

        // 게임룸 나가는 요청
        tapBirdUser.LeaveRoom(new Payload().add(new Packet(endGameReq)), (UserAgent userAgent, ResultCodeLeaveRoom result, bool force, int roomId, Payload payload) =>
        {
            if (result == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
            {
            	// 성공 처리, 응답 받은 메세지 처리
                if (payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes>())
                {
                    Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes endGameRes = Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes.Parser.ParseFrom(payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes>().GetBytes());
C#                }
            }
            else
            {
                // 실패시 처리
            }
        });
```



### 게임에서 정의한 프로토콜 처리

- Send: Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs : 응답을 대기하지 않고 서버로 패킷을 보내고 끝난다.

```c#
        // 전송할 패킷
        var tapMsg = new Com.Nhn.Gameanvil.Sample.Protocol.TapMsg
        {
            Combo = tapCount,
            SelectCardName = UserInfo.Instance.CurrentDeck + "_0" + 1,
            TapScore = 100
        };

        // 응답 없이 서버로 데이터 성으로 전달하는 패킷
        tapBirdUser.Send(new Packet(tapMsg));
```

- Request: Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs : 서버로 요청을 보내고 응답을 받을 때까지 대기.

```c#
        // 게임 유저가 서버로 request로 response를 받아 처리한다.
        gameUser.Request<Com.Nhn.Gameanvil.Sample.Protocol.ShuffleDeckRes>(shuffleDeckReq, (userAgent, shuffleDeckRes) =>
        {
            if (shuffleDeckRes.ResultCode == Com.Nhn.Gameanvil.Sample.Protocol.ErrorCode.None)
            {
                // 성공시 처리
            }
            else
            {
                // 실패 시 처리
            }
        });
```

- 리스너 등록: Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs : 서버에서 클라이언트로 오는 푸쉬해주는 패킷 리스너 등록

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

######  