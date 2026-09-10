<!-- pre-align:aligned sig=dd9a8996ee81 -->

<a id="game-gameanvil-unity-advanced-development-guide-packet"></a>
## Game > GameAnvil > Unity Advanced Development Guide > Packet { #game-gameanvil-unity-advanced-development-guide-packet }

<a id="packets"></a>
## Packets { #packets }

All messages to and from the server are handled by the Packet module and use the interfaces provided by the Packet module.

<a id="create"></a>
### Create { #create }

The connector uses Google Protocol Buffers as the default protocol. Packet generation using Google Protocol Buffers is as follows

```c#
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
```

You can create packets without using Google Protocol Buffers.

```c#
byte[] requestMsg = Encoding.UTF8.GetBytes(JsonString);
Packet packet = Pcket.CreateWithCustomMsg(customMsgId, requestMsg);

byte[] bytes =  packet.GetBytes();
string JsonString = Encoding.UTF8.GetString(bytes);
```

The maximum size of a packet is limited to 64 Kbytes. If the packet size is larger than 64 Kbytes, you can use compression to get around the size limit.
Payloads are also treated internally as packets, so they too cannot be larger than 64 Kbytes.

<a id="compression"></a>
### Compression { #compression }

If the packet size is large, you can compress it to reduce data usage.  

```c#
Messages.SampleRequest requestMsg = new Messages.SampleRequest();
Packet packet = new Packet(requestMsg);
packet.compress();

if (packet.isCompress())
    packet.decompress();

CustomMessage.SampleResponse responseMsg = packet.GetMessage<CustomMessage.SampleResponse>();
```

<a id="sent"></a>
### Sent { #sent }

<!-- TODO: translate body -->

<a id="payload"></a>
### Payload { #payload }

When using the native APIs provided by GameAnvil, additional data may be required. For this purpose, the basic APIs include a parameter called a payload that allows you to pass additional data. The data required by this payload can be packed into packets and stored in a list format. You can add additional data to it and send it to the server, or retrieve messages sent by the server. 

```c#
using GameAnvil;
Messages.UserInfo userInfo = new Messages.UserInfo();
Messages.RoomInfo roomInfo = new Messages.RoomInfo();

Payload payload = new Payload();
payload.add(new Packet(userInfo));
payload.add(new Packet(roomInfo));

Packet packet = payload.getPacket<Messages.UserInfo>();
Messages.RoomInfo roomInfo = payload.GetMessage<Messages.RoomInfo>()
```
