<!-- pre-align:aligned sig=5d1a4121264a -->

<a id="game-gameanvil-server-development-guide-ssl-support"></a>
## Game > GameAnvil > Server Development Guide > SSL Support { #game-gameanvil-server-development-guide-ssl-support }



<a id="ssl-support"></a>
## SSL Support { #ssl-support }

Gateway nodes and support nodes are nodes that are exposed to the public network; therefore, they support Secure Socket Layer (SSL).



<a id="set-up-ssl"></a>
## Set up SSL { #set-up-ssl }

SSL is secured by default through GameAnvilConfig. The root path for the authentication key can be changed with the VM options.

<a id="gatewaynode"></a>
### GatewayNode { #gatewaynode }

You can enable the use of SSL through the "secure" setting, as shown below. If you don't want to use SSL, be sure to delete all key-value pairs.


```java
"gateway": {
	"connectGroup": {
		"tcp_socket": {
    		"port": 18200,
    		// Security settings
			"secure": {
				"useSelfSignedCert": false,
				"keyCertChainPath": "gameanvil.crt", // Certificate path
				"privateKeyPath": "privatekey.pem" // private key path
			}
		}
        
	    "WEB_SOCKET": {
			"port": 18300,
			"secure": {
				"useSelfSignedCert": false,
				"keyCertChainPath": "gameanvil.crt", // Certificate path
				"privateKeyPath": "privatekey.pem" // private key path
			}
		}
	},
},
```

Here's a description of each setting value

| Name              | Description                                                         | Default value |
| ----------------- | ------------------------------------------------------------ | ------ |
| useSelfSignedCert | Sets whether to use self authentication for testing. If true, you can use SSL without a certificate for testing. In this case, the following keyCertChainPath and privateKeyPath settings are ignored. | false  |
| keyCertChainPath  | Certificate Relative Path Path<br>If the -Dsecure option is not used, the root path is resources/ within the project. | -      |
| privateKeyPath    | Private key relative path<br/>If the -Dsecure option is not used, the root path is resources/ within the project. | -      |



<a id="supportnode"></a>
### SupportNode { #supportnode }

Similar to gateway nodes, support nodes can enable SSL use via the "restSecure" setting. If SSL is not going to be used, make sure to delete all of the corresponding key-value pairs.

```java
"support": [
	{
		...
		// Security settings
		"restSecure": {
			"useSelfSignedCert": false,
			"keyCertChainPath": "gameanvil.crt", // certificate path
			"privateKeyPath": "privatekey.pem" // private key path
		}
	}
],
```

Here's a description of each setting value

| Name              | Description                                                         | Default value |
| ----------------- | ------------------------------------------------------------ | ------ |
| useSelfSignedCert | Sets whether to use self authentication for testing. If true, you can use SSL without a certificate for testing. In this case, the following keyCertChainPath and privateKeyPath settings are ignored. | false  |
| keyCertChainPath  | Certificate Relative Path Path<br>If the -Dsecure option is not used, the root path is resources/ within the project. | -      |
| privateKeyPath    | Private key relative path<br/>If the -Dsecure option is not used, the root path is resources/ within the project. | -      |



<a id="vm-option"></a>
### VM Option { #vm-option }

We provide the following VM options to help you set the root path for your authentication key.

<a id="dsecure"></a>
### -Dsecure { #dsecure }

The default value for the root path of the authentication key is the resources directory within the project. If you want to look up the credentials outside of the project, i.e. outside of the jar binary, you can use this VM option.

```
-Dsecure=/cert_dir/secure
```

