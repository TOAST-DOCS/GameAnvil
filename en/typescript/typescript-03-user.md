## Game > GameAnvil > Guide to Typescript Development > User

## GameAnvilUser

GameAnvilUser is the object on the client side that responds to the user that exists on the GameNode of the server. You can synchronize the server and the client by giving commands to or receiving user information from the server's user objects. You can use predefined room features, matchmaking, and more for your engine, but you can also implement a direct protocol to add new features.

### Create

To use GameAnvilUser, you must first create a new GameAnvilUser object and log in to the server through the created user.

```typescript
const connector: GameAnvilConnector;
const serviceName: string;

const user = new GameAnvilUser(connector, serviceName, 1);
```

### Create Multiple GameUserAgnets

GameAnvilConnector objects are generally used only within the process, while GameAnvilUser can create and operate multiple objects at the same time. Multiple users can log in to different services each, and if you want to use multiple GameAnvilUsers for one service, you can use subId to separate and create them.

```typescript
const connector: GameAnvilConnector;
const serviceName: string;
const otherServiceName: string;

const user1 = new GameAnvilUser(connector, serviceName, 1);
const user2 = new GameAnvilUser(connector, serviceName, 2);
const user3 = new GameAnvilUser(connector, otherServiceName, 1);
```

### Login

Request creation of server user objects that respond to clients within GameNode. You must log in to GameNode to use various other features of the user.

You receive the user type and channel ID to log in as required factors and can send additional information as the third factor. When the login is complete, you can check whether the login was successful through Promise, additional data received from the server, and more.

```typescript
const userType: stirng;
const channelId: string;
const payload: Payload;

const loginResult = await user.login(userType, channelId, payload);
console.log(`Login Result : ${ResultCodeLogin[loginResult.errorCode]}`);
```

You can check whether the login is successful through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (loginResult.resultCode === ResultCodeLogin.LOGIN_SUCCESS) {
    console.log("Login Success");
} else {
    console.log("Login Fail");
}
```

If the login fails, you can find out about the source through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|---------------------------------------------|-------|---------------------------------------------------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | `TIMEOUT` |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 301 | Failed: Denied from content | | `LOGIN_FAIL_NOT_EXIST_NODE` | 302 | Failed: Node does not exist | | `LOGIN_FAIL_TIMEOUT_GAME_SERVER` | 303 | Failed: Game server is unresponsive |
| `LOGIN_FAIL_INVALID_SERVICEID` | 310 | Failed: Invalid service ID |
| `LOGIN_FAIL_INVALID_USERTYPE` | 311 | Failed: Invalid user type |
| `LOGIN_FAIL_INVALID_USERID` | 312 | Failed: Invalid user ID |
| `LOGIN_FAIL_INVALID_SUB_ID` | 313 | Failed: Invalid SubId |
| `LOGIN_FAIL_INVALID_CHANNEL_ID` | 314 | Failed: Invalid channel ID |
| `LOGIN_FAIL_LOGINED_OTHER_SERVICE` | 320 | Failed: You are logged in to another service |
| `LOGIN_FAIL_LOGINED_OTHER_CHANNEL` | 321 | Failed: You are logged in to another channel |
| `LOGIN_FAIL_LOGINED_OTHER_USER_TYPE` | 322 | Failed: Another user type is logged in with the same account ID |
| `LOGIN_FAIL_LOGINED_OTHER_DEVICE` | 323 | Failed: Another device ID logged in with the same account ID |

You can check the isLoggedIn attribute to commonly check whether the user is logged in.

```typescript
if (user.isLoggedIn) {
    console.log("Already logged in.");
}
```

### Logout

You can explicitly request the server to delete the user. When the logout is complete, Promise lets you see if the logout was successful, additional data sent from the server, and more.

```typescript
const logoutResult = await user.logout();
```

If the user is not logged in but attempts to log out, it will respond successfully without any requests to the server.

You can check whether logout is successful through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (logoutResult.resultCode === ResultCodeLogout.LOGOUT_SUCCESS) {
    console.log("Logout Success");
} else {
    console.log("Logout Fail");
}
```

If logout fails, you can find out about the cause through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 401 | Failed: Denied from the content |

Depending on the server settings or status, users can be forcibly logged out of the server. For this case, you can register the process function as follows:

```typescript
user.onForceLogout = (user, payload) => {
    console.log(`User ${user.userId}} forced to logout.`);
}
```

### Register Message Reception Callback

When you receive a protobuf message from the server, you can set the processing function to run. For one protobuf, only one process function can be registered, and if you re-register the process function while the number is already registered, the existing process function will be deleted.

Please note that the processing function runs asynchronously, so it may not be called exactly at the intended time.

In the next chapter, we will cover how to transpile the protobuf message that enters the first argument.

```typescript
user.setMessageCallback(UserInfo.descriptor, (connector, resultCode, userInfo) => {
    if (resultCode === ResultCode.SUCCESS) {
        console.log(userInfo.name);
        console.log(userInfo.age);
        console.log(userInfo.job);

        return;
    }
    console.log("Message receive fail.", resultCode);
});
```

### Enter after Creating a New Room

You can create a room on the server and enter it immediately. If room names are not required, you can pass an empty string. The room type must use a value that is previously agreed to with the server.

The matching group is the information to use when matchmaking. If not used, leave it blank.

When the room is created and the entry is complete, Promise lets you check whether the room is successful, your newly created room ID, user custom add information, and more.

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const resultCreateRoom = await user.createRoom("", roomType, matchingGroup, payload);
```

You can check whether the action is successful through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (resultCreateRoom.resultCode === ResultCodeCreateRoom.CREATE_ROOM_SUCCESS) {
     console.log("Create room success");
} else {
    console.log("Create room fail");
}
```

If the creation or entry of a room fails, you can find out about the source through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 601 | Failed: Denied from the content |
| `CREATE_ROOM_FAIL_ALREADY_JOINED_ROOM` | 602 | Failed: Already in the room |
| `CREATE_ROOM_FAIL_CREATE_ROOM_ID` | 603 | Failed: Failed to issue room ID |
| `CREATE_ROOM_FAIL_CREATE_ROOM` | 604 | Failed: Failed to create a room |

If the creation and positioning of the room succeeded, you can check the information that includes roomId through the response.

```typescript
if (resultCreateRoom.resultCode === ResultCodeCreateRoom.CREATE_ROOM_SUCCESS) {
    console.log(`Created room id: ${resultCreateRoom.roomId}`);
    console.log(`Created room name: ${resultCreateRoom.roomName}`);
}
```

You can check the isJoinedRoom attribute to see if you're in the room.

```typescript
if (user.isJoinedRoom) {
    console.log("Alread joined room.");
}
```

If it’s in the room, you can check the ID of the room through the roomId attribute.

```typescript
console.log(`Current joined room id: ${user.roomId}`);
```

### Enter an Existing Room

If you are aware of the room ID generated on the server, you can request entry into the room.

The matching user category is information to use when matchmakng. If not used, keep it blank.

Promise lets you know if your room is successful or more when your entry is complete.

```typescript
const roomType: string;
const roomId: number;
const matchinUserCategory: string;
const payload: Payload;

const resultJoinRoom = user.joinRoom(roomType, roomId, matchingUserCategory, payload);
```

Whether your entry is successful can be checked via the resultCode of Result, the Promise result value as shown below.

```typescript
if (resultJoinRoom.resultCode === ResultCodeJoinRoom.JOIN_ROOM_SUCCESS) {
    console.log("Join room success");

} else {
    console.log("Join room fail");
}
```

If entering the room fails, you can find out about the source through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 701 | Failed: Denied from content |
| `JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST` | 702 | Failed: Room does not exist |
| `JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM` | 703 | Failed: Already in the room |
| `JOIN_ROOM_FAIL_ALREADY_FULL` | 704 | Failed: If the room is already full |
| `JOIN_ROOM_FAIL_ROOM_MATCH` | 705 | Failed: When a problem occurs with room matchmaking |

If entering the room is successful, you can check the details as follows:
```typescript
if (resultJoinRoom.resultCode === ResultCodeJoinRoom.JOIN_ROOM_SUCCESS) {
    console.log(`Joined room id: ${resultJoinRoom.roomId}`);
    console.log(`Joined room name: ${resultJoinRoom.roomName}`);
}
```

### Exit the Room Entering

You can request the server to exit the room entering.

At the time of exit, you can check the success or additional information through Promise.

```typescript
const payload: Payload;

const leaveRoomResult = await user.leaveRoom(payload);
```

You can check whether it’s deleted through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (leaveRoomResult.resultCode === ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS) {
    console.log("Leave room success");
} else {
    console.log("Leave room fail");
}
```

If exiting the room has failed, you can find out about the source through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 801 | Failed: Denied from the content |

If you are force-checked from the room by the server, you can register the process function to run.

```typescript
user.onForceLeaveRoom = (user, roomId, payload) => {
    console.log(`Server forced to leave room. roomId: ${roomId}`);
}
```

### Register in User Matchmaking Pool

User matchmaking is a method to create a user pool and hold the appropriate users in it to enter the newly created room. If the number of users in the user pool that meet the conditions is low, it may take time for the matching to complete. If matchmaking is not completed within the timeout, the matching is canceled.

You can request user matchmaking to the server as shown below:

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const matchUserStartResult = await matchUserStart(roomType, matchingGroup, payload);
```

You can check whether the pool is registered normally through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (matchUserStartResult.resultCode === ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS) {
    console.log("Match user start success.");
} else {
    console.log("Match user start fail.");
}
```
If the registration in the pool failed, you can find out about the reason through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 1101 | Failed: Denied from the content |
| `MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM` | 1102 | Failed: Already in the room |

If the pool is registered normally, the user match is performed on the server. If the match is successful, a pre-registered callback is called to the user. Before requesting the user matching pool, you can register a callback as shown below.

```typescript
user.onMatchUserDone = (user, resultCode, matchResult) => {
    console.log(`Match user done.`, resultCode);

    console.log(`Is request canceled?`, matchResult.isCancel);
    console.log(`Matched room id: `, matchResult.roomId);
    console.log(`Matched room name: `, matchResult.roomName);
    console.log(`Is room created for match?`, matchResult.created);
};
```

You can check whether your registration request is being sent and processed before and after registering to the pool.

```typescript
console.log(`Is in progress of match making?`, user.isUserMatchInPrgress);
```

### Remove User Matchmaking Pool

You can cancel the request if you have requested a user matchmaking, but the matchmaking is still in progress.

```typescript
const roomType: string;

const resultCode = await user.matchUserCancel(roomType);
console.log(`Match user cancel result: `, ResultCodeMatchUserCancel[resultCode]);
```

You can check whether the removal request from the pool succeeded via the resultCode of Result, the Promise result value, as shown below:

```typescript
if (resultCode === ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS) {
    console.log("Match user cancel success.");
} else {
    console.log("Match user cancel fail.");
}
```

If the request failed, you can find out about the source through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 1201 | Failed: Denied from the content |
| `MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM` | 1202 | Failed: Already matched |
| `MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS` | 1203 | Failed: When matching is not in progress |

### Room Matchmaking

Room matchmaking is a method to get the user into a room suitable for conditions. If you have a room that meets the conditions, you will immediately enter the room. If no room meets the conditions, you will create and register a new room.

You can pass arguments for the type of room to enter, additional information for matching, such as matching groups and matching user categories, as well as other options and additional information to pass.

When the room matchmaking operation is complete, Promise lets you see if the action was successful, your room ID, additional user custom information, and more.

```typescript
const isCreateRoomIfNotJoinRoom: boolean;
const isMoveRoomIfJoinedRoom: boolean;
const roomType: string;
const matchingGroup: string;
const matchingUserCategory: string;
const payload: Payload;
const leaveRoomPayload: Payload;

const matchRoomResult = await user.matchRoom(isCreateRoomIfNotJoinRoom, isMoveRoomIsJoinedRoom, roomType, matchingGroup, matchingUserCategory, payload, leaveRoomPayload);
console.log(`Match room result: ${ResultCodeMatchRoom[matchRoomResult.errorCode]}`);
```

You can check whether the action is successful through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (matchRoomResult.resultCode === ResultCodeMatchRoom.MATCH_ROOM_SUCCESS) {
    console.log("Match room success");
} else {
    consoel.log("Match room fail");
}
```

If matchmaking fails, you can find out about the cause through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 901 | Failed: Denied from the content |
| `MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST` | 902 | Failed: Room does not exist |
| `MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM` | 903 | Failed: Already in the room |
| `MATCH_ROOM_FAIL_LEAVE_ROOM` | 904 | Failed: Failed to leave an existing room |
| `MATCH_ROOM_FAIL_IN_PROGRESS` | 905 | Failed: If matchmaking is already in progress |
| `MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST` | 906 | Failed: Looking for a room matched to your condition |
| `MATCH_ROOM_FAIL_CREATE_ROOM_ID` | 907 | Failed: Failed to issue room ID |
| `MATCH_ROOM_FAIL_CREATE_ROOM` | 908 | Failed: Failed to create rooms |
| `MATCH_ROOM_FAIL_INVALID_ROOM_ID` | 909 | Failed: Invalid room ID |
| `MATCH_ROOM_FAIL_INVALID_NODE_ID` | 910 | Failed: Invalid node ID |
| `MATCH_ROOM_FAIL_INVALID_USER_ID` | 911 | Failed: Invalid user ID |
| `MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND` | 912 | Failed: Matched, but no room found |
| `MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY` | 913 | Failed: Invalid matching user category |
| `MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY` | 914 | Failed: If the matching user category size is 0 |
| `MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL` | 915 | Failed: No matching request form |
| `MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL` | 916 | Failed: No matching information |

If the room matchmaking is successful, you can check the information that includes roomId through the response.

```typescript
if (matchRoomResult.resultCode === ResultCodeMatchRoom.MATCH_ROOM_SUCCESS) {
    console.log(`Match room is cancel: ${matchRoomResult.isCancel}`);
    console.log(`Matched room id: ${matchRoomResult.roomId}`);
    console.log(`Matched room name: ${matchRoomResult.roomName}`);
    consoel.log(`Is matched room created? : ${matchRoomResult.created}`);
}
```

### Room with the specified name

You can enter a room with the name you specified, or enter a room for party matching. If there is no room with the name you specified, it will be created and entered.

```typescript
const roomType: stirng;
const roomName: string;
const isParty: boolean;
const payload: Payload;

const namedRoomResult = await user.namedDroom(roomType, roomName, isParty, payload);

console.log(`Named room result: ${ResultCodeNamedRoom[namedRoomResult.errorCode]}`);
```

You can check whether the action is successful through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (namedRoomResult.resultCode === ResultCodeNamedRoom.NAMED_ROOM_SUCCESS) {
    console.log("Named room success");
} else {
    console.log("Named room success");
}
```

If the action fails, you can find out about the cause through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 1001 | Failed: Denied from the content |
| `NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST` | 1002 | Failed: Failed to create a room and the room could not be found |
| `NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM` | 1003 | Failed: Already in the room |
| `NAMED_ROOM_FAIL_INVALID_ROOM_NAME` | 1004 | Failed: Enter an invalid room name |
| `NAMED_ROOM_FAIL_CREATE_ROOM` | 1005 | Failed: Failed to create a room |

If the request succeeded, you can check the information through a response.

```typescript
if (namedRoomResult.resultCode === ResultCodeNamedRoom.NAMED_ROOM_SUCCESS) {
    console.log(`Named room id: ${namedRoomResult.roomId}`);
    console.log(`Named room name: ${namedRoomResult.roomName}`);
    console.log(`Is named room created? : ${namedRoomResult.created}`);
}
```

### Match Party

Party Matchmaking is a special type of user matchmaking that involves having two or more users associated with a party registered in a user pool to find other users who are eligible and enter a newly created room together. The users associated with the party will always be in the same room. The user matched as a party may be another party or individual depending on the server's metric implementation.

To perform a party matchmaking, you must first be in the named room. If you set isParty to true on a name room request, the namedRoom will act as the party. After collecting all party users in NamedRoom, you can start matchmaking the party.

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const matchStartResult = await matchPartyStart(roomType, mathchingGroup, payload);

console.log(`Party match start result: ${ResultCodeMatchPartyStart[matchStartResult.errorCode]}`);
```

You can check whether the party matchmaking is started normally through the resultCode of Result, the Promise result value, as shown below:

```typescript
if (matchStartResult.resultCode === ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS) {
    console.log("Match party start success");
} else {
    console.log("Match party start fail");
}
```

If the party match failed to start, you can find out about the cause through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 1301 | Failed: Denied from content |
| `MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD` | 1302 | Failed: When requesting party match, if the room is not a room for party matching |

If the start of the party match is normal, matching the party starts on the server. At this time, if users in the same party want to receive a notification at the start, you can register a processing function as follows:

```typescript
user.onMatchPartyStart = (user, resultCode, payload) => {
    console.log(`User in party started match. result: ${ResultCodeMatchPartyStart[resultCode]}`);
}
```

When matching the party succeeds, a process function pre-registered to a user within the party is called. You can register a callback before the party match starts as shown below:

```typescript
user.onMatchUserDone = (user, resultCode, matchResult) => {
    console.log(`Match user done.`, resultCode);

    console.log(`Is request canceled?`, matchResult.isCancel);
    console.log(`Matched room id: `, matchResult.roomId);
    console.log(`Matched room name: `, matchResult.roomName);
    console.log(`Is room created for match?`, matchResult.created);
};
```

You can check the status before and after the party match, as shown below:

```typescript
console.log(`Is in progress of match making? ${user.isPartyMatchInProgress}`);
```

### Cancel Party Match

If the party matchmaking is still in progress, you can cancel the request.

```typescript
const roomType: string;

const matchCancelResult = await matchPartyCancel(roomType);

console.log(`Match party cancel result: ${ResultCodeMatchPartyCancel[matchCancelResult.errorCode]}`);
```

You can check whether it was normally canceled using the resultCode of Result, the Promise result value, as shown below:

```typescript
if (matchCancelResult.resultCode === ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS) {
    console.log("Match party cancel success.");
} else {
    consoel.log("Match party cancel fail.");
}
```

If the cancellation fails, you can find out about the source through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 1401 | Failed: Denied from the content |
| `MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD` | 1402 | Failed: When the party match is canceled, if the room is not a room for the party match |

### Send Packet

You can transfer user packets to a game server. Note that only pre-registered protocols can be sent.

For registering protocols or creating messages, see the following guide:

```typescript
const name = "John Doe";
const age = 20;
const job = "artist";
const message = new UserInfo({name, age, job});

user.sendUser(message);
```

### Wait for Response Packet after Sending Packet

After sending the user's packet to the game server, you can receive and process the response from the server. Note that only pre-registered protocols can be sent.

For registering protocols or creating messages, see the following guide:

```typescript
const echoResult = await user.requestUser<EchoRes>(new EchoReq({ message: "Hello World!"}), EchoRes);

console.log(echoResult.message); // Hello World!
```

### Move Channel

You can move the user from the channel the user belongs to to the specified channel. 

```typescript
const channelId: string;
const payload: Payload;

const moveChannelResult = await user.moveChannel(channelId, payload);

console.log(`Move channel result: ${ResultCodeMoveChannel[moveChannelResult.errorCode]}`);
```

You can check whether channel migration proceeded normally through the resultCode of Result, the Promise result value, as shown below:

```typescript
iF (moveChannelResult.resultCode === ResultcodeMoveChannel.MOVE_CHANNEL_SUCCESS) {
    console.log("Move channel success.");
} else {
    consoel.log("Move channel fail.");
}
```

If the channel move fails, you can find out about the source through errorCode. The following are the errorCode types you can receive from the server:

| Code Name | Value | Description |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | Packet parsing error |
| `TIMEOUT` | -1 | Timeout |
| `SYSTEM_ERROR` | 1 | Server system error |
| `INVALID_PROTOCOL` | 2 | Protocol not registered on server |
| `LOGOUT_SUCCESS` | 0 | Success |
| `LOGOUT_FAIL_CONTENT` | 1601 | Failed: Denied from the content |
| `MOVE_CHANNEL_FAIL_NODE_NOT_FOUND` | 1602 | Failed: Channel node not found |
| `MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL` | 1603 | Failed: Already in the requested channel |
| `MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM` | 1604 | Failed: Channel cannot be moved because you already entered the room |

If channel migration succeeded, you can see the results as follows:

```typescript
if (moveChannelResult.resultCode === ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS) {
    console.log(`Move channel forced: ${moveChannelResult.data.force}`);
    consoel.log(`Move channel id: ${moveChannelResult.data.channelId});
}
```

Channel moves may also be forced by the server. In that case, we register and process the callback as shown below:

```typescript
user.onMoveChannel = (user, result) => {
    console.log(`Move channel forced: ${result.force}`);
    consoel.log(`Move channel id: ${result.channelId});

}
```

If you want to know the channel ID information for the current user, please refer to the channelId property.

```typescript
console.log(`Current channel id: ${user.channelId}`);
```

### Notifications from Server

You can register a pre-process function for notifications coming from the server. If you want a more comlicated form of data transfer, consider registering custom protocols.

```typescript 
user.onNotice = (user, message) => {
    console.log(`Message from server: ${message});
}
```

### Disconnect

You can pre-register the process function when it is disconnected by the server or for the other reasons.

```typescript
user.onSessionClose = (user, resultCode, payload) => {
    console.log(`Session closed. ${ResultCodeSessionClosed[resultCode]}`);
}
```

| Code Name | Value | Description |
|----------------------------------------------|-------|------------------------------------------------------------------------------------------------------|
| `SESSION_CLOSE_BASE_USER` | 2011 | The server called BaseUser's `closeConnection()` |
| `SESSION_CLOSE_ADMIN_KICK` | 2012 | The connection was forced to close by the admin |
| `SESSION_CLOSE_DUPLICATE_LOGIN` | 2032 | The connection was forced to close due to a duplicate connection |
| `SESSION_CLOSE_BY_NEW_CONNECTION` | 2040 | The previous connection was closed when a new login request was made with the same account information. Use when connecting back to the network. Inquiry required. |
| `SESSION_CLOSE_DISCONNECT_ALARM_FROM_CLIENT` | 2041 | Detect disconnection with the client. It generally does not occur. If it occurs, you need to contact the GameAnvil development team. |
| `SESSION_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION` | 2042 | If the session could not be found. It generally does not occur. If it occurs, you need to contact the GameAnvil development team.

### Forced Exit by Admin

You can pre-register the process function when it is extracted from the server by the server's administrator tool.

```typescript
user.onAdminKickout = (user, message) => {
    console.log(`Message from server: ${message}`);
}
```
