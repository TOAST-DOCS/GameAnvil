## Game > GameAnvil > CocosCreator 개발 가이드 > UserAgent

## UserAgent

UserAgent는 GameAnvil 서버의 GameNode와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 방 관리 등 기본 기능을 제공하며, 직접 정의한 프로토콜을 기반으로 클라이언트는 자신의 유저 객체를 통해 다른 객체들과 메시지를 주고 받으며 여러 가지 컨텐츠를 구현할 수 있습니다. UserAgent를 사용하기 위해서는 Connector.CreateUserAgent() 함수를 이용해 새로운 UserAgent를 생성해야 합니다. ServiceName과 SubId로 구분되는 여러 개의 UserAgent를 생성할 수 있습니다. 생성된 UserAgent는 Connector 에서 내부적으로 관리되어 Connector.GetUserAgent()함수를 이용해 다시 사용할 수 있습니다. 

```typescript
/**
 * 서비스이름과 서브 아이디에 해당하는 유저 에이전트를 반환
 * @param serviceName 유저에이전트가 사용하는 서비스이름
 * @param subId 서비스별 유저에이전트를 식별 할 수 있는 고유 아아디. 서버 구현에 따라 사용하지 않을 수 있음
 * @returns 해당 유저 에이전트, 없으면 null
 */
let userAgent = connector.GetUserAgent(serviceName, subId);
if(userAgent == null){
    /**
     * 유저 에이전트 생성
     * @param serviceName 유저에이전트가 사용 할 서비스 이름
     * @param subId 서비스별 유저에이전트를 식별 할 수 있는 고유 아이디. 서버 구현에 따라 사용하지 않을 수 있음
     * @returns 생성된 유저 에이전트
     */
    userAgent = connector.CreateUserAgent(serviceName, subId);
}

```

GameAnvil 서버는 여러 개의 서비스를 동시에 운영할 수 있으며, 하나의 UserAgent는 하나의 서비스에 로그인 하여 서로 독립적으로 동작하게 됩니다. 즉 여러 개의 UserAgent를 만들어 서로 다른 서비스에 로그인하여 동시에 사용이 가능합니다. SubId를 다르게 한다면 같은 서비스에 여러 개의 UserAgent를 동시에 로그인하여 사용하는 것도 가능합니다. 

### 로그인/로그아웃

로그인은 클라언트가 서버에 접속한 후 GameNode에 자신의 유저 객체를 만드는 과정이라고 정의할 수 있습니다. 로그아웃은 로그인의 반대 개념입니다. 즉, GameNode 상에서 자신의 유저 객체를 제거하는 과정입니다. 

로그인 시 어떤 UserType으로 어떤 채널에 로그인할지 입력해줘야 합니다. 추가 정보가 필요하다면 Payload에 담아 보낼 수 있습니다. 

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
    } else {
        // 실패
    }
})

/**
 * 현재 서비스에서 로그아웃
 * @param callback 로그아웃 결과를 전달받아 처리 할 콜백
 */
userAgent.Logout((agent: UserAgent, resultCode: ResultCodeLogout, payload: Payload)=>{
    /**
     * Logout()의 결과
     * @param user Logout()을 요청한 유저에이전트
     * @param resultCode  Logout() 결과 코드
     * @param force 강제 로그아웃여부
     * @param payload 서버로 부터 받은 추가정보
     */
    if (ResultCodeLogout.LOGOUT_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

### 방생성, 입장, 퇴장

2명 이상의 유저는 방을 통해 동기화된 메시지 흐름을 만들 수 있습니다. 즉, 유저들의 요청은 방 안에서 모두 순서가 보장됩니다. 물론 1명의 유저를 위한 방 생성도 컨텐츠에 따라서 의미를 가질 수도 있습니다. 방을 어떻게 사용할지는 어디까지나 엔진 사용자의 몫입니다. 

CreateRoom()을 호출하여 방을 생성하고 그 방으로 입장합니다.

```typescript
/**
 * 임의의 방을 생성하고 해당 방에 입장
 * @param roomType 생성할 방의 룸타입
 * @param roomName 생성할 방의 이름
 * @param payload 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 방 생성 결과를 전달 받아 처리 할 콜백
 */
userAgent.CreateRoom(roomType, roomName, payload, (agent: UserAgent, resultCode: ResultCodeCreateRoom, roomId: number, roomName: string, payload: Payload)=>{
    /**
     * CreateRoom()의 결과
     * @param user CreateRoom()을 요청한 유저에이전트
     * @param resultCode CreateRoom() 결과 코드
     * @param roomId 생성된 방의 룸아이디
     * @param roomName 생성된 방의 이름
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    if (ResultCodeCreateRoom.CREATE_ROOM_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

JoinRoom() 을 호출하여 이미 생성된 방에 입장합니다. 

``` typescript
/**
 * 지정한 룸아이디의 방에 입장
 * 
 * 지정한 룸아이디의 방이 없을 경우 실패
 * @param roomId 입장하고자 하는 룸아이디
 * @param roomType 입장할 방의 룸타입
 * @param payload 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 방 입장 결과를 전달받아 처리 할 콜백
 */
userAgent.JoinRoom(roomId, roomType, payload, (agent: UserAgent, resultCode: ResultCodeJoinRoom, roomId: number, roomName: string, payload: Payload)=>{
    /**
     * JoinRoom()의 결과
     * @param user JoinRoom()을 요청한 유저에이전트
     * @param resultCode JoinRoom() 결과 코드
     * @param roomId 입장한 방의 룸아이디
     * @param roomName 입장한 방의 이름
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    if (ResultCodeJoinRoom.JOIN_ROOM_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

LeaveRoom()을 호출하여 입장한 방에서 퇴장할 수 있습니다.

``` typescript
/**
 * 현재 방에서 나가기
 * @param payload 서버에 전달할 추가 정보.서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 방나가기 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.LeaveRoom(payload, (agent: UserAgent, resultCode: ResultCodeLeaveRoom, roomId: number, payload: Payload)=>{
    /**
     * LeaveRoom()의 결과
     * @param user LeaveRoom()을 요청한 유저에이전트 
     * @param resultCode LeaveRoom() 결과 코드
     * @param force 강퇴 여부
     * @param roomId 퇴장한 방의 룸아이디
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    if (ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

NamedRoom()을 호출하여 지정한 이름의 방에 입장할 수 있습니다. 지정한 이름의 방이 없을 경우에는 방을 생성한 후 그방으로 입장합니다.

```typescript
/**
 * 지정한 이름의 방에 입장
 * 
 * 지정한 이름의 방이 없을 경우 지정한 이름의 방을 생성하고 해당 방에 입장
 * @param roomName 입장하고자 하는 방의 이름
 * @param roomType 입장할 방의 룸타입
 * @param isParty true - 파티 매칭을 목적으로 만들어진 방. false - 일반 방
 * @param payload 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 네임드룸 처리 결과를 전달받아 처리 할 콜백
 */
userAgent.NamedRoom(roomName, roomType, isParty, payload, (agent: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payLoad: Payload)=>{
    /**
     * NamedRoom()의 결과
     * @param user NamedRoom()을 요청한 유저에이전트 
     * @param resultCode NamedRoom() 결과 코드
     * @param roomId 입장한 방의 룸아이디
     * @param roomName 입장한 방의 이름
     * @param created 입장한 방을 생성했는지 여부(방장 여부)
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    if (ResultCodeNamedRoom.NAMED_ROOM_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```



### 매치 메이킹

GameAnvil은 두 가지 매치 메이킹을 제공합니다. 각각 방 단위의 매칭을 수행하는 룸 매치 메이킹과 유저 단위의 매칭을 수행하는  유저 매치 메이킹 입니다.

#### 룸 매치 메이킹

룸 매치 메이킹은 조건에 맞는 방으로 유저를 입장시켜주는 방식입니다. 룸 매치 메이킹 요청시 조건에 맞는 방이 있으면 해당방으로 바로 입장 시켜주고 조건에 맞는 방이 없다면 새로운 방을 생성하여 입장 시켜주는 방식입니다. 

MatchRoom()을 호출하여 룸 매치 메이킹을 요청할 수 있습니다. 조건에 맞는 방이 없을 경우 임의의 방을 생성하고 해당 방에 입장할 수도 있습니다. 

```typescript
/**
 * 룸 매치를 요청
 * 
 * 룸이 없을 경우 임의의 방을 생성하고 해당 방에 입장할 수도 있음
 * @param isCreateRoomIfNotJoinRoom true - 입장할 룸이 없을 경우 임의의 룸을 생성후 입장. false - 입장할 룸이 없을 경우 실패
 * @param isMoveRoomIfJoinedRoom true - 룸에 입장한 상태에서 요청했을 경우 다른 룸으로 이동. false - 룸에 입장한 상태에서 요청했을 경우 실패
 * @param roomType 입장할 방의 룸타입
 * @param payload 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param leaveRoomPayload 입장한 룸을 떠날때 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 룸매치 결과를 전달받아 처리 할 콜백
 */
userAgent.MatchRoom(isCreateRoomIfNotJoinRoom, isMoveRoomIfJoinedRoom, roomType, payload, leaveRoomPayload, (agent: UserAgent, resultCode: ResultCodeMatchRoom, roomId: number, roomName: string, created: boolean, payload: Payload) => {
    /**
     * MatchRoom()의 결과
     * @param user MatchRoom()을 요청한 유저에이전트 
     * @param resultCode MatchRoom() 결과 코드
     * @param roomId 매치된 방의 룸아이디
     * @param roomName 매치된 방의 이름
     * @param created 매치된 방의 생성여부(방장 여부)
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    if (ResultCodeMatchRoom.MATCH_ROOM_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

#### 유저 매치 메이킹

유저 메치 메이킹은 유저풀을 만들고 그 안에서 조건에 맞는 유저들을 찾아 새로 생성한 방으로 입장 시켜주는 방식입니다. 유저풀에 조건에 맞는 유저의 수가 모자랄경우 매치 메이킹이 완료될때까지 시간이 걸릴 수 있고, 시간내에 매치 메이킹이 완료되지 않으면 타임아웃이 되어 매칭이 취소될 수 있습니다. 

MatchUserStart()를 호출하여 유저 매치 메이킹을 시작할 수 있습니다. 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패할 수 있습니다. 

```typescript
/**
 * 유저매치를 요청
 * 
 * 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패할 수 있음
 * 
 * 매칭이 성공한 경우 OnMatchUserDone()을 통해 알림을 받을 수 있음
 * @param roomType 매치를 요청할 룸타입
 * @param payload 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 유저 매치 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.MatchUserStart(roomType, payload, (agent: UserAgent, resultCode: ResultCodeMatchUserStart, payLoad: Payload)=>{
    /**
     * MatchUserStart()의 결과
     * @param user MatchUserStart()를 요청한 유저에이전트 
     * @param resultCode MatchUserStart() 결과 코드
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    if (ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

매칭이 성공한 경우 IUserListener.OnMatchUserDone을 통해 알림을 받을 수 있고, 시간 안에 매칭이 성공하지 못한 경우 IUserListener.OnMatchUserTimeout을 통해 알림을 받을 수 있습니다.

```typescript
class MatchUserListener implements IUserListener {
    /**
     * MatchUserStart(), MatchPartyStart()의 매칭 결과
     * @param user 매칭을 요청한 유저에이전트 
     * @param resultCode 매칭 결과 코드
     * @param created 방 생성 여부. true일 경우 매칭 요청한 UserAgent가  방을 생성한 것을 의미한다. 방장을 결정하는 용도 등으로 사용할 수 있음
     * @param roomId 매칭된 방의 룸아이디
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnMatchUserDone(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void {
        if (ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS == resultCode) {
            // 성공
        } else {
            // 실패
        }
    }

    /**
     * MatchUserStart(), MatchPartyStart()의 매칭의 타임아웃시 발생
     * @param user 매칭을 요청한 유저에이전트 
     */
    OnMatchUserTimeout(user: UserAgent): void {
        // 타임아웃
    }
}
userAgent.AddListener(new MatchUserListener());
```

MatchUserCancel()을 호출하여 유저 매치 메이킹을 취소할 수 있습니다. 매치 요청중이 아닌경우, 이미 매치 메이킹이 성공했거나 Timeout이 발생했을 경우 실패할 수 있습니다. 

```typescript
/**
 * 유저매치 요청을 취소
 * 
 * 매치 요청중이 아닌경우나 이미 매칭이 성공했거나 타임아웃이 발생했을 경우는 실패
 * @param roomType 매치를 요청한 룸타입
 * @param callback 유저매치 취소 요청 결과를 전달 받아 처리 할 콜백
 */
userAgent.MatchUserCancel(roomType, (agent: UserAgent, resultCode: ResultCodeMatchUserCancel) => {
    /**
     * MatchUserCancel()의 결과
     * @param user MatchUserCancel()을 요청한 유저에이전트 
     * @param resultCode MatchUserCancel() 결과 코드
     */
    if (ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

#### 파티 매치 메이킹

파티 매치 메이킹은 유저 매치 메이킹의 특수한 형태로, 2명 이상의 유저가 한그룹으로 묶여 유저 풀에 등록이 되고, 조건이 맞는 다른 유저들을 찾아 새로 생성한 방으로 입장 시켜주는 방식입니다. 그룹이 묶은 유들은 항상같은 방으로 입장할 수 있고, 같이 입장하는 유저들 은 서버의 매치 메이커 구현에 따라 그룹일 수도 있고, 개인일 수도 있습니다. 

파티 매치 메이킹을 하기위해서는 먼저 NamedRoom()을 호출하여 파티를 맺을 유저들이 한 방에 모여야 합니다. 

```typescript
/**
 * 지정한 이름의 방에 입장
 * 
 * 지정한 이름의 방이 없을 경우 지정한 이름의 방을 생성하고 해당 방에 입장
 * @param roomName 입장하고자 하는 방의 이름
 * @param roomType 입장할 방의 룸타입
 * @param isParty true - 파티 매칭을 목적으로 만들어진 방. false - 일반 방
 * @param payload 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 네임드룸 처리 결과를 전달받아 처리 할 콜백
 */
let isParty = true;
userAgent.NamedRoom(roomName, roomType, isParty, payload, (agent: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payLoad: Payload) => {
    /**
     * NamedRoom()의 결과
     * @param user NamedRoom()을 요청한 유저에이전트 
     * @param resultCode NamedRoom() 결과 코드
     * @param roomId 입장한 방의 룸아이디
     * @param roomName 입장한 방의 이름
     * @param created 입장한 방을 생성했는지 여부(방장 여부)
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    if (ResultCodeNamedRoom.NAMED_ROOM_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

MatchPartyStart()를 호출하여 파티 매치 메이킹을 시작할 수 있습니다. 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패할 수 있습니다. 

```typescript
/**
 * 파티 매치를 요청
 * @param roomType 매치를 요청할 룸타입
 * @param payload 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 파티 매치 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.MatchPartyStart(roomType, payload, (agent: UserAgent, resultCode: ResultCodeMatchPartyStart, payload: Payload) => {
    /**
     * MatchPartyStart()의 결과
     * @param user MatchPartyStart()을 요청한 유저에이전트 
     * @param resultCode MatchPartyStart() 결과 코드
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.
     */
    if (ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

매칭이 성공한 경우 유저 매치 메이킹에서 사용했던 IUserListener.OnMatchUserDone을 통해 알림을 받을 수 있고, 시간 안에 매칭이 성공하지 못한 경우 역시 IUserListener.OnMatchUserTimeout을 통해 알림을 받을 수 있습니다.

```typescript
class MatchPartyListener implements IUserListener {
    /**
     * MatchUserStart(), MatchPartyStart()의 매칭 결과
     * @param user 매칭을 요청한 유저에이전트 
     * @param resultCode 매칭 결과 코드
     * @param created 방 생성 여부. true일 경우 매칭 요청한 UserAgent가  방을 생성한 것을 의미한다. 방장을 결정하는 용도 등으로 사용할 수 있음
     * @param roomId 매칭된 방의 룸아이디
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnMatchUserDone(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void {
        if (ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS == resultCode) {
            // 성공
        } else {
            // 실패
        }
    }

    /**
     * MatchUserStart(), MatchPartyStart()의 매칭의 타임아웃시 발생
     * @param user 매칭을 요청한 유저에이전트 
     */
    OnMatchUserTimeout(user: UserAgent): void {
        // 타임아웃
    }
}
userAgent.AddListener(new MatchPartyListener());
```

MatchPartyCancel()을 호출하여 유저 매치 메이킹을 취소할 수 있습니다. 매치 요청중이 아닌경우, 이미 매치 메이킹이 성공했거나 Timeout이 발생했을 경우 실패할 수 있습니다. 

```typescript
/**
 * 파티매치 요청을 취소
 * @param roomType 매치를 요청한 룸타입
 * @param callback 파티매치 취소 쵸청 결과를 전달받아 처리 할 콜백
 */
userAgent.MatchPartyCancel(roomType, (agent: UserAgent, resultCode: ResultCodeMatchPartyCancel) => {
    /**
     * MatchPartyCancel()의 결과
     * @param user MatchPartyCancel()을 요청한 유저에이전트 
     * @param resultCode MatchPartyCancel()의 결과 코드
     */
    if (ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

#### 채널 이동

경우에 따라서 매치 메이킹의 결과로 채널이동이 발생할 수 있습니다. 채널 이동이 되었을 경우  IUserListener.OnMoveChannel을 통해 알림을 받을 수 있습니다.

```typescript
class MoveChannelListener implements IUserListener {
    /**
     * MoveChannel()의 결과 또는 서버에서 강제로 수행한 체널 이동 알림
     * @param user 체널을 이동한 유저에이전트 
     * @param resultCode 체널이동 결과 코드
     * @param force 서버에서 강제로 체널을 이동했는지 여부
     * @param channelId 이동한 체널아이디
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnMoveChannel?(user: UserAgent, resultCode: ResultCodeMoveChannel, force: boolean, channelId: string, payload: Payload): void {
        // 채널 이동
    }
}
userAgent.AddListener(new MoveChannelListener());
```



### 채널 정보

GameAnvil은 설정으로 자유롭게 채널을 구성할 수 있습니다. 이런 채널구성은 서버와 클라이언트간에 미리 약속하여 고정된 형태로 사용할 수도 있지만, 상황에 따라 다양하게 변경하여 사용할 수도 있습니다. UserAgent에서는 이렇게 변경된 채널 정보를 얻어오거나 채널을 이동할 수 있도록 몇가지 함수를 제공합니다. 

GetChannelCountInfo()는 채널의 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 

```typescript
/**
 * 접속중인 채널의 유저와 방 개수를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param callback 채널의 유저와 방 개수 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.GetChannelCountInfo((agent: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo) => {
    /**
     * GetChannelCountInfo()의 결과
     * @param user GetChannelCountInfo()를 요청한 유저에이전트
     * @param resultCode GetChannelCountInfo()의 결과 코드
     * @param channelCountInfo 체널의 유저와 방 개수
     */
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});

/**
 * 특정 채널의 유저와 방 개수를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param serviceName 채널 정보를 요청할 서비스이름
 * @param channelId 채널 정보를 요청할 채널아이디
 * @param callback 채널의 유저와 방 개수 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.GetChannelCountInfo(serviceName, channelId, (agent: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo) => {
    /**
     * GetChannelCountInfo()의 결과
     * @param connection GetChannelCountInfo()를 요청한 커넥션에이전트
     * @param resultCode GetChannelCountInfo()의 결과 코드
     * @param channelCountInfo 체널의 유저와 방 개수
     */
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

GetChannelInfo()는 채널의 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 

```typescript
/**
 * 접속중인 채널 정보를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param callback 채널 정보 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.GetChannelInfo((agent: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload) => {
    /**
     * GetChannelInfo()의 결과
     * @param user GetChannelInfo()를 요청한 유저에이전트
     * @param resultCode GetChannelInfo()의 결과 코드
     * @param channelInfo 체널 정보
     */
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});

/**
 * 특정 채널의 정보를 요청
 * 서버에서 지원할 경우 사용할 수 있음
 * @param serviceName 채널 정보를 요청할 서비스이름
 * @param channelId 채널 정보를 요청할 채널아이디
 * @param callback 채널 정보 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.GetChannelInfo(serviceName, channelId, (agent: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload) => {
    /**
     * GetChannelInfo()의 결과
     * @param connection GetChannelInfo()를 요청한 커넥션에이전트
     * @param resultCode GetChannelInfo()의 결과 코드
     * @param channelInfo 체널 정보
     */
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

GetAllChannelCountInfo()는 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 인자로 서비스 이름과 응답을 처리할 콜백을 넘겨줍니다. 

```typescript
/**
 * 접속중인 채널 정보를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param callback 채널 정보 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.GetAllChannelCountInfo((agent: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo) => {
    /**
     * GetAllChannelCountInfo()의 결과
     * @param connection GetAllChannelCountInfo()를 요청한 커넥션에이전트
     * @param resultCode GetAllChannelCountInfo()의 결과 코드
     * @param channelInfoList 체널의 유저와 방 개수
     */
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});

/**
 * 특정 서비스에 있는 모든 채널의 유저와 방의 개수를 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param serviceName 채널 정보를 요청할 서비스이름
 * @param callback 모든 채널의 유저와 방의 개수 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.GetAllChannelCountInfo(serviceName, (agent: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo) => {
    /**
     * GetAllChannelCountInfo()의 결과
     * @param connection GetAllChannelCountInfo()를 요청한 커넥션에이전트
     * @param resultCode GetAllChannelCountInfo()의 결과 코드
     * @param channelInfoList 체널의 유저와 방 개수
     */
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
});
```

GetAllChannelInfo()는 서비스의 모든 채널에 대한 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 인자로 서비스 이름과 응답을 처리할 콜백을 넘겨줍니다. 

```typescript
/**
 * 접속중인 서비스의 모든 채널정보 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param callback 모든 채널 정보 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.GetAllChannelInfo((agent: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo) => {
    /**
     * GetAllChannelInfo()의 결과
     * @param user GetAllChannelInfo()를 요청한 유저에이전트
     * @param resultCode GetAllChannelInfo()의 결과 코드
     * @param allChannelInfo 체널 정보
     */
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})

/**
 * 특정 서비스의 모든 채널정보 요청
 * 
 * 서버에서 지원할 경우 사용할 수 있음
 * @param serviceName 채널 정보를 요청할 서비스아름
 * @param callback 모든 채널 정보 요청 결과를 전달받아 처리 할 콜백
 */
userAgent.GetAllChannelInfo(serviceName, (agent: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo) => {
    /**
     * GetAllChannelInfo()의 결과
     * @param user GetAllChannelInfo()를 요청한 유저에이전트
     * @param resultCode GetAllChannelInfo()의 결과 코드
     * @param allChannelInfo 체널 정보
     */
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```

MoveChannel()을 호출하여 서비스 내의 다른 채널로 이동할 수 있습니다. 

```typescript
/**
 * 지정한 채널로 이동
 * @param channelId 이동할 채널아이디
 * @param payload 서버에 전달할 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
 * @param callback 채널 이동 결과를 전달받아 처리 할 콜백
 */
userAgent.MoveChannel(channelId, payload, (agent: UserAgent, resultCode: ResultCodeMoveChannel, channelId: string, payload: Payload) => {
    /**
     * MoveChannel()의 결과 또는 서버에서 강제로 수행한 체널 이동 알림
     * @param user 체널을 이동한 유저에이전트 
     * @param resultCode 체널이동 결과 코드
     * @param force 서버에서 강제로 체널을 이동했는지 여부
     * @param channelId 이동한 체널아이디
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    if (ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS == resultCode) {
        // 성공
    } else {
        // 실패
    }
})
```



### Listener

IUserListener는 UserAgent의 모든 동작의 결과 또는 알림을 정의한 인터페이스입니다. 이 인터페이스를 구현한 리스너를 UserAgent.AddUserListener()로 등록하면 등록한 리스너로 응답을 받을 수 있습니다. IUserListener의 함수들은 모두 옵셔널 이므로 사용하는 함수만 구현해 사용할 수도 있습니다. 

```typescript
class UserListener implements IUserListener {
    /**
     * Login()의 결과
     * @param user Login()을 요청한 유저에이전트
     * @param resultCode Login() 결과 코드 
     * @param loginInfo Login() 정보
     */
    OnLogin?(user: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo): void { }
    /**
     * Logout()의 결과
     * @param user Logout()을 요청한 유저에이전트
     * @param resultCode  Logout() 결과 코드
     * @param force 강제 로그아웃여부
     * @param payload 서버로 부터 받은 추가정보
     */
    OnLogout?(user: UserAgent, resultCode: ResultCodeLogout, force: boolean, payload: Payload): void { }
    /**
     * MatchRoom()의 결과
     * @param user MatchRoom()을 요청한 유저에이전트 
     * @param resultCode MatchRoom() 결과 코드
     * @param roomId 매치된 방의 룸아이디
     * @param roomName 매치된 방의 이름
     * @param created 매치된 방의 생성여부(방장 여부)
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnMatchRoom?(user: UserAgent, resultCode: ResultCodeMatchRoom, roomId: number, roomName: string, created: boolean, payload: Payload): void { }
    /**
     * CreateRoom()의 결과
     * @param user CreateRoom()을 요청한 유저에이전트
     * @param resultCode CreateRoom() 결과 코드
     * @param roomId 생성된 방의 룸아이디
     * @param roomName 생성된 방의 이름
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnCreateRoom?(user: UserAgent, resultCode: ResultCodeCreateRoom, roomId: number, roomName: string, payload: Payload): void { }
    /**
     * JoinRoom()의 결과
     * @param user JoinRoom()을 요청한 유저에이전트
     * @param resultCode JoinRoom() 결과 코드
     * @param roomId 입장한 방의 룸아이디
     * @param roomName 입장한 방의 이름
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnJoinRoom?(user: UserAgent, resultCode: ResultCodeJoinRoom, roomId: number, roomName: string, payload: Payload): void { }
    /**
     * NamedRoom()의 결과
     * @param user NamedRoom()을 요청한 유저에이전트 
     * @param resultCode NamedRoom() 결과 코드
     * @param roomId 입장한 방의 룸아이디
     * @param roomName 입장한 방의 이름
     * @param created 입장한 방을 생성했는지 여부(방장 여부)
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnNamedRoom?(user: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payload: Payload): void { }
    /**
     * LeaveRoom()의 결과
     * @param user LeaveRoom()을 요청한 유저에이전트 
     * @param resultCode LeaveRoom() 결과 코드
     * @param force 강퇴 여부
     * @param roomId 퇴장한 방의 룸아이디
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnLeaveRoom?(user: UserAgent, resultCode: ResultCodeLeaveRoom, force: boolean, roomId: number, payload: Payload): void { }
    /**
     * MatchUserStart()의 결과
     * @param user MatchUserStart()를 요청한 유저에이전트 
     * @param resultCode MatchUserStart() 결과 코드
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnMatchUserStart?(user: UserAgent, resultCode: ResultCodeMatchUserStart, payload: Payload): void { }
    /**
     * MatchUserCancel()의 결과
     * @param user MatchUserCancel()을 요청한 유저에이전트 
     * @param resultCode MatchUserCancel() 결과 코드
     */
    OnMatchUserCancel?(user: UserAgent, resultCode: ResultCodeMatchUserCancel): void { }
    /**
     * MatchUserStart(), MatchPartyStart()의 매칭 결과
     * @param user 매칭을 요청한 유저에이전트 
     * @param resultCode 매칭 결과 코드
     * @param created 방 생성 여부. true일 경우 매칭 요청한 UserAgent가  방을 생성한 것을 의미한다. 방장을 결정하는 용도 등으로 사용할 수 있음
     * @param roomId 매칭된 방의 룸아이디
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnMatchUserDone?(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void { }
    /**
     * MatchUserStart(), MatchPartyStart()의 매칭의 타임아웃시 발생
     * @param user 매칭을 요청한 유저에이전트 
     */
    OnMatchUserTimeout?(user: UserAgent): void { }
    /**
     * MatchPartyStart()의 결과
     * @param user MatchPartyStart()을 요청한 유저에이전트 
     * @param resultCode MatchPartyStart() 결과 코드
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.
     */
    OnMatchPartyStart?(user: UserAgent, resultCode: ResultCodeMatchPartyStart, payload: Payload): void { }
    /**
     * MatchPartyCancel()의 결과
     * @param user MatchPartyCancel()을 요청한 유저에이전트 
     * @param resultCode MatchPartyCancel()의 결과 코드
     */
    OnMatchPartyCancel?(user: UserAgent, resultCode: ResultCodeMatchPartyCancel): void { }
    /**
     * MoveChannel()의 결과 또는 서버에서 강제로 수행한 체널 이동 알림
     * @param user 체널을 이동한 유저에이전트 
     * @param resultCode 체널이동 결과 코드
     * @param force 서버에서 강제로 체널을 이동했는지 여부
     * @param channelId 이동한 체널아이디
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnMoveChannel?(user: UserAgent, resultCode: ResultCodeMoveChannel, force: boolean, channelId: string, payload: Payload): void { }
    /**
     * Snapshot()의 결과
     * @param user Snapshot()을 요청한 유저에이전트 
     * @param resultCode Snapshot()의 결과 코드
     * @param payload 서버로 부터 받은 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
     OnSnapshot?(user: UserAgent, resultCode: ResultCodeSnapshot, payload: Payload): void { }
    /**
     * GetChannelInfo()의 결과
     * @param user GetChannelInfo()를 요청한 유저에이전트
     * @param resultCode GetChannelInfo()의 결과 코드
     * @param channelInfo 체널 정보
     */
    OnChannelInfo?(user: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload): void { }
    /**
     * GetChannelCountInfo()의 결과
     * @param user GetChannelCountInfo()를 요청한 유저에이전트
     * @param resultCode GetChannelCountInfo()의 결과 코드
     * @param channelCountInfo 체널의 유저와 방 개수
     */
    OnChannelCountInfo?(user: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo): void { }
    /**
     * GetAllChannelInfo()의 결과
     * @param user GetAllChannelInfo()를 요청한 유저에이전트
     * @param resultCode GetAllChannelInfo()의 결과 코드
     * @param allChannelInfo 체널 정보
     */
    OnAllChannelInfo?(user: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo): void { }
    /**
     * GetAllChannelCountInfo()의 결과
     * @param user GetAllChannelCountInfo()를 요청한 유저에이전트
     * @param resultCode GetAllChannelCountInfo()의 결과 코드
     * @param channelInfoList 체널의 유저와 방 개수
     */
    OnAllChannelCountInfo?(user: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo): void { }
    /**
     * 오류 발생
     * @param user 오류가 발생한 유저에이전트 
     * @param errorCode 오류 코드
     * @param msgName 오류가 발생한 기능 또는 메세지
     */
    OnErrorCommand?(user: UserAgent, errorCode: ErrorCode, msgName: string): void { }
    /**
     * 공지 알림
     * @param user 공지 를 받은 유저에이전트 
     * @param message 공지 메세지
     */
    OnNotice?(user: UserAgent, message: string): void { }
    /**
     * 어드민에서 Kickout()한 경우 알림
     * @param user Kickout()된 유저에이전트 
     * @param message 어드민에서 전달한 메세지
     */
    OnAdminKickoutNoti?(user: UserAgent, message: string): void { }
    /**
     * 유저리스너에서 서버의 세션이 닫힌 경우 알림. 이 알람을 받을 경우 다시 로그인하여 재시작 필요 
     * @param user 세션이 닫힌 유저에이전트
     * @param resultCode 세션이 닫힌 이유
     * @param paload 추가 정보. 서버 구현에 따라 사용하지 않을 수 있음
     */
    OnSessionCloseNoti?(user: UserAgent, resultCode: ResultCodeSessionClose, payload: Payload): void { }
}
userAgent.AddListener(new UserListener());
```

