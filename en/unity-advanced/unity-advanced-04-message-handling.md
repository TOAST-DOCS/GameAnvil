<!-- machine_translated: true -->

<!-- pre-align:aligned sig=30f67c2adce6 -->

<a id="game-gameanvil-unity-advanced-development-guide-message-handling"></a>
## Game > GameAnvil > Unity Advanced Development Guide > Message Handling { #game-gameanvil-unity-advanced-development-guide-message-handling }

<a id="message-handling"></a>
## Message Handling { #message-handling }

In addition to the basic functionality of GameAnvilConnector and GameAnvilUser, you can send user-defined messages to the server.

<a id="create-and-register-message"></a>
### Create and Register Message { #create-and-register-message }
The process for creating and registering messages is the same as when using GameAnvilManager. For more information, see [Basic Development Guide to Unity > Message Handling](../unity-basic/unity-basic-06-message-handling.md).

<a id="sending-messages"></a>
### Sending messages { #sending-messages }

<a id="sending-messages-request"></a>
#### Request

In GameAnvilConnector, you can call Request(), and in GameAnvilUser, you can call RequestUser() to send a message and receive a response.

```c#
public async void RequestPacket()
{
    try
    {
        Packet packet = Packet.MakePacket(new Protocol.SampleRequest());
        Result<ResultCode, Protocol.SampleResponse> result = await connector.Request<Protocol.SampleResponse>(packet);
        if (result.ResultCode == ResultCode.SUCCESS)
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

Request\<TResponse\>() and RequestUser\<TResponse\>() take one type parameter and one parameter, as follows:

| Type | Name | Description |
|----------|--------------|----------------|
| Type parameter | TResponse | The message type to receive as a response |
| IMessage | message | The message to send to the server |

The return value is Result\<ResultCode, TResponse\>. You can check the value of the ResultCode field to determine whether the call succeeded. If RequestUser() succeeds, the ResultCode field is set to ResultCode.SUCCESS; otherwise, the message transmission has failed. You can get the response message through the Data field.

The details of ResultCode are as follows:

| Name | Value | Description |
|-------------------|----|--------------------------------------------|
| PARSE_ERROR       | -2 | Packet parsing error. This may occur if the server and client versions differ. |
| TIMEOUT           | -1 | Timeout. No response to the request was received within the allotted time. |
| SYSTEM_ERROR      | 1  | Server system error. Failed due to an unknown server error. |
| INVALID_PROTOCOL  | 2  | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| HANDLER_NOT_EXIST | 10 | Failed. No handler on the server. |
| HANDLER_ERROR     | 11 | Failed. An exception occurred in the server's handler. |
| SUCCESS           | 0  | Success |

<a id="sending-messages-senduser"></a>
#### SendUser

In GameAnvilConnector, you can call Send(), and in GameAnvilUser, you can call SendUser() to send a message to the server without waiting for a separate response.

```c#
public async void SendMessage()
{
    try
    {
        connector.Send(new Protocol.SampleSend());
    } catch (Exception e)
    {
        // Exception
    }
}
```

Send() and SendUser() take one parameter, as follows:

| Type | Name | Description |
|----------|---------|------------|
| IMessage | message | The message to send to the server |

<a id="sending-messages-messagecallback"></a>
#### MessageCallback

To receive messages sent from the server independently of messages sent via Send() or SendUser(), you can register a callback using SetMessageCallback\<TProtoBuffer\>(). To remove a registered callback, use RemoveMessageCallback\<TProtoBuffer\>().

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

SetMessageCallback\<TProtoBuffer\>() takes one type parameter and one parameter, as follows:

| Type | Name | Description |
|---------------------------------------------------------------|--------------|----------------------------|
| Type parameter | TProtoBuffer | The message type to receive from the server |
| Func\<GameAnvilUserController, ResultCode, TProtoBuffer, Task\> | callback | The callback method to be called when the server sends a message |

RemoveMessageCallback\<TProtoBuffer\>() takes one type parameter, as follows:

| Type | Name | Description |
|---------|--------------|-------------------|
| Type parameter | TProtoBuffer | The message type that the registered callback will receive |