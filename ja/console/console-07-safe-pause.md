## Game > GameAnvil > コンソール使用ガイド > Safe Pause

## Safe Pauseとは？

Safe Pauseは、サービスを停止せずに任意のゲームノードを一時中断(Pause)できる機能です。Safe Pauseの出発地ノードは、処理中だったすべてのユーザーとルームの情報を同じサービスを実行している到着地ノードにリアルタイムで転送(Transfer)します。 したがって、同じサービスのゲームノードが2つ以上存在しない場合、Safe Pauseを使用できません。

Safe Pauseの過程で転送を送信する側と受信する側は、下の図のようにそれぞれ**SAFE PAUSE**状態と**READY(LOCK)**状態になります。**READY(LOCK)**状態のノードは、該当Safe Pauseトランザクションが完了するまで、自由に状態を変更できません。 つまり、コンソール上でいかなる操作もできません。

![図](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/state.png)

Safe Pauseが完了すると、**Safe Pause**状態は**Pause**状態に変わります。 当然、そのノードにはユーザーやルームが存在しないため、安全にシャットダウンできます。パッチや点検が必要な場合は、この時に行ってください。**Ready(Lock)**状態もSafe Pauseが完了すると同時に**Ready**状態に切り替わり、コンソール上で操作が可能になります。

## Safe Pauseを使用する

運営メニューを選択すると、Safe Pauseの基本画面が表示されます。この画面には、現在進行中のSafe Pauseに関連するノードのリストが表示されます。**ノード選択**をクリックすると、以下のようにSafe Pauseを実行するノードを選択するポップアップが開きます。

![図](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/node-selection.png)

Safe Pauseの**出発地ノード**と**到着地ノード**をそれぞれ一つ以上チェックボックスをクリックして選択できます。このとき、処理中のユーザーとルームの数が少ないノードを**到着先ノード**として選択するのが有利です。選択が完了したら、**確認**を押してSafe Pauseを**登録**します。まだ実行段階ではなく、登録段階であることに注意してください。

先に登録したSafe Pause関連ノードは次のようにSafe Pauseの基本画面に表示されます。間違って登録したノードがある場合は、リストの右側の**-**をクリックして削除できます。

![図](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/start.png)

全ての準備が完了したら、**チェック実行**をクリックしてSafe Pauseを開始できます。Safe Pauseが開始されると、次のように**サーバー**メニューで進行中の状態をリアルタイムで確認できます。

![図](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/safe-pausing.png)

Safe Pauseプロセスを通じて**出発地ノード**のユーザーとルームは**到着地ノード**に転送され、すべてのプロセスが完了した後、**出発地ノード**はすべて自動的にPAUSE状態になります。これでPAUSEされたノードは安全に終了できます。  

![図](https://static.toastoven.net/prod_gameanvil/images/console/safe-pause/safe-pause-done.png)
