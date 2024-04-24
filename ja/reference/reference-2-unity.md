## Game > GameAnvil > リファレンスプロジェクト > Unityサンプル

# Connector API doc - C#

[Connector API C#](https://gameplatform.toast.com/docs/api/)



# クライアントダウンロード

[Sample Game Client](https://github.com/nhn/gameanvil.sample-game-client-unity.git)



# 構成環境

* Unity3d: 2020.3.37f1

  - Unity Standaloneで製作されました。本サンプルは開発参考用で、エディタ環境でのみ動作が確認されています。

* GameAnvilコネクタ: 1.4.1


# クライアントを起動する

## Unity3dを使う

gitリポジトリでクローン(clone)したり、ダウンロードしたプロジェクトをUnityで実行します。

![reference-2-unity-01](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-01.png) 



### 実行環境の確認

GameAnvilコネクタC#ライブラリを確認します。

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

GameAnvil.dllファイルを右クリックした後、プロパティでバージョン情報が希望するバージョンであることを確認します。



### クライアント実行

Unity EditorのPlayボタンをクリックして実行します。右側のログコンソールにエラーがなければ、正常に起動した状態です。

![reference-2-unity-03](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-03.png) 



## クライアントを確認する

## クライアントのプロジェクト構成

ゲーム開発の参考用にGameAnvilサンプルサーバーと連動するためのクライアントとして作られたプロジェクトです。

- 機能の使いやすさを目的にしているので,ゲーム自体に対するエラーやバグが多い場合があります。
- 詳細は[サポート](https://www.nhncloud.com/kr/support/inquiry)にお問い合わせください。



* Assets
  * GameAnvil: GameAnvilで使用するLibraryの位置
  * GameAnvilSample: GameAnvil Sampleフォルダ
    * StartScene:最初に起動する画面で、基本的なGamebase初期化とゲストログイン処理、プラットフォームテストとゲームテスト分岐

      ![reference-2-unity-04](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-04.png) 

    * LoadingScene:画面が変更されるたびに表示されるローディング画面。

      ![reference-2-unity-05](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-05.png) 

    * Data:データ性オブジェクトのフォルダ
    * GameTest:ゲームテストフォルダ
      * Scenes:ゲームテスト画面フォルダ
        * GameLoginScene: ID入力を受け、全体的なログインを処理する画面

          ![reference-2-unity-06](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-06.png) 

        * GameLobbyScene:ユーザーがログインした後の画面、TapBird(1人)、マルチTapBird(4人)、Snake(2人)のゲームと、ランキング、ユーザー情報、ニックネームの変更を確認

          ![reference-2-unity-07](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-07.png) 

    * PlatformTest: GameAnvil API Testフォルダ
      * Scenes: GameAnvil API画面フォルダ
        * AuthScene ; launching(rest)、コネクト、認証、ログインを処理する画面

          ![reference-2-unity-08](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-08.png)  ![reference-2-unity-09](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-09.png)  ![reference-2-unity-10](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-10.png)  ![reference-2-unity-11](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-11.png)

        * LobbyScene:ログイン後のゲームロビー画面、シングルゲーム、ルームマッチマルチ(4人)、ユーザーマッチ(2)、ランキング、シャッフルデッキ可能画面

          ![reference-2-unity-12](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-12.png) 

        * MultiSnakeGameScene:ユーザーマッチ2人用ゲーム画面

          ![reference-2-unity-13](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-13.png) ![reference-2-unity-14](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-14.png)

        * MultiTapBirdGameScene:ルームマッチ4人ゲーム画面

          ![reference-2-unity-15](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-15.png) 

        * SingleGameScene:シングルルームゲーム画面

          ![reference-2-unity-16](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-16.png)  ![reference-2-unity-17](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-17.png) 

    * Protocols:サーバーと通信するプロトコルフォルダ
    * Snake: Snakeゲームフォルダ、ユーザーマッチゲーム、2人同時にサーバーから送られてきたfoodを画面に表示、ユーザーの移動値を表示、foodを食べた時の処理、ゲームend条件の判断
      * Scenes: Snakeゲーム画面フォルダ
        
        ![reference-2-unity-18](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-18.png) ![reference-2-unity-19](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-19.png)

    * TapBird: TapBirdゲームフォルダ、シングルゲーム＆4人までの最高スコアを記録するゲーム、一緒にプレイしているユーザーのスコアを全て表示
      * Scenes: TapBirdゲーム画面フォルダ
        
        ![reference-2-unity-20](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-20.png) ![reference-2-unity-21](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-21.png) ![reference-2-unity-22](https://static.toastoven.net/prod_gameanvil/images/reference/reference-2-unity-22.png)

  * Plugins: IOS / Android用フォルダ



## クライアントの動作内容

### ConnectHandler : Assets/GameAnvilSample/

- Assets/GameAnvilSample/ConnectHandler.cs
- GameAnvilコネクタの初期化とプロトコル登録処理を行います。

```c#
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

// サーバーと同じ順番でプロトコル登録
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.GameMultiReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.GameSingleReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.ResultReflection.Descriptor);
GameAnvil.ProtocolManager.getInstance().RegisterProtocol(Com.Nhn.Gameanvil.Sample.Protocol.UserReflection.Descriptor);
```

### GameAnvil Connectorリスナーの登録

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
// 接続が切れる部分処理リスナー登録
ConnectHandler.Instance.GetConnectionAgent().onDisconnectListeners += (ConnectionAgent connectionAgent, ResultCodeDisconnect result, bool force, Payload payload) =>
{
    Debug.LogFormat("onDisconnect - {0}", result);
    // 接続が切れた時に必要な処理
};
```

### Connect 

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
        // サーバーに接続試行。
        ConnectHandler.Instance.GetConnectionAgent().Connect(textIP.text, int.Parse(textPort.text),
            (SessionAgent sessionAgent, ResultCodeConnect result) =>
            {
                Debug.Log("Connect " + textID.text + ":" + textPort.text + " " + result);

                if (result == ResultCodeConnect.CONNECT_SUCCESS)
                {
                    // サーバー接続成功時の処理
                }
                else
                {
                    // サーバー接続失敗時の処理
                }
            }
        );
```

### Auth 

- Assets/GameAnvilSample/PlatformTest/Scripts/AuthUi.cs

```c#
            // 認証に必要なプロトコルデータの設定
            var authenticationReq = new Com.Nhn.Gameanvil.Sample.Protocol.AuthenticationReq
            {
                AccessToken = Constants.AUTH_ACCESS_TOKEN
            };

            // サーバーに認証試行。現在はdeviceid, id, pw全てuuid値で伝達
            ConnectHandler.Instance.GetConnectionAgent().Authenticate(inputFieldUUID.text, inputFieldID.text, inputFieldID.text, new Payload().add(new Packet(authenticationReq)),
                (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) =>
                {
                    if (result == ResultCodeAuth.AUTH_SUCCESS)
                    {
                        // 成功時の処理
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

```c#
       // ログインに必要なプロトコルデータの設定
        var loginReq = new Com.Nhn.Gameanvil.Sample.Protocol.LoginReq
        {
            // 必要なデータ設定
        };

        // サーバーにログイン
        ConnectHandler.Instance.CreateUserAgent(Constants.GAME_SPACE_NAME, Constants.userSubId).Login(Constants.SPACE_USER_TYPE, string.Empty, new Payload().add(new Packet(loginReq)),
            (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) =>
            {
                if (result == ResultCodeLogin.LOGIN_SUCCESS)
                {
                    if (loginInfo.Payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>())
                    {
                        // ログインレスポンスプロトコル処理
                        Com.Nhn.Gameanvil.Sample.Protocol.LoginRes loginRes = Com.Nhn.Gameanvil.Sample.Protocol.LoginRes.Parser.ParseFrom(loginInfo.Payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.LoginRes>().GetBytes());

                        // サーバーから受け取ったゲームデータの設定

                        // ルームに入った状態の処理
                        if (loginInfo.isJoinedRoom)
                        {
                            if (loginInfo.RoomPayload.contains<Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg>())
                            {
                                Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg roomInfoMsg = Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg.Parser.ParseFrom(loginInfo.RoomPayload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.RoomInfoMsg>().GetBytes());
                                // ルームタイプに応じた処理
                                if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomSingle)
                                {
                                    // シングル
                                }
                                else if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomSnake)
                                {
                                    // ユーザーマッチ 
                                }
                                else if (roomInfoMsg.RoomType == Com.Nhn.Gameanvil.Sample.Protocol.RoomType.RoomTap)
                                {
                                    // ルームマッチ
                                }
                            }
                        }
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

```c#
// コネクタからユーザーオブジェクトを保存
gameUser = ConnectHandler.Instance.GetUserAgent(Constants.GAME_SPACE_NAME, string.Empty);
```

### CreateRoom: 1人でゲームをするルームの作成と入場

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // 1人でゲームするルームの生成
        gameUser.CreateRoom(Constants.SPACE_ROOM_TYPE_SINGLE, new Payload().add(new Packet(startGameReq)), (UserAgent userAgent, ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) =>
        {
            if (result == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
            {
                // 成功処理
            }
            else
            {
                // 失敗処理
            }
        });
```

### MatchRoom:マルチルームマッチメイキング

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // 作られたルームに入るマッチリクエスト - 1人でもプレイ可能。最大人数まで全員入場
        gameUser.MatchRoom(Constants.SPACE_ROOM_TYPE_MULTI_ROOM_MATCH, "UNLIMITED_TAP", true, false, (UserAgent userAgent, ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload) =>
        {
            if (result == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                // 成功時の処理
            }
            else
            {
                // 失敗処理
            }
        });
```

### MatchUserStart:マルチユーザーマッチメイキング

- Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs

```c#
        // 2人単位でマッチングしてルームを作成し、同時に入場してゲームを進行
        gameUser.MatchUserStart(Constants.SPACE_ROOM_TYPE_MULTI_USER_MATCH, "SNAKE", (UserAgent userAgent, ResultCodeMatchUserStart result, Payload payload) =>
        {
            if (result == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
            {
                // 成功時の処理
            }
            else
            {
                // 失敗処理
            }
        });
```

- ユーザーがマッチされると、サーバーからonMatchUserDoneが呼び出されます。

```c#
        // リスナーを事前に登録し、ユーザーがゲームルームに入った時にゲームレディflagを設定
        gameUser.onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone result, bool created, int roomId, Payload payload) =>
        {
            if (result == ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS)
            {
				// 成功時の処理
            }
        };
```

- ユーザーマッチのタイムアウト時にonMatchUserTimeoutが呼び出されます。 Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs

```c#
        // ユーザーマッチリクエストタイムアウトリスナー
        snakeGameUser.onMatchUserTimeoutListeners += (UserAgent userAgent) =>
        {
        	// ユーザーマッチタイムアウト処理
        };
```

### LeaveRoom:ルームを出るときに呼び出す

- Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/MultiTapBirdGameUI.cs
- Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs

```c#
     // ゲーム終了プロトコル定義
        var endGameReq = new Com.Nhn.Gameanvil.Sample.Protocol.EndGameReq
        {
            EndType = gameEndType
        };

        // ゲームルームの退出リクエスト
        tapBirdUser.LeaveRoom(new Payload().add(new Packet(endGameReq)), (UserAgent userAgent, ResultCodeLeaveRoom result, bool force, int roomId, Payload payload) =>
        {
            if (result == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
            {
            	// 成功処理、レスポンスされたメッセージの処理
                if (payload.contains<Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes>())
                {
                    Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes endGameRes = Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes.Parser.ParseFrom(payload.getPacket<Com.Nhn.Gameanvil.Sample.Protocol.EndGameRes>().GetBytes());
                }
            }
            else
            {
                // 失敗時の処理
            }
        });
```



### ゲームで定義したプロトコル処理

- Send: Assets/GameAnvilSample/PlatformTest/Scripts/SingleGameUi.cs :レスポンスを待たずにサーバーへパケットを送って終了します。

```c#
        // 送信するパケット
        var tapMsg = new Com.Nhn.Gameanvil.Sample.Protocol.TapMsg
        {
            Combo = tapCount,
            SelectCardName = UserInfo.Instance.CurrentDeck + "_0" + 1,
            TapScore = 100
        };

        // レスポンスなしでサーバーにデータ性で伝達するパケット
        tapBirdUser.Send(new Packet(tapMsg));
```

- Request: Assets/GameAnvilSample/PlatformTest/Scripts/LobbyUi.cs :サーバーにリクエストを送り、レスポンスが来るまで待機。

```c#
        // ゲームユーザーがサーバーにrequestを送り、responseを受け取って処理する。
        gameUser.Request<Com.Nhn.Gameanvil.Sample.Protocol.ShuffleDeckRes>(shuffleDeckReq, (userAgent, shuffleDeckRes) =>
        {
            if (shuffleDeckRes.ResultCode == Com.Nhn.Gameanvil.Sample.Protocol.ErrorCode.None)
            {
                // 成功時の処理
            }
            else
            {
                // 失敗時の処理
            }
        });
```

- リスナー登録: Assets/GameAnvilSample/PlatformTest/Scripts/MultiSnakeGameUi.cs :サーバーからクライアントへプッシュしてくれるパケットリスナーを登録

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
