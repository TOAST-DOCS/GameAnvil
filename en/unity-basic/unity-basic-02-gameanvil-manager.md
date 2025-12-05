## Game > GameAnvil > Basic Development Guide to Unity > Manager

## GameAnvilManager

GameAnvilManager is responsible for default settings and agent management, and you can configure or register callbacks to view logs related to internal behaviour. To use GameAnvilManager, you must add GameAnvilManager to the scene first.

### Create

You can create immediately by right-clicking **GameAnvil > GameAnvilManager** in the Unity Hierarchy window.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/02-gameanvil-manager/01-add-gameanvil-manager.png)

Or you can create an empty GameObject and add a GameAnvilManager component.

### Settings

GameAnvilManager has several settings values. When creating GameAnvilManager, it is set to the default value, but if needed, you can change the value directly in the Inspector window.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/02-gameanvil-manager/02-gameanvil-manager-inspector.png)

The following are the types of settings:

| Category | Name | Description | Defaults |
|----------------|-------------------------------|-------------------------------------------------------------------------------------------------------------------------|:---------:|
| Connect | Ip | IP address of the server to connect to in Easy Login | 127.0.0.1 |
| | Port | Port address of the server to connect to in Easy Login | 18200 |
| Authentication | AccountId | User ID to use in Easy Login | test |
| | DeviceId | Unique device value to use in Easy Login | test |
| | Password | Password to use in Easy Login | test |
| Login | User Type | User type to use in Easy Login | |
| | Channel Id | Channel ID to use in Easy Login | |
| | Service Name | Service name to use in Easy Login | |
| Client Check | Pause Client State Check Time | Set the time to pause client state check. <br/>When switching to the background, the client connection status check is paused. During this time, the app will not be forcibly terminated by the server even if it comes to the foreground after remaining in the background. | 600 |
| Log | Threshold | Set the log level. | Info |

GameAnvilManager is now prepared for use.