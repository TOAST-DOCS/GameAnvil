## Game > GameAnvil > Unity 심화 개발 가이드 > 재접속

## 재접속

게임 중 다양한 이유로 서버와의 접속이 끊어질 수 있습니다. 접속이 끊어졌을 때 기존의 플레이를 이어서 할 수 있도록 재접속 기능을 지원합니다. 사용하는 API는 일반적인 접속 방법과 동일하지만 결과로 받아오는 정보에 차이가 있습니다.

### 서버 접속

GameAnvilConnector의 Connect 함수를 이용해 서버에 접속합니다. 일반적인 경우와 동일합니다.

```c#
public async void Connect()
{
    try
    {
        await connector.Connect(host, port);
        // 성공
    }
    catch (Exception e)
    {
        // 실패
    }
}
```

### 인증

GameAnvilConnector의 Authenticate 함수를 이용해 인증 절차를 진행합니다. 입력 값은 일반적인 경우와 동일합니다. 다만 인증의 결과로 받아오는 ResultCodeAuth 값 중 loginedUserInfoList에 이전에 플레이하던 유저 정보가 포함되어 오게 됩니다.

```c#
public async void Authenticate()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        ErrorResult<ResultCodeAuth, AuthenticationResult> result = await connector.Authentication("DeviceId", "AccountId", "Password", );
        if(result.ErrorCode == ResultCodeAuth.AUTH_SUCCESS)
        {
            // 성공
            foreach(AlreadyLoginedUserInfo alreadyLoginedUserInfo in result.Data.LoginUserInfoList)
            {
                // 이전에 플레이하던 유저 정보
            }
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

### 로그인

인증 결과로 받은 유저 정보를 이용해 로그인을 진행합니다. 이때 userType이나 channelId 등 이전 유저 정보와 동일한 값을 이용해 로그인을 해야 합니다. 그렇지 않으면 로그인이 실패할 수 있습니다.

```c#
public async void Login(AlreadyLoginedUserInfo userInfo)
{
    try
    {
        Payload loginPayload = new Payload(new Protocol.LoginData());
        GameAnvilUser user = new GameAnvilUser(connector, userInfo.ServiceName, userInfo.UserID);
        ErrorResult<ResultCodeLogin, LoginResult> result = await user.Login(userInfo.UserType, userInfo.ChannelId, loginPayload);
        if(result.ErrorCode == ResultCodeLogin.LOGIN_SUCCESS)
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

로그인 성공 시 인증의 결과로 받아오는 LoginResult의 IsRelogined가 true이면 재접속 로그인에 성공한 것입니다. LoginResult에 포함된 정보들을 바탕으로 재접속 처리를하면 됩니다. 이때 제일 중요한 것이 재접속 시간 동안 서버의 변경된 데이터를 클라이언트에 동기화하는 것입니다. 이 동기화에 필요한 데이터를 LoginResult의 Payload에 담아 처리할 수 있습니다.

재접속 로그인을 하면 서버에서는 User 객체의 onRelogin() 콜백이 호출됩니다. 이때 동기화에 필요한 메시지를 정의하여 outPayload 매개변수에 추가하면 LoginResult의 Payload로 받아 처리할 수 있습니다.

만약 IsRelogined가 false이면 시간이 지나 이전 유저가 서버에서 로그아웃 처리가 된 후 로그인이 된 것입니다. 이 경우에는 처음 로그인하는 절차를 수행하면 됩니다. 
