## Game > GameAnvil > 테스트 개발 가이드 > 시작하기

## GameHammer

GameAvil 통합 솔루션의 강력한 기능중 하나로,  GameAvlil 엔진을 이용한 게임 서버 개발 도구로, 강력하고 편리한 테스트 도구입니다. 실제 컨넥터에서 제공하는 모든 기능을 사용할 수 있으며, 다양한 테스트 케이스를 만 들 수 있도록 다양한 API를 제공하고 있습니다. 또 스트레스 테스트를 위해 다수의 GameHammer를 동시에 실행하고, 그 결과를 취합해 바로 확인할 수 있습니다.



### 시스템 요구사항

GameHammer를 사용하기위해서는 다음과 같은 사항이 필요합니다. 

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

GameHammer는 다음과 같은 기능을 지원합니다.

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



## 프로젝트에 GameHamer 추가하기.

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
