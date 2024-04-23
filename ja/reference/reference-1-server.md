## Game > GameAnvil > リファレンスプロジェクト > サーバーサンプル

# GameAnvil API JavaDoc

[GameAnvil Server API - Java doc](https://gameplatform.toast.com/docs/api/)



# サーバーのダウンロード

[Sample Game Server](https://github.com/nhn/gameanvil.sample-game-server.git)



# 構成環境

* IDE: Intellij 2022.3.3

* JDK: openjdk version "11.0.16.1" 2022-08-12 LTS

* **GameAnvil 1.4.1**

* DB
  * Jasync-sql 1.2.3:基本使用
    * MyBatis 3.5.3
  * それぞれの環境に合わせてIPアドレスを設定
  * MySQL 8.0.23

* Redis
  * GameAnvilが提供するLettuce APIを使用
  * それぞれの環境に合わせてIPアドレスを設定
  
  

# サーバー駆動

## IntelliJ使用環境

GitリポジトリでcloneしたプロジェクトをIntelliJで実行します。

基本設定はGradle設定Dependenciesに**com.nhn.gameanvil:gameanvil:1.4.1-jdk11**でJDK11バージョンが使われています。

resources/GameAnvilConfig.jsonファイルにIPが127.0.0.0.1になっています。



### 実行環境設定

サンプルサーバーは基本JDK11に設定されているので、IntelliJが他のバージョンに設定されている場合、設定をJDK11に合わせるとビルド時にエラーが発生しません。

もし、IntelliJでGameAnvilライブラリとJDKのバージョンを合わせずに実行すると、次のようなエラーが発生します。

```java
Exception in thread "main" java.lang.UnsupportedClassVersionError: co/paralleluniverse/fibers/SuspendExecution has been compiled by a more recent version of the Java Runtime (class file version 54.0), this version of the Java Runtime only recognizes class file versions up to 52.0
```

IntelliJ JDKの設定は以下をご確認ください。

File > Project Structure > Project Settings > ProjectメニューでJDKを確認します。

![reference-1-server_01](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_01.png) 

IntelliJ IDEA > Settings > Build, Execution, Deployment > Buil Tools > Gradle > Gradle JVMメニューでJDKを確認します。

![reference-1-server_02](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_02.png) 

Gradle JDKを変更したら、GradleタブのReloadを実行してプロジェクトに反映します。

![reference-1-server_03](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_03.png) 

ビルド環境設定は下記の内容を順番に設定します。IntelliJのバージョンによって画面は少し違う場合があります(スクリーンショットは2023.12バージョンです)。

### Redisの設定確認

パスワードなしでRedisに接続する場合は、`com.nhn.gameanvil.sample.common.GameConstants`クラスのRedis接続情報を修正して接続します。

```java
    // Redis接続情報
    public static final String REDIS_URL = "接続アドレス";
    public static final int REDIS_PORT = 7500;
```



パスワード情報が設定されている場合は、Redisに接続する際に`com.nhn.gameanvil.sample.redis.RedisHelper`クラスのコメントアウトされた部分のパスワード設定を使って接続します。

```java
    /**
     * Redis接続処理。使用前に最初に1回呼び出し、接続が必要
     *
     * @param url接続url
     * @param port接続port
     * @throws SuspendExecutionこのメソッドはファイバーをsuspendできることを意味します。
     */
    public void connect(String url, int port) throws SuspendExecution {
        // Redis接続処理
        RedisURI clusterURI = RedisURI.Builder.redis(url, port).build();

        // パスワードが必要な場合はパスワード設定を追加してRedisURIを作成
//        RedisURI clusterURI = RedisURI.Builder.redis(url, port).withPassword("password").build();
        this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI));
        this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient);

        if (this.clusterConnection.isOpen()) {
            logger.info("============= Connected to Redis using Lettuce =============");
        }

        this.clusterAsyncCommands = clusterConnection.async();
    }
```

### DB設定確認

サンプルサーバーでは基本的にJasync-sqlを基本的に使用し、StoredProcedureでクエリを使います。

Mybatis接続時、`com.nhn.gameanvil.sample.common.GameConstants.USE_DB_JASYNC_SQL`の値をfalseに変更すると、DBがmybatisで動作します。

#### サンプル使用DBスキーマ

サンプルで使っているテーブル情報です。

```sql
CREATE TABLE `users` (
  `uuid` varchar(40) NOT NULL,
  `login_type` int(11) NOT NULL,
  `app_version` varchar(45) DEFAULT NULL,
  `app_store` varchar(45) DEFAULT NULL,
  `device_model` varchar(45) DEFAULT NULL,
  `device_country` varchar(45) DEFAULT NULL,
  `device_language` varchar(45) DEFAULT NULL,
  `nickname` varchar(45) DEFAULT NULL,
  `heart` int(11) NOT NULL,
  `coin` bigint(15) DEFAULT '0',
  `ruby` bigint(15) DEFAULT '0',
  `level` int(11) DEFAULT '1',
  `exp` bigint(15) DEFAULT '0',
  `high_score` bigint(15) DEFAULT '0',
  `current_deck` varchar(45) NOT NULL,
  `create_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`uuid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



使用しているStoredProcedure情報です。

```sql
DELIMITER $$
DROP PROCEDURE IF EXISTS sp_users_insert $$
CREATE PROCEDURE sp_users_insert
(
	pi_uuid VARCHAR(40),
	pi_login_type INT,
	pi_app_version VARCHAR(45),
	pi_app_store VARCHAR(45),
	pi_device_model VARCHAR(45),
	pi_device_country VARCHAR(45),
	pi_device_language VARCHAR(45),
	pi_nickname VARCHAR(45),
	pi_heart INT,
	pi_coin BIGINT,
	pi_ruby BIGINT,
	pi_level INT,
	pi_exp BIGINT,
	pi_high_score BIGINT,
	pi_current_deck VARCHAR(45)   
)
BEGIN 
    DECLARE err INT default '0';
	DECLARE continue handler for SQLEXCEPTION set err = -1;

	START TRANSACTION;

	INSERT INTO users (uuid, login_type, app_version, app_store, device_model, device_country, device_language, nickname, heart, coin, ruby, level, exp, high_score, current_deck, create_date, update_date) 
    VALUES (pi_uuid, pi_login_type, pi_app_version, pi_app_store, pi_device_model, pi_device_country, pi_device_language, pi_nickname, pi_heart, pi_coin,	pi_ruby, pi_level, pi_exp, pi_high_score, pi_current_deck, NOW(), NOW());

    SELECT ROW_COUNT();

    IF err < 0 THEN
 	    ROLLBACK;
    ELSE
 	    COMMIT;
    END IF;
END $$
DELIMITER ;

DELIMITER $$
DROP PROCEDURE IF EXISTS sp_users_select_uuid $$
CREATE PROCEDURE sp_users_select_uuid
(
	pi_uuid VARCHAR(40)
)
BEGIN 
	SELECT * 
    FROM users 
    WHERE uuid = pi_uuid;
END $$
DELIMITER ;

DELIMITER $$
DROP PROCEDURE IF EXISTS sp_users_update_high_score $$
CREATE PROCEDURE sp_users_update_high_score
(
	pi_uuid VARCHAR(40),
   	pi_high_score BIGINT
)
BEGIN 
    DECLARE err INT default '0';
	DECLARE continue handler for SQLEXCEPTION set err = -1;

	START TRANSACTION;

	UPDATE users
    SET high_score = pi_high_score, update_date = NOW()
    WHERE uuid = pi_uuid;

    SELECT ROW_COUNT();

    IF err < 0 THEN
 	    ROLLBACK;
    ELSE
 	    COMMIT;
    END IF;
END $$
DELIMITER ;

DELIMITER $$
DROP PROCEDURE IF EXISTS sp_users_update_current_deck $$
CREATE PROCEDURE sp_users_update_current_deck
(
	pi_uuid VARCHAR(40),
	pi_current_deck VARCHAR(45)   
)
BEGIN 
    DECLARE err INT default '0';
	DECLARE continue handler for SQLEXCEPTION set err = -1;

	START TRANSACTION;

	UPDATE users
    SET current_deck = pi_current_deck, update_date = NOW()
    WHERE uuid = pi_uuid;
    
    SELECT ROW_COUNT();    

    IF err < 0 THEN
 	    ROLLBACK;
    ELSE
 	    COMMIT;
    END IF;    
END $$
DELIMITER ;

DELIMITER $$
DROP PROCEDURE IF EXISTS sp_users_update_nickname $$
CREATE PROCEDURE sp_users_update_nickname
(
	pi_uuid VARCHAR(40),
	pi_nickname VARCHAR(45)   
)
BEGIN 
    DECLARE err INT default '0';
	DECLARE continue handler for SQLEXCEPTION set err = -1;

	START TRANSACTION;

	UPDATE users
    SET nickname = pi_nickname, update_date = NOW()
    WHERE uuid = pi_uuid;
	
    SELECT ROW_COUNT();
    
    IF result < 0 THEN
	    ROLLBACK;
    ELSE
	    COMMIT;
    END IF;

END $$
DELIMITER ;
```



#### Jasync-sql

Jasync-sqlを使用すると、`com.nhn.gameanvil.sample.common.GameConstants`クラスのDB接続情報を修正して接続します。

```java
    // DB接続情報
    public static final String DB_USERNAME = "ユーザー名";
    public static final String DB_HOST = "ホスト名";
    public static final int DB_PORT = 3306;
    public static final String DB_PASSWORD = "パスワード";
    public static final String DB_DATABASE = "データベース名";
    public static final int MAX_ACTIVE_CONNECTION = 30;
```



#### Mybatis

Mybatis接続はresources/mybatid-config.xmlの接続設定を修正して接続します。

```xml
  <!-- MySQL接続情報を指定します。 -->
  <properties>
    <property name="hostname" value="ホスト名" />
    <property name="portnumber" value="3306" />
    <property name="database" value="データベース名" />
    <property name="username" value="ユーザー名" />
    <property name="password" value="パスワード" />
  </properties>
```


### サーバー実行

GradleタブのrunMain実行で、IntelliJで実行"gameanvil.sample-game-server [runMain]"

先に設定しておいた「SampleGameServer」構成を使ってサーバーを実行します。

![reference-1-server_04](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_04.png)

![reference-1-server_05](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_05.png) 

サーバーが正常に動作されると、下記のように全てのノードに対してonReadyログが出力されます。

http://127.0.0.1:18400/management/nodeInfoPageページでローカルで実行されたノードの状態を確認できます。全てのノードがREADYになったら、正常に実行されたということです。

![reference-1-server_06](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_06.png) 

構成はGameNode 4, GatewayNode 4, SupportNode2, IpcNode 1, ManagementNode 1, Locationnode 2, LocationNookupNode1,  MatchNode 1, GatewayNetworkNode 1, SupportNetwotNode1合計18個ノードが表示されます。


### エラーの確認

正常にサーバーが実行されない場合、設定を再確認するか、logのエラー部分を確認してお問い合わせください。

DBやRedisの場合、サンプルサーバーに適用された部分はチーム内部で使う部分なので、直接構築して指定する必要があります。

DBやRedisの設定がない場合、サンプルサーバーが正常に動作しません。


### Gradleビルド
GameAnvilバージョン確認
```groovy
dependencies {
    api 'com.nhn.gameanvil:gameanvil:1.4.1-jdk11'
}
```


build.gradle設定

```groovy
import java.nio.file.Paths

plugins {
    id 'java'
    id 'java-library'
}

[compileJava, compileTestJava]*.options*.encoding = 'UTF-8'

group = 'com.nhn.gameanvil'
version = '1.4.1'
java.sourceCompatibility = JavaVersion.VERSION_11
java.targetCompatibility = JavaVersion.VERSION_11

repositories {
    mavenLocal()
    mavenCentral()
}

configurations {
    quasar
    api.setCanBeResolved(true)
    all {
        resolutionStrategy {
            force 'com.esotericsoftware:kryo:4.0.2'
        }
    }
}

// standalone jarを作成します。
jar {
    baseName = "sample_game_server"
    duplicatesStrategy(DuplicatesStrategy.EXCLUDE)
    manifest {
        attributes 'Main-Class': 'com.nhn.gameanvil.sample.Main'
    }

    from {
        configurations.compileClasspath.collect {
            it.isDirectory() ? it : zipTree(it)
        }
    }
}

// GameAnvilサーバーを実行します。
task runMain(dependsOn: build, type: JavaExec) {
    jvmArgs = [
            "-Xms6g",
            "-Xmx6g",
            "-XX:+UseG1GC"]
    main = 'com.nhn.gameanvil.sample.Main'
    classpath = sourceSets.main.runtimeClasspath
}

compileJava {
    dependsOn.processResources

    doLast {
        ant.taskdef(name: 'instrumentation', classname: 'co.paralleluniverse.fibers.instrument.InstrumentationTask', classpath: configurations.api.asPath)
        ant.instrumentation(verbose: 'true', check: 'true', debug: 'true') {
            fileset(dir: 'build/classes/') {
                include(name: '**/*.class')
            }
        }
    }
}

dependencies {
    api files(Paths.get(project.projectDir.absolutePath, './src/main/resources/META-INF/quasar-core-0.8.0-jdk11.jar').toString())
    api 'org.mybatis:mybatis:3.5.3'
    api 'mysql:mysql-connector-java:8.0.23'
    api 'com.nhn.gameanvil:gameanvil:1.4.1-jdk11'
}
```

#### Gradle jarビルド
GradleタブのGameAnvilTutorial > Tasks > build > jarでプロジェクトをビルドします。

![reference-1-server_07](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_07.png)

正常にビルドが完了したら、プロジェクトフォルダbuild/libsにビルドされたjarファイルが作成されます。

![reference-1-server_08](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_08.png)

## Commandでサーバー実行

Commandでサーバーを駆動するためにはGradleでビルドされたsample_game_server-1.4.1.jarを使用します。

![reference-1-server_09](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_09.png)


```
java -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar sample_game_server-1.4.1.jar
```

- 基本的に実行時に別途オプションが指定されない場合、ビルド時に指定された環境ファイルが適用されます。
- GameAnvilConfig.jsonは-Dconfig.fileオプションでパスとファイル名を指定します。
- logback.xmlは -Dlogback.configurationFileオプションでパスとファイル名を指定します。
- mybatis用のconfig設定は -DmybatisConfigオプションを指定します。
  - mybatisはGameAnvilがサポートする機能の一部ではなく、ここでは例として提示しました。
  - com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactoryを参照

実行すると、IntelliJで実行したようにonReadyログが表示され、http://127.0.0.1:18400/management/nodeInfoPageページに全てのノードがREADYと表示されたら正常起動されたということです。

# sample-game-serverサーバーを見る

ゲーム開発の参考用にGameAnvilサンプルクライアントと連動するために作ったプロジェクトです。

- 機能の使いやすさを目的にしているので,ゲーム自体に対するエラーやバグが多い場合があります。
- 詳細は[サポート](https://www.toast.com/kr/support/inquiry)にお問い合わせください。



## プロジェクト構成

### gateway

- 認証
  - GamebaseのuserId, tokenを受け取り、tokenを検証します。

### game

#### multi
- TapBird
  - ルームマッチで1～4人までプレイが可能です。ルームから全員が出るまでプレイしたスコアを全てのユーザーが受け取ります。
  - ルームが1つもない場合は、ルームを作成して入場
    - ルームにユーザーとスコア情報を登録
    - ルーム情報にルームIDと現在入室しているユーザー数を更新します。
  - ルームがある場合は、既存のルームに入場
    - ルームにユーザーとスコア情報を登録
    - ルーム情報に現在のユーザー数を更新します。
  - ルームから退室する時
    - ルームにユーザーとスコア情報を削除します。
    - ルーム情報のユーザー数を更新します。
  - ユーザースコア増加パケット処理
    - レスポンスを必要としない形で受け取ったスコアをルームのユーザー情報を更新します。
    - ユーザーのスコアリストを作成し、現在ルーム内にいる全てのユーザにスコアを伝達します。
- Snake
  - ユーザーマッチで2人が同時にルームに入室してゲームプレイ
  - 2人がマッチした時、1人はルームを作成し、1人は作成されたルームに入ります。
    - 最初のユーザー位置指定後、ルームにユーザー情報とスコア情報を登録します。
    - 2回目の入場時に位置指定とルームにユーザー情報とスコア情報を登録します。
    - ユーザーが2人になると、ユーザーにゲーム情報を送信します。
    - ルームでfood作成タイマーを開始します。
  - ルームでfoodの数がmaxになるまで1秒に1回ずつfoodを作成し、ユーザーにレスポンス不要のデータとして送信
  - ルームから退室する時
    - ルームのユーザー情報とスコア情報を削除します。
    - タイマーを停止します。
    - 全てのユーザーをルームから退室させます。
  - food削除パケット処理
    - 削除するfood情報を受け取り、ルームにあるfood情報を削除します。
    - 相手に削除されたfoodを渡します。
  - user移動パケット処理
    - ユーザーが移動した時の情報を受け取り、ルームにユーザー情報を保存し、相手に移動したユーザー情報を伝えます。

#### single

- 1人で部屋を作ってゲーム
  - ゲーム開始時に渡されたパケットにシングルルームで必要なデータルームに設定
- タブメッセージパケット処理
  - レスポンスを必要としないメッセージで伝達された場合、現在ルームにスコアを記録
- ルームから出る時
  - ログインしたユーザーオブジェクトの最高スコアよりルーム内に保存されたスコアが大きい場合
    - DBにユーザー最高記録アップデート
    - 成功した場合、Redisのランキング情報をアップデート
  - ゲーム終了パケットを処理して、現在のルームでプレイしたスコアをレスポンス

#### user

  - ログイン
    - DBでユーザー識別子を持ってユーザー照会
      - ユーザーがいる場合、DBから取得した情報を使用
      - ユーザーがいない場合、DBに新規情報を記録
    - ログインしたユーザー情報をRedisに記録
    - ユーザー情報レスポンス
  - ニックネーム変更リクエスト
    - DBでユーザーニックネーム変更
    - 成功すると、現在ログインしているユーザーオブジェクトのニックネームを変更
    - ユーザー情報レスポンス
  - デッキ交換リクエスト
    - デッキ交換のための財貨の控除を確認
      - サンプルでは、現在ログインした時に持っているユーザーオブジェクトからのみ控除を確認
    - 正常に差し引かれた場合
      - 現在ユーザーが持っているデッキはリストから削除し、他のデッキをランダムに設定
      - 変更されたデッキをDBに保存
      - 成功すると、現在ログインしているユーザーオブジェクトのデッキが変更
      - 変更されたデッキ、財貨残高をレスポンス
  - シングルゲームランキング情報リクエスト
    - 渡された範囲でRedisからシングルゲームランキングリストを検索
    - ログインするたびに保存したRedisのユーザー情報からランキングリストのニックネームを設定し、ランキングリスト作成
    - 作成されたランキングリストをレスポンス

### support 

- ゲームサーバーに接続する前にゲームセッション情報を呼び出すためのサービスです。

- ローンチ情報リクエストパケット処理

  - http://127.0.0.1:18600/launching?platform=Editor&appStore=GOOGLE&appVersion=1.2.0&deviceId=4D34C127-9C56-5BAB-A3C2-D8F18C0B7B6E形式でリクエストします。

  - 渡されたデータを解析して確認し、GatewayNodeサーバーのIP、PORTを返します。

### redis

- GameAnvilで提供するlettuce連動
- GameNodeのonInit()でRedis接続設定Nodeごとに作って接続
  - Singletonインスタンスで作成したため、全てのノードで1つのRedis接続を使うことはできません。
- GameNodeのonShutdown()でRedisの接続shutdown処理をする必要があります。
- 例機能
  - hmgetでユーザーデータリスト検索
  - zaddでシングルゲームランキングを保存
  - zreverangeWithScoresでランキング情報を検索

### db 

#### jasync-sql

* 非同期SQLを処理できるクラスです。
* インスタンスはノードごとに1つずつ作成して使用し、サーバーが終了する時に閉じてください。
* サンプルではStoredProcedureでクエリが作成されています。

#### mybatis

- resources/mybatisにDB設定情報だけあります。
- com.nhn.gameanvil.sample.mybatis.mappersパッケージ内にmapper.xmlファイルが存在します。
- 例機能
  - ユーザー情報INSERT
  - uuidでユーザー情報SELECT
  - デッキ、ニックネーム、最高スコアUPDATE

### protocol 

- google protobuf 3.0を使用します。
- クライアントと使用するすべてのパケットはgoogle protobufで作成します。
- .protoファイルはクライアントと共用で作成し、build.batにあるようにサーバーで使うため.javaファイルに変換して使います。
- 使用プロトコル例
  - Authentication.proto:認証、ログイン
  
  - GameMulti.proto:マルチゲーム
  
  - GameSingle.proto:シングルゲーム
  
  - Result.proto:レスポンスコード
  
  - User.proto:ユーザー
  
  - プラグインがインストールされたら、次のようにbuild.batファイルを右クリックして次のようなコマンドでintelliJで直接変換できます。
  
    ![reference-1-server_10](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_10.png) 
  
    ![reference-1-server_11](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_11.png) 

## サーバー動作内容

### サーバー動作設定

メインクラスのGameAnvilServerを設定して実行します。設定時にクライアントとのプロトコルを同じ順番で登録し、サーバーで使うスレッドプールを作成します。

```java
        GameAnvilServer gameAnvilServer = GameAnvilServer.getInstance();

        // クライアントと転送するプロトコルの定義 - 順番はクライアントと同じでなければなりません。
        gameAnvilServer.addProtoBufClass(Authentication.getDescriptor());
        gameAnvilServer.addProtoBufClass(GameMulti.getDescriptor());
        gameAnvilServer.addProtoBufClass(GameSingle.getDescriptor());
        gameAnvilServer.addProtoBufClass(Result.getDescriptor());
        gameAnvilServer.addProtoBufClass(User.getDescriptor());

        // ゲームで使用するDBスレッドプール指定
        gameAnvilServer.createExecutorService(GameConstants.DB_THREAD_POOL, 100);
        // ゲームで使用するRedisスレッドプールを指定
        gameAnvilServer.createExecutorService(GameConstants.REDIS_THREAD_POOL, 100);

        // annotationクラス登録処理のためのscan packageを指定
        gameAnvilServer.addPackageToScan("com.nhn.gameanvil.sample");

        // サーバー実行
        gameAnvilServer.run();
```



GameAnvilで使用できるようにGatewayNode, GameNode, SupportNode, MatchMaker, Roomは@annotaionでクラスと名前を指定する必要があります。サーバーが実行される時、宣言されたクラスはエンジンで自動的にクラス登録処理が行われます。

BaseConnectionを継承して実装したコネクションクラスに宣言します。

```java
@Connection()
```

BaseGatewayNodeを継承して実装したゲートウェイノードに宣言します。

```
@GatewayNode()
```

BaseSessionを継承して実装したセッションに宣言します。

```
@Session()
```

BaseGameNodeを継承して実装したゲームノードにGameAnvilConfig.jsonに設定したゲームノードのサービス名を指定します。

```
@ServiceName(GameConstants.GAME_NAME)
```

BaseUserを継承して実装したゲームユーザーにゲームのサービス名とユーザータイプを指定します。ユーザータイプはクライアントに接続して作成されるタイプです。

```
@ServiceName(GameConstants.GAME_NAME)
@UserType(GameConstants.GAME_USER_TYPE)
```

BaseRoomを継承して実装したルームにゲームサービス名とルームタイプを指定します。ルームタイプはクライアントから接続するルームタイプを指定します。

```java
// シングルゲーム
@ServiceName(GameConstants.GAME_NAME)
@RoomType(GameConstants.GAME_ROOM_TYPE_SINGLE)

// ルームマッチゲーム
@ServiceName(GameConstants.GAME_NAME)
@RoomType(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH)

// ユーザーマッチゲーム
@ServiceName(GameConstants.GAME_NAME)
@RoomType(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH)
```

BaseUserMatchMaker, BaseRoomMatchMakerを継承して実装したマッチメーカーには、該当ルームと同じサービス名とルームタイプを指定します。

BaseSupportNodeを継承して実装したサポートノードに、GameAnvilConfig.jsonに設定したサポートノードのサービス名を指定します。

```
@ServiceName(GameConstants.SUPPORT_NAME_LAUNCHING)
```



### GameAnvilで処理されるAPI

#### GameSession - セッション

- com.nhn.gameanvil.sample.gateway.GameSession
  - onAuthenticate() :クライアントの認証要求に対する検証内容を実装

#### GameUser - ユーザー

- com.nhn.gameanvil.sample.game.user.GameUser
  - onLogin() :クライアントログインリクエストに対する内容を実装
  - onMatchRoom() :クライアントのマルチルームマッチリクエストに対する処理を実装
  - onMatchUser() :クライアントユーザーマッチリクエストに対する処理を実装
  - onTransferIn() / onTransferOut() :ユーザーがサーバーを移動する時、ユーザーオブジェクトのデータを転送し、復元する処理

#### SingleGameRoom - シングルルーム

- com.nhn.gameanvil.sample.game.single.SingleGameRoom
  - onCreateRoom():クライアントのシングルルーム作成内容を処理し、ルームを作成し、ルームに入場します。
  - onLeaveRoom():シングルルームを出る時の処理を実装

#### UnlimitedTapRoom - マルチルームマッチ、最大4人

- com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoom
  - onInit() :ルーム作成時の処理、ルームで使用するデータの初期化処理
  - onCreateRoom() :初回のルームマッチ用ルームを作成し、入室する部分を処理
  - onJoinRoom(): 2回目以降に入場するユーザーに対する処理
  - onLeaveRoom():ルームから退室する際の処理
  - onTransferIn() / onTransferOut() :ルームを他のサーバーに移動する時、ルーム内にあるデータを転送し、復元処理

#### UnlimitedTapRoomMatchMaker:ルームマッチロジックを処理

- com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoomMatchMaker

#### SnakeRoom - ユーザーマッチ, 2人

- com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom
  - onInit() :ルーム作成時の処理、ルームで使用するデータの初期化処理
  - onCreateRoom() :初回のルームマッチ用ルームを作成し、入室する部分を処理
  - onJoinRoom() : 2回目以降に入場するユーザーに対する処理
  - onPostLeaveRoom():ルーム退室後の処理、2人でゲームするルームなので、1人が退室するとタイマーを解除し、相手も退室させます。
  - onTransferIn() / onTransferOut() :ルームを他のサーバーに移動する時、ルーム内にあるデータを転送し、復元処理
  - onTimer() :サーバーで定期的にfoodを生成してルームにいるユーザーにデータを送信

#### SnakeRoomMatchMaker : 2人マッチするロジック処理

- com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoomMatchMaker
  - onMatch() :ルームマッチロジックを処理

### パケット登録

ゲームコンテンツで定義して処理するパケットとして、クライアントとやり取りするパケットを登録します。処理するクラスは、登録した種類に合わせたパケットハンドラーインターフェイスを実装する必要があります。

ユーザーがログインした状態で処理するパケット: com.nhn.gameanvil.sample.game.user.GameUser

```java
static private PacketDispatcher packetDispatcher = new PacketDispatcher();

static {
    packetDispatcher.registerMsg(User.ChangeNicknameReq.getDescriptor(), CmdChangeNicknameReq.class);           // ニックネーム変更プロトコル
    packetDispatcher.registerMsg(User.ShuffleDeckReq.getDescriptor(), CmdShuffleDeckReq.class);                 // デッキシャッフルプロトコル
    packetDispatcher.registerMsg(GameSingle.ScoreRankingReq.getDescriptor(), CmdSingleScoreRankingReq.class);   // シングルスコアランキング
}
// 処理するクラスはimplements IPacketHandler<GameUser> を実装して作る必要があります。
```

クライアントからrequestでリクエストされたパケットについては、クライアントでレスポンスを待っているので、サーバーで処理して渡されたユーザーオブジェクトを経由してgameUser.reply()でレスポンス処理をする必要があります。



ルーム内にいるときに処理するパケット: com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom

```java
private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();

static {
    dispatcher.registerMsg(GameMulti.SnakeUserMsg.getDescriptor(), CmdSnakeUserMsg.class);  // ユーザー位置情報
    dispatcher.registerMsg(GameMulti.SnakeFoodMsg.getDescriptor(), CmdSnakeRemoveFoodMsg.class);  // food削除情報処理
}
// 処理するクラスはimplements IRoomPacketHandler<SnakeRoom, GameUser>を実装して作る必要があります。
```

サーバーからクライアントへ渡すパケットはgameUser.send()でレスポンスを待たずに送信します。



restパケット: com.nhn.gameanvil.sample.support.LaunchingSupport

```java
private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher();

static {
    // launching
    restMsgHandler.registerMsg("/launching", RestObject.GET, CmdLaunching.class);
}
// 処理するクラスはimplements IRestPacketHandlerを実装して作る必要があります。
```

restリクエストに対しては、受け取ったrestObject.writeString()でレスポンスメッセージを渡します。


### Redis

com.nhn.gameanvil.sample.redis.RedisHelper

接続処理

```java
private RedisClusterClient clusterClient;
private StatefulRedisClusterConnection<String, String> clusterConnection;
private RedisAdvancedClusterAsyncCommands<String, String> clusterAsyncCommands;
/**
 * Redis接続、使用前に最初に1回呼び出して接続する必要があります。
 *
 * @param url接続url
 * @param port接続port
 * @throws SuspendExecution
 */
public void connect(String url, int port) throws SuspendExecution {    // Redis接続処理
    RedisURI clusterURI = RedisURI.Builder.redis(url, port).build();

    // パスワードが必要な場合、パスワード設定を追加してRedisURIを作成します。
//  RedisURI clusterURI = RedisURI.Builder.redis(url, port).withPassword("password").build();
    this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI));
    this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient);
    this.clusterAsyncCommands = clusterConnection.async();
}
```

終了処理

```java
/**
 * 接続終了サーバがダウンする前に呼び出されなければならない。
 */
public void shutdown() {
    clusterConnection.close();
    clusterClient.shutdown();
}
```

使用

```java
/**
 * ユーザーデータをRedisに保存
 *
 * @param gameUserInfoユーザー情報
 * @return保存の成否
 * @throws SuspendExecution
 */
public boolean setUserData(GameUserInfo gameUserInfo) throws SuspendExecution {
    String value = GameAnvilUtil.Gson().toJson(gameUserInfo);

    boolean isSuccess = false;
    try {
        Lettuce.awaitFuture(clusterAsyncCommands.hset(REDIS_USER_DATA_KEY, gameUserInfo.getUuid(), value)); // この戻り値は最初に設定した時だけtrueで、値を更新する時はfalseでレスポンス
        isSuccess = true;
    } catch (TimeoutException e) {
        logger.error("setUserData - timeout", e);
    }
    return isSuccess;
}
```



### DB

#### Jasync-sql

com.nhn.gameanvil.sample.db.jasyncsql.JAsyncSqlManager

接続処理

```java
    // JAsyncSQL接続
    public JAsyncSqlManager(String username, String host, int port, String password, String database, int maxActiveConnection) {
        jAsyncSql = new JAsyncSql(new com.github.jasync.sql.db.Configuration(
            username,
            host,
            port,
            password,
            database), maxActiveConnection);

        logger.info("JAsyncSqlManager::JAsyncSql connect");
    }
```

終了処理
```java
    // JAsyncSQL終了
    public void close() {
        jAsyncSql.disconnect();
    }
```

使用
```java
    /**
     * ユーザーの最高スコアを保存
     *
     * @param uuid   ユーザーのユニークな識別子
     * @param highScore修正する最高スコア
     * @return修正されたレコード数を返す
     * @throws TimeoutExceptionこの呼び出しに対してタイムアウトが発生する可能性があることを意味します。
     * @throws SuspendExecutionこのメソッドはファイバーをsuspendできることを意味します。
     */
    public int updateUserHigScore(String uuid, int highScore) throws TimeoutException, SuspendExecution {
        String sql = "CALL sp_users_update_high_score( '"
            + uuid + "', "
            + highScore + ")";
        QueryResult queryResult = jAsyncSql.execute(sql);

        return getStoredProcedureRowCount(queryResult);

    }
```



#### Mybatis

DB接続情報設定: resources/maybatis-config.xml

```xml
<!-- MySQL接続情報を指定する。 -->
<properties>
  <property name="hostname" value="ホスト名" />
  <property name="portnumber" value="3306" />
  <property name="database" value="データベース名" />
  <property name="username" value="ユーザー名" />
  <property name="password" value="パスワード" />
  <property name="poolPingQuery" value="select 1"/>
  <property name="poolPingEnabled" value="true"/>
  <property name="poolPingConnectionsNotUsedFor" value="3600000"/>
</properties>
```

使うクエリ登録 - 外部xmlを使う場合、下記のコメント部分を参考にして使います。

```xml
  <mappers>
    <!-- 定義されたSQL構文をマッピングします。基本的にリソースの中にあるmapper.xmlを使う時 -->
    <mapper resource="query/UserDataMapper.xml"/>
    <!-- 外部で指定されたmapper.xmlファイルを指定する場合は、フルパス指定を使用します。 -->
    <!--<mapper url="file:///C:/_KevinProjects/GameServerEngine/sample-game-server/target/query/UserDataMapper.xml"/>-->
  </mappers>
```

クエリ: resources/query/UserDataMapper.xml

```xml
<select id="selectUserByUuid" resultType="com.nhn.gameanvil.sample.mybatis.dto.UserDto">
      SELECT        uuid,
        login_type AS loginType,
        app_version AS appVersion,
        app_store AS appStore,
        device_model AS deviceModel,
        device_country AS deviceCountry,
        device_language AS deviceLanguage,
        nickname,
        heart,
        coin,
        ruby,
        level,
        exp,
        high_score AS highScore,
        current_deck AS currentDeck,
        create_date AS createDate,
        update_date AS updateDate
      FROM users
      WHERE uuid = #{uuid}
  </select>
```

DB接続設定: com.nhn.gameanvil.sample.db.mybatis.GameSqlSessionFactory

```java
/**
 * ゲームで使用するDB接続オブジェクト
 */
public class GameSqlSessionFactory {
    private static Logger logger = LoggerFactory.getLogger(GameSqlSessionFactory.class);

    private static SqlSessionFactory sqlSessionFactory;

    /** XMLで指定された接続情報を読み込みます。 */
    // クラス初期化ブロック:クラス変数の複雑な初期化に使用します。
    // クラスが初めてロードされるときに一度だけ実行されます。
    static {
        // 接続情報を指定しているXMLのパスを読み込む
        try {
            // mybatis_config.xmlファイルのパスを指定
            String mybatisConfigPath = System.getProperty("mybatisConfig"); // パラメータが渡された場合、サーバー(実行時に-DmybatisConfig=オプションで指定)
            logger.info("mybatisConfigPath : {}", mybatisConfigPath);
            if (mybatisConfigPath != null) {
                logger.info("load to mybatisConfigPath : {}", mybatisConfigPath);
                InputStream inputStream = new FileInputStream(mybatisConfigPath);
                if (sqlSessionFactory == null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
                }
            } else {    // パラメータを渡さない場合、内部ファイルから設定を取得する。
                Reader reader = Resources.getResourceAsReader("mybatis/mybatis-config.xml");
                logger.info("load to resource : mybatis/mybatis-config.xml");
                // sqlSessionFactoryが存在しない場合は作成する。
                if (sqlSessionFactory == null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * データベース接続オブジェクトを使ってDATABASEに接続したセッションを返す。
     */
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

実行時、-DmybatisConfig=を使ってない場合、sessionを作る時、次のようにビルド時に入った内部保存環境ファイルに設定されるログが記録されます。

```java
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : null
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:39] load to resource : mybatis-config.xml
```

実行時に-DmybatisConfig=を使用した場合、sessionを作成する時、次のように指定された位置情報に設定されるログが記録されます。

```java
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : .\src\main\resources\mybatis-config.xml
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:32] load to mybatisConfigPath : .\src\main\resources\mybatis-config.xml
```

使用: com.nhn.gameanvil.sample.db.mybatis.UserDbHelperService

```java
/**
 * ユーザー情報DBに保存
 *
 * @param gameUserInfoユーザー情報を伝達
 * @return保存されたレコード数
 * @throws TimeoutException
 * @throws SuspendExecution
 */
public int insertUser(GameUserInfo gameUserInfo) throws TimeoutException, SuspendExecution {    // Callable形式でAsync実行し、結果を返す。
    Integer resultCount = Async.callBlocking(GameConstants.DB_THREAD_POOL, new Callable<Integer>() {
        @Override
        public Integer call() throws Exception {
            SqlSession sqlSession = GameSqlSessionFactory.getSqlSession();
            try {
                UserDataMapper userDataMapper = sqlSession.getMapper(UserDataMapper.class);
                int resultCount = userDataMapper.insertUser(gameUserInfo.toDtoUser());
                if (resultCount == 1) { // 単件保存なので、1つなら正常にDB commit
                    sqlSession.commit();
                }
                return resultCount;
            } finally {
                sqlSession.close();
            }
        }
    });
    return resultCount;
}
```

## サーバー設定
### GameAnvilConfig.json

```json
{
  //-------------------------------------------------------------------------------------
  // 共通情報。

  "common": {
    "ip": "127.0.0.1", // ノードごとに共通で使用するIP (マシンのIPを指定)
    "meetEndPoints": ["127.0.0.1:18000"], // 対象ノードのcommon IPとcommunicatePortを登録。 (該当サーバーendpointを含めることができ、リストで複数指定可能)
    "debugMode": false // デバッグ時に各種タイムアウトが発生しないようにするオプション。リアルでは必ずfalseでなければなりません。
  },

  //-------------------------------------------------------------------------------------
  // LocationNode設定
  "location": {
    "clusterSize": 1, // 合計何台のマシン(VM)で構成されるか?
    "replicaSize": 1, // レプリケーショングループのサイズ(master + slaveの数)
    "shardFactor": 2  // shardingのための引数(下記のコメント参照)
    // 全体shardの数= clusterSize x replicaSize x shardFactor
    // 1つのマシン(VM)で駆動するshardの個数= replicaSize x shardFactor
    // 固有のshardの総数(master shardの数) = clusterSize x shardFactor
  },

  // マッチノード設定
  "match": {
    "nodeCnt": 1
  },

  //-------------------------------------------------------------------------------------
  // クライアントとのコネクションを管理するノード。
  "gateway": {
    "nodeCnt": 4, // ノード数(ノード番号は0から付与される)。
    "ip": "127.0.0.1", // クライアントと接続するIP。
    "dns": "", // クライアントと接続するドメインアドレス。
    "connectGroup": { // コネクションの種類。
      "TCP_SOCKET": {
        "port": 18200, // クライアントと接続するポート。
        "idleClientTimeout": 240000 // データの送受信がない状態以降のタイムアウト(0なら使用しない)。
      },
      "WEB_SOCKET": {
        "port": 18300,
        "idleClientTimeout": 0
      }
    }
  },

  //-------------------------------------------------------------------------------------
  // ゲームロビーの役割をするノード(ゲームルーム、ユーザーを含む)。
  "game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "TapTap",
      "channelIDs": ["","","","",""], // ノードごとに付与するチャンネルID(一意である必要はない。" "文字列でチャンネルを区別せずに重複して使うこともできる)。
      "userTimeout": 5000 // disconnect後のユーザーオブジェクト除去タイムアウト。
    }
  ],

  "support": [
    {
      "nodeCnt": 2,
      "serviceId": 2,
      "serviceName": "Launching",
      "restIp": "127.0.0.1",
      "restPort": 18600
    }
  ]
}
```

### logback.xml

loggerをパッケージ名単位で分けて指定できます。別々に指定すると、そのパッケージ名は指定されたレベルで適用され、指定されなければrootで指定された設定で適用されます。

```
    <logger name="com.nhn.gameanvil" level="INFO"/>
    <logger name="com.nhn.gameanvil.sample" level="DEBUG"/>

    <root>
        <level value="WARN"/>
        <appender-ref ref="ASYNC"/>
        <appender-ref ref="STDOUT"/>
    </root>
```
