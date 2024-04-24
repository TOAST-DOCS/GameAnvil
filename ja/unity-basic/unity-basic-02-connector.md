## Game > GameAnvil > Unity基礎開発ガイド > GameAnvilConnector

## GameAnvilConnector

GameAnvilConnectorは基本設定とエージェント管理を担当し、内部動作に関するログを見ることができるようにオプションを設定したり、コールバックを登録できます。GameAnvilConnectorを使用するには、まずGameAnvilConnectorを作成する必要があります。

### 作成

GameObjectを作成してGameAnvilConnectorコンポーネントを追加します。**Add Component > GameAnvil > GameAnvilConnector**を選択してコンポーネントとして追加できます。

または、Unity Hierarchyで右クリックした後、**GameAnvil > GameAnvilConnector**を選択して直接作成することもできます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/02-connector/01-component.png)

### 設定

GameAnvilConnectorの動作に使われる設定値があります。GameAnvilConnector作成時にデフォルトで設定されますが、必要に応じてインスペクタウィンドウで直接値を変更できます。 

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/02-connector/02-config.png)

設定の種類は次のとおりです。

| 名前                      | 説明                                                                 | デフォルト値 |
| --------------------------- |-----------------------------------------------------------------------| :----: |
| defaultReqestTimeout        | TimeOut基本待機時間                                                   | 3(sec) |
| packetTimeout               | パケットが指定された時間内に更新されない場合、disconnectされたと判断する。 pingIntervalより大きい値に設定。 | 5(sec) |
| pingInterval                | サーバーとの接続を確認するためのping周期。使用しない場合は0に設定。                          | 3(sec) |
| useIPv6                     | IPv6環境でIPv4アドレスを使用してサーバーに接続する際、IPv4アドレスをIPv6アドレスに変換して使用するかどうか       | false  |
| useSocketNoDelay            | ソケットのNodelay使用有無                                                    | true   |
| useArgumentDelegateOnly     | リスナーに登録したコールバックではなく、引数で渡したコールバックだけが呼び出されるようにするオプション                           | true   |
| Request Direct              | Request呼び出し時、レスポンス待ちでない場合、queueに入れずに直接サーバーに送信                   | true   |


### ログ

GameAnvilConnectorは直接ログを残さず、コールバックを通じてログを渡します。次のようにOnLogMessageコールバックにリスナーを登録して渡されたログメッセージを出力できます。

```c#
GameAnvilConnector.getInstance().OnLogMesseage.AddListener(log => Debug.Log("OnLogMesseage test : " + log));
```

GameAnvilConnectorのログに使う設定値があります。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/02-connector/03-log.png)

設定の種類は次のとおりです。

| 名前 | 説明 | デフォルト値 | 
| --- | --- | :----: |
| Use Debug Log | GameAnvilConnectorの内部ログを出力するかどうか | true |
| Enable Ping Pong Logs | PingPongによって発生するログを出力するかどうか | false |
| Log Level | GameAnvilConnectorログレベル | Info |
| on Log Message (String) | GameAnvilConnectorの内部ログがstringの形で渡されるコールバック | - |

これでGameAnvilConnectorの使用準備が完了しました。
