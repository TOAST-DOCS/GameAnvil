## Game > GameAnvil > 서버 개발 가이드 > 콜백 흐름

## 콜백 흐름
GameAnvil에서 동작에 따라 내부적으로 처리되는 콜백 순서입니다.

## 인증
![callback-flow-1.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-1.png)

클라이언트가 인증 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체       | 콜백 메서드         | 순서 | 설명         |
|-------------|----------------|----|------------|
| Client      | Authentication | 1  | 클라이언트에서 요청 |
| IConnection | onAuthenticate | 2  |            |

## 로그인
![callback-flow-2.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-2.png)

클라이언트 로그인 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체    | 콜백 메서드                   | 순서  | 설명                   |
|----------|--------------------------|-----|----------------------|
| Client   | Login                    | 1   | 클라이언트에서 요청           |
| ISession | onBeforeLogin            | 2   | 로그인 처리전 설정 상태 확인     |
| - IUser  | onLoginByOtherDevice     | 2-1 | 다른 디바이스 로그인시 호출      |
| - IUser  | onLoginByOtherUserType   | 2-1 | 다른 유저 타입 로그인시 호출     |
| - IUser  | onLoginByOtherConnection | 2-1 | 새로운 연결로 로그인시 호출      |
| IUser    | onLogin                  | 3-1 | 로그인 안되어 있다면 로그인 호출   |
| IUser    | onReLogin                | 3-1 | 로그인 되어 있다면 재로그인 호훙   |
| - IRoom  | onRejoinRoom             | 3-2 | 리로그인시 방에 들어가 있었다면 호출 |
| IUser    | onAfterLogin             | 4   |                      |
| ISession | onAfterLogin             | 5   |                      |

## 로그아웃
![callback-flow-3.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-3.png)

클라이언트 로그아웃 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체      | 콜백 메서드           | 순서  | 설명                                            |
|------------|------------------|-----|-----------------------------------------------|
| Client     | Logout           | 1   | 클라이언트에서 요청                                    |
| IUser      | canLogout        | 2   | 로그아웃 처리가 가능 한지 확인                             |
| - IRoom    | onLeaveRoom      | 2-1 |                                               |
| - IRoom    | onAfterLeaveRoom | 2-2 | 2-2,2.3 번의 동작은 user와 room에서 처리되어서 순서 보작이 되지않음 |
| - IUser    | onAfterLeaveRoom | 2-3 |                                               |
| IUser      | onLogout         | 3   |                                               |
| ISession   | onAfterLogout    | 4   |                                               |

## 방 생성
![callback-flow-4.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-4.png)

클라이언트 방 생성 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체   | 콜백 메서드                  | 순서  | 설명         |
|---------|-------------------------|-----|------------|
| Client  | CreateRoom or NamedRoom | 1   | 클라이언트에서 요청 |
| IRoom   | onCreateRoom            | 2   |            |

## 방 입장
![callback-flow-5.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-5.png)

클라이언트 방 입장 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체   | 콜백 메서드                | 순서  | 설명         |
|---------|-----------------------|-----|------------|
| Client  | JoinRoom or NamedRoom | 1   | 클라이언트에서 요청 |
| IRoom   | onJoinRoom            | 2   |            |

## 방 나가기
![callback-flow-6.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-6.png)

클라이언트 방 나가기 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체   | 콜백 메서드              | 순서 | 설명                                        |
|---------|---------------------|----|-------------------------------------------|
| Client  | LeaveRoom           | 1  | 클라이언트에서 요청                                |
| IRoom   | canLeaveRoom        | 2  | 방 나가기 처리가 가능 한지 확인                        |
| IRoom   | onLeaveRoom         | 3  |                                           |
| IRoom   | onAfterLeaveRoom    | 4  |                                           |
| IUser   | onAfterLeaveRoom    | 5  | 4,5 번의 동작은 user와 room에서 처리되어서 순서 보작이 되지않음 |
| IRoom   | onDestroy           | 6  |                                           |

## 매치 룸
![callback-flow-7.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-7.png)

클라이언트 룸 매치 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체                  | 콜백 메서드            | 순서 | 설명                                        |
|------------------------|-------------------|----|-------------------------------------------|
| Client                 | MatchRoom         | 1  | 클라이언트에서 요청                                |
| - IRoom                | canLeaveRoom      | 2  | 방 나가기 처리가 가능 한지 확인                        |
| - IRoom                | onLeaveRoom       | 3  |                                           |
| - IRoom                | onAfterLeaveRoom  | 4  |                                           |
| - IUser                | onAfterLeaveRoom  | 5  | 4,5 번의 동작은 user와 room에서 처리되어서 순서 보작이 되지않음 |
| IUser                  | onMatchRoom       | 6  |                                           |
| AbstractRoomMatchMaker | onMatch           | 7  |                                           |
| IRoom                  | onCreateRoom      | 8  |                                           |
| IRoom                  | onJoinRoom        | 8  |                                           |
| IUser                  | onMatchRoom       | 9  |                                           |

## 매치 유저
![callback-flow-8.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-8.png)

클라이언트 유저 매치 시작 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체                    | 콜백 메서드            | 순서  | 설명         |
|--------------------------|-------------------|-----|------------|
| Client                   | MatchUserStart    | 1   | 클라이언트에서 요청 |
| IUser                    | onMatchUser       | 2   |            |
| - IUser                  | onMatchUserFail   | 2-1 | 매치 실패      |
| - AbstractUserMatchMaker | onRefill          | 2-1 | 유저 리필 처리   |
| AbstractUserMatchMaker   | onMatch           | 3   |            |
| IRoom                    | onCreateRoom      | 4   |            |
| IRoom                    | onJoinRoom        | 4   |            |
| - IUser                  | onMatchUserCancel | 4-1 | 방입장 취소     |

## 유저 트랜스퍼
![callback-flow-9.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-9.png)

유저 트랜스퍼 시작 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체   | 콜백 메서드              | 순서 | 설명               |
|---------|---------------------|----|------------------|
| IUser   | canTransferOut      | 1  | 유저 트랜스퍼 가능 한지 확인 |
| IUser   | onTransferOut       | 2  |                  |
| IUser   | onTransferIn        | 3  |                  |
| IUser   | onAfterTransferIn   | 4  |                  |
| - IUser | onPause             | 5  |                  |
| - IUser | onResume            | 6  |                  |

## 룸 트랜스퍼
![callback-flow-10.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-10.png)

룸 트랜스퍼 시작 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체   | 콜백 메서드            | 순서  | 설명                |
|---------|-------------------|-----|-------------------|
| IRoom   | canTransferOut    | 1   | 룸 트랜스퍼 가능 한지 확인   |
| IRoom   | onTransferOut     | 2   |                   |
| - IUser | onTransferOut     | 3-1 | 룸 트랜스퍼 후 유저 트랜스퍼  |
| - IUser | onTransferIn      | 3-2 |                   |
| IRoom   | onTransferIn      | 4   |                   |
| IRoom   | onAfterTransferIn | 5   |                   |
| - IRoom | onPause           | 6   |                   |
| - IRoom | onResume          | 7   |                   |

## 채널 이동
![callback-flow-11.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-11.png)

클라이언트 채널 이동 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체    | 콜백 메서드               | 순서 | 설명         |
|----------|----------------------|----|------------|
| Client   | MoveChannel          | 1  | 클라이언트에서 요청 |
| IUser    | canMoveOutChannel    | 2  |            |
| IUser    | onMoveOutChannel     | 3  |            |
| IUser    | onTransferOut        | 4  |            |
| IUser    | onTransferIn         | 5  |            |
| IUser    | onAfterTransferIn    | 6  |            |
| - IUser  | onPause              | 7  |            |
| - IUser  | onResume             | 8  |            |
| IUser    | onMoveInChannel      | 9  |            |

## 채널 정보 확인
![callback-flow-12.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-12.png)

클라이언트 채널 정보 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체       | 콜백 메서드           | 순서 | 설명         |
|-------------|------------------|----|------------|
| Client      | GetChannelInfo   | 1  | 클라이언트에서 요청 |
| - IGameNode | onChannelInfo    | 2  |            |

## 채널 사용자 정보 갱신
![callback-flow-13.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-13.png)

클라이언트 채널 정보 갱신 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체       | 콜백 메서드                   | 순서 | 설명 |
|-------------|--------------------------|----|----|
| IUser       | updateChannelUserInfo    | 1  |    |
| - IGameNode | onChannelRoomInfoUpdate  | 2  |    |

## 채널 방 정보 갱신
![callback-flow-14.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-14.png)

클라이언트 채널 방 정보 갱신 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체       | 콜백 메서드                   | 순서 | 설명 |
|-------------|--------------------------|----|----|
| IRoom       | updateChannelRoomInfo    | 1  |    |
| - IGameNode | onChannelUserInfoUpdate  | 2  |    |

## 게임데이터 갱신
![callback-flow-15.png](https://static.toastoven.net/prod_gameanvil/images/v2_0/server-impl/05-1-callback-flow/callback-flow-15.png)

매니지먼트 노드를 통해서 게임 데이터 갱신 요청 시 처리되는 콜백 메서드 순서입니다.

| 호출 객체          | 콜백 메서드                           | 순서 | 설명            |
|----------------|----------------------------------|----|---------------|
| ManagementNode | /game-data/publish-to-management | 1  | 매니지먼트 노드로 요청  |
| - INode        | onUpdateGameData                 | 2  |               |

