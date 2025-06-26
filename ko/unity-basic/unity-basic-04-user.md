## Game > GameAnvil > Unity 기초 개발 가이드 > 유저컨트롤러

## GameAnvilUser

는 GameAnvil 서버의 GameNode와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 방 관리 등 기본 기능을 제공하며, 직접 정의한 프로토콜을 기반으로 클라이언트는 자신의 유저 객체를 통해 다른 객체들과 메시지를 주고 받으며 여러 가지 콘텐츠를 구현할 수 있습니다.

GameAnvilManager는 기본적으로 간편 로그인 과정에서 생성된 하나의 GameAnvilUser를 관리하는 GameAnvilUserController를 제공합니다. LoginResult의 UserController를 통해서 얻을 수 있습니다.

### 로그인

로그인은 클라언트가 서버에 접속한 후 GameNode에 자신의 유저 객체를 만드는 과정이라고 정의할 수 있습니다.

로그인은 간편 로그인에서 한 번에 처리되기 때문에 설명을 생략합니다. 자세한 내용은 [Unity 심화 개발 가이드 > 유저](../unity-advanced/unity-advanced-03-user.md)를 참고하십시오.

### 로그아웃

GameAnvilManager에서는 게임 서버에서 로그아웃하고, 자동으로 접속 종료까지 처리됩니다. 로그아웃에 대한 더 자세한 설명은 [Unity 심화 개발 가이드 > 유저](../unity-advanced/unity-advanced-03-user.md)를 참고하십시오.

### 방 생성, 입장, 퇴장

2명 이상의 유저는 방을 통해 동기화된 메시지 흐름을 만들 수 있습니다. 즉, 유저들의 요청은 방 안에서 모두 순서가 보장됩니다. 물론 1명의 유저를 위한 방 생성도 콘텐츠에 따라서 의미를 가질 수도 있습니다. 방을 어떻게 사용할지는 어디까지나 엔진 사용자의 몫입니다.

#### CreateRoom

CreateRoom()을 호출하여 방을 생성하고 그 방으로 입장합니다.

```c#
public async void ManagerCreateRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    
    try
    {
        Payload createRoomPayload = new Payload(new Protocol.CreateRoomData());
        ErrorResult<ResultCodeCreateRoom, CreatedRoomResult> result = await userControll.CreateRoom("RoomName", "RoomType", "MatchingGroup", createRoomPayload);
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

응답으로 ErrorResult<ResultCodeCreateRoom, CreatedRoomResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. CreateRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeCreateRoom.CREATE_ROOM_SUCCESS 가 되며, 아닌 경우 방 생성이 실패한 것입니다. Data 필드를 통해 요청 결과 CreatedRoomResult 를 얻을 수 있습니다. 이를 통해 생성된 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가 정보를 얻을 수도 있습니다.

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

#### JoinRoom

JoinRoom()을 호출하여 이미 생성된 방에 입장합니다.

``` c#
public async void ManagerJoinRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        Payload joinRoomPayload = new Payload(new Protocol.JoinRoomData());
        ErrorResult<ResultCodeJoinRoom, JoinRoomResult> result = await userController.JoinRoom("RoomType", roomId, "MatchingUserCategory", joinRoomPayload);
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

응답으로 ErrorResult<ResultCodeJoinRoom, JoinRoomResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. JoinRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeJoinRoom.JOIN_ROOM_SUCCESS 가 되며, 아닌 경우 방 생성이 실패한 것입니다. Data 필드를 통해 요청 결과 JoinRoomResult 를 얻을 수 있습니다. 이를 통해 입장한 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가 정보를 얻을 수도 있습니다.

ResultCodeJoinRoom 상세 내용은 다음과 같습니다.

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

#### LeaveRoom

LeaveRoom()을 호출하여 입장한 방에서 퇴장할 수 있습니다.

``` c#
public async void ManagerLeaveRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        Payload leaveRoomPayload = new Payload(new Protocol.LeaveRoomData());
        ErrorResult<ResultCodeLeaveRoom, Payload> result = await userControll.LeaveRoom(leaveRoomPayload);
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

응답으로 ErrorResult<ResultCodeLeaveRoom, Payload>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. JoinRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS 가 되며, 아닌 경우 방 생성이 실패한 것입니다. 서버 구현에 따라서 Data 필드의 Payload 를 통해 추가 정보를 얻을 수도 있습니다.

ResultCodeLeaveRoom 상세 내용은 다음과 같습니다.

| 이름                      | 값   | 설명                                         |
|-------------------------|-----|--------------------------------------------|
| PARSE_ERROR             | -2  | 패킷 파싱 에러. 서버와 클라이언트의 버전이 다를 경우 발생할 수 있음.   |
| TIMEOUT                 | -1  | 타임 아웃. 요청에 대한 응답이 정해진 시간내에 오지 않음.          |
| SYSTEM_ERROR            | 1   | 서버 시스템 에러.  서버의 알수 없는 오류로 실패.              |
| INVALID_PROTOCOL        | 2   | 서버에 등록되지 않은 프로토콜. 추가정보에 등록되지 않은 프로토콜이 사용됨. |
| LEAVE_ROOM_SUCCESS      | 0   | 성공.                                        |
| LEAVE_ROOM_FAIL_CONTENT | 801 | 실패. 사용자 코드에서 거부됨.                          |

#### NamedRoom

NamedRoom()을 호출하여 지정한 이름의 방에 입장할 수 있습니다. 지정한 이름의 방이 없을 경우에는 방을 생성한 후 그 방으로 입장합니다.

```c#
public async void ManagerNamedRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;

    try
    {
        bool isParty = false;
        Payload namedRoomPayload = new Payload(new Protocol.NamedRoomData());
        ErrorResult<ResultCodeNamedRoom, NamedRoomResult> result = await userControll.NamedRoom("RoomName", "RoomType", isParty, namedRoomPayload);
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

응답으로 ErrorResult<ResultCodeNamedRoom, NamedRoomResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. NamedRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeNamedRoom.NAMED_ROOM_SUCCESS 가 되며, 아닌 경우 입장 또는 생성이 실패한 것입니다. Data 필드를 통해 요청 결과 NamedRoomResult 를 얻을 수 있습니다. 이를 통해 입장 또는 생성한 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가 정보를 얻을 수도 있습니다.

ResultCodeNamedRoom 상세 내용은 다음과 같습니다.

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

NamedRoomResult 의 상세 내용은 다음과 같습니다.

| 타입      | 이름       | 설명                 |
|---------|----------|--------------------|
| bool    | Created  | 새로운 방이 생성되었는지 여부   |
| int     | RoomId   | 입장한 방의 아이디         |
| String? | RoomName | 입장한 방의 이름          |
| Payload | payload  | 클라이언트에서 필요한 추가 정보. |

### 매치메이킹

GameAnvil은 크게 두 가지 매치메이킹을 제공합니다. 하나는 방 단위의 매칭을 수행하는 룸 매치메이킹이고, 다른 하나는 유저 단위의 매칭을 수행하는 유저 매치메이킹입니다.

#### 룸 매치메이킹

룸 매치메이킹은 조건에 맞는 방으로 유저를 입장시켜 주는 방식입니다. 룸 매치메이킹 요청 시 조건에 맞는 방이 있으면 해당 방으로 바로 입장시켜 주고 조건에 맞는 방이 없다면 새로운 방을 생성하여 입장시켜 줍니다.

MatchRoom()을 호출하여 룸 매치메이킹을 요청할 수 있습니다.

```c#
public async void ManagerMatchRoom()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        var matchRoomPayload = new Payload(new Protocol.MatchRoomData());
        ErrorResult<ResultCodeMatchRoom, MatchResult> result = await userController.MatchRoom(true, true, "RoomType", "MatchingGroup", "MatchingUserCategory", matchRoomPayload);
        if (result.ErrorCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
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

MatchRoom()은 다음과 같은 7개의 매개변수를 가지고 있습니다.

| 타입      | 이름                        | 설명                                                                                                                               |
|---------|---------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| bool    | isCreateRoomIfNotJoinRoom | 조건에 맞는 방을 찾지 못했을 때, 방을 생성하여 입장할지 여부. <br/>true : 방이 없으면 생성한다. <br/>false : 방이 없으면 실패 처리한다.                                       | 
| bool    | isMoveRoomIfJoinedRoom    | 이미 방에 입장한 상태인 경우, 다른 방으로 이동할지 여부. <br/>true : 방에 입장된 상태이면 방을 옮긴다. <br/>false : 방에 입장된 상태로 매칭을 요청한다면 실패 처리한다.                     |
| string  | roomType                  | 방 타입. 같은 타입의 방을 찾는다.                                                                                                             |
| string  | matchingGroup             | 매칭 그룹. 같은 그룹으로 생성된 방을 찾는다.                                                                                                       |
| string  | matchingUserCategory      | 매칭 된 방에서 사용항 유저 카테고리.<br/>각 방에서는 방에 속한 유저를 카테고리로 나누고, 각 카테고리별로 인원수 제한을 적용할 수 있다. 지정한 matchingUserCategory의 현재 인원이 최대가 아닌 방을 찾는다. |
| Payload | payload                   | 매치메이킹 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)                                                                           |
| Payload | leaveRoomPayload          | 다른 방으로 이동하는 경우, 방을 나갈때 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)                                                              |

응답으로 ErrorResult<ResultCodeMatchRoom, MatchResult>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. MatchRoom이 성공하면 ErrorCode 필드의 값이 ResultCodeMatchRoom.NAMED_ROOM_SUCCESS 가 되며, 아닌 경우 입장 또는 생성이 실패한 것입니다. Data 필드를 통해 요청 결과 MatchResult 를 얻을 수 있습니다. 이를 통해 입장 또는 생성한 방의 정보를 얻을 수 있으며, 서버 구현에 따라서 추가정보를 얻을 수도 있습니다.

ResultCodeMatchRoom 상세 내용은 다음과 같습니다.

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
| MATCH_ROOM_FAIL_MATCH_FORM_NULL                | 915 | 실패. 매칭 신청서가 NULL 일 경우.                                                                 |
| MATCH_ROOM_FAIL_MATCH_INFO_NULL                | 916 | 실패. 매칭 정보가 NULL 일 경우.                                                                  |

MatchResult 의 상세 내용은 다음과 같습니다.

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
public async void ManagerMatchUserStart()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    userController.OnMatchUserDone += (GameAnvilUserController userController, ResultCodeMatchUserDone resultCode, MatchResult matchResult) => {
        // 매칭 성공
    };
    userController.OnMatchUserTimeout += (GameAnvilUserController userController) =>
    {
        // 매칭 실패
    };
    try
    {
        Payload matchUserPayload = new Payload(new Protocol.MatchUserData());
        ErrorResult<ResultCodeMatchUserStart, Payload> result = await gameAnvilManager.UserController.MatchUserStart("RoomType", "MatchingGroup", matchUserPayload);
        if(result.ErrorCode == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
        {  
            // 요청 성공
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

MatchUserStart()은 다음과 같은 3개의 매개변수를 가지고 있습니다.

| 타입      | 이름            | 설명                                                                     |
|---------|---------------|------------------------------------------------------------------------|
| String  | roomType      | 생성할 방의 타입.                                                             |
| String  | matchingGroup | 매칭 그룹. 같은 그룹의 유저 풀에서 조건에 맞는 유저를 찾는다. 사용하지 않는 경우 string.Empty(빈 문자열) 입력 |
| Payload | payload       | 유저 매치메이킹 요청을 처리할 서버의 사용자 코드에서 필요한 추가 정보. (default = null)              |

응답으로 ErrorResult<ResultCodeMatchUserStart, Payload>를 리턴하며, ErrorCode 필드를 값을 확인하여 성공 여부를 확인할 수 있습니다. MatchUserStart이 성공하면 ErrorCode 필드의 값이 ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS 가 되며, 아닌 경우 유저 매치메이킹이 실패한 것입니다. 서버 구현에 따라서 Data 필드의 Payload 를 통해 추가 정보를 얻을 수도 있습니다.

ResultCodeMatchUserStart 상세 내용은 다음과 같습니다.

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
public async void ManagerMatchUserCancel()
{
    GameAnvilManager gameAnvilManager = GameAnvilManager.Instance;
    GameAnvilUserController userController = gameAnvilManager.UserController;
    try
    {
        ResultCodeMatchUserCancel result = await userController.MatchUserCancel("RoomType");
        if (result == ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS)
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

응답으로 ResultCodeMatchUserCancel를 리턴하며, 취소가 성공하면 ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS 가 되며, 아닌 경우 취소가 실패한 것입니다. 유저 매치메이킹이 진행중이 아닌 경우, 이미 유저 매치메이킹이 성공했거나 Timeout이 발생한 경우 요청이 실패할 수 있습니다.

ResultCodeMatchUserCancel 상세 내용은 다음과 같습니다.

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

이 밖에 파티 매치메이킹, 채널, 리스너 등 유저 에이전트의 다양한 기능들에 대한 자세한 내용은 [Unity 심화 개발 가이드 > 유저](../unity-advanced/unity-advanced-03-user.md)를 참고하십시오.