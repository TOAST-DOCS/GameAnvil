## Game > GameAnvil > Console User Guide > Services and Channels

## Service

This document describes the 'service' mentioned earlier when dealing with configuration information (Config) registration.

A service has a unique service ID and service name, and GameNodes and SupportNodes can register as different kinds of services depending on their implementation. In other words, a service can be said to define the role of a GameNode or SupportNode.

For example, the 'RPSGame' service is for the game content called 'RPS' implemented on that server, which means it is responsible for serving game content. Similarly, you can define a service for the chat content implemented on that server by adding the 'RPSChat' service. 

This means that the game server has two GameNodes, each implemented separately for its role, as shown in the following pseudocode. Take a close look at the annotations we've used in this code to associate arbitrary service names with their corresponding GameNode implementations.

```java
// GameNode implemented game contents
@ServiceName("RpsGame") 
MyGameNode extends gameanvil.GameNode { 
    ... 
} 
 
// GameNode Implemented chatting contents
@ServiceName("RpsChat") 
MyChatNode extends gameanvil.GameNode { 
    ... 
}
```

The service name registered with the @ServiceName annotation must match the service name used when configuring the server. Because these service names are easy for human to read but unnecessarily long and large for server programs to read, the Configuration Information (Config) mapped a unique integer for each service name, a service ID. The service ID must be an integer value greater than 0. More information is available in the Server Implementation document.


## Channel

Channels provide a logical way to divide a service. For example, RPSGame services can be divided into 'beginner', 'intermediate', and 'advanced' channels. However, they are only available for GameNodes and can be any string of channel names. For more information, see [Game>GameAnvil>Server Development Guide>Channel](../server-impl/server-impl-09-channel.md).

