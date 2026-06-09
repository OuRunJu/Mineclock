---
name: run-android-tests
description: 构建与测试 Mineclock Android 项目。运行测试、验证构建或排查 Gradle 问题时使用。
---

# 构建与测试 Mineclock

## 前置条件

- JDK 8+（项目使用 `java.time`，需 Java 8）
- Android SDK（`compileSdk 29`，`build-tools 29.0.3`）
- 环境变量 `ANDROID_HOME` 或 `ANDROID_SDK_ROOT` 指向 SDK 目录

## 常用命令

```bash
# 确保 gradlew 可执行
chmod +x ./gradlew

# 单元测试（无需设备，推荐首选）
./gradlew test

# 查看单元测试报告
# app/build/reports/tests/test/index.html

# 仪器测试（需连接设备或模拟器）
./gradlew connectedAndroidTest

# Debug APK
./gradlew assembleDebug

# Release APK（启用 R8 压缩）
./gradlew assembleRelease

# 清理
./gradlew clean
```

## 测试结构

| 类型 | 路径 | 运行器 |
|------|------|--------|
| 单元测试 | `app/src/test/` | JUnit 4 |
| 仪器测试 | `app/src/androidTest/` | AndroidJUnit4 |

## 编写新测试

### 单元测试模板

```java
public final class MyUnitTest {
    private FakeWallClock wallClock;

    @Before
    public final void setUp() {
        this.wallClock = new FakeWallClock();
        // inject Configuration + dependencies
    }

    @Test
    public final void feature_isCorrect_whenCondition() {
        this.wallClock.setNow(LocalTime.NOON);
        // assert
    }
}
```

- Fake 实现放在测试文件内或 `app/src/test/` 独立文件
- 命名：`feature_isCorrect_whenCondition`

### 仪器测试

仅用于必须有 `Context` 的场景（如验证包名）。业务逻辑优先单元测试。

## 排查构建失败

1. **Permission denied on gradlew** → `chmod +x ./gradlew`
2. **SDK not found** → 设置 `ANDROID_HOME`，或创建 `local.properties`：
   ```
   sdk.dir=/path/to/Android/Sdk
   ```
3. **jcenter 不可用** → 项目使用 `jcenter()`，必要时迁移到 `mavenCentral()`
4. **依赖冲突** → `./gradlew app:dependencies`

## 当前测试覆盖

`ViewModelAdapterUnitTest` 覆盖帧索引边界：

- 正午 → 帧 0
- 正午前 1 秒 → 帧 63
- 午夜 → 帧 32
- 午夜前 1 秒 → 帧 31

新增帧/时间逻辑应在此补充对应用例。
