## Game > GameAnvil > In-depth Development Guide to Unity > Get Started

## Get Started
GameAnvilConnector makes it easy to use the various features provided by GameAnvil.

## Install GameAnvilConnector

You can include GameAnvilConnector in your project using gameanvil-connector.unitypackage. First download the gameanvil-connector.unitypackage [here](https://static.toastoven.net/prod_gameanvil/files/v2_1/gameanvil-connector.unitypackage). And in the menu, select \*\*Assets > Import Package > Custom Package...\*\*.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/01-import.png)

<br>

And select the downloaded package file to install the package.

![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/02-select-package.png)
![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/03-files.png)
<br>

When you install the package, the GameAnvilConnector DLL files are installed as follows in the GameAnvil folder:


![img.png](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/01-install/04-gameanvil.png)
<br>

DLL files are made in C# and can be used on platforms such as Android, iOS, and PC.


## Create GameAnvilConnector
You must use GameAnvilConnector to connect to the GameAnvil server. You can simply create and use GameAnvilConnector as follows:
The content of GameAnvilManager covered in the [Basic Development Guide to Unity > Manager](../unity-basic/unity-basic-02-gameanvil-manager.md) also uses GameAnvilConnector internally.  
```c#
using GameAnvil;

GameAnvilConnector connector = new GameAnvilConnector();
```

## GameAnvilConfig
GameAnvilConfig defines several settings to be used when running GameAnvilConnector. These settings are defined and set to the default value defined in GameAnvilConfig, but if needed, you can directly change the value of GameAnvilConfig to use.

```c#
using GameAnvil;

GameAnvilConfig.DefaultRequestTimeoutMillis = 3000;
GameAnvilConfig.PacketTimeoutMillis = 5000;
GameAnvilConfig.PingIntervalMillis  = 3000;
GameAnvilConfig.UseIPv6 = false;
GameAnvilConfig.UseSocketNoDelay = true;
```

The following are the types of settings:

| Type | Name | Description |
|------|-----------------------------|-----------------------------------------------------------------------------------------------------|
| int | DefaultRequestTimeoutMillis | Set the default timeout waiting time (unit : milliseconds, default: 3,000) |
| int | PacketTimeoutMillis | If the packet is not updated within the specified time, it is judged that the connection is lost<br/>Must be set higher than the pingInterval value<br/>(Unit: milliseconds, default: 5,000) |
| int | PingIntervalMillis | Set the interval of sending Ping messages to confirm the connection to the server (unit: milliseconds, default: Not used for 3,000, 0 days |
| bool | UseIPv6 | Convert to IPv6 address when connecting (default: false) |
| bool | UseSocketNoDelay | Use Nodelay for socket (default: true) |
## GameAnvilLogger

GameAnvilConnector does not log directly but passes logs through callbacks. You must register a callback to GameAnvilLogger to receive logs that occur from GameAnvilConnector as follows:

```c#
using GameAnvil;

GameAnvilLogger.LogListener += Debug.Log; // register Unity Debug.Log
GameAnvilLogger.LogListener += (logMessage)=>{
    // process log messages
}
```

GameAnvilLogger has the following attributes that allow you to set the type of logs to be exported:

| Type | Name | Description |
|----------|--------------------|------------------------------------------------|
| LogLevel | Threshold | Set the level of the log to be output (default: Info) |
| bool | EnablePingPongLogs | Set whether to output logs derived from PingPong (default: false) |
