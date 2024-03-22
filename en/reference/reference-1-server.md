## Game > GameAnvil > Reference Project > Server Samples

# GameAnvil API JavaDoc

[GameAnvil Server API - Java doc](https://gameplatform.toast.com/docs/api/)



# Server Download

[Sample Game Server](https://github.com/nhn/gameanvil.sample-game-server.git)



# Configuration Environment

* IDE: Intellij 2022.3.3

* JDK: openjdk version "11.0.16.1" 2022-08-12 LTS

* **GameAnvil 1.4.1**

* DB
  * Jasync-sql 1.2.3: basic use
    * MyBatis 3.5.3
  * Set the IP address according to each environment
  * MySQL 8.0.23

* Redis
  * Use the Lettuce API provided by GameAnvil
  * Set the IP address according to each environment
  
  

# Run Server

## InteliJ User Environment

Run projects cloned from Git repository as IntelliJ.

The default setting is **com.nn.gameanvil:1.4.1-jdk11** in Gradle Settings Dependencies, and the JDK11 version is being used.

The resources/GameAnvilConfig.json file has IP of 127.0.0.1.



### Setting up the running environment

The sample server is set to the default JDK11, so if InteliJ is set to a different version, you need to match the settings to JDK11 to avoid errors during build.

If InteliJ runs the GameAnvil library without matching the version of JDK, the following error occurs.

```java
Exception in thread "main" java.lang.UnsupportedClassVersionError: co/paralleluniverse/fibers/SuspendExecution has been compiled by a more recent version of the Java Runtime (class file version 54.0), this version of the Java Runtime only recognizes class file versions up to 52.0
```

For InteliJ JDK settings, please check as follows: 

Check JDK in the File > Project Structure > Project Settings > Project menu.

![reference-1-server_01](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_01.png) 

Check JDK in the IntelliJ IDEA > Settings > Build, Execution, Deployment > Buil Tools > Gradle > Gradle JVM menu.

![reference-1-server_02](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_02.png) 

If you changed the Gradle JDK, run Reload on the Gradle tab and reflect it on the project.

![reference-1-server_03](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_03.png) 

For build preferences set up, please follow the order as follows. Depending on the InteliJ version, the screen may vary slightly. (Screenshot is version 2023.12.)

### Check Redis Settings

If you connect to Redis without a password, modify the Redis connection information for class `com.nn.gameanvil.sample.common.GameConstants` to connect.

```java
    // Redis access information 
    public static final String REDIS_URL = "connection address"; 
    public static final int REDIS_PORT = 7500;
```



If password information is set, use the password settings in the annotated part of class `com.nn.gameanvil.sample.redis.RedisHelper` to connect to Redis.

```java
    /** 
     * Redis connection handling; need to call 1<sup>st</sup> time in advance to connect before use 
     * 
     * @param url  access url 
     * @param port access port 
     * @throws SuspendExecution This method means that fiber can be suspended
     */ 
    public void connect(String url, int port) throws SuspendExecution { 
        // Redis connection handling 
        RedisURI clusterURI = RedisURI.Builder.redis(url, port).build(); 
 
        // If a password is required, add password settings to create RedisURI
//        RedisURI clusterURI = RedisURI.Builder.redis(url, port).withPassword("password").build(); 
        this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI)); 
        this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient); 
 
        if (this.clusterConnection.isOpen()) { 
            logger.info("============= Connected to Redis using Lettuce ============="); 
        } 
 
        this.clusterAsyncCommands = clusterConnection.async(); 
    }
```

### Check DB Settings

The sample server uses Jasync-sql by default and uses queries as StoredProcedure.

When connect to Mybatis, If you change the value `com.nn.gameanvil.sample.common.USE_DB_JASYNC_SQL` to false, the DB will act as mybatis.

#### A DB schema that uses samples

This is table information used in the sample.

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



This is StoredProcedure information in use.

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

Using Jasync-sql, modify and connect DB connection information for class `com.nn.gameanvil.sample.common.GameConstants`.

```java
    // DB connection information 
    public static final String DB_USERNAME = "user name"; 
    public static final String DB_HOST = "host name"; 
    public static final int DB_PORT = 3306; 
    public static final String DB_PASSWORD = "password"; 
    public static final String DB_DATABASE = "database name"; 
    public static final int MAX_ACTIVE_CONNECTION = 30;
```



#### Mybatis

Mybatis connection connects by modifying the connection settings in resources/mybatid-config.xml.

```xml
  <Designate !-- MySQL connection information. --> 
  <properties> 
    <property name="hostname" value="host name" /> 
    <property name="portnumber" value="3306" /> 
    <property name="database" value="database name " /> 
    <property name="username" value="user name" /> 
    <property name="password" value="password" /> 
  </properties>
```


### Run Server

Run on InteliJ with runMain on the Gradle tab "gameanvil.sample-game-server [runMain]"

Run the server using the "SampleGameServer" configuration that you previously set up.

![reference-1-server_04](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_04.png)

![reference-1-server_05](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_05.png) 

The onReady log is displayed in every node if the server works normally.

Page http://127.0.0.1:18400/management/nodeInfoPage allows you to check the status of locally run nodes. If all nodes are READY, they are running normally.

![reference-1-server_06](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_06.png) 

For configuration, GameNode 4, GatewayNode 4, SupportNode2, IpcNode 1, ManagementNode 1, Locationnode 2, LocationNookupNode1,  MatchNode 1, GatewayNetworkNode 1, SupportNetwotNode1  total 18 nodes to be displayed.


### Check for error

If the server does not run properly, please check the settings again or check the error part of the log to contact us.

In the case of DB or Redis, the part applied to the sample server is used within the team, so you have to build and specify it yourself.

The sample server will not function properly if there is no DB or Redis configuration.


### Gradle build
Check GameAnvil version
```groovy
dependencies { 
    api 'com.nhn.gameanvil:gameanvil:1.4.1-jdk11' 
}
```


Set build.gradle 

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
 
// Create standalone jar. 
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
 
//Run GameAnvil server. 
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

#### Gradle jar Build
Build project through GameAnvilTutorial > Tasks > build > jar in Gradle tab

![reference-1-server_07](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_07.png)

When the build is successfully completed, a jar file built in the project folder build/libs is created.

![reference-1-server_08](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_08.png)

## Run Server with Command

To run the server with Command, use sample_game_server-1.4.1.jar built with Gradle.

![reference-1-server_09](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_09.png)


```
java -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar sample_game_server-1.4.1.jar
```

- By default, if no other option is specified at running time, the specified environment file is applied when building.
- GameAnvilConfig.json specifies the path and filename with -Dconfig.file option.
- logback.xml specifies the path and filename with -Dlogback.configurationFile option.
- Config setting for mybatis specifies -DmybatisConfig option.
  - Mybatis is not part of a feature supported by GameAnvil, which is presented as an example. 
  - Refer to com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory

When enabled, onReady log is displayed as if enabled by IntelliJ, and when all nodes are ready on page http://127.0.0.1:18400/management/nodeInfoPage, it is normally done.

# Look at sample-game-server servers

This project was created to link with GameAnvil sample clients that made it a reference for game development.

- There may be many errors or bugs about the game itself, as its purpose is for feature usability.
- For more information, please contact [Customer Center](https://www.toast.com/kr/support/inquiry).



## Project Configuration

### gateway

- Verify
  - We receive userId, token from GAMEBASE to verify token.

### game

#### multi
- TapBird
  - 1-4 players can play with room match. All the players receive their score until everyone leaves the room.
  - If there is no room, create a room and join
    - Register the user and score information to the room
    - Update the count of users joined the room and the room ID in the room information.
  - If there is a room, join the existing room
    - Register the user and score information to the room
    - Update the current count of users in the room information.
  - When leaving the room,
    - Delete users and score information from the room.
    - Update the number of users in the room information.
  - Process the user score increase packet
    - Update the user information in the room with scores delivered in a non-response format
    - Create a user score list and passes the score to every user in the room.
- Snake
  - As a user match, 2 players enter the room at the same time and play the game.
  - When 2 players are matched, one creates and joins a room and the other joins the created room.
    - Register the user information and score information to the room after the location of the first user is specified.
    - Register the user information and score information to the location specification and room when joining the room for the second time.
    - Send game information to the users when the count of users reaches 2.
    - Start the food creation timer in the room.
  - Create food every second until the number of foods reaches the maximum number and transfer it to users in the form of data that does not require a response
  - When leaving the room,
    - Delete user information and score information from the room.
    - Stop the timer.
    - Kick every user out of the room.
  - Processing food discard packets
    - Receive the food information to be deleted and delete the food information from the room.
    - Pass the deleted food to the opponent.
  - Process user move packet
    - Receive the information when the user moves, stores the user information in the room, and transfers the moved user information to the opponent.

#### single

- Play the game by creating a room alone
  - Set packets delivered at the start of the game to the required data room in a single room
- Tap Message Packet Processing
  - Record the score in your current room when receiving a message that does not require a response
- When leaving the room,
  - if the score stored in the room is greater than the high score of the user object logged in,
    - Update user highs in the DB
    - Update the ranking information of Redis if successful
  - The response to the score of the play in the room by processing game end packet

#### user

  - Login
    - Look up users in DB using user identifier
      - Use the information retrieved from DB if there is a user
      - Register new information to DB if there is no user
    - Record the information of logged in user to Redis
    - User information response
  - Nickname change request
    - Change user nickname in DB
    - The nickname of the user object that is currently logged in, if successful
    - User information response
  - Deck replace request
    - Check the currency deduction for replacing deck
      - In the sample, check only the user object the user had at the time of login for deduction
    - When it is normally deducted
      - Remove the current user's deck from the list and randomize a different deck
      - Save the deck changed from DB
      - Change the deck of the user object that is currently logged in, if successful
      - Response to the changed deck and currency balance
  - Request for single game ranking information
    - Retrieve a list of single game rankings from Redis with a passed in scope
    - Create a rankings list by setting a rankings list from the user information of the Redis stored whenever the user logs in
    - Respond to the created rankings list

### support 

- This service is used to retrieve game session information before connecting to a game server.

- Processing launch information request packets

  - Request in http://127.0.0.1:18600/launching?platform=Editor&appStore=GOOGLE&appVersion=1.2.0&deviceId=4D34C127-9C56-5BAB-A3C2-D8F18C0B7B6E format.

  - Parse and check the delivered data and return the IP, PORT of Gateway Node server.

### redis

- Link the lettuce provided by GameAnvil
- Create it in every Redis connection setting Node and connect them in GameNode's onInit
  - You must not use a single Redis connection on all nodes because it is created as a single tone instance.
- The connection shutdown of Redis needs to be processed in GameNode's onShutdown.
- Example features
  - Search the user data list with hmget
  - Store single-player game rankings with ZADD
  - Search ranking information with zreverangeWithScores

### db 

#### jasync-sql

* A class that can handle asynchronous SQL.
* Instances must be created and used, one per node, and closed when the server shuts down.
* In the sample, the query is written as a StoredProcedure.

#### mybatis

- resources/mybatis contains only DB configuration information.
- A mapper.xml file exists inside the com.nhn.gameanvil.sample.mybatis.mappers package.
- Example features
  - INSERT user information
  - SELECT user information with UUID
  - UPDATE deck, nickname, and high score

### protocol 

- Use google protobuf 3.0.
- Write the client and every packet used with google protobuf.
- The .proto file is developed to be shared with the client. Use it after converting it into a .java file that is used by server as in build.bat.
- Example of protocol usage
  - Authentication.proto: Authentication, login
  
  - GameMulti.proto: Multiplayer game
  
  - GameSingle.proto: Single-player game
  
  - Result.proto: Response code
  
  - User.proto: User
  
  - If plug-in is installed, right click on the following build.bat file to directly convert it using the following command in intelliJ.
  
    ![reference-1-server_10](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_10.png) 
  
    ![reference-1-server_11](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_11.png) 

## Server Operation Content

### Server Operation Settings

Set up and run GameAnvilServer for the main class. When set up, register the protocol with the client in the same order and create a thread pool used by the server.

```java
        GameAnvilServer gameAnvilServer = GameAnvilServer.getInstance(); 
 
        // The protocol definition to be sent with the client - the order has to be the same as the client.
        gameAnvilServer.addProtoBufClass(Authentication.getDescriptor()); 
        gameAnvilServer.addProtoBufClass(GameMulti.getDescriptor()); 
        gameAnvilServer.addProtoBufClass(GameSingle.getDescriptor()); 
        gameAnvilServer.addProtoBufClass(Result.getDescriptor()); 
        gameAnvilServer.addProtoBufClass(User.getDescriptor()); 
 
        // Specify DB thread pool used by the game
        gameAnvilServer.createExecutorService(GameConstants.DB_THREAD_POOL, 100); 
        // Specify Redis thread pool used by the game
        gameAnvilServer.createExecutorService(GameConstants.REDIS_THREAD_POOL, 100); 
 
        // Specify scan package for annotation class registration process
        gameAnvilServer.addPackageToScan("com.nhn.gameanvil.sample"); 
 
        // server running 
        gameAnvilServer.run();
```



GatewayNode, GameNode, SupportNode, MatchMaker, Room have to be classified and named with @annotaion for GameAnvil use. Classes that are declared when the server is running are automatically registered class by the engine.

Declare to the connection class that inherited and implemented the BaseConnection.

```java
Connection
```

Declare to the gateway node that inherited and implemented the BaseGatewayNode.

```
@GatewayNode()
```

Declare to the session that you inherited and implemented the Base Session.

```
@Session()
```

Specify the service name of the game node set in GameAnvilConfig.json for the game node that inherited and implemented the BaseGameNode.

```
@ServiceName(GameConstants.GAME_NAME)
```

Specify the service name and user type of the game to the game user who inherited and implemented the BaseUser. The user type is a type created by connecting to the client.

```
@ServiceName(GameConstants.GAME_NAME)
@UserType(GameConstants.GAME_USER_TYPE)
```

Specify the game service name and room type for the room that inherited and implemented BaseRoom. The room type specifies the room type to be connected by the client.

```java
// Single game
@ServiceName(GameConstants.GAME_NAME) 
@RoomType(GameConstants.GAME_ROOM_TYPE_SINGLE) 
 
// Room match game 
@ServiceName(GameConstants.GAME_NAME) 
@RoomType(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH) 
 
// User match game 
@ServiceName(GameConstants.GAME_NAME) 
@RoomType(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH)
```

For matchmakers that inherit and implement BaseUserMatchMaker and BaseRoomMatchMaker, specify the same service name and room type as the corresponding room.

Specify the service name of the support node set in GameAnvilConfig.json for the support node that inherited and implemented the BaseSupportNode.

```
@ServiceName(GameConstants.SUPPORT_NAME_LAUNCHING)
```



### API processed by GameAnvil

#### GameSession - session

- com.nhn.gameanvil.sample.gateway.GameSession
  - onAuthenticate() : Implement verification of client authentication requests

#### GameUser - user

- com.nhn.gameanvil.sample.game.user.GameUser
  - onLogin: Implement the content about client login request
  - onMatchRoom: Implement the process for client multi-room match request
  - onMatchUser: Implement the process for client user match request
  - onTransferIn /onTransferOut: Transfer and recover user object data when the user moves the server

#### SingleGameRoom Single-player room

- com.nhn.gameanvil.sample.game.single.SingleGameRoom
  - onCreateRoom: Process the content of client single-player room, create a room and join it.
  - onLeaveRoom: Implement the process for leaving a single-player room

#### UnlimitedTapRoom Multiplayer room match, up to 4 players

- com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoom
  - onInit(): handled when a room is created, initializing the data used by the room
  - onCreateRoom: Process the part where the room for initial room matching
  - onJoinRoom: Process users after the second
  - onLeaveRoom: Process when leaving room
  - onTransferIn / onTransferOut: Transfer and recover the data in the room when moving the room to a different server

#### UnlimitedTapRoomMatchMaker: Process room match logic

- com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoomMatchMaker

#### SnakeRoom User match, 2 players

- com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom
  - onInit(): handled when a room is created, initializing the data used by the room
  - onCreateRoom: Process the part where the room for initial room matching
  - onJoinRoom: Process users after the second
  - onPostLeaveRoom: Process after leaving the room, as it is a room for two player, if one player leaves the room, remove the timer and have the other player leave the room.
  - onTransferIn / onTransferOut: Transfer and recover the data in the room when moving the room to a different server
  - onTimer: Regularly create food from server and pass it to the users in the room

#### SnakeRoomMatchMaker: Process the logic for matching 2 players

- com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoomMatchMaker
  - onMatch(): Handles room match logic

### Register packet

They are packets defined and processed in the game content and register to send and receive packet to and from clients. The processing class has to implement a packet handler interface suitable for the registered type.

The packet the user processes while logged in: com.nhn.gameanvil.sample.game.user.GameUser

```java
static private PacketDispatcher packetDispatcher = new PacketDispatcher(); 
 
static { 
    packetDispatcher.registerMsg(User.ChangeNicknameReq.getDescriptor(), CmdChangeNicknameReq.class);           // nickname change protocol 
    packetDispatcher.registerMsg(User.ShuffleDeckReq.getDescriptor(), CmdShuffleDeckReq.class);                 // deck shuffle protocol 
    packetDispatcher.registerMsg(GameSingle.ScoreRankingReq.getDescriptor(), CmdSingleScoreRankingReq.class);   // single score ranking 
} 
// For processing class, it has to be made by implementing  implements IPacketHandler<GameUser>. 
```

For packets requested by the client as a request, as the client is waiting for a response, the server have to process the response to gameUser.reply() through the user object that was processed and delivered.



A packet processed when in the room: com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom

```java
private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher(); 
 
static { 
    dispatcher.registerMsg(GameMulti.SnakeUserMsg.getDescriptor(), CmdSnakeUserMsg.class);  // user location information 
    dispatcher.registerMsg(GameMulti.SnakeFoodMsg.getDescriptor(), CmdSnakeRemoveFoodMsg.class);  // food deletion information process 
} 
// For class to process,  it has to be made by implementing  implements IRoomPacketHandler<SnakeRoom, GameUser>. 
```

The packet passed from the server to the client transfers gameUser.send without waiting.



rest packet: com.nhn.gameanvil.sample.support.LaunchingSupport

```java
private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher(); 
 
static { 
    // launching 
    restMsgHandler.registerMsg("/launching", RestObject.GET, CmdLaunching.class); 
} 
// The class to be handled should be created by implementing implements IRestPacketHandler.
```

For rest requests, a response message is forwarded to the received restObject.writeString().


### Redis

com.nhn.gameanvil.sample.redis.RedisHelper

Connection process 

```java
private RedisClusterClient clusterClient; 
private StatefulRedisClusterConnection<String, String> clusterConnection; 
private RedisAdvancedClusterAsyncCommands<String, String> clusterAsyncCommands; 
/** 
 * Redis connection,  have to call 1<sup>st</sup> time in advance to connect before use
 * 
 * @param url  access url 
 * @param port access port 
 * @throws SuspendExecution 
 */ 
public void connect(String url, int port) throws SuspendExecution {    // Redis connection process 
    RedisURI clusterURI = RedisURI.Builder.redis(url, port).build(); 
 
    // If a password is required, add password settings to create RedisURI.
//  RedisURI clusterURI = RedisURI.Builder.redis(url, port).withPassword("password").build(); 
    this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI)); 
    this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient); 
    this.clusterAsyncCommands = clusterConnection.async(); 
}
```

Shutdown handling

```java
/** 
 * Shutdown server must be called before it goes down. 
 */ 
public void shutdown() { 
    clusterConnection.close(); 
    clusterClient.shutdown(); 
}
```

In use

```java
/** 
 * Save to user data Redis 
 * 
 * @param gameUserInfo user information 
 * @return whether successfully save or not 
 * @throws SuspendExecution 
 */ 
public boolean setUserData(GameUserInfo gameUserInfo) throws SuspendExecution { 
    String value = GameAnvilUtil.Gson().toJson(gameUserInfo); 
 
    boolean isSuccess = false; 
    try { 
        Lettuce.awaitFuture(clusterAsyncCommands.hset(REDIS_USER_DATA_KEY, gameUserInfo.getUuid(), value)); // This return value is true only when it is initially set,  False response when updating the value
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

Connection handling

```java
    // JAsyncSQL connection
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

shutdown handling 
```java
    // JAsyncSQL close 
    public void close() { 
        jAsyncSql.disconnect(); 
    }
```

In use
```java
    /** 
     * Save the user's highest score
     * 
     * @param uuid      User Unique Identifier
     * @param highScore Highest score to modify
     * @return Returns the modified number of records
     * @throws TimeoutException means that a timeout may occur for the corresponding call     
* @throws SuspendExecution This method means that fiber can be suspended
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

DB connection information setting: resources/maybatis-config.xml

```xml
<Specify !-- MySQL connection information . --> 
<properties> 
  <property name="hostname" value="host name" /> 
  <property name="portnumber" value="3306" /> 
  <property name="database" value="database name" /> 
  <property name="username" value="user name" /> 
  <property name="password" value="password" /> 
  <property name="poolPingQuery" value="select 1"/> 
  <property name="poolPingEnabled" value="true"/> 
  <property name="poolPingConnectionsNotUsedFor" value="3600000"/> 
</properties>
```

Register a query to use - To use external xml, refer to the comment section below.

```xml
  <mappers> 
    <!—Map defined SQL syntax; by default, when using mapper.xml in resources --> 
    <mapper resource="query/UserDataMapper.xml"/> 
    <!-- Use full path specification when specifying an externally specified mapper.xml file. --> 
    <!--<mapper url="file:///C:/_KevinProjects/GameServerEngine/sample-game-server/target/query/UserDataMapper.xml"/>--> 
  </mappers>
```

Query: resources/query/UserDataMapper.xml

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

DB connection setting: com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory

```java
/** 
 * DB connection objects used in the game
 */ 
public class GameSqlSessionFactory { 
    private static Logger logger = LoggerFactory.getLogger(GameSqlSessionFactory.class); 
 
    private static SqlSessionFactory sqlSessionFactory; 
 
    /** Read the connection information specified in XML. */ 
    // Class Initialization Block: Used for complex initialization of class variables.
    // It is performed only once when class is first loaded.
    static { 
        // Read the path in XML that specifies the connection information
        try { 
            // specify path of mybatis_config.xml file 
            String mybatisConfigPath = System.getProperty("mybatisConfig"); // Server if parameters are delivered (When running -DmybatisConfig= Specify as option) 
            logger.info("mybatisConfigPath : {}", mybatisConfigPath); 
            if (mybatisConfigPath != null) { 
                logger.info("load to mybatisConfigPath : {}", mybatisConfigPath); 
                InputStream inputStream = new FileInputStream(mybatisConfigPath); 
                if (sqlSessionFactory == null) { 
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream); 
                } 
            } else {    // If there is no parameter transfer, the settings are obtained from the internal file.
                Reader reader = Resources.getResourceAsReader("mybatis/mybatis-config.xml"); 
                logger.info("load to resource : mybatis/mybatis-config.xml"); 
                // Creates if sqlSessionFactory not exist. 
                if (sqlSessionFactory == null) { 
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader); 
                } 
            } 
        } catch (IOException e) { 
            e.printStackTrace(); 
        } 
    } 
 
    /** 
     * Returns a session connected to DATABASE through a database connection object. 
     */ 
    public static SqlSession getSqlSession() { 
        return sqlSessionFactory.openSession(); 
    } 
}

```

If -DmybatisConfig= is not used while running, the log specified as internal storage preference file included while building is recorded while creating a session.

```java
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : null 
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:39] load to resource : mybatis-config.xml
```

If -DmybatisConfig= is used while running, records the log that is set to the specified location information while creating a session.

```java
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : .\src\main\resources\mybatis-config.xml 
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:32] load to mybatisConfigPath : .\src\main\resources\mybatis-config.xml
```

use : com.nhn.gameanvil.sample.db.mybatis.UserDbHelperService

```java
/** 
 * Save to user information DB 
 * 
 * @param gameUserInfo Forward user information 
 * @return number of records saved. 
 * @throws TimeoutException 
 * @throws SuspendExecution 
 */ 
public int insertUser(GameUserInfo gameUserInfo) throws TimeoutException, SuspendExecution {    //   Run Async in Callable format, and return result. 
    Integer resultCount = Async.callBlocking(GameConstants.DB_THREAD_POOL, new Callable<Integer>() { 
        @Override 
        public Integer call() throws Exception { 
            SqlSession sqlSession = GameSqlSessionFactory.getSqlSession(); 
            try { 
                UserDataMapper userDataMapper = sqlSession.getMapper(UserDataMapper.class); 
                int resultCount = userDataMapper.insertUser(gameUserInfo.toDtoUser()); 
                if (resultCount == 1) { // Since it's a single storage, it's normal to have one DB commit 
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

## Server Settings
### GameAnvilConfig.json

```json
{ 
  //------------------------------------------------------------------------------------- 
  // common information. 
 
  "common": { 
    "ip": "127.0.0.1", // IP commonly used to each node. (Specify the IP of the machine)

    "meetEndPoints": ["127.0.0.1:18000"], // Register a common IP and communicate port on the destination node. (Can include corresponding server endpoints, multiple lists are available) 
    "debugMode": false // An option that prevents various timeout from occurring during debugging, it must be false in real. 
 }, 
 
  //------------------------------------------------------------------------------------- 

  // LocationNode setting 
  "location": { 
    "clusterSize": 1, // How many machines (VMs) does it consist of in total?
    "replicaSize": 1, // Size of replication group ( the count of master + slave) 
    "shardFactor": 2  // Factor for  sharding (Refer to below footnote) 
    // Number of total shard = clusterSize x replicaSize x shardFactor 
    // Number of shards to run on one machine (VM) = replicaSize x shardFactor 
    // Total number of unique shards (count of master shards)) = clusterSize x shardFactor 
  }, 
 
  // match node setting 
  "match": { 
    "nodeCnt": 1 
  }, 
 
  //------------------------------------------------------------------------------------- 
  // A node that manages the connection to the client.
  "gateway": { 
    "nodeCnt": 4, // count of node (node number is allocated from 0). 
    "ip": "127.0.0.1", //  IP connected to client. 
    "dns": "", // Domain address connected to client. 
    "connectGroup": { // type of connection. 
      "TCP_SOCKET": { 
        "port": 18200, // port that connects to the client. 
        "idleClientTimeout": 240000 // Timeout after no data transmission and reception (not used if 0).
      }, 
      "WEB_SOCKET": { 
        "port": 18300, 
        "idleClientTimeout": 0 
      } 
    } 
  }, 
 
  //------------------------------------------------------------------------------------- 
  // Nodes that act as game lobby (including game rooms, users).
  "game": [ 
    { 
      "nodeCnt": 4, 
      "serviceId": 1, 
      "serviceName": "TapTap", 
      "channelIDs": ["","","","",""], // Channel ID to be given per node (no need to be unique."" string and can be used in duplicate without channel separation).
      "userTimeout": 5000 // Timeout for removal of user objects after disconnect.
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

logger can be distinguished by package name and specified. If separately specified, the specified level is applied to package name, and if not specified, the setting specified as root is applied.

```
    <logger name="com.nhn.gameanvil" level="INFO"/> 
    <logger name="com.nhn.gameanvil.sample" level="DEBUG"/> 
 
    <root> 
        <level value="WARN"/> 
        <appender-ref ref="ASYNC"/> 
        <appender-ref ref="STDOUT"/> 
    </root>
```