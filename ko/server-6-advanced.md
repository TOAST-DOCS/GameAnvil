## Game > GameAnvil > 서버 개발 가이드 > 고급 주제

## 1.ByteCode Instrumentation

GameAnvil은 Fiber 기반의 서버 엔진입니다. 이를 위해 Quasar 라이브러리를 사용합니다.  비동기 지원에서 설명하였듯이 Fiber 기반의 비동기 처리를 위해서 미리 약속된 특수한 예외를 사용합니다. 혹은 @Suspendable 어노테이션을 사용할 수도 있습니다.

```
throws SuspendExecution
```

미리 약속된 이런 코드를 해석하기 위해 GameAnvil 서버 코드는 반드시 Quasar 라이브러리를 이용하여 ByteCode Instrumentation을 진행해야 합니다. ByteCode Instrumentation은 두 가지 방식 중 한 가지를 이용해서 진행할 수 있습니다.

### 1-1.Runtime Instrumentation

서버 실행 VM 옵션의 가장 앞 부분에 아래와 같이 Quasar 바이너리를 javaagent로 추가합니다. 이렇게만 해주면 런타임에 ByteCode Instrumentaion을 진행합니다. 

```
-javaagent:MY_PATH\quasar-core-0.7.10-jdk8.jar=bm
```

### Note

반드시 VM 옵션의 가장 앞 부분에 추가해야 한다. 그리고 아래의 두 가지 flag 옵션은 뒤에서 따로 설명하도록 한다.

이 때, quasar-core의 경로는 본인의 quasar-core를 복사해둔 경로로 설정하면 된다.

### 1-2.AOT Instrumentation

AOT(Ahead-Of-Time) Instrumentation을 진행하고 싶다면 아래의 내용을 프로젝트 객체 관리 파일(pom.xml)에 추가한 후, Maven을 통해 서버 바이너리를 package나 install 혹은 deploy 하면 컴파일 완료 후 Instrumentation을 진행한다. 이 경우에는 당연히 첫째 경우처럼 VM 옵션에서 javaagent가 필요치 않다.

```xml
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

<br>

## 2.RoomTranfer

방 전송은 중급 개념에서 설명한 유저 전송과 매우 비슷한 기능입니다. 유저 보다 큰 개념인 방이 전송되는 차이가 있을 뿐입니다. 게임 노드에는 한 명 이상의 게임 유저가 참여한 방 객체가 생성될 수 있습니다. 이 방 객체는 유저 객체와 마찬가지로 게임 노드 사이에서 전송될 수 있습니다. 예를 들면 1번 게임 노드의 방 객체가 2번 게임 노드로 전송이 되는 것이죠. 이렇게 방이 전송될 때 당연하게도 방 안에 존재하는 유저들도 함께 전송이 일어납니다. 그 덕분에 방이 전송되기 전/후의 게임 흐름은 연속되어 진행될 수 있습니다. 즉, 해당 방 객체가 전송되는 과정은 매우 빠르게 진행되므로 방 안의 유저들은 인지하지 못한 상태에서 게임을 지속할 수 있습니다.  이 기술은 GameAnvil 무점검 패치(NonStopPatch)의 핵심이자 근간이 되며 방 객체를 잘 직렬화/역직렬화 해주어야 의도한대로 동작합니다. 이러한 직렬화/역직렬화 대상은 GameAnvil 사용자가 직접 구현할 수 있습니다. 즉, 해당 방 객체의 어떤 값을 전송할 것인지 결정하는 것이죠.

이러한 방 전송을 발생시키는 것은 오직 무점검 패치(NonStopPatch) 명령 뿐입니다. 이 명령은 일반적으로 GameAnvil Console을 통해 게임 운영자가 명시적으로 전달합니다.

<br>

### 2-1.  방 전송 구현

실제 방 전송은 GameAnvil이 내부적으로 조용하게 처리합니다. 이 때, 클라이언트는 자신의 게임 유저와 더불어 자신이 속한 방 객체가 서버 사이에서 전송되는지 인지하지 못할 가능성이 높습니다. 특별한 문제가 발생하지 않는 한 전체 흐름이 매우 빠르게 진행되기 때문에 전송 전의 게임 흐름을 전송 후에 계속 이어감에 있어 무리가 없습니다.

이 때, 다른 게임 노드로 방 객체를 전송할 때 어떤 데이터를 함께 전송할지는 사용자가 지정할 수 있습니다. 간단히 지정만 해두면 이 데이터는 방 객체의 일부로 함께 직렬화 됩니다. 전송할 대상 게임 노드에서는 역직렬화를 신경쓸 필요 없이 쉽게 해당 객체들에 접근해서 사용할 수 있습니다.

다음은 이러한 유저 전송에 관련된 콜백 메소드들입니다. 우선 "함께 전송할 데이터 지정하기" 입니다. 이를 위해 BaseRoom 추상 클래스를 상속받은 게임 룸 클래스에 아래의 콜백을 구현합니다. 방법은 간단합니다. 아래의 예제와 같이 전송할 데이터를 사용자가 원하는 key값을 이용하여 key-value 쌍으로 매개변수로 넘겨받은 transferPack에 넣기만 하면 됩니다. 

만일 여기에서 전송할 데이터를 지정하지 않으면 대상 게임 노드로 전송된 방 객체의 해당 데이터는 모두 기본값으로 초기화 되므로 주의가 필요 합니다.



**참고**> *앞서 설명하였듯이 방이 전송될 때는 당연하게도 방 안의 유저들이 함께 전송됩니다. 유저 전송에 관해서는 중급 개념에서 설명했으므로 여기에서는 따로 설명하지 않습니다.*



```java
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("RoomInfo", myRoomInfo);
}
```
이제 방 전송이 완료된 후 대상 게임 노드에서 처리할 콜백 메소드를 살펴보겠습니다. 아래와 같이 전송 전에 지정한 key를 이용해서 원하는 객체에 접근할 수 있습니다. 이와 같은 방법으로 전송 완료된 방 객체의 해당 데이터를 원래 상태로 만들어 줍니다. 특히, 전송된 방 안의 유저 객체 리스트를 매개 변수로 전달받고 있음을 확인할 수 있습니다. 이 부분만 제외하면 전체적인 흐름은 유저 전송과 매우 흡사합니다.


```java
@Override
public void onTransferIn(List<GameUser> userList, TransferPack transferPack) throws SuspendExecution {
    this.users.clear();
    for (GameUser user : userList)
        this.users.put(user.getUserId(), user);
    
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setRoomInfo(transferPack.getToLong("RoomInfo"));
}
```

<br>

### 2-2.  전송 가능한 방 타이머

방이 전송될 때 방에 등록해둔 타이머도 함께 전송할 수 있습니다. 전송 가능한 타이머를 사용하기 위해서는 별도로 아래의 콜백 메소드에서 원하는 key를 이용해서 미리 등록해두어야 합니다. Timer 핸들러를 다음과 같이 원하는 key로 매핑해둡니다. 이는 유저 전송과 완전히 동일합니다.

```java
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferRoomTimerHandler1", transferRoomTimerHandler1());
     registerTimerHandler("transferRoomTimerHandler2", (timerObject, object) -> {
         GameRoom room = (GameRoom) object;
         logger.warn("GameRoom::transferRoomTimerHandler2() : roomId({})", room.getId());
     });
 }
```

등록이 완료된 타이머 객체는 언제든 해당 key를 이용해서 아래와 같이 게임 룸 구현부에서 사용할 수 있습니다. 단순히 등록만 한 타이머는 효과가 없으므로 실제 사용을 위해서는 반드시 아래와 같이 유저 객체에 추가해주어야 타이머가 발동합니다. 

```java
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

이렇게 방 객체에 추가해 둔 타이머는 별도로 전송에 대한 처리를 하지 않아도 모두 자동으로 전송됩니다. 즉, 대상 게임 노드에서 해당 방 객체에 대해 동일한 타이머 추가 과정을 거칠 필요가 없습니다.

<br>

## 3.NonStopPatch

일반적으로 점검을 할 때에는 게임 서비스 전체에 대해 명시적으로 점검을 걸고 유저들의 게임 플레이를 차단합니다. 그 후 필요한 패치를 진행하죠. 하지만 GameAnvil을 사용하면 서버 바이너리의 호환성이 깨지는 경우가 아닌 이상 서비스를 멈출 필요 없이 패치를 진행할 수 있습니다. 이를 무점검 패치라고 하며 기본적으로 GameAnvil Console 운영 도구를 이용해서 진행할 수 있습니다. 



무점검 패치의 핵심은 앞서 설명했던 유저 전송과 방 전송 기술입니다. 이 두 기능을 기반으로 패치를 진행할 게임 서버의 유저와 방을 모두 유효한 타 게임 서버로 전송시킨 후 게임 서버가 빈 상태가 되었을 때 패치를 진행하는 것이죠. 이러한 무점검 패치는 서비스 과정에서 진행할 부분이므로 반드시 서비스 전에 선 테스트를 진행하여 문제가 없는지 확인하는 과정을 거치시기 바랍니다. 또한 패치할 서버 바이너리와 이전 서버 바이너리 사이의 호환성이 깨지지 않는지 반드시 체크해보아야 합니다.



무점검 패치를 이용하기 위해서 사용자는 GameNode 클래스에 관련 콜백 메소드들을 재정의 해야 합니다.

```java
    /**
     * 무정지 점검 시작 시 해당 node 가 출발지 node 일 경우 호출되는 callback.
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean onNonStopPatchSrcStart() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검이 종료되면 해당 node 가 출발지 node 일 경우 호출되는 callback.
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검이 시작되고 해당 node 가 출발지 node 일 경우 무정지 점검을 끝낼지 확인하기 위해 호출되는 callback.
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검 시작 시 해당 node 가 도착지 node 일 경우 호출되는 callback.
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean onNonStopPatchDstStart() throws SuspendExecution {
    return true;
}

    /**
     * 무정지 점검이 종료되면 해당 node 가 도착지 node 일 경우 호출되는 callback.
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean onNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}

	/**
     * 무정지 점검이 시작되고 해당 node 가 도착지 node 일 경우 무정지 점검을 끝낼지 확인하기 위해 호출되는 callback.
     *
     * @return boolean type 으로 성공 여부 반환.
     * @throws SuspendExecution 이 메서드는 파이버가 suspend 될 수 있다.
     */
@Override
public boolean canNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
```

<br>

### 3-1. 무점검 패치 가이드

무점검 패치는 GameAnvil의 여러 요소가 복합적으로 잘 맞물려 돌아갔을 때 좋은 결과가 나옵니다. 그러므로 반드시 아래의 가이드 문서를 읽어보고 모든 내용을 이해한 후에 사용할 수 있도록 하세요.

#### [무점검 패치 가이드 ](nonstop-patch)


