## Game > GameAnvil > Unity深層開発ガイド > バックグラウンド接続の切断防止

[Unity基礎開発ガイド > バックグラウンド接続の切断防止](../unity-basic/unity-basic-08-background-connection.md)文書の内容のように、モバイルデバイスでゲームがバックグラウンドに切り替わる場合、サーバー接続が切断されることがありますが、これを防止するためにサーバーとの接続確認機能を一時停止できます。

これを実装したコード例は次のとおりです。

```c#
public class ConnectHandler : MonoBehaviour
{
    ...
    private void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            // アプリがpauseされる前にやるべき作業があればここで処理します。

            // 入力した時間(sec)の間、サーバーのclientStateCheck機能を停止します。
            // この時間が過ぎるとclientStateCheck機能が動作して接続が切れることがあります。
            connector.GetConnectionAgent().PauseClientStateCheck(600);

            // アプリがpauseされる直前にconnector.Update()を呼び出して
            // Connectorに溜まったメッセージを処理して状態をアップデートします。
            connector.Update();
        } else
        {
            // アプリがresumeされた直後にconnector.Update()を呼び出して
            // Connectorに溜まったメッセージを処理して状態をアップデートします。
            connector.Update();

            // サーバーのclientStateCheck機能を再開します。
            connector.GetConnectionAgent().ResumeClientStateCheck();

            // 長時間pauseされた後、resumeされた場合、接続が切断される可能性があるので、状態をチェックします。
            if (connector.IsConnected())
            {
                // アプリがresumeされた後、やるべき作業があればここで処理します。
            }
        }
    }
    ...
}
```
