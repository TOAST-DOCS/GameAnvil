## Game > GameAnvil > サーバー開発ガイド > 転送可能オブジェクト



## オブジェクト転送(Object Transfer)

GameAnvilにおけるオブジェクト転送とは、1つのノードから他のノードにオブジェクトが移動することを意味します。ユーザーが注目すべきオブジェクトの転送はすべてゲームノード間で発生します。その代表的な2つのユーザー転送とルーム転送について説明します。



## ユーザー転送(UserTransfer)

![gamenode-user-transfer2.png](https://static.toastoven.net/prod_gameanvil/images/gamenode-user-transfer2.png)

ゲームノードには、ゲームユーザーオブジェクトが作成されます。このゲームユーザーオブジェクトは、ゲームノード間でいつでも送信されることがあります。例えば、1番のゲームノードのユーザーオブジェクトが2番のゲームノードに作成されたルームに入る過程で、ゲームノード間のユーザーオブジェクト転送が発生します。この技術はGameAnvilの主要かつ根幹となり、ユーザーオブジェクトを正常にシリアル化/逆シリアル化しなければ、意図したとおりに動作しません。このようなシリアル化/逆シリアル化の対象は、GameAnvilユーザーが直接実装できます。つまり、該当ゲームユーザーオブジェクトのどのような値を転送するかどうかを決定することです。

このようなユーザー転送が発生する状況は、大きく3つに分けられます。

- 第一に、マッチメイキングを含むルームの作成やルームへの参加ロジックによって、ユーザーオブジェクトは他のゲームノードに転送されることがあります。この時、実装に応じて別のチャンネルのルームに入場することもあります。
- 第二に、別のチャンネルに移動する時に発生します。1つのゲームノードは1つのチャンネルに属することができます。従って、チャンネルの移動は、ゲームノード間の移動を意味します。
  - 1つ目の場合のように、マッチメイキングなどを利用して別のチャンネルのルームに入場した場合は、エンジン内部で黙示的に処理します。
  - クライアントは、明示的にチャンネル移動をリクエストすることもできます。
- 第三に、任意のゲームノードに対して無停止点検(NonStopPatch)を行うと、該当ノードのユーザーオブジェクトは別の有効なゲームノードに分散して転送されます。この場合は、運営面でGameAnvil Consoleを介して明示的にコマンドを実行した場合です。



### ユーザー転送の実装

実際にユーザー転送はGameAnvilが内部的に静かに処理します。この時、クライアントは自分のゲームユーザーオブジェクトがサーバー間で転送されたか認識できません。つまり、別のゲームノードのルームに入場しても、クライアントはただ1つのGameAnvilサーバー群から任意のルームに入場しただけとなります。

ただし、別のゲームノードにユーザーオブジェクトを転送した時、どのデータを一緒に転送するかはユーザーが指定できます。指定するだけで、このデータはユーザーオブジェクトの一部と共にシリアル化されます。転送対象のゲームノードでは、逆シリアル化を気にすることなく、簡単に該当オブジェクトにアクセスして使用できます。

次はこれらのユーザー転送に関連するコールバックメソッドです。まず、「一緒に転送するデータを指定する」です。このためにBaseUser抽象クラスを継承したゲームユーザークラスに次のコールバックを実装します。方法は簡単です。次の例のように、転送するデータをユーザーが希望するkey値を利用して、key-valueペアで引数として受け取ったtransferPackに入れるだけです。

もしここで転送するデータを指定しなければ、対象のゲームノードに転送されたユーザーオブジェクトの該当データは、すべてデフォルト値に初期化されるため注意が必要です。

```java
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("TotalMoney", myTotalMoney);
    transferPack.put("Message", myMessage);
}
```

次はユーザー転送が完了した後、対象のゲームノードで処理するコールバックメソッドを見ていきましょう。次のように転送前に指定したkeyを利用して、希望するオブジェクトにアクセスできます。このような方法で転送が完了したユーザーオブジェクトの該当データを元の状態に戻してくれます。

```java
@Override
public void onTransferIn(TransferPack transferPack) throws SuspendExecution {
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setTotalMoney(transferPack.getToLong("TotalMoney"));
    setTransferMessage(transferPack.getToString("Message"));
}
```

上記の2つのメソッドは、GameAnvilが自動で呼び出します。ユーザーは実装するだけで構いません。



### 転送可能なユーザータイマー

ユーザーが転送される時、ユーザーに登録しておいたタイマーも一緒に転送できます。この時、転送後も使用するタイマーを登録できるようにonTransferInTimerHandlerコールバックを呼び出します。ユーザーは文字列キーとそれに対応するTimerHandlerを登録できます。文字列キーが転送されたタイマーハンドラキーリストに存在する場合、再登録するタイマーハンドラを登録します。このコールバックで再登録しなかったタイマーは、ユーザーまたはルームの転送後に使用できなくなるため、注意が必要です。

```java
@Override
public void onTransferInTimerHandler(TimerHandlerTransferPack timerHandlerTransferPack) {
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)){
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
    ...
}
```

## ルームの転送(RoomTranfer)

![gamenode-room-transfer2.png](https://static.toastoven.net/prod_gameanvil/images/gamenode-room-transfer2.png)

ルームの転送は、中級概念で説明したユーザー転送と非常に似た機能です。ユーザーよりも大きい概念であるルームが転送される違いがあるだけです。ゲームノードには1人以上のゲームユーザーが参加したルームオブジェクトが作成されることがあります。このルームオブジェクトは、ユーザーオブジェクトと同様にゲームノード間で転送されることがあります。例えば、1番のゲームノードのルームオブジェクトが2番のゲームノードに転送されます。このようにルームが転送される時、当然ながらルーム内に存在するユーザーも一緒に転送されます。そのおかげでルームが転送される前/後のゲームの流れは連続して進行できます。つまり、該当のルームオブジェクトが転送されるプロセスは、非常に素早く進行するため、ルーム内のユーザーはそれを認識することなくゲームを継続できます。この技術は、GameAnvilの無点検パッチ(NonStopPatch)の主要かつ根幹となり、ルームオブジェクトを正常にシリアル化/逆シリアル化しなければ、意図したとおりに動作しません。このようなシリアル化/逆シリアル化の対象は、GameAnvilユーザーが直接実装できます。つまり、該当ルームオブジェクトのどのような値を転送するかどうかを決定することです。

このようなルーム転送を発生させるのは、無点検パッチ(NonStopPatch)コマンドのみです。このコマンドは通常、GameAnvil Consoleを介してゲーム運営者が明示的に送信します。



### ルーム転送の実装

実際にルームの転送はGameAnvilが内部的に静かに処理します。この時、クライアントは自分のゲームユーザーに加えて、自分が属しているルームのオブジェクトがサーバー間で転送されたか認識できない可能性が高いです。特別な問題が発生しない限り、全体の流れが非常に素早く進行するため、転送する前のゲームの流れを転送後も継続するにあたって無理がありません。

この時、別のゲームノードにルームオブジェクトを転送する場合、どのデータを一緒に転送するかはユーザーが指定できます。指定するだけで、このデータはルームオブジェクトの一部と一緒にシリアル化されます。転送対象のゲームノードでは、逆シリアル化を気にすることなく、簡単に該当オブジェクトにアクセスして使用できます。

次はこれらのユーザー転送に関連するコールバックメソッドです。"まず一緒に転送するデータを指定する"です。これのためにBaseRoom抽象クラスを継承したゲームルームクラスに次のコールバックを実装します。方法は簡単です。次の例のように転送するデータをユーザーが希望するkey値を利用して、key-valueペアで引数として受け取ったtransferPackに入れるだけです。

もしここで転送するデータを指定しなければ、対象のゲームノードに転送されたルームオブジェクトの該当データは、すべてデフォルト値に初期化されるため注意が必要です。

> [参考]
> 
> ルームが転送される際は、ルーム内のユーザーが一緒に転送されます。ユーザー転送に関しては中級概念で説明したため、ここでは扱いません。

```java
@Override
public void onTransferOut(TransferPack transferPack) throws SuspendExecution {
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("RoomInfo", myRoomInfo);
}
```

次はルーム転送が完了した後、対象のゲームノードで処理するコールバックメソッドを見ていきましょう。次のように転送前に指定したkeyを利用して、希望するオブジェクトにアクセスできます。このような方法で転送が完了したルームオブジェクトの該当データを元の状態に戻してくれます。特に、転送されたルーム内のユーザーオブジェクトリストを引数として受け取っていることが確認できます。この部分を除けば、全体的な流れは、ユーザー転送と非常によく似ています。

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

### 転送可能なルームタイマー

ルームが転送される時に登録しておいたタイマーも一緒に転送できます。この時、転送後も使用するタイマーを登録できるようにonTransferInTimerHandlerコールバックを呼び出します。ユーザーは文字列キーとそれに対応するTimerHandlerを登録できます。文字列キーが転送されたタイマーハンドラキーリストに存在する場合、再登録するタイマーハンドラを登録します。このコールバックで再登録しなかったタイマーは、ユーザーまたはルームの転送後に使用できなくなるため、注意が必要です。

```java
@Override
public void onTransferInTimerHandler(TimerHandlerTransferPack timerHandlerTransferPack) {
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)){
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
    ...
}
```
