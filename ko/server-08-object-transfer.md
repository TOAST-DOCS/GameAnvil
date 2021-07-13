## Game > GameAnvil > 서버 개발 가이드 > 전송 가능 객체



## 1. 객체 전송(Object Transfer)

GameAnvil에서 객체 전송이란 하나의 노드에서 다른 노드로 객체가 이동하는 것을 의미합니다. 사용자가 관심을 가져야할 객체 전송은 모두 게임 노드 사이에서 발생합니다. 그 대표적인 두 가지인 유저 전송과 룸 전송에 대해 살펴봅니다.



## 2. 유저 전송 (UserTransfer)

![gamenode-user-transfer2.png](https://static.toastoven.net/prod_gameanvil/images/gamenode-user-transfer2.png)

게임 노드에는 게임 유저 객체가 생성됩니다. 이 게임 유저 객체는 게임 노드 사이에서 언제든지 전송될 수 있습니다. 예를 들면 1번 게임 노드의 유저 객체가 2번 게임 노드에 생성된 방으로 들어가는 과정에서 게임 노드 간 유저 객체 전송이 일어납니다. 이 기술은 GameAnvil의 가장 핵심이자 근간이 되며 유저 객체를 잘 직렬화/역직렬화해야 의도한 대로 동작합니다. 이러한 직렬화/역직렬화 대상은 GameAnvil 사용자가 직접 구현할 수 있습니다. 즉, 해당 게임 유저 객체의 어떤 값을 전송할 것인지 결정하는 것이죠.

이러한 유저 전송이 발생하는 경우는 크게 세 가지로 나눌 수 있습니다.

- 첫째, 매치 메이킹을 포함한 방 생성이나 방 참여 로직에 의해 유저 객체는 다른 게임 노드로 전송될 수 있습니다. 이때, 구현에 따라 다른 채널의 방으로 진입할 수도 있습니다.
- 둘째, 다른 채널로 이동할 때 발생합니다. 하나의 게임 노드는 하나의 채널에 속할 수 있습니다. 그러므로 채널을 이동하는 것은 게임 노드 사이의 이동을 의미합니다.
  - 첫 번째 경우처럼 매치 메이킹 등을 이용해서 다른 채널의 방으로 진입할 경우에는 엔진 내부에서 묵시적으로 처리합니다.
  - 클라이언트는 명시적으로 채널 이동을 요청할 수도 있습니다.
- 셋째, 임의의 게임 노드에 대해 무정지 점검(NonStopPatch)을 진행하면 해당 노드의 유저 객체들은 다른 유효한 게임 노드들로 분산되어 전송됩니다. 이 경우는 운영 측면에서 GameAnvil Console을 통해 명시적으로 명령을 내린 경우입니다.



### 2-1. 유저 전송 구현

실제 유저 전송은 GameAnvil이 내부적으로 조용하게 처리합니다. 이때, 클라이언트는 자신의 게임 유저 객체가 서버 사이에서 전송되는지 인지하지 못합니다. 즉, 다른 게임 노드의 방으로 들어가더라도 클라이언트는 단지 하나의 GameAnvil 서버군에서 임의의 방으로 들어간 것뿐이죠.

단, 다른 게임 노드로 유저 객체를 전송할 때 어떤 데이터를 함께 전송할지는 사용자가 지정할 수 있습니다. 간단히 지정만 해두면 이 데이터는 유저 객체의 일부로 함께 직렬화됩니다. 전송할 대상 게임 노드에서는 역직렬화를 신경 쓸 필요 없이 쉽게 해당 객체들에 접근해서 사용할 수 있습니다.

다음은 이러한 유저 전송에 관련된 콜백 메서드들입니다. 우선 '함께 전송할 데이터 지정하기'입니다. 이를 위해 BaseUser 추상 클래스를 상속받은 게임 유저 클래스에 아래의 콜백을 구현합니다. 방법은 간단합니다. 아래의 예제와 같이 전송할 데이터를 사용자가 원하는 key 값을 이용하여 key-value 쌍으로 매개변수로 넘겨받은 transferPack에 넣기만 하면 됩니다.

만일 여기에서 전송할 데이터를 지정하지 않으면 대상 게임 노드로 전송된 유저 객체의 해당 데이터는 모두 기본값으로 초기화되므로 주의가 필요합니다.

```java
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("TotalMoney", myTotalMoney);
    transferPack.put("Message", myMessage);
}
```

이제 유저 전송이 완료된 후 대상 게임 노드에서 처리할 콜백 메서드를 살펴보겠습니다. 아래와 같이 전송 전에 지정한 key를 이용해서 원하는 객체에 접근할 수 있습니다. 이와 같은 방법으로 전송 완료된 유저 객체의 해당 데이터를 원래 상태로 만들어 줍니다.

```java
@Override
public void onTransferIn(TransferPack transferPack) throws SuspendExecution {
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setTotalMoney(transferPack.getToLong("TotalMoney"));
    setTransferMessage(transferPack.getToString("Message"));
}
```

위의 2가지 메서드는 GameAnvil이 알아서 호출합니다. 사용자는 그냥 구현만 하면 됩니다.



### 2-2. 전송 가능한 유저 타이머

유저가 전송될 때 유저에 등록해둔 타이머도 함께 전송할 수 있습니다. 전송 가능한 타이머를 사용하기 위해서는 별도로 아래의 콜백 메서드에서 원하는 key를 이용해서 미리 등록해두어야 합니다. Timer 핸들러를 다음과 같이 원하는 key로 매핑해둡니다.

```java
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferUserTimerHandler1", transferUserTimerHandler1());
     registerTimerHandler("transferUserTimerHandler2", (timerObject, object) -> {
         GameUser user = (GameUser) object;
         logger.warn("GameUser::transferUserTimerHandler2() : userId({})", user.getId());
     });
 }
```

등록이 완료된 타이머 객체는 언제든 해당 key를 이용해서 아래와 같이 게임 유저 구현부에서 사용할 수 있습니다. 단순히 등록만 한 타이머는 효과가 없으므로 실제 사용을 위해서는 반드시 아래와 같이 유저 객체에 추가해야 타이머가 발동합니다.

```java
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

이렇게 유저 객체에 추가해 둔 타이머는 별도로 전송에 대한 처리를 하지 않아도 모두 자동으로 전송됩니다. 즉, 대상 게임 노드에서 해당 유저 객체에 대해 동일한 타이머 추가 과정을 거칠 필요가 없습니다.



## 3. 방 전송 (RoomTranfer)

![gamenode-room-transfer2.png](https://static.toastoven.net/prod_gameanvil/images/gamenode-room-transfer2.png)

방 전송은 중급 개념에서 설명한 유저 전송과 매우 비슷한 기능입니다. 유저 보다 큰 개념인 방이 전송되는 차이가 있을 뿐입니다. 게임 노드에는 한 명 이상의 게임 유저가 참여한 방 객체가 생성될 수 있습니다. 이 방 객체는 유저 객체와 마찬가지로 게임 노드 사이에서 전송될 수 있습니다. 예를 들면 1번 게임 노드의 방 객체가 2번 게임 노드로 전송이 되는 것이죠. 이렇게 방이 전송될 때 당연하게도 방 안에 존재하는 유저들도 함께 전송이 일어납니다. 그 덕분에 방이 전송되기 전/후의 게임 흐름은 연속되어 진행될 수 있습니다. 즉, 해당 방 객체가 전송되는 과정은 매우 빠르게 진행되므로 방 안의 유저들은 인지하지 못한 상태에서 게임을 지속할 수 있습니다.  이 기술은 GameAnvil 무점검 패치(NonStopPatch)의 핵심이자 근간이 되며 방 객체를 잘 직렬화/역직렬화해야 의도한대로 동작합니다. 이러한 직렬화/역직렬화 대상은 GameAnvil 사용자가 직접 구현할 수 있습니다. 즉, 해당 방 객체의 어떤 값을 전송할 것인지 결정하는 것이죠.

이러한 방 전송을 발생시키는 것은 오직 무점검 패치(NonStopPatch) 명령 뿐입니다. 이 명령은 일반적으로 GameAnvil Console을 통해 게임 운영자가 명시적으로 전달합니다.



### 3-1. 방 전송 구현

실제 방 전송은 GameAnvil이 내부적으로 조용하게 처리합니다. 이때, 클라이언트는 자신의 게임 유저와 더불어 자신이 속한 방 객체가 서버 사이에서 전송되는지 인지하지 못할 가능성이 높습니다. 특별한 문제가 발생하지 않는 한 전체 흐름이 매우 빠르게 진행되기 때문에 전송 전의 게임 흐름을 전송 후에 계속 이어감에 있어 무리가 없습니다.

이때, 다른 게임 노드로 방 객체를 전송할 때 어떤 데이터를 함께 전송할지는 사용자가 지정할 수 있습니다. 간단히 지정만 해두면 이 데이터는 방 객체의 일부로 함께 직렬화됩니다. 전송할 대상 게임 노드에서는 역직렬화를 신경 쓸 필요 없이 쉽게 해당 객체들에 접근해서 사용할 수 있습니다.

다음은 이러한 유저 전송에 관련된 콜백 메서드들입니다. 우선 "함께 전송할 데이터 지정하기"입니다. 이를 위해 BaseRoom 추상 클래스를 상속받은 게임 룸 클래스에 아래의 콜백을 구현합니다. 방법은 간단합니다. 아래의 예제와 같이 전송할 데이터를 사용자가 원하는 key값을 이용하여 key-value 쌍으로 매개변수로 넘겨받은 transferPack에 넣기만 하면 됩니다.

만일 여기에서 전송할 데이터를 지정하지 않으면 대상 게임 노드로 전송된 방 객체의 해당 데이터는 모두 기본값으로 초기화되므로 주의가 필요합니다.

**참고**> *앞서 설명하였듯이 방이 전송될 때는 당연하게도 방 안의 유저들이 함께 전송됩니다. 유저 전송에 관해서는 중급 개념에서 설명했으므로 여기에서는 따로 설명하지 않습니다.*

```java
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("RoomInfo", myRoomInfo);
}
```

이제 방 전송이 완료된 후 대상 게임 노드에서 처리할 콜백 메서드를 살펴보겠습니다. 아래와 같이 전송 전에 지정한 key를 이용해서 원하는 객체에 접근할 수 있습니다. 이와 같은 방법으로 전송 완료된 방 객체의 해당 데이터를 원래 상태로 만들어 줍니다. 특히, 전송된 방 안의 유저 객체 리스트를 매개 변수로 전달받고 있음을 확인할 수 있습니다. 이 부분만 제외하면 전체적인 흐름은 유저 전송과 매우 흡사합니다.

```java
@Override
public void onTransferIn(List<GameUser> userList, TransferPack transferPack) throws SuspendExecution {
    this.users.clear();
    for (GameUser user : userList)
        this.users.put(user.getUserId(), user);

    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setRoomInfo(transferPack.getToLong("RoomInfo"));
}
```



### 3-2. 전송 가능한 방 타이머

방이 전송될 때 방에 등록해둔 타이머도 함께 전송할 수 있습니다. 전송 가능한 타이머를 사용하기 위해서는 별도로 아래의 콜백 메서드에서 원하는 key를 이용해서 미리 등록해두어야 합니다. Timer 핸들러를 다음과 같이 원하는 key로 매핑해둡니다. 이는 유저 전송과 완전히 동일합니다.

```java
 @Override
 public void onRegisterTimerHandler() {
     registerTimerHandler("transferRoomTimerHandler1", transferRoomTimerHandler1());
     registerTimerHandler("transferRoomTimerHandler2", (timerObject, object) -> {
         GameRoom room = (GameRoom) object;
         logger.warn("GameRoom::transferRoomTimerHandler2() : roomId({})", room.getId());
     });
 }
```

등록이 완료된 타이머 객체는 언제든 해당 key를 이용해서 아래와 같이 게임 룸 구현부에서 사용할 수 있습니다. 단순히 등록만 한 타이머는 효과가 없으므로 실제 사용을 위해서는 반드시 아래와 같이 유저 객체에 추가해야 타이머가 발동합니다.

```java
addTimer(1, TimeUnit.SECONDS, 20, "transferRoomTimerHandler1", false);
addTimer(2, TimeUnit.SECONDS, 0, "transferRoomTimerHandler2", false);
```

이렇게 방 객체에 추가해 둔 타이머는 별도로 전송에 대한 처리를 하지 않아도 모두 자동으로 전송됩니다. 즉, 대상 게임 노드에서 해당 방 객체에 대해 동일한 타이머 추가 과정을 거칠 필요가 없습니다.

