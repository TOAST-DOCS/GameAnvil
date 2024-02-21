## Game > GameAnvil > CocosCreator Development Guide > Message handling

## Message

In addition to the default feature such as ConnectionAgent and UserAgent, a message can be sent to the server using Request and Send. To send a message, one must be created and registered.

### Create Message

GameAnvil uses [Google Protocol Buffers](https://developers.google.com/protocol-buffers/docs/proto3) as the default message protocol. It defines the message in the proto file and create actual class source code in pbjs. And insert the additional code to be used by GameAnvil into the created code. You can add this created source code to your project and use it. 

To insert additional code, `CodeInserter` must be installed. You can download `CodeInserter` [here](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-CodeInserter.zip). Unzip the downloaded file and create a folder under the project route (outside the asset folder).

![codeInserter](https://static.toastoven.net/prod_gameanvil/images/client-2-codeInserter.png)

`CodeInserter` uses acorn. Enter the following code in the terminal to install acorn. 

```
npm i acorn@5.5.3 --save-dev
```

acorn will be added to the devDependencies of package.json.

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

Now, let's create a message. First, create the protocols folder under the Assets folder and create the messages.proto file as follows.

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

Next, add the script for compiling the messages.proto file to package.json. 

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

Then enter the following code in the terminal to create a message.

```
npm run messages
```

The `message.js` and `message.d.ts` files will be created.

![messages](https://static.toastoven.net/prod_gameanvil/images/client-2-messages.png)

### Register Message

To use the newly created message, it must be pre-registered to ProtocolManager with a value identical to that of the server. If the value is not registered or different than that of the server, it may not work or work erroneously, or triggers an exception.

```typescript
// You must register with the same value as the server. 
ProtocolManager.RegisterProtocol(0, message);
```

### Send Message

Send a message using RequestPb and wait for the server to respond. While waiting for the server response, additional RequestPb is stored in the queue and processed later after the server response is processed. To receive and process a server response, you must register a listener. If no listener is registered, the next message is processed without separate notification even with a server response. If there is no response within a set time, a timeout occurs and the next message is processed. The timeout is transferred to the OnError listener as ErrorCode.TIMEOUT.

When a message is sent to SendPb(), it is immediately sent to the server as soon as SendPb() is called and does not wait for a separate response. While waiting for a response to RequestPb(), messages using SendPb() are sent directly to the server.

```typescript
let connection = GameAnvilManager.GetInstance().GetConnectionAgent(); 
 
connection.AddCallback(Messages.SampleReceive, (connectionAgent, receive)=>{ 
    // Messages.SampleReceive 
}); 
let sampleSend = new Messages.SampleSend ();  
connection.SendPb(sampleSend); 
 
 
connection.AddCallback(Messages.SampleResponse, (connectionAgent, response)=>{ 
    // Messages.SampleResponse 
}); 
let sampleRequest = new Messages.SampleRequest(); 
connection.RequestPb(sampleRequest); 
// In response Messages.SampleResponse. 
 
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
// In response Messages.SampleResponse. 
 
user.RequestPb<Messages.SampleResponse>(sampleRequest, (connectionAgent, response)=>{ 
    // Messages.SampleResponse 
});
```

### Custom Message

By using packet classes, you can serialize any random data other than ProtocolBuffer into a byte stream to use. Refer to [here](cocos-06-packet.md) for more information on packets.

```typescript
let connection = GameAnvilManager.GetInstance().GetConnectionAgent(); 
let reqMsgId = 1; 
let resMsgId = 2; 
let enc = new TextEncoder(); 
 
connection.AddUndefinedProtocolCallback(resMsgId, this.onUndefinedProtocolResponse); 
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data")); 
connection.Request(packet); 
// onUndefinedProtocolResponse Response 
 
connection.Request(packet, (connectionAgent, packet) => { 
    // Response here
}); 
let user = GameAnvilManager.GetInstance().GetUserAgent(this.ServiceName); 
let reqMsgId = 1; 
let resMsgId = 2; 
let enc = new TextEncoder(); 
 
user.AddUndefinedProtocolCallback(resMsgId, this.onUndefinedProtocolResponse); 
let packet = Packet.CreateFromBuffer(reqMsgId, enc.encode("Test Data")); 
user.Request(packet); 
// onUndefinedProtocolResponse Response 
 
user.Request(packet, (connectionAgent, packet) => { 
    // Response here
});
```