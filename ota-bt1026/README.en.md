# Feasycom BT1026 OTA SDK

Firmware upgrade SDK for BT1026 modules using the UART DFU (VMU) protocol over SPP.

- Version: **1.0.0**
- Modules: BT1026 series
- Transport: SPP (Bluetooth Classic serial port)
- minSdk: API 24 (Android 7.0)
- Required AARs: `fsc-core-0.0.0.6.aar`, `fsc-debug-utils-1.0.0.aar`

---

## 1. Features

- **VMU protocol**: SYNC → START → DATA → VALIDATION → COMMIT
- **Reconnect**: automatically reconnects and resumes after the device reboots
- **Watchdog**: fails after 20 s without response
- **Stage/progress callbacks**: `Stage` constants can drive the UI directly
- **Error codes**: `OtaError` distinguishes file/connection/busy failures
- **Bundled ProGuard rules**

---

## 2. Integration

1. Copy `aar/ota-bt1026-1.0.0.aar`, `fsc-core-0.0.0.6.aar` and `fsc-debug-utils-1.0.0.aar` into `libs/`.
2. Add the dependencies:

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/fsc-core-0.0.0.6.aar')
    implementation files('libs/ota-bt1026-1.0.0.aar')
}
```

> This SDK uses `SppManage` from `fsc-core` to establish the SPP connection, so both `fsc-core` and `fsc-debug-utils` are required.

---

## 3. Quick start (Kotlin)

```kotlin
val ota = UartDfuSppOta_1026.getInstance()
ota.setOtaCallback(object : UartDfuOtaCallback_1026 {
    override fun onDfuReady() {}
    override fun onSuccess() {}
    override fun onFailure(erro: String) {}
    override fun onProgress(p: Float) {}
    override fun onDeviceRestart() {}
})
ota.setOtaFile(File(firmwarePath))
ota.startOta(bluetoothDevice)          // keep factory settings
// ota.startOta(bluetoothDevice, true) // restore factory settings after upgrade
```

---

## 4. API

### 4.1 UartDfuSppOta_1026

| Method | Description |
| --- | --- |
| `getInstance()` | Singleton |
| `setOtaCallback(callback)` | Set the upgrade callback |
| `setOtaFile(File)` | Set the firmware file; returns whether it is valid |
| `setOtaFile(byte[])` | Set firmware bytes directly |
| `startOta(BluetoothDevice)` | Start upgrade (keep factory settings) |
| `startOta(BluetoothDevice, boolean)` | Start upgrade, optionally restore factory settings |
| `isUpgrading()` | Whether an upgrade is in progress |

### 4.2 Callback (UartDfuOtaCallback_1026)

| Method | Description |
| --- | --- |
| `onDfuReady()` | Device entered DFU mode |
| `onProgress(float)` | Progress 0~100 |
| `onSuccess()` | Upgrade succeeded |
| `onFailure(String)` | Upgrade failed, with reason |
| `onDeviceRestart()` | Device is rebooting (brief disconnect) |

### 4.3 Constants

- `Stage`: `IDLE/SYNC/START/START_DATA/DATA_TRANSFER/VALIDATION/TRANSFER_COMPLETE/IN_PROGRESS/COMMIT/COMPLETED/ABORTED`
- `OtaError`: `FILE_INVALID/FILE_READ_FAILED/NOT_CONNECTED/ALREADY_UPGRADING/DEVICE_DISCONNECTED`

---

## 5. Obfuscation

The AAR ships `consumer-rules.pro`; public classes and callback interfaces are kept when R8 is enabled.

---

## 6. Changelog

| Version | Notes |
| --- | --- |
| 1.0.0 | First standalone release |