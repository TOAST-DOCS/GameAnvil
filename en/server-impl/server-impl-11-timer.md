<!-- machine_translated: true -->

<!-- pre-align:aligned sig=3e8f6f005ca1 -->

<a id="game-gameanvil-server-development-guide-timer"></a>
## Game > GameAnvil > Server Development Guide > Timer { #game-gameanvil-server-development-guide-timer }

<a id="timer"></a>
## Timer { #timer }

Users may want to run arbitrary code at a set time or at a set interval. Let's take a look at how to handle these timers.

<a id="implement-timerhandler"></a>
### Implement TimerHandler { #implement-timerhandler }

You must implement this interface in order to handle timers. GameAnvil's timer system makes calls to this interface. You can create as many TimerHandlers as you like, in the form below.

```java
private ITimerHandler getMyTimerHandler() {
    return new ITimerHandler() {
        @Override
        public void onTimer(final ITimer timer) {
            // Work to be done in the timer
        }
    };
}
```

The equivalent lambda form of this code would look like the following.

```java
private ITimerHandler getMyTimerHandler() {
    return (timer) -> {
         // Task to perform in the timer
    };
}
```

The timer object passed in as an argument to the onTimer method contains the timer's information. You can use the Timer object to perform additional operations such as pausing, resuming, or stopping the timer.

<a id="add-a-timer"></a>
### Add a timer { #add-a-timer }

The timer handlers we saw earlier need to be added to the engine to actually run. To add a timer, we use the addTimer API.

```java
/**
 * Adds a timer that performs a task once and returns it.
 *
 * @param timerKey Timer key
 * @param delay    Delay interval
 * @param timeUnit Time unit
 * @param handler  Handler to register
 * @return Returns the registered timer as {@link ITimer} type
 */
ITimer scheduleTimer(String timerKey, int delay, TimeUnit timeUnit, ITimerHandler handler);
```

The following table describes each parameter.

| Name | Description | Example |
| --- | --- | --- |
| timerKey | Call function Key<br>Used by User/Room to distinguish which Timers need to be transferred | "MyTimer" |
| delay | Call time interval | 100 |
| TimeUnit | Call time unit | TimeUnit.SECONDS<br>TimeUnit.MILLISECONDS |
| handler | Handler object to execute | Custom |

<a id="remove-a-timer"></a>
### Remove a timer { #remove-a-timer }

You can remove a timer registered with the engine at any time. To do so, use the removeTimer API. Make sure to save the Timer object returned when registering to remove a timer.

```java
userContext.scheduleTimer("MyTimer", 1000, TimeUnit.MILLISECONDS, new ITimerHandler() {
    @Override
    public void onTimer(final ITimer timer) {
        // Tasks to perform in the timer
    }
});

// Remove the object by timer name
userContext.removeTimer("MyTimer");
```

<a id="send-a-timer"></a>
## Send a timer { #send-a-timer }

Earlier, we looked at [transferring](server-impl-08-object-transfer.md) objects [in Transferable Objects](server-impl-08-object-transfer.md). By default, game users and room objects can be transferred between game nodes at any time. Therefore, the timers that you register on these user and room objects must be transferable as well. When transferring timers inside the engine, the timer handler code is not transferable, so it sends a list of timer handler keys. Therefore, if you want to use the timer corresponding to a timer handler key after a user or room transfer, you must write your code to re-register the timer handler to use in the onTransferIn callback.


<a id="re-register-the-timer-handler"></a>
### Re-register the timer handler { #re-register-the-timer-handler }

The onTransferIn callback is called so that the user and room can register a timer to be used after the transfer. At this time, the user can register a string key and its corresponding ITimerHandler. If the string key exists in the list of transferred TimerHandler keys, it registers a TimerHandler to re-register. Timers that are not re-registered in this callback will no longer be used after the user or room transfer.

```java
@Override
public void onTransferIn(IReadOnlyTransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {
    if (timerHandlerTransferPack.containsTimerKey("MyTimer")) {
        timerHandlerTransferPack.reRegister("MyTimer", newMyTimer());
    }
}
```
