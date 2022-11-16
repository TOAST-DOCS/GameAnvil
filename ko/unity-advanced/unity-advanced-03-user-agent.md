## Game > GameAnvil > Unity 심화 개발 가이드 > UserAgent

## UserAgent

UserAgent는 GameAnvil 서버의 GameNode와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 방 관리 등 기본 기능을 제공하며, 직접 정의한 프로토콜을 기반으로 클라이언트는 자신의 유저 객체를 통해 다른 객체들과 메시지를 주고 받으며 여러 가지 컨텐츠를 구현할 수 있습니다. 

UserAgent를 사용하기 위해서는 Connector.CreateUserAgent() 함수를 이용해 새로운 UserAgent를 생성해야 합니다. ServiceName과 SubId로 구분되는 여러 개의 UserAgent를 생성할 수 있습니다. 생성된 UserAgent는 Connector에서 내부적으로 관리되어 Connector.GetUserAgent() 함수를 이용해 다시 사용할 수 있습니다. 

```c#
UserAgent userAgent = ConnectHandler.getInstance().GetUserAgent(serviceName, subID);
if (userAgent == null) {
    userAgent = ConnectHandler.getInstance().CreateUserAgent(serviceName, subID);
}
```

GameAnvil 서버는 여러 개의 서비스를 동시에 운영할 수 있으며, 하나의 UserAgent는 하나의 서비스에 로그인하여 서로 독립적으로 동작하게 됩니다. 즉, 여러 개의 UserAgent를 만들어 서로 다른 서비스에 로그인하여 동시에 사용이 가능합니다. SubId를 다르게 한다면 같은 서비스에 여러 개의 UserAgent를 동시에 로그인하여 사용하는 것도 가능합니다. 

### 로그인/로그아웃

로그인은 클라언트가 서버에 접속한 후 GameNode에 자신의 유저 객체를 만드는 과정이라고 정의할 수 있습니다. 로그아웃은 로그인의 반대 개념입니다. 다시 말해, GameNode 상에서 자신의 유저 객체를 제거하는 과정입니다. 

로그인 시 어떤 UserType으로 어떤 채널에 로그인을 할지 입력해줘야 합니다. 추가 정보가 필요하다면 Payload에 담아 보낼 수 있습니다. 

```c#
/// <summary>
/// 서비스에 로그인
/// </summary>
/// <param name="userType">유저의 타입 </param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="channelId">로그인할 채널의 아이디</param>
/// <param name="onLogin">결과를 받을 대리자</param>
userAgent.Login(userType, channelId, payload, (UserAgent user, Defines.ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => {
    /// <param name="userAgent">Login()을 요청한 유저 에이전트</param>
    /// <param name="result">Login() 요청 결과</param>
    /// <param name="loginInfo">로그인 정보</param>
    if(result == Defines.ResultCodeLogin.LOGIN_SUCCESS){
        // 성공
    } else {
        // 실패
    }
});

/// <summary>
/// 현재 서비스에서 로그아웃 
/// </summary>
/// <param name="onLogout">로그아웃 다리자</param>
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
...
```

### 방 생성, 입장, 퇴장

[Unity 기초 개발 가이드 > UserAgent](../unity-basic/unity-basic-04-user-agent.md) 의 방 생성, 입장, 퇴장 내용과 동일합니다.

### 매치 메이킹

GameAnvil은 두 가지 매치 메이킹을 제공합니다. 하나는 방 단위의 매칭을 수행하는 룸 매치 메이킹이고, 다른 하나는 유저 단위의 매칭을 수행하는 유저 매치 메이킹 입니다. 자세한 내용은 [Unity 기초 개발 가이드 > UserAgent](../unity-basic/unity-basic-04-user-agent.md)의 매치 메이킹 부분을 참고해주세요.

#### 파티 매치 메이킹

파티 매치 메이킹은 유저 매치 메이킹의 특수한 형태로, 2명 이상의 유저가 한 파티로 묶여 유저 풀에 등록이 되고, 조건이 맞는 다른 유저들을 찾아 새로 생성한 방으로 함께 입장 시켜주는 방식입니다. 파티로 묶은 유저들은 항상 같은 방으로 입장합니다. 파티 외에 같이 매칭 유저들은 서버의 매치 메이커 구현에 따라 또 다른 파티일 수도 있고, 개인일 수도 있습니다.

파티 매치 메이킹을 하기 위해서는 먼저 NamedRoom()을 호출해야 합니다. NamedRoom() 호출 시 isParty 매개변수를 true로 넘겨주면, 해당 NamedRoom이 파티의 역할을 합니다. NamedRoom에 파티 유저를 모두 모으고 나서 파티 매치 메이킹을 시작합니다.

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
<br>

MatchPartyStart()를 호출하여 파티 매치 메이킹을 시작할 수 있습니다. 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패할 수 있습니다. 

```c#
/// <summary>
/// 파티 매칭을 요청<para></para>
/// 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패<para></para>
/// 매칭이 성공한 경우 OnMatchUserDone()을 통해 알림
/// </summary>
/// <param name="roomType">매치를 요청할 방의 타입</param>
/// <param name="matchingGroup">방 생성 시 사용될 매칭그룹</param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="onMatchPartyStart">결과를 받을 대리자</param>
userAgent.MatchPartyStart(Constants.RoomType, Constants.ChannelId1, customPayload, (UserAgent user, Defines.ResultCodeMatchPartyStart result, Payload payload) => {
    /// <param name="userAgent">MatchPartyStart()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchPartyStart() 요청 결과</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    if(result == Defines.ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS){
        // 성공
    } else {
        // 실패
    }
});
```
<br>

파티 매칭이 성공한 경우 유저 매치 메이킹에서 사용했던 onMatchUserDoneListeners 또는 IUserListener.OnMatchUserDone을 통해 알림을 받을 수 있고, 시간 안에 매칭이 성공하지 못한 경우 onMatchUserTimeoutListeners 또는 IUserListener.OnMatchUserTimeout을 통해 알림을 받을 수 있습니다.

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

MatchPartyCancel()을 호출하여 파티 매치 메이킹을 취소할 수 있습니다. 파티 매치 요청 중이 아닌 경우, 이미 파티 매치 메이킹이 성공했거나 Timeout이 발생했으면 취소 요청이 실패할 수 있습니다. 

```c#
/// <summary>
/// 파티 매칭 요청을 취소<para></para>
/// 매치 요청 중이 아닌경우, 이미 매칭이 성공했거나 타임아웃이 발생했을 경우 실패
/// </summary>
/// <param name="roomType">매치를 요청한 방의 타입</param>
/// <param name="onMatchPartyCancel">결과를 받을 대리자</param>
userAgent.MatchPartyCancel(Constants.RoomType, (UserAgent user, Defines.ResultCodeMatchPartyCancel result) => {
    /// <param name="userAgent">MatchPartyCancel()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchPartyCancel() 요청 결과</param>
    if(result == Defines.ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS){
        // 성공
    } else {
        // 실패
    }
});
```

#### 채널 이동

경우에 따라서 매치 메이킹의 결과로 채널 이동이 발생할 수 있습니다. 채널 이동이 되었을 경우 onMoveChannelListeners 또는 IUserListener.OnMoveChannel을 통해 알림을 받을 수 있습니다.

```c#
/// <summary>
/// 채널 이동 요청 결과 또는 서버에서 강제로 수행한 채널 이동에 대한 알림을 받을 대리자
/// </summary>
userAgent.onMoveChannelListeners  += (UserAgent userAgent, GameAnvil.Defines.ResultCodeMoveChannel result, bool force, string channelId, Payload payload) => {
    /// <param name="userAgent">MoveChannel()한 유저 에이전트</param>
    /// <param name="result">MoveChannel() 결과 코드</param>
    /// <param name="force">서버에서 강제로 채널을 이동했는지 여부</param>
    /// <param name="channelId">이동한 채널의 아이디</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
};
```
<br>

MoveChannel()을 호출하여 서비스 내의 다른 채널로 이동할 수 있습니다. 

```c#
/// <summary>
/// 지정한 채널로 이동
/// </summary>
/// <param name="channelId">이동할 채널의 아이디</param>
/// <param name="payload">서버에 전달할 추가 정보</param>
/// <param name="onMoveChannel">결과를 받을 대리자</param>
userAgent.MoveChannel(channelId, usePayload ? customPayload : null, (UserAgent user, Defines.ResultCodeMoveChannel result, bool force, string channelID, Payload payload) => {
    /// <param name="userAgent">MoveChannel()한 유저 에이전트</param>
    /// <param name="result">MoveChannel() 결과 코드</param>
    /// <param name="force">서버에서 강제로 채널을 이동했는지 여부</param>
    /// <param name="channelId">이동한 채널의 아이디</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
	if(result == ResultCodeMoveChannel.ALL_CHANNEL_INFO_SUCCESS){
		// 모든 채널 정보 요청 성공
	} else {
		// 모든 채널 정보 요청 실패
	}
});
```

### 채널 정보

GameAnvil은 설정에서 자유롭게 채널 구성을 변경할 수 있습니다. 이런 채널 구성은 서버와 클라이언트간에 미리 약속하여 고정된 형태로 사용할 수도 있지만, 상황에 따라 유연하게 변경하여 사용할 수도 있습니다. UserAgent에서는 이렇게 변경된 채널 정보를 얻어오거나 채널을 이동할 수 있도록 몇 가지 함수를 제공합니다. 

| 함수 | 설명 |
| --- | --- |
| GetChannelCountInfo() | 특정 채널의 카운트 정보(유저와 방 개수) 요청 | 
| GetChannelInfo() | 특정 채널의 정보(사용자 정의) 요청 |
| GetAllChannelCountInfo() | 특정 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수) 요청 |
| GetAllChannelInfo() | 특정 서비스의 모든 채널에 대한 정보(사용자 정의) 요청 |

아래에서 코드를 통해 더욱 자세하게 살펴보겠습니다.

GetChannelCountInfo()는 특정 채널의 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 

```c#
/// <summary>
/// 접속중인 채널의 유저와 방의 개수를 요청<para></para>
/// 서버에서 지원할 경우 사용할 수 있음
/// </summary>
/// <param name="onChannelCountInfo">결과를 전달 받을 대리자</param>
userAgent.GetChannelCountInfo((ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="userAgent">GetChannelCountInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GetChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// 채널 카운트 정보 요청 성공
	} else {
		// 채널 카운트 정보 요청 실패
	}
});

/// <summary>
/// 특정 채널의 유저와 방의 개수를 요청<para></para>
/// 서버에서 지원할 경우 사용할 수 있음
/// </summary>
/// <param name="serviceName">채널 정보를 요청할 서비스의 이름</param>
/// <param name="channelId">채널 정보를 요청할 체널의 아이디</param>
/// <param name="onChannelCountInfo">결과를 전달 받을 대리자</param>
userAgent.GetChannelCountInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="userAgent">GetChannelCountInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GetChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// 채널 카운트 정보 요청 성공
	} else {
		// 채널 카운트 정보 요청 실패
	}
});
```
<br>

GetChannelInfo()는 채널의 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 

```c#
/// <summary>
/// 접속중인 채널 정보를 요청<para></para>
/// 서버에서 지원할 경우 사용할 수 있음
/// </summary>
/// <param name="onChannelInfo">결과를 전달 받을 대리자</param>
userAgent.GetChannelInfo((ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="userAgent">GetChannelInfo()를 용청한 유저 에이전트</param>
    /// <param name="result">GetChannelInfo() 요청 결과</param>
    /// <param name="channelInfo">서버에서 받은 채널 정보</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// 채널 정보 요청 성공
	} else {
		// 채널 정보 요청 실패
	}
});

/// <summary>
/// 특정 채널의 정보를 요청<para></para>
/// 서버에서 지원할 경우 사용할 수 있음
/// </summary>
/// <param name="serviceName">채널 정보를 요청할 서비스의 이름</param>
/// <param name="channelId">채널 정보를 요청할 체널의 아이디</param>
/// <param name="onChannelInfo">결과를 전달 받을 대리자</param>
userAgent.GetChannelInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="userAgent">GetChannelInfo()를 용청한 유저 에이전트</param>
    /// <param name="result">GetChannelInfo() 요청 결과</param>
    /// <param name="channelInfo">서버에서 받은 채널 정보</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// 채널 정보 요청 성공
	} else {
		// 채널 정보 요청 실패
	}
});
```
<br>

GetAllChannelCountInfo()는 서비스의 모든 채널에 대한 카운트 정보(유저와 방 개수)를 요청하여 받아올 수 있습니다. 매개변수로 서비스 이름과 응답을 처리할 콜백을 넘겨줍니다. 

```c#
/// <summary>
/// 접속중인 서비스에 있는 모든 채널의 유저와 방의 개수를 요청<para></para>
/// 서버에서 지원할 경우 사용할 수 있음
/// </summary>
/// <param name="onAllChannelCountInfo">결과를 전달 받을 대리자</param>
userAgent.GetAllChannelCountInfo((ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="userAgent">GetAllChannelCountInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GetAllChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보 목록</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// 모든 채널 카운트 정보 요청 성공
	} else {
		// 모든 채널 카운트 정보 요청 실패
	}
});

/// <summary>
/// 특정 서비스에 있는 모든 채널의 유저와 방의 개수를 요청<para></para>
/// 서버에서 지원할 경우 사용할 수 있음
/// </summary>
/// <param name="serviceName">채널 정보를 요청할 서비스의 이름</param>
/// <param name="onAllChannelCountInfo">결과를 전달 받을 대리자</param>
userAgent.GetAllChannelCountInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="userAgent">GetAllChannelCountInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GetAllChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보 목록</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// 모든 채널 카운트 정보 요청 성공
	} else {
		// 모든 채널 카운트 정보 요청 실패
	}
});
```
<br>

GetAllChannelInfo()는 서비스의 모든 채널에 대한 정보(사용자 정의)를 요청하여 받아올 수 있습니다. 매개변수로 서비스 이름과 응답을 처리할 콜백을 넘겨줍니다. 

```c#
/// <summary>
/// 특정 서비스의 모든 채널정보 요청<para></para>
/// 서버에서 지원할 경우 사용할 수 있음
/// </summary>
/// <param name="serviceName">채널 정보를 요청할 서비스의 이름</param>
/// <param name="onAllChannelInfo">결과를 전달 받을 대리자</param>
userAgent.GetAllChannelInfo((ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="userAgent">GeAllChannelInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GeAllChannelInfo() 정보 요청 결과</param>
    /// <param name="channelInfo">서버에서 받은 채널 정보 목록</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// 모든 채널 정보 요청 성공
	} else {
		// 모든 채널 정보 요청 실패
	}
});

/// <summary>
/// 특정 서비스의 모든 채널 정보 요청<para></para>
/// 서버에서 지원할 경우 사용할 수 있음
/// </summary>
/// <param name="serviceName">채널 정보를 요청할 서비스의 이름</param>
/// <param name="onAllChannelInfo">결과를 전달 받을 대리자</param>
userAgent.GetAllChannelInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="userAgent">GeAllChannelInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GeAllChannelInfo() 정보 요청 결과</param>
    /// <param name="channelInfo">서버에서 받은 채널 정보 목록</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// 모든 채널 정보 요청 성공
	} else {
		// 모든 채널 정보 요청 실패
	}
});
```

### Listener

UserAgent에서 모든 요청에 대한 결과 또는 서버로부터의 알림을 전달받는 방법은 크게 두 가지입니다.
하나는 UserAgent에 정의되어 있는 delegate에 함수를 추가하는 방법입니다. 다른 하나는 IUserListener 인터페이스를 구현한 리스너를 등록하는 방법입니다.

UserAgent는 모든 동작의 결과 또는 알림을 받을 수 있도록 각각의 delegate를 맴버로 가지고 있습니다. 이 delegate에 함수를 등록하면 앞서 설명한 API에 콜백 매개변수를 생략하고 호출했을 경우 또는 서버에서 알림을 보냈을 경우 등록한 함수로 응답을 받을 수 있습니다.

```c#
/// <summary>
/// 로그인 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">Login()을 요청한 유저 에이전트</param>
/// <param name="result">Login() 요청 결과</param>
/// <param name="loginInfo">로그인 정보</param>
public Interface.DelUserOnLogin onLoginListeners;

/// <summary>
/// 룸 매치 요청 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">MatchRoom()을 요청한 유저 에이전트</param>
/// <param name="result">MatchRoom() 요청 결과</param>
/// <param name="resultCode">사용자 결과 코드</param>
/// <param name="roomId">매치된 방의 아이디</param>
/// <param name="roomName">매치된 방의 이름</param>
/// <param name="created">매치된 방의 신설 여부</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnMatchRoom onMatchRoomListeners;

/// <summary>
/// 방 생성 요청 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">CreateRoom()을 요청한 유저 에이전트</param>
/// <param name="result"> CreateRoom() 요청 요청 결과</param>
/// <param name="roomName">생성된 방의 이름</param>
/// <param name="roomId">생성된 방의 아이디</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnCreateRoom onCreateRoomListeners;

/// <summary>
/// 방 입장 요청 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">JoinRoom()을 요청한 유저 에이전트</param>
/// <param name="result">JoinRoom() 요청 결과</param>
/// <param name="roomId">입장한 방의 아이디</param>
/// <param name="roomName">입장한 방의 이름</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnJoinRoom onJoinRoomListeners;

/// <summary>
/// 이름 있는 방 요청 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">NameRoom()을 요청한 유저 에이전트</param>
/// <param name="result">NameRoom() 요청 결과</param>
/// <param name="roomName">방 이름</param>
/// <param name="roomId">입장한 방의 아이디</param>
/// <param name="created">입장한 방의 신설 여부</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnNamedRoom onNamedRoomListeners;

/// <summary>
/// 파티 매칭 요쳥 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">MatchPartyStart()을 요청한 유저 에이전트</param>
/// <param name="result">MatchPartyStart() 요청 결과</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnMatchPartyStart onMatchPartyStartListeners;

/// <summary>
/// 파티 매칭 요청 취소 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">MatchPartyCancel()을 요청한 유저 에이전트</param>
/// <param name="result">MatchPartyCancel() 요청 결과</param>
public Interface.DelUserOnMatchPartyCancel onMatchPartyCancelListeners;

/// <summary>
/// 유저 매칭 요청 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">MatchUserStart()를 요청한 유저 에이전트</param>
/// <param name="result">MatchUserStart() 요청 결과</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnMatchUserStart onMatchUserStartListeners;

/// <summary>
/// 유저 매칭 요청 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">MatchUserStart()나 MatchPartyStart()을 요청한 유저 에이전트</param>
/// <param name="result">MatchUserStart()나 MatchPartyStart() 요청 결과</param>
/// <param name="created">방 신설 여부</param>
/// <param name="roomId">매칭된 방의 아이디</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnMatchUserDone onMatchUserDoneListeners;

/// <summary>
/// 유저 매칭 타임아웃 알림 대리자
/// </summary>
/// <param name="userAgent">MatchUserStart()나 MatchPartyStart()을 요청한 유저 에이전트</param>
public Interface.DelUserOnMatchUserTimeout onMatchUserTimeoutListeners;

/// <summary>
/// 유저 매칭 요청 취소 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">MatchUserCancel()을 요청한 유저 에이전트</param>
/// <param name="result">MatchUserCancel() 요청 결과</param>
public Interface.DelUserOnMatchUserCancel onMatchUserCancelListeners;

/// <summary>
/// 방 퇴장 또는 강제 퇴장 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">LeaveRoom()을 요청한 유저 에이전트</param>
/// <param name="result">LeaveRoom() 요청 결과</param>
/// <param name="force">강제 퇴장 여부</param>
/// <param name="roomId">퇴장한 방의 아이디</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnLeaveRoom onLeaveRoomListeners;

/// <summary>
/// 채널 정보 요청 결과 또는 서버에서 강제로 수행한 채널 이동에 대한 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">GetChannelInfo()를 용청한 유저 에이전트</param>
/// <param name="result">GetChannelInfo() 요청 결과</param>
/// <param name="channelInfo">서버에서 받은 채널 정보</param>
public Interface.DelUserOnChannelInfo onChannelInfoListeners;

/// <summary>
/// 모든 채널 정보 목록 요청 결과 또는 서버에서 강제로 수행한 채널 이동에 대한 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">GeAllChannelInfo()를 요청한 유저 에이전트</param>
/// <param name="result">GeAllChannelInfo() 정보 요청 결과</param>
/// <param name="channelInfo">서버에서 받은 채널 정보 목록</param>
public Interface.DelUserOnAllChannelInfo onAllChannelInfoListeners;

/// <summary>
/// 채널 정보 요청 결과 또는 서버에서 강제로 수행한 채널 이동에 대한 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">GetChannelCountInfo()를 요청한 유저 에이전트</param>
/// <param name="result">GetChannelCountInfo() 요청 결과</param>
/// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보</param>
public Interface.DelUserOnChannelCountInfo onChannelCountInfoListeners;

/// <summary>
/// 모든 채널 정보 요청 결과 또는 서버에서 강제로 수행한 채널 이동에 대한 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">GetAllChannelCountInfo()를 요청한 유저 에이전트</param>
/// <param name="result">GetAllChannelCountInfo() 요청 결과</param>
/// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보 목록</param>
public Interface.DelUserOnAllChannelCountInfo onAllChannelCountInfoListeners;

/// <summary>
/// 채널 이동 요청 결과 또는 서버에서 강제로 수행한 채널 이동에 대한 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">MoveChannel()한 유저 에이전트</param>
/// <param name="result">MoveChannel() 결과 코드</param>
/// <param name="force">서버에서 강제로 채널을 이동했는지 여부</param>
/// <param name="channelId">이동한 채널의 아이디</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnMoveChannel onMoveChannelListeners;

/// <summary>
/// 스냅샷 요청 결과를 받을 대리자
/// </summary>
/// <param name="userAgent">Snapshot()한 유저 에이전트</param>
/// <param name="result">Snapshot() 요청 결과</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnSnapshot onSnapshotListeners;

/// <summary>
/// 기본 기능 사용 중 오류 발생시 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">오류가 발생한 유저 에이전트</param>
/// <param name="errorCode">오류 코드</param>
/// <param name="command">오류 발생 기능</param>
public Interface.DelUserOnErrorCommand onErrorCommandListeners;

/// <summary>
/// 패킷 전송 오류 발생 시 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">오류가 발생한 유저 에이전트</param>
/// <param name="errorCode">오류 코드</param>
/// <param name="command">오류가 발생한 메시지</param>
public Interface.DelUserOnErrorCustomCommand onErrorCustomCommandListeners;

/// <summary>
/// 로그아웃 요청 결과 또는 강제 로그아웃 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">Logout()을 요청한 유저 에이전트</param>
/// <param name="result">Logout() 결과</param>
/// <param name="force">서버에 의한 강제 여부</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnLogout onLogoutListeners;

/// <summary>
/// 고지 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">Notice()를 받은 유저 에이전트</param>
/// <param name="message">고지 메시지</param>
public Interface.DelUserOnNotice onNoticeListeners;

/// <summary>
/// 킥아웃 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">킥아웃 된 유저 에이전트</param>
/// <param name="message">어드민으로부터 받은 메시지</param>
public Interface.DelUserOnAdminKickout onAdminKickoutListeners;

/// <summary>
/// 서버의 세션이 닫힌 경우 알림을 받을 대리자
/// </summary>
/// <param name="userAgent">세션이 닫힌 유저 에이전트</param>
/// <param name="result">세션이 닫힌 이유</param>
/// <param name="payload">서버에서 받은 추가 정보</param>
public Interface.DelUserOnSessionClose onSessionCloseListeners;
```
<br>

IUserListener는 UserAgent의 모든 동작의 결과 또는 알림을 정의한 인터페이스입니다. 이 인터페이스를 구현한 리스너를 UserAgent.AddUserListener()로 등록하면 등록한 리스너로 응답을 받을 수 있습니다.

```c#
class UserListener : IUserListener{
    /// <summary>
    /// Login() 요청 결과
    /// </summary>
    /// <param name="userAgent">Login()을 요청한 유저 에이전트</param>
    /// <param name="result">Login() 요청 결과</param>
    /// <param name="loginInfo">로그인 정보</param>
    void OnLogin(UserAgent userAgent, GameAnvil.Defines.ResultCodeLogin result, UserAgent.LoginInfo loginInfo);
    
    /// <summary>
    /// MatchRoom() 요청 결과
    /// </summary>
    /// <param name="userAgent">MatchRoom()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchRoom() 요청 결과</param>
    /// <param name="resultCode">사용자 결과 코드</param>
    /// <param name="roomId">매치된 방의 아이디</param>
    /// <param name="roomName">매치된 방의 이름</param>
    /// <param name="created">매치된 방의 신설 여부</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnMatchRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload);
    
    /// <summary>
    /// CreateRoom() 요청 결과
    /// </summary>
    /// <param name="userAgent">CreateRoom()을 요청한 유저 에이전트</param>
    /// <param name="result">CreateRoom() 요청 결과</param>
    /// <param name="roomId">생성된 방의 아이디</param>
    /// <param name="roomName">생성된 방의 이름</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnCreateRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeCreateRoom result, int roomId, string roomName, Payload payload);
    
    /// <summary>
    /// JoinRoom() 요청 결과
    /// </summary>
    /// <param name="userAgent">JoinRoom()을 요청한 유저 에이전트</param>
    /// <param name="result">JoinRoom() 요청 결과</param>
    /// <param name="roomId">입장한 방의 아이디</param>
    /// <param name="roomName">입장한 방의 이름</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnJoinRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeJoinRoom result, int roomId, string roomName, Payload payload);
    
    /// <summary>
    /// NameRoom() 요청 결과
    /// </summary>
    /// <param name="userAgent">NameRoom()을 요청한 유저 에이전트</param>
    /// <param name="result">NameRoom() 요청 결과</param>
    /// <param name="roomName">방 이름</param>
    /// <param name="roomId">입장한 방의 아이디</param>
    /// <param name="created">입장한 방의 신설 여부</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnNamedRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeNamedRoom result, int roomId, string roomName, bool created, Payload payload);
    
    /// <summary>
    /// MatchUserStart() 결과
    /// </summary>
    /// <param name="userAgent">MatchUserStart()를 요청한 유저 에이전트</param>
    /// <param name="result">MatchUserStart() 요청 결과</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnMatchUserStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserStart result, Payload payload);
    
    /// <summary>
    /// MatchUserStart()나 MatchPartyStart() 요청 결과
    /// </summary>
    /// <param name="userAgent">MatchUserStart()나 MatchPartyStart()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchUserStart()나 MatchPartyStart() 요청 결과</param>
    /// <param name="created">방 신설 여부</param>
    /// <param name="roomId">매칭된 방의 아이디</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnMatchUserDone(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload);
    
    /// <summary>
    /// MatchUserStart()나 MatchPartyStart() 타임아웃
    /// </summary>
    /// <param name="userAgent">MatchUserStart()나 MatchPartyStart()을 요청한 유저 에이전트</param>
    void OnMatchUserTimeout(UserAgent userAgent);
    
    /// <summary>
    /// MatchUserCancel() 결과
    /// </summary>
    /// <param name="userAgent">MatchUserCancel()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchUserCancel() 요청 결과</param>
    void OnMatchUserCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserCancel result);
    
    /// <summary>
    /// MatchPartyStart() 요청 결과
    /// </summary>
    /// <param name="userAgent">MatchPartyStart()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchPartyStart() 요청 결과</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnMatchPartyStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchPartyStart result, Payload payload);
    
    /// <summary>
    /// MatchPartyCancel() 요청 결과
    /// </summary>
    /// <param name="userAgent">MatchPartyCancel()을 요청한 유저 에이전트</param>
    /// <param name="result">MatchPartyCancel() 요청 결과</param>
    void OnMatchPartyCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchPartyCancel result);
    
    /// <summary>
    /// LeaveRoom() 요청 결과
    /// </summary>
    /// <param name="userAgent">LeaveRoom()을 요청한 유저 에이전트</param>
    /// <param name="result">LeaveRoom() 요청 결과</param>
    /// <param name="force">강제 퇴장 여부</param>
    /// <param name="roomId">퇴장한 방의 아이디</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnLeaveRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeLeaveRoom result, bool force, int roomId, Payload payload);
    
    /// <summary>
    /// GetChannelInfo() 요청 결과
    /// </summary>
    /// <param name="userAgent">GetChannelInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GetChannelInfo() 요청 결과</param>
    /// <param name="channelInfo">서버에서 받은 채널 정보</param>
    void OnChannelInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeChannelInfo result, Payload channelInfo);
    
    /// <summary>
    /// GeAllChannelInfo() 요청 결과
    /// </summary>
    /// <param name="userAgent">GeAllChannelInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GeAllChannelInfo() 정보 요청 결과</param>
    /// <param name="channelInfo">서버에서 받은 채널 정보 목록</param>
    void OnAllChannelInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeAllChannelInfo result, Dictionary<string, Payload> channelInfo);
    
    /// <summary>
    /// GetChannelCountInfo() 요청 결과
    /// </summary>
    /// <param name="userAgent">GetChannelCountInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GetChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은채널의 유저 수와 방 개수 정보</param>
    void OnChannelCountInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo);
    
    /// <summary>
    /// GetAllChannelCountInfo() 요청 결과
    /// </summary>
    /// <param name="userAgent">GetAllChannelCountInfo()를 요청한 유저 에이전트</param>
    /// <param name="result">GetAllChannelCountInfo() 요청 결과</param>
    /// <param name="channelCountInfo">서버에서 받은 채널의 유저 수와 방 개수 정보 목록</param>
    void OnAllChannelCountInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo);
    
    /// <summary>
    /// MoveChannel() 결과
    /// </summary>
    /// <param name="userAgent">MoveChannel()한 유저 에이전트</param>
    /// <param name="result">MoveChannel() 결과 코드</param>
    /// <param name="force">서버에서 강제로 체널을 이동했는지 여부</param>
    /// <param name="channelId">이동한 체널의 아이디</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnMoveChannel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMoveChannel result, bool force, string channelId, Payload payload);
    
    /// <summary>
    /// Snapshot() 요청 결과
    /// </summary>
    /// <param name="userAgent">Snapshot() 요청한 유저 에이전트</param>
    /// <param name="result">Snapshot() 요청 결과</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnSnapshot(UserAgent userAgent, GameAnvil.Defines.ResultCodeSnapshot result, Payload payload);
    
    /// <summary>
    /// 기본 기능 오류 발생
    /// </summary>
    /// <param name="userAgent">오류가 발생한 유저 에이전트</param>
    /// <param name="errorCode">오류 코드</param>
    /// <param name="command">오류 발생 기능</param>
    void OnError(UserAgent userAgent, GameAnvil.Defines.ErrorCode errorCode, User.Defines.Commands command);
    
    /// <summary>
    /// 패킷 전송 오류 발생
    /// </summary>
    /// <param name="userAgent">오류가 발생한 유저 에이전트</param>
    /// <param name="errorCode">오류 코드</param>
    /// <param name="command">오류가 발생한 메시지</param>
    void OnError(UserAgent userAgent, GameAnvil.Defines.ErrorCode errorCode, string command);
    
    /// <summary>
    /// Logout() 결과
    /// </summary>
    /// <param name="userAgent">Logout()을 요청한 유저 에이전트</param>
    /// <param name="result">Logout() 결과</param>
    /// <param name="force">서버에 의한 강제 여부</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnLogout(UserAgent userAgent, GameAnvil.Defines.ResultCodeLogout result, bool force, Payload payload);
    
    /// <summary>
    /// Notice() 알림
    /// </summary>
    /// <param name="userAgent">Notice()를 받은 유저 에이전트</param>
    /// <param name="message">고지 메시지</param>
    void OnNotice(UserAgent userAgent, string message);
    
    /// <summary>
    /// 킥아웃 된 경우 알림
    /// </summary>
    /// <param name="userAgent">킥아웃 된 유저 에이전트</param>
    /// <param name="message">어드민으로부터 받은 메시지</param>
    void OnAdminKickout(UserAgent userAgent, string message);
    
    /// <summary>
    /// 유저 리스너에서 서버의 세션이 닫힌 경우 알림<para></para>
    /// 이 알림을 받을 경우 다시 로그인하여 재시작 한다
    /// </summary>
    /// <param name="userAgent">세션이 닫힌 유저 에이전트</param>
    /// <param name="result">세션이 닫힌 이유</param>
    /// <param name="payload">서버에서 받은 추가 정보</param>
    void OnSessionClose(UserAgent userAgent, GameAnvil.Defines.ResultCodeSessionClose result, Payload payload);
}

UserAgent userAgent = connector.GetUserAgent(serviceId, subId);
userAgent.AddUserListener(new UserListener());
```
