## Game > GameAnvil > CocosCreator開発ガイド > Connector

## Connector

GameAnvilコネクタを使うためにはまず、Connectorを作成する必要があります。基本設定とエージェント管理を担当し、内部動作に関するログを見ることができるようにコールバックを登録できます。

### 作成

次のようにConnectorを作成できます。

``` typescript
import { Connector } from 'GameAnvil/gameanvil';
let connector = Connector.Create();
```

### 設定

Connectorが動作に使用する設定があります。この設定はConnector作成時、デフォルト値に設定されますが、必要な場合は次のように直接値を変更できます。

``` typescript
import { Connector } from 'GameAnvil/gameanvil';
let connector = Connector.Create();
connector.config.PacketTimeout = 10;
```

設定の種類は次のとおりです。

| 名前                | 説明                                                        | デフォルト値 |
| -------------------- | ------------------------------------------------------------ | ------ |
| defaultReqestTimeout | TimeOut基本待機時間設定                                  | 3(sec) |
| packetTimeout        | パケットが指定された時間内にアップデートされない場合、接続が解除されたと判断されます。pingIntervalより高く設定する必要があります。 | 5(sec) |
| pingInterval         | サーバーとの接続を確認するためのping周期設定。 0の場合、使用しない | 3(sec) |

### Update

サーバーとやり取りするメッセージを処理するため、Update() 関数を定期的に呼び出す必要があります。Update()関数の呼び出し周期は自由に設定できますが、呼び出さない場合、サーバーからメッセージを受け取ってもその通知を受けることができません。

```typescript
setInterval(() => { connector.Update(); }, 10);
```

### ConnectHandler

次のようなシングルトンクラスを作って使うと便利です。

```typescript
/*
    Internet Explorerなど古いバージョンのブラウザでコネクタを使うためには 
    core-js, regenerator-runtime などのプラグインを使う必要があります。
*/
import 'core-js';
import 'regenerator-runtime';

import { Connector, ProtocolManager } from 'GameAnvil/gameanvil';

export default class GameAnvilManager {
    private static manager: GameAnvilManager;
    private connector: Connector;
    private constructor() {
        // プロトコル登録。
        ProtocolManager.RegisterProtocol(0, GameMessages);

        // コネクタ作成。
        this.connector = Connector.Create();

        // メッセージループ。10ms毎に呼び出し。
        let updater = setInterval(() => { this.connector.Update(); }, 10);
    }

    static GetInstance() {
        if (this.manager == null) {
            this.manager = new GameAnvilManager();
        }
        return this.manager;
    }

    GetConnectionAgent() {
        return this.connector.GetConnectionAgent();
    }

    GetUserAgent(serviceName: string, subId: string) {
        return this.connector.GetUserAgent(serviceName, subId);
    }

    CreateUserAgent(serviceName: string, subId: string) {
        return this.connector.CreateUserAgent(serviceName, subId);
    }
}
```

このコードはサンプルなので、必要に応じて適切に変更して使ってください。これでGameAnvilコネクタの使用準備が完了しました。
