## Game > GameAnvil > CocosCreator 개발 가이드 > 재접속

## 재접속

게임중 다양한 이유로 서버와의 접속이 끊어질 수 있습니다. 접속이 끊어졌을 때 기존의 플레이를 이어서 할 수 있도록 재접속 기능을 지원합니다. 사용하는 API는 일반적인 접속 방법과 동일하지만 결과로 받아오는 정보에 차이가 있습니다. 

### 서버 접속

ConnectionAgent의 Connect 함수를 이용해 서버에 접속합니다. 일반적인 경우와 동일합니다. 

```typescript
// GameAnvil 서버에 연결
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void;
//  ipAddress : 접속할 서버 주소. 
//  callback : 결과를 전달받아 처리할 콜백
connectionAgent.Connect(ipAddress,(agent: ConnectionAgent, resultCode: ResultCodeConnect) => {
    //  agent : Connect()를 호출한 ConnectionAgent 객체.
	//  resultCode : Connect 결과.
    if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

### 인증

ConnectionAgent의 Authenticate 함수를 이용해 인증절차를 진행합니다. 입력 값은 일반적인 경우와 동일합니다. 다만 인증의 결과로 받아오는 값 중 loginedUserInfoList에 이전에 플레이하던 유저정보가 포함되어 오게 됩니다. 

```typescript
/**
 * GameAnvil 서버에 인증을 요청
 * 
 * 인증에 성공해야 GameAnvil 커넥터의 다른 기능을 사용 가능
 * @param deviceId 사용자 기기를 식별 할 수 있는 고유 아이디. 같은 사용자의 중복 접속을 체크하는데 사용
 * @param accountId 사용자 계정을 식별 할 수 있는 고유 아이디
 * @param password 사용자 계정의 패스워드
 * @param payload 서버로 전달 할 추가 정보
 * @param callback 결과를 전달 받아 처리 할 콜백
 */
connectionAgent.Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    /**
     * Authentication()의 결과
     * @param connection Authentication()을 요청한 커넥션에이전트
     * @param resultCode 인증의 결과 코드
     * @param loginedUserInfoList 서버에 남아있는 로그인 정보 목록
     * @param message 서버로부터 받은 메세지, 인증 실패 이유 등
     * @param payload 서버로 부터 받은 추가 정보
     */
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
		// 성공
        for(let loginedUserInfo of loginedUserInfoList){
            // 플레이 유저정보
        }
    } else {
		// 실패
    }
});
```



### 로그인

인증 결과로 받은 유저정보를 이용해 로그인을 진행합니다. 이때 userType이나 channelId등 이전 유저 정보와 동일한 값을 이용해 로그인을 해야합니다. 그렇지 않으면 로그인이 실패할 수 있습니다. 

```typescript
/**
 * 서버에 로그인
 * @param userType 로그인에 사용할 유저타입
 * @param payload 서버에 전달 할 추가정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param channelId 로그인 할 채널아이디
 * @param callback 로그인 결과를 전달받아 처리 할 콜백
 */
userAgent.Login(userType, payload, channelId, (agent: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo)=>{
    /**
     * Login()의 결과
     * @param user Login()을 요청한 유저에이전트
     * @param resultCode Login() 결과 코드 
     * @param loginInfo Login() 정보
     */
    if (ResultCodeLogin.LOGIN_SUCCESS == resultCode) {
        // 성공
        if (loginInfo.isRelogined) {
            // 재접속
        } else {
            // 처음 접속
        }
    } else {
        // 실패
    }
})
```

로그인 성공시 LoginInfo의 isRelogined가 true이면 재접속 로그인에 성공한것 입니다. LoginInfo에 포함된 정보들을 바탕으로 재접속 처리를 하시면됩니다. 이때 제일 중요한 것이 재접속 시간동안 서버의 변경된 데이터를 클라이언트에 동기화하는것입니다. 이 동기화에 필요한 데이터를 LoginInfo의 Payload에 담아 처리할 수있습니다.  

재접속 로그인을 하게되면 서버에서는 BaseUser.onRelogin() 콜백이 호출되게 됩니다. 이때 동기화에 필요한 메시지를 정의하여 outPayload 파라메터에 추가하면 LoginInfo의 Payload로 받아서 처리할 수 있습니다. 

만약 isRelogined가 false이면 시간이 지나 이전 유저 정보가 제거된 다음 로그인이 된 것입니다. 이경우에는 처음 로그인하는 절차를 수행하면 됩니다. 
