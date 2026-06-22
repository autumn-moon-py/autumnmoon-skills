# ArkTS + Flutter 协同开发流程

## 目标

在 OpenHarmony 原生（ArkTS）与 Flutter 模块之间建立稳定的开发、调试、验证闭环。

## 设计分层

1. Flutter 层
- 跨平台 UI、页面状态、通用业务逻辑。

2. ArkTS 层
- 系统能力、权限、设备接口、宿主生命周期和导航。

3. 桥接层
- Channel/PlatformView/插件注册，定义清晰的数据契约。

## 通信选型

1. `MethodChannel`
- 适用：单次调用、请求-响应。
- 实例：`ohos/channel_demo` 电量获取。

2. `EventChannel`
- 适用：持续事件流。
- 实例：`ohos/channel_demo` 事件推送。

3. `BasicMessageChannel`
- 适用：轻量双向消息/对象编解码。
- 实例：`ohos/channel_demo` 奇偶回包示例。

4. `PlatformView + OhosView`
- 适用：将 ArkTS 原生组件嵌入 Flutter 视图树。
- 实例：`ohos/platform_demo`。

## 宿主集成模式

1. 简单嵌入
- `EntryAbility extends FlutterAbility`
- 页面通过 `FlutterPage({ viewId })` 承载内容。
- 代表：`add_to_app/books/ohos_books`。

2. 混合路由
- `UIAbility + FlutterEntry + FlutterView + FlutterPage`
- 页面生命周期显式透传到 Flutter 引擎。
- 代表：`ohos/flutter_page_sample1`、`ohos/flutter_page_sample2`。

3. 多引擎
- `FlutterEngineGroup` 管理多个引擎实例。
- 必须显式 `attach/detach`，并维护 `appIsResumed/appIsPaused` 等生命周期事件。
- 代表：`add_to_app/multiple_flutters/multiple_flutters_ohos`。

## 联调顺序（建议固定）

1. Flutter 层先自测：参数与返回值结构正确。
2. ArkTS 层补齐：权限、系统能力、异常分支。
3. 宿主页面联调：进入/返回/前后台切换。
4. 多实例或复杂场景回归：重复打开、销毁、重进。
5. 打包验证：`debug` 与 `release` 各跑一轮。

## 高风险检查点

1. HAR 更新
- Flutter 模块变更后，要重新 `fvm flutter build har` 并替换宿主依赖。

2. 依赖安装层级
- `ohpm install` 在项目根执行，避免 entry 层误操作导致依赖不全。

3. 生命周期一致性
- `aboutToAppear/aboutToDisappear` 与 `onPageShow/onPageHide` 必须与引擎状态同步。

4. 通道与视图 ID 对齐
- Channel name、viewType、viewId 一旦不一致，常见表现是白屏或调用无响应。

## 常用命令

```powershell
# Flutter 依赖安装
fvm flutter pub get

# 启动应用
fvm flutter run

# 静态检查（可选）
fvm flutter analyze
```

## 文档入口

- 仓库总览：`README.md`
- OHOS 文档索引：`ohos/docs/README.md`
- 功能开发：`ohos/docs/04_development/README.md`
- 插件开发：`ohos/docs/07_plugin/README.md`
- ArkTS 官方指南：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
