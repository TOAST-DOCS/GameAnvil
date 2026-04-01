## Game > GameAnvil > TypeScript開発ガイド > 接続エージェント

## GameAnvilConnector

GameAnvilConnectorは、サーバーへの接続と通信を担当するクラスで、このオブジェクトを通じてサーバーにリクエストを送信したり、サーバーから受信するメッセージに対するハンドラを登録して管理したりできます。前のインストールチャプターでは、GameAnvilConnectorの作成とconnect機能の使用方法について説明しました。このドキュメントでは、より詳細な使用方法と、GameAnvilConnectorの他の機能について見ていきます。

### 生成

以下のようにGameAnvilConnectorオブジェクトを作成します。通常は、1つのプロセスで1つのコネクタオブジェクトを使用するのが一般的です。

```typescript
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();
```

### サーバー接続

connect()関数を利用してサーバーに接続します。呼び出す前に、hostとportをあらかじめ設定しておく必要があります。

```typescript
connector.host = "127.0.0.1";
connector.port = 18300;

await connector.connect();

console.log("Connect Success");

```

connect()関数は、サーバーへの接続完了または失敗後に完了するPromiseオブジェクトを返します。そのため、以下のように使用することもできます。

```typescript
connector.host = "127.0.0.1";
connector.port = 18300;

connector.connect()
    .then(() => {
        console.log("Connect Success");
    })
    .catch(()=> {
        console.log("Connect Fail");
    })
```

コネクタのほとんどのAPIは、このように非同期で動作し、Promiseオブジェクトを返すため、上記のように状況に合わせてawaitやthenなどの機能を使用できます。

### 切断検知

サーバーによって接続が強制的に終了されたり、ネットワークの問題などで接続が切れたりした場合に実行する動作を指定できます。

```typescript
connector.onDisconnect = (resultCode: ResultCodeDisconnect, payload: Payload) => {
    console.log("Disconnected.");
}
```

関数の最初の引数で、接続が切れた理由を知ることができます。

| コード名                                        | 値  | 説明 |
|---------------------------------------------------|-------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `FORCE_CLOSE_SYSTEM_ERROR` | 2000 | システムエラーによる強制終了の状況です。クライアントでこのコードを受け取った場合は、GameAnvil開発チームにお問い合わせください。 |
| `FORCE_CLOSE_BASE_CONNECTION` | 2010 | サーバーでBaseConnectionのclose()を呼び出した際に受け取るコードです。 |
| `FORCE_CLOSE_BASE_USER` | 2011 | サーバーでBaseUserのcloseConnection()を呼び出した際に受け取るコードです。 |
| `FORCE_CLOSE_ADMIN_KICK` | 2012 | Adminで強制終了された場合に受け取るコードです。 |
| `FORCE_CLOSE_INVALID_NODE` | 2020 | GameNodeがinvalid状態に変更されたため |
| `FORCE_CLOSE_USER_TRANSFER_FAIL` | 2021 | ユーザー転送に失敗した場合 |
| `FORCE_CLOSE_USER_TRANSFER_ERROR` | 2022 | ユーザー転送中にシステムエラーが発生した場合 |
| `FORCE_CLOSE_AUTHENTICATION_FAIL` | 2030 | 認証失敗による強制終了|
| `FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID`| 2031 | 認証失敗による強制終了 - アカウントIDがない場合 |
| `FORCE_CLOSE_DUPLICATE_LOGIN` | 2032 | 重複接続による強制終了 |
| `FORCE_CLOSE_BY_NEW_CONNECTION` | 2040 | 同じアカウント情報で新しいログインリクエストがあった場合。ネットワークの瞬断などによる再接続時に、以前の接続を終了。問い合わせが必要です。 |
| `FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT` | 2041 | クライアントとの接続切断を検知。問い合わせが必要です。|
| `FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION` | 2042 | セッション情報が見つからない場合。問い合わせが必要です。|
| `FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL` | 2043 | クライアントがサーバーのステータスチェックに応答しなかった場合。問い合わせが必要です。 |
| `FORCE_CLOSE_GHOST_USER` | 2044 | ゴーストユーザーの場合。問い合わせが必要です。 |
| `SOCKET_DISCONNECT` | 2100 | ネットワーク接続が切断されました |
| `SOCKET_TIME_OUT` | 2101 | タイムアウトが発生し、コネクタが接続を切断しました |
| `SOCKET_ERROR` | 2102 | ソケットエラーが発生したため、接続を切断しました |


2番目の引数で、サーバーの実装に応じた追加情報を受け取ります。追加情報の処理方法は、後ほど説明します。

### 認証

サーバーへの接続に成功した後、エンジンの全ての機能を使用するためには、まず認証を進める必要があります。authentication()関数は、サーバーとあらかじめ協議されたaccountId、deviceId、password値を引数として受け取り、認証動作を実行してPromiseを返します。認証動作完了時点でPromiseを通じて認証に成功したかどうかと、サーバーから受け取った追加データなどを確認できます。

```typescript
const deviceId, accountId, password;

const authResult = await connector.authentication(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);
```

認証の成否は、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (authResult.resultCode === ResultCodeAuth.AUTH_SUCCESS) {
    console.log("Authentication success");
} else {
    console.log("Authentication fail");
}
```

認証に失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                       | 値  | 説明                               |
|----------------------------------|-------|------------------------------------|
| `PARSE_ERROR` | -2 | 追加情報を渡した際に、その情報のパースに失敗した場合に発生する可能性のあるエラーです。 |
| `TIMEOUT` | -1 | サーバーが制限時間内に応答しなかった場合に返されるエラーです。 |
| `SYSTEM_ERROR`                  | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL` | 2 | サーバーに登録されていないプロトコルを使用して追加情報を送信した場合に発生するエラーです。 |
| `AUTH_SUCCESS`                  | 0     | 成功                              |
| `AUTH_FAIL_CONTENT` | 201 | サーバー開発者が作成した認証関連のコードで拒否され、失敗した場合に返されるエラーです。 |
| `AUTH_FAIL_INVALID_ACCOUNT_ID` | 202 | 空の文字列など、サーバーで処理できない不正なアカウントIDを送信した場合に発生するエラーです。 |

認証の前後に関わらず、コネクタが認証された状態であるかを確認するために、isAuthenticatedプロパティを確認できます。

```typescript
if (connector.isAuthenticated) {
    console.log("Already authenticated.");
}
```

### 接続と認証の両方を進行

接続が完了すると認証が必須となるため、この二つを順次実行する便利な関数を呼び出すと便利です。

```typescript
const deviceId, accountId, password;

connector.host = "127.0.0.1";
connector.port = 18300;

const authResult = await connector.connectAndAuthenticateion(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);
```

接続されていないコネクタに対して、接続と認証を順に実行し、認証結果を返します。

認証結果は、通常、認証のみをリクエストした場合の結果と同じ方法で使用します。

### Pingリクエスト

基本的にPingリクエストは定期的に送信されるように設定されていますが、設定を変更して手動にした場合は、手動でメソッドを呼び出してPingリクエストを送信できます。

```typescript
connector.ping();
```

Pingリクエストに対するレスポンスを受け取った場合、設定によってはpongログが出力されることがあります。


### メッセージ受信コールバック登録

サーバーからプロトコルバッファメッセージを受信した際に、処理関数を実行するように設定できます。1つのプロトコルバッファに対しては、1つの処理関数のみ登録可能で、既に処理関数が登録されている状態で再度登録すると、既存の処理関数は削除されます。

処理関数は非同期で動作するため、意図したタイミングで正確に呼び出されない可能性がある点にご注意ください。

最初の引数となるプロトコルバッファメッセージをトランスパイルする方法は、後の章で説明します。

```typescript
connector.setMessageCallback(UserInfo.descriptor, (connector, resultCode, userInfo) => {
    if (resultCode === ResultCode.SUCCESS) {
        console.log(userInfo.name);
        console.log(userInfo.age);
        console.log(userInfo.job);

        return;
    }
    console.log("Message receive fail.", resultCode);
});
```

### チャンネルユーザー及びルーム数情報リクエスト

サーバーにある特定のサービスの全てのチャネルの、それぞれのユーザー数とルーム数の情報をリクエストできます。

```typescript
const serviceName: string;

const result = await connector.getAllChannelCountInfo(serviceName);
const allChannelCountInfo = result.data;

for (let [key, value] of allChannelCountInfo) {
    const channelId = key;
    const channlCountInfo = value;
    console.log(`${channelId} userCount: ${channlCountInfo.userCount}, roomCount: ${channlCountInfo.roomCount}`);
}
```

特定のサービスの特定のチャネルに限定してリクエストすることもできます。この場合、サービス名と共にチャネルIDを引数として渡します。

```typescript
const serviceName: string;
const channelId: string;

const result = await connector.getChannelCountInfo(serviceName, channelId);
const channlCountInfo = result.data;

console.log(`${channelCountInfo.channelId} userCount: ${channlCountInfo.userCount}, roomCount: ${channlCountInfo.roomCount}`);
```


### チャンネル情報リクエスト

サーバーにある特定のサービスの全てのチャネルの、それぞれの情報をリクエストできます。

```typescript
const serviceName: string;

const result = await connector.getAllChannelInfo(serviceName);
const channelInfo = result.data.channelInfo;

for (let [key, value] of channelInfo) {
    const channelId = key;
    const payload = value;
}
```

特定のサービスの特定のチャネルに限定してリクエストすることもできます。この場合、サービス名と共にチャネルIDを引数として渡します。

```typescript
const serviceName: string;
const channelId: string;

const result = await connector.getChannelInfo(serviceName, channelId);
const payload = result.data;
```

### チャンネル一覧リクエスト

サーバーにある特定のサービスの全てのチャネル一覧をリクエストできます。

```typescript
const serviceName: string;

const result = await getChannelList(serviceName);

for (let channelId of result) {
    console.log(channelId);
}
```

### ユーザー状態チェック一時停止及び再開

アプリがバックグラウンドに移行した場合など、ユーザーステータスチェックに応答できない状況が予想される場合に、アプリを一時停止できます。

指定した時間、サーバーはクライアントの接続状態情報を更新しなくなります。

このメソッドで一時停止を通知しない場合、サーバーからの生存確認に応答できなくなり、接続が強制的に切断されます。

```typescript
const pauseTime: number = 10 * 1000;

connector.pauseClientStateCheck(pauseTime);
```

アプリをバックグラウンドから再開した場合など、ユーザーステータスチェックに応答できるようになったら、アプリを再開します。

```typescript
connector.resumeClientStateCheck();
```

### パケット送信

ゲートウェイサーバーにプロトコルバッファメッセージを送信できます。

メッセージオブジェクトの作成については、続くドキュメントを参照してください。

```typescript
const name: string;
const age: number;
const job: string;

connector.send(new UserInfo({name, age, job}));
```

もしサーバーで該当メッセージに対する応答を待機したい場合は、以下のように記述できます。

この場合、サーバーに事前に登録されたリスナーを通じてレスポンスを送信する必要があります。

```typescript
const result = await connector.requestMessage(new StartReq(), StartRes.descriptor);

if (result.resultCode === ResultCode.Success) {
    const startRes: StartRes = result.data;
}
```

既にメッセージがパケットの形式であれば、そのまま送信することもできます。

```typescript
const packet = PacketFactory.makePacket(new StartReq());
const result = await connector.requestPacket(packet, StartRes.descriptor);

if (result.resultCode === ResultCode.Success) {
    const startRes: StartRes = result.data;
}
```

### 例外ハンドリング

サーバー接続中に例外が発生した場合に実行する動作を指定できます。

```typescript
connector.onException = (exception: Error) => {
    console.log("Error occured.", exception);
}
```

登録された関数の最初の引数には、エラーオブジェクトが渡されます。

### 接続終了

サーバーとの接続を明示的に終了するようにリクエストできます。

```typescript
connector.disconnect();
```

サーバーと接続中でないのに接続終了をリクエストした場合、エラーが発生します。

ゲームプレイを終了する前に、connector.disconnect()関数を呼び出して接続を終了することを推奨します。終了しない場合、サーバーがクライアントの終了を認識できず、不要な動作を続ける可能性があります。connectorを管理するコンポーネントを、ゲーム終了時にのみ破棄されるように作成し、OnDestroy()でconnector.disconnect()を呼び出すように設定するのも良い方法です。
