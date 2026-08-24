---
name: ios-swift-dev
description: Use when 进行 iOS 原生 Swift / SwiftUI 开发，或处理 Swift 并发、导航、性能、URLSession 网络层、安全、可访问性、测试与面向可维护性的原生重构；不适用于 Flutter 页面开发或 Flutter 与 iOS 的桥接层实现。
---

# iOS Swift 原生开发

## 概述

这个技能用于处理 iOS 原生开发问题，重点覆盖 SwiftUI、Swift 并发、性能、网络、安全、可访问性和测试。

它适合原生功能开发，也适合对已有 Swift / SwiftUI 代码进行审查和收敛。只要任务涉及 Flutter 页面、Dart 状态管理、Plugin 注册或 `MethodChannel`，请切换到 `$flutter-app-dev` 或 `$flutter-ios-bridge-dev`。

如需具体模式，请按需读取以下引用资料：

- `references/swiftui-architecture.md`
- `references/navigation-composition.md`
- `references/performance.md`
- `references/concurrency.md`
- `references/networking.md`
- `references/security-accessibility.md`
- `references/testing.md`

## 适用场景

- 新增或重构 SwiftUI 页面与原生功能模块。
- 梳理 `@State`、`@Observable`、`@Bindable`、`@Environment` 的归属。
- 处理 `NavigationStack`、Sheet、Tab、深链等导航问题。
- 排查卡顿、重绘过多、列表性能差、主线程阻塞。
- 设计或审查 `URLSession` 网络层。
- 接入 `Keychain`、`CryptoKit`、生物识别、可访问性支持。
- 使用 Swift Testing 或 XCTest 补齐测试。

## 不适用场景

- 纯 Flutter 页面开发。
- Flutter 与 iOS 的边界层、Plugin、通道协议。
- 需要把 iOS 原生能力暴露给 Flutter 的任务。

## 核心规则

- 新增原生 UI 时优先使用 SwiftUI；只有明确需要时才引入 UIKit。
- 先判断现有工程架构，再决定是否沿用 ViewModel。简单场景不要硬塞厚重架构。
- UI 相关状态必须有清晰的主线程归属。不要用“能跑”替代并发安全。
- 导航、状态、网络、性能、安全、可访问性都属于交付质量，不是上线后再补的附属品。
- 性能问题先代码审查，再上 Instruments，不凭感觉乱改。
- 网络层统一通过 `URLSession` + `async/await` 管理，不新增无必要第三方依赖。
- 密钥、Token、敏感信息只能进 `Keychain` 或更安全的存储路径。
- 所有用户可见界面都必须具备基础可访问性能力。

## 工作流

### 1. 先看上下文与约束

- 确认部署目标和现有项目模式。
- 确认是否启用了 `Swift 6` 严格并发相关设置。
- 确认任务属于纯原生功能，而不是 Flutter 边界层。

涉及状态归属、SwiftUI 组合方式时，读取 `references/swiftui-architecture.md`。

### 2. 设计状态、导航与服务边界

- 先定状态归属，再定视图组合。
- 先定导航路径、Sheet 和 Tab 关系，再实现交互。
- 服务层与视图层分离，网络、缓存、加密不要塞进 View。

涉及 `NavigationStack`、Sheet、深链、页面组合时，读取 `references/navigation-composition.md`。

### 3. 实现前先排除高风险点

- 并发：先明确主线程与后台线程边界。
- 性能：避免在 `body` 里做重计算、解码、排序、过滤。
- 网络：统一响应校验、错误映射和解码策略。

涉及并发时读取 `references/concurrency.md`；涉及性能时读取 `references/performance.md`；涉及网络时读取 `references/networking.md`。

### 4. 把安全与可访问性作为默认要求

- 敏感数据默认进 `Keychain`，不进 `UserDefaults`。
- 所有交互控件都有可访问标签、可读顺序和合适焦点。
- 动态字体、减少动态效果、高对比度等系统偏好需要被尊重。

涉及安全、隐私与可访问性时，读取 `references/security-accessibility.md`。

### 5. 最后补齐验证

- 新增单元测试优先使用 Swift Testing。
- UI 流程、快照、性能或系统 UI 行为保留 XCTest / XCUITest。
- 对并发、性能、主线程更新、取消行为做针对性验证。

涉及测试时，读取 `references/testing.md`。

## 参考路由

| 需求 | 读取文件 |
| --- | --- |
| SwiftUI 状态归属、组合方式、环境注入 | `references/swiftui-architecture.md` |
| 导航、Sheet、Tab、深链 | `references/navigation-composition.md` |
| 卡顿、重绘、身份稳定性、Instruments | `references/performance.md` |
| `@MainActor`、`Sendable`、Task、取消 | `references/concurrency.md` |
| `URLSession`、解码、错误映射、重试 | `references/networking.md` |
| `Keychain`、`CryptoKit`、可访问性 | `references/security-accessibility.md` |
| Swift Testing、XCTest、异步测试 | `references/testing.md` |

## 常见错误

- 不看现有架构，直接引入一套新的 ViewModel 或 Router 模式。
- 把所有状态都绑到一个大对象上，导致整个页面频繁重绘。
- 在 `body` 中做排序、过滤、格式化、图片解码。
- 碰到并发警告就无脑加 `@MainActor`。
- `URLSession` 收到 4xx / 5xx 后直接按成功处理。
- 把 Token、密码、密钥存进 `UserDefaults`。
- 图标按钮没有可访问标签，Sheet 关闭后焦点回不去。
- 新写单元测试还沿用冗长的 XCTest 风格，而不使用 Swift Testing。

## 审查清单

- [ ] 任务属于纯原生 iOS 开发，没有越界到 Flutter 边界层。
- [ ] SwiftUI 状态归属、生命周期与依赖注入清楚。
- [ ] 导航结构可解释，Sheet / Tab / 深链关系明确。
- [ ] 并发边界清晰，没有用错误注解掩盖线程问题。
- [ ] 性能热点没有留在 `body` 或主线程。
- [ ] 网络层统一校验响应、解码和错误映射。
- [ ] 敏感数据使用 `Keychain` 或更安全方案存储。
- [ ] 页面满足基础可访问性要求。
- [ ] 单元、异步、UI 或性能验证覆盖了关键风险。
