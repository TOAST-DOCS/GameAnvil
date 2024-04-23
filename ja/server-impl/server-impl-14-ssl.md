## Game > GameAnvil > サーバー開発ガイド > SSLサポート



## SSLサポート

ゲートウェイノードとサポートノードは共用ネットワークに表示されるノードです。したがって、この2つのノードはSSL(secure socket layer)をサポートします。



## SSLを設定する

SSLは基本的にGameAnvilConfigによってセキュリティ設定を行います。認証キーのルートパスはVMオプションで変更できます。

### GatewayNode

次のように「secure」設定でSSL使用を有効にできます。もしSSLを使用しない場合は、該当キー値のペアをすべて削除します。


```java
"gateway": {
	"connectGroup": {
		"TCP_SOCKET": {
    		"port": 18200,
    		// セキュリティ設定
			"secure": {
				"useSelfSignedCert": false,
				"keyCertChainPath": "gameanvil.crt", 	// 証明書パス
				"privateKeyPath": "privatekey.pem" 		// 秘密鍵パス
			}
		},
        
	    "WEB_SOCKET": {
			"port": 18300,
			"secure": {
				"useSelfSignedCert": false,
				"keyCertChainPath": "gameanvil.crt",	// 証明書パス
				"privateKeyPath": "privatekey.pem"		// 秘密鍵パス
			}
		}
	},
},
```

次は各設定値についての説明です。

| 名前            | 説明                                                                                                                      | デフォルト値 |
| ----------------- |---------------------------------------------------------------------------------------------------------------------------| ------ |
| useSelfSignedCert | テスト用の自己認証を使用するかどうかを設定します。trueの場合はテスト用に証明書がなくてもSSLを使用できます。この時、次のkeyCertChainPath, privateKeyPath設定は無視されます。 | false  |
| keyCertChainPath  | 証明書の相対パス<br>-Dsecureオプションを使用しない場合、ルートパスはプロジェクト内のresources/ です。                                                   | -      |
| privateKeyPath    | 秘密鍵の相対パス<br/>-Dsecureオプションを使用しない場合、ルートパスはプロジェクト内のresources/ です。                                                    | -      |



### SupportNode

サポートノードもゲートウェイノードと同じように"restSecure"設定により、SSL使用を有効にできます。もしSSLを使用しない場合は、該当キー値のペアをすべて削除します。

```java
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

次は各設定値についての説明です。

| 名前            | 説明                                                                                                                      | デフォルト値 |
| ----------------- |---------------------------------------------------------------------------------------------------------------------------| ------ |
| useSelfSignedCert | テスト用の自己認証を使用するかどうかを設定します。trueの場合はテスト用に証明書がなくてもSSLを使用できます。この時、次のkeyCertChainPath, privateKeyPath設定は無視されます。 | false  |
| keyCertChainPath  | 証明書の相対パス<br>-Dsecureオプションを使用しない場合、ルートパスはプロジェクト内のresources/ です。                                                   | -      |
| privateKeyPath    | 秘密鍵の相対パス<br/>-Dsecureオプションを使用しない場合、ルートパスはプロジェクト内のresources/ です。                                                    | -      |



### VM Option

認証キーのルートパスを設定できるように、次のようなVMオプションを提供します。

### -Dsecure

認証キーのルートパスに対するデフォルト値はプロジェクト内のresourcesディレクトリです。もしプロジェクト外部、すなわちjarバイナリ外の認証情報を照会したい場合は、このVMオプションを使用します。

```
-Dsecure=/cert_dir/secure
```
