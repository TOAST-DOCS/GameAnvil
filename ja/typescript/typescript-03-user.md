## Game > GameAnvil > Typescript開発ガイド > User

## GameAnvilUser

GameAnvilUserは、サーバーのGameNode上に存在するユーザーに対応するクライアント側のオブジェクトです。サーバーのユーザーオブジェクトに命令を下したり、ユーザーの情報を受け取ってサーバーとクライアントを同期させたりできます。エンジンに事前に定義されたルーム機能やマッチメイキング機能なども利用できますが、直接プロトコルを実装して新しい機能を追加することも可能です。

### 生成

GameAnvilUserを使用するには、まず新しいGameAnvilUserオブジェクトを作成し、作成されたユーザーを通じてサーバーにログインする必要があります。

```typescript
const connector: GameAnvilConnector;
const serviceName: string;

const user = new GameAnvilUser(connector, serviceName, 1);
```

### 多数のGameUserAgent生成

GameAnvilConnectorオブジェクトはプロセス内で1つだけ使用するのが一般的ですが、GameAnvilUserは複数同時に作成して運用することがサポートされています。それぞれが異なるサービスにログインすることが可能で、もし1つのサービスで複数のGameAnvilUserを使用したい場合は、subIdを利用して区別して作成できます。

```typescript
const connector: GameAnvilConnector;
const serviceName: string;
const otherServiceName: string;

const user1 = new GameAnvilUser(connector, serviceName, 1);
const user2 = new GameAnvilUser(connector, serviceName, 2);
const user3 = new GameAnvilUser(connector, otherServiceName, 1);
```

### ログイン

GameNode内に、クライアントに対応するサーバーユーザーオブジェクトの作成をリクエストします。GameNodeへのログインが完了しないと、ユーザーの様々な機能は使用できません。

ログインするユーザータイプとチャネルIDを必須の引数として受け取り、3番目の引数で追加情報を送信できます。ログイン動作の完了時点で、Promiseを通じてログインに成功したかどうかや、サーバーから渡された追加データなどを確認できます。

```typescript
const userType: stirng;
const channelId: string;
const payload: Payload;

const loginResult = await user.login(userType, channelId, payload);
console.log(`Login Result : ${ResultCodeLogin[loginResult.errorCode]}`);
```

ログインの成否は、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (loginResult.resultCode === ResultCodeLogin.LOGIN_SUCCESS) {
    console.log("Login Success");
} else {
    console.log("Login Fail");
}
```

ログインに失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                                  | 値  | 説明                                                                     |
|---------------------------------------------|-------|---------------------------------------------------------------------------|
| `PARSE_ERROR`                               | -2    | パケットパースエラー                                                          |
| `TIMEOUT`                                   | -1    | タイムアウト                                                               |
| `SYSTEM_ERROR`                              | 1     | サーバーシステムエラー                                                        |
| `INVALID_PROTOCOL`                          | 2     | サーバーに登録されていないプロトコル                                           |
| `LOGIN_SUCCESS`                             | 0     | 成功                                                                    |
| `LOGIN_FAIL_CONTENT` | 301 | 失敗: コンテンツにより拒否されました |
| `LOGIN_FAIL_NOT_EXIST_NODE` | 302 | 失敗: ノードが存在しません |
| `LOGIN_FAIL_TIMEOUT_GAME_SERVER` | 303 | 失敗: ゲームサーバーが応答しません |
| `LOGIN_FAIL_INVALID_SERVICEID` | 310 | 失敗: 不正なサービスIDです |
| `LOGIN_FAIL_INVALID_USERTYPE` | 311 | 失敗: 不正なユーザータイプです |
| `LOGIN_FAIL_INVALID_USERID` | 312 | 失敗: 不正なユーザーIDです |
| `LOGIN_FAIL_INVALID_SUB_ID` | 313 | 失敗: 不正なSubIdです |
| `LOGIN_FAIL_INVALID_CHANNEL_ID` | 314 | 失敗: 不正なチャネルIDです |
| `LOGIN_FAIL_LOGINED_OTHER_SERVICE` | 320 | 失敗: 他のサービスにログインしています |
| `LOGIN_FAIL_LOGINED_OTHER_CHANNEL` | 321 | 失敗: 他のチャネルにログインしています |
| `LOGIN_FAIL_LOGINED_OTHER_USER_TYPE` | 322 | 失敗: 同じアカウントIDで別のユーザータイプがログインしています |
| `LOGIN_FAIL_LOGINED_OTHER_DEVICE` | 323 | 失敗: 同じアカウントIDで別のデバイスIDがログインしています |

ログインの前後に関わらず、ユーザーがログイン状態であるかを確認するために、isLoggedInプロパティを確認できます。

```typescript
if (user.isLoggedIn) {
    console.log("Already logged in.");
}
```

### ログアウト

サーバーに明示的にユーザーを削除するようにリクエストできます。ログアウト動作の完了時点で、Promiseを通じてログアウトに成功したかどうかや、サーバーから渡された追加データなどを確認できます。

```typescript
const logoutResult = await user.logout();
```

ユーザーがログイン状態でないのにログアウトを試みた場合、サーバーには何もリクエストせず、成功として応答します。

ログアウトの成否は、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (logoutResult.resultCode === ResultCodeLogout.LOGOUT_SUCCESS) {
    console.log("Logout Success");
} else {
    console.log("Logout Fail");
}
```

ログアウトに失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名             | 値  | 説明                             |
|------------------------|-------|-----------------------------------|
| `PARSE_ERROR` | -2 | パケットパースエラー |
| `TIMEOUT`              | -1    | タイムアウト                       |
| `SYSTEM_ERROR`         | 1     | サーバーシステムエラー                |
| `INVALID_PROTOCOL`     | 2     | サーバーに登録されていない プロトコル   |
| `LOGOUT_SUCCESS`       | 0     | 成功                            |
| `LOGOUT_FAIL_CONTENT` | 401 | 失敗: コンテンツにより拒否されました |

サーバーの設定や状況によっては、サーバー側でユーザーを強制的にログアウトさせることがあります。この場合に備え、以下のように処理関数を登録できます。

```typescript
user.onForceLogout = (user, payload) => {
    console.log(`User ${user.userId}} forced to logout.`);
}
```

### メッセージ受信コールバック登録

サーバーからプロトコルバッファメッセージを受信した際に、処理関数を実行するように設定できます。1つのプロトコルバッファに対しては、1つの処理関数のみ登録可能で、既に処理関数が登録されている状態で再度登録すると、既存の処理関数は削除されます。

処理関数は非同期で動作するため、意図したタイミングで正確に呼び出されない可能性がある点にご注意ください。

最初の引数となるプロトコルバッファメッセージをトランスパイルする方法は、後の章で説明します。

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

### ルーム新規生成後に入室

サーバーにルームを作成した後、すぐに入室できます。ルーム名が不要な場合は、空の文字列を渡します。ルームタイプは、サーバーと事前に協議した値を使用する必要があります。

マッチンググループは、マッチメイキング時に使用する情報です。使用しない場合は、空の文字列にしておいてください。

ルームの作成と入室が完了した時点で、Promiseを通じて成功したかどうかや、新しく作成されたルームのID、ユーザーカスタム追加情報などを確認できます。

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const resultCreateRoom = await user.createRoom("", roomType, matchingGroup, payload);
```

動作の成否は、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (resultCreateRoom.resultCode === ResultCodeCreateRoom.CREATE_ROOM_SUCCESS) {
     console.log("Create room success");
} else {
    console.log("Create room fail");
}
```

ルームの作成または入室に失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                               | 値  | 説明                               |
|------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR` | -2 | パケットパースエラー |
| `TIMEOUT`                                | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                           | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`                       | 2     | サーバーに登録されていない プロトコル     |
| `CREATE_ROOM_SUCCESS`                    | 0     | 成功                              |
| `CREATE_ROOM_FAIL_CONTENT` | 601 | 失敗: コンテンツにより拒否されました |
| `CREATE_ROOM_FAIL_ALREADY_JOINED_ROOM` | 602 | 失敗: 既にルームに入室しています |
| `CREATE_ROOM_FAIL_CREATE_ROOM_ID` | 603 | 失敗: ルームIDの発行に失敗しました |
| `CREATE_ROOM_FAIL_CREATE_ROOM` | 604 | 失敗: ルームの作成に失敗しました |

ルームの作成と入室に成功した場合、レスポンスを通じてroomIdを含む情報を確認できます。

```typescript
if (resultCreateRoom.resultCode === ResultCodeCreateRoom.CREATE_ROOM_SUCCESS) {
    console.log(`Created room id: ${resultCreateRoom.roomId}`);
    console.log(`Created room name: ${resultCreateRoom.roomName}`);
}
```

ルームに入室している状態かを確認するために、isJoinedRoomプロパティを確認できます。

```typescript
if (user.isJoinedRoom) {
    console.log("Alread joined room.");
}
```

ルームに入室している状態の場合、入室しているルームのID情報をroomIdプロパティで確認できます。

```typescript
console.log(`Current joined room id: ${user.roomId}`);
```

### 既存ルームに入室

サーバーで作成されたルームIDが分かっている場合、そのルームへの入室をリクエストできます。

マッチングユーザーカテゴリは、マッチメイキング時に使用する情報です。使用しない場合は、空の文字列にしておいてください。

ルームへの入室が完了した時点で、Promiseを通じて成功したかどうかや追加情報などを知ることができます。

```typescript
const roomType: string;
const roomId: number;
const matchinUserCategory: string;
const payload: Payload;

const resultJoinRoom = user.joinRoom(roomType, roomId, matchingUserCategory, payload);
```
ルーム入室成否は、Promise結果値であるResultのresultCodeを通じて以下のように確認できます。

```typescript
if (resultJoinRoom.resultCode === ResultCodeJoinRoom.JOIN_ROOM_SUCCESS) {
    console.log("Join room success");

} else {
    console.log("Join room fail");
}
```

ルームへの入室に失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                              | 値  | 説明                               |
|-----------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR` | -2 | パケットパースエラー |
| `TIMEOUT`                               | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                          | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`                      | 2     | サーバーに登録されていない プロトコル     |
| `JOIN_ROOM_SUCCESS`                     | 0     | 成功                              |
| `JOIN_ROOM_FAIL_CONTENT` | 701 | 失敗: コンテンツで拒否されました |
| `JOIN_ROOM_FAIL_ROOM_DOES_NOT_EXIST` | 702 | 失敗: ルームが存在しません |
| `JOIN_ROOM_FAIL_ALREADY_JOINED_ROOM` | 703 | 失敗: 既にルームに入室済みです |
| `JOIN_ROOM_FAIL_ALREADY_FULL` | 704 | 失敗: 既にルームが満員の場合 |
| `JOIN_ROOM_FAIL_ROOM_MATCH`             | 705   | 失敗: ルームマッチメイキングで問題が発生した場合 |

ルームへの入室に成功した場合、以下のように詳細情報を確認できます。
```typescript
if (resultJoinRoom.resultCode === ResultCodeJoinRoom.JOIN_ROOM_SUCCESS) {
    console.log(`Joined room id: ${resultJoinRoom.roomId}`);
    console.log(`Joined room name: ${resultJoinRoom.roomName}`);
}
```

### 入室中のルームから退場

入室中のルームから退室するよう、サーバーにリクエストできます。

退室完了時点で、Promiseを通じて成功したかどうかや追加情報などを確認できます。

```typescript
const payload: Payload;

const leaveRoomResult = await user.leaveRoom(payload);
```

退室が完了したかどうかは、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (leaveRoomResult.resultCode === ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS) {
    console.log("Leave room success");
} else {
    console.log("Leave room fail");
}
```

ルームからの退室に失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名             | 値  | 説明                               |
|------------------------|-------|-------------------------------------|
| `PARSE_ERROR`          | -2    | パケットパースエラー                    |
| `TIMEOUT`              | -1    | タイムアウト                         |
| `SYSTEM_ERROR`         | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`     | 2     | サーバーに登録されていない プロトコル     |
| `LEAVE_ROOM_SUCCESS`   | 0     | 成功                              |
| `LEAVE_ROOM_FAIL_CONTENT` | 801 | 失敗: コンテンツで拒否されました |

サーバーによって強制的にルームから退出させられた場合に実行する処理関数を登録できます。

```typescript
user.onForceLeaveRoom = (user, roomId, payload) => {
    console.log(`Server forced to leave room. roomId: ${roomId}`);
}
```

### ユーザーマッチメイキングプールに登録

ユーザーマッチメイキングは、ユーザープールを作成し、その中から条件に合うユーザーをまとめて新しく作成したルームに入室させる方式です。ユーザープールに条件に合うユーザーの数が足りない場合、マッチメイキングが完了するまでに時間がかかることがあります。制限時間内にマッチメイキングが完了しないと、マッチングはキャンセルされます。

以下のように、サーバーにユーザーマッチメイキングをリクエストできます。

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const matchUserStartResult = await matchUserStart(roomType, matchingGroup, payload);
```

プールに正常に登録されたかどうかは、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (matchUserStartResult.resultCode === ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS) {
    console.log("Match user start success.");
} else {
    console.log("Match user start fail.");
}
```
プールへの登録に失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                                   | 値  | 説明                               |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | パケットパースエラー                    |
| `TIMEOUT`                                    | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                               | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`                           | 2     | サーバーに登録されていない プロトコル     |
| `MATCH_USER_START_SUCCESS`                   | 0     | 成功                              |
| `MATCH_USER_START_FAIL_CONTENT` | 1101 | 失敗: コンテンツで拒否されました |
| `MATCH_USER_START_FAIL_ALREADY_JOINED_ROOM` | 1102 | 失敗: 既にルームに入室済みです |

プールへの登録が正常に完了すると、サーバーでユーザーマッチングが進行し、マッチングに成功すると、ユーザーに事前に登録されたコールバックが呼び出されます。ユーザーマッチングプールへの登録をリクエストする前に、以下のようにコールバックを登録します。

```typescript
user.onMatchUserDone = (user, resultCode, matchResult) => {
    console.log(`Match user done.`, resultCode);

    console.log(`Is request canceled?`, matchResult.isCancel);
    console.log(`Matched room id: `, matchResult.roomId);
    console.log(`Matched room name: `, matchResult.roomName);
    console.log(`Is room created for match?`, matchResult.created);
};
```

プールへの登録前後で、プールへの登録リクエストが送信され、処理されている状態かを確認できます。

```typescript
console.log(`Is in progress of match making?`, user.isUserMatchInPrgress);
```

### ユーザーマッチメイキングプールから削除

ユーザーマッチメイキングをリクエストしたものの、マッチメイキングがまだ進行中の状態で、リクエストをキャンセルできます。

```typescript
const roomType: string;

const resultCode = await user.matchUserCancel(roomType);
console.log(`Match user cancel result: `, ResultCodeMatchUserCancel[resultCode]);
```

プールからの削除リクエストが成功したかどうかは、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (resultCode === ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS) {
    console.log("Match user cancel success.");
} else {
    console.log("Match user cancel fail.");
}
```

リクエストに失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                                   | 値  | 説明                               |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | パケットパースエラー                    |
| `TIMEOUT`                                    | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                               | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`                           | 2     | サーバーに登録されていない プロトコル     |
| `MATCH_USER_CANCEL_SUCCESS`                  | 0     | 成功                              |
| `MATCH_USER_CANCEL_FAIL` | 1201 | 失敗: コンテンツで拒否されました |
| `MATCH_USER_CANCEL_FAIL_ALREADY_JOINED_ROOM` | 1202 | 失敗: 既にマッチングが成立しています |
| `MATCH_USER_CANCEL_FAIL_NOT_IN_PROGRESS` | 1203 | 失敗: マッチングが進行中でない場合 |

### ルームマッチメイキング

ルームマッチメイキングは、条件に合うルームにユーザーを入室させる方式です。ルームマッチメイキングをリクエストした際に、条件に合うルームがあればそのルームにすぐに入室させ、条件に合うルームがなければ新しいルームを作成して入室させるか、リクエストを失敗として処理します。

入室するルームのタイプ、マッチングのための追加情報であるマッチンググループとマッチングユーザーカテゴリ、そしてその他のオプションと追加の伝達情報を引数として渡すことができます。

ルームマッチメイキングの動作完了時点で、Promiseを通じて成功したかどうかや、入室したルームのID、ユーザーカスタムの追加情報などを確認できます。

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

動作の成否は、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (matchRoomResult.resultCode === ResultCodeMatchRoom.MATCH_ROOM_SUCCESS) {
    console.log("Match room success");
} else {
    consoel.log("Match room fail");
}
```

マッチメイキングに失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                                   | 値  | 説明                               |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | パケットパースエラー                    |
| `TIMEOUT`                                    | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                               | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`                           | 2     | サーバーに登録されていない プロトコル     |
| `MATCH_ROOM_SUCCESS`                         | 0     | 成功                              |
| `MATCH_ROOM_FAIL_CONTENT` | 901 | 失敗: コンテンツで拒否されました |
| `MATCH_ROOM_FAIL_ROOM_DOES_NOT_EXIST` | 902 | 失敗: ルームが存在しません |
| `MATCH_ROOM_FAIL_ALREADY_JOINED_ROOM` | 903 | 失敗: 既にルームに入室済みです |
| `MATCH_ROOM_FAIL_LEAVE_ROOM` | 904 | 失敗: 既存のルームからの退室に失敗した場合 |
| `MATCH_ROOM_FAIL_IN_PROGRESS`               | 905   | 失敗: すでにマッチメイキングが進行中の場合 |
| `MATCH_ROOM_FAIL_MATCHED_ROOM_DOES_NOT_EXIST` | 906 | 失敗: 条件に合うルームを探している途中でルームが削除されました |
| `MATCH_ROOM_FAIL_CREATE_ROOM_ID` | 907 | 失敗: ルームIDの発行に失敗しました |
| `MATCH_ROOM_FAIL_CREATE_ROOM` | 908 | 失敗: ルームの作成に失敗しました |
| `MATCH_ROOM_FAIL_INVALID_ROOM_ID` | 909 | 失敗: 不正なルームIDです |
| `MATCH_ROOM_FAIL_INVALID_NODE_ID` | 910 | 失敗: 不正なノードIDです |
| `MATCH_ROOM_FAIL_INVALID_USER_ID` | 911 | 失敗: 不正なユーザーIDです |
| `MATCH_ROOM_FAIL_MATCHED_ROOM_NOT_FOUND` | 912 | 失敗: マッチングを行いましたが、ルームが見つかりませんでした |
| `MATCH_ROOM_FAIL_INVALID_MATCHING_USER_CATEGORY` | 913 | 失敗: 不正なマッチングユーザーカテゴリです |
| `MATCH_ROOM_FAIL_MATCHING_USER_CATEGORY_EMPTY` | 914 | 失敗: マッチングユーザーカテゴリのサイズが0の場合 |
| `MATCH_ROOM_FAIL_BASE_ROOM_MATCH_FORM_NULL` | 915 | 失敗: マッチング申込書がありません |
| `MATCH_ROOM_FAIL_BASE_ROOM_MATCH_INFO_NULL` | 916 | 失敗: マッチング情報がありません |

ルームマッチメイキングに成功した場合、レスポンスを通じてroomIdを含む情報を確認できます。

```typescript
if (matchRoomResult.resultCode === ResultCodeMatchRoom.MATCH_ROOM_SUCCESS) {
    console.log(`Match room is cancel: ${matchRoomResult.isCancel}`);
    console.log(`Matched room id: ${matchRoomResult.roomId}`);
    console.log(`Matched room name: ${matchRoomResult.roomName}`);
    consoel.log(`Is matched room created? : ${matchRoomResult.created}`);
}
```

### 指定した名前のルーム

指定した名前のルームに入室したり、パーティマッチング用のルームに入室したりできます。指定した名前のルームがない場合は、新しく作成して入室します。

```typescript
const roomType: stirng;
const roomName: string;
const isParty: boolean;
const payload: Payload;

const namedRoomResult = await user.namedDroom(roomType, roomName, isParty, payload);

console.log(`Named room result: ${ResultCodeNamedRoom[namedRoomResult.errorCode]}`);
```

動作の成否は、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (namedRoomResult.resultCode === ResultCodeNamedRoom.NAMED_ROOM_SUCCESS) {
    console.log("Named room success");
} else {
    console.log("Named room success");
}
```

動作に失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                                   | 値  | 説明                               |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | パケットパースエラー                    |
| `TIMEOUT`                                    | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                               | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`                           | 2     | サーバーに登録されていない プロトコル     |
| `NAMED_ROOM_SUCCESS`                         | 0     | 成功                              |
| `NAMED_ROOM_FAIL_CONTENT` | 1001 | 失敗: コンテンツで拒否されました |
| `NAMED_ROOM_FAIL_ROOM_DOES_NOT_EXIST` | 1002 | 失敗: ルームの作成に失敗したため、ルームが見つかりません |
| `NAMED_ROOM_FAIL_ALREADY_JOINED_ROOM` | 1003 | 失敗: 既にルームに入室済みです |
| `NAMED_ROOM_FAIL_INVALID_ROOM_NAME` | 1004 | 失敗: 不正なルーム名が入力されました |
| `NAMED_ROOM_FAIL_CREATE_ROOM` | 1005 | 失敗: ルームの作成に失敗しました |

リクエストに成功した場合、レスポンスを通じて情報を確認できます。

```typescript
if (namedRoomResult.resultCode === ResultCodeNamedRoom.NAMED_ROOM_SUCCESS) {
    console.log(`Named room id: ${namedRoomResult.roomId}`);
    console.log(`Named room name: ${namedRoomResult.roomName}`);
    console.log(`Is named room created? : ${namedRoomResult.created}`);
}
```

### パーティーマッチング

パーティマッチメイキングは、ユーザーマッチメイキングの特殊な形態で、2人以上のユーザーが1つのパーティとしてユーザープールに登録され、条件に合う他のユーザーを探して新しく作成したルームに一緒に入室するものです。パーティとしてまとまったユーザーは、常に同じルームに入室します。パーティと一緒にマッチングされるユーザーは、サーバーのマッチメーカーの実装によって、別のパーティであったり、個人であったりします。

パーティマッチメイキングを行うには、まずネームドルームに入室している必要があります。ネームドルームをリクエストする際にisPartyをtrueに設定すると、そのNamedRoomがパーティの役割を果たします。NamedRoomにパーティのユーザーをすべて集めてから、パーティマッチメイキングを開始できます。

```typescript
const roomType: string;
const matchingGroup: string;
const payload: Payload;

const matchStartResult = await matchPartyStart(roomType, mathchingGroup, payload);

console.log(`Party match start result: ${ResultCodeMatchPartyStart[matchStartResult.errorCode]}`);
```

正常にパーティーマッチメイキングが開始されたかどうかは、Promise結果値であるResultのresultCodeを通じて以下のように確認できます。

```typescript
if (matchStartResult.resultCode === ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS) {
    console.log("Match party start success");
} else {
    console.log("Match party start fail");
}
```

パーティマッチの開始に失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                                   | 値  | 説明                               |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | パケットパースエラー                    |
| `TIMEOUT`                                    | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                               | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`                           | 2     | サーバーに登録されていない プロトコル     |
| `MATCH_PARTY_START_SUCCESS`                  | 0     | 成功                              |
| `MATCH_PARTY_START_FAIL_CONTENT` | 1301 | 失敗: コンテンツで拒否されました |
| `MATCH_PARTY_START_FAIL_PARTY_MATCH_WEIRD` | 1302 | 失敗: パーティマッチングをリクエストした際、ルームがパーティマッチング用のルームでない場合 |

パーティマッチが正常に開始されると、サーバーでパーティマッチングが開始されます。このとき、同じパーティにいるユーザーが開始時に通知を受け取るには、以下のように処理関数を登録できます。

```typescript
user.onMatchPartyStart = (user, resultCode, payload) => {
    console.log(`User in party started match. result: ${ResultCodeMatchPartyStart[resultCode]}`);
}
```

パーティマッチングに成功すると、パーティ内のユーザーに事前に登録された処理関数が呼び出されます。パーティマッチを開始する前に、以下のようにコールバックを登録します。

```typescript
user.onMatchUserDone = (user, resultCode, matchResult) => {
    console.log(`Match user done.`, resultCode);

    console.log(`Is request canceled?`, matchResult.isCancel);
    console.log(`Matched room id: `, matchResult.roomId);
    console.log(`Matched room name: `, matchResult.roomName);
    console.log(`Is room created for match?`, matchResult.created);
};
```

パーティマッチングの前後で、以下のように状態を確認できます。

```typescript
console.log(`Is in progress of match making? ${user.isPartyMatchInProgress}`);
```

### パーティーマッチングキャンセル

パーティマッチメイキングがまだ進行中の状態であれば、リクエストをキャンセルできます。

```typescript
const roomType: string;

const matchCancelResult = await matchPartyCancel(roomType);

console.log(`Match party cancel result: ${ResultCodeMatchPartyCancel[matchCancelResult.errorCode]}`);
```

正常にキャンセルされたかどうかは、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
if (matchCancelResult.resultCode === ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS) {
    console.log("Match party cancel success.");
} else {
    consoel.log("Match party cancel fail.");
}
```

キャンセルに失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                                   | 値  | 説明                               |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | パケットパースエラー                    |
| `TIMEOUT`                                    | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                               | 1     | サーバーシステムエラー                  |
| `MATCH_PARTY_CANCEL_SUCCESS`                 | 0     | 成功                              |
| `MATCH_PARTY_CANCEL_FAIL_CONTENT` | 1401 | 失敗: コンテンツで拒否されました |
| `MATCH_PARTY_CANCEL_FAIL_PARTY_MATCH_WEIRD` | 1402 | 失敗: パーティマッチングをキャンセルした際、ルームがパーティマッチング用のルームでない場合 |

### パケット送信

ゲームサーバーにユーザーのパケットを送信できます。事前に登録されたプロトコルのみ送信可能である点にご注意ください。

プロトコルの登録やメッセージの作成については、ガイドを参照してください。

```typescript
const name = "John Doe";
const age = 20;
const job = "artist";
const message = new UserInfo({name, age, job});

user.sendUser(message);
```

### パケット送信後に応答パケット待機

ゲームサーバーにユーザーのパケットを送信した後、サーバーから応答があれば、それを受け取って処理できます。事前に登録されたプロトコルのみ送信可能である点にご注意ください。

プロトコルの登録やメッセージの作成については、ガイドを参照してください。

```typescript
const echoResult = await user.requestUser<EchoRes>(new EchoReq({ message: "Hello World!"}), EchoRes);

console.log(echoResult.message); // Hello World!
```

### チャンネル移動

ユーザーが属するチャネルから退出させ、指定したチャネルに移動させることができます。

```typescript
const channelId: string;
const payload: Payload;

const moveChannelResult = await user.moveChannel(channelId, payload);

console.log(`Move channel result: ${ResultCodeMoveChannel[moveChannelResult.errorCode]}`);
```

チャネル移動が正常に行われたかどうかは、Promiseの結果値であるResultのresultCodeを通じて、以下のように確認できます。

```typescript
iF (moveChannelResult.resultCode === ResultcodeMoveChannel.MOVE_CHANNEL_SUCCESS) {
    console.log("Move channel success.");
} else {
    consoel.log("Move channel fail.");
}
```

チャネル移動に失敗した場合、errorCodeを通じてその原因を知ることができます。以下は、サーバーから受け取る可能性のあるerrorCodeの種類です。

| コード名                                   | 値  | 説明                               |
|----------------------------------------------|-------|-------------------------------------|
| `PARSE_ERROR`                                | -2    | パケットパースエラー                    |
| `TIMEOUT`                                    | -1    | タイムアウト                         |
| `SYSTEM_ERROR`                               | 1     | サーバーシステムエラー                  |
| `INVALID_PROTOCOL`                           | 2     | サーバーに登録されていない プロトコル     |
| `MOVE_CHANNEL_SUCCESS`                       | 0     | 成功                              |
| `MOVE_CHANNEL_FAIL_CONTENT` | 1601 | 失敗: コンテンツで拒否されました |
| `MOVE_CHANNEL_FAIL_NODE_NOT_FOUND` | 1602 | 失敗: チャネルノードが見つかりません |
| `MOVE_CHANNEL_FAIL_ALREADY_JOINED_CHANNEL` | 1603 | 失敗: 既にリクエストしたチャネルにいます |
| `MOVE_CHANNEL_FAIL_ALREADY_JOINED_ROOM` | 1604 | 失敗: 既にルームに入室しているため、チャネル移動はできません |

チャネル移動に成功した場合、結果を以下のように確認できます。

```typescript
if (moveChannelResult.resultCode === ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS) {
    console.log(`Move channel forced: ${moveChannelResult.data.force}`);
    consoel.log(`Move channel id: ${moveChannelResult.data.channelId});
}
```

チャネル移動は、サーバーによって強制的に行われることもあります。その場合は、以下のようにコールバックを登録して処理します。

```typescript
user.onMoveChannel = (user, result) => {
    console.log(`Move channel forced: ${result.force}`);
    consoel.log(`Move channel id: ${result.channelId});

}
```

現在のユーザーのチャネルID情報を知りたい場合は、channelIdプロパティを参照してください。

```typescript
console.log(`Current channel id: ${user.channelId}`);
```

### サーバーからの通知

サーバーからの通知に対して、事前に処理関数を登録できます。より複雑な形式のデータ伝達を希望する場合は、カスタムプロトコルの登録を検討してみてください。

```typescript 
user.onNotice = (user, message) => {
    console.log(`Message from server: ${message});
}
```

### 接続解除

サーバーによって接続が解除されたり、その他の理由で接続が切れたりした場合の処理関数を事前に登録できます。

```typescript
user.onSessionClose = (user, resultCode, payload) => {
    console.log(`Session closed. ${ResultCodeSessionClosed[resultCode]}`);
}
```

| コード名                                   | 値  | 説明                                                                                                 |
|----------------------------------------------|-------|------------------------------------------------------------------------------------------------------|
| `SESSION_CLOSE_BASE_USER` | 2011 | サーバーでBaseUserの`closeConnection()`を呼び出し |
| `SESSION_CLOSE_ADMIN_KICK` | 2012 | 管理画面からの強制終了 |
| `SESSION_CLOSE_DUPLICATE_LOGIN` | 2032 | 重複接続による強制終了 |
| `SESSION_CLOSE_BY_NEW_CONNECTION` | 2040 | 同じアカウント情報で新しいログインリクエストがあった場合に、以前の接続を終了。ネットワークの瞬断など、再接続時に使用。問い合わせが必要です。 |
| `SESSION_CLOSE_DISCONNECT_ALARM_FROM_CLIENT` | 2041 | クライアントとの接続切断を検知。通常は発生せず、発生した場合はGameAnvil開発チームへの問い合わせが必要です。 |
| `SESSION_CLOSE_DISCONNECT_ALARM_NOT_FIND_SESSION` | 2042 | セッションが見つからない場合。通常は発生せず、発生した場合はGameAnvil開発チームへの問い合わせが必要です。 |

### 管理者による強制退場

サーバーの管理ツールによってサーバーから強制退出させられた場合に、実行する処理関数を事前に登録できます。

```typescript
user.onAdminKickout = (user, message) => {
    console.log(`Message from server: ${message}`);
}
```
