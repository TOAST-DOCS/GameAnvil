## Game > GameAnvil > Unity Advanced Development Guide > Disconnection

## Terminate the GameAnvil connector

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