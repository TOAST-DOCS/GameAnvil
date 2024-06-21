## Game > GameAnvil > Unity Basic Development Guide > UserAgent

## UserAgent

The UserAgent is responsible for operations related to GameNodes on the GameAnvil server. It provides basic functionality such as login(), logout(), and room management. Based on the protocols you define, clients can message other objects through their user objects and implement different content. 

GameAnvilConnector provides a single UserAgent by default, created during the QuickConnect process. You can get it via GameAnvilConnector.getUserAgent().

### Sign in

Login can be defined as the process by which a client connects to the server and creates its own user object in GameNode. 

Login is handled in one step by QuickConnect, so we omit the description. For more information, see [Unity Advanced Development Guide > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md).

### Log out

Logging out of GameAnvilConnector automatically handles connection termination, see [Unity Advanced Development Guide > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md) for more information on logging out.

```c#
/// <summary>
/// Logs out of the current service 
/// </summary>
/// <param name="onLogout">Logout agent</param>
userAgent.Logout((UserAgent user, Defines.ResultCodeLogout result, bool force, Payload payload) => {
    /// <param name="userAgent">The user agent that requested Logout()</param>
    /// <param name="result">Logout() result</param>
    /// <param name="force">Whether forced by the server</param>
    /// <param name="payload">Additional information received from the server</param>
    if(result == Defines.ResultCodeLogout.LOGOUT_SUCCESS){
        // Success
    } else {
        // Failure
    }
});
```

### Create, enter, and leave rooms

Two or more users can create a synchronized message flow through a room, meaning that their requests are all in order within the room. Of course, creating a room for a single user can also make sense depending on the content. It's up to the engine user to decide how to use the room. 

Call CreateRoom() to create a room and enter it.

```c#
/// <summary>
/// Creates a random room and enters it.
/// </summary>
/// <param name="roomName">Name of the room to create</param>
/// <param name="roomType">Type of room to create</param>
/// <param name="matchingGroup">Matching group of rooms to match</param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="onCreateRoom">Agent to receive the result</param>
userAgent.CreateRoom(roomName, roomType, matchingGroup, payload, (UserAgent user, Defines.ResultCodeCreateRoom result, int roomId, string roomName, Payload payload) => {
    /// <param name="userAgent">The user agent that requested CreateRoom()</param>.
    /// <param name="result">Result of the CreateRoom() request</param>
    /// <param name="roomName">Name of the created room</param>
    /// <param name="roomId">Id of the created room</param>
    /// <param name="payload">Additional information received from the server</param>
	if(result == Defines.ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
        // Success
    } else {
        // Failure
    }
});
```
<br>

Call JoinRoom() to enter a room that has already been created. 

``` c#
/// <summary>
/// Enters the specified user's room<para></para>
/// Fails if the room with the specified username does not exist.
/// </summary>
/// <param name="roomType">Type of room to enter</param>
/// <param name="roomId">Id of the room you want to enter</param>
/// <param name="matchingUserCategory">Use when entering a room with room matching enabled</param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="onJoinRoom">Agent to receive the result</param>
userAgent.JoinRoom(roomType, roomId, matchingUserCategory, payload, (UserAgent user, Defines.ResultCodeJoinRoom result, int id, string roomName, Payload payload) => {
    /// <param name="result">JoinRoom() request result</param>
    /// <param name="roomId">Id of the entered room</param>
    /// <param name="roomName">Name of the room you joined</param>
    /// <param name="payload">Additional information received from the server</param>
	if(result == Defines.ResultCodeJoinRoom.JOIN_ROOM_SUCCESS){
        // Success
    } else {
        // Failure
    }
});
```
<br>

You can call LeaveRoom() to leave the room you entered.

``` c#
/// <summary>
/// Leaves the room the user agent is in.
/// </summary>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="onLeaveRoom">A delegate to receive the result</param>
userAgent.LeaveRoom(payload, (UserAgent user, Defines.ResultCodeLeaveRoom result, bool force, int roomId, Payload payload) => {
    /// <param name="userAgent">The user agent that requested LeaveRoom()</param>
    /// <param name="result">Result of the LeaveRoom() request</param>
    /// <param name="force">Whether to force the user to leave</param>
    /// <param name="roomId">Id of the room to leave</param>
    /// <param name="payload">Additional information received from the server</param>
	if(result == Defines.ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS){
        // Success
    } else {
        // Failure
    }
});
```
<br>

You can enter a room with the specified name by calling NamedRoom(). If a room with the specified name doesn't exist, a room is created and you enter it.

```c#
/// <summary>
/// Enters a room with the specified name<para></para>
/// Creates a room with the specified name if it doesn't exist and enters it
/// </summary>
/// <param name="roomType">Type of room to enter</param>
/// <param name="roomName">Name of the room you want to enter</param>
/// <param name="isParty">Is it a party</param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="onNamedRoom">The agent to receive the results</param>.
userAgent.NamedRoom(roomType, roomName, isParty, payload, (UserAgent user, Defines.ResultCodeNamedRoom result, int roomId, string roomName, bool created, Payload payload) => {
    /// <param name="userAgent">The user agent that requested NameRoom()</param>
    /// <param name="result">Result of the NameRoom() request</param>
    /// <param name="roomName">Room name</param>
    /// <param name="roomId">Id of the room entered</param>
    /// <param name="created">Whether the entered room was created</param>
    /// <param name="payload">Additional information received from the server</param>
    if(result == Defines.ResultCodeNamedRoom.NAMED_ROOM_SUCCESS){
        // Success
    } else {
        // Failure
    }
});
```

### Matchmaking

GameAnvil offers two main types of matchmaking. One is Room Matchmaking, which performs room-by-room matching, and the other is User Matchmaking, which performs user-by-user matching.

#### Room matchmaking

Room matchmaking is a way to admit users to rooms that meet their criteria. When a room matchmaking request is made, if a room that meets the criteria is available, the user is admitted directly to that room, otherwise a new room is created.

You can request room matchmaking by calling MatchRoom().

```c#
/// <summary>
/// Request a room match<para></para>
/// If no room exists, create a random room and enter it or handle match failure
/// </summary>
/// <param name="roomType">Type of room to enter</param>
/// <param name="matchingGroup">Matching group for the room to match</param>
/// <param name="matchingUserCategory">Matching user category to use when matching</param>
/// <param name="isCreateRoomIfNotJoinRoom">Allow room creation</param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="onMatchRoom">Agent to receive results</param>
userAgent.MatchRoom(roomType, matchingGroup, matchingUserCategory, isCreateRoomIfNotJoinRoom, isMoveRoomIfJoinedRoom, payload, leaveRoomPayload, (UserAgent user, Defines.ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload) => {
    /// <param name="userAgent">The user agent that requested MatchRoom()</param>
    /// <param name="result">Result of the MatchRoom() request</param>
    /// <param name="resultCode">User result code</param>
    /// <param name="roomId">Id of the matched room</param>
    /// <param name="roomName">Name of the matched room</param>
    /// <param name="created">Whether the matched room was created</param>
    /// <param name="payload">Additional information received from the server</param>
    if(result == Defines.ResultCodeMatchRoom.MATCH_ROOM_SUCCESS){
        // Matchmaking canceled successfully
    } else {
        // Failed to cancel matchmaking
    }
});
```

#### User matchmaking

User matchmaking works by creating a user pool and searching within it for users who meet the criteria to enter the newly created room. If there aren't enough users in the user pool to meet the criteria, matchmaking can take a while to complete, and if it doesn't complete in time, a timeout may occur, canceling the match. 

You can call MatchUserStart() to start matchmaking users. The request might fail depending on conditions on the server, such as if you've already entered the room. 

```c#
/// <summary>
/// Requests a user match<para></para>.
/// Fail the request based on conditions on the server, such as if the user has already entered the room<para></para>
/// Notify OnMatchUserDone() if the match is successful<para></para>
/// </summary>
/// <param name="roomType">The type of room to request to match</param>
/// <param name="matchingGroup">Matching group to use when creating the room</param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="onMatchUserStart">The agent to receive the results</param>.
userAgent.MatchUserStart(roomType, matchingGroup, payload, (UserAgent user, Defines.ResultCodeMatchUserStart result, Payload payload) => {
    /// <param name="userAgent">The user agent that requested MatchUserStart()</param>
    /// <param name="result">Result of the MatchUserStart() request</param>
    /// <param name="payload">Additional information received from the server</param>
	if(result == Defines.ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS){
        // Matchmaking request succeeded
    } else {
        // Matchmaking request failed
    }
});
```
<br>

You can be notified via onMatchUserDoneListeners or IUserListener.OnMatchUserDone if the match was successful, or via onMatchUserTimeoutListeners or IUserListener.OnMatchUserTimeout if the match was not successful in time.

```c#
/// <summary>
/// Representative to receive user matching results
/// </summary>
userAgent.onMatchUserDoneListeners += (UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload) => {
    /// <param name="userAgent">The user agent that requested MatchUserStart() or MatchPartyStart()</param>.
    /// <param name="result">Result of the MatchUserStart() or MatchPartyStart() request</param>
    /// <param name="created">Whether a room was created</param>
    /// <param name="roomId">Id of the matched room</param>
    /// <param name="payload">Additional information received from the server</param>
};

/// <summary>
/// User matching timeout notification proxy
/// </summary>
userAgent.onMatchUserTimeoutListeners += (UserAgent userAgent) => {
    /// <param name="userAgent">The user agent that requested MatchUserStart() or MatchPartyStart()</param>.
};
```
<br>

You can cancel user matchmaking by calling MatchUserCancel(). If you are not in the middle of requesting a user match, the cancel request might fail if user matching has already succeeded or a timeout has occurred. 

```c#
/// <summary>
/// Cancel a user match request<para></para>
/// If not in the middle of a match request, fails if a match has already succeeded or a timeout has occurred<para></para>
/// </summary>
/// <param name="roomType">The type of bar requested to be matched</param>
/// <param name="onMatchUserCancel">The agent to cancel the result</param>.
userAgent.MatchUserCancel(Constants.RoomType, (UserAgent user, Defines.ResultCodeMatchUserCancel result) => {
    /// <param name="userAgent">The user agent that requested MatchUserCancel()</param>
    /// <param name="result">Result of the MatchUserCancel() request</param>
	if(result == Defines.ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS){
        // Matchmaking cancel success
    } else {
        // Failed to cancel matchmaking
    }
});
```

Other features of the User Agent, such as party matchmaking, channels, listeners, and more, are described in the [Unity Advanced Development Guide > UserAgent](../unity-advanced/unity-advanced-03-user-agent.md).