<!-- pre-align:aligned sig=eeed95e57cb0 -->

<a id="game-gameanvil-server-development-guide-ids"></a>
## Game > GameAnvil > Server Development Guide > IDs { #game-gameanvil-server-development-guide-ids }

<a id="identity-id"></a>
## Identity (ID) { #identity-id }

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

<a id="id-support-api"></a>
### ID Support API { #id-support-api }

Some of the features related to the ID mentioned above are provided to engine users as shown in the list below. To obtain or confirm the ID, the API below must be used.

<a id="id-support-api-verify-valid-id-api"></a>
#### Verify Valid ID API

<!-- TODO: translate body -->

<a id="id-support-api-verify-registered-services-api-insecure"></a>
#### Verify Registered Services API (Insecure)

<!-- TODO: translate body -->

