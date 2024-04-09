## Game > GameAnvil > Test Development Guide > Getting Started

## GameHammer

GameHammer is a powerful and easy-to-use game server development testing tool based on the GameAnvil engine. It provides all the features of an actual connector and the API that can be used for a variety of test cases. In addition, multiple GameHammers can be run for stress testing and aggregate the results and check them.

### System Requirements

The following requirements must be met to use GameHammer. 

- Supported Languages
    - Java
- Targeted Development Environment
    - IntelliJ
- Supported Network Protocols
    - TCP/IP
    - SSL over TCP/IP
- Available Application Protocol Types
    - Google Protocol Buffers
    - Google FlatBuffers (Planned)
    - Custom byte stream
    - HTTP/HTTPS (Limited to specific uses)

### Characteristics

GameHammer supports the following features.

- Supports all the features for connector
- Supports both sync and async mode
    - Provides API in async mode
    - Provides future for Sync mode
- Thousands or more connections available concurrently
- Supports the state-based scenario management feature

### Reference project

| Server                                                         | Test Code                                                  | Description                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------ |
| [sample-game-server](https://github.com/nhn/gameanvil.sample-game-server.git) | [sample-game-test](https://github.com/nhn/gameanvil.sample-game-test.git) | Test code using actual game server and GameHammer |

## Adding GameHammer to Project

GameHammer is deployed through Maven, just like GameAnvil. GameHammer is available by adding the following to the dependencies factor of pom.xml file.

```
... 
<dependencies> 
        ... 
        <!-- GameHammer --> 
        <dependency> 
			<groupId>com.nhn.gameanvil</groupId> 
			<artifactId>gamehammer</artifactId> 
			<version>1.4.1-jdk11</version> 
		</dependency> 
        ... 
<dependencies> 
...        
```

## Create GameHammer jar File with Maven

After creating a test scenario using GameHammer, you can create a jar file for the purpose of testing on the GameAnvil console. Here, you create a jar file for uploading via Maven.

Run the command below in the directory where the pom.xml of the project you added GameHammer.

```
mvn package
```

After the command is run, the build process is output and the last message that the build was successful is confirmed. If it shows as below, it's a success.

```
[INFO] ------------------------------------------------------------------------ 
[INFO] BUILD SUCCESS 
[INFO] ------------------------------------------------------------------------ 
[INFO] Total time:  7.880 s 
[INFO] Finished at: 2023-11-13T17:49:48+09:00 
[INFO] ------------------------------------------------------------------------ 
Process finished with exit code 0
```

You can check the build files within the newly created target directory.
