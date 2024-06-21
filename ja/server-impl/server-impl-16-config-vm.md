## Game > GameAnvil > サーバー開発ガイド > サーバー構成と駆動



## 構成(Configuration)

GameAnvilは大きく分けて2つの方法でサーバーを構成できます。最も代表的な方法は、NHN Cloudのコンソールを介して動作するサーバーをGUI上で構成する方法です。この方法は、クラウド上でVMベースのサービスまたはテスト用に使用する方法です。しかし、この方法は開発過程で使用するには不便です。そこで、ユーザーが開発中にPCで直接サーバーを構成できるようにGameAnvilConfig.jsonファイルを提供しています。このファイルの基本パスは、プロジェクトのresources/を使用します。もし複数のプロセスでサーバーを構成する場合は、それぞれのプロセスごとに固有のGameAnvilConfig.jsonが必要です。

この構成ファイルに入力する値の一部は、サーバーコード上でアノテーションによってエンジンに登録する時に使用する値と一致する必要があります。例えば、上記のように構成し、GameNodeで使用された"MyGame"というサービス名は、必ずGameAnvilConfig.jsonにも同じように設定されていなければなりません。そうでなければ、該当サービスが見つからず、サーバーの動作過程でエラーが発生します。

```java
@ServiceName("MyGame") // "MyGame"というサービス用にGameNodeでエンジンに登録
public class SampleGameNode extends BaseGameNode {
    ...
}
```

次にこれらの内容をもう一度見ていきましょう。



## GameAnvilConfig.jsonの編集

GameAnvilConfigは、サーバーを柔軟に構成するために非常に多くの設定を提供しています。ほとんどはエンジンデフォルト値で事足りるため、ここではユーザーの理解が必要な設定値についてのみ説明します。大きく5つに分けられます。



### 共通設定(common)

ノード構成に関係なく、必須の共通情報を設定します。 

```json
"common": {
    "ipcIp": "127.0.0.1", // サーバープロセス間通信(IPC)に使用するIP(該当マシンのIPを指定)
    "meetEndPoints": ["127.0.0.1:18000"], // クラスタリング情報を同期するピア情報(ポートは変更しなかった場合、18000を使用)
    "debugMode": false // デバッグ時に各種timeoutが発生しないようにするオプション、リアルでは必ずfalseでなければならない。
},
```



各構成項目の説明は次のとおりです。

| 名前          | 説明                                                       | デフォルト値 |
|---------------| ------------------------------------------------------------ | ------- |
| icpIp         | ノードごとに共通で使用するIP。(マシンのIPを指定)値がない場合はprivate ipに自動指定される。 | - |
| meetEndPoints | 対象ノードのcommon IPとcommunicatePort登録該当サーバーendpointを含むことができるリストに複数登録可能 |-|
| debugMode     | デバッグ時に各種timeoutが発生しないようにするオプション、リアルでは必ずfalseでなければならない | false   |



### location

ロケーションノードは、サーバー全体のユーザーとルームの位置情報を担当するシステムノードです。エンジンが管理し、直接使用するため、ユーザーが追加で実装する必要はありません。しかし、このようなシステムノードもいくつのノードで構成するかは、ユーザーの選択次第であるため、別途の構成方法を提供します。開発過程では、次の使用例をそのまま使用しても構いません。一方で、実際にサービスのための構成は、ゲームのコンテンツやボリュームに応じて適切に適用する必要があるため、GameAnvil担当者と別途で話し合うことを推薦します。各設定項目は次のとおりです。

```json
"location": {
    "clusterSize": 1, // 合計いくつのマシン(VM)で構成されるのか？
    "replicaSize": 1, // レプリケーショングループのサイズ(master+slaveの数)
    "shardFactor": 2  // sharding用の因数 
},
```

> [参考]
> 
> 該当設定(location)のキー値がない場合やclusterSizeが0の場合は、該当プロセスでロケーションノードを作成しません。


各構成項目の説明は次のとおりです。

| 名前      | 説明                                                       | デフォルト値 |
| ----------- | ------------------------------------------------------------ | ------ |
| clusterSize | 構成されるサーバーの数(VM)                                       | -      |
| replicaSize | レプリケーショングループのサイズmaster+slaveの数                     | -      |
| shardFactor | sharding用の因数<br />-全体shardの数= clusterSize x replicaSize x shardFactor <br />-1つのマシン(VM)で動作するshardの数= replicaSize x shardFactor <br />-固有のshardの総数(masterシャードの数) = clusterSize x shardFactor | -      |

### Location Cluster

masterロケーションノードにリクエストして、ユーザーやルームなどの位置情報を照会できます。ただし、すべてのロケーションノードがクラスタリングが完了した後にのみリクエストを送信できます。ロケーションノードを使用するに設定した場合、エンジン内部ではロケーションノードを動作し、すべてのロケーションノードがクラスタリングが完了したかどうかをチェックします。一定時間内にクラスタリング全体が完了なければ、エラーログを残します。

### Location Fail-over

replicaSizeを2以上に設定した場合、masterロケーションノードとslaveロケーションノードが存在します。もしmasterロケーションノードが死んだ場合は、slaveロケーションノードがmasterの役割を代わりに行うようにlocation fail-over機能が実装されています。masterロケーションノードがあったサーバーを再駆動する場合は、VmOptionに`-DrestartedAfterDown=true`を追加して区別できるようにします。この時、再駆動するロケーションノードはすべてslaveで駆動します。

### match

マッチノードはマッチメイキングを行うノードです。つまり、ユーザーが実装したマッチメイカーを作動させます。これらのマッチノードは、いくつのノードを動作させるかを決定するだけです。一般的な開発過程や小規模のサービスは、1つのマッチノードで十分です。

```json
"match": {
    "nodeCnt": 1
},
```

> [参考]
> 
> 該当設定(match)のキー値がない場合やnodeCntが0の場合は、該当プロセスでマッチノードを作成しません。



各構成項目の説明は次のとおりです。

| 名前  | 説明          | デフォルト値 |
| ------- | --------------- | ------ |
| nodeCnt | match node数 | -      |




### gateway

ゲートウェイノードはクライアントが接続を結ぶノードです。したがって、接続するクライアント規模に応じて適切な数のノードを準備する必要があります。

```json
"gateway": {
    "nodeCnt": 4, // ノード数。(ノード番号は0から付与される)
    "ip": "127.0.0.1", // クライアントと接続されるIP。
    "connectGroup": {
        "TCP_SOCKET": {
            "port": 18200, // クライアントと接続されるポート。
            "idleClientTimeout": 240000 // データの送受信がない状態後のタイムアウト。(0の場合は使用しない)   
        },
        "WEB_SOCKET": {
            "port": 18300,
            "idleClientTimeout": 240000
        }
    }
},
```


> [参考]
> 
> 該当設定(gateway)のキー値がない場合やnodeCntが0の場合は、該当プロセスでゲートウェイノードを作成しません。


各構成項目の説明は次のとおりです。

| 名前                           | 説明                                                       | デフォルト値 |
| -------------------------------- | ------------------------------------------------------------ | ------ |
| nodeCnt                          | ノード数                                                  | -      |
| ip                               | クライアントが接続するIPアドレス<br>(設定値がない場合は、該当マシンのプライベートIPに自動設定) | -      |
| connectGroup                     | ゲートウェイノードはTCPソケットとWEBソケットをサポートします          | -      |
| connectGroup : port              | クライアントが接続するポート                                   | -      |
| connectGroup : idleClientTimeout | データの送受信がない状態から接続整理までのタイムアウト(0の場合は使用しない) | 4000   |



### game

ゲームノードは、実際のゲーム関連オブジェクトが作成され、コンテンツがプレイされるノードです。ゲームコンテンツの特性に合わせて、ノード数やチャンネルなどを構成できます。


```json
"game": [
    {
        "nodeCnt": 4, // 4個のゲームノードはそれぞれのチャンネル"初心者"、"中級者"、"初心者"、"中級者"に対応するようになります。
        "serviceId": 1,
        "serviceName": "Todo - Input My Service Name",
        "channelIDs": ["初心者"、"中級者"、"初心者"、"中級者"]、// ノードごとに付与するチャンネルID。(ユニークでなくてもよい。""はチャンネルを使用しないことを意味する)
        "userTimeout": 5000 // disconnect後のユーザーオブジェクトの削除タイムアウト。
    }
]
```


> [参考]
> 
> サービス名は、必ずサーバーコード上でユーザーが作成したゲームノードクラスをエンジンに登録するために使用したサービス名を入力する必要があります。次はゲームノードに対するアノテーションの例です。"MyGame"というサービス名を使用しました。
> 
> ```java
> @ServiceName("MyGame") // "MyGame"というサービス用にGameNodeでエンジンに登録
> public class SampleGameNode extends BaseGameNode {
>     ...
> }
> ```
>
> したがって、必ずGameAnvilConfigで次のようにサービス名を"MyGame"に設定する必要があります。次の例は、"MyGame"のサービス名をサービスID "1"にマッピングした例です。このサービス名とサービスIDは、構成全体で唯一の値でなければなりません。
>
> ```json
> "game": [
>     {
>         "nodeCnt": 4, // 4個のゲームノードはそれぞれのチャンネル"初心者"、"中級者"、"初心者"、"中級者"に対応するようになります。
>         "serviceId": 1,
>         "serviceName": "MyGame", // サービス名
>         "channelIDs": ["初心者"、"中級者"、"初心者"、"中級者"]、// ノードごとに付与するチャンネルID。(ユニークでなくてもよい。""はチャンネルを使用しないことを意味する)
>         "userTimeout": 5000 // クライアントの接続が切断された後、ユーザーオブジェクトをサーバーから削除せずにどのくらいの期間管理するかを設定
>     },
>     {
>         "nodeCnt": 1,
>         "serviceId": 2,
>         "serviceName": "MyChatting", // サービス名
>         "channelIDs": [""], // ノードごとに付与するチャンネルID。(ユニークでなくてもよい。""はチャンネルを使用しないことを意味する)
>         "userTimeout": 5000 // クライアントの接続が切断された後、ユーザーオブジェクトをサーバーから削除せずにどのくらいの期間管理するかを設定
>     }
> ]
> ```
>
> 上記の構成例は、追加でMyChattingサービスを示しています。サービスIDは、「MyGame」とは異なる「2」であることが確認できます。

> [参考]
> 
> 該当設定("game")のキー値がない場合やnodeCntが0の場合は、該当プロセスでゲームノードを作成しません。

各構成項目の説明は次のとおりです。

| 名前      | 説明                                                                       | デフォルト値 |
| ----------- |----------------------------------------------------------------------------| ------ |
| nodeCnt     | ノード数                                                                    |        |
| serviceId   | サービスIDとして、サーバー全体の構成で唯一の数値である必要があります。ただし、1～99の数値のみ使用可能です。           |        |
| serviceName | サービス名として、サーバー全体の構成で唯一の文字列である必要があります。このサービス名は、ゲームノードをエンジンに登録するためにアノテーションに使用されます。 |        |
| channelIDs  | ノードごとに付与するチャンネルIDとして、唯一の値である必要はありません。<br>ただし、""はチャンネルを使用しないことを意味します。          |        |
| userTimeout | クライアントの接続が切断された後、ユーザーオブジェクトをサーバーから削除せずにどのくらいの期間管理するかを設定します。                       | 10000  |


### support

サポートノードは、補助的な役割を行うノードです。クライアントとの直接通信も可能であるため、ゲームに関する情報の交換や定期的な作業、ゲーム以外で独立した実装が必要な作業などを委任して処理することに適しています。

```json
"support": [
    {
        "nodeCnt": 2,
        "serviceId": 3,
        "serviceName": "DB",
        "restIp": "127.0.0.1",
        "restPort": 17251
    },
    {
        "nodeCnt": 2,
        "serviceId": 4,
        "serviceName": "SampleWebSupport",
        "restIp": "127.0.0.1",
        "restPort": 17250
    }
],
```


> [参考]
> 
> 前述したゲームノードと同様に、サーバーコード上でユーザーが作成したサポートノードクラスをエンジンに登録するために使用したサービス名は、こちらの設定で必ず入力する必要があります。

> [参考]
> 
> 該当設定("support")のキー値がない場合やnodeCntが0の場合は、該当プロセスでサポートノードを作成しません。

各構成項目の説明は次のとおりです。


| 名前      | 説明                                            | デフォルト値 |
| ----------- | ------------------------------------------------- | ------ |
| nodeCnt     | ノード数                                       | - |
| serviceId   | サービスIDとして、サーバー全体の構成で唯一の数値である必要があります。ただし、1～99の数値のみ使用可能です。 | - |
| serviceName | サービス名として、サーバー全体の構成で唯一の文字列である必要があります。このサービス名は、サポートノードをエンジンに登録するためにアノテーションに使用されます。 | - |
| restIp      | RESTfulリクエスト用のIPアドレス<br/>(設定値がない場合は該当マシンのプライベートIPに自動設定) | - |
| restPort    | RESTfulリクエスト用のポート                       |-|



## VMオプション

GameAnvilサーバーを駆動するための開発チームで使用するVMオプションの核心的な部分を共有します。ここで推奨するVMオプションは、これまで数回にわたる大規模性能テストで検証されました。これを参考に、ユーザーの希望に応じて適切に変更して、使用してください。



### 推奨のVMオプション

最初の行のjavaagentオプションは、ファイバーコードをJIT方式で使用する場合にのみ必要なため、AOT方式で使用する場合はこの行を削除してください。これらの[Bytecode Instrumentation](../server-basic/server-basic-06-bytecode-instrument)については後述します。メモリサイズはシステムに合わせて設定します。開発チームは8GBマシンでは46GBを、16GBマシンでは1012GBを使用しており、GameAnvilはG1GCを公式GCとして使用しています。したがって、特別な理由がなければ、G1GCの使用を推奨します。最後に、GCログのための最小限のオプションを追加することを強く推奨します。特に、開発過程では必須です。

####  Java 8 

```
-javaagent:YOUR_PATH\quasar-core-0.7.10-jdk8.jar=bm
-Xms4g
-Xmx4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication

-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:gc.log
```

#### Java 11

```
-javaagent:YOUR_PATH\quasar-core-0.8.0-jdk11.jar=bm

--add-opens=java.base/java.lang=ALL-UNNAMED
--illegal-access=deny

-Xms4g
-Xmx4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication

-Xlog:gc*,safepoint:./log/gc.log:time,level,tags
```



### GCログ用のVMオプション

GCログ用のオプションはメモリリークなどを追跡するために必須です。したがって、特別な理由がなければ、少なくとも開発過程では次のようなGCログ関連オプションを追加することを推奨します。しかし、実際のサービスでは性能に影響を与える可能性があるため、一部の最適化されたオプションのみを必要に応じて追加する必要があります。先ほど推奨したオプションに加えて、Javaバージョンによって次のようなオプションを追加できます。

#### Java 8
```
-XX:+PrintGCDetails
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-XX:+PrintHeapAtGC
-XX:+PrintReferenceGC
-Xloggc:YOUR_PATH/gc.log
```
次のようにファイルローテーションオプションを追加することもできます。
```
-XX:+PrintGCDetails
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-XX:+PrintHeapAtGC
-XX:+PrintReferenceGC
-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=100
-XX:GCLogFileSize=10M
-Xloggc:YOUR_PATH/gc.log
```

#### Java 11
```
-Xlog:gc*,safepoint:YOUR_PATH/gc.log:time,level,tags,uptime
```
次のようにファイルローテーションオプションを追加することもできます。
```
-Xlog:gc*,safepoint:YOUR_PATH/gc.log:time,level,tags,uptime:filecount=100,filesize=10M
```
