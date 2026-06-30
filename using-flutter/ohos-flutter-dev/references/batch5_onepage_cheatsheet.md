# OHOS + Flutter 单页速查卡

## 1. 四种接入模式

1. 纯 Flutter on OHOS
- 适用：业务基本都在 Flutter。
- 典型：`ohos/animation_demo`。

2. FlutterPage 嵌入
- 适用：宿主已有 ArkTS 工程，快速插入 Flutter 页面。
- 关键：`FlutterAbility + FlutterPage(viewId)`。

3. FlutterEntry 混合路由
- 适用：宿主导航主导，需精细生命周期控制。
- 关键：`FlutterEntry + FlutterView + FlutterPage`。

4. FlutterEngineGroup 多引擎
- 适用：同屏/多页并发 Flutter 实例。
- 关键：`attach/detach`、生命周期通道同步。

## 2. 最常用命令（当前约定）

```powershell
# 统一使用
fvm flutter pub get
fvm flutter run

# 模块更新后重建 HAR
fvm flutter build har --debug

# 应用打包
fvm flutter build hap --debug
fvm flutter build hap --release
```

## 3. 核心类速查

1. 宿主与页面
- `FlutterAbility`：宿主入口能力类
- `FlutterEntry`：混合路由场景入口
- `FlutterPage`：页面渲染承载
- `FlutterView`：真实 view 实例（`viewId` 来源）

2. 引擎
- `FlutterEngine`
- `FlutterEngineGroup`
- `FlutterEngineCache`
- `FlutterEnginePreload`

3. 通道
- `MethodChannel`：请求-响应
- `EventChannel`：流式事件
- `BasicMessageChannel`：双向消息

4. 原生视图
- `PlatformView`
- `PlatformViewFactory`

## 4. 高优先级检查清单

1. HAR 是否最新
- 改 Flutter 模块后是否重新 `build har`。

2. 依赖安装层级是否正确
- `ohpm install` 是否在正确根目录执行。

3. 配置是否对齐
- `oh-package.json5` 的 `overrides` 与 har 路径是否正确。

4. 通道与视图是否一致
- channel name、viewType、viewId 是否完全对齐。

5. 生命周期是否完整
- `aboutToAppear/aboutToDisappear/onPageShow/onPageHide` 是否与引擎状态一致。

## 5. 高价值样例入口

1. 通信：`ohos/channel_demo`
2. 原生视图：`ohos/platform_demo`
3. 混合路由：`ohos/flutter_page_sample1`、`ohos/flutter_page_sample2`
4. 多引擎：`add_to_app/multiple_flutters/multiple_flutters_ohos`
5. 预渲染：`ohos/multiple_flutters_predraw`、`ohos/flutter_it_preload`
6. 插件模板：`ohos/sqflite_helper`、`ohos/test_uni_links/packages/x_uni_links`
7. WebView 桥接：`ohos/flutter_webview_demo`、`ohos/js_dart_demo`

## 6. 文档入口

1. 环境：`ohos/docs/03_environment/`
2. 开发：`ohos/docs/04_development/`
3. 性能：`ohos/docs/05_performance/`
4. 调试：`ohos/docs/06_debug/`
5. 插件：`ohos/docs/07_plugin/`
6. FAQ：`ohos/docs/08_FAQ/`
7. 规范：`ohos/docs/09_specifications/`
8. 升级：`ohos/docs/10_appendix/`
9. API：`ohos/docs/11_flutter_api_docs/`
