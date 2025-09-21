# UI系统

<cite>
**Referenced Files in This Document**   
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs)
- [UIForm.cs](file://Scripts/Runtime/UI/UIForm.cs)
- [UIFormLogic.cs](file://Scripts/Runtime/UI/UIFormLogic.cs)
- [DefaultUIFormHelper.cs](file://Scripts/Runtime/UI/DefaultUIFormHelper.cs)
- [OpenUIFormSuccessEventArgs.cs](file://Scripts/Runtime/UI/OpenUIFormSuccessEventArgs.cs)
- [OpenUIFormFailureEventArgs.cs](file://Scripts/Runtime/UI/OpenUIFormFailureEventArgs.cs)
- [CloseUIFormCompleteEventArgs.cs](file://Scripts/Runtime/UI/CloseUIFormCompleteEventArgs.cs)
</cite>

## 目录
1. [简介](#简介)
2. [核心组件](#核心组件)
3. [UI组件与界面管理](#ui组件与界面管理)
4. [界面组与渲染层级](#界面组与渲染层级)
5. [界面生命周期与逻辑脚本](#界面生命周期与逻辑脚本)
6. [异步加载与事件系统](#异步加载与事件系统)
7. [典型使用示例](#典型使用示例)
8. [DefaultUIFormHelper扩展点](#defaultuiformhelper扩展点)

## 简介
本文档详细阐述了Unity游戏框架中的UI系统模块。该系统提供了一套完整的界面管理解决方案，包括界面的打开、关闭、层级管理、生命周期控制以及事件通知机制。通过本系统，开发者可以高效地管理游戏中的用户界面，实现复杂的UI交互逻辑。

## 核心组件

UI系统由多个核心组件构成，它们协同工作以实现完整的界面管理功能。

```mermaid
classDiagram
class UIComponent {
+OpenUIForm(string, string, int, bool, object) int
+CloseUIForm(int, object) void
+GetUIForm(int) UIForm
+AddUIGroup(string, int) bool
}
class UIForm {
+SerialId int
+UIFormAssetName string
+UIGroup IUIGroup
+DepthInUIGroup int
+Logic UIFormLogic
+OnInit(int, string, IUIGroup, bool, bool, object) void
+OnOpen(object) void
+OnClose(bool, object) void
}
class UIFormLogic {
+UIForm UIForm
+Available bool
+Visible bool
+CachedTransform Transform
+OnInit(object) void
+OnOpen(object) void
+OnClose(bool, object) void
+OnUpdate(float, float) void
}
class DefaultUIFormHelper {
+InstantiateUIForm(object) object
+CreateUIForm(object, IUIGroup, object) IUIForm
+ReleaseUIForm(object, object) void
}
UIComponent --> UIForm : "管理"
UIForm --> UIFormLogic : "包含"
UIComponent --> DefaultUIFormHelper : "使用"
UIFormLogic <|-- CustomFormLogic : "继承"
```

**Diagram sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L34-L730)
- [UIForm.cs](file://Scripts/Runtime/UI/UIForm.cs#L16-L303)
- [UIFormLogic.cs](file://Scripts/Runtime/UI/UIFormLogic.cs#L14-L205)
- [DefaultUIFormHelper.cs](file://Scripts/Runtime/UI/DefaultUIFormHelper.cs#L15-L72)

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L34-L730)
- [UIForm.cs](file://Scripts/Runtime/UI/UIForm.cs#L16-L303)
- [UIFormLogic.cs](file://Scripts/Runtime/UI/UIFormLogic.cs#L14-L205)
- [DefaultUIFormHelper.cs](file://Scripts/Runtime/UI/DefaultUIFormHelper.cs#L15-L72)

## UI组件与界面管理

`UIComponent`是UI系统的核心管理组件，负责所有界面的创建、销毁和生命周期管理。它提供了`OpenUIForm`和`CloseUIForm`系列方法来控制界面的显示与隐藏。

```mermaid
sequenceDiagram
participant Client as "客户端代码"
participant UIComponent as "UIComponent"
participant UIManager as "IUIManager"
Client->>UIComponent : OpenUIForm("Login", "Default")
UIComponent->>UIManager : OpenUIForm("Login", "Default", ...)
UIManager-->>UIComponent : 返回序列号
UIComponent-->>Client : 返回序列号
Client->>UIComponent : CloseUIForm(1001)
UIComponent->>UIManager : CloseUIForm(1001)
UIManager-->>UIComponent : 界面关闭
UIComponent-->>Client : 操作完成
```

**Diagram sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L492-L584)
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L590-L622)

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L492-L622)

## 界面组与渲染层级

UI系统通过`UIGroup`（界面组）来管理界面的渲染层级。每个界面组都有一个`Depth`属性，该属性决定了界面组在渲染顺序中的优先级，深度值越大，渲染层级越高。

```mermaid
classDiagram
class UIComponent.UIGroup {
-m_Name string
-m_Depth int
+Name string
+Depth int
}
class UIComponent {
+AddUIGroup(string, int) bool
+GetUIGroup(string) IUIGroup
}
UIComponent --> UIComponent.UIGroup : "包含"
```

**Diagram sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.UIGroup.cs#L12-L39)

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.UIGroup.cs#L12-L39)

## 界面生命周期与逻辑脚本

每个UI界面（`UIForm`）都有一个与之关联的逻辑脚本（`UIFormLogic`），用于处理界面的业务逻辑。`UIFormLogic`定义了完整的生命周期方法，包括初始化、打开、关闭、暂停、恢复等。

```mermaid
stateDiagram-v2
[*] --> 初始化
初始化 --> 打开
打开 --> 运行中
运行中 --> 暂停
暂停 --> 恢复
恢复 --> 运行中
运行中 --> 关闭
关闭 --> [*]
state 初始化 {
[*] --> OnInit
}
state 打开 {
[*] --> OnOpen
}
state 运行中 {
[*] --> OnUpdate
}
state 暂停 {
[*] --> OnPause
}
state 恢复 {
[*] --> OnResume
}
state 关闭 {
[*] --> OnClose
}
```

**Diagram sources**
- [UIForm.cs](file://Scripts/Runtime/UI/UIForm.cs#L16-L303)
- [UIFormLogic.cs](file://Scripts/Runtime/UI/UIFormLogic.cs#L14-L205)

**Section sources**
- [UIForm.cs](file://Scripts/Runtime/UI/UIForm.cs#L16-L303)
- [UIFormLogic.cs](file://Scripts/Runtime/UI/UIFormLogic.cs#L14-L205)

## 异步加载与事件系统

UI系统采用异步方式加载界面资源，通过事件系统通知操作结果。当界面成功打开时，会触发`OpenUIFormSuccessEventArgs`事件；打开失败时，触发`OpenUIFormFailureEventArgs`事件；界面关闭完成时，触发`CloseUIFormCompleteEventArgs`事件。

```mermaid
sequenceDiagram
participant Client as "客户端代码"
participant UIComponent as "UIComponent"
participant EventComponent as "EventComponent"
UIComponent->>UIComponent : 加载界面资源
alt 加载成功
UIComponent->>EventComponent : Fire(OpenUIFormSuccessEventArgs)
EventComponent-->>Client : 事件通知
else 加载失败
UIComponent->>EventComponent : Fire(OpenUIFormFailureEventArgs)
EventComponent-->>Client : 事件通知
end
UIComponent->>UIComponent : 关闭界面
UIComponent->>EventComponent : Fire(CloseUIFormCompleteEventArgs)
EventComponent-->>Client : 事件通知
```

**Diagram sources**
- [OpenUIFormSuccessEventArgs.cs](file://Scripts/Runtime/UI/OpenUIFormSuccessEventArgs.cs#L15-L93)
- [OpenUIFormFailureEventArgs.cs](file://Scripts/Runtime/UI/OpenUIFormFailureEventArgs.cs#L15-L129)
- [CloseUIFormCompleteEventArgs.cs](file://Scripts/Runtime/UI/CloseUIFormCompleteEventArgs.cs#L16-L106)
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L700-L727)

**Section sources**
- [OpenUIFormSuccessEventArgs.cs](file://Scripts/Runtime/UI/OpenUIFormSuccessEventArgs.cs#L15-L93)
- [OpenUIFormFailureEventArgs.cs](file://Scripts/Runtime/UI/OpenUIFormFailureEventArgs.cs#L15-L129)
- [CloseUIFormCompleteEventArgs.cs](file://Scripts/Runtime/UI/CloseUIFormCompleteEventArgs.cs#L16-L106)
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L700-L727)

## 典型使用示例

以下是一个典型的UI操作流程示例：打开登录界面，用户登录成功后关闭登录界面并打开主界面。

```mermaid
sequenceDiagram
participant Game as "游戏主逻辑"
participant UI as "UIComponent"
participant Login as "登录界面"
participant Main as "主界面"
Game->>UI : OpenUIForm("LoginForm", "Default")
UI-->>Game : 返回序列号
Login->>Game : 用户点击登录
Game->>Login : 验证登录信息
alt 登录成功
Game->>UI : CloseUIForm(loginForm)
Game->>UI : OpenUIForm("MainForm", "Default")
UI-->>Game : 主界面打开成功
else 登录失败
Game->>Login : 显示错误信息
end
```

**Section sources**
- [UIComponent.cs](file://Scripts/Runtime/UI/UIComponent.cs#L492-L622)

## DefaultUIFormHelper扩展点

`DefaultUIFormHelper`是UI系统的一个重要扩展点，它定义了界面实例化、创建和释放的具体行为。开发者可以通过继承`UIFormHelperBase`并重写相应方法来自定义UI的创建流程。

```mermaid
classDiagram
class UIFormHelperBase {
<<abstract>>
+InstantiateUIForm(object) object
+CreateUIForm(object, IUIGroup, object) IUIForm
+ReleaseUIForm(object, object) void
}
class DefaultUIFormHelper {
+InstantiateUIForm(object) object
+CreateUIForm(object, IUIGroup, object) IUIForm
+ReleaseUIForm(object, object) void
}
UIFormHelperBase <|-- DefaultUIFormHelper
UIFormHelperBase <|-- CustomUIFormHelper
UIComponent --> UIFormHelperBase : "使用"
```

**Diagram sources**
- [DefaultUIFormHelper.cs](file://Scripts/Runtime/UI/DefaultUIFormHelper.cs#L15-L72)

**Section sources**
- [DefaultUIFormHelper.cs](file://Scripts/Runtime/UI/DefaultUIFormHelper.cs#L15-L72)