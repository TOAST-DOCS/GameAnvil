## Game > GameAnvil > Unity基礎開発ガイド > GameAnvilコネクタの終了

## GameAnvilコネクタの終了

ゲームプレイを終了する前にGameAnvilConnector.Disconnect()関数を呼び出して接続を終了することを推奨します。終了しない場合、サーバーがクライアントの終了を認識しない可能性があり、この場合、不要な動作を継続する可能性があります。
GameAnvilConnectorのOnDestroy()にその機能が含まれています。

```c#
private void OnDestroy()
{
    if (connector.IsConnected())
    {
        Disconnect();
    }
}
```
