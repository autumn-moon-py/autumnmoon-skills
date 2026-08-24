# Flutter插件协议治理（拆分）

## 1. 错误协议
- 统一 `code/message/recoverable/action`。
- 禁止直接 `notImplemented` 无语义返回。

## 2. 生命周期
- `onAttachedToEngine` 与 `onDetachedFromEngine` 对称。
- detach 时解绑 handler 并清理引用。

## 3. 版本兼容
- 协议对象带 `schemaVersion`。
- 新增字段只追加；破坏性变更走双写双读窗口。

## 4. 契约测试
- 方法名、参数、错误结构纳入自动化校验。