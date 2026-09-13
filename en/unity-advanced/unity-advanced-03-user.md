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

Call Login() to log in to the service. When logging in, you must specify the UserType and the channel to log in to. If additional information is needed, you can include it in a Payload.

```c#
public async void Login()
{
    try
    {
        Payload loginPayload = new Payload(new Protocol.LoginData());
        Result<ResultCodeLogin, LoginResult> result = await user.Login("UserType", "ChannelId", loginPayload);
        if(result.ResultCode == ResultCodeLogin.LOGIN_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    }
    catch (Exception e)
    {
        // Exception
    }
}
```

Login() has the following four parameters:

| Type | Name | Description |
|---------|----------------|------------------------------------------------------|
| String | userType | The type of user to create upon login. |
| String | channelId | The ID of the channel to log in to. |
| Payload | requestPayload | Additional information required by the server's user code to process the login request. (default = null) |

It returns Result<ResultCodeLogin, LoginResult> as the response. You can check whether the request was successful by examining the value of the ResultCode field. If the login is successful, the value of the ResultCode field becomes ResultCodeLogin.LOGIN_SUCCESS; otherwise, the login has failed. You can obtain the LoginResult from the request result via the Data field. This allows you to retrieve information about the logged-in user, and depending on the server implementation, you may also obtain additional information.

The details of ResultCodeLogin are as follows:

| Name                               | Value | Description                                                                                             |
|------------------------------------|-------|---------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                        | -2    | Packet parsing error. May occur when the server and client versions differ.                             |
| TIMEOUT                            | -1    | Timeout. Response to the request did not arrive within the specified time.                              |
| SYSTEM_ERROR                       | 1     | Server system error. Failed due to an unknown server error.                                             |
| INVALID_PROTOCOL                   | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| JOIN_ROOM_SUCCESS                  | 0     | Success.                                                                                                |
| JOIN_ROOM_FAIL_CONTENT             | 701   | Failed. Rejected by user code.                                                                          |
| JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702   | Failed. The requested room does not exist.                                                              |
| JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM | 703   | Failed. Already in a room.                                                                              |
| JOIN_ROOM_FAIL_ALREADY_FULL        | 704   | Failed. The requested room is full.                                                                     |
| JOIN_ROOM_FAIL_ROOM_MATCH          | 705   | Failed. An issue occurred during room matchmaking.                                                      |

The details of LoginResult are as follows:

| Type    | Name         | Description                                               |
|---------|--------------|-----------------------------------------------------------|
| int     | UserId       | Logged-in user ID                                         |
| string  | UserType     | Logged-in user type                                       |
| string  | ServiceName  | Name of the logged-in service                             |
| string  | ChannelId    | Logged-in channel ID                                      |
| Payload | Payload      | Additional information received from the server           |
| bool    | IsRelogined  | Whether re-logged in                                      |
| bool    | IsJoinedRoom | Whether joined to a room                                  |
| int     | RoomId       | ID of the room the user belongs to                        |
| string  | RoomName     | Name of the room the user belongs to                      |
| Payload | RoomPayload  | Additional information about the room the user belongs to |
| bool    | IsMatching   | Whether matching has been requested                       |

<a id="logout"></a>
### Logout { #logout }

Call `Logout()` to log out of the service.

```c#
public async void Logout()
{
    try
    {
        Payload logoutPayload = new Payload(new Protocol.LogoutData());
        Result<ResultCodeLogout, LogoutResult> result = await user.Logout(logoutPayload);
        if (result.ResultCode == ResultCodeLogout.LOGOUT_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    }
    catch (Exception e)
    {
        // Exception
    }
}
```
`Logout()` has the following 1 parameter.

| Type     | Name    | Description                                                                                              |
|----------|---------|----------------------------------------------------------------------------------------------------------|
| Payload? | payload | Additional information required by the user code on the server that processes the logout request. (default = null) |

Returns `Result<ResultCodeLogout, LogoutResult>` as a response. You can check whether the request succeeded by checking the value of the `ResultCode` field. If the logout succeeds, the value of the `ResultCode` field is `ResultCodeLogout.LOGOUT_SUCCESS`; otherwise, the logout has failed. You can retrieve the `LogoutResult` of the request through the `Data` field. Depending on the server implementation, you may also get additional information through the `Payload` field of `LogoutResult`.

The details of `ResultCodeLogout` are as follows.

| Name                | Value | Description                                                                                       |
|---------------------|-------|---------------------------------------------------------------------------------------------------|
| PARSE_ERROR         | -2    | Packet parsing error. This may occur when the server and client versions differ.                  |
| TIMEOUT             | -1    | Timeout. The response to the request did not arrive within the specified time.                    |
| SYSTEM_ERROR        | 1     | Server system error. Failed due to an unknown server error.                                       |
| INVALID_PROTOCOL    | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| LOGOUT_SUCCESS      | 0     | Success.                                                                                          |
| LOGOUT_FAIL_CONTENT | 401   | Failed. Rejected by the user code.                                                                |

<a id="logout-force-logout-notification"></a>
#### Force Logout Notification

<!-- TODO: translate body -->

<a id="create-enter-and-leave-rooms"></a>
### Create, enter, and leave rooms { #create-enter-and-leave-rooms }

This is the same as creating, entering, and leaving rooms in [Unity Basic Development Guide > UserAgent](../unity-basic/unity-basic-04-user-agent.md).

<a id="create-enter-and-leave-rooms-create-room"></a>
#### Create Room

Call `CreateRoom()` to create a room and enter it.

```c#
public async void CreateRoom()
{
    try
    {
        Payload createRoomPayload = new Payload(new Protocol.CreateRoomData());
        Result<ResultCodeCreateRoom, CreatedRoomResult> result = await user.CreateRoom("RoomName", "RoomType", "MatchingGroup", createRoomPayload);
        if (result.ResultCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    }
    catch(Exception e)
    {
        // Exception
    }
}
```

`CreateRoom()` has the following four parameters:

| Type    | Name          | Description                                                                                          |
|---------|---------------|------------------------------------------------------------------------------------------------------|
| String  | roomName      | Name of the room to create. Enter `string.Empty` (an empty string) if not used.                     |
| String  | roomType      | Type of the room to create. Enter a room type registered on the server.                              |
| String  | matchingGroup | Name of the matching group to use for matching. Enter `string.Empty` (an empty string) if not used. |
| Payload | payload       | Additional information required by the user code on the server that processes the room creation request. (default = null) |

It returns `Result<ResultCodeCreateRoom, CreatedRoomResult>` as the response. Check the value of the `ResultCode` field to determine whether the call succeeded. If `CreateRoom` succeeds, the `ResultCode` field is set to `ResultCodeCreateRoom.CREATE_ROOM_SUCCESS`; otherwise, room creation has failed. You can obtain the `CreatedRoomResult` from the request result via the `Data` field. This allows you to retrieve information about the created room, and, depending on the server implementation, you may also receive additional information.

The details of `ResultCodeCreateRoom` are as follows:

| Name                                 | Value | Description                                                                                              |
|--------------------------------------|-------|----------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                          | -2    | Packet parsing error. May occur when the server and client versions differ.                              |
| TIMEOUT                              | -1    | Timeout. The response to the request did not arrive within the allotted time.                            |
| SYSTEM_ERROR                         | 1     | Server system error. Failed due to an unknown server error.                                              |
| INVALID_PROTOCOL                     | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CREATE_ROOM_SUCCESS                  | 0     | Success.                                                                                                 |
| CREATE_ROOM_FAIL_CONTENT             | 601   | Failed. Rejected by user code.                                                                           |
| CREATE_ROOM_FAIL_ALREADY_JOINED_ROOM | 602   | Failed. Already in a room.                                                                               |
| CREATE_ROOM_FAIL_CREATE_ROOM_ID      | 603   | Failed. Room ID creation failed.                                                                         |
| CREATE_ROOM_FAIL_CREATE_ROOM         | 604   | Failed. Room creation failed.                                                                            |

The details of `CreatedRoomResult` are as follows:

| Type    | Name     | Description                                   |
|---------|----------|-----------------------------------------------|
| int     | RoomId   | ID of the created room.                       |
| String? | RoomName | Name of the created room.                     |
| Payload | payload  | Additional information required by the client. |

<a id="create-enter-and-leave-rooms-enter-room"></a>
#### Enter Room

Call JoinRoom() to enter a room that has already been created.

``` c#
public async void JoinRoom()
{
    try
    {
        Payload joinRoomPayload = new Payload(new Protocol.JoinRoomData());
        Result<ResultCodeJoinRoom, JoinRoomResult> result = await user.JoinRoom("RoomType", roomId, "MatchingUserCategory", joinRoomPayload);
        if(result.ResultCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    }
    catch (Exception e)
    {
        // Exception
    }
}
```

JoinRoom() has the following four parameters:

| Type    | Name                 | Description                                                                                                                                                                                                                                                                                                                        |
|---------|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String  | roomType             | The type of the room to enter.                                                                                                                                                                                                                                                                                                     |
| int     | roomId               | The ID of the room to enter.                                                                                                                                                                                                                                                                                                       |
| String  | matchingUserCategory | The matchingUserCategory to use in the room to enter. Enter string.Empty (empty string) if not used. <br/>Each room can divide users by category and apply a user count limit per category. JoinRoom may fail if the current number of users for the specified matchingUserCategory is at its maximum. |
| Payload | payload              | Additional information required by the user code on the server that will process the room entry request. (default = null)                                                                                                                                                                                                          |

The response returns Result<ResultCodeJoinRoom, JoinRoomResult>. You can check whether the request succeeded by examining the value of the ResultCode field. If JoinRoom succeeds, the ResultCode field is set to ResultCodeJoinRoom.JOIN_ROOM_SUCCESS; otherwise, room entry has failed. You can retrieve the JoinRoomResult from the Data field. This allows you to get information about the room entered, and you may also receive additional information depending on the server implementation.

The details of ResultCodeJoinRoom are as follows:

| Code Name                          | Value | Description                                                                                   |
|------------------------------------|-------|-----------------------------------------------------------------------------------------------|
| PARSE_ERROR                        | -2    | Packet parsing error. May occur when the server and client versions differ.                   |
| TIMEOUT                            | -1    | Timeout. No response to the request within the specified time.                                |
| SYSTEM_ERROR                       | 1     | Server system error. Failed due to an unknown error on the server.                            |
| INVALID_PROTOCOL                   | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| JOIN_ROOM_SUCCESS                  | 0     | Success.                                                                                      |
| JOIN_ROOM_FAIL_CONTENT             | 701   | Failed. Rejected by the user code.                                                            |
| JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702   | Failed. The requested room does not exist.                                                    |
| JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM | 703   | Failed. Already in a room.                                                                    |
| JOIN_ROOM_FAIL_ALREADY_FULL        | 704   | Failed. The requested room is full.                                                           |
| JOIN_ROOM_FAIL_ROOM_MATCH          | 705   | Failed. An issue occurred during room matchmaking.                                            |

The details of JoinRoomResult are as follows:

| Type    | Name     | Description                                    |
|---------|----------|------------------------------------------------|
| int     | RoomId   | The ID of the room entered.                    |
| String? | RoomName | The name of the room entered.                  |
| Payload | payload  | Additional information required by the client. |

<a id="create-enter-and-leave-rooms-leave-room"></a>
#### Leave Room

You can leave a room that you have entered by calling LeaveRoom().

``` c#
public async void LeaveRoom()
{
    try
    {
        Payload leaveRoomPayload = new Payload(new Protocol.LeaveRoomData());
        Result<ResultCodeLeaveRoom, Payload> result = await user.LeaveRoom(leaveRoomPayload);
        if (result.ResultCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    }
    catch (Exception e)
    {
        // Exception
    }
}
```

LeaveRoom() has the following parameter:

| Type    | Name    | Description                                                                                           |
|---------|---------|-------------------------------------------------------------------------------------------------------|
| Payload | payload | Additional information required by the user code on the server that processes the room leave request. (default = null) |

The response returns Result<ResultCodeLeaveRoom, Payload>. You can check whether the call succeeded by examining the value of the ResultCode field. If LeaveRoom succeeds, the ResultCode field is set to ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS; otherwise, the room leave request has failed. Depending on the server implementation, you may also receive additional information through the Payload in the Data field.

The details of ResultCodeLeaveRoom are as follows:

| Name                    | Value | Description                                                                                         |
|-------------------------|-------|-----------------------------------------------------------------------------------------------------|
| PARSE_ERROR             | -2    | Packet parsing error. This may occur when the server and client versions differ.                    |
| TIMEOUT                 | -1    | Timeout. A response to the request was not received within the specified time.                      |
| SYSTEM_ERROR            | 1     | Server system error. The request failed due to an unknown server error.                             |
| INVALID_PROTOCOL        | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| LEAVE_ROOM_SUCCESS      | 0     | Success.                                                                                            |
| LEAVE_ROOM_FAIL_CONTENT | 801   | Failed. Rejected by user code.                                                                      |

<a id="create-enter-and-leave-rooms-notification-for-forced-to-leave-the-room"></a>
#### Notification for Forced to Leave the Room

<!-- TODO: translate body -->

<a id="create-enter-and-leave-rooms-enter-the-room-with-the-specified-name"></a>
#### Enter the room with the specified name

You can call `NamedRoom()` to enter a room with the specified name. If a room with the specified name does not exist, a room is created and then entered.

```c#
public async void NamedRoom()
{
    try
    {
        bool isParty = false;
        Payload namedRoomPayload = new Payload(new Protocol.NamedRoomData());
        Result<ResultCodeNamedRoom, NamedRoomResult> result = await user.NamedRoom("RoomName", "RoomType", isParty, namedRoomPayload);
        if (result.ResultCode == ResultCodeNamedRoom.NAMED_ROOM_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

`NamedRoom()` has the following four parameters.

| Type    | Name     | Description                                                                                                                                                                                       |
|---------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String  | roomType | Type of the room to enter or create.                                                                                                                                                              |
| String  | roomName | Name of the room to enter or create.                                                                                                                                                              |
| bool    | isParty  | Whether the room is for party matchmaking.<br/>Enter true if you create a room for users connected to the same party to wait together until the party matchmaking is complete. |
| Payload | payload  | Additional information required from the user code of the server to process the entry or creation request. (default = null)                                                                       |

The response returns `Result<ResultCodeNamedRoom, NamedRoomResult>`. You can check whether the request was successful by examining the value of the `ResultCode` field. If NamedRoom succeeds, the `ResultCode` field is set to `ResultCodeNamedRoom.NAMED_ROOM_SUCCESS`; otherwise, the entry or creation has failed. You can obtain the `NamedRoomResult` from the request result through the `Data` field. This allows you to retrieve information about the room that was entered or created, and you may also obtain additional information depending on the Server Implementation.

The details of `ResultCodeNamedRoom` are as follows.

| Name                                | Value | Description                                                                                                                                     |
|-------------------------------------|-------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                         | -2    | Packet parsing error. This may occur when the server and client versions differ.                                                                |
| TIMEOUT                             | -1    | Timeout. A response to the request did not arrive within the specified time.                                                                    |
| SYSTEM_ERROR                        | 1     | Server system error. Failed due to an unknown server error.                                                                                     |
| INVALID_PROTOCOL                    | 2     | Protocol not registered on server. A protocol not registered in the additional information was used.                                            |
| NAMED_ROOM_SUCCESS                  | 0     | Success.                                                                                                                                        |
| NAMED_ROOM_FAIL_CONTENT             | 701   | Failed. Rejected by user code.                                                                                                                  |
| NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702   | Failed. The room does not exist.<br/>This may occur when all users leave the room while the room entry is being processed.                      |
| NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM | 703   | Failed. Already in a room.                                                                                                                      |
| NAMED_ROOM_FAIL_INVALID_ROOM_NAME   | 704   | Failed. An invalid room name was requested.                                                                                                     |
| NAMED_ROOM_FAIL_CREATE_ROOM         | 705   | Failed. Room creation failed.                                                                                                                   |

The details of `NamedRoomResult` are as follows.

| Type    | Name     | Description                                      |
|---------|----------|--------------------------------------------------|
| bool    | Created  | Whether a new room was created.                  |
| int     | RoomId   | ID of the room entered.                          |
| String? | RoomName | Name of the room entered.                        |
| Payload | payload  | Additional information required by the client.   |

<a id="matchmaking"></a>
### Matchmaking { #matchmaking }

GameAnvil offers two types of matchmaking. One is Room Matchmaking, which performs room-by-room matching, and the other is User Matchmaking, which performs user-by-user matching. For more information, see Matchmaking in the [Unity Basic Development Guide > UserAgent](../unity-basic/unity-basic-04-user-agent.md).

<a id="matchmaking-room-matchmaking"></a>
#### Room Matchmaking

Room matchmaking is a method that places users into a room that matches specified conditions. When a room matchmaking request is made, if a matching room exists, the user is placed in that room immediately; if no matching room is found, a new room is created and the user is placed in it.

You can request room matchmaking by calling MatchRoom().

```c#
public async void MatchRoom()
{
    try
    {
        var matchRoomPayload = new Payload(new Protocol.MatchRoomData());
        Result<ResultCodeMatchRoom, MatchResult> result = await user.MatchRoom(true, true, "RoomType", "MatchingGroup", "MatchingUserCategory", matchRoomPayload);
        if (result.ResultCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

MatchRoom() has the following 7 parameters:

| Type    | Name                      | Description                                                                                                                                                                                                                                                 |
|---------|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| bool    | isCreateRoomIfNotJoinRoom | Whether to create and enter a room when no matching room is found. <br/>true: Creates a room if none exists. <br/>false: Treated as a failure if no room exists.                                                                                            |
| bool    | isMoveRoomIfJoinedRoom    | Whether to move to a different room if the user has already joined a room. <br/>true: Moves to another room if already in one. <br/>false: Treated as a failure if matchmaking is requested while already in a room.                                        |
| string  | roomType                  | Room type. Finds rooms of the same type.                                                                                                                                                                                                                    |
| string  | matchingGroup             | Matching group. Finds rooms created with the same group.                                                                                                                                                                                                    |
| string  | matchingUserCategory      | User category to use in the matched room. Each room can divide its users into categories and apply a capacity limit per category. Finds rooms with a specified matchingUserCategory that doesn't have the maximum number of users. |
| Payload | payload                   | Additional information required by the server's user code to process the matchmaking request. (default = null)                                                                                                                                              |
| Payload | leaveRoomPayload          | Additional information required by the server's user code to process the room exit when moving to a different room. (default = null)                                                                                                                        |

The response returns `Result<ResultCodeMatchRoom, MatchResult>`. You can check the value of the ResultCode field to determine whether the request succeeded. If MatchRoom succeeds, the ResultCode field is set to `ResultCodeMatchRoom.MATCH_ROOM_SUCCESS`; otherwise, room matchmaking has failed. You can obtain the MatchResult from the Data field to get information about the matched room and, depending on the server implementation, additional information as well.

The details of ResultCodeMatchRoom are as follows:

| Name                                           | Value | Description                                                                                                                                                        |
|------------------------------------------------|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                    | -2    | Packet parsing error. May occur when the server and client versions differ.                                                                                        |
| TIMEOUT                                        | -1    | Timeout. No response received within the specified time.                                                                                                           |
| SYSTEM_ERROR                                   | 1     | Server system error. Failed due to an unknown server error.                                                                                                        |
| INVALID_PROTOCOL                               | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used.                                                           |
| NAMED_ROOM_SUCCESS                             | 0     | Success.                                                                                                                                                           |
| NAMED_ROOM_FAIL_CONTENT                        | 701   | Failed. Rejected by the user code.                                                                                                                                 |
| NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST            | 702   | Failed. The room disappeared while processing room entry.                                                                                                          |
| NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM            | 703   | Failed. Already in a room.                                                                                                                                         |
| NAMED_ROOM_FAIL_INVALID_ROOM_NAME              | 704   | Failed. An invalid room name was requested.                                                                                                                        |
| NAMED_ROOM_FAIL_CREATE_ROOM                    | 705   | Failed. Room creation failed.                                                                                                                                      |
| MATCH_ROOM_SUCCESS                             | 0     | Success.                                                                                                                                                           |
| MATCH_ROOM_FAIL_CONTENT                        | 901   | Failed. Rejected by the user code.                                                                                                                                 |
| MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST            | 902   | Failed. The room does not exist.                                                                                                                                   |
| MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM            | 903   | Failed. Already in a room.                                                                                                                                         |
| MATCH_ROOM_FAIL_LEAVE_ROOM                     | 904   | Failed. Failed to leave the current room when moving to a different room.                                                                                          |
| MATCH_ROOM_FAIL_IN_PROGRESS                    | 905   | Failed. Matchmaking is already in progress.                                                                                                                        |
| MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST    | 906   | Failed. The room disappeared while attempting to join it after a matching room was found.<br/>This may occur when all users in the room leave during room entry processing. |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID          | 907   | Failed. Room ID creation failed.                                                                                                                                   |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM             | 908   | Failed. Room creation failed.                                                                                                                                      |
| MATCH_ROOM_FAIL_INVALID_ROOM_ID                | 909   | Failed. An invalid room ID was used.                                                                                                                               |
| MATCH_ROOM_FAIL_INVALID_NODE_ID                | 910   | Failed. An invalid node ID was used.                                                                                                                               |
| MATCH_ROOM_FAIL_INVALID_USER_ID                | 911   | Failed. An invalid user ID was used.                                                                                                                               |
| MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND         | 912   | Failed. Matchmaking was performed but no room was found.                                                                                                           |
| MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY | 913   | Failed. An invalid matching user category was used.                                                                                                                |
| MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY   | 914   | Failed. The user category size in the matching room is 0.                                                                                                          |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL      | 915   | Failed. The match application form is NULL.                                                                                                                        |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL      | 916   | Failed. The matching information is NULL.                                                                                                                          |

The details of MatchResult are as follows:

| Type     | Name     | Description                                   |
|----------|----------|-----------------------------------------------|
| bool     | IsCancel | Whether the request was canceled.             |
| int      | RoomId   | ID of the room you entered.                   |
| bool     | Created  | Whether a new room has been created.          |
| String   | RoomName | Name of the room you entered.                 |
| Payload? | payload  | Additional information required by the client. |

<a id="matchmaking-user-matchmaking"></a>
#### User Matchmaking

User matchmaking creates a user pool, finds users in that pool that meet certain conditions, and places them in a newly created room. If the user pool does not have enough users that meet the conditions, matchmaking may take some time to complete. If matchmaking does not complete within the allotted time, it times out and the match may fail.

You can start user matchmaking by calling MatchUserStart(). Even if this request succeeds, it does not mean that user matchmaking is complete. It simply means that the start request has succeeded, and the matchmaking result is delivered through a separate callback.

```c#
public async void MatchUserStart()
{
    user.OnMatchUserDone +=(GameAnvilUser user, ResultCodeMatchUserDone resultCode, MatchResult matchResult) => {
        // Matching successful
    };
    user.onMatchUserTimeOut +=(GameAnvilUser user, ResultCodeMatchUserTimeOut resultCode) =>
    {
        // Matching failed
    };
    try
    {
        Payload matchUserPayload = new Payload(new Protocol.MatchUserData());
        Result<ResultCodeMatchUserStart, Payload> result = await user.MatchUserStart("RoomType", "MatchingGroup", matchUserPayload);
        if (result.ResultCode == ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS)
        {
            // Request successful
        } else
        {
            // Failed
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

MatchUserStart() has the following three parameters:

| Type | Name | Description |
|---------|---------------|------------------------------------------------------------------------|
| String  | roomType      | Type of room to create.                                                             |
| String  | matchingGroup | Matching group. Finds users that meet the conditions in the user pool of the same group. |
| Payload | payload       | Additional information required by the user code on the server that processes the user matchmaking request. (default = null) |

Returns Result<ResultCodeMatchUserStart, Payload> as the response. You can check the value of the ResultCode field to determine whether the request succeeded. If MatchUserStart succeeds, the value of the ResultCode field is ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS; otherwise, the request has failed. Depending on the server implementation, you may also receive additional information through the Payload in the Data field.

The details of ResultCodeMatchUserStart are as follows:

| Name                                        | Value | Description                                                                 |
|-------------------------------------------|------|---------------------------------------------|
| PARSE_ERROR                               | -2   | Packet parsing error. May occur when the server and client versions differ. |
| TIMEOUT                                   | -1   | Timeout. The response to the request did not arrive within the allotted time. |
| SYSTEM_ERROR                              | 1    | Server system error. Failed due to an unknown error on the server.          |
| INVALID_PROTOCOL                          | 2    | Protocol not registered on server. A protocol not registered in additional information was used. |
| MATCH_USER_START_SUCCESS                  | 0    | Success.                                                                    |
| MATCH_USER_START_FAIL_CONTENT             | 1101 | Failed. Rejected by user code.                                              |
| MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM | 1102 | Failed. Already in a room.                                                  |

<br>

When user matchmaking succeeds, you are notified through onMatchUserDone. You can get the result code through the parameter ResultCodeMatchUserDone resultCode, and you can get information about the matched room through the parameter MatchResult matchResult. Depending on the server implementation, you may also receive additional information through the payload of MatchResult.
If matchmaking does not succeed within the allotted time, you are notified through onMatchUserTimeout.

The details of ResultCodeMatchUserDone are as follows:

| Name                                       | Value | Description                                                                 |
|------------------------------------------|------|------------------------------------------------------------------|
| PARSE_ERROR                              | -2   | Packet parsing error. May occur when the server and client versions differ. |
| TIMEOUT                                  | -1   | Timeout. The response to the request did not arrive within the allotted time. |
| SYSTEM_ERROR                             | 1    | Server system error. Failed due to an unknown error on the server.          |
| INVALID_PROTOCOL                         | 2    | Protocol not registered on server. A protocol not registered in additional information was used. |
| MATCH_USER_DONE_SUCCESS                  | 0    | Success.                                                                    |
| MATCH_USER_DONE_FAIL_CONTENT             | 1501 | Failed. Rejected by user code.                                              |
| MATCH_USER_DONE_FAIL_ROOM_DOES_NOT_EXIST | 1502 | Failed. The room disappeared while finding a room that meets the conditions and joining it. |
| MATCH_USER_DONE_FAIL_TRANSFER            | 1503 | Failed. Failed during the transfer process to join a room while finding a room that meets the conditions. |
| MATCH_USER_DONE_FAIL_CREATE_ROOM         | 1504 | Failed. Room creation failed.                                               |

<br>

You can cancel an ongoing user matchmaking session by calling MatchUserCancel().

```c#
public async void MatchUserCancel()
{
    try
    {
        ResultCodeMatchUserCancel result = await user.MatchUserCancel("RoomType");
        if (result == ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

Returns ResultCodeMatchUserCancel as the response. If the cancellation succeeds, the value is ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS; otherwise, the cancellation has failed. The request may fail if user matchmaking is not in progress, if matchmaking has already succeeded, or if a timeout has occurred.

The details of ResultCodeMatchUserCancel are as follows:

| Name                                         | Value | Description                                                                 |
|--------------------------------------------|------|---------------------------------------------|
| PARSE_ERROR                                | -2   | Packet parsing error. May occur when the server and client versions differ. |
| TIMEOUT                                    | -1   | Timeout. The response to the request did not arrive within the allotted time. |
| SYSTEM_ERROR                               | 1    | Server system error. Failed due to an unknown error on the server.          |
| INVALID_PROTOCOL                           | 2    | Protocol not registered on server. A protocol not registered in additional information was used. |
| MATCH_USER_CANCEL_SUCCESS                  | 0    | Success.                                                                    |
| MATCH_USER_CANCEL_FAIL                     | 1201 | Failed. Rejected by user code.                                              |
| MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM | 1202 | Failed. Already in a room.                                                  |
| MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS     | 1203 | Failed. User matchmaking is not in progress.                                |

<a id="matchmaking-party-matchmaking"></a>
#### Party matchmaking

Party Matchmaking is a special form of user matchmaking, in which two or more users are grouped into a party and registered in the user pool. The system then finds other users who meet the matching conditions and places them together into a newly created room. Users grouped into a party always enter the same room. Other users matched alongside the party may be another party or individual users, depending on the server's match maker implementation.

To use Party Matchmaking, you must first call NamedRoom(). When calling NamedRoom(), pass true for the isParty parameter so that the NamedRoom acts as a party. Once all party users have gathered in the NamedRoom, start the party matchmaking.

```c#
public async void PartyRoom()
{
    try
    {
        bool isParty = true;
        Payload partyRoomPayload = new Payload(new Protocol.PartyRoomData());
        Result<ResultCodeNamedRoom, NamedRoomResult> result = await user.NamedRoom("RoomName", "RoomType", isParty, partyRoomPayload);
        if (result.ResultCode == ResultCodeNamedRoom.NAMED_ROOM_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

<br>

You can start party matchmaking by calling MatchPartyStart(). Even if this request succeeds, it does not mean that party matchmaking is complete. It only means that the start request was successful; the result of the matchmaking is delivered through a separate callback.

```c#
public async void MatchPartyStart()
{
    try
    {
        Payload partyRoomPayload = new Payload(new Protocol.PartyRoomData());
        Result<ResultCodeMatchPartyStart, Payload> result = await user.MatchPartyStart("RoomType", "MatchingGroup", partyRoomPayload);
        if (result.ResultCode == ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

MatchPartyStart() has the following three parameters.

| Type    | Name            | Description                                                                                                                        |
|---------|-----------------|-----------------------------------------------------------------------------------------------------------------------------------|
| String  | roomType        | Room type to create.                                                                                                              |
| String  | matchingGroup   | Matching group. Finds users that meet the conditions from the user pool of the same group. Enter string.Empty (empty string) if not used. |
| Payload | payload         | Additional information required by the user code on the server that processes the party matchmaking request. (default = null)     |

The response returns Result<ResultCodeMatchPartyStart, Payload>. You can check the value of the ResultCode field to determine whether the request succeeded. If MatchPartyStart succeeds, the ResultCode field will be ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS; otherwise, party matchmaking has failed. Depending on the server implementation, you may also obtain additional information through the Payload in the Data field.

The details of ResultCodeMatchPartyStart are as follows.

| Name                                       | Value | Description                                                                                      |
|--------------------------------------------|-------|--------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                | -2    | Packet parsing error. May occur when the server and client versions differ.                      |
| TIMEOUT                                    | -1    | Timeout. A response to the request did not arrive within the allotted time.                      |
| SYSTEM_ERROR                               | 1     | Server system error. Failed due to an unknown server error.                                      |
| INVALID_PROTOCOL                           | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| MATCH_PARTY_START_SUCCESS                  | 0     | Success.                                                                                         |
| MATCH_PARTY_START_FAIL_CONTENT             | 1301  | Failed. Rejected by user code.                                                                   |
| MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD   | 1302  | Failed. When requesting the party match, the room is not a room for party matching.              |

<br>
When party matching succeeds, you can receive a notification through onMatchUserDone, which is the same callback used in user matchmaking. You can also obtain information about the matched room through the MatchResult parameter, and additional information may be available depending on the server implementation.
If matching does not succeed within the allotted time, you can receive a notification through onMatchUserTimeout.

<br>

You can cancel an in-progress party matchmaking by calling MatchPartyCancel().

```c#
public async void MatchPartyCancel()
{
    try
    {
        ResultCodeMatchPartyCancel result = await user.MatchPartyCancel("RoomType");
        if (result == ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

MatchPartyCancel() has the following one parameter.

| Type   | Name     | Description                              |
|--------|----------|------------------------------------------|
| String | roomType | Room type of the matchmaking to cancel.  |

The response returns ResultCodeMatchPartyCancel. If MatchPartyCancel succeeds, ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS is returned; otherwise, the cancel request has failed.

The details of ResultCodeMatchPartyStart are as follows.

| Name                                          | Value | Description                                                                                      |
|-----------------------------------------------|-------|--------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                   | -2    | Packet parsing error. May occur when the server and client versions differ.                      |
| TIMEOUT                                       | -1    | Timeout. A response to the request did not arrive within the allotted time.                      |
| SYSTEM_ERROR                                  | 1     | Server system error. Failed due to an unknown server error.                                      |
| INVALID_PROTOCOL                              | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| MATCH_PARTY_CANCEL_SUCCESS                    | 0     | Success.                                                                                         |
| MATCH_PARTY_CANCEL_FAIL_CONTENT               | 1401  | Failed. Rejected by user code.                                                                   |
| MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD     | 1402  | Failed. When canceling the party match, the room is not a room for party matching.               |
| MATCH_PARTY_CANCEL_FAIL_ALREADY_JOINED_ROOM   | 1403  | Failed. When canceling the party match, if already in the room.                                  |
| MATCH_PARTY_CANCEL_FAIL_NOT_IN_PROGRESS       | 1404  | Failed. When trying to cancel while the party match is not in progress.                          |

<a id="channel"></a>
### Channel { #channel }

<!-- TODO: translate body -->

<a id="channel-move-notification"></a>
#### Channel move notification

<!-- TODO: translate body -->

<a id="channel-moving-channels"></a>
#### Moving channels

You can call `MoveChannel()` to move to another channel within the service.

```c#
public async void MoveChannel()
{
    try
    {
        Payload moveChannelPayload = new Payload(new Protocol.MoveChannelData());
        Result<ResultCodeMoveChannel, MoveChannelResult> result = await user.MoveChannel("ChannelId", moveChannelPayload);
        if (result.ResultCode == ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

`MoveChannel()` has the following two parameters:

| Type    | Name      | Description                                                                                                     |
|---------|-----------|----------------------------------------------------------------------------------------------------------------|
| string  | channelId | ID of the channel to move to                                                                                   |
| Payload | payload   | Additional information required by the user code on the server that handles the channel move request. (default = null) |

The response returns `Result<ResultCodeMoveChannel, MoveChannelResult>`. You can check whether the operation succeeded by examining the value of the `ResultCode` field. If `MoveChannel` succeeds, the value of the `ResultCode` field is `ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS`; otherwise, the channel move has failed. You can obtain the `MoveChannelResult` of the request through the `Data` field. This allows you to get information about the channel that was moved to, and additional information may also be available depending on the server implementation.
Therefore, additional information may also be obtained.

The details of `ResultCodeMoveChannel` are as follows:

| Name                                     | Value | Description                                                                                      |
|------------------------------------------|-------|--------------------------------------------------------------------------------------------------|
| PARSE_ERROR                              | -2    | Packet parsing error. May occur when the server and client versions differ.                      |
| TIMEOUT                                  | -1    | Timeout. A response to the request did not arrive within the set time.                           |
| SYSTEM_ERROR                             | 1     | Server system error. Failed due to an unknown server error.                                      |
| INVALID_PROTOCOL                         | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| MOVE_CHANNEL_SUCCESS                     | 0     | Success.                                                                                         |
| MOVE_CHANNEL_FAIL_CONTENT                | 1601  | Failed. Rejected by user code.                                                                   |
| MOVE_CHANNEL_FAIL_NODE_NOT_FOUND         | 1602  | Failed. Channel node not found.                                                                  |
| MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL | 1603  | Failed. Already in the requested channel.                                                        |
| MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM    | 1604  | Failed. Cannot move channels because you are already in a room.                                  |

The details of `MoveChannelResult` are as follows:

| Type     | Name      | Description                                                                                                                                                           |
|----------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| bool     | Force     | Indicates whether the channel move was forced by the server. <br/>true: The channel was moved by the server.<br/>false: The channel was moved at the request of the client. |
| string   | ChannelId | ID of the room that was entered.                                                                                                                                      |
| Payload? | payload   | Additional information required by the client.                                                                                                                        |

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

You can use `GetChannelCountInfo()` to request and receive count information (the number of users and rooms) for a specific channel.

```c#
public async void ChannelCountInfo()
{
    try
    {
        Result<ResultCodeChannelCountInfo, ChannelCountResult> result = await user.GetChannelCountInfo("ServiceName", "ChannelId");
        if (result.ResultCode == ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

`GetChannelCountInfo()` has the following two parameters:

| Type   | Name        | Description                                     |
|--------|-------------|-------------------------------------------------|
| String | ServiceName | Service to request channel information from     |
| String | channelId   | ID of the channel to request information from   |

The function returns `Result<ResultCodeChannelCountInfo, ChannelCountResult>` as the response. You can check the value of the `ResultCode` field to verify whether the request was successful. If `GetChannelCountInfo` succeeds, the value of the `ResultCode` field is `ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS`; otherwise, the request has failed. You can retrieve `ChannelCountResult`, the result of the request, through the `Data` field.

The details of `ResultCodeChannelCountInfo` are as follows:

| Name                                       | Value | Description                                                                                               |
|--------------------------------------------|-------|-----------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                | -2    | Packet parsing error. This may occur when the server and client versions differ.                          |
| TIMEOUT                                    | -1    | Timeout. A response to the request did not arrive within the specified time.                              |
| SYSTEM_ERROR                               | 1     | Server system error. The request failed due to an unknown server error.                                   |
| INVALID_PROTOCOL                           | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used.  |
| CHANNEL_COUNT_INFO_SUCCESS                 | 0     | Success                                                                                                   |
| CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1921  | Failed. Channel information not found.                                                                    |
| CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1922  | Failed. Invalid service ID.                                                                               |
| CHANNEL_COUNT_INFO_FAIL_INVALID_CHANNEL_ID | 1923  | Failed. Invalid channel ID.                                                                               |
| CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924  | Failed. Channel not found.                                                                                |

The details of `ChannelCountResult` are as follows:

| Type   | Name      | Description                                  |
|--------|-----------|----------------------------------------------|
| string | ChannelId | Returns the channel ID.                      |
| int    | UserCount | Returns the number of users in the channel.  |
| int    | RoomCount | Returns the number of rooms in the channel.  |

<br>

<a id="channel-information-getchannelinfo"></a>
#### GetChannelInfo

You can use GetChannelInfo() to request and retrieve information (user-defined) for a specific channel.

```c#
public async void ChannelInfo()
{
    try
    {
        Result<ResultCodeChannelInfo, Payload> result = await user.GetChannelInfo("ServiceName", "ChannenId");
        if (result.ResultCode == ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

GetChannelInfo() has the following two parameters:

| Type   | Name        | Description                                      |
|--------|-------------|--------------------------------------------------|
| String | ServiceName | Service to request channel information from      |
| String | channelId   | ID of the channel to request channel information from |

The response returns Result<ResultCodeChannelInfo, Payload>. You can check the value of the ResultCode field to determine whether the request succeeded. If GetChannelInfo() succeeds, the value of the ResultCode field is ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS; otherwise, the request has failed. On success, you can also obtain user-defined channel information from the Payload in the Data field.

The details of ResultCodeChannelInfo are as follows:

| Name                                 | Value | Description                                                                                      |
|--------------------------------------|-------|--------------------------------------------------------------------------------------------------|
| PARSE_ERROR                          | -2    | Packet parsing error. May occur when the server and client versions differ.                      |
| TIMEOUT                              | -1    | Timeout. The response to the request did not arrive within the specified time.                   |
| SYSTEM_ERROR                         | 1     | Server system error. Failed due to an unknown server error.                                      |
| INVALID_PROTOCOL                     | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CHANNEL_INFO_SUCCESS                 | 0     | Success                                                                                          |
| CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1921  | Failed. Channel information not found.                                                           |
| CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1922  | Failed. Invalid service ID.                                                                      |
| CHANNEL_INFO_FAIL_INVALID_CHANNEL_ID | 1923  | Failed. Invalid channel ID.                                                                      |
| CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924  | Failed. Channel not found.                                                                       |

<a id="channel-information-getallchannelcountinfo"></a>
#### GetAllChannelCountInfo

You can use GetAllChannelCountInfo() to request and retrieve count information (the number of users and rooms) for all channels in a specific service.

```c#
public async void AllChannelCountInfo()
{
    try
    {
        Result<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>> result = await user.GetAllChannelCountInfo("ServiceName");
        if (result.ResultCode == ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

GetAllChannelCountInfo() takes the following parameter:

| Type   | Name        | Description                                 |
|--------|-------------|---------------------------------------------|
| String | ServiceName | Service to request channel information from |

The response returns Result<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>>. You can check whether the request was successful by examining the value of the ResultCode field. If GetAllChannelCountInfo succeeds, the ResultCode field is set to ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS; otherwise, the request has failed. You can retrieve the request result, Dictionary<string, ChannelCountResult>, through the Data field. This dictionary uses the channel ID as the key and ChannelCountResult as the value.

The details of ResultCodeAllChannelCountInfo are as follows:

| Name                                           | Value | Description                                                                                        |
|------------------------------------------------|-------|----------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                    | -2    | Packet parsing error. This error may occur when the server and client versions differ.             |
| TIMEOUT                                        | -1    | Timeout. A response to the request did not arrive within the specified time.                       |
| SYSTEM_ERROR                                   | 1     | Server system error. Failed due to an unknown server error.                                        |
| INVALID_PROTOCOL                               | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| ALL_CHANNEL_COUNT_INFO_SUCCESS                 | 0     | Success                                                                                            |
| ALL_CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1931  | Failed. Channel information not found.                                                             |
| ALL_CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1932  | Failed. Invalid service ID.                                                                        |
| ALL_CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1933  | Failed. Channel not found.                                                                         |

<a id="channel-information-getallchannelinfo"></a>
#### GetAllChannelInfo

`GetAllChannelInfo()` requests and retrieves information (user-defined) for all channels of a specific service.

```c#
public async void AllChannelInfo()
{
    try
    {
        Result<ResultCodeAllChannelInfo, ChannelInfoResult> result = await user.GetAllChannelInfo("ServiceName");
        if (result.ResultCode == ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS)
        {
            // Success
        } else
        {
            // Failure
        }
    } catch (Exception e)
    {
        // Exception
    }
}
```

`GetAllChannelInfo()` takes the following 1 parameter:

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | Service to request channel information from |

The method returns `Result<ResultCodeAllChannelInfo, ChannelInfoResult>`. You can check whether the request succeeded by examining the value of the `ResultCode` field. If `GetAllChannelInfo` succeeds, the `ResultCode` field is set to `ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS`; otherwise, the request has failed. You can obtain the `ChannelInfoResult` from the `Data` field.

The details of `ResultCodeAllChannelInfo` are as follows:

| Name                                     | Value | Description                                                                                      |
|------------------------------------------|-------|--------------------------------------------------------------------------------------------------|
| PARSE_ERROR                              | -2    | Packet parsing error. Can occur when the server and client versions differ.                      |
| TIMEOUT                                  | -1    | Timeout. The response to the request did not arrive within the specified time.                   |
| SYSTEM_ERROR                             | 1     | Server system error. Failed due to an unknown error on the server.                               |
| INVALID_PROTOCOL                         | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| ALL_CHANNEL_INFO_SUCCESS                 | 0     | Success                                                                                          |
| ALL_CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1911  | Failed. Channel information not found.                                                           |
| ALL_CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1912  | Failed. Invalid service ID.                                                                      |
| ALL_CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1913  | Failed. Channel not found.                                                                       |

The `channelInfo` field of `ChannelInfoResult` is a `Dictionary<string, Payload>` that uses a channel ID as the key and a `Payload` containing user-defined channel information as the value. You can use this to retrieve user-defined information for each channel.

