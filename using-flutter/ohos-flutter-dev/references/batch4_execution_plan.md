# 批次4执行计划（仅基于阅读结论）

## 目标

把前3批阅读结论转成可落地的学习-实战任务清单，形成“按周推进 + 可验收输出”的执行模板。

## 使用方式

1. 每周按阶段执行，不跨阶段跳任务。
2. 每个任务必须产出“改动文件 + 复现命令 + 结果说明”。
3. 当前模式是“阅读优先”，若后续切实测模式，可直接在每个任务后增加运行验证项。

## 第1周：环境与最小闭环

### 任务1：工程骨架认知

- 阅读并记录：
  - `ohos/docs/03_environment/`
  - `ohos/docs/09_specifications/OpenHarmony-flutter化工程的目录结构.md`
- 验收：
  - 能说清 `app.json5/build-profile.json5/oh-package.json5/module.json5` 作用。

### 任务2：通信三件套认知

- 深读：
  - `ohos/channel_demo/lib/main.dart`
  - `ohos/channel_demo/ohos/entry/src/main/ets/entryability/BatteryPlugin.ets`
- 验收：
  - 解释 `MethodChannel/EventChannel/BasicMessageChannel` 适用场景与调用链。

### 任务3：PlatformView 认知

- 深读：
  - `ohos/platform_demo/lib/custom_ohos_view.dart`
  - `ohos/platform_demo/ohos/entry/src/main/ets/entryability/CustomView.ets`
  - `ohos/platform_demo/ohos/entry/src/main/ets/entryability/CustomPlugin.ets`
- 验收：
  - 能描述 `viewType`、`viewId`、channel 命名一致性要求。

## 第2周：混合开发与生命周期

### 任务1：FlutterPage 模式

- 深读：
  - `ohos/docs/04_development/如何使用 FlutterPage.md`
  - `add_to_app/books/ohos_books/entry/src/main/ets/entryability/EntryAbility.ets`
  - `add_to_app/books/ohos_books/entry/src/main/ets/pages/Index.ets`
- 验收：
  - 写出最小宿主接入步骤（EntryAbility + FlutterPage + viewId）。

### 任务2：FlutterEntry 模式

- 深读：
  - `ohos/flutter_page_sample1`
  - `ohos/flutter_page_sample2`
- 验收：
  - 说明 `aboutToAppear/aboutToDisappear/onPageShow/onPageHide` 与引擎关系。

### 任务3：多引擎模式

- 深读：
  - `add_to_app/multiple_flutters/multiple_flutters_ohos/entry/src/main/ets/pages/EngineBindings.ets`
  - `ohos/multiple_flutters_predraw`
  - `ohos/docs/04_development/如何使用多引擎 FlutterEngineGroup.md`
- 验收：
  - 说明 `attach/detach` 触发时机和状态同步风险。

## 第3周：插件、数据层、WebView

### 任务1：插件实现模板

- 深读：
  - `ohos/sqflite_helper`
  - `ohos/test_uni_links/packages/x_uni_links`
  - `ohos/flutter-pag`
- 验收：
  - 输出一个“插件模板骨架清单”（Dart接口、ETS插件、注册、示例）。

### 任务2：数据层选型

- 深读：
  - `ohos/sqflite_test`
  - `ohos/floor_test`
  - `ohos/ohos_sqlite3_demo`
- 验收：
  - 给出 `sqflite/floor/sqlite3` 的选型建议与风险对比。

### 任务3：WebView 桥接

- 深读：
  - `ohos/flutter_webview_demo`
  - `ohos/js_dart_demo`
- 验收：
  - 输出“页面跳转型 vs JSBridge协议型”对照表。

## 第4周：规范升级与 API 速查

### 任务1：版本升级流程模板

- 深读：
  - `ohos/docs/10_appendix/Flutter-OH版本升级指导.md`
  - `ohos/docs/08_FAQ/`
- 验收：
  - 产出项目升级 checklist（小版本/大版本分开）。

### 任务2：插件结构升级模板

- 深读：
  - `ohos/docs/09_specifications/更新Flutter插件项目结构.md`
- 验收：
  - 产出“插件结构升级最小步骤 + 常见报错处理”。

### 任务3：API 速查表

- 深读：
  - `ohos/docs/11_flutter_api_docs/`
- 验收：
  - 形成 1 页 API 速查（Ability/Entry/Engine/View/Channel/PlatformView）。

## 每周交付格式（统一）

1. 本周阅读范围（路径清单）
2. 本周核心结论（不超过10条）
3. 关键代码位置（文件路径）
4. 风险点与误区
5. 下周计划

## 通过标准

1. 能基于样例独立画出 OHOS + Flutter 调用链。
2. 能按业务场景选择正确接入模式（FlutterPage / FlutterEntry / EngineGroup）。
3. 能说明插件接入、HAR 依赖、生命周期、通道设计四类高频坑。
