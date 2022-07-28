## Game > GameAnvil > サーバー開発ガイド > レファレンスサーバー

## ダウンロード

https://github.nhnent.com/game-server-engine/sample-game-server.git

![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-1.png)



## レファレンス開発環境

- IDE: IntelliJ 2019.3
- JDK: AdoptOpenJDK build 1.8.0_275-b01
- **GameAnvil 1.1.0**
- DB
  - MyBatis 3.5.3
  - 環境に合わせてIPアドレスを設定
  - MySQL 5.7.29
- Redis
  - GameAnvilで提供するLettuce APIを使用
  - 環境に合わせてIPアドレスを設定



## GameAnvil API Java doc

[GameAnvil Server API - Java doc](http://10.162.4.61:9090/gameanvil)



## IntelliJで実行環境を設定

- GitリポジトリからcloneしたプロジェクトをIntelliJで実行します。
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-2.png)
- sample_game_sever設定の確認 - 基本ローカルで実行する基本環境
  - Maven設定Dependenciesに**com.nhn.gameanvil:gameanvil:1.1.0-jdk8**確認
  - GameAnvileはJDK8/JDK11をサポート
    - JDK8: 1.1.0-jdk8を使用
    - JDK11: 1.1.0-jdk11を使用
  - resources/GameAnvilConfig.jsonファイルでIPが127.0.0.1になっていることを確認
- IntelliJサーバー実行環境設定
  - プロジェクト設定
    ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-3.png)
    ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-4.png)
    - GameAnvilはJDK 1.8で作成し、複数のバージョンのJDKがインストールされている場合は、1.8に設定されているかを確認し、指定します。
      - 1.8ではない場合、maven packageまたはinstall時にエラーが発生します。
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-5.png)
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-6.png)
  - 以下の内容を順序どおりに設定します。
    - 1：クリックして新しいビルド環境設定を追加
    - 2： +でApplicationを追加→ 3作成
    - 4：ビルド環境名を設定sample_server
    - 5：プロジェクトMainクラスを選択(com.nhn.gameanvil.sample.Main)
    - 6: resources/setting.txtの # VM Optionsにある内容を入力(Macで開発する場合はパスの形式に注意)

```
-Dco.paralleluniverse.fibers.detectRunawayFibers=FALSE
-Dco.paralleluniverse.fibers.verifyInstrumentation=FALSE
-Xms6g
-Xmx6g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication
```

- 7： resources/setting.txtファイルで# Program Arguments項目の内容を入力

```
src/main/resources/
```

- 8： JRE 1.8に設定
- 9：設定を保存



## IntelliJでサーバーを実行する

- Mavenタブのinstallコマンドでサーバーをインストールします。この時、コンパイルタイムにAOT Instrumentationを進行します。

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-7.png)

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-8.png)

- 設定しておいた"sample_server"構成を利用してサーバーを実行します。

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-9.png)

- サーバーが正常に起動したら、以下のようにすべてのノードのonReadyログが出力されます。

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-10.png)

- http://127.0.0.1:25150/management/nodeInfoPage

   URLで現在ローカルのsample_game_serverの状態を確認できます。

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-11.png)

- エラー確認

  - 正常にサーバーが実行できなかった場合は、設定をもう一度確認してみるか、logのエラー部分を確認してお問い合わせください。
  - DBまたはRedisの場合は、直接使用するIPアドレスを設定する必要があります。



## MavenビルドとCommand実行

##### pom.xmlの設定確認

- GameAnvilバージョン

```
    <!-- gameanvil-->
    <dependency>
      <groupId>com.nhn.gameanvil</groupId>
      <artifactId>gameanvil</artifactId>
      <version>1.1.0-jdk8</version>
    </dependency>
```

- build設定

```
<build>
<plugins>
  <plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
      <source>1.8</source>
      <target>1.8</target>
      <encoding>UTF-8</encoding>
    </configuration>
  </plugin>

  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
      <archive>
        <manifest>
          <!-- executable jarでmainで実行されるクラス -->
          <mainClass>com.nhn.gameanvil.sample.Main</mainClass>
          <!-- jarファイル内のMETA-INF/MANIFEST.MFにclasspath情報が追加される -->
          <addClasspath>true</addClasspath>
        </manifest>
      </archive>
    </configuration>
  </plugin>

    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-shade-plugin</artifactId>
      <version>3.2.4</version>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>shade</goal>
          </goals>
          <configuration>
            <transformers>
              <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>com.nhn.gameanvil.sample.Main</mainClass>
              </transformer>
              <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/io.netty.versions.properties</resource>
              </transformer>
              <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/services/java.sql.Driver</resource>
              </transformer>
              <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/LICENSE</resource>
              </transformer>
              <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/NOTICE</resource>
              </transformer>
              <transformer implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/services/reactor.blockhound.integration.BlockHoundIntegration</resource>
              </transformer>
            </transformers>
            <artifactSet>
              <excludes>
                <exclude>javax.activation:javax.activation-*</exclude>
                <exclude>org.javassist:javassist*</exclude>
              </excludes>
            </artifactSet>
            <filters>
              <filter>
                <artifact>*:*</artifact>
                <excludes>
                  <exclude>module-info.class</exclude>
                  <exclude>META-INF/*.SF</exclude>
                  <exclude>META-INF/*.DSA</exclude>
                  <exclude>META-INF/*.RSA</exclude>
                  <exclude>META-INF/*.MF</exclude>
                  <exclude>META-INF/*.txt</exclude>
                  <exclude>about.html</exclude>
                </excludes>
              </filter>
            </filters>
            <createDependencyReducedPom>false</createDependencyReducedPom>
          </configuration>
        </execution>
      </executions>
    </plugin>

    <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
      <version>3.0.0</version>
      <executions>
        <execution>
          <goals>
            <goal>exec</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <executable>java</executable>
        <arguments>
          <argument>-classpath</argument>
          <!-- automatically creates the classpath using all project dependencies, also adding the project build directory -->
          <classpath/>
          <!-- Main class -->
          <argument>com.nhn.gameanvil.sample.Main</argument>

        </arguments>
      </configuration>
    </plugin>

    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-antrun-plugin</artifactId>
      <executions>
        <!-- Ant task for Quasar AOT instrumentation -->
        <execution>
          <id>Running AOT instrumentation</id>
          <phase>compile</phase>
          <configuration>
            <tasks>
              <taskdef name="instrumentationTask" classname="co.paralleluniverse.fibers.instrument.InstrumentationTask" classpathref="maven.dependency.classpath"/>
              <instrumentationTask>
                <fileset dir="${project.build.directory}/classes/" includes="**/*.class"/>
              </instrumentationTask>
            </tasks>
          </configuration>
          <goals>
            <goal>run</goal>
          </goals>
        </execution>
        <execution>
          <phase>package</phase>
          <configuration>
            <tasks>
              <copy todir="target/config/" overwrite="false">
                <fileset dir="target/classes/">
                  <include name="logback.xml" />
                  <include name="mybatis-config.xml" />
                  <include name="GameAnvilConfig.json" />
                </fileset>
              </copy>
              <copy todir="target/query/" overwrite="false">
                <fileset dir="target/classes/query/">
                  <include name="*.xml" />
                </fileset>
              </copy>
            </tasks>
          </configuration>
          <goals>
            <goal>run</goal>
          </goals>
        </execution>
      </executions>
    </plugin>

  </plugins>
  <resources>
    <resource>
      <directory>src/main/resources</directory>
    </resource>
  </resources>
</build>
```

##### Maven packageビルド実行

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-12.png)

- 正常に実行できたら、./target/ フォルダにビルドされたファイルがあります。

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-13.png)

- サーバーを起動するには、sample_game_server-1.0.1.jarファイルとconfig, queryフォルダのファイルをコピーして使用してください。

- コマンドプロンプトから実行

  - コマンドプロンプト(cmd)を実行してビルドされたtargetフォルダに移動します(自分の環境に合ったパスで進行)。

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-14.png)

  - command実行

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-15.png)

    - ```
      java -Dco.paralleluniverse.fibers.detectRunawayFibers=false -Dco.paralleluniverse.fibers.verifyInstrumentation=false -Dconfig.file=.\config\GameAnvilConfig.json -Dlogback.configurationFile=.\config\logback.xml -DmybatisConfig=.\config\mybatis-config.xml -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar .\sample_game_server-1.1.0.jar
      ```

      - 基本的に、実行時に別途オプションが指定されていない場合はビルドする時に指定されている環境ファイルを適用
      - GameAnvilConfig.jsonは -Dconfig.fileオプションにパスとファイル名を指定
      - logback.xmlは -Dlogback.configurationFileオプションにパスとファイル名を指定
      - mybatis用config設定は -DmybatisConfigオプション指定
        - mybatisは、GameAnvilがサポートする機能の一部ではありません。この部分はサンプル用です。
      - com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory参考

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-16.png)

    - onReadyが表示されたら正常

- 毎回mavenビルドでテストするのが面倒で、ローカルでテストする時はVm Optionに`-javaagent:.\src\main\resources\META-INF\quasar-core-0.7.10-jdk8.jar=bm`オプションを追加し、intelliJですぐにローカルサーバーを実行できます。



## JIT設定(Optional)

- GameAnvilは、AOT Instrumentationだけでなく、JIT Instrumentationもサポートします。

### pom設定

- 以下のAOT Instrumentation ( )プラグイン部分をコメント処理するか、削除します。

```
<!-- Ant task for Quasar AOT instrumentation -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>
    <executions>
        <execution>
            <id>Running AOT instrumentation</id>
            <phase>compile</phase>

            <configuration>
                <tasks>
                    <taskdef name="instrumentationTask" classname="co.paralleluniverse.fibers.instrument.InstrumentationTask" classpathref="maven.dependency.classpath"/>
                    <instrumentationTask>
                        <fileset dir="${project.build.directory}/classes/" includes="**/*.class"/>
                    </instrumentationTask>
                </tasks>
            </configuration>

            <goals>
                <goal>run</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### VM option設定

- JITモードの場合、必ずVMオプションに以下のようなjavaagent項目を追加してquasar agentが起動するようにする必要があります。

```
-javaagent:.\src\main\resources\META-INF\quasar-core-0.7.10-jdk8.jar=bm
```

### ビルドおよび実行

- IntelliJ
  - intelliJを利用して実行
- CLI
  - maven clean
  - maven package
  - command実行(スクリプトまたはバッチファイルを作成して起動することを推薦)
    - `java -javaagent:.\lib\quasar-core-0.7.10-jdk8.jar=bm -Dco.paralleluniverse.fibers.detectRunawayFibers=false -Dconfig.file=.\config\GameAnvilConfig.json -Dlogback.configurationFile=.\config\logback.xml -DmybatisConfig=.\config\mybatis-config.xml -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar .\sample_game_server-1.1.0.jar`



## sample_game_serverの詳細表示

- GameAnvilのサンプルとして作られたプロジェクトです。参考にしていただき、気になる部分がある場合はいつでもお問い合わせください。

### Gamebase

- NHN CloudのGamebase登録
  - サーバーでアプリID、SecretKey必要

### Gateway

- 認証
  - GamebaseのuserId、tokenを受け取ってtokenを検証

### Game

- ユーザー
  - ログイン
    - DBでユーザー識別子を用いてユーザー照会
      - ユーザーがいる場合、DBで取得した情報を使用
      - ユーザーがいない場合、DBに新規情報を記録
    - ログインしたユーザー情報をRedisに記録
    - ユーザー情報レスポンス
  - ニックネーム変更リクエスト
    - DBにユーザーニックネーム変更
    - 成功したら、現在ログインしているユーザーオブジェクトのニックネームを変更
    - ユーザー情報レスポンス
  - デッキ交換リクエスト
    - デッキを交換するための財貨差し引き確認
      - サンプルでは現在ログインした時に持っているユーザーオブジェクトでのみ差し引きを確認
    - 正常に差し引かれる場合
      - 現在のユーザーが持っているデッキはリストから削除した後、他のデッキをランダムに設定
      - DBに変更されたデッキを保存
      - 成功すると、現在ログインしているユーザーオブジェクトのデッキを変更
      - 変更されたデッキ、財貨差し引きレスポンス
  - シングルゲームランキング情報リクエスト
    - 渡された範囲でRedisからシングルゲームランキングリストを検索
    - ログインするたびに保存しておいたRedisのユーザー情報からランキングリストのニックネームを設定してランキングリスト作成
    - 作成されたランキングリストレスポンス
- シングルゲーム
  - 一人でルームを作ってゲーム
    - ゲームを始める時、渡されたパケットにシングルルームで必要なデータをルームに設定
  - タブメッセージパケット処理
    - レスポンスが必要ないメッセージで渡されたら、現在のルームにスコアを記録
  - ルームから退出する時
    - ログインしているユーザーオブジェクトの最高スコアよりもルーム内に保存されたスコアが高い場合
      - DBにユーザー最高記録をアップデート
      - 成功するとRedisのランキング情報をアップデート
    - ゲーム終了パケット処理して、現在ルームでプレイしたスコアをレスポンス
- マルチゲーム
  - TapBird
    - ルームマッチで1～4人までプレイできます。ルームから全員退出するまで、プレイするスコアをすべてのユーザーが受け取ります。
    - ルームが1つもない場合はルームを作って入場
      - ルームにユーザーとスコア情報を登録
      - ルーム情報にルームIDと現在入場したユーザー数を更新します。
    - ルームがある場合、既にあるルームに入場
      - ルームにユーザーとスコア情報を登録
      - ルーム情報に現在のユーザー数を更新します。
    - ルームから退出する時
      - ルームのユーザーとスコア情報を削除します。
      - ルーム情報のユーザー数を更新します。
    - ユーザースコア増加パケット処理
      - レスポンスが必要ない形で渡されたスコアを、ルームのユーザー情報を更新
      - ユーザースコアリストを作成して、現在ルーム内にいるすべてのユーザーにスコアを伝達します。
  - Snake
    - ユーザーマッチで2人が同時にルームに入場してゲームプレイ
    - 2人がマッチした時、1人はルームを作って入り、1人は作成されたルームに入ります。
      - 最初のユーザーの位置を指定してルームにユーザー情報とスコア情報を登録します。
      - 2番目に入場する時、位置を指定してルームにユーザー情報とスコア情報を登録します。
      - ユーザーが2人になったらユーザーにゲーム情報を転送します。
      - ルームでfood作成タイマーを開始します。
    - ルームでfood数がmaxになるまで毎秒1回ずつfoodを作成してユーザーにレスポンスが必要ないデータとして転送
    - ルームから退出する時
      - ルームからユーザー情報とスコア情報を削除します。
      - タイマーを停止します。
      - すべてのユーザーをルームから出します。
    - food削除パケット処理
      - 削除するfood情報を受け取ってルームにあるfood情報を削除します。
      - 相手に削除されたfoodを伝達します。
    - user移動パケット処理
      - ユーザーが移動する時の情報を受け取ってルームにユーザー情報を保存し、相手に移動したユーザー情報を伝達します。

### Support - rest service

- ゲームサーバー接続前にゲームセッション情報を取得するためのサービス
- http://127.0.0.1:10080/launching?platform=Editor&appStore=GOOGLE&appVersion=1.2.0&deviceId=4D34C127-9C56-5BAB-A3C2-D8F18C0B7B6E形式でリクエスト
- ローンチ情報リクエストパケット処理
  - 渡されたデータを解析し、ゲームサーバーセッションサーバーのIP、PORTをレスポンスします。

### Redis

- GameAnvilで提供するlettuce連動
- GameNodeのonInit()でRedis接続設定Nodeごとに作って接続
  - Singletonで作ってすべてのノードで1つのRedis接続を使用してください。
- GameNodeのonShutdown()で、Redisの接続shutdown処理を行う必要があります。
- 例機能
  - hmgetでユーザーデータリストを検索
  - zaddでシングルゲームランキングを保存
  - zreverangeWithScoresでランキング情報を検索

### DB - MyBatis

- resources/mybatisにDB設定情報だけあります。
- com.nhn.gameanvil.sample.mybatis.mappersパッケージ内にmapper.xmlファイルが存在します。
- 機能例
  - ユーザー情報をINSERT
  - uuidでユーザー情報をSELECT
  - デッキ、ニックネーム、最高スコアをUPDATE

### protocol - google protobuf 3.0

- クライアントと使用するすべてのパケットはgoogle protobufで成
- .protoファイルはクライアントと共用で製作し、build.batにあるようにサーバーで使用するための.javaファイルに変換して使用します。
- 例使用プロトコル
  - Authentication.proto：認証、ログイン
  - GameMulti.proto：マルチゲーム
  - GameSingle.proto：シングルゲーム
  - Result.proto：レスポンスコード
  - User.proto：ユーザー
-
  - プラグインがインストールされている場合は、次のようにbuild.batファイルを右クリックして、次のようなコマンドでintelliJから変換できます。
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-18.png)
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-19.png)

### GameAnvilBootstrap : com.nhn.gameanvil.sample.Main

```
public static void main(String[] args) {
    GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

    // クライアントと転送するプロトコルの定義 - 順序はクライアントと同じでなければいけない。
    bootstrap.addProtoBufClass(0, Authentication.getDescriptor());
    bootstrap.addProtoBufClass(1, GameMulti.getDescriptor());
    bootstrap.addProtoBufClass(2, GameSingle.getDescriptor());
    bootstrap.addProtoBufClass(3, Result.getDescriptor());
    bootstrap.addProtoBufClass(4, User.getDescriptor());

    // ゲームで使用するDBスレッドプールの指定
    bootstrap.createExecutorService(GameConstants.DB_THREAD_POOL, 100);
    // ゲームで使用するスレッドプールの指定
    bootstrap.createExecutorService(GameConstants.REDIS_THREAD_POOL, 100);

    // セッション設定
    bootstrap.setGateway()
        .connection(GameConnection.class)
        .session(GameSession.class)
        .node(GameGatewayNode.class)
        .enableWhiteModules();

    // ゲームスペースの設定
    bootstrap.setGame(GameConstants.GAME_NAME)
        .node(GameNode.class)

        // シングルゲーム
        .user(GameConstants.GAME_USER_TYPE, GameUser.class)
        .room(GameConstants.GAME_ROOM_TYPE_SINGLE, SingleGameRoom.class)

        // ルームマッチマルチゲーム - ルームに入ってゲーム：無制限タブ
        .room(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH, UnlimitedTapRoom.class)
        .roomMatchMaker(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH, UnlimitedTapRoomMatchMaker.class, UnlimitedTapRoomInfo.class)

        // ユーザーマッチマルチゲーム - ユーザーマッチングによりゲーム同時入場：ステーキゲーム
        .room(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH, SnakeRoom.class)
        .userMatchMaker(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH, SnakeRoomMatchMaker.class, SnakeRoomInfo.class);

    // サービス設定
    bootstrap.setSupport(GameConstants.SUPPORT_NAME_LAUNCHING)
        .node(LaunchingSupport.class);

    bootstrap.run();
}
```

### GameAnvil内部で処理されるメソッド

- GameSession - セッション
  - com.nhn.gameanvil.sample.gateway.GameSession
    - onAutenticate() ：クライアント認証リクエストの検証内容を実装
- GameUser - ユーザー
  - com.nhn.gameanvil.sample.game.user.GameUser
    - onLogin() ：クライアントログインリクエストの内容を実装
    - onMatchRoom() ：クライアントマルチルームマッチリクエストの処理を実装
    - onMatchUser() ：クライアントユーザーマッチリクエストの処理を実装
    - onTransferIn() / onTransferOut() ：ユーザーがサーバーを移す時、ユーザーオブジェクトのデータを転送し、復元処理
- SingleGameRoom - シングルルーム
  - com.nhn.gameanvil.sample.game.single.SingleGameRoom
    - onCreateRoom()：クライアントシングルルーム作成内容処理、ルームを作ってルームに入場します。
    - onLeaveRoom()：シングルルームを退出する時の処理を実装
- UnlimitedTapRoom - マルチルームマッチ、最大4人
  - com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoom
    - onCreateRoom() ：最初のルームマッチ用ルームを作成して入場する部分の処理
    - onJoinRoom()： 2番目以降に入場するユーザーの処理
    - onLeaveRoom()：ルームから退出する時の処理
    - onTransferIn() / onTransferOut() ：ルームを他のサーバーに移す時、ルーム内にあるデータを転送し、復元処理
- UnlimitedTapRoomMatchMaker：ルームマッチロジック処理
  - com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoomMatchMaker
- SnakeRoom - ユーザーマッチ、2人
  - com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom
    - onCreateRoom() ：最初のルームマッチ用ルームを作成して入場する部分の処理
    - onJoinRoom() ： 2番目以降に入場するユーザーの処理
    - onPostLeaveRoom()：ルームを退出した後の処理、2人でゲームするルームのため、1人が退出したらタイマーを削除し、相手も出す。
    - onTransferIn() / onTransferOut() ：ルームを他のサーバーに移す時、ルーム内にあるデータを転送し、復元処理
    - onTimer() ：サーバーで定期的にfoodを作成してルームにいるユーザーにデータ転送
- SnakeRoomMatchMaker ： 2人マッチするロジック処理
  - com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoomMatchMaker

### パケット登録

- クライアントが転送するパケットの登録
- ゲームコンテンツで定義して処理するパケット
- 処理するクラスは登録された種類に合ったパケットハンドラインターフェイスを実装する必要があります。
- ユーザーがログイン状態で処理するパケット:com.nhn.gameanvil.sample.game.user.GameUser

```
static private PacketDispatcher packetDispatcher = new PacketDispatcher();

static {
    packetDispatcher.registerMsg(User.ChangeNicknameReq.getDescriptor(), CmdChangeNicknameReq.class);           // ニックネーム変更プロトコル
    packetDispatcher.registerMsg(User.ShuffleDeckReq.getDescriptor(), CmdShuffleDeckReq.class);                 // デッキシャッフルプロトコル
    packetDispatcher.registerMsg(GameSingle.ScoreRankingReq.getDescriptor(), CmdSingleScoreRankingReq.class);   // シングルスコアランキング
}
// 処理するクラスはimplements IPacketHandler<GameUser> を実装して作成する必要がある。
```

- クライアントからrequestでリクエストしたパケットには、クライアントでレスポンスを待機しているため、サーバーで処理して伝達されたユーザーオブジェクトを利用してgameUser.reply()でレスポンス処理をする必要があります。
- ルーム内にいる時に処理するパケット： com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom

```
private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();

static {
    dispatcher.registerMsg(GameMulti.SnakeUserMsg.getDescriptor(), CmdSnakeUserMsg.class);  // ユーザー位置情報
    dispatcher.registerMsg(GameMulti.SnakeFoodMsg.getDescriptor(), CmdSnakeRemoveFoodMsg.class);  // food削除情報処理
}
// 処理するクラスはimplements IRoomPacketHandler<SnakeRoom, GameUser>を実装して作成する必要がある。
```

- サーバーからクライアントに伝達するパケットは、gameUser.send()でレスポンスを待機せずに転送します。
- restパケット： com.nhn.gameanvil.sample.support.LaunchingSupport

```
private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher();

static {
    // launching
    restMsgHandler.registerMsg("/launching", RestObject.GET, CmdLaunching.class);
}
// 処理するクラスはimplements IRestPacketHandlerを実装して作成する必要がある。
```

- restリクエストは、渡されたrestObject.writeString()でレスポンスメッセージを伝達します。



### 外部httpリクエスト処理使用：com.nhn.gameanvil.sample.gateway.GameConnection

- ゲームサーバー → 外部サーバー例) Gamebase token検証
- GatewayサーバーonAutenticate()で検証リクエスト処理

```
// Gamebse認証
//----------------------------------- トークンの有効性を検証Gamebase
String gamebaseUrl = String.format(GameConstants.GAMEBASE_DEFAULT_URL + "/tcgb-gateway/v1.2/apps/X2bqX5du/members/%s/tokens/%s", accountId, authenticationReq.getAccessToken());
HttpRequest httpRequest = new HttpRequest(gamebaseUrl);
httpRequest.getBuilder().addHeader("Content-Type", "application/json");
httpRequest.getBuilder().addHeader("X-Secret-Key", GameConstants.GAMEBASE_SECRET_KEY);
logger.info("httpRequest url [{}]", gamebaseUrl);
HttpResponse response = httpRequest.GET();
logger.info("httpRequest response:[{}]", response.toString());

// Gamebaseレスポンスjsonデータオブジェクト解析
AuthenticationResponse gamebaseResponse = response.getContents(AuthenticationResponse.class);
if (gamebaseResponse.getHeader().isSuccessful())
{
    resultCode = ErrorCode.NONE;
} else {
    resultCode = ErrorCode.TOKEN_NOT_VALIDATED;
}
//------------------------------------
```

### Redis : com.nhn.gameanvil.sample.redis.RedisHelper

- 接続

```
private RedisClusterClient clusterClient;
private StatefulRedisClusterConnection<String, String> clusterConnection;
private RedisAdvancedClusterAsyncCommands<String, String> clusterAsyncCommands;
/**
 * Redis接続。使用する前に、最初に1度呼び出して接続する必要がある。
 *
 * @param url接続url
 * @param port接続port
 * @throws SuspendExecution
 */
public void connect(String url, int port) throws SuspendExecution {    // Redis接続処理
    RedisURI clusterURI = RedisURI.Builder.redis(url, port).build();
    this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI));
    this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient);
    this.clusterAsyncCommands = clusterConnection.async();
}
```

- 終了

```
/**
 * 接続終了サーバーが落ちる前に呼び出される必要がある。
 */
public void shutdown() {
    clusterConnection.close();
    clusterClient.shutdown();
}
```

- 使用

```
/**
 * ユーザーデータRedisに保存
 *
 * @param gameUserInfoユーザー情報
 * @return保存成否
 * @throws SuspendExecution
 */
public boolean setUserData(GameUserInfo gameUserInfo) throws SuspendExecution {
    String value = GameAnvilUtil.Gson().toJson(gameUserInfo);

    boolean isSuccess = false;
    try {
        Lettuce.awaitFuture(clusterAsyncCommands.hset(REDIS_USER_DATA_KEY, gameUserInfo.getUuid(), value)); // この戻り値は、最初にsetする時だけtrueで、値の更新時にはfalseを返す
        isSuccess = true;
    } catch (TimeoutException e) {
        logger.error("setUserData - timeout", e);
    }
    return isSuccess;
}
```

### DB

- 設定:resources/maybatis-config.xml
  - DB接続情報

```
<!-- MySQL接続情報を指定する。 -->
<properties>
  <property name="hostname" value="호스트명" />
  <property name="portnumber" value="3306" />
  <property name="database" value="데이터베이스명" />
  <property name="username" value="유저명" />
  <property name="password" value="패스워드" />
  <property name="poolPingQuery" value="select 1"/>
  <property name="poolPingEnabled" value="true"/>
  <property name="poolPingConnectionsNotUsedFor" value="3600000"/>
</properties>
```

- 使用するクエリーの登録 - 外部xmlを使用するには、以下のコメント部分を参考にして使用してください。

```
  <mappers>
    <!-- 定義されたSQL構文をマッピングする。基本的にリソース内にあるmapper.xmlを使用する時-->
    <mapper resource="query/UserDataMapper.xml"/>
    <!-- 外部のmapper.xmlファイルを指定する時は、全体パスを指定して使用する。 -->
    <!--<mapper url="file:///C:/_KevinProjects/GameServerEngine/sample-game-server/target/query/UserDataMapper.xml"/>-->
  </mappers>
```

- クエリー： resources/query/UserDataMapper.xml

```
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

- DB接続設定: com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory

```
/**
 * ゲームで使用するDB接続オブジェクト
 */
public class GameSqlSessionFactory {
    private static Logger logger = LoggerFactory.getLogger(GameSqlSessionFactory.class);

    private static SqlSessionFactory sqlSessionFactory;

    /** XMLに記載された接続情報を読み込む。 */
    // クラス初期化ブロック：クラス変数の複雑な初期化に使われる。
    // クラスが初めてロードされる時に一度だけ実行される。
    static {
        // 接続情報を明示しているXMLのパスの読み取り
        try {
            // mybatis_config.xmlファイルのパス指定
            String mybatisConfigPath = System.getProperty("mybatisConfig"); // パラメータが渡された場合、サーバー(実行時に-DmybatisConfig=オプションで指定)
            logger.info("mybatisConfigPath : {}", mybatisConfigPath);
            if (mybatisConfigPath != null) {
                logger.info("load to mybatisConfigPath : {}", mybatisConfigPath);
                InputStream inputStream = new FileInputStream(mybatisConfigPath);
                if (sqlSessionFactory '- null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
                }
            } else {    // パラメータがない場合は、内部ファイルで設定する
                Reader reader = Resources.getResourceAsReader("mybatis/mybatis-config.xml");
                logger.info("load to resource : mybatis/mybatis-config.xml");
                // sqlSessionFactoryが存在しない場合は作成する。
                if (sqlSessionFactory '- null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * データベース接続オブジェクトを使用してDATABASEに接続したセッションを返す。
     */
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

- 実行時、-DmybatisConfig=を使用しない場合、 sessionを作成する時、次のようにビルド時に入って内部保存環境ファイルに設定されているログが記録されます。

```
[2020-12-18 17:36:35,634] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : null
[2020-12-18 17:36:35,636] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:39] load to resource : mybatis-config.xml
```

- 実行時、-DmybatisConfig=を使用した場合、sessionを作成する時、次のように指定された位置情報で設定されているログが記録されます。

```
[2020-12-18 17:43:37,871] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : .\src\main\resources\mybatis-config.xml
[2020-12-18 17:43:37,871] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:32] load to mybatisConfigPath : .\src\main\resources\mybatis-config.xml
```

- 使用: com.nhn.gameanvil.sample.mybatis.UserDbHelperService

```
/**
 * ユーザー情報DBに保存
 *
 * @param gameUserInfoユーザー情報伝達
 * @return保存されたレコードの数
 * @throws TimeoutException
 * @throws SuspendExecution
 */
public int insertUser(GameUserInfo gameUserInfo) throws TimeoutException, SuspendExecution {    // Callable形式でAsyncを実行して結果を返す。
    Integer resultCount = Async.callBlocking(GameConstants.DB_THREAD_POOL, new Callable<Integer>() {
        @Override
        public Integer call() throws Exception {
            SqlSession sqlSession = GameSqlSessionFactory.getSqlSession();
            try {
                UserDataMapper userDataMapper = sqlSession.getMapper(UserDataMapper.class);
                int resultCount = userDataMapper.insertUser(gameUserInfo.toDtoUser());
                if (resultCount '- 1) { // 1件保存のため、1個なら正常にDB commit
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

- サンプル使用DBスキーマ

```
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

## GameAnvilConfig : resources/GameAnvilConfig.json



```
{
  //-------------------------------------------------------------------------------------
  // 共通情報。

  "common": {
    "ip": "127.0.0.1", // ノードごとに共通で使用するIP。(マシンのIPを指定)
    "meetEndPoints": ["127.0.0.1:16000"], // 対象ノードのcommon IPとcommunicatePort登録。 (該当サーバーのendpointを含められる。リストで複数可)
    "ipcPort": 16000, // 他のipc nodeと通信する時に使われるport
    "publisherPort" : 13300, // publish socket用のport
    "debugMode": false //デバッグ時、各種timeoutが発生しないようにするオプション。リアルでは必ずfalseにする必要がある。
  },

  //-------------------------------------------------------------------------------------
  // LocationNode設定
  "location": {
    "clusterSize": 1, // 複数のマシン(VM)で構成されているか？
    "replicaSize": 3, // レプリケーショングループのサイズ(master + slaveの数)
    "shardFactor": 3  // shardingのための引数(以下のコメント参照)
    // shardの総数= clusterSize x replicaSize x shardFactor
    // 1つのマシン(VM)で起動するshardの数= replicaSize x shardFactor
    // 固有のshardの総数(masterシャードの数) = clusterSize x shardFactor
  },

  // マッチノード設定
  "match": {
    "nodeCnt": 1,
    "useLocationDirect": true
  },

  //-------------------------------------------------------------------------------------
  // クライアントとのコネクションを管理するノード。
  "gateway": {
    "nodeCnt": 4, // ノード数。(ノード番号は0から付与される)
    "ip": "127.0.0.1", // クライアントと接続されるIP。
    "dns": "", // クライアントと接続しているドメインのアドレス。
    "maintenance": false,
    "tcpNoDelay": false, // Netty Bootstrap設定時に使用される。(デフォルトでフィールド未使用および基本値false)
    "connectGroup": { // コネクションの種類。
      "TCP_SOCKET": {
        "port": 11200, // クライアントと接続しているポート。
        "idleClientTimeout": 240000 // データの送受信がない状態以降のタイムアウト。(0の場合は使用しない)
        //        ,"secure": { // セキュリティ設定。
        //          "useSelf": true
        ////          ,"keyCertChainPath": "gameanvil.crt" // 証明書のパス。
        ////          ,"privateKeyPath": "privatekey.pem" // 秘密鍵のパス。
        //        }
      },
      "WEB_SOCKET": {
        "port": 11400,
        "idleClientTimeout": 0
        //        ,"secure": {
        //          "useSelf": true
        ////          ,"keyCertChainPath": "gameanvil.crt"
        ////          ,"privateKeyPath": "privatekey.pem"
        //        }
      }
    }
  },

  //-------------------------------------------------------------------------------------
  // ゲームロビーの役割をするノード。(ゲームルーム、ユーザーを含んでいる)
  "game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "TapTap",
      "channelIDs": ["","","","",""], // ノードごとに付与するチャンネルID。(ユニークでなくてもよい。""文字列でチャンネルの区別なく重複使用も可能)
      "userTimeout": 5000 // disconnect以降のユーザーオブジェクト削除タイムアウト。
    }
  ],

  "support": [
    {
      "nodeCnt": 2,
      "serviceId": 2,
      "serviceName": "Launching",
      "restIp": "127.0.0.1",
      "restPort": 10080
    }
  ],

  //-------------------------------------------------------------------------------------
  // JMXまたはREST API使用して他のノードの管理を行うことができるノード。(サービスポーズ、全体ユーザーカウントなど)
  "management": {
    "nodeCnt": 2,
    "restIp": "127.0.0.1",
    "restPort": 25150,
    "consoleProxyPort" : 18081, // admin web console port
    "logProxyPort" : 18082,     // admin log download port

    "db": {
      "user": "root",
      "password": "1234",
      "url": "jdbc:h2:mem:gameanvil_admin;DB_CLOSE_DELAY'-1"
    }
  }
}
```

## logback : resources/logback.xml

- loggerをパッケージ名単位で区切って指定できます。別途指定すると、そのパッケージ名は指定されたレベルで適用され、なければrootに指定された設定で適用されます。

```
    <logger name="com.nhn.gameanvil" level="INFO"/>
    <logger name="com.nhn.gameanvil.sample" level="DEBUG"/>

    <root>
        <level value="WARN"/>
        <appender-ref ref="ASYNC"/>
        <appender-ref ref="STDOUT"/>
    </root>
```
