## Game > GameAnvil > Unity 基礎開発ガイド > マネージャー

## GameAnvilManager

GameAnvilManagerは基本設定とエージェント管理を担当し、内部動作に関連するログを確認できるようにオプションを設定したりコールバックを登録したりできます。GameAnvilManagerを使用するには、まずシーンにGameAnvilManagerを追加する必要があります。

### 生成

Unity Hierarchyウィンドウでマウスの右ボタンをクリックした後、**GameAnvil > GameAnvilManager**を選択してすぐに生成できます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/02-gameanvil-manager/01-add-gameanvil-manager.png)

または空のGameObjectを生成し、GameAnvilManagerコンポーネントを追加することもできます。

### 設定

GameAnvilManagerには様々な設定値があります。GameAnvilManager生成時にデフォルト値として設定されますが、必要であればInspectorウィンドウで直接値を変更できます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/02-gameanvil-manager/02-gameanvil-manager-inspector.png)

設定の種類は以下のとおりです。

| 区分           | 名前                          | 説明                                                                                                                    |  デフォルト値  |
|----------------|-------------------------------|-------------------------------------------------------------------------------------------------------------------------|:---------:|
| Connect        | Ip                            | 簡易ログインで接続するサーバーのip                                                                                                                                                                      | 127.0.0.1 |
|                | Port                          | 簡易ログインで接続するサーバーのport                                                                                                                                                                    |   18200   |
| Authentication | AccountId                     | 簡易ログインで使用するユーザー識別ID                                                                                                                                                                   |   test    |
|                | DeviceId                      | 簡易ログインで使用するデバイス固有値                                                                                                                                                                      |   test    |
|                | Password                      | 簡易ログインで使用するパスワード                                                                                                                                                                        |   test    |
| Login          | User Type                     | 簡易ログインで使用するユーザータイプ                                                                                                                                                                       |           |
|                | Channel Id                    | 簡易ログインで使用するチャンネルID                                                                                                                                                                      |           |
|                | Service Name                  | 簡易ログインで使用するサービス名                                                                                                                                                                      |           |
| Client Check   | Pause Client State Check Time | クライアント状態チェック確認停止時間設定。<br/>バックグラウンド切替時にクライアントの接続状態チェックを一時停止する。この間はアプリがバックグラウンドに留まってからフォアグラウンドに戻っても、サーバーから強制終了させない。 |    600    |
| Log            | Threshold                     | ログレベルを設定します。                                                                                                           |   Info    |

これでGameAnvilManagerの使用準備が完了しました。
