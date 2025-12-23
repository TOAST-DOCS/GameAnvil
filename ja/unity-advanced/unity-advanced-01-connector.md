## Game > GameAnvil > Unity 応用開発ガイド > 始める

## 始める
GameAnvilConnectorは、GameAnvilが提供する多様な機能を手軽に利用できるように支援します。

## GameAnvilConnectorのインストール

gameanvil-connector.unitypackageを利用して、GameAnvilConnectorをプロジェクトに含めることができます。まず[こちら](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector.unitypackage)からgameanvil-connector.unitypackageをダウンロードします。そしてメニューの**Assets > Import Package > Custom Package...**を選択します。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/01-import.png)

<br>

そしてダウンロードしたパッケージファイルを選択してパッケージをインストールします。

![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/02-select-package.png)
![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/03-files.png)
<br>

パッケージをインストールすると、次のようにGameAnvilフォルダの下にGameAnvilConnector DLLファイルがインストールされます。


![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/04-gameanvil.png)
<br>

DLLファイルはC#で作成されたもので、Android、iOS、PCなどのプラットフォームで全て使用可能です。


## GameAnvilConnectorの生成
GameAnvilサーバーと接続するためには、必ずGameAnvilConnectorを使用する必要があります。次のように簡単にGameAnvilConnectorを生成して使用できます。
[Unity 基礎開発ガイド > マネージャー](../unity-basic/unity-basic-02-gameanvil-manager.md)で扱うGameAnvilManagerも、内部的にGameAnvilConnectorを使用します。  
```c#
using GameAnvil;

GameAnvilConnector connector = new GameAnvilConnector();
```

## GameAnvilConfig
GameAnvilConfigには、GameAnvilConnector動作時に使用されるいくつかの設定が定義されています。これらの設定は定義されており、GameAnvilConfigに定義されているデフォルト値として設定されますが、必要であればGameAnvilConfigの値を直接変更して使用できます。

```c#
using GameAnvil;

GameAnvilConfig.DefaultRequestTimeoutMillis = 3000;
GameAnvilConfig.PacketTimeoutMillis = 5000;
GameAnvilConfig.PingIntervalMillis  = 3000;
GameAnvilConfig.UseIPv6 = false;
GameAnvilConfig.UseSocketNoDelay = true;
```

設定の種類は以下のとおりです。

| タイプ | 名前                        | 説明                                                                                                |
|------|-----------------------------|-----------------------------------------------------------------------------------------------------|
| int  | DefaultRequestTimeoutMillis | タイムアウト基本待機時間設定(単位 : ミリ秒、デフォルト値 : 3000)                                                                                              |
| int  | PacketTimeoutMillis         | パケットが指定された時間内に更新されない場合、接続が切れたと判断<br/>pingInterval値より高く設定する必要がある<br/>(単位 : ミリ秒、デフォルト値 : 5000) |
| int  | PingIntervalMillis          | サーバーとの接続を確認するためにPingメッセージを送る周期設定 (単位 : ミリ秒、デフォルト値 : 3000、0の場合は使用しない)                                           |
| bool | UseIPv6                     | 接続時にIPv6アドレスへ変換するかどうか (デフォルト値 : false)                                                                                                       |
| bool | UseSocketNoDelay            | ソケットのNodelay使用有無 (デフォルト値 : true)                                                                                                        |
## GameAnvilLogger

GameAnvilConnectorは直接ログを残さず、コールバックを通じてログを伝達します。次のようにGameAnvilLoggerにコールバックを登録することで、GameAnvilConnectorで発生するログを受け取ることができます。

```c#
using GameAnvil;

GameAnvilLogger.LogListener += Debug.Log; // Unity Debug.Log登録
GameAnvilLogger.LogListener += (logMessage)=>{
    // ログメッセージ処理
}
```

GameAnvilLoggerには、出力されるログの種類を設定できる以下の属性があります。

| タイプ     | 名前               | 説明                                           |
|----------|--------------------|------------------------------------------------|
| LogLevel | Threshold          | 出力するログのレベルを設定(デフォルト値: Info)                     |
| bool     | EnablePingPongLogs | PingPongによって発生するログを出力するかどうかを設定(デフォルト値 : false) |
