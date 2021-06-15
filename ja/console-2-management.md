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

登録されているすべてのインスタンスの状態が動作中ではない場合にのみ、マシンの設定を初期化できます。

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

![ManagementInstance-3](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_1_ja.png)

インスタンスの基本情報(インスタンス名、サーバービルドアップロードパス)を入力すると、インスタンスを配布するマシンを選択できます。

マシン管理で登録したマシンのリストがポップアップに表示されます。

同じ構成のインスタンスを複数のマシンを選択して登録できます。

![ManagementInstance-12](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_2_ja.png)

基本情報に続き、各ノードの設定を行います。

ノードタイプのうち、**COMMON、VM_OPTION**は必ず設定する必要があり、**GATEWAY、GAME、SUPPORT、MATCH**のうち1つ以上のノードを必ず設定する必要があります。

ノードの設定は、**設定の読み込み**を利用して登録されているテンプレートを選択する方式でのみ行うことができます。

![ManagementInstance-4](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_3_ja.png)

設定の読み込みポップアップから登録されているテンプレートの各ノード設定を確認できます。

テンプレート一覧から1つを選択すると、基本設定を除くすべての設定が登録画面に入力されます。

同じ構成のインスタンスを登録または管理する時に役立つように提供している機能です。

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_4_ja.png)

**設定の読み込み**を行ったら、**COMMON、GATEWAY、SUPPORT**のPort情報を入力する必要があります。

**GATEWAY、SUPPORT**ノードは設定に応じてPortの入力を要求します。

**GATEWAY**ノードが設定された画面です。

**TCP/WEB SOCKET**の使用状況に応じてPortの入力が動的に構成されます。

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_5_ja.png)

**SUPPORT**ノードが設定された画面です。

設定された**サービス数**に応じてPortの入力が動的に構成されます。

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_6_ja.png)

**GATEWAY、SUPPORT**設定に応じて構成されたPort入力領域です。

Portは**18000～20000**の間の値を使用し、同じマシン内で重複しないようにする必要があります。

構成されたPortの入力を終えた後、**ポート重複確認**を行い、登録されているインスタンスとのPort重複チェックを行う必要があります。  

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_7_ja.png)

**ポート重複確認**により重複したPortが判明した場合は、案内ポップアップで重複したPort情報を確認できます。

この情報を元にPortを再入力した後、全ての入力値が重複しなくなるまで**ポート重複確認**プロセスを繰り返す必要があります。

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_8_ja.png)

**ポート重複確認**が正常に完了したら、インスタンス登録画面内のすべての入力フィールドが**無効**状態になり、最終的に登録ができる状態になります。

保存する前に一部の設定を変更するには、**設定を再入力**ボタンを押して**有効**状態に切り替えた後、変更できます。

**有効**状態になったら、再び**ポート重複確認**プロセスを踏むと登録可能な状態になります。

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_9_ja.png)

![ManagementInstance-14](https://static.toastoven.net/prod_gameanvil/images/management_instance_register_10_ja.png)

### 修正および削除

登録されたインスタンスの修正、削除を行うことができます。

インスタンスの修正、削除は、インスタンスの状態が**起動待機、停止、エラー**状態の時のみ可能です。

**モニタリング > インスタンスモニタリング**でインスタンスの現在状態を確認、変更できます。

![ManagementInstance-15](https://static.toastoven.net/prod_gameanvil/images/management_instance_modify_delete_ja.png)

![ManagementInstance-16](https://static.toastoven.net/prod_gameanvil/images/management_instance_start_ja.png)

インスタンス修正で**サーバービルドアップロードパス**と**マシン**設定は変更できません。

インスタンスのノード設定は、登録方法と同じように**設定の読み込み**を行って変更できます。

![ManagementInstance-17](https://static.toastoven.net/prod_gameanvil/images/management_instance_modify_ja.png)

## 設定

インスタンスの各タイプのノードのテンプレートを管理します。

インスタンスの登録時、各ノードの設定は直接入力せず、テンプレート選択によってのみ設定できます。

よく使用する各ノードの設定を登録しておき、インスタンスの登録時に使用できます。

### リスト

登録されたテンプレートのリストを確認できます。

テンプレート名でソートされ、そのテンプレートを使用中のインスタンスの状況も確認できます。

テンプレート名検索機能を提供します。

![ManagementTemplate-1](https://static.toastoven.net/prod_gameanvil/images/management_template_list_ja.png)

![ManagementTemplate-2](https://static.toastoven.net/prod_gameanvil/images/management_template_list_popup_ja.png)

### 登録

テンプレート登録画面です。

インスタンスの各ノードに適用する設定をテンプレートに登録します。

インスタンス登録画面では、各ノードの設定を直接入力できず、テンプレートでのみ設定が可能なため、必要な設定を事前に登録する必要があります。

基本設定値を提供するため、これを利用してテンプレートの登録を行うことができます。

ゲーム特性に合わせてテンプレートを登録して活用してください。

![ManagementTemplate-3](https://static.toastoven.net/prod_gameanvil/images/management_template_3_ja_old.png)

各ノードの設定は、直接設定値を入力して登録することができ、**設定の読み込み**を行って登録されているテンプレート設定を読み込んで登録することもできます。

類似したテンプレートを登録したい時、**設定の読み込み**で設定を読み込んだ後、変更が必要な一部設定値のみを直接入力して登録してください。

![ManagementTemplate-3](https://static.toastoven.net/prod_gameanvil/images/management_template_register_1_ja.png)

ノードタイプのうち、**COMMON、VM_OPTION**は、必ず設定する必要があり、**GATEWAY、GAME、SUPPORT、MATCH**の中から1つ以上のノードを必ず設定する必要があります。

**COMMON**ノードの設定値です。

![ManagementTemplate-4](https://static.toastoven.net/prod_gameanvil/images/management_template_register_2_ja.png)

**VM OPTION**ノードの設定値です。

![ManagementTemplate-5](https://static.toastoven.net/prod_gameanvil/images/management_template_register_3_ja.png)

**GATEWAY**ノードの設定値です。

TCP/WEB SOCKET、SSLの使用状況に応じて設定値が異なります。

![ManagementTemplate-6](https://static.toastoven.net/prod_gameanvil/images/management_template_register_4_ja.png)

**GAME**ノードの設定値です。

![ManagementTemplate-7](https://static.toastoven.net/prod_gameanvil/images/management_template_register_5_ja.png)

GAMEノードは**サービス**を選択して設定する必要があります。

**サービス選択**ポップアップから最大99個のサービスを選択して設定できます。 

サービスは**管理 > インスタンス > サービス**メニューで管理できます。 

![ManagementTemplate-8](https://static.toastoven.net/prod_gameanvil/images/management_template_register_6_ja.png)

サービスを選択して確認したら、各サービスに必要な設定値を追加で入力します。

![ManagementTemplate-9](https://static.toastoven.net/prod_gameanvil/images/management_template_register_7_ja.png)  

SUPPORTノードも**サービス**を選択して設定する必要があります。

GAMEノードと同じように、**サービス選択**ポップアップで最大99個のサービスを選択した後、設定値を追加で入力します。

![ManagementTemplate-10](https://static.toastoven.net/prod_gameanvil/images/management_template_register_8_ja.png)

テンプレートの登録中に、あらかじめ登録できなかったサービスがある場合は、**サービス選択**ポップアップから登録できます。

**サービス登録**ボタンをクリックすると、登録ポップアップが表示されます。
 
ここでサービスID、サービス名を入力してサービスを登録できます。

登録のみ行うことができます。修正、削除は**管理 > インスタンス > サービス**メニューで行うことができます。

![ManagementTemplate-11](https://static.toastoven.net/prod_gameanvil/images/management_template_register_9_ja.png)

![ManagementTemplate-12](https://static.toastoven.net/prod_gameanvil/images/management_template_register_10_ja.png)

最後に**MATCH**ノードの設定値です。

![ManagementTemplate-13](https://static.toastoven.net/prod_gameanvil/images/management_template_register_11_ja.png)

テンプレートは入力項目ごとに基本値とMin/Max値制限が適用されている項目があります。

**設定値初期化**機能は、該当ノードの設定を基本値に戻します。

**GAME**、**SUPPORT**ノードは、**設定値初期化**機能を提供しません。

Min/Max範囲を超えた値が入力された場合、該当フィールドからフォーカスアウトした時、フィールドに設定に Min/Max値に自動変更されます。

### 修正および削除

登録されたテンプレートの修正、削除を行うことができます。

テンプレートの修正、削除は、そのテンプレートが適用されたすべてのインスタンスの状態が**起動待機、停止、エラー**状態の時のみ可能です。

テンプレートを使用中のインスタンスは、**設定リスト**からポップアップで確認できます。まt**インスタンスモニタリング**からインスタンスの現在の状態を確認、変更できます。

![ManagementTemplate-5](https://static.toastoven.net/prod_gameanvil/images/management_template_5_ja_old.png)

テンプレートの修正では、**設定の読み込み**機能を提供しないため、必要な設定値を直接修正する必要があります。

![ManagementTemplate-16](https://static.toastoven.net/prod_gameanvil/images/management_template_modify_ja.png)

そしてインスタンスに適用されたテンプレートは、インスタンスの状態と関係なく削除ができません。

該当インスタンスでテンプレートを変更するか、インスタンスを削除した後、テンプレートを削除できます。

![ManagementTemplate-17](https://static.toastoven.net/prod_gameanvil/images/management_template_delete_ja.png)

## サービス

サービスとは、**GameAnvil**で製作されたサーバーコードのBootstrapプロセスにおいて**GAME**、**SUPPORT**ノードで使用する識別子です。

### リスト

登録されたサービス一覧を確認できる画面です。

サービスタイプ(GAME、SUPPORTの順)でソートされ、その後、サービスIDでソートされます。

該当サービスが使用されているテンプレートがあるか、ある場合はどのテンプレートなのかを確認できます。

サービス名で検索する機能が提供されます。

![ManagementService-1](https://static.toastoven.net/prod_gameanvil/images/management_service_list_ja.png)

### 登録

**サービスID**は1～99の数値を入力します。重複した数値は使用できません。

**サービス名**はサーバーコードのBootstrapに使われた識別子と同じでなければならず、大文字/小文字を区別するため、誤字に注意してください。サービスIDと同様に重複した値は使用できません。

サービスの登録方法は2つあります。直接入力方式とファイルアップロード方式です。

**直接入力**は、以下のように直接入力をクリックして、値を直接入力する方式です。

![ManagementService-2](https://static.toastoven.net/prod_gameanvil/images/management_service_add_ja.png)

**ファイルアップロード**方式は、以下のようにファイルアップロードをクリックして、***テンプレートサンプルファイル***をクリックしてテンプレート様式に合わせてデータを入力した後、ファイル選択を行うか、ドラッグアンドドロップして複数のサービスをまとめて保存できます。

このとき注意する点は、ファイルをアップロードできるのは1つのサービスタイプだけのため、GAMEとSUPPORTを分けて保存することです。

![ManagementService-3](https://static.toastoven.net/prod_gameanvil/images/management_service_fileupload_1_ja.png)

保存を押した後、以下のような画面が表示されます。入力した値が正しいか確認した後、一番下の保存ボタンをもう一度押して、入力した値をサーバーに送ります。

![ManagementService-4](https://static.toastoven.net/prod_gameanvil/images/management_service_fileupload_2_ja.png)

### 修正および削除

**サービス名**を修正、削除する場合は、リストから該当サービスをクリックすると詳細画面表示に切り替わります。

修正は、該当サービスが登録されたテンプレートを利用して動作しているインスタンスがない時に行うことができます。

削除は、該当サービスが登録されたテンプレートがない時に行うことができます。

![ManagementService-5](https://static.toastoven.net/prod_gameanvil/images/management_service_detail_ja.png)

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
