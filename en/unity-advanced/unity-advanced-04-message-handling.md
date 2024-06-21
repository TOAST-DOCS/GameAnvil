## Game > GameAnvil > Unity Advanced Development Guide > Message Handling

## Message handling

In addition to the basic functionality of ConnectionAgent and UserAgent, you can send messages to the server using Request() and Send().

### Sending messages

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

### Custom packets

You can use the Packet class to serialize arbitrary data other than a ProtocolBuffer into a byte stream. For more information about Packets, see [Unity Advanced Development Guide > Packets](unity-advanced-05-packet.md).

This section introduces examples of using Request() and Send() through ConnectionAgents that were not covered in the [Unity Basic Development Guide > Message Handling](../unity-basic/unity-basic-06-message-handling.md).

```c#
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
int reqMsgId = 1;
int resMsgId = 2;

connection.AddListener(resMsgId, (ConnectionAgent connection, Packet packet)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend(); 
// using the packet class
Packet sampleSendPacket = new Packet(reqMsgId, sampleSend.ToByteArray())
connection.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
// using the packet class
Packet sampleRequestPacket = new Packet(reqMsgId, sampleRequest.ToByteArray())
connection.Request(sampleRequestPacket, (ConnectionAgent connection, Packet packet)=> { });
```