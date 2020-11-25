## Game > GameAnvil > 클라이언트 개발 가이드 > Reference Client 가이드
## 다운받기

* [https://github.nhnent.com/game-server-engine/sample-game-client-unity.git](https://github.nhnent.com/game-server-engine/sample-game-client-unity.git)
* ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-1.png)

## 샘플 개발 환경

* Unity3d : 2018.4.1f1
    * unity Standalone으로 제작되어 Editor환경에서만 동작 확인이 되었습니다.
* GameAnvil Connector : 1.0.0.0
* protocol : google protobuf 3.0

## Connector API doc - C#

* [Connector API C#](http://10.162.4.61:9090/csharp)

## 실행환경 설정 with Unity3d

* Git 저장소에서 클론 한 프로젝트를 Unity3d로 실행
* ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-2.png)
* sample\_game\_client\_unity 설정확인
    * GameAnvil Library - 필요한 라이브러리 확인
        * Assets/Plugins
            * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-3.png)
            * GameAnvil.dll
            * GameAnvil.xml
            * Google.Protobuf.dll
            * Google.Protobuf.xml
            * Google.Protobuf.pdb
            * link.xml
            * LZ4.dll
            * SingleServer.dll
            * SuperSocket.ClientEngine.dll
            * SuperSocket.ClientEngine.pdb
            * WebSocket4Net.dll
            * WebSocket4Net.pdb

## 클라이언트 실행 with Unity3d

* Unity3d 실행
* ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-4.png)
* ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-5.png)
* 클라이언트 정상 동작 확인
    * Gamebase 연동 되어 있어서 시작하자마자 초기화와 게스트 로그인 처리가 되어 아래에 정보가 표시된다.
    * log console에 에러가 없으면 정상 동작

## sample\_game\_client\_unity 살펴보기

* 게임개발에 참고 할 수있게 만든 GameAnvil샘플 서버와 연동하기 위한 클라이언트로 제작된 프로젝트 입니다.
    * 기능 사용성에 목적을 두어서 게임자체에대한 에러나 버그가 많을수 있습니다.
    * 참고하시고 불편하시거나 필요하거나 궁금한 부분은 언제든지 문의 해주세요.

### Assets

* StartScene : 처음 시작하는 신으로 기본 Gamebase초기화와 게스트로그인처리, 플랫폼테스트와 게임테스트 분기
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-6.png)
* LoadingScene : 신이 변경될때마다 보이는 로딩
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-7.png)

#### Data : 데이터성 객체

#### Gamebase : Gamebase용 폴더

#### GameTest : 게임테스트

* GameLoginScene : 아이디 입력 받아 전체적인 로그인 처리하는 화면
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-8.png)
* GameLobbyScene  : 유저가 로그인하고나서의 화면, TapBird(1인), 멀티TapBird(4인), Snake(2인) 게임과, 랭킹, 유저 정보, 닉네임 변경를 확인
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-9.png)

#### PlatformTest : GameAnvil API 테스트

* AuthScene ; launching(rest), connect, auth, login 처리하는 화면
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-10.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-11.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-12.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-13.png)
* LobbyScene : 로그인이후 게임 로비 화면, 싱글게임, 룸매치 멀티(4인), 유저매치(2), 랭킹, 셔플덱 가능 화면
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-14.png)
* MultiSnakeGameScene : 유저 매치 2인 게임 화면
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-15.png)  ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-16.png)
* MultiTapBirdGameScene : 룸매치 4명 게임 화면
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-17.png)
* SingleGameScene : 싱글룸 게임 화면
    * ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-18.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-19.png)

#### Plugins : GameAnvil에서 사용 하는 Library 위치

#### Protocols : 서버와 통신할 프로토콜,서버에서 사용한것과 같은 .proto 파일을 proto\_gen\_client.bat 파일을 사용해서 변환

#### Snake : 유저 매치 게임, 2인 동시에 서버에서 보내준 food를 화면에보여주고, 유저의 이동값을 표시, food먹었을때의처리, 게임 end조건 판단

* ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-20.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-21.png)

#### StreamingAssets : Gamebase용 폴더

#### TapBird : 싱글게임 & 4명까지 최고스코어를 기록하는 게임, 같이 게임하는 유저의 점수를 모두 표시

* ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-22.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-23.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-24.png)
* ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-25.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-26.png)

#### GameAnvil : 클라이언트용 connector 객체 설정

#### TOAST : Gamebase용 폴더

## Sample Code

### ConnectHandler : Assets/GameAnvil/

![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-27.png)

* GameAnvil Connector초기화 밎 프로토콜 등록 Assets/GameAnvil/ConnectHandler.cs

```C#
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

* Assets/PlatformTest/Scripts/AuthUi.cs

```C#
        // 연결 끊기는 부분 처리 리스너 등록
        ConnectHandler.Instance.GetConnectionAgent().onDisconnectListeners += (SessionAgent sessionAgent, ResultCodeDisconnect result, bool force, Payload payload) =>
        {
            Debug.LogFormat("onDisconnect - {0}", result);
            UserInfo.Instance.MoveScene(Constants.SCENE_AUTH);
        };
```

### Connect

* Assets/PlatformTest/Scripts/AuthUi.cs

```C#
        // 서버에 접속 시도.
        ConnectHandler.Instance.GetConnectionAgent().Connect(textIP.text, int.Parse(textPort.text),
            (SessionAgent sessionAgent, ResultCodeConnect result) =>
            {
                Debug.Log("Connect " + textID.text + ":" + textPort.text + " " + result);

                if (result == ResultCodeConnect.CONNECT_SUCCESS)
                {
                    // 성공시 입력 UI 처리
                    buttonConnect.gameObject.SetActive(false);
                    buttonAuth.gameObject.SetActive(true);
                    textID.gameObject.SetActive(true);
                }
                else
                {
                    // 서버 접속 실패 처리
                }
                buttonConnect.interactable = true;
            }
        );
```

### Auth

* Assets/PlatformTest/Scripts/AuthUi.cs

```C#
            // 인증에 필요한 프로토콜 데이터 설정
            var authenticationReq = new Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReq
            {
                AccessToken = Constants.AUTH_ACCESS_TOKEN
            };
            Debug.Log("authenticationReq " + authenticationReq);

            // 서버에 인증 시도. 현재는 deviceid, id, pw 모두 uuid값으로 전달
            ConnectHandler.Instance.GetConnectionAgent().Authenticate(inputFieldUUID.text, inputFieldID.text, inputFieldID.text, new Payload().add(new Packet(authenticationReq)),
                (SessionAgent sessionAgent, ResultCodeAuth result, List<SessionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) =>
                {
                    Debug.Log("Auth " + result);

                    // 성공인 경우 다음 단계로 진행.
                    if (result == ResultCodeAuth.AUTH_SUCCESS)
                    {
                        // 성공시 입력 UI 처리
                    }
                    else
                    {
                        // 실패시 처리
                    }
                }
            );
```

### Login

* Assets/PlatformTest/Scripts/AuthUi.cs

```C#
        // 로그인에 필요한 프로토콜 데이터 설정
        var loginReq = new Com.Nhn.Gameanvil.Sample.Protocol.LoginReq
        {
            // 임의로 필요한 데이터 설정
            Uuid = inputFieldUUID.text,
            LoginType = Com.Nhn.Gameanvil.Sample.Protocol.LoginType.LoginGuest,
            AppVersion = "0.0.1",
            AppStore = "None",
            DeviceModel = SystemInfo.deviceModel,
            DeviceCountry = "KR",
            DeviceLanguage = "ko"

        };
        Debug.Log("loginReq " + loginReq);

        // 서버에 로그인
        ConnectHandler.Instance.CreateUserAgent(Constants.GAME_SPACE_NAME, string.Empty).Login(Constants.SPACE_USER_TYPE, string.Empty, new Payload().add(new Packet(loginReq)),
            (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) =>
            {
                Debug.Log("Login " + result + ", " + loginInfo);

                // 성공시 다음 단계.
                if (result == ResultCodeLogin.LOGIN_SUCCESS)
                {
                    if (loginInfo.Payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>())
                    {
                        // 로그인 응답 프로토콜 처리
                        Com.Nhn.Gameanvil.Sample.Protocol.LoginRes loginRes = Com.Nhn.Gameanvil.Sample.Protocol.LoginRes.Parser.ParseFrom(loginInfo.Payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>().GetBytes());
                        Debug.Log("LoginRes " + loginRes);

                        // 서버에서 받은 게임 데이터 설정
                        UserInfo.Instance.Uuid = inputFieldUUID.text;
                        UserInfo.Instance.Nickname = loginRes.Userdata.Nickname;
                        UserInfo.Instance.Heart = loginRes.Userdata.Heart;
                        UserInfo.Instance.Coin = loginRes.Userdata.Coin;
                        UserInfo.Instance.Ruby = loginRes.Userdata.Ruby;
                        UserInfo.Instance.Level = loginRes.Userdata.Level;
                        UserInfo.Instance.Exp = loginRes.Userdata.Exp;
                        UserInfo.Instance.HighScore = loginRes.Userdata.HighScore;
                        UserInfo.Instance.CurrentDeck = loginRes.Userdata.CurrentDeck;

                        // 신전환신 버튼 리스너 모두 해재
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

* Assets/PlatformTest/Scripts/LobbyUi.cs

```C#
        // 커넥터로 부터 유저 객체 저장
        gameUser = ConnectHandler.Instance.GetUserAgent(Constants.GAME_SPACE_NAME, string.Empty);
```

### CreateRoom : 싱글룸 생성및 입장

* Assets/PlatformTest/Scripts/LobbyUi.cs

```C#
        // 게임 싱글룸 생성
        gameUser.CreateRoom(Constants.SPACE_ROOM_TYPE_SINGLE, new Payload().add(new Packet(startGameReq)), (UserAgent userAgent, ResultCodeCreateRoom result, string roomId, Payload payload) =>
        {
            Debug.Log("CreateRoom " + result);

            if (result == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
            {
                // 성공시 게임신으로 변경, 등록된 리스너 제거
            }
            else
            {
                // 실패 처리
            }
        });
```

### MatchRoom : 멀티 룸매치

* Assets/PlatformTest/Scripts/LobbyUi.cs

```C#
        // 만들어 져있는 방에 들어가는 룸매치 요청 - 혼자서도 플레이가 가능하다. 최대 인원수 까지 모두 입장
        gameUser.MatchRoom(Constants.SPACE_ROOM_TYPE_MULTI_ROOM_MATCH, true, false, (UserAgent userAgent, ResultCodeMatchRoom result, string roomId, Payload payload) =>
        {
            Debug.Log("MatchRoom " + result);
            if (result == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                // 성공시 게임신으로 변경, 등록된 리스너 제거
            }
            else
            {
                // 실패 처리
            }
        });
```

### MatchUserStart : 멀티 유저 매치

* Assets/PlatformTest/Scripts/LobbyUi.cs

```C#
        // 유저 매치 두명이서 방을 만들어서 동시에 입장해서 게임진행
        gameUser.MatchUserStart(Constants.SPACE_ROOM_TYPE_MULTI_USER_MATCH, (UserAgent userAgent, ResultCodeMatchUserStart result, Payload payload) =>
        {
            Debug.Log("MatchUser " + result);
            if (result == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
            {
                // 성공시 게임신으로 변경, 등록된 리스너 제거
            }
            else
            {
                // 실패 처리
            }
        });
```

* 유저가 매치될때 서버로 부터 onMatchUserDone이 호출된다.

```C#
        // 타이밍 이슈상 리스너를 미리 등록,  유저가 게임방에 들어 갔을때 게임 레디 flag설정
        gameUser.onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone result, bool created, string roomId, Payload payload) =>
        {
            Debug.Log("onMatchUserDoneListeners!!!!!! " + userAgent.GetUserId());

            if (result == ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS)
            {
                UserInfo.Instance.gameState = UserInfo.GameState.Wait;
            }
        };
```

* 유저 매치 타임아웃시 onMatchUserTimeout이 호출된다. : Assets/PlatformTest/Scripts/MultiSnakeGameUi.cs

```C#
        // 유저 매치 요청 타임 아웃 리스너
        snakeGameUser.onMatchUserTimeoutListeners += (UserAgent userAgent) =>
        {
            Debug.Log("onMatchUserTimeoutListeners!!!!!! " + userAgent.GetUserId());

            // 로비신으로 이동
        };
```

### LeaveRoom : 룸에서 나갈때 호출

* Assets/PlatformTest/Scripts/MultiSnakeGameUi.cs
* Assets/PlatformTest/Scripts/MultiTapBirdGameUI.cs
* Assets/PlatformTest/Scripts/SingleGameUi.cs

```C#
        // 게임종료 프로토콜 정의
        var endGameReq = new Com.Nhn.Gameanvil.Sample.Protocol.EndGameReq
        {
            EndType = gameEndType
        };

        // 게임룸 나가는 요청
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

                // 로비신으로 이동
            }
            else
            {
                // 실패시 처리
            }
        });
```

### 게임에서 정의한 프로토콜 처리

* Send : Assets/PlatformTest/Scripts/SingleGameUi.cs : 응답을 대기하지않고 서로보 패킷을 보내고 끝난다.

```C#
        // 전송할 패킷
        var tapMsg = new Com.Nhn.Gameanvil.Sample.Protocol.TapMsg
        {
            Combo = tapCount,
            SelectCardName = UserInfo.Instance.CurrentDeck + "_0" + 1,
            TapScore = 100
        };

        // 응답없이 서버로 데이터 성으로 전달하는 패킷
        tapBirdUser.Send(new Packet(tapMsg));
```

* Request : Assets/PlatformTest/Scripts/LobbyUi.cs : 서버로 요청을 보내고 응답을 받을때까지 대기

```C#
        // 게임유저가 서버로 request로 response를 받아 처리 한다.
        gameUser.Request<Com.Nhn.Gameanvil.Sample.Protocol.ShuffleDeckRes>(shuffleDeckReq, (userAgent, shuffleDeckRes) =>
        {
            Debug.Log("shuffleDeckRes" + shuffleDeckRes);

            if (shuffleDeckRes.ResultCode == Com.Nhn.Gameanvil.Sample.Protocol.ErrorCode.None)
            {
                // 서버에 셔플 성공 하고 응답값 갱신
                UserInfo.Instance.CurrentDeck = shuffleDeckRes.Deck;
                UserInfo.Instance.Coin = shuffleDeckRes.BalanceCoin;
                UserInfo.Instance.Ruby = shuffleDeckRes.BalanceRuby;

                // 유저 정보 UI 갱신
            }
            else
            {
                // 실패시 처리
            }
        });
```

* 리스너 등록 : Assets/PlatformTest/Scripts/MultiSnakeGameUi.cs : 서버 --> 클라이언트로 오는 패킷 리스너 등록

```C#
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