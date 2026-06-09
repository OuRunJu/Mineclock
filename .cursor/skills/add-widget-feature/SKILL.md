---
name: add-widget-feature
description: 扩展 Mineclock App Widget 功能（新 UI 元素、新广播处理、刷新逻辑）。修改 ClockAppWidgetProvider、ViewModelAdapter 或 Widget 资源时使用。
paths:
  - "app/src/main/java/com/example/mineclock/ClockAppWidgetProvider.java"
  - "app/src/main/java/com/example/mineclock/ViewModelAdapter.java"
  - "app/src/main/java/com/example/mineclock/ViewModel.java"
  - "app/src/main/res/layout/**"
  - "app/src/main/res/xml/clock_widget_provider_info.xml"
  - "app/src/main/AndroidManifest.xml"
---

# 扩展 Widget 功能

## 架构决策（先回答）

| 变更类型 | 放置位置 |
|----------|----------|
| 时间/帧计算 | `ViewModelAdapter` |
| 展示数据 | `ViewModel`（不可变字段 + 访问方法） |
| Android 生命周期/广播 | `ClockAppWidgetProvider` |
| RemoteViews 操作 | `RemoteViewsExtensions` 静态方法 |
| 定时刷新 | `UpdateAlarmManager` / `DefaultUpdateAlarmManager` |

**原则**：业务逻辑优先放可单测的纯 Java 类，Provider 只做编排。

## 标准实现流程

### Step 1 — 扩展数据模型

1. 在 `ViewModel` 添加 `final` 字段和访问方法
2. 在 `ViewModelAdapter.viewModel()` 计算新字段
3. 在 `ViewModelAdapterUnitTest` 添加测试

### Step 2 — 更新 UI

1. 修改 `res/layout/clock_widget.xml`
2. 新字符串写入 `res/values/strings.xml`
3. 为交互/装饰元素添加 `contentDescription`

### Step 3 — 绑定 RemoteViews

在 `ClockAppWidgetProvider.updateRemoteViews()` 中：

```java
views.setTextViewText(R.id.new_view_id, this.viewModel.newField());
```

复杂操作提取到 `RemoteViewsExtensions`。

### Step 4 — 广播/Alarm（如需要）

1. 新 Action 常量放 `UpdateAlarmManager` 或专用接口
2. `ClockAppWidgetProvider.onReceive()` 添加分支
3. `AndroidManifest.xml` 注册对应 `intent-filter`
4. Alarm 变更走 cancel → update → schedule 模式

### Step 5 — 配置（如需要）

按 `@add-config-option` skill 扩展 `Configuration`。

## 帧动画注意

- 帧索引由 `ViewModelAdapter.image()` 计算（`lerp` + `imageOffset` + `% imagesPerDay`）
- 修改帧数必须同步 drawable level-list 和 `IMAGES_PER_DAY`
- 正午 = 帧 0，午夜 = 帧 32（默认偏移下）

## 验证清单

- [ ] 单元测试通过：`./gradlew test`
- [ ] Provider 生命周期方法保持 cancel → update → schedule 顺序
- [ ] Manifest intent-filter 与 `onReceive` 分支一致
- [ ] 无硬编码用户可见字符串
- [ ] release 构建不受 ProGuard 影响（无反射除 RemoteViews `setInt`）

## 参考

- 现有更新流程：`ClockAppWidgetProvider.updateAppWidget()`
- 帧测试用例：`ViewModelAdapterUnitTest`（正午/午夜边界）
