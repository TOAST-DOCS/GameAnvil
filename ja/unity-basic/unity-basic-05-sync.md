## Game > GameAnvil > Unity 基礎開発ガイド > 同期

## 同期

GameAnvilManagerでは、ゲームオブジェクトの生成/破棄、Transform、Animation、Rigidbody2D、Rigidbody属性を簡単に同期できる機能を提供します。

同期するゲームオブジェクトにそれぞれSync、TransformSync、AnimatorSync、Rigidbody2DSync、RigidbodySyncコンポーネントを付けるだけで、該当属性が同期されます。

同期コンポーネントを使用するゲームオブジェクトには、Syncコンポーネントも必ず一緒に追加する必要があります。各同期コンポーネントを追加した際にSyncコンポーネントがない場合、自動的に追加されます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/01-component.png)

あるクライアントで特定の属性を変更すると同期リクエストパケットが送信され、サーバーでこれを同じルームに属する全てのユーザーに共有し、他のクライアントで同期できるようにします。

## SyncController

同期機能を使用したいシーンにはSyncControllerが存在する必要があります。同期ゲームオブジェクトを生成及び破棄するInstantiate()、Destroy()メソッドを含み、同期機能の動作において核心的な役割を果たします。

### SyncController 生成

Unity Hierarchyウィンドウでマウスの右ボタンをクリックした後、**GameAnvil > SyncController**を選択してすぐに生成できます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/02-add-sync-controller.png)

または空のGameObjectを生成し、SyncControllerコンポーネントを追加することもできます。
SyncControllerには次のようなオプションがあります。

| オプション                | 説明                                                                                                                                                    |
|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| Use Synchronize Log | 同期関連ログを出力するかどうか                                                                                                                                                                                                                                              |
| Lazy Loading        | ルームに入室した直後、自動的にすぐに既存データを同期するかどうか <br/>もしこのオプションをfalseに設定する場合、既存データを同期したい時点で直接SyncController::InstantiateSyncObject()を呼び出す。 <br/>(デフォルト値: true) |

## ゲームオブジェクト生成/破棄同期、Sync

ゲームオブジェクトの生成/破棄を同期するゲームオブジェクトにSyncコンポーネントを追加すると、同じルームにいるユーザー同士が生成したゲームオブジェクトの生成/破棄が同期されます。
コンポーネントを追加するゲームオブジェクトを選択した後、メニューから**Component > GameAnvil > GameAnvil Sync > Sync**を選択してコンポーネントとして追加できます。インスペクターウィンドウで**Add Component**ボタンを押し、Syncコンポーネントを探して追加することもできます。

### Sync Id

ユーザーごとに固有の同期キーが付与され、ゲームオブジェクトごとにオブジェクトIDが付与されます。この2つを組み合わせて生成した同期IDで全てのユーザー別同期ゲームオブジェクトが区別されます。

Sync.SyncIdで同期ゲームオブジェクトの同期IDを取得できます。

### Create Option

同期ゲームオブジェクトが生成された方式を示します。クライアントで直接生成したゲームオブジェクトの場合はLOCALと表示され、他のクライアントで生成したゲームオブジェクトが生成同期により自身のシーンでも作成される場合にはREMOTEと表示されます。

### ゲームオブジェクト生成同期

Syncコンポーネントを追加したゲームオブジェクトをprefabにした後、UnityのAssets/Resourcesフォルダ配下に保存します。ルームに入室した後、SyncControllerのInstantiate()を通じて希望する時点で該当prefabを生成します。

ルームに入室した状態でのみ、生成/破棄が同期されるゲームオブジェクトを生成できます。

```c#
public void Instanticate()
{
    SyncController.Instance.Instantiate("prefabName", Vector3.zero, Quaternion.identity);
}
```

### ルームに途中参加した場合のゲームオブジェクト生成同期

プレイ中のルームに新しいユーザーが入室した場合、入室と同時にルームでプレイ中だった他のユーザーの同期データを受信することになります。このように受信された同期データを利用して、他のユーザーのゲームオブジェクトを自動的に生成して同期します。

しかし、ルームに入室した直後にすぐゲームオブジェクトを生成したくない場合があります。このような場合にはSyncControllerのLazy Loadingオプションをfalseに設定し、希望する時点でSyncController.InstantiateSyncObject()を呼び出して他のユーザーのゲームオブジェクトを生成できます。
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/03-lazy-loading.png)


```c#
public void InstantiateSyncObject()
{
    SyncController.Instance.InstantiateSyncObject();
}
```

### ゲームオブジェクト破棄同期

Syncコンポーネントが付いたゲームオブジェクトが削除されると、自動的に同期処理が行われ、他のユーザーのシーンでも該当ゲームオブジェクトが消えます。

ユーザーがルームに入室して同期ゲームオブジェクトを生成した後、シーンを移動するとゲームオブジェクトが全て破棄され、これを同期しながらサーバーに保存されたデータも削除されます。

したがって、他のシーンへ移動してから元のシーンに戻った時、生成していた同期ゲームオブジェクトがなくなっている可能性があります。そして一方のクライアントでシーン移動を行いながら同期ゲームオブジェクトが全て破棄されると、破棄同期により他のクライアントでもゲームオブジェクトが破棄される可能性があります。これに留意してシーン移動を行う必要があります。これは今後改善される予定です。

## Transform同期、TransformSync

Transformの同期を行いたいゲームオブジェクトにTransformSyncコンポーネントを追加すると、同期ゲームオブジェクトのTransformが同期されます。
コンポーネントを追加するゲームオブジェクトを選択した後、メニューから**Component > GameAnvil > GameAnvil Sync > TransformSync**を選択してコンポーネントとして追加できます。インスペクターウィンドウで**Add Component**ボタンを押し、TransformSyncコンポーネントを探して追加することもできます。
このゲームオブジェクトをprefabにした後、UnityのAssets/Resourcesフォルダ配下に保存し、ルームに入室してSyncControllerのInstantiate()を通じて該当prefabを生成した後、Transformを変化させると、他のクライアントでも全て変化したTransformに同期されることが確認できます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/04-transform-sync.gif)

### Transform同期オプション

Transformのうち同期するプロパティをオプションによって選択できます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/05-transform-sync-option.png)

| オプション                 | 説明                                                                                                              |
|----------------------|-------------------------------------------------------------------------------------------------------------------|
| Synchronize Position | Positionプロパティを同期するかどうかを設定します。trueならPosition値を同期し、falseならPosition値を同期しません。                                |
| Synchronize Rotation | Rotationプロパティを同期するかどうかを設定します。trueならRotation値を同期し、falseならRotation値を同期しません。                                |
| Synchronize Scale    | Scaleプロパティを同期するかどうかを設定します。trueならScale値を同期し、falseならScale値を同期しません。                                         |
| Use Local            | localPosition及びlocalRotationを使用すべきかどうかを設定します。Scaleはこの設定を無視し、常にlocalScaleを使用してlossyScale関連の問題を防ぎます。 |

## Animation同期、AnimatorSync

Animationの同期を行いたいゲームオブジェクトにAnimatorSyncコンポーネントを付け、Animatorのパラメータ値を変更してAnimation Stateを変更させると、該当変更事項が他のクライアントでも同期されます。
コンポーネントを追加するゲームオブジェクトを選択した後、メニューから**Component > GameAnvil > GameAnvil Sync > AnimatorSync**を選択してコンポーネントとして追加できます。インスペクターウィンドウで**Add Component**ボタンを押し、AnimatorSyncコンポーネントを探して追加することもできます。
このゲームオブジェクトをprefabにした後、UnityのAssets/Resourcesフォルダ配下に保存し、ルームに入室してSyncControllerのInstantiate()を通じて該当prefabを生成した後、Animationを変化させると、他のクライアントでも全て変化したAnimationに同期されることが確認できます。
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/06-animator-sync.gif)

## Rigidbody2D同期、Rigidbody2DSync

Rigidbody2Dの同期を行いたいゲームオブジェクトにRigidbody2DSyncコンポーネントを付けると、同期ゲームオブジェクトのRigidbody2Dが同期されます。
コンポーネントを追加するゲームオブジェクトを選択した後、メニューから**Component > GameAnvil > GameAnvil Sync > Rigidbody2DSync**を選択してコンポーネントとして追加できます。インスペクターウィンドウで**Add Component**ボタンを押し、Rigidbody2DSyncコンポーネントを探して追加することもできます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/07-rigidbody2d-sync.gif)

Rigidbody2DSyncコンポーネントを付けたゲームオブジェクトをprefabにした後、UnityのAssets/Resourcesフォルダ配下に保存し、ルームに入室してSyncControllerのInstantiate()を通じて該当prefabを生成した後、Rigidbody2Dを変化させると、他のクライアントでも全て変化したRigidbody2Dに同期されることが確認できます。

### Rigidbody2D同期オプション

Rigidbody2Dのうち同期するプロパティをオプションによって選択できます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/08-rigidbody2d-sync-option.png)

| オプション                              | 説明                                                                                                                                     |
|-----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| Synchronize Velocity              | Velocityプロパティを同期するかどうかを設定します。trueならVelocity値を同期し、falseならVelocity値を同期しません。                                                                                                                                                                 |
| Synchronize Angular Velocity      | Angular Velocityプロパティを同期するかどうかを設定します。trueならAngular Velocity値を同期し、falseならAngular Velocity値を同期しません。                                                                                                                                         |
| Teleport Enabled                  | Teleportを許可するかどうかを設定します。                                                                                                                                                                                                                                 |
| Teleport if distance greater than | 距離が設定した基準以上に差が出ると、Rigidbody2DのPositionに同期する位置値を適用した後、Velocity値を利用して同期します。<br/>Teleport Enabledがチェックされている場合にのみ表示されます。                                                                                                                    |
| Teleport if angle greater than    | 角度が設定した基準以上に差が出ると、Rigidbody2DのRotationに同期する角度値を適用した後、Angular Velocity値を利用して同期します。<br/>Teleport Enabledがチェックされている場合にのみ表示されます。                                                                                                            |

## Rigidbody同期、RigidbodySync

Rigidbodyの同期を行いたいゲームオブジェクトにRigidbodySyncコンポーネントを追加すると、同期ゲームオブジェクトのRigidbodyが同期されます。
コンポーネントを追加するゲームオブジェクトを選択した後、メニューから**Component > GameAnvil > GameAnvil Sync > RigidbodySync**を選択してコンポーネントとして追加できます。インスペクターウィンドウで**Add Component**ボタンを押し、RigidbodySyncコンポーネントを探して追加することもできます。

RigidbodySyncコンポーネントを付けたゲームオブジェクトをprefabにした後、UnityのAssets/Resourcesフォルダ配下に保存し、ルームに入室してSyncControllerのInstantiate()を通じて該当prefabを生成した後、Rigidbodyを変化させると、他のクライアントでも全て変化したRigidbodyに同期されることが確認できます。

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/09-rigidbody-sync.gif)

### Rigidbody同期オプション

Rigidbodyのうち同期するプロパティをオプションによって選択できます。
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/10-rigidbody-sync-option.png)

| オプション                              | 説明                                                                                                                                   |
|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| Synchronize Velocity              | Velocityプロパティを同期するかどうかを設定します。trueならVelocity値を同期し、falseならVelocity値を同期しません。                                                                                                                                                                |
| Synchronize Angular Velocity      | Angular Velocityプロパティを同期するかどうかを設定します。trueならAngular Velocity値を同期し、falseならAngular Velocity値を同期しません。                                                                                                                                        |
| Teleport Enabled                  | Teleportを許可するかどうかを設定します。                                                                                                                                                                                                                                |
| Teleport if distance greater than | 距離が設定した基準以上に差が出ると、RigidbodyのPositionに同期する位置値を適用した後、Velocity値を利用して同期します。<br/>Teleport Enabledがチェックされている場合にのみ表示されます。                                                                                                                     |
| Teleport if angle greater than    | 角度が設定した基準以上に差が出ると、RigidbodyのRotationに同期する角度値を適用した後、Angular Velocity値を利用して同期します。<br/>Teleport Enabledがチェックされている場合にのみ表示されます。                                                                                                            |

## ユーザー定義値同期

int、float、bool、stringタイプのユーザー定義値を同期する機能を提供します。

### ユーザー定義値の追加または変更

SetCustomProperty\<T\>()でユーザー定義値を追加または変更できます。サーバーでは該当ユーザー定義値データを保存し、同じルームの全てのユーザーにブロードキャストして同期できるようにします。
SetCustomProperty\<T\>()は次のように1つの型パラメータと2つのパラメータを持っています。

| タイプ    | 名前  | 説明                                            |
|---------|-------|-------------------------------------------------|
| 型パラメータ | T     | 保存するユーザー定義値のタイプ。int、float、bool、stringのいずれか |
| String  | key   | ユーザー定義値を区別するためのキー                             |
| T       | value | ユーザー定義値                                      |

```c#
public void SetCustomProperty()
{
    SyncController.Instance.SetCustomProperty("IntValue", 1);
    SyncController.Instance.SetCustomProperty("FloatValue", 1.0f);
    SyncController.Instance.SetCustomProperty("BoolValue", false);
    SyncController.Instance.SetCustomProperty("StringValue", "Value");
}
```

### ユーザー定義値が最新状態か確認後に変更

SetCustomPropertyCas\<T\>()を呼び出すと、クライアントに保存されていたユーザー定義値とサーバーに保存されているユーザー定義値を比較し、同じ場合にのみユーザー定義値データを保存し、同じルームの全てのユーザーにブロードキャストして同期できるようにします。もしクライアントとサーバーにそれぞれ保存されていたユーザー定義値が異なる場合、該当リクエストを無視します。
SetCustomPropertyCas\<T\>()は次のように1つの型パラメータと2つのパラメータを持っています。

| タイプ    | 名前  | 説明                                            |
|---------|-------|-------------------------------------------------|
| 型パラメータ | T     | 保存するユーザー定義値のタイプ。int、float、bool、stringのいずれか |
| String  | key   | ユーザー定義値を区別するためのキー                             |
| T       | value | ユーザー定義値                                      |

```c#
public void SetCustomPropertyCas()
{
    SyncController.Instance.SetCustomPropertyCas("IntValue", 1);
    SyncController.Instance.SetCustomPropertyCas("FloatValue", 1.0f);
    SyncController.Instance.SetCustomPropertyCas("BoolValue", false);
    SyncController.Instance.SetCustomPropertyCas("StringValue", "Value");
}
```

### ユーザー定義値の照会

GetCustomProperty\<T\>()でユーザー定義値を照会できます。
SetCustomPropertyCas\<T\>()は次のように1つの型パラメータと1つのパラメータを持っています。

| タイプ    | 名前 | 説明                                            |
|---------|-----|-------------------------------------------------|
| 型パラメータ | T   | 照会するユーザー定義値のタイプ。int、float、bool、stringのいずれか |
| String  | key | ユーザー定義値を区別するためのキー                             |

```c#
public void GetCustomProperty()
{
    int intValue = SyncController.Instance.GetCustomProperty<int>("IntValue");
    float floatValue = SyncController.Instance.GetCustomProperty<float>("FloatValue");
    bool boolValue = SyncController.Instance.GetCustomProperty<bool>("BoolValue");
    string stringValue = SyncController.Instance.GetCustomProperty<string>("StringValue");        
}
```

### ユーザー定義値の削除

RemoveCustomProperty\<T\>()でユーザー定義値を削除できます。サーバーでも保存していたユーザー定義値データを削除します。
SetCustomPropertyCas\<T\>()は次のように1つのパラメータを持っています。

| タイプ   | 名前 | 説明                |
|--------|-----|---------------------|
| String | key | ユーザー定義値を区別するためのキー |

```c#
public void RemoveCustomProperty()
{
    SyncController.Instance.RemoveCustomProperty("IntValue");
    SyncController.Instance.RemoveCustomProperty("FloatValue");
    SyncController.Instance.RemoveCustomProperty("BoolValue");
    SyncController.Instance.RemoveCustomProperty("StringValue");
}
```
