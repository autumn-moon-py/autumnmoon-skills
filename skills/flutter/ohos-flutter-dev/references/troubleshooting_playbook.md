# 常见问题排查手册（实践版）

## 1. `fvm flutter run`/`fvm flutter build hap` 失败

1. 先看环境：
- `fvm flutter doctor -v`
- DevEco SDK、`ohpm`、`hvigor`、`node` 是否在 PATH 中。

2. 再看工程：
- `oh-package.json5` 的 `overrides` 是否包含 `flutter_embedding*.har`、`flutter_module.har`。
- `entry/oh-package.json5` 与根 `oh-package.json5` 依赖是否一致。

3. 再看依赖安装层级：
- 优先在工程根执行 `ohpm install`，不要只在 `entry` 层执行。

## 2. 集成后改了 Flutter 代码，宿主不生效

原因：宿主引用的是旧 HAR。

处理：
1. 进入 Flutter 模块目录重新构建：
```powershell
fvm flutter build har --debug
```
2. 更新宿主 `har/` 目录或依赖路径。
3. 再次 `ohpm install` 与编译运行。

## 3. 多引擎页面黑屏或状态错乱

重点检查：
1. 页面 `aboutToAppear` 是否执行 `attach()`。
2. 页面 `aboutToDisappear` 是否执行 `detach()`。
3. `onPageShow/onPageHide` 是否同步引擎生命周期（`appIsResumed/appIsPaused` 等）。
4. `viewId` 是否与当前 `FlutterView` 对齐。

## 4. Channel 调用无响应

1. Dart 侧 channel 名称与 ArkTS 侧完全一致。
2. ArkTS 侧插件是否已注册到当前引擎。
3. 方法名是否匹配（大小写与参数类型都要对齐）。
4. 对应页面是否已经进入且引擎已 attach。

## 5. PlatformView 显示异常

1. ArkTS 侧 `registerViewFactory(viewType, factory)` 是否执行。
2. Flutter 侧 `OhosView(viewType: ...)` 是否一致。
3. 双向 MethodChannel 是否按 `viewId` 维度构造唯一名称。
4. 宿主页面层级是否把 `FlutterPage` 和原生视图叠放冲突。

## 6. `--local-engine` 相关策略

1. 常规业务开发：可不传 `--local-engine`，使用默认流程。
2. 引擎定制调试：再使用 `--local-engine/--local-engine-host`。
3. 团队内统一：不要在日常命令强制写死本地引擎路径。
