## Game > GameAnvil > コンソール使用ガイド > 管理


## マシン

GameAnvilプロセスが実行される機器を登録し、登録されたマシンを管理するメニューです。 

マシン登録前に、該当のマシンにGameAnvil Agentをインストールする必要があります。[ [GameAnvil-Agent Download](https://static.toastoven.net/prod_gameanvil/files/gameanvil-agent-1.1.4.1.tar) ] 

![management_machine_main.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_main_ja.png)

### マシン登録

GameAnvilプロセスが実行される機器を登録します。 

GameAnvilサービスを有効化した後、まず最初にマシン登録作業を行う必要があります。

![management_machine_register.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_register_ja.png)
* 入力タイプ：コンソールウィンドウに直接マシン情報を入力して登録できます。.csv形式ファイルをアップロードして、複数のマシンを登録できます。
* ホスト名：登録する機器のホスト名を入力します。
* IPアドレス：登録する機器のPublic IPアドレスを入力します。このアドレスでコンソールと通信するため、正確に入力する必要があります。
* GameAnvil Agent Port ：該当の機器にインストールされたGameAnvil Agent Port情報を入力します。 (基本値は19080です。 )
    * ファイルアップロード：テンプレートサンプルファイルをダウンロードして登録するマシン情報を入力し、ファイルをアップロードします。[ [Template File](https://static.toastoven.net/prod_gameanvil/files/GameAnvil_Machine_Template.csv)]


### マシンリスト
登録されたマシンリストを確認できます。ホスト名、IPアドレス、説明検索機能を提供します。 
![management_machine_list.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_list_ja.png)

### マシン設定 

マシン登録後、マスターマシンとロケーション管理マシンを設定します。 

登録したマシンの中から1つのマスターマシンと1つ以上のロケーション管理マシンを選択します。 

![management_machine_setup.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_setup_ja.png)
* マスターマシン： Management Nodeが実行されるマシンを選択します。マスターマシンが登録されている時のみインスタンス / ノードの制御が可能です。 
* ロケーション管理マシン： Location Nodeが実行されるマシンを選択します。
* Javaバージョン： GameAnvilサーバービルドに使用されているJavaのバージョンを選択します。 

Management NodeとLocation Nodeは、GameAnvilを構成する必須ノードですが、コンテンツの実装および制御ができないノードです。 
マシン設定 / マシン初期化機能を利用してManagement NodeとLocation Nodeを管理できます。 

### マシン設定の初期化

マシン設定機能を利用して、登録したマスターマシンとロケーション管理マシン設定を初期化する機能です。

既存のマシン設定値が初期化され、Management NodeとLocation Nodeも終了します。

登録されたすべてのインスタンスの状態が動作中ではない場合にのみ、マシン設定の初期化を行えます。

![management_machine_init.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_init_ja.png)

![management_machine_setup_none.png](https://static.toastoven.net/prod_gameanvil/images/management_machine_setup_none_ja.png)



## インスタンス

マシン管理を利用して、登録したマシンで起動するインスタンスを管理します。

インスタンスは複数の[ノード](server-2-basic)で構成され、ゲームの環境に応じてインスタンスを自由に構成できます。

### リスト

登録されたインスタンスのリストを確認できます。

インスタンスの基本的な情報と構成されたノード情報が表示されます。

![ManagementInstance-0](https://static.toastoven.net/prod_gameanvil/images/management_instance_list_ja.png)

ノードタイプごとにフィルタ機能とインスタンス名、ホスト名検索機能を提供します。

フィルタと検索ワードはAND条件で動作します。

![ManagementInstance-1](https://static.toastoven.net/prod_gameanvil/images/management_instance_filter_ja.png)

インスタンスに構成されたノードの詳細設定は**確認**ボタンを押して確認できます。

上部にあるタブからノードごとに確認できます。

![ManagementInstance-2](https://static.toastoven.net/prod_gameanvil/images/management_instance_config_popup_ja.png)

### 登録

インスタンス登録画面です。

インスタンスの基本情報を入力し、任意の構成のノードを設定します。

ノード設定の場合、テンプレート形式でのみ入力が可能で、テンプレートの詳細な説明は**インスタンス > 設定**ガイドで確認できます。

インスタンスの基本情報を入力する登録画面の最上部です。

各項目の入力値はリアルタイムで確認して、形式に合っていない値が入力された場合は案内メッセージを表示します。

**サーバービルドアップロードパス**と**マシン**の場合、一度登録した後は修正できない点に注意する必要があります。

![ManagementInstance-3](https://static.toastoven.net/prod_gameanvil/images/management_instance_3_ja_old.png)

インスタンスの基本情報(インスタンス名、サーバービルドアップロードパス)を入力すると、インスタンスを配布するマシンを選択できます。

マシン管理で登録したマシンのリストがポップアップに表示されます。

同じ構成のインスタンスを複数のマシンを選択して登録できます。

![ManagementInstance-12](https://static.toastoven.net/prod_gameanvil/images/management_instance_12_ja_old.png)

基本情報に続き、各ノードの設定を行います。

ノードタイプのうち、**COMMON、VM_OPTION**は必ず設定する必要があり、**GATEWAY、GAME、SUPPORT、MATCH**のうち1つ以上のノードを必ず設定する必要があります。

ノードの設定はテンプレートからのみ行うことができます。

![ManagementInstance-4](https://static.toastoven.net/prod_gameanvil/images/management_instance_4_ja_old.png)

先に登録されたテンプレートを選択するか、設定選択項目から**設定テンプレート追加**を行って新しいテンプレートを追加して指定できます。

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_14_ja_old.png)

ノード設定を行ったら、**COMMON、GATEWAY、SUPPORT**のPort情報を入力する必要があります。

**GATEWAY、SUPPORT**ノードは設定に応じてPortの入力を求めます。

Portは**18000～20000**の間の値が使用され、同じマシン内で重複しないようにする必要があります。

![ManagementInstance-5](http://static.toastoven.net/prod_gameanvil/images/management_instance_5_ja_old.png)

**GATEWAY**ノードが設定された画面です。

**TCP/WEB SOCKET**の使用状況に応じてPortの入力が動的に構成されます。

![ManagementInstance-6](https://static.toastoven.net/prod_gameanvil/images/management_instance_6_ja_old.png)

**SUPPORT**ノードが設定された画面です。

設定された**サービス数**に応じてPortの入力が動的に構成されます。

![ManagementInstance-7](https://static.toastoven.net/prod_gameanvil/images/management_instance_7_ja_old.png)

**GATEWAY、SUPPORT**設定に応じて構成されたPort入力領域です。

構成されたPort入力を終えた後、**重複確認**を行い、先に登録されたインスタンスとのPort重複チェックを行う必要があります。

![ManagementInstance-8](https://static.toastoven.net/prod_gameanvil/images/management_instance_8_ja_old.png)

重複したPortが入力されている場合、**重複確認**を行うと、案内ポップアップで重複したPort情報を確認できます。

この情報を元にPortを再入力した後、重複していない入力値が全て入力されるまで**重複確認**プロセスを繰り返す必要があります。

![ManagementInstance-9](https://static.toastoven.net/prod_gameanvil/images/management_instance_9_ja_old.png)

Port重複確認が正常に完了したら、インスタンス登録画面内のすべての入力フィールドが**無効**状態になり、最終的に登録ができる状態になります。

保存する前に一部の設定を変更するには**設定を再入力**ボタンを押して**有効**状態に切り替えた後、変更できます。

**有効**状態になったら、再び**重複確認**プロセスを踏むと登録可能な状態になります。

![ManagementInstance-11](https://static.toastoven.net/prod_gameanvil/images/management_instance_11_ja_old.png)

![ManagementInstance-10](https://static.toastoven.net/prod_gameanvil/images/management_instance_10_ja_old.png)

上で説明した登録方法の他にもインスタンスを登録できる別の方法があります。

**設定インポート**機能です。

![ManagementInstance-13](https://static.toastoven.net/prod_gameanvil/images/management_instance_13_ja_old.png)

**設定インポート**は、既に登録されているインスタンスの設定をそのままコピーする機能です。

設定インポートポップアップを利用して、登録されたインスタンスのノード設定を確認できます。

インスタンスリストの中から1つを選択すると、基本設定を除くすべての設定が登録画面に入力されます。

同じ構成のインスタンスを追加登録する時に便利な機能です。

### 修正および削除

登録されたインスタンスの修正、削除を行うことができます。

インスタンスの修正、削除は、インスタンスの状態が**起動待機、停止、エラー**状態の時のみ可能です。

**インスタンスモニタリング**でインスタンスの現在の状態を確認、変更できます。

![ManagementInstance-15](https://static.toastoven.net/prod_gameanvil/images/management_instance_15_ja_old.png)

![ManagementInstance-16](https://static.toastoven.net/prod_gameanvil/images/management_instance_16_ja_old.png)

## 設定

インスタンスの各タイプのノードのテンプレートを管理します。

インスタンスの登録時、各ノードの設定は直接入力せず、テンプレート選択によってのみ設定できます。

よく使用する各ノードの設定を登録しておき、インスタンスの登録時に使用できます。

### リスト

登録されたテンプレートのリストを確認できます。

ノードタイプごとにソートされ、そのテンプレートを使用中のインスタンスの状況も確認できます。

テンプレートもインスタンスと同様に、ノードタイプ別フィルタ機能とテンプレート名検索機能を提供します。

フィルタと検索ワードはAND条件で動作します。

![ManagementTemplate-1](https://static.toastoven.net/prod_gameanvil/images/management_template_1_ja_old.png)

![ManagementTemplate-2](https://static.toastoven.net/prod_gameanvil/images/management_template_2_ja_old.png)

### 登録

テンプレート登録画面です。

インスタンスの各ノードに適用する設定をテンプレートに登録します。

インスタンス登録画面では、各ノードの設定を直接入力できず、テンプレートでのみ設定が可能なため、必要な設定を事前に登録する必要があります。

該当商品が有効になる時、基本的に使用可能な**STANDARD**テンプレートを提供するため、これを利用してインスタンスの登録を行うことができます。

ゲーム特性に合わせてテンプレートを登録して活用してください。

![ManagementTemplate-3](https://static.toastoven.net/prod_gameanvil/images/management_template_3_ja_old.png)

テンプレートは各ノードタイプごとに区別され、入力項目ごとに基本値とMin、Max値制限が適用された項目があります。

**設定値の初期化**機能は、現在選択中のタイプの全ての設定を基本値に戻します。

Min、Max範囲を超える設定値が入力された場合、該当のフィールドがフォーカスアウトした時、フィールドに設定されたMin、Max値に自動変更されます。

![ManagementTemplate-4](https://static.toastoven.net/prod_gameanvil/images/management_template_4_ja_old.png)


### 修正および削除

登録されたテンプレートの修正、削除を行うことができます。

テンプレートの修正、削除は、そのテンプレートが適用されたすべてのインスタンスの状態が**起動待機、停止、エラー**状態の時のみ可能です。

テンプレートを使用中のインスタンスは、**設定リスト**からポップアップで確認できます。まt**インスタンスモニタリング**からインスタンスの現在の状態を確認、変更できます。

![ManagementTemplate-5](https://static.toastoven.net/prod_gameanvil/images/management_template_5_ja_old.png)

そしてインスタンスに適用されたテンプレートは、インスタンスの状態と関係なく削除ができません。

該当インスタンスでテンプレートを変更するか、インスタンスを削除した後、テンプレートを削除できます。

![ManagementTemplate-6](https://static.toastoven.net/prod_gameanvil/images/management_template_6_ja_old.png)



## 配布ファイル

GameAnvilインスタンスに配布されるプロセスファイルを管理するメニューです。

![ManagementDeployFile-1](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_ja.png)

### ファイルのアップロード

ファイルアップロードメニューから配布ファイルをアップロードできます。一度に1個のファイルのみアップロードが可能です。

拡張子が.jarのファイルのみアップロードをサポートし、ファイルサイズは最大1GBまで許容されます。

アップロードボタンを押すと、ファイルのアップロードが実行され、インスタンスモニタリングメニューで**配布する**機能を実行すると配布が完了します。

![ManagementDeployFile-2](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_upload_ja.png)

### 配布履歴

配布ファイルリストから配布履歴確認を押すと、その配布ファイルの配布履歴が下に表示されます。

配布が行われたインスタンス、マシン、配布日時などの情報を確認できます。 


![ManagementDeployFile-3](https://static.toastoven.net/prod_gameanvil/images/management_deployfile_history_ja.png)
