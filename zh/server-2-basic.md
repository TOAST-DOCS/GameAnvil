## Game > GameAnvil > 서버 개발 가이드 > 기본 개념

## 1. Node

GameAnvil 서버 구성의 가장 기본이 되는 단위는 노드입니다. 각 노드는 그 역할에 맞는 기능을 독립적으로 수행합니다.  몇 개의 노드로 어떤 역할을 수행할지는 별도로 설정 가능합니다. 노드를 자세히 설명하기에 앞서 역할별로 노드를 나누면 다음과 같습니다.

| 노드       | 필수 | 기능                                        | 콘텐츠 구현 | 네트워크 접근       |
| ---------- | ---- | ------------------------------------------- | ----------- | ------------------- |
| Gateway    | 필수 | 클라이언트 접속과 인증을 처리               | 가능        | public              |
| Game       | 필수 | 실제 게임 서버로서 콘텐츠를 처리            | 가능        | private             |
| Support    | 선택 | 필요에 따라 독립된 서비스로 구현하도록 지원 | 가능        | private 또는 public |
| Match      | 선택 | 매치 메이킹을 수행                          | 가능        | private             |
| Location   | 필수 | 유저와 룸 등의 위치 정보를 저장 및 관리     | 불가능      | private             |
| Management | 필수 | 서버 정보 취합 및 Console/Agent와 통신      | 불가능      | private             |
| Ipc        | 필수 | GameAnvil 서버의 Inter-process 통신 처리    | 불가능      | private             |

GameAnvil 사용자는 이 중에서 콘텐츠 구현이 가능한 노드들(Gateway, Game, Support, Match)에만 집중하면 됩니다. 나머지 노드는 GameAnvil 서버 엔진이 내부적으로 사용하며 사용자가 추가로 구현할 부분은 없습니다. 또한 필수 노드는 반드시 서버군에 하나 이상 존재해야 구동이 가능합니다.

이러한 노드의 계층 구조는 아래의 그림과 같은 모습입니다.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



## 2. 싱글 스레드 (Single-Threaded)

GameAnvil에서 하나의 노드는 하나의 스레드로 처리됩니다. 이것은 매우 중요합니다. 각 노드는 기본적으로 모든 처리를 비동기적으로 해야 하며 이 노드 스레드는 블로킹 없이 지속적으로 구동되는 것이 보장되어야 합니다. 이러한 구동 모델은 Vert.x나 Node.js의 그것과 매우 흡사합니다.

싱글 스레드의 가장 큰 장점은 역시 lock-free입니다. 사용자는 특수한 경우를 제외하고는 명시적으로 lock을 사용할 필요가 없고 사용해서도 안 됩니다. 앞서 설명했듯 lock을 쓰는 순간 해당 노드 스레드가 처리를 멈출 수 있습니다. 이는 해당 노드 전체의 로직이 멈추는 것을 의미하므로 GameAnvil에서 개발할 때는 절대로 노드 스레드에 대해 lock을 사용해서는 안됩니다. 그러므로 노드 스레드와 외부 스레드 사이에는 절대로 게임 관련 객체가 공유되어서는 안 됩니다. 단, 별도로 외부 스레드를 생성하고 작업을 위임하는 경우에 외부 스레드들 사이의 lock은 사용해도 괜찮습니다. 만일 외부 스레드로 작업을 위임하거나 위임한 작업에 대한 결과를 획득할 필요가 있을 때는 GameAnvil에서 제공하는 [비동기 지원 API](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api)를 사용하십시오.



## 3. 파이버 (Fiber)

파이버는 일종의 경량 유저 스레드(Lightweight User Thread)로서 GameAnvil 서버 코드의 기본 흐름 단위입니다. 앞서 살펴본 노드의 싱글 스레드는 많은 수의 세션, 유저 그리고 방 등을 동시에 효과적으로 처리하기 위해 다시 다수의 파이버로 코드 흐름이 나뉩니다. 즉, GameAnvil은 파이버 기반의 Continuation을 지원합니다.

다수의 파이버들은 스레드풀(Executor)상에서 스케줄링됩니다. 이때, 스레드풀의 크기를 1로 고정하면 바로 GameAnvil 노드의 모델이 됩니다. 즉, 노드는 다수의 파이버를 동시에 처리하기 위한 애너테이션(annotation) 스케줄러인 것입니다. 이를 그림으로 표현하면 아래와 같습니다.

![image.png](http://static.toastoven.net/prod_gameanvil/images/FiberConcept.png)

이와 같이 파이버를 사용할 때의 장점은 순차적인 코드 작성이 가능하다는 점입니다. 서버 코드는 일반적인 블로킹 코드를 작성하는 것과 매우 흡사해집니다. 별도의 콜백 처리나 완료 통보에 신경쓸 필요가 전혀 없습니다. 이런 파이버의 장점에 더해 GameAnvil 사용자는 이 파이버 단위에 대해 크게 신경 쓸 필요가 없습니다. GameAnvil 엔진단에서 모든 파이버를 관리하고 있으므로 사용자는 일반적인 싱글 스레딩 코드를 작성하듯이 개발하면 됩니다.

![image.png](http://static.toastoven.net/prod_gameanvil/images/FiberConcept2.png)

GameAnvil 서버 코드는 비동기 처리를 기반으로 합니다. 이를 위해 [비동기 지원 API](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api)를 제공합니다. 이러한 비동기 API를 사용하여  임의의 파이버 상에서 블로킹 호출을 할 경우에는 해당 파이버만 suspend(대기 상태)됩니다. 이 내용은 문서 아래에서 더 자세히 다루겠습니다.



### **[주의 사항]**

다음의 몇 가지 내용은 파이버 기반의 코드를 작성할 때 주의할 사항입니다. 사용자는 파이버 단위에 크게 신경 쓸 필요가 없지만 아래의 주의 사항만큼은 반드시 지켜야 합니다.

- **Suspend**는 파이버가 코드 실행을 다른 파이버로 양보하고 대기 상태로 들어가는 것을 의미합니다. 이를 파이버 블로킹으로 표현할 수도 있습니다.
- Suspend될 수 있는 메서드는 **throws SuspendExecution**이라는 예외 시그니처를 반드시 추가해야 합니다. 이는 해당 메서드가 파이버를 중단시킬 수도 있는 블로킹 코드가 포함되어 있음을 엔진에 알려주는 약속입니다.

```
void someSuspendableMethod() throws SuspendExecution {

// some fiber-blocking call can be here

}
```

- 만일 특수한 이유로 throws SuspendExecution 예외 시그니처를 명시할 수 없다면 **@Suspendable** 애너테이션(annotation)을 사용할 수 있습니다. 그 외의 경우에는 반드시 throws SuspendExecution 예외 시그니처를 우선해서 사용하세요.

```
@Suspendable
void someSuspendableMethod() {

// some blocking call can be here

}
```

- 당연하게도 이러한 Suspendable 메서드를 호출하는 메서드 또한 Suspendable 합니다. 예를 들어 앞서 본 someSuspendableMethod()를 호출하는 someCaller() 라는 메서드가 있다고 할 때, someCaller() 또한 반드시 Suspendable하게 선언되어야 합니다.

```
void someCaller() throws SuspendExecution {

    someSuspendableMethod();

}
```

- SuspendExecution은 엔진에서 사용하는 Quasar 라이브러리가 파이버를 처리하기 위해 사용하는 특수한 기법입니다. 이는 **실제 예외가 아니며** 파이버가 아직 Java의 표준이 아니기 때문에 Quasar 라이브러리가 쓰는 일종의 우회 기법 정도로 이해할 수 있습니다. **그러므로 절대 이 SuspendExecution 예외를 catch해서는 안됩니다!** 흔하게 발생할 수 있는 오류이자 매우 중요한 부분입니다.

```
// !! 잘못된 사용 법 !!

void someCaller() {

    try {

        someSuspendableMethod();

    } catch (SuspendExecution e) {
        // 절대 SuspendExecution을 명시적으로 catch하면 안 된다!
    }
}
```

- 하지만 전체 예외를 catch하는 것은 문제없습니다. 이 부분에 대해서는 알아서 처리해 줍니다.

```
void someCaller() throws SuspendExeuction {

    try {

        someSuspendableMethod();

    } catch (Exception e) {
        // 문제 없음
    }
}
```



## 4. 파이버 기반의 비동기 처리 (Asynchronous on fibers)

코드는 파이버 상에서 비동기로 처리될 수 있어야 합니다. 즉, 하나의 파이버에서 임의의 시간이 소요되는 I/O 호출을 했을 경우에 해당 파이버는 호출이 완료될 때까지 실행 권한을 다른 파이버에게 양보할 수 있습니다. 심지어 스레드 블로킹 호출조차도 이를 파이버 블로킹 호출로 전환하여 비동기화할 수 있도록 API를 제공합니다. 즉, 스레드 블로킹 호출은 반드시 [비동기 지원 API](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api)를 사용해서 처리해야 하며, 만일 이를 어기고 직접 호출하면 해당 스레드가 블로킹되고 그 결과 스레드 상의 모든 파이버들이 블로킹되어서 노드 전체가 멈추게 되는 결과를 초래하게 됩니다.

```
    void someProblematicMethod() {

        someThreadBlockingCall(); // 해당 파이버 뿐만 아니라 전체 스레드가 블로킹된다!

    }
```

이에 대한 자세한 설명은 구현 방법에서 비동기 지원](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/server-3-implementation/#12-api)에서 자세히 다룹니다.



## 5. 분산 서버

앞서 기본 개념에서 GameAnvil의 노드 구성은 아래의 그림과 같다고 했습니다. 즉, 하나의 프로세스는 여러 가지의 노드를 자유롭게 구성해서 구동할 수 있습니다. 단, 모든 GameAnvil 프로세스는 반드시 하나의 IPC(Inter-Process Communication) 노드가 필수적으로 포함됩니다. 이 IPC 노드는 GameAnvil 프로세스 간의 통신을 담당합니다. 사실 실제 네트워크 처리를 담당하는 로우 레벨 노드가 더 있지만 사용자는 이 부분을 통틀어 IPC 노드로 이해하면 됩니다.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)

### 5-1. 노드간 통신

이러한 IPC 노드를 통해 두 개 이상의 GameAnvil 프로세스가 통신하는 모습은 아래의 그림과 같습니다. 이 그림에서 두 개의 GameAnvil 프로세스는 서로 다른 구성의 노드들을 구동합니다. 이때, 각각의 노드들은 서로 통신이 가능합니다.

서로 다른 프로세스의 노드들과 통신하기 위해 각 노드들은 IPC 노드를 통해 메시지를 전달합니다. 반면에 동일한 프로세스 상의 노드들은 큐를 이용해서 상호 통신합니다. 그러므로 이 경우에는 IPC 노드를 통하지 않습니다.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/IPC.png)

### 5-2. Meetpoint

그럼, 이러한 IPC를 위해 프로세스는 상호 접속을 어떻게 하는 걸까요? 그 답은 Meetpoint입니다. GameAnvil은 Meetpoint 주소를 설정할 수 있습니다. 아래와 같은 형태인데 하나 이상의 IP 주소 쌍을 설정할 수 있습니다. GameAnvil 프로세스는 최초 구동 시에 설정된 Meetpoint 주소 중 하나에 대해 접속을 시도하여 전체 서버군 정보를 동기화합니다.

```
"common": {

    "meetEndPoints": [
      "10.1.2.1:16000",
      "10.1.2.2:16000",
    ],

}
```



## 6. 커넥션 (Connection)과 세션 (Session)

클라이언트는 Gateway 노드로 커넥션(Connection)을 생성합니다. 이 커넥션을 통해 계정과 유저 정보를 바탕으로 인증과 로그인을 진행할 수 있습니다. 로그인까지 완료되면 임의의 Game 노드에 유저 객체가 생성됩니다. 이는 Gateway 노드와 해당 Game 노드 사이에는 논리적인 세션이 생성되었음을 의미합니다. 이렇게 커넥션과 세션 생성이 완료되면 해당 유저는 게임 진행이 가능해집니다.

### 6-1. Connection Recovery

만일, 클라이언트와 Gateway 노드 사이의 커넥션이 끊기면 아래의 그림과 같이 커넥션 복구(Connection Recovery)가 진행됩니다. 재접속을 하는 과정에서 클라이언트는 여러 대의 Gateway 노드 중 이전과 다른 곳에 커넥션을 시도할 수도 있습니다. 이 경우 유저 객체가 존재하는 Game 노드에 대한 위치 정보를 바탕으로 새롭게 세션을 복구합니다. 그러므로 유저는 게임 진행 중에 재접속을 하더라도 이전의 게임 상태를 이어갈 수 있습니다.

![Node Layer.png](http://static.toastoven.net/prod_gameanvil/images/ConnectionRecovery.png)

### 6-2. Location Node

앞서 살펴본 커넥션 복구(Connection Recovery) 그림에서 Location 노드가 보입니다. 이 Location 노드는 GameAnvil이 내부적으로 유저와 방 등의 위치 정보를 관리하는 용도로 사용합니다. 사용자는 Location 노드에 대해 직접 구현하거나 사용할 수 없습니다. 하지만 위치 정보를 관리하는 Location 노드의 존재를 알아야 전체적인 GameAnvil 시스템의 흐름을 이해할 수 있기 때문에 여기에서 간단하게 언급하고자 합니다.

위의 커넥션 복구(Connection Recovery)를 예로 들어 보겠습니다. 클라이언트가 최초 접속을 하여 Game 노드로 로그인을 시도하는 과정에서 이와 관련된 세션과 유저에 대한 위치 정보는 모두 Location 노드에 저장됩니다. 그러므로 재접속을 진행할 경우에는 이전 접속 과정에서 저장해 두었던 이 위치 정보를 조회할 수 있습니다. 이러한 위치 정보는 GameAnvil 내부적으로 유저나 방의 위치 정보를 조회하고 이를 바탕으로 메시지를 전달하는 용도 등으로 매우 중요하게 사용됩니다.



## 7. 핵심 라이브러리

아래의 4가지가 GameAnvil에서 사용하는 핵심 라이브러리입니다. Qusar와 ZeroMQ 그리고 Netty는 엔진 내부에서 사용하므로 GameAnvil 사용자가 직접 사용할 일은 없습니다. Protocol Buffers는 메시지를 직렬화/역직렬화하는 과정에서 사용합니다. 직접 사용 여부와 관계없이 아래의 4가지 라이브러리를 잘 이해하고 있다면 엔진 사용에 많은 도움이 될 것입니다.

| 라이브러리       | 용도                           |
| ---------------- | ------------------------------ |
| Quasar           | 파이버 기반의 Continuation 지원 |
| ZeroMQ           | 서버의 IPC                     |
| Netty            | 서버-클라이언트 통신           |
| Protocol Buffers | 서버-클라이언트 메시지 직렬화  |



## 8. ByteCode Instrumentation

GameAnvil은 파이버 기반의 서버 엔진입니다. 이를 위해 Quasar 라이브러리를 사용합니다.  파이버 기반의 비동기 처리를 위해서 미리 약속된 특수한 예외를 사용합니다. 혹은 @Suspendable 애너테이션(annotation)을 사용할 수도 있습니다.

```
throws SuspendExecution
```

미리 약속된 이런 코드를 해석하기 위해 GameAnvil 서버 코드는 반드시 Quasar 라이브러리를 이용하여 ByteCode Instrumentation을 진행해야 합니다. ByteCode Instrumentation은 두 가지 방식 중 한 가지를 이용해서 진행할 수 있습니다.

### 8-1. Runtime Instrumentation

서버 실행 VM 옵션의 가장 앞 부분에 아래와 같이 Quasar 바이너리를 javaagent로 추가합니다. 이렇게만 하면 런타임에 ByteCode Instrumentaion을 진행합니다.

```
-javaagent:MY_PATH\quasar-core-0.7.10-jdk8.jar=bm
```

### Note

이 항목은 반드시 VM 옵션의 가장 앞 부분에 추가해야 합니다. 이때, quasar-core의 경로는 본인의 quasar-core를 복사해둔 경로로 설정하세요.

### 8-2. AOT Instrumentation

AOT(ahead-of-time) Instrumentation을 진행하고 싶다면 아래의 내용을 프로젝트 객체 관리 파일(pom.xml)에 추가한 후, Maven을 통해 서버 바이너리를 package나 install 혹은 deploy하면 컴파일 완료 후 Instrumentation을 진행합니다. 이 경우에는 당연히 첫째 경우처럼 VM 옵션에서 javaagent가 필요하지 않습니다.

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>

    <executions>
        <execution>
            <id>instrument-classes</id>
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
