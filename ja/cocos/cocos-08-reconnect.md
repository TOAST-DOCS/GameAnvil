## Game > GameAnvil > CocosCreator開発ガイド > 再接続

## 再接続

ゲーム中に様々な理由でサーバーとの接続が切断されることがあります。接続が切断された時、既存のプレイを継続できるように再接続機能をサポートします。使用するAPIは一般的な接続方法と同じですが、結果として受け取る情報に違いがあります。

### サーバー接続

ConnectionAgentのConnect関数を利用してサーバーに接続します。一般的な場合と同じです。

```typescript
// GameAnvilサーバーに接続
// Connect(ipAdress: string, callback?: (agent: ConnectionAgent, resultCode: ResultCodeConnect) => void): void;
//  ipAddress :接続するサーバーのアドレス。 
//  callback :結果を受け取って処理するコールバック
connectionAgent.Connect(ipAddress,(agent: ConnectionAgent, resultCode: ResultCodeConnect) => {
    //  agent : Connect()を呼び出した ConnectionAgentオブジェクト。
	  //  resultCode : Connect結果。
    if (ResultCodeConnect.CONNECT_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

### 認証

ConnectionAgentのAuthenticate関数を使って認証手続きを行います。入力値は一般的な場合と同じです。ただ、認証の結果として受け取る値のうちloginedUserInfoListに以前プレイしていたユーザー情報が含まれます。

```typescript
/**
 * GameAnvilサーバーに認証をリクエスト
 * 
 * 認証に成功すると、GameAnvilコネクタの他の機能を使用できます。
 * @param deviceId ユーザーのデバイスを識別できる固有のID。同じユーザーの重複接続をチェックするために使用
 * @param accountId ユーザーアカウントを識別できる固有のID。
 * @param password ユーザーアカウントのパスワード
 * @param payload サーバーに渡す追加情報
 * @param callback 結果を受け取って処理するコールバック
 */
connectionAgent.Authenticate(deviceId, accountId, password, payload
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
    /**
     * Authentication()の結果
     * @param connection Authentication()をリクエストしたコネクションエージェント
     * @param resultCode認証の結果コード
     * @param loginedUserInfoList サーバーに残っているログイン情報のリスト
     * @param message サーバーから受け取ったメッセージ、認証失敗の理由など
     * @param payload サーバーから受け取った追加情報
     */
    if (result == ResultCodeAuth.AUTH_SUCCESS) {
		// 成功
        for(let loginedUserInfo of loginedUserInfoList){
            // プレイユーザー情報
        }
    } else {
		// 失敗
    }
});
```



### ログイン

認証結果で受け取ったユーザー情報を使ってログインを行います。この時、userTypeやchannelIdなど以前のユーザー情報と同じ値を使ってログインする必要があります。 そうしないと、ログインが失敗する可能性があります。

```typescript
/**
 * サーバーにログイン
 * @param userType ログインに使うユーザータイプ
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります。
 * @param channelId ログインするチャンネルID
 * @param callback ログイン結果を受け取って処理するコールバック
 */
userAgent.Login(userType, payload, channelId, (agent: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo)=>{
    /**
     * Login()の結果
     * @param user Login()をリクエストしたユーザーエージェント
     * @param resultCode Login()結果コード 
     * @param loginInfo Login()情報
     */
    if (ResultCodeLogin.LOGIN_SUCCESS == resultCode) {
        // 成功
        if (loginInfo.isRelogined) {
            // 再接続
        } else {
            // 最初の接続
        }
    } else {
        // 失敗
    }
})
```

ログイン成功時、LoginInfoのisReloginedがtrueなら再接続ログインに成功したことになります。LoginInfoに含まれている情報を基に再接続処理を行います。この時、一番重要なことは再接続時間の間、サーバーの変更されたデータをクライアントに同期することです。 この同期に必要なデータをLoginInfoのPayloadに入れて処理できます。 

再接続ログインをすると、サーバーではBaseUser.onRelogin()コールバックが呼び出されます。この時、同期に必要なメッセージを定義してoutPayloadパラメータに追加すると、LoginInfoのPayloadで受け取って処理できます。

もし、isReloginedがfalseであれば、時間が過ぎて以前のユーザー情報が削除された後、ログインされたことになります。 この場合は、初めてログインする手順を実行します。
