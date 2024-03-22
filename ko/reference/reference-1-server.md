## Game > GameAnvil > 레퍼런스 프로젝트 > 서버 샘플

# GameAnvil API JavaDoc

[GameAnvil Server API - Java doc](https://gameplatform.toast.com/docs/api/)



# 서버 다운로드

[Sample Game Server](https://github.com/nhn/gameanvil.sample-game-server.git)



# 구성 환경

* IDE: Intellij 2022.3.3

* JDK: openjdk version "11.0.16.1" 2022-08-12 LTS

* **GameAnvil 1.4.1**

* DB
  * Jasync-sql 1.2.3: 기본 사용
    * MyBatis 3.5.3
  * 각자의 환경에 맞게 IP 주소를 설정
  * MySQL 8.0.23

* Redis
  * GameAnvil에서 제공하는 Lettuce API를 사용
  * 각자의 환경에 맞게 IP주소를 설정
  
  

# 서버 구동

## IntelliJ 사용 환경

Git 저장소에서 clone한 프로젝트를 IntelliJ로 실행합니다.

기본 설정은 Gradle 설정 Dependencies에 **com.nhn.gameanvil:gameanvil:1.4.1-jdk11**로 JDK11 버전이 사용되고 있습니다.

resources/GameAnvilConfig.json 파일에 IP가 127.0.0.1로 되어 있습니다.



### 실행 환경 설정

샘플 서버는 기본 JDK11로 설정되어 있기 때문에 IntelliJ가 다른 버전으로 설정되어 있는 경우 설정을 JDK11로 맞추어야 빌드 시 오류가 발생하지 않습니다.

만약 IntelliJ에서 GameAnvil 라이브러리와 JDK의 버전을 맞추지 않고 실행하면 다음과 같은 오류가 발생합니다.

```java
Exception in thread "main" java.lang.UnsupportedClassVersionError: co/paralleluniverse/fibers/SuspendExecution has been compiled by a more recent version of the Java Runtime (class file version 54.0), this version of the Java Runtime only recognizes class file versions up to 52.0
```

IntelliJ JDK 설정은 다음을 확인하십시오.

File > Project Structure > Project Settings > Project 메뉴에서 JDK를 확인합니다.

![reference-1-server_01](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_01.png) 

IntelliJ IDEA > Settings > Build, Execution, Deployment > Buil Tools > Gradle > Gradle JVM 메뉴에서 JDK를 확인 합니다.

![reference-1-server_02](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_02.png) 

Gradle JDK를 변경했다면 Gradle 탭의 Reload를 실행해 프로젝트에 반영합니다.

![reference-1-server_03](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_03.png) 

빌드 환경 설정은 아래의 내용을 순서대로 설정합니다. IntelliJ 버전에 따라 화면은 조금 다를 수 있습니다. (스크린샷은 2023.12 버전입니다.)

### Redis 설정 확인

패스워드 없이 Redis에 연결한다면 `com.nhn.gameanvil.sample.common.GameConstants` 클래스의 Redis 접속 정보를 수정해 연결합니다.

```java
    // Redis 접속 정보
    public static final String REDIS_URL = "연결 주소";
    public static final int REDIS_PORT = 7500;
```



패스워드 정보가 설정된 것이라면 Redis에 연결할 때 `com.nhn.gameanvil.sample.redis.RedisHelper` 클래스의 주석된 부분의 패스워드 설정을 사용해 접속합니다.

```java
    /**
     * Redis 연결 처리. 사용 전 최초 1회 호출해 연결 필요
     *
     * @param url  접속 url
     * @param port 접속 port
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
     */
    public void connect(String url, int port) throws SuspendExecution {
        // Redis 연결 처리
        RedisURI clusterURI = RedisURI.Builder.redis(url, port).build();

        // 패스워드가 필요한 경우에는 패스워드 설정을 추가해서 RedisURI를 생성 
//        RedisURI clusterURI = RedisURI.Builder.redis(url, port).withPassword("password").build();
        this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI));
        this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient);

        if (this.clusterConnection.isOpen()) {
            logger.info("============= Connected to Redis using Lettuce =============");
        }

        this.clusterAsyncCommands = clusterConnection.async();
    }
```

### DB 설정 확인

샘플 서버에서는 기본적으로 Jasync-sql을 기본으로 사용하며, StoredProcedure로 쿼리를 사용합니다.

Mybatis 연결 시 `com.nhn.gameanvil.sample.common.GameConstants.USE_DB_JASYNC_SQL` 값을 false로 바꾸면 DB가 mybatis로 동작합니다.

#### 샘플 사용 DB 스키마

샘플에서 사용하고 있는 테이블 정보입니다.

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



사용하고 있는 StoredProcedure 정보입니다.

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

Jasync-sql을 사용하면 `com.nhn.gameanvil.sample.common.GameConstants` 클래스의  DB 접속 정보를 수정해서 연결합니다.

```java
    // DB 접속 정보
    public static final String DB_USERNAME = "유저명";
    public static final String DB_HOST = "호스트명";
    public static final int DB_PORT = 3306;
    public static final String DB_PASSWORD = "패스워드";
    public static final String DB_DATABASE = "데이터베이스명";
    public static final int MAX_ACTIVE_CONNECTION = 30;
```



#### Mybatis

Mybatis 연결은 resources/mybatid-config.xml의 접속 설정을 수정해서 연결합니다.

```xml
  <!-- MySQL 접속 정보를 지정한다. -->
  <properties>
    <property name="hostname" value="호스트명" />
    <property name="portnumber" value="3306" />
    <property name="database" value="데이터베이스명" />
    <property name="username" value="유저명" />
    <property name="password" value="패스워드" />
  </properties>
```


### 서버 실행

Gradle 탭의 runMain 실행으로 IntelliJ에서 실행 "gameanvil.sample-game-server [runMain]"

앞서 설정해 두었던 "SampleGameServer" 구성을 이용하여 서버를 실행합니다.

![reference-1-server_04](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_04.png)

![reference-1-server_05](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_05.png) 

서버가 정상적으로 구동되면 아래와 같이 모든 노드에 대해 onReady 로그가 출력됩니다.

http://127.0.0.1:18400/management/nodeInfoPage 페이지를 통해서 로컬에서 실행된 노드의 상태를 확인할 수 있습니다. 모든 노드가 READY가 되면 정상 실행된 것입니다.

![reference-1-server_06](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_06.png) 

구성은 GameNode 4, GatewayNode 4, SupportNode2, IpcNode 1, ManagementNode 1, Locationnode 2, LocationNookupNode1,  MatchNode 1, GatewayNetworkNode 1, SupportNetwotNode1 총 18개 노드가 표시됩니다.


### 오류 확인

정상적으로 서버가 실행되지 않을 경우 설정을 다시 확인하거나 log의 오류 부분을 확인하여 문의하십시오.

DB나 Redis의 경우 샘플 서버에 적용된 부분은 팀 내부에서 사용하는 부분이므로 직접 구축해 지정해야 합니다.

DB나 Redis의 설정이 없다면 샘플 서버가 정상적으로 동작하지 않습니다.


### Gradle 빌드
GameAnvil 버전 확인
```groovy
dependencies {
    api 'com.nhn.gameanvil:gameanvil:1.4.1-jdk11'
}
```


build.gradle 설정

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

// standalone jar을 생성합니다.
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

// GameAnvil 서버를 실행합니다.
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

#### Gradle jar 빌드
Gradle 탭의 GameAnvilTutorial > Tasks > build > jar 통해 프로젝트를 빌드합니다.

![reference-1-server_07](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_07.png)

정상적으로 빌드가 완료되면 프로젝트 폴더 build/libs에 빌드된 jar 파일이 생성됩니다.

![reference-1-server_08](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_08.png)

## Command로 서버 실행

Command로 서버를 구동하려면 Gradle로 빌드된 sample_game_server-1.4.1.jar를 사용합니다.

![reference-1-server_09](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_09.png)


```
java -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar sample_game_server-1.4.1.jar
```

- 기본으로 실행 시에 별도의 옵션이 지정되지 않으면 빌드할 때 지정되어 있는 환경 파일이 적용됩니다.
- GameAnvilConfig.json은 -Dconfig.file 옵션으로 경로와 파일 이름을 지정합니다.
- logback.xml은 -Dlogback.configurationFile 옵션으로 경로와 파일 이름을 지정합니다.
- mybatis용 config 설정은 -DmybatisConfig 옵션을 지정합니다.
  - mybatis는 GameAnvil이 지원하는 기능의 일부가 아니며, 여기에서는 예시로 제시하였습니다.
  - com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory 참고

실행하면 IntelliJ에서 실행한 것과 같이 onReady 로그가 표시되고, http://127.0.0.1:18400/management/nodeInfoPage 페이지에 모든 노드가 READY 되면 정상 기동된 것입니다.

# sample-game-server 서버 살펴보기

게임 개발에 참고할 수 있게 만든 GameAnvil 샘플 클라이언트와 연동하기 위해  제작된 프로젝트입니다.

- 기능 사용성에 목적을 두어서 게임 자체에 대한 오류나 버그가 많을 수 있습니다.
- 더 자세한 내용은 [고객센터](https://www.toast.com/kr/support/inquiry) 로 문의하십시오.



## 프로젝트 구성

### gateway

- 인증
  - Gamebase의 userId, token을 전달 받아 token을 검증합니다.

### game

#### multi
- TapBird
  - 룸 매치로 1\~4인까지 플레이가 가능합니다. 룸에서 모두 나갈 때까지 플레이하는 점수를 모든 사용자가 받습니다.
  - 룸이 하나도 없다면 룸을 만들어서 입장
    - 룸에 유저와 점수 정보를 등록
    - 룸 정보에 룸 아이디와 현재 입장한 유저 수를 갱신합니다.
  - 룸이 있다면 기존에 있는 룸에 입장
    - 룸에 유저와 점수 정보를 등록
    - 룸 정보에 현재 유저 수를 갱신합니다.
  - 룸에서 나갈 때
    - 룸에 유저와 점수 정보를 삭제합니다.
    - 룸 정보의 유저 수를 갱신합니다.
  - 유저 점수 증가 패킷 처리
    - 응답이 필요 없는 형태로 전달 받은 점수를 룸의 유저 정보를 갱신
    - 유저 점수 리스트를 만들어서 현재 룸 안에 있는 모든 유저에게 점수를 전달합니다.
- Snake
  - 유저 매치로 2명이 동시에 룸에 입장해서 게임 플레이
  - 둘이 매치되었을 때 한 명은 룸을 만들어 들어가고 한 명은 생성된 룸에 들어갑니다.
    - 첫 번째 유저 위치 지정 후 룸에 유저 정보와 점수 정보를 등록합니다.
    - 두 번째 입장할 때 위치 지정과 룸에 유저 정보와 점수 정보를 등록합니다.
    - 유저가 2명이 되면 유저에게 게임 정보를 전송합니다.
    - 룸에서 food 생성 타이머를 시작합니다.
  - 룸에서 food 개수가 max가 될 때까지 1초에 한 번씩 food를 생성해서 유저들에게 응답이 필요 없는 데이터로 전송
  - 룸에서 나갈 때
    - 룸에 유저 정보와 점수 정보를 삭제합니다.
    - 타이머를 정지합니다.
    - 모든 유저를 방에서 내보냅니다.
  - food 삭제 패킷 처리
    - 삭제할 food 정보를 받아서 룸에 있는 food 정보를 삭제합니다.
    - 상대에게 삭제된 food를 전달합니다.
  - user 이동 패킷 처리
    - 유저가 이동할 때의 정보를 전달 받아 룸에 유저 정보를 저장하고 상대방에게 이동된 유저 정보를 전달합니다.

#### single

- 혼자 방을 만들어 게임
  - 게임 시작할 때 전달 받은 패킷에 싱글 룸에서 필요한 데이터 룸에 설정
- 탭 메시지 패킷 처리
  - 응답이 필요 없는 메시지로 전달 받으면 현재 룸에 스코어 점수 기록
- 방에서 나올 때
  - 로그인된 유저 객체의 최고 점수보다 방 안에 저장된 점수가 크면
    - DB에 유저 최고 기록 업데이트
    - 성공하면 Redis의 랭킹 정보 업데이트
  - 게임 종료 패킷 처리해서 현재 방에서 플레이한 점수 응답

#### user

  - 로그인
    - DB에서 유저 식별자를 가지고 유저 조회
      - 유저가 있는 경우 DB에서 가져온 정보 사용
      - 유저가 없는 경우 DB에 신규 정보 기록
    - 로그인된 유저 정보 Redis에 기록
    - 유저 정보 응답
  - 닉네임 변경 요청
    - DB에 유저 닉네임 변경
    - 성공하면 현재 로그인된 유저 객체의 닉네임 변경
    - 유저 정보 응답
  - 덱 교체 요청
    - 덱 교체를 위한 재화 차감 확인
      - 샘플에서는 현재 로그인했을 때 가지고 있는 유저 객체에서만 차감 확인
    - 정상 차감될 경우
      - 현재 유저가 가지고 있는 덱은 리스트에서 제거 후 다른 덱을 랜덤으로 설정
      - DB에 변경된 덱 저장
      - 성공하면 현재 로그인된 유저 객체의 덱 변경
      - 바뀐 덱, 재화 잔액 응답
  - 싱글 게임 랭킹 정보 요청
    - 전달 받은 범위로 Redis에서 싱글 게임 랭킹 리스트 검색
    - 로그인할 때마다 저장해 둔 Redis의 유저 정보에서 랭킹 리스트의 닉네임 설정해서 랭킹 목록 생성
    - 생성된 랭킹 리스트 응답

### support 

- 게임 서버 접속 전 게임 세션 정보를 불러오기 위한 서비스입니다.

- 런칭 정보 요청 패킷 처리

  - http://127.0.0.1:18600/launching?platform=Editor&appStore=GOOGLE&appVersion=1.2.0&deviceId=4D34C127-9C56-5BAB-A3C2-D8F18C0B7B6E 형식으로 요청을 합니다.

  - 전달 받은 데이터를 파싱하고 확인하고 GatewayNode 서버의 IP, PORT를 반환해 줍니다.

### redis

- GameAnvil에서 제공하는 lettuce 연동
- GameNode의 onInit()에서 Redis 연결 설정 Node마다 만들어서 연결
  - 싱글톤 인스턴스로 생성하였으므로 모든 노드에서 하나의 Redis 연결을 사용하면 안 됩니다.
- GameNode의 onShutdown()에서 Redis의 연결 shutdown 처리를 해야 합니다.
- 예제 기능
  - hmget으로 유저 데이터 리스트 검색
  - zadd로 싱글 게임 랭킹 저장
  - zreverangeWithScores로 랭킹 정보 검색

### db 

#### jasync-sql

* 비동기 SQL을 처리할 수 있는 클래스입니다.
* 인스턴스는 노드당 하나씩 생성하여 사용하고, 서버가 종료될 때 닫아 주어야 합니다.
* 샘플에서는 StoredProcedure로 쿼리가 작성되어 있습니다.

#### mybatis

- resources/mybatis에 DB 설정 정보만 있습니다.
- com.nhn.gameanvil.sample.mybatis.mappers 패키지 안에 mapper.xml 파일이 존재합니다.
- 예제 기능
  - 유저 정보 INSERT
  - uuid로 유저 정보 SELECT
  - 덱, 닉네임, 최고 점수 UPDATE

### protocol 

- google protobuf 3.0을 사용합니다.
- 클라이언트와 사용하는 모든 패킷은 google protobuf로 작성합니다.
- .proto 파일은 클라이언트와 공용으로 제작하며 build.bat에 있는 것처럼 서버에서 사용하기 위한 .java 파일로 변환을 해서 사용합니다.
- 예제 사용 프로토콜
  - Authentication.proto: 인증, 로그인
  
  - GameMulti.proto: 멀티 게임
  
  - GameSingle.proto: 싱글 게임
  
  - Result.proto: 응답 코드
  
  - User.proto: 유저
  
  - 플러그인이 설치되어 있다면 다음과 같이 build.bat 파일을 마우스 오른쪽 버튼으로 클릭해 다음과 같은 명령으로 intelliJ에서 바로 변환할 수 있습니다.
  
    ![reference-1-server_10](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_10.png) 
  
    ![reference-1-server_11](https://static.toastoven.net/prod_gameanvil/images/reference/reference-1-server_11.png) 

## 서버 동작 내용

### 서버 동작 설정

메인 클래스의 GameAnvilServer를 설정하고 실행합니다. 설정할 때 클라이언트와의 프로토콜을 같은 순서로 등록하고 서버에서 사용하는 스레드 풀을 생성합니다.

```java
        GameAnvilServer gameAnvilServer = GameAnvilServer.getInstance();

        // 클라이언트와 전송할 프로토콜 정의 - 순서는 클라이언트와 동일해야 한다.
        gameAnvilServer.addProtoBufClass(Authentication.getDescriptor());
        gameAnvilServer.addProtoBufClass(GameMulti.getDescriptor());
        gameAnvilServer.addProtoBufClass(GameSingle.getDescriptor());
        gameAnvilServer.addProtoBufClass(Result.getDescriptor());
        gameAnvilServer.addProtoBufClass(User.getDescriptor());

        // 게임에서 사용하는 DB 스레드 풀 지정
        gameAnvilServer.createExecutorService(GameConstants.DB_THREAD_POOL, 100);
        // 게임에서 사용하는 Redis 스레드 풀 지정
        gameAnvilServer.createExecutorService(GameConstants.REDIS_THREAD_POOL, 100);

        // annotation 클래스 등록 처리를 위해 scan package 지정
        gameAnvilServer.addPackageToScan("com.nhn.gameanvil.sample");

        // 서버 실행
        gameAnvilServer.run();
```



GameAnvil에서 사용할 수 있도록 GatewayNode, GameNode, SupportNode, MatchMaker, Room은 @annotaion으로 클래스와 이름을 지정해야 합니다. 서버가 실행될 때 선언된 클래스들은 엔진에서 자동으로 클래스 등록 처리가 이루어집니다.

BaseConnection을 상속 받아 구현한 커넥션 클래스에 선언해 줍니다.

```java
@Connection()
```

BaseGatewayNode를 상속 받아 구현한 게이트웨이 노드에 선언해 줍니다.

```
@GatewayNode()
```

BaseSession을 상속 받아 구현한 세션에 선언해 줍니다.

```
@Session()
```

BaseGameNode를 상속 받아 구현한 게임 노드에 GameAnvilConfig.json에 설정한 게임 노드의 서비스 이름을 지정합니다.

```
@ServiceName(GameConstants.GAME_NAME)
```

BaseUser를 상속 받아 구현한 게임 유저에 게임의 서비스 이름과 유저 타입을 지정합니다. 유저 타입은 클라이언트에 접속해 생성되는 타입입니다.

```
@ServiceName(GameConstants.GAME_NAME)
@UserType(GameConstants.GAME_USER_TYPE)
```

BaseRoom을 상속 받아 구현한 룸에 게임 서비스 이름과 룸 타입을 지정합니다. 룸 타입은 클라이언트에서 접속할 룸 타입을 지정합니다.

```java
// 싱글 게임
@ServiceName(GameConstants.GAME_NAME)
@RoomType(GameConstants.GAME_ROOM_TYPE_SINGLE)

// 룸 매치 게임
@ServiceName(GameConstants.GAME_NAME)
@RoomType(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH)

// 유저 매치 게임
@ServiceName(GameConstants.GAME_NAME)
@RoomType(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH)
```

BaseUserMatchMaker, BaseRoomMatchMaker를 상속 받아 구현한 매치 메이커에는 해당 룸과 같은 서비스 이름과 룸 타입을 지정합니다.

BaseSupportNode를 상속 받아 구현한 서포트 노드에 GameAnvilConfig.json에 설정한 서포트 노드의 서비스 이름을 지정합니다.

```
@ServiceName(GameConstants.SUPPORT_NAME_LAUNCHING)
```



### GameAnvil에서 처리되는 API

#### GameSession - 세션

- com.nhn.gameanvil.sample.gateway.GameSession
  - onAuthenticate() : 클라이언트 인증 요청에 대한 검증 내용 구현

#### GameUser - 유저

- com.nhn.gameanvil.sample.game.user.GameUser
  - onLogin() : 클라이언트 로그인 요청에 대한 내용 구현
  - onMatchRoom() : 클라이언트 멀티 룸 매치 요청에 대한 처리 구현
  - onMatchUser() : 클라이언트 유저 매치 요청에 대한 처리 구현
  - onTransferIn() / onTransferOut() : 유저가 서버를 옮겨갈 때 유저 객체의 데이터를 전송하고 복원 처리

#### SingleGameRoom - 싱글 룸

- com.nhn.gameanvil.sample.game.single.SingleGameRoom
  - onCreateRoom(): 클라이언트 싱글 룸 생성 내용 처리, 룸을 만들고 룸에 입장합니다.
  - onLeaveRoom(): 싱글 룸을 나갈 때에 대한 처리 구현

#### UnlimitedTapRoom - 멀티 룸 매치, 최대 4인

- com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoom
  - onInit() : 룸 생성 시 처리, 룸에서 사용하는 데이터 초기화 처리
  - onCreateRoom() : 최초 룸 매치용 룸을 생성하고 입장하는 부분 처리
  - onJoinRoom(): 두 번째 이후 입장하는 유저에 대한 처리
  - onLeaveRoom(): 룸 나갈 때에 대한 처리
  - onTransferIn() / onTransferOut() : 룸을 다른 서버로 옮길 때 룸 안에 있는 데이터 전송하고 복원 처리

#### UnlimitedTapRoomMatchMaker: 룸 매치 로직 처리

- com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoomMatchMaker

#### SnakeRoom - 유저 매치, 2인

- com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom
  - onInit() : 룸 생성 시 처리, 룸에서 사용하는 데이터 초기화 처리
  - onCreateRoom() : 최초 룸 매치용 룸을 생성하고 입장하는 부분 처리
  - onJoinRoom() : 두 번째 이후 입장하는 유저에 대한 처리
  - onPostLeaveRoom(): 룸 나간 후에 대한 처리, 둘이서 게임하는 룸이므로 한 명이 나가면 타이머 제거하고, 상대편도 내보냅니다.
  - onTransferIn() / onTransferOut() : 룸을 다른 서버로 옮길 때 룸 안에 있는 데이터 전송하고 복원 처리
  - onTimer() : 서버에서 주기적으로 food 생성해서 룸에 있는 유저들에게 데이터 전송

#### SnakeRoomMatchMaker : 2인 매치하는 로직 처리

- com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoomMatchMaker
  - onMatch() : 룸 매치 로직을 처리

### 패킷 등록

게임 콘텐츠에서 정의해 처리하는 패킷으로 클라이언트와 주고받는 패킷을 등록합니다. 처리하는 클래스는 등록된 종류에 맞는 패킷 핸들러 인터페이스를 구현해야 합니다.

유저가 로그인 상태에서 처리하는 패킷 : com.nhn.gameanvil.sample.game.user.GameUser

```java
static private PacketDispatcher packetDispatcher = new PacketDispatcher();

static {
    packetDispatcher.registerMsg(User.ChangeNicknameReq.getDescriptor(), CmdChangeNicknameReq.class);           // 닉네임 변경 프로토콜
    packetDispatcher.registerMsg(User.ShuffleDeckReq.getDescriptor(), CmdShuffleDeckReq.class);                 // 덱 셔플 프로토콜
    packetDispatcher.registerMsg(GameSingle.ScoreRankingReq.getDescriptor(), CmdSingleScoreRankingReq.class);   // 싱글 점수 랭킹
}
// 처리하는 클래스는 implements IPacketHandler<GameUser> 를 구현해서 만들어야 한다.
```

클라이언트에서 request로 요청 온 패킷에 대해서는 클라이언트에서 응답을 대기하고 있기 때문에 서버에서 처리하고 전달 받은 유저 객체를 통해서 gameUser.reply()로 응답 처리를 해야 합니다.



룸 안에 있을 때 처리하는 패킷 : com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom

```java
private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();

static {
    dispatcher.registerMsg(GameMulti.SnakeUserMsg.getDescriptor(), CmdSnakeUserMsg.class);  // 유저 위치 정보
    dispatcher.registerMsg(GameMulti.SnakeFoodMsg.getDescriptor(), CmdSnakeRemoveFoodMsg.class);  // food 삭제 정보처리
}
// 처리할 클래스는 implements IRoomPacketHandler<SnakeRoom, GameUser>를 구현해 만들어야 한다.
```

서버에서 클라이언트로 전달하는 패킷은 gameUser.send()로 응답 대기 없이 전송합니다.



rest 패킷 : com.nhn.gameanvil.sample.support.LaunchingSupport

```java
private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher();

static {
    // launching
    restMsgHandler.registerMsg("/launching", RestObject.GET, CmdLaunching.class);
}
// 처리할 클래스는 implements IRestPacketHandler를 구현해 만들어야 한다.
```

rest 요청에 대해서는 전달 받은 restObject.writeString()으로 응답 메시지를 전달합니다.


### Redis

com.nhn.gameanvil.sample.redis.RedisHelper

연결 처리

```java
private RedisClusterClient clusterClient;
private StatefulRedisClusterConnection<String, String> clusterConnection;
private RedisAdvancedClusterAsyncCommands<String, String> clusterAsyncCommands;
/**
 * Redis 연결, 사용 전 최초 1회 호출하여 연결해야 한다.
 *
 * @param url  접속 url
 * @param port 접속 port
 * @throws SuspendExecution
 */
public void connect(String url, int port) throws SuspendExecution {    // Redis 연결 처리
    RedisURI clusterURI = RedisURI.Builder.redis(url, port).build();

    // 패스워드가 필요할 경우 패스워드 설정을 추가해 RedisURI를 생성한다.
//  RedisURI clusterURI = RedisURI.Builder.redis(url, port).withPassword("password").build();
    this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI));
    this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient);
    this.clusterAsyncCommands = clusterConnection.async();
}
```

종료 처리

```java
/**
 * 접속 종료 서버가 내려가기 전에 호출되어야 한다.
 */
public void shutdown() {
    clusterConnection.close();
    clusterClient.shutdown();
}
```

사용

```java
/**
 * 유저 데이터 Redis에 저장
 *
 * @param gameUserInfo 유저 정보
 * @return 저장 성공 여부
 * @throws SuspendExecution
 */
public boolean setUserData(GameUserInfo gameUserInfo) throws SuspendExecution {
    String value = GameAnvilUtil.Gson().toJson(gameUserInfo);

    boolean isSuccess = false;
    try {
        Lettuce.awaitFuture(clusterAsyncCommands.hset(REDIS_USER_DATA_KEY, gameUserInfo.getUuid(), value)); // 해당 리턴값은 최초에 set 할 때만 true이고 있는 값 갱신 시에는 false 응답
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

연결처리

```java
    // JAsyncSQL 연결
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

종료처리
```java
    // JAsyncSQL 종료
    public void close() {
        jAsyncSql.disconnect();
    }
```

사용
```java
    /**
     * 유저의 최고 점수 저장
     *
     * @param uuid      유저 유니크 식별자
     * @param highScore 수정할 최고 점수
     * @return 수정된 레코드 수 반환
     * @throws TimeoutException 해당 호출에 대해 timeout이 발생할 수 있음을 의미
     * @throws SuspendExecution 이 메서드는 파이버를 suspend할 수 있음을 의미
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

DB 연결 정보 설정 : resources/maybatis-config.xml

```xml
<!-- MySQL 접속 정보를 지정한다. -->
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

사용할 쿼리 등록 - 외부 xml을 사용하려면 아래 주석 부분을 참고하여 사용합니다.

```xml
  <mappers>
    <!-- 정의된 SQL 구문을 매핑해 준다. 기본적으로 리소스 안에 있는 mapper.xml을 사용할 때 -->
    <mapper resource="query/UserDataMapper.xml"/>
    <!-- 외부 지정된 mapper.xml 파일을 지정할 때는 전체 경로 지정을 사용한다. -->
    <!--<mapper url="file:///C:/_KevinProjects/GameServerEngine/sample-game-server/target/query/UserDataMapper.xml"/>-->
  </mappers>
```

쿼리 : resources/query/UserDataMapper.xml

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

DB 연결 설정 : com.nhn.gameanvil.sample.db.mybatis.GameSqlSessionFactory

```java
/**
 * 게임에서 사용하는 DB 연결 객체
 */
public class GameSqlSessionFactory {
    private static Logger logger = LoggerFactory.getLogger(GameSqlSessionFactory.class);

    private static SqlSessionFactory sqlSessionFactory;

    /** XML에 명시된 접속 정보를 읽어들인다. */
    // 클래스 초기화 블럭 : 클래스 변수의 복잡한 초기화에 사용된다.
    // 클래스가 처음 로딩될 때 한번만 수행된다.
    static {
        // 접속 정보를 명시하고 있는 XML의 경로 읽기
        try {
            // mybatis_config.xml 파일의 경로 지정
            String mybatisConfigPath = System.getProperty("mybatisConfig"); // 파라미터 전달된 경우 서버(실행 시 -DmybatisConfig= 옵션으로 지정)
            logger.info("mybatisConfigPath : {}", mybatisConfigPath);
            if (mybatisConfigPath != null) {
                logger.info("load to mybatisConfigPath : {}", mybatisConfigPath);
                InputStream inputStream = new FileInputStream(mybatisConfigPath);
                if (sqlSessionFactory == null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
                }
            } else {    // 파라미터 전달이 없는 경우 내부 파일에서 설정을 얻는다.
                Reader reader = Resources.getResourceAsReader("mybatis/mybatis-config.xml");
                logger.info("load to resource : mybatis/mybatis-config.xml");
                // sqlSessionFactory가 존재하지 않는다면 생성한다.
                if (sqlSessionFactory == null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 데이터베이스 접속 객체를 통해 DATABASE에 접속한 세션를 리턴한다.
     */
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

실행 시 -DmybatisConfig=를 사용하지 않는 경우, session을 만들 때 다음과 같이 빌드 시 들어간 내부 저장 환경 파일로 설정되는 로그가 기록됩니다.

```java
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : null
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:39] load to resource : mybatis-config.xml
```

실행 시 -DmybatisConfig=를 사용한 경우, session을 만들 때 다음과 같이 지정된 위치 정보로 설정되는 로그가 기록됩니다.

```java
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : .\src\main\resources\mybatis-config.xml
[INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:32] load to mybatisConfigPath : .\src\main\resources\mybatis-config.xml
```

사용 : com.nhn.gameanvil.sample.db.mybatis.UserDbHelperService

```java
/**
 * 유저 정보 DB에 저장
 *
 * @param gameUserInfo 유저 정보 전달
 * @return 저장된 레코드 수
 * @throws TimeoutException
 * @throws SuspendExecution
 */
public int insertUser(GameUserInfo gameUserInfo) throws TimeoutException, SuspendExecution {    // Callable 형태로 Async 실행하고 결과 리턴.
    Integer resultCount = Async.callBlocking(GameConstants.DB_THREAD_POOL, new Callable<Integer>() {
        @Override
        public Integer call() throws Exception {
            SqlSession sqlSession = GameSqlSessionFactory.getSqlSession();
            try {
                UserDataMapper userDataMapper = sqlSession.getMapper(UserDataMapper.class);
                int resultCount = userDataMapper.insertUser(gameUserInfo.toDtoUser());
                if (resultCount == 1) { // 단건 저장이기에 1개면 정상으로 디비 commit
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

## 서버설정
### GameAnvilConfig.json

```json
{
  //-------------------------------------------------------------------------------------
  // 공통 정보.

  "common": {
    "ip": "127.0.0.1", // 노드마다 공통으로 사용하는 IP. (머신의 IP를 지정)
    "meetEndPoints": ["127.0.0.1:18000"], // 대상 노드의 common IP와 communicatePort 등록. (해당 서버 endpoint 포함 가능 , 리스트로 여러 개 가능)
    "debugMode": false // 디버깅 시 각종 timeout이 발생 안 하도록 하는 옵션, 리얼에서는 반드시 false이어야 한다.
  },

  //-------------------------------------------------------------------------------------
  // LocationNode 설정
  "location": {
    "clusterSize": 1, // 총 몇 개의 머신(VM)으로 구성되는가?
    "replicaSize": 1, // 복제 그룹의 크기(master + slave의 개수)
    "shardFactor": 2  // sharding을 위한 인수(아래의 주석 참고)
    // 전체 shard의 개수 = clusterSize x replicaSize x shardFactor
    // 하나의 머신(VM)에서 구동할 shard의 개수 = replicaSize x shardFactor
    // 고유한 shard의 총 개수(master shard의 개수) = clusterSize x shardFactor
  },

  // 매치 노드 설정
  "match": {
    "nodeCnt": 1
  },

  //-------------------------------------------------------------------------------------
  // 클라이언트와의 커넥션을 관리하는 노드.
  "gateway": {
    "nodeCnt": 4, // 노드 개수(노드 번호는 0부터 부여됨).
    "ip": "127.0.0.1", // 클라이언트와 연결되는 IP.
    "dns": "", // 클라이언트와 연결되는 도메인 주소.
    "connectGroup": { // 커넥션 종류.
      "TCP_SOCKET": {
        "port": 18200, // 클라이언트와 연결되는 포트.
        "idleClientTimeout": 240000 // 데이터 송수신이 없는 상태 이후의 타임아웃(0이면 사용하지 않음).
      },
      "WEB_SOCKET": {
        "port": 18300,
        "idleClientTimeout": 0
      }
    }
  },

  //-------------------------------------------------------------------------------------
  // 게임 로비 역할을 하는 노드(게임 룸, 유저를 포함하고 있음).
  "game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "TapTap",
      "channelIDs": ["","","","",""], // 노드마다 부여할 채널 ID(유니크하지 않아도 됨. " " 문자열로 채널 구분 없이 중복 사용도 가능).
      "userTimeout": 5000 // disconnect 이후의 유저 객체 제거 타임아웃.
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

logger를 패키지 이름 단위로 구분해서 지정할 수 있습니다. 따로 지정하면 해당 패키지 이름은 지정된 레벨로 적용되고, 없으면 root로 지정된 설정으로 적용됩니다.

```
    <logger name="com.nhn.gameanvil" level="INFO"/>
    <logger name="com.nhn.gameanvil.sample" level="DEBUG"/>

    <root>
        <level value="WARN"/>
        <appender-ref ref="ASYNC"/>
        <appender-ref ref="STDOUT"/>
    </root>
```