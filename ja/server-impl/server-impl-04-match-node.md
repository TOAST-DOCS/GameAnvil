## Game > GameAnvil > サーバー開発ガイド > マッチノード実装



## MatchNodeとMatchMaker



![MatchNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_matchnode_on_network.png)


ユーザーは簡単にマッチングロジックのみを実装し、MatchNodeを有効にしてマッチメイキングを適用できます。マッチメイカーは、マッチングロジックに基づいてユーザーを任意のルームに入場させます。GameAnvilは2つのマッチメイカーを提供しています。それぞれユーザー単位のマッチングを行うUserMatchMakerとルーム単位のマッチングを行うRoomMatchMakerです。

これらのマッチメイカーは、MatchNodeから独立して動作します。この時、MatchNodeはマッチメイキングを行う用途以外に追加でコンテンツを実装できません。したがって、ユーザーはMatchNodeではなく、マッチメイカーにのみ集中できます。ただし、マッチメイカーを実行するためには、少なくとも1つ以上のMatchNodeを有効にする必要があります。

> [参考] 
> 
> マッチメイカーはマッチンググループ単位で作成されます。すなわち、同じマッチンググループ同士でマッチメイキングを行います。 
> 
> MatchNodeは必須ノードではないため、マッチメイキングを使用しない場合は、動作する必要がありません。

## マッチンググループ(MatchingGroup)

マッチンググループもチャネルと同様に単一サーバー群を論理的に分ける方法の1つです。ただし、マッチンググループはチャネルとは異なり、あらかじめ明確に設定して使用する値ではありません。また、チャネルはGameNodeを論理的に分かるための方法であるのに対し、マッチンググループは、マッチメイキングを論理的に分けるための方法です。同じマッチンググループでユーザーマッチングまたはルームマッチングをリクエストした場合、マッチングをリクエストしたチャネル内でマッチングしたルームが作成されます。 

ゲームユーザーで説明したユーザーマッチメイキングのコールバックメソッドであるonMatchUserについて再度説明します。コールバックメソッドの文字列引数であるmatchingGroupは、クライアントがユーザーマッチメイキングをリクエストした時に一緒に送信した値です。すなわち、マッチンググループは、クライアントとサーバー間で事前に約束された任意の文字列です。その種類や数はユーザーが好きなだけ使用できます。すべてのマッチメイキングリクエストは、それぞれのマッチンググループ単位でキューが管理され、該当キュー内でのみマッチングが可能です。例えば、引数であるmatchingGroupに"初心者"という値が送信された場合、このリクエストは他の"初心者"リクエストとのマッチングが保証されます。

```java
    /**
     * クライアントがuserMatchをリクエストした場合、呼び出されるコールバック
     *
     * @param roomType クライアントとサーバー間にあらかじめ定義したルームの種類を区別する任意の値
     * @param payload  クライアントから送信されたペイロード
     * @param outPayloadクライアントに送信するペイロード
     * @returnの戻り値がtrueの場合、ユーザーマッチメイキングリクエストが成功し、falseの場合は失敗
     * @throws SuspendExecution このメソッドはファイバーをsuspendできることを意味する
     */
    @Override
    public final boolean onMatchUser(final String roomType, final String matchingGroup, final Payload payload, Payload outPayload) throws SuspendExecution {

        UserMatchInfo userMatchInfo = new UserMatchInfo(getUserId());
        userMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, userMatchInfo, payload);
    }
```

マッチンググループは"初心者"、"中級者"、"上級者"のように実力に基づいて定義でき、"韓国"、"日本"、"米国"のように国別で定義することもできます。つまり、ユーザーが希望するどのような値もマッチンググループになることができます。



## ユーザーマッチメイカーの実装

ユーザーマッチメイキングは、ゲームユーザーのマッチングリクエストをキューに積みます。特定の時間周期でこのリクエストキューの内容を比較、分析してユーザーが希望する基準で任意のユーザーを1つのルームに入場させます。ここで、ユーザーはリクエストキューの内容をどのように比較、分析し、どのような基準でユーザーをマッチングするかについてのロジックにのみ集中できます。また、最も代表的なユーザーマッチメイキングゲームとして「リーグ・オブ・レジェンド」が挙げられます。


### ユーザーマッチリクエストの実装

このようなユーザーマッチメイキングの最も基本的なものは、マッチングリクエストそのものです。マッチリクエストを次のようにエンジンが提供しているBaseUserMatchInfo抽象クラスを継承して実装します。この時、リクエスト者を区別できるゲームユーザーのIDを提供できるようにgetId()メソッドを必ず実装する必要があります。また、リクエストはいつでもシリアル化できる必要があるため、Serializableインターフェースを実装する必要があります。次の例はマッチングリクエスト間の比較のために、Comparableインターフェースを追加で実装しています。

```java
public class UserMatchInfo extends BaseUserMatchInfo implements Serializable, Comparable<UserMatchInfo> {

    private int userId;
    private int rating;

    public UserMatchInfo(int userId, int rating) {
        this.userId = userId;
        this.rating = rating;
    }

    public int getRating() {
        return rating;
    }

    /**
     * 該当のユーザーマッチリクエストがどのユーザーが送ったものなのかを判断するために使用されます。
     *
     * @returnパーティマッチングリクエストの場合はRoomIdを、ユーザーマッチングリクエストの場合はUserIdを返す
     */
    @Override
    public int getId() {
        return userId;
    }

    /**
     * リクエストパーティのサイズを返します。
     *
     * @returnパーティマッチングリクエストの場合はパーティのサイズ(ユーザー数)を、ユーザーマッチングリクエストの場合は0を返す
     */
    @Override
    public int getPartySize() {
        return 0;
    }

    // もしUserMatchInfoオブジェクト間の比較が必要な場合は、Comparableインターフェースを実装します。
    @Override
    public int compareTo(SampleUserMatchInfo o) {
        if (this.rating < o.getRating())
            return -1;
        else if (this.rating > o.getRating())
            return 1;

        if (this.id < o.getId())
            return -1;
        else if (this.id > o.getId())
            return 1;

        return 0;
    }
}
```



ユーザーマッチリクエストで再定義する必要があるメソッドを表にまとめると、次のとおりです。

| コールバック名  | 意味               | 説明                                                                                                                       |
| ------------ | ------------------- |----------------------------------------------------------------------------------------------------------------------------|
| getId        | マッチリクエスト者の情報  | 該当ユーザーマッチリクエストがどのユーザーが送信したものかを判断するために使用されます。したがって、必ずリクエスト者のIDを返すように再定義する必要があります。                                           |
| getPartySize | マッチリクエストパーティの規模 | リクエストパーティのサイズを返します。この値でパーティマッチメイキングリクエストの有無を判断するため、ユーザーマッチメイキングリクエストの場合は、必ず0を返すように再定義します。パーティマッチリクエストの場合は、該当パーティのユーザー数を返します。 |



### ユーザーマッチメイカー

ユーザーマッチメイカーは、ユーザーマッチリクエストを実際に処理し、エンジンが提供するBaseUserMatchMaker抽象クラスを継承実装します。特にonMatch()メソッドは、実際にマッチングを行うために呼び出されるコールバックであるため注意が必要です。onRefill()メソッドはすでに完了したマッチメイキングに対してユーザーの補充リクエストを処理するコールバックです。例えば、4人がマッチメイキングした状態で1人がゲームを終了した時に1人を補充するために使用できます。次のサンプルコードは、これらのユーザーマッチメイカーを実装する方法を示しています。


特にマッチメイカーを実装したクラスは、@ServiceNameアノテーションを使用して特定のサービスに対する用途でエンジンに登録します。また、マッチメイカーによって作成されるルームのタイプを@RoomTypeアノテーションであらかじめ定義します。この時、1つのマッチメイカークラスは1つのサービスに対してのみ登録できます。

```java
@ServiceName("MyGame") // "MyGame"というサービスに対するマッチメイカーをエンジンに登録
@RoomType("1vs1") // このマッチメイカーによって作成されるルームの種類を意味する文字列
public class UserMatchMaker extends BaseUserMatchMaker<UserMatchInfo> {

    private static final Logger logger = getLogger(UserMatchMaker.class);

    public UserMatchMaker() {
        super(2, 5000); // マッチメイカーの定員は2人であり、リクエストが5秒以内にマッチングしなければ、Timeout処理されるように引数を設定
    }

    private Multiset<UserMatchInfo> ratingSet = TreeMultiset.create();
    private final int matchMultiple = 1; // match定員の何倍のユーザーを集めてから、rating別にソートしてマッチングするのか？
    private int currentMultiple = matchMultiple;
    private long lastMatchTime = System.currentTimeMillis();
    private int totalMatchMakings = 0;

    /**
     * ユーザーマッチメイキングとパーティマッチメイキングリクエストを処理。最初の呼び出し後、定期的に呼び出されます。
     * 
     * getMatchRequests(int)を利用して現在まで累積されたマッチリクエストリストの全体を取得できます。
     * ユーザーは、このリストを利用して条件に合うリクエストを別途のCollectionに順番に並べた後、matchSingles(Collection)やassignRoom(BaseUserMatchInfo)または、assignRoom(Collection) APIを利用してマッチングを完了できます。
     * もし最小限のリクエスト数が集まらなければ、次の呼び出しで処理します。
     */
    @Override
    public void onMatch() {
        List<UserMatchInfo> matchRequests = getMatchRequests(matchSize * currentMultiple);

        // 最小数(minAmount)が積まれなかった
        if (matchRequests == null) {
            if (System.currentTimeMillis() - lastMatchTime >= 10000)
                currentMultiple = Math.max(--currentMultiple, 1);

            return;
        }

        // matchingが成立していない項目は、ratingSetにそのまま残っている可能性があるが、別途で保管する必要はない。
        // この項目は次のgetMatchRequests()で再送信される。
        ratingSet.clear();
        ratingSet.addAll(matchRequests);

        if (ratingSet.size() >= matchSize) {

            // ratingSetの順にmatchingAmount * matchSizeだけの項目を消費
            int matchingAmount = matchSingles(ratingSet);

            if (matchingAmount > 0) {
                totalMatchMakings += matchingAmount;
                logger.info("{} match(s) made (total: {}) - {}", matchingAmount, totalMatchMakings, this.getMatchingGroup());

                lastMatchTime = System.currentTimeMillis();
                currentMultiple = matchMultiple;
            }
        }
    }

    /**
     * ユーザー/パーティマッチメイキングプロセス中に任意のユーザーが退出した場合に新しいユーザーを補充するための処理
     * 
     * 通常、マッチメイキングしたルームに対してonLeaveRoomが呼び出された時、matchRefillを呼び出して連動できます。すなわち、マッチングしたルームから誰かが退出した時に補充をリクエストすることです。補充はキューに溜まっているマッチリクエストには使用しません。補充リクエスト後に新しく入ってくるマッチリクエストのみ対象とします。
     * 
     * getRefillRequests APIを利用して、補充リクエストを取得できます。ゲームによっては、補充を使用するよりも任意のユーザーが退出した状態でゲームを進行したり、マッチングしたゲームをキャンセルした後、再度マッチメイキングをリクエストしたりする方が適している場合もあります。
     *
     * @param req補充に使用する新規マッチリクエストの情報
     * @return補充に成功した場合はtrueを、失敗した場合はfalseを返します。
     */
    @Override
    public boolean onRefill(UserMatchInfo matchReq) {
        try {
            List<UserMatchInfo> refillRequests = getRefillRequests();

            if (refillRequests.isEmpty()) {
                return false;
            }

            for (UserMatchInfo refillInfo : refillRequests) {
                // 100点以上の差が出なければ補充
                if (Math.abs(matchReq.getRating() - refillInfo.getRating()) < 100) {
                    if (refillRoom(matchReq, refillInfo)) { // 該当マッチングリクエストを補充が必要なルームとしてマッチング
                        return true;
                    }
                }
            }
        } catch (Exception e) {
            logger.error("UserMatchMaker::refill()", e);
        }

        return false;
    }
}
```



これらのユーザーマッチメイキングのコールバックメソッドをまとめると、次の表のようになります。

| コールバック名 | 意味               | 説明                                                                                                                                                                                                                                                                                                                                                                                      |
| --------- | ------------------- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onMatch   | マッチリクエストを処理 | ユーザーは、BaseUserMatchMakerが提供するAPIを使用して直接マッチリクエストを処理できます。この時、ユーザーの思いどおりにロジックを直接実装できます。サンプルコードの処理フローが最も基本的な方式です。<br>すなわち、getMatchRequests APIを利用して最小限のマッチリクエストを取得した後、ユーザーの希望する人数に合わせてリクエストを組み合わせて、任意のCollectionに順番に入れます。このCollectionをmatchSingles APIに引数として送信すると、定員に合わせてマッチメイキングが行われます。サンプルコードの場合、定員が2人のユーザーマッチメイキングであるため、Collectionを巡回し、順番に2人ずつ抽出して1つのゲームでマッチングさせます。 |
| onRefill  | マッチ補充リクエストの処理 | ユーザー/パーティマッチメイキングプロセス中に任意のユーザーが退出した場合、新しいユーザーを補充するための処理を行います。通常、マッチメイキングしたルームに対してonLeaveRoomが呼び出された時、matchRefillを呼び出して連動できます。すなわち、マッチングしたルームから誰かが退出した時に補充をリクエストするということです。補充はキューに溜まったリクエストには使用しません。補充リクエスト後に新しく入ってくるマッチリクエストのみ対象とします。                                                                                                                                                      |



### GameUserからマッチメイカーにリクエストを送信する

クライアントはサーバーにユーザーマッチメイキングをリクエストできます。このリクエストはGameUserに送信された後、エンジンによってonMatchUserコールバックメソッドを呼び出します。これについては、GameNodeとGameUserを説明しながら見てきました。ユーザーは、このコールバックメソッドでGameAnvilが提供しているユーザーマッチメイカーや、直接実装した別のマッチメイカー、他のソリューションを使用できます。しかし、特別な理由がなければ、GameAnvilのユーザーマッチメイキングの使用を推奨します。

次の例は、エンジンが提供しているユーザーマッチメイカーを使用するために、前述したマッチリクエストを作成してmatchUser APIを呼び出す様子です。特に、引数で送信されたマッチンググループは、マッチメイキングを論理的に分けることができる概念であると先ほども説明しました。仮に"Newbie"というマッチンググループを送信した場合、同じ"Newbie"マッチンググループ同士で同じマッチングキューを共有します。

```java
    /**
     * クライアントがuserMatchをリクエストした場合、呼び出されるコールバック
     *
     * @param roomType クライアントとサーバー間にあらかじめ定義したルームの種類を区別する任意の値
     * @param matchingGroupクライアントがリクエストしたマッチンググループ
     * @param payload  クライアントから送信されたペイロード
     * @param outPayloadクライアントに送信するペイロード
     * @returnの戻り値がtrueの場合、ユーザーマッチメイキングリクエストが成功し、false場合は失敗
     * @throws SuspendExecution このメソッドはファイバーがsuspendになることがある。
     */
    public boolean onMatchUser(final String roomType, final String matchingGroup,
                               final Payload payload, Payload outPayload) throws SuspendExecution {

        UserMatchInfo userMatchInfo = new UserMatchInfo(getUserId()); // マッチリクエスト作成
        userMatchInfo.setRating(rating);

        return matchUser(matchingGroup, roomType, userMatchInfo, payload);
    }
```



最後に、クライアントはいつでも事前にリクエストしたユーザーマッチメイキングをキャンセルできます。この時、エンジンはキャンセル処理が成功すると、次のようにGameUserのonMatchUserCancelコールバックメソッドを呼び出します。ユーザーは、このコールバックでキャンセルのタイミングに処理したい部分を実装できます。

```java
    /**
     * クライアントでuserMatchがキャンセルされた時に呼び出されるコールバック
     *
     * @param reasonキャンセルされた理由(TIMEOUT/CANCEL)
     * @return正常にキャンセルされた場合はtrueを、キャンセルできない場合はfalseを返す。
     * @throws SuspendExecution このメソッドはファイバーがsuspendになることがある。
     */
    public boolean onMatchUserCancel(final MatchCancelReason reason) throws SuspendExecution {
    }
```



## ルームマッチメイカーの実装

ルームマッチマッチングは、ユーザーを最適なルームに自動的に入場させる機能です。ルームマッチメイキングをリクエストしたユーザーをどのような基準で、どのルームに入場させるかは、ユーザーの実装にかかっています。最もユーザーが多いルームに入場させることや、最もユーザーが少ないルームに入場させることもあります。もしくは、平均スコアが最も高いルームに入場させることもあります。ユーザーは、これらのマッチングロジックにのみ集中できます。また、最も代表的なルームマッチメイキングゲームには「ハンゲームポーカー」や「カートライダー」などが挙げられます。


> [参考]
>
> マッチメイカーは、マッチンググループ単位で作成されます。すなわち、同じマッチンググループ同士でマッチメイキングを行います。
> 
> ルームマッチメイカーとユーザーマッチメイカーは、互いに独立して運営されています。すなわち、同じマッチンググループでユーザーマッチングとルームマッチングをそれぞれリクエストしても、この2つのリクエストが同時にマッチングすることはありません。


### ルームマッチリクエストの実装

このようなルームマッチメイキングの最も基本的なものは、マッチングリクエストそのものです。マッチングリクエストとは、1人のユーザーが送ったリクエストを意味し、次のようにエンジンが提供しているBaseRoomMatchForm抽象クラスを継承実装します。リクエストはいつでもシリアル化できる必要があるため、Serializableインターフェースを追加で実装する必要があります。以下は、これらのマッチリクエストを実装した例です。

```java
public class GameRoomMatchForm extends BaseRoomMatchForm implements Serializable {
	
	// ユーザーがマッチングで使用する条件を追加
	private int money;
	private int level;

	public GameRoomMatchForm(int money, int level) {
        this.money = money;
        this.level = level;
    }
    
    public GameRoomMatchForm(String matchingUserCategory) {
        super(matchingUserCategory);
    }
    
    ...
}
```

ルームマッチのリクエストは基本的にマッチングロジックで使用する情報が含まれています。これは、ユーザーがマッチメイキングロジックを直接実装する時に使用されます。ルームマッチのリクエストにおいて重要な情報は、マッチングユーザーカテゴリーです。マッチングユーザーカテゴリーは、1つのルームでユーザーが属するグループを区別するための任意の文字列です。例えば、4人が定員のルームで2チームに分けて2vs2のゲームをする場合、それぞれのユーザーが属するチームを指定するために使用されることがあります。もし何の値も指定しなかった場合は、エンジンのデフォルト値が使用されます。



### ルームマッチ情報の実装

マッチング対象となるルームは、ルームマッチメイカーによってマッチ情報が管理されます。すなわち、1つのルームマッチ情報は1つのマッチング可能なルーム情報を示していると考えることができます。この時、該当ルームのさまざまな情報と状態値を含めることができます。BaseRoomMatchInfoを継承実装し、必ずルームIDとマッチングユーザーカテゴリーそして、各マッチングカテゴリー別に最大定員数を設定する必要があります。そして最後に、ルームマッチ情報もシリアル化のために必ずSerializableインターフェースを実装する必要があります。以下はサンプルコードを示しています。 

```java
public class GameRoomMatchInfo extends BaseRoomMatchInfo implements Serializable {
	private static final int MAX_USER = 3;
	
	public GameRoomMatchInfo(int roomId) {
		// 別途のマッチングユーザーカテゴリーが不要な場合はデフォルト値を使用できます。
    	super(roomId, MAX_USER);
	}
    
    ...
}
```


次の例は、ユーザーが任意のマッチングユーザーカテゴリーを設定する場合です。1つのゲームルームにレッドチームとブルーチームがそれぞれ3人ずつで構成されています。このようにカテゴリーを分ける場合、ルームマッチメイカーがそれぞれのマッチングカテゴリー別にユーザー数を管理してくれます。また、マッチメイキングに使用する追加情報を好きなように構成できます。

```java
public class GameRoomMatchInfo extends BaseRoomMatchInfo implements Serializable {    
    private static final int MAX_USER = 3;

	// マッチング条件情報
	int mapId;
    int maxLevel;
	int avgLevel = 0;
    
	public GameRoomMatchInfo(int roomId) {
		// RED_TEAM最大人数は3人で人数情報を追加します。
    	super(roomId, "RED_TEAM", 3);
        
        // 必要な場合は追加マッチカテゴリーを登録します。
        // BLUE_TEAM最大人数は3人で人数情報を追加します。
		addMatchingUserCategory("BLUE_TEAM", 3);
	}
    
    public void setAvgLevel(int avgLevel) {
		this.avgLevel = avgLevel;
	}
    
    ...
}
```



### ルームマッチ情報の登録/更新

これらのルームマッチ情報は、ユーザーが直接登録/更新できます。つまり、ユーザーが希望しない場合、特定のルームをルームマッチメイキングの対象に登録しないこともあります。このような登録手続きは通常、次のようにルームを作成するonCreateRoomコールバックメソッドで主に実行します。

```java
@Override
public final boolean onCreateRoom(final GameUser user, final Payload payload, Payload outPayload) throws SuspendExecution {
    
    ...
        
	// これからこのルームはルームマッチメイキングに使用されます。
	registerRoomMatch(gameRoomMatchInfo, user.getUserId());
}
```

任意のルームに対してルームマッチメイキングを登録するかどうかを確認するために、BaseRoomクラスは次のようなAPIを提供しています。

```java
/**
 * ルームマッチに登録されたルームかどうかを確認します。
 *
 * @return登録されたルームである場合は、trueを返します。
 */
final public boolean isRegisteredRoomMatch()
```

このように登録したルームマッチ情報は、必要に応じていつでもユーザーがその情報を更新できます。更新するためには、新しいルームマッチ情報を作成した後、updateRoomMatch APIを呼び出してください。

```java
GameRoomMatchInfo gameRoomMatchInfo = new GameRoomMatchInfo(getId());
gameRoomMatchInfo.setMemberMoney(1000);

updateRoomMatch(gameRoomMatchInfo); // このルームマッチ情報を更新します。
```



該当ルームが削除される場合、ルームマッチ情報はエンジンが自動的に削除するため、ユーザーが削除する必要はありません。



### ルームマッチメイカー

次はルームマッチメイカーを作成する番です。ルームマッチメイカーは、エンジンが提供しているBaseRoomMatchMaker抽象クラスを継承実装します。ルームマッチメイキングは、最適な部屋を検索するプロセスであるため、実際にマッチング前後用に特別なコールバックメソッドが提供されています。ユーザーは、このコールバックメソッドを再定義して、希望するマッチングを行えます。以下のサンプルコードは、このようなルームマッチメイカーをどのように実装するかを示しています。



特にマッチメイカーを実装したクラスは、@ServiceNameアノテーションを使用して特定のサービスに対する用途でエンジンに登録します。また、マッチメイカーによって作成されるルームのタイプを@RoomTypeアノテーションで事前に定義します。この時、1つのマッチメイカークラスは1つのサービスに対してのみ登録できます。

```java
@ServiceName("MyGame") // "MyGame"というサービスに対するマッチメイカーをエンジンに登録
@RoomType("REDvsBLUE") // このマッチメイカーによって作成されるルームの種類を意味する文字列
public class GameRoomMatchMaker extends BaseRoomMatchMaker<GameRoomMatchForm, GameRoomMatchInfo> {
    
    /**
     * ルームマッチを開始する前に、リクエスト情報に基づいて必要な事前処理を実装します。
     *
     * @param baseRoomMatchForm送信されたルームマッチングリクエスト
     * @param args追加で送信されたデータ
     * @returnマッチング結果コード
     */
    @Override
    public RoomMatchResultCode onPreMatch(GameRoomMatchForm roomMatchForm, Object... args) {
        // マッチングリクエスト書のMoneyが100より少ない場合、マッチングを開始せず、マッチングを申請したクライアントに失敗結果コード(1000)を送信
        if (roomMatchForm.getMoney() < 100)
            return RoomMatchResultCode.FAIL(1000);

        return RoomMatchResultCode.SUCCESS;
    }
    
    /**
     * ルームマッチリクエストと任意のルームマッチ情報がマッチング可能な状態であるかどうかを確認します。
     * エンジンは、ルームマッチメイキングが成功するまで、ルームリスト全体に対して1度ずつこのコールバックメソッドを呼び出します。
     *
     * @param baseRoomMatchForm送信されたルームマッチリクエスト
     * @param baseRoomMatchInfoマッチングプールに登録されているルームマッチ情報(マッチング可能なルーム情報)
     * @param args追加で送信されたデータ
     * @returnマッチングの成否
     */
    @Override
    public boolean canMatch(GameRoomMatchForm roomMatchForm, GameRoomMatchInfo roomMatchInfo, Object... args) {
        if (roomMatchForm.getLevel() > roomMatchInfo.getAvgLevel())
            return false;
        return true;
    }
    
    /**
     * ルームマッチに成功した後、必要な処理を実装します。
     *
     * @param baseRoomMatchFormルームマッチリクエスト
     * @param baseRoomMatchInfoマッチングしたルームマッチ情報
     * @param args追加で送信されたデータ
     */
    @Override
    public void onPostMatch(GameRoomMatchForm baseRoomMatchForm, GameRoomMatchInfo baseRoomMatchInfo, Object... args) {
        // マッチングが正常に進んでいるかどうかを確認する。
        logger.debug("GameRoomMatchMaker::onPostMatch() matching success, roomId({})", baseRoomMatchInfo.getRoomId());
    }
    
    /**
     * ルームマッチ情報をソートするために実装します。
     *
     * @return比較結果を返す
     */
    @Override
    public int compare(GameRoomMatchInfo o1, GameRoomMatchInfo o2) {
        if (o1.getCreateTime() < o2.getCreateTime()) {
            return -1;
        } else if (o1.getCreateTime() > o2.getCreateTime()) {
            return 1;
        } else {
            return 0;
        }
    }
    
    /**
     * ルームマッチメイキングの対象であるルームでユーザー数が増加した時に呼び出されます。
     *
     * @param roomIdユーザー数が増加したルームのID
     * @param matchingUserCategory増加したユーザーのマッチングユーザーカテゴリー
     * @param currentUserCount該当マッチングユーザーカテゴリーに属するユーザー数
     */
    @Override
    public void onIncreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {
        // ユーザー数が変更されたため、マッチングプールを再度ソートして、ユーザーが少ないルームが先にマッチングするようにするため
        sort();
    }
    
    /**
     * ルームマッチメイキングの対象であるルームでユーザー数が減少した時に呼び出されます。
     *
     * @param roomIdユーザー数が減少したルームのID
     * @param matchingUserCategory減少したマッチングユーザーカテゴリー
     * @param currentUserCount該当マッチングユーザーカテゴリーに属するユーザー数
     */
    @Override
    public void onDecreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {
        // ユーザー数が変更されたため、マッチングプールを再度ソートして、ユーザーが少ないルームが先にマッチングするようにするため
        sort();
    }
    
    /**
     * 任意のルームマッチ情報のリストを返します。 
     * 引数として渡された現在のマッチングプール(マッチング可能なルームの全体リスト)を好きなように加工し、希望するリストを作成して返せます。
     * もしこのメソッドを再定義しない場合は、ルームの全体リストを返します。
     *
     * @return加工したルームマッチ情報のリスト
     */
    @Override
    public List<GameRoomMatchInfo> getRooms(List<GameRoomMatchInfo> rooms){
        // マッチングプールからルーム10個を取得する。
        List<GameRoomMatchInfo> gameRoomMatchInfoList = rooms.stream().limit(10).collect(Collectors.toList());
        // 取得したルーム10個の順番をランダムに並び替える。
        Collections.shuffle(gameRoomMatchInfo);

        return gameRoomMatchInfoList;
    }    
}
```



これらのルームマッチメイカーのコールバックメソッドを表にまとめると、次のようになります。

| コールバック名         | 意味                             | 説明                                                                                                                                                  |
| ------------------- | --------------------------------- |-------------------------------------------------------------------------------------------------------------------------------------------------------|
| onPreMatch          | ルームマッチの開始前処理            | ルームマッチリクエストに保存された各種情報を使用して、現在ルームマッチメイキングを実行できる状態であるかどうかを事前にチェックするために使用します。例えば、1度のルームマッチメイキングに100コインが必要な場合、該当ユーザーが100コイン以上を保有しているかどうかをここで確認できます。 |
| canMatch            | マッチ確認                       | 引数として送信されたルームマッチリクエストとルームマッチ情報間でマッチングが可能かどうかを確認するために呼び出します。つまり、ルームマッチプール全体を巡回しながら、該当ルームマッチリクエストが入る最適なルームを検索するためのロジックをここに実装できます。                       |
| onPostMatch         | ルームマッチ成功後の処理            | ルームマッチメイキングに成功した後に呼び出されます。ルームマッチが完了した後、処理するロジックはここに実装できます。                                                                                           |
| onIncreaseUserCount | 任意のマッチング対象であるルームでユーザーが増加 | 任意のルームでユーザーが増加した場合に呼び出されます。この時、増加したユーザーがどのマッチングユーザーカテゴリーなのか、そのマッチングユーザーカテゴリーの総員は何人なのかも引数として送信されます。                                                        |
| onDecreaseUserCount | 任意のマッチング対象であるルームでユーザーが減少 | 任意のルームでユーザーが減少した場合に呼び出されます。この時、減少したユーザーがどのマッチングユーザーカテゴリーなのか、そのマッチングユーザーカテゴリーの総員は何人なのかも引数として送信されます。                                                       |
| getRooms            | 任意のルームリストを取得            | エンジンの基本実装は、全体ルームマッチメイキングが可能なルームのリストを返します。ユーザーは、これを再定義して、希望する特定のルームリストのみ返せます。                                                                       |



### GameUserからマッチメイカーにリクエストを送信する

クライアントはサーバーにルームマッチメイキングをリクエストできます。このリクエストはGameUserに送信された後、エンジンによってonMatchRoomコールバックメソッドを呼び出します。これについては、GameNodeとGameUserを説明しながら見てきました。ユーザーは、このコールバックメソッドでGameAnvilが提供しているルームマッチメイカーや、直接実装した別のマッチメイカー、他のソリューションを使用できます。しかし、特別な理由がなければ、GameAnvilのルームマッチメイカーの使用を推奨します。

次の例は、エンジンが提供しているルームマッチメイカーを使用するために前述したマッチリクエストを作成してmatchRoom APIを呼び出す様子です。特に、引数で送信されたマッチンググループは、マッチメイカーを論理的に分けることができる概念であると先ほども説明しました。仮に"Newbie"というマッチンググループを送信した場合、同じ"Newbie"マッチンググループ同士で同じマッチングキューを共有します。

```java
    /**
     * クライアントがroomMatchをリクエストした場合に発生するコールバック
     *
     * @param roomTypeクライアントとサーバー間にあらかじめ定義したルームの種類を区別する任意の値
     * @param matchingGroupクライアントがリクエストしたマッチンググループ
     * @param matchingUserCategory該当ユーザーが属するルーム内のカテゴリー
     * @param payloadクライアントから送信されたペイロード
     * @return {@link MatchRoomResult}によってmatchingしたroomの情報を返す。nullを返した場合は、クライアントリクエストオプションによって新しいRoomが作成されるか、リクエストが失敗処理される。
 * @throws SuspendExecution このメソッドはファイバーがsuspendになることがある。
     */
    public MatchRoomResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final Payload payload) throws SuspendExecution {
        
        GameRoomMatchForm gameRoomMatchForm = new GameRoomMatchForm(RoomMode.NORMAL, innerPayload.getOption(), 0);
        
        return matchRoom(matchingGroup, roomType, gameRoomMatchForm);
    }
```
