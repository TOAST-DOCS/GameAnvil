## Game > GameAnvil > Unity 심화 개발 가이드 > 시작하기

## 시작하기
GameAnvilConnector는 GameAnvil이 제공하는 다양한 기능들을 간편하게 이용할 수 있도록 도와줍니다.

## GameAnvilConnector 설치

gameanvil-connector.unitypackage를 이용해 GameAnvilConnector를 프로젝트에 포함할 수 있습니다. 먼저 [여기](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector.unitypackage)에서 gameanvil-connector.unitypackage를 다운로드합니다. 그리고 메뉴의 **Assets > Import Package > Custom Package...** 를 선택합니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/01-import.png)

<br>

그리고 다운로드 받은 패키지 파일을 선택하여 패키지를 설치합니다.

![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/02-select-package.png)
![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/03-files.png)
<br>

패키지를 설치하면 다음과 같이 GameAnvil 폴더 아래에 GameAnvilConnector DLL 파일들이 설치됩니다.


![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/04-gameanvil.png)
<br>

DLL 파일들은 C#으로 만든 것으로 Android, iOS, PC 등의 플랫폼에서 모두 사용 가능합니다.


## GameAnvilConnector 생성
GameAnvil 서버와 연결하기 위해서는 반드시 GameAnvilConnector를 사용해야합니다. 다음과 같이 간단하게 GameAnvilConnector를 생성하여 사용할 수 있습니다.
[Unity 기초 개발 가이드 > 매니저](../unity-basic/unity-basic-02-gameanvil-manager.md)에서 다루는 GameAnvilManager도 내부적으로 GameAnvilConnector를 사용합니다.  
```c#
using GameAnvil;

GameAnvilConnector connector = new GameAnvilConnector();
```

## GameAnvilConfig
GameAnvilConfig에는 GameAnvilConnector 동작시 사용되는 몇가지 설정이 정의되어있습니다. 이 설정들은 정의 되어있으며, GameAnvilConfig에 정의되어있는 기본값으로 사용되설정되지만 필요하다면 GameAnvilConfig의 값을 직접 변경하여 사용할 수 있습니다.

```c#
using GameAnvil;

GameAnvilConfig.DefaultRequestTimeoutMillis = 3000;
GameAnvilConfig.PacketTimeoutMillis = 5000;
GameAnvilConfig.PingIntervalMillis  = 3000;
GameAnvilConfig.UseIPv6 = false;
GameAnvilConfig.UseSocketNoDelay = true;
```

설정의 종류는 다음과 같습니다.

| 타입   | 이름                          | 설명                                                                                                  |
|------|-----------------------------|-----------------------------------------------------------------------------------------------------|
| int  | DefaultRequestTimeoutMillis | 타임아웃 기본 대기시간 설정(단위 : 밀리 초, 기본값 : 3000)                                                              |
| int  | PacketTimeoutMillis         | 패킷이 지정된 시간안에 업데이트 되지 않으면 연결이 끊겼다고 판단<br/>pingInterval 값보다 보다 높게 설정해야 한다<br/>(단위 : 밀리 초, 기본값 : 5000) |
| int  | PingIntervalMillis          | 서버와의 연결을 확인하기 위해 Ping 메시지를 보내는 주기 설정 (단위 : 밀리 초, 기본값 : 3000, 0일 경우 사용 안함)                           |
| bool | UseIPv6                     | 접속시 IPv6 주소로 변환 여부 (기본값 : false)                                                                    |
| bool | UseSocketNoDelay            | 소켓의 Nodelay 사용 여부 (기본값 : true)                                                                      |
## GameAnvilLogger

GameAnvilConnector는 직접 로그를 남기지 않고 콜백을 통해 로그를 전달합니다. 다음과 같이 GameAnvilLogger에 콜백을 등록해야 GameAnvilConnector에서 발생하는 로그를 받을 수 있습니다.

```c#
using GameAnvil;

GameAnvilLogger.LogListener += Debug.Log; // 유니티 Debug.Log 등록
GameAnvilLogger.LogListener += (logMessage)=>{
    // 로그 메시지 처리
}
```

GameAnvilLogger에는 출력될 로그의 종류를 설정할 수 있는 다음과 속성이 있습니다.

| 타입       | 이름                 | 설명                                             |
|----------|--------------------|------------------------------------------------|
| LogLevel | Threshold          | 출력할 로그의 레벨을 설정(기본값 : Info)                     |
| bool     | EnablePingPongLogs | PingPong에 의해 발생하는 로그를 출력할지 여부를 설정(기본값 : false) |
