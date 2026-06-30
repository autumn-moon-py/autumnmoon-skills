# OHOS 样例能力矩阵

说明：该矩阵按 `flutter_samples/ohos` 与关键 Add-to-App OHOS 样例梳理，用于“按需求反查样例”。

## A. 工程骨架与混合开发

| 项目 | 主要能力 | 建议用途 |
| --- | --- | --- |
| `ohos/flutter_page_sample1` | `FlutterEntry + FlutterPage` 生命周期管理 | 单页混合开发基线 |
| `ohos/flutter_page_sample2` | 混合路由、参数传递、插件注册时机 | 多页混合导航 |
| `add_to_app/books/ohos_books` | 宿主通过 HAR 集成 Flutter 模块 | 标准 Add-to-App 参考 |
| `add_to_app/fullscreen/ohos_fullscreen` | 全屏嵌入 Flutter 页 | 全屏容器场景 |
| `add_to_app/prebuilt_module/ohos_using_prebuilt_module` | 预编译模块接入 | 模块化交付 |
| `add_to_app/plugin/ohos_using_plugin` | 宿主 + 插件 + 模块协同 | 插件型 Add-to-App |
| `add_to_app/multiple_flutters/multiple_flutters_ohos` | `FlutterEngineGroup` 多引擎 | 多实例并发展示 |
| `ohos/multiple_flutters_predraw` | 多引擎预渲染与切换 | 性能敏感多实例 |

## B. 通信与平台能力

| 项目 | 主要能力 | 建议用途 |
| --- | --- | --- |
| `ohos/channel_demo` | `MethodChannel/EventChannel/BasicMessageChannel` | 通信范式模板 |
| `ohos/platform_demo` | `PlatformViewFactory + OhosView + MethodChannel` | 原生视图嵌入 Flutter |
| `ohos/platformchannel_demo` | 多平台能力接口演示 | 通道能力清单验证 |
| `ohos/load_native_resource_demo` | 资源加载与通道交互 | 原生资源桥接 |
| `ohos/js_dart_demo` | JS 与 Dart 协同 | WebView/JS 互通 |
| `ohos/test_uni_links` | Deep Link 能力 | 拉起与路由链路 |

## C. UI、交互与主题

| 项目 | 主要能力 | 建议用途 |
| --- | --- | --- |
| `ohos/animation_demo` | 动画基础 | 环境验证首跑 |
| `ohos/gesture_intercept_demo` | 手势冲突处理 | 复杂触控页 |
| `ohos/scrollview_demo` | 滚动容器实践 | 滚动性能排查 |
| `ohos/flutter_ohos_theme_fontsizescale` | 主题/字体跟随系统 | 系统风格适配 |
| `ohos/ohos_themeAdaptation` | 主题适配 | 深浅色适配 |
| `ohos/video_full_screen` | 视频全屏交互 | 媒体播放页 |
| `ohos/testpicture` | 图片展示 | 媒体浏览页 |
| `ohos/path_drawing_test` | 路径绘制 | 自定义绘制 |

## D. 网络、存储与数据

| 项目 | 主要能力 | 建议用途 |
| --- | --- | --- |
| `ohos/http_test` | HTTP 请求 | 网络基线 |
| `ohos/dio_test` | Dio 生态与权限协同 | 网络工程化 |
| `ohos/http_parser_test` | HTTP 解析 | 协议解析 |
| `ohos/automated_testing_demo` | `sqflite` 与测试 | 数据层测试 |
| `ohos/floor_test` | `floor` ORM | ORM 场景验证 |
| `ohos/ohos_sqlite3_demo` | `sqlite3` 使用 | 本地数据库 |
| `ohos/sqflite_test` | `sqflite` 示例 | DB CRUD |
| `ohos/sqflite_helper` | 插件形态 DB 封装 | 数据访问抽象 |
| `ohos/pictures_provider_demo` | `path_provider` 场景 | 文件路径与媒体访问 |

## E. 三方库适配样例

| 项目 | 主要能力 | 建议用途 |
| --- | --- | --- |
| `ohos/flutter_svg_test` | SVG 渲染链路 | 图形库适配 |
| `ohos/cached_network_image_sample` | 图片缓存库 | 缓存策略验证 |
| `ohos/flutter_webview_demo` | WebView | 混合内容展示 |
| `ohos/flutter-pag` | 外接纹理 + PAG | 高性能动效 |
| `ohos/flutter_huawei_login` | 华为登录 + WebView/Toast | 登录链路示例 |
| `ohos/flutter_it_image` | 自定义图像处理链路 | 图像插件验证 |
| `ohos/flutter_it_preload` | 预加载能力 | 首帧优化 |
| `ohos/ohos_flutter_photoviewpicker` | 组件/插件组合 | 复杂插件场景 |

## F. 语言与库验证（工具型样例）

| 项目 | 主要能力 | 建议用途 |
| --- | --- | --- |
| `ohos/async_test`、`ohos/asynchronous` | 异步模型 | 并发行为验证 |
| `ohos/event_bus_test` | 事件总线 | 状态广播 |
| `ohos/provider_partrefresh` | Provider 局部刷新 | 状态管理策略 |
| `ohos/rxdart_test` | 响应式流 | 事件流编排 |
| `ohos/clock_test` | 时间库 | 计时逻辑 |
| `ohos/logging_test` | 日志库 | 诊断链路 |
| `ohos/tuple_test`、`ohos/uuid_test` | 基础工具库 | 数据结构与标识 |
| `ohos/vector_math_test` | 数学库 | 图形/动画计算 |
| `ohos/petitparser_test`、`ohos/string_scanner_test`、`ohos/xml_test`、`ohos/path_parsing_test` | 解析库验证 | 解析器适配 |

## G. 其他业务演示

| 项目 | 主要能力 | 建议用途 |
| --- | --- | --- |
| `ohos/testcamera` | 相机能力 | 媒体采集 |
| `ohos/localtion_demo` | 定位能力 | LBS 场景 |
| `ohos/testchat` | 聊天 UI 场景 | 即时通信页原型 |
| `ohos/component_demo` | 组件演示 | UI 组件验证 |
| `ohos/multi_products` | 多产品配置 | 多渠道/多配置打包 |
| `ohos/platform_test` | 平台识别与调用 | 平台差异逻辑 |

## H. 非应用样例（辅助）

| 项目 | 说明 |
| --- | --- |
| `ohos/performance` | 性能测试相关目录，偏基础设施 |
| `ohos/node_test_server` | 服务端测试目录，不是 Flutter 客户端样例 |
| `ohos/docs` | OpenHarmony Flutter 文档总入口 |
