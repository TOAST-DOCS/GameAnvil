## Game > GameAnvil > Unity基礎開発ガイド > バックグラウンド接続の切断を防止

## バックグラウンド接続の切断を防止

モバイル端末でゲームがバックグラウンドに切り替わる場合、Unityのアプリケーションが停止します。アプリケーションが停止すると、GameAnvilConnectorのUpdate()を呼び出す過程が行われず、ゲームサーバーとパケットを送受信できなくなります。この状態が続くと、接続確認のためのパケットも送受信できなくなるため、最終的にサーバーとの接続が切断される可能性があります。

特定の時間の間、ゲームがバックグラウンドに切り替わっても接続が切れることを望まない場合は、ConnectionAgentのPauseClientStateCheck(), ResumeClientStateCheck()を使用してサーバーの接続確認機能を一時停止させることができます。

サーバーの接続確認の一時停止時間はGameAnvilConnectorのpauseClientStateCheckTime値を利用して調節できます。

詳細は、[Unity深層開発ガイド > バックグラウンド接続切断防止](../unity-advanced/unity-advanced-06-background-connection.md)を参照してください。

### 再接続

PauseClientStateCheck()に入力した時間が過ぎると、サーバーの接続確認機能が再び動作し、接続が切断されることがあります。この場合、再接続を行う必要があります。

再接続に関する詳細は[Unity深層発ガイド > 再接続](../unity-advanced/unity-advanced-07-reconnect.md)で確認できます。
