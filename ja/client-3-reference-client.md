## Game > GameAnvil > クライアント開発ガイド > リファレンスクライアント

## ダウンロード

https://github.nhnent.com/game-server-engine/sample-game-client-unity.git

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-1.png)

## サンプル開発環境

- Unity3d: 2018.4.1f1
  - Unity Standaloneで製作しました。このサンプルは開発の参考用で、エディタ環境でのみ動作が確認されました。
- GameAnvilコネクタ: 1.1.0.0

## Connector API doc - C

- [Connector API C#](http://10.162.4.61:9090/csharp)

## 実行環境設定with Unity3d

- Git保存場所でクローンしたプロジェクトをUnity3dで実行します。
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-2.png)
- sample_game_client_unityの設定確認
  - GameAnvil Library - 必要なライブラリを確認
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

## Unity3dでクライアント実行

- Unity3d実行
  - StartSceneneを選択し、左下の画面のようにシーン(scene)を選択します。
    ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-4.png)
  - Unityプロジェクトの中のPlayボタンをクリックしてクライアントを実行します。
- クライアント正常動作を確認
  - Gamebaseと連携しており、起動するとすぐに初期化とゲストログイン処理が行われ、下に情報が表示されます。
  - log consoleにエラーがなければ正常動作

## sample_game_client_unityフォルダ構造を確認する

- ゲーム開発の参考用に作ったGameAnvilサンプルサーバーと連携するためのクライアントとして作られたプロジェクトです。
  - 機能ユーザビリティに主眼を置いており、ゲーム自体のエラーやバグが多いことがあります。
  - 気になる点がある場合は[サポート](https://alpha.toast.com/kr/support/inquiry)へお問い合わせください。

### Assets

#### GameAnvil：GameAnvilで使用するLibraryの位置



#### GameAnvilSample：GameAnvil Sampleフォルダ

- StartScene：開始画面。基本Gamebase初期化とゲストログイン処理、プラットフォームテストとゲームテストの分岐
- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-6.png)
- LoadingScene：画面が変わるたびに表示されるローディング画面
- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-7.png)

##### Data：データ性オブジェクトフォルダ

##### GameTest：ゲームテストフォルダ

###### Scenes：ゲームテスト画面フォルダ

- GameLoginScene ： IDを入力して全体的なログイン処理を行う画面
- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-8.png)
- GameLobbyScene：ユーザーがログインした後の画面、TapBird(1人)、マルチTapBird(4人)、Snake(2人)ゲームと、ランキング、ユーザー情報、ニックネーム変更を確認
- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-9.png)

##### PlatformTest ： GameAnvil API Testフォルダ

###### Scenes ： GameAnvil API画面フォルダ

- AuthScene ： launching(rest)、コネクタ、認証、ログイン処理を行う画面
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-10.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-11.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-12.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-13.png)
- LobbyScene：ログイン後、ゲームロビー画面、シングルゲーム、ルームマッチマルチ(4人)、ユーザーマッチ(2)、ランキング、シャッフルデッキ可能画面
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-14.png)
- MultiSnakeGameScene：ユーザーマッチ2人ゲーム画面
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-15.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-16.png)
- MultiTapBirdGameScene：ルームマッチ4人ゲーム画面
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-17.png)
- SingleGameScene：シングルルームゲーム画面
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-18.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-19.png)

##### Protocols：サーバーと通信するプロトコルフォルダ

##### Snake：Snakeゲームフォルダ、ユーザーマッチゲーム、2人同時にサーバーから送ったfoodを画面に表示し、ユーザーの移動値を表示、foodを食べた時の処理、ゲームend条件を判断

###### Scenes： Snakeゲーム画面フォルダ

![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-20.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-21.png)

##### TapBird：TapBirdゲームフォルダ、シングルゲーム& 4人まで最高スコアを記録するゲーム、一緒にプレイするユーザーのスコアを全て表示

###### Scenes：TapBirdゲーム画面フォルダ

![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-22.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-23.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-24.png)
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-25.png) ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceClient-26.png)



#### Gamebase： Gamebase用フォルダ



#### Plugins： IOS / Android用フォルダ



#### StreamingAssets： Gamebase使用フォルダ



#### TOAST： Gamebase使用フォルダ



## Sample Code

### ConnectHandler ： Assets/GameAnvilSample/

- GameAnvilコネクタ初期化、プロトコル登録Assets/GameAnvilSample/ConnectHandler.cs

```
connector = new GameAnvil.Connector(config);
            // コネクタログ追加
            connector.Logger += (level, log) =>
            {
                Debug.Log(string.Format("Log[{0}]:{1}", level, log));
            };
            connector.LvNetLogger += (level, log) =>
            {
                Debug.Log(string.Format("Net[{0}]:{1}", level, log));
            };

            // サーバーと同じ順序でプロトコル登録
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(0, Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReflection.Descriptor);
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(1, Com.Nhn.Gameanvil.Sample.Protocol.GameMultiReflection.Descriptor);
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(2, Com.Nhn.Gameanvil.Sample.Protocol.GameSingleReflection.Descriptor);
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(3, Com.Nhn.Gameanvil.Sample.Protocol.ResultReflection.Descriptor);
            GameAnvil.ProtocolManager.getInstance().RegisterProtocol(4, Com.Nhn.Gameanvil.Sample.Protocol.UserReflection.Descriptor);
```

### GameAnvil Connectorリスナー登録

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```
        // 接続が切れる部分処理リスナーの登録
        ConnectHandler.Instance.GetConnectionAgent().onDisconnectListeners += (SessionAgent sessionAgent, ResultCodeDisconnect result, bool force, Payload payload) =>
        {
            Debug.LogFormat("onDisconnect - {0}", result);
            UserInfo.Instance.MoveScene(Constants.SCENE_AUTH);
        };
```

### Connect

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```
        // サーバーに接続試行。
        ConnectHandler.Instance.GetConnectionAgent().Connect(textIP.text, int.Parse(textPort.text),
            (SessionAgent sessionAgent, ResultCodeConnect result) =>
            {
                Debug.Log("Connect " + textID.text + ":" + textPort.text + " " + result);

                if (result == ResultCodeConnect.CONNECT_SUCCESS)
                {
                    // 成功時、入力UI処理
                    buttonConnect.gameObject.SetActive(false);
                    buttonAuth.gameObject.SetActive(true);
                    textID.gameObject.SetActive(true);
                }
                else
                {
                    // サーバー接続失敗処理
                }
                buttonConnect.interactable = true;
            }
        );
```

### Auth

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```
            // 認証に必要なプロトコルデータ設定
            var authenticationReq = new Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReq
            {
                AccessToken = Constants.AUTH_ACCESS_TOKEN
            };
            Debug.Log("authenticationReq " + authenticationReq);

            // サーバーに認証試行。現在はデバイスID、ID、パスワードを全てuuid値で伝達
            ConnectHandler.Instance.GetConnectionAgent().Authenticate(inputFieldUUID.text, inputFieldID.text, inputFieldID.text, new Payload().add(new Packet(authenticationReq)),
                (SessionAgent sessionAgent, ResultCodeAuth result, List<SessionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) =>
                {
                    Debug.Log("Auth " + result);

                    // 成功の場合、次の段階に進む。
                    if (result == ResultCodeAuth.AUTH_SUCCESS)
                    {
                        // 成功時、入力UI処理
                    }
                    else
                    {
                        // 失敗時の処理
                    }
                }
            );
```

### Login

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```
        // ログインに必要なプロトコルデータ設定
        var loginReq = new Com.Nhn.Gameanvil.Sample.Protocol.LoginReq
        {
            // 任意で必要なデータ設定
            Uuid = inputFieldUUID.text,
            LoginType = Com.Nhn.Gameanvil.Sample.Protocol.LoginType.LoginGuest,
            AppVersion = Application.version,
            AppStore = "None",
            DeviceModel = SystemInfo.deviceModel,
            DeviceCountry = "KR",
            DeviceLanguage = "ko"

        };
        Debug.Log("loginReq " + loginReq);

        // サーバーにログイン
        ConnectHandler.Instance.CreateUserAgent(Constants.GAME_SPACE_NAME, string.Empty).Login(Constants.SPACE_USER_TYPE, string.Empty, new Payload().add(new Packet(loginReq)),
            (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) =>
            {
                Debug.Log("Login " + result + ", " + loginInfo);

                // 成功時、次の段階。
                if (result == ResultCodeLogin.LOGIN_SUCCESS)
                {
                    if (loginInfo.Payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>())
                    {
                        // ログインレスポンスプロトコル処理
                        Com.Nhn.Gameanvil.Sample.Protocol.LoginRes loginRes = Com.Nhn.Gameanvil.Sample.Protocol.LoginRes.Parser.ParseFrom(loginInfo.Payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>().GetBytes());
                        Debug.Log("LoginRes " + loginRes);

                        // サーバーから受け取ったゲームデータ設定
                        UserInfo.Instance.Uuid = inputFieldUUID.text;
                        UserInfo.Instance.Nickname = loginRes.Userdata.Nickname;
                        UserInfo.Instance.Heart = loginRes.Userdata.Heart;
                        UserInfo.Instance.Coin = loginRes.Userdata.Coin;
                        UserInfo.Instance.Ruby = loginRes.Userdata.Ruby;
                        UserInfo.Instance.Level = loginRes.Userdata.Level;
                        UserInfo.Instance.Exp = loginRes.Userdata.Exp;
                        UserInfo.Instance.HighScore = loginRes.Userdata.HighScore;
                        UserInfo.Instance.CurrentDeck = loginRes.Userdata.CurrentDeck;

                        // シーン切り替え時、ボタンリスナーを全て解除
                    }
                }
                else
                {
                    // 失敗処理
                }
            }
       );
```

### GetUserAgent

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```
        // コネクタからユーザーオブジェクトを保存
        gameUser = ConnectHandler.Instance.GetUserAgent(Constants.GAME_SPACE_NAME, string.Empty);
```

### CreateRoom：一人でゲームするルームの作成と入場

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```
        // 一人でゲームするルームの作成
        gameUser.CreateRoom(Constants.SPACE_ROOM_TYPE_SINGLE, new Payload().add(new Packet(startGameReq)), (UserAgent userAgent, ResultCodeCreateRoom result, string roomId, Payload payload) =>
        {
            Debug.Log("CreateRoom " + result);

            if (result == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
            {
                // 成功時、ゲームシーンに変更、登録されたリスナーを削除
            }
            else
            {
                // 失敗処理
            }
        });
```

### MatchRoom ：マルチルームマッチメイキング

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```
        // 作られたルームに入るマッチリクエスト - 一人でもプレイ可能。最大人数まで全員入場
        gameUser.MatchRoom(Constants.SPACE_ROOM_TYPE_MULTI_ROOM_MATCH, true, false, (UserAgent userAgent, ResultCodeMatchRoom result, string roomId, Payload payload) =>
        {
            Debug.Log("MatchRoom " + result);
            if (result == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                // 成功時、ゲームシーンに変更、登録されたリスナーを削除
            }
            else
            {
                // 失敗処理
            }
        });
```

### MatchUserStart ：マルチユーザーマッチメイキング

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```
        // 2人単位でマッチングしてルームを作り、同時に入場した後、ゲーム進行
        gameUser.MatchUserStart(Constants.SPACE_ROOM_TYPE_MULTI_USER_MATCH, (UserAgent userAgent, ResultCodeMatchUserStart result, Payload payload) =>
        {
            Debug.Log("MatchUser " + result);
            if (result == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
            {
                // 成功時、ゲームシーンに変更、登録されたリスナーを削除
            }
            else
            {
                // 失敗処理
            }
        });
```

- ユーザーがマッチする時、サーバーでonMatchUserDoneが呼び出されます。

```
        // タイミングの問題上、リスナーを事前に登録、ユーザーがゲームルームに入った時、ゲームレディflag設定
        gameUser.onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone result, bool created, string roomId, Payload payload) =>
        {
            Debug.Log("onMatchUserDoneListeners!!!!!! " + userAgent.GetUserId());

            if (result == ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS)
            {
                UserInfo.Instance.gameState = UserInfo.GameState.Wait;
            }
        };
```

- ユーザーマッチがタイムアウトすると、onMatchUserTimeoutが呼び出されます。  Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs

```
        // ユーザーマッチリクエストタイムアウトリスナー
        snakeGameUser.onMatchUserTimeoutListeners += (UserAgent userAgent) =>
        {
            Debug.Log("onMatchUserTimeoutListeners!!!!!! " + userAgent.GetUserId());

            // ロビーシーンに移動
        };
```

### LeaveRoom：ルームから出る時に呼び出し

- Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/MultiTapBirdGameUI.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs

```
        // ゲーム終了プロトコル定義
        var endGameReq = new Com.Nhn.Gameanvil.Sample.Protocol.EndGameReq
        {
            EndType = gameEndType
        };

        // ゲームルームを出るリクエスト
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

                // ロビーシーンに移動
            }
            else
            {
                // 失敗時の処理
            }
        });
```



### ゲームで定義したプロトコル処理

- Send: Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs：レスポンスを待機せず、サーバーにパケットを送って終える。

```
        // 送信するパケット
        var tapMsg = new Com.Nhn.Gameanvil.Sample.Protocol.TapMsg
        {
            Combo = tapCount,
            SelectCardName = UserInfo.Instance.CurrentDeck + "_0" + 1,
            TapScore = 100
        };

        // レスポンスせず、サーバーにデータ性で伝達するパケット
        tapBirdUser.Send(new Packet(tapMsg));
```

- Request: Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs：サーバーへリクエストを送り、レスポンスを受け取るまで待機。

```
        // ゲームユーザーがサーバーへrequestでresponseを受け取って処理する。
        gameUser.Request<Com.Nhn.Gameanvil.Sample.Protocol.ShuffleDeckRes>(shuffleDeckReq, (userAgent, shuffleDeckRes) =>
        {
            Debug.Log("shuffleDeckRes" + shuffleDeckRes);

            if (shuffleDeckRes.ResultCode == Com.Nhn.Gameanvil.Sample.Protocol.ErrorCode.None)
            {
                // シャッフルが成功し、サーバーにレスポンス値を更新
                UserInfo.Instance.CurrentDeck = shuffleDeckRes.Deck;
                UserInfo.Instance.Coin = shuffleDeckRes.BalanceCoin;
                UserInfo.Instance.Ruby = shuffleDeckRes.BalanceRuby;

                // ユーザー情報UI更新
            }
            else
            {
                // 失敗時の処理
            }
        });
```

- リスナー登録： Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs ：サーバー -> クライアントに来るパケットリスナーの登録。

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
