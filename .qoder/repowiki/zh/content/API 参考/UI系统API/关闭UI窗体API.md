# 关闭UI窗体API

<cite>
**本文档中引用的文件**   
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs)
- [CloseUIFormCompleteEventArgs.cs](file://Scripts/Runtime/UI/CloseUIFormCompleteEventArgs.cs)
- [UIForm.cs](file://Scripts/Runtime/UI/UIForm.cs)
</cite>

## 目录
1. [简介](#简介)
2. [核心功能](#核心功能)
3. [CloseUIForm方法详解](#closeuiform方法详解)
4. [userData参数说明](#userdata参数说明)
5. [CloseUIFormCompleteEventArgs事件](#closeuiformcompleteeventargs事件)
6. [代码示例](#代码示例)
7. [异步操作与对象池机制](#异步操作与对象池机制)
8. [最佳实践](#最佳实践)

## 简介
本文档详细说明了UnityGameFramework中用于关闭UI窗体的API。`UIComponent`类提供了`CloseUIForm`方法，支持通过窗体序列号（serialId）或直接传入`UIForm`实例两种方式来关闭界面。该操作是异步的，并与对象池机制集成，确保资源的高效回收。同时，文档还介绍了`CloseUIFormCompleteEventArgs`事件类，用于在窗体完全关闭并释放后执行后续逻辑。

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L586-L630)
- [CloseUIFormCompleteEventArgs.cs](file://Scripts/Runtime/UI/CloseUIFormCompleteEventArgs.cs#L16-L106)

## 核心功能
`UIComponent`作为界面管理的核心组件，封装了对UI窗体的打开、关闭、查询等操作。其核心功能包括：
- 通过序列号或实例引用关闭UI窗体
- 支持传递用户自定义数据（userData）
- 提供事件系统以响应窗体关闭完成
- 与对象池（Object Pool）集成，实现窗体实例的复用与管理

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L25-L25)
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L131-L181)

## CloseUIForm方法详解
`CloseUIForm`方法提供了四种重载，允许开发者以灵活的方式关闭UI窗体。

### 通过序列号关闭窗体
```csharp
public void CloseUIForm(int serialId)
```
此方法通过窗体的唯一序列号来关闭指定的UI窗体。每个打开的UI窗体都会被分配一个唯一的`serialId`，可通过`OpenUIForm`方法的返回值获取。

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L590-L593)

### 通过序列号关闭窗体并传递数据
```csharp
public void CloseUIForm(int serialId, object userData)
```
此重载在通过序列号关闭窗体的同时，允许传入一个`userData`对象。该数据将在后续的关闭完成事件中被传递，可用于记录关闭原因或附加信息。

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L600-L603)

### 通过UIForm实例关闭窗体
```csharp
public void CloseUIForm(UIForm uiForm)
```
此方法直接接收一个`UIForm`实例作为参数来关闭窗体。这在持有窗体引用时非常方便，避免了通过序列号查找的开销。

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L609-L612)

### 通过UIForm实例关闭窗体并传递数据
```csharp
public void CloseUIForm(UIForm uiForm, object userData)
```
此重载结合了实例关闭和数据传递的功能，是最常用的关闭方式之一。

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L619-L622)

## userData参数说明
`userData`是一个类型为`object`的可选参数，用于在关闭窗体时传递用户自定义的数据。其主要用途包括：
- **记录关闭原因**：例如，传递一个枚举值表示“用户点击关闭”或“超时自动关闭”。
- **传递上下文信息**：例如，在关闭登录界面时，传递登录结果或错误信息。
- **回调数据**：在关闭完成事件中，可以获取此数据并执行相应的后续操作。

该参数的值会被封装在`CloseUIFormCompleteEventArgs`事件中，在事件回调函数里可以访问。

**Section sources**
- [CloseUIFormCompleteEventArgs.cs](file://Scripts/Runtime/UI/CloseUIFormCompleteEventArgs.cs#L70-L75)

## CloseUIFormCompleteEventArgs事件
当一个UI窗体被完全关闭并从内存中释放后，系统会触发`CloseUIFormCompleteEventArgs`事件。该事件是异步的，标志着窗体生命周期的终结。

### 事件结构
```csharp
public sealed class CloseUIFormCompleteEventArgs : GameEventArgs
{
    public int SerialId { get; private set; }
    public string UIFormAssetName { get; private set; }
    public IUIGroup UIGroup { get; private set; }
    public object UserData { get; private set; }
}
```

| 属性 | 类型 | 描述 |
| :--- | :--- | :--- |
| `SerialId` | `int` | 被关闭窗体的唯一序列编号 |
| `UIFormAssetName` | `string` | 被关闭窗体的资源名称 |
| `UIGroup` | `IUIGroup` | 被关闭窗体所属的界面组 |
| `UserData` | `object` | 用户在关闭时传入的自定义数据 |

### 触发条件
该事件在以下流程完成后触发：
1. UI窗体的`OnClose`逻辑方法被调用。
2. 窗体的GameObject被销毁或归还到对象池。
3. 所有相关的资源引用被清理。

### 事件订阅
开发者可以通过`EventComponent`来订阅此事件，以便在窗体关闭后执行清理、跳转或其他业务逻辑。

```csharp
// 示例：订阅关闭完成事件
EventComponent eventComponent = GameEntry.GetComponent<EventComponent>();
eventComponent.Subscribe(CloseUIFormCompleteEventArgs.EventId, OnCloseUIFormComplete);

private void OnCloseUIFormComplete(object sender, GameEventArgs e)
{
    CloseUIFormCompleteEventArgs args = (CloseUIFormCompleteEventArgs)e;
    // 执行后续逻辑，例如跳转到主菜单
    Debug.Log($"窗体 {args.UIFormAssetName} 已关闭，序列号: {args.SerialId}");
}
```

**Section sources**
- [CloseUIFormCompleteEventArgs.cs](file://Scripts/Runtime/UI/CloseUIFormCompleteEventArgs.cs#L16-L106)
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L724-L727)

## 代码示例
以下示例展示了如何安全地关闭UI窗体并在完成后执行操作。

### 示例1：通过序列号关闭并处理完成事件
```csharp
public class UIManagerExample : MonoBehaviour
{
    private int loginFormSerialId;

    // 打开登录窗体
    public void OpenLoginForm()
    {
        UIComponent uiComponent = GameEntry.GetComponent<UIComponent>();
        loginFormSerialId = uiComponent.OpenUIForm("LoginForm", "Default");
    }

    // 关闭登录窗体
    public void CloseLoginForm(string reason)
    {
        UIComponent uiComponent = GameEntry.GetComponent<UIComponent>();
        // 传递关闭原因
        uiComponent.CloseUIForm(loginFormSerialId, reason);
    }

    // 订阅关闭完成事件
    private void Start()
    {
        EventComponent eventComponent = GameEntry.GetComponent<EventComponent>();
        eventComponent.Subscribe(CloseUIFormCompleteEventArgs.EventId, OnLoginFormClosed);
    }

    private void OnLoginFormClosed(object sender, GameEventArgs e)
    {
        CloseUIFormCompleteEventArgs args = (CloseUIFormCompleteEventArgs)e;
        if (args.SerialId == loginFormSerialId)
        {
            // 执行清理或跳转
            string closeReason = (string)args.UserData;
            Debug.Log($"登录窗体已关闭，原因: {closeReason}");
            // 例如，跳转到游戏主界面
            // LoadMainScene();
        }
    }
}
```

### 示例2：通过UIForm实例关闭
```csharp
public class PopupManager : MonoBehaviour
{
    [SerializeField] private UIForm currentPopup;

    public void CloseCurrentPopup()
    {
        if (currentPopup != null && GameEntry.GetComponent<UIComponent>().IsValidUIForm(currentPopup))
        {
            // 直接使用实例关闭
            GameEntry.GetComponent<UIComponent>().CloseUIForm(currentPopup, "用户手动关闭");
            currentPopup = null; // 清理引用
        }
    }
}
```

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L586-L630)
- [CloseUIFormCompleteEventArgs.cs](file://Scripts/Runtime/UI/CloseUIFormCompleteEventArgs.cs#L86-L94)
- [UIForm.cs](file://Scripts/Runtime/UI/UIForm.cs#L176-L221)

## 异步操作与对象池机制
`CloseUIForm`操作是**异步**的。调用该方法后，系统会立即开始关闭流程，但窗体的完全销毁和资源释放可能需要几个帧的时间。因此，不能假设在调用`CloseUIForm`后，窗体实例会立即消失。

### 与对象池的集成
`UIComponent`内部使用了对象池（Object Pool）来管理UI窗体的实例。当调用`CloseUIForm`时：
1. 窗体的`OnClose`方法被调用，执行业务逻辑清理。
2. 窗体的GameObject被禁用并移出场景。
3. 实例被归还到其所属的`ObjectPool`中，而不是被立即销毁。
4. 当再次需要相同类型的窗体时，系统会优先从对象池中取出一个已存在的实例进行复用，从而避免了频繁的Instantiate和Destroy操作，提高了性能。

这种机制极大地优化了内存使用和运行效率，特别是在频繁打开和关闭同类窗体的场景下。

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L183-L219)
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L43-L43)

## 最佳实践
- **始终检查窗体有效性**：在通过实例关闭前，使用`IsValidUIForm`方法检查窗体是否仍然有效。
- **及时清理引用**：在关闭窗体后，将持有的`UIForm`引用置为`null`，防止出现悬空引用。
- **合理使用userData**：利用`userData`传递上下文，避免使用全局变量。
- **订阅完成事件**：对于需要在关闭后执行关键操作的场景，务必订阅`CloseUIFormCompleteEventArgs`事件。
- **避免在OnClose中执行耗时操作**：`OnClose`是UI逻辑的一部分，应保持轻量，避免阻塞主线程。