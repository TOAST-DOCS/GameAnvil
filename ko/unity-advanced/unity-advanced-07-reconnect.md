## Game > GameAnvil > Unity 심화 개발 가이드 > 재접속

## 재접속

게임 중 다양한 이유로 서버와의 접속이 끊어질 수 있습니다. 접속이 끊어졌을 때 기존의 플레이를 이어서 할 수 있도록 재접속 기능을 지원합니다. 사용하는 API는 일반적인 접속 방법과 동일하지만 결과로 받아오는 정보에 차이가 있습니다. 

### 서버 접속

ConnectionAgent의 Connect 함수를 이용해 서버에 접속합니다. 일반적인 경우와 동일합니다. 

```c#
/// <summary>
/// GameAnvil 서버에 연결 시도
/// </summary>
/// <param name="ip">대상 아이피 주소</param>
/// <param name="port">대상 포트</param>
/// <param name="onConnect">연결 시도 결과를 전달받을 대리자</param>
connector.GetConnectionAgent().Connect(ip, port, (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
    /// <param name="connectionAgent">Connect()를 요청한 커넥션 에이전트</param>
    /// <param name="result">Connect() 요청 결과 코드</param>
    if (result == ResultCodeConnect.CONNECT_SUCCESS) {
      // 접속 성공
    } 
    else {
      // 접속 실패
    }
});
```

### 인증

ConnectionAgent의 Authenticate 함수를 이용해 인증절차를 진행합니다. 입력 값은 일반적인 경우와 동일합니다. 다만 인증의 결과로 받아오는 값 중 loginedUserInfoList에 이전에 플레이하던 유저 정보가 포함되어 오게 됩니다. 

```c#
/// <summary>
/// GameAnvil 서버에 인증 요청<para></para>
/// 인증 성공 후 커넥터 사용 가능
/// </summary>
/// <param name="deviceId">사용자 기기 식별용 고유 아이디. 서버 구현에 따라 사용하지 않는 경우 빈 문자열 전달</param>
/// <param name="accountId">사용자 계정을 식별 할 수 있는 고유 아이디</param>
/// <param name="passwd">사용자 계정의 비밀번호. 서버 구현에 따라 사용하지 않는 경우 빈 문자열 전달</param>
/// <param name="onAuthentication">요청 결과를 전달받을 대리자</param>
connector.GetConnectionAgent().Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    /// <param name="connectionAgent">Authentication()를 요청한 커넥션 에이전트</param>
    /// <param name="result">Authentication() 요청 결과</param>
    /// <param name="loginedUserInfoList">서버에 남아있는 로그인 정보 목록</param>
    /// <param name="message">서버로부터 받은 메시지</param>
    /// <param name="payload">서버로부터 받은 추가 정보</param>
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
      foreach(ConnectionAgent.LoginedUserInfo loginedUserInfo in loginedUserInfoList)
      {
        // 플레이 유저정보
      }
    } 
});
```

### 로그인

인증 결과로 받은 유저정보를 이용해 로그인을 진행합니다. 이때 userType이나 channelId등 이전 유저 정보와 동일한 값을 이용해 로그인을 해야합니다. 그렇지 않으면 로그인이 실패할 수 있습니다. 

```c#
/// <summary>
/// 유저 에이전트를 찾아 반환
/// </summary>
/// <param name="serviceName">유저에이전트가 사용할 서비스 이름</param>
/// <param name="subId">서비스별 유저에이전트를 식별 할 수 있는 고유 아이디</param>
/// <returns>해당 유저 에이전트, 없으면 널</returns>
UserAgent userAgent = Connector.GetUserAgent(loginedUserInfo.ServiceName, loginedUserInfo.SubId);

/// <summary>
/// 서비스에 로그인
/// </summary>
/// <param name="userType">유저의 타입 </param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="channelId">로그인할 채널의 아이디</param>
/// <param name="onLogin">결과를 받을 대리자</param>
userAgent.Login(loginedUserInfo.UserType, loginedUserInfo.ChannelId, null,
                 (UserAgent agent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => {
    /// <param name="userAgent">Login()을 요청한 유저 에이전트</param>
    /// <param name="result">Login() 요청 결과</param>
    /// <param name="loginInfo">로그인 정보</param>
	if (result == ResultCodeLogin.LOGIN_SUCCESS) {
        if(loginInfo.IsRelogined){
            // 재접속
        }else{
            // 처음 접속
        }
	}
});
```

로그인 성공 시 UserAgent.LoginInfo의 IsRelogined가 true이면 재접속 로그인에 성공한 것입니다. UserAgent.LoginInfo에 포함된 정보들을 바탕으로 재접속 처리를 하시면 됩니다. 이 때 제일 중요한 것이 재접속 시간 동안 서버의 변경된 데이터를 클라이언트에 동기화하는 것입니다. 이 동기화에 필요한 데이터를 UserAgent.LoginInfo의 Payload에 담아 처리할 수있습니다.  

재접속 로그인을 하게 되면 서버에서는 BaseUser.onRelogin() 콜백이 호출되게 됩니다. 이때 동기화에 필요한 메시지를 정의하여 outPayload 매개변수에 추가하면 UserAgent.LoginInfo의 Payload로 받아서 처리할 수 있습니다. 

만약 IsRelogined가 false이면 시간이 지나 이전 유저 정보가 제거된 다음 로그인이 된 것입니다. 이 경우에는 처음 로그인하는 절차를 수행하면 됩니다. 
