<!-- pre-align:aligned sig=6470327d4b32 -->

<a id="game-gameanvil-server-concept-description-packet"></a>
## Game > GameAnvil > Server Concept Description > Packet { #game-gameanvil-server-concept-description-packet }

<a id="section-1"></a>
## Packet { #section-1 }

A packet is the unit used to transfer messages between a server and a client in GameAnvil. You can create packets by using Java's protocol buffers or builders. Packets can be created from various types supported by the GameAnvil engine. Here is how to use them:
```java
// 응답 프로토 버퍼 작성
EchoSend res = EchoSend.newBuilder()
    .setData("hello");

// 전달 패킷 생성
Packet packet = Packet.makePacket(res);

// 클라이언트로 전달
userContext.send(packet);
```

The code above is the code that delivers a packet from a game user to the client. First, define the protocol buffer to be delivered, then create a packet and deliver it to the client.

<a id="section-2"></a>
## Optimize Performance { #section-2 }

Packets cache the serialized proto buffer data internally. Therefore, when sending a proto buffer message to multiple users, it is better to create the packet only once instead of creating it multiple times. The example below demonstrates how to send a packet to two users by making a shallow copy of the packet.
```java
EchoSend.Builder res = EchoSend.newBuilder()
    .setData("hello");

List<MyUser> allUsers = roomContext.getAllUsers();

// 유저가 여러명 있다고 가정하고 
// 모든 유저에게 메세지를 보냅니다.

// 나쁜 방법
for (GameUser myUser : allUsers) {
    IUserContext userContext = user.getUserContext();
    userContext.send(Packet.makePacket(res)); // 이때 res를 전송 시 유저 수 만큼 직렬화 합니다 주의!
}

// 좋은 방법
Packet packet = Packet.makePacket(res); // 이 패킷은 멤버 변수 등으로 저장 후 여러번 재활용 가능합니다
                                        // 단 패킷을 만든 후 res 의 수정은 반영되지 않습니다
for (GameUser myUser : allUsers) {
    IUserContext userContext = user.getUserContext();
    userContext.send(packet.duplicate()); // 패킷의 얕은 복사를 합니다 직렬화는 1번!
}
```
The `duplicate` function performs a shallow copy. Because the GameAnvil engine internally checks whether a packet has been processed normally, reusing a packet may cause it to not be processed normally. In this case, performing a shallow copy of the packet and treating it as a new packet causes GameAnvil to recognize it as a different packet and count it again.

Of course, this addition can make the code harder to read, and for simple code like the example above, there may not be a significant difference in actual performance even with more serialization operations. We recommend that you write this type of code when you are sending packets that contain large amounts of data and performance optimization is needed.