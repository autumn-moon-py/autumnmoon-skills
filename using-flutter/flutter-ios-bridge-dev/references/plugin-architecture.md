# 直接桥接与 Plugin 架构

## 先决定承载方式

### 直接桥接

适合：

- 只在当前业务 App 内使用。
- 能力少、生命周期短。
- 复用价值不高。

优点：

- 进入成本低。
- 改动集中在业务工程。

缺点：

- 复用差。
- 后续容易散乱。

### 独立 Plugin

适合：

- 有复用价值。
- 多个页面或多个项目会用到。
- 希望把 Dart API 与平台实现隔离。

优点：

- 边界清晰。
- 易于测试和发布。

缺点：

- 初始结构更重。

## 推荐结构

```text
my_plugin/
  lib/
    my_plugin.dart
    src/
      my_plugin_platform_interface.dart
      my_plugin_method_channel.dart
  ios/
    Classes/
      MyPlugin.swift
  example/
    lib/
      main.dart
```

如果后续要做多平台实现，再考虑平台接口和 federated 结构，不要一开始就为了“专业”过度拆分。

## 边界规则

- Dart 层对外暴露稳定 API。
- 平台层负责 iOS 实现细节。
- 业务流程不要写进公共 Plugin。

## 从直接桥接升级到 Plugin 的时机

出现以下情况时应考虑升级：

- 不止一个功能在用同一套原生能力。
- 业务工程里到处散落通道名和参数拼接。
- 测试和联调越来越依赖人工点点点。

## 示例应用

Plugin 必须有 `example`：

- 验证注册。
- 验证参数。
- 验证回调。
- 作为联调与回归入口。

## 快速检查

- [ ] 当前能力是否值得独立成 Plugin。
- [ ] 公共 API 是否和业务逻辑解耦。
- [ ] 是否有 `example` 作为联调入口。
