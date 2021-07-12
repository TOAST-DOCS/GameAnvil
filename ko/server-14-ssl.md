## Game > GameAnvil > 서버 개발 가이드 > SSL 지원



## 1. SSL 지원

게이트웨이 노드와 서포트 노드는 공용망에 노출되는 노드들입니다. 그러므로 이 두 노드는 SSL(Secure Socket Layer)을 지원합니다.



## 2. SSL 설정하기

SSL은 기본적으로 GameAnvilConfig을 통해 보안 설정을 합니다. 인증키의 루트 경로는 VM 옵션으로 변경할 수 있습니다.

### 2.1. GatewayNode

다음과 같이 "secure" 설정을 통해 SSL 사용을 활성화할 수 있습니다. 만일 SSL을 사용하지 않을 경우에는 해당 키-값 쌍을 모두 삭제하도록 합니다.


```java
"gateway": {
	"connectGroup": {
		"TCP_SOCKET": {
    		"port": 18200,
    		// 보안 설정
			"secure": {
				"useSelfSignedCert": false,
				"keyCertChainPath": "gameanvil.crt", 	// 인증서 경로
				"privateKeyPath": "privatekey.pem" 		// 개인 키 경로
			}
		},
        
	    "WEB_SOCKET": {
			"port": 18300,
			"secure": {
				"useSelfSignedCert": false,
				"keyCertChainPath": "gameanvil.crt",	// 인증서 경로
				"privateKeyPath": "privatekey.pem"		// 개인 키 경로
			}
		}
	},
},
```

다음은 각 설정 값에 대한 설명입니다.

| 이름              | 설명                                                         | 기본값 |
| ----------------- | ------------------------------------------------------------ | ------ |
| useSelfSignedCert | 테스트를 위한 자체 인증을 사용할지 여부를 설정합니다. true이면 테스트용으로 인증서 없이도 SSL을 사용할 수 있습니다. 이 때, 다음의 keyCertChainPath, privateKeyPath 설정은 무시됩니다. | false  |
| keyCertChainPath  | 인증서 상대 경로 경로<br>-Dsecure 옵션을 사용하지 않을 경우, 루트 경로는 프로젝트 내의 resources/ 입니다. | -      |
| privateKeyPath    | 개인키 상대 경로<br/>-Dsecure 옵션을 사용하지 않을 경우, 루트 경로는 프로젝트 내의 resources/ 입니다. | -      |



### 2.2. SupportNode

서포트 노드도 게이트웨이 노드와 비슷하게 "restSecure" 설정을 통해 SSL 사용을 활성화할 수 있습니다. 만일 SSL을 사용하지 않을 경우에는 해당 키-값 쌍을 모두 삭제하도록 합니다.

```java
"support": [
	{
		...
		// 보안 설정
		"restSecure": {
			"useSelfSignedCert": false,
			"keyCertChainPath": "gameanvil.crt", 	// 인증서 경로
			"privateKeyPath": "privatekey.pem" 		// 개인 키 경로
		}
	}
],
```

다음은 각 설정 값에 대한 설명입니다.

| 이름              | 설명                                                         | 기본값 |
| ----------------- | ------------------------------------------------------------ | ------ |
| useSelfSignedCert | 테스트를 위한 자체 인증을 사용할지 여부를 설정합니다. true이면 테스트용으로 인증서 없이도 SSL을 사용할 수 있습니다. 이 때, 다음의 keyCertChainPath, privateKeyPath 설정은 무시됩니다. | false  |
| keyCertChainPath  | 인증서 상대 경로 경로<br>-Dsecure 옵션을 사용하지 않을 경우, 루트 경로는 프로젝트 내의 resources/ 입니다. | -      |
| privateKeyPath    | 개인키 상대 경로<br/>-Dsecure 옵션을 사용하지 않을 경우, 루트 경로는 프로젝트 내의 resources/ 입니다. | -      |



### 2.3. VM Option

인증키의 루트 경로를 설정할 수 있도록 다음과 같은 VM 옵션을 제공합니다.

### -Dsecure

인증키의 루트 경로에 대한 기본 값은 프로젝트 내의 resources 디렉토리입니다. 만일 프로젝트 외부, 즉 jar 바이너리 바깥의 인증 정보를 조회하고자 할 경우에는 이 VM 옵션을 사용하면 됩니다.

```
-Dsecure=/cert_dir/secure
```

