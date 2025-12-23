## Game > GameAnvil > サーバー開発ガイド > タイマーの使用

## タイマー

指定された時間または周期ごとに任意のコードを実行できます。ここでは、このようなタイマーを設定する方法を説明します。

### TimerHandlerの実装

タイマー処理を行うには、必ずこのインターフェースを実装する必要があります。GameAnvilのタイマーシステムはこのインターフェースを呼び出します。TimerHandlerは以下のような形で必要な数だけ作成できます。

```java
private ITimerHandler getMyTimerHandler() {
    return new ITimerHandler() {
        @Override
        public void onTimer(final ITimer timer) {
            // タイマーで行う処理
        }
    };
}
```

このコードと同じラムダ式は以下の通りです。

```java
private ITimerHandler getMyTimerHandler() {
    return (timer) -> {
         // タイマーで行う処理
    };
}
```

このとき、onTimerメソッドの引数として渡されるtimerオブジェクトには、タイマーの情報が入っています。タイマーオブジェクトを活用して、タイマーを一時停止、再開、停止するなどの追加操作を行うことができます。

### タイマーの追加

前述のタイマーハンドラは、エンジンに追加して初めて実際に駆動します。タイマーを追加するためにaddTimer APIを使用します。

```java
/**
 * 一度処理を実行するタイマーを追加した後、受け取る
 *
 * @param timerKey タイマーキー
 * @param delay    遅延間隔
 * @param timeUnit 時間単位
 * @param handler 登録するハンドラ
 * @return {@link ITimer} タイプで登録されたタイマーを返却
 */
ITimer scheduleTimer(String timerKey, int delay, TimeUnit timeUnit, ITimerHandler handler);
```

それぞれのパラメータに関する説明は、次の表のとおりです。

| 名前      | 説明                                                         | 例                                      |
|-----------|--------------------------------------------------------------|-------------------------------------------|
| timerKey  | 呼び出し関数Key<br>User/RoomからTransferする必要があるTimerを区別するために使用      | "MyTimer"                                 |
| delay     | 呼び出し時間間隔                                                   | 100                                       |
| TimeUnit  | 呼び出し時間単位                                                   | TimeUnit.SECONDS<br>TimeUnit.MILLISECONDS |
| handler   | 実行するハンドラオブジェクト                                                 | ユーザー定義                                  |


### タイマーの削除

エンジンに登録されたタイマーはいつでも削除できます。このとき、removeTimer APIを使用します。タイマー削除のために、登録時に返されたタイマーオブジェクトを保持しておく必要があります。

```java

userContext.scheduleTimer("MyTimer", 1000, TimeUnit.MILLISECONDS, new ITimerHandler() {
    @Override
    public void onTimer(final ITimer timer) {
        // タイマーで行う処理
    }
});

// タイマー名でオブジェクトを削除します
userContext.removeTimer("MyTimer");
```

## タイマー転送

[転送可能オブジェクト](server-impl-08-object-transfer.md)で、オブジェクトの転送について確認しました。基本的にゲームユーザーとルームオブジェクトは、いつでもゲームノード間で転送される可能性があります。そのため、これらのユーザーとルームオブジェクトに登録したタイマーも一緒に転送できる必要があります。エンジン内部でタイマーを転送する際、タイマーハンドラコードは転送できないため、タイマーハンドラキーのリストを転送します。したがって、タイマーハンドラキーに該当するタイマーをユーザーまたはルーム転送後も使用する場合は、onTransferInコールバックで使用するタイマーハンドラを再登録するようにコードを記述する必要があります。


### タイマーハンドラの再登録

ユーザーとルームは転送後も使用するタイマーを登録できるように、onTransferInコールバックを呼び出します。このとき、ユーザーは文字列キーとそれに対応するITimerHandlerを登録できます。文字列キーが転送されたタイマーハンドラキーのリストに存在する場合、再登録するタイマーハンドラを登録します。このコールバックで再登録しなかったタイマーは、ユーザーまたはルームの転送以降、使用されません。

```java
@Override
public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {
    if (timerHandlerTransferPack.containsTimerKey("MyTimer")) {
        timerHandlerTransferPack.reRegister("MyTimer", newMyTimer());
    }
}
```
