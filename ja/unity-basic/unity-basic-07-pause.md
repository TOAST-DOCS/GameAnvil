<!-- pre-align:aligned sig=d1684502910b -->

<a id="game-gameanvil-unity-basic-development-guide-preventing-background-disconnection"></a>
## Game > GameAnvil > Unity 基礎開発ガイド > バックグラウンド接続切れ防止 { #game-gameanvil-unity-basic-development-guide-preventing-background-disconnection }

<a id="prevent-background-disconnections"></a>
## バックグラウンド接続切れ防止 { #prevent-background-disconnections }

モバイルデバイスでゲームがバックグラウンドに切り替わると、Unityのアプリケーションが停止します。アプリケーションが停止するとゲームサーバーとパケットをやり取りできなくなり、この状態が続くと接続確認のためのパケットもやり取りできなくなるため、結局サーバーとの接続が切れる可能性があります。

GameAnvilManagerには、ゲームがバックグラウンドに切り替わっても接続が切れるのを防ぐために、アプリがバックグラウンドに切り替わる時にサーバーの接続確認機能を一定時間停止するようになっています。この時間はpauseClientStateCheckTime値を利用して調整でき、デフォルト値は600(秒)です。 
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/07-pause/01-pause-client-check.png)

<a id="reconnect"></a>
### 再接続 { #reconnect }

pauseClientStateCheckTime設定した時間が過ぎると、サーバーの接続確認機能が再び動作するようになり、接続が切れる可能性があります。このような場合には再度簡易ログインを行う必要があります。
