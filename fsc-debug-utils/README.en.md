# Feasycom Debug Utils (FeasyLog remote diagnostics)

Writes app logs to both Logcat and an on-device txt file, so users/colleagues can export or share it (e.g. via WeCom) without a USB cable.

- Version: **1.0.0**
- minSdk: API 24 (Android 7.0)
- Required AARs: none (the host app must provide `appcompat`, `material`, `core-ktx`)

---

## 1. Features

- **Dual output**: Logcat + private log file, categorized by `LogSource`
- **One-tap export**: `LogShareHelper.showExportOptions(activity)` offers "share" or "save to Downloads"
- **Share-safe**: shares a snapshot file, so the main log can be cleared immediately after sharing
- **Device info included**: model, OS version, app version, process/thread, timestamps
- **Bundled ProGuard rules**: `FeasyLog`, `LogSource`, `LogShareHelper` are kept automatically

---

## 2. Integration

1. Copy `aar/fsc-debug-utils-1.0.0.aar` into the app's `libs/` directory.
2. Add the dependency:

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
}
```

3. Initialize in `Application.onCreate()`:

```kotlin
FeasyLog.init(this)
```

4. Sharing requires a FileProvider in the host app (authority is fixed to `${applicationId}.fileprovider`):

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

`res/xml/file_paths.xml` must contain at least the cache directory:

```xml
<paths>
    <cache-path name="feasylog" path="feasylog/" />
</paths>
```

---

## 3. API

### 3.1 FeasyLog

| Method | Description |
| --- | --- |
| `init(context)` | Initialize log directory and device info (call from `Application`) |
| `refreshPhoneInfo()` | Refresh model / OS / app version info |
| `clear()` | Clear the main log file |
| `v/d/i/w/e(source, tag, msg)` | Write a log entry at the given level |
| `v/d/i/w/e(source, msg)` | Same as above, using `source.name` as tag |
| `exportForShare(): File?` | Create a share snapshot; returns null when there is no log |

`LogSource` is an enum used to categorize logs (`NET_LOGIN`, `BLE_MANAGER`, `SPP_MANAGER`, `OTA_XMODEM`, `OTA_FMODEM`, `COMMON`, ...). Its `title` is written into the log.

```kotlin
FeasyLog.e(LogSource.OTA_XMODEM, "XmodemBleOta", "upgrade failed: ${e.message}")
```

### 3.2 LogShareHelper

| Method | Description |
| --- | --- |
| `showExportOptions(activity)` | Show the "share / save to Downloads" dialog |

```kotlin
LogShareHelper.showExportOptions(this)
```

---

## 4. Obfuscation

The AAR ships `consumer-rules.pro`, keeping the following when the app enables R8:

- public members of `com.feasycom.debug_utils.feasylog.FeasyLog`
- public members of `com.feasycom.debug_utils.feasylog.LogShareHelper`
- all of `com.feasycom.debug_utils.feasylog.LogSource` (enum names are logged)
- generics, inner classes, annotations and line numbers

---

## 5. Changelog

| Version | Notes |
| --- | --- |
| 1.0.0 | First standalone release: dual-channel logging, share/save, device info |