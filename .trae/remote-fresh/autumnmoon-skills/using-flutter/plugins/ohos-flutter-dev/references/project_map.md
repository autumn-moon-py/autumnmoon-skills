# 项目学习地图

本地图基于 `flutter_samples` 的 OHOS 与 Add-to-App 样例整理，目标是“先跑通，再混合，再工程化”。

## 第一阶段：基础可运行

1. `ohos/animation_demo`
- 目标：验证环境可运行，熟悉最小 Flutter on OHOS 工程结构。

2. `ohos/channel_demo`
- 目标：掌握 `MethodChannel`、`BasicMessageChannel`、`EventChannel` 三种通信模型。

3. `ohos/platform_demo`
- 目标：掌握 `OhosView + PlatformViewFactory + MethodChannel` 的双向通信。

## 第二阶段：页面混合开发

1. `ohos/flutter_page_sample1`
- 目标：理解 `FlutterEntry + FlutterView + FlutterPage` 生命周期串联。

2. `ohos/flutter_page_sample2`
- 目标：理解混合路由、插件延迟注册、页面参数传递。

3. `add_to_app/books/ohos_books`
- 目标：理解宿主应用通过 HAR 接入 Flutter 模块的标准骨架。

## 第三阶段：复杂宿主场景

1. `add_to_app/multiple_flutters/multiple_flutters_ohos`
- 目标：掌握 `FlutterEngineGroup` 多引擎 attach/detach 与状态同步。

2. `ohos/multiple_flutters_predraw`
- 目标：掌握多引擎预渲染与页面切换细节。

## 第四阶段：插件与三方库

1. `ohos/docs/07_plugin/developing-an-ohos-plugin-using-flutter.md`
- 目标：掌握 OHOS 插件开发流程（Dart 接口 + ETS 实现 + 示例验证）。

2. `ohos/docs/07_plugin/ohos平台适配flutter三方库指导.md`
- 目标：掌握现有 Flutter 三方库迁移到 OHOS 的适配路径。

## 第五阶段：工程化与上线

1. `ohos/docs/06_debug/`
- 目标：掌握 Dart/Native 调试、符号化、崩溃排查。

2. `ohos/docs/08_FAQ/`
- 目标：规避 `ohpm/hvigor/local-engine/versionName` 等高频坑。

3. `ohos/docs/05_performance/`
- 目标：建立性能定界、线程分析、帧追踪与内存优化流程。
