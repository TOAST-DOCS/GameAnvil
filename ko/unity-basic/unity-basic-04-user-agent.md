## Game > GameAnvil > Unity 기초 개발 가이드 > UserAgent

## UserAgent

UserAgent는 GameAnvil 서버의 GameNode와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 방 관리 등 기본 기능을 제공하며, 직접 정의한 프로토콜을 기반으로 클라이언트는 자신의 유저 객체를 통해 다른 객체들과 메시지를 주고 받으며 여러 가지 컨텐츠를 구현할 수 있습니다. 

GameAnvilConnector는 기본적으로 QuickConnect 과정에서 생성된 하나의 UserAgent를 제공합니다. GameAnvilConnector.getUserAgent()를 통해서 얻을 수 있습니다.

### 로그인

로그인은 클라언트가 서버에 접속한 후 GameNode에 자신의 유저 객체를 만드는 과정이라고 정의할 수 있습니다. 

로그인은 QuickConnect에서 한 번에 처리되기 때문에 설명을 생략합니다. 자세한 내용은 [Unity 심화 개발 가이드 > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md) 를 참고해주세요.

### 로그아웃

GameAnvilConnector에서는 Logout 하면 자동으로 접속 종료까지 처리되며, 로그아웃에 대한 더 자세한 설명은 [Unity 심화 개발 가이드 > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md) 를 참고해주세요.

```c#
/// <summary>
/// 현재 서비스에서 로그아웃 
/// </summary>
/// <param name="onLogout">로그아웃 대리자</param>
userAgent.Logout((UserAgent user, Defines.ResultCodeLogout result, bool force, Payload payload) => {
    /// <param name="userAgent">Logout()을 요청한 유저 에이전트</param>
    /// <param name="result">Logout() 결과</param>
    /// <param name="force">서버에 의한 강제 여부</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    if(result == Defines.ResultCodeLogout.LOGOUT_SUCCESS){
        // 성공
    } else {
        // 실패
    }
});
```

### 방 생성, 입장, 퇴장

2명 이상의 유저는 방을 통해 동기화된 메시지 흐름을 만들 수 있습니다. 즉, 유저들의 요청은 방 안에서 모두 순서가 보장됩니다. 물론 1명의 유저를 위한 방 생성도 컨텐츠에 따라서 의미를 가질 수도 있습니다. 방을 어떻게 사용할지는 어디까지나 엔진 사용자의 몫입니다. 

CreateRoom()을 호출하여 방을 생성하고 그 방으로 입장합니다.

```c#
/// <summary>
/// 임의의 방 생성 후 해당 방에 입장
/// </summary>
/// <param name="roomName">생성할 방의 이름</param>
/// <param name="roomType">생성할 방의 타입</param>
/// <param name="matchingGroup">매칭할 방의 매칭 그룹</param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="onCreateRoom">결과를 받을 대리자</param>
userAgent.CreateRoom(roomName, roomType, matchingGroup, payload, (UserAgent user, Defines.ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) => {
    /// <param name="userAgent">CreateRoom()을 요청한 유저 에이전트</param>
    /// <param name="result"> CreateRoom() 요청 요청 결과</param>
    /// <param name="roomName">생성된 방의 이름</param>
    /// <param name="roomId">생성된 방의 아이디</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
	if(result == Defines.ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
        // 성공
    } else {
        // 실패
    }
});
```
<br>

JoinRoom()을 호출하여 이미 생성된 방에 입장합니다. 

``` c#
/// <summary>
/// 지정한 아이디의 방에 입장<para></para>
/// 지정한 아이디의 방이 없을 경우 실패
/// </summary>
/// <param name="roomType">입장할 방의 타입</param>
/// <param name="roomId">입장하고자 하는 방의 아이디</param>
/// <param name="matchingUserCategory">룸매칭을 사용중인 방에 입장할 경우 사용</param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="onJoinRoom">결과를 받을 대리자</param>
userAgent.JoinRoom(roomType, roomId, matchingUserCategory, payload, (UserAgent user, Defines.ResultCodeJoinRoom result, int id, string roomName, Payload payload) => {
    /// <param name="result">JoinRoom() 요청 결과</param>
    /// <param name="roomId">입장한 방의 아이디</param>
    /// <param name="roomName">입장한 방의 이름</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
	if(result == Defines.ResultCodeJoinRoom.JOIN_ROOM_SUCCESS){
        // 성공
    } else {
        // 실패
    }
});
```
<br>

LeaveRoom()을 호출하여 입장한 방에서 퇴장할 수 있습니다.

``` c#
/// <summary>
/// 유저 에이전트가 속한 방에서 퇴장
/// </summary>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="onLeaveRoom">결과를 받을 대리자</param>
userAgent.LeaveRoom(payload, (UserAgent user, Defines.ResultCodeLeaveRoom result, bool force, int roomId, Payload payload) => {
    /// <param name="userAgent">LeaveRoom()을 요청한 유저 에이전트</param>
    /// <param name="result">LeaveRoom() 요청 결과</param>
    /// <param name="force">강제 퇴장 여부</param>
    /// <param name="roomId">퇴장한 방의 아이디</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
	if(result == Defines.ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS){
        // 성공
    } else {
        // 실패
    }
});
```
<br>

NamedRoom()을 호출하여 지정한 이름의 방에 입장할 수 있습니다. 지정한 이름의 방이 없을 경우에는 방을 생성한 후 그 방으로 입장합니다.

```c#
/// <summary>
/// 지정한 이름의 방에 입장<para></para>
/// 지정한 이름의 방이 없을 경우 지정한 이름의 방을 생성하고 해당 방에 입장
/// </summary>
/// <param name="roomType">입장할 방의 타입</param>
/// <param name="roomName">입장하고자 하는 방의 이름</param>
/// <param name="isParty">파티 여부</param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="onNamedRoom">결과를 받을 대리자</param>
userAgent.NamedRoom(roomType, roomName, isParty, payload, (UserAgent user, Defines.ResultCodeNamedRoom result, int roomId, string roomName, bool created, Payload payload) => {
    /// <param name="userAgent">NameRoom()을 요청한 유저 에이전트</param>
    /// <param name="result">NameRoom() 요청 결과</param>
    /// <param name="roomName">방 이름</param>
    /// <param name="roomId">입장한 방의 아이디</param>
    /// <param name="created">입장한 방의 신설 여부</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    if(result == Defines.ResultCodeNamedRoom.NAMED_ROOM_SUCCESS){
        // 성공
    } else {
        // 실패
    }
});
```

### 매치 메이킹

GameAnvil은 크게 두 가지 매치 메이킹을 제공합니다. 하나는 방 단위의 매칭을 수행하는 룸 매치 메이킹이고, 다른 하나는 유저 단위의 매칭을 수행하는 유저 매치 메이킹입니다.

#### 룸 매치 메이킹

룸 매치 메이킹은 조건에 맞는 방으로 유저를 입장시켜주는 방식입니다. 룸 매치 메이킹 요청 시 조건에 맞는 방이 있으면 해당 방으로 바로 입장 시켜주고 조건에 맞는 방이 없다면 새로운 방을 생성하여 입장 시켜줍니다.

MatchRoom()을 호출하여 룸 매치 메이킹을 요청할 수 있습니다.

```c#
/// <summary>
/// 룸 매칭 요청<para></para>
/// 방이 없을 경우, 임의의 방을 생성하고 해당 방에 입장하거나 매칭 실패 처리
/// </summary>
/// <param name="roomType">입장할 방의 타입</param>
/// <param name="matchingGroup">매칭할 방의 매칭 그룹</param>
/// <param name="matchingUserCategory">매칭시 사용할 매칭 유저 카테고리</param>
/// <param name="isCreateRoomIfNotJoinRoom">방 생성 허용</param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="onMatchRoom">결과를 받을 대리자</param>
userAgent.MatchRoom(roomType, matchingGroup, matchingUserCategory, isCreateRoomIfNotJoinRoom, isMoveRoomIfJoinedRoom, payload, leaveRoomPayload, (UserAgent user, Defines.ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload) => {
    /// <param name="userAgent">MatchRoom()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchRoom() 요청 결과</param>
    /// <param name="resultCode">사용자 결과 코드</param>
    /// <param name="roomId">매치된 방의 아이디</param>
    /// <param name="roomName">매치된 방의 이름</param>
    /// <param name="created">매치된 방의 신설 여부</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    if(result == Defines.ResultCodeMatchRoom.MATCH_ROOM_SUCCESS){
        // 매치 메이킹 취소 성공
    } else {
        // 매치 메이킹 취소 실패
    }
});
```

#### 유저 매치 메이킹

유저 메치 메이킹은 유저풀을 만들고 그 안에서 조건에 맞는 유저들을 찾아 새로 생성한 방으로 입장 시켜주는 방식입니다. 유저풀에 조건에 맞는 유저의 수가 모자랄 경우 매치 메이킹이 완료될 때까지 시간이 걸릴 수 있고, 시간 내에 매치 메이킹이 완료되지 않으면 타임아웃이 되어 매칭이 취소될 수 있습니다. 

MatchUserStart()를 호출하여 유저 매치 메이킹을 시작할 수 있습니다. 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패할 수 있습니다. 

```c#
/// <summary>
/// 유저 매칭을 요청<para></para>
/// 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패<para></para>
/// 매칭이 성공한 경우 OnMatchUserDone()을 통해 알림<para></para>
/// </summary>
/// <param name="roomType">매치를 요청할 방의 타입</param>
/// <param name="matchingGroup">방 생성 시 사용될 매칭그룹</param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="onMatchUserStart">결과를 받을 대리자</param>
userAgent.MatchUserStart(roomType, matchingGroup, payload, (UserAgent user, Defines.ResultCodeMatchUserStart result, Payload payload) => {
    /// <param name="userAgent">MatchUserStart()를 요청한 유저 에이전트</param>
    /// <param name="result">MatchUserStart() 요청 결과</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
	if(result == Defines.ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS){
        // 매치 메이킹 요청 성공
    } else {
        // 매치 메이킹 요청 실패
    }
});
```
<br>

매칭이 성공한 경우 onMatchUserDoneListeners 또는 IUserListener.OnMatchUserDone을 통해 알림을 받을 수 있고, 시간 안에 매칭이 성공하지 못한 경우 onMatchUserTimeoutListeners 또는 IUserListener.OnMatchUserTimeout을 통해 알림을 받을 수 있습니다.

```c#
/// <summary>
/// 유저 매칭 결과를 받을 대리자
/// </summary>
userAgent.onMatchUserDoneListeners += (UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload) => {
    /// <param name="userAgent">MatchUserStart()나 MatchPartyStart()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchUserStart()나 MatchPartyStart() 요청 결과</param>
    /// <param name="created">방 신설 여부</param>
    /// <param name="roomId">매칭된 방의 아이디</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
};

/// <summary>
/// 유저 매칭 타임아웃 알림 대리자
/// </summary>
userAgent.onMatchUserTimeoutListeners  += (UserAgent userAgent) => {
    /// <param name="userAgent">MatchUserStart()나 MatchPartyStart()을 요청한 유저 에이전트</param>
};
```
<br>

MatchUserCancel()을 호출하여 유저 매치 메이킹을 취소할 수 있습니다. 유저 매치 요청 중이 아닌 경우, 이미 유저 매치 메이킹이 성공했거나 Timeout이 발생했으면 취소 요청이 실패할 수 있습니다. 

```c#
/// <summary>
/// 유저 매칭 요청을 취소<para></para>
/// 매치 요청 중이 아닌 경우, 이미 매칭이 성공했거나 타임아웃이 발생했을 경우 실패<para></para>
/// </summary>
/// <param name="roomType">매치를 요청한 바의 타입</param>
/// <param name="onMatchUserCancel">결과를 받을 대리자</param>
userAgent.MatchUserCancel(Constants.RoomType, (UserAgent user, Defines.ResultCodeMatchUserCancel result) => {
    /// <param name="userAgent">MatchUserCancel()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchUserCancel() 요청 결과</param>
	if(result == Defines.ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS){
        // 매치 메이킹 취소 성공
    } else {
        // 매치 메이킹 취소 실패
    }
});
```

이외에도 파티 매치 메이킹, 채널, 리스너 등 유저 에이전트의 다양한 기능들에 대한 설명이 [Unity 심화 개발 가이드 > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md) 에 소개되어 있습니다.