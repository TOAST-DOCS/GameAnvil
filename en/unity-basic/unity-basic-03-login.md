## Game > GameAnvil > Basic Development Guide to Unity > Quick Login

## Quick Login

The quick login feature provided by GameAnvilManager allows you to process the procedures for accessing, authenticating and logging in to the GameAnvil server at once. For more information about the procedures for accessing, authenticating, and logging in, see the [In-depth Development Guide to Unity > Connector](../unity-advanced/unity-advanced-02-connection.md) or Server Development Guide.

### Quick Login Settings

To use a quick login, you must pre-set the value to be used for the Login operation during the GameAnvilManager setting. The following types of settings are used for quick login:

| Category | Name | Description | Default |
|----------------|--------------|-------------------------|:---------:|
| Connect | Ip | IP of the server to connect to in Easy Login | 127.0.0.1 |
| | Port | Port of the server to connect to in Easy Login | 18200 |
| Authentication | AccountId | User identification ID to use in Easy Login | test | | | DeviceId | Device unique value to use in Easy Login | test | | | Password | Password to use in Easy Login | test | | Login | User Type | User type to use in Easy Login | | | | Channel Id | Channel ID to use in Easy Login | | | | Service Name | Service name to use in Easy Login | |

These settings can be set in the Inspector window of GameAnvilManager in the Unity editor or in the script code.

```c# public void Start() { GameAnvilManager gameAnvilManager = GameAnvilManager.Instance; gameAnvilManager.ip = managerIp; gameAnvilManager.port = managerPort; gameAnvilManager.accountId = managerAccountId; gameAnvilManager.deviceId = managerDeviceId; gameAnvilManager.password = managerPassword; gameAnvilManager.userType = managerUserType; gameAnvilManager.channelId = managerChannelId; gameAnvilManager.serviceName = managerServiceName; } ```

### Use Quick Login

Quick Login allows you to connect, authenticate, and log in to the server. You can check the result using the return value LoginResult. If there are problems with the connection to the server at this time, exceptions may occur.

```c# public async void ManagerLogin() { GameAnvilManager gameAnvilManager = GameAnvilManager.Instance; try { var result = await gameAnvilManager.Login(); if (result.loginResultCode == GameAnvilManager.LoginResultCode.SUCCESS){ // 성공 } else { // 실패 } } catch (Exception e) { // 서버 연결과 관련된 문제가 발생할 경우 예외 발생 // eg.) 연결 실패, 연결 끊김 등 } } ```

<br>
Depending on the server's implementation, additional information may be required during the server's authentication process or login process. In this case, you can send additional information to authenticatePayload or loginPayload. The information required for the authentication process is authenticatePayload, and the information required for the login process is forwarded to loginPayload.

``` c# public async void ManagerLogin() { GameAnvilManager gameAnvilManager = GameAnvilManager.Instance; try { var authenticatePayload = new Payload(new Protocol.AuthenticateData()); var loginPayload = new Payload(new Protocol.LoginData()); var result = await gameAnvilManager.Login(authenticatePayload, loginPayload); if (retult.loginResultCode == GameAnvilManager.LoginResultCode.SUCCESS){ // 성공 } else { // 실패 } } catch (Exception e) { // 서버 연결과 관련된 문제가 발생할 경우 예외 발생 // eg.) 연결 실패, 연결 끊김 등 } } ```

When the SMS login succeeds, the GameAnvilUserController instance of the user that has logged in to the LoginResult UserController is assigned. You can use this instance to access the user objects on the server. If the SMS login fails, the userController will be assigned null.
<br>

You can check the reason for the failure when the SMS login failed, using the loginResultCode for LoginResult. The following can be found for failure:

| Name | Description | | |------------------------------------|-----------------------------------------------|-------------------------------------------------------------| | START\_FAIL\_LOGIN\_IN\_PROGRESS | Login is already in progress. | | | START\_FAIL\_ALREADY\_LOGGED\_IN | You are already logged in. | | | CONNECT\_FAIL | Failed to connect to server. | |                    
| CONNECT\_FAIL\_INVALID\_IP | Server connection failed. Invalid IP. | |         
| CONNECT\_FAIL\_INVALID\_PORT | Server connection failed. Invalid Port. | |     
| AUTH\_FAIL\_CONTENT | Authentication failed. Authentication denied from the user code. | |               
| AUTH\_FAIL\_INVALID\_ACCOUNT\_ID | Authentication failed. Invalid AccountId. | Empty string, etc. | | AUTH\_FAIL\_SYSTEM\_ERROR | Authentication failed. Failure due to an unknown server error. | |    
| AUTH\_FAIL\_TIMEOUT | Authentication failed. The response to the request does not arrive within the set time period. | |   
| AUTH\_FAIL\_PARSE\_ERROR | Authentication failed. Message parsing for additional information failed. | This can occur if the protocol definition of the server and client is different. | | LOGIN\_FAIL\_CONTENT | Login failed. Login denied from the user code. | | | LOGIN\_FAIL\_NOT\_EXIST\_NODE | Login failed. The login target node does not exist. | This may occur if the server containing the target node is not booted while configuring the server. | | LOGIN\_FAIL\_INVALID\_SERVICE\_NAME | Login failed. The login target service does not exist. | If the service name is entered incorrectly or the service name is set incorrectly in the server settings, this may occur. | | LOGIN\_FAIL\_TIMEOUT\_GAME\_SERVER | Login failed. The request will not be responded within the specified time period. | | | LOGIN\_FAIL\_INVALID\_USER\_TYPE | Login failed. The userType to be logged in does not exist. | If the userType is entered incorrectly or is not implemented on the server, this may occur. | | LOGIN\_FAIL\_INVALID\_USER\_ID | Login failed. Creating UserId failed due to unknown reasons on the server. | | | LOGIN\_FAIL\_INVALID\_CHANNEL\_ID | Login failed. The login target channel does not exist. | If the ChannelId is entered incorrectly or the ChannelId is not present in the server settings, this may occur. | | LOGIN\_FAIL\_INVALID\_SUB\_ID | Login failed. Invalid SubId. | | | LOGIN\_FAIL\_LOGINED\_OTHER\_SERVICE | Login failed. Already logged in to another service with the same AccountId. | Occurred if you do not allow duplicate logs when configuring the server. | | LOGIN\_FAIL\_LOGINED\_OTHER\_CHANNEL | Login failed. Logged in to another channel with the same AccountId. | | | | LOGIN\_FAIL\_LOGINED\_OTHER\_USER\_TYPE | Login failed. You are logged in with the same AccountId and a different UserType. | When implementing the server, you can select between an existing user and a newly logged-in user. | | LOGIN\_FAIL\_LOGINED\_OTHER\_DEVICE | Login failed. You are logged in from another device with the same AccountId. | When implementing the server, you can select between an existing user and a newly logged-in user. | | LOGIN\_FAIL\_SYSTEM\_ERROR | Login failed. Failed due to an unknown server error. | | | LOGIN\_FAIL\_PARSE\_ERROR | Login failed. Message parsing for more information failed. | | | LOGIN\_FAIL\_TIMEOUT | Login failed. The request failed within the specified time period. | | | UNKNOWN\_ERROR | Failed due to an unknown server error. | |

More detailed reasons for failure can be found using the authenticationResult or loginResult in LoginResult.

## Disconnect

You can use GameAnvilManager's Logout() method to disconnect from the server.

When calling Logout(), you can process the process of logout, server connection termination, and callback calls to be processed after disconnection at one time. Depending on the server's implementation, additional information may be required during the logout process. In this case, you can pass additional information to logoutPayload.

```c# public async void ManagerLogout() { GameAnvilManager gameAnvilManager = GameAnvilManager.Instance; var logoutPayload = new Payload(new Protocol.LogoutData()); await gameAnvilManager.Logout(logoutPayload); } ```

## Notification for Status Change

You may receive notification that the status of GameAnvilManager may change, such as having a problem with the network because you are not calling Logout(), or being forcibly logged out of the server. ```c# public void addStateChangeListener() { gameAnvilManager.onStateChange.AddListener((GameAnvilManager.LoginState oldState, GameAnvilManager.LoginState newState) => { // Status Change Notification }); } ``` You can see the status before changing to oldState and after changing to newState.  