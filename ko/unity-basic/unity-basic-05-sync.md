## Game > GameAnvil > Unity 기초 개발 가이드 > 동기화

## 동기화

GameAnvilManager에서는 게임 오브젝트의 생성/파괴, Transform, Animation, Rigidbody2D, Rigidbody 속성을 간편하게 동기화할 수 있는 기능을 제공합니다.

동기화할 게임 오브젝트에 각각 Sync, TransformSync, AnimatorSync, Rigidbody2DSync, RigidbodySync 컴포넌트를 붙이기만 하면 해당 속성이 동기화됩니다.

동기화 컴포넌트를 사용할 게임 오브젝트에는 Sync 컴포넌트도 반드시 함께 추가되어야 합니다. 각 동기화 컴포넌트를 추가했을 때 Sync 컴포넌트가 없을 경우 자동으로 추가됩니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/01-component.png)

한 클라이언트에서 특정 속성을 변화시키면 동기화 요청 패킷이 전송되고, 서버에서 이를 같은 방에 속한 모든 유저에게 전파하여 다른 클라이언트에서 동기화될 수 있도록 합니다.

## SyncController

동기화 기능을 사용하려는 씬에 SyncController가 존재해야 합니다. 동기화 게임오브젝트를 생성하고 파괴하는 Instantiate(), Destroy() 메소드를 포함하여 동기화 기능 동작에 핵심적인 역할을 합니다.

### SyncController 생성

Unity Hierarchy 창에서 마우스 오른쪽 버튼을 클릭한 뒤 **GameAnvil > SyncController**를 선택해 바로 생성할 수 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/02-add-sync-controller.png)

또는 또는 빈 GameObject를 생성하고 SyncController 컴포넌트를 추가할 수도 있습니다.
SyncController에는 다음과 같은 옵션이 있습니다.

| 옵션                  | 설명                                                                                                                                                      |
|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| Use Synchronize Log | 동기화 관련 로그 출력 여부                                                                                                                                         |
| Lazy Loading        | 방에 입장한 직후 자동으로 바로 기존 데이터를 동기화할지 여부 <br/>만약 이 옵션을 false로 설정할 경우, 기존 데이터를 동기화하고 싶은 시점에 직접 SyncController::InstantiateSyncObject()를 호출한다. <br/>(기본값: true) |

## 게임 오브젝트 생성/파괴 동기화, Sync

게임 오브젝트 생성/파괴를 동기화할 게임 오브젝트에 Sync 컴포넌트를 추가하면 같은 방에 있는 유저들이 서로 생성한 게임 오브젝트의 생성/파괴가 동기화됩니다.
컴포넌트를 추가할 게임 오브젝트를 선택한 후 메뉴에서 **Component > GameAnvil > GameAnvil Sync > Sync**를 선택해 컴포넌트로 추가할 수 있습니다. 인스펙터 윈도우에서 **Add Component** 버튼을 누르고 Sync 컴포넌트를 찾아 추가할 수도 있습니다.

### Sync Id

유저별로 고유한 동기화 키가 부여되고, 게임 오브젝트마다 오브젝트 아이디가 부여되며 이 둘을 조합하여 생성한 동기화 아이디로 모든 유저별 동기화 게임오브젝트가 구분됩니다.

Sync.SyncId로 동기화 게임오브젝트의 동기화 아이디를 얻을 수 있습니다.

### Create Option

동기화 게임 오브젝트가 생성된 방식을 나타냅니다. 클라이언트에서 직접 생성한 게임오브젝트의 경우 LOCAL로 표시하고, 다른 클라이언트에서 생성한 게임오브젝트가 생성 동기화에 의해 자신의 씬에서도 만들어지는 경우에는 REMOTE로 표시됩니다.

### 게임 오브젝트 생성 동기화

Sync 컴포넌트를 추가한 게임 오브젝트를 prefab으로 만든 뒤 Unity의 Assets/Resources 폴더 하위에 저장합니다. 방에 입장한 뒤 SyncController의 Instantiate()를 통해 원하는 시점에 해당 prefab을 생성합니다.

방에 입장한 상태에서만 생성/파괴가 동기화되는 게임 오브젝트를 생성할 수 있습니다.

```c#
public void Instanticate()
{
    SyncController.Instance.Instantiate("prefabName", Vector3.zero, Quaternion.identity);
}
```

### 게임 중 방에 새 유저가 입장한 경우 게임 오브젝트 생성 동기화

플레이중인 방에 새 유저가 입장한 경우 입장과 동시에 방에서 플레이 중이던 다른 유저들의 동기화 데이터를 전송받게 됩니다. 이렇게 전송된 동기화 데이터를 이용해 다른 유저들의 게임 오브젝트를 자동으로 생성하고 동기화합니다.

하지만 방에 입장한 직후 바로 게임 오브젝트를 생성하고 싶지 않을 수 있습니다. 이런 경우에는 SyncController의 Lazy Loading 옵션을 false로 설정하고, 원하는 시점에 SyncController.InstantiateSyncObject()를 호출하여 다른 유저들의 게임 오브젝트를 생성할 수 있습니다.
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/03-lazy-loading.png)


```c#
public void InstantiateSyncObject()
{
    SyncController.Instance.InstantiateSyncObject();
}
```

### 게임 오브젝트 파괴 동기화

Sync 컴포넌트가 붙은 게임 오브젝트가 삭제되면 자동으로 동기화가 처리되어서 다른 유저의 씬에서도 해당 게임오브젝트가 사라집니다.

유저가 방에 입장하고 동기화 게임 오브젝트들을 생성한 뒤 씬을 이동하면 게임 오브젝트들이 모두 파괴되고, 이를 동기화하면서 서버에 저장된 데이터도 삭제됩니다.

따라서 다른 씬으로 이동했다가 다시 원래 씬으로 돌아왔을 때 생성했던 동기화 게임오브젝트들이 없어져 있을 수 있습니다. 그리고 한 쪽 클라이언트에서 씬 이동을 하면서 동기화 게임오브젝트가 모두 파괴되면, 파괴 동기화에 의해 다른 클라이언트에서도 게임오브젝트들이 파괴될 수 있습니다. 이에 유념해서 씬 이동을 해야 합니다. 이는 추후에 개선될 예정입니다.

## Transform 동기화, TransformSync

Transform의 동기화를 원하는 게임 오브젝트에 TransformSync 컴포넌트를 추가하면 동기화 게임 오브젝트의 Transform이 동기화됩니다.
컴포넌트를 추가할 게임 오브젝트를 선택한 후 메뉴에서 **Component > GameAnvil > GameAnvil Sync > TransformSync**를 선택해 컴포넌트로 추가할 수 있습니다. 인스펙터 윈도우에서 **Add Component** 버튼을 누르고 TransformSync 컴포넌트를 찾아 추가할 수도 있습니다.
이 게임 오브젝트를 prefab으로 만든 뒤 Unity의 Assets/Resources 폴더 하위에 저장하고, 방에 입장해서 SyncController의 Instantiate()를 통해서 해당 prefab을 생성한 다음 Transform을 변화시켰을 때, 다른 클라이언트에서도 모두 변화된 Transform으로 동기화되는 것을 확인할 수 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/04-transform-sync.gif)

### Transform 동기화 옵션

Transform 중 동기화할 속성을 옵션에 따라 선택할 수 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/05-transform-sync-option.png)

| 옵션                   | 설명                                                                                                                |
|----------------------|-------------------------------------------------------------------------------------------------------------------|
| Synchronize Position | Position 속성 동기화 여부를 설정합니다. true면 Position 값을 동기화하고, false면 Position 값을 동기화하지 않습니다.                                |
| Synchronize Rotation | Rotation 속성 동기화 여부를 설정합니다. true면 Rotation 값을 동기화하고, false면 Rotation 값을 동기화하지 않습니다.                                |
| Synchronize Scale    | Scale 속성 동기화 여부를 설정합니다. true면 Scale 값을 동기화하고, false면 Scale 값을 동기화하지 않습니다.                                         |
| Use Local            | localPosition 및 localRotation을 사용해야 하는지 여부를 설정합니다. Scale은 이 설정을 무시하고 항상 localScale을 사용하여 lossyScale 관련 문제를 방지합니다. |

## Animation 동기화, AnimatorSync

Animation의 동기화를 원하는 게임 오브젝트에 AnimatorSync 컴포넌트를 붙이고, Animator의 파라미터 값을 바꿔서 Animation State를 변경시키면 해당 변경 사항이 다른 클라이언트에서도 동기화됩니다.
컴포넌트를 추가할 게임 오브젝트를 선택한 후 메뉴에서 **Component > GameAnvil > GameAnvil Sync > AnimatorSync**를 선택해 컴포넌트로 추가할 수 있습니다. 인스펙터 윈도우에서 **Add Component** 버튼을 누르고 AnimatorSync 컴포넌트를 찾아 추가할 수도 있습니다.
이 게임 오브젝트를 prefab으로 만든 뒤 Unity의 Assets/Resources 폴더 하위에 저장하고, 방에 입장해서 SyncController의 Instantiate()를 통해서 해당 prefab을 생성한 다음 Animation을 변화시켰을 때, 다른 클라이언트에서도 모두 변화된 Animation으로 동기화되는 것을 확인할 수 있습니다.
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/06-animator-sync.gif)

## Rigidbody2D 동기화, Rigidbody2DSync

Rigidbody2D의 동기화를 원하는 게임 오브젝트에 Rigidbody2DSync 컴포넌트를 붙이면 동기화 게임오브젝트의 Rigidbody2D가 동기화됩니다.
컴포넌트를 추가할 게임 오브젝트를 선택한 후 메뉴에서 **Component > GameAnvil > GameAnvil Sync > Rigidbody2DSync**를 선택해 컴포넌트로 추가할 수 있습니다. 인스펙터 윈도우에서 **Add Component** 버튼을 누르고 Rigidbody2DSync 컴포넌트를 찾아 추가할 수도 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/07-rigidbody2d-sync.gif)

Rigidbody2DSync 컴포넌트를 붙인 게임 오브젝트를 prefab으로 만든 뒤 Unity의 Assets/Resources 폴더 하위에 저장하고, 방에 입장해서 SyncController의 Instantiate()를 통해서 해당 prefab을 생성한 다음 Rigidbody2D를 변화시켰을 때 다른 클라이언트에서도 모두 변화된 Rigidbody2D로 동기화되는 것을 확인할 수 있습니다.

### Rigidbody2D 동기화 옵션

Rigidbody2D 중 동기화할 속성을 옵션에 따라 선택할 수 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/08-rigidbody2d-sync-option.png)

| 옵션                                | 설명                                                                                                                                       |
|-----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| Synchronize Velocity              | Velocity 속성의 동기화 여부를 설정합니다. true면 Velocity 값을 동기화하고, false면 Velocity 값을 동기화하지 않습니다.                                                      |
| Synchronize Angular Velocity      | Angular Velocity 속성의 동기화 여부를 설정합니다. true면 Angular Velocity 값을 동기화하고, false면 Angular Velocity 값을 동기화하지 않습니다.                              |
| Teleport Enabled                  | Teleport 허용 여부를 설정합니다.                                                                                                                   |
| Teleport if distance greater than | 거리가 설정한 기준 이상으로 차이가 나면 Rigidbody2D의 Position에 동기화할 위치 값을 적용한 다음, Velocity 값을 이용해서 동기화합니다. <br/>Teleport Enabled가 체크된 경우에만 표시됩니다.         |
| Teleport if angle greater than    | 각도가 설정한 기준 이상으로 차이가 나면 Rigidbody2D의 Rotation에 동기화할 각도 값을 적용한 다음, Angular Velocity 값을 이용해서 동기화합니다. <br/>Teleport Enabled가 체크된 경우에만 표시됩니다. |

## Rigidbody 동기화, RigidbodySync

Rigidbody의 동기화를 원하는 게임 오브젝트에 RigidbodySync 컴포넌트를 추가하면 동기화 게임 오브젝트의 Rigidbody가 동기화됩니다.
컴포넌트를 추가할 게임 오브젝트를 선택한 후 메뉴에서 **Component > GameAnvil > GameAnvil Sync > RigidbodySync**를 선택해 컴포넌트로 추가할 수 있습니다. 인스펙터 윈도우에서 **Add Component** 버튼을 누르고 RigidbodySync 컴포넌트를 찾아 추가할 수도 있습니다.

RigidbodySync 컴포넌트를 붙인 게임 오브젝트를 prefab으로 만든 뒤 Unity의 Assets/Resources 폴더 하위에 저장하고, 방에 입장해서 SyncController의 Instantiate()를 통해서 해당 prefab을 생성한 다음 Rigidbody를 변화시켰을 때 다른 클라이언트에서도 모두 변화된 Rigidbody로 동기화되는 것을 확인할 수 있습니다.

![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/09-rigidbody-sync.gif)

### Rigidbody 동기화 옵션

Rigidbody 중 동기화할 속성을 옵션에 따라 선택할 수 있습니다.
![](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_gameanvil/images/v2_0/unity-basic/05-sync/10-rigidbody-sync-option.png)

| 옵션                                | 설명                                                                                                                                     |
|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| Synchronize Velocity              | Velocity 속성의 동기화 여부를 설정합니다. true면 Velocity 값을 동기화하고, false면 Velocity 값을 동기화하지 않습니다.                                                    |
| Synchronize Angular Velocity      | Angular Velocity 속성의 동기화 여부를 설정합니다. true면 Angular Velocity 값을 동기화하고, false면 Angular Velocity 값을 동기화하지 않습니다.                            |
| Teleport Enabled                  | Teleport 허용 여부를 설정합니다.                                                                                                                 |
| Teleport if distance greater than | 거리가 설정한 기준 이상으로 차이가 나면 Rigidbody의 Position에 동기화할 위치 값을 적용한 다음, Velocity 값을 이용해서 동기화합니다. <br/>Teleport Enabled가 체크된 경우에만 표시됩니다.         |
| Teleport if angle greater than    | 각도가 설정한 기준 이상으로 차이가 나면 Rigidbody의 Rotation에 동기화할 각도 값을 적용한 다음, Angular Velocity 값을 이용해서 동기화합니다. <br/>Teleport Enabled가 체크된 경우에만 표시됩니다. |

## 사용자 정의 값 동기화

int, float, bool, string 타입의 사용자 정의 값을 동기화하는 기능을 제공합니다.

### 사용자 정의 값 추가 또는 변경

SetCustomProperty\<T\>()로 사용자 정의 값을 추가 또는 변경할 수 있습니다. 서버에서는 해당 사용자 정의 값 데이터를 저장하고, 같은 방의 모든 유저에게 브로드캐스트하여 동기화할 수 있도록 합니다.
SetCustomProperty\<T\>()은 다음과 같이 1개의 타입 매개변수와 2개의 매개변수를 가지고 있습니다.

| 타입      | 이름    | 설명                                              |
|---------|-------|-------------------------------------------------|
| 타입 매개변수 | T     | 저장할 사용자 정의 값의 타입. int, float, bool, string 중 하나 |
| String  | key   | 사용자 정의 값을 구분하기 위한 키                             |
| T       | value | 사용자 정의 값                                        |

```c#
public void SetCustomProperty()
{
    SyncController.Instance.SetCustomProperty("IntValue", 1);
    SyncController.Instance.SetCustomProperty("FloatValue", 1.0f);
    SyncController.Instance.SetCustomProperty("BoolValue", false);
    SyncController.Instance.SetCustomProperty("StringValue", "Value");
}
```

### 사용자 정의 값이 최신 상태인지 확인 후 변경

SetCustomPropertyCas\<T\>()를 호출하면 클라이언트에 저장되어 있던 사용자 정의 값과 서버에 저장되어 있는 사용자 정의 값을 비교하여 같은 경우에만 사용자 정의 값 데이터를 저장하고, 같은 방의 모든 유저에게 브로드캐스트하여 동기화할 수 있도록 합니다. 만약 클라이언트와 서버에 각각 저장되어 있던 사용자 정의 값이 다른 경우 해당 요청을 무시합니다.
SetCustomPropertyCas\<T\>()은 다음과 같이 1개의 타입 매개변수와 2개의 매개변수를 가지고 있습니다.

| 타입      | 이름    | 설명                                              |
|---------|-------|-------------------------------------------------|
| 타입 매개변수 | T     | 저장할 사용자 정의 값의 타입. int, float, bool, string 중 하나 |
| String  | key   | 사용자 정의 값을 구분하기 위한 키                             |
| T       | value | 사용자 정의 값                                        |

```c#
public void SetCustomPropertyCas()
{
    SyncController.Instance.SetCustomPropertyCas("IntValue", 1);
    SyncController.Instance.SetCustomPropertyCas("FloatValue", 1.0f);
    SyncController.Instance.SetCustomPropertyCas("BoolValue", false);
    SyncController.Instance.SetCustomPropertyCas("StringValue", "Value");
}
```

### 사용자 정의 값 조회

GetCustomProperty\<T\>()로 사용자 정의 값을 조회할 수 있습니다.
SetCustomPropertyCas\<T\>()은 다음과 같이 1개의 타입 매개변수와 1개의 매개변수를 가지고 있습니다.

| 타입      | 이름  | 설명                                              |
|---------|-----|-------------------------------------------------|
| 타입 매개변수 | T   | 조회할 사용자 정의 값의 타입. int, float, bool, string 중 하나 |
| String  | key | 사용자 정의 값을 구분하기 위한 키                             |

```c#
public void GetCustomProperty()
{
    int intValue = SyncController.Instance.GetCustomProperty<int>("IntValue");
    float floatValue = SyncController.Instance.GetCustomProperty<float>("FloatValue");
    bool boolValue = SyncController.Instance.GetCustomProperty<bool>("BoolValue");
    string stringValue = SyncController.Instance.GetCustomProperty<string>("StringValue");        
}
```

### 사용자 정의 값 삭제

RemoveCustomProperty\<T\>()로 사용자 정의 값을 삭제할 수 있습니다. 서버에서도 저장했던 사용자 정의 값 데이터를 삭제합니다.
SetCustomPropertyCas\<T\>()은 다음과 같이 1개의 매개변수를 가지고 있습니다.

| 타입     | 이름  | 설명                  |
|--------|-----|---------------------|
| String | key | 사용자 정의 값을 구분하기 위한 키 |

```c#
public void RemoveCustomProperty()
{
    SyncController.Instance.RemoveCustomProperty("IntValue");
    SyncController.Instance.RemoveCustomProperty("FloatValue");
    SyncController.Instance.RemoveCustomProperty("BoolValue");
    SyncController.Instance.RemoveCustomProperty("StringValue");
}
```