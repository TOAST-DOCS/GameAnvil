<!-- machine_translated: true -->

<!-- pre-align:aligned sig=343635dd9ba0 -->

<a id="game-gameanvil-unity-advanced-development-guide-user"></a>
## Game > GameAnvil > Unity Advanced Development Guide > User { #game-gameanvil-unity-advanced-development-guide-user }

<a id="useragent"></a>
## User { #useragent }

GameAnvilUser is responsible for operations related to users on the GameAnvil server. It provides basic functionality such as Login(), Logout(), and room management.
The GameAnvil server can run multiple services simultaneously, and a single GameAnvilUser logs in to one service and operates independently of others. This means that you can create multiple GameAnvilUsers to log in to different services and use them simultaneously. You can also have multiple GameAnvilUsers logged into the same service at the same time by using different SubIds.

<a id="create"></a>
### Create { #create }

To use GameAnvilUser, you must first create a GameAnvilUser object.

```c#
public void CreateUser()
{
    try
    {
        user = new GameAnvilUser(connector, serviceName, subId);
        // Success
    }
    catch(Exception e)
    {
        // Failure
    }
}
```

You can create multiple GameAnvilUsers, separated by ServiceName and SubId.

```c#
public void CreateUsers()
{
    try
    {
        gameUser1 = new GameAnvilUser(connector, "GameService", 1);
        gameUser2 = new GameAnvilUser(connector, "GameService", 1);
        chatUser = new GameAnvilUser(connector, "ChatService", 1);
        // Success
    } catch (Exception e)
    {
        // Failure
    }
}
```

<a id="disable"></a>
### Disable { #disable }

When you are done using a GameAnvilUser, you must call Dispose() to release it.

```c#
public void DisposeUser()
{
    try
    {
        user.Dispose();
    } catch (Exception e)
    {
        // Failure
    }
}
```

<a id="loginlogout"></a>
### Login/Logout { #loginlogout }

Login can be defined as the process by which a client connects to the server and creates its own user object in GameNode. Logout is the opposite of login. In other words, it is the process of removing a user object from the GameNode.

<a id="loginlogout-login"></a>
#### Login

Call Login() to log in to a service. When logging in, you need to specify which UserType is logging in and which channel to log in to. If you need additional information, you can send it in the Payload.

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

Login() has the following 4 parameters:

| Type    | Name           | Description                                                                                      |
|---------|----------------|--------------------------------------------------------------------------------------------------|
| String  | userType       | Type of the user to create upon login.                                                           |
| String  | channelId      | ID of the channel to log in to.                                                                  |
| Payload | requestPayload | Additional information required by the user code on the server that processes the login request. (default = null) |

The response returns Result<ResultCodeLogin, LoginResult>. You can check the value of the ResultCode field to determine whether the call succeeded. If Login succeeds, the value of the ResultCode field is ResultCodeLogin.LOGIN_SUCCESS; otherwise, the login has failed. You can obtain the LoginResult from the request result through the Data field. This allows you to retrieve information about the logged-in user, and you may also obtain additional information depending on the server implementation.

The details of ResultCodeLogin are as follows:

| Name                               | Value | Description                                                                                    |
|------------------------------------|-------|------------------------------------------------------------------------------------------------|
| PARSE_ERROR                        | -2    | Packet parsing error. May occur when the server and client versions differ.                    |
| TIMEOUT                            | -1    | Timeout. A response to the request was not received within the specified time.                 |
| SYSTEM_ERROR                       | 1     | Server system error. Failed due to an unknown error on the server.                             |
| INVALID_PROTOCOL                   | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| JOIN_ROOM_SUCCESS                  | 0     | Success.                                                                                       |
| JOIN_ROOM_FAIL_CONTENT             | 701   | Failed. Rejected by user code.                                                                 |
| JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702   | Failed. The room requested for entry does not exist.                                           |
| JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM | 703   | Failed. Already in a room.                                                                     |
| JOIN_ROOM_FAIL_ALREADY_FULL        | 704   | Failed. The room requested for entry is full.                                                  |
| JOIN_ROOM_FAIL_ROOM_MATCH          | 705   | Failed. A problem occurred during room matchmaking.                                            |

The details of LoginResult are as follows:

| Type    | Name         | Description                                          |
|---------|--------------|------------------------------------------------------|
| int     | UserId       | Logged-in user ID                                    |
| string  | UserType     | Logged-in user type                                  |
| string  | ServiceName  | Name of the logged-in service                        |
| string  | ChannelId    | Logged-in channel ID                                 |
| Payload | Payload      | Additional information received from the server      |
| bool    | IsRelogined  | Whether re-logged in                                 |
| bool    | IsJoinedRoom | Whether joined to a room                             |
| int     | RoomId       | ID of the room the user belongs to                   |
| string  | RoomName     | Name of the room the user belongs to                 |
| Payload | RoomPayload  | Additional information about the room the user belongs to |
| bool    | IsMatching   | Whether matching has been requested                  |

<a id="logout"></a>
### Logout { #logout }

Call Logout() to log out of the service.

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
Logout() has the following 1 parameter:

| Type     | Name    | Description                                                                                               |
|----------|---------|-----------------------------------------------------------------------------------------------------------|
| Payload? | payload | Additional information required by the user code on the server that processes the logout request. (default = null) |

The response returns Result<ResultCodeLogout, LogoutResult>. You can check the value of the ResultCode field to determine whether the request was successful. If Logout succeeds, the ResultCode field value is ResultCodeLogout.LOGOUT_SUCCESS; otherwise, the logout has failed. You can obtain the LogoutResult from the request result through the Data field. Depending on the server implementation, you may also obtain additional information through the Payload field of LogoutResult.

The details of ResultCodeLogout are as follows:

| Name                | Value | Description                                                                                          |
|---------------------|-------|------------------------------------------------------------------------------------------------------|
| PARSE_ERROR         | -2    | Packet parsing error. This may occur when the server and client versions differ.                     |
| TIMEOUT             | -1    | Timeout. A response to the request was not received within the specified time.                       |
| SYSTEM_ERROR        | 1     | Server system error. Failed due to an unknown server error.                                          |
| INVALID_PROTOCOL    | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| LOGOUT_SUCCESS      | 0     | Success.                                                                                             |
| LOGOUT_FAIL_CONTENT | 401   | Failed. Rejected by user code.                                                                       |

<a id="logout-force-logout-notification"></a>
#### Force Logout Notification
Even if Logout() is not called, you can force the user to log out from the server. In this case, you can be notified via OnForceLogout.
```c#
public void AddOnLogout()
{
    user.OnForceLogout += (GameAnvilUser, Payload) =>
    {
        // Force Logout Notification
    };
}
```
Depending on the implementation of the server, additional information can also be obtained through the parameter Payload payload.

<a id="create-enter-and-leave-rooms"></a>
### Create, enter, and leave rooms { #create-enter-and-leave-rooms }

Two or more users can create a synchronized message flow through a room. In other words, requests from users within a room are all guaranteed to be processed in order. Of course, creating a room for a single user may also be meaningful depending on the content. How rooms are used is entirely up to the engine user.

<a id="create-enter-and-leave-rooms-create-room"></a>
#### Create Room

Call CreateRoom() to create a room and enter it.

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

CreateRoom() has the following four parameters:

| Type    | Name          | Description                                                                                      |
|---------|---------------|--------------------------------------------------------------------------------------------------|
| String  | roomName      | Name of the room to create. Enter string.Empty (empty string) if not used.                      |
| String  | roomType      | Type of the room to create. Enter a room type registered on the server.                          |
| String  | matchingGroup | Name of the matching group to use during matching. Enter string.Empty (empty string) if not used. |
| Payload | payload       | Additional information required by the user code on the server that processes the room creation request. (default = null) |

The response returns Result<ResultCodeCreateRoom, CreatedRoomResult>. You can check the value of the ResultCode field to determine whether the call succeeded. If CreateRoom succeeds, the value of the ResultCode field is ResultCodeCreateRoom.CREATE_ROOM_SUCCESS; otherwise, the room creation has failed. You can obtain the CreatedRoomResult from the request result through the Data field. This allows you to retrieve information about the created room, and you may also obtain additional information depending on the Server Implementation.

The details of ResultCodeCreateRoom are as follows:

| Name                                 | Value | Description                                                                              |
|--------------------------------------|-------|------------------------------------------------------------------------------------------|
| PARSE_ERROR                          | -2    | Packet parsing error. This may occur when the server and client versions differ.         |
| TIMEOUT                              | -1    | Timeout. A response to the request was not received within the specified time.           |
| SYSTEM_ERROR                         | 1     | Server system error. Failed due to an unknown server error.                              |
| INVALID_PROTOCOL                     | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CREATE_ROOM_SUCCESS                  | 0     | Success                                                                                  |
| CREATE_ROOM_FAIL_CONTENT             | 601   | Failed. Rejected by user code.                                                           |
| CREATE_ROOM_FAIL_ALREADY_JOINED_ROOM | 602   | Failed. Already in a room.                                                               |
| CREATE_ROOM_FAIL_CREATE_ROOM_ID      | 603   | Failed. Room ID creation failed.                                                         |
| CREATE_ROOM_FAIL_CREATE_ROOM         | 604   | Failed. Room creation failed.                                                            |

The details of CreatedRoomResult are as follows:

| Type    | Name     | Description                                  |
|---------|----------|----------------------------------------------|
| int     | RoomId   | ID of the created room                       |
| String? | RoomName | Name of the created room                     |
| Payload | payload  | Additional information required by the client |

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

| Type    | Name                 | Description                                                                                                                                                                                                                                                                                                                                            |
|---------|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String  | roomType             | Type of the room to enter.                                                                                                                                                                                                                                                                                                                             |
| int     | roomId               | ID of the room to enter.                                                                                                                                                                                                                                                                                                                               |
| String  | matchingUserCategory | The matchingUserCategory to use in the room to enter. If not used, enter string.Empty (empty string). <br/>Each room can divide users into categories and apply a maximum user count per category. JoinRoom may fail if the current number of users for the specified matchingUserCategory is at its maximum. |
| Payload | payload              | Additional information required by the user code on the server that will process the room entry request. (default = null)                                                                                                                                                                                                                              |

The response returns Result<ResultCodeJoinRoom, JoinRoomResult>. You can check whether the request succeeded by examining the value of the ResultCode field. If JoinRoom succeeds, the ResultCode field value is ResultCodeJoinRoom.JOIN_ROOM_SUCCESS; otherwise, room entry has failed. You can obtain the request result JoinRoomResult through the Data field. This allows you to retrieve information about the room that was entered, and you may also obtain additional information depending on the Server Implementation.

The details of ResultCodeJoinRoom are as follows:

| Name                               | Value | Description                                                                                    |
|------------------------------------|-------|------------------------------------------------------------------------------------------------|
| PARSE_ERROR                        | -2    | Packet parsing error. May occur if the server and client versions differ.                      |
| TIMEOUT                            | -1    | Timeout. The response to the request did not arrive within the set time.                       |
| SYSTEM_ERROR                       | 1     | Server system error. Failed due to an unknown error on the server.                             |
| INVALID_PROTOCOL                   | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| JOIN_ROOM_SUCCESS                  | 0     | Success.                                                                                       |
| JOIN_ROOM_FAIL_CONTENT             | 701   | Failed. Rejected by user code.                                                                 |
| JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702   | Failed. The room requested for entry does not exist.                                           |
| JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM | 703   | Failed. Already in a room.                                                                     |
| JOIN_ROOM_FAIL_ALREADY_FULL        | 704   | Failed. The room requested for entry is full.                                                  |
| JOIN_ROOM_FAIL_ROOM_MATCH          | 705   | Failed. A problem occurred during room matchmaking.                                            |

The details of JoinRoomResult are as follows:

| Type    | Name     | Description                                    |
|---------|----------|------------------------------------------------|
| int     | RoomId   | ID of the room entered.                        |
| String? | RoomName | Name of the room entered.                      |
| Payload | payload  | Additional information required by the client. |

<a id="create-enter-and-leave-rooms-leave-room"></a>
#### Leave Room

You can leave a room that you have joined by calling LeaveRoom().

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

LeaveRoom() has the following 1 parameter:

| Type    | Name    | Description                                                                                      |
|---------|---------|--------------------------------------------------------------------------------------------------|
| Payload | payload | Additional information required by the user code on the server that processes the room leave request. (default = null) |

Returns Result<ResultCodeLeaveRoom, Payload> as the response. Check the value of the ResultCode field to determine whether the request was successful. If LeaveRoom succeeds, the ResultCode field value is ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS; otherwise, the room leave request has failed. Depending on the server implementation, you can also obtain additional information through the Payload in the Data field.

The details of ResultCodeLeaveRoom are as follows:

| Name                    | Value | Description                                                                                       |
|-------------------------|-------|---------------------------------------------------------------------------------------------------|
| PARSE_ERROR             | -2    | Packet parsing error. This may occur if the server and client versions are different.             |
| TIMEOUT                 | -1    | Timeout. No response to the request was received within the specified time.                       |
| SYSTEM_ERROR            | 1     | Server system error. Failed due to an unknown server error.                                       |
| INVALID_PROTOCOL        | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| LEAVE_ROOM_SUCCESS      | 0     | Success.                                                                                          |
| LEAVE_ROOM_FAIL_CONTENT | 801   | Failed. Rejected by user code.                                                                    |

<a id="create-enter-and-leave-rooms-notification-for-forced-to-leave-the-room"></a>
#### Notification for Forced to Leave the Room
Even if you do not call LeaveRoom(), you can force the server to leave the room. In this case, you can be notified via OnForceLeaveRoom.
```c#
public void AddOnLeaveRoom()
{
    user.OnForceLeaveRoom += (GameAnvilUser gameAnvilUser, int roomId, Payload payload) =>
    {
        // Notification for Forced to Leave the Room
    };
}
```
You can find out from which room you were forcibly removed through the parameter int roomId, and depending on the implementation of the server, you can also get additional information through the parameter Payload payload.

<a id="create-enter-and-leave-rooms-enter-the-room-with-the-specified-name"></a>
#### Enter the room with the specified name

You can call `NamedRoom()` to enter a room with the specified name. If no room with the specified name exists, a new room is created and you enter that room.

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

`NamedRoom()` has the following 4 parameters:

| Type    | Name     | Description                                                                                                                                                              |
|---------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String  | roomType | Type of the room to enter or create.                                                                                                                                     |
| String  | roomName | Name of the room to enter or create.                                                                                                                                     |
| bool    | isParty  | Whether the room is for party matchmaking.<br/>Enter true if you create a room for users connected to the same party to wait together until the party matches are complete. |
| Payload | payload  | Additional information required from the user code of the server to process the entry or creation request. (default = null)                                              |

The response returns `Result<ResultCodeNamedRoom, NamedRoomResult>`. You can check whether the request was successful by checking the value of the `ResultCode` field. If `NamedRoom` succeeds, the value of the `ResultCode` field is `ResultCodeNamedRoom.NAMED_ROOM_SUCCESS`; otherwise, the room entry or creation has failed. You can obtain the `NamedRoomResult` of the request result through the `Data` field. This allows you to get information about the room that was entered or created, and you may also obtain additional information depending on the server implementation.

The details of `ResultCodeNamedRoom` are as follows:

| Name                                | Value | Description                                                                                                                                        |
|-------------------------------------|-------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                         | -2    | Packet parsing error. May occur when the server and client versions differ.                                                                        |
| TIMEOUT                             | -1    | Timeout. The response to the request did not arrive within the specified time.                                                                     |
| SYSTEM_ERROR                        | 1     | Server system error. Failed due to an unknown error on the server.                                                                                 |
| INVALID_PROTOCOL                    | 2     | Protocol not registered on the server. A protocol not registered in the additional information is used.                                            |
| NAMED_ROOM_SUCCESS                  | 0     | Success.                                                                                                                                           |
| NAMED_ROOM_FAIL_CONTENT             | 701   | Failed. Rejected by user code.                                                                                                                     |
| NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702   | Failed. The room does not exist.<br/>May occur when all users in the room leave the room while the room entry is being processed.                  |
| NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM | 703   | Failed. Already in a room.                                                                                                                         |
| NAMED_ROOM_FAIL_INVALID_ROOM_NAME   | 704   | Failed. An invalid room name was requested.                                                                                                        |
| NAMED_ROOM_FAIL_CREATE_ROOM         | 705   | Failed. Failed to create the room.                                                                                                                 |

The details of `NamedRoomResult` are as follows:

| Type    | Name     | Description                                     |
|---------|----------|-------------------------------------------------|
| bool    | Created  | Whether a new room was created.                 |
| int     | RoomId   | ID of the room entered.                         |
| String? | RoomName | Name of the room entered.                       |
| Payload | payload  | Additional information required by the client.  |

<a id="matchmaking"></a>
### Matchmaking { #matchmaking }

GameAnvil offers two types of matchmaking. One is Room Matchmaking, which performs room-by-room matching, and the other is User Matchmaking, which performs user-by-user matching.

<a id="matchmaking-room-matchmaking"></a>
#### Room Matchmaking

Room matchmaking is a method that places users into rooms that meet certain conditions. When a room matchmaking request is made, if a matching room exists, the user is placed directly into that room; if no matching room exists, a new room is created and the user is placed into it.

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

MatchRoom() has the following 7 parameters.

| Type | Name | Description |
|---------|---------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| bool | isCreateRoomIfNotJoinRoom | Whether to create and enter a room when no matching room is found. <br/>true: Creates a room if none exists. <br/>false: Returns a failure if no room exists. |
| bool | isMoveRoomIfJoinedRoom | Whether to move to a different room when the user is already in a room. <br/>true: Moves to a different room if the user is already in one. <br/>false: Returns a failure if a matchmaking request is made while the user is already in a room. |
| string | roomType | Room type. Finds rooms of the same type. |
| string | matchingGroup | Matching group. Finds rooms created with the same group. |
| string | matchingUserCategory | User category to use in the matched room.<br/>Each room can divide users into categories and apply a limit per category.<br/>Finds rooms with a specified matchingUserCategory that doesn't have the maximum number of users. |
| Payload | payload | Additional information required by the server's user code to process the matchmaking request. (default = null) |
| Payload | leaveRoomPayload | Additional information required by the server's user code to process the room exit when moving to a different room. (default = null) |

The response returns Result<ResultCodeMatchRoom, MatchResult>. You can check the success of the request by checking the value of the ResultCode field. If MatchRoom succeeds, the value of the ResultCode field is ResultCodeMatchRoom.MATCH_ROOM_SUCCESS; otherwise, the room matchmaking has failed. You can retrieve the MatchResult from the Data field. This provides information about the matched room and, depending on the server implementation, may include additional information.

The details of ResultCodeMatchRoom are as follows.

| Name                                | Value | Description                                                                                                                                        |
|-------------------------------------|-------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                         | -2    | Packet parsing error. May occur when the server and client versions differ.                                                                        |
| TIMEOUT                             | -1    | Timeout. The response to the request did not arrive within the specified time.                                                                     |
| SYSTEM_ERROR                        | 1     | Server system error. Failed due to an unknown error on the server.                                                                                 |
| INVALID_PROTOCOL                    | 2     | Protocol not registered on the server. A protocol not registered in the additional information is used.                                            |
| NAMED_ROOM_SUCCESS                  | 0     | Success.                                                                                                                                           |
| NAMED_ROOM_FAIL_CONTENT             | 701   | Failed. Rejected by user code.                                                                                                                     |
| NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST | 702   | Failed. The room disappeared while processing room entry.                                                                                          |
| NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM | 703   | Failed. Already in a room.                                                                                                                         |
| NAMED_ROOM_FAIL_INVALID_ROOM_NAME   | 704   | Failed. An invalid room name was requested.                                                                                                        |
| NAMED_ROOM_FAIL_CREATE_ROOM         | 705   | Failed. Failed to create the room.                                                                                                                 |
| MATCH_ROOM_SUCCESS                             | 0     | Success                                                                                                                                                                                                                                  |
| MATCH_ROOM_FAIL_CONTENT                        | 901   | Failed. Rejected by user code.                                                                                                                                                                                                           |
| MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST            | 902   | Failed. The room does not exist.                                                                                                                                                                                                         |
| MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM            | 903   | Failed. The user is already in a room.                                                                                                                                                                                                   |
| MATCH_ROOM_FAIL_LEAVE_ROOM                     | 904   | Failed. If moving to another room fails to leave the existing room.                                                                                                                                                                      |
| MATCH_ROOM_FAIL_IN_PROGRESS                    | 905   | Failed. If the matchmaking is already in progress.                                                                                                                                                                                       |
| MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST    | 906   | Failed. While looking for a room matching your condition, the room disappeared<br/>While entering a room, it may occur if all users in the room leave the room.                                                                           |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM_ID          | 907   | Failed. If the creation of room ID failed.                                                                                                                                                                                               |
| MATCH_ROOM_FAIL_CREATE_FAILED_ROOM             | 908   | Failed. If room creation failed.                                                                                                                                                                                                         |
| MATCH_ROOM_FAIL_INVALID_ROOM_ID                | 909   | Failed. An invalid room ID was used.                                                                                                                                                                                                     |
| MATCH_ROOM_FAIL_INVALID_NODE_ID                | 910   | Failed. An invalid node ID was used.                                                           |
| MATCH_ROOM_FAIL_INVALID_USER_ID                | 911   | Failed. An invalid user ID was used.                                                           |
| MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND         | 912   | Failed. Matchmaking was performed, but no room was found.                                      |
| MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY | 913   | Failed. An invalid matching user category was used.                                            |
| MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY   | 914   | Failed. The user category size in the matching room is 0.                                      |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL      | 915   | Failed. The match application form is NULL.                                                    |
| MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL      | 916   | Failed. The match information is NULL.                                                         |

The details of MatchResult are as follows.

| Type     | Name     | Description                                   |
|----------|----------|-----------------------------------------------|
| bool     | IsCancel | Whether the request was canceled.             |
| int      | RoomId   | ID of the room you entered.                   |
| bool     | Created  | Whether a new room has been created.          |
| String   | RoomName | Name of the room you entered.                 |
| Payload? | payload  | Additional information required by the client. |

<a id="matchmaking-user-matchmaking"></a>
#### User Matchmaking

User matchmaking creates a user pool, finds users that match the specified conditions within the pool, and places them into a newly created room. If the user pool does not have enough users matching the conditions, matchmaking may take some time to complete. If matchmaking does not complete within the time limit, it times out and the match may fail.

You can start user matchmaking by calling MatchUserStart(). A successful result from this request does not mean that user matchmaking has completed. It only means that the start request was successful; the result of matchmaking is delivered through a separate callback.

```c#
public async void MatchUserStart()
{
    user.OnMatchUserDone +=(GameAnvilUser user, ResultCodeMatchUserDone resultCode, MatchResult matchResult) => {
        // Matching succeeded
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
            // Request succeeded
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

MatchUserStart() has the following three parameters.

| Type    | Name          | Description                                                                                                        |
|---------|---------------|--------------------------------------------------------------------------------------------------------------------|
| String  | roomType      | Type of the room to create.                                                                                        |
| String  | matchingGroup | Matching group. Finds users matching the conditions from the user pool of the same group.                          |
| Payload | payload       | Additional information required by the user code on the server that processes the user matchmaking request. (default = null) |

The response returns Result<ResultCodeMatchUserStart, Payload>. You can check the value of the ResultCode field to determine whether the request was successful. If MatchUserStart succeeds, the ResultCode field is set to ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS; otherwise, the request has failed. Depending on the server implementation, you may also obtain additional information through the Payload in the Data field.

The details of ResultCodeMatchUserStart are as follows.

| Name                                      | Value | Description                                                                                              |
|-------------------------------------------|-------|----------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                               | -2    | Packet parsing error. This error may occur when the server and client versions differ.                   |
| TIMEOUT                                   | -1    | Timeout. A response to the request was not received within the specified time.                           |
| SYSTEM_ERROR                              | 1     | Server system error. Failed due to an unknown error on the server.                                       |
| INVALID_PROTOCOL                          | 2     | Protocol not registered on server. A protocol not registered in the additional information was used.     |
| MATCH_USER_START_SUCCESS                  | 0     | Success.                                                                                                 |
| MATCH_USER_START_FAIL_CONTENT             | 1101  | Failed. Rejected by user code.                                                                           |
| MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM | 1102  | Failed. The user is already in a room.                                                                   |

<br>

When user matchmaking succeeds, you will be notified through onMatchUserDone. You can retrieve the result code through the parameter ResultCodeMatchUserDone resultCode, and obtain information about the matched room through the parameter MatchResult matchResult. Depending on the server implementation, you may also obtain additional information through the payload of MatchResult.
If matchmaking does not succeed within the time limit, you will be notified through onMatchUserTimeout.

The details of ResultCodeMatchUserDone are as follows.

| Name                                     | Value | Description                                                                                                                         |
|------------------------------------------|-------|-------------------------------------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                              | -2    | Packet parsing error. This error may occur when the server and client versions differ.                                              |
| TIMEOUT                                  | -1    | Timeout. A response to the request was not received within the specified time.                                                      |
| SYSTEM_ERROR                             | 1     | Server system error. Failed due to an unknown error on the server.                                                                  |
| INVALID_PROTOCOL                         | 2     | Protocol not registered on server. A protocol not registered in the additional information was used.                                |
| MATCH_USER_DONE_SUCCESS                  | 0     | Success.                                                                                                                            |
| MATCH_USER_DONE_FAIL_CONTENT             | 1501  | Failed. Rejected by user code.                                                                                                      |
| MATCH_USER_DONE_FAIL_ROOM_DOES_NOT_EXIST | 1502  | Failed. The room disappeared while finding a matching room and joining it.                                                          |
| MATCH_USER_DONE_FAIL_TRANSFER            | 1503  | Failed. The transfer process failed while finding a matching room and joining it.                                                   |
| MATCH_USER_DONE_FAIL_CREATE_ROOM         | 1504  | Failed. Room creation failed.                                                                                                       |

<br>

You can cancel an ongoing user matchmaking by calling MatchUserCancel().

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

The response returns ResultCodeMatchUserCancel. If the cancellation succeeds, the value is ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS; otherwise, the cancellation has failed. The request may fail if user matchmaking is not in progress, if it has already succeeded, or if a timeout has occurred.

The details of ResultCodeMatchUserCancel are as follows.

| Name                                       | Value | Description                                                                                          |
|--------------------------------------------|-------|------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                | -2    | Packet parsing error. This error may occur when the server and client versions differ.               |
| TIMEOUT                                    | -1    | Timeout. A response to the request was not received within the specified time.                       |
| SYSTEM_ERROR                               | 1     | Server system error. Failed due to an unknown error on the server.                                   |
| INVALID_PROTOCOL                           | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| MATCH_USER_CANCEL_SUCCESS                  | 0     | Success.                                                                                             |
| MATCH_USER_CANCEL_FAIL                     | 1201  | Failed. Rejected by user code.                                                                       |
| MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM | 1202  | Failed. The user is already in a room.                                                               |
| MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS     | 1203  | Failed. User matchmaking is not in progress.                                                         |

<a id="matchmaking-party-matchmaking"></a>
#### Party matchmaking

Party matchmaking is a specialized form of user matchmaking where two or more users are grouped together as a party, added to a user pool, and matched with other eligible users to enter a newly created room together. Partyed users will always enter the same room. Outside of parties, the matching users can be other parties or individuals, depending on the server's matchmaker implementation.

To do party matchmaking, you must first call NamedRoom(). If you pass the isParty parameter to true when calling NamedRoom(), that NamedRoom will act as the party. After gathering all the party users in the NamedRoom, start party matchmaking.

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

You can start party matchmaking by calling MatchPartyStart(). A successful result from this request does not mean that party matchmaking has completed. It only means that the start request was successful; the result of matchmaking is delivered through a separate callback.

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

| Type    | Name          | Description                                                                                                                                          |
|---------|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| String  | roomType      | Type of the room to create.                                                                                                                          |
| String  | matchingGroup | Matching group. Finds users matching the conditions from the user pool of the same group. Enter string.Empty (empty string) if not used.             |
| Payload | payload       | Additional information required by the user code on the server that processes the party matchmaking request. (default = null)                        |

The response returns Result<ResultCodeMatchPartyStart, Payload>. You can check the value of the ResultCode field to determine whether the request was successful. If MatchPartyStart succeeds, the ResultCode field value is ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS; otherwise, the party matchmaking has failed. Depending on the server implementation, you may also obtain additional information through the Payload in the Data field.

The details of ResultCodeMatchPartyStart are as follows.

| Name                                     | Value | Description                                                                                          |
|------------------------------------------|-------|------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                              | -2    | Packet parsing error. May occur when the server and client versions differ.                          |
| TIMEOUT                                  | -1    | Timeout. A response to the request was not received within the specified time.                       |
| SYSTEM_ERROR                             | 1     | Server system error. Failed due to an unknown error on the server.                                   |
| INVALID_PROTOCOL                         | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| MATCH_PARTY_START_SUCCESS                | 0     | Success.                                                                                             |
| MATCH_PARTY_START_FAIL_CONTENT           | 1301  | Failed. Rejected by user code.                                                                       |
| MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD | 1302  | Failed. When requesting party matchmaking, the room is not a room for party matchmaking.             |

<br>
When party matchmaking succeeds, you will be notified through onMatchUserDone, which is the same callback used for user matchmaking. You can obtain information about the matched room through the MatchResult parameter, and depending on the server implementation, you may also obtain additional information.
If matchmaking does not succeed within the time limit, you will be notified through onMatchUserTimeout.

<br>

You can cancel an ongoing party matchmaking by calling MatchPartyCancel().

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

MatchPartyCancel() has the following 1 parameter.

| Type   | Name     | Description                                          |
|--------|----------|------------------------------------------------------|
| String | roomType | Room type of the matchmaking to cancel.              |

Returns ResultCodeMatchPartyCancel as the response. If MatchPartyCancel succeeds, ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS is returned; otherwise, the cancellation request has failed.

The details of ResultCodeMatchPartyStart are as follows.

| Name                                        | Value | Description                                                                                          |
|---------------------------------------------|-------|------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                                 | -2    | Packet parsing error. May occur when the server and client versions differ.                          |
| TIMEOUT                                     | -1    | Timeout. A response to the request was not received within the specified time.                       |
| SYSTEM_ERROR                                | 1     | Server system error. Failed due to an unknown error on the server.                                   |
| INVALID_PROTOCOL                            | 2     | Protocol not registered on server. A protocol not registered in the additional information was used. |
| MATCH_PARTY_CANCEL_SUCCESS                  | 0     | Success.                                                                                             |
| MATCH_PARTY_CANCEL_FAIL_CONTENT             | 1401  | Failed. Rejected by user code.                                                                       |
| MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD   | 1402  | Failed. When canceling the party match, the room is not a room for party matchmaking.                |
| MATCH_PARTY_CANCEL_FAIL_ALREADY_JOINED_ROOM | 1403  | Failed. When canceling the party match, the user is already in a room.                               |
| MATCH_PARTY_CANCEL_FAIL_NOT_IN_PROGRESS     | 1404  | Failed. Party matchmaking is not in progress but a cancellation was attempted.                       |

<a id="channel"></a>
### Channel { #channel }

<a id="channel-move-notification"></a>
#### Channel move notification

In some cases, channel movements can occur as a result of matching. If a channel has been moved, you can receive notification through OnMoveChannel. You can also get information about the channel where you moved the MoveChannelResult parameters, and you can also get additional information depending on the server implementation.

```c#
public void AddOnMoveChannel()
{
    user.OnMoveChannel += (GameAnvilUser user, MoveChannelResult result) =>
    {
        // Called when moving channels
    };
}
```

<a id="channel-moving-channels"></a>
#### Moving channels

You can move to a different channel within the service by calling MoveChannel().

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

MoveChannel() has the following 2 parameters:

| Type    | Name      | Description                                                                                              |
|---------|-----------|----------------------------------------------------------------------------------------------------------|
| string  | channelId | ID of the channel to move to.                                                                            |
| Payload | payload   | Additional information required by the user code on the server that processes the channel move request. (default = null) |

The response returns Result<ResultCodeMoveChannel, MoveChannelResult>. You can check the value of the ResultCode field to determine whether the request was successful. If MoveChannel succeeds, the ResultCode field value is ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS; otherwise, the channel move has failed. You can obtain the MoveChannelResult from the request result through the Data field. This allows you to retrieve information about the channel that was moved to, and you may also obtain additional information depending on the server implementation.

The details of ResultCodeMoveChannel are as follows:

| Name                                     | Value | Description                                                                                                     |
|------------------------------------------|-------|-----------------------------------------------------------------------------------------------------------------|
| PARSE_ERROR                              | -2    | Packet parsing error. May occur when the server and client versions differ.                                     |
| TIMEOUT                                  | -1    | Timeout. A response to the request was not received within the specified time.                                  |
| SYSTEM_ERROR                             | 1     | Server system error. Failed due to an unknown error on the server.                                              |
| INVALID_PROTOCOL                         | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used.        |
| MOVE_CHANNEL_SUCCESS                     | 0     | Success.                                                                                                        |
| MOVE_CHANNEL_FAIL_CONTENT                | 1601  | Failed. Rejected by user code.                                                                                  |
| MOVE_CHANNEL_FAIL_NODE_NOT_FOUND         | 1602  | Failed. The channel node could not be found.                                                                    |
| MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL | 1603  | Failed. Already in the requested channel.                                                                       |
| MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM    | 1604  | Failed. Unable to move channels because the user is already in a room.                                         |

The details of MoveChannelResult are as follows:

| Type     | Name      | Description                                                                                                                                            |
|----------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| bool     | Force     | Whether the channel move was forced by the server. <br/>true: The channel was moved by the server.<br/>false: The channel was moved at the client's request. |
| string   | ChannelId | ID of the channel entered.                                                                                                                             |
| Payload? | payload   | Additional information required by the client.                                                                                                         |

<a id="channel-information"></a>
### Channel information { #channel-information }

GameAnvil allows you to freely change channel configurations in the settings. These channel configurations can be pre-agreed between the server and the client and used in a fixed form, or they can be changed flexibly to suit the situation. GameAnvilUser provides several methods to get information about these changed channels.

| Name                     | Description                                                                                 |
|--------------------------|---------------------------------------------------------------------------------------------|
| GetChannelCountInfo()    | Request count information (number of users and rooms) for a specific channel                |
| GetChannelInfo()         | Request information (user-defined) for a specific channel                                   |
| GetAllChannelCountInfo() | Request count information (number of users and rooms) for all channels of a specific service |
| GetAllChannelInfo()      | Request information (user-defined) about all channels for a specific service                |

<a id="channel-information-getchannelcountinfo"></a>
#### GetChannelCountInfo

GetChannelCountInfo() can request and receive count information (number of users and rooms) for a specific channel.

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

GetChannelCountInfo() has the following 2 parameters:

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | The service to request channel information from |
| String | channelId   | ID of the channel to request channel information for |

The response returns Result<ResultCodeChannelCountInfo, ChannelCountResult>. You can check the value of the ResultCode field to determine whether the request was successful. If GetChannelCountInfo succeeds, the value of the ResultCode field is ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS; otherwise, the request has failed. You can obtain the ChannelCountResult from the request result through the Data field.

The details of ResultCodeChannelCountInfo are as follows:

| Name                                       | Value | Description                                                                              |
|--------------------------------------------|-------|------------------------------------------------------------------------------------------|
| PARSE_ERROR                                | -2    | Packet parsing error. This may occur when the server and client versions differ.         |
| TIMEOUT                                    | -1    | Timeout. A response to the request was not received within the specified time.           |
| SYSTEM_ERROR                               | 1     | Server system error. Failed due to an unknown server error.                              |
| INVALID_PROTOCOL                           | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CHANNEL_COUNT_INFO_SUCCESS                 | 0     | Success                                                                                  |
| CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1921  | Failed. Channel information not found.                                                   |
| CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1922  | Failed. Invalid service ID.                                                              |
| CHANNEL_COUNT_INFO_FAIL_INVALID_CHANNEL_ID | 1923  | Failed. Invalid channel ID.                                                              |
| CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924  | Failed. Channel not found.                                                               |

The details of ChannelCountResult are as follows:

| Type   | Name      | Description                        |
|--------|-----------|------------------------------------|
| string | ChannelId | Returns the channel ID.            |
| int    | UserCount | Returns the number of users in the channel. |
| int    | RoomCount | Returns the number of rooms in the channel. |

<br>

<a id="channel-information-getchannelinfo"></a>
#### GetChannelInfo

GetChannelInfo() can request and receive information (user-defined) about a specific channel.

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

GetChannelInfo() has the following 2 parameters:

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | The service to request channel information from |
| String | channelId   | ID of the channel to request channel information for |

The response returns Result<ResultCodeChannelInfo, Payload>. You can check the value of the ResultCode field to determine whether the request was successful. If GetChannelInfo() succeeds, the value of the ResultCode field is ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS; otherwise, the request has failed. On success, you can also obtain user-defined channel information through the Payload in the Data field.

The details of ResultCodeChannelInfo are as follows:

| Name                                 | Value | Description                                                                              |
|--------------------------------------|-------|------------------------------------------------------------------------------------------|
| PARSE_ERROR                          | -2    | Packet parsing error. This may occur when the server and client versions differ.         |
| TIMEOUT                              | -1    | Timeout. A response to the request was not received within the specified time.           |
| SYSTEM_ERROR                         | 1     | Server system error. Failed due to an unknown server error.                              |
| INVALID_PROTOCOL                     | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| CHANNEL_INFO_SUCCESS                 | 0     | Success                                                                                  |
| CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1921  | Failed. Channel information not found.                                                   |
| CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1922  | Failed. Invalid service ID.                                                              |
| CHANNEL_INFO_FAIL_INVALID_CHANNEL_ID | 1923  | Failed. Invalid channel ID.                                                              |
| CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1924  | Failed. Channel not found.                                                               |

<a id="channel-information-getallchannelcountinfo"></a>
#### GetAllChannelCountInfo

GetAllChannelCountInfo() can request and receive count information (number of users and rooms) for all channels in a specific service.

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

GetAllChannelCountInfo() has the following 1 parameter:

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | The service to request channel information from |

The response returns Result<ResultCodeAllChannelCountInfo, Dictionary<string, ChannelCountResult>>. You can check the value of the ResultCode field to determine whether the request was successful. If GetAllChannelCountInfo succeeds, the value of the ResultCode field is ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS; otherwise, the request has failed. You can obtain the Dictionary<string, ChannelCountResult> from the request result through the Data field. This Dictionary uses the channel ID as the key and ChannelCountResult as the value.

The details of ResultCodeAllChannelCountInfo are as follows:

| Name                                           | Value | Description                                                                              |
|------------------------------------------------|-------|------------------------------------------------------------------------------------------|
| PARSE_ERROR                                    | -2    | Packet parsing error. This may occur when the server and client versions differ.         |
| TIMEOUT                                        | -1    | Timeout. A response to the request was not received within the specified time.           |
| SYSTEM_ERROR                                   | 1     | Server system error. Failed due to an unknown server error.                              |
| INVALID_PROTOCOL                               | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| ALL_CHANNEL_COUNT_INFO_SUCCESS                 | 0     | Success                                                                                  |
| ALL_CHANNEL_COUNT_INFO_FAIL_NO_CHANNEL_INFO    | 1931  | Failed. Channel information not found.                                                   |
| ALL_CHANNEL_COUNT_INFO_FAIL_INVALID_SERVICE_ID | 1932  | Failed. Invalid service ID.                                                              |
| ALL_CHANNEL_COUNT_INFO_FAIL_CHANNEL_NOT_FOUND  | 1933  | Failed. Channel not found.                                                               |

<a id="channel-information-getallchannelinfo"></a>
#### GetAllChannelInfo

GetAllChannelInfo() can request and receive information (user-defined) about all channels in a specific service.

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

GetAllChannelInfo() has the following 1 parameter:

| Type   | Name        | Description                              |
|--------|-------------|------------------------------------------|
| String | ServiceName | The service to request channel information from |

The response returns Result<ResultCodeAllChannelInfo, ChannelInfoResult>. You can check the value of the ResultCode field to determine whether the request was successful. If GetAllChannelInfo succeeds, the value of the ResultCode field is ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS; otherwise, the request has failed. You can obtain the ChannelInfoResult from the request result through the Data field.

The details of ResultCodeAllChannelInfo are as follows:

| Name                                     | Value | Description                                                                              |
|------------------------------------------|-------|------------------------------------------------------------------------------------------|
| PARSE_ERROR                              | -2    | Packet parsing error. This may occur when the server and client versions differ.         |
| TIMEOUT                                  | -1    | Timeout. A response to the request was not received within the specified time.           |
| SYSTEM_ERROR                             | 1     | Server system error. Failed due to an unknown server error.                              |
| INVALID_PROTOCOL                         | 2     | Protocol not registered on the server. A protocol not registered in the additional information was used. |
| ALL_CHANNEL_INFO_SUCCESS                 | 0     | Success                                                                                  |
| ALL_CHANNEL_INFO_FAIL_NO_CHANNEL_INFO    | 1911  | Failed. Channel information not found.                                                   |
| ALL_CHANNEL_INFO_FAIL_INVALID_SERVICE_ID | 1912  | Failed. Invalid service ID.                                                              |
| ALL_CHANNEL_INFO_FAIL_CHANNEL_NOT_FOUND  | 1913  | Failed. Channel not found.                                                               |

The `channelInfo` field of ChannelInfoResult is a Dictionary<string, Payload> that uses the channel ID as the key and a Payload containing user-defined channel information as the value. You can use this to retrieve user-defined information for each channel.