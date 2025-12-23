## Game > GameAnvil > サーバー開発ガイド > IDの使用方法

## ID

GameAnvilは様々な種類のIDを使用します。その一部はサーバーが独自に発行し、他のいくつかはユーザーがGameAnvilConfigに直接設定します。接続に必要なアカウント情報などは、クライアントで入力された
情報をサーバーへ伝達します。次はGameAnvilで使用する代表的なIDに関する説明です。

| 名前      | 説明                                                                                                                                 | データ型  | 範囲         |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------|--------|--------------|
| ServiceId | 設定した各サービスを区別するためのID。1つのサービスIDは複数のノードで構成可能                                                                                                       | int    | 0 < id < 100 |
| HostId    | ホストの固有ID - 1つのホストで複数のGameAnvilプロセスを駆動する場合には、GameAnvilConfigに別途vmIdを設定                                                                         | long   | 0 < id < 100 |
| NodeId    | ノードの固有ID - HostId + ServiceId + 内部カウンター値で構成                                                                                                                 | long   | -            |
| AccountId | クライアントが接続する際に入力 - コネクション1つにつき1つのアカウントIDがマッピング                                                                                                                          | string | -            |
| UserId    | ゲームユーザーオブジェクトの固有ID - ゲームユーザーオブジェクトが生成される際にサーバーが発行 - 同一ユーザーが再接続する場合でも、新しいゲームユーザーオブジェクトが生成されると新しいIDを発行                                                   | int    | -            |
| RoomId    | ルームの固有ID - ルームオブジェクトが生成される際にサーバーが発行                                                                                                                                  | int    | -            |
| SubId     | 1つのアカウント(AccountId)内で固有の補助IDとして、クライアントが接続する際に渡す値<br>1つのコネクション内で複数のセッションを区別するために使用し、セッションの固有IDはAccountIdとSubIdを組み合わせて生成する | int    | 0 < id       |

### IDサポートAPI

前述のIDに関する一部の機能を、以下の表のようにエンジンユーザーに提供します。該当するIDを取得または確認するためには、必ず以下のAPIを使用する必要があります。

#### 有効なID確認API
```java
/**
 * GameAnvilで使用するidの有効性検査のためのクラス
 */
public class GameAnvilIdValidator {
    /**
     * String IDが有効な値かを確認する
     *
     * @param id Stringタイプの確認するID
     * @return 戻り値がtrueなら有効、falseなら無効
     */
    public static boolean isValid(String id)

    /**
     * IDが有効かを確認する
     *
     * @param id long タイプの確認するノードID
     * @return 戻り値がtrueなら有効、falseなら無効
     */
    public static boolean isValid(long id)

    /**
     * IDが有効かを確認する
     *
     * @param id int タイプの確認するノードID
     * @return 戻り値がtrueなら有効、falseなら無効
     */
    public static boolean isValid(int id) 

    /**
     * サービスIDの範囲検査、0 < サービスID < 100
     *
     * @param serviceId サービスID
     * @return 引数で受け取ったサービスIDの有効性
     */
    public static boolean isValidOpenId(int serviceId)
}

```

#### 登録されたサービス確認API
```java
/**
 * サービスID < - > 名前を管理するクラス
 */
public enum ServiceInfoMap {
    INSTANCE;

    /**
     * サービスIDでサービス名を取得する
     *
     * @param serviceId サービスID
     * @return サービス名、存在しないIDの場合は空文字("")
     */
    public String getServiceName(int serviceId) 

    /**
     * サービス名でサービスIDを取得する
     *
     * @param serviceName サービス名
     * @return サービスID、存在しない名前の場合は-1
     */
    public int getServiceId(String serviceName) 
}
```
