## RoomMatch
MatchRoom()은 이미 만들어져 있는 Room으로 자동으로 매칭을 시켜 줍니다.
옵션에 따라서 방을 못잧을 경우 새로 만들어 들어가거나, 이미 방에 있을경우 다른 방으로 이동할 수 있습니다.
MatchRoom()을 호출하면 OnMatchRoom() 콜백이 올라옵니다.
``` C#
/// <summary>
/// Room 매치를 요청한다.
/// 방이 없을 경우 임의의 방을 생성하고 해당 방에 입장할 수도 있다.
/// </summary>
/// <param name="roomType">입장할 방의 roomType.</param>
/// <param name="isCreateRoomIfNotJoinRoom">true - 방이 없을 경우 임의의 방을 생성.false - 방이 없을 경우 실패</param>
/// <param name="isMoveRoomIfJoinedRoom">true - 방에 입장한 상태에서 요청했을 경우 다른 방으로 이동.false - 방에 입장한 상태에서 요청했을 경우 실패</param>
/// <param name="payload">서버에 전달할 추가 정보.서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="leaveRoomPayload">입장한 방을 떠날때 서버에 전달할 추가 정보.서버 구현에 따라 사용하지 않을 수 있음.</param>
/// <param name="onMatchRoom">결과를 받을 delegate.</param>
public void MatchRoom(string roomType, bool isCreateRoomIfNotJoinRoom, bool isMoveRoomIfJoinedRoom, Payload payload, Payload leaveRoomPayload, Interface.DelUserOnMatchRoom onMatchRoom)
```
``` C#
public interface IUserOnMatchRoom {
    /// <summary>
    /// MatchRoom의 결과
    /// </summary>
    /// <param name="userAgent">MatchRoom을 요청한 UserAgent 객체</param>
    /// <param name="result">MatchRoom 결과</param>
    /// <param name="roomId">매치된 방의 RoomId</param>
    /// <param name="roomName">매치된 방의 이름</param>
    /// <param name="created">매치된 방의 생성 여부(방장 여부)</param>
    /// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
    void OnMatchRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchRoom result, int roomId, string roomName, bool created, Payload payload); 
}
```
MatchRoom()의 대상은 CreateRoom()으로 생성된 방 또는 MatchRoom()으로 생성된 방 입니다.

## MatchUser
MatchUser()는 조건이 맞는 유저들을 묶어 새로운 방을 생성해 넣어줍니다. (묶이는 조건은 서버의 MatchMaker의 로직에 따릅니다.)
MatchUserStart()를 호츨하면 해당 유저를 매칭 풀에 넣고 OnMatchUserStart콜백을 올려줍니다.
``` C#
/// <summary>
/// UserMatch를 요청한다.
/// 이미 방에 입장한 경우 등 서버의 조건에 따라 요청이 실패할 수 있다.
/// 매칭이 성공한 경우 OnMatchUserDone을 통해 알림을 받을 수 있다.
/// </summary>
/// <param name="roomType">매치를 요청할 roomType.</param>
public void MatchUserStart(string roomType)
```
``` C#
public interface IUserOnMatchUserStart {
    /// <summary>
    /// MatchUserStart의 결과
    /// </summary>
    /// <param name="userAgent">MatchUserStart를 요청한 UserAgent 객체</param>
    /// <param name="result">MatchUserStart 결과</param>
    /// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
    void OnMatchUserStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserStart result, Payload payload); 
}
```

지정된 시간 내에 매칭이 이루어지면 OnMatchUserDone콜백이 올라옵니다.
``` C# 
public interface IUserOnMatchUserDone {
    /// <summary>
    /// 매칭 결과
    /// </summary>
    /// <param name="userAgent">매칭을 요청한 UserAgent 객체</param>
    /// <param name="result">매칭 결과</param>
    /// <param name="created">방 생성 여부. true일 경우 매칭 요청한 UserAgent가  방을 생성한 것을 의미한다. 방장을 결정하는 용도 등으로 사용할 수 있다.</param>
    /// <param name="roomId">매칭된 방의 RoomId</param>
    /// <param name="payload">추가 정보. 서버 구현에 따라 사용하지 않을 수 있음.</param>
    void OnMatchUserDone(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload); 
}
```
지정된 시간 내에 매칭이 이루어지지 않으면 OnMatchUserTimeout이 올라옵니다.
``` C#
public interface IUserOnMatchUserTimeout {
    /// <summary>
    /// 매칭 Timeout
    /// </summary>
    /// <param name="userAgent">매칭을 요청한 UserAgent 객체</param>
    void OnMatchUserTimeout(UserAgent userAgent); 
}
```
매칭이 이루어지기 전에 MatchUserCancel()을 호출하면 해당 유저를 매칭 풀에서 빼고 OnMatchUserCancel콜백이 올라옵니다.
``` C#
public interface IUserOnMatchUserCancel {
    /// <summary>
    /// MatchUserCancel의 결과
    /// </summary>
    /// <param name="userAgent">MatchUserCancel을 요청한 UserAgent 객체</param>
    /// <param name="result">MatchUserCancel 결과</param>
    void OnMatchUserCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserCancel result); 
}
```

매칭 대기중일때는 MatchCancel을 제외한 요청은 모두 무시됩니다.
매칭 대기중일때 재접속을 하게 되면 OnLogin() 콜백의 LoginInfo에 isMatching 플래그가 true로 설정됩니다. 이 플래그를 이용해 재접속 처리를 하면 됩니다.
``` C#
public sealed class LoginInfo
{
    /// <summary>
    /// 로그인한 유저의 Id
    /// </summary>
    public int UserId { get; private set; }
    /// <summary>
    /// 로그인한 유저의 Type
    /// </summary>
    public string UserType { get; private set; }
    /// <summary>
    /// 로그인한 Service의 이름
    /// </summary>
    public string ServiceName { get; private set; }
    /// <summary>
    /// 로그인한 Channel의 Id
    /// </summary>
    public string ChannelId { get; private set; }
    /// <summary>
    /// 서버로부터 받은 Message. 로그인 실패 이유 등.
    /// </summary>
    public string Message { get; private set; }
    /// <summary>
    /// 서버로부터 받은 추가정보.
    /// </summary>
    public Payload Payload { get; private set; }
    /// <summary>
    /// 재로그인 여부. 
    /// </summary>
    public bool IsRelogined { get; private set; }
    /// <summary>
    /// 방에 입장 여부.
    /// </summary>
    public bool isJoinedRoom { get { return !GameAnvil.Defines.ID.EMPTY.Equals(RoomId); } private set { } }
    /// <summary>
    /// 입장한 방의 RoomId.
    /// </summary>
    public int RoomId { get; private set; }
    /// <summary>
    /// 입장한 방의 이름
    /// </summary>
    public string RoomName { get; private set; }
    /// <summary>
    /// 입장한 방에 필요한 추가정보.
    /// </summary>
    public Payload RoomPayload { get; private set; }
    /// <summary>
    /// 매칭을 요청한 상태인지 여부.
    /// </summary>
    public bool IsMatching { get; private set; }
}
```