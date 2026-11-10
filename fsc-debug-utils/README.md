# Debug Utils（远程排查日志）

把 App 运行日志同时写到 Logcat 和手机上的一个 txt 文件，用户在页面点一下即可导出或分享（如微信、企业微信），不用连电脑抓 Logcat。

- 版本 / Version：**1.0.0**
- 最低系统 / minSdk：API 24（Android 7.0）
- 依赖 AAR：无（需宿主 App 提供 `appcompat`、`material`、`core-ktx`）

---

## 1. 特性

- **双通道输出**：`Logcat` + 应用私有目录日志文件，按 `LogSource` 分类
- **一键导出**：`LogShareHelper.showExportOptions(activity)` 弹出「分享 / 保存到下载目录」
- **分享安全**：分享的是快照文件，主日志可在分享后立即清空，不会给对方发空文件
- **自动附带设备信息**：机型、系统版本、App 版本、进程/线程、时间戳
- **内置 ProGuard 规则**：`FeasyLog`、`LogSource`、`LogShareHelper` 自动保留

---

## 2. 集成方式

1. 将 `aar/fsc-debug-utils-1.0.0.aar` 复制到 App 的 `libs/` 目录；
2. 在 `app/build.gradle` 中引入：

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
}
```

3. 在 `Application.onCreate()` 初始化：

```kotlin
FeasyLog.init(this)
```

4. 分享功能需要宿主 App 声明 FileProvider（authority 固定为 `${applicationId}.fileprovider`）：

```xml
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

`res/xml/file_paths.xml` 至少包含缓存目录：

```xml
<paths>
    <cache-path name="feasylog" path="feasylog/" />
</paths>
```

---

## 3. API

### 3.1 FeasyLog

| 方法 | 说明 |
| --- | --- |
| `init(context)` | 初始化日志目录与设备信息，建议在 `Application` 调用 |
| `refreshPhoneInfo()` | 刷新机型/系统/App 版本等信息 |
| `clear()` | 清空主日志文件 |
| `v/d/i/w/e(source, tag, msg)` | 写入指定级别的日志 |
| `v/d/i/w/e(source, msg)` | 省略 tag，使用 `source.name` 作为 tag |
| `exportForShare(): File?` | 生成分享快照，返回文件；无日志时返回 null |

`LogSource` 为枚举，按业务分类（如 `NET_LOGIN`、`BLE_MANAGER`、`SPP_MANAGER`、`OTA_XMODEM`、`OTA_FMODEM`、`COMMON`），枚举的 `title` 会写入日志。

示例：

```kotlin
FeasyLog.e(LogSource.OTA_XMODEM, "XmodemBleOta", "升级失败：${e.message}")
```

### 3.2 LogShareHelper

| 方法 | 说明 |
| --- | --- |
| `showExportOptions(activity)` | 弹出「分享 / 保存到下载目录」选择框 |

```kotlin
LogShareHelper.showExportOptions(this)
```

---

## 4. 混淆说明

AAR 自带 `consumer-rules.pro`，使用方开启 R8 时自动保留：

- `com.feasycom.debug_utils.feasylog.FeasyLog` 公开成员
- `com.feasycom.debug_utils.feasylog.LogShareHelper` 公开成员
- `com.feasycom.debug_utils.feasylog.LogSource` 全部（枚举名会写入日志）
- 泛型、内部类、注解、行号信息

---

## 5. 更新记录

| 版本 | 说明 |
| --- | --- |
| 1.0.0 | 首个独立发布版本：日志双通道、分享/保存、设备信息自动附带 |