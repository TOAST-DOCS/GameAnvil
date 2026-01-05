## Game > GameAnvil > Unity 基礎開発ガイド > バックグラウンド接続切れ防止

## バックグラウンド接続切れ防止

モバイルデバイスでゲームがバックグラウンドに切り替わると、Unityのアプリケーションが停止します。アプリケーションが停止するとゲームサーバーとパケットをやり取りできなくなり、この状態が続くと接続確認のためのパケットもやり取りできなくなるため、結局サーバーとの接続が切れる可能性があります。

GameAnvilManagerには、ゲームがバックグラウンドに切り替わっても接続が切れるのを防ぐために、アプリがバックグラウンドに切り替わる時にサーバーの接続確認機能を一定時間停止するようになっています。この時間はpauseClientStateCheckTime値を利用して調整でき、デフォルト値は600(秒)です。 
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/07-pause/01-pause-client-check.png)

### 再接続

pauseClientStateCheckTime設定した時間が過ぎると、サーバーの接続確認機能が再び動作するようになり、接続が切れる可能性があります。このような場合には再度簡易ログインを行う必要があります。
