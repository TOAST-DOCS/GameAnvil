<!-- pre-align:aligned sig=3180aad95516 -->

<a id="game-gameanvil-guide-to-test-development-get-started"></a>
## Game > GameAnvil > テスト開発ガイド > 始める { #game-gameanvil-guide-to-test-development-get-started }

<a id="overview"></a>
### 概要 { #overview }

GameHammerは、GameAnvilエンジンを利用したゲームサーバー開発後に使用できる性能及び機能テストツールです。実際のコネクタが提供する全ての機能を同様に使用してテストでき、負荷テストのために多数のGameHammerを同時に実行するなどの使い方が可能です。テスト進行中に進行状況を確認したり、テスト最終結果を集計して確認及び保存したりできます。

* コネクタにある全ての機能を同様にサポート
* Sync/Async方式を全てサポート
    * Async方式のAPI提供
    * Sync方式のためのfuture提供
* 数千以上のコネクションを同時に使用可能
* 状態ベースのシナリオ管理機能をサポート

このガイドではGameHammerの使用方法を詳細な例とともに提供します。サーバーエンジンと同様にIntelliJを基準に説明します。

<a id="supported-environment-and-protocol"></a>
### サポート環境及びプロトコル { #supported-environment-and-protocol }

<a id="supported-environment-and-protocol-supported-network-protocol"></a>
#### サポートするネットワークプロトコル

* TCP/IP
* SSL over TCP/IP

<a id="supported-environment-and-protocol-available-application-protocol-format"></a>
#### 使用可能なアプリケーションプロトコル形式

* Google Protocol Buffers
* カスタムバイトストリーム
* HTTP/HTTPS(特定の用途に限定)

<a id="add-gamehammer-dependency-to-project"></a>
### プロジェクトにGameHammer依存関係を追加 { #add-gamehammer-dependency-to-project }

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

<a id="create-gamehammer-jar-file-with-maven"></a>
### MavenでGameHammer jarファイルを生成する { #create-gamehammer-jar-file-with-maven }

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

