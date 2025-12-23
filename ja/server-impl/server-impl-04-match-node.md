## Game > GameAnvil > サーバー開発ガイド > マッチノード実装

## MatchNodeとMatchMaker

![MatchNode on Network.png](https://static.toastoven.net/prod_gameanvil/images/node_matchnode_on_network.png)

ユーザーは簡単にマッチングロジックだけ実装してMatchNodeを有効化しマッチメイキングを適用できます。マッチメーカーはマッチングロジックに基づいてユーザーを任意のルームへ入室させてくれます。GameAnvilは2種類のマッチメーカーを提供します。それぞれユーザー単位のマッチングを実行するUserMatchMakerとルーム単位のマッチングを実行するRoomMatchMakerです。

このようなマッチメーカーはMatchNodeで独立して稼働します。この時、MatchNodeはマッチメイキングを実行する用途以外に追加のコンテンツを実装できません。したがってユーザーはMatchNodeではなくマッチメーカーにのみ集中すればよいです。ただし、マッチメーカーを実行するためには最低1つ以上のMatchNodeを有効化する必要があります。

> [参考]
>
> マッチメーカーはマッチンググループ単位で生成されます。つまり、同一のマッチンググループ同士でマッチメイキングを実行します。
>
> MatchNodeは必須ノードではないため、マッチメイキングを使用しない場合には稼働する必要がありません。

## マッチンググループ(MatchingGroup)

マッチンググループもチャンネルと同様に単一サーバー群を論理的に分けることができる方法の1つです。ただし、マッチンググループはチャンネルと異なり明確にあらかじめ設定しておいて使用する値ではありません。またチャンネルはGameNodeを論理的に分けるための方法である反面、マッチンググループはマッチメイキングを論理的に分けるための方法です。同じマッチンググループでユーザーマッチングまたはルームマッチングをリクエストした場合、マッチングをリクエストしたチャンネル内でマッチングされたルームが生成されます。

先ほどゲームユーザーで見てきたユーザーマッチメイキングコールバックメソッドであるonMatchUserをもう一度見てみます。コールバックメソッドの文字列引数matchingGroupはクライアントがユーザーマッチメイキングをリクエストする時に一緒に伝達した値です。つまり、マッチンググループはクライアントとサーバー間であらかじめ約束された任意の文字列です。その種類や個数はユーザーが望むだけ使用できます。全てのマッチメイキングリクエストはそれぞれのマッチンググループ単位でキューが管理され、該当キュー内でのみ互いにマッチングが可能です。例えば以下の例で引数matchingGroupとして「初心者」という値が伝達された場合、このリクエストは他の「初心者」リクエストたちとマッチングされることが保証されます。

```java
/**
 * クライアントからユーザーマッチメイキングをリクエストした場合に呼び出されるコールバック
 *
 * @param roomType         クライアントとサーバー間で事前定義したルーム種類を区分する任意の値
 * @param matchingGroupマッチンググループ
 * @param payload          クライアントから受け取った{@link IPayload}
 * @param outPayload       クライアントへ伝達する{@link IPayload}
 * @return戻り値がtrueならユーザーマッチメイキングリクエスト成功、falseなら失敗
 */
@Override
public boolean onMatchUser(final String roomType, final String matchingGroup, final IPayload payload, IPayload outPayload) {

    SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId(), rating);
    
    return matchUser(matchingGroup, roomType, userMatchInfo, payload);
}
```

マッチンググループは「初心者」、「中級者」、「上級者」のように実力をベースに定義することもでき、「韓国」、「日本」、「アメリカ」のように国別に定義することもできます。つまり、ユーザーが望むどんな値もマッチンググループになり得ます。

## ユーザーマッチメーカー実装

ユーザーマッチメイキングはゲームユーザーのマッチングリクエストをキューに積載します。特定時間周期でこのリクエストキューの内容を比較、分析してユーザーが望む基準で任意のユーザーを1つのルームに入室させてくれます。ここでユーザーはリクエストキューの内容をどのように比較して分析しどんな基準でユーザーをマッチングするかに対するロジックにのみ集中すればよいです。ちなみに最も代表的なユーザーマッチメイキングゲームは<リーグ・オブ・レジェンド>があります。

### ユーザーマッチリクエスト実装

このようなユーザーマッチメイキングの最も基本はまさにマッチングリクエストそのものです。このようなマッチリクエストを以下のようにエンジンで提供するAbstractUserMatchInfo抽象クラスを継承して実装します。この時、リクエスト者を区分できるゲームユーザーのIDを提供できるようgetId()メソッドは必ず実装しなければなりません。またリクエストはいつでもシリアライズできなければならないためSerializableインターフェースを実装する必要があります。以下の例はマッチングリクエスト間の比較のためにComparableインターフェースを追加で実装しています。

```java
public class SampleUserMatchInfo extends AbstractUserMatchInfo implements Comparable<SampleUserMatchInfo> {
    private int userId;
    private int rating;

    public SampleUserMatchInfo(int userId, int rating) {
        this.userId = userId;
        this.rating = rating;
    }

    public int getRating() {
        return rating;
    }

    /**
     * リクエスト主体のID返却
     * <p/>
     * パーティーマッチメイキングリクエストの場合ルームID、ユーザーマッチメイキングリクエストの場合ユーザーIDを返却
     *
     * @return パーティーマッチメイキングリクエストの場合ルームID、ユーザーマッチメイキングリクエストの場合ユーザーIDを返却
     */
    @Override
    public int getId() {
        return userId;
    }

    /**
     * パーティーサイズ返却
     * <p/>
     * パーティーマッチメイキングリクエストの場合パーティーのサイズ(人数)、ユーザーマッチメイキングリクエストの場合0を返却
     *
     * @return パーティーマッチメイキングリクエストの場合パーティーのサイズ(人数)、ユーザーマッチメイキングリクエストの場合0返却
     */
    @Override
    public int getPartySize() {
        return 0;
    }

    // もし SampleUserMatchInfo オブジェクト間に比較が必要なら Comparable インターフェースを実装します。
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

ユーザーマッチリクエストで再定義すべきメソッドを表で整理すると次のとおりです。

| コールバック名        | 意味          | 説明                                                                                                                                                 |
|--------------|-------------|-------------------------------------------------------------------------------------------------------------------------------|
| getId        | マッチリクエスト者情報   | 該当ユーザーマッチリクエストがどのユーザーから来たものか判断するために使用されます。したがって必ずリクエスト者のIDを返すように再定義しなければなりません。                                                           |
| getPartySize | マッチリクエストパーティー規模 | リクエストパーティーのサイズを返します。この値でパーティーマッチメイキングリクエストかどうかを判断するため、ユーザーマッチメイキングリクエストの場合には必ず0を返すように再定義します。パーティーマッチリクエストの場合には該当パーティーメンバーの人数を返すようにします。 |

### ユーザーマッチメーカー

ユーザーマッチメーカーはユーザーマッチリクエストを実際に処理し、エンジンで提供するAbstractUserMatchMaker抽象クラスを継承実装します。特にonMatch()メソッドは実際のマッチングを実行するために呼び出されるコールバックなので注意深く見てください。onRefill()メソッドはすでに完了したマッチメイキングに対して補充リクエストを処理するコールバックです。例えば4名がマッチメイキングされた状態で1名がゲームを終了した時、1名をさらに補充するために使用できます。以下のサンプルコードはこのようなユーザーマッチメーカーを実装する方法を示しています。

```java
@GameAnvilUserMatchMaker(loadClass = SampleRoom.class) // SampleRoomに登録されたMatchMakerクラス
public class SampleUserMatchMaker extends AbstractUserMatchMaker<SampleUserMatchInfo> {

    public SampleUserMatchMaker() {
        super(2, 5000);
    }

    private int matchSize = 2;

    /**
     * ユーザー/パーティーマッチメイキングリクエストを処理
     * <p>
     * {@link IUser#onMatchUser(String, String, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)}で {@link IUserContext#matchUser(String, String, AbstractUserMatchInfo)}を呼び出すか、
     * {@link IRoom#onMatchParty(String, String, IUser, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)}で {@link IRoomContext#matchParty(String, String, AbstractUserMatchInfo)}を呼び出せばマッチメイキングを処理できる
     * <p>
     * 最初の呼び出し以降、定期的に呼び出される
     * <p>
     * {@link #getMatchRequests(int)}を利用してマッチをリクエストしたユーザー/パーティーのUserMatchInfoリストを取得する
     * <p>
     * このリストを利用して条件に合うリクエストを集め、{@link #matchSingles(Collection)}、{@link #assignRoom(AbstractUserMatchInfo)}、{@link #assignRoom(Collection)}を利用して同じルームへ入れる
     * <p>
     * 条件に合うリクエストの数が足りない場合、次の呼び出し時に処理
     */
    @Override
    public void onMatch() {
        List<SampleUserMatchInfo> matchRequests = getMatchRequests(matchSize);

        if (matchRequests == null) {
            return;
        }

        int matchingAmount = matchSingles(matchRequests);
    }

    /**
     * ユーザー/パーティーマッチングで作られたルームからユーザーが退室した場合、新しいユーザーを割り当て
     * <p>
     * {@link IRoom#canLeaveRoom(IUser, com.nhn.gameanvil.packet.IPayload, com.nhn.gameanvil.packet.IPayload)} で {@link IRoomContext#matchRefill(AbstractUserMatchInfo)} を呼び出すと呼び出され新しいユーザーを補充できる
     * <p>
     * ユーザー/パーティーマッチメイキングリクエストがある場合、まず一度 {@link #onRefill(AbstractUserMatchInfo)}が呼び出され、リフィルが必要なルームからユーザーを補充できる
     * <p>
     * ユーザー/パーティーマッチメイキングリクエストごとに最初の一度だけ {@link #onRefill(AbstractUserMatchInfo)}が呼び出され、定期的に呼び出されない
     * <p>
     * リフィルで使用されなかったリクエストは {@link #getMatchRequests(int)}のリストに残り {@link #onMatch()}で継続使用される
     * <p>
     * {@link #getRefillRequests()} を利用してリフィルをリクエストしたルームのUserMatchInfoリストを取得する
     * <p>
     * このリストで条件 req に合うリクエストを探し {@link #refillRoom(AbstractUserMatchInfo, AbstractUserMatchInfo)}を利用してルームへ入れる
     *
     * @param req ユーザー/パーティーマッチをリクエストしたユーザーまたはパーティーのUserMatchInfo
     * @return 戻り値が true ならリフィル成功、false なら失敗
     */
    @Override
    public boolean onRefill(SampleUserMatchInfo matchReq) {
        return false;
    }
}
```

特にマッチメーカーを実装したクラスは特定サービスにエンジンに登録します。またマッチメーカーにより生成されるルームのタイプを事前に定義します。この時、1つのマッチメーカークラスはただ1つのサービスに対してのみ登録できます。

このようなユーザーマッチメーカーのコールバックメソッドを整理すると以下の表の通りです。

| コールバック名    | 意味          | 説明                                                                                                                                                                                                                                                                                                                                  |
|----------|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| onMatch  | マッチリクエストを処理 | ユーザーはAbstractUserMatchMakerが提供するAPIを使用して直接マッチリクエストを処理できます。この時、ユーザーが独自のロジックを直接実装できます。サンプルコードの処理フローが最も基本的な方式です。<br>
つまり、getMatchRequests APIを利用して最小限のマッチリクエストを獲得した後、任意の人数に合わせてリクエストを組み合わせ、任意のCollectionに順番に入れます。このCollectionをmatchSingles APIに引数として渡すと定員数に合わせてマッチが行われます。サンプルコードの場合、定員が2名のユーザーマッチメイキングなのでCollectionを巡回しながら順番に2名ずつ抽出して1つのゲームとしてマッチングさせます。
| onRefill | マッチリフィルリクエスト処理 | ユーザー/パーティーマッチメイキング過程で任意のユーザーが出た場合に新しいユーザーを補充するための処理をします。一般的にマッチメイキングされたルームに対してonLeaveRoomが呼び出される時にmatchRefillを呼び出して連動できます。つまり、マッチングされたルームから誰かが出る時にリフィルをリクエストするのです。リフィルはキューに積まれているマッチリクエストは使用しません。リフィルリクエスト以後に入ってくる新しいマッチリクエストのみその対象とします。                                                                                                                                                                                          |

### GameUserからマッチメーカーへリクエスト伝達する

これでクライアントはサーバーへユーザーマッチメイキングをリクエストできます。このリクエストはGameUserに伝達された後、エンジンによりonMatchUserコールバックメソッドを呼び出します。これについては先ほどGameNodeとGameUserを説明しながら一度見てきました。ユーザーはこのコールバックメソッドでGameAnvilが提供するユーザーマッチメーカーを使用してもよく、直接実装した別のマッチメーカーや他のソリューションを使用しても構いません。しかし特別な理由がなければGameAnvilのユーザーマッチメイキングを使用することを推奨します。

以下の例はエンジンで提供するユーザーマッチメーカーを使用するために先ほど見てきたマッチリクエストを生成してmatchUser APIを呼び出す様子です。特に、パラメータとして伝達されたマッチンググループはマッチメイキングを論理的に分けることができる概念だと先ほど説明しました。もし「Newbie」というマッチンググループを伝達したなら、同一の「Newbie」マッチンググループ同士で同じマッチングキューを共有することになります。 

```java
/**
 * クライアントからユーザーマッチメイキングをリクエストした場合に呼び出されるコールバック
 *
 * @param roomType         クライアントとサーバー間で事前定義したルーム種類を区分する任意の値
 * @param matchingGroupマッチンググループ
 * @param payload          クライアントから受け取った{@link IPayload}
 * @param outPayload       クライアントへ伝達する{@link IPayload}
 * @return戻り値がtrueならユーザーマッチメイキングリクエスト成功、falseなら失敗
 */
@Override
public boolean onMatchUser(final String roomType, final String matchingGroup, final IPayload payload, IPayload outPayload) {

    SampleUserMatchInfo sampleUserMatchInfo = new SampleUserMatchInfo(getUserId(), rating);

    return matchUser(matchingGroup, roomType, userMatchInfo, payload);
}
```

最後にクライアントはいつでも先ほどリクエストしたユーザーマッチメイキングをキャンセルできます。この時、エンジンはキャンセル処理が正常に進めば次のようにGameUserのonMatchUserCancelコールバックメソッドを呼び出します。ユーザーはこのコールバックでキャンセルのタイミングに処理したい部分を実装すればよいです。

```java
/**
 * ユーザーマッチメイキングがキャンセルされる時に呼び出し
 *
 * @param reasonキャンセルされた理由。タイムアウト(TIMEOUT)、ユーザーのリクエストによるキャンセル(CANCEL)、マッチノードの終了によるキャンセル(SHUTDOWN)
 */
@Override
public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
    
}
```

## ルームマッチメーカー実装

ルームマッチメイキングはユーザーを最も適切なルームへ自動入室させてくれる機能です。ルームマッチメイキングをリクエストしたユーザーをどんな基準でどのルームに入室させるかはユーザーの実装次第です。最もユーザー数が多いルームへ入室させることもでき、最も閑散としたルームへ入室させることもできます。あるいは平均点数が最も高いルームへ入室させることもできます。ユーザーはこのようなマッチングロジックにのみ集中すればよいです。ちなみに最も代表的なルームマッチメイキングゲームは<ハンゲームポーカー>や<カートライダー>などがあります。


> [参考]
>
> マッチメーカーはマッチンググループ単位で生成されます。つまり、同一のマッチンググループ同士でマッチメイキングを実行します。
>
> ルームマッチメーカーとユーザーマッチメーカーは互いに独立して運営されます。つまり、同一のマッチンググループでユーザーマッチングとルームマッチングをそれぞれリクエストしても、この二つのリクエストが一緒にマッチングされることはありません。

### ルームマッチリクエスト実装

このようなルームマッチメイキングの最も基本はまさにマッチングリクエストそのものです。マッチングリクエストは一名のユーザーが送ったリクエストを意味し、以下のようにエンジンで提供するAbstractRoomMatchForm抽象クラスを継承実装します。リクエストはいつでもシリアライズできなければならないためSerializableインターフェースを追加で実装する必要があります。次はこのようなマッチリクエストを実装した例です。

```java
public class SampleRoomMatchForm extends AbstractRoomMatchForm {
    public SampleRoomMatchForm() {
        super();
    }
}
```

ルームマッチリクエストは基本的にマッチングロジックで使用する情報を含みます。これはユーザーがマッチメイキングロジックを直接実装する時に使用することになります。ルームマッチリクエストで1つ重要な情報はマッチングユーザーカテゴリーです。マッチングユーザーカテゴリーは1つのルームでユーザーが属するグループを区分するための任意の文字列です。例えば4人定員のルームで二つのチームに分けて2 vs 2のゲームをする場合、それぞれのユーザーが属するチームを指定するための用途として使用できます。もし何の値も指定しなければエンジンのデフォルト値が使用されます。

### ルームマッチ情報実装

マッチング対象となるルームは、ルームマッチメーカーによりマッチ情報が管理されます。つまり、1つのルームマッチ情報は、1つのマッチング可能なルーム情報を意味すると考えられます。このとき、該当ルームの様々な情報や状態値を含めることができます。BaseRoomMatchInfoを継承して実装し、必須でルームのIDとマッチングユーザーカテゴリー、そしてマッチングカテゴリーごとの最大定員数を設定する必要があります。そして最後に、ルームマッチ情報もシリアライズのために 
必ずSerializableインターフェースを実装する必要があります。以下はコード例を示しています。

```java
public class SampleRoomMatchInfo extends AbstractRoomMatchInfo {
    private static final int MAX_ENTRY_USER = 4;

    public SampleRoomMatchInfo(int roomId) {
        super(roomId, MAX_ENTRY_USER);
    }
}
```

以下の例は、ユーザーが任意のマッチングユーザーカテゴリーを設定する場合です。1つのゲームルームにレッドチームとブルーチームがそれぞれ3名ずつで構成されています。このようにカテゴリーを分けておく場合、ルームマッチメーカーがそれぞれのマッチングカテゴリーごとのユーザー数を自動で管理してくれます。また、マッチメイキングに使用する追加情報を希望通りに構成できます。

```java
public class GameRoomMatchInfo extends AbstractRoomMatchInfo {
    private static final int MAX_USER = 3;

    // マッチング条件情報
    int mapId;
    int maxLevel;
    int avgLevel = 0;

    public GameRoomMatchInfo(int roomId) {
        // RED_TEAM最大人数3名で人数情報を追加します。
        super(roomId, "RED_TEAM", 3);

        // 必要に応じて追加マッチカテゴリーを登録します。
        // BLUE_TEAM最大人数3名で人数情報を追加します。
        addMatchingUserCategory("BLUE_TEAM", 3);
    }

    public void setAvgLevel(int avgLevel) {
        this.avgLevel = avgLevel;
    }
    
    ...
}
```

### ルームマッチ情報登録/更新

このようなルームマッチ情報は、ユーザーが直接登録/更新できます。つまり、ユーザーが望まない場合には、特定のルームはルームマッチメイキング対象として登録しないことも可能です。このような登録手続きは、一般的に次のようにルームが生成されるonCreateRoomコールバックメソッドで主に行います。

```java
/**
 * 新しいルームを生成する時に呼び出し
 * <p/>
 * 戻り値によってルームを生成するかどうかが決定される
 *
 * @param user       リクエストしたユーザーオブジェクト
 * @param inPayload  クライアントから受け取った{@link IPayload}
 * @param outPayloadクライアントへ伝達する{@link IPayload}
 * @return戻り値がtrueならルーム生成成功、falseなら失敗
 */
@Override
public final boolean onCreateRoom(final GameUser user, final IPayload payload, IPayload outPayload) {
    
    ...

    // 今後このルームはルームマッチメイキングに使用されます。
    registerRoomMatch(gameRoomMatchInfo, user.getUserId());
}
```

任意のルームに対してルームマッチメイキング登録可否を確認するため、IRoomContextインターフェースに次のようなAPIを提供します。

```java
/**
 * ルームマッチ登録可否を返却
 *
 * @return 戻り値がtrueであればルームマッチ登録、falseであれば未登録
 */
public boolean isRegisteredRoomMatch()
```

このように登録しておいたルームマッチ情報は、いつでも必要に応じてユーザーがその情報を更新することもできます。更新のため新しいルームマッチ情報を作成した後、updateRoomMatch APIを呼び出せばよいです。

```java
GameRoomMatchInfo gameRoomMatchInfo = new GameRoomMatchInfo(getId());
gameRoomMatchInfo.setMemberMoney(1000);

roomContext.updateRoomMatch(gameRoomMatchInfo); // このルームマッチ情報を更新します。
```

該当するルームが消滅する際、ルームマッチ情報はエンジンで自動的に削除されるため、ユーザーが別途削除する必要はありません。

### ルームマッチメーカー

次はルームマッチメーカーを作成する番です。ルームマッチメーカーはエンジンが提供するAbstractRoomMatchMaker抽象クラスを継承して実装します。ルームマッチメイキングは最も適切なルームを探す過程であるため、実際のマッチング前/後のための特別なコールバックメソッドが提供されます。ユーザーはこのコールバックメソッドを再定義して、希望通りにマッチングを実行できます。以下のコード例は、このようなルームマッチメーカーをどのように実装できるかを示しています。

```java
@GameAnvilUserMatchMaker(loadClass = SampleRoom.class) // SampleRoomに登録されたMatchMakerクラス
public class SampleRoomMatchMaker extends AbstractRoomMatchMaker<SampleRoomMatchForm, SampleRoomMatchInfo> {
    /**
     * ルームマッチメイキングリクエスト時に呼び出し
     * <p>
     * 登録されたルームの中でマッチング条件に合うルームを探し、マッチ成否を決定する
     *
     * @param baseRoomMatchForm ルームマッチメイキング申請条件である{@link AbstractRoomMatchForm}
     * @param baseRoomMatchInfo マッチングプールのルーム情報である{@link AbstractRoomMatchInfo}
     * @param args              追加で受け取ったパラメータ
     * @return 戻り値がtrueであればルームマッチメイキング成功、falseであればルームマッチメイキング失敗
     */
    @Override
    public boolean onMatch(SampleRoomMatchForm roomMatchForm, SampleRoomMatchInfo roomMatchInfo, Object... args) {
        return false;
    }

    /**
     * ソートのためのマッチメイキング情報比較
     * <p>
     * {@link com.nhn.gameanvil.node.game.context.IRoomContext#updateChannelRoomInfo(IChannelRoomInfo)}メソッド呼び出し、ルーム登録、ルーム削除、ソートメソッド呼び出し時に実装したcompareによりマッチングプールがソートされる
     *
     * @param o1 比較する1番目のパラメータ
     * @param o2 比較する2番目のパラメータ
     * @return 比較値を返却 (-1: 昇順、 0: 変動なし、 1: 降順)
     */
    @Override
    public int compare(SampleRoomMatchInfo o1, SampleRoomMatchInfo o2) {
        return 0;
    }
}
```

特にマッチメーカーを実装したクラスは、特定サービスのエンジンに登録します。また、マッチメーカーにより生成されるルームのタイプを事前に定義します。このとき、1つのマッチメーカークラスは、ただ1つのサービスに対してのみ登録できます。

先に確認したルームマッチメーカーのコールバックメソッドを整理すると、以下の表の通りです。

| コールバック名      | 意味                   | 説明                                                                                                                                                                     |
|------------|----------------------|--------------------------------------------------------------------------------------------------------------------------------|
| onMatch    | マッチ確認                | 引数として受け取ったルームマッチリクエストとルームマッチ情報の間にマッチングが可能か確認するために呼び出します。つまり、全ルームマッチプールを巡回しながら、該当ルームマッチリクエストが入る最も適切なルームを探すためのロジックをここに実装できます。 |
| compare    | ソートのためのマッチメイキング情報比較   | ソートのためのマッチメイキング情報を比較します。結果値は -1:昇順、0:変動なし、1:降順です。                                                                                              |


### GameUserからマッチメーカーへリクエスト伝達する

これでクライアントはサーバーへルームマッチメイキングをリクエストできます。このリクエストはGameUserに伝達された後、エンジンによりonMatchRoomコールバックメソッドを呼び出します。これについては先にGameNodeとGameUserを説明しながら一度確認しました。ユーザーはこのコールバックメソッドでGameAnvilが提供するルームマッチメーカーを使用してもよく、直接実装した別のマッチメーカーや他のソリューションを使用しても構いません。しかし特別な理由がない限り、GameAnvilのルームマッチメイキングの使用を推奨します。

以下の例は、エンジンが提供するルームマッチメーカーを使用するために、先ほど確認したマッチリクエストを生成してmatchRoom APIを呼び出す様子です。特に、パラメータとして渡されたマッチンググループは、マッチメイキングを論理的に分けることができる概念であると先に説明しました。もし「Newbie」というマッチンググループを渡した場合、同じ「Newbie」マッチンググループ同士で同じマッチングキューを共有することになります。

```java
/**
 * ルームマッチメイキングリクエストを受け取ると呼び出される
 *
 * @param roomType                クライアントとサーバー間で事前定義したルーム種類を区分する任意の値
 * @param matchingGroup           マッチングされるルームマッチンググループ伝達
 * @param matchingUserCategory マッチングされるマッチングユーザーカテゴリーを伝達
 * @param payload                 クライアントから受け取った{@link IPayload}
 * @return {@link RoomMatchResult}タイプでマッチングされたルームの情報を返す。nullを返す場合クライアントリクエストオプションに従って新しいルームが生成されるかリクエスト失敗処理
 */
@Override
public RoomMatchResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final IPayload payload) {

    SampleRoomMatchForm sampleRoomMatchForm = new SampleRoomMatchForm(RoomMode.NORMAL, innerPayload.getOption(), 0);

    return userContext.matchRoom(matchingGroup, roomType, sampleRoomMatchForm);
}
```
