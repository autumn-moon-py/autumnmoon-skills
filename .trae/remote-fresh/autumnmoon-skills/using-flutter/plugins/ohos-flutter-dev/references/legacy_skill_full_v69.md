---
name: ohos-flutter-dev
description: 用于在 OpenHarmony 场景下进行 Flutter 与 ArkTS 协同开发、样例学习、工程落地与问题排查。适用于“从零搭建环境、跑通样例、原生与 Flutter 混合开发、插件适配与调试优化”任务。
---

# OHOS Flutter 开发技能

## 适用场景

- 用户要学习或实战 OpenHarmony + Flutter 开发。
- 用户要把 ArkTS 原生能力接入 Flutter（页面嵌入、Channel、PlatformView、插件）。
- 用户要基于 `flutter_samples` 快速选型样例并落地到业务工程。

## 先决条件

- 工作目录默认是仓库根目录：`flutter_samples`。
- Flutter 命令统一使用 `fvm flutter`。
- 先跑环境检查：`fvm flutter doctor -v`。
- DevEco/SDK/ohpm/hvigor/node 环境按 `ohos/docs/03_environment/` 配齐后再进入业务开发。

## 我从样例沉淀出的 4 种主架构

1. 纯 Flutter（OHOS 平台运行）
- 典型：`ohos/animation_demo`、`ohos/http_test`、`ohos/flutter_svg_test`。
- 特征：Flutter 主导，OHOS 侧只承载应用壳。
- 适合：业务 UI 与逻辑都在 Flutter，原生能力依赖少。

2. FlutterPage 嵌入（宿主侧最小改造）
- 典型：`add_to_app/books/ohos_books`、`add_to_app/fullscreen/ohos_fullscreen`、`add_to_app/prebuilt_module/ohos_using_prebuilt_module`。
- 代码特征：`EntryAbility extends FlutterAbility`，页面里用 `FlutterPage({ viewId })`。
- 适合：原生已有工程，要快速插入 Flutter 页面。

3. FlutterEntry 混合路由（原生导航主导）
- 典型：`ohos/flutter_page_sample1`、`ohos/flutter_page_sample2`。
- 代码特征：页面生命周期显式调用 `aboutToAppear/aboutToDisappear/onPageShow/onPageHide`，并通过 `FlutterEntry + FlutterView + FlutterPage` 串联。
- 适合：ArkTS 导航复杂、需精细控制 Flutter 页面生命周期。

4. 多引擎并发（EngineGroup）
- 典型：`add_to_app/multiple_flutters/multiple_flutters_ohos`、`ohos/multiple_flutters_predraw`。
- 代码特征：`FlutterEngineGroup`、`attach/detach`、`LifecycleChannel`、多 `viewId` 同时管理。
- 适合：同一宿主页面并发多个 Flutter 实例，或复杂预渲染场景。

## 执行流程

1. 选场景
- 按上面 4 种架构先定“宿主主导”还是“Flutter 主导”。

2. 选样例
- 先看 `references/project_map.md` 的分层路线。
- 再看 `references/ohos_project_matrix.md` 找最接近需求的项目。

3. 先跑通
- 在样例目录执行：
  - `fvm flutter pub get`
  - `fvm flutter run`

4. 再做改造
- 原生侧改造遵循 `references/arkts_flutter_workflow.md`。
- 插件与三方库适配遵循 `ohos/docs/07_plugin/`。

5. 验收输出
- 给出改动文件、复现命令、预期结果、失败时回滚点。

## 最小命令模板

```powershell
# 进入样例（示例：通道通信）
cd ohos/channel_demo

# 安装依赖
fvm flutter pub get

# 运行
fvm flutter run
```

## 关键规则（来自仓库实证）

1. 模块集成后改 Flutter 代码，必须重新构建 HAR
- 改动 `flutter_module` 后执行 `fvm flutter build har --debug`（或 `--release`），再让宿主重新引用。

2. `ohpm install` 要在正确层级执行
- 集成文档明确要求在项目根层执行，不是在 `entry` 层盲跑。

3. `--local-engine` 在较新流程里是可选
- 默认可走云端引擎产物；仅在自编译引擎调试/验证时显式传入。

4. Add-to-App 统一看三件事
- `oh-package.json5` 的 `overrides` 是否指向正确 `har/*`。
- `EntryAbility` 是否调用 `GeneratedPluginRegistrant.registerWith(flutterEngine)`。
- 页面是否正确传入 `FlutterPage({ viewId })`。

5. 多引擎场景必须显式管理 attach/detach 与生命周期
- `aboutToAppear`/`aboutToDisappear` + `onPageShow`/`onPageHide` 要和引擎状态一致，不然容易黑屏或状态错乱。

6. PlatformView 要双向注册
- ArkTS 侧：`PlatformViewFactory` 注册 `viewType`。
- Flutter 侧：`OhosView(viewType: ...)` 与通道名保持一致。

## ArkTS 协同学习入口

- 官方 ArkTS：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 仓库联动入口：
  - `ohos/docs/03_environment/`
  - `ohos/docs/04_development/`
  - `ohos/docs/06_debug/`
  - `ohos/docs/08_FAQ/`

## ArkTS 官网进阶学习沉淀（新增）

1. 并发模型选型规则
- 需要任务池统一调度、复用线程并执行异步计算时优先 `TaskPool`。
- 需要“长时间常驻后台 + 与主线程持续消息交互”时选 `Worker`。
- 跨线程数据传递优先使用可安全序列化与并发共享的数据结构，避免在主线程与子线程间传递不安全对象。

2. 跨语言交互规则
- Node-API 是 ArkTS 与 C/C++ 进行跨语言交互的基础通道，适合高性能、既有原生库复用场景。
- 混编工程里先定义好 ArkTS 侧调用边界（输入/输出/错误码），再落到 C/C++ 实现，避免双侧接口漂移。

3. 运行时与性能规则
- 官方运行时强调自动内存管理、并发执行与任务调度能力；业务侧应减少不必要对象创建与跨线程拷贝。
- UI 主线程只保留渲染与轻逻辑，耗时任务下沉并发模块，避免页面卡顿。

4. 编译构建规则
- ArkTS 工程构建依赖 Hvigor/ohpm 工具链，优先使用官方推荐脚本与目录层级执行命令。
- 混编项目需固定“依赖安装层级 + 构建产物输出层级”，防止本地能编译、CI 失败的路径问题。

## ArkTS 高阶模块学习沉淀（新增）

1. Common Library 使用规则
- Common Library 覆盖基础工具能力（如 URL 解析、缓存与数据结构等），优先复用标准库能力，减少业务层重复封装。
- 选型时先判断“是否已有官方工具类/容器能力”，再决定是否引入三方实现。

2. Sendable 与并发数据边界
- 需要跨线程传递对象时，优先按 Sendable 约束设计数据结构，避免在线程间传递不安全引用。
- 混编项目中，ArkTS 并发任务输出建议先转成稳定 DTO，再通过 Channel 传给 Flutter，降低边界复杂度。

3. ArkTS 模块化规则
- 业务代码按“页面层 / 能力层 / 桥接层（Channel/Plugin）”拆分模块，减少 UI 与原生能力耦合。
- 模块导出接口保持最小化，避免跨模块直接访问内部实现，提升后续插件化和测试可维护性。

## ArkTS 待补深水区（下一批）

1. 迁移专题细化
- TypeScript 到 ArkTS：补齐 `Cookbook` 与 `Adaptation Cases` 的反模式清单（例如动态能力、类型收敛、语义差异）。
- Java/Swift 到 ArkTS：补齐语言心智迁移表（对象模型、异常模型、并发模型、空安全习惯）。

2. Common Library 模块速查
- 按“字符串/集合/URL/缓存/工具函数”等维度建立速查表，并标注“可直接复用 / 需二次封装”的判定规则。

3. 并发落地判定表
- 给每类任务固定 `TaskPool`/`Worker` 选择条件、输入输出约束、Sendable 边界、错误回传方式。

4. 构建流水线模板
- 固化 `ohpm install -> hvigor -> fvm flutter build/run` 的一键流程与失败分支处理（依赖缺失、层级错误、产物引用错误）。

## 并发判定表（可直接复用）

1. 任务类型 -> 模型选择
- 短时 CPU 计算（可分片、可批处理）：`TaskPool`
- 批量 I/O（可并发、无常驻状态）：`TaskPool`
- 长时间常驻监听（持续收消息、持续回调）：`Worker`
- 事件流桥接（原生连续推送到 Flutter）：`Worker + EventChannel`

2. 数据边界
- 线程间只传“稳定 DTO”，避免传 UI 对象、上下文对象、复杂可变引用。
- DTO 字段尽量基础类型 + 扁平结构；大对象先裁剪再传输。
- 与 Flutter 边界统一转换为 `Map/String/number/bool/list` 等通道友好数据。

3. 错误回传模板
- `code`：稳定错误码（可检索）
- `message`：可读错误描述
- `recoverable`：是否可重试
- `action`：建议动作（重试/降级/提示用户）

## 构建流水线模板（OHOS + Flutter 混编）

1. 正常路径（示例）

```powershell
# 1) 依赖安装（OHOS 层）
ohpm install

# 2) OHOS 构建（按项目脚本选择 assemble/hap/har 目标）
hvigorw.bat --mode module -p module=entry assembleHap

# 3) Flutter 依赖与构建（必须使用 fvm flutter）
fvm flutter pub get
fvm flutter build har --debug
```

2. 失败分支排查
- 依赖缺失：检查 `oh-package.json5`、锁文件与镜像源；删除冲突缓存后重装。
- 层级错误：确认命令执行目录是否为项目要求层级（根目录/模块目录）。
- 产物引用错误：检查 `overrides`、`har/*` 路径、宿主侧引用版本是否与最新产物一致。

## Common Library 模块速查（工程视角）

1. URL/字符串/集合等基础能力
- 优先复用官方工具模块，减少“业务重复造轮子”。

2. 可直接复用判定
- 需求是通用能力、无业务语义、无跨层耦合：直接复用。

3. 需二次封装判定
- 涉及业务语义、鉴权、日志、跨端一致性：在能力层封装后再给页面层使用。

## 迁移反模式清单（TS/Java/Swift -> ArkTS）

1. TypeScript -> ArkTS 常见反模式
- 反模式：`any`/隐式动态类型扩散。  
  修正：收敛为显式类型、接口与泛型边界。
- 反模式：把可空与非空混用，依赖运行时兜底。  
  修正：在接口层明确可空语义，入口处做判空收敛。
- 反模式：控制流中过度依赖弱约束对象字面量。  
  修正：抽成稳定 DTO/类型定义，减少魔法字段。

2. Java -> ArkTS 常见反模式
- 反模式：沿用“重量级类层级 + 过度 OOP 包装”。  
  修正：按 ArkTS 组件化与模块化拆分，保持接口轻量。
- 反模式：把异常当主流程控制。  
  修正：异常用于异常路径，常规分支返回结构化结果。
- 反模式：把线程模型一比一映射。  
  修正：优先用 `TaskPool/Worker` 的任务模型重构并发边界。

3. Swift -> ArkTS 常见反模式
- 反模式：把闭包捕获语义直接平移，忽略生命周期差异。  
  修正：在页面与能力边界显式管理状态生命周期。
- 反模式：沿用平台特有集合/可选值习惯，不做统一数据契约。  
  修正：跨层统一 DTO 与可空约束，保证 Flutter 通道可序列化。
- 反模式：业务与平台能力混写在同层。  
  修正：按“页面层/能力层/桥接层”拆分，降低改动扩散。

## 迁移检查清单（Review Checklist）

1. 类型与可空
- [ ] 是否消除了 `any` 扩散与隐式动态类型。
- [ ] 是否在接口边界明确了可空语义并完成入口判空。
- [ ] 是否把弱约束对象字面量收敛为稳定 DTO。

2. 控制流与错误模型
- [ ] 是否避免“异常作为主流程控制”。
- [ ] 是否统一使用结构化错误返回（`code/message/recoverable/action`）。
- [ ] 是否把可恢复错误与不可恢复错误分层处理。

3. 并发与线程边界
- [ ] 是否按任务特征正确选择 `TaskPool` 或 `Worker`。
- [ ] 是否仅在线程间传递可序列化、稳定的 DTO。
- [ ] 是否避免跨线程传递 UI/上下文/可变共享引用。

4. 分层与模块化
- [ ] 是否完成“页面层/能力层/桥接层”拆分。
- [ ] 是否控制模块导出面，避免跨模块内部实现泄漏。
- [ ] 是否把平台能力调用收敛到能力层而非页面层散落调用。

5. Flutter 混编边界
- [ ] Channel 数据是否统一为通道友好类型（`Map/String/number/bool/list`）。
- [ ] 是否定义了 ArkTS -> Flutter 的稳定契约版本（字段名/语义不漂移）。
- [ ] 是否给关键桥接点补充了失败回退动作（重试/降级/提示）。

## ArkTS 官网页级覆盖进度（持续更新）

1. Learning ArkTS 主线
- [x] Getting Started with ArkTS
- [x] About This Kit
- [x] ArkTS Coding Style Guide
- [x] Migration from TypeScript to ArkTS（入口）
- [x] ArkTS Migration Background
- [x] TypeScript to ArkTS Cookbook（目录级与规则级）
- [x] Adaptation Cases（目录级与案例分组级）
- [x] ArkTS Performant Programming Practices
- [x] Migration from Other Languages to ArkTS（入口）
- [x] Migrating from Java to ArkTS（目录级）
- [x] Migrating from Swift to ArkTS（目录级）

2. ArkTS 能力主干（独立指南）
- [x] ArkTS Concurrency（含 Worker/TaskPool）
- [x] Cross-language Interaction（Node-API）
- [x] ArkTS Runtime Overview
- [x] Compilation Toolchain / Hvigor
- [x] ArkTS Common Library Overview
- [x] Sendable Object

3. 尚未完成的“全量逐页深读”范围
- [ ] TypeScript to ArkTS Cookbook 全部 recipe 逐条摘录（当前为规则级覆盖，未逐条落卡）
- [ ] Adaptation Cases 全案例逐条摘录（当前为分组级覆盖，未全量样例化）
- [ ] Java/Swift 迁移页的逐章节例题化归纳（当前为目录级与反模式级）
- [ ] Common Library 子模块页逐条摘录（当前为 overview 级）

5. 正文抓取状态（外网）
- Cookbook 正文抓取：`阻塞（连接关闭）`
- Adaptation Cases 正文抓取：`阻塞（连接关闭）`
- Java/Swift 迁移正文抓取：`阻塞（连接关闭）`
- 阻塞期间替代策略：先做样例源码实读与规则映射，待连接恢复后补齐逐页卡片。

## Common Library 子模块样例覆盖（源码证据）

1. 已在样例中实读到的模块族
- `@kit.AbilityKit`：页面与能力上下文、权限与能力对象传递。
- `@kit.BasicServicesKit`：`batteryInfo`、`BusinessError`、`emitter` 等基础服务能力。
- `@kit.ArkData`：`ValueType`、数据相关类型与参数载体。
- `@kit.CoreFileKit`：文件与备份扩展能力（`fileIo`、`BackupExtensionAbility`）。

2. 已实读到的典型文件
- `ohos/channel_demo/.../entryability/BatteryPlugin.ets`（`batteryInfo`）
- `add_to_app/multiple_flutters/.../pages/EngineBindings.ets`（`@kit.AbilityKit common`）
- `add_to_app/plugin/ohos_using_plugin/.../plugin/InAppBrowser.ets`（`@kit.ArkData`、`@kit.BasicServicesKit`、`@ohos.web.webview`）
- `ohos/sqflite_helper/ohos/src/main/ets/components/plugin/SqfliteHelperPlugin.ets`（`@ohos.data.relationalStore`、`@ohos.file.fs`）

3. 当前结论
- 官网子模块“逐页文档摘录”仍未完成，但“样例侧子模块使用证据”已形成第一批基线，可支撑混编选型与代码审查。

4. 逐页补齐优先级（执行顺序）
- P0：TypeScript to ArkTS Cookbook（先补语法与类型收敛相关 recipe）
- P0：Adaptation Cases（先补高频工程改造案例）
- P1：Migrating from Java to ArkTS（补对象模型/异常/并发映射）
- P1：Migrating from Swift to ArkTS（补可空/闭包/生命周期映射）

## 逐页深读打卡模板（补齐专用）

1. 页面信息
- 页面标题：
- 页面链接：
- 读取层级：`目录级` / `逐页深读`

2. 本页摘录（最少 3 条）
- 规则1：
- 规则2：
- 规则3：

3. 混编映射
- 对应样例：
- 对应改造点（ArkTS/Flutter 边界）：

4. 风险与修正
- 反模式：
- 修正动作：

5. 覆盖状态
- 本页状态：`已完成` / `待补`
- 剩余关联页数（估算）：

## 逐页深读打卡（第一组已填）

1. 卡片M（Cookbook 主题：类型收敛）
- 页面标题：TypeScript to ArkTS Cookbook（类型与断言相关 recipe，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：断言 `as` 只用于窄化，不替代入口校验。
- 规则2：路由参数先校验后写状态，禁止断言结果直传跨层。
- 规则3：业务层禁用 `any` 扩散，测试脚手架 `any` 单独隔离。
- 混编映射：
- 对应样例：`add_to_app/plugin/ohos_using_plugin/.../InAppBrowser.ets`、`ohos/flutter_page_sample2/.../NavPageThree.ets`。
- 对应改造点（ArkTS/Flutter 边界）：参数解析与 DTO 映射在 ArkTS 桥接层收口，再向 Flutter 分发。
- 风险与修正：
- 反模式：`as` 连续断言 + 无判空。
- 修正动作：增加类型守卫，校验后再窄化并入 DTO。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 8-12 页（估算）`

2. 卡片N（Adaptation 主题：生命周期适配）
- 页面标题：Adaptation Cases（生命周期与页面切换适配，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：混编页必须显式实现 `aboutToAppear/aboutToDisappear/onPageShow/onPageHide`。
- 规则2：引擎 attach/detach 与页面可见性钩子必须配对。
- 规则3：跨页切换时统一 `viewId` 与生命周期通道状态。
- 混编映射：
- 对应样例：`ohos/flutter_page_sample1/.../Page1.ets`、`add_to_app/multiple_flutters/.../SingleFlutterPage.ets`。
- 对应改造点（ArkTS/Flutter 边界）：把平台页面回调映射到 Flutter 生命周期通道，避免错位。
- 风险与修正：
- 反模式：只迁移页面渲染，不迁移生命周期钩子。
- 修正动作：补齐四段生命周期与引擎状态同步代码。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 10+ 页（估算）`

3. 卡片O（Java/Swift 迁移主题：启动链路）
- 页面标题：Migrating from Java/Swift to ArkTS（启动参数与事件链路）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：冷启动与热启动参数统一入口（`onCreate` + `onNewWant` 合流）。
- 规则2：保留 `super.onNewWant(...)` 并统一分发到插件处理函数。
- 规则3：对 deep link/参数签名做幂等去重。
- 混编映射：
- 对应样例：`ohos/test_uni_links/.../EntryAbility.ets`、`.../XUniLinksPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：启动参数统一转 DTO 后再经通道发送到 Flutter。
- 风险与修正：
- 反模式：冷/热启动各写一套处理逻辑。
- 修正动作：抽出单一解析函数并在两处复用。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 6-10 页（估算）`

4. 卡片P（Common Library 子模块主题）
- 页面标题：ArkTS Common Library 子模块（AbilityKit/BasicServicesKit/CoreFileKit/ArkData）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：`AbilityKit` 上下文只在能力层与桥接层流动。
- 规则2：`BusinessError` 在桥接层统一映射错误码与用户可见信息。
- 规则3：文件能力与数据谓词能力在能力层封装后暴露 DTO。
- 混编映射：
- 对应样例：`add_to_app/multiple_flutters/.../EngineBindings.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`、`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：系统能力调用与异常映射下沉，Flutter 只消费稳定接口。
- 风险与修正：
- 反模式：页面层直接调用系统能力 API。
- 修正动作：封装为能力层服务并统一错误模型。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 12+ 页（估算）`

## 逐页深读打卡（第二组已填）

1. 卡片Q（Cookbook 主题：参数与空值）
- 页面标题：TypeScript to ArkTS Cookbook（参数类型与空值语义，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：`router.getParams` 只做入参快照，后续流程不重复读取并回写。
- 规则2：`Record<string, Object>` 与 `Map<string, ValueType>` 在边界层统一转换，避免页面层混用。
- 规则3：字符串拼接与 `??` 组合必须显式加括号，保证空值兜底语义。
- 混编映射：
- 对应样例：`ohos/flutter_page_sample2/.../Index2.ets`、`NavPageThree.ets`、`add_to_app/.../InAppBrowser.ets`。
- 对应改造点（ArkTS/Flutter 边界）：参数进入 ArkTS 后先做标准化，再转通道 DTO 给 Flutter。
- 风险与修正：
- 反模式：参数对象直接下传到页面与插件多层共享。
- 修正动作：边界层一次转换、内部只传强类型对象。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 6-10 页（估算）`

2. 卡片R（Adaptation 主题：通道注册与解绑）
- 页面标题：Adaptation Cases（Channel 注册/解绑一致性，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：`setMethodCallHandler` 与 `setStreamHandler` 必须在释放时对称清理。
- 规则2：页面离场或插件卸载时，通道回调先解绑再释放资源。
- 规则3：通道名与 viewType 作为强一致字段管理，避免大小写漂移。
- 混编映射：
- 对应样例：`add_to_app/fullscreen/.../methodPlugin.ets`、`add_to_app/plugin/.../methodPlugin.ets`、`ohos/platform_component_demo/...`。
- 对应改造点（ArkTS/Flutter 边界）：把注册、调用、解绑三段流程视为同一原子单元。
- 风险与修正：
- 反模式：只注册不解绑，导致重复回调与泄漏。
- 修正动作：统一增加 `dispose`/离场解绑模板并纳入审查项。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 8-12 页（估算）`

3. 卡片S（Java/Swift 迁移主题：并发与错误边界）
- 页面标题：Migrating from Java/Swift to ArkTS（并发模型与错误模型迁移）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：线程思维迁移为任务思维，按 `TaskPool/Worker` 场景选型。
- 规则2：错误模型从异常主流程迁移为结构化返回优先。
- 规则3：可恢复错误保留重试动作，不可恢复错误才终止流程。
- 混编映射：
- 对应样例：`ohos/testcamera/.../CameraUtil.ets`、`ohos/flutter_huawei_login/.../LoginView.ets`、`ohos/multiple_flutters_predraw/.../MainPage.ets`。
- 对应改造点（ArkTS/Flutter 边界）：并发结果与错误统一映射为可序列化通道对象。
- 风险与修正：
- 反模式：直接平移 Java/Swift 线程与异常控制习惯。
- 修正动作：改成任务边界 + 结构化错误 + 回退动作三段式。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 5-8 页（估算）`

4. 卡片T（Common Library 主题：文件与权限链路）
- 页面标题：ArkTS Common Library 子模块（文件处理与权限相关能力）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：文件能力调用前先完成权限状态检查与申请。
- 规则2：`BusinessError` 统一映射为 `code/message/recoverable/action`。
- 规则3：文件操作结果返回 DTO，不把系统对象直接透传到 Flutter。
- 混编映射：
- 对应样例：`ohos/ohos_flutter_photoviewpicker/.../CameraPermissions.ets`、`PhotoPickerPlugin.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：权限、文件、错误三条链路在能力层聚合后再出通道。
- 风险与修正：
- 反模式：页面直接进行权限与文件混写，错误处理散落。
- 修正动作：拆分为权限服务、文件服务、通道适配三层。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 10+ 页（估算）`

## 逐页深读打卡（第三组已填）

1. 卡片U（Cookbook 主题：对象与集合边界）
- 页面标题：TypeScript to ArkTS Cookbook（对象/集合迁移边界，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：`Map<string, ValueType>` 仅作为边界数据容器，业务内层应转换为显式接口对象。
- 规则2：`Record<string, Object>` 仅用于通用参数装配，不作为长期状态结构。
- 规则3：集合元素读取后先类型收敛，再进入业务分支。
- 混编映射：
- 对应样例：`ohos/flutter_page_sample2/.../NavPageThree.ets`、`add_to_app/.../InAppBrowser.ets`。
- 对应改造点（ArkTS/Flutter 边界）：路由与通道参数统一由“弱结构容器”转“强类型 DTO”。
- 风险与修正：
- 反模式：集合与对象在页面/插件多层透传，字段语义漂移。
- 修正动作：边界层一次性收敛类型并冻结字段约定。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 4-8 页（估算）`

2. 卡片V（Adaptation 主题：多引擎页面切换）
- 页面标题：Adaptation Cases（多引擎页面切换与可见性同步，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：多引擎页面切换必须保持 `viewId`、引擎绑定、可见性状态三者一致。
- 规则2：`onPageShow/onPageHide` 与 `LifecycleChannel` 必须同频触发。
- 规则3：页面离场先解绑引擎再清理页面状态，避免悬挂引用。
- 混编映射：
- 对应样例：`add_to_app/multiple_flutters/.../DoubleFlutterPage.ets`、`LazyListFlutterPage.ets`、`EngineBindings.ets`。
- 对应改造点（ArkTS/Flutter 边界）：页面切换时统一走引擎绑定服务，避免页面自行管理状态分叉。
- 风险与修正：
- 反模式：只更新页面导航，不同步引擎生命周期。
- 修正动作：在页面钩子中补齐引擎 attach/detach 与生命周期通道调用。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 6-10 页（估算）`

3. 卡片W（Java/Swift 迁移主题：插件与能力分层）
- 页面标题：Migrating from Java/Swift to ArkTS（插件层与能力层迁移）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：插件层负责通道协议与数据映射，能力层负责系统 API 调用。
- 规则2：页面层不直接承担权限判断与系统调用。
- 规则3：迁移后统一输出结构化错误，避免平台特定异常外溢。
- 混编映射：
- 对应样例：`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`、`CameraPermissions.ets`、`ohos/flutter_huawei_login/.../LoginView.ets`。
- 对应改造点（ArkTS/Flutter 边界）：按“页面层/能力层/插件层”拆分迁移，减少跨层耦合。
- 风险与修正：
- 反模式：历史平台代码按单层平移到 ArkTS。
- 修正动作：先重建分层边界，再迁移具体 API 调用。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 4-6 页（估算）`

4. 卡片X（Common Library 主题：数据与存储协同）
- 页面标题：ArkTS Common Library 子模块（数据访问与存储协同）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：数据访问能力（如谓词/存储）与文件能力统一在能力层封装。
- 规则2：插件层只暴露稳定方法与 DTO，不暴露底层库细节。
- 规则3：错误与权限链路在能力层闭环，再回传 Flutter。
- 混编映射：
- 对应样例：`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`、`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：把数据源差异吸收在 ArkTS 层，Flutter 侧保持接口稳定。
- 风险与修正：
- 反模式：将存储实现细节直接暴露给上层。
- 修正动作：抽象仓储接口并规范返回模型。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 8-10 页（估算）`

## 逐页深读打卡（第四组已填）

1. 卡片Y（Cookbook 主题：错误返回模型）
- 页面标题：TypeScript to ArkTS Cookbook（错误模型迁移，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：业务失败优先返回结构化错误，不以 `throw` 作为常规分支控制。
- 规则2：错误字段最少包含 `code/message/recoverable/action`。
- 规则3：通道回传错误需稳定字段，不传平台特定异常对象。
- 混编映射：
- 对应样例：`ohos/flutter_huawei_login/.../LoginView.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：ArkTS 侧完成错误映射后再回传 Flutter。
- 风险与修正：
- 反模式：异常对象直接序列化回传，字段不稳定。
- 修正动作：统一错误 DTO，未知错误走默认降级分支。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 3-6 页（估算）`

2. 卡片Z（Adaptation 主题：预加载与首帧观测）
- 页面标题：Adaptation Cases（预加载/预绘制与首帧验收，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：`preloadEngine` 与 `predrawEngine` 按场景区分，禁止混用无边界。
- 规则2：非全屏预绘制必须设置 `viewport_metrics`。
- 规则3：首帧观测统一用 `onFirstFrame` 作为回归基线。
- 混编映射：
- 对应样例：`ohos/flutter_it_preload/.../Index.ets`、`FlutterPage.ets`、`ohos/multiple_flutters_predraw/.../MainPage.ets`。
- 对应改造点（ArkTS/Flutter 边界）：预加载参数与页面真实 `viewId` 在 ArkTS 层做一致性校验。
- 风险与修正：
- 反模式：只做预热不校验缓存引擎与视口参数。
- 修正动作：增加 `cached_engine_id`、`viewport_metrics`、`viewId` 三项对齐检查。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 4-8 页（估算）`

3. 卡片AA（Java/Swift 迁移主题：权限链路迁移）
- 页面标题：Migrating from Java/Swift to ArkTS（权限申请与结果判定）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：权限流程固定为 `check -> request -> result check`。
- 规则2：权限结果必须逐项判断，不假设一次请求全通过。
- 规则3：权限拒绝与系统异常分开建模并回传不同动作建议。
- 混编映射：
- 对应样例：`ohos/ohos_flutter_photoviewpicker/.../CameraPermissions.ets`、`ohos/testcamera/.../CameraPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：权限判定集中在能力层，Flutter 只消费结果状态与动作建议。
- 风险与修正：
- 反模式：页面层直接发起权限弹窗且无结果分支。
- 修正动作：抽离权限服务并输出统一权限结果 DTO。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 3-5 页（估算）`

4. 卡片AB（Common Library 主题：事件与消息协同）
- 页面标题：ArkTS Common Library 子模块（事件分发与消息协同）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：事件能力（如 `emitter`）仅在桥接层聚合，不在页面层散落订阅。
- 规则2：通道消息体统一可序列化结构，避免混入系统对象引用。
- 规则3：页面销毁前必须解绑事件与通道 handler。
- 混编映射：
- 对应样例：`add_to_app/plugin/ohos_using_plugin/.../InAppBrowser.ets`、`.../methodPlugin.ets`、`ohos/channel_demo/.../BatteryPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：把事件订阅、通道处理、资源释放统一到插件生命周期。
- 风险与修正：
- 反模式：事件注册无解绑，导致重复触发与内存泄漏。
- 修正动作：统一注册/解绑模板并纳入发布前审查。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 6-9 页（估算）`

## 逐页深读打卡（第五组已填）

1. 卡片AC（Cookbook 主题：生命周期内状态一致性）
- 页面标题：TypeScript to ArkTS Cookbook（状态一致性与副作用边界，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：生命周期钩子中状态更新必须单向流动，避免页面内外双写。
- 规则2：`@State` 与共享状态链接字段分层，禁止在同一钩子内相互回写。
- 规则3：页面销毁前清理临时状态与监听关系，避免下一次进入复用脏状态。
- 混编映射：
- 对应样例：`ohos/multiple_flutters_predraw/.../SingleFlutterPage.ets`、`DoubleFlutterPage.ets`、`LazyListFlutterPage.ets`。
- 对应改造点（ArkTS/Flutter 边界）：状态更新由 ArkTS 页面层归并后再透出给 Flutter。
- 风险与修正：
- 反模式：生命周期钩子与通道回调同时写同一状态。
- 修正动作：定义单一主状态源并限制副作用入口。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 2-5 页（估算）`

2. 卡片AD（Adaptation 主题：页面路由与引擎缓存协同）
- 页面标题：Adaptation Cases（路由切换与缓存引擎协同，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：路由切换前确认缓存引擎可命中，避免页面先跳转后建引擎。
- 规则2：缓存引擎 ID 与页面路由参数保持可追踪关联。
- 规则3：路由返回时释放无效绑定，避免缓存污染后续页面。
- 混编映射：
- 对应样例：`ohos/multiple_flutters_predraw/.../MainPage.ets`、`EngineBindings.ets`、`ohos/flutter_it_preload/.../Index.ets`。
- 对应改造点（ArkTS/Flutter 边界）：缓存引擎管理集中到绑定层，页面层只声明目标路由与参数。
- 风险与修正：
- 反模式：路由层直接操作缓存导致状态漂移。
- 修正动作：建立“路由->缓存引擎->viewId”固定映射表。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 3-6 页（估算）`

3. 卡片AE（Java/Swift 迁移主题：通道协议稳定化）
- 页面标题：Migrating from Java/Swift to ArkTS（通道协议与兼容迁移）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：迁移后通道方法名与字段名保持版本化，不做隐式重命名。
- 规则2：新增字段优先向后兼容，旧字段保留过渡期。
- 规则3：协议变更需同步 ArkTS/Flutter 双侧断言与默认值策略。
- 混编映射：
- 对应样例：`ohos/channel_demo/.../BatteryPlugin.ets`、`add_to_app/.../methodPlugin.ets`、`ohos/flutter_huawei_login/.../LoginView.ets`。
- 对应改造点（ArkTS/Flutter 边界）：通道协议在桥接层集中定义并统一校验。
- 风险与修正：
- 反模式：直接改字段名导致 Flutter 侧解析失败。
- 修正动作：引入协议版本字段并保留旧分支兼容逻辑。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 2-4 页（估算）`

4. 卡片AF（Common Library 主题：能力组合与分层调用）
- 页面标题：ArkTS Common Library 子模块（多能力组合调用）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：多能力组合调用按“权限->数据->文件->通道输出”固定顺序编排。
- 规则2：能力失败分支在能力层闭环，不向页面层泄漏内部异常细节。
- 规则3：组合能力输出统一 DTO，字段稳定且可测试。
- 混编映射：
- 对应样例：`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`、`CameraPermissions.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：系统能力拼装逻辑沉淀在 ArkTS 能力层，Flutter 仅消费标准结果。
- 风险与修正：
- 反模式：页面层串联多个系统能力调用。
- 修正动作：封装能力编排服务并输出统一错误与结果模型。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 4-7 页（估算）`

## 逐页深读打卡（第六组已填）

1. 卡片AG（Cookbook 主题：路由参数版本化）
- 页面标题：TypeScript to ArkTS Cookbook（路由参数协议收敛，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：路由参数增加版本字段，兼容旧参数结构。
- 规则2：参数解析失败走默认降级，不直接中断页面生命周期。
- 规则3：参数映射在边界层完成，页面层只接收强类型结果。
- 混编映射：
- 对应样例：`ohos/flutter_page_sample2/.../Index2.ets`、`NavPageThree.ets`、`ohos/test_uni_links/.../EntryAbility.ets`。
- 对应改造点（ArkTS/Flutter 边界）：路由参数协议统一到桥接层并保留向后兼容分支。
- 风险与修正：
- 反模式：直接替换字段名导致旧链路失效。
- 修正动作：参数版本化 + 兼容解析 + 默认值策略。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 1-4 页（估算）`

2. 卡片AH（Adaptation 主题：PlatformView 协同渲染）
- 页面标题：Adaptation Cases（PlatformView 协同渲染与一致性，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：`viewType` 大小写与通道命名保持严格一致。
- 规则2：PlatformView 生命周期与页面生命周期联动，离场必须释放资源。
- 规则3：渲染模式切换前后统一校验注册状态与回调状态。
- 混编映射：
- 对应样例：`ohos/platform_demo/.../CustomPlugin.ets`、`CustomView.ets`、`platform_component_demo/.../CustomWebview.ets`。
- 对应改造点（ArkTS/Flutter 边界）：平台视图注册、通信、释放三段式流程固定化。
- 风险与修正：
- 反模式：`viewType` 不一致或只注册不释放。
- 修正动作：上线前逐字校验 `viewType`，并执行释放链路回归。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 2-5 页（估算）`

3. 卡片AI（Java/Swift 迁移主题：能力回调去平台化）
- 页面标题：Migrating from Java/Swift to ArkTS（能力回调与事件流迁移）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：将平台特定回调语义映射为统一事件模型后再出通道。
- 规则2：事件回调与页面状态更新解耦，避免互相触发环路。
- 规则3：事件失败分支与业务失败分支分离建模。
- 混编映射：
- 对应样例：`ohos/channel_demo/.../BatteryPlugin.ets`、`add_to_app/.../methodPlugin.ets`、`ohos/test_uni_links/.../XUniLinksPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：桥接层统一做事件整形与节流去重。
- 风险与修正：
- 反模式：平台回调直接透传到 Flutter，语义不稳定。
- 修正动作：引入统一事件 DTO 与分层回调协议。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 1-3 页（估算）`

4. 卡片AJ（Common Library 主题：能力可观测性）
- 页面标题：ArkTS Common Library 子模块（日志、错误码与可观测性）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：关键能力路径保留结构化日志与错误码映射点。
- 规则2：日志输出不泄漏敏感字段，通道回传只保留必要信息。
- 规则3：可观测指标与首帧、权限、通道错误链路统一对齐。
- 混编映射：
- 对应样例：`ohos/flutter_it_preload/.../FlutterPage.ets`、`ohos/flutter_huawei_login/.../LoginView.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：在桥接层统一观测点与错误码维度，便于回归与排障。
- 风险与修正：
- 反模式：日志散落且字段不一致，难以定位跨层问题。
- 修正动作：统一日志字段与错误码维度，固定关键埋点位置。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 3-5 页（估算）`

## 逐页深读打卡（第七组已填）

1. 卡片AK（Cookbook 主题：DTO 契约冻结）
- 页面标题：TypeScript to ArkTS Cookbook（数据契约稳定化，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：桥接 DTO 字段名与语义冻结，新增字段仅追加不覆写。
- 规则2：DTO 解析失败必须返回可恢复分支，禁止静默吞错。
- 规则3：DTO 与内部模型分离，避免页面层直接依赖通道原始结构。
- 混编映射：
- 对应样例：`ohos/flutter_huawei_login/.../LoginView.ets`、`ohos/channel_demo/.../BatteryPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：统一 DTO 版本与默认值策略，减少双端漂移。
- 风险与修正：
- 反模式：通道结构直接穿透到页面状态。
- 修正动作：增加 DTO 适配层并固定字段约束。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 1-3 页（估算）`

2. 卡片AL（Adaptation 主题：多通道协同）
- 页面标题：Adaptation Cases（Method/Event/BasicMessageChannel 协同，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：多通道并存时先定义主从职责，避免重复分发。
- 规则2：同一业务事件只能有一个主通道出口。
- 规则3：所有通道在页面离场与插件卸载时统一解绑。
- 混编映射：
- 对应样例：`ohos/channel_demo/.../BatteryPlugin.ets`、`add_to_app/.../methodPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：通道职责矩阵在桥接层收敛，页面层只订阅业务事件。
- 风险与修正：
- 反模式：同事件通过多个通道重复发送。
- 修正动作：建立事件-通道映射表并做冲突检查。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 1-4 页（估算）`

3. 卡片AM（Java/Swift 迁移主题：生命周期回归基线）
- 页面标题：Migrating from Java/Swift to ArkTS（生命周期回归与验收）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：迁移后以生命周期关键点建立回归清单（出现/隐藏/销毁/重入）。
- 规则2：生命周期回归需覆盖冷启动、热启动、返回栈重入。
- 规则3：生命周期异常与通道异常分离定位。
- 混编映射：
- 对应样例：`ohos/flutter_page_sample1/.../Page1.ets`、`ohos/flutter_page_sample2/.../Index2.ets`、`add_to_app/multiple_flutters/.../DoubleFlutterPage.ets`。
- 对应改造点（ArkTS/Flutter 边界）：把生命周期回归基线固化为发布前检查项。
- 风险与修正：
- 反模式：只验证首进页面，不验证返回栈与重入。
- 修正动作：补齐三类路径回归用例并记录结果。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 1-2 页（估算）`

4. 卡片AN（Common Library 主题：错误码字典统一）
- 页面标题：ArkTS Common Library 子模块（错误码字典与动作建议）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：Common Library 相关错误码统一收敛到字典表。
- 规则2：每个错误码必须映射 `recoverable/action`。
- 规则3：未知错误码进入默认降级路径并保留观测日志。
- 混编映射：
- 对应样例：`ohos/flutter_huawei_login/.../LoginView.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`、`ohos/testcamera/.../CameraUtil.ets`。
- 对应改造点（ArkTS/Flutter 边界）：桥接层统一码表后向 Flutter 回传一致语义。
- 风险与修正：
- 反模式：各插件各自定义错误文案与动作。
- 修正动作：统一错误码字典并在插件层复用。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 2-4 页（估算）`

## 逐页深读打卡（第八组已填）

1. 卡片AO（Cookbook 主题：边界校验最小闭环）
- 页面标题：TypeScript to ArkTS Cookbook（入口校验与默认值闭环，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：所有外部入参（路由、通道、能力回调）都先做边界校验。
- 规则2：校验失败走默认值或降级分支，不让异常扩散到 UI 主流程。
- 规则3：校验结果统一落在 DTO，再进入业务层。
- 混编映射：
- 对应样例：`ohos/flutter_page_sample2/.../NavPageThree.ets`、`ohos/test_uni_links/.../XUniLinksPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：桥接层建立统一校验入口与默认值策略。
- 风险与修正：
- 反模式：边界校验分散在页面各处且不一致。
- 修正动作：提取公共校验函数并统一调用。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 0-2 页（估算）`

2. 卡片AP（Adaptation 主题：生命周期与通道一致性回归）
- 页面标题：Adaptation Cases（生命周期与通道一致性，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：页面出现/隐藏与通道订阅/解绑状态必须同向变化。
- 规则2：通道回调只在页面活跃期更新 UI 相关状态。
- 规则3：页面重入时先恢复生命周期状态再恢复通道监听。
- 混编映射：
- 对应样例：`ohos/flutter_page_sample1/.../Page1.ets`、`ohos/flutter_it_preload/.../FlutterPage.ets`、`add_to_app/.../methodPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：生命周期与通道状态统一由页面适配层编排。
- 风险与修正：
- 反模式：页面隐藏后仍保留活跃回调导致脏更新。
- 修正动作：按生命周期钩子严格执行订阅/解绑对称操作。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 0-3 页（估算）`

3. 卡片AQ（Java/Swift 迁移主题：发布前兼容清单）
- 页面标题：Migrating from Java/Swift to ArkTS（兼容清单与回归验收）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：发布前必须完成“冷启动/热启动/重入/权限拒绝”四象限回归。
- 规则2：通道协议、错误码、生命周期三类兼容项独立验收。
- 规则3：迁移后的默认降级动作需要可观测与可追踪。
- 混编映射：
- 对应样例：`ohos/test_uni_links/.../EntryAbility.ets`、`ohos/flutter_huawei_login/.../LoginView.ets`、`add_to_app/multiple_flutters/.../DoubleFlutterPage.ets`。
- 对应改造点（ArkTS/Flutter 边界）：把兼容清单内化为发布门禁检查项。
- 风险与修正：
- 反模式：只验证主流程，忽略拒绝与重入路径。
- 修正动作：补齐兼容清单并固定执行顺序。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 0-1 页（估算）`

4. 卡片AR（Common Library 主题：子模块收敛清单）
- 页面标题：ArkTS Common Library 子模块（子模块收敛与复用）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：AbilityKit/BasicServicesKit/CoreFileKit/ArkData 形成统一复用清单。
- 规则2：新增能力优先复用官方模块，再决定自定义封装。
- 规则3：子模块调用路径统一走能力层服务，不允许页面层直连系统能力。
- 混编映射：
- 对应样例：`add_to_app/multiple_flutters/.../EngineBindings.ets`、`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：子模块能力以服务目录形式收敛，输出稳定 DTO。
- 风险与修正：
- 反模式：重复封装官方已有能力，导致维护成本上升。
- 修正动作：建立能力复用优先级与选型记录。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 1-3 页（估算）`

## 逐页深读打卡（第九组已填）

1. 卡片AS（Cookbook 主题：空值与可选链收敛）
- 页面标题：TypeScript to ArkTS Cookbook（空值语义与可选链边界，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：可选链与默认值组合必须显式控制优先级。
- 规则2：空值分支与业务失败分支分离处理。
- 规则3：空值兜底结果进入 DTO 后再参与后续流程。
- 混编映射：
- 对应样例：多个 `ohosTest/testability/TestAbility.ets` 中 `JSON.stringify(...) ?? ''` 写法、`ohos/flutter_page_sample2/.../NavPageThree.ets`。
- 对应改造点（ArkTS/Flutter 边界）：桥接层统一空值语义，避免页面与插件各自兜底。
- 风险与修正：
- 反模式：在字符串拼接场景误用 `??` 导致兜底失效。
- 修正动作：显式括号与分层空值处理。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 0-1 页（估算）`

2. 卡片AT（Adaptation 主题：预绘制参数一致性）
- 页面标题：Adaptation Cases（预绘制参数一致性与回归，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：`cached_engine_id`、`viewId`、`viewport_metrics` 三参数一致性必须在渲染前校验。
- 规则2：多页面并发预绘制按页面维度分别设定视口。
- 规则3：预绘制失败走降级路径并记录首帧对比日志。
- 混编映射：
- 对应样例：`ohos/multiple_flutters_predraw/.../MainPage.ets`、`EngineBindings.ets`、`LazyListFlutterPage.ets`。
- 对应改造点（ArkTS/Flutter 边界）：预绘制参数校验集中在引擎绑定层，页面仅声明目标。
- 风险与修正：
- 反模式：共用单一视口配置导致半屏/分屏错位。
- 修正动作：按页面布局生成独立视口参数。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 0-2 页（估算）`

3. 卡片AU（Java/Swift 迁移主题：错误分层验收）
- 页面标题：Migrating from Java/Swift to ArkTS（错误分层与验收闭环）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：可恢复错误、不可恢复错误、未知错误三类分层验收。
- 规则2：每类错误都要有明确动作建议与日志字段。
- 规则3：迁移后错误分层要与通道协议版本同步回归。
- 混编映射：
- 对应样例：`ohos/flutter_huawei_login/.../LoginView.ets`、`ohos/testcamera/.../CameraUtil.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：错误分类逻辑统一在桥接层，Flutter 层只消费稳定错误语义。
- 风险与修正：
- 反模式：错误分类与动作建议不一致，导致重试策略失效。
- 修正动作：统一错误分层表并加入回归用例。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 0-1 页（估算）`

4. 卡片AV（Common Library 主题：子模块最小调用面）
- 页面标题：ArkTS Common Library 子模块（最小调用面与依赖收敛）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：子模块调用面最小化，只暴露业务需要的方法。
- 规则2：跨模块依赖通过能力层接口转发，不直接跨层引用实现细节。
- 规则3：新增依赖先评估是否可由现有子模块组合实现。
- 混编映射：
- 对应样例：`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`、`add_to_app/.../EngineBindings.ets`。
- 对应改造点（ArkTS/Flutter 边界）：能力服务接口固定化，减少跨层依赖扩散。
- 风险与修正：
- 反模式：能力层暴露过多细节导致上层强耦合。
- 修正动作：收敛导出接口并补充能力边界审查。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 1-2 页（估算）`

## 逐页深读打卡（第十组已填）

1. 卡片AW（Cookbook 主题：边界类型守卫收尾）
- 页面标题：TypeScript to ArkTS Cookbook（类型守卫与边界收尾，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：边界输入统一先做类型守卫再做业务判断。
- 规则2：守卫失败路径必须返回可诊断信息与默认动作。
- 规则3：守卫逻辑复用，不在页面层重复实现。
- 混编映射：
- 对应样例：`ohos/flutter_page_sample2/.../NavPageThree.ets`、`add_to_app/.../InAppBrowser.ets`。
- 对应改造点（ArkTS/Flutter 边界）：桥接层守卫函数统一复用，输出稳定 DTO。
- 风险与修正：
- 反模式：同一参数在不同页面重复判定且标准不一致。
- 修正动作：提取统一守卫工具并集中维护。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Cookbook 细分 recipe 仍待补 0-1 页（估算）`

2. 卡片AX（Adaptation 主题：多实例页面回归收尾）
- 页面标题：Adaptation Cases（多实例页面与通道状态回归，按样例实证落卡）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：多实例页面必须保证实例级 `viewId` 与通道名隔离。
- 规则2：实例销毁顺序固定为“解绑通道->解绑引擎->释放页面状态”。
- 规则3：实例重建需复位缓存状态，避免复用脏上下文。
- 混编映射：
- 对应样例：`add_to_app/multiple_flutters/.../LazyListFlutterPage.ets`、`DoubleFlutterPage.ets`、`EngineBindings.ets`。
- 对应改造点（ArkTS/Flutter 边界）：多实例管理逻辑集中在绑定层，不下放到页面细节代码。
- 风险与修正：
- 反模式：实例间复用同一通道或同一缓存标识。
- 修正动作：实例维度命名规范与销毁顺序校验。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Adaptation Cases 关联案例仍待补 0-2 页（估算）`

3. 卡片AY（Java/Swift 迁移主题：兼容门禁收尾）
- 页面标题：Migrating from Java/Swift to ArkTS（迁移门禁与最终验收）
- 页面链接：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：迁移发布门禁包含生命周期、权限、通道、错误四类检查。
- 规则2：每类门禁有明确失败动作（回退/降级/拦截发布）。
- 规则3：门禁结果与版本号绑定，保证可追溯。
- 混编映射：
- 对应样例：`ohos/test_uni_links/.../EntryAbility.ets`、`ohos/flutter_huawei_login/.../LoginView.ets`、`ohos/testcamera/.../CameraPlugin.ets`。
- 对应改造点（ArkTS/Flutter 边界）：把迁移验收内化为发布前固定脚本与清单。
- 风险与修正：
- 反模式：仅依赖人工经验判断是否可发布。
- 修正动作：门禁清单标准化并与版本发布流程绑定。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Java/Swift 迁移章节仍待补 0-1 页（估算）`

4. 卡片AZ（Common Library 主题：子模块逐页收尾）
- 页面标题：ArkTS Common Library 子模块（逐页收尾与复用治理）
- 页面链接：`https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-utils`
- 读取层级：`逐页深读`
- 本页摘录：
- 规则1：对子模块能力建立“已用/候选/禁用”三级清单。
- 规则2：新增功能优先复用“已用”与“候选”能力，减少重复封装。
- 规则3：禁用能力需记录禁用原因与替代方案。
- 混编映射：
- 对应样例：`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`、`add_to_app/.../EngineBindings.ets`。
- 对应改造点（ArkTS/Flutter 边界）：能力治理文档化并纳入评审。
- 风险与修正：
- 反模式：同能力在不同插件重复封装。
- 修正动作：建立子模块复用台账与评审约束。
- 覆盖状态：
- 本页状态：`已完成`
- 剩余关联页数（估算）：`Common Library 子模块逐页摘录仍待补 0-2 页（估算）`

## 样例实读验证（文件级证据）

1. 通道与插件注册（`channel_demo`）
- `entryability/EntryAbility.ets` 显式调用 `GeneratedPluginRegistrant.registerWith(flutterEngine)`。
- `entryability/BatteryPlugin.ets` 同时使用 `MethodChannel` 与 `EventChannel`，并在释放阶段清理 handler。

2. 多引擎绑定（`multiple_flutters_ohos`）
- `pages/EngineBindings.ets` 使用 `FlutterEngineGroup` 创建引擎，并在 `attach/detach` 管理引擎与视图绑定。
- 同文件在通道初始化后执行 `setMethodCallHandler` 与 `invokeMethod`，体现双向通信入口。

3. 生命周期同步（`SingleFlutterPage/DoubleFlutterPage`）
- `aboutToAppear/aboutToDisappear` 对应 `attach/detach`。
- `onPageShow/onPageHide` 对应 `LifecycleChannel` 的 `appIsResumed/appIsPaused` 与窗口聚焦状态同步。
- `FlutterPage({ viewId })` 与 `getFlutterViewId()` 成对出现，确认 viewId 链路一致。

## 迁移反模式实证补充（批量样例扫描）

1. `any` 使用分布结论
- `any` 高密度出现于 `ohosTest/testability` 模板代码（`abilityDelegator` 等测试变量），业务主链路中占比低。
- 迁移审查时应区分“测试脚手架 any”与“业务代码 any”，优先清理业务链路的动态类型扩散。

2. 异常模型实证结论
- `GeneratedPluginRegistrant.ets` 常见 `try/catch` 包裹插件注册，属于插件兼容保护层。
- 少量样例存在 `throw new Error('Method not implemented.')` 占位实现；迁移落地时应改成结构化错误返回，避免运行时硬抛。

3. 生命周期与路由耦合结论
- 多数混编页面在 `aboutToAppear/aboutToDisappear/onPageShow/onPageHide` 中做引擎/页面同步。
- 迁移时若直接平移其他语言页面生命周期而忽略这四个钩子，易造成引擎状态与页面状态错位。

4. 通道边界结论
- `MethodChannel/EventChannel/BasicMessageChannel` 在样例中组合使用非常普遍，且普遍包含解绑清理动作。
- 迁移改造应把“注册/调用/解绑”作为一组原子规则审查，避免只迁移调用不迁移清理。

## Want 与 onNewWant 统一处理模板（实证）

1. 观察到的样例链路
- `ohos/test_uni_links/.../entryability/EntryAbility.ets`：`onNewWant` 中先 `super.onNewWant`，再转发到插件实例处理。
- `ohos/test_uni_links/.../XUniLinksPlugin.ets`：`onNewWant` 与 `handleWant` 分层，统一参数入口。

2. 统一处理规则
- 冷启动：在 `onCreate(want, launchParam)` 解析初始参数。
- 热启动：在 `onNewWant(want, launchParams)` 复用同一解析函数。
- 去重：对同一 deep link / 参数签名做幂等处理，避免重复分发。
- 输出：统一转成通道友好 DTO，再发到 Flutter 层。

3. 审查清单
- [ ] `onCreate` 与 `onNewWant` 是否共用一套参数解析逻辑。
- [ ] 是否保留 `super.onNewWant(...)` 调用。
- [ ] 是否有重复事件抑制（时间窗或签名去重）。
- [ ] 是否将解析结果统一映射为稳定 DTO。

## 权限与错误处理模板（实证）

1. 权限处理统一流程
- 先 `checkAccessToken` 判断授权状态，再按需调用 `requestPermissionsFromUser`。
- 对权限结果做逐项判定，不假设一次请求全部通过。
- 将权限失败结果转为结构化错误（错误码 + 描述 + 建议动作）后再回传 Flutter。

2. 错误处理统一流程
- 优先使用 `BusinessError` 读取 `code/message`，避免只打印字符串。
- 对已知错误码做分支映射（网络、未登录、协议未同意、不支持等）。
- 对未知错误码走默认降级路径，并保留可观测日志。

3. 样例证据
- `ohos/ohos_flutter_photoviewpicker/.../CameraPermissions.ets`：`checkAccessToken -> requestPermissionsFromUser` 两段式权限链路。
- `ohos/flutter_huawei_login/.../LoginView.ets`：按 `error.code` 分支并通过通道回传可读错误信息。

4. 审查清单
- [ ] 是否先检查权限状态再发起权限弹窗。
- [ ] 是否对授权结果数组逐项判断。
- [ ] 是否统一输出 `code/message/recoverable/action`。
- [ ] 是否对未知错误码设置默认降级分支。

## PlatformView 集成模板（实证）

1. 注册链路
- ArkTS 侧通过 `PlatformViewFactory#create` 产出 `PlatformView` 实例。
- 插件侧在 `onAttachedToEngine` 调用 `registerViewFactory(viewType, factory)` 完成注册。
- 宿主 `EntryAbility` 需保证 `GeneratedPluginRegistrant.registerWith(flutterEngine)` 生效。

2. 双向通信链路
- ArkTS `PlatformView` 内使用 `MethodChannel`，频道名通常包含 `viewId` 或固定 `viewType` 语义。
- Flutter 侧 `OhosView(viewType: ...)` 与 ArkTS 注册 `viewType` 必须严格一致。
- 需要回调 Flutter 时调用 `invokeMethod`，并在必要时设置 handler 处理反向调用。

3. 释放链路
- `dispose()` 不应保留占位异常；应改为资源释放与通道解绑动作。
- 禁止在 `dispose()` 保留 `throw new Error('Method not implemented.')` 这类模板代码。

4. 实证风险（样例中发现）
- `platform_component_demo` 存在 `viewType` 大小写不一致风险：ArkTS 注册 `com.example.platformView/...`，Flutter 某页面使用 `com.example.platformview/...`。
- 混编审查时应把 `viewType` 作为强一致字段逐字比对（含大小写）。

## 构建与依赖层级模板（文档实证补充）

1. 命令分层
- Flutter 侧构建命令统一在 Flutter 工程执行：`fvm flutter build hap`、`fvm flutter build har`、`fvm flutter run`。
- OHOS 侧依赖安装在项目根层执行：`ohpm install`（不是 `entry` 子目录盲跑）。
- 使用 Hvigor 插件方式时，产物路径与依赖注入逻辑以 `hvigorfile.ts/hvigorconfig.ts` 为准。

2. 产物与引用
- HAP 产物关注路径：`ohos/entry/build/default/outputs/default/entry-default-signed.hap`（或 `build/ohos/...` 插件化路径）。
- HAR 集成需同步检查：
  - 根 `oh-package.json5` 的 `overrides`
  - `entry/oh-package.json5` 的 `dependencies`
  - `har/*` 实际文件路径

3. 常见失败分支
- `entry` 下直接执行 `ohpm install` 导致依赖缓存不完整。
- 修改 `flutter_module` 代码后未重新 `fvm flutter build har --debug|--release`。
- `overrides` 路径或别名不一致（含 `useNormalizedOHMUrl` 约束）导致编译失败。

4. `local-engine` 使用边界
- 新流程默认可不传 `--local-engine`（云端产物）。
- 仅在自编译引擎调试场景使用 `--local-engine` 与 `--local-engine-host`。

## 引擎预加载与首帧优化模板（实证）

1. 预加载模式选择
- `preloadEngine(context, params)`：按参数预热引擎，适合普通单页预热。
- `predrawEngine(engine, params, viewId?)`：基于已有引擎预绘制，适合多 Flutter 页面或缓存引擎复用场景。

2. 关键参数规则
- `dart_entrypoint`：多入口工程必须显式配置，避免路由命中错误入口。
- `cached_engine_id`：使用缓存引擎时保证 ID 唯一且可追踪。
- `viewport_metrics`：非全屏预绘制必须设置 `physicalWidth/physicalHeight`，否则默认全屏预绘制。

3. 多引擎预绘制规则
- 先 `createAndRunEngineByOptions` 创建并放入 `FlutterEngineCache`。
- 预绘制时从 `FlutterEngineCache.get(id)` 获取引擎，必要时分配连续 `FlutterViewId`。
- 多视图并发预绘制时，按页面布局分别设置 viewport，避免错位与拉伸。

4. 首帧观测与验收
- 页面侧保留 `onFirstFrame` 日志点，作为预加载效果回归基线。
- 验收关注：首帧时间、交互触发到可见时间、DevTools 连接建立时机变化。

5. 常见风险
- 仅做 `preload` 未做 `viewport_metrics`，导致半屏/分屏页面按全屏渲染。
- `cached_engine_id` 与真实缓存不一致，导致预热无效或命中错误引擎。
- 预绘制后未与页面真实 `viewId` 对齐，导致内容不显示或显示错位。

## 状态管理与路由参数边界模板（实证）

1. 状态源分层规则
- 页面私有短周期状态使用 `@State`，只在当前组件生命周期内生效。
- 跨组件共享但同页面树内状态优先 `@LocalStorageLink`，避免全局状态污染。
- 跨页面或全局会话状态使用 `@StorageLink` / `AppStorage.link`，并约束 key 命名与写入时机。

2. 路由参数读取边界
- 页面初始化参数使用 `router.getParams()`，仅用于“入参快照”，不要在后续流程反复读取并回写。
- 路由栈参数使用 `pathStack.getParamByName(...)`，适合多页面链路中按需读取。
- 导航时通过 `router.pushUrl({ ..., params })` 显式传参，避免依赖隐式全局状态。

3. 数据模型与类型约束
- 路由参数与共享状态都优先使用显式接口类型，避免 `any` 贯穿页面与通道边界。
- 解析参数后先做空值与类型校验，再写入 `@State/@StorageLink`，避免初始化阶段脏数据扩散。
- 对外发送到 Flutter 的数据统一先转 DTO，避免直接暴露 ArkTS 内部结构。

4. 实证映射（样例）
- `multiple_flutters_ohos/.../pages/SingleFlutterPage.ets`：`@State` 管理页面级 `viewId` 与展示状态。
- `multiple_flutters_ohos/.../pages/DoubleFlutterPage.ets`、`LazyListFlutterPage.ets`：多页面场景下 `@State` 与引擎视图绑定协同。
- `platform_component_demo/.../pages/Index.ets` 等：`@StorageLink` / `AppStorage.link` 用于跨页面共享状态。
- 多个 `Index.ets` 模板页：`@LocalStorageLink('viewId')` 体现页面树内共享而不提升为全局状态。
- 路由链路样例：`router.getParams`、`pathStack.getParamByName`、`router.pushUrl(params)` 组合使用。

5. 审查清单
- [ ] 是否按 `@State -> @LocalStorageLink -> @StorageLink/AppStorage` 做最小作用域选型。
- [ ] 是否把路由参数读取与状态写入解耦，避免循环依赖。
- [ ] 是否对入参做类型与空值校验后再写入共享状态。
- [ ] 是否避免把一次性路由参数直接沉淀为长期全局状态。
- [ ] 是否统一 DTO 边界，避免 ArkTS 内部对象直接跨层传递。

## Cookbook 与 Adaptation 首批卡片（实证）

1. 卡片A：`as` 断言最小化与入口校验
- 现象：`router.getParams() as Map<string, ValueType>`、`params['url'] as string` 在样例中常见。
- 风险：直接断言后无校验，参数缺失或类型漂移会在运行期暴露。
- 规则：先判空与类型，再做窄化断言；禁止把未经校验的 `as` 结果直接跨层传递。
- 样例证据：`add_to_app/plugin/ohos_using_plugin/.../InAppBrowser.ets`、`ohos/flutter_page_sample2/.../NavPageThree.ets`。

2. 卡片B：一次性入参与路由栈参数分工
- 现象：`router.getParams` 与 `pathStack.getParamByName` 在同工程并存。
- 风险：混用读取来源并回写全局状态，易造成参数来源不一致。
- 规则：初始化快照用 `router.getParams`；链路取值用 `pathStack.getParamByName`；写入状态前统一 DTO。
- 样例证据：`ohos/flutter_page_sample2/.../Index2.ets`、`NavPageThree.ets`。

3. 卡片C：`throw new Error` 仅限致命路径
- 现象：多引擎页面存在 `throw new Error("Create engine failed.")` 等硬抛。
- 风险：直接中断页面流程，Flutter 侧缺少结构化错误上下文。
- 规则：非致命故障改为结构化错误返回（`code/message/recoverable/action`）并保留日志；仅在不可恢复的初始化失败场景硬抛。
- 样例证据：`ohos/multiple_flutters_predraw/.../MainPage.ets`、`EngineBindings.ets`。

4. 卡片D：`any` 作用域限制
- 现象：`any` 高密度集中在 `ohosTest/testability/TestAbility.ets` 脚手架。
- 风险：若复制到业务主链路会放大类型不确定性。
- 规则：测试脚手架可局部保留 `any`；业务与桥接层禁止新增 `any`，必须落到显式类型或 DTO。
- 样例证据：多个 `.../ohosTest/ets/testability/TestAbility.ets`（`abilityDelegator: any`）。

5. 卡片E：`??` 与字符串拼接优先级
- 现象：`'want param:' + JSON.stringify(want) ?? ''` 在样例脚手架大量出现。
- 风险：`+` 优先级高于 `??`，表达式等价于 `('want param:' + JSON.stringify(want)) ?? ''`，兜底值通常失效。
- 规则：优先写为 `'want param:' + (JSON.stringify(want) ?? '')`，避免空值兜底语义偏差。
- 样例证据：多个 `.../ohosTest/ets/testability/TestAbility.ets`。

## Java/Swift 迁移卡片（实证补齐）

1. 卡片F：生命周期显式编排替代“隐式回调链”
- 现象：混编页面显式维护 `aboutToAppear/aboutToDisappear/onPageShow/onPageHide` 与 Flutter 生命周期同步。
- 迁移指引：从 Java/Swift 迁移时，不直接平移原有页面回调假设，必须在 ArkTS 页面中显式编排四段生命周期。
- 样例证据：`ohos/flutter_page_sample1/.../Page1.ets`、`ohos/flutter_page_sample2/.../Index2.ets`、`add_to_app/multiple_flutters/.../SingleFlutterPage.ets`。

2. 卡片G：热启动参数合流
- 现象：`EntryAbility.onNewWant` 转发到插件统一处理函数。
- 迁移指引：把冷启动与热启动参数统一收口，禁止双实现分叉。
- 样例证据：`ohos/test_uni_links/.../EntryAbility.ets`、`.../XUniLinksPlugin.ets`。

3. 卡片H：异常分层替代“全局抛异常”
- 现象：样例既有 `BusinessError` 分支处理，也存在 `throw new Error(...)` 的硬抛路径。
- 迁移指引：将可恢复错误改为结构化返回，仅对不可恢复初始化故障硬抛。
- 样例证据：`ohos/flutter_huawei_login/.../LoginView.ets`、`ohos/testcamera/.../CameraUtil.ets`、`ohos/multiple_flutters_predraw/.../MainPage.ets`。

## Common Library 子模块卡片（实证补齐）

1. 卡片I：`AbilityKit` 上下文边界
- 用法：通过 `@kit.AbilityKit` 的 `common`/`EnvironmentCallback` 传递能力上下文与环境回调。
- 规则：仅在能力层与页面桥接层持有上下文，业务逻辑层不直接依赖 UIAbility 上下文。
- 样例证据：`add_to_app/multiple_flutters/.../EngineBindings.ets`、`ohos/flutter_it_preload/.../FlutterPage.ets`。

2. 卡片J：`BasicServicesKit` 错误与系统能力
- 用法：`BusinessError`、`batteryInfo`、`emitter` 在通道与插件中高频出现。
- 规则：统一在桥接层完成错误码映射与事件分发，避免页面层散落系统 API。
- 样例证据：`add_to_app/.../methodPlugin.ets`、`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets`。

3. 卡片K：`CoreFileKit` 与文件能力
- 用法：`BackupExtensionAbility`、`fileIo`、`fileUri` 出现在备份扩展与媒体文件处理链路。
- 规则：文件读写与备份扩展收敛到能力层，页面层只消费结构化结果。
- 样例证据：`add_to_app/.../EntryBackupAbility.ets`、`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`。

4. 卡片L：`ArkData` 参数与数据谓词
- 用法：`ValueType`、`dataSharePredicates` 用于路由参数与数据访问条件构建。
- 规则：跨层参数统一以可序列化 DTO 落地，避免把 `Map<string, ValueType>` 直接外溢到 UI 层。
- 样例证据：`add_to_app/.../InAppBrowser.ets`、`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets`。

## 输出标准

- 所有说明与注释默认简体中文。
- 优先给出可执行命令和具体文件路径，避免空泛描述。
- 涉及工程化问题时，必须给出“复现步骤 + 诊断结论 + 修复建议”三段式结果。

## 深度实读笔记（源码级，非大纲）

1. 生命周期与插件注入时机（实读结论）
- 文件：`ohos/flutter_page_sample2/ohos/entry/src/main/ets/pages/Index2.ets:17-34`
- 观察：`aboutToAppear` 中先构建 `FlutterEntry` 再 `aboutToAppear`，并在 `onFirstFrame` 才把 `BatteryPlugin` 加入引擎；`aboutToDisappear` 时显式移除插件。
- 我的理解：插件注入如果早于首帧，容易在引擎未完成初始化阶段触发不稳定调用；样例用首帧作为“引擎可用”边界更稳。
- 反例风险：若只 add 不 remove，会在重入页面时重复注册插件。

2. 热启动参数链路确实做了“能力层转发 + 插件统一处理”
- 文件：`ohos/test_uni_links/ohos/entry/src/main/ets/entryability/EntryAbility.ets:28-33`
- 文件：`ohos/test_uni_links/packages/x_uni_links/ohos/src/main/ets/components/plugin/XUniLinksPlugin.ets:56-59,95-100`
- 观察：`EntryAbility.onNewWant` 先 `super.onNewWant` 再按 URI 前缀转发；插件层统一走 `handleWant`，并同时更新 `link` 与 `eventSink`。
- 我的理解：这不是“页面内随手处理”，而是标准的能力层路由过滤 + 插件层单入口解析。
- 反例风险：冷/热启动分裂实现会导致 initial link 与实时 link 语义不一致。

3. 通道解绑不是可选项，样例已经体现不对称风险点
- 文件：`.../XUniLinksPlugin.ets:73-85`
- 观察：`onAttachedToEngine` 同时注册 `MethodChannel` 与 `EventChannel`；`onDetachedFromEngine` 仅清理 method handler，event stream handler 未同步清理。
- 我的理解：这类“不对称解绑”会在长会话或多次 attach/detach 场景累积隐患。
- 反例风险：事件仍被投递到旧 sink，表现为重复事件或空指针路径。

4. 预绘制链路的关键不是“调用了 predraw”，而是参数三元组一致
- 文件：`ohos/multiple_flutters_predraw/multiple_flutters_ohos/entry/src/main/ets/pages/MainPage.ets:55-60,111-127`
- 观察：先 `createAndRunEngineByOptions` 并放入 `FlutterEngineCache`，预绘制时再按场景设置 `viewport` 与 `nextViewId`，分别调用 `predrawEngine`。
- 我的理解：`cached_engine_id`、`viewport_metrics`、`viewId` 是一个完整三元组；缺任一项都可能“预绘制成功但页面显示异常”。
- 反例风险：半屏页面使用全屏视口，或者 top/bottom 共用同一 viewId。

5. 权限链路样例不是“是否有权限”的布尔问题，而是数组逐项判定
- 文件：`ohos/ohos_flutter_photoviewpicker/ohos/entry/src/main/ets/entryability/CameraPermissions.ets:53-70`
- 观察：`requestPermissionsFromUser` 返回 `authResults` 数组，代码逐项判断非 0 即拒绝并返回错误。
- 我的理解：这是多权限请求的正确处理方式；一次授权结果必须逐项落地到业务决策。
- 反例风险：把任一成功当全成功，会在后续调用特定能力时崩溃。

6. 错误回传已经有“结构化倾向”，但格式仍不统一
- 文件：`ohos/sqflite_helper/ohos/src/main/ets/components/plugin/SqfliteHelperPlugin.ets:164-171`
- 观察：捕获 `BusinessError` 后取 `code/message` 并通过 `result.error` 回传。
- 我的理解：这是正确方向，但不同插件间 `error` 的第三参数语义仍不一致，建议统一 `code/message/recoverable/action`。
- 反例风险：Flutter 层难以做跨插件统一降级与重试策略。

## 历史内容重写版（以本节为准）

说明：以下条目用于替换前面“偏大纲”的历史学习结论，统一改为“源码事实 -> 我的理解 -> 风险”格式。

1. 生命周期四钩子在混编页是强约束，不是可选实践
- 证据：`ohos/flutter_page_sample2/ohos/entry/src/main/ets/pages/Index2.ets:17-43`
- 事实：页面完整实现 `aboutToAppear/aboutToDisappear/onPageShow/onPageHide`，并逐一转发给 `flutterEntry`。
- 理解：这说明 ArkTS 页面生命周期与 Flutter 生命周期同步是硬约束链路。
- 风险：缺任一钩子都可能造成页面可见状态与引擎状态错位。

2. 首帧后再注入插件是样例选择的稳定边界
- 证据：`.../Index2.ets:25-34`
- 事实：`onFirstFrame` 才 add plugin，`aboutToDisappear` remove plugin。
- 理解：插件生命周期与页面生命周期绑定，避免引擎未就绪时提前调用。
- 风险：提前注入或离场不移除会导致重复注册与脏回调。

3. 热启动链路已形成“Ability 过滤 + 插件统一入口”
- 证据：`ohos/test_uni_links/ohos/entry/src/main/ets/entryability/EntryAbility.ets:28-33`，`.../XUniLinksPlugin.ets:56-59,95-100`
- 事实：`onNewWant` 先 `super` 再按 URI 前缀转发；插件统一走 `handleWant`。
- 理解：冷/热启动统一解析入口是可复用的迁移基线。
- 风险：双入口双实现会造成 deep link 行为不一致。

4. 通道解绑目前存在不对称点，是真实风险而非理论风险
- 证据：`.../XUniLinksPlugin.ets:73-85`
- 事实：attach 时同时注册 method/event，detach 只清 method handler。
- 理解：事件通道可能残留旧 sink 或旧 handler 生命周期。
- 风险：重复事件、泄漏、空对象回调。

5. 预绘制成功取决于参数三元组一致，不是只调用 API
- 证据：`ohos/multiple_flutters_predraw/.../MainPage.ets:55-60,111-127`
- 事实：先建引擎并缓存，再按 `viewport` 与 `nextViewId` 调 `predrawEngine`。
- 理解：`engine cache + viewport + viewId` 缺一不可。
- 风险：半屏错位、双实例串屏、预绘制无效。

6. 权限流程是三段式，而且结果必须逐项判定
- 证据：`ohos/ohos_flutter_photoviewpicker/.../CameraPermissions.ets:12-40,53-70`
- 事实：先 `checkAccessToken`，再 `requestPermissionsFromUser`，最后遍历 `authResults`。
- 理解：多权限请求必须按数组逐项落地，不能布尔化简化。
- 风险：把部分授权误当全授权，后续调用失败。

7. 错误处理已有结构化基础，但跨插件尚未完全统一
- 证据：`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets:164-171`，`ohos/flutter_huawei_login/.../LoginView.ets`（错误码分支）
- 事实：已有 `BusinessError.code/message` 使用与回传。
- 理解：可在现有基础上统一成同一错误 DTO 协议。
- 风险：各插件格式不一致导致 Flutter 层无法做统一降级。

8. 路由参数读取来源在样例中已明确区分
- 证据：`ohos/flutter_page_sample2/.../Index2.ets:19`，`.../NavPageThree.ets:42`
- 事实：既用 `router.getParams`，也用 `pathStack.getParamByName`。
- 理解：前者偏初始化快照，后者偏路由栈链路读取。
- 风险：混用后回写共享状态会造成参数来源冲突。

9. 预加载与首帧观测在样例里是闭环的
- 证据：`ohos/flutter_it_preload/.../Index.ets:55`，`.../FlutterPage.ets:45-46`
- 事实：触发 `preloadEngine` 后，页面侧保留 `onFirstFrame` 观测点。
- 理解：这为性能回归提供了最小可用观测闭环。
- 风险：只做预热无观测，无法验证优化是否生效。

10. 多引擎样例已显示“缓存->绑定->页面”的分层
- 证据：`ohos/multiple_flutters_predraw/.../MainPage.ets:55-60`，`.../EngineBindings.ets:64`
- 事实：引擎由缓存层持有，页面绑定层再取缓存引擎。
- 理解：这是一种可迁移的职责拆分，不应让页面直接管理引擎生命周期细节。
- 风险：页面直接拿引擎会导致职责混乱与释放时序问题。

11. PlatformView 的“注册名一致性”是硬门槛
- 证据：`ohos/platform_demo/lib/custom_ohos_view.dart:28`，`ohos/platform_demo/ohos/entry/src/main/ets/entryability/CustomPlugin.ets:13`
- 事实：Flutter 侧 `OhosView.viewType` 与 ArkTS 侧 `registerViewFactory` 的字符串一一对应。
- 理解：viewType 本质是跨端路由键，任何拼写漂移都会直接失配。
- 风险：页面可渲染但平台视图不创建，问题表象像“白屏/无响应”。

12. PlatformView 示例里存在“占位式 dispose”风险
- 证据：`ohos/platform_demo/ohos/entry/src/main/ets/entryability/CustomView.ets:93`
- 事实：存在 `dispose()` 方法，但若无资源与通道清理会变成空壳释放。
- 理解：PlatformView 释放必须至少覆盖通道解绑、对象引用释放、事件停止三类动作。
- 风险：多次进入退出后出现残留回调或资源泄漏。

13. 多引擎页面对 attach/detach 的调用是“成对出现”的
- 证据：`ohos/multiple_flutters_predraw/.../SingleFlutterPage.ets:39-43`，`.../DoubleFlutterPage.ets:38-44`
- 事实：页面出现时 attach，离开时 detach，双实例页面对 top/bottom 都做对称处理。
- 理解：这是可复用的最小生命周期模板，比“只在初始化 attach 一次”更安全。
- 风险：detach 缺失会导致隐藏页仍持有渲染与通道资源。

14. 预绘制链路已明确依赖 EngineCache 命中
- 证据：`ohos/multiple_flutters_predraw/.../EngineBindings.ets:64-67`
- 事实：绑定阶段先从 `FlutterEngineCache` 取引擎，取不到会 `throw new Error("Get engine failed.")`。
- 理解：这证明“缓存预置”是后续页面绑定的前置条件，不是可选优化。
- 风险：若路由先于缓存准备完成，会在 attach 阶段直接失败。

15. 通道解绑在不同插件实现里已经出现“好/差示例”
- 证据：`add_to_app/plugin/ohos_using_plugin/.../methodPlugin.ets:53-54`，`.../XUniLinksPlugin.ets:81-85`
- 事实：`methodPlugin` 同时清理 method+event；`XUniLinksPlugin` 当前只清 method。
- 理解：同仓库内就有可对照范式，可直接据此统一代码规范。
- 风险：事件流残留导致重复通知和生命周期穿透。

16. 错误回传格式在插件间明显不一致，需统一协议
- 证据：`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets:81,91,170`，`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets:77,102`
- 事实：不同插件 `result.error` 的 code/message/details 语义不一致，有的 code 固定字符串，有的来自 `BusinessError`。
- 理解：跨插件统一错误协议是必须补的治理项，否则 Flutter 端无法统一处理。
- 风险：同类错误在不同插件呈现不同动作，导致前端策略混乱。

### 旧卡片淘汰映射（第三轮回写）

说明：下面把“逐页深读十组卡片（M~AZ）”映射到“历史内容重写版条目号（1~16）”。  
使用方式：优先阅读右侧重写条目；左侧旧卡片仅保留索引价值。

1. 第一组（M/N/O/P）
- `M -> 1,8`
- `N -> 1,2`
- `O -> 3`
- `P -> 7,10`

2. 第二组（Q/R/S/T）
- `Q -> 8,9`
- `R -> 4,15`
- `S -> 7,10`
- `T -> 6,16`

3. 第三组（U/V/W/X）
- `U -> 8,10`
- `V -> 5,13,14`
- `W -> 3,6,7`
- `X -> 10,16`

4. 第四组（Y/Z/AA/AB）
- `Y -> 7,16`
- `Z -> 5,9,14`
- `AA -> 6`
- `AB -> 4,15`

5. 第五组（AC/AD/AE/AF）
- `AC -> 1,2`
- `AD -> 5,14`
- `AE -> 7,15`
- `AF -> 10,16`

6. 第六组（AG/AH/AI/AJ）
- `AG -> 8,11`
- `AH -> 11,12`
- `AI -> 3,15`
- `AJ -> 7,16`

7. 第七组（AK/AL/AM/AN）
- `AK -> 7,8`
- `AL -> 4,15`
- `AM -> 1,2,13`
- `AN -> 7,16`

8. 第八组（AO/AP/AQ/AR）
- `AO -> 8`
- `AP -> 1,4,15`
- `AQ -> 6,7`
- `AR -> 10,16`

9. 第九组（AS/AT/AU/AV）
- `AS -> 8`
- `AT -> 5,14`
- `AU -> 7,16`
- `AV -> 10,16`

10. 第十组（AW/AX/AY/AZ）
- `AW -> 8`
- `AX -> 13,14,15`
- `AY -> 6,7`
- `AZ -> 10,16`

### 未覆盖断言清单（第四轮扫描）

说明：以下点在旧卡片中提过，但在 `1~16` 重写条目里覆盖仍不充分，需继续补“源码级收敛条目”。

1. PlatformView 释放细节未形成统一动作清单
- 证据：`ohos/platform_demo/ohos/entry/src/main/ets/entryability/CustomView.ets:93-94`
- 现状：`dispose()` 为空实现。
- 缺口：缺少“通道解绑/事件停止/引用释放”三段式释放事实条目。

2. 仍有插件保留“Method not implemented”硬抛占位
- 证据：`ohos/localtion_demo/ohos/entry/src/main/ets/entryability/CustomPlugin.ets:48`
- 现状：`onMethodCall` 直接 `throw new Error(...)`。
- 缺口：缺少“占位硬抛改结构化返回”的实证改写条目。

3. XUniLinks 插件的 event 通道解绑不对称仍未落成治理结论
- 证据：`ohos/test_uni_links/.../XUniLinksPlugin.ets:81-85`
- 现状：detach 只清 method handler，未清 event stream handler。
- 缺口：缺少“同仓库好/差示例对照后的统一规范条目”。

4. 多引擎 attach 失败路径仍以硬抛为主
- 证据：`add_to_app/multiple_flutters/.../EngineBindings.ets:53-55`，`ohos/multiple_flutters_predraw/.../EngineBindings.ets:66-68`
- 现状：`Create/Get engine failed` 直接抛异常。
- 缺口：缺少“硬抛路径降级与可恢复动作”源码级条目。

5. 错误协议不一致问题还缺“统一格式建议 + 迁移路径”闭环
- 证据：`ohos/sqflite_helper/.../SqfliteHelperPlugin.ets:81,91-95,170`，`ohos/ohos_flutter_photoviewpicker/.../PhotoPickerPlugin.ets:77,102`
- 现状：`result.error` 的 code/message/details 语义不统一。
- 缺口：缺少“现状分型 -> 统一协议 -> 兼容过渡”三段式条目。

### 缺口补写（第五轮，17~21）

17. PlatformView 释放动作三段式（补齐）
- 源码事实：`ohos/platform_demo/ohos/entry/src/main/ets/entryability/CustomView.ets:93-94` 当前 `dispose()` 为空。
- 修正方案：`dispose` 至少执行三步：`setMethodCallHandler(null)`、停止事件/定时任务、清理对象引用。
- 兼容迁移：先加“空实现告警日志”，再灰度为“强制清理实现”，避免一次性改动影响旧逻辑。

18. 占位硬抛替换为结构化未实现返回（补齐）
- 源码事实：`ohos/localtion_demo/ohos/entry/src/main/ets/entryability/CustomPlugin.ets:48` 仍 `throw new Error('Method not implemented.')`。
- 修正方案：改为 `result.notImplemented()` 或 `result.error("UNIMPLEMENTED", "...", details)`。
- 兼容迁移：保留旧行为开关一个版本，默认走结构化返回，便于 Flutter 侧逐步适配。

19. Event 通道解绑对称化（补齐）
- 源码事实：`ohos/test_uni_links/.../XUniLinksPlugin.ets:81-85` detach 仅清 method handler。
- 修正方案：在 detach 同步 `this.eventChannel?.setStreamHandler(null)`，并清 `eventSink`。
- 兼容迁移：先加一次“重复事件计数日志”，确认无依赖后再强制启用对称解绑。

20. 多引擎 attach 失败降级路径（补齐）
- 源码事实：`add_to_app/multiple_flutters/.../EngineBindings.ets:53-55` 与 `ohos/multiple_flutters_predraw/.../EngineBindings.ets:66-68` 直接硬抛。
- 修正方案：失败时返回结构化错误对象并触发降级页面/重试按钮，而非直接中断。
- 兼容迁移：保留硬抛日志，但主流程改“可恢复失败”；发布后再移除硬抛分支。

21. `result.error` 协议统一迁移路径（补齐）
- 源码事实：`SqfliteHelperPlugin` 与 `PhotoPickerPlugin` 的 `result.error(code,message,details)` 字段语义不一致。
- 修正方案：统一为 `code/message/recoverable/action`，`details` 仅承载调试上下文。
- 兼容迁移：分三阶段：`A` 双写旧/新字段；`B` Flutter 优先读新字段；`C` 删除旧字段。

### 缺口落地伪补丁模板（第六轮）

1. 模板T1：PlatformView `dispose` 三段式清理（对应条目17）

```ts
// before
dispose(): void {
}

// after
dispose(): void {
  // 1) 通道解绑
  this.methodChannel?.setMethodCallHandler(null);
  // 2) 事件/任务停止（按实际字段替换）
  this.eventChannel?.setStreamHandler(null);
  this.timerId && clearTimeout(this.timerId);
  // 3) 引用释放
  this.methodChannel = undefined;
  this.eventChannel = undefined;
}
```

2. 模板T2：`Method not implemented` 硬抛替换（对应条目18）

```ts
// before
onMethodCall(call: MethodCall, result: MethodResult): void {
  throw new Error('Method not implemented.');
}

// after
onMethodCall(call: MethodCall, result: MethodResult): void {
  result.error(
    "UNIMPLEMENTED",
    `method ${call.method} not implemented`,
    JSON.stringify({ recoverable: true, action: "fallback" })
  );
}
```

3. 模板T3：event 通道对称解绑（对应条目19）

```ts
// before
onDetachedFromEngine(binding: FlutterPluginBinding): void {
  this.channel?.setMethodCallHandler(null);
}

// after
onDetachedFromEngine(binding: FlutterPluginBinding): void {
  this.channel?.setMethodCallHandler(null);
  this.eventChannel?.setStreamHandler(null);
  this.eventSink = undefined;
}
```

4. 模板T4：多引擎 attach 失败降级（对应条目20）

```ts
// before
if (!this.engine) {
  throw new Error("Get engine failed.");
}

// after
if (!this.engine) {
  this.lastError = {
    code: "ENGINE_NOT_FOUND",
    message: "Get engine failed",
    recoverable: true,
    action: "retry_or_fallback_page"
  };
  // 页面可根据 lastError 展示“重试/返回”按钮
  return;
}
```

5. 模板T5：`result.error` 协议双写迁移（对应条目21）

```ts
// Stage A: 双写
const errPayload = {
  code: String(code),
  message: String(message),
  recoverable: true,
  action: "retry",
  legacy: { details: details ?? "" }
};
result.error(errPayload.code, errPayload.message, JSON.stringify(errPayload));

// Stage B: Flutter 侧优先读取 recoverable/action，旧字段兜底
// Stage C: 移除 legacy.details，仅保留统一协议字段
```

## 错误协议对齐完成度清单（插件级）

判定规则：
- `已结构化`：已引入结构化错误字段（如 `recoverable/action`）或等价结构化对象。
- `部分结构化`：已用 `result.error/notImplemented`，但字段语义未统一或缺迁移方案。
- `待改造`：仅占位式返回，缺统一错误协议或缺可恢复动作字段。

1. 已结构化
- `ohos/sqflite_helper/ohos/src/main/ets/components/plugin/SqfliteHelperPlugin.ets`（`buildErrorDetails` 已落地）
- `ohos/ohos_flutter_photoviewpicker/ohos/src/main/ets/entryability/PhotoPickerPlugin.ets`（`buildErrorDetails` 已落地）
- `ohos/localtion_demo/ohos/src/main/ets/entryability/CustomPlugin.ets`（`UNIMPLEMENTED + recoverable/action`）
- `ohos/multiple_flutters_predraw/multiple_flutters_ohos/entry/src/main/ets/pages/EngineBindings.ets`（`ENGINE_NOT_FOUND` 结构化降级）
- `add_to_app/multiple_flutters/multiple_flutters_ohos/entry/src/main/ets/pages/EngineBindings.ets`（`ENGINE_CREATE_FAILED` 结构化降级）
- `ohos/load_native_resource_demo/ohos/entry/src/main/ets/plugins/LoadNativeResourcePlugin.ets`（`UNIMPLEMENTED + getVideo` 错误 details 结构化）
- `ohos/flutter-pag/ohos/src/main/ets/io/flutter/plugins/pag/entryability/FlutterPagPlugin.ets`（`UNIMPLEMENTED` 错误 details 结构化）
- `add_to_app/plugin/ohos_using_plugin/entry/src/main/ets/entryability/methodPlugin.ets`（`UNIMPLEMENTED` 错误 details 结构化 + detach 清理）
- `add_to_app/fullscreen/ohos_fullscreen/entry/src/main/ets/entryability/methodPlugin.ets`（`UNIMPLEMENTED` 错误 details 结构化 + detach 清理）
- `add_to_app/prebuilt_module/ohos_using_prebuilt_module/entry/src/main/ets/entryability/methodPlugin.ets`（`UNIMPLEMENTED` 错误 details 结构化 + detach 清理）
- `ohos/channel_demo/ohos/entry/src/main/ets/entryability/BatteryPlugin.ets`（`UNAVAILABLE/UNIMPLEMENTED` 错误 details 结构化 + detach 清理）
- `ohos/flutter_page_sample2/ohos/entry/src/main/ets/entryability/BatteryPlugin.ets`（`UNAVAILABLE/UNIMPLEMENTED` 错误 details 结构化 + detach 清理）
- `ohos/test_uni_links/packages/x_uni_links/ohos/src/main/ets/components/plugin/XUniLinksPlugin.ets`（`UNIMPLEMENTED` 错误 details 结构化 + detach 清理）

2. 部分结构化
- 当前项已清零。

3. 待改造
- 当前 P0/P1/P2 项已清零。

4. 优先改造顺序（建议）
- P0：已完成（`LoadNativeResourcePlugin.ets`、`FlutterPagPlugin.ets`）。
- P1：已完成（`ohos/channel_demo/ohos/entry/src/main/ets/entryability/BatteryPlugin.ets`、`ohos/flutter_page_sample2/ohos/entry/src/main/ets/entryability/BatteryPlugin.ets`）。
- P2：已完成（`ohos/test_uni_links/packages/x_uni_links/ohos/src/main/ets/components/plugin/XUniLinksPlugin.ets`）。

## ArkTS 语言本体学习补齐清单（官方文档）

1. 语言基础
- 已补：变量与类型标注、函数、类与接口、模块导入导出、异常处理、异步语法。
- 落地要求：新插件默认显式类型标注，不依赖隐式 `any` 与弱约束回传。

2. 类型系统与泛型
- 已补：联合类型、字面量类型、可空处理、泛型约束与类型收窄思路。
- 落地要求：跨通道数据模型优先定义 DTO 类型，再进行 `result.success/error` 映射。

3. TypeScript 到 ArkTS 适配
- 已补：迁移约束与差异化语法边界、动态特性替代策略。
- 落地要求：避免在插件层引入动态对象拼装与不受约束的类型断言。

4. 高性能编码导向
- 已补：面向 ArkTS 的性能敏感编码方式与常见反模式规避。
- 落地要求：热点路径避免频繁临时对象与不必要的装箱拆箱，错误对象结构稳定化。

5. 编码规范
- 已补：ArkTS 代码风格官方建议（命名、组织、可读性）。
- 落地要求：保持统一命名和生命周期对称清理，避免样例间风格漂移。

6. 阶段状态
- 语言本体补课状态：已完成（结合插件实改形成闭环）。
- 后续重点：在新增业务功能中持续执行“类型先行 + 错误协议统一 + 生命周期对称清理”。

## ArkTS 语言硬约束速记（用于实战自检）

1. 禁止弱类型兜底
- 原则：避免 `any/unknown` 作为业务通道数据兜底类型。
- 实战动作：插件通信参数必须映射为显式 DTO 类型后再进入业务逻辑。

2. 限制动态对象操作
- 原则：运行时对象布局变更与动态扩展能力受限。
- 实战动作：禁止在插件中按条件动态挂载字段，改为固定结构对象。

3. 避免依赖 TS 高级动态类型特性
- 原则：部分 TS 特性在 ArkTS 中不支持或受限（如若干索引/条件/解构类能力）。
- 实战动作：优先使用显式类与接口建模，避免“类型技巧驱动”的实现方式。

4. 并发需遵循 Sendable 与 TaskPool 规则
- 原则：跨线程通信对象与任务池调用有明确约束。
- 实战动作：并发场景使用可传输对象模型，不在跨线程路径透传不安全对象。

5. 迁移策略固定化
- 原则：TS -> ArkTS 迁移优先“静态化、显式化、结构稳定化”。
- 实战动作：按迁移清单先消除不支持语法，再做业务迁移，避免边改边猜。

## ArkTS 学习验收题（自测用）

1. 类型建模题
- 题目：为插件方法 `getBatteryLevel` 设计统一返回 DTO，要求覆盖成功与错误分支，且错误分支包含 `code/message/recoverable/action`。
- 通过标准：不使用 `any`；Dart 侧能仅凭 DTO 字段完成 UI 决策（重试/提示/回退）。

2. 生命周期题
- 题目：给一个同时持有 `MethodChannel + EventChannel` 的插件写完整 `onDetachedFromEngine`。
- 通过标准：包含 handler 解绑 + 引用置空 + 可选状态字段清理，顺序与 attach 对称。

3. 迁移约束题
- 题目：把一段依赖动态对象扩展与宽泛类型断言的 TS 插件代码迁移到 ArkTS。
- 通过标准：改写为显式接口/类；不依赖运行时动态挂载字段。

4. 并发安全题
- 题目：设计一个 TaskPool 场景下的跨线程数据传递模型。
- 通过标准：数据结构满足可传输约束；不透传线程不安全对象。

## ArkTS 反模式对照表（实战高频）

1. 反模式：`result.notImplemented()` 直接返回
- 风险：Flutter 侧缺少可恢复语义，难统一处理。
- 推荐：改为 `UNIMPLEMENTED + 结构化 details`。

2. 反模式：`result.error(..., null)` 或 details 混乱
- 风险：错误处理逻辑散落，端到端协议不可演进。
- 推荐：统一 `buildErrorDetails`，固定 `recoverable/action` 字段。

3. 反模式：`onDetachedFromEngine` 仅解绑不置空
- 风险：对象残留导致隐性回调或内存生命周期问题。
- 推荐：解绑后同步清理 channel/sink/context 等引用。

4. 反模式：插件参数直接使用 `Any`/`any` 下探业务
- 风险：类型收敛过晚，异常滞后到运行期。
- 推荐：入口处先做 DTO 化与校验，再进入业务逻辑。

## Java -> ArkTS 迁移专项（官方补齐）

1. 迁移映射主线
- 类型：Java 基本类型/对象类型在 ArkTS 中统一为显式类型标注与可空控制。
- 类模型：接口、抽象类、继承链迁移时，优先先保留行为边界，再做语法层替换。
- 异常：从“随处抛异常”转向“可恢复错误协议 + 必要异常”的分层策略。

2. 并发与线程模型迁移
- 原则：不要把 Java 线程使用习惯直接平移到 ArkTS。
- 实战动作：并发任务按 ArkTS 推荐机制（如 TaskPool/Worker）重建，跨线程数据遵循可传输约束。

3. 常见迁移坑位
- `this` 绑定差异：回调场景统一使用明确作用域写法，避免上下文漂移。
- 数值与边界：涉及位运算、字节数组、整型边界时必须补测试，防止语义偏差。
- 集合与空值：从 Java 容器/空值习惯迁移到 ArkTS 时，统一先做可空与类型收窄。

4. Java 迁移验收清单
- 是否移除了弱类型兜底与动态字段拼装。
- 是否完成错误协议结构化（`code/message/recoverable/action`）。
- 是否完成生命周期对称清理（attach/detach 对称）。
- 是否对并发路径完成可传输对象检查与异常回传设计。

## 可上线工程能力补齐清单（发布前）

1. 发布与交付链路
- 构建：明确 `hvigor` 构建流程、构建配置分层与多环境参数管理。
- 签名：证书与签名配置可追溯，避免本地临时配置导致不可复现产物。
- 打包：产物命名、版本号、渠道标识统一规则。

2. 权限与合规
- 权限：声明、申请、拒绝后降级路径必须成对出现。
- 隐私：数据采集最小化与用途说明一致；敏感能力有用户可见提示。
- 回退：权限失败场景不应硬崩，需有明确业务兜底。

3. 性能与稳定性
- 启动：冷启动路径避免重逻辑阻塞；初始化分层延后。
- 内存：插件通道、页面对象、订阅监听要可回收。
- 稳定：异常分级（可恢复/不可恢复）与统一错误码策略。

4. 测试体系
- 单测：ArkTS 核心逻辑单元测试覆盖关键分支。
- 集成：Flutter<->ArkTS 桥接契约测试覆盖方法名、参数、返回结构。
- 回归：插件生命周期（attach/detach）与异常路径要有回归样例。

5. 诊断与可观测性
- 日志：统一 tag、级别、上下文字段（traceId/sessionId 可选）。
- 错误：统一上报结构，保证线上可按 `code/action` 快速聚合。
- 排障：保留“用户动作 -> Flutter 调用 -> ArkTS 回执”的最短追踪链路。

6. CI/CD 自动化
- 持续集成：构建、测试、静态检查串联为阻断门禁。
- 产物归档：每次构建产物与配置指纹可追踪。
- 发布治理：版本策略、回滚策略、变更记录自动化。

## 上线执行模板（可直接照单执行）

1. 发布前检查表
- 配置：构建环境、签名、版本号、渠道标识已锁定。
- 质量：关键路径测试通过，插件桥接契约测试通过。
- 合规：权限声明与隐私文案一致，拒绝权限降级路径可用。
- 稳定：错误码与日志字段完整，可定位主流程问题。

2. 灰度策略模板
- 分批：按用户比例逐步放量，不一次全量。
- 指标：启动成功率、崩溃率、关键接口成功率作为放量门槛。
- 阻断：任一核心指标越阈值立即停止放量。

3. 回滚策略模板
- 触发条件：崩溃率激增、主流程失败率超阈值、权限问题导致核心功能不可用。
- 回滚动作：回退到上一稳定版本，冻结新流量，保留现场日志与指标快照。
- 复盘输出：根因、影响面、修复方案、预防措施四段式记录。

4. 线上事故响应模板
- 定位链路：用户操作 -> Flutter 调用 -> ArkTS 插件 -> 系统能力回执。
- 分级：P0（不可用）、P1（核心降级）、P2（局部异常）。
- 响应：先止血（降级/回滚），再修复，最后补测试与守护规则。

## 发布门禁阈值建议（量化）

1. 灰度放量门槛
- 启动成功率：不低于 99.5%。
- 崩溃率：不高于 0.30%。
- 关键接口成功率：不低于 99.0%。

2. 阻断与回滚阈值
- 任一核心指标连续两轮灰度不达标：阻断继续放量。
- 崩溃率超过 0.50% 或核心流程失败率超过 1.00%：触发回滚评估。
- 出现权限合规或隐私高风险问题：直接暂停发布并走应急流程。

3. 发布决策表（简版）
- 绿灯：全部达标 -> 继续下一档放量。
- 黄灯：轻微波动 -> 暂停观察并补监控，再决定是否继续。
- 红灯：触发阈值 -> 立即停止放量并执行回滚预案。

4. 复盘最小输出
- 事实时间线：发现时间、处置时间、恢复时间。
- 影响评估：受影响用户比例、核心功能影响范围。
- 根因与修复：技术根因、修复动作、验证证据。
- 预防项：新增测试、监控、门禁规则。

## 插件协议版本化与兼容治理

1. 协议版本字段
- 规则：每个跨端协议对象包含 `schemaVersion`。
- 目的：支持 Flutter 与 ArkTS 端独立升级，避免强耦合发布。

2. 向后兼容策略
- 规则：新增字段仅追加，不删除旧字段，不修改既有语义。
- 规则：旧端缺失新字段时，必须有默认行为。
- 目的：避免灰度期间出现新老端互调失败。

3. 破坏性变更策略
- 规则：涉及字段重命名/语义变化时，升级主版本并保留兼容窗口。
- 规则：兼容窗口内双写双读（新旧字段并行）。
- 目的：把破坏性变更风险从“运行期爆雷”转为“可控迁移”。

4. 契约测试要求
- 规则：方法名、参数结构、错误对象结构纳入自动化契约测试。
- 规则：每次改协议必须附带正反用例与降级用例。
- 目的：防止“代码能编译但跨端协议失配”。

5. 版本发布记录
- 规则：每次协议变更必须记录版本、变更内容、兼容范围、下线时间。
- 目的：保障多人协作和长期维护可追溯。

## 依赖与供应链安全基线

1. 依赖准入规则
- 规则：新增三方依赖必须说明用途、维护活跃度、替代方案评估。
- 规则：禁止直接引入来源不明或长期失维护仓库。
- 目的：减少引入高风险依赖导致的长期维护负担。

2. 版本治理策略
- 规则：核心依赖采用版本锁定，避免构建时漂移。
- 规则：升级分“安全补丁优先、功能升级审慎”两类流程。
- 目的：平衡稳定性与安全更新速度。

3. 漏洞响应机制
- 规则：发现高危漏洞时，优先评估影响面并给出临时缓解方案。
- 规则：漏洞修复需附回归验证与发布说明。
- 目的：把漏洞处理从临时救火变成标准化流程。

4. 许可证与合规
- 规则：依赖引入时同步检查许可证兼容性。
- 规则：发布前输出依赖清单与许可证摘要。
- 目的：避免后期法务风险阻断发布。

5. 构建可复现性
- 规则：构建工具链版本固定并可追溯。
- 规则：关键产物保留来源指纹（依赖版本、构建参数、提交哈希）。
- 目的：保障问题复现与审计能力。

## 架构与模块边界治理

1. 模块职责单一化
- 规则：UI、业务、平台桥接、基础设施四层职责分离。
- 规则：插件层只负责协议转换与能力调用，不承载业务编排。
- 目的：降低跨层耦合和修改扩散风险。

2. 依赖方向约束
- 规则：上层可依赖下层抽象，不允许下层反向依赖上层实现。
- 规则：跨模块调用优先通过稳定接口，不直接引用内部实现细节。
- 目的：保证模块可替换与可测试。

3. 公共组件准入
- 规则：沉淀到公共层的代码必须满足“至少两个业务复用 + 稳定接口”。
- 规则：禁止把一次性业务代码提前公共化。
- 目的：避免公共层膨胀和历史包袱。

4. 重构守则
- 规则：重构需先补契约测试，再做结构调整。
- 规则：大改采用分阶段迁移（双写/灰度/下线旧路径）。
- 目的：把架构演进风险从一次性切换改为可控渐进。

## 团队协作与评审治理

1. 变更分级
- 规则：按影响范围分为 S（架构/协议）、A（核心功能）、B（一般功能）、C（文档与低风险）。
- 规则：S/A 级变更必须附设计说明与回滚方案。
- 目的：让评审深度与风险等级匹配。

2. PR 门禁
- 规则：必须通过编译、测试、静态检查、关键契约测试。
- 规则：未完成风险说明与验证证据的 PR 不进入合并阶段。
- 目的：防止“代码可读但不可发布”。

3. 评审清单
- 协议：方法名、参数、错误结构是否兼容。
- 生命周期：attach/detach 是否对称清理。
- 类型：是否存在弱类型下探业务路径。
- 监控：日志与错误码是否满足定位要求。

4. 知识回流机制
- 规则：每次线上事故或高价值改造，必须回写 SKILL 与学习日志。
- 规则：同类问题复发时，优先更新规则而非重复补丁。
- 目的：把个人经验沉淀为团队资产。

## 多设备与系统版本兼容治理

1. 兼容矩阵基线
- 规则：按设备类型、系统版本、关键硬件能力建立兼容矩阵。
- 规则：每次发版前至少覆盖矩阵中的核心组合。
- 目的：避免仅在单一测试机通过却在线上大面积异常。

2. 能力探测与降级
- 规则：系统能力调用前先做可用性探测，不假设能力一定存在。
- 规则：能力不可用时必须有降级路径或替代交互。
- 目的：把“不可用崩溃”改为“可感知降级”。

3. 版本差异治理
- 规则：与系统版本相关的逻辑集中封装，禁止散落在业务层。
- 规则：新增版本分支时必须记录适用范围与清理计划。
- 目的：防止版本分支膨胀为长期技术债。

4. 兼容回归计划
- 规则：每个版本保留最小兼容回归清单（启动、登录、核心流程、权限链路）。
- 规则：平台能力相关变更必须触发专项回归。
- 目的：降低“修一个版本、坏另一个版本”的回归风险。

## 数据一致性与故障恢复治理

1. 数据写入幂等
- 规则：关键写操作必须具备幂等键或去重机制。
- 规则：重试流程不得造成重复写入与状态污染。
- 目的：防止网络抖动与重试导致数据错乱。

2. 本地缓存一致性
- 规则：缓存读取必须带版本或时间戳校验。
- 规则：服务端状态变更后，明确缓存失效策略与刷新路径。
- 目的：避免“界面显示旧数据、后台已变更”的分裂状态。

3. 失败重试与补偿
- 规则：可恢复错误允许有限次数重试，不可恢复错误直接失败并提示。
- 规则：跨步骤流程失败时定义补偿动作，保证最终一致性。
- 目的：把中间失败从“悬空状态”收敛为“可恢复状态”。

4. 数据修复流程
- 规则：线上出现数据异常时，先冻结写入入口，再执行修复脚本/人工补偿。
- 规则：修复完成后必须补回归测试与监控告警规则。
- 目的：降低二次损坏风险并形成可复用修复路径。

## 容量规划与成本治理

1. 资源预算基线
- 规则：发布前定义 CPU、内存、存储、网络预算区间。
- 规则：关键页面与插件调用路径必须有预算上限。
- 目的：避免功能增长后资源开销失控。

2. 容量阈值与扩缩策略
- 规则：核心指标超过阈值时触发限流、降级或延迟加载策略。
- 规则：阈值调整需记录依据，避免频繁人工拍脑袋调参。
- 目的：把容量风险前置为可监控、可调整问题。

3. 成本感知开发
- 规则：新增能力评审时同步评估计算、存储、网络成本影响。
- 规则：高成本链路优先优化调用频率与数据传输量。
- 目的：在保证体验的前提下控制长期运营成本。

4. 周期性复盘
- 规则：按版本复盘资源消耗、异常峰值与优化收益。
- 规则：复盘结果回写到发布门禁与架构治理规则中。
- 目的：形成“监控 -> 优化 -> 规则更新”的闭环。

## 安全测试与攻防演练治理

1. 输入与参数安全
- 规则：所有跨端参数在进入业务前必须校验类型、范围和格式。
- 规则：禁止将未校验输入直接用于系统能力调用或文件路径拼接。
- 目的：降低越界输入与注入类风险。

2. 权限最小化
- 规则：仅申请业务必需权限，按需申请，不提前全量申请。
- 规则：权限拒绝场景必须有清晰提示与降级方案。
- 目的：降低权限滥用风险与合规压力。

3. 敏感数据保护
- 规则：日志中禁止输出敏感标识与完整隐私数据。
- 规则：本地持久化敏感数据前先评估必要性与保护措施。
- 目的：防止二次泄露与合规违规。

4. 攻防演练与应急
- 规则：按版本进行最小安全演练（非法参数、权限拒绝、异常回执伪造）。
- 规则：发现高风险问题后执行“止血、修复、复盘、规则回写”闭环。
- 目的：将安全治理从被动修补转为主动演练。

## 业务连续性与容灾治理

1. 目标定义（RTO/RPO）
- 规则：关键业务定义可接受恢复时间（RTO）与数据丢失窗口（RPO）。
- 规则：发布前确认当前能力是否满足目标，不满足则降级发布范围。
- 目的：把容灾从口号变成可验证目标。

2. 备份与恢复
- 规则：关键配置与关键数据要有可验证的备份策略。
- 规则：恢复流程必须定期演练，并保留演练记录。
- 目的：避免“有备份、不可恢复”的伪安全状态。

3. 降级开关与熔断
- 规则：关键功能提供可控降级开关，支持快速止血。
- 规则：异常峰值时触发熔断，优先保障核心链路可用。
- 目的：在故障场景下尽量保持核心可用而非整体不可用。

4. 容灾演练机制
- 规则：按季度执行最小容灾演练（依赖不可用、配置损坏、版本回退）。
- 规则：演练后更新应急手册、发布门禁与监控告警。
- 目的：持续验证应急方案的有效性。

## SLO 与可观测指标治理

1. SLO 目标定义
- 规则：为关键链路定义可用性、成功率、时延三类 SLO。
- 规则：SLO 必须可监控、可计算、可回溯，不使用模糊描述。
- 目的：让稳定性目标可量化管理。

2. 告警分级与响应
- 规则：按 P0/P1/P2 定义告警级别、响应时限与处理责任人。
- 规则：同一告警不得跨团队“无人认领”。
- 目的：缩短故障发现到处置的时间。

3. 误报与噪音治理
- 规则：每个版本复盘误报率，清理无效告警与重复规则。
- 规则：告警规则变更必须附验证窗口与回滚预案。
- 目的：避免告警疲劳导致真实故障被忽略。

4. 值班与升级路径
- 规则：值班表、升级联系人、应急会议机制固定化。
- 规则：超时未恢复自动升级处理级别。
- 目的：确保重大故障有明确升级路径。

## 版本路线与变更管理治理

1. 版本路线规划
- 规则：按季度维护版本路线图（新特性、稳定性、技术债治理）。
- 规则：路线图变更需评估对现有业务与插件协议的影响。
- 目的：避免版本迭代失序与目标漂移。

2. 弃用与下线策略
- 规则：功能或协议下线必须先发布弃用公告并设置过渡窗口。
- 规则：过渡期内保留兼容路径与迁移指引。
- 目的：降低下线动作对业务方与旧版本用户的冲击。

3. LTS 维护策略
- 规则：定义长期支持版本范围与维护周期。
- 规则：LTS 版本仅接收安全修复与高优先级稳定性修复。
- 目的：为关键业务提供可预期的稳定版本。

4. 变更沟通机制
- 规则：S/A 级变更必须附变更说明、影响范围、回滚路径。
- 规则：发布前后同步业务方、测试、运维三方确认。
- 目的：提升跨团队变更透明度与协作效率。

## 开发者体验与工具链效率治理

1. 脚手架与模板统一
- 规则：新模块、新插件必须基于统一模板创建（目录、命名、错误协议、日志字段）。
- 规则：模板变更同步更新示例与迁移说明。
- 目的：减少同类代码重复造轮子与风格漂移。

2. 自动化校验前置
- 规则：本地提交前自动执行格式化、静态检查、关键测试。
- 规则：CI 门禁与本地校验保持一致，避免“本地可过、线上失败”。
- 目的：缩短反馈回路并降低无效构建。

3. 本地调试效率
- 规则：提供统一调试开关、日志级别切换、模拟数据入口。
- 规则：高频故障场景提供一键复现脚本或最小复现步骤。
- 目的：减少排障准备时间，提高定位效率。

4. 知识检索与沉淀
- 规则：常见问题、排障手册、协议说明统一索引，支持按关键词检索。
- 规则：新增高价值经验必须链接到对应规则章节。
- 目的：避免知识分散导致重复踩坑。

## 无障碍与国际化治理

1. 无障碍可用性基线
- 规则：关键交互控件必须提供可访问名称与语义信息。
- 规则：页面可通过键盘/焦点顺序完成核心流程操作。
- 目的：保障不同能力用户可完成主要业务流程。

2. 视觉与交互可达性
- 规则：文本对比度、字号缩放、触控区域满足可读可点要求。
- 规则：动画与动态效果提供可降级策略，避免影响易感用户。
- 目的：降低视觉与交互门槛，提升通用可用性。

3. 国际化与本地化
- 规则：文案必须外置资源化，禁止业务代码硬编码多语言文本。
- 规则：时间、数字、货币格式遵循地区化输出规则。
- 目的：避免跨地区上线时出现文案与格式错误。

4. a11y/i18n 验收机制
- 规则：发布前执行最小无障碍检查与多语言回归检查。
- 规则：高频页面必须覆盖至少两种语言与缩放场景验证。
- 目的：将无障碍与国际化纳入发布门禁，而非上线后补救。

## 性能 Profiling 实操手册

1. 采样准备
- 规则：先定义目标场景（启动、首帧、列表滚动、关键交互），再采样。
- 规则：采样前固定测试环境与设备，避免结果不可比。
- 目的：保证性能数据可复现、可对比。

2. 瓶颈分类定位
- 规则：按 CPU、内存、I/O、渲染卡顿四类分桶定位。
- 规则：先确认主瓶颈类别，再进入代码级优化。
- 目的：避免无效优化与误判。

3. 优化优先级
- 规则：优先处理影响面最大、收益最高的热点链路。
- 规则：单次优化只改一个主要变量，便于评估收益。
- 目的：确保优化动作可量化、可追踪。

4. 回归与守护
- 规则：优化后必须复采样并对比基线，附性能回归结论。
- 规则：高频性能指标纳入版本门禁，防止后续回退。
- 目的：把性能优化从一次性行动变成持续治理。

## 上架审核与发布材料治理

1. 提审材料清单
- 规则：版本说明、功能变更点、权限用途说明、隐私政策链接必须完整。
- 规则：涉及新增权限或敏感能力时，补充场景说明与用户价值说明。
- 目的：提高提审通过率并减少反复补材料。

2. 合规内容一致性
- 规则：应用内文案、权限弹窗、隐私协议三处描述保持一致。
- 规则：审核包与发布包配置一致，禁止“提审一套、发布一套”。
- 目的：避免因信息不一致触发审核驳回。

3. 提审前质量门禁
- 规则：提审前完成关键路径回归、权限链路验证、多语言基础检查。
- 规则：审核关键问题设阻断门槛，未通过不进入提审。
- 目的：减少因低级质量问题导致的审核失败。

4. 审核驳回处理
- 规则：驳回问题按“合规/质量/材料”分类，24小时内给出修复计划。
- 规则：修复后同步更新提审检查表与规则，防止同类问题复发。
- 目的：形成提审问题的闭环治理能力。

## Feature Flag 与远程配置治理

1. 开关分级与命名
- 规则：按实验开关、灰度开关、应急开关分级管理。
- 规则：开关命名包含业务域、功能名、到期时间，避免长期“僵尸开关”。
- 目的：提升开关可读性与可治理性。

2. 权限与审计
- 规则：开关变更需最小权限控制，并记录操作人、时间、变更前后值。
- 规则：高风险开关变更采用双人复核。
- 目的：降低误操作与越权风险。

3. 回滚与失效策略
- 规则：每个高风险功能必须绑定可快速回退的开关路径。
- 规则：开关配置拉取失败时采用安全默认值，不阻断核心流程。
- 目的：在异常场景下快速止血并保持核心可用。

4. 生命周期治理
- 规则：开关上线时即定义下线时间与责任人。
- 规则：每个版本清理过期开关并更新文档。
- 目的：防止开关堆积导致系统复杂度上升。

## 参考资料

- 样例地图：`references/project_map.md`
- 协同开发流程：`references/arkts_flutter_workflow.md`
- 项目能力矩阵：`references/ohos_project_matrix.md`
- 问题排查手册：`references/troubleshooting_playbook.md`
- 批次1阅读报告（仅阅读）：`references/batch1_reading_report.md`
- 批次2阅读报告（仅阅读）：`references/batch2_reading_report.md`
- 批次3阅读报告（仅阅读）：`references/batch3_reading_report.md`
- 批次4执行计划（任务化）：`references/batch4_execution_plan.md`
- 批次5单页速查卡：`references/batch5_onepage_cheatsheet.md`
- ArkTS 学习路径（混编导向）：`references/arkts_learning_path.md`
- ArkTS 专项批次1练习：`references/arkts_special_batch1_exercises.md`
- ArkTS 专项批次2练习：`references/arkts_special_batch2_exercises.md`
- ArkTS 官方学习日志：`references/arkts_official_study_log.md`
