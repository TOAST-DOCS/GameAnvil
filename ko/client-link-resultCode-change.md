#### ResultCode 추가 및 이름 변경 세부 사항

* ResultCodeAuth
  * 추가
    * AUTH_FAIL_INVALID_ACCOUNT_ID
* ResultCodeLogin
  * 이름변경
    * LOGIN_FAIL_EMPTY_SUB_ID -> LOGIN_FAIL_INVALID_SUB_ID
  * 제거
    * LOGIN_FAIL_LOGINED_SAME_SERVICE
  * 추가.
    * LOGIN_FAIL_INVALID_USERID : 잘못된 유저 아이디.
    * LOGIN_FAIL_LOGINED_OTHER_USER_TYPE : 동일 Account 아이디로 다른 UserType이 로그인 되어있음.
    * LOGIN_FAIL_LOGINED_OTHER_DEVICE : 동일 Account 아이디로 다른 DeviceId가 로그인 되어있음.
* ResultCodeMatchRoom
  * 제거
    * MATCH_ROOM_FAIL_UNKNOWN_ERROR
* ResultCodeMatchUserStart
  * 제거
    * MATCH_USER_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchUserCancel
  * 제거
    * MATCH_USER_CANCEL_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyStart
  * 제거
    * MATCH_PARTY_START_FAIL_MATCH_UNKNOWN_ERROR
* ResultCodeMatchPartyCancel
  * 제거
    * MATCH_PARTY_CANCEL_FAIL_MATCH_UNKNOWN_ERROR        
* ResultCodeNamedRoom
  * 추가.
    * NAMED_ROOM_FAIL_INVALID_ROOM_NAME : 잘못된 방 이름을 보냈을 경우.
* ResultCodeDisconnect
  * 이름 변경
    * FORCE_CLOSE_SYSTEM -> FORCE_CLOSE_SYSTEM_ERROR
    * FORCE_CLOSE_CONTENT -> 
      FORCE_CLOSE_BASE_CONNECTION : 서버에서 BaseConnection의 close() 호출
      FORCE_CLOSE_BASE_USER : 서버에서 BaseUser의 closeConnection() 호출
  * 추가.
    * FORCE_CLOSE_INVALID_NODE : GameNode가 invalid 상태로 변경된 경우.
    * FORCE_CLOSE_USER_TRANSFER_FAIL : 유저 트렌스퍼가 실패한 경우.
    * FORCE_CLOSE_USER_TRANSFER_ERROR : 유저 트렌스퍼중 시스템 에러가 발생한 경우.
  * 추가되었으나 클라이언트에서 받을 수 없는 경우.
    서버에서는 클라이언트의 연결이 이미 끊겨있을것으로 예상하고 접속을 강제 종료하면서 아래 코드를 사용한다.
    클라이언트의 연결이 끈겨 있기 때문에 아래 코드는 받을 수 없어야 한다.
    이 코드를 받았다면 GameAnvil 개발팀에 제보해 주시길 바란다.
      * FORCE_CLOSE_BY_NEW_CONNECTION : 같은 계정 정보로 새로운 로그인 요청이 들어온 경우. 
        * <span style="color:#eb6420">현제 FORCE_CLOSE_DUPLICATE_LOGIN 케이스에 이 코드가 넘어오는 문제가 있다.
          다음 릴리즈 때 수정 될 예정.</span>
      * FORCE_CLOSE_DISCONNECT_ALARM : 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우.
      * FORCE_CLOSE_CHECK_CLIENT_STATE_FAIL : 클라이언트가 서버의 상태 체크에 응답을 하지 않은 경우.
      * FORCE_CLOSE_GHOST_USER : 고스트 유저인 경우.