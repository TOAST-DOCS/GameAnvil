## Game > GameAnvil > CocosCreator 개발 가이드 > 재접속

### 재접속

게임 도중, 다양한 이유로 서버와의 접속이 끊어질 수 있습니다. 접속이 끊어졌을 때 기존의 플레이를 이어서 할 수 있도록 재접속 기능을 지원합니다. 사용하는 API는 일반적인 접속 방법과 동일하게 사용하실 수 있습니다. 결과를 통해 재접속에 대한 더 상세한 정보를 받습니다.

#### 인증

재접속 후 인증을 진행하게 되면, 결과 값으로 이전에 플레이하던 유저 정보가 포함되어 오게 됩니다.

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

#### 로그인

인증 결과로 받은 유저 정보를 이용해 로그인을 진행합니다. 이 때, userType이나 channelId 등 이전 유저 정보와 동일한 값을 이용해 로그인을 해야 합니다. 그렇지 않으면 로그인이 실패할 수 있습니다.

```typescript
const userType: stirng;
const channelId: string;
const payload: Payload;

const loginResult = await user.login(userType, channelId, payload);
```

로그인 성공 시 결과 데이터에서 isRelogined가 true이면 재접속 로그인에 성공한 것입니다. LoginInfo에 포함된 정보들을 바탕으로 재접속합니다. 이때 제일 중요한 것이 재접속 시간동안 서버의 변경된 데이터를 클라이언트에 동기화하는 것입니다. 이 동기화에 필요한 데이터를 LoginInfo의 Payload에 담아 처리할 수 있습니다.  

재접속 로그인을 하면. 서버에서는 IUser.onRelogin() 콜백이 호출됩니다. 이때 동기화에 필요한 메시지를 정의하여 outPayload 파라미터에 추가하면 LoginInfo의 Payload로 받아서 처리할 수 있습니다. 

만약 isRelogined가 false이면 시간이 지나 이전 유저 정보가 제거된 뒤 로그인된 것입니다. 이 경우에는 처음 로그인하는 절차를 수행하면 됩니다. 