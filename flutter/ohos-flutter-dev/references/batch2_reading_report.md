# 批次2阅读报告（仅阅读，不含运行验证）

## 范围

本批次聚焦批次1之外的项目族，按“技术模式”抽样深读：

1. 插件与平台桥接
- `ohos/sqflite_helper`
- `ohos/flutter-pag`
- `ohos/test_uni_links/packages/x_uni_links`

2. WebView 与 JS 互通
- `ohos/flutter_webview_demo`
- `ohos/js_dart_demo`

3. 数据存储路线
- `ohos/sqflite_test`
- `ohos/floor_test`
- `ohos/ohos_sqlite3_demo`

4. 主题与系统适配
- `ohos/ohos_themeAdaptation`
- `ohos/flutter_ohos_theme_fontsizescale`

5. 其余样例组
- 对 `ohos` 目录其余项目完成 README/依赖/关键模式扫描，并回写到能力矩阵。

## 结论摘要

1. OHOS 插件实现模式清晰且统一：
- Dart 侧通过 `MethodChannel` 发起调用。
- OHOS 侧 ETS 插件 `implements FlutterPlugin`，在 `onAttachedToEngine` 建立 channel 并注册 handler。
- 需要 Ability 生命周期的插件（如 `x_uni_links`）会额外实现 `AbilityAware/NewWantListener`。

2. 复杂插件通常是“双通道 + 视图工厂”组合：
- `flutter-pag` 同时使用 `MethodChannel`（控制命令）与 `PlatformViewFactory`（承载渲染视图）。
- 这类插件适合高性能媒体、原生组件渲染、外接纹理场景。

3. WebView 路线分为两类：
- 页面跳转型：`flutter_webview_demo`（`go_router` + `WebViewController` + JS channel）。
- JSBridge 协议型：`js_dart_demo`（注册方法、回调、错误分支，更接近可复用桥接框架）。

4. 数据层有三条清晰路径：
- `sqflite_test`：原生 SQL + CRUD 场景覆盖广，偏底层验证。
- `floor_test`：ORM 路线，结构更工程化，依赖代码生成。
- `ohos_sqlite3_demo`：`sqlite3` 直接 API，适合轻量或自定义 SQL 场景。

5. 系统主题/字体适配存在两种实现风格：
- Provider/ThemeManager 路线：`ohos_themeAdaptation`。
- 直接读 `MediaQuery.platformBrightness/textScaleFactor` 路线：`flutter_ohos_theme_fontsizescale`。

## 关键实现观察

### 1) `sqflite_helper`（插件模板价值高）

- OHOS 侧 `SqfliteHelperPlugin.ets` 展示了：
  - `onAttachedToEngine` / `onDetachedFromEngine`
  - `onAttachedToAbility` 获取 `context`
  - Channel 命令分发（`pathExist`、`makeDir`、`writeDataToFile`）
- Dart 侧 `sqflite_helper_method_channel.dart` 与 ETS 方法一一对应，适合作为“自定义文件/数据库辅助插件”的脚手架。

### 2) `flutter-pag`（复杂插件参考）

- `FlutterPagPlugin.ets` 同时负责：
  - MethodChannel 命令入口（`initPag/start/stop/pause/...`）
  - `registerViewFactory('flutter_pag_plugin', PagFactory(...))`
- 该模式说明：需要原生渲染承载时，不能只靠通道，还要明确 viewType 与原生工厂注册。

### 3) `x_uni_links`（Ability 感知型插件）

- `XUniLinksPlugin.ets` 既是 `FlutterPlugin`，又是 `AbilityAware + NewWantListener`。
- `handleWant()` 把 `want.uri` 同步到：
  - `getInitialLink`（MethodChannel）
  - `events`（EventChannel）
- 这是一种典型“冷启动 + 热启动统一 DeepLink 通路”实现。

### 4) `flutter_webview_demo` 与 `js_dart_demo`

- `flutter_webview_demo`：
  - `go_router` 管理首页/网页页；
  - `WebViewController` 加载本地 `assets/index.html`；
  - JS channel 回调后跳转到 Flutter 页面。
- `js_dart_demo`：
  - 有独立 `js_bridge.dart` 封装注册、调用、回调协议；
  - 更适合沉淀为业务内“Web 容器桥接基类”。

### 5) `floor_test` / `sqflite_test` / `ohos_sqlite3_demo`

- `floor_test`：
  - `@Database` + `database.g.dart` 生成访问层；
  - 对团队协作更友好（实体和 DAO 边界清晰）。
- `sqflite_test`：
  - 覆盖表创建、更新、删除、原始 SQL，验证面广。
- `ohos_sqlite3_demo`：
  - 用 `sqlite3.openInMemory()` 展示最小链路；
  - 可用于验证底层库是否在 OHOS 工程可用。

## 阅读中发现的工程风险点

1. 部分示例更偏“能力演示”而非“生产规范”：
- 例如主题示例中有重复状态更新、调试打印较多等情况。
- 迁移到业务工程时需二次整理结构与状态管理。

2. 插件样例普遍依赖本地/路径引用：
- 引入新工程时要重点核对 `pubspec.yaml` 与 OHOS `har` 依赖路径映射。

3. WebView 与 JSBridge 示例未体现完整安全策略：
- 实际业务应补充域名白名单、脚本注入边界、错误回退策略。

## 本批次产出价值

1. 把“插件型开发”从示例代码提炼为通用模板。
2. 把“数据层选型”从库名对比转成可执行路线（sqflite / floor / sqlite3）。
3. 把“WebView + JSBridge”从 Demo 升级为可复用的双向桥接设计。

## 本批次边界

- 本报告为“阅读理解结论”，尚未做编译运行实测。
- 若转实测模式，建议优先实测：`sqflite_helper`、`flutter-pag`、`x_uni_links`、`flutter_webview_demo`。
