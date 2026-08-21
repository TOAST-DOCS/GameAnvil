<!-- pre-align:aligned sig=3180aad95516 -->

<a id="game-gameanvil-guide-to-test-development-get-started"></a>
## Game > GameAnvil > Guide to Test Development > Get Started { #game-gameanvil-guide-to-test-development-get-started }

<a id="overview"></a>
### Overview { #overview }

<!-- TODO: translate body -->

<a id="supported-environment-and-protocol"></a>
### Supported Environment and Protocol { #supported-environment-and-protocol }

<a id="supported-environment-and-protocol-supported-network-protocol"></a>
#### Supported Network Protocol

* TCP/IP
* SSL over TCP/IP

<a id="supported-environment-and-protocol-available-application-protocol-format"></a>
#### Available Application Protocol Format

* Google Protocol Buffers
* Custom Byte Stream
* HTTP/HTTPS (only for specific purposes)

<a id="add-gamehammer-dependency-to-project"></a>
### Add GameHammer Dependency to Project { #add-gamehammer-dependency-to-project }

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

<a id="create-gamehammer-jar-file-with-maven"></a>
### Create GameHammer jar file with Maven { #create-gamehammer-jar-file-with-maven }

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

