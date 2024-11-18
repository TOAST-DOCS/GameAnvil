## Game > GameAnvil > 콘솔 사용 가이드 > Safe Pause

## Safe Pause란?

Safe Pause는 서비스 중지 없이 임의의 게임 노드를 일시 중단(Pause)할 수 있는 기능입니다. Safe Pause의 출발지 노드는 처리 중이던 모든 유저와 방 정보를 동일한 서비스를 수행 중인 도착지 노드로 실시간 전송(Transfer)합니다. 그러므로 동일한 서비스의 게임 노드가 2개 이상 존재하지 않는다면 Safe Pause를 사용할 수 없습니다.

Safe Pause 과정에서 전송을 보내는 쪽과 받는 쪽은 아래의 그림처럼 각각 **SAFE PAUSE** 상태와 **READY(LOCK)** 상태로 바뀝니다. **READY(LOCK)** 상태의 노드는 해당 Safe Pause 트랜잭션이 완료될 때까지 임의로 상태를 변경할 수 없습니다. 즉, 콘솔상에서 그 어떤 조작도 할 수 없습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/safepause_list.png)

Safe Pause가 완료되면 **Safe Pause** 상태는 **Pause** 상태로 바뀝니다. 당연히 해당 노드에는 그 어떤 유저나 방도 존재하지 않으므로 안전하게 셧다운할 수 있습니다. 패치나 점검이 필요하다면 이때 진행하면 됩니다. **Ready(Lock)** 상태 또한 Safe Pause가 완료됨과 동시에 **Ready** 상태로 전환되며 콘솔상에서 조작이 가능해집니다.

## Safe Pause 사용하기

Safe Pause 메뉴를 선택하면 Safe Pause 기본 화면을 볼 수 있습니다. 이 화면은 Safe Pause를 실행할 수 있는 노드의 목록을 보여줍니다. Safe Pause를 실행할 노드를 선택한 후 **Safe Pause 실행**을 클릭하면 아래와 같이 Safe Pause를 실행하기전 팝업이 열리며, 실행에 대한 메모를 작성할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/safepause_exec_popup.png)

**실행 중**탭을 클릭하여 현재 진행중인 Safe Pause 목록을 확인 할 수 있습니다.

![그림](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/running_safepause_list.png)

Safe Pause 과정을 통해 **출발지 노드**들의 유저와 방은 **도착지 노드**로 전송되며, 모든 과정이 완료된 뒤 **출발지 노드**들은 모두 자동으로 PAUSE 상태가 됩니다. 이제 PAUSE 된 노드들은 안전하게 종료할 수 있습니다.   

![그림](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/paused_node_server_list.png)
