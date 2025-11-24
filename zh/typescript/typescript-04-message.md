## Game > GameAnvil > CocosCreator 개발 가이드 > 메시지 핸들링

### 메시지

GameAnvilConnector, GameAnvilUser 기본 기능 외에 request()와 send()를 이용하여 커스텀 메시지를 서버로 전송할 수 있습니다. 메시지를 전송하려면 메시지를 미리 등록해야하고, 메시지 객체를 생성하는 과정이 필요합니다.

#### 메시지 생성

GameAnvil은 메시지 프로토콜로 [Google Protocol Buffers](https://developers.google.com/protocol-buffers/docs/proto3)를 기본으로 제공합니다. .proto 파일에 메시지를 정의하고, protoc로 실제 클래스 소스 코드로 트랜스파일 하게 됩니다. 이렇게 생성된 소스 코드를 프로젝트에 추가하여 사용합니다.

먼저 사용할 메시지를 작성해서 proto 폴더 안에 위치시킵니다.

```protocolbuffer
// message.proto
syntax = "proto3";

package Messages;

message UserInfo
{
  string name = 1;
  int32 age = 2;
  string job = 3;
}

message EchoReq
{
  string message = 1;
}

message EchoRes
{
  string message = 1;
}
```

다음으로 message.proto 파일을 트랜스파일 하기 위한 라이브러리를 설치합니다.

```shell
npm install protobufjs@7.2.6 protobufjs-cli@1.1.2
```

먼저 pbjs를 이용해 .js 파일로 트랜스 파일을 마친 후에는 game-server-connector의 insertCode.js를 이용해 필요한 코드를 삽입합니다. 이후 pbts를 사용해 .ts 파일로 트랜스파일합니다. 아래는 특정 디렉토리에 대해 설명한 작업을 자동으로 수행하는 예제 코드입니다.

```js
import fs from 'fs';
import { execSync } from 'child_process';
import path from 'path';

((srcDirectoryPath, distDirectoryPath) => {

    console.log(`Compile protobuf files in path: ${srcDirectoryPath}`);

    const srcFiles = fs.readdirSync(srcDirectoryPath);
    for ( const srcFileName of srcFiles){
        const distFileName = srcFileName.replace('.proto', '.js');

        const srcFilePath = path.join(srcDirectoryPath, srcFileName);
        const distFilePath = path.join(distDirectoryPath, distFileName);

        const packageName = getPackageNameFromProtoFile(srcFilePath);
        execSync(`pbjs --force-message --no-verify --no-convert --no-delimited -t static-module --wrap es6 --out ${distFilePath} ${srcFilePath}`);
        execSync(`node ../../game-server-connector/GameAnvil-connector-typescript/bin/insertCode.js ${distFilePath} ${srcFileName} ${packageName}`);
        execSync(`pbts --no-comments --out ${distFilePath.replace('.js', '.d.ts')} ${distFilePath}`);
    }

})(...process.argv.slice(2));

function getPackageNameFromProtoFile(filePath){
    const content = fs.readFileSync(filePath, {encoding:'utf8', flag:'r'});
    const packageLine = content.split(/\r?\n/).find(line => line.indexOf('package ') > -1);
    
    return packageLine.replace('package ', '').replace(';', '');
}
```

위 스크립트를 bin/protoc.js로 저장하고, package.json에 아래와 같이 등록해서 사용합니다. ./proto 폴더에 있는 .proto 파일을 트랜스파일한 결과를 ./protocol에 저장하는 스크립트입니다.

```json
{
  "name": "my-game",
  "scripts": {
    "protoc" : "node ./bin/protoc.js ./proto ./protocol"
  }
}
```

파일이 모두 제 위치에 있는지 다시 한번 확인합니다.

* /bin/protoc.js
* /proto/message.proto
* /protocol 폴더 미리 생성

문제가 없다면, npm을 이용해 protoc.js를 실행합니다.

```shell
npm run protoc
```

그러면 ./protocol 폴더에 .ts와 .d.ts 파일이 생성된것을 확인할 수 있습니다.

#### 메시지 등록

새로 생성한 메시지를 사용하려면 사용하려는 메시지를 GameAnvilProtocolManager에 미리 등록해야합니다. Authenticate전에 미리 등록하지 않거나, 서버와 프로토콜이 다를 경우 오동작 할 수 있습니다.

```typescript
import { Messages } from "./protocol/message";

GameAnvilProtocolManager.registerProtocol(Messages);
```

등록했던 메시지를 사용 해제하려면 unregisterProtol() 메서드를 사용합니다.

```typescript
GameAnvilProtocolManager.unregisterProtocol(Messages);
```

#### 메시지 전송

커넥터나 유저를 통해 생성한 메시지를 전송할 수 있습니다. 아래는 커넥터를 통해 메시지를 전송하는 예제입니다.

```typescript
import { Messages } from "./protocol/message";

const name: string;
const age: number;
const job: string;

const userInfo: UserInfo = new UserInfo({name, age, job});
connector.send(userInfo);
```

아래는 커넥터를 통해 메시지를 전송하고, 서버의 응답을 기다리는 예제입니다.

```typescript
const message = "Hello World!";
const echoReq: EchoReq = new EchoReq({message});

const result = await connector.requestMessage(echoReq, EchoRes.descriptor);

if (result.resultCode === ResultCode.Success) {
    const echoRes: EchoRes = result.data;
    consoel.log(echoRes.message); // Hello World! 출력
}
```

아래는 유저를 통해 메시지를 전송하는 예제입니다.

```typescript
import { Messages } from "./protocol/message";

const name = "John Doe";
const age = 20;
const job = "artist";

const message: UserInfo = new UserInfo({name, age, job});
user.sendUser(message);
```

아래는 유저를 통해 메시지를 전송하고, 서버의 응답을 기다리는 얘제입니다.

```typescript
const echoResult = await user.requestUser<EchoRes>(new EchoReq({ message: "Hello World!"}), EchoRes);

console.log(echoResult.message); // Hello World! 출력
```