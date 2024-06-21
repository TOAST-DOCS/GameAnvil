## Game > GameAnvil > Server Development Guide > Use IDs

## Identity (ID)

GameAnvil uses many different IDs. Some of them are issued by the server and the rest are configured by the user in GameAnvilConfig. The account information needed to connect passes the information received from the client to the server. Following is the description of the popular ID used by GameAnvil.

| Name      | Description                                                                                                                                   | Data type | Range      |
| --------- |--------------------------------------------------------------------------------------------------------------------------------------| ------ | --------- |
| ServiceId | The ID to distinguish each service you set up - one service ID can consist of multiple nodes                                                                           | int    | 0<id< 100 |
| HostId    | The unique ID of the host - if you are running multiple GameAnvil processes on one host, set separate vmIds in GameAnvilConfig                                                  | long   | -         |
| NodeId    | The unique ID of the node - consisting of HostId + ServiceId + internal counter value                                                                                      | long   | -         |
| AccountId | Enter when the client connects - one account ID maps per connection                                                                                             | string | -         |
| UserId    | The unique ID of the game user object - issued by the server when the game user object is created - a new ID is issued when a new game user object is created, even if the same user reconnects.                                         | int    | -         |
| RoomId    | The unique ID of the room - issued by the server when the room object is created.                                                                                                       | int    | -         |
| SubId     | A unique secondary ID within a single account (AccountId) that clients pass when they connect.<br>Used to distinguish between multiple sessions within a single connection. The session's unique ID is a combination of AccountId and SubId. | int    | 0 < id    |

### ID Support API

Some of the features related to the ID mentioned above are provided to engine users as shown in the list below. To obtain or confirm the ID, the API below must be used.

#### ServiceId API

| Method                                | Description                                    |
| ------------------------------------- | --------------------------------------- |
| boolean isValid(int serviceId)        | Checks if serviceId is valid               |
| int findServiceId(String serviceName) | Obtains the ServiceId corresponding to the ServiceName |
| String findServiceName(int serviceId) | Obtains the ServiceName corresponding to the ServiceId |

#### HostId API

| Method                       | Description                     |
| ---------------------------- | ------------------------ |
| boolean isValid(long hostId) | Checks for a valid hostId   |
| long get()                   | Obtains the hostId of the process |

#### NodeId API

| Method                        | Name                          |
| ----------------------------- | ----------------------------- |
| boolean isValid(long nodeId)  | Checks for a valid nodeId        |
| long getHostId(long nodeId)   | Obtains hostId from nodeId    |
| int getServiceId(long nodeId) | Obtains serviceId from nodeId |
| int getNodeNum(long nodeId)   | Obtains nodeNum from nodeId   |

#### UserId API

| Method                      | Description                   |
| --------------------------- | ---------------------- |
| boolean isValid(int userId) | Checks for a valid userId |

#### RoomId API

| Method                      | Description                   |
| --------------------------- | ---------------------- |
| boolean isValid(int roomId) | Checks for a valid roomId |

#### SubId API

| Method                     | Description                  |
| -------------------------- | --------------------- |
| boolean isValid(int subId) | Checks for a valid subId |
