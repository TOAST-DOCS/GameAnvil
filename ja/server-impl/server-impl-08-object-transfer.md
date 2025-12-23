## Game > GameAnvil > サーバー開発ガイド > 転送可能オブジェクト

## オブジェクト転送(Object Transfer)

GameAnvilでオブジェクト転送とは、1つのノードから別のノードへオブジェクトが移動することを意味します。ユーザーが関心を持つべきオブジェクト転送は全てゲームノード間で発生します。その代表的な二つであるユーザー転送とルーム転送について見ていきます。

## ユーザー転送(UserTransfer)

![gamenode-user-transfer2.png](https://static.toastoven.net/prod_gameanvil/images/gamenode-user-transfer2.png)

ゲームノードにはゲームユーザーオブジェクトが生成されます。このゲームユーザーオブジェクトはゲームノード間でいつでも転送される可能性があります。例えば、1番ゲームノードのユーザーオブジェクトが2番ゲームノードに生成されたルームに入る過程で、ゲームノード間のユーザーオブジェクト転送が発生します。この技術はGameAnvilの最も核心かつ根幹となり、ユーザーオブジェクトを適切にシリアライズ/デシリアライズしてこそ意図通りに動作します。このようなシリアライズ/デシリアライズ対象はGameAnvilユーザーが直接実装できます。つまり、該当ゲームユーザーオブジェクトのどの値を転送するか決定するということです。

このようなユーザー転送が発生する場合は大きく3つに分けられます。

- 第一に、マッチメイキングを含むルーム生成やルーム参加ロジックにより、ユーザーオブジェクトは別のゲームノードへ転送されることがあります。このとき、実装によっては別のチャンネルのルームへ進入することもあります。
- 第二に、別のチャンネルへ移動する際に発生します。1つのゲームノードは1つのチャンネルに属することができます。したがってチャンネルを移動することはゲームノード間の移動を意味します。
    - 最初の場合のようにマッチメイキングなどを利用して別のチャンネルのルームへ進入する場合には、エンジン内部で暗黙的に処理します。
    - クライアントは明示的にチャンネル移動をリクエストすることもできます。
- 第三に、任意のゲームノードに対してSafe Pauseを進行すると、該当ノードのユーザーオブジェクトは他の有効なゲームノードへ分散されて転送されます。この場合は運営側面からGameAnvil Consoleを
  通じて明示的に命令を下した場合です。

### ユーザー転送及び転送可能なユーザータイマー実装

実際のユーザー転送はGameAnvilが内部的に静かに処理します。このとき、クライアントは自分のゲームユーザーオブジェクトがサーバー間で転送されていることを認知しません。つまり、別のゲームノードのルームに入ったとしても、クライアントは単に1つのGameAnvilサーバー群で任意のルームに入ったに過ぎません。

ただし、別のゲームノードへユーザーオブジェクトを転送する際、どのデータを一緒に転送するかはユーザーが指定できます。簡単に指定さえしておけば、このデータはユーザーオブジェクトの一部として一緒にシリアライズされます。転送先のゲームノードではデシリアライズを気にする必要なく簡単に該当オブジェクトにアクセスして使用できます。

次はこのようなユーザー転送に関連するコールバックメソッドです。まず「一緒に転送するデータの指定」です。このためにIUserインターフェースに定義されたメソッドをゲームユーザークラスに以下のコールバックを実装します。方法は簡単です。以下の例のように転送するデータをユーザーが希望するkey値を利用してkey-valueペアとしてパラメータで受け取ったtransferPackに入れるだけで済みます。

もしここで転送するデータを指定しないと、対象ゲームノードへ転送されたユーザーオブジェクトの該当データは全てデフォルト値で初期化されるため注意が必要です。

```java

@Override
public void onTransferOut(ITransferPack transferPack) {
    // 転送が必要なユーザーデータの保存
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("TotalMoney", myTotalMoney);
    transferPack.put("Message", myMessage);
}
```

続いてユーザー転送完了後、対象ゲームノードで処理するコールバックメソッドを見ていきます。以下のように転送前に指定したkeyを利用して希望するオブジェクトにアクセスできます。このような方法で転送完了したユーザーオブジェクトの該当データを元の状態に戻します。ユーザーが転送される際、ユーザーに登録しておいたタイマー情報も一緒に転送されます。ユーザーは文字列キーとそれに対応するTimerHandlerを登録できます。文字列キーが転送されたタイマーハンドラキーリストに存在する場合、再登録するタイマーハンドラを登録します。このコールバックで再登録しなかったタイマーは、ユーザーまたはルーム転送以降これ以上使用されないため注意が必要です。

```java

@Override
public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {
    // 転送されたユーザーデータの再保存
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setTotalMoney(transferPack.getToLong("TotalMoney"));
    setTransferMessage(transferPack.getToString("Message"));

    // 登録が必要なタイマーキーが伝達された場合、タイマー再登録処理
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)) {
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
}
```

上記の2つのメソッドはGameAnvilが自動的に呼び出します。ユーザーはただ実装するだけです。

## ルーム転送(RoomTransfer)

![gamenode-room-transfer2.png](https://static.toastoven.net/prod_gameanvil/images/gamenode-room-transfer2.png)

ルーム転送は中級概念で説明したユーザー転送と非常に似た機能です。ユーザーより大きな概念であるルームが転送される違いがあるだけです。ゲームノードには1名以上のゲームユーザーが参加したルームオブジェクトが生成されることがあります。このルームオブジェクトはユーザーオブジェクトと同様にゲームノード間で転送されることがあります。例えば、1番ゲームノードのルームオブジェクトが2番ゲームノードへ転送されるのです。このようにルームが転送される際、当然ながらルームの中に存在するユーザーたちも一緒に転送が行われます。そのおかげでルームが転送される前/後のゲームフローは連続して進行できます。つまり、該当ルームオブジェクトが転送される過程は非常に高速に進行するため、ルームの中のユーザーは認知できない状態でゲームを継続できます。この技術はGameAnvil Safe Pauseの核心かつ根幹となり、ルームオブジェクトを適切にシリアライズ/デシリアライズしてこそ意図通りに動作します。このようなシリアライズ/デシリアライズ対象はGameAnvilユーザーが直接実装できます。つまり、該当ルームオブジェクトのどの値を転送するか決定するということです。

このようなルーム転送を発生させるのはSafe Pause命令のみです。この命令は一般的にGameAnvil Consoleを通じてゲーム運営者が明示的に伝達します。

### ルーム転送及び転送可能なルームタイマー実装

実際のルーム転送はGameAnvilが内部的に静かに処理します。このとき、クライアントは自分のゲームユーザーに加え、自分が属するルームオブジェクトがサーバー間で転送されていることを認知できない可能性が高いです。特別な問題が発生しない限り、全体のフローが非常に高速に進行するため、転送前のゲームフローを転送後に継続することに無理がありません。

このとき、別のゲームノードへルームオブジェクトを転送する際、どのデータを一緒に転送するかはユーザーが指定できます。指定さえしておけば、このデータはルームオブジェクトの一部として一緒にシリアライズされます。転送先のゲームノードではデシリアライズを気にする必要なく簡単に該当オブジェクトにアクセスして使用できます。

次はこのようなユーザー転送に関連するコールバックメソッドです。まず「一緒に転送するデータの指定」です。このためにIRoomインターフェースに定義されたメソッドをゲームルームクラスに以下のコールバックを実装します。方法は簡単です。 

以下の例のように転送するデータをユーザーが希望するkey値を利用してkey-valueペアとしてパラメータで受け取ったtransferPackに入れるだけで済みます。

もしここで転送するデータを指定しないと、対象ゲームノードへ転送されたルームオブジェクトの該当データは全てデフォルト値で初期化されるため注意が必要です。

> [参考]
>
> ルームが転送される際はルームの中のユーザーたちが一緒に転送されます。ユーザー転送に関しては中級概念で説明したため、ここでは扱いません。

```java

@Override
public void onTransferOut(ITransferPack transferPack) {
    // 転送が必要なルームデータの保存
    transferPack.put("DataTopic", myDataTopic);
    transferPack.put("RoomInfo", myRoomInfo);
}
```

これでルーム転送完了後、対象ゲームノードで処理するコールバックメソッドを見ていきます。以下のように転送前に指定したkeyを利用して希望するオブジェクトにアクセスできます。このような方法で転送完了したルームオブジェクトの該当データを元の状態に戻します。特に、転送されたルームの中のユーザーオブジェクトリストをパラメータとして受け取っていることが確認できます。この部分を除けば全体的なフローはユーザー転送と非常によく似ています。ルームが転送される際、ルームに登録しておいたタイマー情報も一緒に転送されます。ユーザーは文字列キーとそれに対応するTimerHandlerを登録できます。文字列キーが転送されたタイマーハンドラキーリストに存在する場合、再登録するタイマーハンドラを登録します。このコールバックで再登録しなかったタイマーは、ユーザーまたはルーム転送以降これ以上使用されないため注意が必要です。

```java

@Override
public void onTransferIn(List<GameUser> userList, ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {
    this.users.clear();
    for (GameUser user : userList) {
        this.users.put(user.getUserId(), user);
    }

    // 転送されたルームデータの再保存
    setTestTransferDataTopic(transferPack.getToString("DataTopic"));
    setRoomInfo(transferPack.getToLong("RoomInfo"));

    // 登録が必要なタイマーキーが伝達された場合、タイマー再登録処理
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)) {
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
}
```
