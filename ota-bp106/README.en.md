# BP106 SUOTA SDK

SUOTA (Software Update Over The Air) BLE firmware upgrade SDK for BP106 modules.

- Version: **1.0.0**
- Modules: BP106 series
- Transport: BLE
- minSdk: API 24 (Android 7.0)
- Required AARs: none (standalone AAR)

---

## 1. Features

- **Standard SUOTA flow**: read device info → select image → chunked upload → end signal → reboot
- **Multiple firmware headers**: 58x / 68x / 69x `HeaderInfo`
- **Speed statistics**: `updateSpeedStatistics` / `updateCurrentSpeed` for UI
- **Block/chunk progress**: `onChunkSend`, `onBlockSent`, `onUploadProgress`
- **Storage helpers**: `SuotaManager.checkStorage()`, `getFilePathUri()` for SAF
- **Bundled ProGuard rules**

---

## 2. Integration

```gradle
dependencies {
    implementation files('libs/ota-bp106-1.0.0.aar')
}
```

---

## 3. Quick start

```kotlin
val manager = SuotaManager(context, device, object : SuotaManagerCallback() {
    override fun onDeviceReady() {}
    override fun onUploadProgress(percent: Float) {}
    override fun onSuccess(totalElapsedSeconds: Double, imageUploadElapsedSeconds: Double) {}
    override fun onFailure(errorCode: Int) {}
})

manager.setSuotaFile(suotaFile)   // pick firmware via SuotaFile.listFiles(...)
manager.startUpdate()

// leaving the page
manager.disconnect()
```

---

## 4. API

### 4.1 SuotaManager

| Method | Description |
| --- | --- |
| `SuotaManager(Context, BluetoothDevice, SuotaManagerCallback)` | Create and bind device |
| `setSuotaFile(SuotaFile)` | Set firmware to upload |
| `startUpdate()` | Start upgrade |
| `disconnect()` | Disconnect |
| `getState()` | Current state (`ManagerState`) |
| `checkStorage(Context)` / `getFilePathUri()` / `setFilePathUri()` | SAF directory helpers |

### 4.2 Callback (ISuotaManagerCallback)

`onConnectionStateChange`, `onServicesDiscovered`, `onDeviceInfoReadCompleted`, `onDeviceReady`, `onChunkSend`, `onBlockSent`, `onUploadProgress`, `updateSpeedStatistics`, `updateCurrentSpeed`, `onSuccess`, `onFailure`, `onRebootSent`, etc.

### 4.3 SuotaFile

| Method | Description |
| --- | --- |
| `listFiles(List<String>, String, boolean, boolean)` | Collect firmware from paths |
| `listFiles(Context, DocumentFile, String, boolean, boolean)` | Collect from a SAF directory |
| `createDirectory()` | Create the default directory |

---

## 5. Obfuscation

The AAR ships `consumer-rules.pro`; public API and callbacks are kept when R8 is enabled.

---

## 6. Changelog

| Version | Notes |
| --- | --- |
| 1.0.0 | First standalone release |