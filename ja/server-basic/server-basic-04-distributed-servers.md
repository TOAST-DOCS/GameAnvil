## Game > GameAnvil > サーバー概念の説明 > 分散サーバー



## 分散サーバー

前述した基本概念でGameAnvilのノード構成は。次の図と同じだと言いました。1つのプロセスは複数のノードを自由に構成して駆動できます。ただし、すべてのGameAnvilプロセスには、必ず1つのIPC(Inter-Process Communication)ノードが含まれます。このIPCノードはGameAnvilプロセス間の通信を担当します。実際は、ネットワーク処理を担当するローレベルノードがさらにありますが、ユーザーは、これらをまとめてIPCノードと理解してください。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/NodeLayer.png)



### ノード間の通信

このようなIPCノードを介して、2つ以上のGameAnvilプロセスが通信する様子は次の図のとおりです。この図にある2つのGameAnvilプロセスは、互いに異なる構成のノードを駆動します。この時、それぞれのノードは互いに通信が可能です。

異なるプロセスのノードと通信するために、各ノードはIPCノードを通じてメッセージを伝達します。一方で、同じプロセス上のノードはキューを利用して相互通信するため、この場合はIPCノードを通しません。

![Node Layer.png](https://static.toastoven.net/prod_gameanvil/images/IPC.png)



### Meetpoint

このようなIPCのために、プロセスはMeetpointを介して相互接続します。GameAnvilは以下のような形で一つ以上のMeetpoint IPアドレスのペアを設定できます。GameAnvilプロセスは、初回駆動時に設定されたMeetpointアドレスのいずれかに接続を試みて、サーバー群全体の情報を同期します。

```
"common": {

    "meetEndPoints": [
      "10.1.2.1:18000",
      "10.1.2.2:18000",
    ],

}
```
