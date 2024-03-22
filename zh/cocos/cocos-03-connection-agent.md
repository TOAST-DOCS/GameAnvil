## Game > GameAnvil > CocosCreator Development Guide > ConnectionAgent

## ConnectionAgent

Connection Agent is responsible for tasks related to the connection nodes of GameAnvil server. It provides basic session management features and channel lists, such as Connect() and Authentication() and allows you to implement various contents based on your own defined protocols. ConnectionAgent is automatically created when the connector is initialized and can be obtained using Connector.GetConnectionAgent() function.

```typescript
let connectionAgent = connector.GetConnectionAgent();
```

### Connect to Server

Connect to the server using Connection Agent's Connect function. 

```typescript
// Connect to GameAnvil server 
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void; 
//  ipAddress : Server address to connect.  
//  callback : Callback to receive and process results
connectionAgent.Connect(ipAddress,(agent: ConnectionAgent, resultCode: ResultCodeConnect) => { 
    //  agent :ConnectionAgent object that called Connect(). 
	//  resultCode : Connect Result. 
    if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) { 
        // Success 
    } else { 
        // Failure 
    } 
});
```

### Authenticate

Use the Authenticate function of ConnectionAgent to proceed with the authentication procedure. Hand over deviceId, accountId, password, payload and callback to handle responses as factors. deviceId is used to process duplicate connections and can be used to process authentication on the server using accountId and password. If you need additional information other than deviceId, accountId and password for authentication, you can send it in payload. When the Authenticate function is called, the server calls an onAuthentication() callback for BaseConnection and the result of this callback determines the success or failure of the authentication.

```typescript
/** 
 * Request authentication to GameAnvil server 
 *  
 * Once authentication is successful, other features of the GameAnvil connector become available
 * @param deviceId Unique ID that identifies the user's device. Used to check duplicate connections of the same user 
 * @param accountId Unique ID to identify the user account
 * @param password Password for the user account
 * @param payload Additional information to pass to the server
 * @param callback Callback to receive and process results
 */ 
connectionAgent.Authenticate(deviceId, accountId, password, payload 
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => { 
    /** 
     * Result of Authentication() 
     * @param connection Connection agent requested Authentication() 
     * @param resultCode Result code of authentication 
     * @param loginedUserInfoList List of login information remaining on the server
     * @param message Messages received from the server, reasons for authentication failure, etc.     
* @param payload Additional information received from the server 
     */ 
    if (result == ResultCodeAuth.AUTH_SUCCESS) { 
		// Success  
    } else { 
		// Failure 
    } 
});
```

### Channel Information

GameAnvil is free to configure channel with settings. These channel configurations can be used in a fixed format by agreement between the server and the client in advance, but they can also be changed in various ways depending on the situation. ConnectionAgent provides several functions to help obtain this changed channel information. 

GetChannelList() can request and receive a list of channel IDs for specific service. 

```typescript
/** 
 * Request a list of available channels for a specific service
 * @param serviceName Service name to request channel information
 * @param callback Callback to receive and process results
 */ 
connectionAgent.GetChannelList(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeChannelList, channelIdList: Array<string>)=>{ 
    /** 
     * Result of GetChannelList() 
     * @param connection Connection agent requested GetChannelList()
     * @param resultCode Result of GetChannelList() 
     * @param channelIdList Channel list 
     */ 
    if (ResultCodeChannelList.CHANNEL_LIST_SUCCESS == resultCode) { 
        // Success  
    } else { 
        // Failure 
    } 
});
```

GetChannelCountInfo() can request and receive the count information of specific channel (count of users and rooms). 

```typescript
/** 
 * Request the count of rooms and users of the specific channel
 *  
 * Available when supported by the server
 * @param service Name Service name to request channel information
 * @param channelId Channel ID to request channel information
 * @param callback Callback to receive and process results
 */ 
connectionAgent.GetChannelCountInfo(serviceName, channelId, (agent: ConnectionAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo)=>{ 
    /** 
     * Result of GetChannelCountInfo() 
     * @param connection Connection agent requested GetChannelCountInfo() 
     * @param resultCode Result code of GetChannelCountInfo() 
     * @param channelCountInfo  Count of users and rooms of Channel
     */ 
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) { 
        // Success  
    } else { 
        // Failure 
    } 
});
```

GetChannelInfo() can request and receive information about specific channel (customization). 

```typescript
/** 
 * Request information of specific channel 
 *  
 * Available when supported by the server
 * @param serviceName Service name to request channel information
 * @param channelId Channel ID to request channel information 
 * @param callback Callback to receive and process results
 */ 
connectionAgent.GetChannelInfo(serviceName, channelId, (agent: ConnectionAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload)=>{ 
    /** 
     * Result of GetChannelInfo() 
     * @param connection Connection agent requested GetChannelInfo() 
     * @param resultCode Result code of GetChannelInfo() 
     * @param channelInfo Channel information 
     */ 
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) { 
        // Success  
    } else { 
        // Failure 
    } 
});
```

GetAllChannelCountInfo() can request and receive the count information of all channel for specific service. (count of users and rooms). 

```typescript
/** 
 * Request the count of users and rooms for all channels in a specific service 
 *  
 * Available when supported by the server
 * @param serviceName Service name to request channel information 
 * @param callback Callback to receive and process results
 */ 
connectionAgent.GetAllChannelCountInfo(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo)=>{ 
    /** 
     * Result of GetAllChannelCountInfo() 
     * @param connection Connection agent requested  GetAllChannelCountInfo() 
     * @param resultCode Result code of GetAllChannelCountInfo() 
     * @param channelInfoList Count of users and rooms of Channel
     */ 
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) { 
        // Success  
    } else { 
        // Failure 
    } 
});
```

GetAllChannelInfo() can request and receive information about all channels for specific service (customization). 

```typescript
/** 
 * Request the count of users and rooms for all channels in a specific service
 *  
 * Available when supported by the server
 * @param serviceName Service name to request channel information 
 * @param callback Callback to receive and process results
 */ 
connectionAgent.GetAllChannelInfo(serviceName, (agent: ConnectionAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo)=>{ 
    /** 
     * Result of GetAllChannelInfo() 
     * @param connection Connection agent requested GetAllChannelInfo() 
     * @param resultCode Result of GetAllChannelInfo 
     * @param allChannelInfo Channel information
     */ 
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) { 
        // Success  
    } else { 
        // Failure 
    } 
});
```

### Terminate Connection

Use the Disconnect function of ConnectionAgent to terminate the connection to the server. 

```typescript
/** 
 * Disconnect from GameAnvil Server
 * @param reason Reasons to disconnect to pass to callback
 * @param callback Callback to be called upon termination of connection
 */ 
connectionAgent.Disconnect(reason, (agent: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string) =>{ 
    /** 
     *Result of Disconnect() or notify forced disconnection  
     * @param connection Connection agent requested Disconnect() 
     * @param resultCode Result code of Disconnect() 
     * @param reason Reason for disconnection. Reason to pass to ConnectionAgent.Disconnect() as factor  
     */ 
    if (ResultCodeDisconnect.SOCKET_DISCONNECT == resultCode) { 
        // Success  
    } else { 
        // Failure 
    } 
});
```

### Listener

IConnectionListener is an interface that defines the results or notifications of all the actions of ConnectionAgent. If you register a listener that implements this interface as a ConnectionAgent.AddConnectionListener(), you can receive a response from the registered listener. 

```typescript
class ConnectionListener implements IConnectionListener{ 
    /** 
     *Result of Connect() 
     * @param connection Connection agent requested Connect() 
     * @param resultCode Result code of connection 
     */ 
    OnConnect(connection: ConnectionAgent, resultCode: ResultCodeConnect): void { } 
    /** 
     * Result of Authentication() 
     * @param connection Connection agent requested Authentication() 
     * @param resultCode Result code of authentication 
     * @param loginedUserInfoList List of login information remaining on the server
     * @param message Messages received from the server, reasons for authentication failure, etc.         
   * @param payload Additional information received from the server
     */ 
    OnAuthentication(connection: ConnectionAgent, resultCode: ResultCodeAuth, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload): void { } 
    /** 
*Result of GetChannelList() 
     * @param connection Connection result requested GetChannelList() 
     * @param resultCode Result code of GetChannelList() 
     * @param channelIdList Channel list 
     */ 
    OnChannelList(connection: ConnectionAgent, resultCode: ResultCodeChannelList, channelIdList: Array<string>): void { } 
    /** 
     * Result of GetChannelInfo()
     * @param connection Connection agent requested GetChannelInfo() 
     * @param resultCode Result code of GetChannelInfo() 
     * @param channelInfo Channel information
     */ 
    OnChannelInfo(connection: ConnectionAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload): void { } 
    /** 
     * Result of GetChannelCountInfo() 
     * @param connection Connection agent requested GetChannelCountInfo() 
     * @param resultCode Result code of GetChannelCountInfo() 
     * @param channelCountInfo Count of users and rooms of Channel
     */ 
    OnChannelCountInfo(connection: ConnectionAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo): void { } 
    /** 
     * Result of GetAllChannelInfo() 
     * @param connection Connection agent requested GetAllChannelInfo() 
     * @param resultCode Result code of GetAllChannelInfo 
     * @param allChannelInfo Channel information
     */ 
    OnAllChannelInfo(connection: ConnectionAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo): void { } 
    /** 
     * Result code of GetAllChannelCountInfo() 
     * @param connection Connection agent requested GetAllChannelCountInfo() 
     * @param resultCode Result code of GetAllChannelCountInfo() 
     * @param channelInfoList Count of users and rooms of Channel
     */ 
    OnAllChannelCountInfo(connection: ConnectionAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo): void { } 
    /** 
     * Result of Disconnect() or notify forced disconnection  
     * @param connection Connection result requested Disconnect()
     * @param resultCode Result code of Disconnect() 
     * @param reason Reason for disconnection. 
Reason to pass to ConnectionAgent.Disconnect()  as a factor 
     * @param force Forced to disconnect or not 
     * @param payload Additional information received from the server
     */ 
    OnDisconnect(connection: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload): void { } 
    /** 
     * Error occurred 
     * @param connection Connection agent that error occurred 
     * @param errorCode Error code 
     * @param msgName Message name that error occurred 
     */ 
    OnErrorCommand(connection: ConnectionAgent, errorCode: ErrorCode, msgName: string): void { } 
}; 
 
connectionAgent.AddListener(new ConnectionListener());
```

