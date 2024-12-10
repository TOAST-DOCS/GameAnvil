## Game > GameAnvil > Unity Advanced Development Guide > Connector

## ConnectionAgent

The ConnectionAgent is responsible for operations related to the Connection node on the GameAnvil server. It provides basic session management functions such as Connect() and Authentication(), as well as a list of channels, and can implement different content based on your own defined protocols. The ConnectionAgent is automatically created when the connector is initialized and can be obtained using the Connector.GetConnectionAgent() function.

```c#
ConnectionAgent connectionAgent = connector.GetConnectionAgent();
```

### Connect to the server

Connect to the server using the ConnectionAgent's Connect function. 

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
		// Successful connection
    } else {
	    // Connection failed
    }
});
```

### Authentication

Use the ConnectionAgent's Authenticate function to perform the authentication process. It takes as parameters deviceId, accountId, password, payload, and a callback to handle the response. The deviceId is used to handle duplicate connections, and the accountId and password can be used to handle authentication on the server. If additional information beyond the deviceId, accountId, and password is needed for authentication, it can be sent in the payload, and the payload parameter can be omitted if not used.
When you call the Authenticate function, the server calls the onAuthentication() callback of the BaseConnection, and the processing of this callback determines whether the authentication succeeds or fails.

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
		// Authentication success
    } else {
		// Authentication failed
    }
});
```

### Channel information

GameAnvil allows you to freely configure channels in the settings. These channel configurations can be pre-agreed between the server and client and used in a fixed form, or they can be varied to suit the situation. ConnectionAgent provides a few functions to get this changed channel information. 

| Functions | Description |
| --- | --- |
| GetChannelList | Request a list of channel IDs for a specific service | 
| GetChannelCountInfo() | Request count information (number of users and rooms) for a specific channel | 
| GetChannelInfo() | Request information (user-defined) for a specific channel |
| GetAllChannelCountInfo() | Request count information (number of users and rooms) for all channels of a specific service |

<br>

Let's take a closer look at this in code below.


GetChannelList() can request and receive a list of channel IDs for a specific service. 

```c#
/// <summary>
/// Request a list of available channels for a service 
/// </summary>
/// <param name="serviceName">Name of the target service</param>
/// <param name="onChannelList">The agent to receive the results of the request</param>
connector.GetConnectionAgent().GetChannelList(serviceName, (ConnectionAgent connection, ResultCodeChannelList result, List<string> channelIdList) => {
    /// <param name="connectionAgent">The connection agent that requested GetChannelList()</param>
    /// <param name="result">Result of the GetChannelList() request</param>
    /// <param name="channelIdList">Channel list received from the server</param>
	if(result == ResultCodeChannelList.CHANNEL_LIST_SUCCESS){
		// Channel list request success
	} else {
		// Channel list request failed
	}
});
```
<br>

GetChannelCountInfo() can request and receive count information (number of users and rooms) for a specific channel. 

```c#
/// <summary>
/// Request the number of users and rooms in a channel<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="serviceName">Name of the service the target channel belongs to</param>
/// <param name="channelId">Identifier of the target channel</param>
/// <param name="onChannelCountInfo">A delegate to receive the request result</param>
connector.GetConnectionAgent().GetChannelCountInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="connectionAgent">The connection agent that requested GetChannelCountInfo()</param>
    /// <param name="result">Result of the GetChannelCountInfo() request</param>
    /// <param name="channelCountInfo">Channel's user count and room count information received from the server</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// Channel count information request success
	} else {
		// Channel count information request failed
	}
});
```
<br>

GetChannelInfo() can request and receive information (user-defined) for a specific channel. 

```c#
/// <summary>
/// Requests channel information<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="serviceName">Name of the service to which the target channel belongs</param>
/// <param name="channelId">Identifier of the target channel</param>
/// <param name="onChannelInfo">A delegate to receive the request result</param>
connector.GetConnectionAgent().GetChannelInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="connectionAgent">The connection agent that requested GetChannelInfo()</param>
    /// <param name="result">Result of the GetChannelInfo() request</param>
    /// <param name="channelInfo">Channel information received from the server</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// Channel information request success
	} else {
		// Channel information request failed
	}
});
```
<br>

GetAllChannelCountInfo() can request and receive count information (number of users and rooms) for all channels in a specific service. 

```c#
/// <summary>
/// Gets the number of users and rooms on all channels in a service<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="serviceName">Name of the target service</param>
/// <param name="onAllChannelCountInfo">A delegate to receive the request result</param>
connector.GeConnectionAgent().GetAllChannelCountInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="connectionAgent">The connection agent that requested GetAllChannelCountInfo().
    /// <param name="result">Result of the GetAllChannelCountInfo() request</param>
    /// <param name="channelCountInfo">The number of users and rooms in the channel received from the server</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// All channel count information request succeeded
	} else {
		// All channel count information request failed
	}
});
```
<br>

GetAllChannelInfo() can request and receive information (user-defined) about all channels of a specific service. 

```c#
/// <summary>
/// Request information from all channels in the service<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="serviceName">Name of the target service</param>
/// <param name="onAllChannelInfo">A delegate to receive request results</param>
connector.GetConnectionAgent().GetAllChannelInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="connectionAgent"> The connection agent that requested GetAllChannelInfo().
    /// <param name="result">Result of the GetAllChannelInfo() request</param>
    /// <param name="channelInfo">List of channel information received from the server</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// All channel information request success
	} else {
		// All channel information request failed
	}
});
```

### Terminate the connection

Use the ConnectionAgent's Disconnect function to terminate the connection to the server. 

```c#
/// <summary>
/// Request to disconnect from the GameAnvil server.
/// </summary>
/// <param name="onDisconnect">A delegate to receive the result of the request</param>.
connector.GetConnectionAgent().Disconnect((ConnectionAgent connectionAgent, ResultCodeDisconnect result) => {
    /// <param name="connectionAgent">The connection agent where the disconnect()occurred</param>
    /// <param name="result">>Disconnect() result</param>
    if (result == ResultCodeDisconnect.SOCKET_DISCONNECT) {
		// Normal exit
    } else {
	    // Abnormal termination
    }
});
```

### Listener

There are two main ways that ConnectionAgent can receive results or notifications from the server for every request.
One is to add a function to the delegate defined on the ConnectionAgent. The other is to register a listener that implements the IConnectionListener interface.

First, the first method. The ConnectionAgent has a delegate as a member for each of its actions so that it can receive the result or notification of any action. If you call the APIs described earlier without callback parameters, or if the server sends you a notification, you'll receive the response as a function registered with the delegate.  

```c#
/// <summary>
/// The result of Connect()
/// </summary>
/// <param name="connectionAgent">The connection agent that requested Connect().
/// <param name="result">Result of Connect()</param>
public Interface.DelConnectionOnConnect onConnectListeners;

/// <summary>
/// Authentication() result
/// </summary>
/// <param name="connectionAgent">The connection agent that requested Authentication()</param>.
/// <param name="result">Authentication() request result</param>
/// <param name="loginedUserInfoList">List of login information left on the server</param>
/// <param name="message">Message received from the server</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelConnectionOnAuthentication onAuthenticationListeners;

/// <summary>
/// GetChannelList() request result
/// </summary>
/// <param name="connectionAgent">The connection agent that requested GetChannelList()</param>.
/// <param name="result">Result of the GetChannelList() request</param>
/// <param name="channelIdList">The list of channels received from the server</param>
public Interface.DelConnectionOnChannelList onChannelListListeners;

/// <summary>
/// GetChannelInfo() request result
/// </summary>
/// <param name="connectionAgent">The connection agent that requested GetChannelInfo()</param>.
/// <param name="result">Result of the GetChannelInfo() request</param>
/// <param name="channelInfo">Channel information received from the server</param>
public Interface.DelConnectionOnChannelInfo onChannelInfoListeners;

/// <summary>
/// GetAllChannelInfo() request result
/// </summary>
/// <param name="connectionAgent"> The connection agent that made the GetAllChannelInfo() request</param>.
/// <param name="result">The result of the GetAllChannelInfo() request</param>
/// <param name="channelInfo">List of channel information received from the server</param>
public Interface.DelConnectionOnAllChannelInfo onAllChannelInfoListeners;

/// <summary>
/// The result of the GetChannelCountInfo() request.
/// </summary>
/// <param name="connectionAgent">The connection agent that requested GetChannelCountInfo()</param>.
/// <param name="result">Result of the GetChannelCountInfo() request</param>
/// <param name="channelCountInfo">The number of users and rooms in the channel received from the server</param>
public Interface.DelConnectionOnChannelCountInfo onChannelCountInfoListeners;

/// <summary>
/// The result of the GetAllChannelCountInfo() request.
/// </summary>
/// <param name="connectionAgent">The connection agent that requested GetAllChannelCountInfo()</param>.
/// <param name="result">Result of the GetAllChannelCountInfo() request</param>
/// <param name="channelCountInfo">The number of users and rooms for the channel received from the server</param>
public Interface.DelConnectionOnAllChannelCountInfo onAllChannelCountInfoListeners;

/// <summary>
/// Disconnect() notification
/// </summary>
/// <param name="connectionAgent">The connection agent where the Disconnect() occurred</param>
/// <param name="result">>Disconnect() reason</param>
/// <param name="force">Whether to force termination</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelConnectionOnDisconnect onDisconnectListeners;

/// <summary>
/// Errors while using the basic functionality of the connection.
/// </summary>
/// <param name="connectionAgent">The connection agent where the error occurred</param>.
/// <param name="errorCode">Error code</param>
/// <param name="commands">Commands to raise an error</param>
public Interface.DelConnectionOnErrorCommand onErrorCommandListeners;

/// <summary>
/// Raises an error after a packet is sent.
/// </summary>
/// <param name="connectionAgent">The connection agent to request processing from</param>.
/// <param name="errorCode">Error code</param>
/// <param name="command">Message name of the packet</param>
public Interface.DelConnectionOnErrorCustomCommand onErrorCustomCommandListeners;
```
<br>

Next, the second method. IConnectionListener is an interface that defines the results or notifications of any action of the ConnectionAgent. You can register a listener that implements this interface with ConnectionAgent.AddConnectionListener() to receive responses from the registered listener. 

```c#
public class ConnectionListener : IConnectionListener
{
    /// <summary>
    /// The result of Connect()
    /// </summary>
    /// <param name="connectionAgent">The connection agent that requested Connect()</param>
    /// <param name="result">The result of Connect()</param>
    public void OnConnect(ConnectionAgent connectionAgent, ResultCodeConnect result) { }
    
    /// <summary>
    /// Authentication() result
    /// </summary>
    /// <param name="connectionAgent">The connection agent that requested Authentication()</param>.
    /// <param name="result">Authentication() request result</param>
    /// <param name="loginedUserInfoList">List of login information left on the server</param>
    /// <param name="message">Message received from the server</param>
    /// <param name="payload">Additional information received from the server</param>
    public void OnAuthentication(ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) { }

    /// <summary>
    /// GetChannelList() request result 
    /// </summary>
    /// <param name="connectionAgent">The connection agent that requested GetChannelList()</param>.
    /// <param name="result">Result of the GetChannelList() request</param>
    /// <param name="channelIdList">List of channels received from the server</param>
    public void OnChannelList(ConnectionAgent connectionAgent, ResultCodeChannelList result, List<string> channelIdList) { }

    /// <summary>
    /// The result of GetChannelInfo
    /// </summary>
    /// <param name="connectionAgent">ConnectionAgent</param>
    /// <param name="result">Result of GetChannelInfo</param>
    /// <param name="channelInfo">Channel information</param>
    public void OnChannelInfo(ConnectionAgent connectionAgent, ResultCodeChannelInfo result, Payload channelInfo) { }

    /// <summary>
    /// Results of all channel information requests.
    /// </summary>
    /// <param name="connectionAgent">The connection agent that requested GetAllChannelInfo().
    /// <param name="result">Result of the GetAllChannelInfo() request</param>
    /// <param name="channelInfo">Channel information received from the server</param>
    public void OnAllChannelInfo(ConnectionAgent connectionAgent, ResultCodeAllChannelInfo result, Dictionary<string, Payload> channelInfo) { }

    /// <summary>
    /// GetChannelCountInfo() request result
    /// </summary>
    /// <param name="connectionAgent">The connection agent that requested GetChannelCountInfo().
    /// <param name="result">Result of the GetChannelCountInfo() request</param>
    /// <param name="channelCountInfo">Channel's user count and room count information received from the server</param>
    public void OnChannelCountInfo(ConnectionAgent connectionAgent, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) { }

    /// <summary>
    /// GetAllChannelCountInfo() information request result
    /// </summary>
    /// <param name="connectionAgent">The connection agent that requested GetAllChannelCountInfo().
    /// <param name="result">Result of the GetAllChannelCountInfo() request</param>
    /// <param name="channelCountInfo">The channel's user count and room count information received from the server</param>
    public void OnAllChannelCountInfo(ConnectionAgent connectionAgent, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) { }
    
    /// <summary>
    /// Notifies the result of Disconnect() or forced connection termination.
    /// </summary>
    /// <param name="connectionAgent">The connection agent where Disconnect() occurred</param>
    /// <param name="result">Disconnect() result or reason for forced connection termination</param>
    /// <param name="force">Whether to force termination</param>
    /// <param name="payload">Additional information received from the server</param>
    public void OnDisconnect(ConnectionAgent connectionAgent, ResultCodeDisconnect result, bool force, Payload payload) { }
    
    /// <summary>
    /// Error while using the basic functions of the connection.
    /// </summary>
    /// <param name="connectionAgent">The connection agent where the error occurred</param>
    /// <param name="errorCode">Error code</param>
    /// <param name="commands">Commands to throw on error</param>
    public void OnError(ConnectionAgent connectionAgent, ErrorCode errorCode, Commands commands) { }
    
    /// <summary>
    /// Raises an error after sending a packet.
    /// </summary>
    /// <param name="connectionAgent">The connection agent to request processing from</param>.
    /// <param name="errorCode">Error code</param>
    /// <param name="command">Message name of the packet</param>
    public void OnError(ConnectionAgent connectionAgent, ErrorCode errorCode, string command) { }
}

/// <param name="command" /> connector.GetConnectionAgent().AddConnectionListener(new ConnectionListener);
```

