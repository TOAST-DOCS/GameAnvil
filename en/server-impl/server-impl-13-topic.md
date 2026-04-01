## Game > GameAnvil > Server Development Guide > About Topic



## Subscription and Publishing

GameAnvil supports a subscription-publishing model, meaning that any audience that subscribes to a topic can all receive the same message through a publication. The usage of these subscriptions and publications is centered around topics.



### Topics

Users can subscribe to any topic at any time. GameAnvil also subscribes to a few topics internally by default. These topics are broadly divided into node topics and user topics. These allow messages to be sent on a per-node basis and then delivered to objects within the node. The following is an example of code that uses these node topics and user topics to publish.

```java
/**
 * Send packets to all users who have registered for the given node topic and general topic
 *
 * @param nodeTopic Topic registered by the target node that will receive the packet
 * @param topic     Topic registered by the user who will receive the packet
 * @param packet    {@link Packet} to send
 */
void publishToUser(String nodeTopic, String topic, Packet packet);
```


### GameAnvil Topic

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
 * @return If the return value is true, the user is subscribed; if false, the user is not subscribed.
 */
boolean hasTopic(String topic)

/**
 * Import a list of topics you are subscribed to
 * 
 * @returns Return the topic list
 */ 
Set<String> getTopics()

/*
 * Subscribe to a topic
 * 
 * @param topic the topic to subscribe to
 * @return If the return value is true, the subscription is successful; if it is false, the subscription is unsuccessful.
 */
boolean addTopic(String topic)

/**
 * Subscribe to multiple topics.
 * 
 * @param topics Topic to be deleted
 */
void removeTopic(String topic);
```

### Publish

Users can publish messages to any topic. Anyone who is subscribed to that topic will receive the same message.


The below is an API list for publishing topic:
```java
/**
 *  * Send packets to all clients
 *
 *  * @param packet {@link Packet} to be sent
 */
void publishToAllClient(Packet packet);

/**
 * Send messages to all clients
 *
 * @param message Message to be sent
 */
void publishToAllClient(GeneratedMessage message);

/**
 * Send packets to all nodes that have registered a given node topic
 *
 * @param serviceName Service name of the target node that will receive the packet
 * @param packet      {@link Packet} to be sent
 */
void publishToNodeWithServiceName(final String serviceName, final Packet packet);

/**
 * Send a message to all nodes that have registered for a given node topic
 *
 * @param serviceName Service name of the target node that will receive the message
 * @param message     Message to be sent
 */
void publishToNodeWithServiceName(final String serviceName, final GeneratedMessage message);

/**
 * Send packets to all nodes that have registered a given node topic
 *
 * @param hostId Host ID of the target node that will receive the packet
 * @param packet {@link Packet} to be sent
 */
void publishToNodeWithHostId(final long hostId, final Packet packet);

/**
 * Send a message to all nodes that have registered for a given node topic
 *
 * @param hostId  Host ID of the target node that will receive the message
 * @param message Message to be sent
 */
void publishToNodeWithHostId(final long hostId, final GeneratedMessage message);

/**
 * Send packets to all nodes that have registered a given node topic
 *
 * @param nodeId Node ID that will receive the packet
 * @param packet {@link Packet} to be sent
 */
void publishToNodeWithNodeId(final long nodeId, final Packet packet);

/**
 * Send a message to all nodes that have registered for a given node topic
 *
 * @param nodeId  The node ID that will receive the message
 * @param message Message to be sent
 */
void publishToNodeWithNodeId(final long nodeId, final GeneratedMessage message);

/**
 * Send packets to all nodes that have registered a given node topic
 *
 * @param serviceName Service name of the target node that will receive the packet
 * @param channelId   Channel ID of the target node that will receive the packet
 * @param packet      {@link Packet} to be sent
 */
void publishToNodeWithChannelId(final String serviceName, final String channelId, final Packet packet);

/**
 * Send a message to all nodes that have registered for a given node topic
 *
 * @param serviceName Service name of the target node that will receive the message
 * @param channelId   Channel ID of the target node that will receive the message
 * @param message     Message to be sent
 */
void publishToNodeWithChannelId(final String serviceName, final String channelId, final GeneratedMessage message);

/**
 * Send packets to all nodes that have registered a given node topic
 *
 * @param nodeTopic Topic registered by the target node that will receive the packet
 * @param packet    {@link Packet} to be sent
 */
void publishToNode(final String nodeTopic, final Packet packet);

/**
 * Send a message to all nodes that have registered for a given node topic
 *
 * @param nodeTopic Topic registered by the target node to receive the message
 * @param message   Message to be sent
 */
void publishToNode(final String nodeTopic, final GeneratedMessage message);

/**
 * Send packets to all users who have registered for the given node topic and general topic
 *
 * @param serviceName Service name of the target node that will receive the packet
 * @param topic       Topic registered by the user who will receive the packet
 * @param packet      {@link Packet} to be sent
 */
void publishToUserWithServiceName(final String serviceName, final String topic, final Packet packet);

/**
 * Send a message to all users who have registered for the given node topic and general topic
 *
 * @param serviceName Service name of the target node that will receive the message
 * @param topic       Topic registered by the user who will receive the packet
 * @param message     Message to be sent
 */
void publishToUserWithServiceName(final String serviceName, final String topic, final GeneratedMessage message);

/**
 * Send packets to all users who have registered for the given node topic and general topic
 *
 * @param hostId Host ID of the target node that will receive the packet
 * @param topic  Topic registered by the user who will receive the packet
 * @param packet {@link Packet} to be sent
 */
void publishToUserWithHostId(final long hostId, final String topic, final Packet packet);

/**
 * Send a message to all users who have registered for the given node topic and general topic
 *
 * @param hostId  Host ID of the target node that will receive the message
 * @param topic   Topic registered by the user who will receive the packet
 * @param message Message to be sent
 */
void publishToUserWithHostId(final long hostId, final String topic, final GeneratedMessage message);

/**
 * Send packets to all users who have registered for the given node topic and general topic
 *
 * @param nodeId Node ID that will receive the packet
 * @param topic  Topic registered by the user who will receive the packet
 * @param packet {@link Packet} to be sent
 */
void publishToUserWithNodeId(final long nodeId, final String topic, final Packet packet);


/**
 * Send a message to all users who have registered for the given node topic and general topic
 *
 * @param nodeId  The node ID that will receive the message
 * @param topic   Topic registered by the user who will receive the packet
 * @param message Message to be sent
 */
void publishToUserWithNodeId(final long nodeId, final String topic, final GeneratedMessage message);

/**
 * Send packets to all users who have registered for the given node topic and general topic
 *
 * @param serviceName Service name of the target node that will receive the packet
 * @param channelId   Channel ID of the target node that will receive the packet
 * @param topic       Topic registered by the user who will receive the packet
 * @param packet      {@link Packet} to be sent
 */
void publishToUserWithChannelId(final String serviceName, final String channelId, final String topic, final Packet packet);

/**
 * Send a message to all users who have registered for the given node topic and general topic
 *
 * @param serviceName Service name of the target node that will receive the message
 * @param channelId   Channel ID of the target node that will receive the message
 * @param topic       Topic registered by the user who will receive the packet
 * @param message     Message to be sent
 */
void publishToUserWithChannelId(final String serviceName, final String channelId, final String topic, final GeneratedMessage message);

/**
 * Send packets to all users who have registered for the given node topic and general topic
 *
 * @param nodeTopic Topic registered by the target node that will receive the packet
 * @param topic     Topic registered by the user who will receive the packet
 * @param packet    {@link Packet} to be sent
 */
void publishToUser(String nodeTopic, String topic, Packet packet);

/**
 * Send a message to all users who have registered for the given node topic and general topic
 *
 * @param nodeTopic Topic registered by the target node to receive the message
 * @param topic     Topic registered by the user who will receive the packet
 * @param message   Message to be sent
 */
void publishToUser(String nodeTopic, String topic, GeneratedMessage message);

/**
 * Send packets to all rooms that have registered the given node topic and general topic
 *
 * @param serviceName Service name of the target node that will receive the packet
 * @param topic       Topic registered by the user who will receive the packet
 * @param packet      {@link Packet} to be sent
 */
void publishToRoomWithServiceName(final String serviceName, final String topic, final Packet packet);

/**
 * Send a message to all rooms that have registered the given node topic and general topic.
 *
 * @param serviceName Service name of the target node that will receive the message
 * @param topic       Topic registered by the user who will receive the packet
 * @param message     Message to be sent
 */
void publishToRoomWithServiceName(final String serviceName, final String topic, final GeneratedMessage message);

/**
 * Send packets to all rooms that have registered the given node topic and general topic
 *
 * @param hostId Host ID of the target node that will receive the packet
 * @param topic  Topic registered by the user who will receive the packet
 * @param packet {@link Packet} to be sent
 */
void publishToRoomWithHostId(final long hostId, final String topic, final Packet packet);

/**
 * Send a message to all rooms that have registered the given node topic and general topic.
 *
 * @param hostId  Host ID of the target node that will receive the message
 * @param topic   Topic registered by the user who will receive the packet
 * @param message Message to be sent
 */
void publishToRoomWithHostId(final long hostId, final String topic, final GeneratedMessage message);

/**
 * Send packets to all rooms that have registered the given node topic and general topic
 *
 * @param nodeId Node ID that will receive the packet
 * @param topic  Topic registered by the user who will receive the packet
 * @param packet {@link Packet} to be sent
 */
void publishToRoomWithNodeId(final long nodeId, final String topic, final Packet packet);

/**
 * Send a message to all rooms that have registered the given node topic and general topic.
 *
 * @param nodeId  The node ID that will receive the message
 * @param topic   Topic registered by the user who will receive the packet
 * @param message Message to be sent
 */
void publishToRoomWithNodeId(final long nodeId, final String topic, final GeneratedMessage message);

/**
 * Send packets to all rooms that have registered the given node topic and general topic
 *
 * @param serviceName Service name of the target node that will receive the packet
 * @param channelId   Channel ID of the target node that will receive the packet
 * @param topic       Topic registered by the user who will receive the packet
 * @param packet      {@link Packet} to be sent
 */
void publishToRoomWithChannelId(final String serviceName, final String channelId, final String topic, final Packet packet);

/**
 * Send a message to all rooms that have registered the given node topic and general topic.
 *
 * @param serviceName Service name of the target node that will receive the message
 * @param channelId   Channel ID of the target node that will receive the message
 * @param topic       Topic registered by the user who will receive the packet
 * @param message     Message to be sent
 */
void publishToRoomWithChannelId(final String serviceName, final String channelId, final String topic, final GeneratedMessage message);

/**
 * Send packets to all rooms that have registered the given node topic and general topic
 *
 * @param nodeTopic Topic registered by the target node that will receive the packet
 * @param topic     Topic registered by the room that will receive the packet
 * @param packet    {@link Packet} to be sent
 */
void publishToRoom(String nodeTopic, String topic, Packet packet);

/**
 * Send a message to all rooms that have registered the given node topic and general topic.
 *
 * @param nodeTopic  * @param nodeTopic Topic registered by the target node to receive the message

 * @param topic     Topic registered by the room that will receive the message
 * @param message   Message to be sent
 */
void publishToRoom(String nodeTopic, String topic, GeneratedMessage message);