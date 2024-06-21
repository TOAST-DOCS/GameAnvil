## Game > GameAnvil > Unity深層開発ガイド > Connector

## Connector

コネクタは基本設定とエージェント管理を担当し、内部動作に関するログを見ることができるようにオプションの設定や、コールバックの登録を行うことができます。

[Unity基礎開発ガイド > GameAnvilConnector](../unity-basic/unity-basic-02-connector.md)文書でコネクタの主な機能を簡単に使えるGameAnvilConnectorについて説明しました。

今回の文書では、コネクタをもっと拡張的に使うことができるConnectorについて説明します。
Connectorを使うためにはまず、Connectorを作成する必要があります。

### 作成

次のようにConnectorを作成できます。

```c#
using GameAnvil;
Connector connector = new Connector();
```

### 設定

Connectorが動作するために使う設定があります。この設定はConnectorを作成する時、デフォルトで設定されますが、必要に応じて次のように直接値を変更できます。

```c#
using GameAnvil;
Connector connector = new Connector();
connector.config.packetTimeout = 10;
```

`Connector.Config`を予め作成しておいて、これをパラメータで受け取ってConnectorを作成することもできます。また、このように作成した`Connector.Config`をMonoBehaviorのメンバーにしておけば、UnityのInspectorウィンドウで設定値を変更することもできます。

```c#
using GameAnvil;
var config = new Connector.Config();
config.packetTimeout = 10;
Connector connector = new Connector(config);
```

![](https://static.toastoven.net/prod_gameanvil/images/unity-advanced/01-connector/01-config.png)

設定の種類は次のとおりです。

| 名前                   | 説明                                                             | デフォルト値 |
| --------------------------- |----------------------------------------------------------------------| :----: |
| defaultReqestTimeout        | TimeOut基本待機時間                                               | 3(sec) |
| packetTimeout               | パケットが指定された時間内に更新されない場合、disconnectされたと判断。 pingIntervalより大きい値に設定。 | 5(sec) |
| pingInterval                | サーバーとの接続を確認するためのping周期。使用しない場合は0に設定。                         | 3(sec) |
| useIPv6                     | IPv6環境でIPv4アドレスを使用してサーバーに接続する際、IPv4アドレスをipv6アドレスに変換して使用するかどうか   | false  |
| useSocketNoDelay            | ソケットのNodelayを使用するかどうか                                                | true   |
| useArgumentDelegateOnly     | リスナーに登録したコールバックではなく、引数で渡したコールバックだけが呼び出されるようにするオプション                         | true   |
| Request Direct              | Request呼び出し時、レスポンス待機中でない場合、queueに入れずに直接サーバーに送信する               | true   |


### ログ

Connectorは直接ログを残さず、コールバックを通じてログを渡します。次のようにコールバックを登録することでConnectorで発生するログを受け取ることができます。

| Connectorログ関連関数 | 説明 |
| --------------------- | --- |
| AddLogListener | ロギングメッセージを処理するコールバック登録 |
| RemoveLogListener | ロギングメッセージを処理するコールバックリストから特定のコールバックを削除 |
| ContainsLogListener | ロギングメッセージを処理するコールバックリストに特定のコールバックが含まれているかどうかを検査 |
| EnablePingPongLogs | PingPongによって発生したログを出力するかどうかを設定 |
| GetEnablePingPongLogs | PingPongによって発生するログ出力の有無を照会 |
| SetLogLevel | ログレベル設定 |
| GetLogLevel | ログレベル照会 |

下記のコードを使ってもっと詳しく説明します。

Connector.AddLogListener()でロギングメッセージを処理するコールバックを登録できます。

```c#
/// <summary>
/// ロギングメッセージを処理するコールバックを登録
/// </summary>
/// <param name="logger">ロギングメッセージを処理するコールバック</param>
public void AddLogListener (Action<string> logListener);
```

Connector.RemoveLogListener()で登録したロギングメッセージ処理コールバックを削除できます。

```c#
/// <summary>
/// ロギングメッセージを処理するコールバックのリストから特定のコールバックを削除
/// </summary>
/// <param name="logListener">削除するコールバック</param>
public void RemoveLogListener (Action<string> logListener);
```

Connector.ContainsLogListener()で特定のコールバックがロギングメッセージを処理するコールバックのリストに含まれているか確認できます。

```c#
/// <summary>
/// ロギングメッセージを処理するコールバックのリストに特定のコールバックが含まれているか検査
/// </summary>
/// <returns></returns>
public bool ContainsLogListener (Action<string> logListener);
```

Connector.EnablePingPongLogs()でPingPongによって発生するログを出力するかどうかを設定できます。PingPongはサーバーと接続がうまくできているか確認するためパケットを送受信するプロセスです。

```c#
/// <summary>
/// PingPongによって発生するログを出力するかどうかを設定します。 trueの場合はログを出力し、falseの場合は関連ログを出力しません。
/// </summary>
/// <param name="enable">PingPongのログを出力するかどうか</param>
public void EnablePingPongLogs (bool enable);
```

Connector.GetEnablePingPongLogs()でPingPongによって発生するログを出力するかどうか確認できます。

```c#
/// <summary>
/// PingPongによって発生するログを出力するかどうかを照会します。trueの場合はログを出力し、falseの場合はログを出力しません。
/// </summary>
/// <returns>ingPongのログを出力するかどうか</returns>
public bool GetEnablePingPongLogs ();
```

Connector.SetLogLevel()でログレベルを設定することができます。ログレベルによって表示されるログの種類が変わります。

| ログレベル | 説明 |
| :------: | --- |
| Debug | サーバーとクライアントが送受信する全てのパケットの詳細ログ(開発用)|
| Info | サーバーとクライアントが送受信する全てのパケットの簡略ログ |
| Warn | コネクタの使用自体には問題がないが、見落とすと実際のサービスでは問題になる可能性がある場合、packetTimeoutを0に設定してサーバーに接続した場合、サーバーからパケットを受け取ったので、これを処理するリスナーが登録されていない場合、サーバーからClientStateCheckを受け取った場合のログ |
| Error | ソケットエラー、サーバーから受け取ったエラー、不明なエラーなどのログ |

```c#
/// <summary>
/// ログレベルを設定する。
/// </summary>
/// <param name="level">設定するログレベル</param>
public void SetLogLevel (LogLevel level);
```

Connector.GetLogLevel()で設定したログレベルを照会できます。

```c#
/// <summary>
/// 設定したログレベルを照会する。
/// </summary>
/// <returns>ログレベル</returns>
public Loglevel GetLogLevel ();
```

Connector内部ログを使用する例です。

```c#
connector = new GameAnvil.Connector();

// LogListenerを登録してコネクタ内部のログをUnityで確認できるようにします。
connector.AddLogListener(message =>{
    OnLogMesseage?.Invoke(message);
    Debug.Log(message);
});

connector.SetLogLevel(GameAnvil.Internal.LogLevel.Info);
connector.EnablePingPongLogs(true);
```


### Update

サーバーと送受信するメッセージを処理するため、Update()関数を定期的に呼び出す必要があります。MonoBehaviourのUpdate()で呼び出すようにするのが一番簡単な方法ですが、必要に応じて他の方法で呼び出しても構いません。しかし、Thread safeではないので、別のThreadで呼び出してはいけません。
Update()関数の呼び出し周期は自由に設定してもいいですが、呼び出さない場合、サーバーからメッセージを受け取ってもこれに対する通知を受けることができません。

```c#
connector.Update();
```

### ConnectorHandler

MonoBehaviourを継承してコネクタを管理するConnectorHandlerスクリプトの例です。必要に応じて新しく作ったり、適切に変更して使うことができます。
ConnectorHandlerをシーンのGameObjectにコンポーネントとして追加して使うことができます。

```c#
using GameAnvil.Connection;
using UnityEngine;

namespace GameAnvil
{
    public class ConnectorHandler : MonoBehaviour
    {

        private static ConnectorHandler instance;
        public static Connector connector = null;
        public ConnectionAgent connectionAgent;

        [Header("Configuration")]
        public Connector.Config config;
        public int PauseClientStateCheckMiliSecond = 10 * 60;

        private void Awake()
        {
            if (instance == null)
            {
                instance = this;
                DontDestroyOnLoad(gameObject);
                connector = new Connector(config);
                connectionAgent = connector.GetConnectionAgent();
            } 
            else
            {
                Debug.Log("ConnectorHandler instance already exist. Destroy new one.");
                Destroy(gameObject);
            }
        }

        private void Update()
        {
            connector.Update();
        }

        void OnDisable()
        {
            if (connector.IsConnected())
            {
                connectionAgent.Disconnect();
            }
        }

        private void OnApplicationPause(bool pause)
        {
            if (pause)
            {
                connector.GetConnectionAgent().PauseClientStateCheck(PauseClientStateCheckMiliSecond);
                connector.Update();
            } 
            else
            {
                connector.Update();
                connector.GetConnectionAgent().ResumeClientStateCheck();
            }

            if (connector.IsConnected())
            {
                // アプリがresumeされた後、やるべき作業があればここで処理します。
            }
        }
    }
}
```

これでGameAnvilコネクタの使用準備が完了しました。
