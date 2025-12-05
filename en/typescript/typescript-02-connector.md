## Game > GameAnvil > TypeScript Development Guide > Connection Agent

## GameAnvilConnector

GameAnvilConnector is a class for server connections and communication, which allows you to register and manage the handler for messages coming from the server via this object. The previous installation chapter covered the creation and use of the connect feature for GameAnvilConnector. In this article, we will learn more about how to use and other features of GameAnvilConnector.

### Create

Create a GameAnvilConnector object as shown below: It is common to use one connector object in one process.

```typescript
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();
```

### Server Connection

Connect to the server using the connect() function. You must pre-set the host and port before calling.

```typescript
connector.host = "127.0.0.1";
connector.port = 18300;

await connector.connect();

console.log("Connect Success");

```

The connect() function returns a Promise object that has been completed after the server connection is completed or failed. Accordingly, it can be used as follows:

```typescript
connector.host = "127.0.0.1";
connector.port = 18300;

connector.connect()
    .then(() => {
        console.log("Connect Success");
    })
    .catch(()=> {
        console.log("Connect Fail");
    })
```

Most APIs on the connector operate asynchronously like this and return the Promise object, allowing you to use await, then, and other features for the situation above.

### Detect Disconnection

You can specify what to do when the connection is forcibly terminated or disconnected due to network problems, etc. by the server.

```typescript
connector.onDisconnect = (resultCode: ResultCodeDisconnect, payload: Payload) => {
    console.log("Disconnected.");
    
}
```

The first argument in the function lets you know why the connection has been lost.

| Code Name | Value | Description |
|---------------------------------------------------|-------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `FORCE_CLOSE_SYSTEM_ERROR` | 2000 | This is a forced close situation due to a system error. If you receive this code from the client, please contact the GameAnvil development team. |
| `FORCE_CLOSE_BASE_CONNECTION`  | 2010 | This code is received when the server calls close() on BaseConnection. |
| `FORCE_CLOSE_BASE_USER` | 2011 | This code is received when the server calls closeConnection() on BaseUser. |
| `FORCE_CLOSE_ADMIN_KICK` | 2012 | This code is received when the Admin user forcibly closes the game. |
| `FORCE_CLOSE_INVALID_NODE` | 2020 | The GameNode has become invalid. |
| `FORCE_CLOSE_USER_TRANSFER_FAIL` | 2021 | If a user transfer fails. |
| `FORCE_CLOSE_USER_TRANSFER_ERROR` | 2022 | If a system error occurs during a user transfer. |
| `FORCE_CLOSE_AUTHENTICATION_FAIL` | 2030 | Forced close due to authentication failure |
| `FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID` | 2031 | Forced close due to authentication failure - if the account ID does not exist |
| `FORCE_CLOSE_DUPLICATE_LOGIN` | 2032 | Forced close due to duplicate login |
| `FORCE_CLOSE_BY_NEW_CONNECTION` | 2040 | If a new login request is received with the same account information. Close the previous connection when re-connecting to the network order, etc. Inquiry required. |
| `FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT` | 2041 | Detect disconnection with the client. Inquiry required.|
| `FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION` | 2042 | Could not find the session information. Inquiry required.|
| `FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL` | 2043 | The client does not respond to the server status check. Inquiry required. |
| `FORCE_CLOSE_GHOST_USER` | 2044 | For ghost users. Inquiry required. |
| `SOCKET_DISCONNECT` | 2100 | Network connection lost |
| `SOCKET_TIME_OUT`  | 2101 | A timeout occurred, the connector closed the connection |
| `SOCKET_ERROR` | 2102 | A socket error occurred, the connection was closed |


The second argument receives additional information based on server implementation. How to process additional information will be explained further later.

### Authentication

After successfully connecting to the server, authentication must be proceeded first to use all functions of the engine. The authentication() function indices the pre-conferred accountId, deviceId, and password values with the server to perform the authentication action and returns the Promise. When the authentication is complete, you can check whether the authentication was successful through Promise, additional data received from the server, and more.

```typescript
const deviceId, accountId, password;

const authResult = await connector.authentication(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);
```

You can check whether authentication is successful through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (authResult.resultCode === ResultCodeAuth.AUTH_SUCCESS) {
    console.log("Authentication success");
} else {
    console.log("Authentication fail");
}
```

If authentication fails, you can find out about the source through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|----------------------------------|-------|------------------------------------|
| `PARSE_ERROR` | -2 | Error occurred if parsing of additional information fails when additional information is sent. |
| `TIMEOUT`  | -1 | Error sent when the server does not respond within the timeout period. |
| `SYSTEM_ERROR`  | 1 | Server system error. |
| `INVALID_PROTOCOL` | 2 | Error occurred when additional information is sent using a protocol not registered with the server. |
| `AUTH_SUCCESS` | 0 | Success. |
| `AUTH_FAIL_CONTENT` | 201 | Error sent when the authentication code written by the server developer fails due to rejection. |
| `AUTH_FAIL_INVALID_ACCOUNT_ID` | 202 | Error occurred when an invalid account ID that the server cannot process, such as an empty string, is sent. |

You can check the isAuthenticated attribute to commonly check whether the connector is authenticated before and after authentication.

```typescript
if (connector.isAuthenticated) {
    console.log("Already authenticated.");
}
```

### Proceed with Both Connection and Authentication

Once connected, authentication must be proceeded as required, making it convenient to call the convenience function to run both successively.

```typescript
const deviceId, accountId, password;

connector.host = "127.0.0.1";
connector.port = 18300;

const authResult = await connector.connectAndAuthenticateion(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);
```

Perform the connection and authentication to the unconnected connector in turn, and return the authentication result.

Authentication results can generally be used in the same way as the results when authentication is requested only.

### Request Ping

By default, ping requests are to be sent periodically, but if you modify the settings manually, you can call the method manually to make a ping request.

```typescript
connector.ping();
```

If you have received a response to a ping request, pong logs may be output depending on the settings.


### Register Message Reception Callback

When you receive a protobuf message from the server, you can set the processing function to run. For one protobuf, only one process function can be registered, and if you re-register the process function while the number is already registered, the existing process function will be deleted.

Please note that the processing function runs asynchronously, so it may not be called exactly at the intended time.

In the next chapter, we will cover how to transpile the protobuf message that enters the first argument.

```typescript
connector.setMessageCallback(UserInfo.descriptor, (connector, resultCode, userInfo) => {
    if (resultCode === ResultCode.SUCCESS) {
        console.log(userInfo.name);
        console.log(userInfo.age);
        console.log(userInfo.job);

        return;
    }
    console.log("Message receive fail.", resultCode);
});
```

### Request Channel User and Room Count Information

You can request information on the number of servers and users of each channel for each particular service on the server.

```typescript
const serviceName: string;

const result = await connector.getAllChannelCountInfo(serviceName);
const allChannelCountInfo = result.data;

for (let [key, value] of allChannelCountInfo) {
    const channelId = key;
    const channlCountInfo = value;
    console.log(`${channelId} userCount: ${channlCountInfo.userCount}, roomCount: ${channlCountInfo.roomCount}`);
}
```

You can also request to be restricted to certain channels in certain services. In this case, the channel ID is an argument along with the service name.

```typescript
const serviceName: string;
const channelId: string;

const result = await connector.getChannelCountInfo(serviceName, channelId);
const channlCountInfo = result.data;

console.log(`${channelCountInfo.channelId} userCount: ${channlCountInfo.userCount}, roomCount: ${channlCountInfo.roomCount}`);
```


### Request Channel Information

You can request information on each channel of a particular service on the server.

```typescript
const serviceName: string;

const result = await connector.getAllChannelInfo(serviceName);
const channelInfo = result.data.channelInfo;

for (let [key, value] of channelInfo) {
    const channelId = key;
    const payload = value;
}
```

You can also request to be restricted to certain channels in certain services. In this case, the channel ID is an argument along with the service name.

```typescript
const serviceName: string;
const channelId: string;

const result = await connector.getChannelInfo(serviceName, channelId);
const payload = result.data;
```

### Request Channel List

You can request a list of all channels for specific services on the server.

```typescript
const serviceName: string;

const result = await getChannelList(serviceName);

for (let channelId of result) {
    console.log(channelId);
}
```

### Pause and Resume User Status Check

You can stop the app if you expect situations such as being unable to respond to user status check when the app is in the background.

The client access status information will not be updated from the server for the specified time period.

If you use this method to stop the app from responding, the server will periodically terminate the client connection.

```typescript
const pauseTime: number = 10 * 1000;

connector.pauseClientStateCheck(pauseTime);
```

If you restart the app from the background, reset the app if you can now respond to the user status check.

```typescript
connector.resumeClientStateCheck();
```

### Send Packet

You can send the protobuf message to the gateway server.

See the documentation that follows the creation of message objects:

```typescript
const name: string;
const age: number;
const job: string;

connector.send(new UserInfo({name, age, job}));
```

If you want to wait for a response to the message from the server, you can do the following:

In this case, the response must be sent through a pre-registered listener on the server.

```typescript
const result = await connector.requestMessage(new StartReq(), StartRes.descriptor);

if (result.resultCode === ResultCode.Success) {
    const startRes: StartRes = result.data;
}
```

If the message is already in the form of a packet, it can also be sent as it is.

```typescript
const packet = PacketFactory.makePacket(new StartReq());
const result = await connector.requestPacket(packet, StartRes.descriptor);

if (result.resultCode === ResultCode.Success) {
    const startRes: StartRes = result.data;
}
```

### Except Handling

You can specify what to do if an exception occurs during a server connection.

```typescript
connector.onException = (exception: Error) => {
    console.log("Error occured.", exception);
}
```

The first argument of the registered function sends an error object.

### End Connection

You can explicitly request that the connection to the server be terminated.

```typescript
connector.disconnect();
```

If the server is not connected but the connection is terminated, an error occurs.

It is recommended to terminate the connection before terminating the gameplay by calling the connector.disconnect() function. If not terminated, the server might not recognize the client's termination, and if not, you can continue to operate unnecessarily. It is also a good option to make the components managing connectors to be destroyed only at the end of the game and make them call connector.disconnect() in OnDestroy(). 