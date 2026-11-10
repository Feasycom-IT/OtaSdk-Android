# BT1035 OTA SDK

Firmware upgrade SDK for BT1035 modules using the Fmodem protocol, over SPP or BLE.

- Version: **1.0.0**
- Modules: BT1035 series
- Transport: SPP + BLE
- minSdk: API 24 (Android 7.0)
- Required AARs: `fsc-core-0.0.0.6.aar`, `fsc-debug-utils-1.0.0.aar`

---

## 1. Features

- **Dual transport**: SPP (`FmodemSppOta`) or BLE (`FmodemBleOta`)
- **DFU entry**: sends `EnterDFU` and waits for the `FS0000000000` handshake
- **Reconnect**: up to 3 automatic reconnects with resume
- **Progress**: `onProgress(float)` 0~100
- **Optional factory reset after upgrade**
- **Bundled ProGuard rules**

---

## 2. Integration

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/fsc-core-0.0.0.6.aar')
    implementation files('libs/ota-bt1035-1.0.0.aar')
}
```

---

## 3. Quick start

### 3.1 SPP

```kotlin
val ota = FmodemSppOta.getInstance()
ota.setOtaFile(File(firmwarePath))
ota.startOta(device, object : FmodemSppOtaCallBack {
    override fun onDfuReady() {}
    override fun onSuccess() {}
    override fun onFailure(erro: String) {}
    override fun onProgress(p: Float) {}
    override fun onDeviceRestart() {}
})
```

### 3.2 BLE

```kotlin
val ota = FmodemBleOta.getInstance()
ota.setOtaFile(File(firmwarePath))
ota.startOta(device, object : FmodemBleOtaCallBack {
    override fun onSuccess() {}
    override fun onFailure(erro: String) {}
    override fun onProgress(progress: Float) {}
    override fun onDeviceRestart() {}
})
```

---

## 4. API

### 4.1 FmodemSppOta

| Method | Description |
| --- | --- |
| `getInstance()` | Singleton |
| `setOtaCallback(callback)` | Set callback |
| `setOtaFile(File)` / `setOtaFile(byte[])` | Set firmware |
| `startOta(device, callback)` | Start upgrade (keep factory settings) |
| `startOta(device, callback, boolean)` | Optionally restore factory settings |
| `onConnected()` / `onpacketReceived(byte[])` / `onDisConnected()` / `onErro(String)` | Transport events forwarded by the host |

### 4.2 FmodemBleOta

| Method | Description |
| --- | --- |
| `getInstance()` | Singleton |
| `setOtaFile(File)` | Set firmware |
| `startOta(device, callback)` | Start upgrade |
| `startOta(device, callback, boolean)` | Optionally restore factory settings |
| `disconnect()` | Disconnect |

### 4.3 Callbacks

`FmodemSppOtaCallBack`: `onDfuReady()`, `onSuccess()`, `onFailure(String)`, `onProgress(float)`, `onDeviceRestart()`  
`FmodemBleOtaCallBack`: `onSuccess()`, `onFailure(String)`, `onProgress(float)`, `onDeviceRestart()`

---

## 5. Obfuscation

The AAR ships `consumer-rules.pro`; public API and callbacks are kept when R8 is enabled.

---

## 6. Changelog

| Version | Notes |
| --- | --- |
| 1.0.0 | First standalone release (SPP + BLE) |