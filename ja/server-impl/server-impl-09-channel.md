## Game > GameAnvil > サーバー開発ガイド > チャンネル



## チャンネル(Channel)

![channel-sync2.png](https://static.toastoven.net/prod_gameanvil/images/channel-sync2.png)

チャンネルは、単一サーバー群を論理的に分けることができる方法の1つです。GameAnvilは1つ以上のゲームノードを含む場合、チャンネルを設定できます。基本的にGameAnvilConfigによってゲームノードに次の例のようなチャンネルを設定できます。この例では4個のゲームノードに対して、それぞれch1、ch1、ch2、ch2を設定します。

```json
"game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "MyGame",
      "channelIDs": [
        "ch1",
        "ch1",
        "ch2",
        "ch2"
      ],
    }
```

チャンネルを使用したくない場合は、チャンネルIDをすべて次のように""で設定してください。この場合、すべてのチャンネル機能は使用できません。つまり、次のゲームノード4個はすべて、チャンネルに関係なく独立して動作します。

```json
"game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "MyGame",
      "channelIDs": [
        "",
        "",
        "",
        ""
      ],
    }
```

もしゲームノード全体を1つのチャンネルで管理したい場合は、次のように有効な文字列でチャンネルIDを入力します。

```json
"game": [
    {
      "nodeCnt": 4,
      "serviceId": 1,
      "serviceName": "MyGame",
      "channelIDs": [
        "OnlyOneChannel",
        "OnlyOneChannel",
        "OnlyOneChannel",
        "OnlyOneChannel"
      ],
    }
```

このようなチャンネルによって次の機能を使用できます。

* チャンネル情報を管理できます。該当チャンネルに属しているユーザーとルーム情報を使用できるようになります。
* チャンネル別のユーザーとルームの数を照会できます。
* チャンネル単位でメッセージを送信できます。publishToChannel APIを使用すると、対象チャンネルに属しているすべてのゲームノードにメッセージを送信します。



## チャンネル情報管理

ユーザーはチャンネルで管理する情報を直接実装できます。これらの情報は、同じチャンネル内で自動的に同期されます。

### チャンネルユーザー情報

まず、チャンネルでユーザー情報を管理するためには、次のようにユーザークラスを実装する時にアノテーションによってチャンネルユーザー情報を有効にします。

```java
@ServiceName("MyGame")
@UserType("BasicUser")
@UseChannelInfo // チャンネルユーザー情報を有効化
public class SampleGameUser extends BaseUser implements TimerHandler {
	...
}
```

そして、追加でBaseChannelUserInfoを実装します。この時、ユーザーがチャンネルで管理したい情報をすべて加えてください。

```java
public class GameChannelUserInfo implements Serializable, BaseChannelUserInfo {
	// チャンネルで表示するユーザー情報
	private int userId = 0;
	private String accountId = "";
	private int level = 0;

    ...
  
    /**
     * 変更されるChannel User情報のUser Id。
     *
     * @return int typeでUserIdを返す。
     */
    @Override
    int getUserId() {
        return userId;
    }

    /**
     * 変更されるChannel User情報のAccount Id。
     *
     * @return String typeでAccountIdを返す。
     */
    @Override
    String getAccountId() {
        return accountId;
    }
}
```

こうして作成したチャンネルユーザー情報は、次のようにユーザーオブジェクトで追加や更新できます。この時、updateChannelUserInfo APIを使用します。もし該当ユーザーオブジェクトがサーバーからログアウトすると、該当チャンネルユーザー情報も一緒に自動的に削除されます。以下はこれに対するpseudoコードです。

```java
public class SampleGameUser extends BaseUser {
	GameChannelUserInfo channelUserInfo = new GameChannelUserInfo();
  
	...

	// ユーザーがログイン時にチャンネルユーザー情報を追加します。
	onLogin(...) throws SuspendExecution {
		...
		channelUserInfo.setUserId(getId())
		channelUserInfo.setAccountId(getAccountId());
		channelUserInfo.setLevel(getLevel());
      
		updateChannelUserInfo(channelUserInfo);
		...
	}
  
	// チャンネル移動時に対象チャンネルに新しいチャンネルユーザー情報を追加します。
	onMoveInChannel(...) throws SuspendExecution {
		...
		channelUserInfo.setUserId(getId());
		channelUserInfo.setAccountId(getAccountId());
		channelUserInfo.setLevel(getLeve());
      
		updateChannelUserInfo(channelUserInfo);
		...
	}
  
	// ユーザーコンテンツで変更されたユーザー情報をいつでも更新できます。
	updateLevel(int level) {
		channelUserInfo.setLevel(getLevel());
      
		updateChannelUserInfo(channelUserInfo);
	}
}
```

また、チャンネル移動時に以前のチャンネルでは自動的に該当チャンネルユーザー情報を削除するため、ユーザーは対象チャンネルで新たに追加する情報にのみ集中できます。

### チャンネルのルーム情報

チャンネルでルーム情報を管理するためには、前述したゲームユーザーと同様に、ルームクラスを実装する時にアノテーションによってチャンネルのルーム情報を有効にする必要があります。

```java
@ServiceName("MyGame")
@RoomType("BasicRoom")
@UseChannelInfo // チャンネル情報の有効化
public class GameRoom extends BaseUser implements TimerHandler {
    ...
}
```

そして、追加でBaseChannelRoomInfoを実装します。この時、ユーザーがチャンネルで管理したいルームの関連情報をすべて加えてください。

```java
public class GameChannelRoomInfo implements Serializable, BaseChannelRoomInfo {
	private int roomId = 0;
	private String roomName = "";
	private int userCount = 0;

    ...
      
    /**
     * Room情報のRoom Id。
     *
     * @return int typeでRoomIdを返す。
     */
    @Override
    public int getRoomId() {
        return roomId;
    }

    /**
     * Room情報をコピーする。
     *
     * @return RoomInfoでコピーされたRoom情報を返す。
     * @throws CloneNotSupportedExceptionコピーできない場合。
     */
    @Override
    public BaseChannelRoomInfo copy() throws CloneNotSupportedException {
        GameChannelRoomInfo channelRoomInfo = (GameChannelRoomInfo)super.clone();
      
        // データコピーが必要な場合はここで実行します。
      
        return channelRoomInfo;
    }   
}
```

こうして作成したチャンネルのルーム情報は、次のようにルームオブジェクトで追加や更新できます。この時、updateChannelRoomInfo APIを使用します。もし該当ルームオブジェクトがサーバーから削除された場合は、該当チャンネルのルーム情報も一緒に自動的に削除されます。以下はこれに対するpseudoコードです。

```java
public class GameRoom extends BaseRoom<GameUser> {
	GameChannelRoomInfo channelRoomInfo = new GameChannelRoomInfo();
  
	...
      
	// ルーム作成時にupdateChannelRoomInfo関数でチャンネルのルーム情報を追加します。
	onCreateRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setRoomId(getId());
		channelRoomInfo.setRoomName(getRoomName());
		channelRoomInfo.setUserCount(0);
      
		updateChannelRoomInfo(channelUserInfo);
	}
  
	// ルームに新しいユーザーが入場した場合、updateChannelRoomInfo関数で変更されたルーム情報を適用します。
	onJoinRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setUserCount(++userCount);
      
		updateChannelRoomInfo(channelUserInfo);
	}
  
    // ルームからユーザーが退出した場合、updateChannelRoomInfo関数で変更されたルーム情報を適用します。
	onPostLeaveRoom(...) throws SuspendExecution {
		...
		channelRoomInfo.setUserCount(--userCount);
      
		updateChannelRoomInfo(channelUserInfo);
	}
}
```

## チャンネル情報の同期

同じチャンネルのゲームノードは、互いにチャンネル関連情報を共有します。例えば、同じチャンネルに属している1つのゲームノードにおいて、前述した方式でユーザーやルームの情報が変更されると、該当チャンネルの残りのゲームノードは、次のようなコールバックメソッドが呼び出されます。これらのコールバックを利用して、同じチャンネル内のすべてのゲームノードが情報を同期できます。以下は、ゲームノードでこれらのチャンネル同期用に使用されるコールバックメソッドです。

```java
 	/**
     * 同じチャンネルの別のノードでユーザー変化が発生した時に呼び出し
     * すなわち、updateChannelUser() APIを呼び出した時に発生
     *
     * @param type            Channel情報の変更タイプ(更新/削除)を送信。
     * @param channelUserInfo変更されるUser情報を送信。
     * @param userId        変更対象のUser Idを送信。
     * @param accountId     変更対象のAccount Idを送信。
     * @throws SuspendExecution このメソッドはファイバーがsuspendになることがある。
     */
    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType type, ChannelUserInfo channelUserInfo, final int userId, final String accountId) throws SuspendExecution {  
    }

    /**
     * 同じチャンネルの別のノードでルームの状態変化が発生した時に呼び出し
     * すなわち、updateChannelRoomInfo() APIを呼び出した時に発生
     *
     * @param type            Channel情報の変更タイプ(更新/削除)を送信。
     * @param channelRoomInfo変更されるRoom情報を送信。
     * @param roomId        変更対象のRoom Idを送信。
     * @throws SuspendExecution このメソッドはファイバーがsuspendになることがある。
     */
    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType type, ChannelRoomInfo channelRoomInfo, final int roomId) throws SuspendExecution {  
    }

    /**
     * クライアントがチャンネル情報をリクエストした時に呼び出し
     *
     * @param outPayload Clientに送信されるChannel情報を送信。
     * @throws SuspendExecution このメソッドはファイバーがsuspendになることがある。
     */
    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {  
    }
```

### クライアントにチャンネル情報を同期

クライアントはサーバーにいつでもチャンネル情報をリクエストできます。この時、前述したゲームノードのコールバックメソッドのうち、onChannelInfoが呼び出されます。ただし、クライアントの誤った実装または、悪意のある使用を防ぐために、このコールバックメソッドの呼び出しは最小限の再呼び出しサイクル(デフォルト値1秒)を備えています。例えば、クライアントが1秒間に10回、チャンネル情報をリクエストしても、サーバーは1回のみonChannelInfoコールバックメソッドを呼び出します。残りの9回のリクエストは、キャッシュしておいた情報を送信します。以下は、これらのonChannelInfoを実装したpseudoコードです。

```java
public void onChannelInfo(Payload outPayload) throws SuspendExecution {

    // クライアントに送信するユーザー定義のチャンネル情報を作成
	Game.GameChannelInfo.Builder channelInfoBuilder = Game.GameChannelInfo.newBuilder();

	channelInfoBuilder.setChannelId(this.getChannelId());

	// getChannelUserInfo APIでチャンネルのユーザー情報を取得します。
	for (BaseChannelUserInfo channelUserInfo : getChannelUserInfo(UserType_1)) {
		GameChannelUserInfo gameChannelUserInfo = (GameChannelUserInfo) channelUserInfo;

        Game.GameChannelUserInfo.Builder channelUserInfoBuilder = Game.GameChannelUserInfo.newBuilder();
        channelUserInfoBuilder.setUserName(gameChannelUserInfo.getUserName());
        channelUserInfoBuilder.setLevel(gameChannelUserInfo.getlevel());

    	channelInfoBuilder.addChannelUserInfos(channelUserInfoBuilder.build());
    }
  
	// getChannelRoomInfo APIでチャンネルのルーム情報を取得します。
    for (BaseChannelRoomInfo channelRoomInfo : getChannelRoomInfo(RoomType_2)) {  
	   	GameChannelRoomInfo gameChannelRoomInfo = (GameChannelRoomInfo) channelRoomInfo;
  
    	Game.GameChannelRoomInfo.Builder channelRoomInfoBuilder = Game.GameChannelRoomInfo.newBuilder();
        channelRoomInfoBuilder.setRoomId(gameChannelRoomInfo.getRoomId());
        channelRoomInfoBuilder.setRoomName(gameChannelRoomInfo.getRoomName());
        channelRoomInfoBuilder.setUserCount(gameChannelRoomInfo.getUserCnt());

        channelInfoBuilder.addChannelRoomInfos(channelRoomInfoBuilder.build());
	}

    // outPayloadにクライアントに送信するチャンネル情報を追加します。
	outPayload.add(channelInfoBuilder);
}
```

### クライアントにチャンネルに属しているユーザーとルームの数を送信する

GameAnvilコネクタは、これらの情報をリクエストするために、GetChannelCountInfo APIを提供しています。エンジンが常にチャンネル単位のユーザー/ルームの数を管理しているため、ユーザーは別途の実装をする必要がありません。
