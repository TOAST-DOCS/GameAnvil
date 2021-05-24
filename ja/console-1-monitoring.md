## Game > GameAnvil > コンソール使用ガイド > モニタリング

## ダッシュボードモニタリング

管理ページで登録したマシンとインスタンス、各ノードの状態を確認できます。

![monitoring_dashboard_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_dashboard_main_ja.png)

- マシン
    - 管理ページに登録した全体マシン数と、マシンにインストールしたAgentと正常に接続された全体マシン数を表します。
        - 接続されたマシン数 / 全体マシン数
    - Agentと接続に問題がある場合、赤色状態表示になります。
    - モニタリングボタンを押すと「[システムモニタリング](./console-1-monitoring/#_5)」ページに移動します。
- インスタンス
    - 管理ページに登録した全てのインスタンス数と、動作(Running)中のインスタンス数を表示します。
        - 動作中のインスタンス数 / すべてのインスタンス数
    - エラーのインスタンスが1つでもあれば、赤色状態表示になります。
    - モニタリングボタンを押すと「[インスタンスモニタリング](./console-1-monitoring/#_9)」ページに移動します。
- ユーザー
    - GAMEノードに接続したユーザー数
- ルーム
    - GAMEノードに作成されたルーム数
- セッション
    - GATEWAYノードで確認されたセッション数
- GATEWAY
    - 全体GATEWAYノード数と、READY状態のGATEWAYノード数を表示します。 
        - READY状態のGATEWAYノード数 / 全体GATEWAYノード数
    - DISABLE状態のGATEWAYノードが1つでもある場合、赤色状態表示になります。
    - モニタリングボタンを押すと「[GATEWAYノードモニタリング](./console-1-monitoring/#gateway)」ページに移動します。
- GAME
    - 全体GAMEノード数とREADY状態のGAMEノード数を表示します。 
        - READY状態のGAMEノード数 / 全体GAMEノード数
    - DISABLE状態のGAMEノードが1つでもある場合、赤色状態表示になります。
    - モニタリングボタンを押すと「[GAMEノードモニタリング](./console-1-monitoring/#game)」ページに移動します。
- SUPPORT
    - 全体SUPPORTノード数と、READY状態のSUPPORTノード数を表示します。
        - READY状態のSUPPORTノード数 / 全体SUPPORTノード数
    - DISABLE状態のSUPPORTノードが1つでもある場合、赤色状態表示になります。
    - モニタリングボタンを押すと「[SUPPORTノードモニタリング](./console-1-monitoring/#support)」ページに移動します。
- MATCH
    - 全体MATCHノード数とREADY状態のMATCHノード数を表示します。
        - READY状態のMATCHノード数 / 全体MATCHノード数
    - DISABLE状態のMATCHノードが1つでもある場合、赤色状態表示になります。
    - モニタリングボタンを押すと「[MATCHノードモニタリング](./console-1-monitoring/#match)」ページに移動します。

### 同時接続者変化グラフ

GAMEノードに接続したユーザーの数をグラフでひと目で確認できます。

![monitoring_dashboard_graph](https://static.toastoven.net/prod_gameanvil/images/monitoring_dashboard_graph_ja.png)

- グラフは1分毎に自動更新され、今日と昨日、そして先週のグラフを確認できます。
- 昨日または先週のデータがない場合、グラフは表示されません。
- フィルタを利用して、特定GAMEノードサービスの同時接続者グラフを確認できます。



## ユーザー分布モニタリング

GAMEノードの接続した同時接続者、分布平均、そしてマシン、インスタンス、ノードの数を確認できます。

![monitoring_userdistribution_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_userdistribution_main_ja.png)

- 同時接続者
    - 動作中(READY)のGAMEノードで集計された同時接続者数を意味します。
    - フィルタによって同時接続者数は異なります。

- マシン
    - GAMEノードが動作中(READY)のマシンの数を意味します。
    - フィルタによってマシンの数は異なります。

- インスタンス
    - GAMEノードが動作中(READY)のマシンの数を意味します。
    - フィルタによってインスタンス数は異なります。

- GAMEノード
    - GAMEノードが動作中(READY)のノード数を意味します。
    - フィルタによってノード数は異なります。

- 各ノードのユーザー分布の平均
    - 同時接続者を全ての起動中のGAMEノード数で割った平均です。
    - フィルタによって平均は異なります。
        - 同時接続者 / 全ての起動中のGAMEノードの数

※ユーザーモニタリングページは自動更新をサポートしません。<br>
データの更新が必要な時は、更新ボタンを押して更新してください。

### ユーザー分布グラフ

グラフはタブ<span style='color: #EB4927'>①</span>でマシン、インスタンス、ノードごとにユーザー分布状態を確認できます。

- マシン
    - GAMEノードが動作中(READY)のマシンのみ表示されます。
    - 縦軸のラベルは、「ホスト名」に表示されます。
- インスタンス
    - GAMEノードが動作中(READY)のインスタンスのみ表示されます。
    - 縦軸のラベルは、「インスタンス名@ホスト名」に表示されます。
- ノード
    - 動作中(READY)のGAMEノードのみ表示されます。
    - 縦軸のラベルは、「ノードID@ホスト名」に表示されます。

グラフのデータは基本的に50個のみ表示され、50個以上の情報は「さらに表示」<span style='color: #EB4927'>②</span>を押して確認できます。

## システムモニタリング

システムモニタリングでは、マシンとインスタンスの状態と簡略な情報を簡単に確認できます。

![monitoring_system_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_main_ja.png)

### ツリービュー

管理ページで登録したマシンとインスタンスの階層構造をツリー形式で表示します。

ツリーの構造は、次のように構成されています。

![monitoring_system_tree](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_tree_ja.png)

マシンとそれぞれのマシンに属すインスタンスが表示されます。
ホスト名またはインスタンス名を選択すると、それぞれのマシンまたはインスタンス詳細ページが表示されます。

またホスト名検索を利用して、マシンを簡単に探せます。

### マシン(ホスト名)を選択した時

選択したマシンと、属すインスタンスの情報を表示します。

![monitoring_system_machine](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_machine_ja.png)

- マシンとインスタンスの情報

1. マシン
    - ホスト名：マシンのホスト名
    - マシン状態： Agentとの接続状態
    - IPアドレス：マシンのIPアドレス
    - GameAnvil Agent Port： Agentのポート番号

2. インスタンス
    - インスタンス名
    - インスタンス状態
    - 総接続ノード数：READY状態のノード数 / すべてのノード数
    - ユーザー数：インスタンスに属すGAMEノードに接続したユーザー数
        - GAMEノードが動作中(READY)の時のみ表示
        - その他は「-」で表示
    - ルーム数：インスタンスに属すGAMEノードに作成されたルーム数
        - GAMEノードが動作中(READY)の時のみ表示
        - その他は「-」で表示

### インスタンスを選択した時

選択したインスタンスに詳細な情報を表示します。

![monitoring_system_instance](https://static.toastoven.net/prod_gameanvil/images/monitoring_system_instance_ja.png)

- インスタンスの情報
    - インスタンス名
    - インスタンスの状態
    - インスタンス設定：インスタンス設定情報をポップアップウィンドウで確認
    - GATEWAY
        - 動作中のGATEWAYノード数 / 全体GATEWAYノード数
        - 該当のノードがない場合は「-/-」と表記
    - GAME
        - 動作中のGAMEノード数 / 全体GAMEノード数
        - 該当のノードがない場合は「-/-」と表記
    - SUPPORT
        - 動作中のSUPPORTノード数 / 全体MASUPPORTTCHノード数
        - 該当のノードがない場合は「-/-」と表記
    - MATCH
        - 動作中のMATCHノード数 / 全体MATCHノード数
        - 該当のノードがない場合は「-/-」と表記されます。
    - ユーザー数：インスタンスに属すGAMEノードに接続したユーザー数
        - GAMEノードが動作中(READY)の時のみ表示
        - それ以外は「-」と表示
    - ルーム数：インスタンスに属すGAMEノードに作成されたルーム数
        - GAMEノードが動作中(READY)の時のみ表示
        - それ以外は「-」と表示

## インスタンスモニタリング

管理ページでインスタンスの情報を確認し、設定された各ノードを開始/中止/配布できます。

![monitoring_instance_main](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_main_ja.png)

1. 検索
    - ホスト名やインスタンス名で特定のインスタンスを検索できます。

2. フィルタ
    - インスタンスおよび配布状態とインスタンス設定タイプに応じて特定のインスタンスをフィルタできます。

    ![monitoring_instance_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_filter_ja.png)

### インスタンステーブル

![monitoring_instance_table](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_table_ja.png)

#### アクション<span style='color: #EB4927'>①</span>

テーブルのチェックボックスが選択されたインスタンスに対して、アクションを実行できます。

- 開始
- 中止
- 配布


#### 配布ファイルアップロード<span style='color: #EB4927'>②</span>

管理 > 配布ファイルページと同じように配布ファイルをアップロードできます。

![monitoring_instance_upload](https://static.toastoven.net/prod_gameanvil/images/monitoring_instance_upload_ja.png)


#### テーブル

インスタンステーブルで各インスタンスの状態と、開始/中止/配布アクションを実行できます。

- インスタンス名
- ホスト名
- インスタンス設定
- GATEWAY
    - 動作中のGATEWAYノード数 / 全体GATEWAYノード数
    - 該当のノードがない場合は「-」と表記
- GAME
    - 動作中のGAMEノード数 / 全体GAMEノード数
    - 該当のノードがない場合は「-」と表記
- SUPPORT
    - 動作中のSUPPORTノード数 / 全体SUPPORTノード数
    - 該当のノードがない場合は「-」と表記
- MATCH
    - 動作中のMATCHノード数 / 全体MATCHノード数
    - 該当のノードがない場合は「-」と表記
- インスタンス状態
    - 「エラー」状態では「エラー」<span style='color: #EB4927'>③</span>をクリックすると、詳細エラーメッセージを確認できます。
- 配布状態
    - 「エラー」状態では「エラー」<span style='color: #EB4927'>③</span>をクリックすると、詳細エラーメッセージを確認できます。
- アクション
    - 1つのインスタンスにアクション(開始/中止/配布)を実行できます。

## ノードモニタリング

### ALL

動作中(READY)のすべてのノードをモニタリングできます。

![monitoring_node_all](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_all_ja.png)

- ノードID
- サービスID
    - GAME、SUPPORTノードに限り表示されます。
- サービス名
- ノードタイプ
- インスタンス名
- ホスト名
- ノードの状態
- アクション
    - Resume
        - Pause状態のノードを開始できます。
    - Pause
        - Ready状態のノードを停止させます。
    - ※ MATCHノードはResume、Pauseアクションを実行できません。

#### フィルタ

![monitoring_node_all_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_all_filter_ja.png)

動作中の全てのノードに次のフィルタを適用して特定のノードをフィルタできます。
- ノードタイプ
- ノードの状態
- サービス名

### GATEWAY

動作中(READY)のGATEWAYノードをモニタリングできます。

![monitoring_node_gateway](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_gateway_ja.png)

- ノードID
- ノードタイプ
- インスタンス名
- ホスト名
- セッション数
    - 各GATEWAYノードに接続されたセッション数を表示します。
- ノードの状態
- アクション
    - Resume
        - Pause状態のノードを開始できます。
    - Pause
        - Ready状態のノードを停止させます。

#### フィルタ

![monitoring_node_gateway_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_gateway_filter_ja.png)

動作中のGATEWAYノードに次のフィルタを適用して特定のノードをフィルタできます。
- ノードの状態

### GAME

動作中(READY)のGAMEノードをモニタリングできます。

![monitoring_node_game](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_game_ja.png)

- ノードID
- ノードタイプ
- サービスID
- チャンネルID
- インスタンス名
- ホスト名
- ユーザー数
    - 各GAMEノードに接続したユーザー数を表示します。
- ルーム数
    - 各GAMEノードに作成されたルーム数を表示します。
- ノードの状態
- アクション
    - Resume
        - Pause状態のノードを開始できます。
    - Pause
        - Ready状態のノードを停止させます。

#### フィルタ

![monitoring_node_game_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_game_filter_ja.png)

動作中のGAMEノードに次のフィルタを適用して特定のノードをフィルタできます。
- ノードの状態
- サービス名

### SUPPORT

動作中(READY)のSUPPORTノードをモニタリングできます。

![monitoring_node_support](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_support_ja.png)

- ノードID
- ノードタイプ
- サービスID
- サービス名
- インスタンス名
- ホスト名
- ノードの状態
- アクション
    - Resume
        - Pause状態のノードを開始できます。
    - Pause
        - Ready状態のノードを停止させます。

#### フィルタ

![monitoring_node_support_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_support_filter_ja.png)

動作中のSUPPORTノードに次のフィルタを適用して特定のノードをフィルタできます。
- ノードの状態
- サービス名

### MATCH

動作中(READY)のMATCHノードをモニタリングできます。

![monitoring_node_match](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_match_ja.png)

- ノードID
- ノードタイプ
- インスタンス名
- ホスト名
- Room Match Room Count
- User Match Queue Size
- ノードの状態

#### フィルタ

![monitoring_node_match_filter](https://static.toastoven.net/prod_gameanvil/images/monitoring_node_match_filter_ja.png)

動作中のMATCHノードに次のフィルタを適用して特定のノードをフィルタできます。
- ノードの状態
