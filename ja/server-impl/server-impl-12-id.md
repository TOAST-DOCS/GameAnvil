## Game > GameAnvil > サーバー開発ガイド > IDの使用方法

## ID

GameAnvilはさまざまな種類のIDを使用しています。その一部はサーバーが発行し、その他の一部はユーザーがGameAnvilConfigに直接設定します。接続に必要なアカウント情報などは、クライアントが入力した情報をサーバーに送信します。次はGameAnvilで使用する代表的なIDについての説明です。

| 名前    | 説明                                                                                                                               | データ型 | 範囲    |
| --------- |------------------------------------------------------------------------------------------------------------------------------------| ------ | --------- |
| ServiceId | 設定したそれぞれのサービスを区別するためのID - 1つのサービスIDは複数のノードで構成できる                                                                        | int    | 0<id< 100 |
| HostId    | ホストの固有ID - 1つのホストに複数のGameAnvilプロセスを実行する場合はGameAnvilConfigに別途のvmIdを設定                                              | long   | -         |
| NodeId    | ノードの固有ID - HostId + ServiceId + 内部カウンター値で構成                                                                                  | long   | -         |
| AccountId | クライアントが接続した時に入力 - コネクション1つにつき1つのアカウントIDがマッピング                                                                                         | string | -         |
| UserId    | ゲームユーザーオブジェクトの固有ID - ゲームユーザーオブジェクトが作成された時にサーバーが発行 - 同じユーザーが再接続した場合でも、新しいゲームユーザーオブジェクトが作成されると新しいIDを発行                                      | int    | -         |
| RoomId    | ルームの固有ID - ルームオブジェクトが作成された時にサーバーが発行                                                                                                   | int    | -         |
| SubId     | 1つのアカウント(AccountId)内で固有の補助IDとしてクライアントが接続した時に送信する値<br>1つのコネクション内で複数のセッションを区別するために使用し、セッションの固有IDはAccountIdとSubIdを組み合わせて作成する | int    | 0 < id    |

### IDサポートAPI

前述したIDに関する一部の機能を次の表のとおりにエンジンユーザーに提供します。該当IDを取得、確認するためには必ず以下のAPIを使用する必要があります。

#### ServiceId API

| メソッド                              | 説明                                 |
| ------------------------------------- | -------------------------------------- |
| boolean isValid(int serviceId)        | 有効なserviceIdであるかを確認            |
| int findServiceId(String serviceName) | ServiceNameに該当するServiceIdを取得 |
| String findServiceName(int serviceId) | ServiceIdに該当するServiceNameを取得 |

#### HostId API

| メソッド                     | 説明                   |
| ---------------------------- | ------------------------ |
| boolean isValid(long hostId) | 有効なhostIdであるかを確認 |
| long get()                   | プロセスのhostIdを取得 |

#### NodeId API

| メソッド                      | 名前                        |
| ----------------------------- | ----------------------------- |
| boolean isValid(long nodeId)  | 有効なnodeIdであるかを確認      |
| long getHostId(long nodeId)   | nodeIdからhostIdを取得   |
| int getServiceId(long nodeId) | nodeIdからserviceIdを取得 |
| int getNodeNum(long nodeId)   | nodeIdからnodeNumを取得  |

#### UserId API

| メソッド                    | 説明                 |
| --------------------------- | ---------------------- |
| boolean isValid(int userId) | 有効なuserIdであるかを確認 |

#### RoomId API

| メソッド                    | 説明                 |
| --------------------------- | ---------------------- |
| boolean isValid(int roomId) | 有効なroomIdであるかを確認 |

#### SubId API

| メソッド                   | 説明                |
| -------------------------- | --------------------- |
| boolean isValid(int subId) | 有効なsubIdであるかを確認 |
