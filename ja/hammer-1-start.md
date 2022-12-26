## Game > GameAnvil > テスト開発ガイド > 始める

## GameHammer

GameHammerは、GameAvlilエンジンを利用したゲームサーバー開発ツールで、強力で便利なテストツールです。実際のコネクタで提供するすべての機能を使用することができ、多様なテストケースを作成できるAPIを提供しています。またストレステストを行うための複数のGameHammerを同時に実行し、その結果をまとめて確認できます。

### システム要求事項

GameHammerを使用するには次のような事項が必要です。 

- サポートする言語
    - Java
- ターゲット開発環境
    - InteliJ
- サポートするネットワークプロトコル
    - TCP/IP
    - SSL over TCP/IP
- 使用可能な応用プロトコル形式
    - Google Protocol Buffers
    - Google FlatBuffers(予定)
    - カスタムバイトストリーム
    - HTTP/HTTPS(特定の用途に限定)

### 特徴

GameHammerは次のような機能をサポートします。

- コネクタと対応するすべての機能をサポート
- Sync/Async方式を全てサポート
    - Async方式のAPIを提供
    - Sync方式用のfutureを提供
- 数千個以上のコネクションを同時に使用可能
- 状態ベースのシナリオ管理機能をサポート

### リファレンスプロジェクト

| サーバー                                                     | テストコード                                              | 説明                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| [RPS](https://github.nhnent.com/game-server-engine/GameAnvil-rps) | [RPS-test](https://github.nhnent.com/game-server-engine/GameHammer-rps-test) | 実際のゲームサーバーとGameHammerを使用したテストコード |

## プロジェクトにGameHamerを追加する

GameHammerはGameAnvilと同様にMavenを介して配布されます。pom.xmlファイルのdependencies要素に次のように追加するとGameHammerを使用できます。

```
...    
<dependencies>
        ...
        <!-- GameHammer -->
        <dependency>
			<groupId>com.nhn.gameanvil</groupId>
			<artifactId>gamehammer</artifactId>
			<version>1.2.1-jdk11</version>
		</dependency>
        ...
<dependencies>
...        
```
