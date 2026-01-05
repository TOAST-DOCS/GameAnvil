## Game > GameAnvil > サーバー開発ガイド > SSLサポート



## SSLサポート

ゲートウェイノードとサポートノードは、パブリックネットワークに公開されるノードです。そのため、この2つのノードはSSL(secure socket layer)をサポートします。

## SSL設定

SSLは基本的にGameAnvilConfigを通じてセキュリティ設定を行います。認証キーのルートパスはVMオプションで変更できます。

### GatewayNode

次のように 'secure' 設定を通じてSSL使用を有効化できます。もしSSLを使用しない場合は、該当するキー-値ペアを全て削除します。


```json
"gateway": {
    "connectGroup": {
        "TCP_SOCKET": {
            "port": 18200,
            // セキュリティ設定
            "secure": {
                "useSelfSignedCert": false,
                "keyCertChainPath": "gameanvil.crt",    // 証明書パス
                "privateKeyPath": "privatekey.pem"      // 秘密鍵パス
            }
        },
        "WEB_SOCKET": {
            "port": 18300,
            "secure": {
                "useSelfSignedCert": false,
                "keyCertChainPath": "gameanvil.crt",	// 証明書パス
                "privateKeyPath": "privatekey.pem"      // 秘密鍵パス
            }
        }
    },
},
```

次は各設定値に関する説明です。

| 名前              | 説明                                                                                                                       | デフォルト値 |
|-------------------|----------------------------------------------------------------------------------------------------------------------------|-------|
| useSelfSignedCert | テストのための自己署名を使用するかどうかを設定します。trueの場合、テスト用として証明書なしでもSSLを使用できます。このとき、次のkeyCertChainPath、privateKeyPath設定は無視されます。  | false |
| keyCertChainPath  | 証明書の相対パス<br>-Dsecureオプションを使用しない場合、ルートパスはプロジェクト内のresources/です。                                                                                       | -      |
| privateKeyPath    | 秘密鍵の相対パス<br/>-Dsecureオプションを使用しない場合、ルートパスはプロジェクト内のresources/です。                                                                                          | -      |



### SupportNode

サポートノードもゲートウェイノードと同様に "restSecure" 設定を通じてSSL使用を有効化できます。もしSSLを使用しない場合は、該当するキー-値ペアを全て削除します。

```json
"support": [
    {
        ...
        // セキュリティ設定
        "restSecure": {
            "useSelfSignedCert": false,
            "keyCertChainPath": "gameanvil.crt", 	// 証明書パス
            "privateKeyPath": "privatekey.pem" 		// 秘密鍵パス
        }
    }
],
```

次は各設定値に関する説明です。

| 名前              | 説明                                                                                                                        | デフォルト値 |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------|-------|
| useSelfSignedCert | テストのための自己署名を使用するかどうかを設定します。trueの場合、テスト用として証明書なしでもSSLを使用できます。このとき、次のkeyCertChainPath、privateKeyPath設定は無視されます。   | false |
| keyCertChainPath  | 証明書の相対パス<br>-Dsecureオプションを使用しない場合、ルートパスはプロジェクト内のresources/です。                                                                                        | -      |
| privateKeyPath    | 秘密鍵の相対パス<br/>-Dsecureオプションを使用しない場合、ルートパスはプロジェクト内のresources/です。                                                                                           | -      |



### VM Option

認証キーのルートパスを設定できるように、次のようなVMオプションを提供します。

### -Dsecure

認証キーのルートパスのデフォルト値は、プロジェクト内のresourcesディレクトリです。もしプロジェクト外部、つまりjarバイナリの外にある認証情報を参照したい場合は、このVMオプションを使用します。

```
-Dsecure=/cert_dir/secure
```
