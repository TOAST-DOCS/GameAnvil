## Game > GameAnvil > TypeScript 개발 가이드 > 연결 에이전트

## GameAnvilConnector

GameAnvilConnector는 서버 연결과 통신을 담당하는 클래스로, 이 객체를 통해 서버에 요청하거나 서버로부터 오는 메시지에 대한 핸들러를 등록하여 관리할 수 있습니다. 이전 설치 챕터에서 GameAnvilConnector의 생성과 connect 기능 사용에 대해서 다루었습니다. 이번 문서에서는 더 자세한 사용법과, GameAnvilConnector의 다른 기능들을 알아보겠습니다.

### 생성

아래와 같이 GameAnvilConnector 객체를 생성합니다. 보통은 하나의 프로세스에서 하나의 커넥터 객체를 사용하는 것이 일반적입니다.

```typescript
import { GameAnvilConnector } from "gameanvil-connector";

const connector = new GameAnvilConnector();
```

### 서버 접속

connect() 함수를 이용해 서버에 접속합니다. 호출 전에 host와 port를 미리 설정 해야 합니다.

```typescript
connector.host = "127.0.0.1";
connector.port = 18300;

await connector.connect();

console.log("Connect Success");

```

connect() 함수는 서버 접속 완료 또는 실패 이후에 완료되는 Promise 객체를 반환합니다. 따라서 아래와 같이 사용할 수도 있습니다.

```typescript
connector.host = "127.0.0.1";
connector.port = 18300;

connector.connect()
    .then(() => {
        console.log("Connect Success");
    })
    .catch(()=> {
        console.log("Connect Fail");
    })
```

커넥터의 대부분의 API가 이처럼 비동기적으로 작동하며 Promise 객체를 반환하므로 위와 같이 상황에 맞게 await, then등의 기능을 사용할 수 있습니다.

### 연결 끊김 감지

서버에 의해 연결을 강제적으로 종료되거나, 네트워크 문제 등으로 연결이 끊겼을 때 수행할 동작을 지정할 수 있습니다.

```typescript
connector.onDisconnect = (resultCode: ResultCodeDisconnect, payload: Payload) => {
    console.log("Disconnected.");
    
}
```

함수의 첫 번째 인자로는 연결이 끊긴 이유를 알 수 있습니다.

| 코드 이름                                         | 값   | 설명 |
|---------------------------------------------------|-------|-----------------------------------------------------------------------------------------------------------------------------------------|
| `FORCE_CLOSE_SYSTEM_ERROR`                       | 2000  | 시스템 오류로 인한 강제 종료 상황입니다. 클라이언트에서 이 코드를 받은 경우 GameAnvil 개발팀에 문의하세요. |
| `FORCE_CLOSE_BASE_CONNECTION`                    | 2010  | 서버에서 BaseConnection의 close() 호출 시 받게 되는 코드입니다. |
| `FORCE_CLOSE_BASE_USER`                          | 2011  | 서버에서 BaseUser의 closeConnection() 호출 시 받게 되는 코드입니다. |
| `FORCE_CLOSE_ADMIN_KICK`                         | 2012  | Admin에서 강제 종료 하였을 경우 받게 되는 코드입니다. |
| `FORCE_CLOSE_INVALID_NODE`                       | 2020  | GameNode가 invalid 상태로 변경 되어 |
| `FORCE_CLOSE_USER_TRANSFER_FAIL`                 | 2021  | 유저 트렌스퍼가 실패한 경우 |
| `FORCE_CLOSE_USER_TRANSFER_ERROR`                | 2022  | 유저 트렌스퍼 중 시스템 에러가 발생한 경우 |
| `FORCE_CLOSE_AUTHENTICATION_FAIL`                | 2030  | 인증 실패로 인한 강제 종료|
| `FORCE_CLOSE_AUTHENTICATION_FAIL_EMPTY_ACCOUNT_ID`| 2031  | 인증 실패로 인한 강제 종료 - 어카운트아이디가 없을 경우 |
| `FORCE_CLOSE_DUPLICATE_LOGIN`                    | 2032  | 중복 접속으로 인한 강제 종료 |
| `FORCE_CLOSE_BY_NEW_CONNECTION`                  | 2040  | 같은 계정 정보로 새로운 로그인 요청이 들어온 경우. 네트워크 순단 등으로 재접속 시 이전 접속을 종료. 문의 필요. |
| `FORCE_CLOSE_DISCONNECT_ALARM_FROM_CLIENT`       | 2041  | 클라이언트와의 연결 끊김을 감지. 문의 필요.|
| `FORCE_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION`  | 2042  | 세션 정보를 찾을 수 없는 경우. 문의 필요.|
| `FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL`            | 2043  | 클라이언트가 서버 상태 체크에 응답하지 않은 경우. 문의 필요. |
| `FORCE_CLOSE_GHOST_USER`                         | 2044  | 고스트 유저인 경우. 문의 필요. |
| `SOCKET_DISCONNECT`                              | 2100  | 네트워크 연결이 끊어짐 |
| `SOCKET_TIME_OUT`                                | 2101  | 타임아웃이 발생, 컨넥터에서 연결을 끊음 |
| `SOCKET_ERROR`                                   | 2102  | 소켓 에러가 발생하여 연결을 끊음 |


두 번째 인자로는 서버 구현에 따른 추가 정보를 받습니다. 추가 정보 처리 방법은 이후에 추가로 설명합니다.

### 인증

서버에 접속 성공한 이후, 엔진의 모든 기능을 사용하기 위해서는 먼저 인증을 진행 해야 합니다. authenticaion() 함수는 서버와 미리 협의된 accountId, deviceId, password 값을 인자로 받아 인증 동작을 수행하고 Promise를 반환합니다. 인증 동작 완료 시점에 Promise를 통해 인증에 성공했는지 여부와 서버로부터 전달 받은 추가 데이터 등을 확인할 수 있습니다.

```typescript
const deviceId, accountId, password;

const authResult = await connector.authentication(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);
```

인증 성공 여부는 Promise 결과값인 ErrorResult의 errorCode를 통해 아래와 같이 확인할 수 있습니다.

```typescript
if (authResult.errorCode === ResultCodeAuth.AUTH_SUCCESS) {
    console.log("Authentication success");
} else {
    console.log("Authentication fail");
}
```

인증에 실패했을 경우 errorCode를 통해 그 원인을 알 수 있습니다. 다음은 서버로부터 받을 수 있는 errorCode 종류입니다.

| 코드 이름                        | 값   | 설명                                |
|----------------------------------|-------|------------------------------------|
| `PARSE_ERROR`                   | -2    | 추가 정보를 전달했을 때, 추가 정보를 파싱하는데 실패하면 발생할 수 있는 에러입니다. |
| `TIMEOUT`                       | -1    | 서버에서 제한 시간 내에 응답하지 않았을 때 전달 되는 에러입니다. |
| `SYSTEM_ERROR`                  | 1     | 서버 시스템 에러                   |
| `INVALID_PROTOCOL`              | 2     | 서버에 등록되지 않은 프로토콜을 이용한 추가 정보를 보냈을 때 발생하는 에러입니다. |
| `AUTH_SUCCESS`                  | 0     | 성공                               |
| `AUTH_FAIL_CONTENT`             | 201   | 서버 개발자가 작성한 인증 관련 코드에서 거부로 실패하였을 때 전달 되는 에러입니다. |
| `AUTH_FAIL_INVALID_ACCOUNT_ID`  | 202   | 빈 문자열 등, 서버에서 처리 불가능한 잘못된 계정 아이디를 전송했을 때 발생하는 에러입니다. |

인증 전/후 공통으로 커넥터가 인증 된 상태인지 확인하기 위해서, isAuthenticated 속성을 확인할 수 있습니다.

```typescript
if (connector.isAuthenticated) {
    console.log("Already authenticated.");
}
```

### 연결과 인증을 모두 진행

연결을 하고 나면 인증을 필수로 진행해야하므로 이 둘을 순차적으로 실행 시켜주는 편의 함수를 호출하면 편리합니다.

```typescript
const deviceId, accountId, password;

connector.host = "127.0.0.1";
connector.port = 18300;

const authResult = await connector.connectAndAuthenticateion(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);
```

연결 되지 않은 커넥터에 대해서 연결과 인증을 차례대로 수행하고, 인증 결과를 반환합니다.

인증 결과는 일반적으로 인증만 요청했을 때의 결과와 같은 방법으로 사용하면 됩니다.

### 핑 요청

기본적으로 핑 요청은 주기적으로 보내지도록 되어 있지만, 설정을 수정하여 수동으로 바꾸었을 경우 수동으로 메서드를 호출하여 핑 요청을 할 수 있습니다.

```typescript
connector.ping();
```

핑 요청에 대한 응답을 받았을 경우 설정에 따라 pong 로그가 출력될 수 있습니다.


### 메시지 수신 콜백 등록

서버로부터 프로토버퍼 메시지를 수신 받았을 때, 처리 함수를 실행하도록 설정할 수 있습니다. 하나의 프로토버퍼에 대해서 하나의 처리 함수만 등록이 가능하며, 이미 처리함수를 등록한 상태에서 다시 등록한다면 기존 처리 함수는 지워집니다.

처리 함수는 비동기로 동작하므로 의도한 타이밍에 정확히 호출 되지 않을 수 있음에 주의하십시오.

첫 번째 인자로 들어가는 프로토버퍼 메시지를 트랜스파일 하는 방법은 이후 챕터에서 설명합니다.

```typescript
connector.setMessageCallback(UserInfo.descriptor, (connector, resultCode, userInfo) => {
    if (resultCode === ResultCode.SUCCESS) {
        console.log(userInfo.name);
        console.log(userInfo.age);
        console.log(userInfo.job);

        return;
    }
    console.log("Message receive fail.", resultCode);
});
```

### 채널 유저 및 방 수 정보 요청

서버에 있는 특정 서비스의 모든 채널 각각의 유저와 방 수 정보를 요청할 수 있습니다.

```typescript
const serviceName: string;

const result = await connector.getAllChannelCountInfo(serviceName);
const allChannelCountInfo = result.data;

for (let [key, value] of allChannelCountInfo) {
    const channelId = key;
    const channlCountInfo = value;
    console.log(`${channelId} userCount: ${channlCountInfo.userCount}, roomCount: ${channlCountInfo.roomCount}`);
}
```

특정 서비스의 특정 채널로 한정지어서 요청할 수도 있습니다. 이 경우 서비스명과 함께 채널 아이디를 인자로 합니다.

```typescript
const serviceName: string;
const channelId: string;

const result = await connector.getChannelCountInfo(serviceName, channelId);
const channlCountInfo = result.data;

console.log(`${channelCountInfo.channelId} userCount: ${channlCountInfo.userCount}, roomCount: ${channlCountInfo.roomCount}`);
```


### 채널 정보 요청

서버에 있는 특정 서비스의 모든 채널 각각의 정보를 요청할 수 있습니다.

```typescript
const serviceName: string;

const result = await connector.getAllChannelInfo(serviceName);
const channelInfo = result.data.channelInfo;

for (let [key, value] of channelInfo) {
    const channelId = key;
    const payload = value;
}
```

특정 서비스의 특정 채널로 한정지어서 요청할 수도 있습니다. 이 경우 서비스명과 함께 채널 아이디를 인자로 합니다.

```typescript
const serviceName: string;
const channelId: string;

const result = await connector.getChannelInfo(serviceName, channelId);
const payload = result.data;
```

### 채널 목록 요청

서버에 있는 특정 서비스의 모든 채널 목록을 요청할 수 있습니다.

```typescript
const serviceName: string;

const result = await getChannelList(serviceName);

for (let channelId of result) {
    console.log(channelId);
}
```

### 유저 상태 체크 일시 정지 및 재개

앱이 백그라운드로 내려간 경우 등 유저 상태 체크에 응답할 수 없는 상황이 예상되는 경우 앱을 일시 정지 시킬 수 있습니다.

지정한 시간 동안 서버에서 클라이언트 접속 상태 정보를 갱신하지 않게 됩니다.

이 메서드로 앱을 일시 정지 시키지 않고 응답할 수 없게 된다면, 서버에서 주기적으로 클라이언트 접속을 종료시킵니다.

```typescript
const pauseTime: number = 10 * 1000;

connector.pauseClientStateCheck(pauseTime);
```

앱을 백그라운드에서 다시 실행한 경우 등 이제 유저 상태 체크에 응답할 수 있게 된다면, 앱을 재개합니다.

```typescript
connector.resumeClientStateCheck();
```

### 패킷 전송

게이트웨이 서버로 프로토버퍼 메시지를 전송할 수 있습니다.

메시지 객체의 제작은 이어지는 문서를 참고하세요.

```typescript
const name: string;
const age: number;
const job: string;

connector.send(new UserInfo({name, age, job}));
```

만약 서버에서 해당 메시지에 대한 응답을 대기하고 싶다면 아래와 같이 작성할 수 있습니다.

이 경우 서버에 미리 등록된 리스너를 통해서 응답을 보내야 합니다.

```typescript
const result = await connector.requestMessage(new StartReq(), StartRes.descriptor);

if (result.errorCode === ResultCode.Success) {
    const startRes: StartRes = result.data;
}
```

이미 메시지가 패킷의 형태라면 그대로 보낼 수도 있습니다.

```typescript
const packet = PacketFactory.makePacket(new StartReq());
const result = await connector.requestPacket(packet, StartRes.descriptor);

if (result.errorCode === ResultCode.Success) {
    const startRes: StartRes = result.data;
}
```

### 예외 핸들링

서버 접속 중에 예외가 발생할 경우 수행할 동작을 지정할 수 있습니다.

```typescript
connector.onException = (exception: Error) => {
    console.log("Error occured.", exception);
}
```

등록된 함수의 첫 번째 인자로는 에러 객체를 전달합니다.

### 연결 종료

서버와 명시적으로 연결을 종료하도록 요청할 수 있습니다.

```typescript
connector.disconnect();
```

서버와 연결 중이 아닌데 연결 종료 요청을 한 경우 에러가 발생합니다.

게임 플레이 종료 전 connector.disconnect() 함수를 호출해 연결을 종료하는 것이 좋습니다. 종료하지 않으면 서버에서 클라이언트의 종료를 인지하지 못할 수 있으며, 그럴 경우 불필요한 동작을 계속할 수 있습니다. connector를 관리하는 컴포넌트를 게임 종료시에만 파괴 되도록 만들고 OnDestroy()에서 connector.disconnect()를 호출하도록 만들어 놓는것도 좋은 방법입니다. 