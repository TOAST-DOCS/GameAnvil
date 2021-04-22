## Game > GameAnvil > サーバー開発ガイド > 無停止パッチ方法

この文書は、無停止パッチを進行する方法を説明します。また無停止パッチの過程で発生しうる問題についても整理します。

### Note

- 無停止メンテナンスを使用する前に、必ずアルファで先に進行して問題がないことを確認してください。
- このガイド文書を必ず読み、すべての内容を理解してから使用してください。



## 1. NonStopPatchを使用してはいけない状況

- パッチするサーバーのファイルと既に配布されているサーバーのファイルのGameAnvilバージョンが異なる場合
- パケットが修正または追加されている場合
- Transfer間で渡されるData Classが変更された場合



## 2. NonStopPatchを使用するために必要な作業

### 2-1. Match機能を使用中の場合

- 旧バージョンのサーバーと、新バージョンのサーバーのバージョン情報を確認するロジックを入れる必要があります。

### 2-2. TransferData

- 旧バージョンのサーバーと、新バージョンのサーバーのtransferDataが異なる場合、新バージョンのGameUser/GameRoomのonTranferIn関数から削除するか、追加されたtransferDataの処理をする必要があります。
- protoオブジェクトにデータを渡す場合、kryoにそのclassを登録する必要があります。



## 3. NonStopPatch使用時に知っておくべきこと

- NonStopPatch Report情報提供
  - Rest API
    - http://127.0.0.1:25150/managementEndPoint/non-stop-patch/report-page
    - NonStopPatch進行事項および結果提供
  - File
    - log/nonStopPatch/report-2020-08-10 205510.log
    - NonStopPatch終了時、ReportをJson形式でFile保存
- NonStopPatch強制終了機能
  - NonStopPatchが進行中の場合、他のNodeの状態を変更できません。
  - NonStopPatch進行中に問題が発生して、終了ができない場合があります。
  - NonStopPatch Reportとログを利用して問題点を把握し、強制的にNonStopPatchを終了するかどうかを決定します。
  -
    - NonStopPatch強制終了ボタンを押すと、NonStopPatchが強制的に終了します。
  - PauseしたNodeを処理します。

## 4. NonStopPatch開始

- NonStopPatchの進行はAdminで行います。

### 4-1. NonStopPatchを進行するNode選択

- 出発地を選択
- ![NonStopPatch-2.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-2.png)
- 到着地を選択
- ![NonStopPatch-3.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-3.png)

### 4-2. NonStopPatch Report Pageを開く

- http://10.162.3.241:25150/non-stop-patch/report-page?refreshTime=1000

### 4-3. NonStopPatchボタンをクリック

- NonStopPatchボタンをクリックしてNonStopPatchを進行
- ![NonStopPatch-4.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-4.png)
- ![NonStopPatch-5.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-5.png)
- 出発地NodeはNonStopPatch Src状態に変わる
  - NonStopPatchが終了すると、Pause状態になる
- 到着地NodeはNonStopPatch Dst状態に変わる
  - NonStopPatchが終了すると、Ready状態になる

### 4-4. NonStopPatch結果確認

- NonStopPatch Report確認
- ![NonStopPatch-6.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-6.png)
- 現在GameNodeヘルスチェック
- ![NonStopPatch-7.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-7.png)

### 4-5. サーバー終了

- Pause中のサーバーを終了
- ![NonStopPatch-8.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-8.png)
- 終了したサーバーを確認
- ![NonStopPatch-9.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-9.png)

### 4-6. サーバーパッチ

- 配布されたファイルを消去し、パッチするサーバーのファイルを配布
  - 直接サーバーのファイル名を指定する場合は消さなくてもよい
- ![NonStopPatch-10.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-10.png)

### 4-7. サーバー再起動

- サーバー再起動
- ![NonStopPatch-11.png](http://static.toastoven.net/prod_gameanvil/images/NonStopPatch-11.png)

### 4-8. 反対側のNodeもNonStopPatch進行

- 上と同じように反対側のNodeも進行

### 4-9. サーバー終了

- 上と同じように反対側のNodeも進行

### 4-10. サーバーパッチ

- 上と同じように反対側のNodeも進行

### 4-11. サーバー再起動

- 上と同じように反対側のNodeも進行

### 4-12. NonStopPatchが成功したかを確認

- サーバーパッチ後、問題がないことを確認

## 5. NonStopPatch時の関数呼び出し順序

### 5-1. BaseGameRoom

- 作成時に呼び出されるCallback関数
  - 状況に合わせてCallback関数の内容を入力します。
  - GameRoom作成
    - onInit -> onCreate
  - NonStopPatch時、GameRoom作成
    - onInit -> onTransferIn
- NonStopPatch終了後に呼び出されるCallback関数
  - NonStopPatch終了後、GameNodeのStateが変更され、GameRoomでCallback関数が呼び出されます。
  - 出発地Node
    - onPause
    - 出発地NodeはNonStopPatchが終了し、pause状態になるため、
  - 到着地Node
    - X

### 5-2. BaseUserRoom

- 作成時に呼び出されるCallback関数
  - 状況に合わせてCallback関数の内容を入力する必要があります。
  - GameUser作成
    - onLogin
  - NonStopPatch時、GameUser作成
    - onTransferIn
- NonStopPatch終了後に呼び出されるCallback関数
  - NonStopPatch終了後、GameNodeのStateが変更され、GameUserでCallback関数が呼び出されます。
  - 出発地Node
    - onPause
    - 出発地NodeはNonStopPatchが終了し、pause状態になるため、
  - 到着地Node
    - X

### 5-3. BaseGameNode

- NonStopPatch時に呼び出されるCallback関数
- 出発地Nodeの場合

```
// NonStopPatchが始まる場合に呼び出し
@Override
public boolean onNonStopPatchSrcStart() throws SuspendExecution {
    return true;
}
// NonStopPatchが終了する場合に呼び出し
@Override
public boolean onNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}
// NonStopPatch終了チェック関数
@Override
public boolean canNonStopPatchSrcEnd() throws SuspendExecution {
    return true;
}
```

- 到着地Nodeの場合

```
// NonStopPatchが始まる場合に呼び出し
@Override
public boolean onNonStopPatchDstStart() throws SuspendExecution {
    return true;
}
// NonStopPatchが終了する場合に呼び出し
@Override
public boolean onNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
// NonStopPatch終了チェック関数
@Override
public boolean canNonStopPatchDstEnd() throws SuspendExecution {
    return true;
}
```
