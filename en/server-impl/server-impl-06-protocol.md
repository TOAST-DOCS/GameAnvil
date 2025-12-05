## Game > GameAnvil > Server Development Guide > Protocol Definition



##  Protocol Definition and Compile

GameAnvil uses [Google Protocol Buffers](https://protobuf.dev/) to define and build protocols. The examples below describe how to define and build these protocols. Firstly, create SampleGame.proto file as a text editor and define the desired protocol. For more detailed grammar on protocol buffer, refer to [Official Protocol Buffers Guide](https://protobuf.dev/programming-guides/proto3/).

```protobuf
package [Package name];  
  
// If any other proto file to refer to import  
import [proto file name]  
  
enum SampleUserTypeEnum {  
    USER_TYPE_NONE = 0;  
    USER_TYPE_ONE = 1;  
}  
  
message SampleUserData {  
    int32 data = 1;  
}  
  
enum SampleRoomTypeEnum {  
    ROOM_TYPE_NONE = 0;  
    ROOM_TYPE_ONE = 1;  
}  
  
message SampleRoomData {  
    int32 data = 1;  
}  
  
message SampleUserMessage {  
    SampleUserTypeEnum sampleUserTypeEnum = 1;  
    int32 id = 2;  
    int64 money = 3;  
    string nickname = 4;  
    double score = 5;  
    repeated SampleUserData sampleUserData = 6; // list  
}  
  
message SampleRoomMessage {  
    SampleRoomTypeEnum sampleRoomTypeEnum = 1;  
    int32 id = 2;  
    int64 potMoney = 3;  
    string roomName = 4;  
    repeated SampleRoomData sampleRoomData = 5; // list  
}
```

You can also compile protocol scripts from the following command lines. At this time, it is convenient to create and use shell scripts or batch files. If you have configured a project using GameAnvil templates, you can refer to protocol scripts and batch files in the proto folder within the project. If compilation is successful, all Java classes related to the protocol are automatically created.

```bash
protoc  ./MyGame.proto --java_out=../java
```

If you need the C# protocol for Unity client, you can also create it by adding a language, as shown below. In this case, you may need to manually copy C# result file to Unity project according to its generation path.

```bash
protoc ./MyGame.proto --java_out=../java --csharp_out=./
```


## GeneratedMessageV3 and packet

GameAnvil servers support most classes of `com.google.protobuf.GeneratedMessageV3` so that proto-buffer objects can be used in any transferable method. In normal situations, there is no problem with using proto buffer object as it is, but in certain situations, such as sending it to multiple clients, you can use `com.nhn.gameanvil.packet.Packet` class to improve performance.

When sending a message to multiple clients, you can use it like this:
```java
/**
 * Send a message to the user clients in the list provided.
 *
 * @param userList List of users to send the message to.
 * @param message Message to send.
 * @param <P> Type of message to send.
 */
default <P extends GeneratedMessage.Builder<P>> void sendToClients(@NotNull final Collection<IUserContext> userList, @NotNull final GeneratedMessage.Builder<P> message) {
}

/**
 * Send a message to the user clients in the list provided.
 *
 * @param userList List of users to send the message to.
 * @param message Message to send.
 */
default void sendToClients(@NotNull final Collection<IUserContext> userList, @NotNull final GeneratedMessageV3 message) {
}

/**
 * Send a message to the user clients in the list provided.
 *
 * @param userList List of users to send the message to.
 * @param packet   Packet to send
 */
void sendToClients(@NotNull final Collection<IUserContext> userList, @NotNull final Packet packet)