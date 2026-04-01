## Game > GameAnvil > サーバー開発ガイド > チャネル

## チャネル(Channel)

![channel-sync2.png](https://static.toastoven.net/prod_gameanvil/images/channel-sync2.png)

チャネルは、単一のサーバー群を論理的に分割する方法の1つです。GameAnvilは、1つ以上のゲームノードを含む場合にチャネルを設定できます。基本的には、GameAnvilConfigを通じて、以下の例のようにゲームノードにチャネルを設定できます。この例では、4つのゲームノードに対し、それぞれ「初心者」「初心者」「上級者」「上級者」を設定します。

```json
"game": [
  {
    "serviceId": 1, // サーバー群でユニークである必要あり [0 ～ 99]
    "serviceName": "MyGame", // サーバー群でユニークである必要があります
    "nodeCnt": 4, // ノード数。nodeCntが0の場合Game Nodeを生成しない。マシンのコア数で設定することを推奨 [0 ～99]
    "channelIDs": [
      ["初心者"],
      ["初心者"],
      ["上級者"],
      ["上級者"]
    ], // ノードごとに付与するチャネルID。(ユニークでなくても可。""という文字列でチャネルを区別せずに重複使用も可能)
    "userTimeout": 5000       // 切断後のユーザーオブジェクト削除タイムアウト。[0 ～ 604800000(7日)]
  }
],
```

チャネルを使用したくない場合は、チャネルIDを全て以下のように""に設定します。この場合、全てのチャネル機能も使用できなくなります。つまり、以下の4つのゲームノードは、全てチャネルとは関係なく独立して動作します。

```json
"game": [
  {
    "serviceId": 1, // サーバー群でユニークである必要あり [0 ～ 99]
    "serviceName": "MyGame", // サーバー群でユニークである必要があります
    "nodeCnt": 4, // ノード数。nodeCntが0の場合Game Nodeを生成しない。マシンのコア数で設定することを推奨 [0 ～99]
    "channelIDs": [
      [""],
      [""],
      [""],
      [""]
    ], // ノードごとに付与するチャネルID。(ユニークでなくても可。""という文字列でチャネルを区別せずに重複使用も可能)
    "userTimeout": 5000       // 切断後のユーザーオブジェクト削除タイムアウト。[0 ～ 604800000(7日)]
  }
],
```
もし、全てのゲームノードを1つのチャネルで管理したい場合は、以下のように有効な文字列でチャネルIDを入力します。

```json
"game": [
  {
    "serviceId": 1,           // サーバー群で一意である必要あり [0 ～ 99]
    "serviceName": "MyGame",  // サーバー群でユニークである必要があります
    "nodeCnt": 4,             // ノード数。nodeCntが0の場合、Game Nodeを生成しません。マシンのコア数で設定することを推奨 [0 ～99]
    "channelIDs": [
      ["初心者"],
      ["初心者"],
      ["初心者"],
      ["初心者"]
    ],                        // ノードごとに付与するチャネルID。(ユニークでなくても可。""という文字列でチャネルを区別せずに重複使用も可能)
    "userTimeout": 5000       // 切断後のユーザーオブジェクト削除タイムアウト。[0 ～ 604800000(7日)]
  }
],
```

基本的には、1つのスレッドに1つのゲームノードが設定されますが、効率的なリソース活用のために、1つのスレッドに複数のゲームノードを以下のように設定することも可能です。ゲームノードの総数は、nodeCntで指定した4つではなく、実際にチャネル情報に設定した3つ(nodeCntの値は無視)になります。ゲームノードのスレッドは2つのグループに分かれ、2つになります。スレッド1にはゲームノード1(「初心者」)、ゲームノード2(「中級者」)が、スレッド2にはゲームノード3(「上級者」)が設定されます。
```json
"game": [
  {
    "serviceId": 1,           // サーバー群で一意である必要あり [0 ～ 99]
    "serviceName": "MyGame",  // サーバー群でユニークである必要があります
    "nodeCnt": 4,             // ノード数。nodeCntが0の場合、Game Nodeを生成しません。マシンのコア数で設定することを推奨 [0 ～99]
    "channelIDs": [
      ["初心者", "中級者"], // スレッド1
      ["上級者"]         // スレッド2
    ],                        // ノードごとに付与するチャネルID。(ユニークでなくても可。""という文字列でチャネルを区別せずに重複使用も可能)
    "userTimeout": 5000       // 切断後のユーザーオブジェクト削除タイムアウト。[0 ～ 604800000(7日)]
  }
],
```

このようなチャネルを通じて、以下の機能を使用できます。

* チャネル情報を管理できます。該当のチャネルに属するユーザーとルームの情報が利用可能になります。
* チャネルごとのユーザー数とルーム数を照会できます。
* チャネル単位でメッセージを送信できます。publishToChannel APIを使用すると、対象のチャネルに属する全てのゲームノードにメッセージを伝達します。

## チャネル情報管理

ユーザーは、チャネルで管理する情報を直接実装できます。これらの情報は、同じチャネル内で自動的に同期されます。

### チャネルユーザー情報

まず、チャネルでユーザー情報を管理するには、以下のようにユーザークラスを実装する際に、useChannelInfo設定を通じてチャネルユーザー情報管理を有効化する必要があります。
```java
@GameAnvilUser(
    gameServiceName = "MyGame", // ユーザーが所属するノード(上記のSampleGameNodeと同じサービス名)
    gameType = "BasicUser",     // ユーザーの固有タイプ、"BasicUser"というユーザータイプのユーザーをエンジンに登録
    useChannelInfo = true       // チャネル間の情報同期設定
)
public class SampleGameUser implements IUser {
    // ... 
}
```

そして、追加でIChannelUserInfoインターフェースを実装します。このとき、ユーザーがチャネルで管理したい情報を全て含めることができます。

```java
public class GameChannelUserInfo implements IChannelUserInfo, Comparable<GameChannelUserInfo> {

    private String userType = "";
    private int userId = 0;
    private String accountId = "";

    public GameChannelUserInfo(String userType, int userId, String accountId) {
        this.userType = userType;
        this.userId = userId;
        this.accountId = accountId;
    }

    public void setUserType(String userType) {
        this.userType = userType;
    }

    public void setUserId(int userId) {
        this.userId = userId;
    }

    public void setAccountId(String accountId) {
        this.accountId = accountId;
    }

    /**
     * ユーザーIDを返す
     *
     * @return ユーザーIDを返す
     */
    @Override
    public int getUserId() {
        return userId;
    }

    /**
     * アカウントIDを返す
     *
     * @return アカウントIDを返す
     */
    @Override
    public String getAccountId() {
        return accountId;
    }

    @Override
    public int compareTo(GameChannelUserInfo o) {
        if (o.userId != this.userId) {
            return -1;
        } else {
            return 0;
        }
    }
}
```

このように作成したチャネルユーザー情報は、以下のようにユーザーオブジェクトで追加または更新できます。このとき、userContext.updateChannelUserInfo() APIを使用します。もし、該当のユーザーオブジェクトがサーバーからログアウトすると、そのチャネルユーザー情報も自動的に削除されます。以下は、これに関する疑似コードです。

```java
public class SampleGameUser implements IUser {
    GameChannelUserInfo channelUserInfo = new GameChannelUserInfo();
  
	...

    // ユーザーがログインした際に、チャネルユーザー情報を追加します。
    onLogin(...) {
		...
        channelUserInfo.setUserId(getId())
        channelUserInfo.setAccountId(getAccountId());
        channelUserInfo.setLevel(getLevel());

        userContext.updateChannelUserInfo(channelUserInfo);
		...
    }

    // チャネルを移動する際に、移動先のチャネルに新しいチャネルユーザー情報を追加します。
    onMoveInChannel(...) {
		...
        channelUserInfo.setUserId(getId());
        channelUserInfo.setAccountId(getAccountId());
        channelUserInfo.setLevel(getLeve());

        userContext.updateChannelUserInfo(channelUserInfo);
		...
    }

    // ユーザーコンテンツで変更されたユーザー情報をいつでも更新できます。
    updateLevel(int level) {
        channelUserInfo.setLevel(getLevel());

        userContext.updateChannelUserInfo(channelUserInfo);
    }
}
```

参考までに、チャネルを移動する際には、移動元のチャネルから自動的に該当のチャネルユーザー情報が削除されるため、ユーザーは移動先のチャネルで新しく追加する情報のみを考慮すれば済みます。

### チャネルルーム情報

チャネルでルーム情報を管理するには、前述のゲームユーザーと同様に、ルームクラスを実装する際にuseChannelInfoをtrueに設定します。
```java
 @GameAnvilRoom(
    gameServiceName = "MyGame", // ルームが所属するノード(上記のSampleGameNodeと同じサービス名)
    gameType = "BasicRoom",     // ルームが固有タイプ、"BasicRoom"というタイプのルームをエンジンに登録
    useChannelInfo = true       // チャネル間の情報同期設定
)
public class SampleRoom implements IRoom<SampleUser> {
    // ...
}
```

そして、追加でIChannelRoomInfoインターフェースを実装します。このとき、ユーザーがチャネルで管理したいルーム関連の情報を全て含めることができます。

```java
public class GameChannelRoomInfo implements IChannelRoomInfo, Comparable<GameChannelRoomInfo> {

    public static final int MAX_ENTRY_USER = 4;
    private int roomId = 0;

    private long allUserMoney; // for room sorting comparator.


    /**
     * ルームIDを返す
     *
     * @return ルームIDを返す
     */
    @Override
    public int getRoomId() {
        return roomId;
    }

    /**
     * ルーム情報をコピー
     *
     * @return コピーされたルーム情報を返す
     * @throws CloneNotSupportedException コピーできない場合に発生
     */
    @Override
    public IChannelRoomInfo copy() throws CloneNotSupportedException {
        GameChannelRoomInfo channelRoomInfo = (GameChannelRoomInfo)super.clone();
        channelRoomInfo.testListInfo = new ArrayList<>(testListInfo);

        return channelRoomInfo;
    }

    @Override
    public int compareTo(GameChannelRoomInfo o) {
        return (int)(o.getAllUserMoney() - this.allUserMoney); // descending.
    }

}
```

このように作成したチャネルルーム情報は、以下のようにルームオブジェクトで追加または更新できます。このとき、roomContext.updateChannelRoomInfo() APIを使用します。もし、該当のルームオブジェクトがサーバーから削除されると、そのチャネルルーム情報も自動的に削除されます。以下は、これに関する疑似コードです。

```java
public class SampleGameRoom implements IRoom<SampleGameUser> {
    GameChannelRoomInfo channelRoomInfo = new GameChannelRoomInfo();
  
	...

    // ルーム作成時、updateChannelRoomInfo関数を介してチャネルルーム情報を追加します。
    onCreateRoom(...) {
		...
        channelRoomInfo.setRoomId(getId());
        channelRoomInfo.setRoomName(getRoomName());
        channelRoomInfo.setUserCount(0);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }

    // ルームに新しいユーザーが入室すると、updateChannelRoomInfo関数を介して変更されたルーム情報を適用します。
    onJoinRoom(...) {
		...
        channelRoomInfo.setUserCount(++userCount);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }

    // ルームからユーザーが退室すると、updateChannelRoomInfo関数を介して変更されたルーム情報を適用します。
    onPostLeaveRoom(...) {
		...
        channelRoomInfo.setUserCount(--userCount);

        roomContext.updateChannelRoomInfo(channelUserInfo);
    }
}
```

## チャネル情報の同期

同じチャネルのゲームノードは、互いにチャネル関連の情報を共有します。例えば、同じチャネルに属する1つのゲームノードで、前述の方法でユーザーやルーム情報が変更されると、そのチャネルの他のゲームノードでは以下のコールバックメソッドが呼び出されます。これらのコールバックを利用して、同じチャネル内の全てのゲームノードが情報を同期できます。以下は、ゲームノードでこのようなチャネル同期のために使用されるコールバックメソッドです。

```java
/**
 * 同じチャネルの他のノードでユーザーの変更があった場合に呼び出されます
 * <p>
 * updateChannelUser()の呼び出し時に発生。
 *
 * @param type           チャネル情報変更タイプ(更新/削除)である{@link ChannelUpdateType}
 * @param channelUserInfo 変更されるユーザー情報である{@link IChannelUserInfo}
 * @param userId         変更対象のユーザーID
 * @param accountId      変更対象のアカウントID
 */
@Override
public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {
}

/**
 * 同じチャネルの他のノードでルームの状態が変更された場合に呼び出されます
 * <p>
 * updateChannelRoomInfo()の呼び出し時に発生
 *
 * @param type           チャネル情報変更タイプ(更新/削除) {@link ChannelUpdateType}
 * @param channelRoomInfo 変更されるルーム情報である{@link IChannelRoomInfo}
 * @param roomId         変更対象のルームID
 */
@Override
public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {
}

/**
* クライアントからチャネル情報がリクエストされた場合に呼び出し  (Base.GetChannelInfoReq)
 *
 * @param outPayload クライアントに渡されるチャネル情報
 */
@Override
public void onChannelInfo(IPayload payload) {
}
```

### クライアントへのチャネル情報同期

クライアントはサーバーにいつでもチャネル情報をリクエストできます。このとき、前述のゲームノードのコールバックメソッドのうち、onChannelInfoが呼び出されます。ただし、クライアントの不適切な実装や悪意のある使用を防ぐため、このコールバックメソッドの呼び出しには最小限の再呼び出し周期(デフォルト1秒)が設定されています。例えば、クライアントが1秒間に10回チャネル情報をリクエストしても、サーバーは1回だけonChannelInfoコールバックメソッドを呼び出します。残りの9回のリクエストには、以前にキャッシュした情報を伝達します。以下は、このようなonChannelInfoを実装した疑似コードです。

```java
public void onChannelInfo(Payload outPayload) {

    // クライアントに送信するユーザー定義のチャネル情報を作成
    Game.GameChannelInfo.Builder channelInfoBuilder = Game.GameChannelInfo.newBuilder();

    channelInfoBuilder.setChannelId(gameNodeContext.getChannelId());

    // getChannelUserInfo APIを介してチャネルユーザー情報を取得します。
    for (IChannelUserInfo channelUserInfo : getContext().getChannelUserInfo(StringValues.userType)) {
        Game.GameChannelInfo.Builder channelUserInfoBuilder = Game.GameChannelInfo.newBuilder();
        channelUserInfoBuilder.setAccountId(channelUserInfo.getAccountId());
        channelUserInfoBuilder.setUserId(channelUserInfo.getUserId());

        channelInfoBuilder.addChannelUserInfo(channelUserInfoBuilder.build());
    }

    // getChannelRoomInfo APIを介してチャネルルーム情報を取得します。
    for (IChannelRoomInfo channelRoomInfo : getContext().getChannelRoomInfo(StringValues.roomType_2_User_Match)) {
        Game.GameChannelRoomInfo.Builder channelRoomInfoBuilder = Game.GameChannelRoomInfo.newBuilder();
        channelRoomInfoBuilder.setRoomId(channelRoomInfo.getRoomId());
        channelRoomInfoBuilder.setUserCount(((GameChannelRoomInfo)channelRoomInfo).getUserCnt());

        channelInfoBuilder.addChannelRoomInfo(channelRoomInfoBuilder.build());
    }

    // outPayloadにクライアントに送信するチャネル情報を追加します。
    outPayload.add(channelInfoBuilder);
}
```

### クライアントへのチャネルに属するユーザー数とルーム数の伝達

GameAnvilコネクタは、このような情報をリクエストするためにGetChannelCountInfo APIを提供します。エンジンが常にチャネル単位のユーザー数/ルーム数を管理しているため、ユーザーが別途実装する必要はありません。
