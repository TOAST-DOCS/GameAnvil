<!-- pre-align:aligned sig=ca641a919c72 -->

<a id="game-gameanvil-console-user-guide-state"></a>
## Game > GameAnvil > コンソール使用ガイド > 状態 { #game-gameanvil-console-user-guide-state }


<a id="state"></a>
## 状態 { #state }
1つのゲームサービスのために、複数台のゲームサーバーを構成できます。そして、各ゲームサーバーは複数のノードで構成できます。これらのサーバーとノードは、別々の状態値を持ちます。このドキュメントの残りの部分では、サーバーとノードの状態について説明します。



<a id="server-state"></a>
## サーバーの状態 { #server-state }

サーバーの状態は、プロセス(S/W)とインスタンス(H/W)の状態を複合的に表します。サーバー管理ページで確認できるダッシュボードは、これらのサーバー状態を一覧表示し、各状態に属するサーバーの台数を表示しています。また、区別しやすい色で各状態を表します。

![図](https://static.toastoven.net/prod_gameanvil/images/console/v2/state/monitoring_dashboard.png)

<br>
次は、各サーバーの状態に関する説明です。

| サーバーの状態       | 意味                                                                                            |
|------------|-------------------------------------------------------------------------------------------|
| ERROR       | サーバーが正常でない全ての状態を表します。この場合、そのサーバーには再起動コマンドのみ実行可能です。再起動してSTANDBY状態になると、他のコマンドが有効になります。 |
| RUNNING     | サーバーが正常に稼働している状態です。つまり、ゲームサーバープロセスが問題なく正常に起動しました。                                            |
| STANDBY     | サーバーインスタンスが正常に起動し、GameAnvilサービスと通信が可能です。この場合、開始コマンドを使用してゲームサーバープロセスを起動できます。     |
| SAFE PAUSE | サーバーに属する一部のノードがSafe Pauseを実行中です。                                                       |
| TRANSIT     | 2つの状態間を遷移中です。目標の状態への遷移が完了するまで、そのサーバーに対してコマンドを実行することはできません。                                     |


<a id="node-state"></a>
## ノードの状態 { #node-state }

ノードの状態は、1つのゲームサーバーを構成する複数のノードの状態を示します。また、同一サーバー上のノードであっても、別々の状態の場合があります。 

| ノードの状態 | 意味 |
| ----------- | --------------------------- |
| INIT | ノード初期化中 |
| PREPARE | ノード初期化後の準備作業進行中 |
| READY | ノード起動完了及びサービス可能状態 |
| READY(LOCK) | SAFE PAUSE中のノードからユーザー/ルームの転送を受けている最中(排他的サービス可能状態) |
| PAUSE | 一時停止状態 |
| RESUME | PAUSEから再開 |
| SAFE PAUSE | SAFE PAUSE進行中です。SAFE PAUSEが完了するとPAUSE状態になります。 |
| INVALID | ノードの負荷が高い場合、一時的にノード情報を確認できない状態(自動的に復旧) |
| DISABLE | INVALID状態が長く続くか、障害などによりサービス不可能な状態(自動的に復旧しない) |



<a id="states-related-to-safe-pause"></a>
## Safe Pause関連の状態 { #states-related-to-safe-pause }

サーバーの状態とノードの状態の中には、Safe Pauseに関連するものがあります。任意のノードに対してSafe Pauseを実行する場合、サーバーとノードはその状態に遷移(Transit)します。次はこれに関する追加説明です。


<a id="states-related-to-safe-pause-state-of-nodes-undergoing-safe-pause"></a>
#### Safe Pauseを実行するノードの状態

* SAFE PAUSE: 任意のノードを安全に停止(Safe Pause)させる場合、そのノードが処理中だった情報は安全に他のノードへ移管されます。このような一連の過程はSAFE PAUSE状態として表されます。この状態のノードは、全ての情報を移管した後、PAUSE状態になります。
* READY(LOCK): Safe Pauseさせるノードで処理中だったユーザー/ルームなどの情報を受け入れる(移管される)ノードの状態です。この状態のノードは、移管が完了するまで外部コマンドを実行できません。つまり、Safe Pause完了まで排他的なREADY状態になります。

<a id="states-related-to-safe-pause-state-of-the-server-running-safepause"></a>
#### Safe Pauseを実行するサーバーの状態

* サーバーを構成する複数のノードのうち、一部がSAFE PAUSEまたはREADY(LOCK)状態である場合があります。このようなサーバーの状態は、包括的にSAFE PAUSE状態として表されます。


Safe Pauseに関するより詳細な説明は、[Safe Pause](console-07-safe-pause.md)を参照してください。
