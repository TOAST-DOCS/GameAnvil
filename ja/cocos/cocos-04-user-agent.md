## Game > GameAnvil > CocosCreator開発ガイド > UserAgent

## UserAgent

UserAgentはGameAnvilサーバーのGameNodeと関連した作業を担当します。ログイン(Login())、ログアウト(Logout())及びルーム管理などの基本的な機能を提供し、直接定義したプロトコルに基づいてクライアントは自分のユーザーオブジェクトを通して他のオブジェクトとメッセージをやり取りして様々なコンテンツを実装できます。UserAgentを使うためには、Connector.CreateUserAgent() 関数を利用して新しいUserAgentを作成する必要があります。ServiceNameとSubIdで区分される複数のUserAgentを作成することができます。作成されたUserAgentはConnectorで内部的に管理され、Connector.GetUserAgent() 関数を使って再使用できます。

```typescript
/**
 * サービス名とサブIDに該当するユーザーエージェントを返す
 * @param serviceNameユーザーエージェントが使用するサービス名
 * @param subId サービスごとにユーザーエージェントを識別できる固有のID。サーバーの実装によって使用しない場合があります。
 * @returns 該当するユーザーエージェント、ない場合は null
 */
let userAgent = connector.GetUserAgent(serviceName, subId);
if(userAgent == null){
    /**
     * ユーザーエージェントの作成
     * @param serviceName ユーザーエージェントが使用するサービス名
     * @param subId サービス別ユーザーエージェントを識別できる固有のID。サーバーの実装によって使用しない場合があります。
     * @returns作成されたユーザーエージェント
     */
    userAgent = connector.CreateUserAgent(serviceName, subId);
}

```

GameAnvilサーバーは複数のサービスを同時に運営することができ、一つのUserAgentは一つのサービスにログインしてお互いに独立して動作します。 つまり、複数のUserAgentを作成して違うサービスにログインして同時に使用できます。SubIdを変えれば、同じサービスに複数のUserAgentを同時にログインして使うことも可能です。

### ログイン/ログアウト

ログインはクライアントがサーバーに接続した後、GameNodeに自分のユーザーオブジェクトを作る過程と定義できます。ログアウトはログインの反対概念で、GameNode上から自分のユーザーオブジェクトを削除するプロセスです。

ログイン時、どのUserTypeでどのチャンネルにログインするかを入力する必要があります。 追加情報が必要な場合は、Payloadに含めることができます。

```typescript
/**
 * サーバーにログイン
 * @param userType ログインに使うユーザータイプ
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使わない場合があります。
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
    } else {
        // 失敗
    }
})

/**
 * 現在のサービスからログアウト
 * @param callbackログアウト結果を受け取って処理するコールバック
 */
userAgent.Logout((agent: UserAgent, resultCode: ResultCodeLogout, payload: Payload)=>{
    /**
     * Logout()の結果
     * @param user Logout()をリクエストしたユーザーエージェント
     * @param resultCode  Logout()結果コード
     * @param force 強制ログアウトを行うかどうか
     * @param payload サーバーから受け取った追加情報
     */
    if (ResultCodeLogout.LOGOUT_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

### ルーム作成、入室、退室

2人以上のユーザーはルームを通じて同期されたメッセージの流れを作ることができます。 つまり、ユーザーのリクエストはルーム内ですべて順番が保証されます。もちろん、1人のユーザーのためのルーム作成もコンテンツによって意味を持つことができます。ルームをどのように使うかはあくまでエンジンユーザー次第です。

CreateRoom()を呼び出してルームを作成し、そのルームに入ります。

```typescript
/**
 * 任意のルームを作成し、そのルームに入室
 * @param roomType 作成するルームのルームタイプ
 * @param roomName 作成するルームの名前
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param callback ルームの作成結果を受け取って処理するコールバック。
 */
userAgent.CreateRoom(roomType, roomName, payload, (agent: UserAgent, resultCode: ResultCodeCreateRoom, roomId: number, roomName: string, payload: Payload)=>{
    /**
     * CreateRoom()の結果
     * @param user CreateRoom()をリクエストしたユーザーエージェント
     * @param resultCode CreateRoom()結果コード
     * @param roomId 作成されたルームのルームID
     * @param roomName 作成されたルームの名前
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    if (ResultCodeCreateRoom.CREATE_ROOM_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

JoinRoom() を呼び出して作成済みのルームに入ります。

``` typescript
/**
 * 指定したルームIDのルームに入室
 * 
 * 指定したルームIDのルームがない場合は失敗
 * @param roomId 入室しようとするルームID
 * @param roomType 入室するルームのルームタイプ
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param callback ルームの入室結果を受け取って処理するコールバック
 */
userAgent.JoinRoom(roomId, roomType, payload, (agent: UserAgent, resultCode: ResultCodeJoinRoom, roomId: number, roomName: string, payload: Payload)=>{
    /**
     * JoinRoom()の結果
     * @param user JoinRoom()をリクエストしたユーザーエージェント
     * @param resultCode JoinRoom()結果コード
     * @param roomId 入室したルームのルームID
     * @param roomName 入室したルームの名前
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    if (ResultCodeJoinRoom.JOIN_ROOM_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

LeaveRoom()を呼び出して入室したルームから退室できます。

``` typescript
/**
 * 現在のルームから出る
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param callback ルーム退出リクエスト結果を受け取って処理するコールバック
 */
userAgent.LeaveRoom(payload, (agent: UserAgent, resultCode: ResultCodeLeaveRoom, roomId: number, payload: Payload)=>{
    /**
     * LeaveRoom()の結果
     * @param user LeaveRoom()をリクエストしたユーザーエージェント 
     * @param resultCode LeaveRoom()結果コード
     * @param force 強制退出かどうか
     * @param roomId 退出したルームのルームID
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    if (ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

NamedRoom()を呼び出して指定した名前のルームに入ることができます。指定した名前のルームがない場合は、ルームを作成した後、そのルームに入室します。

```typescript
/**
 * 指定した名前のルームに入室
 * 
 * 指定した名前のルームがない場合、指定した名前のルームを作成してそのルームに入る
 * @param roomName 入室したいルームの名前
 * @param roomType 入室するルームのルームタイプ
 * @param isParty true - パーティーマッチングを目的に作られたルーム。false - 一般的なルーム
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param callback ネームドルームの処理結果を受け取って処理するコールバック
 */
userAgent.NamedRoom(roomName, roomType, isParty, payload, (agent: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payLoad: Payload)=>{
    /**
     * NamedRoom()の結果
     * @param user NamedRoom()をリクエストしたユーザーエージェント 
     * @param resultCode NamedRoom()結果コード
     * @param roomId 入室したルームのルームID
     * @param roomName 入室したルームの名前
     * @param created 入室したルームを作成したかどうか(ルーム長かどうか)
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    if (ResultCodeNamedRoom.NAMED_ROOM_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```



### マッチメイキング

GameAnvilは二つのマッチメイキングを提供しています。それぞれルーム単位のマッチングを行うルームマッチメイキングとユーザー単位のマッチングを行うユーザーマッチメイキングです。

#### ルームマッチメイキング

ルームマッチメイキングは条件に合うルームにユーザーを入室させてくれる方式です。ルームマッチメイキングのリクエスト時に条件に合うルームがあればそのルームに直接入室させて、条件に合うルームがなければ新しいルームを作成して入室させる方式です。

MatchRoom()を呼び出してルームマッチメイキングをリクエストできます。条件に合うルームがない場合、任意のルームを生成してそのルームに入室することもできます。

```typescript
/**
 * ルームマッチをリクエスト
 * 
 * ルームがない場合、任意のルームを生成してそのルームに入場することもできます。
 * @param isCreateRoomIfNotJoinRoom true - 入室するルームがない場合、ランダムなルームを生成して入場。false - 入室するルームがない場合、失敗
 * @param isMoveRoomIfJoinedRoom true - ルームに入室した状態でリクエストした場合、他のルームに移動。false - ルームに入室した状態でリクエストした場合、失敗。
 * @param roomType 入室するルームのルームタイプ
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param leaveRoomPayload 入室したルームを出る時にサーバーに渡す追加情報。サーバーの実装によって使用しない場合があります。
 * @param callback ルームマッチの結果を受け取って処理するコールバック。
 */
userAgent.MatchRoom(isCreateRoomIfNotJoinRoom, isMoveRoomIfJoinedRoom, roomType, payload, leaveRoomPayload, (agent: UserAgent, resultCode: ResultCodeMatchRoom, roomId: number, roomName: string, created: boolean, payload: Payload) => {
    /**
     * MatchRoom()の結果
     * @param user MatchRoom()をリクエストしたユーザーエージェント 
     * @param resultCode MatchRoom()結果コード
     * @param roomId マッチされたルームのルームID
     * @param roomName マッチされたルームの名前
     * @param created マッチされたルームを作成したかどうか(ルーム長かどうか)
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    if (ResultCodeMatchRoom.MATCH_ROOM_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

#### ユーザーマッチメイキング

ユーザーマッチメイキングは、ユーザープールを作成し、その中で条件に合うユーザーを探し、新しく作成したルームに入室させる方式です。ユーザープールに条件に合うユーザーの数が足りない場合、マッチメイキングが完了するまで時間がかかることがあり、時間内にマッチメイキングが完了しないとタイムアウトしてマッチングがキャンセルされることがあります。

MatchUserStart()を呼び出して、ユーザーのマッチメイキングを開始できます。すでにルームに入室している場合など、サーバーの条件によってリクエストが失敗することがあります。

```typescript
/**
 * ユーザーマッチをリクエスト
 * 
 * 既にルームに入室している場合など、サーバーの条件によってリクエストが失敗することがあります。
 * 
 * マッチングが成功した場合、OnMatchUserDone()で通知を受けることができます。
 * @param roomType マッチをリクエストするルームタイプ
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param callback ユーザーマッチリクエストの結果を受け取って処理するコールバック
 */
userAgent.MatchUserStart(roomType, payload, (agent: UserAgent, resultCode: ResultCodeMatchUserStart, payLoad: Payload)=>{
    /**
     * MatchUserStart()の結果
     * @param user MatchUserStart()をリクエストしたユーザーエージェント 
     * @param resultCode MatchUserStart()結果コード
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    if (ResultCodeMatchUserStart.MATCH_USER_START_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

マッチングが成功した場合、IUserListener.OnMatchUserDoneで通知を受けることができ、時間内にマッチングが成功しなかった場合、IUserListener.OnMatchUserTimeoutで通知を受けることができます。

```typescript
class MatchUserListener implements IUserListener {
    /**
     * MatchUserStart(), MatchPartyStart()のマッチング結果
     * @param userマッチングをリクエストしたユーザーエージェント 
     * @param resultCodeマッチング結果コード
     * @param created ルームの作成有無。 true の場合、マッチングを要求した UserAgent がルームを作成したことを意味します。ルームを決定する用途などに使用できます。
     * @param roomId マッチングされたルームのルームID。
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnMatchUserDone(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void {
        if (ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS == resultCode) {
            // 成功
        } else {
            // 失敗
        }
    }

    /**
     * MatchUserStart(), MatchPartyStart()のマッチングのタイムアウト時に発生
     * @param userマッチングをリクエストしたユーザーエージェント 
     */
    OnMatchUserTimeout(user: UserAgent): void {
        // タイムアウト
    }
}
userAgent.AddListener(new MatchUserListener());
```

MatchUserCancel()を呼び出してユーザーマッチメイキングをキャンセルできます。マッチリクエスト中でない場合、すでにマッチメイキングが成功した場合やTimeoutが発生した場合は失敗することがあります。

```typescript
/**
 * ユーザーマッチリクエストをキャンセル
 * 
 * マッチリクエスト中でない場合や、すでにマッチングが成功した場合、またはタイムアウトが発生した場合は失敗
 * @param roomType マッチをリクエストしたルームタイプ
 * @param callbackユーザーマッチキャンセルリクエスト結果を受け取って処理するコールバック
 */
userAgent.MatchUserCancel(roomType, (agent: UserAgent, resultCode: ResultCodeMatchUserCancel) => {
    /**
     * MatchUserCancel()の結果
     * @param user MatchUserCancel()をリクエストしたユーザーエージェント 
     * @param resultCode MatchUserCancel()結果コード
     */
    if (ResultCodeMatchUserCancel.MATCH_USER_CANCEL_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

#### パーティーマッチメイキング

パーティーマッチメイキングは、ユーザーマッチメイキングの特殊な形で、2人以上のユーザーが一つのグループとしてユーザープールに登録され、条件が合う他のユーザーを探し、新しく作成したルームに入室させる方式です。 グループ化されたユーザーは常に同じルームに入室することができ、一緒に入室するユーザーは、サーバーのマッチメーカーの実装によってグループである場合もあれば、個人である場合もあります。

パーティーマッチメイキングをするためには、まずNamedRoom()を呼び出して、パーティーを組むユーザーが1つのルームに集まる必要があります。

```typescript
/**
 * 指定した名前のルームに入室
 * 
 * 指定した名前のルームがない場合、指定した名前のルームを作成してそのルームに入る
 * @param roomName 入室したいルームの名前
 * @param roomType 入室するルームのルームタイプ
 * @param isParty true - パーティーマッチングを目的に作られたルーム。false - 一般的なルーム
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param callback ネームドルームの処理結果を受け取って処理するコールバック
 */
let isParty = true;
userAgent.NamedRoom(roomName, roomType, isParty, payload, (agent: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payLoad: Payload) => {
    /**
     * NamedRoom()の結果
     * @param user NamedRoom()をリクエストしたユーザーエージェント 
     * @param resultCode NamedRoom()結果コード
     * @param roomId 入室したルームのルームID
     * @param roomName 入室したルームの名前
     * @param created 入室したルームを作成したかどうか(ルーム長かどうか)
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    if (ResultCodeNamedRoom.NAMED_ROOM_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

MatchPartyStart()を呼び出してパーティマッチメイキングを開始できます。すでにルームに入っている場合など、サーバーの条件によってリクエストが失敗することがあります。 

```typescript
/**
 * パーティーマッチをリクエスト
 * @param roomType マッチをリクエストするルームタイプ
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param callbackパーティマッチリクエスト結果を受け取って 処理するコールバック
 */
userAgent.MatchPartyStart(roomType, payload, (agent: UserAgent, resultCode: ResultCodeMatchPartyStart, payload: Payload) => {
    /**
     * MatchPartyStart()の結果
     * @param user MatchPartyStart()をリクエストしたユーザーエージェント 
     * @param resultCode MatchPartyStart()結果コード
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります。
     */
    if (ResultCodeMatchPartyStart.MATCH_PARTY_START_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

マッチングが成功した場合、ユーザーマッチメイキングで使ったIUserListener.OnMatchUserDoneを使って通知を受けることができ、時間内にマッチングが成功しなかった場合、IUserListener.OnMatchUserTimeoutを通じて通知を受けることができます。

```typescript
class MatchPartyListener implements IUserListener {
    /**
     * MatchUserStart(), MatchPartyStart()のマッチング結果
     * @param userマッチングをリクエストしたユーザーエージェント 
     * @param resultCodeマッチング結果コード
     * @param created ルームの作成有無。 true の場合、マッチングを要求した UserAgent がルームを作成したことを意味します。ルームを決定する用途などに使用できます。
     * @param roomId マッチングされたルームのルームID。
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnMatchUserDone(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void {
        if (ResultCodeMatchUserDone.MATCH_USER_DONE_SUCCESS == resultCode) {
            // 成功
        } else {
            // 失敗
        }
    }

    /**
     * MatchUserStart(), MatchPartyStart()のマッチングのタイムアウト時に発生
     * @param userマッチングをリクエストしたユーザーエージェント 
     */
    OnMatchUserTimeout(user: UserAgent): void {
        // タイムアウト
    }
}
userAgent.AddListener(new MatchPartyListener());
```

MatchPartyCancel()を呼び出してユーザーマッチメイキングをキャンセルできます。マッチリクエスト中でない場合、すでにマッチメイキングが成功している場合や、Timeoutが発生した場合は失敗することがあります。

```typescript
/**
 * パーティーマッチのリクエストをキャンセル
 * @param roomType マッチをリクエストしたルームタイプ
 * @param callback パーティーマッチのキャンセルリクエストの結果を受け取って処理するコールバック
 */
userAgent.MatchPartyCancel(roomType, (agent: UserAgent, resultCode: ResultCodeMatchPartyCancel) => {
    /**
     * MatchPartyCancel()の結果
     * @param user MatchPartyCancel()をリクエストしたユーザーエージェント 
     * @param resultCode MatchPartyCancel()の結果コード
     */
    if (ResultCodeMatchPartyCancel.MATCH_PARTY_CANCEL_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

#### チャンネル移動

場合によってはマッチメイキングの結果、チャンネル移動が発生することがあります。チャンネル移動が行われた場合、IUserListener.OnMoveChannelを通じて通知を受けることができます。

```typescript
class MoveChannelListener implements IUserListener {
    /**
     * MoveChannel() の結果またはサーバーで強制的に行ったチャンネル移動を通知
     * @param user チャンネルを移動したユーザーエージェント
     * @param resultCode チャンネル移動結果コード
     * @param force サーバーから強制的にチャンネルを移動したかどうか。
     * @param channelId 移動したチャンネルの ID
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnMoveChannel?(user: UserAgent, resultCode: ResultCodeMoveChannel, force: boolean, channelId: string, payload: Payload): void {
        // チャンネル移動
    }
}
userAgent.AddListener(new MoveChannelListener());
```



### チャンネル情報

GameAnvilは設定で自由にチャンネルを構成できます。このようなチャンネル構成はサーバーとクライアント間であらかじめ約束して固定された形で使うこともできますが、状況によって自由に変更して使うこともできます。UserAgentではこのように変更されたチャンネル情報を取得したり、チャンネルを移動できるようにいくつかの関数を提供します。

GetChannelCountInfo()はチャンネルのカウント情報(ユーザーとルームの数)をリクエストして取得できます。

```typescript
/**
 * 接続中のチャンネルのユーザーとルーム数をリクエスト
 * 
 * サーバーでサポートする場合、使用可能
 * @param callbackチャンネルのユーザーとルーム数のリクエスト結果を受け取って 処理するコールバック
 */
userAgent.GetChannelCountInfo((agent: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo) => {
    /**
     * GetChannelCountInfo()の結果
     * @param user GetChannelCountInfo()をリクエストしたユーザーエージェント
     * @param resultCode GetChannelCountInfo()の結果コード
     * @param channelCountInfoチャンネルのユーザーとルーム数
     */
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});

/**
 * 特定チャンネルのユーザーとルーム数をリクエスト
 * 
 * サーバーでサポートする場合、使用可能
 * @param serviceNameチャンネル情報をリクエストするサービス名
 * @param channelIdチャンネル情報をリクエストするチャンネルID
 * @param callbackチャンネルのユーザーとルーム数のリクエスト結果を受け取って 処理するコールバック
 */
userAgent.GetChannelCountInfo(serviceName, channelId, (agent: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo) => {
    /**
     * GetChannelCountInfo()の結果
     * @param connection GetChannelCountInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetChannelCountInfo()の結果コード
     * @param channelCountInfoチャンネルのユーザーとルーム数
     */
    if (ResultCodeChannelCountInfo.CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

GetChannelInfo()はチャンネルの情報(ユーザー定義)をリクエストして取得できます。

```typescript
/**
 * 接続中のチャンネル情報をリクエスト
 * 
 * サーバーでサポートする場合、使用可能
 * @param callbackチャンネル情報リクエスト結果を受け取って 処理するコールバック
 */
userAgent.GetChannelInfo((agent: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload) => {
    /**
     * GetChannelInfo()の結果
     * @param user GetChannelInfo()をリクエストしたユーザーエージェント
     * @param resultCode GetChannelInfo()の結果コード
     * @param channelInfoチャンネル情報
     */
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});

/**
 * 特定チャンネルの情報をリクエスト
 * サーバーでサポートする場合、使用可能
 * @param serviceNameチャンネル情報をリクエストするサービス名
 * @param channelIdチャンネル情報をリクエストするチャンネルID
 * @param callbackチャンネル情報リクエスト結果を受け取って 処理するコールバック
 */
userAgent.GetChannelInfo(serviceName, channelId, (agent: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload) => {
    /**
     * GetChannelInfo()の結果
     * @param connection GetChannelInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetChannelInfo()の結果コード
     * @param channelInfoチャンネル情報
     */
    if (ResultCodeChannelInfo.CHANNEL_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

GetAllChannelCountInfo()はサービスの全てのチャンネルに対するカウント情報(ユーザーとルームの数)をリクエストして取得できます。引数でサービス名とレスポンスを処理するコールバックを渡します。

```typescript
/**
 * 接続中のチャンネル情報をリクエスト
 * 
 * サーバーでサポートする場合、使用可能
 * @param callbackチャンネル情報リクエスト結果を受け取って 処理するコールバック
 */
userAgent.GetAllChannelCountInfo((agent: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo) => {
    /**
     * GetAllChannelCountInfo()の結果
     * @param connection GetAllChannelCountInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetAllChannelCountInfo()の結果コード
     * @param channelInfoList チャンネルのユーザーとルーム数
     */
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});

/**
 * 特定のサービスにある全てのチャンネルのユーザーとルームの数をリクエスト
 * 
 * サーバーでサポートする場合、使用可能
 * @param serviceNameチャンネル情報をリクエストするサービス名
 * @param callback すべてのチャンネルのユーザーとルームの数のリクエスト結果を受け取って処理するコールバック
 */
userAgent.GetAllChannelCountInfo(serviceName, (agent: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo) => {
    /**
     * GetAllChannelCountInfo()の結果
     * @param connection GetAllChannelCountInfo()をリクエストしたコネクションエージェント
     * @param resultCode GetAllChannelCountInfo()の結果コード
     * @param channelInfoList チャンネルのユーザーとルーム数
     */
    if (ResultCodeAllChannelCountInfo.ALL_CHANNEL_COUNT_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
});
```

GetAllChannelInfo()はサービスのすべてのチャンネルについての情報(ユーザー定義)をリクエストして取得できます。引数でサービス名とレスポンスを処理するコールバックを渡します。

```typescript
/**
 * 接続中のサービスのすべてのチャンネル情報リクエスト
 * 
 * サーバーでサポートする場合、使用可能
 * @param callbackすべてのチャンネル情報リクエスト結果を受け取って 処理するコールバック
 */
userAgent.GetAllChannelInfo((agent: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo) => {
    /**
     * GetAllChannelInfo()の結果
     * @param user GetAllChannelInfo()をリクエストしたユーザーエージェント
     * @param resultCode GetAllChannelInfo()の結果コード
     * @param allChannelInfoチャンネル情報
     */
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})

/**
 * 特定サービスのすべてのチャンネル情報リクエスト
 * 
 * サーバーでサポートする場合、使用可能
 * @param serviceNameチャンネル情報をリクエストするサービス名
 * @param callbackすべてのチャンネル情報リクエスト結果を受け取って 処理するコールバック
 */
userAgent.GetAllChannelInfo(serviceName, (agent: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo) => {
    /**
     * GetAllChannelInfo()の結果
     * @param user GetAllChannelInfo()をリクエストしたユーザーエージェント
     * @param resultCode GetAllChannelInfo()の結果コード
     * @param allChannelInfoチャンネル情報
     */
    if (ResultCodeAllChannelInfo.ALL_CHANNEL_INFO_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```

MoveChannel()を呼び出してサービス内の他のチャンネルに移動できます。 

```typescript
/**
 * 指定したチャンネルに移動
 * @param channelId 移動するチャンネルID
 * @param payload サーバーに渡す追加情報。サーバーの実装によって使用しない場合があります
 * @param callback チャンネル移動結果を受け取って 処理するコールバック
 */
userAgent.MoveChannel(channelId, payload, (agent: UserAgent, resultCode: ResultCodeMoveChannel, channelId: string, payload: Payload) => {
    /**
     * MoveChannel() の結果またはサーバーで強制的に行ったチャンネル移動を通知
     * @param user チャンネルを移動したユーザーエージェント
     * @param resultCode チャンネル移動結果コード
     * @param force サーバーから強制的にチャンネルを移動したかどうか。
     * @param channelId 移動したチャンネルの ID
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    if (ResultCodeMoveChannel.MOVE_CHANNEL_SUCCESS == resultCode) {
        // 成功
    } else {
        // 失敗
    }
})
```



### Listener

IUserListenerはUserAgentの全ての動作の結果や通知を定義したインターフェイスです。 このインターフェイスを実装したリスナーをUserAgent.AddUserListener()で登録すると、登録したリスナーでレスポンスを受けることができます。IUserListenerの関数は全てオプションなので、使う関数だけ実装して使うこともできます。

```typescript
class UserListener implements IUserListener {
    /**
     * Login()の結果
     * @param user Login()をリクエストしたユーザーエージェント
     * @param resultCode Login()結果コード 
     * @param loginInfo Login()情報
     */
    OnLogin?(user: UserAgent, resultCode: ResultCodeLogin, loginInfo: LoginInfo): void { }
    /**
     * Logout()の結果
     * @param user Logout()をリクエストしたユーザーエージェント
     * @param resultCode  Logout()結果コード
     * @param force 強制ログアウトを行うかどうか
     * @param payload サーバーから受け取った追加情報
     */
    OnLogout?(user: UserAgent, resultCode: ResultCodeLogout, force: boolean, payload: Payload): void { }
    /**
     * MatchRoom()の結果
     * @param user MatchRoom()をリクエストしたユーザーエージェント 
     * @param resultCode MatchRoom()結果コード
     * @param roomId マッチされたルームのルームID
     * @param roomName マッチされたルームの名前
     * @param created マッチされたルームを作成したかどうか(ルーム長かどうか)
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnMatchRoom?(user: UserAgent, resultCode: ResultCodeMatchRoom, roomId: number, roomName: string, created: boolean, payload: Payload): void { }
    /**
     * CreateRoom()の結果
     * @param user CreateRoom()をリクエストしたユーザーエージェント
     * @param resultCode CreateRoom()結果コード
     * @param roomId 作成されたルームのルームID
     * @param roomName 作成されたルームの名前
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnCreateRoom?(user: UserAgent, resultCode: ResultCodeCreateRoom, roomId: number, roomName: string, payload: Payload): void { }
    /**
     * JoinRoom()の結果
     * @param user JoinRoom()をリクエストしたユーザーエージェント
     * @param resultCode JoinRoom()結果コード
     * @param roomId 入室したルームのルームID
     * @param roomName 入室したルームの名前
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnJoinRoom?(user: UserAgent, resultCode: ResultCodeJoinRoom, roomId: number, roomName: string, payload: Payload): void { }
    /**
     * NamedRoom()の結果
     * @param user NamedRoom()をリクエストしたユーザーエージェント 
     * @param resultCode NamedRoom()結果コード
     * @param roomId 入室したルームのルームID
     * @param roomName 入室したルームの名前
     * @param created 入室したルームを作成したかどうか(ルーム長かどうか)
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnNamedRoom?(user: UserAgent, resultCode: ResultCodeNamedRoom, roomId: number, roomName: string, created: boolean, payload: Payload): void { }
    /**
     * LeaveRoom()の結果
     * @param user LeaveRoom()をリクエストしたユーザーエージェント 
     * @param resultCode LeaveRoom()結果コード
     * @param force 強制退出かどうか
     * @param roomId 退出したルームのルームID
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnLeaveRoom?(user: UserAgent, resultCode: ResultCodeLeaveRoom, force: boolean, roomId: number, payload: Payload): void { }
    /**
     * MatchUserStart()の結果
     * @param user MatchUserStart()をリクエストしたユーザーエージェント 
     * @param resultCode MatchUserStart()結果コード
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnMatchUserStart?(user: UserAgent, resultCode: ResultCodeMatchUserStart, payload: Payload): void { }
    /**
     * MatchUserCancel()の結果
     * @param user MatchUserCancel()をリクエストしたユーザーエージェント 
     * @param resultCode MatchUserCancel()結果コード
     */
    OnMatchUserCancel?(user: UserAgent, resultCode: ResultCodeMatchUserCancel): void { }
    /**
     * MatchUserStart(), MatchPartyStart()のマッチング結果
     * @param userマッチングをリクエストしたユーザーエージェント 
     * @param resultCodeマッチング結果コード
     * @param created ルームの作成有無。 true の場合、マッチングを要求した UserAgent がルームを作成したことを意味します。ルームを決定する用途などに使用できます。
     * @param roomId マッチングされたルームのルームID。
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnMatchUserDone?(user: UserAgent, resultCode: ResultCodeMatchUserDone, created: boolean, roomId: number, payload: Payload): void { }
    /**
     * MatchUserStart(), MatchPartyStart()のマッチングのタイムアウト時に発生
     * @param userマッチングをリクエストしたユーザーエージェント 
     */
    OnMatchUserTimeout?(user: UserAgent): void { }
    /**
     * MatchPartyStart()の結果
     * @param user MatchPartyStart()をリクエストしたユーザーエージェント 
     * @param resultCode MatchPartyStart()結果コード
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります。
     */
    OnMatchPartyStart?(user: UserAgent, resultCode: ResultCodeMatchPartyStart, payload: Payload): void { }
    /**
     * MatchPartyCancel()の結果
     * @param user MatchPartyCancel()をリクエストしたユーザーエージェント 
     * @param resultCode MatchPartyCancel()の結果コード
     */
    OnMatchPartyCancel?(user: UserAgent, resultCode: ResultCodeMatchPartyCancel): void { }
    /**
     * MoveChannel() の結果またはサーバーで強制的に行ったチャンネル移動を通知
     * @param user チャンネルを移動したユーザーエージェント
     * @param resultCode チャンネル移動結果コード
     * @param force サーバーから強制的にチャンネルを移動したかどうか。
     * @param channelId 移動したチャンネルの ID
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
    OnMoveChannel?(user: UserAgent, resultCode: ResultCodeMoveChannel, force: boolean, channelId: string, payload: Payload): void { }
    /**
     * Snapshot()の結果
     * @param user Snapshot()をリクエストしたユーザーエージェント 
     * @param resultCode Snapshot()の結果コード
     * @param payload サーバーから受け取った追加情報。サーバーの実装によって使用しない場合があります
     */
     OnSnapshot?(user: UserAgent, resultCode: ResultCodeSnapshot, payload: Payload): void { }
    /**
     * GetChannelInfo()の結果
     * @param user GetChannelInfo()をリクエストしたユーザーエージェント
     * @param resultCode GetChannelInfo()の結果コード
     * @param channelInfoチャンネル情報
     */
    OnChannelInfo?(user: UserAgent, resultCode: ResultCodeChannelInfo, channelInfo: Payload): void { }
    /**
     * GetChannelCountInfo()の結果
     * @param user GetChannelCountInfo()をリクエストしたユーザーエージェント
     * @param resultCode GetChannelCountInfo()の結果コード
     * @param channelCountInfoチャンネルのユーザーとルーム数
     */
    OnChannelCountInfo?(user: UserAgent, resultCode: ResultCodeChannelCountInfo, channelCountInfo: ChannelCountInfo): void { }
    /**
     * GetAllChannelInfo()の結果
     * @param user GetAllChannelInfo()をリクエストしたユーザーエージェント
     * @param resultCode GetAllChannelInfo()の結果コード
     * @param allChannelInfoチャンネル情報
     */
    OnAllChannelInfo?(user: UserAgent, resultCode: ResultCodeAllChannelInfo, allChannelInfo: AllChannelInfo): void { }
    /**
     * GetAllChannelCountInfo()の結果
     * @param user GetAllChannelCountInfo()をリクエストしたユーザーエージェント
     * @param resultCode GetAllChannelCountInfo()の結果コード
     * @param channelInfoList チャンネルのユーザーとルーム数
     */
    OnAllChannelCountInfo?(user: UserAgent, resultCode: ResultCodeAllChannelCountInfo, allChannelCountInfo: AllChannelCountInfo): void { }
    /**
     * エラー発生
     * @param userエラーが発生したユーザーエージェント 
     * @param errorCodeエラーコード
     * @param msgNameエラーが発生した機能またはメッセージ
     */
    OnErrorCommand?(user: UserAgent, errorCode: ErrorCode, msgName: string): void { }
    /**
     * 告知通知
     * @param user告知を受け取ったユーザーエージェント 
     * @param message告知メッセージ
     */
    OnNotice?(user: UserAgent, message: string): void { }
    /**
     * AdminがKickout()した場合に通知
     * @param user Kickout()されたユーザーエージェント 
     * @param message Adminが伝えたメッセージ
     */
    OnAdminKickoutNoti?(user: UserAgent, message: string): void { }
    /**
     * ユーザーリスナーでサーバーのセッションが閉じられた場合に通知。このアラームを受け取った場合、再度ログインして再起動する必要 がある
     * @param user セッションが閉じられたユーザーエージェント
     * @param resultCode セッションが閉じられた理由
     * @param paload 追加情報。サーバーの実装によって使用しない場合があります。
     */
    OnSessionCloseNoti?(user: UserAgent, resultCode: ResultCodeSessionClose, payload: Payload): void { }
}
userAgent.AddListener(new UserListener());
```
