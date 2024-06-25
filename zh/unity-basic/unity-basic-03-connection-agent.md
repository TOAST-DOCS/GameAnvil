## Game > GameAnvil > Unity Basic Development Guide > ConnectionAgent

## QuickConnect

GameAnvilConnector provides a QuickConnect feature that allows you to connect, authenticate, and log in to a server in one step. For more information on connecting, authenticating, and logging in, see the [Unity Advanced Development Guide > ConnectionAgent](../unity-advanced/unity-advanced-02-connection-agent.md) or the Server Development Guide.

### QuickConnect settings

There are settings used for QuickConnect. They are set to the default values when the GameAnvilConnector is created, but you can change them directly in the inspector window if needed.

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/03-connection-agent/01-config.png)

There are the following types of settings

| Name | Description |
| --- | --- |
| ip | Destination IP address |
| Port | Destination port number |
| accountId | A unique ID to identify the user account |
| deviceId | A unique ID to identify the user's device. Depending on server implementation, pass an empty string if not used |
| password | The password for the user account. Depending on the server implementation, pass an empty string if not used |
| userType | User's type |
| channelId | The ID of the target channel |
| serviceName | Name of the target service |

### Using QuickConnect

Connect, authenticate, and log in to the server via QuickConnect(). Passes the authentication payload, login payload, and DelOnQuickConnect callback as parameters. You can omit the authentication payload and login payload.

```c#
/// <summary>
/// Attempts to connect to the GameAnvil server, requests authentication if connection succeeds, and requests login if authentication succeeds.
/// QUICK_CONNECT_SUCCESS if connection/authentication/login all succeed QUICK_CONNECT_SUCCESS
/// </summary>
/// <param name="authenticatePayload">Additional information to pass for authentication</param>
/// <param name="loginPayload">Additional information to pass for login</param>
/// <param name="onQuickConnect">A delegate to receive the QuickConnect request results</param>
public void QuickConnect(Payload authenticatePayload, Payload loginPayload, DelOnQuickConnect onQuickConnect){
    if (quickConnect != null)
    { 
        CallQuickConnectCallback(onQuickConnect, ResultCodeQuickConnect.QUICK_CONNECT_ALREADY_REQUESTED, null, null);
        return;
    }

     quickConnect = StartCoroutine(QuickConnectCoroutines(authenticatePayload, loginPayload, onQuickConnect, new QuickConnectResult()));
}
```

<br>

You can get the QuickConnect results through the DelOnQuickConnect callback.

```c#
/// <summary>
/// Delegate to receive the results of the QuickConnect request.
/// </summary>
/// <param name="resultCode">QuickConnect request result</param>
/// <param name="userAgent">An agent that enables user-related functionality, such as room creation, entry, etc.</param>
/// <param name="quickConnectResult">A set of additional information about authentication received from the server, additional information about login received from the server, the result of the connection request, the result of the authentication request, and the result of the login request</param>
public delegate void DelOnQuickConnect(ResultCodeQuickConnect resultCode, UserAgent userAgent, QuickConnectResult quickConnectResult);
```

## Disconnect

You can use the GameAnvilConnector's Disconnect() and QuickDisconnect() functions to disconnect from the server. 

When you call QuickDisconnect(), you can log out, terminate the connection to the server, and call a callback to process after disconnecting all at once.

```c#
/// <summary>
/// Logs out, requests to disconnect from the GameAnvil server, and calls callbacks.
/// </summary>
/// <param name="onQuickDisconnect">Delegate to receive the request result</param>
public void QuickDisconnect(DelOnQuickDisconnect onQuickDisconnect);
```

When Disconnect() is called, the connection to the server is terminated.

```c#
/// <summary>
/// Request to disconnect from the GameAnvil server.
/// </summary>
public void Disconnect();
```

