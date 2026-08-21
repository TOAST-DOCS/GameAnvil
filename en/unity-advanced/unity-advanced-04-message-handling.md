<!-- pre-align:aligned sig=30f67c2adce6 -->

<a id="game-gameanvil-unity-advanced-development-guide-message-handling"></a>
## Game > GameAnvil > Unity Advanced Development Guide > Message Handling { #game-gameanvil-unity-advanced-development-guide-message-handling }

<a id="message-handling"></a>
## Message handling { #message-handling }

In addition to the basic functionality of ConnectionAgent and UserAgent, you can send messages to the server using Request() and Send().

<a id="create-and-register-message"></a>
### Create and Register Message { #create-and-register-message }

<!-- TODO: translate body -->

<a id="sending-messages"></a>
### Sending messages { #sending-messages }

When you send a message to Request(), you wait for a server response. There are two ways to receive and process the server response: first, you can pass callback parameters, as introduced in the [Unity Fundamentals Development Guide > Message Handling](../unity-basic/unity-basic-06-message-handling.md). 

Another way is to register a listener. If you don't apply either method, when you receive a server response, you'll process the next message without any notification. 

Let's look at an example of registering a listener to receive a response from Request().

This section introduces examples of using Request() and Send() through ConnectionAgents that were not covered in the [Unity Basic Development Guide > Message Handling](../unity-basic/unity-basic-06-message-handling.md).

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
// Register a listener to receive server notifications delivered to the ConnectionAgent
connection.AddListener((ConnectionAgent connection, Messages.SampleReceive msg)=> { });

// ConnectionAgent Send
Messages.SampleSend sampleSend= new Messages.SampleSend(); 
connection.Send(sampleSend);

// ConnectionAgent Request
Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
connection.Request(sampleRequest, (ConnectionAgent connection, Packet packet)=> { });
```

<a id="sending-messages-request"></a>
#### Request

<!-- TODO: translate body -->

<a id="sending-messages-senduser"></a>
#### SendUser

<!-- TODO: translate body -->

<a id="sending-messages-messagecallback"></a>
#### MessageCallback

<!-- TODO: translate body -->

