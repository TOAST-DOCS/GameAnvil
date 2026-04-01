## Game > GameAnvil > Server Development Guide > Transferable Objects

## Object Transfer

In GameAnvil, an object transfer is the movement of an object from one node to another. All of the object transfers you should be interested in occur between game nodes. We'll discuss two of the most common: user transfer and room transfer.

## User Transfer (UserTransfer)

![gamenode-user-transfer2.png](https://static.toastoven.net/prod_gameanvil/images/gamenode-user-transfer2.png)

User objects are created in game node. This game user object can be transferred at any time among game nodes. For example, user objects are transferred among game nodes while the user object of #1 game node is entering the room created by the user object of #2 game node. This technology is the core of GameAnvil and it works as intended only when user object is correctly serialized/de-serialized. The target of serialize/de-serialize can be directly implemented by GameAnvil users. In order words, they decide what values of the game user object should be transferred.

This type of user transfers can be categorized in three types.

- First, user objects can be transferred to a different game node by room creation or room join logic including matchmaking. At this time, the objects can enter the room in a different channel, according to implementation.
- Second, it occurs when switching channels. A game node can belong to a channel. Therefore, switching channels means moving through game nodes.
  - When entering the room in a different channel using matchmaking as in the first case, it is processed implicitly inside the engine.
  - The client can request channel switch explicitly.
- Third, when uninterrupted maintenance for arbitrary game nodes NonStopPatch is executed, the user objects of the node are distributed to other valid game nodes and transferred. In this case,
explicit command is issued on GameAnvil Console by admins.

### User Transfer Implementation

Actual user transfer is internally processed by GameAnvil. At this time, the client does not recognize that its own game user object is being transferred among servers. In other words, even when it enters the room of a different game node, the client enters an arbitrary room in the sole GameAnvil server family.

However, when user objects are being transferred to a different game node, the user can determine which data to be transferred with them. Simply specifying the data allows it to be serialized with the user object as part of it. If done, the objects can easily be accessed and used without concerning de-serialization from the game node to which transfer the data.

The following are the callback methods related to user transfer. First to do is 'specifying the data to be transferred at the same time.' To do this, inherit the BaseUser abstract class and implement the callback below in the game user class. The user simply has to place the data to be transferred in the transferPack passed as the key-value pair using the key value that the user wants.

If the data to be transferred is not specified at this time, be careful as the data of the user object that is transferred to the target game node is reset to default.

```java

@Override
public void onTransferOut(ITransferPack transferPack) {
    // Store user data that needs to be transferred
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("TotalMoney", myTotalMoney);
    transferPack.put("Message", myMessage);
}
```

Now, let's take a look at the callback method to be processed in the target game node after user transfer is complete. The desired object can be accessed by using the key specified before the transfer. Restore the data of the user object that is transferred in this way.

```java

@Override
public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {
    // Resave received user data
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setTotalMoney(transferPack.getToLong("TotalMoney"));
    setTransferMessage(transferPack.getToString("Message"));

    // When a timer key requiring registration is delivered, timer re-registration is processed
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)) {
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
}
```

The two methods above are automatically called by GameAnvil. The user only has to implement them.

## Room Transfer (RoomTranfer)

![gamenode-room-transfer2.png](https://static.toastoven.net/prod_gameanvil/images/gamenode-room-transfer2.png)

Room transfer is a feature similar to the user transfer that is explained in the intermediate concepts. It is different only in that room, a bigger concept than user, is transferred. In game node, a room object with one or more game users can be created. This room object can be transferred among game nodes just like user object. For example, the room object of #1 game node can be transferred to the #2 game node. When the room is being transferred, the users in that room are also transferred. For this reason, the game can be played without interruption before and after the room is transferred. As the room is transferred quickly, the users in the room can continue playing the game without noticing the transfer. This technology is the core of GameAnvil's NonStopPatch and it only works when the room object is serialized/de-serialized well. The target of serialize/de-serialize can be directly implemented by GameAnvil users. In other words, the users determine which value will be transferred to the room object.

Only the NonStopPatch command can trigger this type of room transfer. This command is explicitly transferred by the game operator through GameAnvil Console.

### Implement Room Transfer and Transferable Room Timer

The actual room transfer is internally processed by GameAnvil. At this time, the client is not likely to recognize the room object being transferred among the servers with game users. Unless a special problem occurs, the overall flow is quickly progressed and the game flow before the transfer can be continued after the transfer.

At this time, when transferring room object to a different game node, the user can specify which data will be transferred as well. Simply specifying the data allows it to be serialized as part of the room object. In the game node to be transferred, the user can easily access those objects without considering de-serialization.

The following are the callback methods related to user transfer. First is "specifying the data to be transferred with." To do this, implement the callback below in the game room class that inherited the BaseRoom abstract class.

Simply place the data to be transferred to the passed transferPack as a key-value pair using the key value desired by the user.

If the data to be transferred is not specified at this time, be careful as the data of the room object that is transferred to the target game node is reset to default.

> [Note]
>
> As explained earlier, when room is transferred, the users in the room are transferred as well. As user transfer is explained in the intermediate concepts, it will not covered here.*

```java

@Override
public void onTransferOut(ITransferPack transferPack) {
    // Save room data needed to be sent
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("RoomInfo", myRoomInfo);
}
```

Let's take a look at the callback methods that need to be processed by the target game node after the transfer. Users can access the desired object before it is transferred using the specified key. Restore the data of the room object that is transferred in this way. Especially, users can see that the list of the user objects in the room transferred is passed as a parameter. Except this, the overall flow is very similar to that of user transfer.

```java

@Override
public void onTransferIn(List<GameUser> userList, ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {
    this.users.clear();
    for (GameUser user : userList) {
        this.users.put(user.getUserId(), user);
    }

    // Resave received room data
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setRoomInfo(transferPack.getToLong("RoomInfo"));

    // When a timer key requiring registration is delivered, timer re-registration is processed
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)) {
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
}