## Game > GameAnvil > サーバー開発ガイド > トピックを使用する



## 購読と発行

GameAnvilは購読-発行モデルをサポートします。つまり、任意のトピックを購読した対象はすべて、発行を通じて同じようにメッセージを受け取ることができます。これらの購読と発行の使用方法は、トピックを中心に成り立ちます。



### トピック

ユーザーはいつでも任意のトピックを購読できます。また、GameAnvilは内部的にいくつかのトピックを基本的に購読しています。これらのトピックは、ノードトピックとユーザートピックに大きく分けられます。これにより、メッセージはノード単位で送信された後、ノード内のオブジェクトに送信されます。以下は、これらのノードトピックとユーザートピックを利用して発行するコードの例です。

```java
publishToUser(NodeTopic, Topic, Packet);
```


### GameAnvilのトピック

GameAnvilは内部的に以下のトピックを基本的に購読します。GameAnvilTopicは絶対にユーザーが任意で購読してはいけません。

| トピック                       | 発行対象               | 購読対象 |
| ---------------------------- |-----------------------| ----------- |
| GameAnvilTopic.GAME_NODE     | すべてのゲームノード            | GameNode    |
| GameAnvilTopic.GATEWAY_NODE  | すべてのゲートウェイノード         | GatewayNode |
| GameAnvilTopic.SUPPORT_NODE  | すべてのサポートノード           | SupportNode |
| GameAnvilTopic.ALL_CLIENT    | 接続中のすべてのクライアント      | Session     |
| GameAnvilTopic.ALL_GAME_USER | ゲームノードにあるすべてのゲームユーザーオブジェクト | GameUser    |



### トピックの購読および購読キャンセル

トピックは、ノードトピックとユーザートピックに大きく分けられると説明しました。ノードトピックは、BaseNodeを継承するすべての種類のノードクラスで購読します。一方で、ユーザートピックはノード内部のオブジェクトのためのものであるため、BaseUserとBaseRoomを継承するすべてのユーザーとルームクラスで購読可能です。ノードトピックとユーザートピックの購読および購読キャンセル方法は、次のサンプルコードのとおりです。

```java
// "GameUser1"トピックを購読します。
addTopic("GameUser1");

// "GameUser1"トピックの購読をキャンセルします。
removeTopic("GameUser1");
```

以下は、トピックを使用するためのAPI全体リストです。
```java
/**
 * トピックが購読状態であるかを確認
 *
 * @param topic確認するトピック
 * @return該当トピックを購読中の場合はtrueを返す
 */
boolean hasTopic(String topic)

/**
 * 購読中のトピックリストを返す
 *
 * @return StringのSetでトピックリストを返す
 */
Set<String> getTopics()

/**
 * トピックを購読
 *
 * @param topic購読するトピック
 *
 * @return正常に購読した場合はtrueを返す
 */
boolean addTopic(String topic)

/**
 * 複数のトピックを購読
 *
 * @param topics購読するトピックリスト
 *
 * @return正常に購読した場合はtrueを返す
 */
boolean addTopics(List<String> topics)

 /**
  * トピックを購読キャンセル
  *
  * @param topic購読をキャンセルするトピック
  */
void removeTopic(String topic)

 /**
  * 複数のトピックの購読をキャンセル
  *
  * @param topics購読をキャンセルするトピックリスト
  */
void removeTopics(List<String> topics)
```



### クライアントトピック

クライアントトピックはサーバー内のオブジェクトではなく、クライアントとして発行するためのトピックです。つまり、ユーザーは次のサンプルコードのように、クライアントを対象にトピックを購読できます。

```java
@Override
public final boolean onLogin(final Payload payload, final Payload sessionPayload, Payload outPayload) throws SuspendExecution {

    ...
        
	// 該当ユーザーと接続されたクライアントにトピックを購読
	if (isVIP())
		addClientTopics(Arrays.asList("VIP"));
}
```
以下は、これらのクライアントトピックを使用するためのAPIリストです。
```java
/**
 * 複数のクライアントトピックリストを購読します。
 *
 * @param topics登録するトピックリストを送信。
 */
public void addClientTopics(List<String> topics)

/**
 * 複数のクライアントトピックリストの購読をキャンセルします。
 *
 * @param topics購読をキャンセルするトピックリスト
 */
public void removeClientTopics(List<String> topics)

/**
 * クライアントトピックリストを返す
 *
 * @return購読中のクライアントトピックの文字列を含むSetを返す
 */
public Set<String> getClientTopics() {
    return this.gameUserHelper.getClientTopics();
}
```



### 発行する

ユーザーは任意のトピックでメッセージを発行できます。この時、該当トピックを購読中の対象はすべて、同じメッセージを受信します。

```java
// ノードトピックとトピックを区別して使用する必要があります。
publishToUser(NodeTopic, Topic, Packet)

// クライアントトピックはノードトピックが必要ありません。
publishToClient(Topic, Packet)
```
