## Game > GameAnvil > 심화 튜토리얼

### GameAnvil로 게임 서버 쉽게 만들기

GameAnvil은 실시간 멀티플레이어 게임 서버 제작 플랫폼입니다.
GameAnvil은 서버 엔진뿐만 아니라, 서버에 클라이언트를 연결하기 위한 커넥터를 제공합니다.
GameAnvil을 사용하면 손쉽게 게임 서버와 클라이언트를 개발하고 운영할 수 있습니다.

이 문서는 GameAnvil의 기본적인 기능과 더불어, 사용자가 정의한 프로토콜을 이용하여 인게임 채팅 및 실제 플레이가 가능한 멀티플레이어 퍼즐 게임을 개발하는 과정을 다룹니다.

서버와 클라이언트가 상호 작용하는 모습을 볼 수 있는 샘플을 완성해 나가면서 GameAnvil을 사용하여 게임을 개발하는 전체적인 흐름에 익숙해질 수 있습니다.

제공된 템플릿을 활용하여 간단히 서버를 구축하고, GameAnvil 커넥터가 제공하는 기능을 통해 Unity 클라이언트에서 채팅, 멀티플레이어 동기화, 매치메이킹 등을 구현할 수 있습니다.

이 문서는 서버의 개념과 API를 단순 나열하는 대신 좀 더 구체적인 예시를 통해 설명하고 이해를 돕기 위해 실제 플레이가 가능한 멀티플레이어 직소 퍼즐 게임을 개발하는 과정을 순서대로 설명합니다. 문서의 내용을 직접 하나씩 따라하며 자연스럽게 GameAnvil과 멀티플레이어 게임 개발에 대한 이해도를 높일 수 있습니다.

## 프로젝트 구성

멀티플레이어 게임을 만들려면 클라이언트와 대응하는 서버 프로그램이 필요합니다. 이 예제에서는 클라이언트 프로그램 제작에 Unity와 GameAnvil 커넥터를, 서버 프로그램 제작에 앞서 소개한 서버 엔진 GameAnvil을 사용합니다. 먼저 GameAnvil을 이용한 서버 프로그램 프로젝트를 생성한 뒤 Unity와 GameAnvil 커넥터를 이용해 클라이언트 프로그램 프로젝트를 생성합니다.

아래 단계를 진행하면 만들어지는 최종 서버 샘플 프로젝트는 아래 링크에서 다운로드할 수 있습니다. 초기 템플릿에서 여러 단계를 거쳐 서버 기능을 구현하면 어떤 구조가 되는지 미리 확인하려면 해당 프로젝트를 내려받아 참고할 수 있습니다.

[서버 샘플 프로젝트 다운로드](https://static.toastoven.net/prod_gameanvil/files/tutorial/advanced-tutorial/2.1/GameAnvil_Tutorial_Advanced_Server.zip?disposition=attachment)

### GameAnvil 프로젝트 구성

프로젝트에 GameAnvil을 적용하려면 Maven 저장소에서 GameAnvil 라이브러리를 내려받고 GameAnvil을 구동하는 데 필수인 설정 파일을 작성해야 합니다. 마지막으로 약간의 보일러플레이트 코드를 작성하면 개발 초기 설정이 끝납니다. 이번 챕터에서는 개발을 시작하기 위한 초기 설정을 완료하는 것을 목표로 합니다. 실제 프로세스를 실행해 서버를 구동하는 것은 다음 챕터에서 다룹니다.

GameAnvil에서는 이와 같은 일련의 과정을 대신해 주는 IntelliJ 템플릿을 제공하여 보다 간단하게 초기 작업을 완료할 수 있습니다. 다음 링크에서 IntelliJ용 프로젝트 파일 템플릿을 다운로드할 수 있습니다. 다운로드한 템플릿은 압축을 풀지 않도록 합니다.

[템플릿 다운로드](https://static.toastoven.net/prod_gameanvil/files/v2_1/tutorial/basic-tutorial/GameAnvil%20Template.zip?disposition=attachment)

다운로드한 템플릿을 적용하기 위해 IntelliJ를 실행합니다. **Welcome to IntelliJ IDEA** 화면 좌측 메뉴에서 **Customize**를 선택한 뒤 **Import Settings...** 를 클릭합니다. 또는 전체 검색창에서 **Import Settings...** 를 검색합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/1_import_gameanvil_template.png)

<br>

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/2_search_import_settings.png)

<br>

파인더 또는 파일 탐색기에서 템플릿을 다운로드한 경로로 이동해 압축 파일을 선택합니다. **Select Components to Import** 창이 열리면 `File templates` 항목과 `Project Templates` 항목을 모두 체크해 선택합니다. **OK**를 클릭한 뒤 가져오기가 완료되면 IntelliJ를 다시 시작해 템플릿 적용을 완료합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/3_select_import.png)

IntelliJ 오른쪽 상단의 버튼 그룹에서 **New Project**를 클릭한 뒤 왼쪽 목록을 스크롤하여 하단의 **Templates**에 있는 `GameAnvil 2.0.0 Template`을 선택합니다. 프로젝트 이름을 설정합니다. 이름에 공백이 있어서는 안 됩니다. 프로젝트 위치를 확인한 뒤 프로젝트를 생성합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/4_imported_gameanvil_template.png)

이제 IntelliJ에 서버 프로젝트 골격이 구성되었습니다. Project 패널을 보면 코드와 설정 파일들이 생성된 것을 확인할 수 있습니다.

- Main: 프로그램의 진입점 Main 함수를 포함하는 클래스입니다.
- protocol 패키지: java로 컴파일된 프로토콜 버퍼 파일을 포함하는 패키지입니다.
- BasicProtocol.proto, Puzzle.proto: 프로토콜 버퍼를 이용해 작성된 프로토콜 파일입니다.
- GameAnvilConfig.json: GameAnvil 구동에 필요한 서버 설정 정보를 기록한 파일입니다. 서버 구현에 맞게 수정할 수 있습니다.
- logback.xml: Java 프로젝트에서 로깅을 구성하는 데 사용되는 파일입니다. Logback 프레임워크의 설정 파일로서, 로깅 시스템의 동작 방식과 로그의 형식, 저장 위치 등을 지정합니다. 이 파일을 사용하여 로깅 수준, 로그 형식, 로그 파일의 경로 및 이름, 로그 롤링 정책 등을 설정할 수 있습니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/5_gameanvil_project_view_init.png)

먼저 JDK 설정을 확인합니다. GameAnvil은 Java 21 버전을 지원합니다. 버전에 따라 일부 설정 방법이 다를 수 있으며, 여기에서는 Java 21 버전을 사용하였습니다.

왼쪽 상단 메뉴에서 **File > Project Structure**를 선택을 클릭합니다. Mac 사용자의 경우 `Command + ;` 단축키를 사용할 수 있습니다.

Project 탭에서 SDK 설정을 확인합니다. 만약 설정된 SDK가 없다면 `Add SDK > Download JDK`를 통해서 원하는 버전의 JDK를 다운로드해 설정합니다. Language level은 SDK default로 설정합니다. 다음으로 Modules 탭에서 Language level을 Project default로 설정합니다.

**Project** 탭에서 SDK 설정을 확인합니다. 만약 설정된 SDK가 없다면 **Add SDK > Download JDK**를 클릭해 원하는 버전의 JDK를 다운로드해 설정합니다. Language level은 `SDK default(Java 21)`로 설정합니다. 다음으로 **Modules** 탭에서 Language level을 `Project default (Java 21)`로 설정합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/6_project_structure.png)

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/7_module_language_level.png)

**설정** 메뉴에서 **gradle** 설정을 확인합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/8_gradle_config.png)

프로젝트 준비가 거의 끝났지만 실행을 위해서는 몇 가지 설정이 필요합니다. 여기에서는 우선 클라이언트 프로젝트를 먼저 생성한 뒤 서버 설정을 마치고 실행합니다.

### Unity 프로젝트 구성

Unity Hub를 실행합니다. 오른쪽 상단의 **NEW**를 클릭해 새로운 프로젝트 생성 창을 엽니다

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/9_unity_hub.png)

템플릿 목록에서 **2D**를 선택하고, 프로젝트 이름과 위치를 확인한 뒤 **CREATE**를 클릭해 프로젝트 생성을 완료합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/10_new_unity_project.png)

다음 링크에서 GameAnvil 커넥터를 내려받으십시오. 커넥터는 GameAnvil 서버와의 통신에 필요한 클라이언트 API를 제공하여 간단한 코드만으로 클라이언트를 구현할 수 있도록 도와주는 패키지입니다.

[gameanvil_connector_2.0.0.unitypackage](https://static.toastoven.net/prod_gameanvil/files/gameanvil-connector-2.0.0.unitypackage)

실습에 필요한 클라이언트 프로젝트 생성을 위해 아래 링크에서 튜토리얼용 코드와 이미지 소스 등이 포함된 Unity 패키지를 다운로드합니다.

[gameanvil_tutorial_advanced.unitypackage](https://static.toastoven.net/prod_gameanvil/files/tutorial/advanced-tutorial/gameanvil_tutorial_advanced.unitypackage)

다운로드한 패키지 파일을 Unity 프로젝트로 드래그해 가져옵니다. 또는 **Asset > Import Package > Custom Package...** 메뉴를 열어 파인더 또는 파일 탐색기에서 패키지 파일을 선택합니다. Import Unity Package 대화 상자에서 목록의 모든 체크 박스를 선택한 뒤 **Import**를 클릭합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/11_import_unity_package.png)

마지막으로, 편리하게 테스트하기 위해서는 Project Settings 윈도우에서 Player 탭을 선택하고, Resoultion and Presentation 항목에서 아래와 같이 프로젝트 기본 창 폭과 높이를 각각 1920, 1080으로 설정합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/12_unity_project_setting_resolution.png)

클라이언트 프로젝트 설정이 완료되었습니다.

## 서버 구동 및 연결

### GameAnvil 서버 구동

실행 설정이 완료되면 우측의 gradle 메뉴에서 Tasks > other > `runMain` 실행을 더블 클릭합니다. 이렇게 한 번 실행한 이후에는 IntelliJ 우측 상단의 초록색 삼각형 Run 아이콘을 클릭해도 서버가 실행됩니다.

`runMain`으로 실행을 해야 GameAnvil 서버 실행에 필요한 VM 옵션이 적용이 됩니다. 만약 Main 클래스의 main() 함수를 그냥 실행하는 경우에는, **Edit Configurations...**에서 아래의 필수 VM 옵션을 추가해주어야 합니다.

```
"--add-opens", "java.base/java.lang=ALL-UNNAMED",
"--add-opens", "java.base/java.lang.invoke=ALL-UNNAMED" 
"-XX:+UseG1GC"
```

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/13_gameanvil_run.png)

서버가 정상적으로 구동되면 서버 구동 상태 관련 로그들이 다수 출력됩니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/14_gameanvil_run_log.png)

GameAnvil 서버는 여러 개의 노드들로 구성되어 있습니다. 이 노드들은 서버가 수행할 기능을 여러 개의 역할로 분담합니다. 아직은 서버 초기 구동만 확인했을 뿐, 노드나 다른 서버 구동을 위한 코드 작성을 하지 않았기 때문에 완전히 준비된 상태는 아닙니다.

각각의 노드는 코드를 실행하기 위해 준비하는데 시간이 필요하며, 각 노드가 준비 완료되면 onReady 로그를 출력합니다. 클라이언트가 서버로 접속하는데 직접적인 역할을 수행하는 노드는 게이트웨이 노드입니다. 게이트웨이 노드가 준비되어 GatewayNode의 onReady 로그가 출력 되었다면 GameAnvil 서버는 언제든 접속이 가능한 상태가 된 것입니다.

### 커넥트 핸들러 작성

이제 Unity 프로젝트로 이동하여 GameAnvil 서버에 접속할 수 있도록 코드를 작성해 보겠습니다. 서버와 연결하려면 먼저 커넥터 객체를 생성해야 합니다.

Asset 패널에서 Scene 폴더 안의 Connect.scene을 더블 클릭해서 씬을 준비합니다. Hierarchy 뷰 상의 ConnectHandler 게임 오브젝트에 컴포넌트로 추가되어 있는 ConnectHandler 스크립트를 소스 코드 편집기에서 열어 구현 내용을 확인하며 각 과정을 직접 실습합니다.

GameAnvil 서버와 클라이언트는 주기적으로 상태 체크 패킷을 주고받아야 합니다. 클라이언트가 일시 중지된 상황 등에서 상태 체크 패킷을 주고받는 것을 멈추려면 PauseClientStateCheck() 메서드를 이용합니다. 상태 체크를 재개하려면 ResumeClientStateCheck() 메서드를 호출합니다. OnApplicationPause() 함수에서 클라이언트의 일시 중지를 감지해 알맞은 메서드를 호출합니다.

```c#
using System;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil;
using GameAnvil.Defines;
using Protocol;
using UnityEngine.SceneManagement;
using Random = UnityEngine.Random;

public class ConnectHandler : MonoBehaviour
{
    private static ConnectHandler instance;

    [Header("Object Reference")]
    public Text serverInfoText;
    public InputField serverInfoInput;
    public Text clientInfoText;
    public Text logText;

    public string ip = "127.0.0.1";
    public int port = 18200;

    public string deviceId = "";
    public string password = "";
    public string accountId = "";

    public int roomId;

    public GameObject popupCanvas;
    public InputField roomIdInput;

    private GameAnvilConnector connector;
    private GameAnvilUser user;

    void Start()
    {
        // 연결 정보를 화면에 출력합니다.
        serverInfoText.text = "서버정보:";
        serverInfoInput.text = ip;
    }

    private void Awake()
    {
        DontDestroyOnLoad(this.gameObject);
    }

    private void OnApplicationPause(bool pause)
    {
        if(!getConnector().IsConnected)
            return;

        if (pause)
        {
            // 입력한 시간동안 서버의 clientStateCheck 기능을 정지시킨다.
            // 이시간이 지나면 clientStateCheck 기능이 동작하여 연결이 끊어질 수 있다.
            getConnector().PauseClientStateCheck(10 * 60);
        }
        else
        {
            // 서버의 clientStateCheck 기능을 다시 동작시킨다.
            getConnector().ResumeClientStateCheck();
        }
    }

    private void OnDisable()
    {
        if (getConnector().IsConnected)
        {
            getConnector().Disconnect();
        }
    }

    public void Quit()
    {
#if UNITY_EDITOR
        UnityEditor.EditorApplication.isPlaying = false;
#else
        Application.Quit();
#endif
    }
}
```

### Connector와 User의 생성

커넥터에서 여러 기능을 이용하기 위해서 Connector와 User를 생성해야 합니다. Connector는 주로 서버 접속, 인증 등의 기능을 제공하며 User는 로그인, 방 생성 및 입장 등 유저와 관련된 기능을 제공합니다.

getConnector() 함수에서 새로운 GameAnvilConnector 객체를 생성하여 connector 필드에 저장합니다. 그리고 getUser() 함수에서 새로운 GameAnvilUser 객체를 생성하여 user 필드에 저장합니다. user 생성 시 사용하는 서비스 이름은 서버 프로젝트에서도 사용하므로 기억해 둡니다.

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();
    }

    return connector;
}

public GameAnvilUser getUser()
{
    if (user == null)
    {
        user = new GameAnvilUser(getConnector(), "BASIC_SERVICE", 1);
    }

    return user;
}
```

<br>

### 서버 연결

커넥터에서 제공하는 API를 사용하여 서버에 접속하는 Connect() 메서드는 다음과 같습니다.

```c#
// Client-side

public async void Connect()
{
    ip = serverInfoInput.text;

    getConnector().OnDisconnect += (resultCode, payload) =>
    {
        Debug.Log("Disconnected");
    };

    try
    {
        await getConnector().Connect(ip, port);
        Debug.Log("CONNECT_SUCCESS");
        logText.text = "CONNECT_SUCCESS";
    }
    catch (Exception e)
    {
        Debug.Log("CONNECT_FAIL");
        logText.text = "CONNECT_FAIL";
        throw;
    }
}
```

<br>

Connect() 함수에서는 GameAnvilConnector의 Connect 메서드를 호출하고 있습니다. 이를 통해 인자로 주어진 ip와 port 정보로 서버 접속을 시도합니다.

이것으로 서버가 접속을 받아들일 준비가 된 것처럼 클라이언트도 서버에 접속할 준비가 완료되었습니다.

### 서버 연결 확인

이제 Unity 클라이언트에서 플레이 모드로 진입하여 콘솔 상에 결과 코드가 제대로 출력되는지 확인합니다. 게임 화면의 텍스트 상에 IP, Port의 접속 정보와 더불어 연결 성공 메시지를 확인할 수 있습니다. 게임 서버에 접속 완료된 클라이언트는 이제 서버를 통해 메시지를 주고받을 수 있습니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/15_connect_success.png)

## Room 및 User 생성

서버에 접속한 클라이언트를 **게임 유저**라고 합니다. 서버에 접속한 클라이언트는 서버상에서 하나 이상의 게임 유저(User)로 로그인할 수 있습니다(이 예제에서는 하나의 유저로 로그인하는 경우를 다룹니다). 게임 유저는 하나의 **게임 룸**에 속함으로써 동일한 룸에 속한 다른 유저들과 통신할 수 있습니다. 즉, 유저들이 서로 다른 유저와 게임 관련 메지시를 교환하려면 해당 유저들은 같은 방(Room) 안에 속해 있어야 합니다.

GameAnvil에서는 게임 유저와 게임 룸의 기본 구현을 미리 준비해 두었으므로 엔진의 클래스를 확장하고 커넥터의 API를 이용해 쉽게 게임 유저와 룸의 구조를 완성할 수 있습니다. 엔진 측에서는 게임 유저와 룸을 정의하는 방법을 다루고, 커넥터 측에서는 방 생성이나 참여 등을 요청하는 API를 사용하는 예제를 다루고자 합니다.

### User

서버에서는 게임 유저와 게임 룸의 기능을 클래스로 정의합니다. 우선 게임 유저를 정의해 보겠습니다.

프로젝트 패널에서 Main 클래스가 위치한 경로를 마우스 오른쪽 버튼으로 클릭한 뒤 **New > Package**를 선택해 **game**이라는 이름의 새로운 패키지를 생성합니다. 그리고 **game** 패키지를 다시 마우스 오른쪽 버튼으로 클릭한 뒤 **New > GameAnvil User**를 선택합니다. 파일 생성 대화 상자가 열리면 **File name**에 **BasicUser**를, **Service name**에 **User type**에 **USER_TYPE_BASIC**을 입력한 뒤 **OK**를 클릭합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/16_create_user.png)

GameAnvil에서 제공하는 IUser 인터페이스를 구현하여 게임 유저를 구현하는 기본 코드가 작성된 파일이 생성됩니다. GameAnvil에서 원하는 기능의 게임 유저를 구현하려면, IUser 인터페이스를 구현한 후 상황에 맞게 호출되는 여러 콜백 함수들을 오버라이딩하여 원하는 코드를 실행하도록 설정하면 됩니다. 다음은 지원하는 콜백 함수 목록의 일부입니다.

- onLogin: 로그인 요청 시에 실행되는 콜백입니다. 반환값으로 로그인 요청 허용 여부를 전달해야 합니다. false를 반환하면 클라이언트는 로그인에 실패할 것입니다. 파라미터를 통해 클라이언트로부터 받은 페이로드를 참조할 수 있으며, 클라이언트로 되돌려줄 페이로드에 대한 참조를 통해 로그인 허용 정보 외에 추가 정보를 클라이언트에 전달할 수 있습니다.
- onPostLogin: 로그인 이후에 실행되는 콜백입니다.
- onReLogin: 이미 로그인 된 바 있는 유저에 대해서 또 다시 로그인 요청이 오는 경우는 이 콜백에서 따로 처리할 수 있습니다.
- onDisconnect: 클라이언트 요청에 따른 로그아웃 또는 서버에 의한 강제 로그아웃에 의해서 서버에 등록된 유저 정보와 연결이 끊어졌을 때 호출되는 콜백입니다.

이외의 콜백 함수들은 로그인 과정에 관여하지 않으므로 당장 모든 콜백이 무슨 의미인지 완벽하게 알아야 할 필요는 없습니다.

onLogin 콜백 메서드에서는 로그인 과정에 실행되어야 하는 동작을 구현합니다. 이 예제에서는 별다른 로그인 구현 없이, 무조건 로그인에 성공하도록 true를 반환하도록 합니다. 그리고 userContext의 getter를 생성합니다.

```java
@GameAnvilUser(
        gameServiceName = "BASIC_SERVICE",
        gameType = "USER_TYPE_BASIC",
        useChannelInfo = true
)
public class BasicUser implements IUser {

    private static final Logger logger = getLogger(BasicUser.class);

    private IUserContext userContext;

    public IUserContext getUserContext() {
        return userContext;
    }

    @Override
    public void onCreate(IUserContext userContext) {
        this.userContext = userContext;
    }

    @Override
    public boolean onLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onAfterLogin(boolean isRelogined) {

    }

    @Override
    public boolean onReLogin(IPayload payload, IPayload sessionPayload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onDisconnect() {

    }

    @Override
    public void onPause() {

    }

    @Override
    public void onResume() {

    }

    @Override
    public void onLogout(IPayload payload, IPayload outPayload) {

    }

    @Override
    public boolean canLogout() {
        return true;
    }

    @Override
    public void onAfterLeaveRoom() {

    }

    @Override
    public boolean canTransfer() {
        return true;
    }

    @Override
    public boolean onLoginByOtherDevice(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean onLoginByOtherUserType(String s, IPayload payload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onLoginByOtherConnection(IPayload payload) {

    }

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();
        try {
            return userContext.matchRoom(matchingGroup, roomType, gameRoomMatchForm);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicUser::onMatchRoom", e);
            return RoomMatchResult.FAILED;
        }
    }

    @Override
    public void onMatchRoomFail(MatchRoomFailCode matchRoomFailCode) {

    }

    @Override
    public void onMatchUserFail(MatchUserFailCode matchUserFailCode) {

    }

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;

        try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(userContext.getUserId());
            return userContext.matchUser(matchingGroup, roomType, term);
        } catch (TimeoutException | NodeNotFoundException | GameAnvilException e) {
            logger.error("BasicUser::onMatchUser", e);
        }

        return isSuccess;
    }

    @Override
    public void onMatchUserCancel(MatchCancelReason matchCancelReason) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    @Override
    public void onAfterTransferIn() {

    }

    @Override
    public void onSnapshot(IPayload payload, IPayload outPayload) {

    }

    @Override
    public boolean canMoveOutChannel(String channelId, IPayload payload, IPayload outPayload) {
        return false;
    }

    @Override
    public void onMoveOutChannel(String channelId, IPayload payload) {

    }

    @Override
    public void onAfterMoveOutChannel() {

    }

    @Override
    public void onMoveInChannel(String channelId, IPayload payload, IPayload outPayload) {

    }
}
```

<br>

### Room

로그인 가능한 유저 구현을 완료했습니다. 이제 게임 룸을 구현합니다. 유저 생성 방법과 마찬가지로 **GameAnvil Room** 파일 템플릿을 이용해 IRoom 인터페이스를 구현한 클래스를 생성합니다. **File name**에는 **BasicRoom**을, **Service name**에는 **BASIC_SERVICE**를, **Room type**에는 **ROOM_TYPE_BASIC**을, **User**에는 이전 단계에서 생성한 IUser 구현 클래스의 클래스명 **BasicUser**를 입력합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/17_create_room.png)

**OK**를 클릭하면 지원하는 콜백 메서드가 자동으로 작성됩니다. GameAnvil에서 지원하는 콜백을 설명하기 위해 잠시 클라이언트의 API를 간단하게 설명하겠습니다. 클라이언트에서는 커넥터로 로그인 후에 다른 유저들과 통신하기 위해 방 관련 API를 호출할 수 있습니다. 방을 만들거나, 다른 유저가 만든 방에 참여하거나, 방에서 나가는 등의 동작을 지원합니다.

- onCreateRoom: 유저가 방 생성을 요청했을 때 실행됩니다. 방 생성을 요청한 유저 정보와 함께 방 생성 시 클라이언트에서 전달한 추가 정보를 파라미터로 제공하므로 이 정보를 토대로 방 생성을 허용할 것인지 여부를 결정해 반환해서 구현합니다.
- onJoinRoom: 다른 유저가 생성한 방에 입장을 요청할 때 실행됩니다. 다른 콜백들과 마찬가지로 방 입장을 요청한 유저 정보와 함께 방 입장 시 클라이언트에서 전달한 추가 정보를 제공하므로 이 정보를 토대로 방 입장을 허용할 것인지 여부를 결정해 반환해서 구현합니다.
- onLeaveRoom: 유저가 방에서 퇴장을 요청할 때 실행됩니다. 다른 콜백들과 마찬가지로 방 퇴장으로 요청한 유저 정보와 함께 방 퇴장 요청 시에 같이 전달한 추가 정보를 제공하므로 이 정보를 토대로 방 퇴장을 허용할 것인지 여부를 결정해 반환해서 구현합니다.

방에 속해있는 유저에 대한 정보는 **roomContext.getAllUsers()** 함수를 통해 얻을 수 있습니다. 이를 활용하여 방에 속해있는 모든 유저에게 동일한 패킷을 전달하는 **broadcast()** 함수를 추가합니다. 또한 방 매칭 정보를 저장할 필드와, 퍼즐 위치 동기화를 위한 자료구조를 추가합니다.

그리고 **onInit** 콜백에서 방 매칭 정보 객체를 생성하고, **onCreateRoom/onJoinRoom** 콜백에서 각각 방 매칭 정보를 등록하고 업데이트하는 코드를 추가합니다. 이를 통해 해당 방이 매칭 목록에 추가되도록 하여, 다른 유저가 해당 방으로 매칭될 수 있도록 합니다.

```java
package org.example.game;

import com.nhn.gameanvil.exceptions.GameAnvilException;
import com.nhn.gameanvil.exceptions.NodeNotFoundException;
import com.nhn.gameanvil.game.GameAnvilRoom;
import com.nhn.gameanvil.node.game.IRoom;
import com.nhn.gameanvil.node.game.context.IRoomContext;
import com.nhn.gameanvil.node.game.data.MatchCancelReason;
import com.nhn.gameanvil.packet.IPayload;
import com.nhn.gameanvil.packet.Packet;
import com.nhn.gameanvil.serializer.ITimerHandlerTransferPack;
import com.nhn.gameanvil.serializer.ITransferPack;
import org.example.StringValues;
import org.example.match.BasicRoomMatchInfo;
import protocol.Puzzle;
import org.slf4j.Logger;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.TimeoutException;

import static org.slf4j.LoggerFactory.getLogger;

@GameAnvilRoom(
        gameServiceName = "BASIC_SERVICE",
        gameType = "ROOM_TYPE_BASIC",
        useChannelInfo = true
)
public class BasicRoom implements IRoom<BasicUser> {

    private static final Logger logger = getLogger(BasicRoom.class);

    private IRoomContext roomContext;

    // 코드 추가
    private BasicRoomMatchInfo gameRoomMatchInfo;
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();

    public void broadcast(Packet packet) {
        final var allUsers = roomContext.getAllUsers();

        for (final var user : allUsers) {
            final var userContext = ((BasicUser)user).getUserContext();
            userContext.send(packet);
        }
    }

    @Override
    public void onCreate(IRoomContext<BasicUser> roomContext) {
        this.roomContext = roomContext;
    }

    @Override
    public void onInit() {
        gameRoomMatchInfo = new BasicRoomMatchInfo(roomContext.getId());
    }

    @Override
    public void onDestroy() {

    }

    @Override
    public boolean onCreateRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public boolean canLeaveRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        return true;
    }

    @Override
    public void onLeaveRoom(BasicUser user) {

    }

    @Override
    public void onAfterLeaveRoom() {

    }

    @Override
    public void onRejoinRoom(BasicUser user, IPayload payload) {

    }

    @Override
    public void onTransferOut(ITransferPack transferPack) {

    }

    @Override
    public void onTransferIn(List<BasicUser> list, ITransferPack transferPack, ITimerHandlerTransferPack timerHandlerTransferPack) {

    }

    @Override
    public void onAfterTransferIn() {

    }

    @Override
    public void onPause() {

    }

    @Override
    public void onResume() {

    }

    @Override
    public boolean onMatchParty(String roomType, String matchingGroup, BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        return isSuccess;
    }

    @Override
    public void onMatchPartyCancel(MatchCancelReason matchCancelReason) {

    }

    @Override
    public void onForceMatchRoomUnregistered(MatchCancelReason matchCancelReason) {

    }

    @Override
    public boolean canTransfer() {
        return true;
    }
}
```

<br>

### GameNode

이제 게임 유저와 게임 방이 준비되었습니다. 하지만 아직 게임 유저/게임 룸의 생성과 삭제 요청을 처리하는 노드가 없습니다. 게임 유저와 게임 룸을 관리하는 역할을 하는 노드는 GameNode입니다. 이 노드는 일반적으로 게임 서버가 하기를 기대하는 대부분의 게임 로직 처리 역할을 수행하는 노드입니다. GameAnvil에 노드를 추가하는 방법은 자연스럽고 간단합니다. 게임 유저와 게임 룸을 정의했던 것과 마찬가지로, 미리 정의된 인터페이스를 구현하여 클래스를 만든 뒤 원하는 기능을 추가 구현하면 됩니다.
**GameAnvil GameNode** 템플릿 선택 후 파일명을 **BasicGameNode**로, 서비스 이름을 **BASIC_SERVICE**로 설정하고 **OK** 버튼을 눌러 게임 노드 클래스를 생성합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/18_create_game_node.png)

노드가 역할을 수행하기 위해서는 우선 노드가 루프를 실행해야 합니다. 노드가 실행될 때는 일련의 과정을 거치게 되므로 약간의 시간이 필요합니다. 노드의 실행 여부나 실행 과정 중 어느 단계에 있느냐를 나타내는 지표를 노드의 상태라고 부릅니다. 노드의 상태는 보통 아래 순서에 따라서 순차적으로 변경되면서 READY 상태에 도달합니다.

- INIT
- PREPARE
- READY

READY 상태에 도달한 노드는 이제 미리 사용자가 작성한 로직들을 실행할 준비가 된 상태입니다. 각 준비 단계에 도달했을 때 특정한 코드를 실행하고 싶다면, 콜백 메서드를 구현하여 엔진에서 콜백 메서드를 호출했을 때 해당 코드가 실행되도록 설정할 수 있습니다. 지금은 특별하게 실행해야 할 코드가 없으므로 생성된 코드를 그대로 사용합니다.

```java
import com.nhn.gameanvil.game.GameAnvilGameNode;
import com.nhn.gameanvil.node.game.ChannelUpdateType;
import com.nhn.gameanvil.node.game.IGameNode;
import com.nhn.gameanvil.node.game.context.IGameNodeContext;
import com.nhn.gameanvil.node.game.data.IChannelRoomInfo;
import com.nhn.gameanvil.node.game.data.IChannelUserInfo;
import com.nhn.gameanvil.packet.IPayload;

@GameAnvilGameNode(gameServiceName = "BASIC_SERVICE")
public class BasicGameNode implements IGameNode {
    private IGameNodeContext gameNodeContext;

    public IGameNodeContext getContext() {
        return gameNodeContext;
    }

    @Override
    public void onCreate(IGameNodeContext gameNodeContext) {
        this.gameNodeContext = gameNodeContext;
    }

    @Override
    public void onChannelUserInfoUpdate(ChannelUpdateType channelUpdateType, IChannelUserInfo channelUserInfo, int userId, String accountId) {

    }

    @Override
    public void onChannelRoomInfoUpdate(ChannelUpdateType channelUpdateType, IChannelRoomInfo channelRoomInfo, int userId) {

    }

    @Override
    public void onChannelInfo(IPayload payload) {

    }

    @Override
    public void onInit() {

    }

    @Override
    public void onPrepare() {

    }

    @Override
    public void onReady() {

    }

    @Override
    public void onPause(IPayload payload) {

    }

    @Override
    public void onShuttingdown() {

    }

    @Override
    public void onResume(IPayload payload) {

    }
}
```

작성한 게임 유저와 게임 룸을 GameNode에 연동하기 위해 설정 파일에도 이를 등록합니다. GameAnvilConfig.json 파일의 'game' 항목에 아래의 내용을 추가합니다.

```json
// 게임 로비 역할을 하는 노드. (게임 룸, 유저를 포함 하고 있음)
  "game": [
    {
      "nodeCnt": 1,
      "serviceId": 1,
      "serviceName": "BASIC_SERVICE",
      "channelIDs": [["ch1"]], // 노드마다 부여할 채널 ID. (유니크하지 않아도 됨. ""는 채널 사용하지 않음을 의미)
      "userTimeout": 5000 // 클라이언트의 접속 끊김 이후 유저 객체를 서버에서 제거하지 않고 얼마동안 관리할지 설정
    }
  ],
```

<br>

### 게임 노드, 유저, 룸 설정

 마우스 오른쪽 버튼으로 클릭한 뒤 **New > Java Class**를 선택해 직접 클래스를 생성할수도 있습니다. 

```java
@GameAnvilGameNode(gameServiceName = StringValues.serviceName)
public class BasicGameNode implements IGameNode {
    // ...
}

@GameAnvilRoom(
    gameServiceName = StringValues.serviceName,
    gameType = StringValues.roomType,
    useChannelInfo = false
)
public class BasicRoom implements IRoom<BasicUser> {
    // ...
}

@GameAnvilUser(
    gameServiceName = StringValues.serviceName,
    gameType = StringValues.userType,
    useChannelInfo = false)
public class BasicUser implements IUser {
    // ...
}
```

그리고 엔진에서 제공하는 @GameAnvilGameNode 어노테이션을 통해 게임 노드, @GameAnvilUser 어노테이션을 통해 유저, @GameAnvilRoom 어노테이션을 통해 룸 관련 설정이 자동으로 등록됩니다. 

유저 타입은 각 유저 구현을 구분하는 서버와 클라이언트 간 약속된 문자열이고, 룸 타입은 각 룸 구현을 구분하는 서버와 클라이언트 간 약속된 문자열입니다.

이후 클라이언트 프로젝트 구현 시 해당 타입을 사용해야 하므로 기억해 둡니다. 예제에서 사용한 유저와 룸 타입은 다음과 같습니다.

```java
public class StringValues {
    public static final String serviceName = "BASIC_SERVICE";
    public static final String userType = "USER_TYPE_BASIC";
    public static final String roomType = "ROOM_TYPE_BASIC";
}
```

예제에서 사용할 BasicGameNode, BasicUser, BasicRoom 생성자를 각각 파라미터로 입력합니다.

마지막으로 config를 등록하는 부분에서는 채널 정보 사용 여부 설정, 프로토콜 등록 등의 작업 등을 진행합니다. 프로토콜을 등록하는 부분은 이후 인게임 채팅과 직소 퍼즐 로직을 구현하는 부분에서 다루게 됩니다.

이제 클라이언트가 서버에 접속해서 게임 유저로서 로그인하고, 게임 룸을 생성할 수 있는 기능이 구현 완료되었습니다. 하지만 서버에 접속한다고 해서 바로 게임 관련 기능(게임 유저 생성, 게임 룸 생성 등)을 요청할 수 있는 것은 아닙니다. 지금 상태에서 서버와 클라이언트를 실행한다고 해도 클라이언트는 게임 서버의 기능을 사용할 수 없을 것입니다. 서버에 이러한 것들을 요청하려면 서버 접속 이후에 클라이언트 인증 과정이 필요합니다. 다음 챕터에서는 서버와 클라이언트에서 인증을 어떻게 처리하는지 다룹니다.

## 서버 접속

클라이언트가 서버에 접속하고 나서 게임에 로그인하기 전에 유저의 신원을 확인하고 인증 과정을 거칠 필요가 있습니다.
게임 노드가 게임 유저와 게임 룸 생성 역할을 담당한다면, 게이트웨이 노드는 유저의 접속과 인증 기능을 담당합니다. 게임 노드를 클래스 작성을 통해 구현한 것처럼 게이트웨이 노드도 일관성 있는 방식으로 구현할 수 있습니다.
**GameAnvil GatewayNode** 템플릿 선택 후 파일명을 **BasicGatewayNode**로 설정하고 **OK** 버튼을 눌러 게이트웨이 노드 클래스를 생성합니다. 게이트웨이 노드 클래스는 기본 생성된 코드 외에 별도로 추가 코드는 필요하지 않습니다.

### 프로토콜 등록

인증 과정을 거치면서 서버와 클라이언트는 서로 사용할 프로토콜을 확인하는 과정을 거칩니다. 따라서 인증 과정을 거치기 전에 프로토콜을 등록해야 합니다. 프로토콜 등록은 한 번만 하면 되기 때문에, getConnector() 코드 내부에 프로토콜 등록 코드를 추가해보겠습니다. 튜토리얼을 따라 진행하다보면 구현하게 되는 인게임 채팅에서 사용할 프로토콜을 미리 등록해봅니다.

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();

        // RegisterProtocol은 Auth 전에 해야 한다.
        GameAnvilProtocolManager.RegisterProtocol(BasicProtocolReflection.Descriptor);
    }

    return connector;
}
```

<br>

### 인증 코드 추가

유니티 프로젝트로 이동해서 클라이언트 측 구현을 해보겠습니다. 클라이언트에서는 연결 요청과 마찬가지로 인증 요청을 커넥터 GameAnvilConnector API를 통해 요청할 수 있습니다. 인증 요청은 연결 요청 이후에만 성립할 수 있습니다. 인증 요청 결과를 화면 상의 텍스트와 콘솔을 통해 출력하여 확인할 수 있도록 하겠습니다.

```c#
// Client-side

public async void Auth()
{
    accountId = Random.Range(1000, 9999) + "";
    clientInfoText.text = "클라이언트정보 : " + accountId;

    try
    {
        var result = await connector.Authentication(deviceId, accountId, password);
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
    }
    catch (Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
        throw;
    }
}
```

이제 클라이언트가 서버에 접속할 뿐만 아니라 인증 과정까지 요청할 수 있도록 설정되었습니다.

### 인증 확인

Unity 클라이언트에서 플레이 모드로 진입합니다. 콘솔 상에 로그가 접속, 인증 순으로 순차 출력됨을 확인합니다. 다시 정리하면, 클라이언트의 Auth 요청에 따라 서버의 게이트웨이 노드에서 사용자 인증이 완료되어 서버에 로그인할 수 있는 상태가 된 것입니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/19_auth_success.png)

## 로그인

이제 마지막으로 수행할 과정은 게임 노드에 로그인하여 게임 유저를 생성하는 것입니다. 게임 유저는 서버에 접속한 다른 클라이언트와 통신하기 위해 필요한 개념으로, 클라이언트 간에 통신을 하려면 각 클라이언트는 게임 유저를 생성하여 게임 룸을 통해서 메시지를 주고받도록 하고 있습니다.

### 로그인 코드 추가

접속 및 인증이 완료된 클라이언트는 게임 노드로 로그인을 할 수 있습니다. 서버에 로그인하면 게임 노드에 해당 클라이언트를 위한 게임 유저 객체가 생성됩니다. 클라이언트는 자신의 서버측 게임 유저 객체를 통해 서버 혹은 다른 유저들과 메시지를 주고 받을 수 있게 됩니다. 앞서 작성한 인증 코드를 아래와 같이 수정하여 인증이 성공한 경우에 바로 로그인을 진행하도록 합니다.

```c#
// Client-side

public async void Login()
{
    try
    {
        var result = await getUser().Login("USER_TYPE_BASIC", "ch1");
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
    }
    catch (Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
        throw;
    }
}
```

로그인을 요청할 때 같이 전달한 콜백 메서드가 로그인 요청 결과를 로그로 출력하도록 합니다.

### 로그인 확인

Unity 테스트 모드를 통해 성공적으로 로그인되는 것을 확인합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/20_login_success.png)

## 방 생성 및 참가

### 클라이언트 작업

Unity 프로젝트에서 ConnectHandler 코드에 방 생성을 요청하는 메서드를 작성합니다. 이때, CreateRoom 메서드의 두 번째 인자인 RoomType은 반드시 서버에서 지정한 값과 같아야 함에 유의합니다. 일반적으로 이러한 RoomType 등의 프로토콜은 서버와 클라이언트 개발자가 사전에 값을 미리 정의해두고 사용합니다. 방 생성 결과 코드를 콘솔에 출력합니다. 또, 방 생성에 성공하면 roomId를 클라이언트 측에 저장해두고, 게임 씬으로 이동하도록 코드를 작성합니다.

```c#
public async void CreateRoom()
{
    try
    {
        var result = await getUser().CreateRoom(string.Empty, "ROOM_TYPE_BASIC", string.Empty);
        Debug.Log(result.ErrorCode.ToString());
        logText.text = result.ErrorCode.ToString();
        if (result.ErrorCode == ResultCodeCreateRoom.CREATE_ROOM_SUCCESS)
        {
            Debug.Log("Room Id : " + result.Data.RoomId);
            this.roomId = result.Data.RoomId;
            SceneManager.LoadScene("GameScene");
        }
    }
    catch(Exception e)
    {
        Debug.Log(e.ToString());
        logText.text = e.ToString();
    }
}
```

여기까지 완료했다면 클라이언트를 통해 방을 생성하는 기능을 구현한 것입니다. Create Room 버튼의 OnClick 이벤트와 CreateRoom 메서드를 연결합니다. 실제로 실행해 보기 전에 방 참가 기능까지 구현한 뒤 테스트해 보겠습니다. 이번에는 ConnectHandler에 방 참가 코드를 추가합니다.

JoinRoom 메서드의 첫 번째 인자가 서버 개발에서 사용한 사전 정의된 RoomType 문자열과 일치하는지 확인합니다. 결과 코드를 통해 방 참가 요청에 대한 서버 측의 응답을 확인하고 방 참가에 성공했다면 방 생성과 동일한 동작을 하도록 작성합니다. 메서드 작성이 완료되었다면, Join Room 버튼의 OnClick 이벤트와 JoinRoom 메서드를 연결합니다. 유니티의 Update() 함수에서 엔터 버튼이 입력되면 JoinRoom()을 호출하도록 코드를 작성합니다.

```c#
private void Update()
{
    if (popupCanvas && popupCanvas.activeInHierarchy && Input.GetKeyDown(KeyCode.Return))
    {
        JoinRoom();
    }
}

public async void JoinRoom()
{
    if (!string.IsNullOrEmpty(roomIdInput.text))
    {
        try
        {
            var result = await getUser().JoinRoom("ROOM_TYPE_BASIC", int.Parse(roomIdInput.text), string.Empty);
            Debug.Log(result.ErrorCode.ToString());
            logText.text = result.ErrorCode.ToString();
            if (result.ErrorCode == ResultCodeJoinRoom.JOIN_ROOM_SUCCESS)
            {
                Debug.Log("Room Id : " + result.Data.RoomId);
                this.roomId = result.Data.RoomId;
                SceneManager.LoadScene("GameScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
        }
    }
}
```

이제 GameScene으로 이동해서 방 참가 후에 어떤 동작을 할 것인지 구현해 보겠습니다. 우선 초기 GameManager 코드는 아래와 같습니다. 룸 아이디를 게임 상에서 표시할 수 있도록 만듭니다.

```c#
public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public InputField ChatInputText;

    private ConnectHandler connectHandler;


    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }
}
```

이제 클라이언트에 서버 접속, 인증, 로그인 기능뿐만 아니라 방 참가와 생성 기능까지 모두 구현되었습니다.

### Room 생성 및 참가 테스트

Unity 프로젝트의 상단 도구 모음 바에서 **File > Build Setting**을 선택합니다. 아래와 같이 필요한 씬들을 빌드할 씬 목록에 차례대로 추가합니다. 만약 씬 순서가 잘못되었다면 리스트 상의 항목을 드래그하여 ConnectScene이 맨 위로 오도록 순서를 알맞게 조정합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/21_add_scene.png)

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 새로운 윈도우가 열리면서 ConnectScene이 로드됩니다. 서버의 IP주소를 입력하고 Connect 버튼을 눌러 서버 접속을 시도합니다. 예제에서는 로컬 서버를 띄워 사용하므로 `127.0.0.1`을 입력합니다. 서버 접속에 성공했다면 콘솔 로그와 함께 게임 화면 상에서 연결 상태가 `CONNECT_SUCCESS`로 보이는 것을 확인할 수 있습니다. 만약 중간 과정에서 실패 로그가 남는다면, 실패 코드를 통해서 원인을 알 수 있습니다. 만약 그것으로 부족하다면 서버의 로그를 확인해서 원인을 분석해 보아야 합니다. 로그인까지 성공했다면 Create Room 버튼을 눌러서 방 생성을 요청합니다. GameScene으로 씬이 이동되고 생성된 방의 아이디가 함께 출력됩니다. 방 아이디는 다음의 예제 이미지와 다를 수 있습니다.

그 상태로 Unity 플레이모드를 실행한 후 로그인까지 완료되는 것을 확인합니다. 그리고 Join Room 버튼을 클릭한 후 앞서 생성한 방의 RoomId를 직접 입력해서 방에 참가하도록 합니다. 방 참가에 성공한다면 게임 씬으로 이동되고, 두 게임 화면에서 동일한 방 아이디를 가지고 있는 것을 확인할 수 있을 것입니다. 테스트 화면은 아래와 비슷하게 나타나면 성공입니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/22_join_room.png)

먄약 제대로 진행되지 않는다면 아래 사항을 다시 확인하십시오.

- 서버 구현 수정 후 서버 프로세스를 재시작하였는가?
- 서비스명은 GameAnvilConfig.json에 설정한 것과 동일하게 서버/클라이언트에 구현되었는가?

## 인게임 채팅 구현

이제 클라이언트 간에 서버를 통해 통신할 수 있는 환경이 구성되었습니다. 여기에서는 클라이언트에서 생성한 데이터를 원격의 클라이언트가 받아볼 수 있는 간단한 예제를 구현해 보겠습니다. 예제 프로젝트 내부에 미리 구현된 프로토콜을 이용해 채팅 기록을 주고받는 방법을 알아봅니다. 이 예제 한정으로 통신 데이터를 클라이언트로부터 서버로 전송할 때는 MessageRequest 클래스를 사용합니다. 서버로부터 클라이언트로 통신 데이터를 전송할 때는 MessageResponse또는 MessageBroadcast 클래스를 사용합니다.

(프로젝트 템플릿을 사용해서 직접 프로젝트를 생성한 경우 Message 클래스들을 이용할 수 없습니다. 튜토리얼용으로 만들어진 프로젝트를 다운로드해 내부의 BasicProtocol.java가 프로젝트에 포함될 수 있도록 설정하십시오. 또한 서버 실행 전에 프로토콜을 등록하는 과정도 필요합니다. 서버 측의 프로토콜 관련 설정은 미리 준비된 튜토리얼용 프로젝트에 미리 완료되어 있으므로 튜토리얼 프로젝트를 그대로 사용했다면 진행하지 않아도 됩니다.)

### 클라이언트 측 전송 구현

먼저 클라이언트에서 서버 방향으로 메시지를 보내는 코드를 작성해 보겠습니다. 일단 로그인한 유저로 서버에 연결되면 유저 에이전트를 통해 서버의 기능을 활용할 수 있습니다. 여기에서는 유저 에이전트의 Send 기능을 활용해 방에 패킷을 전송합니다. 메시지 전송을 위해서는 전송할 내용을 담은 패킷을 인자로 넘겨 주어야 합니다.

Unity 프로젝트의 GameManager 스크립트에 메시지 전송 스크립트를 추가합니다. SendMessage 메서드가 호출되면 화면상의 입력 필드에 사용자가 입력한 메시지와 자신의 유저 아이디를 담은 패킷을 userAgent를 통해 서버로 전송합니다. 그리고 Enter 키를 누를 때마다 SendMessage 메서드가 실행되도록 Update 함수에 구현을 추가합니다. 아래의 예제 코드는 SendMessage 메서드의 구현을 보여줍니다.

```c#
using System;
using UnityEngine;
using UnityEngine.UI;
using GameAnvil.Defines;
using Protocol;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    [Header("Object References")]
    public Text roomIdText;

    public Text ChatLogText;
    public InputField ChatInputText;

    private ConnectHandler connectHandler;

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Return) && !string.IsNullOrEmpty(ChatInputText.text))
        {
            SendMessage();
        }
    }

    public void SendMessage()
    {
        MessageRequest messageRequest = new MessageRequest();
        messageRequest.Message = "\n[" + connectHandler.accountId + "]:" + ChatInputText.text;
        connectHandler.getUser().SendUser(messageRequest);
        ChatInputText.text = string.Empty;
    }
}
```

이렇게 하면 클라이언트에서 서버 방향으로 패킷을 전송하는 기능이 구현되었습니다. 하지만 지금은 패킷을 서버로 보낸다고 해도 서버에서는 아무런 응답이 없을 것입니다. 그 이유는 서버 측에서 해당 패킷을 받았을 때 내용을 어떻게 분석하고 어떤 동작을 할지 정의해 주지 않았기 때문입니다. 다음 챕터에서는 서버 측 구현을 다루겠습니다.

### 서버 측 응답 구현

우선 서버에서 채팅 프로토콜을 사용할 수 있도록 서버 구동 전 프로토콜 등록을 합니다.

```java
@GameAnvilApplication
public class Main {
    public static void main(String[] args) {
        final var server = GameAnvilServer.getInstance();

        server.addProtocol(BasicProtocol.class);

        server.run(Main.class, args);
    }
}
```

여기에서는 게임 유저가 전송한 메시지를 서버가 받아 방 안의 유저들에게 전송해 주는 기능을 작성합니다. 클라이언트가 전송한 메시지를 서버의 게임 룸에서 처리하기 위해서는 핸들러를 사용합니다. 핸들러란, 특정 프로토콜을 처리하기 위한 코드 묶음을 의미합니다. 핸들러는 프로토콜 종류에 따라서 여러 개가 될 수 있고, 방에 핸들러를 여러 개 등록할 수 있습니다. 따라서 방은 복수의 프로토콜을 처리 가능합니다.

핸들러도 인터페이스 구현을 통해 생성됩니다. 프로젝트 패널에서 Main 클래스가 위치한 경로를 마우스 오른쪽 버튼으로 클릭한 뒤 **New > Package**를 선택해 **handler**이라는 이름의 새로운 패키지를 생성합니다. 그리고 **handler** 패키지를 다시 마우스 오른쪽 버튼으로 클릭한 뒤 **New > GameAnvil RoomMessageHandler**을 선택합니다. 파일 생성 대화 상자가 열리면 **File name**에 **BasicHandler**, **Message**에 **BasicProtocol.MessageRequest**를, **Room**에 앞서 생성한 IRoom 구현 클래스의 클래스명 **BasicRoom**을 입력한 뒤 **OK**를 클릭합니다.

이렇게 하면 BasicHandler 클래스가 생성되고 @GameAnvilController 어노테이션과 @GameRoomMapping 어노테이션으로 BasicRoom 에서 사용하는 핸들러로 등록되어 별도의 등록 절차 없이 사용할 수 있습니다. 이제 BasicRoom은 BasicProtocol의 MessageRequest 메시지를 BasicHandler를 통해서 처리할 수 있게 되었습니다.

실행될 내용, 즉, execute 메서드 내부 구현은 아래와 같이 작성합니다. 아래의 핸들러 예제 구현에서는 수신한 메시지에 대해 송신자에게 응답 메시지를 전송함과 더불어 방 전체 유저에게 방 단위 브로드캐스트용 메시지를 추가로 송신합니다. 이때, 클라이언트는 방 단위 브로드캐스트 메시지를 기준으로 게임을 동기화하도록 구현되어 있습니다.

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.BasicProtocol;

@GameAnvilController
public class BasicHandler {

    @GameRoomMapping(value = BasicProtocol.MessageRequest.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, BasicProtocol.MessageRequest request) {
        BasicProtocol.MessageResponse response = BasicProtocol.MessageResponse.newBuilder().setMessage(request.getMessage()).build();
        BasicProtocol.MessageBroadcast broadcast = BasicProtocol.MessageBroadcast.newBuilder().setMessage(request.getMessage()).build();

        BasicRoom room = ctx.getRoom();
        room.broadcast(Packet.makePacket(broadcast));
        ctx.reply(response);
    }
}
```

위 코드에서는 먼저 MessageRequest 객체의 Message 값을 이용하여 MessageResponse, MessageBroadcast 객체를 각각 새로 생성합니다. MessageResponse 타입의 객체는 패킷을 룸으로 전송한 유저 객체에게 전송합니다. MesageBroadcast 객체는 room을 통해 방 내부의 모든 유저에게 전송합니다.

이렇게 해서 클라이언트가 송신한 패킷을 서버가 수신하고, 약간의 처리를 한 뒤 다시 되돌려주는 기능이 서버에 추가되었습니다. 이때 클라이언트 또한 서버에서 송신한 패킷을 어떻게 처리할지 지정해야 합니다.

### 클라이언트 측 수신 구현

클라이언트에서도 서버 측의 패킷을 처리하기 위해 미리 핸들러를 등록해야 합니다. 그렇지 않으면 패킷을 받았을 때 처리 방법을 알 수 없는 프로토콜이라고 판단해서 내용을 버리게 됩니다. 서버에서 보내는 내용을 수신했을 때 이것을 감지하고 내용을 처리하려면 서버에서 보내는 패킷의 프로토콜 타입의 핸들러를 등록하면 됩니다. 즉, MessageBroadcast 타입의 메시지를 처리하는 핸들러 등록 코드를 작성해서 등록합니다.

다시 Unity 프로젝트로 이동해 GameManager의 Start 메서드에 구현을 추가합니다. 우선 프로토콜 매니저를 이용해 서버와 클라이언트 간에 합의된 프로토콜을 등록합니다. 여기에서는 0번으로 BasicProtocol을 등록하였습니다. 번호에 대해서는 이후 이어지는 내용에서 설명합니다.

이후 유저 에이전트를 통해 리스너를 등록합니다. 메서드 타입으로는 프로토콜의 타입을 사용합니다. 서버에서 보내주기로 설정한 프로토콜은 MessageBroadcast이므로 이것을 사용하면 됩니다. 핸들러 구현은 수신한 메시지를 화면에 준비된 텍스트에 그대로 출력하는 간단한 로직으로 작성합니다. 바로 채팅의 가장 기본적인 기능입니다.

```c#
void Start()
{
    connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

    roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

    // 코드 추가
    connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
    {
        ChatLogText.text += messageBroadcast.Message;
    });
}
```

<br>

### 메시지 전달 확인

서버 수정 후 새로 실행했는지 확인하고, Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 빌드된 게임에서 방을 생성하고 서버측 로그를 확인합니다. 그 상태로 Unity 에디터에서 플레이모드로 진입한 후 앞서 생성한 방의 RoomId를 입력하여 해당 방에 참가합니다.

임의의 게임 프로세스에서 입력란에 텍스트를 입력한 뒤 엔터를 누릅니다. 그러면 다른 게임 프로세스에서도 동일한 텍스트가 나타날 것입니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/23_unity_chat_test.png)

간단한 채팅 서버 구현을 통해서 메시지 처리 과정을 학습했습니다. 다음에는 좀 더 실용적인 예제의 구현 과정을 살펴보겠습니다.

## 퍼즐 게임 구현

게임 씬에는 싱글 플레이가 가능한 퍼즐 게임이 미리 구현되어 있습니다. 플레이 모드로 진입하여 퍼즐 조각을 드래그해 알맞은 위치 근처에 놓으면 격자상의 정확한 위치로 보정됩니다. 이번 챕터에서는 이 게임을 멀티플레이어 게임으로 만들어 보겠습니다.

같은 방 안의 유저들 간 주고받는 메시지는 MessageRequest, MessageResponse, MessageBroadcast 외에 그 어떤 타입의 값도 모두 표현할 수 있습니다. 이번 챕터에서는 직접 프로토콜을 정의하고 서버와 클라이언트에 등록하는 방법을 살펴보겠습니다.

메시지는 미리 정의한 프로토콜에 기반하여 정의되기만 하면 서버와 클라이언트 간에 송수신이 가능합니다. XML, json 등 여러 가지 표현 수단이 있겠지만 GameAnvil은 Google Protocol Buffers를 사용합니다. 이는 속도와 안정성 측면에서 가장 좋은 솔루션 중 하나입니다.

### Google Protocol Buffers를 이용한 메시지 직렬화/역직렬화

프로토콜 버퍼를 사용하려면 먼저 메시지를 어떻게 정의할 것인지 명세를 작성해야 합니다. 예를 들면 MessageRequet의 명세에는 단일 문자열을 포함시킵니다. 그 후 프로토콜 명세를 컴파일해 원하는 언어의 파일로 변환합니다. 이후에는 MessageRequest를 사용했던 방식과 비슷하게 패킷의 메시지 프로토콜로 사용할 수 있습니다.

튜토리얼 샘플 프로젝트를 그대로 사용한다면 아래의 Puzzle 프로토콜 생성 과정은 이미 완료되어 있으므로 개념 이해에 참고합니다.
퍼즐 위치 동기화를 위한 메시지 프로토콜을 작성을 시작하겠습니다. 서버 프로젝트의 src/main/proto 경로에 Puzzle.proto 파일을 추가한 후 아래와 같이 프로토콜 명세를 작성합니다.

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}
```

PuzzlePosition 프로토콜은 퍼즐 조각의 최신 위치를 나타내기 위해서 각 퍼즐 조각의 고유 번호 정보(Index)와 퍼즐의 위치 정보(PositionX와 PositionY)로 구성합니다. 이제 프로토콜 정의는 끝났습니다. 다음은 이러한 프로토콜 정의를 각 개발 언어에 맞는 클래스로 컴파일하겠습니다.

protoc 실행 파일은 프로젝트 최상위 경로에 있습니다. 해당 프로젝트로 이동하고 다음 명령줄을 이용하여 Java와 C#을 위한 컴파일을 수행합니다. 명령줄을 실행했을 때, 아무런 결과 메시지가 뜨지 않는다면 성공한 것입니다.

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

이제 src/main/java 경로에 새롭게 Puzzle.java 클래스가 생성된 것을 확인할 수 있습니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/24_puzzle_proto_java.png)

C# 클래스 파일은 파인더와 파일 탐색기 등의 프로그램을 이용해서 Unity 프로젝트의 Asset/Protocol 경로로 이동합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/25_puzzle_proto_csharp.png)

퍼즐 위치 동기화를 위한 메시지 프로토콜 작성이 끝났습니다.

### 프로토콜 등록

프로토콜을 정의하고 컴파일까지 무사히 마쳤다면 서버와 클라이언트 양쪽에 해당 프로토콜 클래스를 등록해야 합니다. GameAnvil 서버의 Main 메서드에서 아래와 같이 프로토콜을 등록합니다.

```java
@GameAnvilApplication
public class Main {
    public static void main(String[] args) {
        final var server = GameAnvilServer.getInstance();

        server.addProtocol(BasicProtocol.class);
        // 코드 추가
        server.addProtocol(Puzzle.class);

        server.run(Main.class, args);
    }
}
```

Unity 프로젝트는 ConnectHandler 스크립트의 getConnector() 메서드에 아래와 같이 프로토콜을 추가합니다.

```c#
public GameAnvilConnector getConnector()
{
    if (connector == null)
    {
        connector = new GameAnvilConnector();

        GameAnvilProtocolManager.RegisterProtocol(BasicProtocolReflection.Descriptor);
        // 코드 추가
        GameAnvilProtocolManager.RegisterProtocol(PuzzleReflection.Descriptor);
    }

    return connector;
}
```

<br>

### 클라이언트 측 전송 구현

이제 게임을 위한 프로토콜 정의 및 등록까지 모두 마쳤습니다. 지금부터는 이러한 프로토콜에 기반한 메시지를 실제로 전송하는 기능을 구현합니다. 우선 클라이언트 측에서 데이터를 전송하는 부분을 먼저 구현합니다. 퍼즐 조각을 드래그하는 동안 그 위치를 서버로 전송해 보겠습니다.

Unity 프로젝트로 이동해 Puzzle.cs 파일을 열고 아래와 같이 코드를 추가합니다. 드래그되는 동안 퍼즐의 고유한 위치 정보를 담은 메시지 객체를 생성합니다. 그리고 GameAnvilUser를 통해 서버로 전송합니다. 메시지의 프로토콜만 다를 뿐 앞서 실습한 MessageRequest와 동일한 방식임을 알 수 있습니다.

```c#
using UnityEngine;
using UnityEngine.EventSystems;
using Protocol;

public class Puzzle : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    public int index;

    public Transform puzzleHolder;
    public float tolerance;


    private ConnectHandler connectHandler;

    void Start()
    {
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

    public void FixPosition()
    {
        if (Vector2.Distance(transform.position, puzzleHolder.position) < tolerance)
        {
            transform.position = puzzleHolder.position;
        }
    }

    public void SendPuzzlePosition(Vector2 puzzlePosition, bool onEndDrag)
    {
        PuzzlePosition position = new PuzzlePosition();
        position.Index = index;
        position.PositionX = (int)puzzlePosition.x;
        position.PositionY = (int)puzzlePosition.y;
        position.OnEndDrag = onEndDrag;

        connectHandler.getUser().SendUser(position);
    }
}
```

<br>

### 서버 측 응답 구현

클라이언트 측에서는 지속적으로 퍼즐의 위치를 서버에 보내게 되었습니다. 이제 퍼즐의 위치를 서버에서 어떻게 처리할지 작성해야 합니다. MessageRequest를 가공하여 MessageResponse, MessageBroadcast로 유저에게 되돌려주었던 것과 같이 퍼즐 위치를 다시 게임 룸의 모든 유저에게 되돌려주도록 구현해 보겠습니다.

서버 프로젝트로 돌아와서 **GameAnvil RoomMessageHandler** 파일 템플릿을 이용하여 PuzzlePositionHandler 클래스 파일을 생성합니다. 그리고 받은 패킷을 그대로 방 안의 모든 유저들에게 전달하도록 broadcast 메서드를 사용합니다.

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionHandler {

    @GameRoomMapping(value = Puzzle.PuzzlePosition.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePosition request) {
        BasicRoom room = ctx.getRoom();
        room.broadcast(Packet.makePacket(request)); // 방 전체 유저에게 메시지를 전송
    }
}

```

PuzzlePositionHandler도 역시 @GameAnvilController 어노테이션과 @GameRoomMapping 어노테이션으로 BasicRoom 에서 사용하는 핸들러로 등록됩니다. 이제 BasicRoom은 Puzzle 프로토콜의 PuzzlePosition 메시지를 PuzzlePositionHandler를 통해서 처리할 수 있게 되었습니다.


<br>

### 클라이언트 측 수신 구현

앞서 서버에서 송신한 MessageBroadcast를 처리하기 위해 미리 핸들러를 등록해야 했습니다. 이번에도 마찬가지로 퍼즐 위치를 처리하는 핸들러를 만들기 위해 GameManager의 Start 메서드에 PuzzlePosition 메시지를 처리하는 핸들러 등록 코드를 작성합니다. 이 핸들러는 메시지를 수신하면 퍼즐 객체를 찾아서 서버에서 받은 위치로 이동시킵니다.

```c#
public class GameManager : MonoBehaviour{

    ...(생략)...

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
        {
            ChatLogText.text += messageBroadcast.Message;
        });

        // 추가
        connectHandler.getUser().SetMessageCallback<PuzzlePosition>((_, _, puzzlePosition) =>
        {
            Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);

            if (puzzlePosition.OnEndDrag)
            {
                puzzle.FixPosition();
            }
        });
    }

    ...(생략)...
}
```

<br>

### 퍼즐 위치 동기화 확인

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 빌드된 게임에서 방을 생성한 뒤 Unity 플레이 모드를 실행하고 생성된 방의 RoomId를 입력해서 방에 참가합니다. 이제 퍼즐 조각을 드래그해 위치를 이동하면 해당 퍼즐 조각의 위치가 동기화되어 원격 클라이언트에 반영되는 것을 확인할 수 있습니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_0/tutorial/advanced-tutorial/26_unity_puzzle_position_playmode.gif)

<br>

### 중입 유저 처리하기

게임 중 임의의 퍼즐 조각 위치가 변경된 후에 새로운 유저가 방에 입장할 경우를 생각해 봅시다. 이때, 기존 유저와 신규 유저 사이의 퍼즐 상태는 서로 다릅니다. 새롭게 들어온 유저는 초기의 퍼즐 상태를 가지고 있으므로 기존 유저와 퍼즐 상태를 동기화해 주어야 합니다. 이를 해결하기 위해서 서버 측 로직을 수정해 주겠습니다. 이제 서버는 퍼즐의 위치 정보를 모두 보관하고, 새로운 유저가 입장하면 해당 정보를 이용하여 동기화하도록 로직을 수정합니다.

BasicRoom에 puzzlePositions 맵을 추가합니다. 이 맵은 각 퍼즐 조각별 위치 정보를 관리합니다.

```java
public class BasicRoom implements IRoom<BasicUser> {

    private static final Logger logger = getLogger(BasicRoom.class);

    private IRoomContext roomContext;

    // 코드 추가
    public Map<Integer, Puzzle.PuzzlePosition> puzzlePositions = new HashMap<>();

    ...(생략)...
}
```

이제 PuzzlePositionHandler 코드를 수정해서 퍼즐 위치를 각 방에 저장하도록 합니다.

```Java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionHandler {

    @GameRoomMapping(value = Puzzle.PuzzlePosition.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePosition request) {
        BasicRoom room = ctx.getRoom();

        // 코드 추가
        room.puzzlePositions.put(request.getIndex(), request);
        room.broadcast(Packet.makePacket(request)); // 방 전체 유저에게 메시지를 전송
    }
}

```

새로운 유저가 방에 들어올 때 저중해 둔 퍼즐 위치 정보를 받을 수 있도록 BasicRoom의 onJoinRoom을 수정해 서버에 저장해 둔 퍼즐의 위치 정보를 전송하도록 합니다.

```java
public class BasicRoom implements IRoom<BasicUser> {

    ...(생략)...

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        // 코드 추가
        // 임시로 작성된 코드로, 다음 과정에서 문제를 해결하며 해당 코드는 제거하게 됩니다.
        for(Puzzle.PuzzlePosition puzzlePosition : puzzlePositions.values()) {
            broadcast(Packet.makePacket(puzzlePosition));
        }
        return isSuccess;
    }

    ...(생략)...
}
```

이렇게 수정한 뒤 다시 테스트해 봅니다. 클라이언트 하나에서 먼저 퍼즐 위치를 수정한 뒤 다른 클라이언트에서 게임 방에 참여해 퍼즐 위치가 정상적으로 동기화되는지 확인합니다. 의도와 다르게 여전히 동작하지 않는 것을 확인할 수 있습니다.

이는 onJoinRoom 호출 이후에 클라이언트에서 씬 이동이 일어나기 때문입니다. 즉, 리스너 등록이 채 완료되기 전에 동기화 메시지가 전달되어 문제가 생긴 것입니다. 이 문제를 해결하기 위해서는 클라이언트에서 씬 이동이 끝나고 리스너 등록까지 마친 이후에 퍼즐의 위치 정보를 동기화해야합니다.

이 문제는 당장 해결하지 않고 이후 내용에서 다루겠습니다. 이러한 문제는 바로 다음에 설명할 퍼즐 섞기에서도 여실히 드러납니다. 그러므로 퍼즐 섞기부터 살펴본 후, 이를 해결하기 위해 수정한 내용을 다시 다루도록 합니다.

<br>

## 퍼즐 섞기 구현

퍼즐 위치를 랜덤으로 섞는 로직을 구현해 보겠습니다. 클라이언트에서 퍼즐 위치 섞기를 요청하면 서버에서 퍼즐을 섞은 후 새로운 위치를 결정합니다. 그리고 변경된 위치 정보를 다시 클라이언트로 되돌려주는 것이 기본 아이디어입니다.

### 프로토콜 등록

우선 퍼즐 섞기 요청을 하기 위한 프로토콜을 제작해 보겠습니다. 서버 프로젝트로 이동하여 Puzzle.proto 파일에 프로토콜 명세를 추가합니다. 이 프로토콜은 특별히 클라이언트에서 서버로 보낼 정보가 없으므로 필드가 하나도 없습니다. 이 또한 프로토콜로서 충분히 유의미합니다.

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}

message ScatterPuzzle { } // 퍼즐 섞기 요청 프로토콜
```

프로토콜 버퍼 파일을 수정하면 컴파일도 다시 해 주어야 합니다.

```
./protoc  ./src/main/proto/Puzzle.proto --java_out=./src/main/java --csharp_out=./
```

생성된 C# 클래스를 다시 파인더나 파일 탐색기 등의 프로그램을 이용해 Unity 프로젝트의 Asset/Protocol 폴더로 옮깁니다.

<br>

### 클라이언트 측 구현

유니티 프로젝트로 이동하여 GameManager.cs에 아래와 같이 섞기 요청을 위한 코드를 작성합니다. Scatter 메서드가 호출되면 GameAnvilUser를 통해 새로운 ScatterPuzzle 타입의 메시지가 게임 룸으로 전송됩니다.

```c#
public class GameManager : Monobehaviour {

    ...(생략)...

    public void Scatter()
    {
        connectHandler.getUser().SendUser(new ScatterPuzzle());
    }

    ...(생략)...
}
```

**Hierarchy** 패널의 **Scatter Puzzle Button**을 클릭합니다. **Inspector**의 **Button** 컴포넌트에서 **OnClick** 리스너에 항목을 추가한 뒤 GameManager 컴포넌트를 드래그해 등록하고, 드롭다운에서 Scatter 메서드를 선택합니다.

### 서버 측 구현

섞기 요청이 들어왔을 때의 처리는 앞서 사용한 방식대로 핸들러를 이용합니다. **GameAnvil RoomMessageHandler** 파일 템플릿을 통해 ScatterPuzzleHandler 클래스를 생성합니다. 16개 각 퍼즐의 위치를 랜덤하게 설정한 후 PuzzlePositon 타입의 메시지를 송신합니다. 또한 서버의 puzzlePositions 맵도 새로운 위치 정보로 갱신합니다.

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;
import java.util.stream.Stream;

@GameAnvilController
public class ScatterPuzzleHandler {
    private static final int mapSize = 400;

    private static class Point {
        public int x;
        public int y;

        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    }

    @GameRoomMapping(value = Puzzle.ScatterPuzzle.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.ScatterPuzzle request) {
        BasicRoom room = ctx.getRoom();
        room.puzzlePositions = new HashMap<>();
        List<ScatterPuzzleHandler.Point> random = IntStream.rangeClosed(-4, 4).boxed()
                .map(i -> new Point(i * mapSize / 4, i % 2 == 0 ? mapSize : -mapSize))
                .collect(Collectors.toList());
        random = Stream.concat(random.stream(), random.stream().map(p -> new Point(p.y, p.x))).collect(Collectors.toList());
        Collections.shuffle(random);

        for (int i = 0; i < 16; i++) {
            ScatterPuzzleHandler.Point point = random.get(i);
            int x = 1300 + point.x;
            int y = 550 + point.y;

            Puzzle.PuzzlePosition puzzlePosition = Puzzle.PuzzlePosition.newBuilder().setIndex(i).setPositionX(x).setPositionY(y).build();
            room.puzzlePositions.put(i, puzzlePosition);
            room.broadcast(Packet.makePacket(puzzlePosition));
        }
    }
}
```

이제 클라이언트 측의 전송 기능과 서버 측의 응답 기능이 모두 완성되었습니다.

### 퍼즐 섞기 기능 확인

Unity 에디터에서 플레이 모드로 진입합니다. Scatter Puzzle 버튼을 클릭하여 퍼즐 섞기 기능이 잘 작동하는지 확인합니다.

## 더 나은 중입 유저 처리

앞에서 중입 유저를 처리하는 과정에서 동기화 문제가 발생했습니다. 그 원인은 유저가 방에 입장하는 시점을 퍼즐 조각의 위치 동기화 시점으로 적합하다고 생각했기 때문입니다. 하지만 우리가 구현 중인 퍼즐 게임은 유저가 방에 입장하는 시점에 씬 이동이 발생합니다.

그러므로 우리는 리스너 등록과 onJoinRoom 콜백 호출의 두 가지 시점으로 나누어서 생각해야 합니다. 이는 게임 클라이언트의 구현에 따라 문제가 없을 수도 있고, 문제가 될 수도 있습니다. onJoinRoom 콜백 호출 이후 씬 이동이 시작되기 때문에 씬 이동이 완료된 직후에 클라이언트가 직접 서버로 퍼즐 위치 동기화를 요청하고자 합니다.

### 프로토콜 등록

서버 프로젝트로 이동하여, 퍼즐 위치 동기화를 요청하기 위한 프로토콜을 Puzzle.proto에 추가합니다.

```protobuf
syntax="proto3";

package protocol;

message PuzzlePosition
{
	int32 Index = 1;
	int32 PositionX = 2;
	int32 PositionY = 3;
	bool OnEndDrag = 4;
}

message ScatterPuzzle {}

message PuzzlePositionReq {} // 퍼즐 동기화 요청
```

프로토콜을 다시 컴파일한 후 생성된 C# 클래스를 Unity 프로젝트로 옮깁니다.

<br>

### 클라이언트 측 구현

씬 이동 직후 서버에 퍼즐 위치를 요청하도록 하기 위해서는 GameScence으로 씬 이동 직후 실행되는 Start 함수를 수정해야 합니다. GameManager의 Start 메서드에 다음과 같이 새로 만든 PuzzlePositionReq 프로토콜 메시지를 전송하는 코드를 추가합니다.

```c#
public class GameManager : MonoBehaviour
{
    ...(생략)...

    void Start()
    {
        connectHandler = GameObject.Find("ConnectHandler").GetComponent<ConnectHandler>();

        roomIdText.text = "PuzzleRoom:" + connectHandler.roomId;

        connectHandler.getUser().SetMessageCallback<MessageBroadcast>((_, _, messageBroadcast) =>
        {
            ChatLogText.text += messageBroadcast.Message;
        });

        connectHandler.getUser().SetMessageCallback<PuzzlePosition>((_, _, puzzlePosition) =>
        {
            Puzzle puzzle = GameObject.Find("Puzzle " + puzzlePosition.Index).GetComponent<Puzzle>();
            puzzle.transform.position = new Vector2(puzzlePosition.PositionX, puzzlePosition.PositionY);

            if (puzzlePosition.OnEndDrag)
            {
                puzzle.FixPosition();
            }
        });

        // 코드 추가
        connectHandler.getUser().SendUser(new PuzzlePositionReq());
    }

  	...(생략)...
}
```

<br>

### 서버 측 구현

onJoinRoom에 잘못 구현했던 퍼즐 위치 송신 코드는 제거하고, Puzzle.PuzzlePositionReq 메시지에 대한 핸들러 PuzzlePositionReqHandler 클래스를 생성 후 새롭게 작성합니다.

```java
package org.example.handler;

import com.nhn.gameanvil.common.GameAnvilController;
import com.nhn.gameanvil.game.GameRoomMapping;
import com.nhn.gameanvil.game.IRoomDispatchContext;
import com.nhn.gameanvil.packet.Packet;
import org.example.game.BasicRoom;
import protocol.Puzzle;

@GameAnvilController
public class PuzzlePositionReqHandler {
    @GameRoomMapping(value = Puzzle.PuzzlePositionReq.class, loadClass =  BasicRoom.class)
    public void execute(IRoomDispatchContext ctx, Puzzle.PuzzlePositionReq request) {
        BasicRoom room = ctx.getRoom();
        room.puzzlePositions.values().stream().forEach(puzzlePosition -> { room.broadcast(Packet.makePacket(puzzlePosition)); });
    }
}

```

<br>

### 중입 유저 처리 확인

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 이제 빌드된 게임에서 방을 생성하고 퍼즐 섞기를 실행합니다. 그 상태에서 Unity 에디터의 플레이 모드로 진입하여 이 방에 참가한 후 퍼즐 위치가 동기화되는 것을 확인합니다.

## 유저 매치메이킹 구현

### 서버 측 구현

유저 매치 메이킹은 유저들의 매치 메이킹 요청을 한데 모아 적절한 기준에 맞춰 비슷한 수준의 유저들끼리 서로 같은 방에서 게임을 시작할 수 있게 합니다. 승점이나 점수 등 다양한 요소를 사용자가 직접 구현해 유저들을 적절하게 구분하고 매칭할 수 있습니다. 여기에서는 유저 2명을 하나의 게임으로 매칭해 주는 로직을 구현합니다.

프로젝트 패널에서 Main 클래스가 위치한 경로를 마우스 오른쪽 버튼으로 클릭한 뒤 **New > Package**를 선택해 **match**라는 이름의 새로운 패키지를 생성합니다. 그리고 **match** 패키지를 다시 마우스 오른쪽 버튼으로 클릭한 뒤 **New > GameAnvil UserMatchInfo**를 선택합니다. 파일 생성 대화 상자가 열리면 **File name**에 **BasicUserMatchInfo로**를 입력한 뒤 **OK**를 클릭합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/27_create_user_match_info.png)

이 클래스에는 매칭에 사용될 유저의 정보를 담게 됩니다. 매치 메이킹에 사용될 요소가 있다면 여기에 추가하면 됩니다. 이번 예제에서는 별다른 요소를 추가하지 않고, 기본적으로 구현된 메서드만을 사용하겠습니다. 한 가지 주의할 점은 getId() 메서드가 반드시 요청한 유저의 아이디를 반환하게 구현되어있는지 확인합니다. 그리고 파티 매치메이킹 기능은 사용하지 않으므로 0을 반환하도록 설정합니다.

```java
package org.example.match;

import com.nhn.gameanvil.node.match.usermatch.AbstractUserMatchInfo;

public class BasicUserMatchInfo extends AbstractUserMatchInfo implements Comparable<BasicUserMatchInfo> {

    private int userId;

    public BasicUserMatchInfo(int userId) {
        this.userId = userId;
    }

    @Override
    public int getId() {
        return userId;
    }

    @Override
    public int getPartySize() {
        return 0;
    }

    @Override
    public int compareTo(BasicUserMatchInfo o) {
        return 0;
    }
}

```

이러한 UserMatchInfo는 클라이언트가 유저 매치 메이킹을 요청할 때 서버의 게임 유저에서 onMatchUser 콜백을 구현하는 과정에서 생성한 후 사용합니다. GameAnvil은 기본적인 유저 매치 메이커를 제공합니다. 아래의 onMatchUser는 이러한 엔진의 기본 유저 매치 메이킹을 matchUser API를 통해 사용하고 있습니다.

```java
public class BasicUser implements IUser {

    ...(생략)...

    @Override
    public boolean onMatchUser(String roomType, String matchingGroup, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;

        try {
            BasicUserMatchInfo term = new BasicUserMatchInfo(userContext.getUserId());
            return userContext.matchUser(matchingGroup, roomType, term);
        } catch (TimeoutException | NodeNotFoundException | GameAnvilException e) {
            logger.error("BasicUser::onMatchUser", e);
        }

        return isSuccess;
    }

    ...(생략)...
}
```

유저 매치 메이킹을 사용하기 위한 기본적인 준비가 되었으면 이제 실제 매치 메이킹을 수행하는 매치 메이커를 작성합니다.

프로젝트 패널의 **match** 패키지를 마우스 오른쪽 버튼으로 클릭한 뒤 **New > GameAnvil UserMatchMaker**를 선택합니다. 파일 생성 대화 상자가 열리면 **File name**에 **BasicUserMatchMaker**, **Room**에 **BasicRoom**, **User Match Info**에 **BasicUserMatchInfo**를 입력한 뒤 **OK**를 클릭합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/28_create_user_match_maker.png)

생성자에서는 부모 클래스의 생성자를 호출하면서 인자로 매치 인원 수와 매치 신청 유효 시간을 전달합니다. 유효 시간이 지나면 해당 매치 요청은 자동으로 취소됩니다. 그리고 실제 매치 메이킹을 수행하는 match 메서드는 내부적으로 엔진에 의해 1초에 한 번씩 호출됩니다.

getMatchRequests는 인자로 매칭을 위한 최소 인원수를 받아 현재 매칭 풀 전체를 조회하여 최적의 매칭을 가능한 만큼 만들어 냅니다. 즉, 매칭 요청이 많이 쌓여 있다면 getMatchRequests에 의해 한 번에 100개 또는 1,000개의 매칭도 만들어질 수 있습니다. 이때, 매칭이 성공하면 매칭된 유저들의 UserMatchInfo 목록을 반환합니다. 이 목록을 엔진에서 제공하는 matchSingles API에 전달하면 해당 목록 안의 유저들에 대한 방 생성 및 이동이 알아서 진행됩니다.

만일 매칭에 충분한 요청이 쌓이지 않았거나, 조건에 맞는 대상이 없을 경우에는 null을 반환합니다. 이 경우에는 다음 1초 후의 match 호출에서 다시 동일한 매칭 검색이 수행됩니다.

```java
package org.example.match;

import com.nhn.gameanvil.game.GameAnvilUserMatchMaker;
import com.nhn.gameanvil.node.match.AbstractUserMatchMaker;
import org.example.game.BasicRoom;

import java.util.List;

@GameAnvilUserMatchMaker(loadClass = BasicRoom.class)
public class BasicUserMatchMaker extends AbstractUserMatchMaker<BasicUserMatchInfo> {

    public BasicUserMatchMaker() {
        super(2, 5000);
    }

    private int matchSize = 2;

    @Override
    public void onMatch() {
        List<BasicUserMatchInfo> matchRequests = getMatchRequests(matchSize);

        if (matchRequests == null) {
            return;
        }

        int matchingAmount = matchSingles(matchRequests);
    }

    @Override
    public boolean onRefill(BasicUserMatchInfo matchReq) {
        return false;
    }
}
```

<br>

### 클라이언트 측 구현

매치 메이킹 로직은 모두 서버에 구현되어 있기 때문에 클라이언트에서는 매치 메이킹이 필요한 시점에 요청을 보내기만 하면 됩니다. ConnectHandler에 MatchUser 메서드를 추가합니다. 그리고 매치 메이킹이 끝난 시점에 씬을 이동하도록 코드를 추가합니다.

```c#
public class ConnectHandler : MonoBehaviour
{
	...(생략)...

    public void MatchUser()
    {
        try
        {
            getUser().OnMatchUserDone += OnMatchUserDone;
            var result = getUser().MatchUserStart("ROOM_TYPE_BASIC", string.Empty);
            Debug.Log("MATCHING_USER...");
            logText.text = "MATCHING_USER...";
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
            throw;
        }
    }

    private void OnMatchUserDone(GameAnvilUser user, ResultCodeMatchUserDone resultCode, MatchResult matchResult)
    {
        getUser().OnMatchUserDone -= OnMatchUserDone;
        Debug.Log("Room Id : " + matchResult.RoomId);
        this.roomId = matchResult.RoomId;
        SceneManager.LoadScene("GameScene");
    }

	...(생략)...
}
```

씬에서 MatchUser 버튼의 OnClick 리스너에 ConnectHandler 컴포넌트를 드래그해서 등록하고, 드롭다운에서 MatchUser 메서드를 선택합니다.

### 유저 매치 메이킹 테스트

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이합니다. 그 상태로 Unity 에디터에서 플레이 모드에 진입합니다. 양측에서 모두 User Match Making 버튼을 눌러 매칭이 성사되고 동일한 방 번호로 묶이게 되는 것을 확인합니다.

## 룸 매치 메이킹 구현

룸 매치 메이킹은 매치 메이커가 관리하는 방들 중에서 유저의 요구 사항에 가장 적합한 방으로 자동 입장시킬 수 있는 기능입니다. 즉, 유저 매치 메이킹이 유저와 유저를 매칭시켜주는 기능이라면, 룸 매치 메이킹은 유저와 방을 매칭시켜주는 기능입니다. 이때, 구현 방식에 따라서 다양한 조건으로 유저를 방에 매칭할 수 있습니다. 여기에서는 아직 정원이 차지 않은 방 중에 인원이 가장 적은 방으로 입장하는 매치 메이킹을 구현합니다.

우선 매치 메이킹을 실제로 수행하는 클래스가 필요합니다. 그리고 룸 매치 메이커에서 방을 관리하기 위한 정보를 담는 인스턴스가 각 방마다 하나씩 있어야 합니다. 마지막으로 유저가 매치 메이킹을 신청할 때마다 유저의 요구 사항을 담은 신청서 인스턴스가 필요합니다. 이렇게 세 개의 새로운 클래스를 작성해 보겠습니다.

추가로 기존 로직을 일부 수정합니다. 룸 매치 메이킹은 모든 방이 아닌 룸 매치 메이킹 대상으로 신청한 방들만을 대상으로 수행됩니다. 따라서 방 생성 시점에 룸 매치 메이킹을 대상으로 신청하는 코드를 추가합니다.

### 서버 측 구현

우선 매치 메이킹 요청을 나타낼 클래스를 구현합니다. BasicRoomMatchForm 클래스를 생성합니다.

프로젝트 패널의 **match** 패키지를 마우스 오른쪽 버튼으로 클릭한 뒤 **New > GameAnvil RoomMatchForm**를 선택합니다. 파일 생성 대화 상자가 열리면 **File name**에 **BasicRoomMatchForm**을 입력한 뒤 **OK**를 클릭합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/29_create_room_match_form.png)

유저가 매치 메이킹 요청을 할 때마다 BasicRoomMatchForm 객체가 생성되어 사용됩니다.

```java
package org.example.match;

import com.nhn.gameanvil.node.match.roommatch.AbstractRoomMatchForm;

public class BasicRoomMatchForm extends AbstractRoomMatchForm {
    public BasicRoomMatchForm() {
        super();
    }
}
```

다음은 매칭 대상이 되는 방의 정보를 표현하는 클래스를 구현합니다. 다음과 같이 BasicRoomMatchInfo 클래스를 생성합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/30_create_room_match_info.png)

이때, 방의 최대 정원은 MAX_ENTRY_USER 필드에 지정된 4명입니다. 이러한 최대 정원과 roomId를 반드시 상속 받은 AbstractRoomMatchInfo 생성자에 인자로 전달해야 합니다.

```java
package org.example.match;

import com.nhn.gameanvil.node.match.roommatch.AbstractRoomMatchInfo;

public class BasicRoomMatchInfo extends AbstractRoomMatchInfo {
    private static final int MAX_ENTRY_USER = 4;

    public BasicRoomMatchInfo(int roomId) {
        super(roomId, MAX_ENTRY_USER);
    }
}
```

다음으로 실제로 룸 매치 메이킹을 처리할 룸 매치 메이커를 생성합니다.

![](https://static.toastoven.net/prod_gameanvil/images/v2_1/tutorial/advanced-tutorial/31_create_room_match_maker.png)

다음과 같이 BasicRoomMatchMaker 가 생성됩니다.

```java
package org.example.match;

import com.nhn.gameanvil.game.GameAnvilRoomMatchMaker;
import com.nhn.gameanvil.node.match.AbstractRoomMatchMaker;
import org.example.game.BasicRoom;

@GameAnvilRoomMatchMaker(loadClass = BasicRoom.class)
public class BasicRoomMatchMaker extends AbstractRoomMatchMaker<BasicRoomMatchForm, BasicRoomMatchInfo> {

    @Override
    public boolean onMatch(BasicRoomMatchForm roomMatchForm, BasicRoomMatchInfo roomMatchInfo, Object... args) {
        return false;
    }

    @Override
    public int compare(BasicRoomMatchInfo o1, BasicRoomMatchInfo o2) {
        return 0;
    }
}
```

compare 메서드는 매칭 풀에 들어 있는 방을 정렬하는 조건을 구현합니다. 예제는 인원수에 따라 방을 정렬하도록 구현했습니다.

이제 룸 매치 메이킹을 위한 준비가 거의 끝났습니다. 클라이언트가 룸 매치 요청을 보내면 서버의 BasicUser는 onMatchRoom 콜백이 호출됩니다. 이 콜백에서 앞서 살펴본 BasicRoomMatchForm 객체를 생성한 뒤 matchRoom API에 인자로 전달하여 호출합니다. 즉, 클라이언트가 보낸 룸 매칭 요청을 여기에서 매치 메이커로 전달했습니다.

```java
public class BasicUser implements IUser {

    ...(생략)...

    @Override
    public RoomMatchResult onMatchRoom(String roomType, String matchingGroup, String matchingUserCategory, IPayload payload) {
        BasicRoomMatchForm gameRoomMatchForm = new BasicRoomMatchForm();
        try {
            return userContext.matchRoom(matchingGroup, roomType, gameRoomMatchForm);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicUser::onMatchRoom", e);
            return RoomMatchResult.FAILED;
        }
    }

    ...(생략)...
}
```

방이 생성된 시점에 룸 매치 메이킹 대상으로 만들기 위해서 BasicRoom의 onCreateRoom 콜백에서 아래와 같이 registerRoomMatch API를 호출합니다.

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    ...(생략)...

    @Override
    public boolean onCreateRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        try {
            roomContext.registerRoomMatch(gameRoomMatchInfo, user.getUserContext().getUserId());
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicRoom::onCreateRoom()", e);
        }
        return isSuccess;
    }

    ...(생략)...
}
```

또한 방 정보가 변동될 때마다 룸 매치 메이킹 정보도 갱신되어야 합니다. 참고로 방의 인원수 변동은 룸 매치 메이커가 자동으로 동기화합니다. 그러므로 인원수를 제외한 추가 정보에 대한 갱신만 수행하면 됩니다.

다음은 onJoinRoom 콜백을 수정해 방에 유저가 참여할 때 매치 메이킹 정보를 갱신하는 코드입니다.

```java
public class BasicRoom extends BaseRoom<BasicUser> {

    ...(생략)...

    @Override
    public boolean onJoinRoom(BasicUser user, IPayload payload, IPayload outPayload) {
        boolean isSuccess = true;
        try {
            roomContext.updateRoomMatch(gameRoomMatchInfo);
        } catch (NodeNotFoundException | TimeoutException | GameAnvilException e) {
            logger.error("BasicRoom::onJoinRoom()", e);
        }
        return isSuccess;
    }

    ...(생략)...
}
```

<br>

### 클라이언트 구현

유저 매치 메이킹과 마찬가지로 클라이언트는 매치 메이킹이 필요한 시점에 요청을 보내기만 하면 됩니다. ConnectHandler에 RoomMatchMaking 메서드를 추가합니다.

```c#
public class ConnectHandler : MonoBehaviour {

    ...(생략)...

    public async void MatchRoom()
    {
        try
        {
            var result = await user.MatchRoom(true, true, "ROOM_TYPE_BASIC", "", "");
            if (result.ErrorCode == ResultCodeMatchRoom.MATCH_ROOM_SUCCESS)
            {
                Debug.Log("Room Id : " + result.Data.RoomId);
                this.roomId = result.Data.RoomId;
                SceneManager.LoadScene("GameScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            logText.text = e.ToString();
            throw;
        }
    }

    ...(생략)...
}
```

씬에서 MatchRoom 버튼의 OnClick 리스너에 ConnectHandler 컴포넌트를 드래그해 등록하고, 드롭다운 메뉴에서 MatchRoom 메서드를 선택합니다.

<br>

### 룸 매치 메이킹 테스트

Unity에서 `cmd+b` 또는 `ctrl+b`로 빌드 후 플레이 상태에서 방을 생성합니다. 그 상태로 Unity 에디터에서 플레이 모드로 진입합니다. 플레이 모드에서 Room Match Making 버튼을 눌러 빌드 모드에서 생성한 방으로 이동하는지 확인합니다.

<br>

## Room 떠나기 구현

마지막으로 방을 떠나는 기능 구현을 위해 Unity 클라이언트의 GameManager에 아래 메서드를 추가합니다.

```c#
public class GameManager : MonoBehaviour
{
    ...(생략)...

    public async void LeaveRoom()
    {
        try
        {
            var result = await connectHandler.getUser().LeaveRoom();
            Debug.Log(result.ErrorCode);
            if (result.ErrorCode == ResultCodeLeaveRoom.LEAVE_ROOM_SUCCESS)
            {
                Destroy(connectHandler.gameObject);
                SceneManager.LoadScene("ConnectScene");
            }
        }
        catch (Exception e)
        {
            Debug.Log(e.ToString());
            throw;
        }
    }

    ...(생략)...
}
```

씬에서 Leave Room 버튼의 OnClick 리스너에 GameManager 컴포넌트를 드래그해 등록하고, 드롭다운 메뉴에서 LeaveRoom 메서드를 선택합니다.

## 프로젝트 마무리

이상 GameAnvil과 Unity를 이용해 실시간 멀티플레이가 가능한 퍼즐 게임을 구현해 보았습니다. 그 과정에서 GameAnvil의 핵심 기능 다수를 사용하였습니다. 하지만 GameAnvil은 이 튜토리얼에 포함되지 않은 더욱 풍성하고 다양한 기능들을 지원합니다. 이러한 기능들에 대해서는 이어지는 문서를 참고하십시오. 또한 함께 제공되는 레퍼런스 샘플 프로젝트와 JavaDoc도 GameAnvil을 이해하는 데 많은 도움이 될 것입니다.
