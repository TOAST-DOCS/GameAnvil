<!-- pre-align:aligned sig=9cc80f834dc0 -->

<a id="game-gameanvil-server-concept-description-distributed-server"></a>
## Game > GameAnvil > サーバー概念説明 > 分散サーバー { #game-gameanvil-server-concept-description-distributed-server }



<a id="distributed-server"></a>
## 分散サーバー { #distributed-server }

先ほどの基本概念でGameAnvilのノード構成は下の図のようだと述べました。つまり、1つのプロセスは様々なノードを自由に構成して稼働できます。ただし、全てのGameAnvilプロセスは必ず1つのIPC(inter-process communication)ノードが必ず含まれます。このIPCノードはGameAnvilプロセス間の通信を担当します。事実、実際のネットワーク処理を担当するローレベルノードがさらにありますが、ユーザーはこの部分をまとめてIPCノードとして理解すればよいです。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



<a id="node-to-node-communication"></a>
### ノード間通信 { #node-to-node-communication }

このようなIPCノードを通じて二つ以上のGameAnvilプロセスが通信する姿は下の図のようです。この図で二つのGameAnvilプロセスは別々の構成のノードを稼働します。この時、それぞれのノードは互いに通信が可能です。

別々のプロセスのノードと通信するために各ノードはIPCノードを通じてメッセージを伝達します。反面、同一プロセス上のノードはキューを利用して相互通信します。したがってこの場合にはIPCノードを通じません。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/IPC.png)



<a id="meetpoint"></a>
### Meetpoint { #meetpoint }

このようなIPCのためにプロセスはMeetpointを通じて相互接続します。GameAnvilは以下のような形で1つ以上のMeetpoint IPアドレスペアを設定できます。GameAnvilプロセスは初回稼働時に設定されたMeetpointアドレスの1つに接続を試み、全体サーバー群情報を同期します。

```
"common": {

    "meetEndPoints": [
      "10.1.2.1:18000",
      "10.1.2.2:18000",
    ],

}
```
