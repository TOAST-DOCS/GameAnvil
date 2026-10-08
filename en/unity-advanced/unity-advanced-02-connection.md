<!-- machine_translated: true -->

<!-- pre-align:aligned sig=7eecd13a676d -->

<a id="game-gameanvil-unity-advanced-development-guide-connector"></a>
## Game > GameAnvil > Unity Advanced Development Guide > Connector { #game-gameanvil-unity-advanced-development-guide-connector }

<a id="connectionagent"></a>
## Connector { #connectionagent }

The GameAnvilConnector is responsible for managing the connection to the GameAnvil server. It provides basic session management functions such as Connect() and Authentication(), as well as a list of channels.

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
        // Success
    }
    catch (Exception e)
    {
        // Failure
    }
}
```

Connect() has the following 2 parameters:

| Type   | Name | Description                              |
|--------|------|------------------------------------------|
| String | host | IP address or hostname of the server to connect to |
| int    | port | Port of the server to connect to         |

There is no return value. If the connection succeeds, the next code is executed; if it fails, an exception is thrown.

<a id="authentication"></a>
### Authentication { #authentication }

After connecting to the server, call Authentication() to perform the authentication process.
When you call the Authentication() method, the server calls the onAuthenticate() callback of the Connection object, and the processing of this callback determines whether the authentication succeeds or fails.

```c#
public async void Authenticate()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        Result<ResultCodeAuth, AuthenticationResult> result = await connector.Authentication("DeviceId", "AccountId", "Password", authenticationPayload);
        if(result.ResultCode == ResultCodeAuth.AUTH_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    }
    catch (Exception e)
    {
        // Exception
    }
}
```

Authentication() has the following 4 parameters:

| Type    | Name      | Description                                                                                                                                                          |
|---------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String  | deviceId  | Device ID to use for authentication. <br/>When authentication requests come from different devices, this value is used to identify which device made the request. Enter string.Empty (empty string) if not used. |
| String  | accountId | Account name to use for authentication.                                                                                                                              |
| String  | password  | Password to use for authentication. Enter string.Empty (empty string) if not used.                                                                                   |
| Payload | payload   | Additional information required by the user code on the server that processes the authentication request. (default = null)                                            |

The response returns Result<ResultCodeAuth, AuthenticationResult>. You can check the value of the ResultCode field to determine whether the request succeeded. If authentication succeeds, the value of the ResultCode field is ResultCodeAuth.AUTH_SUCCESS; otherwise, authentication has failed. You can obtain the AuthenticationResult from the Data field, which provides authentication result information and, depending on the server implementation, may also include additional information.

The details of ResultCodeAuth are as follows:

| Name                         | Value | Description                                                                                  |
|------------------------------|-------|----------------------------------------------------------------------------------------------|
| PARSE_ERROR                  | -2    | Packet parsing error. May occur if the server and client versions differ.                    |
| TIMEOUT                      | -1    | Timeout. A response to the request did not arrive within the specified time.                 |
| SYSTEM_ERROR                 | 1     | Server system error. Failed due to an unknown error on the server.                           |
| INVALID_PROTOCOL             | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| AUTH_SUCCESS                 | 0     | Success                                                                                      |
| AUTH_FAIL_CONTENT            | 201   | Failed. Rejected by user code.                                                               |
| AUTH_FAIL_INVALID_ACCOUNT_ID | 602   | Failed. Invalid account ID.                                                                  |

The details of AuthenticationResult are as follows:

| Type                         | Name              | Description                                                              |
|------------------------------|-------------------|--------------------------------------------------------------------------|
| List<AlreadyLoginedUserInfo> | LoginUserInfoList | List of logged-in user information remaining on the server at the time of successful authentication |
| String                       | Message           | Authentication message                                                   |
| Payload                      | payload           | Additional information required by the client                            |

<a id="connection-and-authentication"></a>
### Connection and Authentication { #connection-and-authentication }

You can also store the values needed for connection or authentication as properties of the GameAnvilConnector and use them from there.

```c#
public  void initializeConnector()
{
    connector.Host = "127.0.0.1";
    connector.Port = 18200;
    connector.DeviceId = "DeviceId";
    connector.AccountId = "AccountId";
    connector.Password = "Password";
}

public async void ConnectAndAuthentication1()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        var (resultCode, result) = await connector.ConnectAndAuthentication(authenticationPayload);
        if (resultCode == ResultCodeAuth.AUTH_SUCCESS)
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

public async void ConnectAndAuthentication2()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        await connector.Connect();
        var (resultCode, result) = await connector.Authentication(authenticationPayload);
        if (resultCode == ResultCodeAuth.AUTH_SUCCESS)
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

<a id="secure-connection"></a>
### Secure Connection { #secure-connection }
When security settings are configured on the GameAnvil server, you must call ConnectSecure() to establish a Secure Connection.
```c#
public async void ConnectSecure()
{
    try
    {
        await connector.ConnectSecure(host, port);
        // Success
    }
    catch (Exception e)
    {
        // Failure
    }
}
```
ConnectSecure() has the following 2 parameters:

| Type   | Name | Description                              |
|--------|------|------------------------------------------|
| String | host | IP address or hostname of the server to connect to |
| int    | port | Port of the server to connect to         |

There is no return value. If the connection succeeds, the next code is executed; if it fails, an exception is thrown.

<a id="channel-information"></a>
### Channel information { #channel-information }

GameAnvil allows you to freely configure channels in the settings. These channel configurations can be pre-agreed between the server and client and used in a fixed form, or they can be varied to suit the situation. GameAnvilConnector provides a few methods to get this changed channel information.

| Name                       | Description                                                                       |
|----------------------------|-----------------------------------------------------------------------------------|
| GetChannelList()           | Request a list of channel IDs for a specific service                              |
| GetChannelCountInfo()      | Request count information (number of users and rooms) for a specific channel      |
| GetChannelInfo()           | Request information (user-defined) for a specific channel                         |
| GetAllChannelCountInfo()   | Request count information (number of users and rooms) for all channels of a specific service |
| GetAllChannelInfo()        | Request information (user-defined) for all channels of a specific service         |

<a id="channel-information-getchannellist"></a>
#### GetChannelList

GetChannelList() can request and receive a list of channel IDs for a specific service.

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

GetChannelList() has the following 1 parameter:

| Type   | Name        | Description                            |
|--------|-------------|----------------------------------------|
| String | ServiceName | The service to request channel information from |

The response returns Result<ResultCodeChannelList, List<string>>. You can check the value of the ResultCode field to determine whether the request succeeded. If GetChannelList succeeds, the value of the ResultCode field is ResultCodeChannelList.CHANNEL_LIST_SUCCESS; otherwise, the request has failed. On success, you can obtain the list of channel IDs from the Data field.

The details of ResultCodeChannelList are as follows:

| Name                              | Value | Description                                                                                  |
|-----------------------------------|-------|----------------------------------------------------------------------------------------------|
| PARSE_ERROR                       | -2    | Packet parsing error. May occur if the server and client versions differ.                    |
| TIMEOUT                           | -1    | Timeout. A response to the request did not arrive within the specified time.                 |
| SYSTEM_ERROR                      | 1     | Server system error. Failed due to an unknown error on the server.                           |
| INVALID_PROTOCOL                  | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CHANNEL_LIST_SUCCESS              | 0     | Success                                                                                      |
| CHANNEL_LIST_FAIL_NO_CHANNEL_LIST | 1801  | Failed. Channel list not found.                                                              |

<a id="channel-information-getchannelcountinfo"></a>
#### GetChannelCountInfo

GetChannelCountInfo() can request and receive count information (number of users and rooms) for a specific channel.

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

GetChannelCountInfo() has the following 2 parameters:

| Type   | Name        | Description                                    |
|--------|-------------|------------------------------------------------|
| String | ServiceName | The service from which to request channel information |
| String | channelId   | The channel ID of the channel from which to request channel information |

It returns `Result<ResultCodeChannelCountInfo, ChannelCountResult>` as a response. You can check whether the request succeeded by checking the value of the `ResultCode` field. If GetChannelCountInfo succeeds, the value of the `ResultCode` field is `ResultCodeChannelCountInfo.CHANNEL_LIST_SUCCESS`; otherwise, the request has failed. You can obtain the `ChannelCountResult` from the `Data` field.

The details of ResultCodeChannelCountInfo are as follows:

| Name                                         | Value | Description                                                                                         |
|----------------------------------------------|-------|-----------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                  | -2    | Packet parsing error. May occur when the server and client versions differ.                         |
| TIMEOUT                                      | -1    | Timeout. The response to the request did not arrive within the specified time.                      |
| SYSTEM_ERROR                                 | 1     | Server system error. Failed due to an unknown error on the server.                                  |
| INVALID_PROTOCOL                             | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CHANNEL_COUNT_INFO_SUCCESS                   | 0     | Success                                                                                             |
| CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO      | 1921  | Failed. Channel information not found.                                                              |
| CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID   | 1922  | Failed. Invalid service ID.                                                                         |
| CHANNEL_COUNT_INFO_FAIL_INVALID_CHANNEL_ID   | 1923  | Failed. Invalid channel ID.                                                                         |
| CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND    | 1924  | Failed. Channel not found.                                                                          |

The details of ChannelCountResult are as follows:

| Type   | Name      | Description                          |
|--------|-----------|--------------------------------------|
| string | ChannelId | Returns the channel ID.              |
| int    | UserCount | Returns the number of users in the channel. |
| int    | RoomCount | Returns the number of rooms in the channel. |

<br>

<a id="channel-information-getchannelinfo"></a>
#### GetChannelInfo

GetChannelInfo() can request and receive information (user-defined) for a specific channel.

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

GetChannelInfo() has the following 2 parameters:

| Type   | Name        | Description                                  |
|--------|-------------|----------------------------------------------|
| String | ServiceName | The service to request channel information from |
| String | channelId   | The channel ID of the channel to request information from |

The response returns `Result<ResultCodeChannelInfo, Payload>`. You can check the value of the ResultCode field to determine whether the request succeeded. If GetChannelInfo() succeeds, the value of the ResultCode field is `ResultCodeChannelInfo.CHANNEL_LIST_SUCCESS`; otherwise, the request has failed. On success, you can also obtain the user-defined channel information from the Payload in the Data field.

The details of ResultCodeChannelInfo are as follows:

| Name                                 | Value | Description                                                                                          |
|--------------------------------------|-------|------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                          | -2    | Packet parsing error. May occur when the server and client versions differ.                          |
| TIMEOUT                              | -1    | Timeout. A response to the request did not arrive within the specified time.                         |
| SYSTEM_ERROR                         | 1     | Server system error. Failed due to an unknown error on the server.                                   |
| INVALID_PROTOCOL                     | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CHANNEL_INFO_SUCCESS                 | 0     | Success                                                                                              |
| CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1921  | Failed. Channel information not found.                                                               |
| CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1922  | Failed. Invalid service ID.                                                                          |
| CHANNEL_INFO_FAIL_INVALID_CHANNEL_ID | 1923  | Failed. Invalid channel ID.                                                                          |
| CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924  | Failed. Channel not found.                                                                           |

<a id="channel-information-getallchannelcountinfo"></a>
#### GetAllChannelCountInfo

GetAllChannelCountInfo() can request and receive count information (number of users and rooms) for all channels of a specific service.

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

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | The service for which to request channel information |

It returns Result<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>> as a response. You can check whether the request succeeded by checking the value of the ResultCode field. If GetAllChannelCountInfo succeeds, the value of the ResultCode field is ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS; otherwise, the request has failed. You can get the request result, Dictionary<string, ChannelCountResult>, through the Data field. This Dictionary uses the channel ID as the key and ChannelCountResult as the value.

The details of ResultCodeAllChannelCountInfo are as follows:

| Name                                           | Value | Description                                                                                          |
|------------------------------------------------|-------|------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                    | -2    | Packet parsing error. May occur when the server and client versions differ.                          |
| TIMEOUT                                        | -1    | Timeout. A response to the request did not arrive within the specified time.                         |
| SYSTEM_ERROR                                   | 1     | Server system error. The request failed due to an unknown server error.                              |
| INVALID_PROTOCOL                               | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| ALL_CHANNEL_COUNT_INFO_SUCCESS                 | 0     | Success.                                                                                             |
| ALL_CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1931  | Failed. Channel information not found.                                                               |
| ALL_CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1932  | Failed. Invalid service ID.                                                                          |
| ALL_CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1933  | Failed. Channel not found.                                                                           |

<a id="channel-information-getallchannelinfo"></a>
#### GetAllChannelInfo

GetAllChannelInfo() can request and receive information (user-defined) about all channels of a specific service.

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

GetAllChannelInfo() takes the following 1 parameter.

| Type | Name | Description |
|--------|-------------|----------------|
| String | ServiceName | The service for which to request channel information |

It returns a Result<ResultCodeAllChannelInfo, ChannelInfoResult>. You can check whether the request succeeded by checking the value of the ResultCode field. If GetAllChannelInfo succeeds, the ResultCode field value is ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS; otherwise, the request has failed. You can obtain the ChannelInfoResult from the Data field.

The details of ResultCodeAllChannelInfo are as follows:

| Name | Value | Description |
|------------------------------------------|------|----------------------------------------------|
| PARSE_ERROR | -2 | Packet parsing error. May occur if the server and client versions differ. |
| TIMEOUT | -1 | Timeout. The response to the request did not arrive within the allotted time. |
| SYSTEM_ERROR | 1 | Server system error. Failed due to an unknown server error. |
| INVALID_PROTOCOL | 2 | Protocol not registered on server. A protocol not registered in the additional information was used. |
| ALL_CHANNEL_INFO_SUCCESS | 0 | Success |
| ALL_CHANNEL_INFO_FAIL_NO_CHANNEL_INFO | 1911 | Failed. Channel information not found. |
| ALL_CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1912 | Failed. Invalid service ID. |
| ALL_CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND | 1913 | Failed. Channel not found. |

The channelInfo field of ChannelInfoResult is a Dictionary<string, Payload> that uses the channel ID as the key and a Payload containing user-defined channel information as the value. You can use this to obtain user-defined information for each channel.

<a id="terminate-the-connection"></a>
### Terminate the connection { #terminate-the-connection }

Use the Disconnect() method to terminate the connection to the server.

```c#
public async void Disconnect()
{
    try
    {
        await connector.Disconnect();
        // Success
    } catch (Exception e)
    {
        // Failure
    }
}
```

There is no separate return value. If successful, the following code is executed; if it fails, an exception is thrown.

<a id="terminate-the-connection-end-connection-notification"></a>
#### End Connection Notification
Even if you do not call Disconnect(), the connection can be terminated if the server forcibly closes it or if a network issue occurs, and you can receive a notification for this.
```c#
private void addOnDisconnect()
{
    connector.OnDisconnect += (ResultCodeDisconnect resultCode, Payload payload)=>{
        // Disconnection notification
    };
}
```

You can obtain information about the cause of the disconnection through the ResultCodeDisconnect parameter. If the server forcibly terminated the connection, you may also obtain additional information depending on the server implementation.