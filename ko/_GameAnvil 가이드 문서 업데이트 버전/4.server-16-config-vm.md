## Game > GameAnvil > 서버 개발 가이드 > 서버 구성과 구동



## 1. 구성 (Configuration)

GameAnvil은 크게 두 가지 방법으로 서버를 구성할 수 있습니다. 가장 대표적인 방법은 NHN Cloud의 콘솔을 통해 구동할 서버를 GUI 상에서 구성할 수 있습니다. 이 방법은 클라우드 상에서 VM 기반의 서비스 혹은 테스트를 위해 사용하게 될 방법입니다. 하지만 이 방법은 개발 과정에서 사용하기는 번거롭고 불편합니다. 그래서 사용자가 개발중에 PC에서 직접 서버를 구성할 수 있도록 GameAnvilConfig.json 파일을 제공합니다. 이 파일의 기본 경로는 프로젝트의 resources/ 를 사용합니다. 만일 여러개의 프로세스로 서버를 구성할 경우에는 각각의 프로세스마다 고유한 GameAnvilConfig.json이 필요합니다.

이 구성 파일에 입력하는 값 중 일부는 서버 코드 상에서 어노테이션으로 엔진에 등록할 때 사용하는 값과 일치해야 합니다. 예를 들면 앞서 우리는 아래와 같이 GameNode를 구성했습니다. 여기에 사용된 "MyGame"이라는 서비스명은 반드시 GameAnvilConfig.json에도 동일하게 설정되어 있어야 합니다. 그렇지 않으면 해당 서비스를 찾지 못하여 서버 구동 과정에서 오류가 발생합니다.

```java
@ServiceName("MyGame") // "MyGame"이라는 서비스를 위한 GameNode로 엔진에 등록
public class SampleGameNode extends BaseGameNode {
    ...
}
```

그럼, 이러한 내용을 한 번 살펴보도록 하겠습니다.



## 2. GameAnvilConfig.json 편집

GameAnvilConfig은 서버를 유연하게 구성하기 위해 매우 많은 수의 설정을 제공합니다. 하지만 대부분은 엔진 기본값으로 충분합니다. 그러므로 사용자의 이해가 필요한 설정 값들만 설명하도록 합니다. 크게 다섯 가지로 나뉩니다.



### 2.1. 공통 설정 (common)

노드 구성과 상관없이 필수적인 공통 정보를 설정합니다. 

```json
"common": {
    "ip": "127.0.0.1", // 서버 프로세스간-통신(IPC)에 사용할 IP (해당 머신의 IP를 지정)
    "meetEndPoints": ["127.0.0.1:18000"], // 클러스터링 정보를 동기화할 피어 정보 (포트는 변경하지 않았다면 18000 사용)
    "debugMode": false // 디버깅시 각종 timeout 이 발생안하도록 하는 옵션 , 리얼에서는 반드시 false 이어야 함
},
```



각 구성 항목의 설명은 다음과 같습니다.

| 이름                            | 설명                                                         | 기본값  |
| ------------------------------- | ------------------------------------------------------------ | ------- |
| ip                              | 노드마다 공통으로 사용하는 IP. (머신의 IP를 지정) 값이 없을 경우 private ip로 자동 지정된다. | - |
| meetEndPoints                   | 대상 노드의 common IP와 communicatePort 등록 해당 서버 endpoint 포함 가능 리스트로 여러 개 등록 가능 |-|
| debugMode                       | 디버깅 시 각종 timeout 이 발생안하도록 하는 옵션리얼에서는 반드시 false 이어야 한다. | false   |



### 2.2. location

사실 로케이션 노드는 전체 서버의 유저와 방 위치 정보를 담당하는 시스템 노드입니다. 엔진이 관리하며 직접 사용하는 용도이므로 사용자가 추가적인 구현을 할 필요는 없습니다. 하지만 이러한 시스템 노드도 몇 개의 노드로 구성할지는 사용자가 선택하기 나름이기 때문에 별도의 구성 방법을 제공합니다. 개발 과정에서는 아래의 사용 예시 그대로 사용해도 무방합니다. 반면에 실제 서비스를 위한 구성은 게임의 컨텐츠나 볼륨에 따라 알맞게 적용해야 하므로 GameAnvil 담당자와 별도의 논의를 진행하는 것을 추천합니다. 각 설정 항목은 아래와같습니다.

```json
"location": {
	"clusterSize": 1, // 총 몇개의 머신(VM)으로 구성되는가?
    "replicaSize": 1, // 복제 그룹의 크기 (master + slave의 개수)
    "shardFactor": 2  // sharding을 위한 인수 (아래의 주석 참고)
},
```



### Note

*해당 설정("location") 키-값이 없거나 clusterSize가 0이면 해당 프로세스에서 로케이션 노드를 생성하지 않습니다.*



각 구성 항목의 설명은 다음과 같습니다.

| 이름        | 설명                                                         | 기본값 |
| ----------- | ------------------------------------------------------------ | ------ |
| clusterSize | 구성되는 서버의 수(VM)                                       | -      |
| replicaSize | 복제 그룹의 크기 master + slave의 개수                       | -      |
| shardFactor | sharding을 위한 인수 <br />-전체 shard의 개수 = clusterSize x replicaSize x shardFactor <br />-하나의 머신(VM)에서 구동할 shard의 개수 = replicaSize x shardFactor <br />-고유한 shard의 총 개수 (master 샤드의 개수) = clusterSize x shardFactor | -      |




### 2.3. match

매치 노드는 매치메이킹을 수행하는 노드입니다. 즉, 사용자가 구현한 매치케이커를 구동합니다. 이러한 매치 노드는 몇 개의 노드를 구동할지만 결정하면 됩니다. 일반적인 개발 과정이나 작은 규모의 서비스는 하나의 매치 노드로도 충분합니다.

```json
"match": {
    "nodeCnt": 1
},
```



### Note

*해당 설정("match") 키-값이 없거나 nodeCnt가 0이면 해당 프로세스에서  매치 노드를 생성하지 않습니다.*



각 구성 항목의 설명은 다음과 같습니다.

| 이름    | 설명            | 기본값 |
| ------- | --------------- | ------ |
| nodeCnt | match node 개수 | -      |




### 2.4. gateway

게이트웨이 노드는 클라이언트가 접속을 맺는 노드입니다. 그러므로 접속할 클라이언트 규모에 맞춰 적당한 개수의 노드를 준비해야 합니다.

```json
"gateway": {
    "nodeCnt": 4, // 노드 개수. (노드 번호는 0 부터 부여 됨)
    "ip": "127.0.0.1", // 클라이언트와 연결되는 IP.
    "dns": "", // 클라이언트와 연결되는 도메인 주소.
    "connectGroup": {
        "TCP_SOCKET": {
            "port": 18200, // 클라이언트와 연결되는 포트.
            "idleClientTimeout": 240000 // 데이터 송수신이 없는 상태 이후의 타임아웃. (0 이면 사용하지 않음)        
        },
        "WEB_SOCKET": {
            "port": 18300,
            "idleClientTimeout": 240000
        }
    }
},
```



### Note

*해당 설정("gateway") 키-값이 없거나 nodeCnt가 0이면 해당 프로세스에서 게이트웨이 노드를 생성하지 않습니다.*



각 구성 항목의 설명은 다음과 같습니다.

| 이름                             | 설명                                                         | 기본값 |
| -------------------------------- | ------------------------------------------------------------ | ------ |
| nodeCnt                          | 노드 개수                                                    | -      |
| ip                               | 클라이언트가 접속할 IP 주소<br> (설정값이 없을 경우에는 해당 머신의 사설 IP로 자동 설정) | -      |
| dns                              | 클라이언트가 접속할 도메인 주소                              | -      |
| connectGroup                     | 게이트웨이 노드는 TCP 소켓과 WEB 소켓을 지원합니다           | -      |
| connectGroup : port              | 클라이언트가 접속할 포트                                     | -      |
| connectGroup : idleClientTimeout | 데이터 송수신이 없는 상태에서 접속 정리까지의 타임아웃 (0 이면 사용하지 않음) | 4000   |



### 2.5. game

게임 노드는 실제 게임 관련 객체들이 생성되고 컨텐츠가 플레이되는 노드입니다. 게임 컨텐츠의 특성에 맞게 노드 수와 채널 등을 구성할 수 있습니다.


```json
"game": [
    {
        "nodeCnt": 4, // 4개의 게임 노드는 각각 채널 "초보", "중수", "초보", "중수"에 대응하게 됩니다.
        "serviceId": 1,
        "serviceName": "Todo - Input My Service Name",
        "channelIDs": ["초보","중수","초보","중수"], // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. ""는 채널 사용하지 않음을 의미)
        "userTimeout": 5000 // disconnect 이후의 유저객체 제거 타임아웃.
    }
]
```



### Note 1

*서버 코드 상에서 사용자가 작성한 게임 노드 클래스를 엔진에 등록하기 위해 사용한 서비스 이름은 이 곳 설정에서 반드시 입력해야합니다. 다음은 게임 노드에 대한 어노테이션 예제입니다. "MyGame"이라는 서비스명을 사용했습니다.*

```java
@ServiceName("MyGame") // "MyGame"이라는 서비스를 위한 GameNode로 엔진에 등록
public class SampleGameNode extends BaseGameNode {
    ...
}
```

*그러므로 반드시 GameAnvilConfig에서 다음과 같이 "MyGame" 서비스명을 설정해주어야 합니다. 아래의 예는 "MyGame" 서비스명을 서비스 아이디 "1"에 매핑한 예제입니다.  이 서비스명과 서비스 아이디는 반드시 전체 구성에서 유일한 값이어야 합니다.*

```json
"game": [
    {
        "nodeCnt": 4, // 4개의 게임 노드는 각각 채널 "초보", "중수", "초보", "중수"에 대응하게 됩니다.
        "serviceId": 1,
        "serviceName": "MyGame", // 서비스명
        "channelIDs": ["초보","중수","초보","중수"], // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. ""는 채널 사용하지 않음을 의미)
        "userTimeout": 5000 // 클라이언트의 접속 끊김 이후 유저 객체를 서버에서 제거하지 않고 얼마동안 관리할지 설정
    },
    {
        "nodeCnt": 1,
        "serviceId": 2,
        "serviceName": "MyChatting", // 서비스명
        "channelIDs": [""], // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. ""는 채널 사용하지 않음을 의미)
        "userTimeout": 5000 // 클라이언트의 접속 끊김 이후 유저 객체를 서버에서 제거하지 않고 얼마동안 관리할지 설정
    }
]
```

*위의 구성 예제는 추가로 "MyChatting" 서비스를 보여줍니다. 서비스 아이디는 "MyGame"과 다른 "2"인 것을 볼 수 있습니다.*



### Note 2

*해당 설정("game") 키-값이 없거나 nodeCnt가 0이면 해당 프로세스에서 게임 노드를 생성하지 않습니다.*




각 구성 항목의 설명은 다음과 같습니다.

| 이름        | 설명                                                         | 기본값 |
| ----------- | ------------------------------------------------------------ | ------ |
| nodeCnt     | 노드 개수                                                    |        |
| serviceId   | 서비스 아이디로서 전체 서버 구성에서 유일한 숫자 값이어야 합니다. 단, 1 ~ 99 사이의 값만 사용 가능합니다. |        |
| serviceName | 서비스명으로 전체 서버 구성에서 유일한 문자열이어야 합니다. 이 서비스명은 게임 노드를 엔진에 등록하기 위해 어노테이션에 사용됩니다. |        |
| channelIDs  | 노드마다 부여할 채널 아이디로서 유일한 값일 필요는 없습니다. <br>단, ""는 채널을 사용하지 않음을 의미합니다. |        |
| userTimeout | 클라이언트의 접속 끊김 이후 유저 객체를 서버에서 제거하지 않고 얼마동안 관리할지 설정 | 10000  |



### 2.6. support

서포트 노드는 보조적인 역할을 수행하는 노드입니다. 클라이언트와 직접 통신도 가능하므로 게임에 관련된 정보를 교환하거나 주기적인 작업 혹은 게임 외적으로 독립된 구현이 필요한 작업 등을 위임하여 처리하기에 알맞습니다.

```json
"support": [
    {
        "nodeCnt": 2,
        "serviceId": 3,
        "serviceName": "DB",
        "restIp": "127.0.0.1",
        "restPort": 17251
    },
    {
        "nodeCnt": 2,
        "serviceId": 4,
        "serviceName": "SampleWebSupport",
        "restIp": "127.0.0.1",
        "restPort": 17250
    }
],
```



### Note 1

*앞서 살펴본 게임 노드와 마찬가지로 서버 코드 상에서 사용자가 작성한 서포트 노드 클래스를 엔진에 등록하기 위해 사용한 서비스 이름은 이 곳 설정에서 반드시 입력해야 합니다.* 



### Note 2

*해당 설정("support") 키-값이 없거나 nodeCnt가 0이면 해당 프로세스에서 서포트 노드를 생성하지 않습니다.*



각 구성 항목의 설명은 다음과 같습니다.


| 이름        | 설명                                              | 기본값 |
| ----------- | ------------------------------------------------- | ------ |
| nodeCnt     | 노드 개수                                         | - |
| serviceId   | 서비스 아이디로서 전체 서버 구성에서 유일한 숫자 값이어야 합니다. 단, 1 ~ 99 사이의 값만 사용 가능합니다. | - |
| serviceName | 서비스명으로 전체 서버 구성에서 유일한 문자열이어야 합니다. 이 서비스명은 서포트 노드를 엔진에 등록하기 위해 어노테이션에 사용됩니다. | - |
| restIp      | RESTful 요청을 위한 IP 주소<br/> (설정값이 없을 경우에는 해당 머신의 사설 IP로 자동 설정) | - |
| restPort    | RESTful 요청을 위한 포트                         |-|



## 3. VM 옵션

GameAnvil 서버의 구동을 위해 개발팀에서 사용하는 VM 옵션의 핵심 적인 부분을 공유합니다. 여기에서 권장하는 VM 옵션은 그 동안 여러차례의 대규모 성능 테스트를 통해 검증되었습니다. 이를 참고하여 사용자가 원하는대로 적절하게 변경하면서 사용하시길 바랍니다.



### 3.1. 권장 VM 옵션

가장 윗 라인의 javaagent 옵션은 파이버 코드를 JIT 방식으로 사용할 경우에만 필요합니다. 만일 AOT 방식으로 사용하는 경우에는 이 라인을 지우도록 합니다. 이러한 [Bytecode Instrumentation](know-06-bytecode-instrument)에 대해서는 뒤에서 따로 자세히 다루도록 합니다. 메모리 크기는 시스템에 맞추어 설정합니다. 참고로 개발팀은 8GB 머신에서는 4~6GB를, 16GB 머신에서는 10~12GB를 사용합니다. 그리고 GameAnvil은 G1GC를 공식 GC로 사용합니다. 그러므로 특별한 이유가 없다면 사용자도 반드시 G1GC를 사용하기 바랍니다. 마지막으로 GC 로그를 위한 최소한의 옵션을 추가하길 강력히 권장합니다. 특히, 개발 과정에서는 필수입니다.

####  Java 8 

```
-javaagent:YOUR_PATH\quasar-core-0.7.10-jdk8.jar=bm
-Xms4g
-Xmx4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication

-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:gc.log
```

#### Java 11

```
-javaagent:YOUR_PATH\quasar-core-0.8.0-jdk11.jar=bm

--add-opens=java.base/java.lang=ALL-UNNAMED
--illegal-access=deny

-Xms4g
-Xmx4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+UseStringDeduplication

-Xlog:gc*,safepoint:./log/gc.log:time,level,tags
```



### 3.2. GC 로그를 위한 VM 옵션

GC 로그를 위한 옵션은 메모리 릭 등을 추적하기 위해 필수입니다. 그러므로 특별한 이유가 없다면 최소한 개발 과정에서는 다음과 같은 GC 로그 관련 옵션들을 추가하기를 권합니다. 하지만 실제 서비스에서는 성능에 영향을 줄 수 있으므로 일부 최적화된 옵션들만 필요에 따라 추가해야할 수도 있습니다. 앞 서 권장한 옵션들과 더불어 각 Java 버전에 따라 다음과 같은 옵션들을 추가할 수 있습니다.

#### Java 8
```
-XX:+PrintGCDetails
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-XX:+PrintHeapAtGC
-XX:+PrintReferenceGC
-Xloggc:YOUR_PATH/gc.log
```
다음과 같이 파일 로테이션 옵션을 추가할 수도 있습니다.
```
-XX:+PrintGCDetails
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-XX:+PrintHeapAtGC
-XX:+PrintReferenceGC
-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=100
-XX:GCLogFileSize=10M
-Xloggc:YOUR_PATH/gc.log
```

#### Java 11
```
-Xlog:gc*,safepoint:YOUR_PATH/gc.log:time,level,tags,uptime
```
다음과 같이 파일 로테이션 옵션을 추가할 수도 있습니다.
```
-Xlog:gc*,safepoint:YOUR_PATH/gc.log:time,level,tags,uptime:filecount=100,filesize=10M
```

