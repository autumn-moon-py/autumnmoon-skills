# 批次1阅读报告（仅阅读，不含运行验证）

## 范围

1. 文档主线：
- `ohos/docs/03_environment`
- `ohos/docs/04_development`
- `ohos/docs/05_performance`
- `ohos/docs/06_debug`
- `ohos/docs/07_plugin`
- `ohos/docs/08_FAQ`

2. 核心样例源码：
- `ohos/channel_demo`
- `ohos/platform_demo`
- `ohos/flutter_page_sample1`
- `ohos/flutter_page_sample2`
- `add_to_app/books/ohos_books`
- `add_to_app/fullscreen/ohos_fullscreen`
- `add_to_app/prebuilt_module/ohos_using_prebuilt_module`
- `add_to_app/plugin/ohos_using_plugin`
- `add_to_app/multiple_flutters/multiple_flutters_ohos`
- `ohos/multiple_flutters_predraw`

## 结论摘要

1. 开发主线可归纳为 4 种模式：
- 纯 Flutter on OHOS
- FlutterPage 嵌入
- FlutterEntry 混合路由
- FlutterEngineGroup 多引擎

2. Add-to-App 在 OHOS 的稳定骨架高度一致：
- `EntryAbility` 继承 `FlutterAbility`
- `configureFlutterEngine` 中调用 `GeneratedPluginRegistrant.registerWith`
- 页面用 `FlutterPage({ viewId })`
- `oh-package.json5` 通过 `overrides` 指向 `har/*`

3. 通信与原生扩展路径明确：
- Channel：`MethodChannel` / `EventChannel` / `BasicMessageChannel`
- 平台视图：`PlatformViewFactory` + `OhosView(viewType)` + 按 `viewId` 构造 MethodChannel

4. 多引擎样例显示了真实的生命周期要求：
- 页面切换时需要显式 `attach/detach`
- `onPageShow/onPageHide` 需要同步引擎生命周期通道

## 关键代码定位

1. 三通道通信：
- Dart 端：`ohos/channel_demo/lib/main.dart`
- ArkTS 端：`ohos/channel_demo/ohos/entry/src/main/ets/entryability/BatteryPlugin.ets`

2. PlatformView：
- Dart 端：`ohos/platform_demo/lib/custom_ohos_view.dart`
- ArkTS 端：`ohos/platform_demo/ohos/entry/src/main/ets/entryability/CustomView.ets`
- 插件注册：`ohos/platform_demo/ohos/entry/src/main/ets/entryability/CustomPlugin.ets`

3. FlutterEntry 混合路由：
- `ohos/flutter_page_sample1/ohos/entry/src/main/ets/pages/Page1.ets`
- `ohos/flutter_page_sample2/ohos/entry/src/main/ets/pages/Index2.ets`

4. 多引擎：
- 引擎绑定：`add_to_app/multiple_flutters/multiple_flutters_ohos/entry/src/main/ets/pages/EngineBindings.ets`
- 单页示例：`add_to_app/multiple_flutters/multiple_flutters_ohos/entry/src/main/ets/pages/SingleFlutterPage.ets`

## 风险与注意点（阅读结论）

1. `flutter_module` 改动后需重建 HAR，否则宿主不会生效。
2. `ohpm install` 执行层级错误会导致依赖异常。
3. `--local-engine` 在新流程下通常可选，仅在引擎定制场景启用。
4. 多引擎未正确做生命周期同步时，容易出现黑屏或状态错乱。
5. 通道名、viewType、viewId 任一不一致都会导致桥接失效。

## 本批次边界

- 本报告是“阅读理解结论”，不是“运行验证结论”。
- 后续如果转回实测模式，需要补齐每个核心样例的编译与运行结果记录。
