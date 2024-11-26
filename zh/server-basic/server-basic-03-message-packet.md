## Game > GameAnvil > 서버 개념 설명 > 메세지 처리와 패킷

## 패킷

패킷은 GameAnvil 에서 서버와 클라이언트간 메세지를 전달하는 단위입니다. 자바의 프로토 버퍼나 빌더를 사용하여 패킷을 생성할 수 있습니다. 패킷은 GameAnvil 엔진에서 지원하는 여러 타입에서 만들 수 있는데 간단한 사용법은 다음과 같습니다.
```java
// 응답 프로토 버퍼 작성
EchoSend res = EchoSend.newBuilder()
    .setData("hello");

// 전달 패킷 생성
Packet packet = Packet.makePacket(res);

// 클라이언트로 전달
userContext.send(packet);
```

위 코드는 게임 유저에서 클라이언트로 패킷을 전달하는 코드입니다. 먼저 전달할 프로토 버퍼를 정의한 뒤 패킷을 생성하고 클라이언트에게 전달합니다. 

## 패킷 활용 성능 최적화 

패킷은 내부적으로 프로토 버퍼를 직렬화 한 데이터의 캐싱을 하고 있습니다. 그러므로 여러 유저에게 프로토 버퍼 메세지를 보낼 때는 패킷을 여러 번 만들어서 보내는 대신 한번만 만들어서 보내는게 좋습니다. 아래 예제에서는 2명의 유저에게 패킷의 얕은 복사를 하여 전달하는 방법을 다루고 있습니다. 
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
`duplicate` 함수는 얕은 복사를 합니다. GameAnil 엔진은 내부적으로 패킷이 정상적으로 처리되지 않았는지 검사를 하고 있기 때문에 재활용 시 정상적으로 처리되지 않을 수 있습니다. 이때 패킷의 얕은 복사를 하여 새로운 패킷처럼 취급하면 GameAnvil 은 다른 패킷으로 인식하고 다시 카운트 하게 됩니다. 

물론 이러한 추가는 코드 읽기가 어려울 수 있고 위 예제와 같이 단순한 코드는 직렬화 작업을 많이 하더라도 실제 성능 상 크게 차이가 없을 수 있습니다. 많은 데이터가 들어가는 패킷을 전송하고 성능 최적화가 필요한 시점에서 이러한 구문을 작성하는것이 좋겠습니다. 