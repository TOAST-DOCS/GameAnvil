## Game > GameAnvil > Unity 기초 개발 가이드 > Simple Login

## 간편 로그인

GameAnvilManager에서 제공하는 간편 로그인 기능은 GameAnvil 서버에 접속, 인증, 로그인하는 절차를 한 번에 처리할 수 있도록 합니다. 접속, 인증, 로그인하는 절차에 대한 좀 더 자세한 내용은 [Unity 심화 개발 가이드 > ConnectionAgent](../unity-advanced/unity-advanced-02-connection) 또는 서버 개발 가이드를 참고하십시오.

### 간편 로그인 설정

간편 로그인을 사용하기 위해서는 GameAnvilManager 설정중 Login 동작시 사용되는 값을 미리 설정해 주어야 합니다. 간편 로그인에서 사용되는 설정의 종류는 다음과 같습니다.

| 구분             | 이름           | 설명                      |    기본값    |
|----------------|--------------|-------------------------|:---------:|
| Connect        | Ip           | 간편 로그인 에서 연결할 서버의 ip    | 127.0.0.1 |
|                | Port         | 간편 로그인 에서 연결할 서버의 port  |   18200   |
| Authentication | AccountId    | 간편 로그인 에서 사용할 사용자 식별 ID |   test    |
|                | DeviceId     | 간편 로그인 에서 사용할 기기 고유값    |   test    |
|                | Password     | 간편 로그인 에서 사용할 패스워드      |   test    |
| Login          | User Type    | 간편 로그인 에서 사용할 유저 타입     |           |
|                | Channel Id   | 간편 로그인 에서 사용할 채널 아이디    |           |
|                | Service Name | 간편 로그인 에서 사용할 서비스 이름    |           |

이 설정들은 Unity 에디터에서 GameAnvilManager의 Inspector 창에서 설정하거나 스크립트 코드에서 설정할 수 있습니다.

```c#
public void Start()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    gameAnvilManager.ip = managerIp;
    gameAnvilManager.port = managerPort;
    gameAnvilManager.accountId = managerAccountId;
    gameAnvilManager.deviceId = managerDeviceId;
    gameAnvilManager.password = managerPassword;
    gameAnvilManager.userType = managerUserType;
    gameAnvilManager.channelId = managerChannelId;
    gameAnvilManager.serviceName = managerServiceName;
}
```

### 간편 로그인 사용

간편 로그인을 통해 서버에 접속, 인증, 로그인합니다. 리턴값 LoginResult를 이용해 결과를 확인할 수 있습니다. 이때 서버와의 연결에 문제가 발생하는 경우 예외가 발생할 수도 있습니다.

```c#
public async void ManagerLogin()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    try
    {
        var result = await gameAnvilManager.Login();
        if (retult.loginResultCode == GameAnvilManager.LoginResultCode.SUCCESS){
            // 성공
        } else {
            // 실패
        }
    }
    catch (Exception e)
    {
        // 서버 연결과 관련된 문제가 발생할 경우 예외 발생
        // eg.) 연결 실패, 연결 끊김 등
    }
}
```

<br>
서버의 구현에 따라서 서버의 인증 과정이나 로그인 과정에서 추가 정보가 필요할 수 있습니다. 이런 경우에는 authenticatePayload 나 loginPayload 로 추가 정보를 전달할 수 있습니다. 인증 과정에서 필요한 정보는 authenticatePayload, 로그인 과정에서 필요한 정보는 loginPayload로 전달합니다.

``` c#
public async void ManagerLogin()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    try
    {   
        var authenticatePayload = new Payload(new Protocol.AuthenticateData());
        var loginPayload = new Payload(new Protocol.LoginData());
        var result = await gameAnvilManager.Login(authenticatePayload, loginPayload);
        if (retult.loginResultCode == GameAnvilManager.LoginResultCode.SUCCESS){
            // 성공
        } else {
            // 실패
        }
    }
    catch (Exception e)
    {
        // 서버 연결과 관련된 문제가 발생할 경우 예외 발생
        // eg.) 연결 실패, 연결 끊김 등
    }
}
```

간편 로그인이 성공하면 LoginResult의 UserController에 로그인 완료된 유저의 GameAnvilUserController 인스턴스가 할당이 됩니다. 이 인스턴스를 이용하여 서버의 유저 객체에 접근할 수 있습니다. 간편 로그인이 실패하면 UserController 에는 null 이 할당됩니다.
<br>

간편 로그인이 실패했을 때 어떤 이유로 실패했는지는 LoginResult의 loginResultCode를 이용해 확인할 수 있습니다. 확인할 수 있는 실패 이유는 다음과 같습니다.

| 이름                                 | 설명                                            |                                                             |
|------------------------------------|-----------------------------------------------|-------------------------------------------------------------|
| START_FAIL_LOGIN_IN_PROGRESS       | 이미 Login 이 진행중.                               |                                                             | 
| START_FAIL_ALREADY_LOGGED_IN       | 이미 Login 이 되어있음.                              |                                                             |
| CONNECT_FAIL                       | 서버 연결 실패.                                     |                                                             |                    
| CONNECT_FAIL_INVALID_IP            | 서버 연결 실패. 잘못된 IP.                             |                                                             |         
| CONNECT_FAIL_INVALID_PORT          | 서버 연결 실패. 잘못된 Port.                           |                                                             |     
| AUTH_FAIL_CONTENT                  | 인증 실패. 사용자 코드에서 인증 거부.                        |                                                             |               
| AUTH_FAIL_INVALID_ACCOUNT_ID       | 인증 실패. 잘못된 AccountId.                         | 빈 문자열 등                                                     |
| AUTH_FAIL_SYSTEM_ERROR             | 인증 실패. 서버의 알수 없는 오류로 실패.                      |                                                             |    
| AUTH_FAIL_TIMEOUT                  | 인증 실패. 요청에 대한 응답이 정해진 시간내에 오지 않음.             |                                                             |   
| AUTH_FAIL_PARSE_ERROR              | 인증 실패. 추가 정보를 위한 메시지 파싱 실패.                   | 서버와 클라이언트의 프로토콜 정의가 다를경우 발생할 수 있다.                          |
| LOGIN_FAIL_CONTENT                 | 로그인 실패. 사용자 코드에서 로그인 거부.                      |                                                             |
| LOGIN_FAIL_NOT_EXIST_NODE          | 로그인 실패. 로그인 대상 노드가 존재하지 않음.                   | 서버 구성시 대상 노드를 포함한 서버를 시작하지 경우 발생할 수 있다.                     |
| LOGIN_FAIL_INVALID_SERVICEID       | 로그인 실패. 로그인 대상 서비스가 존재하지 않음.                  | 요청시 서비스 이름을 잘못 입력하였거나, 서버 설정의 서비스 이름 설정이 잘못된 경우 발생할 수 있다.   |
| LOGIN_FAIL_INVALID_SERVICE_NAME    | 로그인 실패. 로그인 대상 서비스가 존재하지 않음.                  | 요청시 서비스 이름을 잘못 입력하였거나, 서버 설정의 서비스 이름 설정이 잘못된 경우 발생할 수 있다.   |
| LOGIN_FAIL_TIMEOUT_GAME_SERVER     | 로그인 실패. 요청에 대한 응답이 정해진 시간내에 오지 않음.            |                                                             |
| LOGIN_FAIL_INVALID_USERTYPE        | 로그인 실패. 로그인 대상 UserType이 존재하지 않음.             | 요청시 UserType을 잘못 입력하였거나, 서버에 구현되지 않았을 경우 발생할 수 있다.          |
| LOGIN_FAIL_INVALID_USERID          | 로그인 실패. 서버의 알수없는 이유로 UserId 생성 실패.            |                                                             |
| LOGIN_FAIL_INVALID_CHANNEL_ID      | 로그인 실패. 로그인 대상 채널이 존재하지 않음.                   | 요청시 ChannelId를 잘못 입력하였거나, 서버 설정에 ChannelId가 없는 경우 발생할 수 있다. |
| LOGIN_FAIL_INVALID_SUB_ID          | 로그인 실패. 잘못된 SubId.                            |                                                             |
| LOGIN_FAIL_LOGINED_OTHER_SERVICE   | 로그인 실패. 같은 AccountId로 이미 다른 서비스에 로그인 되어있음.    | 서버 구성시 중복 로그인을 허용하지 않을 경우 발생.                               |
| LOGIN_FAIL_LOGINED_OTHER_CHANNEL   | 로그인 실패. 같은 AccountId로 다른 채널에 로그인 되어있음.        |                                                             |
| LOGIN_FAIL_LOGINED_OTHER_USER_TYPE | 로그인 실패. 같은 AccountId, 다른 UserType으로 로그인 되어있음. | 서버 구현시 기존 유저와 새로 로그인한 유저 중 선택할 수 있다.                        |
| LOGIN_FAIL_LOGINED_OTHER_DEVICE    | 로그인 실패. 같은 AccountId로 다른 기기에서 로그인 되어있음.       | 서버 구현시 기존 유저와 새로 로그인한 유저 중 선택할 수 있다.                        |
| LOGIN_FAIL_SYSTEM_ERROR            | 로그인 실패. 서버의 알수 없는 오류로 실패.                     |                                                             |
| LOGIN_FAIL_PARSE_ERROR             | 로그인 실패. 추가 정보를 위한 메시지 파싱 실패.                  |                                                             |
| LOGIN_FAIL_TIMEOUT                 | 로그인 실패. 요청에 대한 응답이 정해진 시간내에 오지 않음.            |                                                             |
| UNKNOWN_ERROR                      | 서버의 알수 없는 오류로 실패.                             |                                                             |

보다 자세한 실패 이유를 확인하고 싶을 때는 LoginResult의 authenticationResult이나 loginResult를 이용해 확인할 수 있습니다.

## Disconnect

GameAnvilManager의 Logout() 함수를 이용해서 서버와의 연결을 해제할 수 있습니다.

Logout() 호출 시 로그아웃, 서버 접속 종료, 연결 해제 후 처리할 콜백 호출의 과정을 한 번에 처리할 수 있습니다.

```c#
public async void ManagerLogout()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    await gameAnvilManager.Logout();
}
```

