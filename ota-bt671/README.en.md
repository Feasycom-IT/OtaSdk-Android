# BT671 / BT3721Vx OTA SDK

BLE firmware upgrade SDK for BT671 and BT3721Vx modules (67x protocol).

- Version: **1.0.0**
- Modules: BT671 series, BT3721Vx series
- Transport: BLE
- minSdk: API 24 (Android 7.0)
- Required AARs: `fsc-core-0.0.0.6.aar`, `fsc-debug-utils-1.0.0.aar`

---

## 1. Features

- **Two entry points**: `FeasyOta67xSdk` (BT671) and `FeasyOta3721VxSdk` (3721Vx)
- **Fast mode**: `fastMode=true` improves throughput in stable RF conditions
- **Progress callback**: `onProgress(double)` 0~100
- **Disconnect control**: disconnect when leaving the page or cancelling
- **Bundled ProGuard rules**

---

## 2. Integration

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/fsc-core-0.0.0.6.aar')
    implementation files('libs/ota-bt671-1.0.0.aar')
}
```

---

## 3. Quick start

```kotlin
val ota = FeasyOta67xSdk.getInstance()      // use FeasyOta3721VxSdk for 3721Vx
ota.setOtaFile(File(firmwarePath))
ota.startOta(context, true, device, object : OtaCallback {
    override fun onStart() {}
    override fun onSuccess() {}
    override fun onFail(erro: String) {}
    override fun onProgress(p: Double) {}
})

// leaving the page
ota.disconnect()
```

---

## 4. API

### 4.1 FeasyOta67x

| Method | Description |
| --- | --- |
| `startOta(Context, boolean fastMode, BluetoothDevice, OtaCallback)` | Start upgrade |
| `setOtaFile(File)` | Set firmware file |
| `disconnect()` | Disconnect |

### 4.2 OtaCallback

| Method | Description |
| --- | --- |
| `onStart()` | Upgrade started |
| `onProgress(double)` | Progress 0~100 |
| `onSuccess()` | Upgrade succeeded |
| `onFail(String)` | Upgrade failed, with reason |

### 4.3 Entry points

| Class | Applies to |
| --- | --- |
| `FeasyOta67xSdk.getInstance()` | BT671 series |
| `FeasyOta3721VxSdk.getInstance()` | BT3721Vx series |

---

## 5. Obfuscation

The AAR ships `consumer-rules.pro`; public API is kept when R8 is enabled.

---

## 6. Changelog

| Version | Notes |
| --- | --- |
| 1.0.0 | First standalone release |