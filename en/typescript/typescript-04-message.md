## Game > GameAnvil > Guide to CocosCreator Development > Message Handling

## Message

In addition to the GameAnvilConnector and GameAnvilUser default features, you can send custom messages to the server using request() and send(). To send a message, you must pre-register the message, and a process to create the message object is required.

### Create Message

GameAnvil provides a message protocol based on [Google Protocol Buffers](https://developers.google.com/protocol-buffers/docs/proto3) by default. The message is defined in the .proto file and translated to the actual class source code with protoc. It adds the generated source code to your project and use the code.

Write the message you want to use first and place it in the proto folder.

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

Next, install a Library to transfile the message.proto file.

```shell
npm install protobufjs@7.2.6 protobufjs-cli@1.1.2
```

First use pbjs to complete the transfile to the .js file and then insert the required code using the insertCode.js of the game-server-connector. Transfile the file to a .ts file using later pbts. The below is an example code to automatically perform the tasks described for a specific directory:

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

The above script is saved as bin/protoc.js and registered as follows in package.json. This is a script to store the result of transpiling the .proto file in the ./proto folder in ./protocol.

```json
{
  "name": "my-game",
  "scripts": {
    "protoc" : "node ./bin/protoc.js ./proto ./protocol"
  }
}
```

Check if all files are at their locations.

* /bin/protoc.js
* /proto/message.proto
* Pre-create /protocol folder

If there is no problem, run protoc.js using npm.

```shell
npm run protoc
```

You can then check that the .ts and .d.ts files are created in the ./protocol folder.

### Register Message

To use newly created messages, you must pre-register the messages you want to use in GameAnvilProtocolManager. If you do not pre-register before Authenticate, or if the server and protocol are different, it may be disabled.

```typescript
import { Messages } from "./protocol/message";

GameAnvilProtocolManager.registerProtocol(Messages);
```

To disable the registered message, use the unregisterProtol() method.

```typescript
GameAnvilProtocolManager.unregisterProtocol(Messages);
```

### Send Message

You can send messages created through connectors or users. The following is an example of sending messages through a connector:

```typescript
import { Messages } from "./protocol/message";

const name: string;
const age: number;
const job: string;

const userInfo: UserInfo = new UserInfo({name, age, job});
connector.send(userInfo);
```

Below is an example of sending messages through connectors and waiting for the response from the server:

```typescript
const message = "Hello World!";
const echoReq: EchoReq = new EchoReq({message});

const result = await connector.requestMessage(echoReq, EchoRes.descriptor);

if (result.resultCode === ResultCode.Success) {
    const echoRes: EchoRes = result.data;
    consoel.log(echoRes.message); // Output Hello World!
}
```

The following is an example of sending messages through a user:

```typescript
import { Messages } from "./protocol/message";

const name = "John Doe";
const age = 20;
const job = "artist";

const message: UserInfo = new UserInfo({name, age, job});
user.sendUser(message);
```

Below is the example of sending messages through the user and waiting for the response from the server:

```typescript
const echoResult = await user.requestUser<EchoRes>(new EchoReq({ message: "Hello World!"}), EchoRes);

console.log(echoResult.message); // Out Hello World!
```