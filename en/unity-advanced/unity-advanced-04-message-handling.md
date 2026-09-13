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

In `GameAnvilConnector`, you can call `Request()`, and in `GameAnvilUser`, you can call `RequestUser()` to send messages and receive responses.

```c#
public async void RequestPacket()
{
    try
    {
        Packet packet = Packet.MakePacket(new Protocol.SampleRequest());
        Result<ResultCode, Protocol.SampleResponse> result = await connector.Request<Protocol.SampleResponse>(packet);
        if (result.ResultCode == ResultCode.SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}
```

`Request<\TResponse\>()` and `RequestUser\<TResponse\>()` each take one type parameter and one parameter, as shown below.

| Type | Name | Description |
|----------|--------------|----------------|
| Type parameter | TResponse | Message type to receive as a response |
| IMessage | message | Message to send to the server |

The return value is `Result<ResultCode, TResponse>`. You can check the value of the `ResultCode` field to determine whether the request succeeded. If `RequestUser()` succeeds, the `ResultCode` field is set to `ResultCode.SUCCESS`; otherwise, the message transmission has failed. You can obtain the response message through the `Data` field.

The details of `ResultCode` are as follows.

| Name | Value | Description |
|-------------------|----|--------------------------------------------|
| PARSE_ERROR | -2 | Packet parsing error. This may occur when the server and client versions differ. |
| TIMEOUT | -1 | Timeout. No response to the request was received within the specified time. |
| SYSTEM_ERROR | 1 | Server system error. Failed due to an unknown server error. |
| INVALID_PROTOCOL | 2 | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| HANDLER_NOT_EXIST | 10 | Failed. No handler on the server. |
| HANDLER_ERROR | 11 | Failed. An exception occurred in the server's handler. |
| SUCCESS | 0 | Success |

<a id="sending-messages-senduser"></a>
#### SendUser

Call Send() from GameAvnilConnector and SendUser() from GameAvilUser to send a message to the server without waiting for a response.

```c#
public async void SendMessage()
{
    try
    {
        connector.Send(new Protocol.SampleSend());
    } catch (Exception e)
    {
        // 예외
    }
}
```

Send() and SendUser() have one parameter as follows:

| Type | Name | Description |
|----------|---------|------------|
| IMessage | message | Message to send to the server |

<a id="sending-messages-messagecallback"></a>
#### MessageCallback

To receive messages sent from the server regardless of messages sent via Send() or SendUser(), you can register a callback using SetMessageCallback\<TProtoBuffer\>(). To deregister a registered callback, use RemoveMessageCallback\<TProtoBuffer\>().

```c#
public async void MessageCallback()
{
    connector.SetMessageCallback((GameAnvilConnector connector, ResultCode resultCode, Protocol.SampleReceive receive) =>
    {
        return Task.CompletedTask;
    });

    connector.RemoveMessageCallback<Protocol.SampleReceive>();
}
```

SetMessageCallback\<TProtoBuffer\>() has one type parameter and one parameter as follows:

| Type                                                          | Name         | Description                                          |
|---------------------------------------------------------------|--------------|------------------------------------------------------|
| Type parameter                                                | TProtoBuffer | The type of message to receive from the server       |
| Func<GameAnvilUserController, ResultCode, TProtoBuffer, Task> | callback     | The callback method to be called when the server sends a message |

RemoveMessageCallback\<TProtoBuffer\>() has one type parameter as follows:

| Type           | Name         | Description                                          |
|----------------|--------------|------------------------------------------------------|
| Type parameter | TProtoBuffer | The message type that the registered callback receives |

