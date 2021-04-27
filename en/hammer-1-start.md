## Game > GameAnvil > Test Development Guide > Getting Started

## GameHammer

GameHammer is a powerful and easy-to-use game server development testing tool based on the GameAnvil engine. It provides all the features of actual connector and the API that can be used to a variety of test cases. In addition, multiple GameHammers can be run for stress testing and aggregate the result and check it.

### System requirements

The following requirements must be met to use GameHammer. 

- Supported languages
    - Java
- Targeted development environment
    - InteliJ
- Supported network protocols
    - TCP/IP
    - SSL over TCP/IP
- Available application protocol types
    - Google Protocol Buffers
    - Google FlatBuffers(planned)
    - Custom byte stream
    - HTTP/HTTPS(limited to specific purposes)

### Benefits

GameHammer supports the following features.

- Supports all the features for connector
- Supports both sync and async mode
    - Provides API in async mode
    - Provides future for Sync mode
- Able to use thousands or more connections at the same time
- Supports the status-based scenario management feature

### Reference project

| Server                                                         | Test code                                                  | Description                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| [RPS](https://github.nhnent.com/game-server-engine/GameAnvil-rps) | [RPS-test](https://github.nhnent.com/game-server-engine/GameHammer-rps-test) | Test code using actual game server and GameHammer |

## Adding GameHammer to Project

Like GameAnvil, GameHammer is deployed through Maven. Add the following items to the dependencies element of the pom.xml file to use GameHammer. If GameHammer fails to be installed, check if the following URLs are correctly registered to the in-house nexus of the repository element.

```
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
