## Game > GameAnvil > Typescript 개발 가이드 > User

### GameAnvilUser

GameAnvilUser는 서버의 GameNode상에 존재하는 유저와 대응하는 클라이언트측의 객체입니다. 서버의 유저 객체에 명령을 내리거나 유저의 정보를 받아와서 서버와 클라이언트를 동기화할 수 있습니다. 엔진에 미리 정의된 방 기능, 매치메이킹 기능 등을 사용할 수도 있지만 직접 프로토콜을 구현하여 새로운 기능을 추가할 수도 있습니다.

#### 생성

GameAnvilUser를 사용하기 위해서는 먼저 새로운 GameAnvilUser객체를 생성하고, 생성된 유저를 통해 서버에 로그인해야합니다.

```typescript
const connector: GameAnvilConnector;
const serviceName: string;

const user = new GameAnvilUser(connector, serviceName, 1);
```

#### 다수의 GameUserAgnet 생성

GameAnvilConnector객체는 프로세스내에서 하나만 사용하는 것이 일반적인 반면에 GameAnvilUser는 여러개를 동시에 생성해서 운용하는 것이 지원됩니다. 여러개가 각각 다른 서비스로 로그인 하는 것이 가능하며, 만일 하나의 서비스에 여러개의 GameAnvilUser를 사용하고 싶다면 subId를 이용해 구분해서 생성할 수 있습니다.

```typescript
const connector: GameAnvilConnector;
const serviceName: string;
const otherServiceName: string;

const user1 = new GameAnvilUser(connector, serviceName, 1);
const user2 = new GameAnvilUser(connector, serviceName, 2);
const user3 = new GameAnvilUser(connector, otherServiceName, 1);
```

#### 로그인

GameNode 안에 클라이언트와 대응하는 서버 유저 객체 생성을 요청합니다. GameNode에 로그인을 완료해야만 유저의 여러 다른 기능들을 사용할 수 있습니다.

로그인 할 유저 타입과 채널 아이디를 필수 인자로 받으며 세번째 인자로 추가 정보를 보낼 수 있습니다. 로그인 동작 완료 시점에 Promise를 통해 로그인에 성공했는지 여부와 서버로부터 전달 받은 추가 데이터 등을 확인할 수 있습니다.

```typescript
const userType: stirng;
const channelId: string;
const payload: Payload;

const loginResult = await user.login(userType, channelId, payload);
console.log(`Login Result : ${ResultCodeLogin[loginResult.errorCode]}`);
```

로그인 성공 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (loginResult.errorCode === ResultCodeLogin.LOGIN_SUCCESS) {
    console.log("Login Success");
} else {
    console.log("Login Fail");
}
```

로그인에 실패했을 경우 errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                   | 값   | 설명                                                                      |
|---------------------------------------------|-------|---------------------------------------------------------------------------|
| `PARSE_ERROR`                               | -2    | 패킷 파싱 에러                                                           |
| `TIMEOUT`                                   | -1    | 타임 아웃                                                                |
| `SYSTEM_ERROR`                              | 1     | 서버 시스템 에러                                                         |
| `INVALID_PROTOCOL`                          | 2     | 서버에 등록되지 않은 프로토콜                                            |
| `LOGIN_SUCCESS`                             | 0     | 성공                                                                     |
| `LOGIN_FAIL_CONTENT`                        | 301   | 실패: 컨텐츠에서 거부됨                                                  |
| `LOGIN_FAIL_NOT_EXIST_NODE`                 | 302   | 실패: 노드가 존재하지 않음                                               |
| `LOGIN_FAIL_TIMEOUT_GAME_SERVER`            | 303   | 실패: 게임서버가 응답하지 않음                                           |
| `LOGIN_FAIL_INVALID_SERVICEID`              | 310   | 실패: 잘못된 서비스 아이디                                               |
| `LOGIN_FAIL_INVALID_USERTYPE`               | 311   | 실패: 잘못된 유저 타입                                                   |
| `LOGIN_FAIL_INVALID_USERID`                 | 312   | 실패: 잘못된 유저 아이디                                                 |
| `LOGIN_FAIL_INVALID_SUB_ID`                 | 313   | 실패: 잘못된 SubId                                                       |
| `LOGIN_FAIL_INVALID_CHANNEL_ID`             | 314   | 실패: 잘못된 채널 아이디                                                 |
| `LOGIN_FAIL_LOGINED_OTHER_SERVICE`          | 320   | 실패: 다른 서비스에 로그인 되어있음                                      |
| `LOGIN_FAIL_LOGINED_OTHER_CHANNEL`          | 321   | 실패: 다른 채널에 로그인 되어있음                                        |
| `LOGIN_FAIL_LOGINED_OTHER_USER_TYPE`        | 322   | 실패: 동일 어카운트 아이디로 다른 유저 타입이 로그인 되어있음             |
| `LOGIN_FAIL_LOGINED_OTHER_DEVICE`           | 323   | 실패: 동일 어카운트 아이디로 다른 디바이스 아이디가 로그인 되어있음       |

로그인 전/후 공통으로 유저가 로그인 된 상태인지 확인하기 위해서, isLoggedIn 속성을 확인할 수 있습니다.

```typescript
if (user.isLoggedIn) {
    console.log("Already logged in.");
}
```

#### 로그아웃

서버에서 명시적으로 유저를 삭제하도록 요청할 수 있습니다. 로그아웃 동작 완료 시점에 Promise를 통해 로그아웃에 성공했는지 여부와 서버로부터 전달 받은 추가 데이터 등을 확인할 수 있습니다.

```typescript
const logoutResult = await user.logout();
```

유저가 로그인 상태가 아닌데 로그아웃을 시도한 경우 서버에는 아무런 요청을 하지 않고 성공으로 응답합니다.

로그아웃 성공 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (logoutResult.errorCode === ResultCodeLogout.LOGOUT_SUCCESS) {
    console.log("Logout Success");
} else {
    console.log("Logout Fail");
}
```

로그아웃에 실패했을 경우 errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름              | 값   | 설명                              |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR`          | -2    | 패킷 파싱 에러                   |
| `TIMEOUT`              | -1    | 타임 아웃                        |
| `SYSTEM_ERROR`         | 1     | 서버 시스템 에러                 |
| `INVALID_PROTOCOL`     | 2     | 서버에 등록되지 않은 프로토콜    |
| `LOGOUT_SUCCESS`       | 0     | 성공                             |
| `LOGOUT_FAIL_CONTENT`  | 401   | 실패: 컨텐츠에서 거부됨          |

서버 설정이나 상황에 따라서 서버에서 유저를 강제로 로그아웃 시킬 수도 있습니다. 이 경우에 대비해 아래와 같이 처리 함수를 등록할 수 있습니다.

```typescript
user.onForceLogout = (user, payload) => {
    console.log(`User ${user.userId}} forced to logout.`);
}
```

#### 메시지 수신 콜백 등록

서버로부터 프로토버퍼 메시지를 수신 받았을 때, 처리 함수를 실행하도록 설정할 수 있습니다. 하나의 프로토버퍼에 대해서 하나의 처리 함수만 등록이 가능하며, 이미 처리 함수를 등록한 상태에서 다시 등록한다면 기존 처리 함수는 지워집니다.

처리 함수는 비동기로 동작하므로 의도한 타이밍에 정확히 호출 되지 않을 수 있음에 주의하싶시오.

첫 번째 인자로 들어가는 프로토버퍼 메시지를 트랜스파일 하는 방법은 이후 챕터에서 설명합니다.

```typescript
user.setMessageCallback(UserInfo.descriptor, (connector, resultCode, userInfo) => {
    if (resultCode === ResultCode.SUCCESS) {
        console.log(userInfo.name);
        console.log(userInfo.age);
        console.log(userInfo.job);

        return;
    }
    console.log("Message receive fail.", resultCode);
});
```

#### 방 새로 생성 후 입장

서버에 방 생성 후 바로 입장할 수 있습니다. 방 이름은 필요하지 않은 경우 빈 문자열을 전달하면 됩니다. 방 타입은 서버와 미리 협의된 값을 사용하여야 합니다.

매칭 그룹은 매치메이킹시 사용할 정보입니다. 사용하지 않는 경우에는 빈 문자열로 비워 놓으세요.

방 생성 및 입장 동작 완료 시점에 Promise를 통해 성공 여부와 새로 생성된 방의 아이디, 사용자 커스텀 추가 정보 등을 확인할 수 있습니다.

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const resultCreateRoom = await user.createRoom("", roomType, matchingGroup, payload);
```

동작 성공 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (resultCreateRoom.errorCode === ResultCodeCreateRoom.CREATE_ROOM_SUCCESS) {
     console.log("Create room success");
} else {
    console.log("Create room fail");
}
```

방 생성이나 방 입장에 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                | 값   | 설명                                |
|------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                            | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                                | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                           | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`                       | 2     | 서버에 등록되지 않은 프로토콜      |
| `CREATE_ROOM_SUCCESS`                    | 0     | 성공                               |
| `CREATE_ROOM_FAIL_CONTENT`               | 601   | 실패: 컨텐츠에서 거부됨            |
| `CREATE_ROOM_FAIL_ALREADY_JOINED_ROOM`   | 602   | 실패: 이미 방에 들어가 있음        |
| `CREATE_ROOM_FAIL_CREATE_ROOM_ID`        | 603   | 실패: 방 아이디 발급 실패          |
| `CREATE_ROOM_FAIL_CREATE_ROOM`           | 604   | 실패: 방 생성 실패                 |

방 생성과 방 입장에 성공했을 경우, 응답을 통해 roomId를 포함한 정보를 확인할 수 있습니다.

```typescript
if (resultCreateRoom.errorCode === ResultCodeCreateRoom.CREATE_ROOM_SUCCESS) {
    console.log(`Created room id: ${resultCreateRoom.roomId}`);
    console.log(`Created room name: ${resultCreateRoom.roomName}`);
}
```

방에 입장된 상태인지 확인하기 위해서, isJoinedRoom 속성을 확인할 수 있습니다.

```typescript
if (user.isJoinedRoom) {
    console.log("Alread joined room.");
}
```

방에 입장된 상태일 경우, 입장된 방의 아이디 정보를 roomId 속성을 통해 확인할 수 있습니다.

```typescript
console.log(`Current joined room id: ${user.roomId}`);
```

#### 기존 방에 입장

서버에 생성된 룸 아이디를 알고 있는 경우 해당 룸에 입장을 요청할 수 있습니다.

매칭 유저 카테고리는 매치메이킹시 사용할 정보입니다. 사용하지 않을 경우 빈 문자열로 비워 두세요.

방 입장 동작 완료 시점에 Promise를 통해 성공 여부와 추가 정보 등을 알 수 있습니다.

```typescript
const roomType: string;
const roomId: number;
const matchinUserCategory: string;
const payload: Payload;

const resultJoinRoom = user.joinRoom(roomType, roomId, matchingUserCategory, payload);
```
방 입장 성공 여부는 Promise 결과같인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (resultJoinRoom.errorCode === ResultCodeJoinRoom.JOIN_ROOM_SUCCESS) {
    console.log("Join room success");

} else {
    console.log("Join room fail");
}
```

방 입장에 실패했을 경우 errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                               | 값   | 설명                                |
|-----------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                           | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                               | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                          | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`                      | 2     | 서버에 등록되지 않은 프로토콜      |
| `JOIN_ROOM_SUCCESS`                     | 0     | 성공                               |
| `JOIN_ROOM_FAIL_CONTENT`                | 701   | 실패: 컨텐츠에서 거부됨            |
| `JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST`    | 702   | 실패: 방이 존재하지 않음           |
| `JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM`    | 703   | 실패: 이미 방에 들어가 있음        |
| `JOIN_ROOM_FAIL_ALREADY_FULL`           | 704   | 실패: 이미 룸의 인원수가 차있을 경우|
| `JOIN_ROOM_FAIL_ROOM_MATCH`             | 705   | 실패: 룸매치에서 문제가 발생할 경우 |

방 입장에 성공했을 경우, 아래와 같이 세부 정보를 확인할 수 있습니다.
```typescript
if (resultJoinRoom.errorCode === ResultCodeJoinRoom.JOIN_ROOM_SUCCESS) {
    console.log(`Joined room id: ${resultJoinRoom.roomId}`);
    console.log(`Joined room name: ${resultJoinRoom.roomName}`);
}
```

#### 입장 중인 방에서 퇴장

입장 중인 방에서 퇴장하도록 서버에 요청할 수 있습니다.

퇴장 완료 시점에 Promise를 통해 성공 여부와 추가 정보 등을 확인할 수 있습니다.

```typescript
const payload: Payload;

const leaveRoomResult = await user.leaveRoom(payload);
```

퇴장 완료 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (leaveRoomResult.errorCode === ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS) {
    console.log("Leave room success");
} else {
    console.log("Leave room fail");
}
```

방 퇴장에 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름              | 값   | 설명                                |
|------------------------|-------|-------------------------------------|
| `PARSE_ERROR`          | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`              | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`         | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`     | 2     | 서버에 등록되지 않은 프로토콜      |
| `LEAVE_ROOM_SUCCESS`   | 0     | 성공                               |
| `LEAVE_ROOM_FAIL_CONTENT` | 801 | 실패: 컨텐츠에서 거부됨            |

서버에 의해 강제로 방에서 쫒겨났을 경우 실행할 처리 함수를 등록할 수 있습니다.

```typescript
user.onForceLeaveRoom = (user, roomId, payload) => {
    console.log(`Server forced to leave room. roomId: ${roomId}`);
}
```

#### 유저 매치메이킹 풀에 등록

유저 매치메이킹은 유저 풀을 만들고 그 안에서 조건에 맞는 유저들을 묶어서 새로 생성한 방으로 입장 시켜 주는 방식입니다. 유저 풀에 조건에 맞는 유저의 수가 모자랄 경우, 매치메이킹이 완료될 때 까지 시간이 걸릴 수 있습니다. 제한 시간내에 매치메이킹이 완료되지 않으면 매칭이 취소됩니다.

아래와 같이 서버에 유저 매치메이킹을 요청할 수 있습니다.

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const matchUserStartResult = await matchUserStart(roomType, matchingGroup, payload);
```

풀에 정상적으로 등록되었는지 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (matchUserStartResult.errorCode === ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS) {
    console.log("Match user start success.");
} else {
    console.log("Match user start fail.");
}
```
풀에 등록 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                    | 값   | 설명                                |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                                    | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                               | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`                           | 2     | 서버에 등록되지 않은 프로토콜      |
| `MATCH_USER_START_SUCCESS`                   | 0     | 성공                               |
| `MATCH_USER_START_FAIL_CONTENT`              | 1101  | 실패: 컨텐츠에서 거부됨            |
| `MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM`  | 1102  | 실패: 이미 방에 들어가 있음        |

풀에 정상적으로 등록이 되었다면 서버에서 유저 매칭이 진행되고, 매칭에 성공을 하면 유저에 미리 등록된 콜백이 호출됩니다. 유저 매칭 풀 등록 요청 전에 아래와 같이 콜백을 등록하면 됩니다.

```typescript
user.onMatchUserDone = (user, resultCode, matchResult) => {
    console.log(`Match user done.`, resultCode);

    console.log(`Is request canceled?`, matchResult.isCancel);
    console.log(`Matched room id: `, matchResult.roomId);
    console.log(`Matched room name: `, matchResult.roomName);
    console.log(`Is room created for match?`, matchResult.created);
};
```

풀에 등록 전/후로, 풀에 등록 요청이 전송되고 처리되고 있는 상태인지 확인할 수 있습니다.

```typescript
console.log(`Is in progress of match making?`, user.isUserMatchInPrgress);
```

#### 유저 매치메이킹 풀에서 제거

유저 매치메이킹을 요청했지만 매치메이킹이 아직 진행 중인 상태에서, 요청을 취소할 수 있습니다.

```typescript
const roomType: string;

const resultCode = await user.matchUserCancel(roomType);
console.log(`Match user cancel result: `, ResultCodeMatchUserCancel[resultCode]);
```

풀에서 제거 요청 성공 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (resultCode === ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS) {
    console.log("Match user cancel success.");
} else {
    console.log("Match user cancel fail.");
}
```

요청에 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                    | 값   | 설명                                |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                                    | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                               | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`                           | 2     | 서버에 등록되지 않은 프로토콜      |
| `MATCH_USER_CANCEL_SUCCESS`                  | 0     | 성공                               |
| `MATCH_USER_CANCEL_FAIL`                     | 1201  | 실패: 컨텐츠에서 거부됨            |
| `MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM` | 1202  | 실패: 이미 매칭이 이루어짐         |
| `MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS`     | 1203  | 실패: 매칭이 진행 중이지 않은 경우 |

#### 룸 매치메이킹

룸 매치메이킹은 조건에 맞는 방 으로 유저를 입장시켜 주는 방식입니다. 룸 매치메이킹 요청 시 조건에 맞는 방이 있으면 해당 방으로 바로 입장시켜 주고 조건에 맞는 방이 없다면 새로운 방을 생성하여 입장시켜주거나 요청을 실패 처리 합니다.

입장할 방의 타입과 매칭을 위한 추가 정보인 매칭 그룹과 매칭 유저 카테고리, 그리고 기타 옵션과 추가 전달 정보를 인자로 넘길 수 있습니다.

룸 매치메이킹 동작 완료 시점에 Promise를 통해 성공 여부와 입장한 방의 아이디, 사용자 커스텀 추가 정보 등을 확인할 수 있습니다.

```typescript
const isCreateRoomIfNotJoinRoom: boolean;
const isMoveRoomIfJoinedRoom: boolean;
const roomType: string;
const matchingGroup: string;
const matchingUserCategory: string;
const payload: Payload;
const leaveRoomPayload: Payload;

const matchRoomResult = await user.matchRoom(isCreateRoomIfNotJoinRoom, isMoveRoomIsJoinedRoom, roomType, matchingGroup, matchingUserCategory, payload, leaveRoomPayload);
console.log(`Match room result: ${ResultCodeMatchRoom[matchRoomResult.errorCode]}`);
```

동작 성공 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (matchRoomResult.errorCode === ResultCodeMatchRoom.MATCH_ROOM_SUCCESS) {
    console.log("Match room success");
} else {
    consoel.log("Match room fail");
}
```

매치메이킹에 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                    | 값   | 설명                                |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                                    | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                               | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`                           | 2     | 서버에 등록되지 않은 프로토콜      |
| `MATCH_ROOM_SUCCESS`                         | 0     | 성공                               |
| `MATCH_ROOM_FAIL_CONTENT`                    | 901   | 실패: 컨텐츠에서 거부됨            |
| `MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST`        | 902   | 실패: 방이 존재하지 않음           |
| `MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM`        | 903   | 실패: 이미 방에 들어가 있음        |
| `MATCH_ROOM_FAIL_LEAVE_ROOM`                 | 904   | 실패: 기존 방에서 나가기가 실패한 경우 |
| `MATCH_ROOM_FAIL_IN_PROGRESS`                | 905   | 실패: 이미 매치 메이킹이 진행 중인 경우 |
| `MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST`| 906   | 실패: 조건에 맞는 방을 찾던 중 방이 사라짐 |
| `MATCH_ROOM_FAIL_CREATE_ROOM_ID`             | 907   | 실패: 방 아이디 발급 실패          |
| `MATCH_ROOM_FAIL_CREATE_ROOM`                | 908   | 실패: 방 생성 실패                 |
| `MATCH_ROOM_FAIL_INVALID_ROOM_ID`            | 909   | 실패: 잘못된 룸아이디              |
| `MATCH_ROOM_FAIL_INVALID_NODE_ID`            | 910   | 실패: 잘못된 노드아이디            |
| `MATCH_ROOM_FAIL_INVALID_USER_ID`            | 911   | 실패: 잘못된 유저아이디            |
| `MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND`     | 912   | 실패: 매칭을 진행했으나 방을 찾지 못함 |
| `MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY` | 913 | 실패: 잘못된 매칭 유저 카테고리     |
| `MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY` | 914 | 실패: 매칭 유저 카테고리 사이즈가 0일 경우 |
| `MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL`  | 915   | 실패: 매칭 신청서가 없음           |
| `MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL`  | 916   | 실패: 매칭 정보가 없음             |

룸 매치메이킹에 성공했을 경우, 응답을 통해 roomId를 포함한 정보를 확인할 수 있습니다.

```typescript
if (matchRoomResult.errorCode === ResultCodeMatchRoom.MATCH_ROOM_SUCCESS) {
    console.log(`Match room is cancel: ${matchRoomResult.isCancel}`);
    console.log(`Matched room id: ${matchRoomResult.roomId}`);
    console.log(`Matched room name: ${matchRoomResult.roomName}`);
    consoel.log(`Is matched room created? : ${matchRoomResult.created}`);
}
```

#### 지정한 이름의 방

지정한 이름의 방에 입장하거나, 파티 매칭을 위한 방에 입장할 수 있습니다. 지정한 이름의 방이 없을 경우 새롭게 생성하여 입장합니다.

```typescript
const roomType: stirng;
const roomName: string;
const isParty: boolean;
const payload: Payload;

const namedRoomResult = await user.namedDroom(roomType, roomName, isParty, payload);

console.log(`Named room result: ${ResultCodeNamedRoom[namedRoomResult.errorCode]}`);
```

동작 성공 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (namedRoomResult.errorCode === ResultCodeNamedRoom.NAMED_ROOM_SUCCESS) {
    console.log("Named room success");
} else {
    console.log("Named room success");
}
```

동작에 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                    | 값   | 설명                                |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                                    | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                               | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`                           | 2     | 서버에 등록되지 않은 프로토콜      |
| `NAMED_ROOM_SUCCESS`                         | 0     | 성공                               |
| `NAMED_ROOM_FAIL_CONTENT`                    | 1001  | 실패: 컨텐츠에서 거부됨            |
| `NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST`        | 1002  | 실패: 방 생성이 실패하여 방을 찾을 수 없음 |
| `NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM`        | 1003  | 실패: 이미 방에 들어가 있음        |
| `NAMED_ROOM_FAIL_INVALID_ROOM_NAME`          | 1004  | 실패: 잘못된 방 이름 입력          |
| `NAMED_ROOM_FAIL_CREATE_ROOM`                | 1005  | 실패: 방 생성 실패                 |

요청에 성공했을 경우, 응답을 통해 정보를 확인할 수 있습니다.

```typescript
if (namedRoomResult.errorCode === ResultCodeNamedRoom.NAMED_ROOM_SUCCESS) {
    console.log(`Named room id: ${namedRoomResult.roomId}`);
    console.log(`Named room name: ${namedRoomResult.roomName}`);
    console.log(`Is named room created? : ${namedRoomResult.created}`);
}
```

#### 파티 매칭

파티 매치메이킹은 유저 매치메이킹의 특수한 형태로, 2명 이상의 유저가 한 파티로 묶여 유저 풀에 등록되고, 조건이 맞는 다른 유저들을 찾아 새로 생성한 방으로 함께 입장하는 것입니다. 파티로 묶인 유저들은 항상 같은 방으로 입장하게 됩니다. 파티와 같이 매칭되는 유저들은 서버의 매치메이커 구현에 따라 또 다른 파티거나 개인일 수도 있습니다.

파티 매치메이킹을 하기 위해서는 먼저 네임드 룸에 입장한 상태여야 합니다. 네임드 룸 요청시 isParty를 true로 설정하면, 해당 NamedRoom이 파티의 역할을 합니다. NamedRoom에 파티 유저를 모두 모으고 나서 파티 매치메이킹을 시작할 수 있습니다.

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const matchStartResult = await matchPartyStart(roomType, mathchingGroup, payload);

console.log(`Party match start result: ${ResultCodeMatchPartyStart[matchStartResult.errorCode]}`);
```

정상적으로 파티 매치 메이킹이 시작되었는지 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (matchStartResult.errorCode === ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS) {
    console.log("Match party start success");
} else {
    console.log("Match party start fail");
}
```

파티 매치 시작에 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                    | 값   | 설명                                |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                                    | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                               | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`                           | 2     | 서버에 등록되지 않은 프로토콜      |
| `MATCH_PARTY_START_SUCCESS`                  | 0     | 성공                               |
| `MATCH_PARTY_START_FAIL_CONTENT`             | 1301  | 실패: 컨텐츠에서 거부됨            |
| `MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD`   | 1302  | 실패: 파티 매칭 요청 시, 방이 파티 매칭용 방이 아닌 경우 |

파티 매치 시작이 정상적으로 되었다면 서버에서 파티 매칭이 시작됩니다. 이 때 같은 파티에 있는 유저가 시작시에 알림을 받으려면 아래와 같이 처리 함수를 등록할 수 있습니다.

```typescript
user.onMatchPartyStart = (user, resultCode, payload) => {
    console.log(`User in party started match. result: ${ResultCodeMatchPartyStart[resultCode]}`);
}
```

파티 매칭에 성공을 하면 파티 내의 유저에 미리 등록된 처리 함수가 호출됩니다. 파티 매치 시작 전에 아래와 같이 콜백을 등록하면 됩니다.

```typescript
user.onMatchUserDone = (user, resultCode, matchResult) => {
    console.log(`Match user done.`, resultCode);

    console.log(`Is request canceled?`, matchResult.isCancel);
    console.log(`Matched room id: `, matchResult.roomId);
    console.log(`Matched room name: `, matchResult.roomName);
    console.log(`Is room created for match?`, matchResult.created);
};
```

파티 매칭 전/후로, 아래와 같이 상태를 확인할 수 있습니다.

```typescript
console.log(`Is in progress of match making? ${user.isPartyMatchInProgress}`);
```

#### 파티 매칭 취소

파티 매치메이킹이 아직 진행 중인 상태라면 요청을 취소할 수 있습니다.

```typescript
const roomType: string;

const matchCancelResult = await matchPartyCancel(roomType);

console.log(`Match party cancel result: ${ResultCodeMatchPartyCancel[matchCancelResult.errorCode]}`);
```

정상적으로 취소되었는지 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (matchCancelResult.errorCode === ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS) {
    console.log("Match party cancel success.");
} else {
    consoel.log("Match party cancel fail.");
}
```

취소하는데 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                    | 값   | 설명                                |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                                    | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                               | 1     | 서버 시스템 에러                   |
| `MATCH_PARTY_CANCEL_SUCCESS`                 | 0     | 성공                               |
| `MATCH_PARTY_CANCEL_FAIL_CONTENT`            | 1401  | 실패: 컨텐츠에서 거부됨            |
| `MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD`  | 1402  | 실패: 파티 매칭 취소 시, 방이 파티 매칭용 방이 아닌 경우 |

#### 패킷 전송

게임 서버로 유저의 패킷을 전송할 수 있습니다. 사전에 등록된 프로토콜만 보낼 수 있는 것에 주의하세요.

프로토콜 등록이나 메시지 생성에 대해서는 이어지는 가이드를 참고하세요.

```typescript
const name = "John Doe";
const age = 20;
const job = "artist";
const message = new UserInfo({name, age, job});

user.sendUser(message);
```

#### 패킷 전송 후 응답 패킷 대기

게임 서버로 유저의 패킷을 전송한 후, 서버에서 응답을 보낸다면 받아서 처리할 수 있습니다. 사전에 등록된 프로토콜만 보낼 수 있는 것에 주의하세요.

프로토콜 등록이나 메시지 생성에 대해서는 이어지는 가이드를 참고하세요.

```typescript
const echoResult = await user.requestUser<EchoRes>(new EchoReq({ message: "Hello World!"}), EchoRes);

console.log(echoResult.message); // Hello World!
```

#### 채널 이동

유저가 속한 채널에서 나와서 지정한 채널로 이동시킬 수 있습니다. 

```typescript
const channelId: string;
const payload: Payload;

const moveChannelResult = await user.moveChannel(channelId, payload);

console.log(`Move channel result: ${ResultCodeMoveChannel[moveChannelResult.errorCode]}`);
```

채널 이동이 정상적으로 진행되었는지 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
iF (moveChannelResult.errorCode === ResultcodeMoveChannel.MOVE_CHANNEL_SUCCESS) {
    console.log("Move channel success.");
} else {
    consoel.log("Move channel fail.");
}
```

채널 이동에 실패했을 경우, errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                                    | 값   | 설명                                |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | 패킷 파싱 에러                     |
| `TIMEOUT`                                    | -1    | 타임 아웃                          |
| `SYSTEM_ERROR`                               | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`                           | 2     | 서버에 등록되지 않은 프로토콜      |
| `MOVE_CHANNEL_SUCCESS`                       | 0     | 성공                               |
| `MOVE_CHANNEL_FAIL_CONTENT`                  | 1601  | 실패: 컨텐츠에서 거부됨            |
| `MOVE_CHANNEL_FAIL_NODE_NOT_FOUND`           | 1602  | 실패: 채널 노드를 찾을 수 없음     |
| `MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL`   | 1603  | 실패: 이미 요청한 채널에 있음      |
| `MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM`      | 1604  | 실패: 이미 방에 입장하여 채널 이동을 할 수 없음 |

채널 이동에 성공했을 경우 결과를 아래와 같이 확인할 수 있습니다.

```typescript
if (moveChannelResult.errorCode === ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS) {
    console.log(`Move channel forced: ${moveChannelResult.data.force}`);
    consoel.log(`Move channel id: ${moveChannelResult.data.channelId});
}
```

채널 이동은 서버에 의해 강제로 일어날 수도 있습니다. 그 경우 아래와 같이 콜백을 등록해서 처리합니다.

```typescript
user.onMoveChannel = (user, result) => {
    console.log(`Move channel forced: ${result.force}`);
    consoel.log(`Move channel id: ${result.channelId});

}
```

현재 유저의 채널 아이디 정보를 알고 싶다면 channelId 프로퍼티를 참조하세요.

```typescript
console.log(`Current channel id: ${user.channelId}`);
```

#### 서버로부터의 알림

서버로부터 알림이 오는 것에 대해서 미리 처리 함수를 등록할 수 있습니다. 좀 더 복잡한 형태의 데이터 전달을 원하는 경우에는 커스텀 프로토콜 등록을 고려해보십시오.

```typescript 
user.onNotice = (user, message) => {
    console.log(`Message from server: ${message});
}
```

#### 연결 해제

서버에 의해 연결 해제되거나 기타 이유로 연결이 끊어졌을 때의 처리 함수를 미리 등록할 수 있습니다.

```typescript
user.onSessionClose = (user, resultCode, payload) => {
    console.log(`Session closed. ${ResultCodeSessionClosed[resultCode]}`);
}
```

| 코드 이름                                    | 값   | 설명                                                                                                  |
|----------------------------------------------|-------|------------------------------------------------------------------------------------------------------|
| `SESSION_CLOSE_BASE_USER`                    | 2011  | 서버에서 BaseUser의 `closeConnection()` 호출                                                        |
| `SESSION_CLOSE_ADMIN_KICK`                   | 2012  | 어드민에서 강제 종료                                                                                 |
| `SESSION_CLOSE_DUPLICATE_LOGIN`              | 2032  | 중복 접속으로 인한 강제 종료                                                                         |
| `SESSION_CLOSE_BY_NEW_CONNECTION`            | 2040  | 같은 계정 정보로 새로운 로그인 요청 시 이전 접속 종료. 네트워크 순단 등 재접속 시 사용. 문의 필요.     |
| `SESSION_CLOSE_DISCONNECT_ALARM_FROM_CLIENT` | 2041  | 클라이언트와의 연결 끊김 감지. 일반적으로 발생하지 않으며, 발생 시 GameAnvil 개발팀에 문의 필요.       |
| `SESSION_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION` | 2042 | 세션을 찾을 수 없는 경우. 일반적으로 발생하지 않으며, 발생 시 GameAnvil 개발팀에 문의 필요.            |

#### 어드민에 의해 강제 퇴장

서버의 어드민 툴에 의해 서버에서 쫒겨났을 때의 처리 함수를 미리 등록할 수 있습니다.

```typescript
user.onAdminKickout = (user, message) => {
    console.log(`Message from server: ${message}`);
}
```
