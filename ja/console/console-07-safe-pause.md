## Game > GameAnvil > コンソール使用ガイド > Safe Pause

## Safe Pauseとは？

Safe Pauseは、サービスを停止せずに任意のゲームノードを一時中断(Pause)できる機能です。Safe Pauseの出発地ノードは、処理中だった全てのユーザーとルーム情報を、同じサービスを実行中の到着地ノードへリアルタイム転送(Transfer)します。そのため、同じサービスのゲームノードが2つ以上存在しない場合、Safe Pauseを使用できません。

Safe Pauseの過程で、転送を送る側と受ける側は、下の図のようにそれぞれ**SAFE PAUSE**状態と**READY(LOCK)**状態に変わります。**READY(LOCK)**状態のノードは、そのSafe Pauseトランザクションが完了するまで、任意に状態を変更することはできません。つまり、コンソール上でいかなる操作もできません。

![図](https://static.toastoven.net/prod_gameanvil/images/console/v2/safe-pause/safepause_list.png)

Safe Pauseが完了すると、**Safe Pause**状態は**Pause**状態に変わります。当然、そのノードにはいかなるユーザーやルームも存在しないため、安全にシャットダウンできます。パッチやメンテナンスが必要な場合は、この時に行います。**Ready(Lock)**状態もまた、Safe Pauseが完了すると同時に**Ready**状態に切り替わり、コンソール上での操作が可能になります。

## Safe Pauseを使用する

Safe Pauseメニューを選択すると、Safe Pauseの基本画面が表示されます。この画面は、Safe Pauseを実行できるノードの一覧を表示します。Safe Pauseを実行するノードを選択した後、**Safe Pause実行**をクリックすると、以下のようにSafe Pauseを実行する前のポップアップが開き、実行に関するメモを作成できます。

![図](https://static.toastoven.net/prod_gameanvil/images/console/v2/safe-pause/safepause_exec_popup.png)

**実行中**タブをクリックして、現在進行中のSafe Pause一覧を確認できます。

![図](https://static.toastoven.net/prod_gameanvil/images/console/v2/safe-pause/running_safepause_list.png)

Safe Pauseの過程を通じて、**出発地ノード**のユーザーとルームは**到着地ノード**へ転送され、全ての過程が完了した後、**出発地ノード**は全て自動的にPAUSE状態になります。これで、PAUSEされたノードは安全に終了できます。   

![図](https://static.toastoven.net/prod_gameanvil/images/console/v2/safe-pause/paused_node_server_list.png)
