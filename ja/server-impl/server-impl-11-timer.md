## Game > GameAnvil > サーバー開発ガイド > タイマーを使用する

## タイマー

決められた時間または、サイクルごとに任意のコードを実行できます。ここでは、このようなタイマーを設定する方法を説明します。

### TimerHandlerの実装

タイマー処理をするためには、必ずこのインターフェイスを実装する必要があります。GameAnvilのタイマーシステムは、このインターフェイスに対する呼び出しを行います。希望する数のTimerHandlerを次のような形で作成できます。

```java
private TimerHandler getMyTimerHandler() {
    return new TimerHandler() {
            @Override
            public void onTimer(Timer timer, Object object) throws SuspendExecution {
                ...
            }
        };
}
```

このコードと同じラムダ式は次のとおりです。

```java
private TimerHandler getMyTimerHandler() {
    return (timer, object) -> {
        ...
    };
}
```

この時、引数で送信されるtimerオブジェクトにはタイマーの情報が含まれています。また、objectは該当タイマーを登録したオブジェクトです。例えば、ユーザーのGameUserにタイマーを登録した場合、そのobjectは該当GameUserオブジェクトです。

### タイマーの追加

前述したタイマーハンドラは、エンジンに追加すると動作します。タイマーを追加するためにはaddTimer APIを使用します。

```java
/**
 * タイマーを追加
 *
 * @param interval    遅延間隔
 * @param timeUnit    時間単位
 * @param times        反復回数(0==無限反復)
 * @param handlerKey  ハンドラキーを送信
 * @param isTickFromRun ffalseの場合、タイマーは直近の動作時点を基準にinterval後に動作。trueの場合、タイマーは登録時点を基準にintervalサイクルで反復
 * @return登録されたタイマーを返す
 */
public Timer addTimer(int interval, TimeUnit timeUnit, int times, String handlerKey, boolean isTickFromRun)
```

それぞれの引数についての説明は次の表のとおりです。

| 名前 | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | 例 |
| --- |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| --- |
| Interval | 呼び出し時間の間隔                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 1 |
| TimeUnit | 呼び出し時間の単位                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | TimeUnit.SECONDS<br>TimeUnit.MILLISECONDS |
| Times | 呼び出し回数                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | 20 |
| HandlerKey | 呼び出し関数Key<br>User/RoomでTransferしなければならないTimerを区別するために使用                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | timerHandlerKey |
| IsTickFromRun | 次のTimer呼び出し時間の基準を直前に実行したTimerの時間に設定するか、Timerが登録された時間を基準に設定するかを決定<br><br>falseの場合は<br>  - Timerが登録された時間を基準に呼び出し<br>  - 例えば、1秒サイクルのタイマーは、直前のタイマーが負荷によって0.9秒遅れて0.1秒前に呼び出されても、タイマーの登録時間を基準に0.1秒後に再度呼び出される<br>  - 指定された時間内の呼び出し回数が重要な場合に推薦<br>  - 負荷がかかっても呼び出し回数は保証されるが、一斉に押されたタイマー処理が同時に何度も呼び出されることがある<br>  - 使用例: 1秒に1回バフをかけるアイテム<br><br>trueの場合は<br>  - 直前に実行したTimerの時間を基準に呼び出し<br>  - 例えば、1秒サイクルのタイマーは以前のタイマーが処理された時間を基準に1秒後に再度呼び出される<br>  - 呼び出し回数の正確性よりサーバー負荷に合わせて柔軟に呼び出されることが重要な場合に推薦<br>  - 負荷がかかると、全体的な呼び出し回数は減少する可能性があるが、呼び出しが集中して発生することはない<br>  - 使用例:定期的なDBクエリ | true/false |

### タイマーの削除

エンジンに登録されたタイマーはいつでも削除できます。この時、removeTimer APIを使用します。タイマーを削除するために登録した時に返されたタイマーオブジェクトを保存しておく必要があります。

```java
// 戻り値であるTimerオブジェクトを保管します。
Timer timer = addTimer(1, TimeUnit.SECONDS, 20, "sampleTimerHandler1", false);

// 保管しておいたTimerオブジェクトを使用してエンジンから削除します。
removeTimer(timer);
```

## タイマーの転送

[転送可能オブジェクト](server-impl-08-object-transfer)でオブジェクトの転送について説明しました。基本的にゲームユーザーとルームオブジェクトは、いつでもゲームノード間で転送されることがあります。そのため、これらのユーザーとルームオブジェクトに登録したタイマーも一緒に転送できなければなりません。エンジン内部からタイマーを転送する時、タイマーハンドラコードは転送できないため、タイマーハンドラキーリストを転送します。したがって、タイマーハンドラキーに該当するタイマーをユーザーまたはルームの転送後も使用するのであれば、onTransferInTimerHandlerコールバックで使用するタイマーハンドラを再登録するようにコードを作成する必要があります。


### タイマーハンドラの再登録

ユーザーとルームの転送後も使用するタイマーを登録できるようにonTransferInTimerHandlerコールバックを呼び出します。この時、ユーザーは文字列キーとそれに対応するTimerHandlerを登録できます。文字列キーが転送されたタイマーハンドラキーリストに存在する場合、再登録するタイマーハンドラを登録します。このコールバックで再登録しなかったタイマーはユーザーまたはルームの転送後に使用できなくなります。

```java
@Override
public void onTransferInTimerHandler(TimerHandlerTransferPack timerHandlerTransferPack) {
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)){
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
    ...
}
```
