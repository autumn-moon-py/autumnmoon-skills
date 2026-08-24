---
name: flutter-ios-bridge-dev
description: Use when 处理 Flutter 与 iOS 原生之间的边界层开发，包括 MethodChannel、EventChannel、BasicMessageChannel、Pigeon、Flutter Plugin、PlatformView、原生页面嵌入、双端联调与测试；不适用于纯 Flutter 页面开发或纯 iOS 原生业务实现。
---

# Flutter iOS 桥接开发

## 概述

这个技能专门处理 Flutter 与 iOS 原生之间的边界层问题。

它覆盖两类工作：

1. 在业务 App 内直接把 iOS 原生能力桥接给 Flutter。
2. 将 iOS 能力封装成可复用的 Flutter Plugin。

只要任务涉及通道设计、Pigeon、Plugin 结构、`PlatformView`、原生页面嵌入、双端联调或错误映射，就应该优先使用这个技能。

如需具体模式，请按需读取以下引用资料：

- `references/method-channel-pigeon.md`
- `references/plugin-architecture.md`
- `references/ios-embedding-and-platform-view.md`
- `references/interop-testing-debugging.md`

## 适用场景

- Flutter 需要调用 iOS 原生 API。
- 需要设计 `MethodChannel`、`EventChannel`、`BasicMessageChannel` 或 `Pigeon` 协议。
- 需要将 iOS SDK、系统能力或原生页面暴露给 Flutter。
- 需要封装 Flutter Plugin 或拆分 Plugin 工程结构。
- 需要处理 Dart 侧与 Swift 侧的数据映射、错误映射、线程切换。
- 需要排查注册失败、消息不通、线程错误、`PlatformView` 显示异常等联调问题。

## 不适用场景

- 纯 Flutter 页面开发、主题、状态管理、导航。
- 纯 iOS 原生业务页面、网络、安全、性能治理。
- 不涉及 Flutter 与 iOS 边界层的任务。

## 核心规则

- 先选边界方案，再写桥接代码。直接桥接、Plugin 封装、`PlatformView` 嵌入是 3 类不同问题。
- 保持桥接层薄。通道处理器只负责协议转换和调用转发，不承载业务规则。
- 简单、低频、一次性的接口可以使用 `MethodChannel`；复杂、结构化、长期维护的接口优先 `Pigeon`。
- 高频事件流不要硬塞进请求-响应式通道；持续事件使用 `EventChannel` 或更合适的流式协议。
- iOS UI 相关能力必须明确主线程要求，不能在错误线程上回调 Flutter。
- 原生 UI 只有在 Flutter 难以稳定承载时才考虑 `PlatformView`；优先保持界面主体在 Flutter。
- 只要原生能力有复用价值，就尽快从业务 App 内直连升级为 Plugin 结构。
- Dart 侧和 Swift 侧都必须具备可测试性，不能把联调当成唯一验证方式。

## 工作流

### 1. 先确定桥接模式

- 业务 App 内部一次性能力接入：可先用直接桥接。
- 多页面、多功能、可复用能力：优先考虑 Plugin。
- 原生 UI 必须嵌入 Flutter：评估 `PlatformView` 或原生路由托管。

涉及通道选型与协议边界时，读取 `references/method-channel-pigeon.md`。

### 2. 明确协议与所有权

- 定义 Dart 侧调用入口、参数、返回值、错误语义。
- 定义 Swift 侧服务边界，不把 Apple API 细节暴露给 Dart。
- 统一空值、异常、枚举、线程和生命周期语义。

如果接口会长期维护，优先使用 `Pigeon` 生成类型安全的协议代码。

### 3. 决定工程承载方式

- 先判断是继续放在 App 工程内部，还是独立为 Plugin。
- Plugin 有复用价值时，拆分出 Dart API、平台实现、示例应用和测试。
- 不要把业务耦合直接写死在公共 Plugin 里。

涉及工程结构与插件封装时，读取 `references/plugin-architecture.md`。

### 4. 处理原生页面与视图嵌入

- 如果只是调用原生能力，不要急着上 `PlatformView`。
- 如果必须展示原生视图或原生页面，再选择 `UiKitView`、`FlutterPlatformView`、`UIHostingController` 等方案。
- SwiftUI 页面进入 Flutter 前，先在 iOS 侧整理成稳定的 `UIViewController` 或承载层。

涉及原生视图嵌入、SwiftUI 托管、生命周期时，读取 `references/ios-embedding-and-platform-view.md`。

### 5. 最后做联调与验证

- Dart 侧验证协议调用与错误映射。
- iOS 侧验证线程、注册、原生 API 调用与页面生命周期。
- 至少准备一条贯穿 Dart 与原生侧的集成验证路径。

涉及测试、日志、常见故障定位时，读取 `references/interop-testing-debugging.md`。

## 参考路由

| 需求 | 读取文件 |
| --- | --- |
| 通道选型、Pigeon、错误映射、消息协议 | `references/method-channel-pigeon.md` |
| 直接桥接与 Plugin 封装取舍、工程结构 | `references/plugin-architecture.md` |
| `PlatformView`、原生页面嵌入、SwiftUI 承载 | `references/ios-embedding-and-platform-view.md` |
| 双端测试、日志、注册与线程排错 | `references/interop-testing-debugging.md` |

## 常见错误

- 还没想清边界层职责，就先写 Dart 和 Swift 双边实现。
- 用原始 `MethodChannel` 承载一整个长期演进的复杂协议。
- 通道处理器里直接写业务逻辑、缓存逻辑和权限流程。
- 原生 UI 回调不切回主线程，导致显示或交互异常。
- 明明是持续事件，却仍然走同步请求-响应接口。
- 没有统一错误码和错误消息，导致 Dart 侧只能靠字符串猜问题。
- 原生能力已经明显可复用，却仍塞在业务 App 内部散落实现。
- 联调只看 Flutter 日志，不看 iOS 注册、线程和生命周期日志。

## 审查清单

- [ ] 任务确实属于 Flutter 与 iOS 的边界层。
- [ ] 已经明确是直接桥接、Plugin 封装还是原生视图嵌入。
- [ ] 桥接层保持薄，只负责协议转换和调用转发。
- [ ] 通道类型与调用频率、复杂度匹配。
- [ ] Dart 与 Swift 两侧的数据模型、错误语义一致。
- [ ] iOS UI 相关逻辑遵守主线程要求。
- [ ] 需要原生 UI 时，已经评估过 `PlatformView` 的代价。
- [ ] 工程结构支持后续复用，而不是把边界层写成一次性脚本。
- [ ] Dart 侧、iOS 侧和双端联调都有最小验证路径。
