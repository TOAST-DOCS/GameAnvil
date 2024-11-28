## Game > GameAnvil > 서버 개발 가이드 > 콜백 흐름

## 인증
#### Client.Authentication 
#### BaseConnection.onAuthenticate

## 로그인
#### Client.Login
#### BaseSession.onBeforeLogin**
#### - BaseUser.onLoginByOtherDevice
#### - BaseUser.onLoginByOtherUserType
#### - BaseUser.onLoginByOtherConnection
#### BaseUser.onLogin
#### BaseUser.onReLogin
#### - BaseRoom.onRejoinRoom
#### BaseUser.onAfterLogin**
#### BaseSession.onAfterLogin**

## 로그아웃
#### Client.Logout
#### BaseUser.canLogout**
#### - BaseRoom.onLeaveRoom
#### - BaseRoom.onAfterLeaveRoom**
#### - BaseUser.onAfterLeaveRoom**
#### BaseUser.onLogout
#### BaseSession.onAftterLogout**

## 방 생성
#### Client.CreateRoom or NamedRoom
#### BaseRoom.onCreateRoom

## 방 입장
#### Client.JoinRoom or NamedRoom
#### BaseRoom.onJoinRoom

## 방 나가기
#### Client.LeaveRoom
#### BaseRoom.canLeaveRoom**
#### BaseRoom.onLeaveRoom
#### BaseUser.onAfterLeaveRoom**
#### BaseRoom.onAfterLeaveRoom**
#### BaseRoom.onDestroy

## 매치 룸
#### Client.MatchRoom
#### - BaseRoom.canLeaveRoom**
#### - BaseRoom.onLeaveRoom
#### - BaseUser.onAfterLeaveRoom**
#### - BaseRoom.onAfterLeaveRoom**
#### BaseUser.onMatchRoom
#### BaseRoomMatchMaker.onMatch**
#### * BaseRoom.onCreateRoom
#### * BaseRoom.onJoinRoom
#### - BaseUser.onMatchRoomFail

## 매치 유저
#### Client.MatchUserStart
#### BaseUser.onMatchUser
#### - BaseUserMatchMaker.onRefill
#### BaseUserMatchMaker.onMatch
#### * BaseRoom.onCreateRoom
#### * BaseRoom.onJoinRoom
#### - BaseUser.onMatchUserCancel
#### - BaseUser.onMatchUserFail

## 매치 파티
#### Client.MatchPartyStart
#### BaseRoom.onMatchParty
#### - BaseUserMatchMaker.onRefill
#### BaseUserMatchMaker.onMatch
#### BaseRoom.onLeaveRoom
#### BaseUser.onAfterLeaveRoom**
#### BaseRoom.onAfterLeaveRoom**
#### * BaseRoom.onCreateRoom
#### * BaseRoom.onJoinRoom
#### - BaseRoom.onMatchPartyCancel*
#### - BaseUser.onMatchUserFail

## 유저 트랜스퍼
#### BaseUser.canTransferOut**
#### BaseUser.onTransferOut
#### BaseUser.onTransferIn**
#### BaseUser.onAfterTransferIn**
#### - BaseUser.onPause
#### - BaseUser.onResume

## 룸 트랜스퍼
#### BaseRoom.canTransferOut**
#### BaseRoom.onTransferOut
#### - BaseUser.onTransferOut
#### - BaseUser.onTransferIn**
#### BaseRoom.onTransferIn**
#### BaseRoom.onAfterTransferIn**
#### - BaseRoom.onPause
#### - BaseRoom.onResume

## 채널 이동
#### Client.MoveChannel
#### BaseUser.canMoveOutChannel**
#### BaseUser.onMoveOutChannel
#### BaseUser.onTransferOut
#### BaseUser.onTransferInTimerHandler
#### BaseUser.onTransferIn**
#### BaseUser.onAfterTransferIn**
#### - BaseUser.onPause
#### - BaseUser.onResume
#### BaseUser.onMoveInChannel

## 채널 정보 확인
#### Client.GetChannelInfo
#### - BaseGameNode.onChannelInfo

## 채널 사용자 정보 갱신
#### BaseUser.updateChannelUserInfo
#### - BaseGameNode.onChannelRoomInfoUpdate

## 채널 방 정보 갱신
#### BaseRoom.updateChannelRoomInfo
#### - BaseGameNode.onChannelUserInfoUpdate

## 게임데이터 갱신
#### /game-data/publish-to-management
#### BaseNode.onUpdateGameData

