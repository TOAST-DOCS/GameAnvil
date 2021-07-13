## Game > GameAnvil > 서버 개발 가이드 > 아이디 사용법

## 1. 아이디 (ID)

GameAnvil은 여러 종류의 아이디를 사용합니다. 그 중 일부는 서버가 자체 발급하고 다른 일부는 사용자가 GameAnvilConfig에 직접 설정합니다. 접속에 필요한 계정 정보 등은 클라이언트에서 입력받은 정보를 서버로 전달합니다. 다음은 GameAnvil에서 사용하는 대표적인 아이디에 대한 설명입니다.

| 이름      | 설명                                                         | 자료형 | 범위      |
| --------- | ------------------------------------------------------------ | ------ | --------- |
| ServiceId | 설정한 각각의 서비스를 구분하기 위한 아이디 - 하나의 서비스 아이디는 여러 개의 노드로 구성할 수 있음 | int    | 0<id< 100 |
| HostId    | 호스트의 고유 아이디 - 하나의 호스트에 여러 개의 GameAnvil 프로세스를 구동할 경우에는 GameAnvilConfig에 별도의 vmId를 설정 | long   | -         |
| NodeId    | 노드의 고유 아이디 - HostId + ServiceId + 내부 카운터 값으로 구성 | long   | -         |
| AccountId | 클라이언트가 접속할 때 입력 - 커넥션 하나당 하나의 계정 아이디가 매핑 | string | -         |
| UserId    | 게임 유저 객체의 고유 아이디 - 게임 유저 객체가 생성될 때 서버가 발급 - 동일한 유저가 재접속 할 경우라도 새로운 게임 유저 객체가 생성되면 새로운 아이디 발급 | int    | -         |
| RoomId    | 방의 고유 아이디 - 방 객체가 생성될 때 서버가 발급 | int    | -         |
| SubId     | 하나의 계정(AccountId) 내에서 고유한 보조 아이디로서 클라이언트가 접속할 때 전달하는 값<br>하나의 커넥션 내에서 여러개의 세션을 구분하기 위해 사용하며, 세션의 고유 아이디는 AccountId와 SubId를 조합해서 생성함 | int    | 0 < id    |

### 1-1.아이디 지원 API

앞서 살펴본 아이디에 관한 일부 기능을 아래의 표와 같이 엔진 사용자에게 제공합니다. 해당 아이디를 획득하거나 확인하기 위해서는 반드시 아래의 API를 사용해야 합니다.

#### ServiceId API

| 메서드                                | 설명                                    |
| ------------------------------------- | --------------------------------------- |
| boolean isValid(int serviceId)        | 유효한 serviceId인지 확인               |
| int findServiceId(String serviceName) | ServiceName에 해당하는 ServiceId을 획득 |
| String findServiceName(int serviceId) | ServiceId에 해당하는 ServiceName을 획득 |

#### HostId API

| 메서드                       | 설명                     |
| ---------------------------- | ------------------------ |
| boolean isValid(long hostId) | 유효한 hostId인지 확인   |
| long get()                   | 프로세스의 hostId를 획득 |

#### NodeId API

| 메서드                        | 이름                          |
| ----------------------------- | ----------------------------- |
| boolean isValid(long nodeId)  | 유효한 nodeId인지 확인        |
| long getHostId(long nodeId)   | nodeId로부터 hostId를 획득    |
| int getServiceId(long nodeId) | nodeId로부터 serviceId를 획득 |
| int getNodeNum(long nodeId)   | nodeId로부터 nodeNum을 획득   |

#### UserId API

| 메서드                      | 설명                   |
| --------------------------- | ---------------------- |
| boolean isValid(int userId) | 유효한 userId인지 확인 |

#### RoomId API

| 메서드                      | 설명                   |
| --------------------------- | ---------------------- |
| boolean isValid(int roomId) | 유효한 roomId인지 확인 |

#### SubId API

| 메서드                     | 설명                  |
| -------------------------- | --------------------- |
| boolean isValid(int subId) | 유효한 subId인지 확인 |
