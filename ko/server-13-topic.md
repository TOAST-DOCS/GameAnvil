## Game > GameAnvil > 서버 개발 가이드 > 토픽 사용하기



## 1. 구독과 발행

GameAnvil은 구독-발행 모델을 지원합니다. 즉, 임의의 토픽을 구독한 대상들은 모두 발행을 통해 동일하게 메시지를 전달받을 수 있습니다. 이러한 구독과 발행에 대한 사용법은 토픽을 중심으로 이루어집니다.



### 1.1. 토픽

사용자는 언제든 임의의 토픽을 구독할 수 있습니다. 또한 GameAnvil은 내부적으로 몇 가지 토픽을 기본적으로 구독하고 있습니다. 이러한 토픽은 크게 노드 토픽과 사용자 토픽으로 나뉩니다. 이를 통해 메시지는 노드 단위로 전송 된 뒤 노드 내의 객체에 전달됩니다. 다음은 이러한 노드 토픽과 사용자 토픽을 이용해서 발행하는 코드의 예입니다.

```java
publishToUser(NodeTopic, Topic, Packet);
```



### 1.2. GameAnvil 토픽

GameAnvil은 내부적으로 아래의 토픽들을 기본적으로 구독합니다. GameAnvilTopic은 절대 사용자가 임의로 구독하지 말아야 합니다.

| 토픽                         | 발행 대상                            | 구독 대상   |
| ---------------------------- | ------------------------------------ | ----------- |
| GameAnvilTopic.GAME_NODE     | 모든 게임 노드                       | GameNode    |
| GameAnvilTopic.GATEWAY_NODE  | 모든 게이트웨이 노드                 | GatewayNode |
| GameAnvilTopic.SUPPORT_NODE  | 모든 서포트 노드                     | SupportNode |
| GameAnvilTopic.ALL_CLIENT    | 접속중인 모든 클라이언트             | Session     |
| GameAnvilTopic.ALL_GAME_USER | 게임 노드에 있는 모든 게임 유저 객체 | GameUser    |



### 1.3. 토픽 구독 및 구독 취소

앞 서 토픽은 크게 노드 토픽과 사용자 토픽으로 나뉜다고 했습니다. 노드 토픽은 BaseNode를 상속하는 모든 종류의 노드 클래스에서 구독합니다. 반면에 사용자 토픽은 노드 내부의 객체들을 위한 것이므로 BaseUser와 BaseRoom을 상속하는 모든 유저와 방 클래스에서 구독 가능합니다. 노드 토픽과 사용자 토픽의 구독 및 구독 취소 방법은 다음의 예제 코드와 같이 동일합니다.

```java
// "GameUser1" 토픽을 구독합니다.
addTopic("GameUser1");

// "GameUser1" 토픽을 구독 취소합니다.
removeTopic("GameUser1");
```

아래는 토픽 사용을 위한 전체 API 목록입니다.
```java
/**
 * 토픽이 구독 상태인지 확인
 *
 * @param topic 확인할 토픽
 * @return 해당 토픽을 구독 중이면 true를 반환
 */
boolean hasTopic(String topic)

/**
 * 구독중인 토픽 목록을 반환
 *
 * @return String 의 Set으로 토픽 목록을 반환
 */
Set<String> getTopics()

/**
 * 토픽을 구독
 *
 * @param topic 구독할 토픽
 *
 * @return 성공적으로 구독할 경우 true를 반환
 */
boolean addTopic(String topic)

/**
 * 여러 개의 토픽을 구독
 *
 * @param topics 구독할 토픽 목록
 *
 * @return 성공적으로 구독할 경우 true를 반환
 */
boolean addTopics(List<String> topics)

 /**
  * 토픽을 구독 취소
  *
  * @param topic 구독 취소할 토픽
  */
void removeTopic(String topic)

 /**
  * 여러 개의 토픽을 구독 취소
  *
  * @param topics 구독 취소할 토픽 목록
  */
void removeTopics(List<String> topics)
```



### 1.4. 클라이언트 토픽

클라이언트 토픽은 서버 내 객체가 아닌 클라이언트로 발행하기 위한 토픽입니다. 즉, 사용자는 아래의 예제 코드와 같이 클라이언트를 대상으로 토픽을 구독할 수 있습니다.

```java
@Override
public final boolean onLogin(final Payload payload, final Payload sessionPayload, Payload outPayload) throws SuspendExecution {

    ...
        
	// 해당 유저와 연결된 클라이언트에 토픽을 구독
	if (isVIP())
		addClientTopics(Arrays.asList("VIP"));
}
```
아래는 이러한 클라이언트 토픽 사용을 위한 API 목록입니다.
```java
/**
 * 여러 개의 클라이언트 토픽 목록을 구독합니다.
 *
 * @param topics 등록할 토픽 리스트 전달.
 */
public void addClientTopics(List<String> topics)

/**
 * 여러 개의 클라이언트 토픽 목록을 구독 최소 합니다.
 *
 * @param topics 구독 취소할 토픽 목록
 */
public void removeClientTopics(List<String> topics)

/**
 * 클라이언트 토픽 목록을 반환
 *
 * @return 구독중인 클라이언트 토픽 문자열을 포함한 Set을 반환
 */
public Set<String> getClientTopics() {
    return this.gameUserHelper.getClientTopics();
}
```



### 1.5. 발행하기

사용자는 임의의 토픽으로 메시지를 발행할 수 있습니다. 이 때, 해당 토픽을 구독중인 대상은 모두 동일한 메시지를 수신하게 됩니다.

```java
// 노드 토픽과 토픽을 구분하여 사용해야 합니다.
publishToUser(NodeTopic, Topic, Packet)

// 클라이언트 토픽은 노드 토픽이 필요 없습니다.
publishToClient(Topic, Packet)
```
