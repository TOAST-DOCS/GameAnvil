## Game > GameAnvil > 서버 개발 가이드 > 토픽 사용하기



## 구독과 발행

GameAnvil은 구독-발행 모델을 지원합니다. 즉, 임의의 토픽을 구독한 대상들은 모두 발행을 통해 동일하게 메시지를 전달받을 수 있습니다. 이러한 구독과 발행에 대한 사용법은 토픽을 중심으로 이루어집니다.



### 토픽

사용자는 언제든 임의의 토픽을 구독할 수 있습니다. 또한 GameAnvil은 내부적으로 몇 가지 토픽을 기본적으로 구독하고 있습니다. 이러한 토픽은 크게 노드 토픽과 사용자 토픽으로 나뉩니다. 이를 통해 메시지는 노드 단위로 전송된 뒤 노드 내의 객체에 전달됩니다. 다음은 이러한 노드 토픽과 사용자 토픽을 이용해서 발행하는 코드의 예입니다.

```java
/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 패킷 전송
 *
 * @param nodeTopic 패킷을 수신할 대상 노드가 등록한 토픽
 * @param topic     패킷을 수신할 유저가 등록한 토픽
 * @param packet    전송할 {@link Packet}
 */
void publishToUser(String nodeTopic, String topic, Packet packet);
```


### GameAnvil 토픽

GameAnvil은 내부적으로 아래의 토픽들을 기본적으로 구독합니다. GameAnvilTopic은 절대 사용자가 임의로 구독하지 말아야 합니다.

| 토픽                         | 발행 대상                 | 구독 대상   |
| ---------------------------- |-----------------------| ----------- |
| GameAnvilTopic.GAME_NODE     | 모든 게임 노드              | GameNode    |
| GameAnvilTopic.GATEWAY_NODE  | 모든 게이트웨이 노드           | GatewayNode |
| GameAnvilTopic.SUPPORT_NODE  | 모든 서포트 노드             | SupportNode |
| GameAnvilTopic.ALL_CLIENT    | 접속 중인 모든 클라이언트        | Session     |
| GameAnvilTopic.ALL_GAME_USER | 게임 노드에 있는 모든 게임 유저 객체 | GameUser    |



### 토픽 구독 및 구독 취소

앞서 토픽은 크게 노드 토픽과 사용자 토픽으로 나뉜다고 했습니다. 노드 토픽은 모든 종류의 노드 클래스에서 구독합니다. 반면에 사용자 토픽은 노드 내부의 객체들을 위한 것이므로 IUserContext와 IRoomContext를 구현한 모든 유저와 방 클래스에서 구독 가능합니다. 노드 토픽과 사용자 토픽의 구독 및 구독 취소 방법은 다음의 예제 코드와 같이 동일합니다.

```java
// "GameUser1" 토픽을 구독합니다.
addTopic("GameUser1");

// "GameUser1" 토픽을 구독 취소합니다.
removeTopic("GameUser1");
```

아래는 토픽 사용을 위한 전체 API 목록입니다.
```java
/**
 * 토픽이 구독 상태인지 확인한다
 *
 * @param topic 확인할 토픽
 * @return 반환값이 true 이면 구독 중, false 이면 미구독
 */
boolean hasTopic(final String topic);

/**
 * 구독중인 토픽 목록을 가져온다
 *
 * @return 토픽 목록 반환
 */
Set<String> getTopics();

/**
 * 토픽을 구독한다
 *
 * @param topic 구독할 토픽
 * @return 반환값이 true 이면 구독 성공, false 이면 구독 실패
 */
boolean addTopic(String topic);

/**
 * 구독 중인 토픽을 제거한다
 *
 * @param topic 제거할 토픽
 */
void removeTopic(String topic);
```

### 클라이언트 토픽

클라이언트 토픽은 서버 내 객체가 아닌 클라이언트로 발행하기 위한 토픽입니다. 즉, 사용자는 아래의 예제 코드와 같이 클라이언트를 대상으로 토픽을 구독할 수 있습니다.

```java
@Override
public void onCreate(IUserContext userContext) {
    this.userContext = userContext;
}

@Override
public final boolean onLogin(final IPayload payload, final IPayload sessionPayload, IPayload outPayload) {
    ...

    // 구독할 topic 등록.
    final var channelId = userContext.getChannelId();
    final var serviceName = userContext.getServiceName();
    userContext.addClientTopics(Arrays.asList(channelId, serviceName));

    ...
}
```
아래는 이러한 클라이언트 토픽 사용을 위한 API 목록입니다.
```java
/**
 * 클라이언트 토픽 리스트 등록
 *
 * @param topics 등록할 토픽 리스트
 */
void addClientTopics(List<String> topics);

/**
 * 클라이언트 토픽 리스트 제거
 *
 * @param topics 제거할 토픽 리스트
 */
void removeClientTopics(List<String> topics);

/**
 * 클라이언트 토픽 리스트 반환
 *
 * @return String 의 Set 타입으로 등록된 클라이언트 토픽 리스트 반환
 */
Set<String> getClientTopics();
```

### 발행하기

사용자는 임의의 토픽으로 메시지를 발행할 수 있습니다. 이때, 해당 토픽을 구독 중인 대상은 모두 동일한 메시지를 수신하게 됩니다.

```java
/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 패킷 전송
 *
 * @param nodeTopic 패킷을 수신할 대상 노드가 등록한 토픽
 * @param topic     패킷을 수신할 유저가 등록한 토픽
 * @param packet    전송할 {@link Packet}
 */
void publishToUser(String nodeTopic, String topic, Packet packet);

/**
 * 주어진 토픽을 등록한 모든 클라이언트에게 패킷 전송
 *
 * @param topic  패킷을 수신할 대상이 등록한 토픽
 * @param packet 전송할 {@link Packet}
 */
void publishToClient(String topic, Packet packet);
```

아래는 토픽 발행을 위한 API 목록입니다.
```java
/**
 * 주어진 토픽을 등록한 모든 클라이언트에게 패킷 전송
 *
 * @param topic  패킷을 수신할 대상이 등록한 토픽
 * @param packet 전송할 {@link Packet}
 */
void publishToClient(String topic, Packet packet);

/**
 * 주어진 토픽을 등록한 모든 클라이언트에게 메시지 전송
 *
 * @param topic   메시지를 수신할 대상이 등록한 토픽
 * @param message 전송할 메시지
 */
void publishToClient(String topic, GeneratedMessage message);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 패킷 전송
 *
 * @param serviceName 패킷을 수신할 대상 노드의 서비스 이름
 * @param packet      전송할 {@link Packet}
 */
void publishToNodeWithServiceName(final String serviceName, final Packet packet);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 메시지 전송
 *
 * @param serviceName 메시지를 수신할 대상 노드의 서비스 이름
 * @param message     전송할 메시지
 */
void publishToNodeWithServiceName(final String serviceName, final GeneratedMessage message);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 패킷 전송
 *
 * @param hostId 패킷을 수신할 대상 노드의 호스트 아이디
 * @param packet 전송할 {@link Packet}
 */
void publishToNodeWithHostId(final long hostId, final Packet packet);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 메시지 전송
 *
 * @param hostId  메시지를 수신할 대상 노드의 호스트 아이디
 * @param message 전송할 메시지
 */
void publishToNodeWithHostId(final long hostId, final GeneratedMessage message);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 패킷 전송
 *
 * @param nodeId 패킷을 수신할 대상 노드 아이디
 * @param packet 전송할 {@link Packet}
 */
void publishToNodeWithNodeId(final long nodeId, final Packet packet);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 메시지 전송
 *
 * @param nodeId  메시지를 수신할 대상 노드 아이디
 * @param message 전송할 메시지
 */
void publishToNodeWithNodeId(final long nodeId, final GeneratedMessage message);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 패킷 전송
 *
 * @param serviceName 패킷을 수신할 대상 노드의 서비스 이름
 * @param channelId   패킷을 수신할 대상 노드의 채널 아이디
 * @param packet      전송할 {@link Packet}
 */
void publishToNodeWithChannelId(final String serviceName, final String channelId, final Packet packet);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 메시지 전송
 *
 * @param serviceName 메시지를 수신할 대상 노드의 서비스 이름
 * @param channelId   메시지를 수신할 대상 노드의 채널 아이디
 * @param message     전송할 메시지
 */
void publishToNodeWithChannelId(final String serviceName, final String channelId, final GeneratedMessage message);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 패킷 전송
 *
 * @param nodeTopic 패킷을 수신할 대상 노드가 등록한 토픽
 * @param packet    전송할 {@link Packet}
 */
void publishToNode(final String nodeTopic, final Packet packet);

/**
 * 주어진 노드 토픽을 등록한 모든 노드에게 메시지 전송
 *
 * @param nodeTopic 메시지를 수신할 대상 노드가 등록한 토픽
 * @param message   전송할 메시지
 */
void publishToNode(final String nodeTopic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 패킷 전송
 *
 * @param serviceName 패킷을 수신할 대상 노드의 서비스 이름
 * @param topic       패킷을 수신할 유저가 등록한 토픽
 * @param packet      전송할 {@link Packet}
 */
void publishToUserWithServiceName(final String serviceName, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 메시지 전송
 *
 * @param serviceName 메시지를 수신할 대상 노드의 서비스 이름
 * @param topic       메시지를 수신할 유저가 등록한 토픽
 * @param message     전송할 메시지
 */
void publishToUserWithServiceName(final String serviceName, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 패킷 전송
 *
 * @param hostId 패킷을 수신할 대상 노드의 호스트 아이디
 * @param topic  패킷을 수신할 유저가 등록한 토픽
 * @param packet 전송할 {@link Packet}
 */
void publishToUserWithHostId(final long hostId, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 메시지 전송
 *
 * @param hostId  메시지를 수신할 대상 노드의 호스트 아이디
 * @param topic   메시지를 수신할 유저가 등록한 토픽
 * @param message 전송할 메시지
 */
void publishToUserWithHostId(final long hostId, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 패킷 전송
 *
 * @param nodeId 패킷을 수신할 대상 노드 아이디
 * @param topic  패킷을 수신할 유저가 등록한 토픽
 * @param packet 전송할 {@link Packet}
 */
void publishToUserWithNodeId(final long nodeId, final String topic, final Packet packet);


/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 메시지 전송
 *
 * @param nodeId  메시지를 수신할 대상 노드 아이디
 * @param topic   메시지를 수신할 유저가 등록한 토픽
 * @param message 전송할 메시지
 */
void publishToUserWithNodeId(final long nodeId, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 패킷 전송
 *
 * @param serviceName 패킷을 수신할 대상 노드의 서비스 이름
 * @param channelId   패킷을 수신할 대상 노드의 채널 아이디
 * @param topic       패킷을 수신할 유저가 등록한 토픽
 * @param packet      전송할 {@link Packet}
 */
void publishToUserWithChannelId(final String serviceName, final String channelId, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 메시지 전송
 *
 * @param serviceName 메시지를 수신할 대상 노드의 서비스 이름
 * @param channelId   메시지를 수신할 대상 노드의 채널 아이디
 * @param topic       메시지를 수신할 유저가 등록한 토픽
 * @param message     전송할 메시지
 */
void publishToUserWithChannelId(final String serviceName, final String channelId, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 패킷 전송
 *
 * @param nodeTopic 패킷을 수신할 대상 노드가 등록한 토픽
 * @param topic     패킷을 수신할 유저가 등록한 토픽
 * @param packet    전송할 {@link Packet}
 */
void publishToUser(String nodeTopic, String topic, Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 유저에게 메시지 전송
 *
 * @param nodeTopic 메시지를 수신할 대상 노드가 등록한 토픽
 * @param topic     메시지를 수신할 유저가 등록한 토픽
 * @param message   전송할 메시지
 */
void publishToUser(String nodeTopic, String topic, GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 패킷 전송
 *
 * @param serviceName 패킷을 수신할 대상 노드의 서비스 이름
 * @param topic       패킷을 수신할 유저가 등록한 토픽
 * @param packet      전송할 {@link Packet}
 */
void publishToRoomWithServiceName(final String serviceName, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 메시지 전송
 *
 * @param serviceName 메시지를 수신할 대상 노드의 서비스 이름
 * @param topic       메시지를 수신할 유저가 등록한 토픽
 * @param message     전송할 메시지
 */
void publishToRoomWithServiceName(final String serviceName, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 패킷 전송
 *
 * @param hostId 패킷을 수신할 대상 노드의 호스트 아이디
 * @param topic  패킷을 수신할 유저가 등록한 토픽
 * @param packet 전송할 {@link Packet}
 */
void publishToRoomWithHostId(final long hostId, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 메시지 전송
 *
 * @param hostId  메시지를 수신할 대상 노드의 호스트 아이디
 * @param topic   메시지를 수신할 유저가 등록한 토픽
 * @param message 전송할 메시지
 */
void publishToRoomWithHostId(final long hostId, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 패킷 전송
 *
 * @param nodeId 패킷을 수신할 대상 노드 아이디
 * @param topic  패킷을 수신할 유저가 등록한 토픽
 * @param packet 전송할 {@link Packet}
 */
void publishToRoomWithNodeId(final long nodeId, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 메시지 전송
 *
 * @param nodeId  메시지를 수신할 대상 노드 아이디
 * @param topic   메시지를 수신할 유저가 등록한 토픽
 * @param message 전송할 메시지
 */
void publishToRoomWithNodeId(final long nodeId, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 패킷 전송
 *
 * @param serviceName 패킷을 수신할 대상 노드의 서비스 이름
 * @param channelId   패킷을 수신할 대상 노드의 채널 아이디
 * @param topic       패킷을 수신할 유저가 등록한 토픽
 * @param packet      전송할 {@link Packet}
 */
void publishToRoomWithChannelId(final String serviceName, final String channelId, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 메시지 전송
 *
 * @param serviceName 메시지를 수신할 대상 노드의 서비스 이름
 * @param channelId   메시지를 수신할 대상 노드의 채널 아이디
 * @param topic       메시지를 수신할 유저가 등록한 토픽
 * @param message     전송할 메시지
 */
void publishToRoomWithChannelId(final String serviceName, final String channelId, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 패킷 전송
 *
 * @param nodeTopic 패킷을 수신할 대상 노드가 등록한 토픽
 * @param topic     패킷을 수신할 룸이 등록한 토픽
 * @param packet    전송할 {@link Packet}
 */
void publishToRoom(String nodeTopic, String topic, Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 룸에 메시지 전송
 *
 * @param nodeTopic 메시지를 수신할 대상 노드가 등록한 토픽
 * @param topic     메시지를 수신할 룸이 등록한 토픽
 * @param message   전송할 메시지
 */
void publishToRoom(String nodeTopic, String topic, GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 스팟에 패킷 전송
 *
 * @param serviceName 패킷을 수신할 대상 노드가 등록한 토픽
 * @param topic       패킷을 수신할 유저가 등록한 토픽
 * @param packet      전송할 {@link Packet}
 */
void publishToSpotWithServiceName(final String serviceName, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 스팟에 메시지 전송
 *
 * @param serviceName 메시지를 수신할 대상 노드가 등록한 토픽
 * @param topic       메시지를 수신할 유저가 등록한 토픽
 * @param message     전송할 메시지
 */
void publishToSpotWithServiceName(final String serviceName, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 스팟에 패킷 전송
 *
 * @param hostId 패킷을 수신할 대상 노드가 등록한 토픽
 * @param topic  패킷을 수신할 유저가 등록한 토픽
 * @param packet 전송할 {@link Packet}
 */
void publishToSpotWithHostId(final long hostId, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 스팟에 메시지 전송
 *
 * @param hostId  메시지를 수신할 대상 노드가 등록한 토픽
 * @param topic   메시지를 수신할 유저가 등록한 토픽
 * @param message 전송할 메시지
 */
void publishToSpotWithHostId(final long hostId, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 스팟에 패킷 전송
 *
 * @param nodeId 패킷을 수신할 대상 노드가 등록한 토픽
 * @param topic  패킷을 수신할 유저가 등록한 토픽
 * @param packet 전송할 {@link Packet}
 */
void publishToSpotWithNodeId(final long nodeId, final String topic, final Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 스팟에 메시지 전송
 *
 * @param nodeId  메시지를 수신할 대상 노드가 등록한 토픽
 * @param topic   메시지를 수신할 유저가 등록한 토픽
 * @param message 전송할 메시지
 */
void publishToSpotWithNodeId(final long nodeId, final String topic, final GeneratedMessage message);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 스팟에 패킷 전송
 *
 * @param nodeTopic 패킷을 수신할 대상 노드가 등록한 토픽
 * @param topic     패킷을 수신할 스팟이 등록한 토픽
 * @param packet    전송할 {@link Packet}
 */
void publishToSpot(String nodeTopic, String topic, Packet packet);

/**
 * 주어진 노드 토픽과 일반 토픽을 등록한 모든 스팟에 메시지 전송
 *
 * @param nodeTopic 메시지를 수신할 대상 노드가 등록한 토픽
 * @param topic     메시지를 수신할 스팟이 등록한 토픽
 * @param message   전송할 메시지
 */
void publishToSpot(String nodeTopic, String topic, GeneratedMessage message);
```
