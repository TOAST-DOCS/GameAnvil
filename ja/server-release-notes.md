## Game > GameAnvil > リリースノート



### 1.0.5 (2020.11.17)

#### Fix

- RoomMatchReqでエラーを返した時、パケットヘッダの復元(restore)ができない問題を修正



### 1.0.4 (2020.10.29)

#### Fix

- 任意のSessionにまだパケットが渡されたことがない場合は、このSessionにsend呼び出し時、NPE(Null Pointer Exception)が発生する問題を修正
- publishToClient使用時、問題発生(Noticeを含む)
- Roomで使用する繰り返しTimerによりルームの破棄が遅延する問題を修正
- protobuf構造体を変数として持っているClassをdeserializeする時に例外が発生する問題を修正
- java.lang.UnsupportedOperationExceptionが発生



### 1.0.3 (2020.10.26)

#### Change

- 以前、スペックから除外していた告知機能APIを再度有効化



### 1.0.2 (2020.10.12)

#### New

- Logに表示されるNode情報をユーザーが自由に設定できる機能を追加
- HostIdを直接入力できる機能を追加
- nodeInfoByHostIdからHostIdごとに情報を送る機能を追加

#### Fix

- IP 0.0.0.0をバインディングする時にエラーが発生する問題を修正
- ConfigModuleの /config/get使用時、成否値が失敗で返される問題を修正
- GameNodeが起動している状態で他のGameNodeを起動する場合、 Publish Packet処理によりエラーが発生することがある問題の例外処理を追加

#### Change

- UserId,RoomIdの作成時、最後の桁に使用するShardIndex値の範囲を指定できるように修正
- RestObjectのgetRemoteAddress値を追加
- NodeKeyリクエストの数を指定できるように修正



### 1.0.1 (2020.09.07)

### Fix

- onLogin()でfalseを返す場合、Payloadがクライアントに伝達できない問題を修正
- DebugとTraceログ文の前のログレベル確認条件文の不足していた部分を追加



### 1.0.0 (2020.08.31)

#### New

- 性能と安定性が大幅に向上(0.9バージョンに比べ約7倍)
- 性能に直接的な影響を与える多数のロジックを最適化し、問題点を修正
- 完全に新しくなったGameNodeのロードバランシング
- クラウドVMの状態を反映して、速く正確に最適なGameNodeに負荷を分散
- Integer型IDを導入
- String型だったIDを全てIntegerに変更し、最適化を進行
- サーバーLogレベルをランタイムに変更できるAPIを追加
- Node単位でサービス中にLogレベルを変更可能

#### Fix

- ユーザー転送問題を修正、使用方法の改善と最適化
- GameAnvilユーザーは、さらに簡単にユーザー転送を使用可能
- 無停止メンテナンスパッチの完全修正
- これまでルームの転送と無停止メンテナンスパッチが正常に動作しなかった問題が全て修正
- サーバー起動時に不明なタイミングの問題で一部のNodeが正常に起動しなかった問題を修正
- Nodeの一部のTimerオブジェクトが解放されず、継続的に累積していたMemory Leakを修正
- クライアント接続チェックプロセスで新たに接続するユーザーが意図せず切断されていた問題を修正

#### Change

- Node名を以下の表のように直感的に変更

| v1.0       | ~ v0.10       | 必須 | 機能                                    | コンテンツ実装 | ネットワークアクセス |
| ---------- | ------------- | ---- | ------------------------------------------- | ----------- | ----------------- |
| Gateway    | Session       | 必須 | クライアント接続と認証を処理           | 可能    | public            |
| Game       | Space         | 必須 | 実際のゲームサーバーとしてコンテンツを処理        | 可能    | private           |
| Support    | Service       | 任意 | 必要に応じて独立したサービスとして実装するようにサポート | 可能    | private or public |
| Match      | Match         | 任意 | マッチメイキングを実行                          | 可能    | private           |
| Location   | Location      | 必須 | ユーザーとルームなどの位置情報を保存し、管理 | 不可能  | private           |
| Management | Management    | 必須 | サーバーの情報を収集し、Admin/Agentと通信    | 不可能  | private           |
| Ipc        | Communication | 必須 | GameflexサーバーのInter-process通信処理 | 不可能  | private           |

- GameAnvilコードの可読性とユーザビリティの向上
- パッケージの構造を効率的に改善
- クラス継承構造の改善
- ネーミングを直感的で一貫性があるものに修正
- Nodeの無効化状態はInvalidとDisableに細分化
- Invalidは一時的に反応しない状態
- Disableは完全に無効になっている状態
- Loopbackアドレスは自動的にバインディング
- GameAnvilで使用するすべてのライブラリを最新バージョンにアップデート
- 全てのErrorCodeを1つに統合
- 不要なReConnectとMoveService機能を削除
- 登録時点ではなく、起動時点から次の時間を測定する新しいTimerを追加



### 0.10.2 (2020.04.08)

#### New

- GameAnvil APIレファレンス(JavaDoc)サイトオープン

- AOT Instrumentationサポート

- エンジンをランタイムに毎回JIT Instrumentationする必要なくコンパイルタイムにAOTで進行

- 公式GCでG1GCを使用するようにガイドを開始

- 最も基本的なVMオプション

  `java -javaagent:QUASAR_PATH\quasar-core-0.7.10-jdk8.jar=bm -Xms6g -Xmx6g -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseStringDeduplication`

#### Fix

- Quasarスタックバグを修正
- ルームマッチングユーザーが別々のルームにマッチングされるバグを修正
- 圧縮されたパケットが転送されない問題を修正



### 0.10.1 (2020.02.11)

#### New

- ユーザーに提供するツールを集めたUtilクラスを提供
- エンジンの複数の場所に散らばっていたツールを1つのクラスに整理

#### Change

- Topic処理コードの性能改善



### 0.10.0 (2020.02.06)

#### New

- 問題が多かった非同期サポートAPIを新たにリファクタリング
- Redis Clusterサポートと、Lettuce公式サポートに伴う、新たな非同期APIを追加
- 非同期HttpRequestとHttpResponse APIを追加
- ByteStringベースのカスタムプロトコル機能をサポート
- Google protobuf以外のユーザーが希望する方式でメッセージをシリアル化可能

#### Fix

- 以前のバージョンで最も深刻な問題だったファイバーの状態が壊れる問題を全て修正

#### Change

- エンジンで使用するパケットとヘッダサイズの最適化
- 0.9バージョンに比べ性能が約2.5倍向上
- Embedded Redis Spec-out
- インフラで提供するサービスを使用するように案内



### 0.9.9 (2019.10.25)

#### New

- プロセス単位でWatchDogを連動できるようにポートを追加で開放

#### Fix

- ゴーストユーザーのバグを修正
- SpaceNodeで発生するConcurrent Modificationエラーを修正
- NodeInfoManagerのComparatorエラーを修正
- ログアウトしたユーザーのセッションオブジェクトがクリーンアップされない問題を修正
- マッチンググループが別々のチャンネルのユーザーに正常に適用されない問題を修正
- パーティマッチングメイキング時に、Timeoutのコールバックを正常に受け取れなかった問題を修正

#### Change

- ルームマッチメイキングをすでに進行中の状態で重複リクエストがあった場合のエラーコードを追加
- MATCH_ROOM_FAIL_IN_PROGRESS



### 0.9.8 (2019.09.05)

#### New

- SessionにClientTopicを追加、削除できるaddTopic(),removeTopic() APIを追加
- Packet TTL機能を追加
- Dangling Locationを定期的にチェックして整理するロジックを追加

#### Fix

- AdminからユーザーをKickする場合、クライアントにForceLogoutNotiを送らないように修正
- LocationNodeのSpot誤動作を修正

#### Change

- onAuthenticate()コールバック失敗時に、そのソケットを閉じる前にForceCloseNotiをクライアントに送るように変更



### 0.9.7 (2019.07.27)

#### New

- 異常に残っているRoomを定期的にチェックし、整理するロジックを追加
- ユーザーマッチメイキングは、リクエストをした後に再ログインをしても以前のマッチリクエストがキャンセルされず、進行中の場合があるが、このような以前のユーザーマッチメイキング処理状態を伝えるために再ログインレスポンスに新しいフラグを追加
- Packet Expire可能追加(デフォルト値30秒)

#### Fix

- Connection ID(CID)が重複する問題を修正
- Disconnectionしたユーザーによりルームファイバーがcloseするバグを修正
- ログインが失敗の場合、レスポンスメッセージにserviceIdフィールドが正常にセッティングされない問題を修正
- コンテンツコールバック処理部のうち、try-catchが抜けていた部分を修正
- クライアントReconnectが約3秒以上かかっていた問題を、すぐに処理されるように修正
- ゴーストユーザーチェック時に発生するスレッド同時性問題を修正
- ルームマッチメイキング重複処理バグを修正
- ルームに接続が切れたユーザーが残る問題を修正

#### Change

- Session Disconnectのすべての場合にログを追加
- CreateRoomは、現在のノードにすぐルームを作成するように変更



### 0.9.6 (2019.06.23)

#### New

- 転送可能な時点をコンテンツから調整できるようにUserとRoomにcanTransfer()インターフェイスを追加

#### Fix

- エンジン内部コードに存在していたメモリリークを修正
- ZMQ Router,Publiserソケットの接続バグを修正
- Loginが無限に繰り返される問題を修正
- isLogined() APIが正常に動作しない問題を修正
- NodeInfoManagerの同期の問題を修正
- ユーザーオブジェクトが正常にクリーンアップされない問題を修正

#### Change

- AsyncAwaitHttpRequestをFiberHttpRequestに名前変更
- FiberHttpRequestとHttpRequestに非同期関数を追加
- Location Index関連ログを追加
- SpaceNodeの分配方式をidleからroundRobinに変更
- NodeStarterでLocationNodeがある場合にのみsleepするように変更



### 0.9.5 (2019.06.21)

#### Fix

- 無停止メンテナンスパッチの時に新しいユーザーがログインするか、ユーザー転送中の場合、Pauseしたノードで該当ユーザーが整理されないエラーを修正

#### Change

- 文字列型のserviceIdを整数型に変更
- プロトコルのservice_codeをservice_idに変更し、関連コードを修正
- RESTfulサポートAPIの内部実装と使用方法を変更



### 0.9.4 (2019.05.24)

#### New

- コンテンツで使用するプロトコルをファイル単位で登録できる機能を追加
- 内部パケット処理を効率的にするため
- MoveService機能追加

#### Fix

- ルームマッチメイキングで使用する比較演算子(Comparator)のエラーを修正

#### Change

- Authenticationレスポンスに最近のログイン情報を追加して、次のログイン時に活用可能
- setReady()を明示的に呼び出してonReady状態になるように変更
- LocationNodeがCommunicationNodeのonPrepare時点で初期化されるように変更



### 0.9.3 (2019.04.10)

#### Fix

- AsyncAwait.call() APIでtimeoutが発生すると、NPE(Null Pointer Exception)も発生する問題を修正
- Epollが有効になると止まる問題を修正
- ログイン時に発生するInvalid System Target Locationエラーを修正

#### Change

- ManagementNodeは、エンジンユーザーが拡張して使用できなくなります。
- 代わりにServiceNodeを使用
- AdminからMachine情報の登録、修正、削除をリクエストした瞬間に適用されるように変更



### 0.9.2 (2019.02.11)

#### New

- Adminを利用した予約告知、予約メンテナンス機能を追加
- Adminを利用したWhite IP,White User,White Device機能を追加

#### Fix

- AsyncAwait.run() APIで誤った方法で例外が処理されるコードを修正
- ManagementNodeが他のNodeより先に初期化されるように修正
- チャンネルIDを使用しない場合、一部チャンネル関連publishが抜け落ちる問題修正
- CreateRoom,JoinRoomについてコンテンツが失敗を返す場合に、誤ったエラーコードがヘッダに入力される問題を修正
- パケットヘッダの作成時に発生していたメモリリークを修正
- ルームマッチメイキングで発生していたメモリリークを修正

#### Change

- onLogout()コールバックにpayloadを追加
- ResultCodeの修正に伴い、いくつかのプロトコルで重複フィールドを削除
- Dynamic Moduleユーザビリティの改善



### 0.9.1 (2018.11.12)

#### New

- マッチメイキングを担当するMatchNodeを追加
- ユーザーマッチメイキングにrefill機能を追加
- ユーザーマッチメイキングの新しいタイプ「パーティマッチメイキング」を追加
- AdminのMachine情報でノードの種類ごとに付加情報を追加
- ユーザーが同じDeviceIdとUserIdで再ログインする時に呼び出されるonLoginByOtherConnection()コールバックを追加

#### Fix

- Dynamic Moduleバグを修正
- Admin接続ができない問題を修正
- Userのreplyバグを修正
- Timer追加/削除エラーを修正
- AsyncAwait.run()に抜けていたtimeoutを適用
- ノードShutdown時に発生していたエラーを修正
- ルームマッチメイキングを使用していないのもかかわらず、マッチング情報を削除してNPE(Null Pointer Exception)が発生する問題を修正

#### Change

- 一部インターフェイスの位置変更
- 別々のDeviceIdでログインした時に、重複ログイン処理の代わりに再ログイン処理を行うように変更
- ルームマッチメイキングのonMatchRoom()コールバックに追加したpayloadが以降の流れに含まれるように修正



### 0.9.0 (2018.06.27)

#### New

- ユーザーマッチメイキング機能を追加
- チャンネルごとにユーザーとルームリスト機能を追加
- Embedded Redisを追加
- Reconnect機能を追加
- (旧) Test Agentを追加

#### Fix

- Account単位の重複ログイン処理問題を修正

#### Change

- OracleJDKからAdpotOpenJDKに変更
- Node(スレッド)単位でZMQ通信していたのを、プロセス単位に変更



### 0.8.6 (2018.06.28)

#### New

- Send from Session to Management機能を追加
- Request from Session to Management機能を追加
- User MatchMakerを追加
- Channel User List機能を追加
- Custom Serializer機能を追加
- RoomFinderにCustom Serializerをplug-in可能

#### Fix

- チャンネルユーザーマネージャーでチャンネルがtopicに使用されるpublishの空白文字列の例外処理を追加
- チャンネルがtopicに使用されるpublishの空白文字列の例外処理を追加
- AutoJoinRoomを使用しない環境でdelNamedRoomInfo()とdelRoomInfo()を呼び出した時にDefaultRoomFinderにより発生する問題を修正
- RoomFinderでnullチェック未実施が原因で発生するNullPointerExceptionを修正
- ユーザーマッチメイキングでtimeoutしたリクエストが削除されない問題を修正
- redisにpasswordが設定されている場合の処理を追加
- cfgManagementのsetRestIp()エラーを修正
- ServiceノードでHttpハンドリングができない問題を修正。
- ブランチ：0.8.6.2_fix_service_node_rest_handling
- タグ：tardis-0.8.6.2.1
- AutoJoinRoom処理時にgameIdが設定されている場合のエラーを修正
- MatchMaker修正およびサンプルを補強
- ISession::onDisconnect()の抜けを修正
- Communication Node間Node情報交換エラーを修正
- FindRoomList性能問題を解決
- Shutdownバグを修正
- BaseNodeがpublishToNodeメッセージを受け取れない問題を修正
- FindRoomList性能問題を解決
- Shutdownバグを修正
- BaseNodeがpublishToNodeメッセージを受け取れない問題を修正
- NamedAutoJoinRoomバグを修正
- UpdateRoomInfo/DelRoomInfoがそのNodeですぐに反映されるように修正
- ファイバー内でblocking callを呼び出す部分により出力されていたErrorを修正

#### Change

- FindRoomListは全てのルームリストcontainerを一度にserialize/deserializeするように修正
- 既存のLocationException,TardisExceptionなどをRuntimeExceptionに変更、Exceptionを整理
- forceLeaveRoomをkickOutRoomにリネーム
- forceLogoutをkickOutにリネーム
- tardisConfig構成を変更



### 0.8.5 (2017.12.18)

#### New

- JWT認証トークンを使用したHTTPセッション認証および管理機能を追加
- 同じID使用して、複数のServiceに重複ログインできる機能を追加
- 1つのGameNodeに複数のRoomTypeを使用できる機能を追加
- Serverのすべての情報をWebで確認できる機能を追加(NodeInfoPage)

#### Fix

- タイマーの削除ができないバグを修正
- onPostLeaveRoomが2回呼び出されるバグを修正

#### Change

- ゴーストユーザーの処理を改善
- ゴーストユーザーのロギングを追加
- Packet compressのロジックを改善
- RestObject改善、使用する関数を追加
- パケットのエラー処理を変更、プロトコルヘッダにエラーコードを入れて処理



### 0.8.4 (2018.01.10)

#### New

- Bootstrap適用
- NGB Bootstrapのような形で実装
- 実装部のそれぞれのクラスのclassインスタンスのみを明示してコードを簡素化
- 複数のメッセージを送受信できるMulti Dispatch機能を追加
- ServiceNodeSender(以前のCustomServiceSender) publishToLobbyメソッドを追加

#### Change

- AsyncAwait,RAsyncAwaitの名前を変更
- Bootstrap RoomFinder設定をspaceに移動
- Cache,Custom Exceptionの名前を変更(Location,Service)
- PayloadクラスでgetFirstメソッドを使用して最初のPacketを取得できるように修正
- ServiceNodeSenderのrequestToLobbyメソッドを削除(同じ機能のrequestToNodeメソッドと重複する問題)
- PacketクラスのmakeDecompressメソッドを呼び出す時、asyncを使用しないように修正
- UserSenderのreplyメソッドオーバーローディング(Packetリスト処理)
- 一部のクラス名を変更
- AsyncCall -> AsyncWaitingCall
- AsyncRun -> AsyncWaitingRun
- ReverseAsyncCall -> RAsyncWaitingCall
- ReverseAsyncRun -> RAsyncWaitingRun

#### Fix

- Session IPエラーを修正



### 0.8.3 (2017.10.26)

#### New

- RESTハンドリング機能を追加
- TardisConfig.jsonにpermissionGroupsでRESTハンドリングに、Path & IPベースのACL機能を実装
- Channel List (チャンネル名リスト),Channel Information(特定チャンネルの詳細情報)
- Space LobbySenderにrequestToCustom, sendToCustomメソッドを追加
- SpaceノードManagement情報に"State"項目を追加
- Ilobby ,ISessionLobby,IServiceインターフェイスに以下のvoid onShutdown() throws SuspendExecutionを追加
- Nodeがshutdownする時、contentsでresourceを解放できるようにonShutdown()インターフェイスを追加
- ServiceNodeSenderにpublishToNode関数を追加

#### Fix

- ManagementのJMX Management APIの修正と追加(SessionGateway, CustomGateway)
- Channel List(チャンネル名リスト)、Channel Information(特定チャンネルの詳細情報)機能を追加
- Session,CustomノードのRESTハンドリングロジックを修正
- Gatewayノード(Session, Custom) isConnectableTypeメソッドの戻り値を修正
- SessionGatewayおよびCustomGatewayラウンドロビン問題とJMXリターン情報を修正
- キャッシュノードtargetMapキー値を使用するコードを修正
- SessionGatewayノードREST RoundRobinコードを修正
- ログインプロセスを改善(Invalid Target Locationエラーログ関連の修正)
- ルームを削除するdelRoomInfoを呼び出した時、そのノードの状態がREADYでなければリターン処理

#### Change

- SessionノードとクライアントのEnd-Pointを1つに統合(Session Gateway)
- CustomノードのREST End-Pointを1つに統合(Custom Gateway)
- ファイバー直接使用コードをReverseAsyncCallに変更
- SpaceSingular Dispatchタイムスタンプを追加
- onTimerを登録し、定期的にチェック(10分)
- タイムスタンプを超えたユーザーは、internalLogout処理
- Asyncタイムアウトを適用
- ゴーストユーザー処理ロジックの追加と改善、TardisConfig.jsonに必要な値を追加
- 各Responseプロトコルのresult_code形式をinteger値からenumに変更し、種類を簡素化。is_payloadフィールドを削除
- クラス、パッケージ、関数とcfg内容をリネーム
- Request,Responseプロトコルで複数のpayloadを処理できるようにclass PayloadデータタイプをPacketに変更、class OutPayloadを削除
- ユーザーTransfer-Inインターフェイスのパラメータタイプをbyte配列に変更



### 0.8.2 (2017.09.21)

#### New

- CustomServiceにRestObject機能を追加
- CustomServiceにRestHandlerを追加

#### Fix

- Cacheからユーザー情報が削除されないバグを修正
- エンジン内部で発生していたNPE(Null Pointer Exception)の例外処理を追加
- パケットの転送にCID(Connection ID)を正しく適用できていなかった問題を修正

#### Change

- Customモジュールのネーミングを変更
- 例) CustomLobby → CustomService
- Customモジュールインターフェイスを一部変更
- CustomServiceSenderのsendToUser()とrequestToUser() APIにserviceId引数を追加
