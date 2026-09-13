<!-- pre-align:aligned sig=7eecd13a676d -->

<a id="game-gameanvil-unity-advanced-development-guide-connector"></a>
## Game > GameAnvil > Unity Advanced Development Guide > Connector { #game-gameanvil-unity-advanced-development-guide-connector }

<a id="connectionagent"></a>
## ConnectionAgent { #connectionagent }

The ConnectionAgent is responsible for operations related to the Connection node on the GameAnvil server. It provides basic session management functions such as Connect() and Authentication(), as well as a list of channels, and can implement different content based on your own defined protocols. The ConnectionAgent is automatically created when the connector is initialized and can be obtained using the Connector.GetConnectionAgent() function.

```c#
ConnectionAgent connectionAgent = connector.GetConnectionAgent();
```

<a id="connect-to-the-server"></a>
### Connect to the server { #connect-to-the-server }

Call Connect() to connect to the server.

```c#
public async void Connect()
{
    try
    {
        // connector is a variable that stores a GameAnvilConnector instance.
        await connector.Connect(host, port);
        // 성공
    }
    catch (Exception e)
    {
        // 실패
    }
}
```

Connect() has the following two parameters:

| Type   | Name | Description                          |
|--------|------|--------------------------------------|
| String | host | IP address or hostname of the server to connect to |
| int    | port | Port of the server to connect to     |

There is no return value. If the call succeeds, the following code is executed; if it fails, an exception is thrown.

<a id="authentication"></a>
### Authentication { #authentication }

After connecting to the server, call Authentication() to proceed with the authentication process. When you call the Authentication() method, the server calls the onAuthenticate() callback of the Connection object, and the result of this callback determines whether authentication succeeds or fails.

```c#
public async void Authenticate()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        Result<ResultCodeAuth, AuthenticationResult> result = await connector.Authentication("DeviceId", "AccountId", "Password", authenticationPayload);
        if(result.ResultCode == ResultCodeAuth.AUTH_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    }
    catch (Exception e)
    {
        // 예외
    }
}
```

Authentication() has the following four parameters:

| Type    | Name      | Description                                                                                                                                                                                                         |
|---------|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String  | deviceId  | Device ID for authentication. <br/>When authentication requests come from different devices, this value can be used to identify which device made the request. If not used, enter string.Empty (empty string). |
| String  | accountId | Account name for authentication.                                                                                                                                                                                    |
| String  | password  | Password for authentication. If not used, enter string.Empty (empty string).                                                                                                                                        |
| Payload | payload   | Additional information required by the user code on the server that processes the authentication request. (default = null)                                                                                          |

The method returns Result<ResultCodeAuth, AuthenticationResult> as the response. You can check the value of the ResultCode field to determine whether the request was successful. If authentication succeeds, the ResultCode field value is ResultCodeAuth.AUTH_SUCCESS; otherwise, authentication has failed. You can obtain the AuthenticationResult from the request through the Data field. This allows you to retrieve authentication result information, and additional information may also be available depending on the server implementation.

The details of ResultCodeAuth are as follows:

| Name                         | Value | Description                                                                                              |
|------------------------------|-------|----------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                  | -2    | Packet parsing error. May occur when the server and client versions differ.                              |
| TIMEOUT                      | -1    | Timeout. No response to the request was received within the specified time.                              |
| SYSTEM_ERROR                 | 1     | Server system error. Failed due to an unknown error on the server.                                       |
| INVALID_PROTOCOL             | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| AUTH_SUCCESS                 | 0     | Success.                                                                                                 |
| AUTH_FAIL_CONTENT            | 201   | Failed. Rejected by user code.                                                                           |
| AUTH_FAIL_INVALID_ACCOUNT_ID | 602   | Failed. Invalid account ID.                                                                              |

The details of AuthenticationResult are as follows:

| Type                         | Name              | Description                                                                                      |
|------------------------------|-------------------|--------------------------------------------------------------------------------------------------|
| List<AlreadyLoginedUserInfo> | LoginUserInfoList | List of logged-in user information remaining on the server at the time of successful authentication. |
| String                       | Message           | Authentication message.                                                                          |
| Payload                      | payload           | Additional information required by the client.                                                   |

<a id="connection-and-authentication"></a>
### Connection and Authentication { #connection-and-authentication }

<!-- TODO: translate body -->

<a id="secure-connection"></a>
### Secure Connection { #secure-connection }

When security settings are configured on the GameAnvil server, you must call ConnectSecure() to establish a secure connection.
```c#
public async void ConnectSecure()
{
    try
    {
        await connector.ConnectSecure(host, port);
        // 성공
    }
    catch (Exception e)
    {
        // 실패
    }
}
```
ConnectSecure() has the following two parameters:

| Type   | Name | Description                                    |
|--------|------|------------------------------------------------|
| String | host | IP address or hostname of the server to connect to |
| int    | port | Port of the server to connect to               |

There is no return value. If the call succeeds, the following code is executed; if it fails, an exception is thrown.

<a id="channel-information"></a>
### Channel information { #channel-information }

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

<a id="channel-information-getchannellist"></a>
#### GetChannelList

GetChannelList() retrieves a list of channel IDs for a specific service.

```c#
public async void ChannelList()
{
    try
    {
        Payload channelInfoPayload = new Payload(new Protocol.ChannelInfoData());
        Result<ResultCodeChannelList, List<string>> result = await connector.GetChannelList("ServiceName");
        if (result.ResultCode == ResultCodeChannelList.CHANNEL_LIST_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

GetChannelList() has the following parameter:

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | Service to request channel information from |

Returns Result<ResultCodeChannelList, List<string>> as a response. You can check whether the request succeeded by checking the value of the ResultCode field. If GetChannelList succeeds, the value of the ResultCode field is ResultCodeChannelList.CHANNEL_LIST_SUCCESS; otherwise, the request has failed. On success, you can get the resulting list of channel IDs through the Data field.

The details of ResultCodeChannelList are as follows:

| Name                              | Value | Description                                                                                      |
|-----------------------------------|-------|--------------------------------------------------------------------------------------------------|
| PARSE_ERROR                       | -2    | Packet parsing error. May occur when the server and client versions differ.                      |
| TIMEOUT                           | -1    | Timeout. The response to the request did not arrive within the allotted time.                    |
| SYSTEM_ERROR                      | 1     | Server system error. Failed due to an unknown error on the server.                               |
| INVALID_PROTOCOL                  | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| CHANNEL_LIST_SUCCESS              | 0     | Success                                                                                          |
| CHANNEL_LIST_FAIL_NO_CHANNEL_LIST | 1801  | Failed. The channel list could not be found.                                                     |

<a id="channel-information-getchannelcountinfo"></a>
#### GetChannelCountInfo

`GetChannelCountInfo()` retrieves count information (number of users and rooms) for a specific channel.

```c#
public async void ChannelCountInfo()
{
    try
    {
        Result<ResultCodeChannelCountInfo, ChannelCountResult> result = await connector.GetChannelCountInfo("ServiceName", "ChannelId");
        if (result.ResultCode == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

`GetChannelCountInfo()` takes the following two parameters:

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | Service for which to request channel information |
| String | channelId   | ID of the channel for which to request channel information |

The method returns `Result<ResultCodeChannelCountInfo, ChannelCountResult>`. You can check the value of the `ResultCode` field to determine whether the request succeeded. If `GetChannelCountInfo` succeeds, the value of the `ResultCode` field is `ResultCodeChannelCountInfo.CHANNEL_LIST_SUCCESS`; otherwise, the request has failed. You can obtain the `ChannelCountResult` from the `Data` field.

The details of `ResultCodeChannelCountInfo` are as follows:

| Name                                       | Value | Description                                                                                       |
|--------------------------------------------|-------|---------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                | -2    | Packet parsing error. May occur when the server and client versions differ.                       |
| TIMEOUT                                    | -1    | Timeout. A response to the request did not arrive within the specified time.                      |
| SYSTEM_ERROR                               | 1     | Server system error. Failed due to an unknown error on the server.                                |
| INVALID_PROTOCOL                           | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| CHANNEL_COUNT_INFO_SUCCESS                 | 0     | Success.                                                                                          |
| CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1921  | Failed. Channel information not found.                                                            |
| CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1922  | Failed. Invalid service ID.                                                                       |
| CHANNEL_COUNT_INFO_FAIL_INVALID_CHANNEL_ID | 1923  | Failed. Invalid channel ID.                                                                       |
| CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924  | Failed. Channel not found.                                                                        |

The details of `ChannelCountResult` are as follows:

| Type   | Name      | Description                          |
|--------|-----------|--------------------------------------|
| string | ChannelId | Returns the channel ID.              |
| int    | UserCount | Returns the number of users in the channel. |
| int    | RoomCount | Returns the number of rooms in the channel. |

<br>

<a id="channel-information-getchannelinfo"></a>
#### GetChannelInfo

You can call GetChannelInfo() to request and retrieve information (user-defined) about a specific channel.

```c#
public async void ChannelInfo()
{
    try
    {
        Result<ResultCodeChannelInfo, Payload> result = await connector.GetChannelInfo("ServiceName", "ChannenId");
        if (result.ResultCode == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

GetChannelInfo() has the following two parameters:

| Type   | Name        | Description                                  |
|--------|-------------|----------------------------------------------|
| String | ServiceName | Service to request channel information from  |
| String | channelId   | ID of the channel to request information from |

The response returns Result<ResultCodeChannelInfo, Payload>. You can check the value of the ResultCode field to determine whether the request succeeded. If GetChannelInfo() succeeds, the value of the ResultCode field is ResultCodeChannelInfo.CHANNEL_LIST_SUCCESS; otherwise, the request has failed. On success, you can also retrieve user-defined channel information from the Payload in the Data field.

The details of ResultCodeChannelInfo are as follows:

| Code Name                            | Value | Description                                                                                   |
|--------------------------------------|-------|-----------------------------------------------------------------------------------------------|
| PARSE_ERROR                          | -2    | Packet parsing error. This may occur when the server and client versions differ.              |
| TIMEOUT                              | -1    | Timeout. A response to the request did not arrive within the allotted time.                   |
| SYSTEM_ERROR                         | 1     | Server system error. The request failed due to an unknown server error.                       |
| INVALID_PROTOCOL                     | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CHANNEL_INFO_SUCCESS                 | 0     | Success                                                                                       |
| CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1921  | Failed. Channel information not found.                                                        |
| CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1922  | Failed. Invalid service ID.                                                                   |
| CHANNEL_INFO_FAIL_INVALID_CHANNEL_ID | 1923  | Failed. Invalid channel ID.                                                                   |
| CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924  | Failed. Channel not found.                                                                    |

<a id="channel-information-getallchannelcountinfo"></a>
#### GetAllChannelCountInfo

GetAllChannelCountInfo() retrieves count information (the number of users and rooms) for all channels in a specific service.

```c#
public async void AllChannelCountInfo()
{
    try
    {
        Result<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>> result = await connector.GetAllChannelCountInfo("ServiceName");
        if (result.ResultCode == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

GetAllChannelCountInfo() has the following 1 parameter:

| Type | Name | Description |
|--------|-------------|----------------|
| String | ServiceName | Service to request channel information from |

It returns Result<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>> as the response. You can check the value of the ResultCode field to determine whether the request was successful. If GetAllChannelCountInfo succeeds, the value of the ResultCode field is ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS; otherwise, the request has failed. You can retrieve the result Dictionary<string, ChannelCountResult> through the Data field. This dictionary uses the channel ID as the key and ChannelCountResult as the value.

The details of ResultCodeAllChannelCountInfo are as follows:

| Name                                           | Value | Description                                                                                     |
|------------------------------------------------|-------|-------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                    | -2    | Packet parsing error. May occur when the server and client versions differ.                     |
| TIMEOUT                                        | -1    | Timeout. A response to the request did not arrive within the specified time.                    |
| SYSTEM_ERROR                                   | 1     | Server system error. Failed due to an unknown server error.                                     |
| INVALID_PROTOCOL                               | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| ALL_CHANNEL_COUNT_INFO_SUCCESS                 | 0     | Success                                                                                         |
| ALL_CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1931  | Failed. Channel information not found.                                                          |
| ALL_CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1932  | Failed. Invalid service ID.                                                                     |
| ALL_CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1933  | Failed. Channel not found.                                                                      |

<a id="channel-information-getallchannelinfo"></a>
#### GetAllChannelInfo

You can use GetAllChannelInfo() to request and retrieve information (user-defined) for all channels in a specific service.

```c#
public async void AllChannelInfo()
{
    try
    {
        Result<ResultCodeAllChannelInfo, ChannelInfoResult> result = await connector.GetAllChannelInfo("ServiceName");
        if (result.ResultCode == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

GetAllChannelInfo() has the following 1 parameter.

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | Service to request channel information from |

It returns Result<ResultCodeAllChannelInfo, ChannelInfoResult> as a response. You can check the value of the ResultCode field to determine whether the request succeeded. If GetAllChannelInfo succeeds, the value of the ResultCode field is ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS; otherwise, the request has failed. You can retrieve the request result ChannelInfoResult through the Data field.

The details of ResultCodeAllChannelInfo are as follows.

| Name                                     | Value | Description                                                                                      |
|------------------------------------------|-------|--------------------------------------------------------------------------------------------------|
| PARSE_ERROR                              | -2    | Packet parsing error. May occur if the server and client versions differ.                        |
| TIMEOUT                                  | -1    | Timeout. No response to the request received within the specified time.                          |
| SYSTEM_ERROR                             | 1     | Server system error. Failed due to an unknown server error.                                      |
| INVALID_PROTOCOL                         | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| ALL_CHANNEL_INFO_SUCCESS                 | 0     | Success                                                                                          |
| ALL_CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1911  | Failed. Channel information not found.                                                           |
| ALL_CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1912  | Failed. Invalid service ID.                                                                      |
| ALL_CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1913  | Failed. Channel not found.                                                                       |

The channelInfo field of ChannelInfoResult is a Dictionary<string, Payload> that uses the channel ID as the key and a Payload containing user-defined channel information as the value. You can use this to retrieve user-defined information for each channel.

<a id="terminate-the-connection"></a>
### Terminate the connection { #terminate-the-connection }

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

<a id="terminate-the-connection-end-connection-notification"></a>
#### End Connection Notification

<!-- TODO: translate body -->

