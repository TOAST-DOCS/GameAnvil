## Game > GameAnvil > 테스트 개발 가이드 > 시작하기

## GameHammer

GameAvil 통합 솔루션의 강력한 기능중 하나로,  GameAvlil 엔진을 이용한 게임 서버 개발 도구로, 강력하고 편리한 테스트 도구입니다. 실제 컨넥터에서 제공하는 모든 기능을 사용할 수 있으며, 다양한 테스트 케이스를 만 들 수 있도록 다양한 API를 제공하고 있습니다. 또 스트레스 테스트를 위해 다수의 GameHammer를 동시에 실행하고, 그 결과를 취합해 바로 확인할 수 있습니다.



### 시스템 요구사항

* 지원하는 언어
  * Java
* 타겟 개발환경
  * InteliJ
* 지원하는 네트워크 프로토콜
  * TCP/IP
  * SSL over TCP/IP
* 사용가능한 응용 프로토콜 형식
  * Google Protocol Buffers
  * Google FlatBuffers (예정)
  * 커스텀 바이트 스트림
  * HTTP/HTTPS (특정한 용도로 한정)



### Features

* Connector와 대응하는 모든 기능 지원
* Sync/Async 방식 모두 지원
  * Async 방식의 API 제공
  * Sync 방식을 위한 future 제공
* 수천개 또는 그 이상의 Connection 동시에 사용가능
* 상태 기반의 시나리오 관리 기능 지원



### 레퍼런스 프로젝트

| 서버                                                         | 테스트코드                                                   | 설명                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| [RPS](https://github.nhnent.com/game-server-engine/GameAnvil-rps) | [RPS-test](https://github.nhnent.com/game-server-engine/GameHammer-rps-test) | 실제 게임 서버와 GameHammer를 사용한 테스트 코드 |



### 시작하기

GameHammer를 사용하기위한 간단한 방법을 소개합니다.



#### 1. 프로젝트에 GameHamer 추가하기.

GameHammer는 GameAnvil과 마찬가지로 maven을 통해 배포됩니다. pom.xml 파일의 dependencies항목에 다음과 같이 추가하시면 GameHammer를 사용할 수 있습니다. GameHammer 설치가 실패하면 repository에 nexus가 등록 되어있는지 확인하고, 등록 되어있지 않다면 아래처럼 추가해 주시면 됩니다.

```pom.xml
...
<repositories>
	<repository>
		<id>releases</id>
		<name>Nhnent Maven Release Repository</name>
		<url>http://nexus.nhnent.com/content/repositories/releases/</url>
	</repository>
	<repository>
		<id>snapshots</id>
		<name>Nhnent Maven Snapshot Repository</name>
		<url>http://nexus.nhnent.com/content/repositories/snapshots/</url>
	</repository>
</repositories>
...    
<dependencies>
		...
        <!-- test agent (java connector) -->
        <dependency>
            <groupId>com.nhn.gameanvil</groupId>
            <artifactId>gameahammer</artifactId>
            <version>DEV-1.0.0</version>
        </dependency>
        ...
<dependencies>
...        
```



#### 2. Tester

GameHammer를 사용하기 위한 기본 모듈 입니다. 기본 설정과 Connection의 관리를 담당합니다. 아래와 같이 Tester를 생성할 수 있습니다.

```java
Tester tester = Tester.newBuilder()
            .addProtoBufClass(0, RPSGame.getDescriptor())
            .Build();
```

`.addProtoBufClass(0, RPSGame.getDescriptor())` 를 이용해 게임서버와 주고받을 응용 프로토콜을 등록합니다.



#### 3. Connection

다음으로 Connection객체를 생성합니다. Connection 객체는 게임서버와의 연결 및 인증 기능을 처리하고, User객제를 관리해줍니다. 아래와 같이 Tester를 생성할 수 있습니다. 

```java
Connection connection = tester.createConnection(uuid);
```



##### 서버에 접속하기

```java
Future<ResultConnect> future = connection.connect(new RemoteInfo("127.0.0.1", 11200), (resultConnect)->{
    if(resultConnect.isSuccess()){
		// connect success
    }
});

ResultConnect resultConnect = futre.get(); // blocked
if(resultConnect.isSuccess()){
	// connect success
}
```



##### 인증하기

```java
Future<ResultAuthentication> future = connection.authentication(accountId, password, deviceId, payloads, (resultAuthentication)->{
	if(resultAuthentication.isSuccess){
		// authentication success
	}
});

ResultAuthentication resultAuthentication = future.get(); // blocked
if(resultAuthentication.isSuccess){
	// authentication success
}

```



#### 4. User

다음으로 User객체를 생성합니다. User 객체는 서비스에 로그인하고, 방생성 및 입장, 매치등의 기능을 제공합니다. 아래와 같이 User를 생성할 수 있습니다. 

```
User user = connection.getUser(serviceName, subId);
if(user == null){
	user = connection.createUser(serviceName, subId);
}
```



##### 로그인하기

```java
Future<ResultLogin> future = user.login(resultLogin -> {
	if(resultLogin.isSuccess()){
		// login success
	}, userType, channelId, payloads);
	
ResultLogin resultLogin = future.get(); // blocked
if(resultLogin.isSuccess()){
	// login success
}
	
```



##### 방생성

```
Future<ResultCreateRoom> future = user.login(resultCreateRoom -> {
	if(resultCreateRoom.isSuccess()){
		// createRoom success
	}, roomType, roomName, sendPayloads);
	
ResultCreateRoom resultCreateRoom = future.get(); // blocked
if(resultCreateRoom.isSuccess()){
	// createRoom success
}
```



##### Request

```
MessagReq req = MessageReq.newBuilder().build();
Future<PacketResult> future = user.request(req, packetResult -> {
	if(packetResult.isSuccess()){
		// request success;
		MessageRes res = MessageRes.parseFrom(packetResult.getStream());
	}
});
	
PacketResult packetResult = future.get(); // blocked
if(packetResult.isSuccess()){
	// request success;
    MessageRes res = MessageRes.parseFrom(packetResult.getStream());
}
```

##### Send

```
MessagSend send = MessageSend.newBuilder().build();
user.send(send);
```



#### 5. 종료

테스트 종료시점에는 Tester.close()를 호출하여 테스트 과정에서 사용된 Connection과 User를 모두 정리합니다. 