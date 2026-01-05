## Game > GameAnvil > テスト開発ガイド > 始める

### 概要

GameHammerは、GameAnvilエンジンを利用したゲームサーバー開発後に使用できる性能及び機能テストツールです。実際のコネクタが提供する全ての機能を同様に使用してテストでき、負荷テストのために多数のGameHammerを同時に実行するなどの使い方が可能です。テスト進行中に進行状況を確認したり、テスト最終結果を集計して確認及び保存したりできます。

* コネクタにある全ての機能を同様にサポート
* Sync/Async方式を全てサポート
    * Async方式のAPI提供
    * Sync方式のためのfuture提供
* 数千以上のコネクションを同時に使用可能
* 状態ベースのシナリオ管理機能をサポート

このガイドではGameHammerの使用方法を詳細な例とともに提供します。サーバーエンジンと同様にIntelliJを基準に説明します。

### サポート環境及びプロトコル

#### サポートするネットワークプロトコル

* TCP/IP
* SSL over TCP/IP

#### 使用可能なアプリケーションプロトコル形式

* Google Protocol Buffers
* カスタムバイトストリーム
* HTTP/HTTPS(特定の用途に限定)

### プロジェクトにGameHammer依存関係を追加

GameHammerはGameAnvilと同様にMavenを通じて配布されます。pom.xmlファイルのdependencies要素に次のように追加すると、GameHammerを使用できます。

```pom
<dependencies>
        <!-- GameHammer -->
        <dependency>
			<groupId>com.nhn.gameanvil</groupId>
			<artifactId>gamehammer</artifactId>
			<version>2.1.0-jdk11</version>
		</dependency>
<dependencies>
```

### MavenでGameHammer jarファイルを生成する

GameHammerを利用してテストシナリオを作成した後、GameAnvilコンソールでテストする目的などでjarファイルを生成できます。

GameHammerを追加したプロジェクトのpom.xmlがあるディレクトリで以下のコマンドを実行します。

```
mvn package
```

コマンド実行後、ビルド過程が出力され、最後にビルドに成功したというメッセージを確認します。以下のように表示されれば成功です。

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.880 s
[INFO] Finished at: 2023-11-13T17:49:48+09:00
[INFO] ------------------------------------------------------------------------
Process finished with exit code 0
```

新しく生成されたtargetディレクトリ内でビルドされたファイルを確認できます。

### 不要なdebugログの削除方法

GameHammerを実行した際、内部ライブラリ関連のDEBUGレベルログが発生する場合があります。動作には異常ありませんが、このようなログを削除したい場合はVMOptionに以下の項目を追加してください。

```
-Dio.netty.tryReflectionSetAccessible=true
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/jdk.internal.misc=ALL-UNNAMED
```
