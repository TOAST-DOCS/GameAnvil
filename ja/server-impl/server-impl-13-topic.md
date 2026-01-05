## Game > GameAnvil > サーバー開発ガイド > トピックの使用



## 購読と発行

GameAnvilは購読-発行モデルをサポートします。つまり、任意のトピックを購読した対象は、全て発行を通じて同様にメッセージを受け取ることができます。このような購読と発行に関する使用法は、トピックを中心に行われます。



### トピック

ユーザーはいつでも任意のトピックを購読できます。また、GameAnvilは内部的にいくつかのトピックをデフォルトで購読しています。これらのトピックは、大きくノードトピックとユーザートピックに分かれます。これを通じてメッセージはノード単位で転送された後、ノード内のオブジェクトに伝達されます。次は、このようなノードトピックとユーザートピックを利用して発行するコードの例です。

```java
/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへパケット転送
 *
 * @param nodeTopic パケットを受信する対象ノードが登録したトピック
 * @param topic      パケットを受信するユーザーが登録したトピック
 * @param packet     転送する{@link Packet}
 */
void publishToUser(String nodeTopic, String topic, Packet packet);
```


### GameAnvilトピック

GameAnvilは内部的に以下のトピックをデフォルトで購読します。GameAnvilTopicは、ユーザーが絶対に任意で購読してはいけません。

| トピック                          | 発行対象               | 購読対象     |
|-------------------------------|-----------------------|-------------|
| GameAnvilTopic.GAME_NODE      | 全てのゲームノード            | GameNode    |
| GameAnvilTopic.GATEWAY_NODE   | 全てのゲートウェイノード         | GatewayNode |
| GameAnvilTopic.SUPPORT_NODE   | 全てのサポートノード           | SupportNode |
| GameAnvilTopic.ALL_CLIENT      | 接続中の全てのクライアント         | Session     |
| GameAnvilTopic.ALL_GAME_USER   | ゲームノードにある全てのゲームユーザーオブジェクト | GameUser    |



### トピック購読及び購読解除

前述のとおり、トピックは大きくノードトピックとユーザートピックに分かれます。ノードトピックは、全ての種類のノードクラスで購読します。一方、ユーザートピックはノード内部のオブジェクトのためのものなので、IUserContextとIRoomContextを実装した全てのユーザーとルームクラスで購読可能です。ノードトピックとユーザートピックの購読及び購読解除方法は、次のサンプルコードと同様です。

```java
// "GameUser1" トピックを購読します。
addTopic("GameUser1");

// "GameUser1" トピックの購読を解除します。
removeTopic("GameUser1");
```

以下はトピック使用のための全APIリストです。
```java
/**
 * トピックが購読状態かを確認する
 *
 * @param topic 確認するトピック
 * @return 戻り値がtrueなら購読中、falseなら未購読
 */
boolean hasTopic(final String topic);

/**
 * 購読中のトピックリストを取得する
 *
 * @return トピックリストを返却
 */
Set<String> getTopics();

/**
 * トピックを購読する
 *
 * @param topic 購読するトピック
 * @return 戻り値がtrueなら購読成功、falseなら購読失敗
 */
boolean addTopic(String topic);

/**
 * 購読中のトピックを削除する
 *
 * @param topic 削除するトピック
 */
void removeTopic(String topic);
```

### 発行

ユーザーは任意のトピックへメッセージを発行できます。このとき、該当トピックを購読している対象は、全て同一のメッセージを受信することになります。


以下はトピック発行のためのAPIリストです。
```java
/**
 * 全てのクライアントへパケット転送
 *
 * @param packet 転送する{@link Packet}
 */
void publishToAllClient(Packet packet);

/**
 * 全てのクライアントへメッセージ転送
 *
 * @param message 転送するメッセージ
 */
void publishToAllClient(GeneratedMessage message);

/**
 * 指定されたノードトピックを登録した全てのノードへパケット転送
 *
 * @param serviceName パケットを受信する対象ノードのサービス名
 * @param packet      転送する{@link Packet}
 */
void publishToNodeWithServiceName(final String serviceName, final Packet packet);

/**
 * 指定されたノードトピックを登録した全てのノードへメッセージ転送
 *
 * @param serviceName メッセージを受信する対象ノードのサービス名
 * @param message     転送するメッセージ
 */
void publishToNodeWithServiceName(final String serviceName, final GeneratedMessage message);

/**
 * 指定されたノードトピックを登録した全てのノードへパケット転送
 *
 * @param hostId パケットを受信する対象ノードのホストID
 * @param packet 転送する{@link Packet}
 */
void publishToNodeWithHostId(final long hostId, final Packet packet);

/**
 * 指定されたノードトピックを登録した全てのノードへメッセージ転送
 *
 * @param hostId  メッセージを受信する対象ノードのホストID
 * @param message 転送するメッセージ
 */
void publishToNodeWithHostId(final long hostId, final GeneratedMessage message);

/**
 * 指定されたノードトピックを登録した全てのノードへパケット転送
 *
 * @param nodeId パケットを受信する対象ノードID
 * @param packet 転送する{@link Packet}
 */
void publishToNodeWithNodeId(final long nodeId, final Packet packet);

/**
 * 指定されたノードトピックを登録した全てのノードへメッセージ転送
 *
 * @param nodeId  メッセージを受信する対象ノードID
 * @param message 転送するメッセージ
 */
void publishToNodeWithNodeId(final long nodeId, final GeneratedMessage message);

/**
 * 指定されたノードトピックを登録した全てのノードへパケット転送
 *
 * @param serviceName パケットを受信する対象ノードのサービス名
 * @param channelId   パケットを受信する対象ノードのチャンネルID
 * @param packet      転送する{@link Packet}
 */
void publishToNodeWithChannelId(final String serviceName, final String channelId, final Packet packet);

/**
 * 指定されたノードトピックを登録した全てのノードへメッセージ転送
 *
 * @param serviceName メッセージを受信する対象ノードのサービス名
 * @param channelId   メッセージを受信する対象ノードのチャンネルID
 * @param message     転送するメッセージ
 */
void publishToNodeWithChannelId(final String serviceName, final String channelId, final GeneratedMessage message);

/**
 * 指定されたノードトピックを登録した全てのノードへパケット転送
 *
 * @param nodeTopic パケットを受信する対象ノードが登録したトピック
 * @param packet     転送する{@link Packet}
 */
void publishToNode(final String nodeTopic, final Packet packet);

/**
 * 指定されたノードトピックを登録した全てのノードへメッセージ転送
 *
 * @param nodeTopic メッセージを受信する対象ノードが登録したトピック
 * @param message    転送するメッセージ
 */
void publishToNode(final String nodeTopic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへパケット転送
 *
 * @param serviceName パケットを受信する対象ノードのサービス名
 * @param topic        パケットを受信するユーザーが登録したトピック
 * @param packet      転送する{@link Packet}
 */
void publishToUserWithServiceName(final String serviceName, final String topic, final Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへメッセージ転送
 *
 * @param serviceName メッセージを受信する対象ノードのサービス名
 * @param topic        メッセージを受信するユーザーが登録したトピック
 * @param message     転送するメッセージ
 */
void publishToUserWithServiceName(final String serviceName, final String topic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへパケット転送
 *
 * @param hostId パケットを受信する対象ノードのホストID
 * @param topic  パケットを受信するユーザーが登録したトピック
 * @param packet 転送する{@link Packet}
 */
void publishToUserWithHostId(final long hostId, final String topic, final Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへメッセージ転送
 *
 * @param hostId  メッセージを受信する対象ノードのホストID
 * @param topic   メッセージを受信するユーザーが登録したトピック
 * @param message 転送するメッセージ
 */
void publishToUserWithHostId(final long hostId, final String topic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへパケット転送
 *
 * @param nodeId パケットを受信する対象ノードID
 * @param topic  パケットを受信するユーザーが登録したトピック
 * @param packet 転送する{@link Packet}
 */
void publishToUserWithNodeId(final long nodeId, final String topic, final Packet packet);


/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへメッセージ転送
 *
 * @param nodeId  メッセージを受信する対象ノードID
 * @param topic   メッセージを受信するユーザーが登録したトピック
 * @param message 転送するメッセージ
 */
void publishToUserWithNodeId(final long nodeId, final String topic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへパケット転送
 *
 * @param serviceName パケットを受信する対象ノードのサービス名
 * @param channelId   パケットを受信する対象ノードのチャンネルID
 * @param topic        パケットを受信するユーザーが登録したトピック
 * @param packet      転送する{@link Packet}
 */
void publishToUserWithChannelId(final String serviceName, final String channelId, final String topic, final Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへメッセージ転送
 *
 * @param serviceName メッセージを受信する対象ノードのサービス名
 * @param channelId   メッセージを受信する対象ノードのチャンネルID
 * @param topic        メッセージを受信するユーザーが登録したトピック
 * @param message     転送するメッセージ
 */
void publishToUserWithChannelId(final String serviceName, final String channelId, final String topic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへパケット転送
 *
 * @param nodeTopic パケットを受信する対象ノードが登録したトピック
 * @param topic      パケットを受信するユーザーが登録したトピック
 * @param packet     転送する{@link Packet}
 */
void publishToUser(String nodeTopic, String topic, Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのユーザーへメッセージ転送
 *
 * @param nodeTopic メッセージを受信する対象ノードが登録したトピック
 * @param topic      メッセージを受信するユーザーが登録したトピック
 * @param message    転送するメッセージ
 */
void publishToUser(String nodeTopic, String topic, GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへパケット転送
 *
 * @param serviceName パケットを受信する対象ノードのサービス名
 * @param topic        パケットを受信するユーザーが登録したトピック
 * @param packet      転送する{@link Packet}
 */
void publishToRoomWithServiceName(final String serviceName, final String topic, final Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへメッセージ転送
 *
 * @param serviceName メッセージを受信する対象ノードのサービス名
 * @param topic        メッセージを受信するユーザーが登録したトピック
 * @param message     転送するメッセージ
 */
void publishToRoomWithServiceName(final String serviceName, final String topic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへパケット転送
 *
 * @param hostId パケットを受信する対象ノードのホストID
 * @param topic  パケットを受信するユーザーが登録したトピック
 * @param packet 転送する{@link Packet}
 */
void publishToRoomWithHostId(final long hostId, final String topic, final Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへメッセージ転送
 *
 * @param hostId  メッセージを受信する対象ノードのホストID
 * @param topic   メッセージを受信するユーザーが登録したトピック
 * @param message 転送するメッセージ
 */
void publishToRoomWithHostId(final long hostId, final String topic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへパケット転送
 *
 * @param nodeId パケットを受信する対象ノードID
 * @param topic  パケットを受信するユーザーが登録したトピック
 * @param packet 転送する{@link Packet}
 */
void publishToRoomWithNodeId(final long nodeId, final String topic, final Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへメッセージ転送
 *
 * @param nodeId  メッセージを受信する対象ノードID
 * @param topic   メッセージを受信するユーザーが登録したトピック
 * @param message 転送するメッセージ
 */
void publishToRoomWithNodeId(final long nodeId, final String topic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへパケット転送
 *
 * @param serviceName パケットを受信する対象ノードのサービス名
 * @param channelId   パケットを受信する対象ノードのチャンネルID
 * @param topic        パケットを受信するユーザーが登録したトピック
 * @param packet      転送する{@link Packet}
 */
void publishToRoomWithChannelId(final String serviceName, final String channelId, final String topic, final Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへメッセージ転送
 *
 * @param serviceName メッセージを受信する対象ノードのサービス名
 * @param channelId   メッセージを受信する対象ノードのチャンネルID
 * @param topic        メッセージを受信するユーザーが登録したトピック
 * @param message     転送するメッセージ
 */
void publishToRoomWithChannelId(final String serviceName, final String channelId, final String topic, final GeneratedMessage message);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへパケット転送
 *
 * @param nodeTopic パケットを受信する対象ノードが登録したトピック
 * @param topic      パケットを受信するルームが登録したトピック
 * @param packet     転送する{@link Packet}
 */
void publishToRoom(String nodeTopic, String topic, Packet packet);

/**
 * 指定されたノードトピックと一般トピックを登録した全てのルームへメッセージ転送
 *
 * @param nodeTopic メッセージを受信する対象ノードが登録したトピック
 * @param topic      メッセージを受信するルームが登録したトピック
 * @param message    転送するメッセージ
 */
void publishToRoom(String nodeTopic, String topic, GeneratedMessage message);
```
