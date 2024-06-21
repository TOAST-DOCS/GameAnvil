## Game > GameAnvil > Unity Advanced Development Guide > Prevent Background Disconnection

As described in the [Unity Basic Development Guide > Prevent Background Disconnection](../unity-basic/unity-basic-08-background-connection.md), your game may lose connection to the server when it goes into the background on a mobile device. To prevent this, you can pause the ability to check for connections to the server.

Here's an example of code that implements this

```c#
public class ConnectHandler : MonoBehaviour
{ 
    ...
    private void OnApplicationPause(bool pause)
    { ...
        if (pause)
        { ...
            // If there's any work to be done before the app pauses, it's handled here.

            // Suspend the server's clientStateCheck function for the entered number of seconds.
            // After this time, the clientStateCheck function may be triggered and the connection may be disconnected. 
            connector.GetConnectionAgent().PauseClientStateCheck(600);

            // Just before the app pauses, call connector.Update() to process the messages that have accumulated on the connector. 
            // to process any messages that have accumulated in the connector and update the state. 
            connector.Update();
        } else
        { 
            // Call connector.Update() immediately after the app resumes to process the messages that have been accumulated in the 
            // process the messages accumulated in the Connector and update the status. 
            connector.Update();

            // Reactivate the server's clientStateCheck function.
            connector.GetConnectionAgent().ResumeClientStateCheck();

            // Check the status of the connection as it may be disconnected if it is resumed after a long pause.
            if (connector.IsConnected())
            { 
                // If there's any work to be done after the app resumes, we'll do it here.
            }
        }
    }
    ...
}
```