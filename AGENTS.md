# Mineclock — Agent 指令

Minecraft 风格 Android 桌面时钟 Widget。支持 Android 8.0+（API 26）。

## 快速参考

| 项目 | 值 |
|------|-----|
| 包名 | `com.example.mineclock` |
| 入口 | `ClockAppWidgetProvider` |
| 业务核心 | `ViewModelAdapter`（时间 → 帧索引） |
| 配置 | `Configuration` / `DefaultConfiguration` |
| 测试 | `./gradlew test` |

## 架构

Interface + `DefaultXxx` 实现 + 构造函数手动注入。`ViewModel` 是不可变数据类，不是 AndroidX ViewModel。

## Cursor 配置

- **Rules**（`.cursor/rules/`）：编码规范与领域知识，按文件类型自动附加
- **Skills**（`.cursor/skills/`）：多步骤工作流，用 `/skill-name` 调用
  - `add-config-option` — 新增配置项
  - `add-widget-feature` — 扩展 Widget 功能
  - `run-android-tests` — 构建与测试

## 开发原则

1. 最小改动，保持现有模式
2. 业务逻辑放可单测的纯 Java 类
3. 不擅自引入 Kotlin、Compose、DI 框架
4. 配置常量走 `Configuration` 接口，避免魔法数字散落
