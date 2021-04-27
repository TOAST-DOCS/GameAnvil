## Game > GameAnvil > Client Development Guide > CocosCreator Development Guide

Guide Environment

- CocosCreator : 2.2.2.
- Node.js : 10.16.3
- Visual Studio Code : 1.43.0
- GameAnvil Connector : 1.1.0

## Installing GameAnvil Connector

First, create a new CocosCreator project. Select **Cocos Creator Dashboard > New Project > Empty Project** and create a new project. 

![new-project](http://static.toastoven.net/prod_gameanvil/images/client-2-new-project.png)

![new-project-empty](http://static.toastoven.net/prod_gameanvil/images/client-2-new-project-empty.png)

Now, Add GameAnvil connector to the created project. GameAnvil connector can be downloaded from [here](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript.zip). Uncompress the downloaded file, create a folder in the assets folder, and place the file there. When the dialog box shown below appears, click **No**.

![set-as-a-plugin](http://static.toastoven.net/prod_gameanvil/images/client-2-set-as-a-plugin.png)

![gameanvil-project](http://static.toastoven.net/prod_gameanvil/images/client-2-gameanvil-project.png)

The GameAnvil connector uses protobufjs. Therefore, if the GameAnvil connector is included in a project, the following error will occur.
protobufjs needs to be included in a project using the install feature of Node.js. Enter the following code in the VSCode terminal to create the package.json file.

```
npm init
```

When the code above is entered, a package.json file creation process is started and the user is prompted to enter the name of the package. Press the Enter key to use the default project name. Other input items also take the default value when the user presses the Enter key. For more information, visit [here](https://docs.npmjs.com/cli/v6/commands/npm-init).  

![npm-init](http://static.toastoven.net/prod_gameanvil/images/client-2-npm-init.png)

The default package.json file contains the following: 

```
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

When the package.json file is created, enter the following code in the terminal to install protobufjs.

```
npm i protobufjs --save
```

![protobufjs](http://static.toastoven.net/prod_gameanvil/images/client-2-protobufjs.png)

When the installation is complete, the following folder will be created in the project folder.

- node_modules
- .bin
- @protobufjs
- @types
- long
- protobufjs

gameanvil_connector will be added in the dependencies of the package.json file as well.

```
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "protobufjs": "^6.10.2"
  }
}
```

core-js and regenerator-runtime need to be installed to support older versions of browsers such as Internet Explorer.

```
npm i core-js regenerator-runtime
```

## Creating and setting the preferences of the connector

To use a connector, import a connector and the ProtocolManager module. 

```
import { Connector, ProtocolManager} from 'GameAnvil/gameanvil';
```

Register the [message](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/## )to be used in the game and create a connector object.

```
export default class GameAnvilManager {
    private connector: Connector;
    constructor() {
        // Register the protocol.
        ProtocolManager.RegisterProtocol(0, GameMessages);

        // Create a connector.
        this.connector = Connector.Create();
    }
}
```

To process the [messages](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/## ) transferred among the servers, the Update() function need to be called regularly. The interval of the Update() function calling can be freely set, but if it is not called, the user will not receive a notification when the server sends a [message](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/## ).

```
setInterval(() => { this.connector.Update(); }, 10);
```

Create and use the following single-turn class for convenience. 

```
/*
    To use a connector on older versions of browsers such as Internet Explorer, 
    the user must use plugins such as core-js and regenerator-runtime.
*/
import 'core-js';
import 'regenerator-runtime';

import { Connector, ProtocolManager } from 'GameAnvil/gameanvil';

export default class GameAnvilManager {
    private static manager: GameAnvilManager;
    private connector: Connector;
    private constructor() {
        // Register the protocol.
        ProtocolManager.RegisterProtocol(0, GameMessages);

        // Create a connector.
        this.connector = Connector.Create();

        // A message loop. Call every 10ms.
        let updater = setInterval(() => { this.connector.Update(); }, 10);
    }

    static GetInstance() {
        if (this.manager == null) {
            this.manager = new GameAnvilManager();
        }
        return this.manager;
    }

    GetConnectionAgent() {
        return this.connector.GetConnectionAgent();
    }

    GetUserAgent(serviceName: string, subId: string) {
        return this.connector.GetUserAgent(serviceName, subId);
    }

    CreateUserAgent(serviceName: string, subId: string) {
        return this.connector.CreateUserAgent(serviceName, subId);
    }
}
```

## Connecting to and Authenticating the Server

ConnectionAgent is used to connect to a server and authenticate it. ConnectionAgent is responsible for tasks related to the connection node of the GameAnvil server. It provides basic session management features including Connect, Authentication and the channel list. It can also provide channel information or send and receive custom [messages](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/## )depending on the server implementation. ConnectionAgent is automatically created when the connector is initialized or can be obtained by using the Connector.GetConnectionAgent() function.

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// Connecting to Server
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void;
//  ipAddress : The address of the server to be connected. 
//  callback : The callback to which the result is received (Optional).
//    agent : The ConnectionAgent object that called Connect().
//    resultCode : The result of Connect.
connection.Connect(
    this.edtIPAddress.string,
    (agent, resultCode) => {
        console.log("Connect : " + ResultCodeConnect[resultCode]);
        if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) {
            // Connection successful
        } else {
            // Connection failed
        }
    }
);
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// Authentication. 
// Authenticate(deviceId: string, accountId: string, password: string, payload?: Payload, callback?: (agent: ConnectionAgent, resultCode: ResultCodeAuth, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload) => void): void;
//  deviceId : Used to check for duplicate logins
//  accountId : User account. 
//  password : Password.
//  payload : Used when additional data is required (Optional).
//  callback : The callback to which the result is received (Optional).
//   agent : The ConnectionAgent object that called Authenticate.
//   resultCode : The result of Authenticate
//   loginedUserInfoList : The information of the user who is using this account ID.
//   message : The additional message sent by server. 
//   payload: The additional data sent by the server content.
connection.Authenticate(
    "deviceId",
    this.edtAccountId.string,
    this.edtPassword.string,
    null,
    (agent, resultCode, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload) => {
        if (ResultCodeAuth.AUTH_SUCCESS == resultCode) {
            // Authentication successful
        } else {
            // Authentication failed
        }
    }
);
```

To disconnect, call Disconnect().

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// Disconnect
// Disconnect(reason: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string) => void, force: boolean, payload: Payload): void;
//   reason : The reason of disconnection. If there are many reasons for calling Disconnect(), it is used to distinguish each instance in callback.
//   callback : Receives the result (optional).
//     agent : The ConnectionAgent object that called Disconnect().
//     resultCode : The result of Disconnect().
//     reason : The reason of disconnection.
//     force : Whether it is forcibly disconnected from the server; duplicate login, Kick, AdminKick etc.
//     payload : The additional data sent by the server content.
connection.Disconnect("click Disconnect");
```

To process the disconnection instances besides calling Disconnect(), use ConnectionAgent.AddOnDisconnect() to separately register callback.

If the IConnectionListener interface is implemented and registered, all the notifications from ConnectionAgent can be received.

```
class ConnectionListener implements IConnectionListener{
    OnConnect?(connection: ConnectionAgent, resultCode: ResultCodeConnect): void { }
    OnAuthentication?(connection: ConnectionAgent, resultCode: ResultCodeAuth, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload): void { }
    OnChannelList?(connection: ConnectionAgent, resultCode: ResultCodeChannelList, channelIdList: Array<string>): void { }
    OnChannelInfo?(connection: ConnectionAgent, resultCode: ResultCodeChannelInfo, channelInfoList: Array<Payload>): void { }
    OnDisconnect?(connection: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string, force: boolean, payload: Payload): void { }
    OnErrorCommand?(connection: ConnectionAgent, errorCode: ErrorCode, msgName: string): void { }
}


let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
connection.AddListener(new ConnectionListener());

// address      : Server Address
connection.Connect(address);
// Respond with ConnectionListener.OnConnect


//  deviceId : Used to check for duplicate logins
//  accountId : User account. 
//  password : Password.
//  payload :  Used when additional data is required (Optional).
connection.Authenticate(deviceId, accountId, password, payload);
// Respond with ConnectionListener.OnAuthenticate
```

## Using Login and Default Features

Login/Logout and basic features can be used through UserAgent. UserAgent is responsible for the tasks related to the game node of GameAnvil server. It provides basic features such as Login, Logout and room management, and may provide custom features depending on the server implementation. To use UserAgent, you must create a new UserAgent using the Connector.CreateUserAgent() function. Several UserAgents, either ServiceId or SubId, and each created UserAgent can be used independently. Created UserAgents are internally managed in Connector and can be reused using the Connector.GetUser() function.

```
// serviceId    : The serviceId to be used by UserAgent. 
// subId        : A unique ID that can be used to identify UserAgent of each service. It may not be used depending on the server implementation
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
if (user == null) {
    user = GameAnvilManager.GetInstance().CreateUserAgent(this.ServiceName);
}
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
//  Log into Service.
// Login(userType: string, payload?: Payload, channelId?: string, callback?: (agent: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo) => void): void;
//   userType : UserType registered to the server. 
//   payload : Used when additional data is required (Optional).
//   channelId : The channel to be used (Optional. If it is left empty, it will be processed as an empty character string, the server must have a channel composed of an empty character string).
//   callback : The callback to which the result is received (Optional).
//     agent : The UserAgent object that called Login().
//     resultCode : The result of Login()
//     loginInfo : The information of the logged in user.
user.Login(this.UserType, null, this.editBoxChannel.string, (UserAgent, resultCode, loginInfo) => {
    if(resultCode == ResultCodeLogin.LOGIN_SUCCESS){
        // Login successful
    }else{
        // Login failed
    }
});
```

If the IUserListener interface is implemented and registered, all the notifications from UserAgent can be received.

```
class UserListener : IUserListener{
    OnLogin?(user: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo): void { }
    OnLogout?(user: UserAgent, resultCode: ResultCodeLogout, force: boolean, payload: Payload): void { }
    OnMatchRoom?(user: UserAgent, resultCode: ResultCodeMatchRoom, roomId: string, payload: Payload): void { }
    OnCreateRoom?(user: UserAgent, resultCode: ResultCodeCreateRoom, roomId: string, payload: Payload): void { }
    OnJoinRoom?(user: UserAgent, resultCode: ResultCodeJoinRoom, roomId: string, payload: Payload): void { }
    OnNamedRoom?(user: UserAgent, resultCode: ResultCodeNamedRoom, roomId: string, payload: Payload): void { }
    OnLeaveRoom?(user: UserAgent, resultCode: ResultCodeLeaveRoom, force: boolean, roomId: string, payload: Payload): void { }
    OnMatchUserStart?(user: UserAgent, resultCode: ResultCodeMatchUserStart, payload: Payload): void { }
    OnMatchUserCancel?(user: UserAgent, resultCode: ResultCodeMatchUserCancel): void { }
    OnMatchUserDone?(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: string, payload: Payload): void { }
    OnMatchUserTimeout?(user: UserAgent): void { }
    OnMatchPartyStart?(user: UserAgent, resultCode: ResultCodeMatchPartyStart, payload: Payload): void { }
    OnMatchPartyCancel?(user: UserAgent, resultCode: ResultCodeMatchPartyCancel): void { }
    OnMoveChannel?(user: UserAgent, resultCode: ResultCodeMoveChannel, force: boolean, channelId: string, payload: Payload): void { }
    OnSnapshot?(user: UserAgent, resultCode: ResultCodeSnapshot, paload: Payload): void { }
    OnErrorCommand?(user: UserAgent, errorCode: ErrorCode, msgName: string): void { }
    OnNotice?(user: UserAgent, message: string): void { }
    OnAdminKickoutNoti?(user: UserAgent, message: string): void { }
}


let user = connector.GetUserAgent(serviceId, subId);
user.AddUserListener(new UserListener())


// payload     :  An additional value to be delivered to the server. It may not be used depending on the server.
// channelId   : The channel ID with which UserAgent logs in. It may not be used depending on the server.
user.Login(payload, channelId);
// Respond with UserListener.OnLogin


user.Logout();
// Respond with UserListener.OnLogout


user.CreateRoom();
// Respond with UserListener.OnCreateRoom


user.MatchRoom();
// Respond with UserListener.OnMatchRoom


user.LeaveRoom();
// Respond with UserListener.OnLeaveRoom


...
```

## Message

In addition to the default feature such as ConnectionAgent and UserAgent, a message can be sent to the server using Request() and Send(). To send a message, one must be created and registered.

### Creating Message

GameAnvil uses [ProtocolBuffer ](https://developers.google.com/protocol-buffers/docs/overview) as its default message protocol. A message is defined in the .proto file and actual class source code is created using pbjs. Insert the additional code to be used on GameAnvil to the created code. The source code created as such can be added to and used for a project. 

To insert additional code, `CodeInserter` needs to be installed. `CodeInserter` can be downloaded from [here](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-CodeInserter.zip). Uncompress the downloaded file and place in a folder outside the assets folder under the project's root.

![codeInserter](http://static.toastoven.net/prod_gameanvil/images/client-2-codeInserter.png)

`CodeInserter` uses acorn. Enter the following code in the terminal to install acorn. 

```
npm i acorn@5.5.3 --save-dev
```

acorn will be added to the devDependencies of package.json.

```
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "protobufjs": "^6.10.2"
  },
  "devDependencies": {
    "acorn": "^5.5.3"
  }
}
```

Now, let's create a message. First, create the protocols folder under the Assets folder and create the messages.proto file as follows.

```
// messages.proto
syntax = "proto3";

package Messages;

message SampleRequest
{
  string msg = 1;
}

message SampleResponse
{
  repeated string msgs = 1;
}

message SampleSend
{
  string msg = 1;
}

message SampleReceive
{
  repeated string msgs = 1;
}
```

Next, add the script for compiling the messages.proto file to package.json. 

```
{
  "name": "newproject",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "messages.js": "pbjs --force-message --no-verify --no-convert --no-delimited -t static-module -w default -r base -o assets/protocols/messages.js assets/protocols/messages.proto",
    "messages.codeInsert": "node codeInserter/CodeInserter assets/protocols/messages.js",
    "messages.ts": "pbts --no-comments -o assets/protocols/messages.d.ts assets/protocols/messages.js",
    "messages": "npm run messages.js && npm run messages.codeInsert && npm run messages.ts",
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "protobufjs": "^6.10.2"
  },
  "devDependencies": {
    "acorn": "^5.5.3"
  }
}
```

Then enter the following code in the terminal to create a message.

```
npm run messages
```

The `message.js` and `message.d.ts` files will be created.

![messages](http://static.toastoven.net/prod_gameanvil/images/client-2-messages.png)

### Registering Message

To use the newly created message, it must be pre-registered to ProtocolManager with a value identical to that of the server. If the value is not registered or different than that of the server, it may not work or work erroneously, or triggers an exception.

```
// The value registered must be equal to that of the server.
ProtocolManager.RegisterProtocol(0, message);
```

### Transferring Message

Send a message using RequestPb() and wait for the server to respond. While waiting for the server response, additional RequestPb()is stored in the queue and processed later after the server response is processed. To receive and process a server response, you must register a listener. If no listener is registered, the next message is processed without separate notification even with a server response. If there is no response within a set time, a timeout occurs and the next message is processed. The timeout is transferred to the OnError() listener as ErrorCode.TIMEOUT.
When a message is sent via SendPb(), it is instantly delivered to the server when SendPb() is called and does not wait for separate response. Messages sent by using SendPb() are immediately transferred to the server even while waiting for the response of RequestPb().

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();

connection.AddCallback(Messages.SampleReceive, (connectionAgent, receive)=>{
    // Messages.SampleReceive
});
let sampleSend= new Messages.SampleSend (); 
connection.SendPb(sampleSend);


connection.AddCallback(Messages.SampleResponse, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
let sampleRequest = new Messages.SampleRequest();
connection.RequestPb(sampleRequest);
// Messages.SampleResponse as a response

connection.RequestPb<Messages.SampleResponse>(sampleRequest, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
user.AddCallback(Messages.SampleReceive, (connectionAgent, receive)=>{
    // Messages.SampleReceive
});
let sampleSend= new Messages.SampleSend (); 
user.SendPb(sampleSend);


user.AddCallback(Messages.SampleResponse, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
let sampleRequest = new Messages.SampleRequest();
user.RequestPb(sampleRequest);
// Messages.SampleResponse as a response.

user.RequestPb<Messages.SampleResponse>(sampleRequest, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
```

### Custom Message

Arbitrary data except ProtocolBuffer can be serialized as byte stream and used via packet class. For more information on packet, visit [here](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##Packet).

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
let reqMsgId = 1;
let resMsgId = 2;
let enc = new TextEncoder();

connection.AddUndefinedProtocolCallback(resMsgId, this.onUndefinedProtocolResponse);
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data"));
connection.Request(packet);
// onUndefinedProtocolResponse response

connection.Request(packet, (connectionAgent, packet) => {
    // Respond here
});
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
let reqMsgId = 1;
let resMsgId = 2;
let enc = new TextEncoder();

user.AddUndefinedProtocolCallback(resMsgId, this.onUndefinedProtocolResponse);
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data"));
user.Request(packet);
// onUndefinedProtocolResponse response

user.Request(packet, (connectionAgent, packet) => {
    // Respond here
});
```

## Packet

All the messages transferred between server are processed in a packet module and use the interface provided by packet module.

### Creation

The GameAnvil connector uses Google Protocol Buffer as the default protocol. Creating a packet using Google Protocol Buffer is as below:

```
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);

let sampleMessage2 = packet.GetPbMessage<Messages.SampleMessage>();
```

A packet can be created without using Google Protocol Buffer.

```
let enc = new TextEncoder();
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data"));

let dec = new TextDecoder("utf-8");
let bytes =  packet.GetBytes();
let string = dec.decode(bytes);
let uint8arr = JsonUtil.Serialize(obj); // object to UInt8Array;
let packet = Packet.CreateFromBuffer(reqMsgId, uint8arr);

let bytes = packet.GetBytes();
let obj = JsonUtil.Deserialize(bytes);
```

### Compression

If the size of a packet is too large, it can be compressed to reduce data usage.  

```
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);
packet.compress();

if (packet.IsCompress())
    packet.Decompress();

let responseMsg = packet.GetPbMessage<Messages.SampleMessage>();
```

## Payload

When using the default API provided by GameAnvil, you may need additional data. For this, the default API includes the payload parameter to deliver additional data. The data required for the payload can be stored in a packet in a list form. You can add more data here and send it to the server or extract the message sent to the server.  

```
let userInfo = Packet.CreateFromPbMsg(new Messages.UserInfo());
let roomInfo = Packet.CreateFromPbMsg(new Messages.RoomInfo());

let payload = Payload.CreateFromPackets([userInfo, roomInfo]);

let userInfoPacket: Packet = payload.GetPacket(Messages.UserInfo);
let roomInfo2: Messages.RoomInfo = payload.GetPBMessage(Messages.RoomInfo);
let userInfo = Packet.CreateFromPbMsg(new Messages.UserInfo());
let roomInfo = Packet.CreateFromPbMsg(new GameMessages.RoomInfo());

let payload = Payload.CreateDefault();
payload.add(userInfo);
payload.add(roomInfo);

let userInfoPacket: Packet = payload.GetPacket(Messages.UserInfo);
let roomInfo2: Messages.RoomInfo = payload.GetPBMessage(Messages.RoomInfo);
```

## Ending GameAnvil Connector

Disconnecting via calling the Connector.CloseSocket() function before shutting down the game play is recommended. Otherwise, the server may not recognize that the client has shut down and keep performing unnecessary tasks.
