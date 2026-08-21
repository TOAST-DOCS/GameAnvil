<!-- pre-align:aligned sig=679b63c58911 -->

<a id="game-gameanvil-unity-advanced-development-guide-disconnection"></a>
## Game > GameAnvil > Unity Advanced Development Guide > Disconnection { #game-gameanvil-unity-advanced-development-guide-disconnection }

<a id="terminate-the-gameanvil-connector"></a>
## Terminate the GameAnvil connector { #terminate-the-gameanvil-connector }

It is recommended that you terminate the connection by calling the GameAnvilConnector.Disconnect() function before the end of gameplay. If you do not terminate, the server may not be aware of the client's termination, which could cause it to continue unnecessary behavior.
GameAnvilConnector's OnDestroy() has built-in functionality for this.

```c#
private void OnDestroy()
{
    if (connector.IsConnected())
    {
        Disconnect();
    }
}
```