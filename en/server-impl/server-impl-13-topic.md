## Game > GameAnvil > Server Development Guide > Use Topics



## Subscription and Publishing

GameAnvil supports a subscription-publishing model, meaning that any audience that subscribes to a topic can all receive the same message through a publication. The usage of these subscriptions and publications is centered around topics.



### Topics

Users can subscribe to any topic at any time. GameAnvil also subscribes to a few topics internally by default. These topics are broadly divided into node topics and user topics. These allow messages to be sent on a per-node basis and then delivered to objects within the node. The following is an example of code that uses these node topics and user topics to publish.

```java
publishToUser(NodeTopic, Topic, Packet);
```


### GameAnvil 토픽

GameAnvil is internally subscribed to the following topics by default. GameAnvilTopic should never be arbitrarily subscribed to by users.

| Topics                         | Publish Target                            | Subscribe Target   |
| ---------------------------- | ------------------------------------ | ----------- |
| GameAnvilTopic.GAME_NODE     | All game nodes                       | GameNode    |
| GameAnvilTopic.GATEWAY_NODE  | All gateway nodes                 | GatewayNode |
| GameAnvilTopic.SUPPORT_NODE  | All support nodes                     | SupportNode |
| GameAnvilTopic.ALL_CLIENT    | All connected clients             | Session     |
| GameAnvilTopic.ALL_GAME_USER | All Game User objects in the Game node | GameUser    |



### Subscribe and unsubscribe to topics

As described above, the topics are broadly divided into node topics and user topics. Node topics are subscribed to by any kind of node class that inherits from BaseNode. User topics, on the other hand, are for objects inside the node and can be subscribed to by all user and room classes that inherit from BaseUser and BaseRoom. Subscribing and unsubscribing to node topics and user topics is the same for both, as shown in the following example code.

```java
// Subscribe to the "GameUser1" topic.
addTopic("GameUser1");

// Unsubscribe from the "GameUser1" topic.
removeTopic("GameUser1");
```

Below is a complete list of APIs for using Topics.
```java
/**
 * Check if the topic is subscribed
 * 
 * @param topic The topic to check
 * @return true if the topic is subscribed to
 */
boolean hasTopic(String topic)

/**
 * Returns a list of subscribed topics
 * 
 * @returns a list of topics as a Set of Strings
 */ 
Set<String> getTopics()

/*
 * Subscribe to a topic
 * 
 * @param topic the topic to subscribe to
 * 
 * @return true on successful subscription
 */
boolean addTopic(String topic)

/**
 * Subscribe to multiple topics.
 * 
 * @param topics list of topics to subscribe to
 * 
 * @return true if subscribed successfully
 */
boolean addTopics(List<String> topics)

 /**
  * Unsubscribes topics
  *
  * @param topic the topic to unsubscribe from
  */ 
void removeTopic(String topic)

 /** 
  * Unsubscribes from multiple topics
  * 
  * @param topics list of topics to unsubscribe from
  */ 
void removeTopics(List<String> topics)
```



### Client Topics

A client topic is a topic that is intended to be published to a client rather than an object on the server. This means that users can subscribe to a topic as a client, as shown in the example code below.

```java
@Override
public final boolean onLogin(final Payload payload, final Payload sessionPayload, Payload outPayload) throws SuspendExecution {

    ...
        
	// Subscribe the topic to the client associated with the user.
	if (isVIP())
		addClientTopics(Arrays.asList("VIP"));
}
```
Below is a list of APIs for using these client topics.
```java
/**
 * Subscribe to multiple client topic lists.
 *
 * @param topics Pass a list of topics to subscribe to.
 */
public void addClientTopics(List<String> topics)

/**
 * Subscribes to a list of multiple client topics.
 *
 * @param topics A list of topics to unsubscribe from.
 */
public void removeClientTopics(List<String> topics)

/**
 * Returns a list of client topics.
 *
 * @returns a Set containing the subscribed client topic strings
 */
public Set<String> getClientTopics() {
    return this.gameUserHelper.getClientTopics();
}
```



### Publish

Users can publish messages to any topic. Anyone who is subscribed to that topic will receive the same message.

```java
// Node topics and topics should be used separately.
publishToUser(NodeTopic, Topic, Packet)

// Client topics do not need a node topic.
publishToClient(Topic, Packet)
```
