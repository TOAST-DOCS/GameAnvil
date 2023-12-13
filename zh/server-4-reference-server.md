## Game > GameAnvil > 서버 개발 가이드 > 레퍼런스 서버

## 다운로드

https://github.nhnent.com/game-server-engine/sample-game-server.git

![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-1.png)



## 레퍼런스 개발 환경

- IDE: IntelliJ 2019.3
- JDK: AdoptOpenJDK build 1.8.0_275-b01
- **GameAnvil 1.1.0**
- DB
  - MyBatis 3.5.3
  - 각자의 환경에 맞게 IP 주소를 설정
  - MySQL 5.7.29
- Redis
  - GameAnvil에서 제공하는 Lettuce API를 사용
  - 각자의 환경에 맞게 IP주소를 설정



## GameAnvil API Java doc

[GameAnvil Server API - Java doc](http://10.162.4.61:9090/gameanvil)



## IntelliJ에서 실행 환경 설정

- Git 저장소에서 clone한 프로젝트를 IntelliJ로 실행합니다.
![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-2.png)
- sample_game_sever 설정 확인 - 기본 로컬에서 실행할 기본 환경
  - Maven 설정 Dependencies에 **com.nhn.gameanvil:gameanvil:1.1.0-jdk8** 확인
  - GameAnvile은 JDK8/JDK11을 지원
    - JDK8: 1.1.0-jdk8을 사용
    - JDK11: 1.1.0-jdk11을 사용
  - resources/GameAnvilConfig.json 파일에 IP가 127.0.0.1로 되어 있는지 확인
- IntelliJ 서버 실행 환경 설정
  - 프로젝트 설정
    ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-3.png)
    ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-4.png)
    - GameAnvil은 JDK 1.8로 만들어 여러 가지 버전의 JDK가 설치되어 있을 경우 1.8로 설정되어 있는지 확인하고 지정합니다.
      - 1.8이 아닐 경우 maven package나 install 시에 에러가 발생합니다.
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-5.png)
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-6.png)
  - 아래의 내용을 순서대로 설정합니다.
    - 1: 클릭으로 새로운 빌드 환경 설정 추가
    - 2: +로 Application 추가 -> 3 생성
    - 4: 빌드 환경 이름 설정 sample_server
    - 5: 프로젝트 Main 클래스 선택(com.nhn.gameanvil.sample.Main)
    - 6: resources/setting.txt의 # VM Options에 있는 내용 입력(Mac에서 개발할 경우 패스 형식 주의)

```
-Dco.paralleluniverse.fibers.detectRunawayFibers=false
-Dco.paralleluniverse.fibers.verifyInstrumentation=false
-Xms6g
-Xmx6g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication
```

- 7: resources/setting.txt 파일에서 # Program Arguments 항목의 내용을 입력

```
src/main/resources/
```

- 8: JRE 1.8로 설정
- 9: 설정 저장



## IntelliJ로 서버 실행하기

- Maven 탭의 install 명령으로 서버를 설치합니다. 이때, 컴파일 타임에 AOT Instrumentation을 진행합니다.

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-7.png)

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-8.png)

- 앞서 설정해 두었던 "sample_server" 구성을 이용하여 서버를 실행합니다.

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-9.png)

- 서버가 정상적으로 구동되면 아래와 같이 모든 노드에 대해 onReady 로그가 출력됩니다.

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-10.png)

- http://127.0.0.1:25150/management/nodeInfoPage

   URL을 통해 현재 로컬의 sample_game_server의 상태를 확인할 수 있습니다.

  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-11.png)

- 오류 확인

  - 정상적으로 서버가 실행되지 않았다면 설정을 다시 한번 확인해 보거나 log의 에러 부분을 확인해서 문의 부탁드립니다.
  - DB나 Redis의 경우에는 직접 사용할 IP 주소를 설정해야 합니다.



## Maven 빌드와 Command 실행

##### pom.xml 설정 확인

- GameAnvil 버전

```
    <!-- gameanvil-->
    <dependency>
      <groupId>com.nhn.gameanvil</groupId>
      <artifactId>gameanvil</artifactId>
      <version>1.1.0-jdk8</version>
    </dependency>
```

- build 설정

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
          <!-- executable jar 에서 main 으로 실행 될 클래스 -->
          <mainClass>com.nhn.gameanvil.sample.Main</mainClass>
          <!-- jar 파일 안의 META-INF/MANIFEST.MF 에 classpath 정보가 추가됨 -->
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

##### Maven package 빌드 실행

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-12.png)

- 정상적으로 실행되면 ./target/ 폴더에 빌드된 파일들이 있습니다.

- ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-13.png)

- 서버를 구동하려면 sample_game_server-1.0.1.jar 파일과 config, query 폴더의 파일을 복사해 사용하면 됩니다.

- 명령 프롬프트에서 실행

  - 명령 프롬프트(cmd)를 실행해서 빌드된 target 폴더로 이동합니다(각자 자신의 환경에 맞는 경로에서 진행).

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-14.png)

  - command 실행

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-15.png)

    - ```
      java -Dco.paralleluniverse.fibers.detectRunawayFibers=false -Dco.paralleluniverse.fibers.verifyInstrumentation=false -Dconfig.file=.\config\GameAnvilConfig.json -Dlogback.configurationFile=.\config\logback.xml -DmybatisConfig=.\config\mybatis-config.xml -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar .\sample_game_server-1.1.0.jar
      ```

      - 기본으로 실행 시에 별도의 옵션이 지정되지 않으면 빌드할 때 지정되어 있는 환경 파일 적용
      - GameAnvilConfig.json은 -Dconfig.file 옵션으로 경로와 파일 이름을 지정
      - logback.xml은 -Dlogback.configurationFile 옵션으로 경로와 파일이름을 지정
      - mybatis용 config 설정은 -DmybatisConfig 옵션 지정
        - mybatis는 GameAnvil이 지원하는 기능의 일부가 아닙니다. 이 부분은 샘플용입니다.
      - com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory 참고

    - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-16.png)

    - onReady가 나오면 정상

- 매번 maven 빌드로 테스트하기 번거로워 로컬에서 테스트할 때는 Vm Option에`-javaagent:.\src\main\resources\META-INF\quasar-core-0.7.10-jdk8.jar=bm` 옵션을 추가하고 intelliJ에서 바로 로컬 서버를 실행할 수 있습니다.



## JIT 설정(Optional)

- GameAnvil는 AOT Instrumentation뿐만 아니라 JIT Instrumentation도 지원합니다.

### pom 설정

- 아래의 AOT Instrumentation ( ) 플러그인 부분을 주석 처리하거나 삭제합니다.

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

### VM option설정

- JIT 모드의 경우 반드시 VM 옵션에 아래와 같은 javaagent 항목을 추가해서 quasar agent가 구동될 수 있도록 해야 합니다.

```
-javaagent:.\src\main\resources\META-INF\quasar-core-0.7.10-jdk8.jar=bm
```

### 빌드 및 실행

- IntelliJ
  - intelliJ를 이용한 실행
- CLI
  - maven clean
  - maven package
  - command 실행(스크립트 혹은 배치 파일을 작성해서 구동할 것을 추천)
    - `java -javaagent:.\lib\quasar-core-0.7.10-jdk8.jar=bm -Dco.paralleluniverse.fibers.detectRunawayFibers=false -Dconfig.file=.\config\GameAnvilConfig.json -Dlogback.configurationFile=.\config\logback.xml -DmybatisConfig=.\config\mybatis-config.xml -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication -jar .\sample_game_server-1.1.0.jar`



## sample_game_server 살펴보기

- GameAnvil 샘플로 제작된 프로젝트입니다. 참고하시고 불편하시거나 필요하거나 궁금한 부분은 언제든지 문의 해주세요.

### Gamebase

- NHN Cloud의 Gamebase 등록
  - 서버에서 앱 ID, SecretKey 필요

### Gateway

- 인증
  - Gamebase의 userId, token을 전달받아 token 검증

### Game

- 유저
  - 로그인
    - DB에서 유저 식별자를 가지고 유저 조회
      - 유저가 있는 경우 DB에서 가져온 정보 사용
      - 유저가 없는경우 DB에 신규 정보 기록
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
    - 전달받은 범위로 Redis에서 싱글 게임 랭킹 리스트 검색
    - 로그인할 때마다 저장해둔 Redis의 유저 정보에서 랭킹 리스트의 닉네임 설정해서 랭킹 목록 생성
    - 생성된 랭킹 리스트 응답
- 싱글 게임
  - 혼자 방을 만들어 게임
    - 게임 시작할 때 전달받은 패킷에 싱글 룸에서 필요한 데이터 룸에 설정
  - 탭 메시지 패킷 처리
    - 응답이 필요 없는 메시지로 전달받으면 현재 룸에 스코어 점수 기록
  - 방에서 나올 때
    - 로그인된 유저 객체의 최고 점수보다 방 안에 저장된 점수가 크면
      - DB에 유저 최고 기록 업데이트
      - 성공하면 Redis의 랭킹 정보 업데이트
    - 게임 종료 패킷 처리해서 현재 방에서 플레이한 점수 응답
- 멀티 게임
  - TapBird
    - 룸 매치로 1\~4인까지 플레이가 가능합니다. 룸에서 모두 나갈 때까지 플레이하는 점수를 모든 사용자가 받습니다.
    - 룸이 하나도 없다면 룸을 만들어서 입장
      - 룸에 유저와 점수 정보를 등록
      - 룸 정보에 룸 아이디와 현재 입장한 유저 수를 갱신합니다.
    - 룸이 있다면 기존에 있는 룸에 입장
      - 룸에 유저와 점수 정보를 등록
      - 룸 정보에 현재 유저 수를 갱신합니다.
    - 룸에서 나갈때
      - 룸에 유저와 점수 정보를 삭제합니다.
      - 룸 정보의 유저 수를 갱신합니다.
    - 유저 점수 증가 패킷 처리
      - 응답이 필요 없는 형태로 전달받은 점수를 룸의 유저 정보를 갱신
      - 유저 점수 리스트를 만들어서 현재 룸 안에 있는 모든 유저에게 점수를 전달합니다.
  - Snake
    - 유저 매치로 2명이 동시에 룸에 입장해서게임 플레이
    - 둘이 매치되었을 때 한 명은 룸을 만들어 들어가고 한 명은 생성된 룸에 들어갑니다.
      - 첫 번째 유저 위치 지정 후 룸에 유저 정보와 점수 정보를 등록합니다.
      - 두 번째 입장할 때 위치 지정과 룸에 유저 정보와 점수 정보를 등록합니다.
      - 유저가 2명이 되면 유저에게 게임 정보를 전송합니다.
      - 룸에서 food 생성 타이머를 시작합니다.
    - 룸에서 food 개수가 max가 될 때까지 1초에 한번씩 food를 생성해서 유저들에게 응답이 필요 없는 데이터로 전송
    - 룸에서 나갈 때
      - 룸에 유저 정보와 점수 정보를 삭제합니다.
      - 타이머를 정지합니다.
      - 모든 유저를 방에서 내보냅니다.
    - food 삭제 패킷 처리
      - 삭제할 food 정보를 받아서 룸에 있는 food 정보를 삭제합니다.
      - 상대에게 삭제된 food를 전달합니다.
    - user 이동 패킷 처리
      - 유저가 이동할 때의 정보를 전달받아 룸에 유저 정보를 저장하고 상대방에게 이동된 유저 정보를 전달합니다.

### Support - rest service

- 게임 서버 접속 전 게임 세션 정보를 얻어오기 위한 서비스
- http://127.0.0.1:10080/launching?platform=Editor&appStore=GOOGLE&appVersion=1.2.0&deviceId=4D34C127-9C56-5BAB-A3C2-D8F18C0B7B6E 형식으로 요청
- 론칭 정보 요청 패킷 처리
  - 전달받은 데이터를 파싱하고 확인하고 게임 서버 세션 서버의 IP, PORT를 응답합니다.

### Redis

- GameAnvil에서 제공하는 lettuce 연동
- GameNode의 onInit()에서 Redis 연결 설정 Node마다 만들어서 연결
  - Singleton으로 만들어서 모든 노드에서 하나의 Redis 연결을 사용하면 안 됩니다.
- GameNode의 onShutdown()에서 Redis의 연결 shutdown 처리를 해야 합니다.
- 예제 기능
  - hmget으로 유저 데이터 리스트 검색
  - zadd로 싱글 게임 랭킹 저장
  - zreverangeWithScores로 랭킹 정보 검색

### DB - MyBatis

- resources/mybatis에 DB 설정 정보만 있습니다.
- com.nhn.gameanvil.sample.mybatis.mappers 패키지 안에 mapper.xml 파일이 존재합니다.
- 예제 기능
  - 유저 정보 INSERT
  - uuid로 유저 정보 SELECT
  - 덱, 닉네임, 최고 점수 UPDATE

### protocol - google protobuf 3.0

- 클라이언트와 사용하는 모든 패킷은 google protobuf로 작성
- .proto 파일은 클라이언트와 공용으로 제작하며 build.bat에 있는 것처럼 서버에서 사용하기 위한 .java 파일로 변환을 해서 사용합니다.
- 예제 사용 프로토콜
  - Authentication.proto: 인증, 로그인
  - GameMulti.proto: 멀티 게임
  - GameSingle.proto: 싱글 게임
  - Result.proto: 응답 코드
  - User.proto: 유저
-
  - 플러그인이 설치되어 있다면 다음과 같이 build.bat 파일을 마우스 오른쪽 버튼으로 클릭해 다음과 같은 명령으로 intelliJ에서 바로 변환할 수 있습니다.
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-18.png)
  - ![image.png](http://static.toastoven.net/prod_gameanvil/images/ReferenceServer-19.png)

### GameAnvilBootstrap : com.nhn.gameanvil.sample.Main

```
public static void main(String[] args) {
    GameAnvilBootstrap bootstrap = GameAnvilBootstrap.getInstance();

    // 클라이언트와 전송할 프로토콜 정의 - 순서는 클라이언트와 동일 해야 한다.
    bootstrap.addProtoBufClass(0, Authentication.getDescriptor());
    bootstrap.addProtoBufClass(1, GameMulti.getDescriptor());
    bootstrap.addProtoBufClass(2, GameSingle.getDescriptor());
    bootstrap.addProtoBufClass(3, Result.getDescriptor());
    bootstrap.addProtoBufClass(4, User.getDescriptor());

    // 게임에서 사용하는 DB 쓰레드풀 지정
    bootstrap.createExecutorService(GameConstants.DB_THREAD_POOL, 100);
    // 게임에서 사용하는 레디스 쓰레드풀 지정
    bootstrap.createExecutorService(GameConstants.REDIS_THREAD_POOL, 100);

    // 세션설정
    bootstrap.setGateway()
        .connection(GameConnection.class)
        .session(GameSession.class)
        .node(GameGatewayNode.class)
        .enableWhiteModules();

    // 게임 스페이스 설정
    bootstrap.setGame(GameConstants.GAME_NAME)
        .node(GameNode.class)

        // 싱글 게임
        .user(GameConstants.GAME_USER_TYPE, GameUser.class)
        .room(GameConstants.GAME_ROOM_TYPE_SINGLE, SingleGameRoom.class)

        // 룸 매치 멀티게임 - 방에 들어가서 게임 : 무제한 탭
        .room(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH, UnlimitedTapRoom.class)
        .roomMatchMaker(GameConstants.GAME_ROOM_TYPE_MULTI_ROOM_MATCH, UnlimitedTapRoomMatchMaker.class, UnlimitedTapRoomInfo.class)

        // 유저 매치 멀티게임 - 유저들 매칭으로 인해 게임동시 입장 : 스테이크 게임
        .room(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH, SnakeRoom.class)
        .userMatchMaker(GameConstants.GAME_ROOM_TYPE_MULTI_USER_MATCH, SnakeRoomMatchMaker.class, SnakeRoomInfo.class);

    // 서비스 설정
    bootstrap.setSupport(GameConstants.SUPPORT_NAME_LAUNCHING)
        .node(LaunchingSupport.class);

    bootstrap.run();
}
```

### GameAnvil 내부에서 처리되는 메서드

- GameSession - 세션
  - com.nhn.gameanvil.sample.gateway.GameSession
    - onAutenticate() : 클라이언트 인증 요청에 대한 검증 내용 구현
- GameUser - 유저
  - com.nhn.gameanvil.sample.game.user.GameUser
    - onLogin() : 클라이언트 로그인 요청에 대한 내용 구현
    - onMatchRoom() : 클라이언트 멀티 룸 매치 요청에 대한 처리 구현
    - onMatchUser() : 클라이언트 유저 매치 요청에 대한 처리 구현
    - onTransferIn() / onTransferOut() : 유저가 서버를 옮겨갈 때 유저 객체의 데이터를 전송하고 복원 처리
- SingleGameRoom - 싱글 룸
  - com.nhn.gameanvil.sample.game.single.SingleGameRoom
    - onCreateRoom(): 클라이언트 싱글 룸 생성 내용 처리, 룸을 만들고 룸에 입장합니다.
    - onLeaveRoom(): 싱글 룸을 나갈 때에 대한 처리 구현
- UnlimitedTapRoom - 멀티 룸 매치, 최대 4인
  - com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoom
    - onCreateRoom() : 최초 룸 매치용 룸을 생성하고 입장하는 부분 처리
    - onJoinRoom(): 두 번째 이후 입장하는 유저에 대한 처리
    - onLeaveRoom(): 룸 나갈 때에 대한 처리
    - onTransferIn() / onTransferOut() : 룸을 다른 서버로 옮길 때 룸 안에 있는 데이터 전송하고 복원 처리
- UnlimitedTapRoomMatchMaker: 룸 매치 로직 처리
  - com.nhn.gameanvil.sample.game.multi.roommatch.UnlimitedTapRoomMatchMaker
- SnakeRoom - 유저 매치, 2인
  - com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom
    - onCreateRoom() : 최초 룸 매치용 룸을 생성하고 입장하는 부분 처리
    - onJoinRoom() : 두 번째 이후 입장하는 유저에 대한 처리
    - onPostLeaveRoom(): 룸 나간 후에 대한 처리, 둘이서 게임하는 룸이므로 한 명이 나가면 타이머 제거하고, 상대편도 내보냅니다.
    - onTransferIn() / onTransferOut() : 룸을 다른 서버로 옮길 때 룸 안에 있는 데이터 전송하고 복원 처리
    - onTimer() : 서버에서 주기적으로 food 생성해서 룸에 있는 유저들에게 데이터 전송
- SnakeRoomMatchMaker : 2인 매치하는 로직 처리
  - com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoomMatchMaker

### 패킷 등록

- 클라이언트가 전송하는 패킷 등록
- 게임 콘텐츠에서 정의해서 처리하는 패킷
- 처리하는 클래스는 등록된 종류에 맞는 패킷 핸들러 인터페이스를 구현해야 합니다.
- 유저가 로그인 상태에서 처리하는 패킷 : com.nhn.gameanvil.sample.game.user.GameUser

```
static private PacketDispatcher packetDispatcher = new PacketDispatcher();

static {
    packetDispatcher.registerMsg(User.ChangeNicknameReq.getDescriptor(), CmdChangeNicknameReq.class);           // 닉네임 변경 프로토콜
    packetDispatcher.registerMsg(User.ShuffleDeckReq.getDescriptor(), CmdShuffleDeckReq.class);                 // 덱 셔플 프로토콜
    packetDispatcher.registerMsg(GameSingle.ScoreRankingReq.getDescriptor(), CmdSingleScoreRankingReq.class);   // 싱글 점수 랭킹
}
// 처리하는 클래스는 implements IPacketHandler<GameUser> 를 구현해서 만들어야 한다.
```

- 클라이언트에서 request로 요청온 패킷에 대해서는 클라이언트에서 응답을 대기하고 있기 때문에 서버에서 처리하고 전달받은 유저 객체를 통해서 gameUser.reply()로 응답 처리를 해야 합니다.
- 룸 안에 있을 때 처리하는 패킷 : com.nhn.gameanvil.sample.game.multi.usermatch.SnakeRoom

```
private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();

static {
    dispatcher.registerMsg(GameMulti.SnakeUserMsg.getDescriptor(), CmdSnakeUserMsg.class);  // 유저 위치 정보
    dispatcher.registerMsg(GameMulti.SnakeFoodMsg.getDescriptor(), CmdSnakeRemoveFoodMsg.class);  // food 삭제 정보처리
}
// 처리하는 클래스는 implements IRoomPacketHandler<SnakeRoom, GameUser> 를 구현해서 만들어야 한다.
```

- 서버에서 클라이언트로 전달하는 패킷은 gameUser.send()로 응답 대기 없이 전송합니다.
- rest 패킷 : com.nhn.gameanvil.sample.support.LaunchingSupport

```
private static RestPacketDispatcher restMsgHandler = new RestPacketDispatcher();

static {
    // launching
    restMsgHandler.registerMsg("/launching", RestObject.GET, CmdLaunching.class);
}
// 처리하는 클래스는 implements IRestPacketHandler 를 구현해서 만들어야 한다.
```

- rest요청에 대해서는 전달받은 restObject.writeString()으로 응답 메시지를 전달합니다.



### 외부 http 요청처리 사용 : com.nhn.gameanvil.sample.gateway.GameConnection

- 게임 서버 -> 외부 서버 예) Gamebase token 검증
- Gateway 서버 onAutenticate()에서 검증 요청 처리

```
// Gamebse 인증
//----------------------------------- 토큰 유효한지에 대한 검증 Gamebase
String gamebaseUrl = String.format(GameConstants.GAMEBASE_DEFAULT_URL + "/tcgb-gateway/v1.2/apps/X2bqX5du/members/%s/tokens/%s", accountId, authenticationReq.getAccessToken());
HttpRequest httpRequest = new HttpRequest(gamebaseUrl);
httpRequest.getBuilder().addHeader("Content-Type", "application/json");
httpRequest.getBuilder().addHeader("X-Secret-Key", GameConstants.GAMEBASE_SECRET_KEY);
logger.info("httpRequest url [{}]", gamebaseUrl);
HttpResponse response = httpRequest.GET();
logger.info("httpRequest response:[{}]", response.toString());

// Gamebase 응답 json 데이터 객체 파싱
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

- 연결

```
private RedisClusterClient clusterClient;
private StatefulRedisClusterConnection<String, String> clusterConnection;
private RedisAdvancedClusterAsyncCommands<String, String> clusterAsyncCommands;
/**
 * 레디스 연결, 사용하기전에 최초에 한번 호출해서 연결 해야 한다.
 *
 * @param url  접속 url
 * @param port 점속 port
 * @throws SuspendExecution
 */
public void connect(String url, int port) throws SuspendExecution {    // 레디스 연결 처리
    RedisURI clusterURI = RedisURI.Builder.redis(url, port).build();
    this.clusterClient = RedisClusterClient.create(Collections.singletonList(clusterURI));
    this.clusterConnection = Lettuce.connect(GameConstants.REDIS_THREAD_POOL, clusterClient);
    this.clusterAsyncCommands = clusterConnection.async();
}
```

- 종료

```
/**
 * 접속 종료 서버가 내려가기전에 호출되어야 한다,
 */
public void shutdown() {
    clusterConnection.close();
    clusterClient.shutdown();
}
```

- 사용

```
/**
 * 유저 데이터 레디스에 저장
 *
 * @param gameUserInfo 유저 정보
 * @return 저장 성고 여부
 * @throws SuspendExecution
 */
public boolean setUserData(GameUserInfo gameUserInfo) throws SuspendExecution {
    String value = GameAnvilUtil.Gson().toJson(gameUserInfo);

    boolean isSuccess = false;
    try {
        Lettuce.awaitFuture(clusterAsyncCommands.hset(REDIS_USER_DATA_KEY, gameUserInfo.getUuid(), value)); // 해당 리턴값은 최초에 set 할때만 true 이고 있는값갱신시에는 false 응답
        isSuccess = true;
    } catch (TimeoutException e) {
        logger.error("setUserData - timeout", e);
    }
    return isSuccess;
}
```

### DB

- 설정 : resources/maybatis-config.xml
  - DB 연결 정보

```
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

- 사용할 쿼리 등록 - 외부 xml을 사용하려면 아래 주석 부분을 참고해서 사용 하면 됩니다.

```
  <mappers>
    <!-- 정의된 SQL구문을 맵핑해준다. 기본적으로 리소스 안에 있는 mapper.xml을 사용 할때-->
    <mapper resource="query/UserDataMapper.xml"/>
    <!-- 외부 지정된 mapper.xml 파일을 지정할때는 전체 경로 지정을 사용한다. -->
    <!--<mapper url="file:///C:/_KevinProjects/GameServerEngine/sample-game-server/target/query/UserDataMapper.xml"/>-->
  </mappers>
```

- 쿼리 : resources/query/UserDataMapper.xml

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

- DB 연결 설정 : com.nhn.gameanvil.sample.mybatis.GameSqlSessionFactory

```
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
            String mybatisConfigPath = System.getProperty("mybatisConfig"); // 파라미터 전달 된경우 서버 (실행시 -DmybatisConfig= 옵선으로지정)
            logger.info("mybatisConfigPath : {}", mybatisConfigPath);
            if (mybatisConfigPath != null) {
                logger.info("load to mybatisConfigPath : {}", mybatisConfigPath);
                InputStream inputStream = new FileInputStream(mybatisConfigPath);
                if (sqlSessionFactory == null) {
                    sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
                }
            } else {    // 파라미터 전달이없는 경우 내부 파일에서 설정 얻는다
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

- 실행 시 -DmybatisConfig=를 사용하지 않는 경우, session을 만들 때 다음과 같이 빌드 시 들어간 내부 저장 환경 파일로 설정되는 로그가 기록됩니다.

```
[2020-12-18 17:36:35,634] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : null
[2020-12-18 17:36:35,636] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:39] load to resource : mybatis-config.xml
```

- 실행시 -DmybatisConfig=를 사용한 경우, session을 만들 때 다음과 같이 지정된 위치 정보로 설정되는 로그가 기록됩니다.

```
[2020-12-18 17:43:37,871] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:30] mybatisConfigPath : .\src\main\resources\mybatis-config.xml
[2020-12-18 17:43:37,871] [INFO ] [GameAnvil-DB_THREAD_POOL-0] [GameSqlSessionFactory.java:32] load to mybatisConfigPath : .\src\main\resources\mybatis-config.xml
```

- 사용 : com.nhn.gameanvil.sample.mybatis.UserDbHelperService

```
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

- 샘플 사용 DB 스키마

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
  // 공통 정보.

  "common": {
    "ip": "127.0.0.1", // 노드마다 공통으로 사용하는 IP. (머신의 IP를 지정)
    "meetEndPoints": ["127.0.0.1:16000"], // 대상 노드의 common IP와 communicatePort 등록. (해당 서버 endpoint 포함가능 , 리스트로 여러개 가능)
    "ipcPort": 16000, // 다른 ipc node 와 통신할때 사용되는 port
    "publisherPort" : 13300, // publish socket 을 위한 port
    "debugMode": false //디버깅시 각종 timeout 이 발생안하도록 하는 옵션 , 리얼에서는 반드시 false 이어야 한다.
  },

  //-------------------------------------------------------------------------------------
  // LocationNode 설정
  "location": {
    "clusterSize": 1, // 총 몇개의 머신(VM)으로 구성되는가?
    "replicaSize": 3, // 복제 그룹의 크기 (master + slave의 개수)
    "shardFactor": 3  // sharding을 위한 인수 (아래의 주석 참고)
    // 전체 shard의 개수 = clusterSize x replicaSize x shardFactor
    // 하나의 머신(VM)에서 구동할 shard의 개수 = replicaSize x shardFactor
    // 고유한 shard의 총 개수 (master 샤드의 개수) = clusterSize x shardFactor
  },

  // 매치 노드 설정
  "match": {
    "nodeCnt": 1,
    "useLocationDirect": true
  },

  //-------------------------------------------------------------------------------------
  // 클라이언트와의 커넥션을 관리하는 노드.
  "gateway": {
    "nodeCnt": 4, // 노드 개수. (노드 번호는 0 부터 부여 됨)
    "ip": "127.0.0.1", // 클라이언트와 연결되는 IP.
    "dns": "", // 클라이언트와 연결되는 도메인 주소.
    "maintenance": false,
    "tcpNoDelay": false, // Netty Bootstrap 설정시 사용 됨. (디폴트로 필드 미사용 및 기본 값 false)
    "connectGroup": { // 커넥션 종류.
      "TCP_SOCKET": {
        "port": 11200, // 클라이언트와 연결되는 포트.
        "idleClientTimeout": 240000 // 데이터 송수신이 없는 상태 이후의 타임아웃. (0 이면 사용하지 않음)
        //        ,"secure": { // 보안 설정.
        //          "useSelf": true
        ////          ,"keyCertChainPath": "gameanvil.crt" // 인증서 경로.
        ////          ,"privateKeyPath": "privatekey.pem" // 개인 키 경로.
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
  // 게임 로비 역할을 하는 노드. (게임 룸, 유저를 포함 하고있음)
  "game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "TapTap",
      "channelIDs": ["","","","",""], // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. "" 문자열로 채널 구분없이 중복사용도 가능)
      "userTimeout": 5000 // disconnect 이후의 유저객체 제거 타임아웃.
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
  // JMX 또는 REST API 사용하여 다른 노드에 대한 관리를 할 수 있는 노드. (서비스 포즈, 전체 유저 카운트 등)
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

- logger를 패키지 이름 단위로 구분해서 지정할 수 있습니다. 따로 지정하면 해당 패키지 이름은 지정된 레벨로 적용되고, 없으면 root로 지정된 설정으로 적용됩니다.

```
    <logger name="com.nhn.gameanvil" level="INFO"/>
    <logger name="com.nhn.gameanvil.sample" level="DEBUG"/>

    <root>
        <level value="WARN"/>
        <appender-ref ref="ASYNC"/>
        <appender-ref ref="STDOUT"/>
    </root>
```
