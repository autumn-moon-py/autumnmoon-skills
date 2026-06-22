---
name: using-flutter
description: 在涉及 Flutter 业务开发、iOS 桥接或鸿蒙 Flutter 工程时使用——确立如何查找和使用 Flutter 技能，要求在任何响应（包括澄清性问题）之前调用 Skill 工具
---

<SUBAGENT-STOP>
如果你是作为子智能体被分派来执行特定任务的，跳过此技能。
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
如果你认为哪怕只有 1% 的可能性某个技能适用于你正在做的事情，你绝对必须调用该技能。

如果一个技能适用于你的任务，你没有选择。你必须使用它。

这不可协商。这不是可选的。你不能通过合理化来逃避。
</EXTREMELY-IMPORTANT>

## 指令优先级

Flutter 技能覆盖默认系统提示行为，但**用户指令始终具有最高优先级**：

1. **用户的明确指令**（CLAUDE.md、GEMINI.md、AGENTS.md、直接请求）——最高优先级
2. **Flutter 技能** ——在冲突处覆盖默认系统行为
3. **默认系统提示** ——最低优先级

## 规则

**在任何响应或操作之前调用相关或被请求的技能。** 哪怕只有 1% 的可能性某个技能适用，你都应该调用该技能来检查。如果调用后发现技能不适合当前情况，你不需要使用它。

## 红线

这些想法意味着停下——你在合理化：

| 想法 | 现实 |
|------|------|
| "这只是一个简单的问题" | 问题就是任务。检查技能。 |
| "我需要先了解更多上下文" | 技能检查在澄清性问题之前。 |
| "让我先探索一下代码库" | 技能告诉你如何探索。先检查。 |
| "这不需要正式的技能" | 如果技能存在，就使用它。 |
| "我记得这个技能" | 技能会迭代更新。阅读当前版本。 |
| "技能太小题大做了" | 简单的事会变复杂。使用它。 |
| "让我先做这一件事" | 在做任何事之前先检查。 |

## 技能优先级

当多个技能可能适用时，使用此顺序：

1. **流程技能优先**（头脑风暴、调试）- 这些决定如何处理任务
2. **平台技能其次**（flutter-app-dev、flutter-ios-bridge-dev、ohos-flutter-dev）- 这些指导执行

"给 Flutter 加个新页面" -> 先头脑风暴，再使用 flutter-app-dev。
"Flutter 调不到 iOS 原生方法" -> 先调试，再使用 flutter-ios-bridge-dev。
"鸿蒙 Flutter 插件报错" -> 先调试，再使用 ohos-flutter-dev。

## 技能类型

**刚性的**（调试、验证）：严格遵循。不要偏离纪律。

**灵活的**（模式、架构）：根据上下文调整原则。

技能本身会告诉你它属于哪种。

## 用户指令

指令说明做什么，而非怎么做。"添加 X"或"修复 Y"不意味着跳过工作流。

# Flutter 技能索引

## flutter-app-dev

**触发：** 进行纯 Flutter 业务开发、页面搭建、状态管理、导航设计、布局调整、主题与动画优化、测试补齐或面向可维护性的 Flutter 重构。

**适用：** 新增或重构纯 Flutter 页面；梳理功能模块的分层和状态边界；处理列表页/详情页/表单页/流程页的导航与布局；统一主题/按钮/颜色/字体和动效；为 Repository/ViewModel/Widget 或集成流程补测试。

**不适用：** Flutter 与 iOS 原生之间的桥接层开发（用 flutter-ios-bridge-dev）；鸿蒙 Flutter 混编工程（用 ohos-flutter-dev）；纯 iOS 原生业务（用 ios-swift-dev）。

## flutter-ios-bridge-dev

**触发：** 处理 Flutter 与 iOS 原生之间的边界层开发，包括 MethodChannel、EventChannel、BasicMessageChannel、Pigeon、Flutter Plugin、PlatformView、原生页面嵌入、双端联调与测试。

**适用：** Flutter 需要调用 iOS 原生 API；需要设计 MethodChannel/EventChannel/BasicMessageChannel 或 Pigeon 协议；需要将 iOS SDK/系统能力或原生页面暴露给 Flutter；需要封装 Flutter Plugin 或拆分 Plugin 工程结构；需要处理 Dart 与 Swift 的数据映射/错误映射/线程切换；需要排查注册失败/消息不通/线程错误/PlatformView 显示异常等联调问题。

**不适用：** 纯 Flutter 页面开发/主题/状态管理/导航（用 flutter-app-dev）；纯 iOS 原生业务页面/网络/安全/性能（用 ios-swift-dev）；不涉及 Flutter 与 iOS 边界层的任务。

## ohos-flutter-dev

**触发：** 鸿蒙 Flutter 工程开发——混编工程落地、插件协议治理、发布门禁与运行治理。

**适用：** HarmonyOS + Flutter 混编工程落地；Flutter 插件在鸿蒙端的协议开发与治理；鸿蒙 Flutter 工程的生命周期与资源清理；测试与门禁；发布/灰度/回滚/事故响应。

**不适用：** ArkTS 语言学习与迁移设计（用 ohos-arkts-dev）；纯 Flutter 业务页面不涉及鸿蒙特有逻辑（用 flutter-app-dev）；纯 iOS 桥接问题（用 flutter-ios-bridge-dev）。
