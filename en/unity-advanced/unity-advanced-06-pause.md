<!-- machine_translated: true -->

<!-- pre-align:aligned sig=a0b0bd0e481a -->

<a id="game-gameanvil-unity-advanced-development-guide-preventing-background-disconnection"></a>
## Game > GameAnvil > Unity Advanced Development Guide > Preventing Background Disconnection { #game-gameanvil-unity-advanced-development-guide-preventing-background-disconnection }

<a id="prevent-background-connection-drop"></a>
## Prevent Background Connection Drop { #prevent-background-connection-drop }

To check the connection status between the server and client, the server periodically sends messages to check the client's state, and the client responds with its own messages. However, when a game goes into the background on a mobile device, the Unity application pauses, and a paused application can no longer exchange packets with the game server. In this state, the connection check messages cannot be exchanged either, which eventually causes the connection to the server to be dropped.

<a id="pause-and-resume-connection-confirmation-feature"></a>
### Pause and Resume Connection Confirmation Feature { #pause-and-resume-connection-confirmation-feature }

To prevent the connection from being dropped due to a failed connection check, you must request the server to pause the connection check feature before the application transitions to the background.
When the application transitions to the background or foreground, Unity's `OnApplicationPause()` callback in `MonoBehaviour` is invoked. Call `PauseClientStateCheck()` when transitioning to the background to pause the connection check feature, and call `ResumeClientStateCheck()` when transitioning to the foreground to resume it.

```c#
public class GameAnvilManager : MonoBehaviour
{
    ...
    
    private void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            connector.PauseClientStateCheck(pauseClientStateCheckTime);
        }
        else
        {
            connector.ResumeClientStateCheck();
        }
    }
    
    ...
}
```

`PauseClientStateCheck()` has one parameter as follows:

| Type | Name | Description |
|---------|----------------|----------------------------------------------|
| int | pauseTime | The duration to pause. <br/>Minimum: 10 seconds, Maximum: 15 minutes, Unit: milliseconds |

If a value smaller than the minimum is entered, the minimum value is applied automatically. If a value larger than the maximum is entered, the connection check feature resumes automatically after the maximum duration has elapsed.