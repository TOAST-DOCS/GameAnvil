## Game > GameAnvil > Unity 심화 개발 가이드 > 유저

## 유저

GameAnvilUser는 GameAnvil 서버의 유저와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 방 관리 등 기본 기능을 제공합니다.
GameAnvil 서버는 여러 개의 서비스를 동시에 운영할 수 있으며, 하나의 GameAnvilUser는 하나의 서비스에 로그인하여 서로 독립적으로 동작하게 됩니다. 즉, 여러 개의 GameAnvilUser를 만들어 서로 다른 서비스에 로그인하여 동시에 사용이 가능합니다. SubId를 다르게 한다면 같은 서비스에 여러 개의 GameAnvilUser를 동시에 로그인하여 사용하는 것도 가능합니다.

### 생성

GameAnvilUser를 사용하기 위해서는 먼저 GameAnvilUser 객체를 생성해야합니다.

```c#
public void CreateUser()
{
    try
    {
        user = new GameAnvilUser(connector, serviceName, subId);
        // 성공
    }
    catch(Exception e)
    {
        // 실패
    }
}
```

ServiceName과 SubId로 구분되는 여러 개의 GameAnvilUser를 생성할 수 있습니다.

```c#
public void CreateUsers()
{
    try
    {
        gameUser1 = new GameAnvilUser(connector, "GameService", 1);
        gameUser2 = new GameAnvilUser(connector, "GameService", 1);
        chatUser = new GameAnvilUser(connector, "ChatService", 1);
        // 성공
    } catch (Exception e)
    {
        // 실패
    }
}
```

### 해제

사용을 완료한 GameAnvilUser는 Dispose()를 호출하여 해제해줘야 합니다.

```c#
public void DisposeUser()
{
    try
    {
        user.Dispose();
    } catch (Exception e)
    {
        // 실패
    }
}
```

### 로그인/로그아웃

로그인은 클라이언트가 서버에 접속한 후 GameNode에 자신의 유저 객체를 만드는 과정이라고 정의할 수 있습니다. 로그아웃은 로그인의 반대 개념입니다. 다시 말해, GameNode 상에서 자신의 유저 객체를 제거하는 과정입니다.

#### 로그인

Login()을 호출하여 서비스에 로그인합니다. 로그인 시 어떤 UserType으로 어떤 채널에 로그인할지 입력해 줘야 합니다. 추가 정보가 필요하다면 Payload에 담아 보낼 수 있습니다.

```c#
public async void Login()
{
    try
    {
        Payload loginPayload = new Payload(new Protocol.LoginData());
        ErrorResult<ResultCodeLogin, LoginResult> result = await user.Login("UserType", "ChannelId", loginPayload);
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

Login()은 다음과 같은 4개의 매개변수를 가지고 있습니다.

| 타입      | 이름             | 설명                                                   |
|---------|----------------|------------------------------------------------------|
| String  | userType       | 로그인하여 생성할 유저의 타입.                                    |
| String  | channelId      | 로그인 할 채널의 아이디.                                       |
| Payload | requestPayload | 로그인 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null) |

응답으로 ErrorResult<ResultCodeLogin, LoginResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. Login이 성공하면 ErrorCode 필드의 값이 ResultCodeLogin.LOGIN_SUCCESS 가 되며, 아닌 경우 로그인이 실패한 것입니다. Data 필드를 통해 요청 결과 LoginResult 를 얻을 수 있습니다. 이를 통해 로그인된 유저 정보를 얻을 수 있으며, 서버 구현에 따라서 추가 정보를 얻을 수도 있습니다.

ResultCodeLogin의 상세 내용은 다음과 같습니다.

| 이름                                 | 값   | 설명                                         |
|------------------------------------|-----|--------------------------------------------|
| PARSE_ERROR                        | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                            | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                       | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL                   | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| JOIN_ROOM_SUCCESS                  | 0   | 성공.                                        |
| JOIN_ROOM_FAIL_CONTENT             | 701 | 실패. 사용자 코드에서 거부됨.                          |
| JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702 | 실패. 입장 요청한 방이 존재하지 않음.                     |
| JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM | 703 | 실패. 이미 방에 들어가 있음.                          |
| JOIN_ROOM_FAIL_ALREADY_FULL        | 704 | 실패. 입장 요청한 방이 꽉 차있음.                       |
| JOIN_ROOM_FAIL_ROOM_MATCH          | 705 | 실패. 룸 매치메이킹에서 문제가 발생함.                     |

LoginResult의 상세 내용은 다음과 같습니다.

| 타입      | 이름           | 설명              |
|---------|--------------|-----------------|
| int     | UserId       | 로그인한 유저의 아이디    |
| string  | UserType     | 로그인한 유저의 타입     |
| string  | ServiceName  | 로그인한 서비스의 이름    |
| string  | ChannelId    | 로그인한 채널의 아이디    |
| Payload | Payload      | 서버로부터 받은 추가정보   |
| bool    | IsRelogined  | 재로그인 여부         |
| bool    | IsJoinedRoom | 방에 속해있는지 여부     |
| int     | RoomId       | 유저가 속한 방의 아이디   |
| string  | RoomName     | 유저가 속한 방의 이름    |
| Payload | RoomPayload  | 유저가 속한 방의 추가정보  |
| bool    | IsMatching   | 매칭을 요청한 상태인지 여부 |

### 로그아웃

Logout()을 호출하여 서비스에서 로그아웃합니다.

```c#
public async void Logout()
{
    try
    {
        Payload logoutPayload = new Payload(new Protocol.LogoutData());
        ErrorResult<ResultCodeLogout, LogoutResult> result = await user.Logout(logoutPayload);
        if (result.ErrorCode == ResultCodeLogout.LOGOUT_SUCCESS)
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
Logout()은 다음과 같은 1개의 매개변수를 가지고 있습니다.

| 타입       | 이름             | 설명                                                      |
|----------|----------------|---------------------------------------------------------|
| Payload? | payload | 로그아웃 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null) |

응답으로 ErrorResult<ResultCodeLogout, LogoutResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. Logout이 성공하면 ErrorCode 필드의 값이 ResultCodeLogout.LOGOUT_SUCCESS 가 되며, 아닌 경우 로그아웃이 실패한 것입니다. Data 필드를 통해 요청 결과 LogoutResult 를 얻을 수 있습니다. 서버 구현에 따라서 LogoutResult의 Payload 필드를 통해 추가 정보를 얻을 수도 있습니다.

ResultCodeLogout의 상세 내용은 다음과 같습니다.

| 이름                  | 값   | 설명                                         |
|---------------------|-----|--------------------------------------------|
| PARSE_ERROR         | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT             | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR        | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL    | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| LOGOUT_SUCCESS      | 0   | 성공.                                        |
| LOGOUT_FAIL_CONTENT | 401 | 실패. 사용자 코드에서 거부됨.                          |

#### 강제 로그아웃 알림
Logout()을 호출하지 않더라도 서버에서 유저를 강제로 로그아웃을 시킬 수 있습니다. 이 경우 OnForceLogout을 통해 이에 대한 알림을 받을 수 있습니다.
```c#
public void AddOnLogout()
{
    user.OnForceLogout += (GameAnvilUser, Payload) =>
    {
        // 강제 로그아웃 알림
    };
}
```
서버의 구현에 따라 매개변수 Payload payload를 통해 추가 정보를 얻을 수도 있습니다.

### 방 생성, 입장, 퇴장

2명 이상의 유저는 방을 통해 동기화된 메시지 흐름을 만들 수 있습니다. 즉, 유저들의 요청은 방 안에서 모두 순서가 보장됩니다. 물론 1명의 유저를 위한 방 생성도 콘텐츠에 따라서 의미를 가질 수도 있습니다. 방을 어떻게 사용할지는 어디까지나 엔진 사용자의 몫입니다.

#### 방 생성

CreateRoom()을 호출하여 방을 생성하고 그 방으로 입장합니다.

```c#
public async void CreateRoom()
{
    try
    {
        Payload createRoomPayload = new Payload(new Protocol.CreateRoomData());
        ErrorResult<ResultCodeCreateRoom, CreatedRoomResult> result = await user.CreateRoom("RoomName", "RoomType", "MatchingGroup", createRoomPayload);
        if (result.ErrorCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
        {
            // 성공
        } else
        {
            // 실패
        }
    }
    catch(Exception e)
    {
        // 예외
    }
}
```

CreateRoom()은 다음과 같은 4개의 매개변수를 가지고 있습니다.

| 타입      | 이름            | 설명                                                    |
|---------|---------------|-------------------------------------------------------|
| String  | roomName      | 생성할 방의 이름. 사용하지 않는 경우 string.Empty(빈 문자열) 입력          |
| String  | roomType      | 생성할 방의 타입. 서버에 등록된 방 타입 입력                            |
| String  | matchingGroup | 매칭 시 사용할 매칭 그룹 이름. 사용하지 않는 경우 string.Empty(빈 문자열) 입력  |
| Payload | payload       | 방 생성 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null) |

응답으로 ErrorResult<ResultCodeCreateRoom, CreatedRoomResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. CreateRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeCreateRoom.CREATE_ROOM_SUCCESS 가 되며, 아닌 경우 방 생성이 실패한 것입니다. Data 필드를 통해 요청 결과 CreatedRoomResult 를 얻을 수 있습니다. 이를 통해 생성된 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가
정보를 얻을 수도 있습니다.

ResultCodeCreateRoom의 상세 내용은 다음과 같습니다.

| 이름                                   | 값   | 설명                                         |
|--------------------------------------|-----|--------------------------------------------|
| PARSE_ERROR                          | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                              | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                         | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패               |
| INVALID_PROTOCOL                     | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| CREATE_ROOM_SUCCESS                  | 0   | 성공                                         |
| CREATE_ROOM_FAIL_CONTENT             | 601 | 실패. 사용자 코드에서 거부됨                           |
| CREATE_ROOM_FAIL_ALREADY_JOINED_ROOM | 602 | 실패. 이미 방에 들어가 있음                           |
| CREATE_ROOM_FAIL_CREATE_ROOM_ID      | 603 | 실패. 방 아이디 생성이 실패하였을 경우                     |
| CREATE_ROOM_FAIL_CREATE_ROOM         | 604 | 실패. 방 생성 실패                                |

CreatedRoomResult의 상세 내용은 다음과 같습니다.

| 타입      | 이름       | 설명                |
|---------|----------|-------------------|
| int     | RoomId   | 생성한 방의 아이디        |
| String? | RoomName | 생성한 방의 이름         |
| Payload | payload  | 클라이언트에서 필요한 추가 정보 |

#### 방 입장

JoinRoom()을 호출하여 이미 생성된 방에 입장합니다.

``` c#
public async void JoinRoom()
{
    try
    {
        Payload joinRoomPayload = new Payload(new Protocol.JoinRoomData());
        ErrorResult<ResultCodeJoinRoom, JoinRoomResult> result = await user.JoinRoom("RoomType", roomId, "MatchingUserCategory", joinRoomPayload);
        if(result.ErrorCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS)
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

JoinRoom()은 다음과 같은 4개의 매개변수를 가지고 있습니다.

| 타입      | 이름                   | 설명                                                                                                                                                                                           |
|---------|----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String  | roomType             | 입장할 방의 타입.                                                                                                                                                                                   |
| int     | roomId               | 입장할 방의 아이디.                                                                                                                                                                                  |
| String  | matchingUserCategory | 입장할 방에서 사용할 matchingUserCategory. 사용하지 않는 경우 string.Empty(빈 문자열) 입력 <br/>각 방에서는 방에 속한 유저를 카테고리로 나누고, 각 카테고리별로 인원수 제한을 적용할 수 있다. 지정한 matchingUserCategory의 현재 인원이 최대인 경우 JoinRoom 이 실패할 수 있다. |
| Payload | payload              | 방 입장 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)                                                                                                                                        |

응답으로 ErrorResult<ResultCodeJoinRoom, JoinRoomResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. JoinRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeJoinRoom.JOIN_ROOM_SUCCESS 가 되며, 아닌 경우 방 입장이 실패한 것입니다. Data 필드를 통해 요청 결과 JoinRoomResult 를 얻을 수 있습니다. 이를 통해 입장한 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가 정보를 얻을 수도 있습니다.

ResultCodeJoinRoom의 상세 내용은 다음과 같습니다.

| 이름                                 | 값   | 설명                                         |
|------------------------------------|-----|--------------------------------------------|
| PARSE_ERROR                        | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                            | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                       | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL                   | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| JOIN_ROOM_SUCCESS                  | 0   | 성공.                                        |
| JOIN_ROOM_FAIL_CONTENT             | 701 | 실패. 사용자 코드에서 거부됨.                          |
| JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702 | 실패. 입장 요청한 방이 존재하지 않음.                     |
| JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM | 703 | 실패. 이미 방에 들어가 있음.                          |
| JOIN_ROOM_FAIL_ALREADY_FULL        | 704 | 실패. 입장 요청한 방이 꽉 차있음.                       |
| JOIN_ROOM_FAIL_ROOM_MATCH          | 705 | 실패. 룸 매치메이킹에서 문제가 발생함.                     |

JoinRoomResult의 상세 내용은 다음과 같습니다.

| 타입      | 이름       | 설명                 |
|---------|----------|--------------------|
| int     | RoomId   | 입장한 방의 아이디.        |
| String? | RoomName | 입장한 방의 이름.         |
| Payload | payload  | 클라이언트에서 필요한 추가 정보. |

#### 방 퇴장

LeaveRoom()을 호출하여 입장한 방에서 퇴장할 수 있습니다.

``` c#
public async void LeaveRoom()
{
    try
    {
        Payload leaveRoomPayload = new Payload(new Protocol.LeaveRoomData());
        ErrorResult<ResultCodeLeaveRoom, Payload> result = await user.LeaveRoom(leaveRoomPayload);
        if (result.ErrorCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
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

LeaveRoom()은 다음과 같은 1개의 매개변수를 가지고 있습니다.

| 타입      | 이름      | 설명                                                    |
|---------|---------|-------------------------------------------------------|
| Payload | payload | 방 퇴장 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null) |

응답으로 ErrorResult<ResultCodeLeaveRoom, Payload>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. LeaveRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS 가 되며, 아닌 경우 방 퇴장이 실패한 것입니다. 서버 구현에 따라서 Data 필드의 Payload 를 통해 추가 정보를 얻을 수도 있습니다.

ResultCodeLeaveRoom의 상세 내용은 다음과 같습니다.

| 이름                      | 값   | 설명                                         |
|-------------------------|-----|--------------------------------------------|
| PARSE_ERROR             | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                 | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR            | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL        | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| LEAVE_ROOM_SUCCESS      | 0   | 성공.                                        |
| LEAVE_ROOM_FAIL_CONTENT | 801 | 실패. 사용자 코드에서 거부됨.                          |

#### 방 강제 퇴장 알림
LeaveRoom()을 호출하지 않더라도 서버에서 강제로 방에서 퇴장을 시킬 수 있습니다. 이 경우 OnForceLeaveRoom를 통해 이에 대한 알림을 받을 수 있습니다.
```c#
public void AddOnLeaveRoom()
{
    user.OnForceLeaveRoom += (GameAnvilUser gameAnvilUser, int roomId, Payload payload) =>
    {
        // 방 강제 퇴장 알림
    };
}
```
매개변수 int roomId를 통해 어떤 방에서 강제로 퇴장 당했는지 알수 있으며, 서버의 구현에 따라 매개변수 Payload payload를 통해 추가 정보를 얻을 수도 있습니다. 

#### 지정한 이름의 방에 입장

NamedRoom()을 호출하여 지정한 이름의 방에 입장할 수 있습니다. 지정한 이름의 방이 없을 경우에는 방을 생성한 후 그 방으로 입장합니다.

```c#
public async void NamedRoom()
{
    try
    {
        bool isParty = false;
        Payload namedRoomPayload = new Payload(new Protocol.NamedRoomData());
        ErrorResult<ResultCodeNamedRoom, NamedRoomResult> result = await user.NamedRoom("RoomName", "RoomType", isParty, namedRoomPayload);
        if (result.ErrorCode == ResultCodeNamedRoom.NAMED_ROOM_SUCCESS)
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

NamedRoom()은 다음과 같은 4개의 매개변수를 가지고 있습니다.

| 타입      | 이름       | 설명                                                                                       |
|---------|----------|------------------------------------------------------------------------------------------|
| String  | roomType | 입장 또는 생성 할 방의 타입                                                                         |
| String  | roomName | 입장 또는 생성 할 방의 이름                                                                         |
| bool    | isParty  | 파티 매치메이킹을 위한 방 여부.<br/>같은 파티로 묶인 유저들이 파티 매치메치킹이 완료될 때 까지 함께 대기하기 위한 방을 만들 경우 true로 입력한다. |
| Payload | payload  | 입장 또는 생성 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)                                |

응답으로 ErrorResult<ResultCodeNamedRoom, NamedRoomResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. NamedRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeNamedRoom.NAMED_ROOM_SUCCESS 가 되며, 아닌 경우 입장 또는 생성이 실패한 것입니다. Data 필드를 통해 요청 결과 NamedRoomResult 를 얻을 수 있습니다. 이를 통해 입장 또는 생성한 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가
정보를 얻을 수도 있습니다.

ResultCodeNamedRoom의 상세 내용은 다음과 같습니다.

| 이름                                  | 값   | 설명                                                               |
|-------------------------------------|-----|------------------------------------------------------------------|
| PARSE_ERROR                         | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.                         |
| TIMEOUT                             | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.                                |
| SYSTEM_ERROR                        | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.                                    |
| INVALID_PROTOCOL                    | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨.                       |
| NAMED_ROOM_SUCCESS                  | 0   | 성공.                                                              |
| NAMED_ROOM_FAIL_CONTENT             | 701 | 실패. 사용자 코드에서 거부됨.                                                |
| NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702 | 실패. 방이 존재하지 않음.<br/>방 입장 처리 중, 방에 있는 모든 유저가 방에서 나가는 경우 발생할 수 있다. |
| NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM | 703 | 실패. 이미 방에 들어가 있음                                                 |
| NAMED_ROOM_FAIL_INVALID_ROOM_NAME   | 704 | 실패. 잘못된 방 이름을 요청함.                                               |
| NAMED_ROOM_FAIL_CREATE_ROOM         | 705 | 실패. 방 생성에 실패함.                                                   |

NamedRoomResult의 상세 내용은 다음과 같습니다.

| 타입      | 이름       | 설명                 |
|---------|----------|--------------------|
| bool    | Created  | 새로운 방이 생성되었는지 여부   |
| int     | RoomId   | 입장한 방의 아이디         |
| String? | RoomName | 입장한 방의 이름          |
| Payload | payload  | 클라이언트에서 필요한 추가 정보. |

### 매치메이킹

GameAnvil은 두 가지 매치메이킹을 제공합니다. 하나는 방 단위의 매칭을 수행하는 룸 매치메이킹이고, 다른 하나는 유저 단위의 매칭을 수행하는 유저 매치메이킹입니다.

#### 룸 매치메이킹

룸 매치메이킹은 조건에 맞는 방으로 유저를 입장시켜 주는 방식입니다. 룸 매치메이킹 요청 시 조건에 맞는 방이 있으면 해당 방으로 바로 입장시켜 주고 조건에 맞는 방이 없다면 새로운 방을 생성하여 입장시켜 줍니다.

MatchRoom()을 호출하여 룸 매치메이킹을 요청할 수 있습니다.

```c#
public async void MatchRoom()
{
    try
    {
        var matchRoomPayload = new Payload(new Protocol.MatchRoomData());
        ErrorResult<ResultCodeMatchRoom, MatchResult> result = await user.MatchRoom(true, true, "RoomType", "MatchingGroup", "MatchingUserCategory", matchRoomPayload);
        if (result.ErrorCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
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

MatchRoom()은 다음과 같은 7개의 매개변수를 가지고 있습니다.

| 타입      | 이름                        | 설명                                                                                                                              |
|---------|---------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| bool    | isCreateRoomIfNotJoinRoom | 조건에 맞는 방을 찾지 못했을 때, 방을 생성하여 입장할지 여부. <br/>true : 방이 없으면 생성한다. <br/>false : 방이 없으면 실패 처리한다.                                      | 
| bool    | isMoveRoomIfJoinedRoom    | 이미 방에 입장한 상태인 경우, 다른 방으로 이동할지 여부. <br/>true : 방에 입장된 상태이면 방을 옮긴다. <br/>false : 방에 입장된 상태로 매칭을 요청한다면 실패 처리한다.                    |
| string  | roomType                  | 방 타입. 같은 타입의 방을 찾는다.                                                                                                            |
| string  | matchingGroup             | 매칭 그룹. 같은 그룹으로 생성된 방을 찾는다.                                                                                                      |
| string  | matchingUserCategory      | 매칭 된 방에서 사용항 유저 카테고리.<br/>각 방에서는 방에 속한 유저를 카테고리로 나누고, 각 카테고리별로 인원수 제한을 적용할 수 있다.<br/>지정한 matchingUserCategory의 현재 인원이 최대가 아닌 방을 찾는다. |
| Payload | payload                   | 매치메이킹 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)                                                                          |
| Payload | leaveRoomPayload          | 다른 방으로 이동하는 경우, 방을 나갈때 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)                                                             |

응답으로 ErrorResult<ResultCodeMatchRoom, MatchResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. MatchRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeMatchRoom.MATCH_ROOM_SUCCESS 가 되며, 아닌 경우 룸 매치메이킹이 실패한 것입니다. Data 필드를 통해 요청 결과 MatchResult 를 얻을 수 있습니다. 이를 통해 매칭된 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가정보를 얻을
수도 있습니다.

ResultCodeMatchRoom의 상세 내용은 다음과 같습니다.

| 이름                                             | 값   | 설명                                                                                     |
|------------------------------------------------|-----|----------------------------------------------------------------------------------------|
| PARSE_ERROR                                    | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.                                               |
| TIMEOUT                                        | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.                                                      |
| SYSTEM_ERROR                                   | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패                                                           |
| INVALID_PROTOCOL                               | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨.                                             |
| NAMED_ROOM_SUCCESS                             | 0   | 성공                                                                                     |
| NAMED_ROOM_FAIL_CONTENT                        | 701 | 실패. 사용자 코드에서 거부됨                                                                       |
| NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST            | 702 | 실패. 방 입장 처리 중, 방이 사라짐.                                                                 |
| NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM            | 703 | 실패. 이미 방에 들어가 있음                                                                       |
| NAMED_ROOM_FAIL_INVALID_ROOM_NAME              | 704 | 실패. 잘못된 방 이름을 요청함.                                                                     |
| NAMED_ROOM_FAIL_CREATE_ROOM                    | 705 | 실패. 방 생성에 실패함.                                                                         |
| MATCH_ROOM_SUCCESS                             | 0   | 성공                                                                                     |
| MATCH_ROOM_FAIL_CONTENT                        | 901 | 실패. 사용자 코드에서 거부됨.                                                                      |
| MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST            | 902 | 실패. 방이 존재하지 않음.                                                                        |
| MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM            | 903 | 실패. 이미 방에 들어가 있음.                                                                      |
| MATCH_ROOM_FAIL_LEAVE_ROOM                     | 904 | 실패. 다른 방으로 이동할 때, 기존방에서 나가기가 실패한 경우.                                                   |
| MATCH_ROOM_FAIL_IN_PROGRESS                    | 905 | 실패. 이미 매치 매이킹이 진행중인 경우.                                                                |
| MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST    | 906 | 실패. 조건에 맞는 방을 찾아 방에 참가 시키는 도중, 방이 사라짐<br/>방 입장 처리 중, 방에 있는 모든 유저가 방에서 나가는 경우 발생할 수 있다. |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID          | 907 | 실패. 방 아이디 생성이 실패하였을 경우.                                                                |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM             | 908 | 실패. 방 생성이 실패하였을 경우.                                                                    |
| MATCH_ROOM_FAIL_INVALID_ROOM_ID                | 909 | 실패. 잘못된 룸아이디가 사용되었을 경우.                                                                |
| MATCH_ROOM_FAIL_INVALID_NODE_ID                | 910 | 실패. 잘못된 노드아이디가 사용되었을 경우.                                                               |
| MATCH_ROOM_FAIL_INVALID_USER_ID                | 911 | 실패. 잘못된 유저아이디가 사용되었을 경우.                                                               |
| MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND         | 912 | 실패. 매칭을 진행하였으나, 방을 찾지 못한 경우.                                                           |
| MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY | 913 | 실패. 잘못된 매칭 유저 카테고리를 사용하였을 경우.                                                          |
| MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY   | 914 | 실패. 매칭룸에서 유저 카테고리 사이즈가 0일 경우.                                                          |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL      | 915 | 실패. 매칭 신청서가 NULL 일 경우.                                                                 |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL      | 916 | 실패. 매칭 정보가 NULL 일 경우.                                                                  |

MatchResult의 상세 내용은 다음과 같습니다.

| 타입       | 이름       | 설명                 |
|----------|----------|--------------------|
| bool     | IsCancel | 요청이 취소되었는지 여부.     |
| int      | RoomId   | 입장한 방의 아이디.        |
| bool     | Created  | 새로운 방이 생성되었는지 여부.  |
| String   | RoomName | 입장한 방의 이름.         |
| Payload? | payload  | 클라이언트에서 필요한 추가 정보. |

#### 유저 매치메이킹

유저 매치메이킹은 유저 풀을 만들고 그 안에서 조건에 맞는 유저들을 찾아 새로 생성한 방으로 입장시켜 주는 방식입니다. 유저 풀에 조건에 맞는 유저의 수가 모자랄 경우 매치메이킹이 완료될 때까지 시간이 걸릴 수 있고, 시간 내에 매치메이킹이 완료되지 않으면 시간 초과되어 매칭이 실패할 수 있습니다.

MatchUserStart()를 호출하여 유저 매치메이킹을 시작할 수 있습니다. 이 요청의 결과가 성공이라고 해도 유저 매치메이킹이 완료된 것은 아닙니다. 단시 시작 요청이 성공한 것이며, 매치메이킹의 결과는 별도의 콜백을 통해 전달됩니다.

```c#
public async void MatchUserStart()
{
    user.OnMatchUserDone +=(GameAnvilUser user, ResultCodeMatchUserDone resultCode, MatchResult matchResult) => {
        // 매칭 성공
    };
    user.onMatchUserTimeOut +=(GameAnvilUser user, ResultCodeMatchUserTimeOut resultCode) =>
    {
        // 매칭 실패
    };
    try
    {
        Payload matchUserPayload = new Payload(new Protocol.MatchUserData());
        ErrorResult<ResultCodeMatchUserStart, Payload> result = await user.MatchUserStart("RoomType", "MatchingGroup", matchUserPayload);
        if (result.ErrorCode == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
        {
            // 요청 성공
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

MatchUserStart()는 다음과 같은 3개의 매개변수를 가지고 있습니다.

| 타입      | 이름            | 설명                                                                     |
|---------|---------------|------------------------------------------------------------------------|
| String  | roomType      | 생성할 방의 타입.                                                             |
| String  | matchingGroup | 매칭 그룹. 같은 그룹의 유저 풀에서 조건에 맞는 유저를 찾는다.  |
| Payload | payload       | 유저 매치메이킹 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)              |

응답으로 ErrorResult<ResultCodeMatchUserStart, Payload>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. MatchUserStart이 성공하면 ErrorCode 필드의 값이 ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS 가 되며, 아닌 경우 요청이 실패한 것입니다. 서버 구현에 따라서 Data 필드의 Payload 를 통해 추가 정보를 얻을 수도 있습니다.

ResultCodeMatchUserStart의 상세 내용은 다음과 같습니다.

| 이름                                        | 값    | 설명                                         |
|-------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                               | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                                   | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                              | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL                          | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| MATCH_USER_START_SUCCESS                  | 0    | 성공.                                        |
| MATCH_USER_START_FAIL_CONTENT             | 1101 | 실패. 사용자 코드에서 거부됨.                          |
| MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM | 1102 | 실패. 이미 방에 들어가 있음.                          |

<br>

유저 매치메이킹이 성공한 경우 onMatchUserDone 을 통해 알림을 받을 수 있습니다. 매개변수 ResultCodeMatchUserDone resultCode 를 통해 결과 코드를 알수 있으며, 매개변수 MatchResult matchResult를 통해 매칭된 방의 정보를 얻을 수 있습니다. 서버 구현에 따라서 MatchResult의 payload를 통해 추가정보를 얻을 수도 있습니다.
시간 안에 매칭이 성공하지 못한 경우 onMatchUserTimeout 을 통해 알림을 받을 수 있습니다.

ResultCodeMatchUserDone의 상세 내용은 다음과 같습니다.

| 이름                                       | 값    | 설명                                                              |
|------------------------------------------|------|-----------------------------------------------------------------|
| PARSE_ERROR                              | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.                        |
| TIMEOUT                                  | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.                               |
| SYSTEM_ERROR                             | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.                                   |
| INVALID_PROTOCOL                         | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨.                      |
| MATCH_USER_DONE_SUCCESS                  | 0    | 성공.                                                             |
| MATCH_USER_DONE_FAIL_CONTENT             | 1501 | 실패. 사용자 코드에서 거부됨.                                               |
| MATCH_USER_DONE_FAIL_ROOM_DOES_NOT_EXIST | 1502 | 실패. 조건에 맞는 방을 찾아 방에 참가 시키는 도중, 방이 사라짐.                          |
| MATCH_USER_DONE_FAIL_TRANSFER            | 1503 | 실패. 조건에 맞는 방을 찾아 방에 참가 시키는 도중, 방에 참가하기 위해 transfer 하는 과정에서 실패함. |
| MATCH_USER_DONE_FAIL_CREATE_ROOM         | 1504 | 실패. 방 생성 실패.                                                    |

<br>

MatchUserCancel()을 호출하여 진행중인 유저 매치메이킹을 취소할 수 있습니다.

```c#
public async void MatchUserCancel()
{
    try
    {
        ResultCodeMatchUserCancel result = await user.MatchUserCancel("RoomType");
        if (result == ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS)
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

응답으로 ResultCodeMatchUserCancel를 리턴하며, 취소가 성공하면 ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS 가 되며, 아닌 경우 취소가 실패한 것입니다. 유저 매치메이킹이 진행중이 아닌 경우, 이미 유저 매치메이킹이 성공했거나 Timeout이 발생한 경우 요청이 실패할 수 있습니다.

ResultCodeMatchUserCancel의 상세 내용은 다음과 같습니다.

| 이름                                         | 값    | 설명                                         |
|--------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                                | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                                    | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                               | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL                           | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| MATCH_USER_CANCEL_SUCCESS                  | 0    | 성공.                                        |
| MATCH_USER_CANCEL_FAIL                     | 1201 | 실패. 사용자 코드에서 거부됨.                          |
| MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM | 1202 | 실패. 이미 방에 들어가 있음.                          |
| MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS     | 1203 | 실패. 유저 매치메이킹이 진행중이 아님.                     |

#### 파티 매치메이킹

파티 매치메이킹은 유저 매치메이킹의 특수한 형태로, 2명 이상의 유저가 한 파티로 묶여 유저 풀에 등록되고, 조건이 맞는 다른 유저들을 찾아 새로 생성한 방으로 함께 입장시켜 주는 방식입니다. 파티로 묶은 유저들은 항상 같은 방으로 입장합니다. 파티 외에 같이 매칭된 유저들은 서버의 매치 메이커 구현에 따라 또 다른 파티일 수도 있고, 개인일 수도 있습니다.

파티 매치메이킹을 하기 위해서는 먼저 NamedRoom()을 호출해야 합니다. NamedRoom() 호출 시 isParty 매개변수를 true로 넘겨주면, 해당 NamedRoom이 파티의 역할을 합니다. NamedRoom에 파티 유저를 모두 모으고 나서 파티 매치메이킹을 시작합니다.

```c#
public async void PartyRoom()
{
    try
    {
        bool isParty = true;
        Payload partyRoomPayload = new Payload(new Protocol.PartyRoomData());
        ErrorResult<ResultCodeNamedRoom, NamedRoomResult> result = await user.NamedRoom("RoomName", "RoomType", isParty, partyRoomPayload);
        if (result.ErrorCode == ResultCodeNamedRoom.NAMED_ROOM_SUCCESS)
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

<br>

MatchPartyStart()를 호출하여 파티 매치메이킹을 시작할 수 있습니다. 이 요청의 결과가 성공이라고 해도 파티 매치메이킹이 완료된 것은 아닙니다. 단시 시작 요청이 성공한 것이며, 매치메이킹의 결과는 별도의 콜백을 통해 전달됩니다.

```c#
public async void MatchPartyStart()
{
    try
    {
        Payload partyRoomPayload = new Payload(new Protocol.PartyRoomData());
        ErrorResult<ResultCodeMatchPartyStart, Payload> result = await user.MatchPartyStart("RoomType", "MatchingGroup", partyRoomPayload);
        if (result.ErrorCode == ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS)
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

MatchPartyStart()은 다음과 같은 3개의 매개변수를 가지고 있습니다.

| 타입      | 이름            | 설명                                                                     |
|---------|---------------|------------------------------------------------------------------------|
| String  | roomType      | 생성할 방의 타입.                                                             |
| String  | matchingGroup | 매칭 그룹. 같은 그룹의 유저 풀에서 조건에 맞는 유저를 찾는다. 사용하지 않는 경우 string.Empty(빈 문자열) 입력 |
| Payload | payload       | 파티 매치메이킹 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)              |

응답으로 ErrorResult<ResultCodeMatchPartyStart, Payload>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. MatchPartyStart이 성공하면 ErrorCode 필드의 값이 ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS 가 되며, 아닌 경우 파티 매치메이킹이 실패한 것입니다. 서버 구현에 따라서 Data 필드의 Payload 를 통해 추가 정보를 얻을 수도 있습니다.

ResultCodeMatchPartyStart의 상세 내용은 다음과 같습니다.

| 이름                                       | 값    | 설명                                         |
|------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                              | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                                  | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                             | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL                         | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| MATCH_PARTY_START_SUCCESS                | 0    | 성공.                                        |
| MATCH_PARTY_START_FAIL_CONTENT           | 1301 | 실패. 사용자 코드에서 거부됨.                          |
| MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD | 1302 | 실패. 파티매칭을 요청할 때, 방이 파티매칭용 방이 아닌경우          |

<br>
파티 매칭이 성공한 경우 유저 매치메이킹에서 사용했던 onMatchUserDone을 통해 알림을 받을 수 있습니다. 그리고 MatchResult 매개변수를 통해 매칭된 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가정보를 얻을 수도 있습니다.
시간 안에 매칭이 성공하지 못한 경우 onMatchUserTimeout 을 통해 알림을 받을 수 있습니다.

<br>

MatchPartyCancel()을 호출하여 진행중인 파티 매치메이킹을 취소할 수 있습니다.

```c#
public async void MatchPartyCancel()
{
    try
    {
        ResultCodeMatchPartyCancel result = await user.MatchPartyCancel("RoomType");
        if (result == ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS)
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

MatchPartyCancel()은 다음과 같은 1개의 매개변수를 가지고 있습니다.

| 타입     | 이름       | 설명               |
|--------|----------|------------------|
| String | roomType | 취소할 매치메이킹의 방 타입. |

응답으로 ResultCodeMatchPartyCancel를 리턴합니다. MatchPartyCancel이 성공하면 ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS 가 리턴되며, 아닌 경우 취소 요청이 실패한 것입니다.

ResultCodeMatchPartyStart의 상세 내용은 다음과 같습니다.

| 이름                                          | 값    | 설명                                         |
|---------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                                 | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                                     | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                                | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL                            | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| MATCH_PARTY_CANCEL_SUCCESS                  | 0    | 성공.                                        |
| MATCH_PARTY_CANCEL_FAIL_CONTENT             | 1401 | 실패. 사용자 코드에서 거부됨.                          |
| MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD   | 1402 | 실패. 파티매칭을 취소할 때, 방이 파티매칭용 방이 아닌경우          |
| MATCH_PARTY_CANCEL_FAIL_ALREADY_JOINED_ROOM | 1403 | 실패, 파티매칭을 취소할 때, 이미 방에 입장해 있는 경우           |
| MATCH_PARTY_CANCEL_FAIL_NOT_IN_PROGRESS     | 1404 | 실패, 파티매칭 진행 중이 아닌데 취소하려고 할 때               | 

### 채널

#### 채널 이동 알림

경우에 따라서 매치메이킹의 결과로 채널 이동이 발생할 수 있습니다. 채널 이동이 되었을 경우 OnMoveChannel을 통해 알림을 받을 수 있습니다. 그리고 MoveChannelResult 매개변수를 이동한 채널의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가정보를 얻을 수도 있습니다.

```c#
public void AddOnMoveChannel()
{
    user.OnMoveChannel += (GameAnvilUser user, MoveChannelResult result) =>
    {
        // 채널 이동시 호출 
    };
}
```

#### 채널 이동

MoveChannel()을 호출하여 서비스 내의 다른 채널로 이동할 수 있습니다.

```c#
public async void MoveChannel()
{
    try
    {
        Payload moveChannelPayload = new Payload(new Protocol.MoveChannelData());
        ErrorResult<ResultCodeMoveChannel, MoveChannelResult> result = await user.MoveChannel("ChannelId", moveChannelPayload);
        if (result.ErrorCode == ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS)
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

MoveChannel()은 다음과 같은 2개의 매개변수를 가지고 있습니다.

| 타입      | 이름        | 설명                                                     |
|---------|-----------|--------------------------------------------------------|
| string  | channelId | 이동할 채널의 아이디                                            |
| Payload | payload   | 채널 이동 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null) |

응답으로 ErrorResult<ResultCodeMoveChannel, MoveChannelResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. MoveChannel이 성공하면 ErrorCode 필드의 값이 ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS 가 되며, 아닌 경우 채널 이동이 실패한 것입니다. Data 필드를 통해 요청 결과 MoveChannelResult 를 얻을 수 있습니다. 이를 통해 이동한 채널의 정보를 얻을 수 있으며, 서버 구현에
따라서 추가정보를 얻을
수도 있습니다.

ResultCodeMoveChannel의 상세 내용은 다음과 같습니다.

| 이름                                       | 값    | 설명                                         |
|------------------------------------------|------|--------------------------------------------|
| PARSE_ERROR                              | -2   | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                                  | -1   | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR                             | 1    | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL                         | 2    | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| MOVE_CHANNEL_SUCCESS                     | 0    | 성공.                                        |
| MOVE_CHANNEL_FAIL_CONTENT                | 1601 | 실패. 사용자 코드에서 거부됨.                          |
| MOVE_CHANNEL_FAIL_NODE_NOT_FOUND         | 1602 | 실패. 채널 노드를 찾을 수 없음.                        |
| MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL | 1603 | 실패. 이미 요청한 채널에 있음.                         |
| MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM    | 1604 | 실패. 이미 방에 입장하여 채널 이동을 할 수 없음.              |

MoveChannelResult 의 상세 내용은 다음과 같습니다.

| 타입       | 이름        | 설명                                                                                  |
|----------|-----------|-------------------------------------------------------------------------------------|
| bool     | Force     | 서버에 의한 강제 채널 이동인지 여부. <br/>값이 true : 서버에서 채널을 이동.<br/>false : 클라이언트의 요청에 의해 채널을 이동. |
| string   | ChannelId | 입장한 방의 아이디.                                                                         |
| Payload? | payload   | 클라이언트에서 필요한 추가 정보.                                                                  |

### 채널 정보

GameAnvil은 설정에서 자유롭게 채널을 구성할 수 있습니다. 이런 채널 구성은 서버와 클라이언트 간에 미리 약속하여 고정된 형태로 사용할 수도 있지만, 상황에 따라 다양하게 변경하여 사용할 수도 있습니다. GameAnvilUser에서는 이렇게 변경된 채널 정보를 얻어올 수 있도록 몇 가지 메소드를 제공합니다.

| 이름                       | 설명                                    |
|--------------------------|---------------------------------------|
| GetChannelCountInfo()    | 특정 채널의 카운트 정보(유저와 방 개수) 요청            | 
| GetChannelInfo()         | 특정 채널의 정보(사용자 정의) 요청                  |
| GetAllChannelCountInfo() | 특정 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수) 요청 |
| GetAllChannelInfo()      | 특정 서비스의 모든 채널에 대한 정보(사용자 정의) 요청       |

#### GetChannelCountInfo

GetChannelCountInfo()는 특정 채널의 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다.

```c#
public async void ChannelCountInfo()
{
    try
    {
        ErrorResult<ResultCodeChannelCountInfo, ChannelCountResult> result = await user.GetChannelCountInfo("ServiceName", "ChannelId");
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

응답으로 ErrorResult<ResultCodeChannelCountInfo, ChannelCountResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. GetChannelCountInfo가 성공하면 ErrorCode 필드의 값이 ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS 가 되며, 아닌 경우 요청이 실패한 것입니다. Data 필드를 통해 요청 결과인 ChannelCountResult 를 얻을 수 있습니다.

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

ChannelCountResult의 상세 내용은 다음과 같습니다.

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
        ErrorResult<ResultCodeChannelInfo, Payload> result = await user.GetChannelInfo("ServiceName", "ChannenId");
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

응답으로 ErrorResult<ResultCodeChannelInfo, Payload>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. GetChannelInfo() 가 성공하면 ErrorCode 필드의 값이 ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS가 되며, 아닌 경우 요청이 실패한 것입니다. 성공시 Data 필드의 Payload 를 사용자가 정의한 채널 정보를 얻을 수도 있습니다. 

ResultCodeChannelInfo의 상세 내용은 다음과 같습니다.

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
        ErrorResult<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>> result = await user.GetAllChannelCountInfo("ServiceName");
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

응답으로 ErrorResult<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. GetAllChannelCountInfo가 성공하면 ErrorCode 필드의 값이 ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS 가 되며, 아닌 경우 요청이 실패한 것입니다. Data 필드를 통해 요청 결과인 Dictionary<string, ChannelCountResult> 를 얻을 수 있습니다. 이 Dictionary는 채널 아이디를 키로, ChannelCountResult를 값으로 가지고 있습니다.

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
        ErrorResult<ResultCodeAllChannelInfo, ChannelInfoResult> result = await user.GetAllChannelInfo("ServiceName");
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


