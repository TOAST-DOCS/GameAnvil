## Game > GameAnvil > Unity Basic Development Guide > Message Handling

## Message handling

In addition to the basic functionality of the UserAgent, you can send messages to the server using Request() and Send(). Sending a message involves creating and registering a message.

### Create a message

GameAnvil uses [ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/proto3)as its default messaging protocol. You will define your messages in a .proto file, and generate the actual class source code with the protoc compiler. You can use the generated source code by adding it to your project. The protoc compiler can be found in the GameAnvil/protoc folder. For a detailed description of protoc, see [here](https://developers.google.com/protocol-buffers/docs/proto3#generating).

Now let's create a message. First, create a protocols folder under the Assets folder and create a messages.proto file like this

```protobuf
// messages.proto
syntax = "proto3";

package Messages;

message SampleRequest
{
  string msg = 1;
}

message SampleResponse
{
  repeated string msgs = 1;
}

message SampleSend
{
  string msg = 1;
}

message SampleReceive
{
  repeated string msgs = 1;
}
```

Then, run the Windows Command Prompt (cmd) to navigate to the protocols folder and enter

```
../GameAnvil/protoc/protoc --csharp_out=./ messages.proto
```

You'll then see that a Messages.cs file has been created in the protocols folder. 

### Register messages

To use newly created messages, you must pre-register the messages you want to use with ProtocolManager. If you don't pre-register them, they might not work, malfunction, or throw exceptions.

```c#
ProtocolManager.getInstance().RegisterProtocol(Messages.MessagesReflection.Descriptor);
```

### Send messages

When you send a message to Request(), it waits for a server response. While waiting for the server response, additional Request() are queued and processed sequentially after the server response is processed. To receive and process a server response, you must pass callback parameters.

```c#
/// <summary>
/// Sends a proto-buff message using the user agent.
/// </summary>
/// <typeparam name="T">Proto buff type message</typeparam>
/// <param name="agent">User agent to send</param>
/// <param name="message">The proto buff message to send</param>
/// <param name="action">Action to handle in response</param>
static public void Request<T>(User.UserAgent agent, IMessage message, Action<User.UserAgent, T> action) where T : IMessage;

/// <summary>
/// Sends a packet using a user agent.
/// </summary>
/// <param name="agent">The user agent to send the packet to</param>.
/// <param name="packet">Packet to send</param>
/// <param name="action">Action to handle in response</param>
static public void Request(User.UserAgent agent, Packet packet, Action<User.UserAgent, Packet> action);
```

You can also register a listener to receive the request response, which can be found in the [Unity Advanced Development Guide > Message Handling](../unity-advanced/unity-advanced-04-message-handling.md).

If no response is received within the specified time, a timeout occurs and the next message is processed. The timeout is passed to the UserAgent.OnErrorCommandListeners listener and the UserAgent.OnErrorCustomCommandListeners listener as ErrorCode.TIMEOUT.

When you send a message with Send(), it is sent to the server immediately upon the call to Send() and does not wait for a separate response. Messages sent with Send() are sent to the server immediately, even if you are waiting for a response to Request().

```c#
Connector connector = new Connector();
UserAgent user = GameAnvilConnector.getUserAgent();
// register a listener to receive server notifications delivered to the UserAgent
user.AddListener((UserAgent user, Messages.SampleReceive msg)=> { }); 

// UserAgent Send
Messages.SampleSend sampleSend = new Messages.SampleSend(); 
user.Send(sampleSend);

// UserAgent Request
Messages.SampleRequest SampleRequest = new Messages.SampleRequest();
user.Request(SampleRequest, (UserAgent user, Messages.SampleResponse res) => { }); // pass callback parameters
```

### Custom packets

You can use the Packet class to serialize arbitrary data other than a ProtocolBuffer into a byte stream. For more information about Packets, see [Unity Advanced Development Guide > Packets](../unity-advanced/unity-advanced-05-packet.md).

```c#
Connector connector = new Connector();
UserAgent user = GameAnvilConnector.getUserAgent();
int reqMsgId = 1;
int resMsgId = 2;

user.AddListener(resMsgId, (UserAgent user, Packet packet)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend(); 
// using the packet class
Packet sampleSendPacket = new Packet(reqMsgIndex, sampleSend.ToByteArray())
user.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
// using the packet class
Packet sampleRequestPacket = new Packet(reqMsgIndex, sampleRequestPacket.ToByteArray())
user.Request(sampleRequestPacket, (UserAgent user, Packet packet)=> { });
```