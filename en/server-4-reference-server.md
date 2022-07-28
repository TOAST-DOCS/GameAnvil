## Game > GameAnvil > Server Development Guide > Reference Server

## Download

https://github.nhnent.com/game-server-engine/sample-game-server.git

![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-1.png)



## Reference development environment

- IDE: IntelliJ 2019.3
- JDK: AdoptOpenJDK build 1.8.0_275-b01
- **GameAnvil 1.1.0**
- DB
  - MyBatis 3.5.3
  - Sets the IP address according to each environment
  - MySQL 5.7.29
- Redis
  - Uses the Lettuce API provided by GameAnvil
  - Sets the IP address according to each environment



## GameAnvil API Java doc

[GameAnvil Server API - Java doc](http://10.162.4.61:9090/gameanvil)



## Sets the run-time environment on IntelliJ

- Run the project cloned from the Git storage with IntelliJ. 
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-2.png)
- Checks the sample_game_sever setting - The basic environment that will be run on default local
  - Checks the Maven setting Dependencies **com.nhn.gameanvil:gameanvil:1.1.0-jdk8**
  - GameAnvil supports JDK8/JDK11
    - Uses JDK8: 1.1.0-jdk8
    - Uses JDK11: 1.1.0-jdk11
  - Checks the IP of resources/GameAnvilConfig.json is 127.0.0.1
- Sets the IntelliJ server run-time environment
  - Sets the project
    ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-3.png)
    ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-4.png)
    - GameAnvil checks if the version of various JDKs and set it to 1.8.
      - If the version is not 1.8, an error occurs while running maven package or installing.
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-5.png)
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-6.png)
  - Set the content below in the order they are shown.
    - 1: Click to add the preferences of a new build environment
    - 2: Add Application using + -> 3 Create
    - 4: Set the build environment sample_server
    - 5: Select project Main class (com.nhn.gameanvil.sample.Main)
    - 6: Enter the content in VM Options of resources/setting.txt # (Be cautious when developing on Mac)

```
-Dco.paralleluniverse.fibers.detectRunawayFibers=false
-Dco.paralleluniverse.fibers.verifyInstrumentation=false
-Xms6g
-Xmx6g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication
```

- 7: Enter the content of # the Program Arguments item in resources/setting.txt

```
src/main/resources/
```

- 8: Set to JRE 1.8
- 9: Save the settings



## Run the server using IntelliJ

- Install the server using the install command of the Maven tab. At this time, process AOT Instrumentation at compile.

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-7.png)

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-8.png)

- Run the server using the predefined "sample_server" configuration.

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-9.png)

- The onReady log is displayed in every node if the server works normally.

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-10.png)

- http://127.0.0.1:25150/management/nodeInfoPage

   The status of the sample_game_server of local through URL.

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-11.png)

- Check for error

  - Check the settings again or the error part of the log and contact us if the server does not run normally.
  - For DB or Redis, manually set the IP address to use.



## Execute the Maven build and Command

##### Check the pom.xml settings

- GameAnvil version

```
    <!-- gameanvil-->
    <dependency>
      <groupId>com.nhn.gameanvil</groupId>
      <artifactId>gameanvil</artifactId>
      <version>1.1.0-jdk8</version>
    </dependency>
```

- build settings

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
          <!-- The class to be executed as main in executable jar -->
          <mainClass>com.nhn.gameanvil.sample.Main</mainClass>
          <!-- classpath information is added to the META-INF/MANIFEST.MF in the jar file -->
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

##### Run the Maven package build

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-12.png)

- The built files can be found in the ./target/ folder if it ran normally.

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-13.png)

- To operate the server, copy the sample_game_server-1.0.1.jar file and the files in the config and query folder and use them.

- Run on the command prompt

  - Run the command prompt (cmd) and move to the built target folder(Proceed in the path that fits to one's environment).

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-14.png)

  - Run command

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-15.png)

    - ```
      java -Dco.paralleluniverse.fibers.detectRunawayFibers=false -Dco.paralleluniverse.fibers.verifyInstrumentation=false -Dconfig.file=.\config\GameAnvilConfig.json -Dlogback.configurationFile=.\config\logback.xml -DmybatisConfig=.\config\mybatis-config.xml -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar .\sample_game_server-1.1.0.jar
      ```

      - Apply the preference file specified at the time of build if there is no option specified when running as default
      - For GameAnvilConfig.json, use the -Dconfig.file option to specify path and file name
      - For logback.xml, use the -Dlogback.configurationFile option to specify path and file name
      - the config setting for mybatis, specify the -DmybatisConfig option
        - mybatis is not part of the features supported by GameAnvil. This part is a sample.
      - Refer to com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-16.png)

    - It is normal if onReady appears

- If testing on local instead of testing with the maven build each time, the `-javaagent:.\src\main\resources\META-INF\quasar-core-0.7.10-jdk8.jar=bm` option can be added to Vm Option and directly run the local server in intelliJ.



## Set JIT (optional)

- GameAnvil supports not only AOT Instrumentation but also JIT Instrumentation.

### Set pom

- Comment out or delete the AOT Instrumentation ( ) plug-in part below.

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

### Set VM option

- For JIT mode, the javaagent item below must be added to the VM option, allowing quasar agent to be run.

```
-javaagent:.\src\main\resources\META-INF\quasar-core-0.7.10-jdk8.jar=bm
```

### Build and run

- IntelliJ
  - Run using intelliJ
- CLI
  - maven clean
  - maven package
  - Run command (It is recommended to write a script or batch file and run it)
    - `java -javaagent:.\lib\quasar-core-0.7.10-jdk8.jar=bm -Dco.paralleluniverse.fibers.detectRunawayFibers=false -Dconfig.file=.\config\GameAnvilConfig.json -Dlogback.configurationFile=.\config\logback.xml -DmybatisConfig=.\config\mybatis-config.xml -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar .\sample_game_server-1.1.0.jar`



## Explore sample_game_server

- This project is created by using GameAnvil samples. Refer to it and feel free to contact us if there is inconvenience or a question.

### Gamebase

- Register the Gamebase of NHN Cloud
  - Server requires app ID and SecretKey

### Gateway

- Authentication
  - Receive the userId and token of Gamebase and authenticate the token

### Game

- User
  - Login
    - Look up users in DB using user identifier
      - Use the information retrieved from DB if there is a user
      - Record new information to DB if there is no user
    - Record the information of logged in user to Redis
    - User information response
  - Nickname change request
    - Change user nickname in DB
    - The nickname of the user object that is currently logged in if successful
    - User information response
  - Deck replace request
    - Check the currency deduction for replacing deck
      - In the sample, check only the user object the user had at the time of login for deduction
    - When it is normally deducted
      - The deck that the user currently has is removed from the list and randomly set a different deck
      - Save the deck changed from DB
      - Change the deck of the user object that is currently logged in if successful
      - Response to the changed deck and currency balance
  - Request for single game ranking information
    - Search for the single game ranking list in Redis with the passed range
    - Create a rankings list by setting a rankings list from the user information of the Redis stored whenever the user logs in
    - Respond to the created rankings list
- Single-player game
  - Play the game by creating a room alone
    - Set to the data room required by single room in the packet passed when starting the game
  - Process tab message packet
    - Record the score in the current room when a message that does not require response is received
  - When leaving the room,
    - if the score stored in the room is greater than the high score of the user object logged in,
      - update DB with the user's high score
      - Update the ranking information of Redis if successful
    - The response to the score of the play in the room by processing game end packet
- Multiplayer game
  - TapBird
    - 1-4 players can play with room match. All the players receive their score until everyone leaves the room.
    - If there is no room, create a room and join
      - Register the user and score information to the room
      - Updates the number of users joined the room and the room ID in the room information.
    - If there is a room, join the existing room
      - Register the user and score information to the room
      - Updates the current number of users in the room information.
    - When leaving the room
      - delete the users and score information from the room.
      - Updates the number of users of the room information.
    - Process the user score increase packet
      - Update the room's user information with the score received that does not require a response
      - Creates a user score list and passes the score to every user in the room.
  - Snake
    - 2 players join the room at the same time with user match and play the game
    - When 2 players are matched, one creates and joins a room and the other joins the created room.
      - Registers the user information and score information to the room after the location of the first user is specified.
      - Registers the user information and score information to the location specification and room when joining the room for the second time.
      - Transfers game information to the users when the number of users reaches 2.
      - Starts the food creation timer in the room.
    - Create food every second until the number of foods reaches to the maximum number and transfer it to users in the form of data that does not require response
    - When leaving the room,
      - delete user information and score information from the room.
      - Stops the timer.
      - Kicks every user out of the room.
    - Process food deletion packet
      - Receives the food information to be deleted and delete the food information from the room.
      - Passes the deleted food to the opponent.
    - Process user move packet
      - Receives the information when the user moves, stores the user information in the room, and transfers the moved user information to the opponent.

### Support - rest service

- The service that is used to retrieve game session information before connecting to game server스
- Requests in the form of http://127.0.0.1:10080/launching?platform=Editor&appStore=GOOGLE&appVersion=1.2.0&deviceId=4D34C127-9C56-5BAB-A3C2-D8F18C0B7B6E
- Process launching information request packet
  - Parses the passed data, checks it, and responds with the IP and PORT of the game server session.

### Redis

- Link the lettuce provided by GameAnvil
- Create it in every Redis connection setting Node and connect them in - GameNode's onInit()
  - Do not use a single Redis connection in every node made with Singleton.
- The connection shutdown of Redis needs to be processed in GameNode's onShutdown().
- Example feature
  - Search the user data list with hmget
  - Store single-player game rankings with ZADD
  - Search ranking information with zreverangeWithScores

### DB - MyBatis

- There is only DB setting information in resources/mybatis.
- The mapper.xml file exists in the com.nhn.gameanvil.sample.mybatis.mappers package.
- Example feature
  - INSERT user information
  - SELECT user information with UUID
  - UPDATE deck, nickname, and high score

### protocol - google protobuf 3.0

- Write the client and every packet used with google protobuf
- The .proto file is developed to be shared with the client. Use it after converting it into a .java file that is used by server as in build.bat.
- Example of protocol usage
  - Authentication.proto: Authentication, login
  - GameMulti.proto: Multiplayer game
  - GameSingle.proto: Single-player game
  - Result.proto: Response code
  - User.proto: User
-
  - If plug-in is installed, right click the following build.bat file to directly convert it using the following command in intelliJ.
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-18.png)
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-19.png)

### GameAnvilBootstrap : com.nhn.gameanvil.sample.Main

```
public static void main(String[] args) {
    GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

    // The client and the order of the protocols to be transferred - must be the same with the client.
    bootstrap.addProtoBufClass(0, Authentication.getDescriptor());
    bootstrap.addProtoBufClass(1, GameMulti.getDescriptor());
    bootstrap.addProtoBufClass(2, GameSingle.getDescriptor());
    bootstrap.addProtoBufClass(3, Result.getDescriptor());
    bootstrap.addProtoBufClass(4, User.getDescriptor());

    // Specify the DB thread pool used in game
    bootstrap.createExecutorService(GameConstants.DB_THREAD_POOL, 100);
    // Specify the Redis thread pool used in game
    bootstrap.createExecutorService(GameConstants.REDIS_THREAD_POOL, 100);

    // Set session
    bootstrap.setGateway()
        .connection(GameConnection.class)
        .session(GameSession.class)
        .node(GameGatewayNode.class)
        .enableWhiteModules();

    // Set game space
    bootstrap.setGame(GameConstants.GAME_NAME)
        .node(GameNode.class)

        // Single-player game
        .user(GameConstants.GAME_USER_TYPE, GameUser.class)
        .room(GameConstants.GAME_ROOM_TYPE_SINGLE, SingleGameRoom.class)

        // Room match multiplayer game - Play the game in the room: Unlimited tab
        .room(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH, UnlimitedTapRoom.class)
        .roomMatchMaker(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH, UnlimitedTapRoomMatchMaker.class, UnlimitedTapRoomInfo.class)

        // User match multiplayer game - Matched users enter a room at the same time: Stake game
        .room(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH, SnakeRoom.class)
        .userMatchMaker(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH, SnakeRoomMatchMaker.class, SnakeRoomInfo.class);

    // Set service
    bootstrap.setSupport(GameConstants.SUPPORT_NAME_LAUNCHING)
        .node(LaunchingSupport.class);

    bootstrap.run();
}
```

### A method that is processed internally by GameAnvil

- GameSession - Session
  - com.nhn.gameanvil.sample.gateway.GameSession
    - onAutenticate(): Implement the content of verification for client authentication request
- GameUser - User
  - com.nhn.gameanvil.sample.game.user.GameUser
    - onLogin(): Implement the content about client login request
    - onMatchRoom(): Implement the process for client multi-room match request
    - onMatchUser(): Implement the process for client user match request
    - onTransferIn()/onTransferOut(): Transfer and recover user object data when the user moves the server
- SingleGameRoom - Single-player room
  - com.nhn.gameanvil.sample.game.single.SingleGameRoom
    - onCreateRoom(): Process the content of client single-player room, create a room and join it.
    - onLeaveRoom(): Implement the process for leaving a single-player room
- UnlimitedTapRoom - Multiplayer room match, up to 4 players
  - com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoom
    - onCreateRoom() : Process the part where the room for initial room matching
    - onJoinRoom(): Process the users who join the room after the second
    - onLeaveRoom(): Process when leaving room
    - onTransferIn() / onTransferOut() : Transfer and recover the data in the room when moving the room to a different server
- UnlimitedTapRoomMatchMaker: Process room match logic
  - com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoomMatchMaker
- SnakeRoom - User match, 2 players
  - com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom
    - onCreateRoom(): Process the part where the room for initial room matching
    - onJoinRoom(): Process users after the second
    - onPostLeaveRoom(): Process after leaving the room, as it is a room for two player, if one player leaves the room, remove the timer and have the other player leave the room.
    - onTransferIn()/onTransferOut(): Transfer and recover the data in the room when moving the room to a different server
    - onTimer(): Regularly create food from server and pass it to the users in the room
- SnakeRoomMatchMaker : Process the logic for matching 2 players
  - com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoomMatchMaker

### Register packet

- Register the packet that is passed by the client
- A packet defined and processed by game content
- The class to be processed must implement the packet handler according to the registered type.
- The packet the user processes while logged in: com.nhn.gameanvil.sample.game.user.GameUser

```
static private PacketDispatcher packetDispatcher = new PacketDispatcher();

static {
    packetDispatcher.registerMsg(User.ChangeNicknameReq.getDescriptor(), CmdChangeNicknameReq.class);           // Nickname change protocol
    packetDispatcher.registerMsg(User.ShuffleDeckReq.getDescriptor(), CmdShuffleDeckReq.class);                 // Deck shuffle protocol
    packetDispatcher.registerMsg(GameSingle.ScoreRankingReq.getDescriptor(), CmdSingleScoreRankingReq.class);   // Single-player score rankings
}
// The class to be processed must be created by implementing implements IPacketHandler<GameUser>.
```

- As the packet passed from the client as a request waits for the response from the client, the response process as gameUser.reply() through the user object processed and passed by server.
- A packet processed when in the room: com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom

```
private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();

static {
    dispatcher.registerMsg(GameMulti.SnakeUserMsg.getDescriptor(), CmdSnakeUserMsg.class);  // User location information
    dispatcher.registerMsg(GameMulti.SnakeFoodMsg.getDescriptor(), CmdSnakeRemoveFoodMsg.class);  // food Process deleted information
}
// The class to be processed must be created by implementing implements IRoomPacketHandler<SnakeRoom, GameUser>.
```

- The packet passed from the server to the client transfers gameUser.send() without waiting.
- rest packet: com.nhn.gameanvil.sample.support.LaunchingSupport

```
private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher();

static {
    // launching
    restMsgHandler.registerMsg("/launching", RestObject.GET, CmdLaunching.class);
}
// The class being processed must implement and create implements IRestPacketHandler.
```

- For a rest request, transfers response message to the passed restObject.writeString().



### Use external http request process: com.nhn.gameanvil.sample.gateway.GameConnection

- Game server -> Example of external server) Authenticate Gamebase token
- Process authentication request in Gateway server onAutenticate()

```
// Authenticate Gamebase
//----------------------------------- Gamebase that authenticates the token's validity
String gamebaseUrl = String.format(GameConstants.GAMEBASE_DEFAULT_URL + "/tcgb-gateway/v1.2/apps/X2bqX5du/members/%s/tokens/%s", accountId, authenticationReq.getAccessToken());
HttpRequest httpRequest = new HttpRequest(gamebaseUrl);
httpRequest.getBuilder().addHeader("Content-Type", "application/json");
httpRequest.getBuilder().addHeader("X-Secret-Key", GameConstants.GAMEBASE_SECRET_KEY);
logger.info("httpRequest url [{}]", gamebaseUrl);
HttpResponse response = httpRequest.GET();
logger.info("httpRequest response:[{}]", response.toString());

// Parse Gamebase response json data object
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

- Link

```
private RedisClusterClient clusterClient;
private StatefulRedisClusterConnection<String, String> clusterConnection;
private RedisAdvancedClusterAsyncCommands<String, String> clusterAsyncCommands;
/**
 * Link to Redis, it must be initially called and linked before used.
 *
 * @param url  Connection URL
 * @param port Connection port
 * @throws SuspendExecution
 */
public void connect(String url, int port) throws SuspendExecution {    // Redis connection process
    RedisURI clusterURI = RedisURI.Builder.redis(url, port).build();
    this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI));
    this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient);
    this.clusterAsyncCommands = clusterConnection.async();
}
```

- End

```
/**
 * It must be called before the connection end server goes down.
 */
public void shutdown() {
    clusterConnection.close();
    clusterClient.shutdown();
}
```

- Use

```
/**
 * Store in user data Redis
 *
 * @param gameUserInfo User information
 * @return Whether or not the storage was successful
 * @throws SuspendExecution
 */
public boolean setUserData(GameUserInfo gameUserInfo) throws SuspendExecution {
    String value = GameAnvilUtil.Gson().toJson(gameUserInfo);

    boolean isSuccess = false;
    try {
        Lettuce.awaitFuture(clusterAsyncCommands.hset(REDIS_USER_DATA_KEY, gameUserInfo.getUuid(), value)); // The corresponding return value is true only when it is initially set, the response is false when the value is updated
        isSuccess = true;
    } catch (TimeoutException e) {
        logger.error("setUserData - timeout", e);
    }
    return isSuccess;
}
```

### DB

- Setting: resources/maybatis-config.xml
  - DB connection information

```
<!-- Specifies MySQL connection information. -->
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

- Register the query to be used - To use external xml, refer to the comment below before using.

```
  <mappers>
    <!-- Map the defined SQL statement. Basically when using the mapper.xml inside a resource-->
    <mapper resource="query/UserDataMapper.xml"/>
    <!-- Use entire path specification when specifying an externally specified mapper.xml file. -->
    <!--<mapper url="file:///C:/_KevinProjects/GameServerEngine/sample-game-server/target/query/UserDataMapper.xml"/>-->
  </mappers>
```

- Query : resources/query/UserDataMapper.xml

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

- DB connection setting : com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory

```
/**
 * The DB connection object used in game
 */
public class GameSqlSessionFactory {
    private static Logger logger = LoggerFactory.getLogger(GameSqlSessionFactory.class);

    private static SqlSessionFactory sqlSessionFactory;

    /** Reads the connection information specified in XML. */
    // Class reset block: Used in the complex reset of class variable.
    // Performed once when a class is initially loaded.
    static {
        // Read the XML path that specifies connection information
        try {
            // Set the path of the mybatis_config.xml file
            String mybatisConfigPath = System.getProperty("mybatisConfig"); // If parameter is passed, when server (is run -DmybatisConfig= Specify as an option)
            logger.info("mybatisConfigPath : {}", mybatisConfigPath);
            if (mybatisConfigPath != null) {
                logger.info("load to mybatisConfigPath : {}", mybatisConfigPath);
                InputStream inputStream = new FileInputStream(mybatisConfigPath);
                if (sqlSessionFactory == null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
                }
            } else {    // If no parameter is passed, obtain the setting from internal file
                Reader reader = Resources.getResourceAsReader("mybatis/mybatis-config.xml");
                logger.info("load to resource : mybatis/mybatis-config.xml");
                // Creates when there is no sqlSessionFactory.
                if (sqlSessionFactory == null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * Returns the session accessed to DATABASE through a database connection object.
     */
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

- If -DmybatisConfig= is not used while running, the log specified as internal storage preference file included while building is recorded while creating a session.

```
[2020-12-18 17:36:35,634] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : null
[2020-12-18 17:36:35,636] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:39] load to resource : mybatis-config.xml
```

- If -DmybatisConfig= is used while running, records the log that is set to the specified location information while creating a session.

```
[2020-12-18 17:43:37,871] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : .\src\main\resources\mybatis-config.xml
[2020-12-18 17:43:37,871] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:32] load to mybatisConfigPath : .\src\main\resources\mybatis-config.xml
```

- Use: com.nhn.gameanvil.sample.mybatis.UserDbHelperService

```
/**
 * Store user information in DB
 *
 * @param gameUserInfo Pass user information
 * @return Number of stored records
 * @throws TimeoutException
 * @throws SuspendExecution
 */
public int insertUser(GameUserInfo gameUserInfo) throws TimeoutException, SuspendExecution {    //  Runs Async in the form of Callable and returns the result.
    Integer resultCount = Async.callBlocking(GameConstants.DB_THREAD_POOL, new Callable<Integer>() {
        @Override
        public Integer call() throws Exception {
            SqlSession sqlSession = GameSqlSessionFactory.getSqlSession();
            try {
                UserDataMapper userDataMapper = sqlSession.getMapper(UserDataMapper.class);
                int resultCount = userDataMapper.insertUser(gameUserInfo.toDtoUser());
                if (resultCount == 1) { // Committed normally to DB if there is only one, fitting to the single item storage
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

- A DB schema that uses samples

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
  // Common Information.

  "common": {
    "ip": "127.0.0.1", // The IP shared among nodes. (Specifies the IP of a machine)
    "meetEndPoints": ["127.0.0.1:16000"], // Registers the common IP and communicatePort of the target node. (Can include the endpoint of the corresponding server, can include multiple items as a list)
    "ipcPort": 16000, // The port used when communicating with a different ipc node
    "publisherPort" : 13300, // The port for publish socket
    "debugMode": false //An option used to prevent various timeouts from being occurred, it must be false in real.
  },

  //-------------------------------------------------------------------------------------
  // Set LocationNode
  "location": {
    "clusterSize": 1, // How many machines (VM) is it consisted of?
    "replicaSize": 3, // The size of the clone group (number of masters + slaves)
    "shardFactor": 3  // A factor for sharding (refer to the comment below)
    // Total number of shards = clusterSize x replicaSize x shardFactor
    // Number of shards that can be run on a single machine (VM) = replicaSize x shardFactor
    // Total number of unique shards (number of master shards) = clusterSize x shardFactor
  },

  // Set match node
  "match": {
    "nodeCnt": 1,
    "useLocationDirect": true
  },

  //-------------------------------------------------------------------------------------
  // A node that manages the connection to the client.
  "gateway": {
    "nodeCnt": 4, // Number of nodes. (The node number starts from 0)
    "ip": "127.0.0.1", // The IP connected to the client.
    "dns": "", // The domain address connected to the client.
    "maintenance": false,
    "tcpNoDelay": false, // Used when setting Netty Bootstrap (field unused and default value false by default)
    "connectGroup": { // The type of connection.
      "TCP_SOCKET": {
        "port": 11200, // The port connected to the client.
        "idleClientTimeout": 240000 // The timeout after there is no data transfer (not used if it is 0).
        //        ,"secure": { // Sets security.
        //          "useSelf": true
        ////          ,"keyCertChainPath": "gameanvil.crt" // The path of certificate.
        ////          ,"privateKeyPath": "privatekey.pem" // Private key path.
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
  // The node that acts as game lobby (includes game rooms and users).
  "game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "TapTap",
      "channelIDs": ["","","","",""], // The channel IDs that are to be assigned to each node. (They do not have to be unique. Can be used in duplicate using the "" character string without distinguishing channels)
      "userTimeout": 5000 // User object removal timeout after disconnect.
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
  // A node that can manage other nodes using JMX or REST API (service pause, count entire users, etc.).
  "management": {
    "nodeCnt": 2,
    "restIp": "127.0.0.1",
    "restPort": 25150,
    "consoleProxyPort" : 18081, // admin web console port
    "logProxyPort" : 18082,     // admin log download port

    "db": {
      "user": "root",
      "password": "1234",
      "url": "jdbc:h2:mem:gameanvil_admin;DB_CLOSE_DELAY=-1"
    }
  }
}
```

## logback : resources/logback.xml

- logger can be distinguished by package name and specified. If separately specified, the specified level is applied to package name, and if not specified, the setting specified as root is applied.

```
    <logger name="com.nhn.gameanvil" level="INFO"/>
    <logger name="com.nhn.gameanvil.sample" level="DEBUG"/>

    <root>
        <level value="WARN"/>
        <appender-ref ref="ASYNC"/>
        <appender-ref ref="STDOUT"/>
    </root>
```
