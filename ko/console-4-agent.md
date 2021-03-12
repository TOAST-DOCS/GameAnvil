## Game > GameAnvil > 콘솔 사용 가이드 > Agent

Console을 통해 GameAnvil 서버를 실행/중지/배포 하기위해 Agent의 설치가 필요합니다. 

### Agent 다운로드 및 실행하기

* [Agent  다운로드](https://static.toastoven.net/prod_gameanvil/files/gameanvil-agent-1.1.2.tar)

* 다운 받은 tar 파일을 풀면 아래의 파일들을 볼 수 있습니다.

  * 파일 구성

    | 파일 이름                           | 설명                      | 비고 |
    | ----------------------------------- | ------------------------- | ---- |
    | gameanvil-agent-*.jar ( *는 버전명) | gameanvil-agent 파일      |      |
    | gameanvil-agent.properties          | gameanvil-agent 설정 파일 |      |
    | start.sh                            | 실행 스크립트             |      |
    | stop.sh                             | 실행 중지 스크립트        |      |

    

* GameAnvil-Agent 설정 파일 변경

  * server.address 와 server.port를 확인합니다.

    | 설정 이름      | 설명                  | 예시                          | 비고                                                         |
    | -------------- | --------------------- | ----------------------------- | ------------------------------------------------------------ |
    | server.address | Agent가 사용하는 ip   | server.address=10.160.194.108 | 해당 설정값이 공백일 경우 머신에 할당된 모든 IP로 접속할 수 있기 때문에 사용할 IP를 꼭 지정해줍니다. |
    | server.port    | Agent가 사용하는 port | server.port=19080             | console에서 설정된 GameAnvil Agent Port와 값이 동일해야 합니다. (기본값 : 19080) |

    

* start.sh 스크립트를 이용해서 GameAnvil-Agent를 실행합니다.

  * 해당 스크립트의 권한이 설정되어 있지 않다면, chmod 명령어를 통해 적당한 권한을 설정하세요.