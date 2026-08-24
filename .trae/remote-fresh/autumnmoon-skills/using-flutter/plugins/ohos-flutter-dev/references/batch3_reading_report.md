# 批次3阅读报告（仅阅读，不含运行验证）

## 范围

1. 规范与升级文档
- `ohos/docs/09_specifications/README.md`
- `ohos/docs/09_specifications/OpenHarmony-flutter化工程的目录结构.md`
- `ohos/docs/09_specifications/更新Flutter插件项目结构.md`
- `ohos/docs/10_appendix/Flutter-OH版本升级指导.md`

2. API 文档主入口与关键类
- `ohos/docs/11_flutter_api_docs/README.md`
- `embedding/ohos/FlutterAbility`
- `embedding/ohos/FlutterEntry`
- `embedding/engine/FlutterEngine`
- `embedding/engine/FlutterEngineGroup` + `Options`
- `embedding/engine/FlutterEngineCache`
- `embedding/engine/FlutterEnginePreload`
- `embedding/ohos/FlutterAbilityLaunchConfigs`
- `embedding/ohos/FlutterManager`
- `view/FlutterView`
- `plugin/common/MethodChannel`、`EventChannel`、`BasicMessageChannel`
- `plugin/platform/PlatformView`、`PlatformViewFactory`

## 结论摘要

1. 规范侧的核心是“工程结构收敛 + 插件结构更新”
- 09 文档明确了 OHOS 工程关键配置文件与入口文件位置。
- 插件项目结构升级时，重点在：
  - `oh-package.json5` 的 `name` 与插件名一致
  - `module.json5` 的模块名一致
  - `hvigorfile.ts` 结构调整（`harTasks` 等）
  - 示例工程中 har 引用路径一致

2. 升级侧的核心是“版本切换流程标准化”
- 10 文档给出升级基线：
  - 先做环境核验与备份
  - `flutter clean`、依赖更新、再构建
  - OHOS 平台重新检查签名与打包链路
- 小版本升级与大版本升级关注点不同，大版本需额外检查 breaking changes 与三方库兼容性。

3. API 侧已经形成稳定分层
- Ability/Entry 层：`FlutterAbility`、`FlutterEntry`
- Engine 层：`FlutterEngine`、`FlutterEngineGroup`、`FlutterEngineCache`、`FlutterEnginePreload`
- View 层：`FlutterView`、`FlutterPage`、PlatformView 体系
- Plugin 通道层：`MethodChannel`、`EventChannel`、`BasicMessageChannel`

## API 速查（阅读提炼）

### 1) 宿主入口与生命周期

1. `FlutterAbility`
- 关键扩展点：`configureFlutterEngine(flutterEngine)`、`cleanUpFlutterEngine(flutterEngine)`。
- 用途：宿主 Ability 侧统一接入插件与引擎配置。

2. `FlutterEntry`
- 关键方法：`aboutToAppear()`、`aboutToDisappear()`、`onPageShow()`、`onPageHide()`。
- 用途：混合路由场景手动管理 Flutter 页面生命周期。

### 2) 引擎管理

1. `FlutterEngine`
- 提供 `getLifecycleChannel()`、`getNavigationChannel()`、`getPlugins()`、`getPlatformViewsController()` 等核心能力访问。
- `destroy()` 负责释放资源。

2. `FlutterEngineGroup` + `Options`
- `createAndRunEngineByOptions(options)` 用于多引擎场景快速创建实例。
- `Options` 可设置 entrypoint、route、args、PlatformViewsController。

3. `FlutterEngineCache`
- `put/get/remove/contains/clear`，用于复用已创建引擎。

4. `FlutterEnginePreload`
- `preloadEngine` / `predrawEngine` 用于预创建与预渲染。
- 适合降低混编跳转首帧延迟。

### 3) View 与嵌入

1. `FlutterView`
- 关键方法：`attachToFlutterEngine`、`detachFromFlutterEngine`、`isAttachedToFlutterEngine`、`getId`、首帧监听。
- 是 `FlutterPage(viewId)` 的承载实体。

2. PlatformView 体系
- `PlatformViewFactory.create(context, viewId, args)`
- `PlatformView.getView()/dispose()` 等生命周期接口
- 用途：在 Flutter 页面嵌入 ArkTS 原生组件。

### 4) 通道通信

1. `MethodChannel`
- `invokeMethod`、`setMethodCallHandler`，用于请求-响应调用。

2. `EventChannel`
- `setStreamHandler`，用于持续事件流。

3. `BasicMessageChannel`
- `send`、`setMessageHandler`，用于双向消息。

## 与样例的对应关系

1. `FlutterAbility/FlutterEntry/FlutterPage`
- 对应 `flutter_page_sample1/2`、`add_to_app/*_ohos`。

2. `FlutterEngineGroup/FlutterEnginePreload`
- 对应 `add_to_app/multiple_flutters/multiple_flutters_ohos`、`ohos/multiple_flutters_predraw`、`ohos/flutter_it_preload`。

3. Channel 与 PlatformView
- 对应 `channel_demo`、`platform_demo`、`flutter-pag` 等插件/组件样例。

## 规范化建议（从文档反推）

1. 新建或升级工程时，先校准“目录与配置”再写业务
- 优先检查 `app.json5`、`build-profile.json5`、`oh-package.json5`、`module.json5`。

2. 插件改造时，先统一 name 与模块结构
- 避免出现 hvigor 构建路径异常与 har 引用错误。

3. 混编性能优化优先考虑 preload/predraw
- 首先评估页面跳转延迟，再决定是否引入预渲染流程。

## 本批次边界

- 本报告为阅读总结，不代表实际构建/运行结果。
- API 文档部分为接口速查视角，未覆盖全部子模块细节。
