## Game > GameAnvil > CocosCreator 개발 가이드 > 메시지 핸들링

## 메시지

ConnectionAgent, UserAgent의 기본 기능 외에 Request()와 Send()를 이용하여 메시지를 서버로 전송할 수 있습니다. 메시지를 전송하려면 메시지를 생성하고 등록하는 과정이 필요합니다.

### 메시지 생성

GameAnvil은 기본 메시지 프로토콜로 [Google Protocol Buffers](https://developers.google.com/protocol-buffers/docs/proto3)를 사용하고 있습니다. .proto파일에 메시지를 정의하고, pbjs 로 실제 클래스 소스 코드를 생성하게 됩니다. 그리고 GameAnvil에서 사용할 추가 코드를 생성된 코드에 삽입합니다. 이렇게 생성된 소스 코드를 프로젝트에 추가하여 사용할 수 있습니다. 

추가 코드를 삽입하려면 `CodeInserter` 를 설치해야합니다. `CodeInserter`는 [여기](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-CodeInserter.zip)에서 다운로드 받을 수 있습니다. 다운로드 받은 파일의 압축을 풀어 프로젝트 루트 아래(assets 폴더 밖에)에 폴더를 만들어 넣어줍니다.

![codeInserter](https://static.toastoven.net/prod_gameanvil/images/client-2-codeInserter.png)

`CodeInserter`는 acorn을 사용합니다. 터미널에서 다음 코드를 입력하여 acorn을 설치합니다. 

```
npm i acorn@5.5.3 --save-dev
```

package.json의 devDependencies에 acorn이 추가된 것을 확인할 수 있습니다.

```json
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
  },j
  "devDependencies": {
    "acorn": "^5.5.3"
  }
}
```

이제 메시지를 만들어 보도록 하겠습니다. 먼저 assets 폴더 아래에 protocols 폴더를 생성하고 다음과 같이 messages.proto 파일을 생성합니다.

```protobuf
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

```json
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

![messages](https://static.toastoven.net/prod_gameanvil/images/client-2-messages.png)

### 메시지 등록

새로 생성한 메시지를 사용하려면 사용하려는 메시지를 ProtocolManager에 서버와 같은 값으로 미리 등록해야 합니다. 등록하지 않거나 서버와 다를 경우 동작하지 않거나 오동작 하거나 예외가 발생 할 수 있습니다.

```typescript
// 서버와 같은 값으로 등록해야한다.
ProtocolManager.RegisterProtocol(0, message);
```

### 메시지 전송

RequestPb()로 메시지를 전송하면 서버 응답을 기다립니다. 서버 응답을 기다리는 동안 추가적인 RequestPb()는 큐에 저장되고 서버 응답을 처리한 후 순차적으로 처리하게 됩니다. 서버 응답을 받아 처리하려면 리스너를 등록해야 합니다. 리스너를 등록하지 않을 경우 서버 응답을 받아도 별도의 알림을 주지 않고 다음 메시지를 처리하게 됩니다. 지정된 시간 내에 응답이 오지 않으면 타임아웃을 발생시키고 다음 메시지를 처리합니다. 타임아웃은 OnError() 리스너에 ErrorCode.TIMEOUT으로 전달이 됩니다.

SendPb()로 메시지를  전송하면 SendPb()의 호출 즉시 서버로 전송되며 별도의 응답을 기다리지 않습니다. RequestPb()에 대한 응답을 기다리는 중에도 SendPb()를 사용한 메시지는 바로 서버로 전송됩니다.

```typescript
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

패킷 클래스를 이용하여 ProtocolBuffer 외의 임의의 데이터를 바이트 스트림으로 직렬화해 사용할 수 있습니다. 패킷에 대한 자세한 내용은 [여기](cocos-06-packet)를 참고합니다.

```typescript
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