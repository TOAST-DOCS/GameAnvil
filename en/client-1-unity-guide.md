## Game > GameAnvil > Client Development Guide > Unity Development Guide

Guide Environment

- Unity 2018.4.1f1
- Visual Studio 2017
- GameAnvil Connector: 1.1.0

## Installing GameAnvil Connector

Install gameanvil-connector.unitypackage to the Unity project. gameanvil-connector.unitypackage can be downloaded [here ](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector.unitypackage). When the package is installed, the following DLL files are installed in the GameAnvil folder. The DLL files are built with C# and are compatible with the Android, iOS, and PC platform.

![unitypackage](http://static.toastoven.net/prod_gameanvil/images/client-1-unitypackage.png)

## Creating Connector

To use GameAnvilConnector, you must first create a connector. It manages default settings and agents and registers delegates so that the log for internal tasks can be seen.

```
using GameAnvil;
Connector connector = new Connector();
```

It periodically calls the Update() function for processing the messages transferring to the server. The simplest way to call it is via Update() of MonoBehaviour, and calling it through other means if necessary is perfectly fine as well. The call interval of the Update() function can be set to any number, but if it is not called, the notice will not be received when a message is received by the server.

```
connector.Update();
```

You can simply inherit MonoBehaviour and build a script that can be used to manage connectors as below:

```
using GameAnvil;

public class ConnectHandler : MonoBehaviour
{
    public static Connector connector = null;

    private void Awake()
    {
        if(connector == null){
            connector = new GameAnvil.Connector();
            // Adding Connector Log
            connector.Logger += (level, log) =>
            {
                Debug.Log(string.Format("Log[{0}]:{1}", level, log));
            };
            connector.LvNetLogger += (level, log) =>
            {
                Debug.Log(string.Format("Net[{0}]:{1}", level, log));
            };
        }
    }

    private void Update()
    {
        if(connector != null)
            connector.Update();
    }

    private void OnDestroy()
    {
        if (connector != null && connector.IsConnected())
        {
            connector.CloseSocket();
        }
    }
}
```

This code is an example and can be altered as necessary. Now, the GameAnvil connector is ready to be used.

## Connecting to and Authenticating the Server

Connection and authentication are executed using ConnectionAgent. ConnectionAgent is responsible for the tasks related to the GameAnvil server's connection nodes. Basic session management features such as (Connect()) and (Authentication()) and channel list are provided, and according to server, channel information can be provided or custom message can be exchanged. ConnectionAgent is automatically created when a connector is initialized and can be obtained using the Connector.GetConnectionAgent() function. The result of connection and authentication can be checked by registering the listener that was used to implement IConnectionListener.

```
class ConnectionListener : IConnectionListener{
    ...
}


ConnectionListener listener = new ConnectionListener();


GameAnvil.ConnectionAgent connectionAgent = connector.GetConnectionAgent();
connectionAgent.AddConnectionListener(listener);


// ip    : ip address
// port    : port
connectionAgent.Connect(ip, port);
// Responding with ConnectionListener.OnConnect


// deviceId    : A unique ID that can be used to identify user device. It may not be used depending on the server.
// accountId   : A unique ID that can be used to identify user account.
// password    : The password of a user account. It may not be used depending on the server.
connectionAgent.Authentication(deviceId, accountId, password);
// Respond with ConnectionListener.OnAuthentication
```

## Using Login and Default Features

Login/Logout and basic features can be used through UserAgent. UserAgent is responsible for the tasks related to the game node of GameAnvil server. It provides basic features such as Login, Logout and room management, and may provide custom features depending on the server implementation. To use UserAgent, you must create a new UserAgent using the Connector.CreateUserAgent() function. Several UserAgents, either ServiceId or SubId, and each created UserAgent can be used independently. Created UserAgents are internally managed in Connector and can be reused using the Connector.GetUser() function. The result of login/logout and basic features can be seen by registering IUserListener. UserAgent can be created by using the Connector.CreateUserAgent() function. Several UserAgents, either ServiceId or SubId, and each UserAgent created can be used independently. The created UserAgent is cached inside Connector and can be reused through using the Connector.GetUserAgent() function.

```
// serviceId    : The serviceId to be used by UserAgent. 
// subId        : A unique ID that can be used to identify the UserAgent by service; it must be an integer of 1 or higher.
UserAgent userAgent = connector.CreateUserAgent(serviceId, subId);


userAgent = connector.GetUserAgent(serviceId, subId);
class UserListener : IUserListener{
    ...
}


UserListener listener = new UserListener();


UserAgent userAgent = connector.GetUserAgent(serviceId, subId);
userAgent.AddUserListener(listener)


// payload     : An additional value to be delivered to the server. It may not be used depending on the server.
// channelId   : The channel ID with which UserAgent logs in. It may not be used depending on the server.
userAgent.Login(payload, channelId);
// Respond with UserListener.OnLogin


userAgent.Logout();
// Respond with UserListener.OnLogout


userAgent.CreateRoom();
// Respond with UserListener.OnCreateRoom


userAgent.MatchRoom();
// Respond with UserListener.OnMatchRoom


userAgent.LeaveRoom();
// Respond with UserListener.OnLeaveRoom
...
```

## Message

Messages can be transferred to a server via by using Request() and Send() in addition to using the basic features of ConnectionAgent and UserAgent. To send a message, one must be created and registered.

### Creating Message

GameAnvil uses [ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/overview) as its default message protocol. A message is defined in the .proto file and actual class source code is created using the protoc compiler. The created source code can be used after it is added to a project. The protoc complier can be found in the GameAnvil/protoc folder. For more information on protoc, visit [here ](https://developers.google.com/protocol-buffers/docs/proto3#generating).

Now, let's create a message. First, create the protocols folder under the Assets folder and create the messages.proto file as follows.

```
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

Run Windows command prompt (cmd) to navigate to the protocols folder and enter the following.

```
../GameAnvil/protoc/protoc --csharp_out=./ messages.proto
```

Then you can see the Messages.cs file created in the protocols folder.

### Registering Message

To use the newly created message, it must be pre-registered to ProtocolManager with a value that is identical to the server. If it is not registered or different than that of the server, it may not work, malfunction, or trigger an exception.

```
// The value registered must be equal to that of the server.
ProtocolManager.getInstance().RegisterProtocol(0, Messages.MessagesReflection.Descriptor);
```

### Transferring Message

Send a message using Request() and wait for the server to respond. While waiting for the server response, additional Requests() are stored in the queue and processed later when the server response is resolved. To receive and process a server response, you must register a listener. If no listener is registered, the next message is registered without separate notification even with a server response. If there is no response within a set time, a timeout occurs and next message is processed. The timeout is transferred to the OnError() listener as ErrorCode.TIMEOUT.

When a message is sent via Send(), it is instantly delivered to the server when Send() is called and does not wait for a separate response. Messages sent via Send() are immediately transferred to the server even while waiting for the response for Request().

```
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
connection.AddListener((ConnectionAgent connection, Messages.SampleReceive msg)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
connection.Send(sampleSend);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
connection.Request(sampleRequest);
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
user.AddListener((UserAgent user, Messages.SampleReceive msg)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
user.Send(sampleSend);

Messages.SampleRequest SampleRequest = new Messages.SampleRequest();
user.Request(SampleRequest);
```

### Custom Message

Arbitrary data except ProtocolBuffer can be serialized as byte stream and used via packet class. For more information on packet, visit [here](./unity-06-packet).

```
Connector connector = new Connector();
ConnectionAgent connection = connector.GetConnectionAgent();
int reqMsgId = 1;
int resMsgId = 2;

connection.AddListener(resMsgId, (ConnectionAgent connection, Packet packet)=> { });

Messages.SampleSend sampleSend= new Messages.SampleSend (); 
Packet sampleSendPacket = new Packet(reqMsgId, sampleSend.ToByteArray())
connection.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
Packet sampleRequestPacket = new Packet(reqMsgId, sampleRequest.ToByteArray())
connection.Request(sampleRequestPacket);
connection.Request(sampleRequestPacket, (ConnectionAgent connection, Packet packet)=> { });
Connector connector = new Connector();
UserAgent user = connector.CreateUser(serviceId, subId);
int reqMsgId = 1;
int resMsgId = 2;

user.AddListener(resMsgId, (UserAgent user, Packet packet)=> { });

Messages.SampleSend sampleSend = new Messages.SampleSend (); 
Packet sampleSendPacket = new Packet(reqMsgIndex, sampleSend.ToByteArray())
user.Send(sampleSendPacket);

Messages.SampleRequest sampleRequest = new Messages.SampleRequest();
Packet sampleRequestPacket = new Packet(reqMsgIndex, sampleRequestPacket.ToByteArray())
user.Request(sampleRequestPacket);
user.Request(sampleRequestPacket, (UserAgent user, Packet packet)=> { });
```

## Packet

All the messages transferred between the server are processed in a packet module and the interface provided by packet module is used.

### Creation

The GameAnvil connector uses Google Protocol Buffer as the default protocol. Creating a packet using Google Protocol Buffer is as below:

```
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
```

Packet

```
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Pcket.CreateWithCustomMsg(customMsgId, requestMsg);

byte[] bytes =  packet.GetBytes();
string JsonString = Encoding.UTF8.GetString(bytes);
```

### Compression

If the size of a packet is too large, it can be compressed to reduce data usage.

```
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
packet.compress();

if (packet.isCompress())
    packet.decompress();

CustomMessage.SampleResponse responseMsg = packet.GetMessage<CustomMessage.SampleResponse>();
```

## Payload

When using the default API provided by GameAnvil, you may need additional data. For this, the default API includes the payload parameter to deliver additional data. The data required for the payload can be stored in a packet in a list form. You can add more data here and send it to the server or extract the message sent to the server.

```
using GameAnvil;
Messages.UserInfo userInfo = new Messages.UserInfo();
Messages.RoomInfo roomInfo = new Messages.RoomInfo();

Payload payload = new Payload();
payload.add(new Packet(userInfo));
payload.add(new Packet(roomInfo));

Packet packet = payload.getPacket<Messages.UserInfo>();
Messages.RoomInfo roomInfo = payload.GetMessage<Messages.RoomInfo>()
```

## Closing GameAnvil Connector

Disconnecting via calling the Connector.CloseSocket() function before shutting down the game play is recommended. Otherwise, the server may not recognize that the client has shut down and keep performing unnecessary tasks.
