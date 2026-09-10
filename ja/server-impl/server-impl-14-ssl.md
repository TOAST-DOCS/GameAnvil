<!-- pre-align:aligned sig=5d1a4121264a -->

<a id="game-gameanvil-server-development-guide-ssl-support"></a>
## Game > GameAnvil > サーバー開発ガイド > SSLサポート { #game-gameanvil-server-development-guide-ssl-support }



<a id="ssl-support"></a>
## SSLサポート { #ssl-support }

ゲートウェイノードとサポートノードは、パブリックネットワークに公開されるノードです。そのため、この2つのノードはSSL(secure socket layer)をサポートします。

<a id="set-up-ssl"></a>
## SSL設定 { #set-up-ssl }

SSLは基本的にGameAnvilConfigを通じてセキュリティ設定を行います。認証キーのルートパスはVMオプションで変更できます。

<a id="gatewaynode"></a>
### GatewayNode { #gatewaynode }

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



<a id="supportnode"></a>
### SupportNode { #supportnode }

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



<a id="vm-option"></a>
### VM Option { #vm-option }

認証キーのルートパスを設定できるように、次のようなVMオプションを提供します。

<a id="dsecure"></a>
### -Dsecure { #dsecure }

認証キーのルートパスのデフォルト値は、プロジェクト内のresourcesディレクトリです。もしプロジェクト外部、つまりjarバイナリの外にある認証情報を参照したい場合は、このVMオプションを使用します。

```
-Dsecure=/cert_dir/secure
```
