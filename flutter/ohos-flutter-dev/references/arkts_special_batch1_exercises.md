# ArkTS 专项批次1练习清单

## 范围

只覆盖四块基础能力：

1. ArkTS 页面与状态
2. Ability 生命周期
3. Want 与路由参数
4. 权限与系统能力接入

## 使用规则

1. 每题都要写“目标、改动文件、预期结果、风险点”。
2. 本清单当前定位为阅读与改造训练，不要求实际运行。
3. 如后续转实测模式，可在每题后补“运行验证记录”。

## A 组：页面与状态（入门）

### A1. 页面结构改造

- 目标：在 ArkTS 页面中新增一个区域（标题+按钮+状态文本）。
- 建议样例：`ohos/flutter_page_sample1/ohos/entry/src/main/ets/pages/Page1.ets`
- 验收：
  - 能正确使用 `@State` 驱动文本变化。

### A2. 本地状态联动

- 目标：实现按钮点击次数统计并显示。
- 建议样例：`ohos/component_demo` 任一页面。
- 验收：
  - 状态变更路径清晰，避免全局变量直接写 UI。

### A3. 页面级存储联动

- 目标：用 `@StorageLink` 或 `LocalStorageLink` 共享一个页面变量。
- 建议样例：`add_to_app/books/ohos_books/entry/src/main/ets/pages/Index.ets`
- 验收：
  - 能解释本地状态与存储状态的差异。

## B 组：Ability 生命周期（基础）

### B1. 生命周期埋点梳理

- 目标：梳理并记录页面生命周期触发顺序。
- 建议样例：`ohos/flutter_page_sample2`
- 验收：
  - 形成时序说明：`aboutToAppear -> onPageShow -> onPageHide -> aboutToDisappear`。

### B2. FlutterEntry 生命周期透传

- 目标：确认并说明 `FlutterEntry` 生命周期调用点。
- 建议样例：`ohos/flutter_page_sample1`、`ohos/flutter_page_sample2`
- 验收：
  - 能指出遗漏调用会导致的潜在问题。

### B3. Back 行为处理

- 目标：分析并调整 `onBackPress` 分发策略。
- 建议样例：`flutter_page_sample1`、`add_to_app/*_ohos`。
- 验收：
  - 明确是路由返回、事件广播还是 Flutter 侧处理优先。

## C 组：Want 与路由参数（进阶）

### C1. Want 参数读取

- 目标：从 Want/路由参数中读取业务字段并展示。
- 建议样例：`ohos/flutter_page_sample2`（带参数入口）
- 验收：
  - 参数缺失时有默认值和兜底。

### C2. NewWant 处理链路

- 目标：梳理冷启动与热启动参数的统一处理。
- 建议样例：`ohos/test_uni_links/packages/x_uni_links/ohos/.../XUniLinksPlugin.ets`
- 验收：
  - 能解释 `onAttachedToAbility` 与 `onNewWant` 的分工。

### C3. ArkTS 到 Flutter 参数透传

- 目标：把 ArkTS 参数传给 Flutter 页面并记录入口读取位置。
- 建议样例：`flutter_page_sample2`。
- 验收：
  - 参数命名与数据结构有统一约定。

## D 组：权限与系统能力（进阶）

### D1. 权限声明核对

- 目标：找出样例中涉及权限的能力（相机、定位、文件）并定位声明位置。
- 建议样例：`ohos/testcamera`、`ohos/localtion_demo`、`ohos/load_native_resource_demo`
- 验收：
  - 形成“能力-权限-代码入口”对照表。

### D2. 权限失败分支设计

- 目标：为一条系统能力调用补充失败兜底说明（拒绝、超时、不可用）。
- 建议样例：`channel_demo` 或任一系统能力样例。
- 验收：
  - 错误路径有明确用户提示与日志记录点。

### D3. 系统能力封装

- 目标：把一段系统 API 调用抽象为单独 ArkTS 服务类。
- 建议样例：任选一条能力调用链。
- 验收：
  - 页面只保留 UI 与调用，不直接堆系统细节。

## 推荐提交模板（每题）

1. 题号：
2. 改动文件：
3. 改动摘要：
4. 关键代码点：
5. 风险点：
6. 后续优化：

## 通过标准

1. 能独立解释 ArkTS 页面与状态机制。
2. 能独立梳理 Ability 与页面生命周期。
3. 能独立设计 Want 参数与 Flutter 参数透传。
4. 能给出系统能力调用的权限与失败分支策略。
