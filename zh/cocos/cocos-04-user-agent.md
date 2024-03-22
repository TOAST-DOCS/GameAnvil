## Game > GameAnvil > CocosCreator Development Guide > UserAgent

## UserAgent

UserAgent is responsible for the GameNode related tasks on GameAnvil server. It provides basic features such as login(), logout(), and room management, And directly defined protocols allow clients to send and receive messages to and from other objects through their own user objects to implement multiple pieces of content. To use the UserAgent, you have to create a new UserAgent using Connector.CreateUserAgent() function. You can create multiple User Agents separated by ServiceName and SubId. Generated UserAgent is managed internally by Connector and can be reused using the Connector.GetUserAgent() function. 

```typescript
/** 
 * Returns the user agent that corresponds to the service name and sub ID
* @paramserviceName Service name used by the user agent
* @param subId Unique id to identify user agents by service. May not be used depending on server implementation
* @returns corresponding user agent, null if none
*/ 
let userAgent = connector.GetUserAgent(serviceName, subId); 
if(userAgent == null){ 
    /**
* CreatE  User Agent 
     * @paramserviceName Name of the service that the user agent will use
     * @param subId Unique ID that can identify user agents by service. May not be used depending on server implementation
* @returns User Agent created
*/ 
    userAgent = connector.CreateUserAgent(serviceName, subId); 
}

```

GameAnvil server can run multiple services at the same time, and one UserAgent can log in to one service and act independently of each other. In other words, you can create multiple User Agents and log in to different services at the same time. If you have a different SubId, you can also log in and use multiple User Agents to the same service at the same time. 

### Login/Logout

Login can be defined as the process of creating one’s own user object in GameNode after the client connects to the server. Logout is the opposite concept of log in, in other words, removing one’s own user object from GameNode. 

When you log in, you have to enter which UserType you want to log in to which channel. If you need additional information, you can send it in Payload. 

```typescript
/** 
 * Log in to server
 * @param userType User type to use for login
 * @param payload Additional information to pass to the server. May not be used depending on the server implementation
 * @param channelId   Channel ID to login
 * @param callback Callback to receive and process login results
 */ 
userAgent.Login(userType, payload, channelId, (agent: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo)=>{ 
    /** 
     * Result of Login()  
     * @param user User agent requested Login()
     * @param resultCode Login() result code
     * @param loginInfo Login() information 
     */ 
    if (ResultCodeLogin.LOGIN_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
}) 
 
/** 
* Log out of the current service 
 * @param callback Callback to receive and process logout results
 */ 
userAgent.Logout((agent: UserAgent, resultCode: ResultCodeLogout, payload: Payload)=>{ 
    /** 
     * Result of Logout() 
     * @param user User agent requested Logout()
     * @param resultCode  Logout() result code
     * @param force Forced logout or not
     * @param payload  Additional information received from the server
     */ 
    if (ResultCodeLogout.LOGOUT_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
});
```

### Create, Join, and Leave a Room

Two or more users can create synchronized message flows through the room, in other words, all user requests are guaranteed to have a sequence in the room. Also, creating a room for one user can also have meaning depending on the content. It is up to the engine user to decide how to use the room. 

You can call CreateRoom() to create a room and enter it.

```typescript
/** 
 * Create a random room and join that room
 * @param roomType Room type to create
 * @param roomName Room name to create
 * @param payload   Additional information to pass to the server. May not be used depending on the server implementation 
 * @param callback Callback to receive and process room creation results 
 */ 
userAgent.CreateRoom(roomType, roomName, payload, (agent: UserAgent, resultCode: ResultCodeCreateRoom, roomId: number, roomName: string, payload: Payload)=>{ 
    /** 
     * Result of CreateRoom() 
     * @param user User agent requesting CreateRoom() 
     * @param resultCode CreateRoom() Result code 
     * @param roomId Room ID of room created 
     * @param roomName Name of room created 
     * @param payload Additional information received from the server. May not be used depending on the server implementation
     */ 
    if (ResultCodeCreateRoom.CREATE_ROOM_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

You can call JoinRoom() to enter the room that has already been created. 

``` typescript
/** 
 * Enter the room of the designated room ID
 *  
 * Failure if room with designated room ID is not available
 * @param roomId Room ID you want to enter
 * @param roomType Room type you want to enter
 * @param payload  Additional information to pass to the server. May not be used depending on the server implementation
 * @param callback Callback to receive and process room entry results
 */ 
userAgent.JoinRoom(roomId, roomType, payload, (agent: UserAgent, resultCode: ResultCodeJoinRoom, roomId: number, roomName: string, payload: Payload)=>{ 
    /** 
     * Result of JoinRoom() 
     * @param user User agent requested JoinRoom()   
     * @param resultCode JoinRoom() Result code 
     * @param roomId Room ID you joined 
     * @param roomName Room name you joined 
     * @param payload Additional information to pass to the server. May not be used depending on the server implementation
     */ 
    if (ResultCodeJoinRoom.JOIN_ROOM_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

You can call LeaveRoom() to leave the room that you have entered.

``` typescript
/** 
 * Leave your current room
 * @param payload Additional information to pass to the server. May not be used depending on the server implementation 
 * @param callback Callback to receive and process the results of the room leave request
 */ 
userAgent.LeaveRoom(payload, (agent: UserAgent, resultCode: ResultCodeLeaveRoom, roomId: number, payload: Payload)=>{ 
    /** 
     * Result of LeaveRoom() 
     * @param user  User agent requested LeaveRoom()   
     * @param resultCode LeaveRoom() Result code 
     * @param force force Forced to leave or not
     * @param roomId Room ID you have left 
     * @param payload Additional information received from the server. May not be used depending on the server implementation
     */ 
    if (ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

You can call NamedRoom() to entered the room that you have designated. If there is no room with the designated name, create the room and enter it.

```typescript
/** 
 * Enter the room you have designated 
 *  
 * If there is no room with the designated name, create the room and enter it.
 * @param roomName Room name you want to enter 
 * @param roomType Room type you want to enter
 * @param isParty true - a room created for party matching purpose. false – general room 
 * @param payload Additional information to pass to the server. May not be used depending on the server implementation 
 * @param callback Callback to receive and process the results of the namedroom processing
 */ 
userAgent.NamedRoom(roomName, roomType, isParty, payload, (agent: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payLoad: Payload)=>{ 
    /** 
     * Result of NamedRoom() 
     * @param user User agent requested NamedRoom()  
     * @param resultCode NamedRoom() Result code 
     * @param roomId Room ID you joined
     * @param roomName Room name you joined
     * @param created Whether or not the room entered has been created (whether or not room captain)
     * @param payload Additional information received from the server. May not be used depending on the server implementation
     */ 
    if (ResultCodeNamedRoom.NAMED_ROOM_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```



### Match Making

GameAnvil offers two types of matchmaking: room matchmaking, which performs room-to-room matching, and user matchmaking, which performs user-to-user matching.

#### Room Match Making

Room matchmaking is a method of entering a room that meets requirements. When requesting room matchmaking, if there is a room that meets the requirements, you can enter the corresponding room immediately, and if there is no room that meets the requirements, you can create a new room and enter. 

You can call MatchRoom() to request room matchmaking. If you don't have a room that meets the conditions, you can also create a room and enter that room 

```typescript
/** 
 * Request room match 
 *  
 *If there’s no room, can create a random room and enter that room
 * @param isCreateRoomIfNotJoinRoom true - If no avilable room meets the conditions, you can also create a room and enter that room. false – failure if no room to enter 
 * @param isMoveRoomIfJoinedRoom true - Go to another room if requested while joining the room. false - Failure when requested while joining the room
 * @param roomType Room type to enter 
 * @param payload   Additional information to pass to the server. May not be used depending on the server implementation 
 * @param leaveRoomPayload  Additional information to pass when leaving the room entered. May not be used depending on the server implementation
 * @param callback Callback to receive and process room match results
 */ 
userAgent.MatchRoom(isCreateRoomIfNotJoinRoom, isMoveRoomIfJoinedRoom, roomType, payload, leaveRoomPayload, (agent: UserAgent, resultCode: ResultCodeMatchRoom, roomId: number, roomName: string, created: boolean, payload: Payload) => { 
    /** 
     * Outcome of MatchRoom() 
     * @param user User agent requested MatchRoom()  
     * @param resultCode MatchRoom() Result code 
     * @param roomId Room ID that has matched 
     * @param roomName Room name that has matched
     * @param created Whether or not a matched room is created (whether or not room captain)
* @param payload Additional information received from the server. May not be used depending on the server implementation
     */ 
    if (ResultCodeMatchRoom.MATCH_ROOM_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

#### User Matchmaking

User catchmaking is a method of creating a user pool, finding users who meet the conditions, and entering a newly created room. If the number of users who meet the conditions is not enough in the user pool, it may take time for the matchmaking to be completed, and if the matchmaking is not completed within time, the match may be canceled due to timeout. 

You can call MatchUserStart() to start User matchmaking. The request may fail depending on the server's conditions, including the case you have already entered the room. 

```typescript
/** 
 * Request user match 
 *  
 * May fail depending on the server's conditions, including the case once you have already entered the room.
 *  
 * if the matching is successful, you can receive notifications through OnMatchUserDone() 
 * @param roomType requesting match 
 * @param payload Additional information to pass to the server. May not be used depending on the server implementation 
 * @param callback Callback to receive and process user match request results
 */ 
userAgent.MatchUserStart(roomType, payload, (agent: UserAgent, resultCode: ResultCodeMatchUserStart, payLoad: Payload)=>{ 
    /** 
     * Result of MatchUserStart() 
     * @param user User agent requested MatchUserStart()  
     * @param resultCode MatchUserStart() Result code 
     * @param payload  Additional information received from the server. May not be used depending on the server implementation
     */ 
    if (ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

When matching is succesful, you can receive notifications through IUserListener.OnMatchUserDone, and when matching is not successful within time, you can receive notifications through IUserListener.OnMatchUserTimeout.

```typescript
class MatchUserListener implements IUserListener { 
    /** 
     * MatchUserStart(), matching result of MatchPartyStart() 
     * @param user User agent requested matching  
     * @param resultCode matching Result code 
     * @param Whether or not the room created . If true, it means that the UserAgent that requested matching created the room. It can be used to decide the room captain, etc. 
     * @param roomId Room ID of matched room 
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnMatchUserDone(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void { 
        if (ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS == resultCode) { 
            // success 
        } else { 
            // failure 
        } 
    } 
 
    /** 
     * MatchUserStart(), MatchPartyStart() matching time out happens 
     * @param user User agent requested matching  
     */ 
    OnMatchUserTimeout(user: UserAgent): void { 
        // Time out 
    } 
} 
userAgent.AddListener(new MatchUserListener());
```

You can call MatchUserCancel() to cancel user match making. When you are not requesting a match, you may fail when the matchmaking has already been successful or if Timeout has occurred. 

```typescript
/** 
 * Request to cancel user match 
 *  
 * Fail if not requesting a match, or matchmaking has already been successful or if Timeout has occurred.
 * @param roomType Room type requested match 
 * @param callback Callback to receive and process user match cancellation request results
 */ 
userAgent.MatchUserCancel(roomType, (agent: UserAgent, resultCode: ResultCodeMatchUserCancel) => { 
    /** 
     * Result of MatchUserCancel() 
     * @param user User agent requested MatchUserCancel()
     * @param resultCode MatchUserCancel() Result code 
     */ 
    if (ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

#### Party Match Making

Party matchmaking is a special form of user matchmaking, in which two or more users are grouped into one group and registered in the user pool, and other users who meet the conditions are found and allowed to enter the newly created room. Groups of users can always enter the same room, and users who enter together can be groups or individuals depending on the server's matchmaker implementation. 

In order to do party match, first, call NamedRoom() so that users who are going to be party should come together in one room. 

```typescript
/** 
 * Enter the room with the designated name
 *  
 * if no room with the designated name is found create a room with the  name and enter it 
 * @param roomName Room name want to enter  
 * @param roomType Room type want to enter  
 * @param isParty true - a room created for party matching purpose. false – general room
 * @param payload  Additional information to pass to the server. May not be used depending on the server implementation
 * @param callback Callback to receive and process the results of the namedroom processing
 */ 
let isParty = true; 
userAgent.NamedRoom(roomName, roomType, isParty, payload, (agent: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payLoad: Payload) => { 
    /** 
     * Result of NamedRoom() 
     * @param user User agent requested NamedRoom()  
     * @param resultCode NamedRoom() Result code 
     * @param roomId Room ID you joined
     * @param roomName Room name you joined  
     * @param created Whether or not the room entered has been created (whether or not room captain)
     * @param payload  Additional information received from the server. May not be used depending on the server implementation
     */ 
    if (ResultCodeNamedRoom.NAMED_ROOM_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

You can call MatchPartyStart() to start party match making. The request may fail depending on the server's conditions, including when you have already entered the room 

```typescript
/** 
 * Request party match 
 * @param roomType Room type to request match 
 * @param payload   Additional information to pass to the server. May not be used depending on the server implementation
 * @param callback Callback to receive and process party match request results
 */ 
userAgent.MatchPartyStart(roomType, payload, (agent: UserAgent, resultCode: ResultCodeMatchPartyStart, payload: Payload) => { 
    /** 
     * Result of MatchPartyStart() 
     * @param user User agent requested MatchPartyStart()  
     * @param resultCode MatchPartyStart() Result code 
     * @param payload Additional information to pass to the server. May not be used depending on the server implementation. 
     */ 
    if (ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

When matching is successful, you can receive notifications through IUserListener.OnMatchUserDone that used for user match making, and when matching is not successful within time, you can also receive notifications through IUserListener.OnMatchUserTimeout.

```typescript
class MatchPartyListener implements IUserListener { 
    /** 
     * MatchUserStart(), matching result of MatchPartyStart() 
     * @param user User agent requsted matching  
     * @param resultCode matching Result code 
     * @param created Whether or not the room created. If true, it means that the UserAgent that requested matching created the room. It can be used to decide the room captain, etc.
     * @param roomId Room ID of matched room
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnMatchUserDone(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void { 
        if (ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS == resultCode) { 
            // success 
        } else { 
            // failure 
        } 
    } 
 
    /** 
     * MatchUserStart(), Occurs when matching time out MatchPartyStart()
     * @param user User agent requested matching  
     */ 
    OnMatchUserTimeout(user: UserAgent): void { 
        // Time out 
    } 
} 
userAgent.AddListener(new MatchPartyListener());
```

You can call MatchPartyCancel() to cancel user match making. When you are not requesting a match, you may fail when the matchmaking has already been successful or if Timeout has occurred. 

```typescript
/** 
 * Cancel to request party match 
 * @param roomType Room type requested match 
 * @param callback  Callback to receive and process party match cancellation invitation results 
 */ 
userAgent.MatchPartyCancel(roomType, (agent: UserAgent, resultCode: ResultCodeMatchPartyCancel) => { 
    /** 
     * Result of MatchPartyCancel() 
     * @param user User agent requested MatchPartyCancel()  
     * @param resultCode Result code of MatchPartyCancel()   
  */ 
    if (ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```

#### Move a Channel

In some cases, channel move can occur as a result of matchmaking. If channel move occur, you can receive notifications through the IUserListener.OnMoveChannel.

```typescript
class MoveChannelListener implements IUserListener { 
    /** 
     * Results of MoveChannel() or notification of forced channel move by the server
     * @param user User agent that moved channel  
     * @param resultCode  Result code of channel move 
     * @param force Whether or not forced to move the channel by the server  
     * @param channelId  channel ID that moved 
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnMoveChannel?(user: UserAgent, resultCode: ResultCodeMoveChannel, force: boolean, channelId: string, payload: Payload): void { 
        // Channel Move
    } 
} 
userAgent.AddListener(new MoveChannelListener());
```



### Channel Information

You can configure channels freely with GameAnvil setting. This channel configuration can be used in a fixed form agreed between the server and the client in advance, but can also be changed in various ways depending on the situation. UserAgent provides several functions to obtain this changed channel information or to move the channel. 

GetChannelCountInfo() can request and receive the channel's count information (count of user and room). 

```typescript
/** 
 * Request the count of rooms and users of the connected channel
 *  
 * Available when supported by the server
 * @param callback Callback to receive and process the results of requesting the count of users and rooms in the channel 
 */ 
userAgent.GetChannelCountInfo((agent: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo) => { 
    /** 
     * Result of GetChannelCountInfo() 
     * @param user User agent requested GetChannelCountInfo() 
     * @param resultCode Result code of GetChannelCountInfo()
     * @param channelCountInfo Count of channel users and rooms 
     */ 
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
}); 
 
/** 
* Request the count of rooms and users of specific channel
 *  
 * Available when supported by the server
 * @param serviceName Service name to request channel information
 * @param channelId Channel ID to request channel information
 * @param callback Callback to receive and process the results of the channel's user and room count request
 */ 
userAgent.GetChannelCountInfo(serviceName, channelId, (agent: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo) => { 
    /** 
     * Result of GetChannelCountInfo() 
     * @param connection Connection agent requested GetChannelCountInfo()
     * @param resultCode Result code of GetChannelCountInfo()
     * @param channelCountInfo Count of channel user and room     .
     */ 
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) { 
        // Success 
    } else { 
        // Failure 
    } 
});
```

GetChannelInfo() can request and receive information about the channel (customization). 

```typescript
/** 
 * Request information of connected channel
 *  
 * Available when supported by the server
 * @param callback Callback to receive and process channel information request results 
*/ 
userAgent.GetChannelInfo((agent: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload) => { 
    /** 
     * Result of GetChannelInfo() 
     * @param user User agent requested GetChannelInfo()
     * @param resultCode GetChannelInfo()의 Result code 
     * @param channelInfo Channel Information 
     */ 
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) { 
        // Success 
    } else { 
        // Failure 
    } 
}); 
 
/** 
 * Request information of specific channel
 * Available when supported by the server
 * @param serviceName Service name to request channel information
 * @param channelId Channel ID to request channel information
 * @param callback Callback to receive and process channel information request results
 */ 
userAgent.GetChannelInfo(serviceName, channelId, (agent: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload) => { 
    /** 
     * Result of GetChannelInfo() 
     * @param connection Connection agent requested GetChannelInfo() 
     * @param resultCode GetChannelInfo()의 Result code 
     * @param channelInfo Channel Information
     */ 
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) { 
        // Success 
    } else { 
        // Failure 
    } 
});
```

GetAllChannelCountInfo() can request and receive count information (user and room count) for all the channels in the service, handing over the service name and the callback to handle the response as a factor. 

```typescript
/** 
 * Request information of connected channel
 *  
 * Available when supported by the server
 * @param callback Callback to receive and process channel information request results 
 */ 
userAgent.GetAllChannelCountInfo((agent: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo) => { 
    /** 
     * Result of GetAllChannelCountInfo()
     * @param connection Connection agent requested GetAllChannelCountInfo()
     * @param resultCode Result code of GetAllChannelCountInfo()
     * @param channelInfoList Count of channel users and rooms 
     */ 
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) { 
        // Success 
    } else { 
        // Failure 
    } 
}); 
 
/** 
 * Request the count of users and rooms for all channels in a specific service
 *  
 * Available when supported by the server
 * @param serviceName Service name to request channel information 
 * @param callback Callback to receive and process the results of requests for the count of users and rooms on all channels
 */ 
userAgent.GetAllChannelCountInfo(serviceName, (agent: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo) => { 
    /** 
     * Result of GetAllChannelCountInfo()
     * @param connection Connection agent requested  GetAllChannelCountInfo()
     * @param resultCode Result code of  GetAllChannelCountInfo()
     * @param channelInfoList Count of channel users and rooms 
     */ 
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) { 
        // Success 
    } else { 
        // Failure 
    } 
});
```

GetAllChannelInfo() can request and receive information (customization) about all channels in the service, handing over the service name and the callback to handle the response as a factor. 

```typescript
/** 
 * Request all information of connected channel
 *  
 * Available when supported by the server
 * @param callback  Callback to receive and process all channel information request results
 */ 
userAgent.GetAllChannelInfo((agent: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo) => { 
    /** 
     * Result of GetAllChannelInfo() 
     * @param user User agent requested GetAllChannelInfo()
     * @param resultCode Result code of GetAllChannelInfo()
     * @param allChannelInfo Channel Information
     */ 
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) { 
        // Success 
    } else { 
        // Failure 
    } 
}) 
 
/** 
 * Request all information of specific channel 
 *  
 * Available when supported by the server
 * @param serviceName Service name to request channel information 
 * @param callback Callback to receive and process all channel information request results 
 */ 
userAgent.GetAllChannelInfo(serviceName, (agent: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo) => { 
    /** 
     * Result of GetAllChannelInfo() 
     * @param user User agent requested GetAllChannelInfo()
     * @param resultCode GetAllChannelInfo()의 Result code 
     * @param allChannelInfo Channel Information
     */ 
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) { 
        // Success 
    } else { 
        // Failure 
    } 
})
```

You can call MoveChannel() to move to another channel within the service. 

```typescript
/** 
 * Go to the designated channel
 * @param channelId Channel ID to move 
 * @param payload   Additional information to pass to the server. May not be used depending on the server implementation
 * @param callback Callback to receive and process channel move results
 */ 
userAgent.MoveChannel(channelId, payload, (agent: UserAgent, resultCode: ResultCodeMoveChannel, channelId: string, payload: Payload) => { 
    /** 
     * Result of MoveChannel() or the notification of forced channel movement on the server     
* @param user User agent that moved the channel
     * @param resultCode Channel move Result code 
     * @param force Whether or not the server forced the channel to move
     * @param channelId Channel ID that moved 
     * @param payload   Additional information received from the server. May not be used depending on the server implementation     
     */ 
    if (ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS == resultCode) { 
        // success 
    } else { 
        // failure 
    } 
})
```



### Listener

IUserListener is an interface that defines the results or notifications of all actions in UserAgent. If you register a listener that implements this interface as UserAgent.AddUserListener(), you can receive a response from the registered listener. All functions in the IUserListener are optional, so you can implement and use only the functions you use. 

```typescript
class UserListener implements IUserListener { 
    /** 
     * Result of Login()
     * @param user User agent requested Login()
     * @param resultCode Login() Result code  
     * @param loginInfo Login() information 
     */ 
    OnLogin?(user: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo): void { } 
    /** 
     * Result of Logout()
     * @param user User agent requested Logout()
     * @param resultCode  Logout() Result code 
     * @param force Forced logout or not
     * @param payload  Additional information received from the server
     */ 
    OnLogout?(user: UserAgent, resultCode: ResultCodeLogout, force: boolean, payload: Payload): void { } 
    /** 
     * Result of MatchRoom() 
     * @param user User agent requested MatchRoom()  
     * @param resultCode MatchRoom() Result code 
     * @param roomId Room ID that has matched
     * @param roomName Room name that has matched
     * @param created Whether or not the room entered has been created (whether or not room captain)
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnMatchRoom?(user: UserAgent, resultCode: ResultCodeMatchRoom, roomId: number, roomName: string, created: boolean, payload: Payload): void { } 
    /** 
     * Result of CreateRoom() 
     * @param user User agent requested CreateRoom()
     * @param resultCode CreateRoom() Result code 
     * @param roomId Room ID of room created  
     * @param roomName Name of room created  
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnCreateRoom?(user: UserAgent, resultCode: ResultCodeCreateRoom, roomId: number, roomName: string, payload: Payload): void { } 
    /** 
     * Result of JoinRoom() 
     * @param user User agent requsted JoinRoom()
     * @param resultCode JoinRoom() Result code 
     * @param roomId Room ID you joined
     * @param roomName Room name you joined 
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnJoinRoom?(user: UserAgent, resultCode: ResultCodeJoinRoom, roomId: number, roomName: string, payload: Payload): void { } 
    /** 
     * Result of NamedRoom()
     * @param user User agent requested NamedRoom()   
     * @param resultCode NamedRoom() Result code 
     * @param roomId Room ID you joined
     * @param roomName Room name you joined  
     * @param created Whether or not the room entered has been created (whether or not room captain)
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnNamedRoom?(user: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payload: Payload): void { } 
    /** 
     * Result of LeaveRoom()  
     * @param user Usergent requested LeaveRoom()     
     * @param resultCode LeaveRoom() Result code 
     * @param force Forced to leave or not
     * @param roomId Room ID you have left
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnLeaveRoom?(user: UserAgent, resultCode: ResultCodeLeaveRoom, force: boolean, roomId: number, payload: Payload): void { } 
    /** 
     * Result of MatchUserStart() 
     * @param user User agent requested MatchUserStart()
     * @param resultCode MatchUserStart() Result code 
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnMatchUserStart?(user: UserAgent, resultCode: ResultCodeMatchUserStart, payload: Payload): void { } 
    /** 
     * Result of MatchUserCancel()
     * @param user User agent requested MatchUserCancel()  
     * @param resultCode MatchUserCancel() Result code 
     */ 
    OnMatchUserCancel?(user: UserAgent, resultCode: ResultCodeMatchUserCancel): void { } 
    /** 
     * MatchUserStart(), Matching result of MatchPartyStart() 
     * @param user User agent requested matching
     * @param resultCode matching Result code 
     * @param created Whether or not the room created. If true, it means that the UserAgent that requested matching created the room. It can be used to decide the room captain, etc.
     * @param roomId Room ID of matched room
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
  OnMatchUserDone?(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void { } 
    /** 
     * MatchUserStart(),Occurs during timeout of MatchPartyStart() matching
     * @param user User agent requested matching
     */ 
    OnMatchUserTimeout?(user: UserAgent): void { } 
    /** 
     * Result of MatchPartyStart()
     * @param user User agent requested MatchPartyStart()
     * @param resultCode MatchPartyStart() Result code 
     * @param payload Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnMatchPartyStart?(user: UserAgent, resultCode: ResultCodeMatchPartyStart, payload: Payload): void { } 
    /** 
     * Result of MatchPartyCancel() 
     * @param user User agent requested MatchPartyCancel()
     * @param resultCode Result code of 
MatchPartyCancel()     
*/ 
    OnMatchPartyCancel?(user: UserAgent, resultCode: ResultCodeMatchPartyCancel): void { } 
    /** 
     * Results of MoveChannel() or notification of forced channel move by the server
     * @param user User agent that moved channel  
     * @param resultCode Result code of channel move
     * @param force Whether or not forced to move the channel by the server
     * @param channelId channel ID that moved
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
    OnMoveChannel?(user: UserAgent, resultCode: ResultCodeMoveChannel, force: boolean, channelId: string, payload: Payload): void { } 
    /** 
     * Result of Snapshot()  
     * @param user User agent requested Snapshot()     
     * @param resultCode Snapshot()의 Result code 
     * @param payload   Additional information received from the server. May not be used depending on the server implementation
     */ 
     OnSnapshot?(user: UserAgent, resultCode: ResultCodeSnapshot, payload: Payload): void { } 
    /** 
     * Result of GetChannelInfo()  
     * @param user User agent requested GetChannelInfo()
     * @param resultCode GetChannelInfo()의 Result code 
     * @param channelInfo Channel Information
     */ 
    OnChannelInfo?(user: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload): void { } 
    /** 
     * Result of GetChannelCountInfo()  
     * @param user User agent requested GetChannelCountInfo()
     * @param resultCode Result code of GetChannelCountInfo()
     * @param channelCountInfo Count of channel users and rooms 
     */ 
    OnChannelCountInfo?(user: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo): void { } 
    /** 
     * Result of GetAllChannelInfo() 
     * @param user User agent requested GetAllChannelInfo()
     * @param resultCode GetAllChannelInfo()의 Result code 
     * @param allChannelInfo Channel Information
     */ 
    OnAllChannelInfo?(user: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo): void { } 
    /** 
     * Result of GetAllChannelCountInfo() 
     * @param user User agent requested GetAllChannelCountInfo()
     * @param resultCode Result code of GetAllChannelCountInfo()
     * @param channelInfoList Count of channel users and rooms 
     */ 
    OnAllChannelCountInfo?(user: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo): void { } 
    /**
     * Error occurred
     * @param user User Agent that error occurred
     * @param errorCode Error code 
     * @param msgName Feature or message that error has failed     
     */ 
    OnErrorCommand?(user: UserAgent, errorCode: ErrorCode, msgName: string): void { } 
    /** 
     * Notification
     * @param user User Agent that get notification
     * @param message notification message
     */ 
    OnNotice?(user: UserAgent, message: string): void { } 
    /** 
     * Notify if Admin Kickout()
     * @param user User agent that Kickout()     
     * @param message A message that Admin has passed
     */ 
    OnAdminKickoutNoti?(user: UserAgent, message: string): void { } 
    /**
     * Notify if the server's session is closed in User listener; if received this alarm, you need to log in again and restart
     * @param user User agent whose session is closed
     * @param resultCode Reason why the session isclosed
     * @param paload Additional information. May not be used depending on the server implementation 
     */ 
    OnSessionCloseNoti?(user: UserAgent, resultCode: ResultCodeSessionClose, payload: Payload): void { } 
} 
userAgent.AddListener(new UserListener());
```

