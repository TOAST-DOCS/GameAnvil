## Game > GameAnvil > Unity Advanced Development Guide > User

## UserAgent

The UserAgent is responsible for operations related to GameNodes on the GameAnvil server. It provides basic functionality such as login(), logout(), and room management. Based on the protocols you define, clients can message other objects through their user objects and implement different content. 

To use UserAgent, you need to create a new UserAgent using the Connector.CreateUserAgent() function. You can create multiple UserAgents, separated by ServiceName and SubId. The created UserAgent is managed internally by the Connector and can be used again using the Connector.GetUserAgent() function. 

```c#
UserAgent userAgent = ConnectHandler.getInstance().GetUserAgent(serviceName, subID);
if (userAgent == null) {
    userAgent = ConnectHandler.getInstance().CreateUserAgent(serviceName, subID);
}
```

The GameAnvil server can run multiple services simultaneously, and a UserAgent can log in to one service and operate independently of another. This means that you can create multiple UserAgents to log in to different services and use them simultaneously. It is also possible to have multiple UserAgents logged into the same service at the same time by using different SubIds. 

### Login/Logout

Login can be defined as the process by which a client connects to the server and creates its own user object in GameNode. Logging out is the opposite of logging in. In other words, it is the process of removing a user object from the GameNode. 

When logging in, you'll need to enter which UserType is logging in to which channel. If you need more information, you can send it in the payload. 

```c#
/// <summary>
/// Logs in to the service
/// </summary>
/// <param name="userType">User's type</param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="channelId">Id of the channel to log in to</param>
/// <param name="onLogin">The agent to receive the result</param>
userAgent.Login(userType, channelId, payload, (UserAgent user, Defines.ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => {
    /// <param name="userAgent">The user agent that requested Login()</param>
    /// <param name="result">Result of the Login() request</param>
    /// <param name="loginInfo">Login information</param>
    if(result == Defines.ResultCodeLogin.LOGIN_SUCCESS){
        // Success
    } else {
        // Failure
    }
});

/// <summary>
/// Logs out of the current service 
/// </summary>
/// <param name="onLogout">Logout bridge</param>
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
...
```

### Create, enter, and leave rooms

This is the same as creating, entering, and leaving rooms in [Unity Basic Development Guide > UserAgent](../unity-basic/unity-basic-04-user-agent.md).

### Matchmaking

GameAnvil offers two types of matchmaking. One is Room Matchmaking, which performs room-by-room matching, and the other is User Matchmaking, which performs user-by-user matching. For more information, see Matchmaking in the [Unity Basic Development Guide > UserAgent](../unity-basic/unity-basic-04-user-agent.md).

#### Party matchmaking

Party matchmaking is a specialized form of user matchmaking where two or more users are grouped together as a party, added to a user pool, and matched with other eligible users to enter a newly created room together. Partyed users will always enter the same room. Outside of parties, the matching users can be other parties or individuals, depending on the server's matchmaker implementation.

To do party matchmaking, you must first call NamedRoom(). If you pass the isParty parameter to true when calling NamedRoom(), that NamedRoom will act as the party. After gathering all the party users in the NamedRoom, start party matchmaking.

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
<br>

You can start party matchmaking by calling MatchPartyStart(). The request might fail depending on conditions on the server, such as if you've already entered the room. 

```c#
/// <summary>
/// Requests party matching<para></para>
/// Request fails based on conditions on the server, such as if you've already entered the room<para></para>
/// Notify via OnMatchUserDone() if matching is successful
/// </summary>
/// <param name="roomType">The type of room to request to match</param>
/// <param name="matchingGroup">Matching group to use when creating the room</param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="onMatchPartyStart">The agent that will receive the results</param>.
userAgent.MatchPartyStart(Constants.RoomType, Constants.ChannelId1, customPayload, (UserAgent user, Defines.ResultCodeMatchPartyStart result, Payload payload) => {
    /// <param name="userAgent">The user agent that requested MatchPartyStart()</param>.
    /// <param name="result">Result of the MatchPartyStart() request</param>
    /// <param name="payload">Additional information received from the server</param>
    if(result == Defines.ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS){
        // Success
    } else {
        // Failure
    }
});
```
<br>

You can be notified via the onMatchUserDoneListeners or IUserListener.OnMatchUserDone that you used for user matchmaking if the party match was successful, or via the onMatchUserTimeoutListeners or IUserListener.OnMatchUserTimeout if the match was not successful in time.

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

You can cancel party matchmaking by calling MatchPartyCancel(). If you are not in the middle of requesting a party match, the cancel request might fail if party matchmaking has already succeeded or a timeout has occurred. 

```c#
/// <summary>
/// Cancels a party matching request<para></para>
/// If not requesting a match, fails if the match has already succeeded or a timeout has occurred.
/// </summary>
/// <param name="roomType">The type of room requested to be matched</param>
/// <param name="onMatchPartyCancel">The agent that will receive the results</param>.
userAgent.MatchPartyCancel(Constants.RoomType, (UserAgent user, Defines.ResultCodeMatchPartyCancel result) => {
    /// <param name="userAgent">The user agent that requested MatchPartyCancel()</param>
    /// <param name="result">Result of the MatchPartyCancel() request</param>
    if(result == Defines.ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS){
        // Success
    } else {
        // Failed
    }
});
```

#### Moving channels

In some cases, channel moves can occur as a result of matchmaking. You can be notified via onMoveChannelListeners or IUserListener.OnMoveChannel when a channel move has occurred.

```c#
/// <summary>
/// A representative to be notified of the results of channel move requests or channel moves forced by the server.
/// </summary>
userAgent.onMoveChannelListeners += (UserAgent userAgent, GameAnvil.Defines.ResultCodeMoveChannel result, bool force, string channelId, Payload payload) => {
    /// <param name="userAgent">The user agent that performed the MoveChannel()</param>.
    /// <param name="result">MoveChannel() result code</param>
    /// <param name="force">Whether the server forced the channel to be moved</param>
    /// <param name="channelId">Id of the moved channel</param>
    /// <param name="payload">Additional information received from the server</param>
};
```
<br>

You can move to a different channel within the service by calling MoveChannel(). 

```c#
/// <summary>
/// Goes to the specified channel
/// </summary>
/// <param name="channelId">Id of the channel to move to</param>
/// <param name="payload">Additional information to pass to the server</param>
/// <param name="onMoveChannel">Agent to receive the result</param>
userAgent.MoveChannel(channelId, usePayload ? customPayload : null, (UserAgent user, Defines.ResultCodeMoveChannel result, bool force, string channelID, Payload payload) => {
    /// <param name="userAgent">The user agent that did the MoveChannel()</param>.
    /// <param name="result">MoveChannel() result code</param>
    /// <param name="force">Whether the server forced the channel to be moved</param>
    /// <param name="channelId">Id of the moved channel</param>
    /// <param name="payload">Additional information received from the server</param>
	if(result == ResultCodeMoveChannel.ALL_CHANNEL_INFO_SUCCESS){
		// All channel information request succeeded
	} else {
		// All channel information request failed
	}
});
```

### Channel information

GameAnvil allows you to freely change channel configurations in the settings. These channel configurations can be pre-agreed between the server and the client and used in a fixed form, or they can be changed flexibly to suit the situation. UserAgent provides several functions to get information about these changed channels or to move channels. 

| Functions | Description |
| --- | --- |
| GetChannelCountInfo() | Request count information (number of users and rooms) for a specific channel | 
| GetChannelInfo() | Request information (user-defined) for a specific channel |
| GetAllChannelCountInfo() | Request count information (number of users and rooms) for all channels of a specific service |
| GetAllChannelInfo() | Request information about all channels for a specific service (custom) |

Let's take a closer look at this in code below.

GetChannelCountInfo() can request and receive count information (number of users and rooms) for a specific channel. 

```c#
/// <summary>
/// Gets the number of users and rooms on the channel being connected<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="onChannelCountInfo">The agent to send the result to</param>.
userAgent.GetChannelCountInfo((ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="userAgent">The user agent that requested GetChannelCountInfo()</param>
    /// <param name="result">Result of the GetChannelCountInfo() request</param>
    /// <param name="channelCountInfo">Channel's user count and room count information received from the server</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// Channel count information request success
	} else {
		// Channel count information request failed
	}
});

/// <summary>
/// Requests the number of users and rooms in a specific channel<para></para>
/// Can be used if supported by the server
/// </summary>
/// <param name="serviceName">Name of the service to request channel information from</param>
/// <param name="channelId">Id of the channel to request channel information for</param>
/// <param name="onChannelCountInfo">An agent to receive the result</param>.
userAgent.GetChannelCountInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo) => {
    /// <param name="userAgent">The user agent that requested GetChannelCountInfo()</param>
    /// <param name="result">Result of the GetChannelCountInfo() request</param>
    /// <param name="channelCountInfo">Channel's user count and room count information received from the server</param>
	if(result == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS){
		// Channel count information request success
	} else {
		// Channel count information request failed
	}
});
```
<br>

GetChannelInfo() can request and receive information about a channel (user-defined). 

```c#
/// <summary>
/// Requests information about the channel being connected<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="onChannelInfo">The agent to send the result to</param>
userAgent.GetChannelInfo((ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="userAgent">The user agent who requested GetChannelInfo()</param>
    /// <param name="result">Result of the GetChannelInfo() request</param>
    /// <param name="channelInfo">Channel information received from the server</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// Channel information request success
	} else {
		// Channel information request failed
	}
});

/// <summary>
/// Requests information about a specific channel<para></para>
/// Can be used if supported by the server
/// </summary>
/// <param name="serviceName">Name of the service for which to request channel information</param>
/// <param name="channelId">Id of the channel to request channel information for</param>
/// <param name="onChannelInfo">The agent to receive the result</param>.
userAgent.GetChannelInfo(serviceName, channelId, (ConnectionAgent connection, ResultCodeChannelInfo result, Payload payload) => {
    /// <param name="userAgent">The user agent that requested GetChannelInfo()</param>
    /// <param name="result">Result of the GetChannelInfo() request</param>
    /// <param name="channelInfo">Channel information received from the server</param>
	if(result == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS){
		// Channel information request success
	} else {
		// Channel information request failed
	}
});
```
<br>

GetAllChannelCountInfo() can request and receive count information (number of users and rooms) for all channels in a service. Pass the service name as a parameter and a callback to handle the response. 

```c#
/// <summary>
/// Gets the number of users and rooms on all channels in the service being connected<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="onAllChannelCountInfo">The agent to receive the result</param>.
userAgent.GetAllChannelCountInfo((ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="userAgent">The user agent that requested GetAllChannelCountInfo()</param>
    /// <param name="result">Result of the GetAllChannelCountInfo() request</param>
    /// <param name="channelCountInfo">List of channel's user count and room count information received from server</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// All channel count information request success
	} else {
		// All channel count information request failed
	}
});

/// <summary>
/// Gets the number of users and rooms in all channels on a specific service<para></para>.
/// Available if supported by the server
/// </summary>
/// <param name="serviceName">Name of the service for which to request channel information</param>
/// <param name="onAllChannelCountInfo">Agent to receive the result</param>
userAgent.GetAllChannelCountInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo) => {
    /// <param name="userAgent">The user agent that requested GetAllChannelCountInfo()</param>.
    /// <param name="result">Result of the GetAllChannelCountInfo() request</param>
    /// <param name="channelCountInfo">List of channel's user count and room count information received from server</param>
	if(result == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS){
		// All channel count information request success
	} else {
		// All channel count information request failed
	}
});
```
<br>

GetAllChannelInfo() can request and receive information (user-defined) about all channels of a service. As parameters, you pass the service name and a callback to handle the response. 

```c#
/// <summary>
/// Request all channel information for a specific service<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="serviceName">Name of the service to request channel information for</param>
/// <param name="onAllChannelInfo">Agent to receive the result</param>
userAgent.GetAllChannelInfo((ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="userAgent">The user agent that requested GeAllChannelInfo()</param>
    /// <param name="result">Result of the GeAllChannelInfo() information request</param>
    /// <param name="channelInfo">List of channel information received from the server</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// All channel information request success
	} else {
		// All channel information request failed
	}
});

/// <summary>
/// Request all channel information for a specific service<para></para>
/// Available if supported by the server
/// </summary>
/// <param name="serviceName">Name of the service for which to request channel information</param>
/// <param name="onAllChannelInfo">Agent to receive the result</param>
userAgent.GetAllChannelInfo(serviceName, (ConnectionAgent connection, ResultCodeAllChannelInfo result, Dictionary<string, Payload> payload) => {
    /// <param name="userAgent">The user agent that requested GeAllChannelInfo()</param>
    /// <param name="result">Result of the GeAllChannelInfo() information request</param>
    /// <param name="channelInfo">List of channel information received from the server</param>
	if(result == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS){
		// All channel information request success
	} else {
		// All channel information request failed
	}
});
```

### Listener

There are two main ways that UserAgent can receive results or notifications from the server for every request.
One is to add a function to the delegate defined on the UserAgent. The other is to register a listener that implements the IUserListener interface.

The UserAgent has each delegate as a member so that it can receive the results or notifications of any action. By registering a function on this delegate, you can call the APIs described earlier without callback parameters, or receive a response from the registered function when the server sends a notification.

```c#
/// <summary>
/// Delegate to receive login results
/// </summary>
/// <param name="userAgent">The user agent that made the Login() request</param>
/// <param name="result">Login() request result</param>
/// <param name="loginInfo">Login information</param>
public Interface.DelUserOnLogin onLoginListeners;

/// <summary>
/// Delegate to receive room match request results.
/// </summary>
/// <param name="userAgent">The user agent that requested MatchRoom().
/// <param name="result">Result of the MatchRoom() request</param>
/// <param name="resultCode">User result code</param>
/// <param name="roomId">Id of the matched room</param>
/// <param name="roomName">Name of the matched room</param>
/// <param name="created">Whether the matched room was created</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnMatchRoom onMatchRoomListeners;

/// <summary>
/// Representatives to receive the results of room creation requests.
/// </summary>
/// <param name="userAgent">The user agent that requested CreateRoom().
/// <param name="result">Result of the CreateRoom() request</param>
/// <param name="roomName">Name of the created room</param>
/// <param name="roomId">Id of the created room</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnCreateRoom onCreateRoomListeners;

/// <summary>
/// Delegates to receive room entry request results.
/// </summary>
/// <param name="userAgent">The user agent that requested JoinRoom().
/// <param name="result">Result of the JoinRoom() request</param>
/// <param name="roomId">Id of the room entered</param>
/// <param name="roomName">Name of the room you joined</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnJoinRoom onJoinRoomListeners;

/// <summary>
/// Delegate to receive the results of a named room request.
/// </summary>
/// <param name="userAgent">The user agent that requested NameRoom().
/// <param name="result">Result of the NameRoom() request</param>
/// <param name="roomName">Room name</param>
/// <param name="roomId">Id of the room entered</param>
/// <param name="created">Whether the entered room was created</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnNamedRoom onNamedRoomListeners;

/// <summary>
/// Delegate to receive the results of the party matching request.
/// </summary>
/// <param name="userAgent">The user agent that requested MatchPartyStart().
/// <param name="result">Result of the MatchPartyStart() request</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnMatchPartyStart onMatchPartyStartListeners;

/// <summary>
/// Delegates to receive the results of canceling a party matching request.
/// </summary>
/// <param name="userAgent">The user agent that requested MatchPartyCancel()</param>.
/// <param name="result">Result of the MatchPartyCancel() request</param>
public Interface.DelUserOnMatchPartyCancel onMatchPartyCancelListeners;

/// <summary>
/// Delegate to receive the result of a user matching request.
/// </summary>
/// <param name="userAgent">The user agent that requested MatchUserStart()</param>.
/// <param name="result">Result of the MatchUserStart() request</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnMatchUserStart onMatchUserStartListeners;

/// <summary>
/// Delegates to receive the results of user matching requests.
/// </summary>
/// <param name="userAgent">The user agent that requested MatchUserStart() or MatchPartyStart()</param>.
/// <param name="result">Result of the MatchUserStart() or MatchPartyStart() request</param>
/// <param name="created">Whether a room was created</param>
/// <param name="roomId">Id of the matched room</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnMatchUserDone onMatchUserDoneListeners;

/// <summary>
/// Delegate for user matching timeout notification.
/// </summary>
/// <param name="userAgent">The user agent that requested MatchUserStart() or MatchPartyStart()</param>.
public Interface.DelUserOnMatchUserTimeout onMatchUserTimeoutListeners;

/// <summary>
/// The agent to receive the result of canceling a user matching request.
/// </summary>
/// <param name="userAgent">The user agent that requested MatchUserCancel()</param>.
/// <param name="result">Result of the MatchUserCancel() request</param>
public Interface.DelUserOnMatchUserCancel onMatchUserCancelListeners;

/// <summary>
/// Delegates to be notified when a user leaves a room or is forced to leave.
/// </summary>
/// <param name="userAgent">The user agent that requested LeaveRoom().
/// <param name="result">Result of the LeaveRoom() request</param>
/// <param name="force">Whether to force the user to leave</param>
/// <param name="roomId">Id of the room to leave</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnLeaveRoom onLeaveRoomListeners;

/// <summary>
/// Delegates to be notified of the results of channel information requests or channel moves forced by the server.
/// </summary>
/// <param name="userAgent">The user agent that requested GetChannelInfo().
/// <param name="result">Result of the GetChannelInfo() request</param>
/// <param name="channelInfo">Channel information received from the server</param>
public Interface.DelUserOnChannelInfo onChannelInfoListeners;

/// <summary>
/// Delegates to be notified of the results of any channel information list requests or channel moves forced by the server.
/// </summary>
/// <param name="userAgent">The user agent that requested GeAllChannelInfo().
/// <param name="result">GeAllChannelInfo() information request result</param>
/// <param name="channelInfo">List of channel information received from the server</param>
public Interface.DelUserOnAllChannelInfo onAllChannelInfoListeners;

/// <summary>
/// Delegates to be notified of channel information request results or channel moves forced by the server.
/// </summary>
/// <param name="userAgent">The user agent that requested GetChannelCountInfo().
/// <param name="result">Result of the GetChannelCountInfo() request</param>
/// <param name="channelCountInfo">The channel's user count and room count information received from the server</param>.
public Interface.DelUserOnChannelCountInfo onChannelCountInfoListeners;

/// <summary>
/// A delegate to be notified of the results of any channel information requests or channel moves forced by the server.
/// </summary>
/// <param name="userAgent">The user agent that requested GetAllChannelCountInfo().
/// <param name="result">Result of the GetAllChannelCountInfo() request</param>
/// <param name="channelCountInfo">List of channel's user count and room count information received from the server</param>
public Interface.DelUserOnAllChannelCountInfo onAllChannelCountInfoListeners;

/// <summary>
/// Delegates to be notified of channel move request results or channel moves forced by the server.
/// </summary>
/// <param name="userAgent">The user agent that performed the MoveChannel()</param>.
/// <param name="result">MoveChannel() result code</param>
/// <param name="force">Whether the server forced the channel to be moved</param>
/// <param name="channelId">Id of the moved channel</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnMoveChannel onMoveChannelListeners;

/// <summary>
/// Delegate to receive snapshot request results.
/// </summary>
/// <param name="userAgent">The user agent that took the snapshot</param>.
/// <param name="result">Snapshot() request result</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnSnapshot onSnapshotListeners;

/// <summary>
/// Delegates to be notified when errors occur while using basic functionality.
/// </summary>
/// <param name="userAgent">The user agent where the error occurred</param>
/// <param name="errorCode">Error code</param>
/// <param name="command">Command to raise the error</param>
public Interface.DelUserOnErrorCommand onErrorCommandListeners;

/// <summary>
/// Delegates to be notified when a packet transmission error occurs.
/// </summary>
/// <param name="userAgent">The user agent where the error occurred</param>
/// <param name="errorCode">Error code</param>
/// <param name="command">The message where the error occurred</param>
public Interface.DelUserOnErrorCustomCommand onErrorCustomCommandListeners;

/// <summary>
/// The delegate to be notified of the result of a logout request or forced logout.
/// </summary>
/// <param name="userAgent">The user agent that requested Logout().
/// <param name="result">Logout() result</param>
/// <param name="force">Whether forced by the server</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnLogout onLogoutListeners;

/// <summary>
/// Representatives to be notified.
/// </summary>
/// <param name="userAgent">The user agent that received the Notice()</param>.
/// <param name="message">Notice message</param>
public Interface.DelUserOnNotice onNoticeListeners;

/// <summary>
/// Delegates to receive kickout notifications.
/// </summary>
/// <param name="userAgent">The user agent that was kicked out</param>.
/// <param name="message">Message received from the child</param>
public Interface.DelUserOnAdminKickout onAdminKickoutListeners;

/// <summary>
/// Delegates to be notified when a session on the server is closed.
/// </summary>
/// <param name="userAgent">The user agent whose session has been closed</param>
/// <param name="result">Reason the session was closed</param>
/// <param name="payload">Additional information received from the server</param>
public Interface.DelUserOnSessionClose onSessionCloseListeners;
```
<br>

IUserListener is an interface that defines the results or notifications of any action of UserAgent. You can register a listener implementing this interface with UserAgent.AddUserListener() to receive responses as a registered listener.

```c#
class UserListener : IUserListener{
    /// <summary>
    /// Login() request result
    /// </summary>
    /// <param name="userAgent">The user agent that made the Login() request</param>
    /// <param name="result">Login() request result</param>
    /// <param name="loginInfo">Login information</param>
    void OnLogin(UserAgent userAgent, GameAnvil.Defines.ResultCodeLogin result, UserAgent.LoginInfo loginInfo);
    
    /// <summary>
    /// MatchRoom() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested MatchRoom()</param>.
    /// <param name="result">Result of the MatchRoom() request</param>
    /// <param name="resultCode">User result code</param>
    /// <param name="roomId">Id of the matched room</param>
    /// <param name="roomName">Name of the matched room</param>
    /// <param name="created">Whether the matched room was created</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnMatchRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchRoom result, int resultCode, int roomId, string roomName, bool created, Payload payload);
    
    /// <summary>
    /// CreateRoom() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested CreateRoom()</param>.
    /// <param name="result">CreateRoom() request result</param>
    /// <param name="roomId">Id of the created room</param>
    /// <param name="roomName">Name of the created room</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnCreateRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeCreateRoom result, int roomId, string roomName, Payload payload);
    
    /// <summary>
    /// JoinRoom() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested JoinRoom()</param>
    /// <param name="result">Result of the JoinRoom() request</param>
    /// <param name="roomId">Id of the room entered</param>
    /// <param name="roomName">Name of the room you joined</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnJoinRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeJoinRoom result, int roomId, string roomName, Payload payload);
    
    /// <summary>
    /// NameRoom() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested NameRoom()</param>
    /// <param name="result">Result of the NameRoom() request</param>
    /// <param name="roomName">Room name</param>
    /// <param name="roomId">Id of the room entered</param>
    /// <param name="created">Whether the entered room was created</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnNamedRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeNamedRoom result, int roomId, string roomName, bool created, Payload payload);
    
    /// <summary>
    /// MatchUserStart() result
    /// </summary>
    /// <param name="userAgent">The user agent that requested MatchUserStart()</param>
    /// <param name="result">Result of the MatchUserStart() request</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnMatchUserStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserStart result, Payload payload);
    
    /// <summary>
    /// The result of a MatchUserStart() or MatchPartyStart() request.
    /// </summary>
    /// <param name="userAgent">The user agent that requested MatchUserStart() or MatchPartyStart()</param>.
    /// <param name="result">Result of the MatchUserStart() or MatchPartyStart() request</param>
    /// <param name="created">Whether a room was created</param>
    /// <param name="roomId">Id of the matched room</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnMatchUserDone(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserDone result, bool created, int roomId, Payload payload);
    
    /// <summary>
    /// MatchUserStart() or MatchPartyStart() timeout
    /// </summary>
    /// <param name="userAgent">The user agent that requested MatchUserStart() or MatchPartyStart()</param>.
    void OnMatchUserTimeout(UserAgent userAgent);
    
    /// <summary>
    /// Results of MatchUserCancel()
    /// </summary>
    /// <param name="userAgent">The user agent that requested MatchUserCancel()</param>.
    /// <param name="result">Result of the MatchUserCancel() request</param>
    void OnMatchUserCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchUserCancel result);
    
    /// <summary>
    /// The result of a MatchPartyStart() request.
    /// </summary>
    /// <param name="userAgent">The user agent that requested MatchPartyStart()</param>.
    /// <param name="result">Result of the MatchPartyStart() request</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnMatchPartyStart(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchPartyStart result, Payload payload);
    
    /// <summary>
    /// The result of a MatchPartyCancel() request.
    /// </summary>
    /// <param name="userAgent">The user agent that requested MatchPartyCancel()</param>.
    /// <param name="result">Result of the MatchPartyCancel() request</param>
    void OnMatchPartyCancel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMatchPartyCancel result);
    
    /// <summary>
    /// LeaveRoom() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested LeaveRoom()</param>
    /// <param name="result">Result of the LeaveRoom() request</param>
    /// <param name="force">Whether to force the user to leave</param>
    /// <param name="roomId">Id of the room to leave</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnLeaveRoom(UserAgent userAgent, GameAnvil.Defines.ResultCodeLeaveRoom result, bool force, int roomId, Payload payload);
    
    /// <summary>
    /// GetChannelInfo() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested GetChannelInfo()</param>.
    /// <param name="result">Result of the GetChannelInfo() request</param>
    /// <param name="channelInfo">Channel information received from the server</param>
    void OnChannelInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeChannelInfo result, Payload channelInfo);
    
    /// <summary>
    /// GeAllChannelInfo() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested GeAllChannelInfo()</param>
    /// <param name="result">Result of the GeAllChannelInfo() information request</param>
    /// <param name="channelInfo">List of channel information received from the server</param>
    void OnAllChannelInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeAllChannelInfo result, Dictionary<string, Payload> channelInfo);
    
    /// <summary>
    /// GetChannelCountInfo() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested GetChannelCountInfo()</param>.
    /// <param name="result">Result of the GetChannelCountInfo() request</param>
    /// <param name="channelCountInfo">The number of users and rooms in the channel received from the server</param>
    void OnChannelCountInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeChannelCountInfo result, ChannelCountInfo channelCountInfo);
    
    /// <summary>
    /// GetAllChannelCountInfo() request result
    /// </summary>
    /// <param name="userAgent">The user agent that requested GetAllChannelCountInfo()</param>.
    /// <param name="result">Result of the GetAllChannelCountInfo() request</param>
    /// <param name="channelCountInfo">List of channel's user count and room count information received from the server</param>
    void OnAllChannelCountInfo(UserAgent userAgent, GameAnvil.Defines.ResultCodeAllChannelCountInfo result, Dictionary<string, ChannelCountInfo> channelCountInfo);
    
    /// <summary>
    /// MoveChannel() result
    /// </summary>
    /// <param name="userAgent">The user agent that did the MoveChannel()</param>.
    /// <param name="result">MoveChannel() result code</param>
    /// <param name="force">Whether the server forced the channel to be moved</param>
    /// <param name="channelId">Id of the moved channel</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnMoveChannel(UserAgent userAgent, GameAnvil.Defines.ResultCodeMoveChannel result, bool force, string channelId, Payload payload);
    
    /// <summary>
    /// Snapshot() request result
    /// </summary>
    /// <param name="userAgent">Snapshot() requested user agent</param>
    /// <param name="result">Snapshot() request result</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnSnapshot(UserAgent userAgent, GameAnvil.Defines.ResultCodeSnapshot result, Payload payload);
    
    /// <summary>
    /// A basic functionality error occurred.
    /// </summary>
    /// <param name="userAgent">User agent where the error occurred</param>
    /// <param name="errorCode">Error code</param>
    /// <param name="command">Command to raise the error</param>
    void OnError(UserAgent userAgent, GameAnvil.Defines.ErrorCode errorCode, User.Defines.Commands command);
    
    /// <summary>
    /// A packet transmission error occurred.
    /// </summary>
    /// <param name="userAgent">The user agent where the error occurred</param>
    /// <param name="errorCode">Error code</param>
    /// <param name="command">The message where the error occurred</param>
    void OnError(UserAgent userAgent, GameAnvil.Defines.ErrorCode errorCode, string command);
    
    /// <summary>
    /// Logout() result
    /// </summary>
    /// <param name="userAgent">The user agent that requested Logout()</param>
    /// <param name="result">Logout() result</param>
    /// <param name="force">Whether forced by the server</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnLogout(UserAgent userAgent, GameAnvil.Defines.ResultCodeLogout result, bool force, Payload payload);
    
    /// <summary>
    /// Notice()
    /// </summary>
    /// <param name="userAgent">The user agent that received the Notice()</param>.
    /// <param name="message">Notice message</param>
    void OnNotice(UserAgent userAgent, string message);
    
    /// <summary>
    /// Notify if kicked out
    /// </summary>
    /// <param name="userAgent">The user agent that was kicked out</param>.
    /// <param name="message">Message received from admin</param>
    void OnAdminKickout(UserAgent userAgent, string message);
    
    /// <summary>
    /// Notifies the user listener when the session on the server is closed<para></para>
    /// If you receive this notification, log back in and restart.
    /// </summary>
    /// <param name="userAgent">The user agent whose session was closed</param>
    /// <param name="result">Reason the session was closed</param>
    /// <param name="payload">Additional information received from the server</param>
    void OnSessionClose(UserAgent userAgent, GameAnvil.Defines.ResultCodeSessionClose result, Payload payload);
}

UserAgent userAgent = connector.GetUserAgent(serviceId, subId);
userAgent.AddUserListener(new UserListener());
```