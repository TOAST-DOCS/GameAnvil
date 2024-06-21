## Game > GameAnvil > Unity Basic Development Guide > Prevent Background Disconnection

## Prevent Background Disconnections

When the game switches to the background on a mobile device, our application freezes. When the application freezes, the process of calling Update() on the GameAnvilConnector doesn't happen, which means it can't send packets to and from the game server. After a while, it won't even be able to send or receive packets to check for connectivity and will eventually disconnect from the server.

In some cases, you may not want to be disconnected if the game goes to the background for a certain amount of time. In this case, you can use ConnectionAgent's PauseClientStateCheck() and ResumeClientStateCheck() methods to suspend the server's ability to check for connections.

The server's connection check pause time can be adjusted via the GameAnvilConnector's pauseClientStateCheckTime value.

For more information, see the [Unity Advanced Development Guide > Prevent background disconnection](../unity-advanced/unity-advanced-06-background-connection.md).

### Reconnect

After the time entered in PauseClientStateCheck() has elapsed, the server will resume checking for connections and you may be disconnected. If this happens, you'll need to reconnect.

You can learn more about reconnecting in the [Unity Advanced Development Guide > Reconnecting](../unity-advanced/unity-advanced-07-reconnect.md).