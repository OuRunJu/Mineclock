---
name: add-config-option
description: 按项目 Interface + DefaultConfiguration 模式新增可配置项。修改 Configuration、DefaultConfiguration 或 Widget 行为参数时使用。
paths:
  - "app/src/main/java/com/example/mineclock/Configuration.java"
  - "app/src/main/java/com/example/mineclock/DefaultConfiguration.java"
  - "app/src/main/java/com/example/mineclock/*Configuration*.java"
---

# 新增配置项

按以下步骤为 Mineclock 添加新的可配置常量。

## 1. 扩展 Configuration 接口

在 `Configuration.java` 添加新方法：

```java
int newOption();
```

- 方法名用名词/动词，不加 `get` 前缀
- 返回类型选最简类型（`boolean`, `int`, `String`）

## 2. 在 DefaultConfiguration 实现

在 `DefaultConfiguration.java`：

1. 添加 `private static final` 常量（`UPPER_SNAKE_CASE`）
2. 实现接口方法，标记 `@Override final`

```java
private static final int NEW_OPTION = 42;

@Override
public final int newOption() {
    return NEW_OPTION;
}
```

## 3. 在消费方注入使用

- 通过构造函数接收 `Configuration`，不要直接引用 `DefaultConfiguration`
- 需要调试日志时，用 `Logger`（受 `debugEnabled()` 控制）

## 4. 关联常量联动

若新配置影响帧率或 Alarm，检查并更新：

| 配置 | 关联 |
|------|------|
| `imagesPerDay` | `imageOffset`、`UPDATE_INTERVAL_MS`、drawable level 数量 |
| `updateIntervalMs` | `DefaultUpdateAlarmManager.scheduleNext()` |
| `debugEnabled` | `DefaultLogger`、`ViewModelAdapter.debugInfo()` |

## 5. 补充测试

- 若新配置影响 `ViewModelAdapter` 输出，在 `ViewModelAdapterUnitTest` 添加用例
- 测试中使用 `DefaultConfiguration` 或测试专用 Configuration 实现

## 6. 验证

```bash
chmod +x ./gradlew
./gradlew test
```

## 禁止

- 不要在多处硬编码同一魔法数字
- 不要跳过接口直接在业务类中 `new DefaultConfiguration()`（`ClockAppWidgetProvider` 入口除外）
