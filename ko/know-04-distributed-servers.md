## Game > GameAnvil > 서버 개념 설명 > 분산 서버



## 1. 분산 서버

앞서 기본 개념에서 GameAnvil의 노드 구성은 아래의 그림과 같다고 했습니다. 즉, 하나의 프로세스는 여러 가지의 노드를 자유롭게 구성해서 구동할 수 있습니다. 단, 모든 GameAnvil 프로세스는 반드시 하나의 IPC(Inter-Process Communication) 노드가 필수적으로 포함됩니다. 이 IPC 노드는 GameAnvil 프로세스 간의 통신을 담당합니다. 사실 실제 네트워크 처리를 담당하는 로우 레벨 노드가 더 있지만 사용자는 이 부분을 통틀어 IPC 노드로 이해하면 됩니다.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



### 1.1. 노드간 통신

이러한 IPC 노드를 통해 두 개 이상의 GameAnvil 프로세스가 통신하는 모습은 아래의 그림과 같습니다. 이 그림에서 두 개의 GameAnvil 프로세스는 서로 다른 구성의 노드들을 구동합니다. 이때, 각각의 노드들은 서로 통신이 가능합니다.

서로 다른 프로세스의 노드들과 통신하기 위해 각 노드들은 IPC 노드를 통해 메시지를 전달합니다. 반면에 동일한 프로세스 상의 노드들은 큐를 이용해서 상호 통신합니다. 그러므로 이 경우에는 IPC 노드를 통하지 않습니다.

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/IPC.png)



### 1.2. Meetpoint

그럼, 이러한 IPC를 위해 프로세스는 상호 접속을 어떻게 하는 걸까요? 그 답은 Meetpoint입니다. GameAnvil은 Meetpoint 주소를 설정할 수 있습니다. 아래와 같은 형태인데 하나 이상의 IP 주소 쌍을 설정할 수 있습니다. GameAnvil 프로세스는 최초 구동 시에 설정된 Meetpoint 주소 중 하나에 대해 접속을 시도하여 전체 서버군 정보를 동기화합니다.

```
"common": {

    "meetEndPoints": [
      "10.1.2.1:18000",
      "10.1.2.2:18000",
    ],

}
```

