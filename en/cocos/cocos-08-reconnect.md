## Game > GameAnvil > CocosCreator Development Guide > Reconnect

## Reconnect

You may lose connection to the server for various reasons when playing the game. It supports reconnection so that you can continue playing when losing connection. The API you use is the same as the typical access method, but there is a difference in the information you receive as a result. 

### Connect to Server

Connect to the server using ConnectionAgent's Connect function. It's the same as general case.  

```typescript
// Connect to the GameAnvil server 
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void; 
//  ipAddress : Server address to connect.  
//  callback : Callback to receive and process results
connectionAgent.Connect(ipAddress,(agent: ConnectionAgent, resultCode: ResultCodeConnect) => { 
    //  agent : ConnectionAgent object requested Connect(). 
	//  resultCode : Connect result. 
    if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) { 
        // Success  
    } else { 
        // Failure 
    } 
});
```

### Authenticate

Use the Authenticate function of the ConnectionAgent to proceed with the authentication process. The input values are the same as general case. However, among the values received as a result of authentication, the previously played user information will be included in loginedUserInfoList. 

```typescript
/** 
 * Request to verify to GameAnvil server
 *  
 * Once authentication is successful, other features of the GameAnvil connector can be used
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
        for(let loginedUserInfo of loginedUserInfoList){ 
            // Player user information 
        } 
    } else { 
		// Failure 
    } 
});
```



### Login

Proceed to log in using the user information received as a result of authentication. At this time, you have to log in using the same value as the previous user information, such as userType or channelId, etc. Otherwise, the login may fail. 

```typescript
/** 
 * Login to the server 
 * @param userType User type used to log in 
 * @param payload Additional information to pass to the server. May not be used depending on the server implementation
 * @param channelId Channel ID to log in
 * @param callback Callback to receive and process login results
 */ 
userAgent.Login(userType, payload, channelId, (agent: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo)=>{ 
    /** 
     * Result of Login() 
     * @param user User agent requested Login() 
     * @param resultCode Login() Result code  
     * @param loginInfo Login() Information
     */ 
    if (ResultCodeLogin.LOGIN_SUCCESS == resultCode) { 
        // Success  
        if (loginInfo.isRelogined) { 
            // Reconnection
        } else { 
            // First connection
        } 
    } else { 
        // Failure 
    } 
})
```

If LoginInfo's isRelogined is true when login is successful, it means successful reconnection login. You can reconnect based on the information contained in LoginInfo. The most important thing at this time is to synchronize the changed data on the server to the client during the reconnection time. The data required for this synchronization can be processed in Payload of LoginInfo.  

After the reconnection login, the server calls the BaseUser.onRelogin() callback. At this time, if you define the messages required for synchronization and add them to the outPayload parameter, you can receive them to Payload of LoginInfo and process them. 

If isRelogined is false, it means that the previous user information is removed over time and then logged in. In this case, you can do the initial login procedure. 
