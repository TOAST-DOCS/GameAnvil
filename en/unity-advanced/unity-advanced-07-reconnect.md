## Game > GameAnvil > Unity Advanced Development Guide > Reconnection

## Reconnect

During a game, you may lose connection to the server for a variety of reasons. We support reconnect so that you can continue playing when you lose connection. The API used is the same as the normal connection method, but the information you receive as a result is different. 

### Connect to the server

Connect to the server using the ConnectionAgent's Connect function. This is the same as in the normal case. 

```c#
/// <summary>
/// Attempts to connect to the GameAnvil server.
/// </summary>
/// <param name="ip">Target IP address</param>
/// <param name="port">Target port</param>
/// <param name="onConnect">A delegate to receive connection attempt results</param>
connector.GetConnectionAgent().Connect(ip, port, (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
    /// <param name="connectionAgent">The connection agent that requested Connect()</param>
    /// <param name="result">Connect() request result code</param>
    if (result == ResultCodeConnect.CONNECT_SUCCESS) {
      // Connection success
    } 
    else {
      // Connection failed
    }
});
```

### Verify

Use the ConnectionAgent's Authenticate function to perform the authentication process. The input values are the same as in the normal case. However, the loginedUserInfoList will contain the information of previously played users. 

```c#
/// <summary>
/// Authenticates with the GameAnvil server<para></para>
/// Enable connector after successful authentication
/// </summary>
/// <param name="deviceId">Unique ID to identify the user's device. Depending on the server implementation, pass an empty string if not used</param>
/// <param name="accountId">Unique ID to identify the user's account</param>
/// <param name="passwd">Password for the user account. Depending on the server implementation, pass an empty string if not used</param>
/// <param name="onAuthentication">A delegate to receive the results of the request</param>.
connector.GetConnectionAgent().Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    /// <param name="connectionAgent">The connection agent that requested Authentication().
    /// <param name="result">Authentication() request result</param>
    /// <param name="loginedUserInfoList">List of login information left on the server</param>
    /// <param name="message">Message received from the server</param>
    /// <param name="payload">Additional information received from the server</param>
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
      foreach(ConnectionAgent.LoginedUserInfo loginedUserInfo in loginedUserInfoList)
      { 
        // Play user info
      }
    } 
});
```

### Sign in

Proceed with login using the user information received as a result of authentication. Make sure to use the same values as the previous user information, such as userType or channelId, otherwise the login may fail. 

```c#
/// <summary>
/// Finds and returns the user agent
/// </summary>
/// <param name="serviceName">The name of the service to be used by the user agent</param>
/// <param name="subId">Unique ID to identify the user agent by service</param>
/// <returns>The corresponding user agent, or null if none</returns>
UserAgent userAgent = Connector.GetUserAgent(loginedUserInfo.ServiceName, loginedUserInfo.SubId);

/// <summary>
/// Logs in to the service
/// </summary>
/// <param name="userType">User's type </param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="channelId">Id of the channel to log in to</param>
/// <param name="onLogin">Agent to receive the result</param>
userAgent.Login(loginedUserInfo.UserType, loginedUserInfo.ChannelId, null,
                 (UserAgent agent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => {
    /// <param name="userAgent">The user agent that requested Login()</param>
    /// <param name="result">Result of the Login() request</param>
    /// <param name="loginInfo">Login information</param>
	if (result == ResultCodeLogin.LOGIN_SUCCESS) {
        if(loginInfo.IsRelogined){
            // Reconnect
        }else{
            // First time login
        }
	}
});
```

If IsRelogined in UserAgent.LoginInfo is true upon successful login, the reconnection login is successful. Based on the information contained in UserAgent.LoginInfo, you can process the reconnection. The most important thing is to synchronize the changed data on the server to the client during the reconnection time. The data required for this synchronization can be handled in the payload of UserAgent.LoginInfo.  

The reconnection login will cause the BaseUser.onRelogin() callback to be called on the server. At this point, you can define the messages required for synchronization and add them to the outPayload parameter so that they can be received and processed as a payload in UserAgent.LoginInfo. 

If IsRelogined is false, then you've been logged in after time has passed and the old user information has been removed. In this case, you can follow the steps to log in for the first time. 
