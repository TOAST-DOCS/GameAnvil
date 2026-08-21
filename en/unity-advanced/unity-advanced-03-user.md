<!-- pre-align:aligned sig=343635dd9ba0 -->

<a id="game-gameanvil-unity-advanced-development-guide-user"></a>
## Game > GameAnvil > Unity Advanced Development Guide > User { #game-gameanvil-unity-advanced-development-guide-user }

<a id="useragent"></a>
## UserAgent { #useragent }

The UserAgent is responsible for operations related to GameNodes on the GameAnvil server. It provides basic functionality such as login(), logout(), and room management. Based on the protocols you define, clients can message other objects through their user objects and implement different content. 

To use UserAgent, you need to create a new UserAgent using the Connector.CreateUserAgent() function. You can create multiple UserAgents, separated by ServiceName and SubId. The created UserAgent is managed internally by the Connector and can be used again using the Connector.GetUserAgent() function. 

```c#
UserAgent userAgent = ConnectHandler.getInstance().GetUserAgent(serviceName, subID);
if (userAgent == null) {
    userAgent = ConnectHandler.getInstance().CreateUserAgent(serviceName, subID);
}
```

The GameAnvil server can run multiple services simultaneously, and a UserAgent can log in to one service and operate independently of another. This means that you can create multiple UserAgents to log in to different services and use them simultaneously. It is also possible to have multiple UserAgents logged into the same service at the same time by using different SubIds. 

<a id="create"></a>
### Create { #create }

<!-- TODO: translate body -->

<a id="disable"></a>
### Disable { #disable }

<!-- TODO: translate body -->

<a id="loginlogout"></a>
### Login/Logout { #loginlogout }

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

<a id="loginlogout-login"></a>
#### Login

<!-- TODO: translate body -->

<a id="logout"></a>
### Logout { #logout }

<!-- TODO: translate body -->

<a id="logout-force-logout-notification"></a>
#### Force Logout Notification

<!-- TODO: translate body -->

<a id="create-enter-and-leave-rooms"></a>
### Create, enter, and leave rooms { #create-enter-and-leave-rooms }

This is the same as creating, entering, and leaving rooms in [Unity Basic Development Guide > UserAgent](../unity-basic/unity-basic-04-user-agent.md).

<a id="create-enter-and-leave-rooms-create-room"></a>
#### Create Room

<!-- TODO: translate body -->

<a id="create-enter-and-leave-rooms-enter-room"></a>
#### Enter Room

<!-- TODO: translate body -->

<a id="create-enter-and-leave-rooms-leave-room"></a>
#### Leave Room

<!-- TODO: translate body -->

<a id="create-enter-and-leave-rooms-notification-for-forced-to-leave-the-room"></a>
#### Notification for Forced to Leave the Room

<!-- TODO: translate body -->

<a id="create-enter-and-leave-rooms-enter-the-room-with-the-specified-name"></a>
#### Enter the room with the specified name

<!-- TODO: translate body -->

<a id="matchmaking"></a>
### Matchmaking { #matchmaking }

GameAnvil offers two types of matchmaking. One is Room Matchmaking, which performs room-by-room matching, and the other is User Matchmaking, which performs user-by-user matching. For more information, see Matchmaking in the [Unity Basic Development Guide > UserAgent](../unity-basic/unity-basic-04-user-agent.md).

<a id="matchmaking-room-matchmaking"></a>
#### Room Matchmaking

<!-- TODO: translate body -->

<a id="matchmaking-user-matchmaking"></a>
#### User Matchmaking

<!-- TODO: translate body -->

<a id="matchmaking-party-matchmaking"></a>
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

<a id="channel"></a>
### Channel { #channel }

<!-- TODO: translate body -->

<a id="channel-move-notification"></a>
#### Channel move notification

<!-- TODO: translate body -->

<a id="channel-moving-channels"></a>
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

<a id="channel-information"></a>
### Channel information { #channel-information }

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

<a id="channel-information-getchannelcountinfo"></a>
#### GetChannelCountInfo

<!-- TODO: translate body -->

<a id="channel-information-getchannelinfo"></a>
#### GetChannelInfo

<!-- TODO: translate body -->

<a id="channel-information-getallchannelcountinfo"></a>
#### GetAllChannelCountInfo

<!-- TODO: translate body -->

<a id="channel-information-getallchannelinfo"></a>
#### GetAllChannelInfo

<!-- TODO: translate body -->

