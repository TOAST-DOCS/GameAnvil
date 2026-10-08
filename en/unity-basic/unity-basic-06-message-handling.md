<!-- machine_translated: true -->

<!-- pre-align:aligned sig=021fed3a8700 -->

<a id="game-gameanvil-unity-basic-development-guide-message-handling"></a>
## Game > GameAnvil > Unity Basic Development Guide > Message Handling { #game-gameanvil-unity-basic-development-guide-message-handling }

<a id="message-handling"></a>
## Message Handling { #message-handling }

You can use the RequestUser() and SendUser() methods of GameAnvilUserController to send user-defined messages to the server. Sending a message involves creating and registering a message.

<a id="create-a-message"></a>
### Create a message { #create-a-message }

GameAnvil uses [ProtocolBuffers](https://developers.google.com/protocol-buffers/docs/proto3) as its default messaging protocol. You will define your messages in a .proto file, and generate the actual class source code with the protoc compiler. You can use the generated source code by adding it to your project. For a detailed description of protoc, see [here](https://developers.google.com/protocol-buffers/docs/proto3#generating).

Now let's create a message. First, create a protocols folder under the Assets folder and create a messages.proto file like this:

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

Then, use protoc in the Windows Command Prompt (cmd) to generate the messages.

```
/protoc --csharp_out=./ messages.proto
```

<a id="register-messages"></a>
### Register messages { #register-messages }

To use newly created messages, you must pre-register the messages you want to use with ProtocolManager. If you don't pre-register them, they might not work, malfunction, or throw exceptions.

```c#
GameAnvilProtocolManager.RegisterProtocol(Messages.MessagesReflection.Descriptor);
```

<a id="send-messages"></a>
### Send messages { #send-messages }

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

RequestUser\<TProtoBuffer\>() has one type parameter and one parameter, as follows:

| Type | Name | Description |
|----------|--------------|----------------|
| Type parameter | TProtoBuffer | The type of the message to receive as a response |
| IMessage | message | The message to send to the server |

It returns an ErrorResult<ResultCode, TProtoBuffer> as the response. You can check the ErrorCode field value to determine whether the request succeeded. If RequestUser() succeeds, the ErrorCode field value is ResultCode.SUCCESS; otherwise, the message transmission has failed. You can get the response message through the Data field.

The details of ResultCode are as follows:

| Name | Value | Description |
|-------------------|----|--------------------------------------------|
| PARSE_ERROR | -2 | Packet parsing error. May occur when the server and client versions differ. |
| TIMEOUT | -1 | Timeout. No response to the request was received within the specified time. |
| SYSTEM_ERROR | 1 | Server system error. Failed due to an unknown error on the server. |
| INVALID_PROTOCOL | 2 | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| HANDLER_NOT_EXIST | 10 | Failed. No handler on the server. |
| HANDLER_ERROR | 11 | Failed. An exception occurred in the server's handler. |
| SUCCESS | 0 | Success |

<a id="send-messages-senduser"></a>
#### SendUser

When you send a message with SendUser(), it is sent to the server immediately upon the call to SendUser() and does not wait for a separate response.

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

SendUser() has one parameter, as follows:

| Type | Name | Description |
|----------|---------|------------|
| IMessage | message | The message to send to the server |

<a id="send-messages-messagecallback"></a>
#### MessageCallback

To receive messages sent from the server independently of messages sent with SendUser(), you can register a callback using SetMessageCallback\<TProtoBuffer\>(). To remove a registered callback, use RemoveMessageCallback\<TProtoBuffer\>().

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

SetMessageCallback\<TProtoBuffer\>() has one type parameter and one parameter, as follows:

| Type | Name | Description |
|---------------------------------------------------------------|--------------|----------------------------|
| Type parameter | TProtoBuffer | The type of the message to receive from the server |
| Func<GameAnvilUserController, ResultCode, TProtoBuffer, Task> | callback | The callback method to call when a message is received from the server |

RemoveMessageCallback\<TProtoBuffer\>() has one type parameter, as follows:

| Type | Name | Description |
|---------|--------------|-------------------|
| Type parameter | TProtoBuffer | The type of message that the registered callback receives |