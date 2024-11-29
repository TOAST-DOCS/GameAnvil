## Game > GameAnvil > Unity 심화 개발 가이드 > 커넥터

## 커넥터

GameAnvilConnector는 GameAnvil 서버와의 연결을 관리하는 작업을 담당합니다. 연결(Connect()), 인증(Authentication()) 등 기본 세션 관리 기능 및 채널 목록 등을 제공합니다.

### 서버 연결

Connect() 를 호출하여 서버에 연결합니다.

```c#
public async void Connect()
{
    try
    {
        // connector 는 GameAnvilConnector 인스턴스를 저장한 변수입니다.
        await connector.Connect(host, port);
        // 성공
    }
    catch (Exception e)
    {
        // 실패
    }
}
```

Connect()은 다음과 같은 2개의 매개변수를 가지고 있습니다.

| 타입     | 이름   | 설명               |
|--------|------|------------------|
| String | host | 연결할 서버의 ip 혹은 주소 |
| int    | port | 연결할 서버의 port     |

별도의 리턴값은 없고 성공할 경우 다음 코드를 실행하게 되며, 실패할 경우 예외가 발생하게 됩니다.

### 인증

서버에 연결한 후 Authentication() 를 호출하여 인증 절차를 진행합니다.
Authentication() 메소드를 호출하면 서버에서는 Connection 객체의 onAuthenticate() 콜백이 호출되며, 이 콜백의 처리 결과로 인증의 성공, 실패가 결정됩니다.

```c#
public async void Authenticate()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        ErrorResult<ResultCodeAuth, AuthenticationResult> result = await connector.Authentication("DeviceId", "AccountId", "Password", authenticationPayload);
        if(result.ErrorCode == ResultCodeAuth.AUTH_SUCCESS)
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

Authentication()은 다음과 같은 4개의 매개변수를 가지고 있습니다.

| 타입      | 이름        | 설명                                                                                                              |
|---------|-----------|-----------------------------------------------------------------------------------------------------------------|
| String  | deviceId  | 인증에 사용할 기기 아이디. <br/>서로 다른 기기에서 인증 요청을 할 경우, 이 값을 이용하여 기기에서 요청한 것입음 알 수 있습니다. 사용하지 않을 경우 string.Empty(빈 문자열) 입력 |
| String  | accountId | 인증에 사용할 계정 명.                                                                                                   |
| String  | password  | 인증에 사용할 비밀번호. 사용하지 않는 경우 string.Empty(빈 문자열) 입력                                                                 |
| Payload | payload   | 인증 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)                                                             |

응답으로 ErrorResult<ResultCodeAuth, AuthenticationResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. Authentication이 성공하면 ErrorCode 필드의 값이 ResultCodeAuth.AUTH_SUCCESS 가 되며, 아닌 경우 인증이 실패한 것입니다. Data 필드를 통해 요청 결과 AuthenticationResult 를 얻을 수 있습니다. 이를 통해 인증 결과 정보를 얻을 수 있으며, 서버 구현에 따라서 추가 정보를 얻을 수도
있습니다.

ResultCodeAuth의 상세 내용은 다음과 같습니다.

| 이름                           | 값   | 설명                                         |
|------------------------------|-----|--------------------------------------------|
| PARSE_ERROR                  | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                      | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                 | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패               |
| INVALID_PROTOCOL             | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| AUTH_SUCCESS                 | 0   | 성공                                         |
| AUTH_FAIL_CONTENT            | 201 | 실패. 사용자 코드에서 거부됨                           |
| AUTH_FAIL_INVALID_ACCOUNT_ID | 602 | 실패. 잘못된 계정 아이디                             |                                |

AuthenticationResult의 상세 내용은 다음과 같습니다.

| 타입                           | 이름                | 설명                              |
|------------------------------|-------------------|---------------------------------|
| List<AlreadyLoginedUserInfo> | LoginUserInfoList | 인증 성공 시점에 서버에 남아있는 로그인된 유저정보 목록 |
| String                       | Message           | 인증 메시지                          |
| Payload                      | payload           | 클라이언트에서 필요한 추가 정보               |

### 연결과 인증

연결 또는 인증시 필요한 값을 GameAnvilConnector의 속성으로 저장하여 사용할 수도 있습니다.

```c#
public  void initializeConnector()
{
    connector.Host = "127.0.0.1";
    connector.Port = 18200;
    connector.DeviceId = "DeviceId";
    connector.AccountId = "AccountId";
    connector.Password = "Password";
}

public async void ConnectAndAuthentication1()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        var (resultCode, result) = await connector.ConnectAndAuthentication(authenticationPayload);
        if (resultCode == ResultCodeAuth.AUTH_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}

public async void ConnectAndAuthentication2()
{
    try
    {
        Payload authenticationPayload = new Payload(new Protocol.AuthenticationData());
        await connector.Connect();
        var (resultCode, result) = await connector.Authentication(authenticationPayload);
        if (resultCode == ResultCodeAuth.AUTH_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}
```

### 보안 연결
GameAnvil 서버에 보안 설정을 하였을때는 ConnectSecure() 을 호출하여 보안 연결을 해야합니다. 
```c#
public async void ConnectSecure()
{
    try
    {
        await connector.ConnectSecure(host, port);
        // 성공
    }
    catch (Exception e)
    {
        // 실패
    }
}
```
ConnectSecure()은 다음과 같은 2개의 매개변수를 가지고 있습니다.

| 타입     | 이름   | 설명               |
|--------|------|------------------|
| String | host | 연결할 서버의 ip 혹은 주소 |
| int    | port | 연결할 서버의 port     |

별도의 리턴값은 없고 성공할 경우 다음 코드를 실행하게 되며, 실패할 경우 예외가 발생하게 됩니다.

### 채널 정보

GameAnvil은 설정에서 자유롭게 채널을 구성할 수 있습니다. 이런 채널 구성은 서버와 클라이언트 간에 미리 약속하여 고정된 형태로 사용할 수도 있지만, 상황에 따라 다양하게 변경하여 사용할 수도 있습니다. GameAnvilConnector에서는 이렇게 변경된 채널 정보를 얻어올 수 있도록 몇 가지 메소드를 제공합니다. 

| 이름                       | 설명                                    |
|--------------------------|---------------------------------------|
| GetChannelList()         | 특정 서비스의 채널 아이디 목록 요청                  | 
| GetChannelCountInfo()    | 특정 채널의 카운트 정보(유저와 방 개수) 요청            | 
| GetChannelInfo()         | 특정 채널의 정보(사용자 정의) 요청                  |
| GetAllChannelCountInfo() | 특정 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수) 요청 |
| GetAllChannelInfo()      | 특정 서비스의 모든 채널에 대한 정보(사용자 정의) 요청       |

#### GetChannelList

GetChannelList()는 특정 서비스의 채널 아이디 목록을 요청하여 받아올 수 있습니다.

```c#
public async void ChannelList()
{
    try
    {
        Payload channelInfoPayload = new Payload(new Protocol.ChannelInfoData());
        ErrorResult<ResultCodeChannelList, List<string>> result = await connector.GetChannelList("ServiceName");
        if (result.ErrorCode == ResultCodeChannelList.CHANNEL_LIST_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}
```

GetChannelList()은 다음과 같은 1개의 매개변수를 가지고 있습니다.

| 타입     | 이름          | 설명             |
|--------|-------------|----------------|
| String | ServiceName | 채널 정보를 요청할 서비스 |

응답으로 ErrorResult<ResultCodeChannelList, List<string>>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. GetChannelList가 성공하면 ErrorCode 필드의 값이 ResultCodeChannelList.CHANNEL_LIST_SUCCESS 가 되며, 아닌 경우 요청이 실패한 것입니다. 성공시 Data 필드를 통해 요청 결과인 채널 아이디 목록을 얻을 수 있습니다.

ResultCodeChannelList의 상세 내용은 다음과 같습니다.

| 이름                                | 값    | 설명                                         |
|-----------------------------------|------|--------------------------------------------|
| PARSE_ERROR                       | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                           | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                      | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패               |
| INVALID_PROTOCOL                  | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| CHANNEL_LIST_SUCCESS              | 0    | 성공                                         |
| CHANNEL_LIST_FAIL_NO_CHANNEL_LIST | 1801 | 실패. 채널 목록을 찾을 수 없음                         |                          

#### GetChannelCountInfo

GetChannelCountInfo()는 특정 채널의 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다.

```c#
public async void ChannelCountInfo()
{
    try
    {
        ErrorResult<ResultCodeChannelCountInfo, ChannelCountResult> result = await connector.GetChannelCountInfo("ServiceName", "ChannelId");
        if (result.ErrorCode == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}
```

GetChannelCountInfo()은 다음과 같은 2개의 매개변수를 가지고 있습니다.

| 타입     | 이름          | 설명                 |
|--------|-------------|--------------------|
| String | ServiceName | 채널 정보를 요청할 서비스     |
| String | channelId   | 채널 정보를 요청할 채널의 아이디 |

응답으로 ErrorResult<ResultCodeChannelCountInfo, ChannelCountResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. GetChannelCountInfo가 성공하면 ErrorCode 필드의 값이 ResultCodeChannelCountInfo.CHANNEL_LIST_SUCCESS 가 되며, 아닌 경우 요청이 실패한 것입니다. Data 필드를 통해 요청 결과인 ChannelCountResult 를 얻을 수 있습니다.

ResultCodeChannelCountInfo의 상세 내용은 다음과 같습니다.

| 이름                                         | 값    | 설명                                         |
|--------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                                | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                                    | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                               | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패               |
| INVALID_PROTOCOL                           | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| CHANNEL_COUNT_INFO_SUCCESS                 | 0    | 성공                                         |
| CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1921 | 실패. 채널 정보를 찾을 수 없음                         |
| CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1922 | 실패. 잘못된 서비스 아이디                            |
| CHANNEL_COUNT_INFO_FAIL_INVALID_CHANNEL_ID | 1923 | 실패. 잘못된 채널 아이디                             |
| CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924 | 실패. 채널을 찾을 수 없음                            |

ChannelCountResult 의 상세 내용은 다음과 같습니다.

| 타입     | 이름        | 설명           |
|--------|-----------|--------------|
| string | ChannelId | 채널 아이디 반환.   |
| int    | UserCount | 채널의 유저 수 반환. |
| int    | RoomCount | 채널의 방 수 반환.  |

<br>

#### GetChannelInfo

GetChannelInfo()는 특정 채널의 정보(사용자 정의)를 요청하여 받아올 수 있습니다.

```c#
public async void ChannelInfo()
{
    try
    {
        ErrorResult<ResultCodeChannelInfo, Payload> result = await connector.GetChannelInfo("ServiceName", "ChannenId");
        if (result.ErrorCode == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}
```

GetChannelInfo()은 다음과 같은 2개의 매개변수를 가지고 있습니다.

| 타입     | 이름          | 설명                 |
|--------|-------------|--------------------|
| String | ServiceName | 채널 정보를 요청할 서비스     |
| String | channelId   | 채널 정보를 요청할 채널의 아이디 |

응답으로 ErrorResult<ResultCodeChannelInfo, Payload>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. GetChannelInfo() 가 성공하면 ErrorCode 필드의 값이 ResultCodeChannelInfo.CHANNEL_LIST_SUCCESS 가 되며, 아닌 경우 요청이 실패한 것입니다. 성공시 Data 필드의 Payload 를 사용자가 정의한 채널 정보를 얻을 수도 있습니다.

ResultCodeChannelInfo 상세 내용은 다음과 같습니다.

| 이름                                   | 값    | 설명                                         |
|--------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                          | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                              | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                         | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패               |
| INVALID_PROTOCOL                     | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| CHANNEL_INFO_SUCCESS                 | 0    | 성공                                         |
| CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1921 | 실패. 채널 정보를 찾을 수 없음                         |
| CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1922 | 실패. 잘못된 서비스 아이디                            |
| CHANNEL_INFO_FAIL_INVALID_CHANNEL_ID | 1923 | 실패. 잘못된 채널 아이디                             |
| CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924 | 실패. 채널을 찾을 수 없음                            |

#### GetAllChannelCountInfo

GetAllChannelCountInfo()는 특정 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다.

```c#
public async void AllChannelCountInfo()
{
    try
    {
        ErrorResult<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>> result = await connector.GetAllChannelCountInfo("ServiceName");
        if (result.ErrorCode == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}
```

GetAllChannelCountInfo()은 다음과 같은 1개의 매개변수를 가지고 있습니다.

| 타입     | 이름          | 설명             |
|--------|-------------|----------------|
| String | ServiceName | 채널 정보를 요청할 서비스 |

응답으로 ErrorResult<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. GetAllChannelCountInfo가 성공하면 ErrorCode 필드의 값이 ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS 가 되며, 아닌 경우 요청이 실패한 것입니다. Data 필드를 통해 요청 결과인 Dictionary<
string, ChannelCountResult> 를 얻을 수 있습니다. 이 Dictionary는 채널 아이디를 키로, ChannelCountResult를 값으로 가지고 있습니다.

ResultCodeAllChannelCountInfo의 상세 내용은 다음과 같습니다.

| 이름                                             | 값    | 설명                                         |
|------------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                                    | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                                        | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                                   | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패               |
| INVALID_PROTOCOL                               | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| ALL_CHANNEL_COUNT_INFO_SUCCESS                 | 0    | 성공                                         |
| ALL_CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1931 | 실패. 채널 정보를 찾을 수 없음                         |
| ALL_CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1932 | 실패. 잘못된 서비스 아이디                            |
| ALL_CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1933 | 실패. 채널을 찾을 수 없음                            |

#### GetAllChannelInfo

GetAllChannelInfo()는 특정 서비스의 모든 채널에 대한 정보(사용자 정의)를 요청하여 받아올 수 있습니다.

```c#
public async void AllChannelInfo()
{
    try
    {
        ErrorResult<ResultCodeAllChannelInfo, ChannelInfoResult> result = await connector.GetAllChannelInfo("ServiceName");
        if (result.ErrorCode == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    } catch (Exception e)
    {
        // 예외
    }
}
```

GetAllChannelInfo()은 다음과 같은 1개의 매개변수를 가지고 있습니다.

| 타입     | 이름          | 설명             |
|--------|-------------|----------------|
| String | ServiceName | 채널 정보를 요청할 서비스 |

응답으로 ErrorResult<ResultCodeAllChannelInfo, ChannelInfoResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. GetAllChannelInfo가 성공하면 ErrorCode 필드의 값이 ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS 가 되며, 아닌 경우 요청이 실패한 것입니다. Data 필드를 통해 요청 결과인 ChannelInfoResult 를 얻을 수 있습니다.

ResultCodeAllChannelInfo의 상세 내용은 다음과 같습니다.

| 이름                                       | 값    | 설명                                         |
|------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                              | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                                  | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                             | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패               |
| INVALID_PROTOCOL                         | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| ALL_CHANNEL_INFO_SUCCESS                 | 0    | 성공                                         |
| ALL_CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1911 | 실패. 채널 정보를 찾을 수 없음                         |
| ALL_CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1912 | 실패. 잘못된 서비스 아이디                            |
| ALL_CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1913 | 실패. 채널을 찾을 수 없음                            |

ChannelInfoResult의 channelInfo 필드는 채널 아이디를 키로, 사용자 정의 채널 정보를 담은 Payload를 값으로 가지는 Dictionary<string, Payload> 입니다. 이를 이용해 채널 별 사용자 정의 정보를 얻을 수 있습니다.

### 연결 종료

Disconnect() 메소드를 이용해 서버와의 연결을 종료합니다.

```c#
public async void Disconnect()
{
    try
    {
        await connector.Disconnect();
        // 성공
    } catch (Exception e)
    {
        // 실패
    }
}
```

별도의 리턴값은 없고 성공할 경우 다음 코드를 실행하게 되며, 실패할 경우 예외가 발생하게 됩니다.


