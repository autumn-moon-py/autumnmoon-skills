# ArkTS 学习路径（面向 Flutter 混编）

## 目标

把 ArkTS 从“语法学习”升级到“可支撑 OHOS + Flutter 混编开发”。

## 学习主线

1. ArkTS 语言基础
- 类型系统、接口、类、泛型、异步。
- 目标：能读懂并改写 `EntryAbility.ets`、页面 `*.ets`。

2. ArkUI 页面与状态
- `@Entry`、`@Component`、`@State`、`@StorageLink`、生命周期。
- 目标：能独立写一个 ArkTS 页面并完成路由跳转。

3. Ability 与应用模型
- `UIAbility`、`Want`、`onNewWant`、页面栈与前后台行为。
- 目标：能解释 DeepLink 到页面的完整链路。

4. 系统能力与权限
- 文件、网络、媒体、定位等系统 API 调用模式与权限声明。
- 目标：能把系统能力封装到可复用 ArkTS 模块。

5. 与 Flutter 的混编桥接
- `FlutterAbility`、`FlutterEntry`、`FlutterPage`、`FlutterView`。
- `MethodChannel`、`EventChannel`、`BasicMessageChannel`。
- `PlatformView`/`PlatformViewFactory`。
- 目标：能完成“ArkTS 能力 -> Flutter 页面”双向调用。

## 与仓库样例的对应关系

1. ArkTS 页面与生命周期
- `ohos/flutter_page_sample1`
- `ohos/flutter_page_sample2`

2. ArkTS 能力桥接（Channel）
- `ohos/channel_demo`
- `ohos/test_uni_links/packages/x_uni_links`

3. ArkTS 原生组件嵌入 Flutter
- `ohos/platform_demo`
- `ohos/flutter-pag`

4. ArkTS 多引擎管理
- `add_to_app/multiple_flutters/multiple_flutters_ohos`
- `ohos/multiple_flutters_predraw`

## 4 周 ArkTS 学习计划（可直接执行）

### 第1周：语言 + ArkUI 基础

1. 阅读官方 ArkTS 入门与类型系统文档。
2. 在样例里做小改：
- 修改一个页面布局与状态逻辑。
- 增加一个按钮触发本地状态变化。

验收：
- 能解释 `@State` 与组件重建关系。

### 第2周：Ability + 路由 + 生命周期

1. 阅读 Ability 生命周期与路由机制。
2. 在 `flutter_page_sample1/2` 跟踪：
- `aboutToAppear/aboutToDisappear/onPageShow/onPageHide`

验收：
- 能画出页面显示/隐藏与 Flutter 引擎事件的对应图。

### 第3周：系统能力 + 通道通信

1. 阅读系统能力调用与权限配置。
2. 基于 `channel_demo` 增加一个自定义方法调用（例如设备信息/本地配置读取）。

验收：
- 能说明何时用 MethodChannel，何时用 EventChannel。

### 第4周：PlatformView + 多引擎

1. 阅读 `PlatformView` 与 `FlutterEngineGroup` 相关 API 文档。
2. 在 `platform_demo`、`multiple_flutters_ohos` 中梳理：
- viewId 流转
- attach/detach 时机
- 生命周期同步

验收：
- 能给出多引擎页面黑屏问题的排查步骤。

## ArkTS 与 Flutter 混编映射速查

1. ArkTS `UIAbility` ↔ Flutter 宿主入口
- 简单场景可用 `FlutterAbility` 继承。
- 复杂场景可用 `UIAbility + FlutterEntry`。

2. ArkTS 页面 ↔ Flutter 内容承载
- `FlutterPage({ viewId })` 是承载点。
- `viewId` 必须来自对应 `FlutterView`。

3. ArkTS 能力 ↔ Flutter 调用
- 单次调用：MethodChannel
- 持续事件：EventChannel
- 双向轻量消息：BasicMessageChannel

4. ArkTS 原生组件 ↔ Flutter 视图树
- `PlatformViewFactory` 注册 `viewType`
- Flutter 侧 `OhosView(viewType: ...)` 对应接入

## 官方学习入口

1. ArkTS 指南：
- https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts

2. 本仓库文档入口：
- `ohos/docs/README.md`
- `ohos/docs/04_development/README.md`
- `ohos/docs/11_flutter_api_docs/README.md`
