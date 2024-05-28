## Game > GameAnvil > Unity基礎開発ガイド > 同期

## 同期

GameAnvilConnectorでは、ゲームオブジェクトの作成/破壊、Transform、Animation、Rigidbody2D、Rigidbodyプロパティを簡単に同期できる機能を提供します。

同期するゲームオブジェクトにそれぞれSync、TransformSync、AnimatorSync、Rigidbody2DSync、RigidbodySyncコンポーネントを付けるだけで、そのプロパティが同期されます。

同期コンポーネントを使用するゲームオブジェクトには、Syncコンポーネントも必ず一緒に追加する必要があります。各同期コンポーネントを追加する際、必要なコンポーネントがない場合は自動的に追加されます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/01-component.png)

あるクライアントで特定のプロパティを変更すると、同期リクエストパケットが送信され、サーバーでこれを同じルームに属する全てのユーザーにブロードキャストして他のクライアントで同期できるようにします。

## SyncController

同期機能を使用するシーンにSyncControllerが存在する必要があります。 SyncControllerは、同期ゲームオブジェクトを作成し、破壊するInstantiate(), Destroy()関数を含め、同期機能の動作に中核的な役割を果たします。

### SyncController作成

GameObjectを作成してSyncControllerコンポーネントを追加します。**Add Component > GameAnvil > SyncController**を選択してコンポーネントとして追加できます。

または、Unity Hierarchyで右クリックした後、**GameAnvil > SyncController**を選択して直接作成することもできます。

## ゲームオブジェクトの作成/破壊の同期、Sync

ゲームオブジェクトの作成/破壊を同期するゲームオブジェクトにSyncコンポーネントを追加すると、同じルームにいるユーザーがお互いに作成したゲームオブジェクトの作成/破壊が同期されます。
**Add Component > GameAnvil > GameAnvil Sync > Sync**を選択してコンポーネントとして追加できます。

### Sync Id

ユーザーごとに固有の同期キーが付与され、ゲームオブジェクトごとにオブジェクトIDが付与され、この2つを組み合わせて作成した同期IDですべてのユーザー別同期ゲームオブジェクトが区別されます。

Sync.SyncIdで同期ゲームオブジェクトの同期IDを取得できます。

### Create Option

同期ゲームオブジェクトの作成方法を指定します。クライアントで直接作成したゲームオブジェクトの場合はLOCALと表示し、他のクライアントで作成したゲームオブジェクトが作成同期によって自分のシーンでも作成される場合はREMOTEと表示します。

### ゲームオブジェクト作成同期

Syncコンポーネントを追加したゲームオブジェクトをprefabで作成し、UnityのAssets/Resourcesフォルダの下に保存します。ルームに入室した後、SyncControllerのInstantiate()を使って好きなタイミングで該当prefabを作成します。

ルームに入室した状態でのみ作成/破壊が同期されるゲームオブジェクトを作成できます。

```c#
/// <summary>
/// 同期ゲームオブジェクトを作成する。
/// </summary>
/// <param name="prefabName">作成するprefabの名前。該当prefabはAssets/Resourcesフォルダの下に存在する必要があります。</param>
/// <param name="v">prefabを作成するposition</param>
/// <param name="r">prefabのrotation</param>
/// <returns></returns>
public GameObject Instantiate(string prefabName, Vector3 v, Quaternion r);
```

### ルームに途中参加した場合、ゲームオブジェクト作成の同期

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/02-instantiate-sync-object-immediatly.png)

| オプション | 説明 |
| --- | --- |
| InstantiateSyncObjectImmediatly | ルームに入室した後、すぐに同期を行うかどうかを設定します。 trueの場合、ユーザーがルームに入室した後、すぐに同期を行い、falseの場合、直接同期関数を呼び出す必要があります。 |

ルームに途中参加した場合、ユーザーが入室時に呼び出されるコールバックでSyncController.roomSyncStart()が呼び出され、サーバーから受け取った既存のルームの同期データを利用してゲームオブジェクトを作成して同期します。

しかし、ルームに入った直後にすぐに同期をしたくない場合は、SyncControllerのInstantiateSyncObjectImmediatlyオプションをfalseに設定して、好きなタイミングでSyncController.InstantiateSyncObject()を呼び出すことができます。

### ゲームオブジェクトの破壊同期

Syncコンポーネントが付いたゲームオブジェクトが削除されると、onDestroy()関数で破壊同期が処理され、他のユーザーのシーンでもそのゲームオブジェクトが消えます。

現在の仕様では、ユーザーがルームに入室して同期ゲームオブジェクトを作成した後、シーンを移動すると、ゲームオブジェクトが全て破壊され、これを同期しながらサーバーに保存されたデータも削除されます。

したがって、別のシーンに移動して、再び元のシーンに戻った時、作成した同期ゲームオブジェクトがなくなっている可能性があります。そして、一方のクライアントでシーン移動を行い、同期ゲームオブジェクトがすべて破壊されると、破壊同期により、もう一方のクライアントでもゲームオブジェクトが破壊される可能性があります。この点に注意してシーン移動を行う必要があります。これは次の仕様で改善される予定です。

## Transform同期、 TransformSync

Transform同期をしたいゲームオブジェクトにTransformSyncコンポーネントを追加すると、同期ゲームオブジェクトのTransformが同期されます。
**Add Component > GameAnvil > GameAnvil Sync > TransformSync**を選択してコンポーネントとして追加できます。Syncコンポーネントが自動的に一緒に追加されます。

TransformSyncコンポーネントを付けたゲームオブジェクトをprefabで作成し、UnityのAssets/Resourcesフォルダの下に保存します。

ルームに入室してSyncControllerのInstantiate()を通じて該当prefabを作成した後、Transformを変化させた時、他のクライアントでも全て変化したTransformで同期されることが確認できます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/03-transform-sync.gif)

### Transform同期オプション

Transform中に同期するプロパティをオプションによって選択できます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/04-transform-sync-option.png)

| オプション | 説明 |
| --- | --- |
| Synchronize Position | Positionプロパティを同期するかどうかを設定します。 trueの場合、Position値を同期させ、falseの場合、Position値を同期しません。 |
| Synchronize Rotation | Rotationプロパティを同期するかどうかを設定します。 trueの場合、Rotation値を同期し、falseの場合、Rotation値を同期しません。 |
| Synchronize Scale | Scaleプロパティを同期するかどうかを設定します。 trueの場合、Scale値を同期し、falseの場合、Scale値を同期しません。 |
| Use Local | localPositionとlocalRotationを使用するかどうかを設定します。Scaleはこの設定を無視して常にlocalScaleを使用し、lossyScale関連の問題を防止します。 |

## Animation同期、 AnimatorSync

Animation同期をしたいゲームオブジェクトにAnimatorSyncコンポーネントを付けて、Animatorのパラメータ値を変更してAnimation Stateを変更すると、その変更が他のクライアントでも同期されます。
**Add Component > GameAnvil > GameAnvil Sync > AnimatorSync**を選択してコンポーネントとして追加できます。Sync、Animatorコンポーネントが自動的に一緒に追加されます。

AnimatorSyncコンポーネントを追加したゲームオブジェクトをprefabで作成し、UnityのAssets/Resourcesフォルダの下に保存します。

ルームに入室してSyncControllerのInstantiate()を通じて該当prefabを作成した後、Animationを変化させた時、他のクライアントでも全て変化したAnimationで同期されることが確認できます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/05-animator-sync.gif)

## Rigidbody2D同期、 Rigidbody2DSync

Rigidbody2D同期をしたいゲームオブジェクトにRigidbody2DSyncコンポーネントを付けると、同期ゲームオブジェクトのRigidbody2Dが同期されます。
**Add Component > GameAnvil > GameAnvil Sync > Rigidbody2DSync** を選択してコンポーネントとして追加できます。Sync、TransformSync、Rigidbody2Dコンポーネントが自動的に一緒に追加されます。

Rigidbody2DSyncコンポーネントを付けたゲームオブジェクトをprefabで作成し、UnityのAssets/Resourcesフォルダの下に保存します。

ルームに入室してSyncControllerのInstantiate()を通じて該当prefabを作成した後、Rigidbody2Dを変化させた時、他のクライアントでも全て変化したRigidbody2Dで同期されることが確認できます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/06-rigidbody2d-sync.gif)

### Rigidbody2D同期オプション

Rigidbody2Dのうち、同期するプロパティをオプションで選択できます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/07-rigidbody2d-sync-option.png)

| オプション | 説明 |
| --- | --- |
| Synchronize Velocity | Velocityプロパティを同期するかどうかを設定します。 trueの場合、Velocity値を同期し、falseの場合、Velocity値を同期しません。 |
| Synchronize Angular Velocity | Angular Velocityプロパティを同期するかどうかを設定します。 trueの場合、Angular Velocity値を同期し、falseの場合、Angular Velocity値を同期しません。 |
| Teleport Enabled | Teleportを許可するかどうかを設定します。 |
| Teleport if distance greater than | 距離が設定した基準以上の差がある場合、Rigidbody2DのPositionに同期する位置の値を適用し、Velocityの値を利用して同期します。 |
| Teleport if angle greater than | 角度が設定した基準より大きく異なる場合、Rigidbody2DのRotationに同期する角度値を適用し、Angular Velocity値を利用して同期します。 |

## Rigidbody同期、 RigidbodySync

Rigidbody同期をしたいゲームオブジェクトにRigidbodySyncコンポーネントを追加すると、同期ゲームオブジェクトのRigidbodyが同期されます。
**Add Component > GameAnvil > GameAnvil Sync > RigidbodySync**を選択してコンポーネントとして追加できます。Sync、TransformSync、Rigidbodyコンポーネントが自動的に一緒に追加されます。

RigidbodySyncコンポーネントを付けたゲームオブジェクトをprefabで作成し、UnityのAssets/Resourcesフォルダの下に保存します。

ルームに入室してSyncControllerのInstantiate()を通じて該当prefabを作成した後、Rigidbodyを変更した時、他のクライアントでも全て変更されたRigidbodyで同期されることを確認することができます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/08-rigidbody-sync.gif)

### Rigidbody同期オプション

Rigidbodyの中で同期するプロパティをオプションによって選択できます。

![](https://static.toastoven.net/prod_gameanvil/images/unity-basic/05-sync/09-rigidbody-sync-option.png)

| オプション | 説明 |
| --- | --- |
| Synchronize Velocity | Velocityプロパティを同期するかどうかを設定します。 trueの場合、Velocity値を同期し、falseの場合、Velocity値を同期しません。 |
| Synchronize Angular Velocity | Angular Velocityプロパティを同期するかどうかを設定します。 trueの場合、Angular Velocity値を同期し、falseの場合、Angular Velocity値を同期しません。 |
| Teleport Enabled | Teleportを許可するかどうかを設定します。 |
| Teleport if distance greater than | 距離が設定した基準以上の差がある場合、RigidbodyのPositionに同期する位置の値を適用した後、Velocity値を利用して同期します。 |
| Teleport if angle greater than | 角度が設定した基準より大きい場合、RigidbodyのRotationに同期する角度値を適用し、Angular Velocityの値を利用して同期します。 |

## ユーザー定義値の同期

int、float、bool、stringタイプのユーザー定義値を同期する機能を提供します。

### ユーザー定義値の追加または変更

SyncController.SetCustomProperty\<T\>()でユーザー定義値を追加または変更できます。関数呼び出し時、ユーザー定義値のタイプを指定します。ユーザー定義値を区別するためのキーをstringタイプの引数で渡します。サーバーではそのユーザー定義値のデータを保存し、同じルームの全てのユーザーにブロードキャストして同期できるようにします。

```c#
/// <summary>
/// ユーザー定義値を追加します。
/// </summary>
/// <typeparam name="T">ユーザー定義値のタイプ</typeparam>
/// <param name="key">ユーザー定義値区分用キー</param>
/// <param name="value">ユーザー定義値</param>
public static void SetCustomProperty<T>(string key, T value);
```

使用例を見てみましょう。

```c#
SyncController.SetCustomProperty<float>("custom_key", 0.9f);
```

### ユーザー定義値が最新の状態であることを確認して変更

SyncController.SetCustomPropertyCAS\<T\>()を呼び出すと、引数で受け取ったユーザー定義値区分用キー、変更するユーザー定義値と、既存のクライアントに保存されていたユーザー定義値を一緒にパケットに入れてサーバーに送信します。

すると、サーバーでは、既存のクライアントに保存されていたユーザー定義値とサーバーに保存されているデータから区別用キーで取得したユーザー定義値を比較します。

もし、この2つが同じであれば、クライアントのユーザー定義値が最新の状態で同期されていたと判断し、変更したい値に置き換えてサーバーに保存し、他のユーザーにもブロードキャストして同期できるようにします。

もし、クライアントとサーバーにそれぞれ保存されたユーザー定義値が違う場合、そのリクエストを無視します。

```c#
/// <summary>
/// ユーザー定義値が最新の状態であることを確認して変更
/// </summary>
/// <typeparam name="T">ユーザー定義値のタイプ</typeparam>
/// <param name="key">ユーザー定義値区分用キー</param>
/// <param name="value">ユーザー定義値</param>
public static void SetCustomPropertyCAS<T>(string key, T value);
```

使用例を見てみましょう。

```c#
SyncController.SetCustomPropertyCAS<float>("custom_key", 0.9f);
```

### ユーザー定義値の照会

SyncController.GetCustomProperty\<T\>()でユーザー定義値を照会できます。関数呼び出し時、ユーザー定義値のタイプを指定します。目的のユーザー定義値を探すため、区別用キーをstringタイプの引数で渡します。

```c#
/// <summary>
/// ユーザー定義値を照会します。
/// </summary>
/// <typeparam name="T">ユーザー定義値のタイプ</typeparam>
/// <param name="key">ユーザー定義値区分用キー</param>
/// <returns>ユーザー定義値</returns>
public static T GetCustomProperty<T>(string key);
```

使用例を見てみましょう。

```c#
float custom_value = SyncController.GetCustomProperty<float>("custom_key");
```

### ユーザー定義値の削除

SyncController.RemoveCustomProperty\<T\>()でユーザー定義値を削除できます。関数呼び出し時、ユーザー定義値のタイプを指定します。希望するユーザー定義値を探すため、区別用キーをstringタイプの引数で渡します。サーバーでも保存していたユーザー定義値のデータを削除します。

```c#
/// <summary>
/// ユーザー定義値を削除します。
/// </summary>
/// <param name="key">ユーザー定義値区分用キー</param>
public static void RemoveCustomProperty(string key)
```

使用例を見てみましょう。

```c#
SyncController.RemoveCustomProperty("custom_key");
```
