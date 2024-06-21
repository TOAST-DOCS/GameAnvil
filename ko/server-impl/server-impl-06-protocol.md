## Game > GameAnvil > 서버 개발 가이드 > 프로토콜 정의



## 프로토콜 정의와 컴파일

GameAnvil은 [Google Protocol Buffers](https://developers.google.com/protocol-buffers)를 사용하여 프로토콜을 정의하고 빌드합니다. 아래의 예제는 이러한 프로토콜을 정의하는 법과 빌드하는 법을 설명합니다. 우선 SampleGame.proto 파일을 텍스트 에디터로 생성한 후 원하는 프로토콜을 정의합니다. 프로토콜 버퍼의 자세한 문법은 [공식 Protocol Buffers 가이드](https://developers.google.com/protocol-buffers/docs/proto3)를 참고할 수 있습니다.

```protobuf
package [패키지명];

// 참조해야 할 다른 proto 파일이 있다면 import
import [proto 파일명]

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

또한 아래와 같은 명령줄로 프로토콜 스크립트를 컴파일할 수 있습니다. 이때 셸 스크립트나 배치 파일을 생성해 두고 사용하면 편리합니다. GameAnvil 템플릿을 이용해 프로젝트를 구성한 경우 프로젝트 내의 proto 폴더에서 프로토콜 스크립트와 배치 파일을 참고할 수 있습니다. 컴파일이 성공적으로 완료되면 프로토콜에 관련된 모든 Java 클래스가 자동으로 생성됩니다.

```bash
protoc  ./MyGame.proto --java_out=../java
```

유니티 클라이언트를 위한 C# 프로토콜이 필요할 경우 아래와 같이 언어를 추가하여 생성할 수도 있습니다. 이 경우 C# 결과 파일은 그 생성 경로에 따라 수동으로 유니티 프로젝트에 복사해 주어야 할 수도 있습니다.

```bash
protoc ./MyGame.proto --java_out=../java --csharp_out=./
```


## GeneratedMessageV3와 패킷

GameAnvil 서버에서는 어떠한 전송이 가능한 메서드에서도 프로토 버퍼 객체를 그대로 사용할 수 있도록 대부분 `com.google.protobuf.GeneratedMessageV3` 클래스를 지원합니다. 일반적인 상황에서는 프로토 버퍼 객체를 그대로 사용하여도 문제가 없지만 여러 명의 클라이언트에게 전송하는 등 특정 상황에서는 `com.nhn.gameanvil.packet.Packet` 클래스를 사용하여 성능을 향상시킬 수 있습니다.

전달된 GeneratedMessageV3 클래스는 내부적으로 패킷으로 변환되어 직렬화 후 전송하기 때문에 아래의 `broadcastMessage`를 호출하는 상황에서는 패킷으로 변경하는 과정에서 같은 프로토 버퍼를 여러 번 직렬화하는 문제가 발생할 수 있습니다.

```java
// 아래와 같은 상황에서는 여러 번 직렬화하여 성능 문제가 발생할 수 있습니다.
void broadcastMessage(List<GameUser> users, GeneratedMessageV3 message) {
    for (GameUser user : users) {
        user.send(message);
    }
}
```

이러한 상황을 방지하기 위해 아래와 같이 수정합니다. 패킷 객체에서는 내부적으로 직렬화된 정보를 들고 있어 여러 번 전송해도 정상적으로 동작합니다.

```java
// 성능 문제 수정
void broadcastMessage(List<GameUser> users, GeneratedMessageV3 message) {
    Packet p = Packet.makePacket(message); 
    for (GameUser user : users) {
        user.send(p);
    }
}
```

아래와 같은 상황에서는 패킷을 사용하지 않아도 크게 문제가 없으며 코드 가독성이 떨어질 수 있기 때문에 프로토 버퍼 객체를 그대로 넘기는 것이 좋습니다.

```java
// 아래와 같은 상황에서는 GeneratedMessageV3 사용이 편리합니다. 
void sendMessage(GameUser user, GeneratedMessageV3 message) {
    user.send(message);
}
```