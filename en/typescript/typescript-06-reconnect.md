## Game > GameAnvil > CocosCreator Development Guide > Relogin

## Relogin

During the game, connection to the server may be lost for various reasons. The relogin feature is supported so you can continue your existing play when your connection is lost. The API you use can be used the same as the normal access method. The result gives you more detailed information about the relogin.

### Authentication

When authentication is proceeded after relogin, the result value will contain the previously played user information.

```typescript
const deviceId, accountId, password;

const authResult = await connector.authentication(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);

const loginedUserInfoList = authResult.data.loginedUserInfoList;
loginedUserInfoList.forEach((userInfo: AlreadyLoginedUserInfo) => {
    const serviceName = userInfo.serviceName;
    const userId = userInfo.userId;
    const subId = userInfo.subId;
    const userType = userInfo.userType;
    const channelId = userInfo.channelId;
    console.log(`AlreadyLoginedUserInfo(serviceName=${serviceName}, userId=${userId}, subId=${subId}, userType=${userType}, channelId=${channelId})`);
});
```

### Login

Proceed by logging in using the user information received as a result of authentication. In this case, you must log in using the same value as the previous user information, such as userType or channelId. If not, the login may fail.

```typescript
const userType: stirng;
const channelId: string;
const payload: Payload;

const loginResult = await user.login(userType, channelId, payload);
```

If isRelogined is true in the result data when the login succeeds, the relogin is successful. Relogin based on the information in LoginInfo. At this time, it is important to synchronize the changed data on the server with the client during the relogin time. You can process the data required for this synchronization into the Payload of LoginInfo.  

When re-logging in. The IUser.onRelogin() callback is called on the server. At this time, if you define the message required for synchronization and add it to the outPayload parameter, it can be received and processed with the Payload from LoginInfo. 

If isRelogined is false, the previous user information has been removed and logged in over time. In this case, you can perform the procedure to log in first. 