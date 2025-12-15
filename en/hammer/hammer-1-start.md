## Game > GameAnvil > Guide to Test Development > Get Started

### Number

GameHammer is a performance and feature test tool that can be used after developing a game server using the GameAnvil engine. It can be tested using all the features provided by the actual connector in the same way, and can be used as running multiple GameHammers at the same time for stress tests, etc. You can check and save the progress during the test process or take the test final result together.

* Support all functions in the connector the same
* Support both Sync/Async methods
    * Provide API in the Async Method
    * Provide Future for the Sync method
* Allow you to use more than thousands of connections at the same time
* Support status-based scenario management

This guide provides how to use GameHammer with detailed examples. It describes the use based on IntelliJ, like the server engine.

### Supported Environment and Protocol

#### Supported Network Protocol

* TCP/IP
* SSL over TCP/IP

#### Available Application Protocol Format

* Google Protocol Buffers
* Custom Byte Stream
* HTTP/HTTPS (only for specific purposes)

### Add GameHammer Dependency to Project

GameHammer is deployed via Maven, like GameAnvil. You can use GameHammer by adding the dependencies element in the pom.xml file as follows:

```pom
<dependencies>
       <!-- GameHammer -->
       <dependency>
			<groupId>com.nhn.gameanvil</groupId>
			<artifactId>gamehammer</artifactId>
			<version>2.1.0-jdk11</version>
		</dependency>
<dependencies>
```

### Create GameHammer jar file with Maven

You can use GameHammer to create a jar file for the purpose of testing in the GameAnvil console after writing a test scenario.

Run the following command from the directory with pom.xml of the project with which GameHammer has been added:

```
mvn package
```

After running the command, check the message to ensure that the build process is outgoing and that the build has been successful at the end. If you see the message below, it is a success:

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.880 s
[INFO] Finished at: 2023-11-13T17:49:48+09:00
[INFO] ------------------------------------------------------------------------
Process finished with exit code 0
```

You can see the built files in the newly created target directory.

### How to Remove Unnecessary Debug Log

When GameHammer is executed, DEBUG level logs related to the internal Library may occur. There is no exception to the action, but if you want to remove these logs, add the following to the VMOption.

```
-Dio.netty.tryReflectionSetAccessible=true
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/jdk.internal.misc=ALL-UNNAMED