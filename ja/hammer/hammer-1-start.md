## Game > GameAnvil > テスト開発ガイド > 始める

## GameHammer

GameHammerはGameAnvilエンジンを利用したゲームサーバー開発ツールで、強力で便利なテストツールです。 実際のコネクタで提供するすべての機能を使用することができ、様々なテストケースを作成できるAPIを提供しています。 また、ストレステストのために複数のGameHammerを同時に実行し、その結果を集計してすぐに確認できます。

### システム要件

GameHammerを使用するには、以下の事項が必要です。

- サポートする言語
    - Java
- ターゲット開発環境
    - IntelliJ
- サポートするネットワークプロトコル
    - TCP/IP
    - SSL over TCP/IP
- 使用可能な応用プロトコル形式
    - Google Protocol Buffers
    - Google FlatBuffers(予定)
    - カスタムバイトストリーム
    - HTTP/HTTPS(特定の用途に限る)

### 特長

GameHammerは以下の機能をサポートします。

- コネクタと対応するすべての機能をサポート
- Sync/Async方式をサポート
    - Async方式のAPIを提供
    - Sync方式のためのfuture提供
- 数千個またはそれ以上のコネクションを同時に使用可能
- 状態ベースのシナリオ管理機能をサポート

### リファレンスプロジェクト

| サーバー                                                        | テストコード                                                 | 説明                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| [sample-game-server](https://github.com/nhn/gameanvil.sample-game-server.git) | [sample-game-test](https://github.com/nhn/gameanvil.sample-game-test.git) | 実際のゲームサーバーとGameHammerを使用したテストコード |

## プロジェクトにGameHammerを追加する

GameHammerはGameAnvilと同じようにMavenを通じて配布されます。 pom.xmlファイルのdependencies要素に次のように追加するとGameHammerを使用できます。

```
...
<dependencies>
        ...
        <!-- GameHammer -->
        <dependency>
			<groupId>com.nhn.gameanvil</groupId>
			<artifactId>gamehammer</artifactId>
			<version>1.4.1-jdk11</version>
		</dependency>
        ...
<dependencies>
...        
```

## MavenでGameHammer jarファイルを作成する

GameHammerを使ってテストシナリオを作成した後、GameAnvilコンソールでテストする目的などでjarファイルを作成できます。ここではMavenを使ってアップロード用jarファイルを作成します。

GameHammerを追加したプロジェクトのpom.xmlがあるディレクトリで下記のコマンドを実行します。

```
mvn package
```

コマンド実行後、ビルド過程が出力され、最後にビルドが成功したというメッセージを確認します。下記のように表示されたら成功です。

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.880 s
[INFO] Finished at: 2023-11-13T17:49:48+09:00
[INFO] ------------------------------------------------------------------------
Process finished with exit code 0
```

新しく作成されたターゲットディレクトリ内でビルドされたファイルを確認できます。
