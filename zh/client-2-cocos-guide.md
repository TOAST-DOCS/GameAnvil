## Game > GameAnvil > 클라이언트 개발 가이드 > CocosCreator 개발 가이드

가이드 환경

- CocosCreator : 2.2.2.
- Node.js : 10.16.3
- Visual Studio Code : 1.43.0
- GameAnvil Connector : 1.1.0

## GameAnvil 커넥터설치

먼저 새 CocosCreator 프로젝트를 생성합니다. **Cocos Creator Dashboard > New Project > Empty Project**를 선택하고 새 프로젝트를 생성합니다. 

![new-project](http://static.toastoven.net/prod_gameanvil/images/client-2-new-project.png)

![new-project-empty](http://static.toastoven.net/prod_gameanvil/images/client-2-new-project-empty.png)

이제 생성된 프로젝트에 GameAnvil 커넥터를 추가합니다. GameAnvil 커넥터는 [여기](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-typescript.zip)에서 다운로드 받을 수 있습니다. 다운로드한 파일의 압축을 풀어 assets 아래에 폴더를 만들어 넣습니다. 이때 다음과 같은 알림이 나타나면 **No**를 클릭합니다.

![set-as-a-plugin](http://static.toastoven.net/prod_gameanvil/images/client-2-set-as-a-plugin.png)

![gameanvil-project](http://static.toastoven.net/prod_gameanvil/images/client-2-gameanvil-project.png)

GameAnvil 커넥터는 protobufjs를 사용합니다. 그래서 GameAnvil 커넥터를 프로젝트에 포함하면 위와 같은 오류가 나오는 것을 볼수 있습니다. 

protobufjs는 Node.js의 install 기능을 이용해 프로젝트에 포함시킵니다. VSCode 터미널에서 다음 코드를 입력하여 package.json 파일을 생성합니다.

```
npm init
```

위 코드를 입력하면 package.json 파일 생성 프로세스가 시작되고 패키지 이름을 입력받습니다. 이때 그냥 Enter를 누르면 기본값으로 프로젝트 이름을 사용합니다. 이후로 입력받는 값들도 Enter를 누르면 기본값을 사용합니다. 자세한 내용은 [여기](https://docs.npmjs.com/cli/v6/commands/npm-init)를 참고하세요.  

![npm-init](http://static.toastoven.net/prod_gameanvil/images/client-2-npm-init.png)

기본값으로 생성한  package.json 파일을 열어보면 다음과 같습니다. 

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

package.json 파일을 생성했으면 이제 터미널에서 다음 코드를 입력하여 protobufjs를 설치합니다.

```
npm i protobufjs --save
```

![protobufjs](http://static.toastoven.net/prod_gameanvil/images/client-2-protobufjs.png)

설치가 완료되면 프로젝트 폴더 아래에 다음과 같은 폴더가 생성됩니다.

- node_modules
- .bin
- @protobufjs
- @types
- long
- protobufjs

package.json 파일의 dependencies에  gameanvil_connector가 추가된 것도 확인할 수 있습니다.

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

Internet Explorer 등 구버전 브라우저를 지원하려면 core-js, regenerator-runtime를 추가로 설치합니다.

```
npm i core-js regenerator-runtime
```

## 커넥터생성 및 기본 설정

커넥터를 사용하려면 커넥터와 ProtocolManager 모듈을 가져옵니다(import). 

```
import { Connector, ProtocolManager} from 'GameAnvil/gameanvil';
```

게임에서 사용할 [메시지](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##메시지)를 등록하고, 커넥터 객체를 생성합니다.

```
export default class GameAnvilManager {
    private connector: Connector;
    constructor() {
        // 프로토콜 등록.
        ProtocolManager.RegisterProtocol(0, GameMessages);

        // 커넥터 생성.
        this.connector = Connector.Create();
    }
}
```

서버와 주고받는 [메시지](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##메시지) 처리를 위해 Update() 함수를 주기적으로 호출해야 합니다. Update() 함수의 호출 주기는 자유롭게 설정해도 무방하나 호출하지 않을 경우 서버로부터 [메시지](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##메시지)를 받더라도 이에 대한 알림을 받을 수 없습니다.

```
setInterval(() => { this.connector.Update(); }, 10);
```

다음과 같은 싱글 턴 클래스를 만들어 사용하면 편리합니다. 

```
/*
    Internet Explorer 등 구버전 브라우저에서 커넥터 사용하기 위해서는 
    core-js, regenerator-runtime 등의 플러그인을 사용해야 한다.
*/
import 'core-js';
import 'regenerator-runtime';

import { Connector, ProtocolManager } from 'GameAnvil/gameanvil';

export default class GameAnvilManager {
    private static manager: GameAnvilManager;
    private connector: Connector;
    private constructor() {
        // 프로토콜 등록.
        ProtocolManager.RegisterProtocol(0, GameMessages);

        // 커넥터 생성.
        this.connector = Connector.Create();

        // 메시지 루프. 10ms 마다 호출.
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

## 서버 접속 및 인증

서버 접속 및 인증은 ConnectionAgent를 이용해 진행합니다. ConnectionAgent는 GameAnvil 서버의 커넥션 노드와 관련된 작업을 담당합니다. 접속(Connect()), 인증(Authentication()) 등 기본 세션 관리 기능 및 채널 목록 등을 제공하며, 서버 구현에 따라 채널 정보를 제공하거나 사용자 정의 [메시지](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##메시지)를 주고받을 수 있습니다. ConnectionAgent는 커넥터가 초기화될 때 자동으로 생성되며 Connector.GetConnectionAgent() 함수를 이용해 얻을 수 있습니다. 

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// 서버 접속
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void;
//  ipAddress : 접속할 서버 주소. 
//  callback : 결과를 전달 받을 콜백.(선택 사항)
//    agent : Connect()를 호출한 ConnectionAgent 객체.
//    resultCode : Connect 결과.
connection.Connect(
    this.edtIPAddress.string,
    (agent, resultCode) => {
        console.log("Connect : " + ResultCodeConnect[resultCode]);
        if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) {
            // 접속 성공
        } else {
            // 접속 실패
        }
    }
);
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// 인증. 
// Authenticate(deviceId: string, accountId: string, password: string, payload?: Payload, callback?: (agent: ConnectionAgent, resultCode: ResultCodeAuth, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload) => void): void;
//  deviceId : 중복 접속을 체크하기 위해 사용.
//  accountId : 사용자 계정. 
//  password : 패스워드.
//  payload : 추가적으로 필요한 데이터가 있을 경우 사용.(선택 사항)
//  callback : 결과를 전달받을 콜백.(선택 사항)
//   agent : Authenticate를 호출한 ConnectionAgent 객체.
//   resultCode : Authenticate 결과
//   loginedUserInfoList : 이 어카운트 Id를 이용중인 로그인된 유저 정보.
//   message : 서버에서 보내주는 추가 메시지. 
//   payload: 서버 콘텐츠에서 보내주는 추가 데이터.
connection.Authenticate(
    "deviceId",
    this.edtAccountId.string,
    this.edtPassword.string,
    null,
    (agent, resultCode, loginedUserInfoList: Array<LoginedUserInfo>, message: string, payload: Payload) => {
        if (ResultCodeAuth.AUTH_SUCCESS == resultCode) {
            // 인증 성공
        } else {
            // 인증 실패
        }
    }
);
```

접속을 종료할 때는 Disconnect()를 호출합니다.

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
// 접속 종료
// Disconnect(reason: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeDisconnect, reason: string) => void, force: boolean, payload: Payload): void;
//   reason : 접송 종료 이유. Disconnect()를 호출하는 경우가 다양할 경우, 콜백에서 각각의 경우를 구분하기위해 사용.
//   callback : 결과를 전달 받음. (선택 사항)
//     agent : Disconnect() 호출한 ConnectionAgent 객체.
//     resultCode : Disconnect() 결과.
//     reason : 접속 종료 이유.
//     force : 서버에서의 강제종료 여부. 중복접속, Kick, AdminKick 등.
//     payload : 서버 컨텐츠에서 보내주는 추가 데이터.
connection.Disconnect("click Disconnect");
```

Disconnect() 호출 외에도 네트워크 환경으로 인한 접속 종료 등의 경우를 처리하기 위해 ConnectionAgent.AddOnDisconnect()를 이용해 별도로 콜백을 등록해 사용합니다.

IConnectionListener 인터페이스를 구현하여 등록하면 ConnectionAgent의 모든 알림을 받을 수도 있습니다.

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

// address      : 서버 주소
connection.Connect(address);
// ConnectionListener.OnConnect 로 응답


//  deviceId : 중복 접속을 체크하기 위해 사용.
//  accountId : 사용자 계정. 
//  password : 패스워드.
//  payload : 추가적으로 필요한 데이터가 있을 경우 사용.(선택 사항)
connection.Authenticate(deviceId, accountId, password, payload);
// ConnectionListener.OnAuthenticate 로 응답
```

## 로그인 및 기본 기능 사용

로그인/로그아웃 및 기본 기능은 UserAgent를 통해 사용할 수 있습니다. UserAgent는 GameAnvil 서버의 게임 노드와 관련된 작업을 담당합니다. 로그인(Login()), 로그아웃(Logout()) 및 룸 관리 등 기본 기능을 제공하며, 서버 구현에 따라 사용자 정의 기능을 추가적으로 제공할 수도 있습니다. UserAgent를 사용하기 위해서는 Connector.CreateUserAgent() 함수를 이용해 새로운 UserAgent를 생성해야 합니다. ServiceId 와 SubId로 구분되는 여러 개의 UserAgent를 생성할 수 있으며 생성된 각 UserAgent는 독립적으로 사용할 수 있습니다. 생성된 UserAgent는 커넥터에서 내부적으로 관리되며 Connector.GetUser() 함수를 이용해 다시 사용할 수 있습니다.

```
// serviceId    : UserAgent가 사용할 serviceId. 
// subId        : 서비스별 UserAgent 식별할 수 있는 고유 ID. 서버 구현에 따라 사용하지 않을 수 있음.
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
if (user == null) {
    user = GameAnvilManager.GetInstance().CreateUserAgent(this.ServiceName);
}
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
// Service에 로그인합니다.
// Login(userType: string, payload?: Payload, channelId?: string, callback?: (agent: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo) => void): void;
//   userType : 서버에 등록된 UserType. 
//   payload : 추가적으로 필요한 데이터가 있을 경우 사용.(선택 사항)
//   channelId : 사용할 할 채널. (선택 사항. 입력하지 않을 경우 빈 문자열로 처리되며, 서버에 빈 문자열로된 채널이 설정되어야 합니다.)
//   callback : 결과를 전달받을 콜백.(선택 사항)
//     agent : Login()을 호출한 UserAgent 객체.
//     resultCode : Login() 결과
//     loginInfo : 로그인된 유저 정보.
user.Login(this.UserType, null, this.editBoxChannel.string, (UserAgent, resultCode, loginInfo) => {
    if(resultCode == ResultCodeLogin.LOGIN_SUCCESS){
        // 로그인 성공
    }else{
        // 로그인 실패
    }
});
```

IUserListener 인터페이스를 구현하여 등록하면 UserAgent의 모든 알림을 받을 수도 있습니다.

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


// payload     : 서버로 전달할 추가적인 값. 서버 구현에 따라 사용하지 않을 수 있음.
// channelId   : UserAgent가 로그인할 채널 ID. 서버 구현에 따라 사용하지 않을 수 있음.
user.Login(payload, channelId);
// UserListener.OnLogin으로 응답


user.Logout();
// UserListener.OnLogout으로 응답


user.CreateRoom();
// UserListener.OnCreateRoom으로 응답


user.MatchRoom();
// UserListener.OnMatchRoom으로 응답


user.LeaveRoom();
// UserListener.OnLeaveRoom으로 응답


...
```

## 메시지

ConnectionAgent, UserAgent의 기본 기능 외에 Request()와 Send()를 이용하여 메시지를 서버로 전송할 수 있습니다. 메시지를 전송하려면 메시지를 생성하고 등록하는 과정이 필요합니다.

### 메시지 생성

GameAnvil은 기본 메시지 프로토콜로 [ProtocolBuffer](https://developers.google.com/protocol-buffers/docs/overview)를 사용하고 있습니다. .proto파일에 메시지를 정의하고, pbjs 로 실제 클래스 소스 코드를 생성하게 됩니다. 그리고 GameAnvil에서 사용할 추가 코드를 생성된 코드에 삽입합니다. 이렇게 생성된 소스 코드를 프로젝트에 추가하여 사용할 수 있습니다. 

추가 코드를 삽입하려면 `CodeInserter` 를 설치해야합니다. `CodeInserter`는 [여기](http://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-CodeInserter.zip)에서 다운로드 받을 수 있습니다. 다운로드 받은 파일의 압축을 풀어 프로젝트 루트 아래(assets 폴더 밖에)에 폴더를 만들어 넣어줍니다.

![codeInserter](http://static.toastoven.net/prod_gameanvil/images/client-2-codeInserter.png)

`CodeInserter`는 acorn을 사용합니다. 터미널에서 다음 코드를 입력하여 acorn을 설치합니다. 

```
npm i acorn@5.5.3 --save-dev
```

package.json의 devDependencies에 acorn이 추가된 것을 확인할 수 있습니다.

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

이제 메시지를 만들어 보도록 하겠습니다. 먼저 assets 폴더 아래에 protocols 폴더를 생성하고 다음과 같이 messages.proto 파일을 생성합니다.

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

다음으로 messages.proto 파일을 컴파일하기 위한 스크립트를 package.json에 추가합니다. 

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

그리고 터미널에서 다음 코드를 입력하여 메시지를 생성합니다.

```
npm run messages
```

다음과 같이 `message.js`, `message.d.ts` 파일이 생성된 것을 확인할 수 있습니다.

![messages](http://static.toastoven.net/prod_gameanvil/images/client-2-messages.png)

### 메시지 등록

새로 생성한 메시지를 사용하려면 사용하려는 메시지를 ProtocolManager에 서버와 같은 값으로 미리 등록해야 합니다. 등록하지 않거나 서버와 다를 경우 동작하지 않거나 오동작 하거나 예외가 발생 할 수 있습니다.

```
// 서버와 같은 값으로 등록해야한다.
ProtocolManager.RegisterProtocol(0, message);
```

### 메시지 전송

RequestPb()로 메시지를 전송하면 서버 응답을 기다립니다. 서버 응답을 기다리는 동안 추가적인 RequestPb()는 큐에 저장되고 서버 응답을 처리한 후 순차적으로 처리하게 됩니다. 서버 응답을 받아 처리하려면 리스너를 등록해야 합니다. 리스너를 등록하지 않을 경우 서버 응답을 받아도 별도의 알림을 주지 않고 다음 메시지를 처리하게 됩니다. 지정된 시간 내에 응답이 오지 않으면 타임아웃을 발생시키고 다음 메시지를 처리합니다. 타임아웃은 OnError() 리스너에 ErrorCode.TIMEOUT으로 전달이 됩니다.

SendPb()로 메시지를  전송하면 SendPb()의 호출 즉시 서버로 전송되며 별도의 응답을 기다리지 않습니다. RequestPb()에 대한 응답을 기다리는 중에도 SendPb()를 사용한 메시지는 바로 서버로 전송됩니다.

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
// 응답으로 Messages.SampleResponse.

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
// 응답으로 Messages.SampleResponse.

user.RequestPb<Messages.SampleResponse>(sampleRequest, (connectionAgent, response)=>{
    // Messages.SampleResponse
});
```

### 커스텀 메시지

패킷 클래스를 이용하여 ProtocolBuffer 외의 임의의 데이터를 바이트 스트림으로 직렬화해 사용할 수 있습니다. 패킷에 대한 자세한 내용은 [여기](https://alpha-docs.toast.com/ko/Game/GameAnvil/ko/client-2-cocos-guide/##Packet)를 참고합니다.

```
let connection = GameAnvilManager.GetInstance().GetConnectionAgent();
let reqMsgId = 1;
let resMsgId = 2;
let enc = new TextEncoder();

connection.AddUndefinedProtocolCallback(resMsgId, this.onUndefinedProtocolResponse);
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data"));
connection.Request(packet);
// onUndefinedProtocolResponse 응답

connection.Request(packet, (connectionAgent, packet) => {
    // 여기로 응답
});
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName);
let reqMsgId = 1;
let resMsgId = 2;
let enc = new TextEncoder();

user.AddUndefinedProtocolCallback(resMsgId, this.onUndefinedProtocolResponse);
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data"));
user.Request(packet);
// onUndefinedProtocolResponse 응답

user.Request(packet, (connectionAgent, packet) => {
    // 여기로 응답
});
```

## 패킷

서버와 주고받는 모든 메시지는 패킷 모듈에 실려서 처리되며 패킷 모듈이 제공하는 인터페이스를 이용하게 됩니다.

### 생성

GameAnvil 커넥터는 Google Protocol Buffer를 기본 프로토콜로 사용하고 있습니다 . Google Protocol Buffer를 이용하는 패킷생성은 다음과 같습니다.

```
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);

let sampleMessage2 = packet.GetPbMessage<Messages.SampleMessage>();
```

Google Protocol Buffer를 사용하지 않고도 패킷을 생성할 수 있습니다 .

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

### 압축

패킷 크기가 클 경우 압축하여 데이터 사용량을 줄일 수 있습니다 .  

```
let sampleMessage = new Messages.SampleMessage();
let packet= Packet.CreateFromPbMsg(sampleMessage);
packet.compress();

if (packet.IsCompress())
    packet.Decompress();

let responseMsg = packet.GetPbMessage<Messages.SampleMessage>();
```

## Payload

GameAnvil에서 제공하는 기본 API를 이용할 때 추가적인 데이터가 필요할 수 있습니다. 이를 위해 기본 API들에는 추가 데이터를 넘겨줄 수 있는 payload라는 매개변수가 포함되어 있습니다. 이 payload에 필요한 데이터를 패킷에 담아 list 형식으로 저장할 수 있습니다. 여기에 추가 데이터를 넣어 서버로 보내거나, 서버에서 보낸 메시지를 꺼낼 수 있습니다.  

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

## GameAnvil 커넥터종료

게임 플레이 종료 전 Connector.CloseSoket() 함수를 호출해 연결을 종료하는 것이 좋습니다. 종료하지 않으면 서버에서 클라이언트의 종료를 인지하지 못할 수 있으며, 그럴 경우 불필요한 동작을 계속할 수 있습니다. 