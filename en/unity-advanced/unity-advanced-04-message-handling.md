<!-- pre-align:aligned sig=30f67c2adce6 -->

<a id="game-gameanvil-unity-advanced-development-guide-message-handling"></a>
## Game > GameAnvil > Unity Advanced Development Guide > Message Handling { #game-gameanvil-unity-advanced-development-guide-message-handling }

<a id="message-handling"></a>
## Message handling { #message-handling }

In addition to the basic functionality of ConnectionAgent and UserAgent, you can send messages to the server using Request() and Send().

<a id="create-and-register-message"></a>
### Create and Register Message { #create-and-register-message }

The process for creating and registering messages is the same as when using GameAnvilManager. For more information, see [Basic Development Guide to Unity > Message Handling](../unity-basic/unity-basic-06-message-handling.md).

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

You can send a message and receive a response by calling Request() on GameAvnilConnector, or RequestUser() on GameAvilUser.

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

Request<\TResponse\>() and RequestUser\<TResponse\>() each take one type parameter and one parameter, as follows:

| Type            | Name      | Description                          |
|-----------------|-----------|--------------------------------------|
| Type parameter  | TResponse | The type of the message to receive as a response |
| IMessage        | message   | The message to send to the server    |

The return value is Result<ResultCode, TResponse>. You can check the value of the ResultCode field to determine whether the request was successful. If RequestUser() succeeds, the ResultCode field will be ResultCode.SUCCESS; otherwise, the message transmission has failed. You can retrieve the response message through the Data field.

The details of ResultCode are as follows:

| Name              | Value | Description                                                                                           |
|-------------------|-------|-------------------------------------------------------------------------------------------------------|
| PARSE_ERROR       | -2    | Packet parsing error. This may occur if the server and client are on different versions.              |
| TIMEOUT           | -1    | Timeout. No response to the request was received within the allotted time.                            |
| SYSTEM_ERROR      | 1     | Server system error. The request failed due to an unknown error on the server.                        |
| INVALID_PROTOCOL  | 2     | Protocol not registered on the server. A protocol that is not registered in the additional information was used. |
| HANDLER_NOT_EXIST | 10    | Failed. No handler on the server.                                                                     |
| HANDLER_ERROR     | 11    | Failed. An exception occurred in the server's handler.                                                |
| SUCCESS           | 0     | Success                                                                                               |

<a id="sending-messages-senduser"></a>
#### SendUser

In GameAvnilConnector, call Send(), and in GameAvilUser, call SendUser() to send a message to the server without waiting for a separate response.

```c#
public async void SendMessage()
{
    try
    {
        connector.Send(new Protocol.SampleSend());
    } catch (Exception e)
    {
        // exception
    }
}
```

Send() and SendUser() have one parameter as follows:

| Type | Name | Description |
|----------|---------|------------|
| IMessage | message | Message to send to the server |

<a id="sending-messages-messagecallback"></a>
#### MessageCallback

To receive messages sent from the server regardless of messages sent via Send() or SendUser(), you can register a callback using SetMessageCallback\<TProtoBuffer\>(). To unregister a registered callback, use RemoveMessageCallback\<TProtoBuffer\>().

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
| Type parameter                                                | TProtoBuffer | Type of the message to receive from the server       |
| Func<GameAnvilUserController, ResultCode, TProtoBuffer, Task> | callback     | Callback method called when the server sends a message |

RemoveMessageCallback\<TProtoBuffer\>() has one type parameter as follows:

| Type           | Name         | Description                                              |
|----------------|--------------|----------------------------------------------------------|
| Type parameter | TProtoBuffer | Type of the message that the registered callback receives |

