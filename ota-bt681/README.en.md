# Feasycom BT681 OTA SDK

BLE Mesh firmware upgrade SDK for BT681 (Airoha) modules.

- Version: **1.0.0**
- Modules: BT681 series
- Transport: BLE
- minSdk: API 24 (Android 7.0)
- Required AARs: `fsc-debug-utils-1.0.0.aar`

---

## 1. Features

- **Airoha upgrade protocol**: chunked writes with verification and retries
- **Connection callbacks**: `onConnectSuccess` / `onConnectFailure`
- **Progress callback**: `onProgress(int)` 0~100
- **Error codes**: readable messages via `ErrorCodeManager`
- **Bundled ProGuard rules**

---

## 2. Integration

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/ota-bt681-1.0.0.aar')
}
```

---

## 3. Quick start

```kotlin
val ota = FeasyOta68xSdk.getInstance()
ota.setOtaFile(File(firmwarePath))
ota.startOta(context, device, object : FeasyMeshUpdateCallbacks {
    override fun onStart() {}
    override fun onConnectSuccess() {}
    override fun onConnectFailure() {}
    override fun onProgress(progress: Int) {}
    override fun onSuccess() {}
    override fun onFail(erro: String) {}
})

// leaving the page
ota.disconnect()
```

---

## 4. API

### 4.1 FeasyOta68x

| Method | Description |
| --- | --- |
| `startOta(Context, BluetoothDevice, FeasyMeshUpdateCallbacks)` | Start upgrade |
| `setOtaFile(File)` | Set firmware file |
| `disconnect()` | Disconnect |

### 4.2 FeasyMeshUpdateCallbacks

| Method | Description |
| --- | --- |
| `onStart()` | Upgrade started |
| `onConnectSuccess()` | Device connected |
| `onConnectFailure()` | Device connection failed |
| `onProgress(int)` | Progress 0~100 |
| `onSuccess()` | Upgrade succeeded |
| `onFail(String)` | Upgrade failed, with reason |

---

## 5. Obfuscation

The AAR ships `consumer-rules.pro`; public API and callbacks are kept when R8 is enabled.

---

## 6. Changelog

| Version | Notes |
| --- | --- |
| 1.0.0 | First standalone release |