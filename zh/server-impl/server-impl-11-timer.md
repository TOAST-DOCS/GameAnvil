## Game > GameAnvil > Server Development Guide > Use Timer

## Timer

Users may want to run arbitrary code at a set time or at a set interval. Let's take a look at how to handle these timers.

### Implement TimerHandler

You must implement this interface in order to handle timers. GameAnvil's timer system makes calls to this interface. You can create as many TimerHandlers as you like, in the form below.

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

The equivalent lambda form of this code would look like the following.

```java
private TimerHandler getMyTimerHandler() {
    return (timer, object) -> {
        ...
    };
}
```

The timer object passed in as an argument contains the timer's information, and object is the object that registered the timer. For example, if you registered the timer with the user's GameUser, that object is the GameUser object.

### Add a timer

The timer handlers we saw earlier need to be added to the engine to actually run. To add a timer, we use the addTimer API.

```java
/**
 * Add a timer
 * @param timer
 * @param interval the delay interval
 * @param timeUnit Time unit
 * @param times Number of times to repeat (0 == infinite)
 * @param handlerKey Passes the handler key
 * @param isTickFromRun If false, the timer will tick after interval from the most recent run. if true, the timer will tick at interval from the time of registration.
 * @return Returns the registered timer
 */
public Timer addTimer(int interval, TimeUnit timeUnit, int times, String handlerKey, boolean isTickFromRun)
```

The following table describes each parameter.

| Name | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Example |
| --- |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| --- |
| Interval | Call time interval                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | 1 |
| TimeUnit | Call time unit                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | TimeUnit.SECONDS<br>TimeUnit.MILLISECONDS |
| Times | Number of calls                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | 20 |
| HandlerKey | Call function Key<br>Used by User/Room to distinguish which Timers need to be transferred                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | timerHandlerKey |
| IsTickFromRun | Whether to base the time of the next Timer call on the time of the immediately preceding fired Timer or the time the Timer was registered.<br><br>If false, the <br>  - Call based on the time the timer is registered<br>  - For example, a timer with a 1-second cycle is called again 0.1 seconds later, just 0.1 seconds after the timer registration time, even though the timer immediately before it was called 0.9 seconds earlier because it was pushed back 0.1 seconds by a load <br>  - Recommended when the number of calls within a set time is critical<br>  - The number of calls is guaranteed even under load, but pushed timer processing can result in multiple calls at once<br>  - Usage example: An item that applies a once per second food buff<br><br>If true, the<br>  - Called based on the time of the immediately preceding fired Timer<br>  - For example, a timer with a 1-second cycle is called again in 1 second, based on the time the previous timer processed<br>  - Recommended when flexibility of calls to match server load is more important than accuracy of the call count<br>  - Under load, the overall number of calls may decrease, but they don't come in clusters<br>  - Usage example: Periodic DB queries | true/false |

### Remove a timer

You can remove a timer registered with the engine at any time. To do so, use the removeTimer API. Make sure to save the Timer object returned when registering to remove a timer.

```java
// Keep the return value, a Timer object, handy.
Timer timer = addTimer(1, TimeUnit.SECONDS, 20, "sampleTimerHandler1", false);

// Remove it from the engine using the Timer object we kept.
removeTimer(timer);
```

## Send a timer

Earlier, we looked at [transferring](server-impl-08-object-transfer)objects [in Transferable Objects](server-impl-08-object-transfer). By default, game users and room objects can be transferred between game nodes at any time. Therefore, the timers that you register on these user and room objects must be transferable as well. When transferring timers inside the engine, the timer handler code is not transferable, so it sends a list of timer handler keys. Therefore, if you want to use the timer corresponding to a timer handler key after a user or room transfer, you must write your code to re-register the timer handler to use in the onTransferInTimerHandler callback.


### Re-register the timer handler

The onTransferInTimerHandler callback is called so that the user and room can register a timer to be used after the transfer. At this time, the user can register a string key and its corresponding TimerHandler. If the string key exists in the list of transferred TimerHandler keys, it registers a TimerHandler to re-register. Timers that are not re-registered in this callback will no longer be used after the user or room transfer.

```java
@Override
public void onTransferInTimerHandler(TimerHandlerTransferPack timerHandlerTransferPack) {
    if (timerHandlerTransferPack.getTimerHandlerKeys().contains(StringValues.TEST_TIMER_HANDLER)){
        timerHandlerTransferPack.reRegister(StringValues.TEST_TIMER_HANDLER, testTimerHandler());
    }
    ...
}
```
