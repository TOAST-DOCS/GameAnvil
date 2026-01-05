## Game > GameAnvil > CocosCreator 開発ガイド > 再接続

## 再接続

ゲーム途中、様々な理由でサーバーとの接続が切れることがあります。接続が切れた際、既存のプレイを継続できるように再接続機能をサポートします。使用するAPIは一般的な接続方法と同様に使用できます。結果を通じて再接続に関するより詳細な情報を受け取ります。

### 認証

再接続後に認証を進めると、結果値として以前プレイしていたユーザー情報が含まれて返ってきます。

```typescript
const deviceId, accountId, password;

const authResult = await connector.authentication(deviceId, accountId, password);
console.log(`Authentication Result : ${ResultCodeAuth[authResult.errorCode]}`);

const loginedUserInfoList = authResult.data.loginedUserInfoList;
loginedUserInfoList.forEach((userInfo: AlreadyLoginedUserInfo) => {
    const serviceName = userInfo.serviceName;
    const userId = userInfo.userId;
    const subId = userInfo.subId;
    const userType = userInfo.userType;
    const channelId = userInfo.channelId;
    console.log(`AlreadyLoginedUserInfo(serviceName=${serviceName}, userId=${userId}, subId=${subId}, userType=${userType}, channelId=${channelId})`);
});
```

### ログイン

認証結果として受け取ったユーザー情報を利用してログインを進めます。この時、userTypeやchannelIdなど、以前のユーザー情報と同じ値を利用してログインする必要があります。そうしないとログインに失敗する可能性があります。

```typescript
const userType: stirng;
const channelId: string;
const payload: Payload;

const loginResult = await user.login(userType, channelId, payload);
```

ログイン成功時、結果データでisReloginedがtrueであれば、再接続ログインに成功したことになります。LoginInfoに含まれる情報をもとに再接続します。この時最も重要なのが、再接続時間中にサーバーの変更されたデータをクライアントに同期することです。この同期に必要なデータをLoginInfoのPayloadに込めて処理できます。  

再接続ログインをすると、サーバーではIUser.onRelogin()コールバックが呼び出されます。この時、同期に必要なメッセージを定義してoutPayloadパラメータに追加すれば、LoginInfoのPayloadとして受け取り処理できます。 

もしisReloginedがfalseであれば、時間が経過して以前のユーザー情報が削除された後にログインされたことになります。この場合には初めてログインする手順を実行すればよいです。 
