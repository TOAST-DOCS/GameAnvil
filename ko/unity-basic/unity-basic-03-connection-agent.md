## Game > GameAnvil > Unity 기초 개발 가이드 > ConnectionAgent

## QuickConnect

GameAnvilConnector에서는 QuickConnect 기능을 제공하여 서버에 접속, 인증, 로그인하는 절차를 한 번에 처리할 수 있도록 합니다. 접속, 인증, 로그인하는 절차에 대한 좀 더 자세한 내용은 [Unity 심화 개발 가이드 > ConnectionAgent](../unity-advanced/unity-advanced-02-connection-agent.md) 또는 서버 개발 가이드를 참고해주세요.

### QuickConnect 설정

QuickConnect에 사용되는 설정값이 있습니다. GameAnvilConnector 생성 시 기본값으로 설정되지만 필요하다면 인스펙터 창에서 직접 값을 변경할 수 있습니다.

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/03-connection-agent/01-config.png)

설정의 종류는 다음과 같습니다.

| 이름 | 설명 |
| --- | --- |
| ip | 대상 아이피 주소 |
| port | 대상 포트 번호 |
| accountId | 사용자 계정을 식별 할 수 있는 고유 아이디 |
| deviceId | 사용자 기기 식별용 고유 아이디. 서버 구현에 따라 사용하지 않는 경우 빈 문자열 전달 |
| password | 사용자 계정의 비밀번호. 서버 구현에 따라 사용하지 않는 경우 빈 문자열 전달 |
| userType | 유저의 타입 |
| channelId | 대상 채널의 아이디 |
| serviceName | 대상 서비스의 이름 |

### QuickConnect 사용

QuickConnect()를 통해 서버에 접속, 인증, 로그인합니다. 매개변수로 인증 페이로드, 로그인 페이로드, DelOnQuickConnect 콜백을 넘겨줍니다. 인증 페이로드와 로그인 페이로드는 생략할 수 있습니다.

```c#
/// <summary>
/// GameAnvil 서버에 연결 시도, 연결 성공 시 인증 요청, 인증 성공 시 로그인 요청
/// 연결/인증/로그인 모두 성공 시 QUICK_CONNECT_SUCCESS
/// </summary>
/// <param name="authenticatePayload">인증을 위해 추가로 전달하는 정보</param>
/// <param name="loginPayload">로그인을 위해 추가로 전달하는 정보</param>
/// <param name="onQuickConnect">QuickConnect 요청 결과를 전달 받을 대리자</param>
public void QuickConnect(Payload authenticatePayload, Payload loginPayload, DelOnQuickConnect onQuickConnect){
    if (quickConnect != null)
    {
        CallQuickConnectCallback(onQuickConnect, ResultCodeQuickConnect.QUICK_CONNECT_ALREADY_REQUESTED, null, null);
        return;
    }

    quickConnect = StartCoroutine(QuickConnectCoroutines(authenticatePayload, loginPayload, onQuickConnect, new QuickConnectResult()));
}
```

<br>

DelOnQuickConnect 콜백을 통해서 QuickConnect 결과를 받을 수 있습니다.

```c#
/// <summary>
/// QuickConnect 요청 결과를 전달 받을 대리자
/// </summary>
/// <param name="resultCode">QuickConnect 요청 결과</param>
/// <param name="userAgent">방 생성, 입장 등 유저와 관련된 기능을 사용할 수 있도록 해주는 에이전트</param>
/// <param name="quickConnectResult">서버로부터 받은 인증 관련 추가 정보, 서버로부터 받은 로그인 관련 추가 정보, 연결 요청 결과, 인증 요청 결과, 로그인 요청 결과 묶음</param>
public delegate void DelOnQuickConnect(ResultCodeQuickConnect resultCode, UserAgent userAgent, QuickConnectResult quickConnectResult);
```

## Disconnect

GameAnvilConnector의 Disconnect(), QuickDisconnect() 함수를 이용해서 서버와의 연결을 해제할 수 있습니다. 

QuickDisconnect() 호출 시 로그아웃, 서버 접속 종료, 연결 해제 후 처리할 콜백 호출의 과정을 한 번에 처리할 수 있습니다.

```c#
/// <summary>
/// 로그아웃, GameAnvil 서버와의 연결 해제 요청, 콜백 호출
/// </summary>
/// <param name="onQuickDisconnect">요청 결과를 전달 받을 대리자</param>
public void QuickDisconnect(DelOnQuickDisconnect onQuickDisconnect);
```

Disconnect() 호출 시 서버 접속이 종료됩니다.

```c#
/// <summary>
/// GameAnvil 서버와의 연결 해제 요청
/// </summary>
public void Disconnect();
```

