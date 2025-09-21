<cite>
**本文档中引用的文件**
- [GameEntry.cs](file://Scripts/Runtime/Base/GameEntry.cs)
- [GameFrameworkComponent.cs](file://Scripts/Runtime/Base/GameFrameworkComponent.cs)
- [ShutdownType.cs](file://Scripts/Runtime/Base/ShutdownType.cs)
- [ResourceComponent.cs](file://Scripts/Runtime/Resource/ResourceComponent.cs)
- [SceneComponent.cs](file://Scripts/Runtime/Scene/SceneComponent.cs)
- [LoadSceneSuccessEventArgs.cs](file://Scripts/Runtime/Scene/LoadSceneSuccessEventArgs.cs)
</cite>

# API 参考

## 目录
1. [核心入口点](#核心入口点)
2. [基础组件](#基础组件)
3. [资源管理模块](#资源管理模块)
4. [场景管理模块](#场景管理模块)
5. [事件系统](#事件系统)

## 核心入口点

`GameEntry` 类是整个游戏框架的静态入口点，提供了访问所有游戏框架组件的统一接口。它维护了一个游戏框架组件的链表，允许通过类型、类型名称或具体类型来获取相应的组件实例。

### 静态方法

#### GetComponent\<T>()
- **功能**: 通过泛型类型获取指定的游戏框架组件。
- **参数**: 
  - `T`: 要获取的组件的类型，必须继承自 `GameFrameworkComponent`。
- **返回值**: 指定类型的 `GameFrameworkComponent` 实例，如果未找到则返回 `null`。
- **使用示例**:
  ```csharp
  ResourceComponent resourceComponent = GameEntry.GetComponent<ResourceComponent>();
  ```

#### GetComponent(Type)
- **功能**: 通过 `Type` 对象获取指定的游戏框架组件。
- **参数**: 
  - `type`: 要获取的组件的 `Type` 对象。
- **返回值**: `GameFrameworkComponent` 实例，如果未找到则返回 `null`。
- **使用示例**:
  ```csharp
  Type resourceType = typeof(ResourceComponent);
  GameFrameworkComponent component = GameEntry.GetComponent(resourceType);
  ```

#### GetComponent(string)
- **功能**: 通过类型的全名或名称获取指定的游戏框架组件。
- **参数**: 
  - `typeName`: 要获取的组件类型的全名（如 `UnityGameFramework.Runtime.ResourceComponent`）或名称（如 `ResourceComponent`）。
- **返回值**: `GameFrameworkComponent` 实例，如果未找到则返回 `null`。
- **使用示例**:
  ```csharp
  GameFrameworkComponent component = GameEntry.GetComponent("ResourceComponent");
  ```

#### Shutdown(ShutdownType)
- **功能**: 关闭游戏框架，并根据指定的关闭类型执行相应操作。
- **参数**: 
  - `shutdownType`: `ShutdownType` 枚举值，定义了关闭框架的行为。
- **异常情况**: 该方法本身不抛出异常，但会记录错误日志。
- **使用示例**:
  ```csharp
  GameEntry.Shutdown(ShutdownType.Quit);
  ```
- **行为说明**:
  - `ShutdownType.None`: 仅关闭框架内部组件，不退出游戏。
  - `ShutdownType.Restart`: 重新加载游戏框架所在的场景。
  - `ShutdownType.Quit`: 调用 `Application.Quit()` 退出游戏，在编辑器中还会将 `EditorApplication.isPlaying` 设置为 `false`。

**Section sources**
- [GameEntry.cs](file://Scripts/Runtime/Base/GameEntry.cs#L18-L145)

## 基础组件

### GameFrameworkComponent

`GameFrameworkComponent` 是所有游戏框架组件的基类，它继承自 `MonoBehaviour`。该类定义了组件的生命周期，并在 `Awake` 事件中自动将自身注册到 `GameEntry` 中。

- **生命周期**: 组件在 `Awake` 时被自动注册，开发者无需手动调用注册方法。
- **继承**: 所有自定义的游戏框架组件都应继承此类。

**Section sources**
- [GameFrameworkComponent.cs](file://Scripts/Runtime/Base/GameFrameworkComponent.cs#L14-L23)

### ShutdownType

`ShutdownType` 是一个枚举，用于定义 `GameEntry.Shutdown` 方法的行为。

- **None**: 仅关闭游戏框架。
- **Restart**: 关闭游戏框架并重启游戏。
- **Quit**: 关闭游戏框架并退出游戏。

**Section sources**
- [ShutdownType.cs](file://Scripts/Runtime/Base/ShutdownType.cs#L10-L30)

## 资源管理模块

`ResourceComponent` 是资源管理的核心组件，负责加载、卸载和管理游戏资源。

### 核心属性

- **ReadOnlyPath**: 获取资源只读路径（通常是 `StreamingAssets` 文件夹）。
- **ReadWritePath**: 获取资源读写路径（通常是 `PersistentDataPath`）。
- **ResourceMode**: 获取当前的资源模式（单机、可更新等）。
- **ApplicableGameVersion**: 获取当前资源适用的游戏版本号。
- **AssetCount**: 获取已加载的资源数量。
- **ResourceCount**: 获取资源总数。

### 核心方法

#### LoadAsset 重载
`LoadAsset` 提供了多个重载方法，用于异步加载资源。

- **参数**:
  - `assetName`: 资源的完整路径，必须以 `Assets/` 开头。
  - `assetType`: 可选，指定资源的预期类型（如 `GameObject`）。
  - `priority`: 可选，加载任务的优先级，默认为 `0`。
  - `loadAssetCallbacks`: 必需，包含加载成功、失败、更新和依赖项回调的委托集合。
  - `userData`: 可选，用户自定义数据，会在回调中返回。
- **返回值**: 无。
- **异常情况**: 如果 `assetName` 无效，会记录错误日志并直接返回。
- **使用示例**:
  ```csharp
  LoadAssetCallbacks callbacks = new LoadAssetCallbacks(OnLoadSuccess, OnLoadFailure);
  GameEntry.GetComponent<ResourceComponent>().LoadAsset("Assets/Prefabs/Player.prefab", callbacks);
  ```

#### 其他重要方法
- **UnloadAsset(object asset)**: 卸载指定的资源。
- **InitResources(InitResourcesCompleteCallback)**: 在单机模式下初始化资源。
- **UpdateResources(UpdateResourcesCompleteCallback)**: 在可更新模式下更新所有资源。
- **HasAsset(string assetName)**: 检查指定资源是否存在。

**Section sources**
- [ResourceComponent.cs](file://Scripts/Runtime/Resource/ResourceComponent.cs#L20-L1365)

## 场景管理模块

`SceneComponent` 负责管理游戏场景的加载和卸载。

### 核心方法

#### LoadScene 重载
`LoadScene` 方法用于异步加载场景。

- **参数**:
  - `sceneAssetName`: 场景资源的完整路径，必须以 `Assets/` 开头，并以 `.unity` 结尾。
  - `priority`: 可选，加载任务的优先级。
  - `userData`: 可选，用户自定义数据。
- **返回值**: 无。
- **异常情况**: 如果 `sceneAssetName` 无效，会记录错误日志并直接返回。
- **使用示例**:
  ```csharp
  GameEntry.GetComponent<SceneComponent>().LoadScene("Assets/Scenes/MainMenu.unity");
  ```

#### UnloadScene
- **功能**: 卸载指定的场景。
- **参数**: `sceneAssetName`, `userData`。
- **注意**: 卸载场景后，会自动从内部的场景顺序字典中移除该场景。

#### SetSceneOrder
- **功能**: 设置场景的加载顺序，用于确定哪个场景是“活动场景”。
- **参数**: `sceneAssetName`, `sceneOrder`。
- **说明**: 数值越大，优先级越高。活动场景会成为 `SceneManager` 的当前活动场景。

**Section sources**
- [SceneComponent.cs](file://Scripts/Runtime/Scene/SceneComponent.cs#L20-L475)

## 事件系统

框架通过事件系统通知外部代码资源和场景的加载状态。

### LoadSceneSuccessEventArgs

此事件在场景成功加载后触发。

- **字段**:
  - `SceneAssetName`: 成功加载的场景资源名称。
  - `Duration`: 加载过程所花费的时间（秒）。
  - `UserData`: 用户在调用 `LoadScene` 时传入的自定义数据。
- **用途**: 用于在场景加载完成后执行初始化逻辑。

**Section sources**
- [LoadSceneSuccessEventArgs.cs](file://Scripts/Runtime/Scene/LoadSceneSuccessEventArgs.cs#L15-L93)