## Game > GameAnvil > Unity 基礎開発ガイド > 接続終了

## 接続終了

ゲームプレイ終了前にGameAnvilConnector.Disconnect()関数を呼び出して接続を終了することを推奨します。終了しない場合、サーバーでクライアントの終了を認知できない可能性があり、この場合不必要な動作を継続する可能性があります。

```c#
public class GameAnvilManager : MonoBehaviour
{
    ...
    
    private void OnDestroy()
    {
        if (connector.IsConnected())
        {
            Disconnect();
        }
    }
    
    ...
}
```
