## Game > GameAnvil > 튜토리얼

### GameAnvil로 게임 서버 쉽게 만들기

이 문서는 GameAnvil을 이용하여 기본적인 서버를 개발하는 과정을 설명합니다. 이를 통해 GameAnvil의 기본 개념, 프로젝트 시작 방법 및 구현 방법 등을 쉽게 익힐 수 있도록 도움을 주고자 합니다. 또한 서버뿐만 아니라 Unity를 이용한 클라이언트도 함께 개발하면서 서버와 클라이언트가 상호 작용하는 모습을 보여주고자 합니다.

최대한 이해를 돕기 위해 실제 플레이가 가능한 멀티플레이 직소 퍼즐 게임을 개발하는 과정을 순서대로 보여줍니다. 글을 읽는 사용자 여러분은 이 문서의 내용을 직접 하나씩 따라해보면 더욱 쉽게 이해할 수 있으리라 믿습니다.

## 1. 프로젝트 구성

### 1.1. GameAnvil 프로젝트 구성

다음 링크를 통해서 InteliJ용 프로젝트 파일 템플릿과 튜토리얼용 프로젝트를 다운로드 받습니다.

[프로젝트 템플릿 다운로드](https://static.toastoven.net/prod_gameanvil/files/GameAnvil Template.zip?disposition=attachment)

[튜토리얼용 프로젝트 다운로드](https://static.toastoven.net/prod_gameanvil/files/GameAnvil Tutorial Project.zip?disposition=attachment)

다운로드 받은 템플릿을 적용하기 위해 InteliJ에서 단축키 `Shift Shift` 로 전체 검색창을 띄운 다음 `Import Settings...` 을 검색합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/search_import_settings.png" />

파인더 창이 뜨면 다운로드 받은 템플릿을 선택합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/select_import.png)

파일 템플릿과 프로젝트 템플릿을 모두 체크한 후 임포트합니다. 가져오기가 완료되면 InteliJ를 다시 시작하고 튜토리얼용 프로젝트를 엽니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_project_gameanvil_tutorial.png"/>



프로젝트 이름과 위치를 확인한 후 Finish를 눌러 프로젝트를 생성합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_project_gameanvil_tutorial_finish.png" />

Trust Project를 선택합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/trust_project.png"/>

이제 IntelliJ에 아래와 같은 서버 프로젝트 골격이 구성되었습니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/gameanvil_projectview_init.png"/>

<br>

### 1.2. Unity 프로젝트 구성

Unity Hub를 실행한 후 우상단 New 버튼을 눌러 새로운 프로젝트 생성 창을 띄웁니다. 

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unityhub.png)

창 좌측의 2D를 선택하고, 프로젝트 이름과 위치를 확인한 후 Create를 눌러 프로젝트 생성을 완료합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unityhub_new_project.png)

다음 링크를 통해서 커넥터와 튜토리얼용 소스들을 다운로드 받습니다. 커넥터는 GameAnvil 서버와의 통신에 필요한 클라이언트 API를 제공합니다.

[[ gameanvil-connector.unitypackage ]](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector.unitypackage)
[[ GameAnvilTutorial.unitypacakge ]](https://static.toastoven.net/prod_gameanvil/files/GameAnvil Tutorial.unitypackage)

다운로드 받은 파일을 Unity 프로젝트에 드래그해서 프로젝트상에 가져옵니다. 또는 Asset - Import Package - Custom Package... 메뉴를 열어 탐색기에서 다운로드 받은 파일을 선택합니다.![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_import_custom_package.png)

모든 체크 박스를 채운 후 Import 버튼을 클릭합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_import_custom_package_window.png"/>

아래와 같이 프로젝트 기본 창 폭과 높이를 각각 1920, 1080으로 설정합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_project_setting_resolution.png"/>

## 2. 서버 구동 및 연결

### 2.1. GameAnvil 서버 구동

GameAnvil 프로젝트의 VM Option에 아래 내용을 추가합니다. 이는 사용 중인 Java 버전에 따라 조금 달라지므로 주의가 필요합니다. GameAnvil은 Java 8과 11 두 가지 버전을 지원합니다.

Java 8 버전을 사용할 경우 아래의 내용을 추가합니다.

```
-javaagent:/YOUR_PATH/.m2/repository/com/nhn/gameanvil/quasar-core/0.7.10/quasar-core-0.7.10-jdk8.jar=bm
```

Java 11 버전을 사용한다면, 대신 아래 내용을 추가합니다.

```
-javaagent:/YOUR_PATH/.m2/repository/com/nhn/gameanvil/quasar-core/0.8.0/quasar-core-0.8.0-jdk11.jar=bm
```

GameAnvil 프로젝트가 구성된 IntelliJ 우측 상단의 Run 아이콘을 클릭해 서버를 실행시킵니다. 혹은 Run 메뉴 > Run 을 선택해서 실행할 수도 있습니다. 이 때, 서버가 정상적으로 구동되면 아래와 같이 onReady 로그들이 다수 출력됩니다. GameAnvil 서버는 여러개의 노드들로 구성되어 있으므로 각각의 노드마다 준비가 완료되면 onReady 로그를 출력합니다. 클라이언트가 직접 접속할 GatewayNode가 onReady 되었다면 GameAnvil 서버는 이제 언제든 접속이 가능한 상태입니다.

<br>

### 2.2. 커넥션 핸들러 작성

이제 Unity 프로젝트로 이동하여 GameAnvil 서버와 연결해 보겠습니다. 서버와 연결하려면 먼저 커넥터를 생성해야 합니다.

Connect.scene의 ConnectHandler 게임오브젝트의 스크립트를 수정합니다.

```c#
// Client-side

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using GameAnvil.User;

public class ConnectHandler : MonoBehaviour
{
 		[SerializeField]
    GameAnvil.Connector.Config config;

    static GameAnvil.Connector connector = null;
		private ConnectionAgent connectionAgent;

    [Header("Object Reference")]
    public Text serverInfoText;
    public Text clientInfoText;
    public Text logText;

    public void Quit(){
        Application.Quit();
    }

    public static GameAnvil.Connector getInstance() {
        return connector;
    }

    private void Awake() {
        DontDestroyOnLoad(this.gameObject);
        connector = new GameAnvil.Connector(config);
    }

    private void Update() {
        connector.Update(); // 메시지 처리를 위해 반드시 주기적으로 처리해야 합니다.
    }

    private void OnDisable() {
        if (connector.IsConnected()) {
            connector.CloseSocket();
        }
    }

    private void OnApplicationPause(bool pause){
        if (pause)
        {
            // 입력한 시간동안 서버의 clientStateCheck 기능을 정지시킨다
            // 이시간이 지나면 clientStateCheck 기능이 동작하여 연결이 끊어질 수 있다.
            connector.GetConnectionAgent().PauseClientStateCheck(10 * 60);

            connector.Update();
        } else {
            connector.Update();

            // 서버의 clientStateCheck 기능을 다시 동작시킨다.
            connector.GetConnectionAgent().ResumeClientStateCheck();
        }
    }
}
```

이 때, 서버와 주고받는 메시지 처리를 위해 커넥터의 Update() 메서드를 주기적으로 호출해야 함에 주의합니다.

<br>

### 2.3. 서버 연결 코드 추가

서버 접속 및 인증은 ConnectionAgent를 이용합니다. Unity가 시작되는 시점에 연결이 수행되도록 ConnectionHandler 코드에 아래와 같이 Start() 메서드와 Connect() 메서드를 추가합니다.

```c#
// Client-side

[SerializeField]
GameAnvil.Connector.Config config;

static GameAnvil.Connector connector = null;
private ConnectionAgent connectionAgent;

public string ip = "127.0.0.1";
public int port = 11200;


[Header("Object Reference")]
public Text serverInfoText;
public Text clientInfoText;
public Text logText;

void Start () {
    // 시작 시점에 바로 연결을 시도합니다.
    Connect();
  
  	// 연결 정보를 화면에 출력합니다.
  	serverInfoText.text = "서버정보:" + ip + ":" + port;
}

public void Connect(){
  connectionAgent = connector.GetConnectionAgent();
	connectionAgent.Connect(ip, port,
            (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
                Debug.Log(result);
            
              	// 게임 화면 상에 결과를 표시합니다.
                if (result == ResultCodeConnect.CONNECT_FAIL) {
                    logText.text = "연결 정보 : 연결 실패";
                } else {
                    logText.text = "연결 정보 : 연결 성공";
                }
            }
        );
}
```

<br>

### 2.4. 서버 연결 확인

이제 Unity 클라이언트를 플레이하여 콘솔 상에 로그가 제대로 출력되는지 확인합니다. 게임 화면의 텍스트상에 IP, Port의 접속 정보와 더불어 연결 성공 메시지를 확인할 수 있습니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_connection_test.png"/>

<br>

## 3. Room 및 User생성

### 3.1. 서버 작업

게임서버에 접속한 클라이언트들은 상호 메시지 교환을 할 수 있습니다. 클라이언트는 서버 상에서 하나 이상의 게임 유저(User)로 로그인할 수 있습니다. 즉, 서버에 접속한 클라이언트를 유저라고 칭할 수 있습니다. 이러한 유저들이 서로 다른 유저와 게임 관련 메지시를 교환하려면 해당 유저들은 같은 방(Room) 안에 속해 있어야합니다. 

이러한 방을 서버에서 어떻게 구현하고, 클라이언트는 어떻게 방 생성이나 참여를 요청하는지 알아보겠습니다. `Ctrl + N` 또는 `Cmd + N` 으로 새 파일 생성 컨텍스트 메뉴를 열어 GameUser를 선택합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gameuser.png"/>

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.BaseUser;
import com.nhn.gameanvil.packet.Packet;
import com.nhn.gameanvil.packet.Payload;

@ServiceName("BASIC_SERVICE")
@UserType("BASIC_USER")
public class BasicUser extends BaseUser {
    @Override
    public boolean onLogin(Payload payload, Payload sessionPayload, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public void onPostLogin() throws SuspendExecution {

    }

    @Override
    public boolean onReLogin(Payload payload, Payload sessionPayload, Payload outPayload) throws SuspendExecution {
        return false;
    }

    @Override
    public void onDisconnect() throws SuspendExecution {

    }

    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {

    }

    @Override
    public void onPause() throws SuspendExecution {

    }

    @Override
    public void onResume() throws SuspendExecution {

    }

    @Override
    public void onLogout(Payload payload, Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean canLogout() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canTransfer() throws SuspendExecution {
        return false;
    }
}

```

BasicUser는 GameAnvil에서 제공하는 BaseUser 클래스를 상속하여 간단하게 구현할 수 있습니다. onLogin 콜백 메서드에서는 로그인 과정에 필요한 구현을 할 수 있습니다. 또한 로그인의 성공 여부를 반환값으로 결정합니다. 이 예제에서는 별다른 로그인 구현 없이, 무조건 로그인에 성공하도록 true를 반환하도록 합니다.

유저를 구현했으니 이제 방을 구현해 보겠습니다. 서버 프로젝트에 기본적인 형태의 Room 클래스를 생성합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gameroom.png"/>

아래와 같이 유저를 저장할 맵을 추가합니다.

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.BaseRoom;
import com.nhn.gameanvil.node.game.BaseUser;
import com.nhn.gameanvil.node.game.RoomPacketDispatcher;
import com.nhn.gameanvil.packet.Packet;
import com.nhn.gameanvil.packet.Payload;
import org.slf4j.Logger;

import java.util.Map;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoom extends BaseRoom<BasicUser> {
    private static final Logger logger = getLogger(BasicRoom.class);
    private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();
    private static Map<Integer, BaseUser> users = new HashMap<>();

    @Override
    public boolean onCreateRoom(BasicUser basicUser, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(basicUser.getUserId(), basicUser);

        logger.debug("onCreateRoom(userId:{})", basicUser);
        return true;
    }

    @Override
    public boolean onJoinRoom(BasicUser basicUser, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(basicUser.getUserId(), basicUser);

        logger.debug("onJoinRoom(userId:{})", basicUser);
        return true;
    }

    @Override
    public void onInit() throws SuspendExecution {

    }

    @Override
    public void onDestroy() throws SuspendExecution {

    }

    @Override
    public void onDispatch(BasicUser user, Packet packet) throws SuspendExecution {

    }

    @Override
    public boolean onLeaveRoom(BasicUser user, Payload inPayload, Payload outPayload) throws SuspendExecution {
        return true;
    }

    @Override
    public void onPostLeaveRoom(BasicUser user) throws SuspendExecution {

    }

    @Override
    public void onRejoinRoom(BasicUser user, Payload outPayload) throws SuspendExecution {

    }

    @Override
    public boolean canTransfer() throws SuspendExecution {
        return false;
    }
}

```

이 예제 코드는 GameAnvil이 제공하는 BaseRoom 클래스를 상속받아 다음과 같은 추가 구현을 하였습니다.

* GameAnvil이 제공하는 RoomPacketDispatcher를 이용하여 방으로 들어오는 패킷을 관리합니다.
* 직접 추가한 users 맵을 통해 방 안의 유저들을 저장하고 관리합니다.
* onCreateRoom 콜백 메서드는 유저가 방을 만들었을 때 엔진에 의해 호출되는 콜백 메서드이고, onJoinRoom 콜백 메서드는 유저가 방에 입장했을 때 호출됩니다.
* 유저가 방에 입장했을 때 유저 목록에 유저를 추가합니다.

*  직접 추가한 users 맵을 통해 방 안의 유저들을 관리합니다.

*  onCreateRoom 콜백 메서드는 유저가 방을 만들었을 때 엔진에 의해 호출되는 콜백 메서드이고, onJoinRoom 콜백 메서드는 유저가 방에 입장했을 때 호출됩니다. 이 콜백들을 이용해 유저가 방에 입장했을 때 유저 목록에 유저를 추가합니다.

이제 유저와 방이 준비되었으니 이들을 처리해줄 GameNode를 구현합니다. 이 노드는 일반적으로 게임 서버가 하는 역할을 수행하는 노드입니다. 

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/new_gamenode.png"/>

```java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.BaseGameNode;
import com.nhn.gameanvil.node.game.data.ChannelUpdateType;
import com.nhn.gameanvil.node.game.data.ChannelUserInfo;
import com.nhn.gameanvil.node.game.data.RoomInfo;
import com.nhn.gameanvil.packet.Packet;
import com.nhn.gameanvil.packet.Payload;

@ServiceName("BASIC_SERVICE")
public class BasicGameNode extends BaseGameNode {
    @Override
    public void onInit() throws SuspendExecution {

    }

    @Override
    public void onPrepare() throws SuspendExecution {
    
    }

    @Override
    public void onReady() throws SuspendExecution {

    }

    @Override
    public void onDispatch(Packet packet) throws SuspendExecution {

    }

    @Override
    public void onPause(Payload payload) throws SuspendExecution {

    }

    @Override
    public void onResume(Payload payload) throws SuspendExecution {

    }

    @Override
    public void onShutdown() throws SuspendExecution {

    }

    @Override
    public boolean onNonStopPatchSrcStart() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchDstStart() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean onNonStopPatchDstEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public boolean canNonStopPatchDstEnd() throws SuspendExecution {
        return false;
    }

    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, ChannelUserInfo channelUserInfo, int i, String s) throws SuspendExecution {

    }

    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, ChannelRoomInfo channelRoomInfo, int i) throws SuspendExecution {

    }

    @Override
    public void onChannelInfo(Payload outPayload) throws SuspendExecution {

    }
}
```

GameAnvil은 GameNode의 유연한 구현을 위해 여러가지 콜백 메서드를 제공합니다. 하지만 이번 예제는 최소한의 기능만 사용할 예정이므로 모두 별도의 구현 없이 비워두도록 하겠습니다.

작성한 유저(User)와 방(Room) 클래스를 GameNode에 연동하려면, 설정파일에도 이를 등록해 주어야 합니다. GameAnvilConfig.json 파일에 아래의 내용을 추가해줍니다.

```
// 게임 로비 역할을 하는 노드. (게임 룸, 유저를 포함 하고있음)
  "game": [
    {
      "nodeCnt": 8,
      "serviceId": 1,
      "serviceName": "BASIC_SERVICE",
      "channelIDs": [
        "",
        "",
        "",
        "",
        "",
        "",
        "",
        ""
      ],
      "userTimeout": 5000,                // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. "" 문자열로 채널 구분없이 중복사용도 가능)
      "safeCreateTime": 1000,             //테스트 위해서 설정 보통 설정 필요 없음 기본 60초

	  "checkClientStateCycle": 10000, 		// 체크 루틴 호출 주기
	  "demandClientStateTimeout": 10000,	// 마지막 메시지를 받은 이후 demandClientStateTimeout 경과하면 demandClientState를 수행한다. 
	  "clientStateOkDeadline": 10000		// demandClientState를 요청에 대해 ClientStateOK를 응답해야할 데드라인(시간)
    }
  ],
```

이런 설정까지 추가한 [GameAnvilConfig.json](https://static.toastoven.net/prod_gameanvil/files/GameAnvilConfig.json?disposition=attachment)도 필요할 경우 다운로드해서 사용하실 수 있습니다.

<br>

## 4. 접속 인증

### 4.1. 접속 인증 코드 추가

클라이언트는 서버에 연결을 맺은 후에 반드시 접속 권한이 있는지 확인하는 과정이 필요합니다. 이를 접속 인증(Authentication) 과정이라고 합니다. 이러한 인증 과정을 처리하기 위해  ConnectionHandler에 Auth() 메서드를 추가하고, Connect() 메서드의 내용을 아래와 같이 수정합니다. Start() 메서드에도 클라이언트 관련 내용을 출력하는 코드를 넣습니다.

```c#
// Client-side

[SerializeField]
GameAnvil.Connector.Config config;

static GameAnvil.Connector connector = null;
private ConnectionAgent connectionAgent;

public string ip = "127.0.0.1";
public int port = 11200;

public string deviceId = "";
public string password = "";
public string accountId = "";

public Text connectInfoText;
public Text accountIdText;

void Start () {
		Connect();
  
    serverInfoText.text = "서버정보 : " + ip + ":" + port;
}

public void Connect(){
    connectionAgent = connector.GetConnectionAgent();
    connectionAgent.Connect(ip, port,
        (ConnectionAgent connectionAgent, ResultCodeConnect result) => {
            Debug.Log(result);

           if (result == ResultCodeConnect.CONNECT_FAIL) {
                logText.text = "연결 정보 : 연결 실패";
            } else {
                logText.text = "연결 정보 : 연결 성공";
           
             		// 연결에 성공하면 바로 인증을 시도합니다.
                Auth();
            }
        }
    );
}
public void Auth(){
    accountId = Random.Range(1000,9999) + "";
  	clientInfoText.text = "클라이언트정보 : " + accountId;
		connectionAgent.Authenticate(deviceId, accountId, password,
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
                Debug.Log(result);
         
           			if (result == ResultCodeAuth.AUTH_SUCCESS) {
                    logText.text = "인증 정보 : 인증 성공";
                } else {
                    logText.text = "인증 정보 : 인증 실패";
                }
            }
			);
}
```

<br>

### 4.2. 접속 인증 확인

Unity 클라이언트를 플레이 해서 콘솔 상에 로그가 출력됨을 확인합니다. 게임 화면 상에 인증 정보와 인증 성공 여부가 나타남을 확인할 수 있습니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_authentication_test.png"/>

<br>

## 5. 로그인

### 5.1. 로그인 코드 추가

접속 및 인증이 완료된 클라이언트는 게임 노드로 로그인을 할 수 있습니다. 서버에 로그인하면 게임 노드에 해당 클라이언트를 위한 게임 유저 객체가 생성됩니다. 클라이언트는 자신의 서버측 게임 유저 객체를 통해 서버 혹은 다른 유저들과 메시지를 주고 받을 수 있게 됩니다. 앞서 작성한 인증 코드를 아래와 같이 수정하여 인증이 성공한 경우에 바로 로그인을 진행하도록 합니다.

```c#
// Client-side

public void Auth(){
    accountId = Random.Range(1000,9999) + "";
		connectionAgent.Authenticate(deviceId, accountId, password,
         (ConnectionAgent connectionAgent, ResultCodeAuth result, List<ConnectionAgent.LoginedUserInfo> loginedUserInfoList, string message, Payload payload) => {
                Debug.Log(result);
         
           			if (result == ResultCodeAuth.AUTH_SUCCESS) {
                    logText.text = "인증 정보 : 인증 성공";
                      
                  	Login(); // 인증에 성공한 경우 바로 로그인을 시도합니다.
                } else {
                    logText.text = "인증 정보 : 인증 실패";
                }
            }
			);
}

public void Login(){
  	userAgent.Login("BASIC_USER", "", null,
                        (UserAgent userAgent, ResultCodeLogin result, UserAgent.LoginInfo loginInfo) => {
        Debug.Log(result);
    });
}
```

로그인을 요청할 때 같이 전달한 콜백 메서드가 로그인 요청 결과를 로그로 출력하도록 합니다.

<br>

### 5.2. 로그인 확인

Unity 테스트 모드를 통해 성공적으로 로그인 되는 것을 확인합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_login_test.png"/>

<br>

## 6. 방 생성 및 참가

### 6.1. 클라이언트 작업

Unity 프로젝트에서 ConnectionHandler에 방 생성을 요청하는 코드를 추가합니다. 이 때, userAgent.CreateRoom 메서드의 첫 번째 인자인 RoomType은 반드시 서버에서 지정한 값과 같아야 함에 유의합니다. 일반적으로 이러한 RoomType 등은 서버와 클라이언트 개발자가 사전에 값을 미리 정의해두고 사용합니다. 방 생성에 성공하면 roomId를 클라이언트 측에 저장해두고, 게임 씬으로 이동합니다.

```c#
using UnityEngine.SceneManagement;

public int roomId;
public UserAgent userAgent;

void Start(){
  	userAgent = CreateUserAgent("BASIC_SERVICE", 1);
}
public void CreateRoom(){
    userAgent.CreateRoom("BASIC_ROOM", (UserAgent ua, ResultCodeCreateRoom resultCode, int roomId, string roomName, Payload payload) => {
        Debug.Log(resultCode); 
        if (resultCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
            this.roomId = roomId;
       
            SceneManager.LoadScene("GameScene");
        }
    });
}
```

씬에서 Create Room 버튼의 OnClick 리스너에 ConnectHandler 컴포넌트를 드래그해서 등록하고, 드롭다운에서 CreateRoom메서드를 선택합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_create_room_on_click.png)

여기까지 완료 되었으면, 이번에는 ConnectionHandler에 방 참가 코드를 추가합니다. userAgent.JoinRoom 메서드의 첫 번째 인자가 서버에서 사용한 사전 정의된 RoomType 문자열과 일치하는지 확인합니다.

```c#
private void Update() {
    connector.Update();

    if (popupCanvas && popupCanvas.activeInHierarchy && Input.GetKeyDown(KeyCode.Return)){
        JoinRoom();
    }
}

public void JoinRoom(){
    if (!string.IsNullOrEmpty(roomIdInput.text)){
        userAgent.JoinRoom("BASIC_ROOM", int.Parse(roomIdInput.text), null, (UserAgent userAgent, ResultCodeJoinRoom resultCode, int roomId, string roomName, Payload payload)=>{
            Debug.Log(resultCode);
            if ( resultCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS){
                this.roomId = roomId;
            
                SceneManager.LoadScene("GameScene");
            }
        });
    }
}
```

GameManager 코드를 아래와 같이 수정해서 룸 아이디를 게임상에서 표시할 수 있도록 만듭니다.

```c#
public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }
}
```

<br>

### 6.2. Room 생성 및 참가 테스트

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 그리고 게임에서 방을 하나 생성합니다. 이 때, 생성된 방의 아이디가 함께 출력됩니다.

그 상태로 Unity 플레이모드를 실행한 후, 앞서 생성한 방의 RoomId를 직접 입력해서 방에 참가하도록 합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_room_test.png)

<br>

## 7. 인게임 채팅 구현

프로젝트 내부에 미리 구현된 프로토콜을 이용해서 채팅 기록을 주고 받는 방법을 알아봅시다. 통신 데이터를 클라이언트로부터 서버로 전송할 때는 MessageRequest타입을 사용합니다. 서버로부터 클라이언트로 통신 데이터를 전송할 때는 MessageResponse또는 MessageBroadcast 타입을 사용합니다.

<br>
### 7.1. 클라이언트 측 전송 구현

Unity 프로젝트의 Asset뷰에서 GameScene.scene을 엽니다. 계층관계 뷰에서 GameManager스크립트에 메시지 전송 스크립트를 추가합니다. SendMessage 메서드가 호출되면 입력된 메시지를 담은 데이터를 userAgent를 통해 서버로 전송합니다. 아래의 예제 코드는 이러한 SendMessage의 구현을 보여줍니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using Protocol;


public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }

    void Update(){
	 	 if ( Input.GetKeyDown(KeyCode.Return) && !string.IsNullOrEmpty(ChatInputText.text)){
	 	 		SendMessage();
	 	 }
	}

	public void SendMessage(){
        MessageRequest messageRequest = new MessageRequest();
        messageRequest.Message = "\n[" + connectHandler.accountId +  "]:" + ChatInputText.text;
        connectHandler.userAgent.Send(new Packet(messageRequest));
        ChatInputText.text = string.Empty;
	}
}


```

<br>

### 7.2. 서버 측 응답 구현

클라이언트가 전송한 메시지를 서버의 Room에서 처리하기 위해 핸들러를 작성합니다. 우리는 앞서 BaseRoom 클래스를 상속받은 BasicRoom 클래스에서 RoomPacketDispatcher를 생성했습니다. 이번에 작성하는 수신 메시지에 대한 핸들러는 바로 이 패킷 디스패처에 등록됩니다.

아래의 핸들러 예제 구현에서는 수신한 메시지에 대해 송신자에게 응답 메시지를 전송함과 더불어 방 전체 유저에게 방 단위 브로드캐스트용 메시지를 추가로 송신합니다. 이 때, 클라이언트는 방 단위 브로드캐스트 메시지를 기준으로 게임을 동기화 하도록 구현되어 있습니다.

우선 BasicHandler라는 메시지 핸들러 클래스를 아래와 같이 작성합니다.

```Java
import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomPacketHandler;
import com.nhn.gameanvil.packet.Packet;
import org.slf4j.Logger;
import protocol.BasicProtocol;

import java.io.IOException;

import static org.slf4j.LoggerFactory.getLogger;

public class BasicHandler implements RoomPacketHandler<BasicRoom, BasicUser> {
    private static final Logger logger = getLogger(BasicHandler.class);
  
  	@Override
    public void execute(BasicRoom room, BasicUser requester, Packet packet) throws SuspendExecution {
        try {
            BasicProtocol.MessageRequest request = BasicProtocol.MessageRequest.parseFrom(packet.getStream());
            BasicProtocol.MessageResponse response = BasicProtocol.MessageResponse.newBuilder().setMessage(request.getMessage()).build();
            BasicProtocol.MessageBroadcast broadcast = BasicProtocol.MessageBroadcast.newBuilder().setMessage(request.getMessage()).build();

            requester.reply(new Packet(response)); // 송신자에게 응답
            room.broadcast(new Packet(broadcast)); // 방 전체 유저에게 메시지를 전송
        } catch (IOException e) {
            logger.error("BasicHandler::execute()", e);;
        }
    }
}

```

핸들러에서는 수신 받은 패킷의 스트림을 역직렬화 하여 MessageRequest 객체를 만들고 있습니다. 이 후, MessageRequest 객체의 Message 값을 이용하여 MessageResponse, MessageBroadcast객체를 각각 새로 생성합니다. 이 중 MesageBroadcast객체는  room을 통해 방 내부의 모든 유저에게 전송합니다.

이와 더불어 BasicRoom 클래스에는 앞서 핸들러에서 사용한 broadcast 메소드를 구현해야 합니다. 그리고 처리할 메시지에 대해 앞서 구현한 핸들러를 등록하는 코드가 추가됩니다. 이 때, 메시지 핸들러 등록은 반드시 static 블럭 내에 구현해야 합니다.

```Java
public class BasicRoom extends BaseRoom<BasicUser> {
    private static final Logger logger = getLogger(BasicRoom.class);
    private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();
    private static Map<Integer, BaseUser> users = new HashMap<>();

    static {
        dispatcher.registerMsg(BasicProtocol.MessageRequest.getDescriptor(), BasicHandler.class); // 처리할 메시지에 대한 핸들러 등록
    }

    public void broadcast(Packet packet) {
        users.values().stream().forEach(user -> user.send(packet));
    }

    ...생략...
  
    @Override
    public void onDispatch(BasicUser user, Packet packet) throws SuspendExecution {
        dispatcher.dispatch(this, user, packet);
    }
}
```

<br>

### 7.3. 클라이언트 측 수신 구현

클라이언트 또한 서버에서 송신한 패킷을 받기 위해 미리 핸들러를 등록해야합니다. GameManager의 Start메서드에 MessageBroadcast타입의 메시지를 처리하는 핸들러 등록 코드를 작성합니다. 이 핸들러 구현은 메시지를 수신하면 화면에 그대로 출력합니다. 바로 채팅의 가장 기본적인 기능입니다.

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using Protocol;

public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public Text ChatInputText;

    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        ProtocolManager.getInstance().RegisterProtocol(0, BasicProtocolReflection.Descriptor);
        connectHandler.userAgent.AddListener<MessageBroadcast>((sendUserAgent, messageBroadcast) => {
            ChatLogText.text += messageBroadcast.Message;
        });
    }
  
		...생략...
}

```

<br>

### 7.4. 메시지 전달 확인

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 빌드된 게임에서 방을 생성하고 서버측 로그를 확인합니다. 그 상태로 Unity 플레이모드를 실행한 후 앞서 생성한 방의 RoomId를 입력하여 해당 방에 참가한 후 채팅이 제대로 동작하는지 확인합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_chat_test.png)

간단한 채팅 서버 구현을 통해서 메시지 처리 과정을 학습했습니다. 다음에는 좀 더 실용적인 예제의 구현 과정을 살펴보겠습니다.

<br>

## 8. 퍼즐 게임 구현

게임 씬에는 싱글 플레이가 가능한 퍼즐 게임이 미리 구현되어 있습니다. 퍼즐 조각을 드래그해서 알맞은 위치 근처에서 놓으면 정확한 위치로 보정됩니다. 이 게임을 멀티플레이어 게임으로 만들어보겠습니다. 먼저 각 퍼즐 위치를 플레이어들간에 동기화하기 위한 코드를 작성하겠습니다.

<br>

### 8.1. Google Protocol Buffers를 이용한 메시지 직렬화/역직렬화

같은 방 안의 유저들은 서로 메시지를 쉽게 교환할 수 있습니다. 이 때, 메시지는 미리 정의한 프로토콜에 기반하여 그 어떤 값도 포함할 수 있습니다. 이러한 메시지 또한 객체이므로 이를 네트워크로 전송하기 위해서는 직렬화 수단이 필요합니다. 마찬가지로 클라이언트는 서버로부터 받은 직렬화된 정보를 원래의 메시지 형태로 역직렬화할 수 있는 수단이 필요합니다. XML, json 등 여러가지 수단이 있겠지만 GameAnvil은 Google Protocol Buffers를 사용합니다. 이는 속도와 안정성 측면에서 가장 좋은 솔루션 중 하나입니다.

서버 프로젝트의 src/main/proto 경로에 Puzzle.proto 파일을 추가한 후 아래와 같이 프로토콜 명세를 작성합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto.png"/>

````
syntax="proto3";

package protocol;

message PuzzlePosition{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}
````

PuzzlePosition은 퍼즐 조각의 위치를 나타내기 위해서 각 퍼즐 조각의 고유 번호 정보(Index)와 퍼즐의 위치 정보(PositionX와 PositionY)로 구성합니다. 이제 프로토콜 정의는 끝났습니다.

다음은 이러한 프로토콜 정의를 각 개발 언어에 맞는 클래스로 컴파일 해주어야 합니다. 다음 명령줄을 이용하여 Java와 C#을 위한 컴파일을 수행합니다.  protoc 실행 파일은 프로젝트 최상위 경로에 있습니다.

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

이제 src/main/java 경로에 새롭게 Puzzle.java 클래스가 생성된 것을 확인할 수 있습니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto_java.png" />

C# 클래스 파일은 Finder 등의 프로그램을 이용해서 Unity 프로젝트의 Asset/Protocol 경로로 이동합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_proto_csharp.png" />

<br>

### 8.2. 프로토콜 등록

프로토콜을 정의하고 컴파일까지 무사히 마쳤다면, 서버와 클라이언트 양쪽에 해당 프로토콜 클래스를 등록해주어야 합니다. 이 때, 아주 중요한 주의 사항이 하나 있습니다. 바로, 프로토콜 클래스의 등록 인덱스가 서버와 클라이언트 양쪽에서 반드시 동일해야 한다는 것입니다. 등록 인덱스가 다를 경우 게임은 의도한대로 동작하지 않습니다. 또한 이 인덱스는 프로토콜 파일의 고유한 값으로 사용되므로 절대 다른 프로토콜과 중복되면 안됩니다.

참고로 채팅 프로토콜의 경우는 기본적으로 템플릿에 등록 되어있기 때문에 추가로 등록을 할 필요가 없었습니다. 하지만 추가로 구현한 Puzzle 프로토콜은 직접 등록을 해야 합니다. GameAnvil 서버의 Main 메서드에서 아래와 같이 프로토콜을 등록합니다. 채팅을 포함한 BasicProtocol이 숫자 0으로 등록되어 있는 것을 확인할 수 있습니다. 이번에는 숫자 1을 사용하여 Puzzle 프로토콜을 추가합니다.

```C#
public class Main {

    public static void main(String[] args) {
        GameAnvilServer gameAnvilServer = GameAnvilServer.getInstance();

        gameAnvilServer.addProtoBufClass(0, BasicProtocol.getDescriptor());
        gameAnvilServer.addProtoBufClass(1, Puzzle.getDescriptor());

        // 부트스트랩 실행.
        gameAnvilServer.run();

    }

}
```

Unity 프로젝트는 ConnectionHandler.cs의 Start 메서드에 아래와 같이 프로토콜을 추가합니다. 이 때, 서버와 같은 인덱스인 1로 등록합니다.

```c#
public class ConnectHandler : MonoBehaviour {
		void Start () {
        Connect();
  
        serverInfoText.text = "서버정보 : " + ip + ":" + port;
  	    clientInfoText.text = "클라이언트정보 : " + accountId;

        userAgent = connector.CreateUserAgent("BASIC_SERVICE", 1);

        // 프로토콜 등록
        ProtocolManager.getInstance().RegisterProtocol(1, PuzzleReflection.Descriptor);
    }
  
  	...생략...
}
```

<br>
### 8.3. 클라이언트 측 전송 구현

이제 게임을 위한 프로토콜 정의 및 등록까지 모두 마쳤습니다. 지금부터는 이러한 프로토콜에 기반한 메시지를 실제 전송하는 구현을 합니다. 우선 퍼즐 조각을 드래그 하는 동안 그 위치를 서버로 전송해 보겠습니다. Puzzle.cs파일을 열어서 아래와 같이 코드를 추가합니다.

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;
using Protocol;
using GameAnvil;
using GameAnvil.Defines;
using GameAnvil.Connection;
using GameAnvil.Connection.Defines;
using GameAnvil.User;

public class Puzzle : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public int index;

    public Transform puzzleHolder;
    public float tolerance;

  
    private ConnectHandler connectHandler;

    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();
    }

    public void OnBeginDrag(PointerEventData eventData)
    {
        transform.position = eventData.position;
    }
  
    public void OnDrag(PointerEventData eventData)
    {
        transform.position = eventData.position;
        SendPuzzlePosition(transform.position, false);
    }
  
    public void OnEndDrag(PointerEventData eventData)
    {
        FixPosition();
        SendPuzzlePosition(transform.position, true);
    }

    public void FixPosition(){
        if (Vector2.Distance(transform.position, puzzleHolder.position) < tolerance){
            transform.position = puzzleHolder.position;
        }
    }

    public void SendPuzzlePosition(Vector2 puzzlePosition, bool onEndDrag){
		PuzzlePosition position = new PuzzlePosition();
		position.Index = index;
        position.PositionX = (int)puzzlePosition.x;
        position.PositionY = (int)puzzlePosition.y;
        position.OnEndDrag = onEndDrag;
      
        connectHandler.userAgent.Send(position);
    }
}


```

퍼즐의 고유한 번호와 퍼즐 위치 정보를 담은 메시지를 생성하여 userAgent를 통해 서버로 전송합니다. 이 과정을 퍼즐 조각이 드래그되는 동안 반복하게 합니다.

<br>

### 8.4. 서버 측 응답 구현

퍼즐의 위치 정보를 받아서 처리하는 핸들러를 작성합니다. src/main/java에 handler 패키지와 game 패키지를 추가하고, 기존 클래스들을 알맞은 패키지에 옮겨 놓습니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/puzzle_position_handler.png"/>

그리고 src/main/java/handler에 PuzzlePositionHandler.java 파일을 생성합니다.

```c#
package handler;

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomPacketHandler;
import com.nhn.gameanvil.packet.Packet;
import game.BasicRoom;
import game.BasicUser;

public class PuzzlePositionHandler implements RoomPacketHandler<BasicRoom, BasicUser> {

    @Override
    public void execute(BasicRoom room, BasicUser user, Packet packet) throws SuspendExecution {
        room.broadcast(packet);
    }
}
```

받은 패킷을 그대로 방 안의 모든 유저들에게 전달해줍니다.

이번에는 앞서 구현한 핸들러를 BasicRoom의 dispacher에 등록하도록 합니다.

```c#
public class BasicRoom extends BaseRoom<BasicUser> {
    private static final Logger logger = getLogger(BasicRoom.class);
    private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();
    private static Map<Integer, BaseUser> users = new HashMap<>();

    static {
        dispatcher.registerMsg(BasicProtocol.MessageRequest.getDescriptor(), BasicHandler.class);
        dispatcher.registerMsg(Puzzle.PuzzlePosition.getDescriptor(), PuzzlePositionHandler.class);
    }
  
    ...생략...
  
}
```

<br>

### 8.5. 클라이언트 측 수신 구현

서버에서 송신한 메시지를 받기 위해서는 미리 핸들러를 등록해야한다고 했습니다. GameManager의 Start 메서드에 PuzzlePosition 메시지를 처리하는 핸들러 등록 코드를 작성합니다. 이 핸들러는 메시지를 수신하면 퍼즐 객체를 찾아서 서버에서 받은 위치로 이동시킵니다.

```c#
public class GameManager : MonoBehaviour{
  
    void Start(){
        ...생략...
       	// 채팅 메시지 리스너
        connectHandler.userAgent.AddListener<MessageBroadcast>((sendUserAgent, messageBroadcast) => {
            ChatLogText.text += messageBroadcast.Message;
        });
			
        // 퍼즐 위치 리스너
        connectHandler.userAgent.AddListener<PuzzlePosition>((sendUserAgent, puzzlePosition) => {
          	Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);
          
            if ( puzzlePosition.OnEndDrag ){
                puzzle.FixPosition();
            }
        });
    }
    ... 생략 ...
}
```

<br>

### 8.6. 퍼즐 위치 동기화 확인

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 빌드된 게임에서 방을 생성한 후, Unity 플레이모드를 실행하여 생성된 방의 RoomId를 입력해서 방에 참가합니다. 이제 퍼즐 조각을 이동하면, 해당 퍼즐 조각의 위치가 클라이언트 사이에서 동기화 되는 것을 확인할 수 있습니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_puzzle_position_playmode.gif)

<br>

### 8.7. 중입 유저 처리하기

게임 중 임의의 퍼즐 조각 위치가 변경된 후에 새로운 유저가 방에 입장할 경우를 생각해 봅시다. 이 때, 기존 유저와 신규 유저 사이의 퍼즐 상태는 서로 다릅니다. 새롭게 들어온 유저는 초기의 퍼즐 상태를 가지고 있으므로 기존 유저와 퍼즐 상태를 동기화 해주어야 합니다. 이를 해결하기 위해서 서버측 로직을 수정해 주겠습니다. 이제 서버는 퍼즐의 위치 정보를 모두 보관하고, 새로운 유저가 입장하면 해당 정보를 이용하여 동기화 하도록 로직을 수정합니다.

BasicRoom에 puzzlePositions 맵을 추가합니다. 이 맵은 각 퍼즐 조각별 위치 정보를 관리합니다.

```c#
public class BasicRoom extends BaseRoom<BasicUser> {
    private static final Logger logger = getLogger(BasicRoom.class);
    private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();
  
    private static Map<Integer, BaseUser> users = new HashMap<>();
    public static Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();
  
  	...생략...
  
}
```

이제 핸들러 코드를 수정해서 퍼즐 위치를 맵에 저장하도록 합니다.

```Java
package handler;

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomPacketHandler;
import com.nhn.gameanvil.packet.Packet;
import game.BasicRoom;
import game.BasicUser;
import org.slf4j.Logger;
import protocol.Puzzle;

import java.io.IOException;

import static org.slf4j.LoggerFactory.getLogger;

public class PuzzlePositionHandler implements RoomPacketHandler<BasicRoom, BasicUser> {
    private static final Logger logger = getLogger(PuzzlePositionHandler.class);
    @Override
    public void execute(BasicRoom room, BasicUser user, Packet packet) throws SuspendExecution {
        try {
            Puzzle.PuzzlePosition position = Puzzle.PuzzlePosition.parseFrom(packet.getStream());

            room.puzzlePositions.put(position.getIndex(), position); // 맵에 위치 정보를 저장

            room.broadcast(packet);
        } catch (IOException e) {
            logger.error("PuzzlePositionHandler::execute()", e);
        }
    }
}

```

또한 BasicRoom의 onJoinRoom을 수정해서 새로운 유저가 방에 들어올 때, 서버에 저장해둔 퍼즐의 위치 정보를 전송하도록 합니다.

```c#
public class BasicRoom extends BaseRoom {
    @Override
    public boolean onJoinRoom(BasicUser basicUser, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(basicUser.getUserId(), basicUser);

        puzzlePositions.values().stream().forEach( puzzlePosition -> broadcast(new Packet(puzzlePosition))); // 퍼즐 위치 동기화

        logger.debug("onJoinRoom(userId:{})", basicUser);
        return true;
    }
}
```

이렇게 수정한 후 다시 테스트 해봅니다. 여전히 우리가 의도한대로 완벽하게 동작하지 않는 것을 확인할 수 있습니다. 이는 onJoinRoom 호출 이후에 클라이언트에서 씬 이동이 일어나기 때문입니다. 이 때, 리스너 등록이 채 완료되기 전에 동기화 메시지가 전달되면 문제가 됩니다. 이 문제를 해결하기 위해서는 클라이언트에서 씬 이동이 끝나고 리스너 등록까지 마친 이후에 퍼즐의 위치 정보를 동기화 해야합니다.

이러한 문제는 바로 다음에 설명할 퍼즐 섞기에서도 여실히 드러납니다. 그러므로 퍼즐 섞기부터 살펴본 후, 이를 해결하기 위해 수정한 내용을 다시 다루도록 합니다.

<br>

## 9. 퍼즐 섞기 구현

퍼즐 위치를 랜덤으로 섞는 로직을 구현해보겠습니다. 클라이언트에서 퍼즐 위치 섞기를 요청하면 서버에서 퍼즐을 섞은 후 새로운 위치를 결정합니다. 그리고 변경된 위치 정보를 다시 클라이언트로 되돌려주는 것이 기본 아이디어입니다.

### 9.1. 프로토콜 등록

우선 퍼즐 섞기 요청을 하기 위한 프로토콜을 Puzzle.proto에 추가합니다. 이 프로토콜은 특별히 클라이언트에서 서버로 보낼 정보가 없으므로 필드가 하나도 없습니다. 이 또한 프로토콜로서 충분히 유의미합니다.

```c#
syntax="proto3";

package protocol;

message PuzzlePosition{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}

message ScatterPuzzle { 
} // 퍼즐 섞기 요청 프로토콜
```

프로토콜 버퍼 파일을 수정하면 컴파일도 다시해주어야합니다.

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

생성된 C# 클래스를 다시 Finder 등의 프로그램을 이용해서 Unity 프로젝트의 Asset/Protocol폴더로 옮깁니다. 이 때, 서버는 출력 경로 덕분에 새롭게 컴파일 된 클래스로 자동으로 대체됩니다.

<br>

### 9.2. 클라이언트 측 구현

GameManager.cs에 아래와 같이 섞기 요청을 위한 코드를 작성합니다.

```
public GameManager : Monobehaviour{
		public void Scatter(){
        	connectHandler.userAgent.Send(new ScatterPuzzle());
    }
}
```

씬 상의 ScatterPuzzle 버튼의 OnClick 리스너에 GameManager 컴포넌트를 드래그해서 등록하고, 드롭다운에서 Scatter메서드를 선택합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/unity_scatter_puzzle_on_click.png"/>

### 9.3. 서버 측 구현

섞기 요청이 들어왔을 때의 처리 핸들러 ScatterPuzzleHandler를 작성합니다.

<img src="https://static.toastoven.net/prod_gameanvil/images/tutorial/scatter_puzzle_handler.png"/>

```Java
package handler;

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomPacketHandler;
import com.nhn.gameanvil.packet.Packet;
import game.BasicRoom;
import game.BasicUser;
import org.slf4j.Logger;
import protocol.Puzzle;

import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

import static org.slf4j.LoggerFactory.getLogger;

public class ScatterPuzzleHandler implements RoomPacketHandler<BasicRoom, BasicUser> {
    private static final Logger logger = getLogger(ScatterPuzzleHandler.class);
    private static final int mapSize = 400;

    private class Point {
        public int x;
        public int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    @Override
    public void execute(BasicRoom room, BasicUser user, Packet packet) throws SuspendExecution {
        room.puzzlePositions = new HashMap<>();
        List<Point> random = IntStream.rangeClosed(-4, 4).boxed()
                .map(i -> new Point(i * mapSize / 4, i%2==0 ? mapSize : -mapSize))
                .collect(Collectors.toList());
        random = Stream.concat(random.stream(), random.stream().map(p -> new Point(p.y, p.x))).collect(Collectors.toList());
        Collections.shuffle(random);

        for (int i = 0; i < 16; i++) {
            Point point = random.get(i);
            int x = 1300 + point.x;
            int y = 550 + point.y;

            Puzzle.PuzzlePosition puzzlePosition = Puzzle.PuzzlePosition.newBuilder().setIndex(i).setPositionX(x).setPositionY(y).build();

            room.puzzlePositions.put(i, puzzlePosition);

            room.broadcast(new Packet(puzzlePosition));
        }
    }
}

```

PuzzleScatter 타입의 패킷이 들어오면, 16개 각 퍼즐의 위치를 랜덤하게 설정한 후 PuzzlePositon 타입의 메시지를 송신합니다. 또한 서버의 puzzlePositions 맵도 새로운 위치 정보로 갱신합니다.

이렇게 구현한 핸들러를 Room에 등록 해줍니다.

```java
public class BasicRoom extends BaseRoom<BasicUser> {
    private static final Logger logger = getLogger(BasicRoom.class);
    private static RoomPacketDispatcher dispatcher = new RoomPacketDispatcher();

    private static Map<Integer, BaseUser> users = new HashMap<>();
    public static Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();

    static {
        dispatcher.registerMsg(BasicProtocol.MessageRequest.getDescriptor(), BasicHandler.class);
        dispatcher.registerMsg(Puzzle.PuzzlePosition.getDescriptor(), PuzzlePositionHandler.class);
        dispatcher.registerMsg(Puzzle.ScatterPuzzle.getDescriptor(), ScatterPuzzleHandler.class);
    }
  
    ...생략...  
}
```

<br>

### 9.4. 퍼즐 섞기 기능 확인

Unity를 테스트 모드로 실행합니다. Scatter Puzzle 버튼을 클릭하여 퍼즐 섞기 기능이 잘 작동하는지 확인합니다.

<br>

## 10. 더 나은 중입 유저 처리

앞에서 중입 유저를 처리하는 과정에서 동기화 문제가 발생했습니다. 그 원인은 유저가 방에 입장하는 시점을 퍼즐 조각의 위치 동기화 시점으로 적합하다고 생각했기 때문입니다. 이는 게임 클라이언트의 구현에 따라 문제가 없을 수도 있고, 문제가 될 수도 있습니다. 하지만 우리가 구현 중인 퍼즐 게임은 유저가 방에 입장하는 시점에 씬 이동이 발생합니다. 그러므로 우리는 리스너 등록과 onJoinRoom 콜백 호출의 두 가지 시점으로 나누어서 생각해야합니다. onJoinRoom 콜백 호출 이후 씬 이동이 시작되기 때문에 씬 이동이 완료된 직후에 클라이언트가 직접 서버로 퍼즐 위치 동기화를 요청하고자 합니다.

<br>

### 10.1. 프로토콜 등록

퍼즐 위치 동기화를 요청 하기 위한 프로토콜을 Puzzle.proto에 추가합니다.

```c#
syntax="proto3";

package protocol;

message PuzzlePosition{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}

message ScatterPuzzle {
}

message PuzzlePositionReq { // 퍼즐 위치 동기화 요청
}
```

프로토콜을 다시 컴파일 한 후 생성된 C# 클래스를 Unity 프로젝트로 옮깁니다.

<br>

### 10.2. 클라이언트 측 구현

씬 이동 직후 서버에 퍼즐 위치를 요청하도록 GameManager를 다음과 같이 수정합니다.

```c#
public class GameManager : MonoBehaviour
{
    void Start(){
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

      
        ProtocolManager.getInstance().RegisterProtocol(0, BasicProtocolReflection.Descriptor);

        connectHandler.userAgent.AddListener<MessageBroadcast>((sendUserAgent, messageBroadcast) => {
            ChatLogText.text += messageBroadcast.Message;
        });

        connectHandler.userAgent.AddListener<PuzzlePosition>((sendUserAgent, puzzlePosition) => {
          	Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);
          
            if ( puzzlePosition.OnEndDrag.Equals("True")){
                puzzle.FixPosition();
            }
        });

        connectHandler.userAgent.Send(new PuzzlePositionReq());
    }
  
  	...생략...
}
```

<br>

### 10.3. 서버 측 구현

onJoinRoom에 구현했던 퍼즐 위치 송신 코드를 PuzzlePositionReqHandler에 새롭게 작성합니다.

```java
package handler;

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.node.game.RoomPacketHandler;
import com.nhn.gameanvil.packet.Packet;
import game.BasicRoom;
import game.BasicUser;
import protocol.Puzzle;

import java.io.IOException;

import static org.slf4j.LoggerFactory.getLogger;

public class PuzzlePositionReqHandler implements RoomPacketHandler<BasicRoom, BasicUser> {
    @Override
    public void execute(BasicRoom room, BasicUser user, Packet packet) throws SuspendExecution {
        room.puzzlePositions.values().stream().forEach( puzzlePosition -> room.broadcast(new Packet(puzzlePosition)));
    }
}
```

그리고 작성한 핸들러를 Room에 등록합니다.

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    static {
        dispatcher.registerMsg(BasicProtocol.MessageRequest.getDescriptor(), BasicHandler.class);
        dispatcher.registerMsg(Puzzle.PuzzlePosition.getDescriptor(), PuzzlePositionHandler.class);
        dispatcher.registerMsg(Puzzle.ScatterPuzzle.getDescriptor(), ScatterPuzzleHandler.class);
        dispatcher.registerMsg(Puzzle.PuzzlePositionReq.getDescriptor(), PuzzlePositionReqHandler.class);
    }
  
    ...생략...
  
}
```

<br>

### 10.4. 중입 유저 처리 확인

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 자, 이제 빌드된 게임에서 방을 생성하고 퍼즐 섞기를 합니다. 그 상태에서 Unity 플레이모드를 실행하여 이 방에 참가한 후 퍼즐 위치가 동기화 되는 것을 확인합니다.

<br>

## 11. 유저 매치메이킹 구현

### 11.1 서버 측 구현

유저 매치메이킹은 유저들의 매치메이킹 요청을 한데 모아서 적절한 기준에 맞춰 비슷한 수준의 유저들끼리 서로 같은 방에서 게임을 시작할 수 있게 해줍니다. 승점이나 점수 등 다양한 요소를 사용자가 직접 구현해서 이러한 유저들을 적절하게 구분하고 매칭할 수 있습니다. 이 튜토리얼에서는 유저 2명을 하나의 게임으로 매칭해주는 로직을 구현해보겠습니다.

match라는 이름의 패키지를 생성하고, UserMatchInfo 클래스를 생성합니다. 파일명은 BasicUserMatchInfo으로 합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_user_match_info.png)

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/user_match_info.png)

이 클래스에는 매칭에 사용될 유저의 정보를 담게 됩니다. 매치메이킹에 사용될 요소가 있다면 여기에 추가하면 됩니다. 이번 예제에서는 별다른 요소를 추가하지 않고, 필수적으로 구현해야하는 메서드만을 작성해서 사용하겠습니다. 한 가지 주의할 점은 getId() 메서드가 반드시 요청한 유저의 아이디를 반환하게 구현하도록 합니다. 그리고 파티 매치메이킹 기능은 사용하지 않으므로 0을 반환하도록 설정합니다.

```java
package match;

import com.nhn.gameanvil.node.match.BaseUserMatchInfo;

import java.io.Serializable;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicUserMatchInfo extends BaseUserMatchInfo implements Serializable {

    private int id;

    public BasicUserMatchInfo(int id) {
        this.id = id;
    }
  
    @Override
    public int getId() {
        return id;
    }

    @Override
    public int getPartySize() {
        return 0;
    }
}
```

이러한 UserMatchInfo는 클라이언트가 유저 매치메이킹 요청을 할 때, 서버의 게임 유저에서 onMatchUser 콜백을 구현하는 과정에서 생성한 후 사용합니다. GameAnvil은 기본적인 유저 매치메이커를 제공합니다. 아래의 onMatchUser는 이러한 엔진의 기본 유저 매치메이킹을 matchUser API를 통해 사용하고 있습니다.

```java
package game;

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.exceptions.NodeNotFoundException;
import match.BasicUserMatchInfo;
import org.slf4j.Logger;

import java.util.concurrent.TimeoutException;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@UserType("BASIC_USER")
public class BasicUser extends BaseUser {
    private static final Logger logger = getLogger(BasicUser.class);

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, Payload payload, Payload outPayload) throws SuspendExecution {
        try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(getUserId());
            return matchUser(matchingGroup, roomType, term, payload); // 유저 매치 메이킹 요청          
        } catch (NodeNotFoundException | TimeoutException e) {
            logger.error("BasicUser::onMatchUser()", e);
        }
        return false;
    }
}

```

유저 매치메이킹을 사용하기 위한 기본적인 준비가 되었으면, 이제 실제 매치메이킹을 수행하는 매치메이커를 작성합니다. 아래와 같이 match 패키지에 BasicUserMatchmaker 클래스를 추가합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_user_match_maker.png)

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/basic_user_match_maker.png)

생성자에서는 부모 클래스의 생성자를 호출하면서 인자로 매치 인원수와 매치 신청 유효시간을 전달합니다. 유효 시간이 지나면 해당 매치 요청은 자동으로 취소됩니다. 그리고 실제 매치메이킹을 수행하는 match 메소드는 엔진에 의해 1초에 한번씩 호출됩니다.

getMatchRequests는 인자로 매칭을 위한 최소 인원수를 받아서 현재 매칭 풀 전체를 조회하여 최적의 매칭을 가능한 만큼 만들어 냅니다 . 즉, 매칭 요청이 많이 쌓여 있다면 getMatchRequests에 의해 한 번에 100개 혹은 1000개의 매칭도 만들어질 수 있습니다. 이 때, 매칭이 성공하면 매칭된 유저들의 UserMatchInfo 목록을 반환합니다. 이 목록을 엔진에서 제공하는 matchSingles API에 전달하면 해당 목록 안의 유저들에 대한 방 생성 및 이동이 알아서 진행됩니다.

만일 매칭에 충분한 요청이 쌓이지 않았거나, 조건에 맞는 대상이 없을 경우에는 null을 반환합니다. 이 경우에는 다음 1초 후의 match 호출에서 다시 동일한 매칭 검색이 수행됩니다.

```Java
package match;

import com.nhn.gameanvil.node.match.BaseUserMatchMaker;
import org.slf4j.Logger;

import java.util.List;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicUserMatchMaker extends BaseUserMatchMaker<BasicUserMatchInfo> {
    private static final Logger logger = getLogger(BasicUserMatchMaker.class);
		private static final int NUM_USER = 2;
  	private static final int TIMEOUT = 10000;
  
    public BasicUserMatchMaker() {
        super(NUM_USER, TIMEOUT);
    }

    @Override
    public void onMatch() {
        List<BasicUserMatchInfo> matchRequests = getMatchRequests(2);

        if (matchRequests == null) {
            return;
        }

        matchSingles(matchRequests);
    }

    @Override
    public boolean onRefill(BasicUserMatchInfo roomInfo) {
        return false;
    }
}
```

<br>

### 11.2. 클라이언트 측 구현

매치메이킹 로직은 모두 서버에 구현되어있기 때문에 클라이언트에서는 매치메이킹이 필요한 시점에 요청을 보내기만 하면 됩니다. ConnectHandler에 UserMatchMaking 메서드를 추가합니다. 그리고 매치메이킹이 끝난 시점에 씬을 이동하도록 하는 핸들러를 추가합니다.

```csharp
public class ConnectHandler : MonoBehaviour
{

		...생략...
	
    void Start () {
  
        ...생략...
      
        userAgent.onMatchUserDoneListeners += (UserAgent userAgent, ResultCodeMatchUserDone resultCode, bool created, int roomId, Payload payload) =>{
            this.roomId = roomId;
            SceneManager.LoadScene("GameScene");
        };
    }
	
		...생략...

    public void UserMatchMaking(){
        userAgent.MatchUserStart("BASIC_ROOM", "BASIC_MATCHING_GROUP",(UserAgent userAgent, ResultCodeMatchUserStart resultCode, Payload payload) =>{
            Debug.Log(resultCode);
        });
    }
}

```

씬에서 User Match Making 버튼의 OnClick 리스너에 ConnectHandler 컴포넌트를 드래그해서 등록하고, 드롭다운에서 UserMatchMaking메서드를 선택합니다.

<br>

### 11.3. 유저 매치메이킹 테스트

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 그 상태로 Unity 플레이모드를 실행합니다. 양측에서 모두 User Match Making 버튼을 눌러 매칭이 성사되고 동일한 방 번호로 묶이게 되는 것을 확인합니다.

<br>

## 12. 룸 매치메이킹 구현

룸 메치메이킹은 매치메이커가 관리하는 방들 중에서 유저의 요구사항에 가장 적합한 방으로 자동 입장시킬 수 있는 기능입니다. 즉, 유저 매치메이킹이 유저와 유저를 매칭시켜주는 기능이라면, 룸 매치메이킹은 유저와 방을 매칭시켜주는 기능입니다. 이 때, 구현 방식에 따라서 다양한 조건으로 유저를 방에 매칭시켜 줄 수 있습니다. 이 튜토리얼에서는 아직 정원이 차지 않은 방 중에 인원이 가장 적은 방으로 입장하는 매치메이킹을 구현해보겠습니다.

우선 매치메이킹을 실제로 수행하는 클래스가 필요합니다. 그리고 룸 매치메이커에서 방을 관리하기 위한 정보를 담는 인스턴스가 각 방마다 하나씩 있어야합니다. 마지막으로 유저가 매치메이킹을 신청할 때 마다 유저의 요구 사항을 담은 신청서 인스턴스가 필요합니다. 이렇게 세 개의 새로운 클래스를 작성해보도록 하겠습니다.

추가로, 기존 로직을 약간 수정하게 됩니다. 룸 매치메이킹은 모든 방에 대해서 수행되지 않습니다. 룸 매치메이킹 대상으로 신청한 방들만을 대상으로 수행됩니다. 따라서 방 생성 시점에 룸 매치메이킹 대상으로 신청하는 코드를 추가할 것입니다.

<br>

### 12.1. 서버 측 구현

우선 매치메이킹 요청을 나타낼 클래스를 구현합니다. match 패키지 안에 BasicRoomMatchForm 클래스를 생성합니다. 

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_maker_form.png)

유저가 매치메이킹 요청을 할 때마다 BasicRoomMatchForm 객체가 생성되어 사용됩니다. 생성자에서 부모 생성자로 전달하는  "BASIC_MATCHING_USER_CATEGORY"는 이 문서에서는 신경쓰지 않아도 됩니다.

```java
package match;

import java.io.Serializable;

public class BasicRoomMatchForm extends BaseRoomMatchForm implements Serializable {
    public BasicRoomMatchForm() {
        super("BASIC_MATCHING_USER_CATEGORY");
    }
}
```



다음은 매칭 대상이 되는 방의 정보를 표현하는 클래스를 구현합니다. 다음과 같이 BasicRoomMatchInfo 클래스를 생성합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_info.png)

이 때, 방의 최대 정원은 static 필드에 지정된 4명입니다. 이러한 최대 정원과 유저 타입을 반드시 상속받은 BaseRoomMatchInfo의 생성자에 인자로 전달해야 합니다.

```java
package match;

import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.match.BaseRoomMatchInfo;

import java.io.Serializable;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoomMatchInfo extends BaseRoomMatchInfo implements Serializable {
    private static final int MAX_USER = 4;

    public BasicRoomMatchInfo(int roomId) {
        super(roomId, "BASIC_USER", MAX_USER);
    }
}
```



자, 다음은 실제로 룸 매치메이킹을 처리할 룸 매치메이커를 생성합니다.

![](https://static.toastoven.net/prod_gameanvil/images/tutorial/new_room_match_maker.png)

다음과 같이 BasicRoomMatchMaker를 작성합니다.

```Java
package match;

import com.nhn.gameanvil.annotation.RoomType;
import com.nhn.gameanvil.annotation.ServiceName;
import com.nhn.gameanvil.node.game.data.RoomMatchResultCode;
import com.nhn.gameanvil.node.match.BaseRoomMatchMaker;

@ServiceName("BASIC_SERVICE")
@RoomType("BASIC_ROOM")
public class BasicRoomMatchMaker extends BaseRoomMatchMaker<BasicRoomMatchForm, BasicRoomMatchInfo> {
    @Override
    public RoomMatchResultCode onPreMatch(BasicRoomMatchForm basicRoomMatchForm, Object... objects) {
        return RoomMatchResultCode.SUCCESS;
    }

    @Override
    public boolean canMatch(BasicRoomMatchForm basicRoomMatchForm, BasicRoomMatchInfo basicRoomMatchInfo, Object... objects) {
        return true;
    }

    @Override
    public int compare(BasicRoomMatchInfo o1, BasicRoomMatchInfo o2) {
        int o1UserCount = getUserCount(o1.getRoomId());
        int o2UserCount = getUserCount(o2.getRoomId());
        if (o1UserCount > o2UserCount) {
            return -1;
        } else if (o1UserCount < o2UserCount) {
            return 1;
        } else {
          return 0;
        }
    }
    
    @Override
    public void onIncreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {
    
    }

    @Override
    public void onDecreaseUserCount(int roomId, String matchingUserCategory, int currentUserCount) {

    }
    
}
```

compare 메서드는 매칭 풀에 들어있는 방을 정렬하는 조건을 구현합니다. 예제는 인원수에 따라 방을 정렬하도록 구현했습니다.

이제 룸 매치메이킹을 위한 준비가 거의 끝났습니다. 클라이언트가 룸매치 요청을 보내면, 서버의 BasicUser는 onMatchRoom 콜백이 호출됩니다. 이 콜백에서 우리는 앞서 살펴보았던 BasicRoomMatchForm 객체를 생성한 후, matchRoom API에 인자로 전달하여 호출합니다. 즉, 클라이언트가 보낸 룸 매칭 요청을 여기에서 매치메이커로 전달했습니다.

```java
package game;

import co.paralleluniverse.fibers.SuspendExecution;
import com.nhn.gameanvil.exceptions.NodeNotFoundException;
import match.BasicUserMatchInfo;
import org.slf4j.Logger;

import java.util.concurrent.TimeoutException;

import static org.slf4j.LoggerFactory.getLogger;

@ServiceName("BASIC_SERVICE")
@UserType("BASIC_USER")
public class BasicUser extends BaseUser {

	@Override
	public final RoomMatchResult onMatchRoom(final String roomType, final String matchingGroup, final String matchingUserCategory, final Payload payload) throws SuspendExecution {
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();

        try {
            return matchRoom(matchingGroup, roomType, gameRoomMatchForm); // 매치메이커로 요청 전달
        } catch (Exception e) {
            logger.error("BasicUser::onMatchRoom()", e);
        }
        return RoomMatchResult.FAILED;
    }
}
```

방이 생성된 시점에 룸 매치메이킹 대상으로 만들기 위해서 BasicRoom의 onCreateRoom 콜백에서 아래와 같이 registerRoomMatch API를 호출합니다.

```java
public class BasicRoom extends BaseRoom<BasicUser> {
    ...생략...

    @Override
    public boolean onCreateRoom(BasicUser basicUser, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(basicUser.getUserId(), basicUser);

        BasicRoomMatchInfo gameRoomMatchInfo = new BasicRoomMatchInfo(getId());
        try {
            registerRoomMatch(gameRoomMatchInfo, "BASIC_MATCHING_USER_CATEGORY", basicUser.getUserId(), users.size()); // 방을 룸 매치메이킹 대상으로 등록
        } catch (Exception e) {
            logger.error("BasicRoom::onCreateRoom():registerRoomMatch", e);
        }

        logger.debug("onCreateRoom(userId:{})", basicUser);
        return true;
    }
  
}
```

그리고 방의 정보 변동이 있을 때마다 룸 매치메이킹 정보도 갱신되어야 합니다. 참고로 방의 인원수 변동은 룸 매치메이커가 알아서 동기화를 해주고 있습니다. 그러므로 인원수를 제외한 추가 정보에 대한 갱신만 수행하면 됩니다.

다음은 onJoinRoom 콜백을 수정해서 방에 유저가 참여할 때 매치메이킹 정보를 갱신하는 코드입니다.

```java
public class BasicRoom extends BaseRoom<BasicUser> {
    ...생략...

    @Override
    public boolean onJoinRoom(BasicUser basicUser, Payload inPayload, Payload outPayload) throws SuspendExecution {
        users.put(basicUser.getUserId(), basicUser);

        BasicRoomMatchInfo gameRoomMatchInfo = new BasicRoomMatchInfo(getId());
        try {
            updateRoomMatch(gameRoomMatchInfo); 
        } catch (Exception e) {
            logger.error("BasicRoom::onCreateRoom():registerRoomMatch", e);
        }

        logger.debug("onJoinRoom(userId:{})", basicUser);
        return true;
    }
}
```

<br>

### 12.2. 클라이언트 구현

유저 매치메이킹과 마찬가지로 클라이언트는 매치메이킹이 필요한 시점에 요청을 보내기만 하면 됩니다. ConnectHandler에 RoomMatchMaking 메서드를 추가합니다.

```csharp
public class ConnectHandler : MonoBehaviour {
    public void RoomMatchMaking(){
        userAgent.MatchRoom("BASIC_ROOM", "BASIC_MATCHING_GROUP", "BASIC_MATCHING_USER_CATEGORY", true, 
            (UserAgent userAgent, ResultCodeMatchRoom resultCode, int integer, int roomId, string roomName, bool created, Payload payload) =>{
            Debug.Log(resultCode);
            if (resultCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS){
            	this.roomId = roomId;

            	SceneManager.LoadScene("GameScene");
            }
        });
    }
}

```

씬 상의 Room Match Making 버튼의 OnClick 리스너에 ConnectHandler 컴포넌트를 드래그해서 등록하고, 드롭다운에서 RoomMatchMaking메서드를 선택합니다.

매칭 그룹이 있는 방을 만들기 위해서, 방 생성 코드를 수정합니다. 이 때, 매칭 그룹은 매치메이킹 대상 방을 논리적으로 나누기 위해 서버와 클라이언트 사이에 사전 정의한 임의의 문자열입니다. 여기에서는 "BASIC_MATCHING_GROUP"이라는 문자열을 사용합니다.

```c#
public class ConnectHandler : MonoBehaviour {
	public void CreateRoom(){
        userAgent.CreateRoom("", "BASIC_ROOM", "BASIC_MATCHING_GROUP", null (UserAgent ua, ResultCodeCreateRoom resultCode, int roomId, string roomName, Payload payload) => {
            Debug.Log(resultCode); 
            if (resultCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS){
                this.roomId = roomId;
              
                SceneManager.LoadScene("GameScene");
            }
        });
    }
}
```

<br>

### 11.3. 룸 매치메이킹 테스트

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이 상태에서 방을 생성합니다. 그 상태로 Unity 플레이모드를 실행합니다. 플레이모드에서 Room Match Making 버튼을 눌러 빌드 모드에서 생성한 방으로 이동하는지 확인합니다.

<br>

## 13. Room 떠나기 구현

마지막으로 방을 떠나는 기능 구현을 위해 Unity 클라이언트의 GameManager에 아래 메서드를 추가합니다.

```c#
public void LeaveRoom(){
		connectHandler.userAgent.LeaveRoom((userAgent, resultCode, force, roomId, payload) =>{
      	if(resultCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS){
      		Destroy(connectHandler.gameObject);
    			SceneManager.LoadScene("ConnectScene");
    		}
    });
}
```

씬 상의 Leave Room 버튼의 OnClick 리스너에 GameManager 컴포넌트를 드래그해서 등록하고, 드롭다운에서 LeaveRoom메서드를 선택합니다.

첫 번째 씬으로 돌아왔을 때 connectHandler 객체가 중복해서 생성되지 않도록 하기 위해 코드를 추가합니다.

```
public class ConnectHandler : MonoBehaviour
{
    static ConnectHandler instance;

    private void Awake() {
        if (instance != null){
            Destroy(gameObject);
        } else {
            instance = this;
        }
        DontDestroyOnLoad(this.gameObject);
        connector = new GameAnvil.Connector(config);
    }
```

## 14. 프로젝트 마무리

이상 GameAnvil과 Unity를 이용하여 실시간 멀티플레이가 가능한 퍼즐 게임을 구현해 보았습니다. 그 과정에서 GameAnvil의 핵심 기능 다수를 사용해보았습니다. 하지만, GameAnvil에는 이 튜토리얼에 포함 되지 않은 더욱 풍성하고 다양한 기능들이 존재합니다. 이러한 기능에 대해서는 이어지는 문서를 참고하세요. 또한 함께 제공되는 레퍼런스 샘플 프로젝트와 JavaDoc도 GameAnvil을 이해하는데 많은 도움이 될 것입니다.
