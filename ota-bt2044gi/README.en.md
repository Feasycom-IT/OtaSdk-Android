# Feasycom BT2044GI OTA SDK

BLE firmware upgrade SDK for BT2044GI modules, with zip package extraction.

- Version: **1.0.0**
- Modules: BT2044GI series
- Transport: BLE
- minSdk: API 24 (Android 7.0)
- Required AARs: `fsc-debug-utils-1.0.0.aar`

---

## 1. Features

- **zip firmware support**: extracts the package and locates the firmware
- **CRC32 verification**: validates firmware integrity
- **Progress and tips**: `onProgress(int)`, `onTips(String)`
- **Bundled ProGuard rules**

---

## 2. Integration

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/ota-bt2044gi-1.0.0.aar')
}
```

---

## 3. Quick start

```kotlin
val ota = Ota2044Sdk.getInstance()
ota.init(context)
ota.setOtaFile(File(firmwarePath))   // .bin or .zip
ota.StartOta(device, object : Ota2044CallBack {
    override fun onSuccess() {}
    override fun onFailure(str: String) {}
    override fun onTips(str: String) {}
    override fun onProgress(progress: Int) {}
})

// leaving the page
ota.Disconnect()
```

---

## 4. API

### 4.1 OtaBt2044Api

| Method | Description |
| --- | --- |
| `init(Context)` | Initialize (Application or page creation) |
| `setOtaFile(File)` | Set firmware file (bin/zip) |
| `StartOta(BluetoothDevice, Ota2044CallBack)` | Start upgrade |
| `Disconnect()` | Disconnect |

### 4.2 Ota2044CallBack

| Method | Description |
| --- | --- |
| `onSuccess()` | Upgrade succeeded |
| `onFailure(String)` | Upgrade failed, with reason |
| `onTips(String)` | Progress tips |
| `onProgress(int)` | Progress 0~100 |

---

## 5. Obfuscation

The AAR ships `consumer-rules.pro`; public API and callbacks are kept when R8 is enabled.

---

## 6. Changelog

| Version | Notes |
| --- | --- |
| 1.0.0 | First standalone release |