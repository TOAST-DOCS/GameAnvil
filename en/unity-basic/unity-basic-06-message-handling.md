<!-- pre-align:aligned sig=021fed3a8700 -->

<a id="game-gameanvil-unity-basic-development-guide-message-handling"></a>
## Game > GameAnvil > Unity Basic Development Guide > Message Handling { #game-gameanvil-unity-basic-development-guide-message-handling }

<a id="message-handling"></a>
## Message handling { #message-handling }

In addition to the basic functionality of the UserAgent, you can send messages to the server using Request() and Send(). Sending a message involves creating and registering a message.

<a id="create-a-message"></a>
### Create a message { #create-a-message }

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

<a id="register-messages"></a>
### Register messages { #register-messages }

To use newly created messages, you must pre-register the messages you want to use with ProtocolManager. If you don't pre-register them, they might not work, malfunction, or throw exceptions.

```c#
ProtocolManager.getInstance().RegisterProtocol(Messages.MessagesReflection.Descriptor);
```

<a id="send-messages"></a>
### Send messages { #send-messages }

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

<a id="send-messages-requestuser"></a>
#### RequestUser

You can send a message and receive a response using RequestUser().

```c#
public async void ManagerRequest()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        var (resultCode, response) = await userController.RequestUser<Protocol.SampleResponse>(new Protocol.SampleRequest());
        if (resultCode == ResultCode.SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

RequestUser\<TProtoBuffer\>() takes one type parameter and one parameter, as described below.

| Type       | Name           | Description                              |
|----------|--------------|----------------|
| Type parameter  | TProtoBuffer | Message type to receive as a response |
| IMessage | message      | Message to send to the server     |

It returns an ErrorResult<ResultCode, TProtoBuffer> as the response. You can check the value of the ErrorCode field to determine whether the request was successful. If RequestUser() succeeds, the value of the ErrorCode field is ResultCode.SUCCESS; otherwise, the message transmission has failed. You can retrieve the response message through the Data field.

The details of ResultCode are as follows.

| Name                | Value  | Description                                         |
|-------------------|----|--------------------------------------------|
| PARSE_ERROR       | -2 | Packet parsing error. This may occur when the server and client versions differ.   |
| TIMEOUT           | -1 | Timeout. The response to the request did not arrive within the allotted time.          |
| SYSTEM_ERROR      | 1  | Server system error. Failed due to an unknown error on the server.              |
| INVALID_PROTOCOL  | 2  | Protocol not registered on the server. A protocol that is not registered in the additional information was used. |
| HANDLER_NOT_EXIST | 10 | Failed. No handler on the server.                            |
| HANDLER_ERROR     | 11 | Failed. An exception occurred in the server's handler.                       |
| SUCCESS           | 0  | Success                                         |

<a id="send-messages-senduser"></a>
#### SendUser

When you send a message using SendUser(), the message is sent to the server immediately upon the call, without waiting for a separate response.

```c#
public async void ManagerSend()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        userController.SendUser(new Protocol.SampleSend());
    } catch (Exception e)
    {
        // exception
    }
}
```

SendUser() has one parameter as follows:

| Type     | Name    | Description                  |
|----------|---------|------------------------------|
| IMessage | message | Message to send to the server |

<a id="send-messages-messagecallback"></a>
#### MessageCallback

To receive messages sent from the server regardless of messages sent with SendUser(), you can register a callback using SetMessageCallback\<TProtoBuffer\>(). To unregister a registered callback, use RemoveMessageCallback\<TProtoBuffer\>().

```c#
public async void ManagerMessageCallback()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    userController.SetMessageCallback((GameAnvilUserController user, ResultCode resultCode, Protocol.SampleReceive receive) =>
    {
        return Task.CompletedTask;
    });
    
    userController.RemoveMessageCallback<Protocol.SampleReceive>();
}
```

SetMessageCallback\<TProtoBuffer\>() has one type parameter and one parameter as follows:

| Type                                                          | Name         | Description                                        |
|---------------------------------------------------------------|--------------|----------------------------------------------------|
| Type parameter                                                | TProtoBuffer | The type of message to receive from the server     |
| Func<GameAnvilUserController, ResultCode, TProtoBuffer, Task> | callback     | The callback method called when the server sends a message |

RemoveMessageCallback\<TProtoBuffer\>() has one type parameter as follows:

| Type           | Name         | Description                                           |
|----------------|--------------|-------------------------------------------------------|
| Type parameter | TProtoBuffer | The type of message that the registered callback receives |

