# Feasycom BT1026 OTA SDK

BT1026 系列模组的 UART DFU（VMU 协议）固件升级 SDK，通过 SPP 通道把固件分包写入设备。

- 版本 / Version：**1.0.0**
- 适用模组 / Modules：BT1026 系列
- 通道 / Transport：SPP（经典蓝牙串口）
- 最低系统 / minSdk：API 24（Android 7.0）
- 依赖 AAR：`fsc-core-0.0.0.6.aar`、`fsc-debug-utils-1.0.0.aar`

---

## 1. 特性

- **VMU 协议升级**：SYNC → START → DATA → VALIDATION → COMMIT 全流程
- **断线重连**：升级中设备重启后自动重连并续传
- **看门狗**：20 秒无响应自动判定失败，避免界面卡死
- **进度/阶段回调**：`Stage` 常量可直接用于界面展示
- **错误码**：`OtaError` 常量区分文件、连接、升级中等失败原因
- **内置 ProGuard 规则**：公开 API 自动保留

---

## 2. 集成方式

1. 将 `aar/ota-bt1026-1.0.0.aar`、`fsc-core-0.0.0.6.aar`、`fsc-debug-utils-1.0.0.aar` 复制到 `libs/`；
2. 引入依赖：

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/fsc-core-0.0.0.6.aar')
    implementation files('libs/ota-bt1026-1.0.0.aar')
}
```

> 本 SDK 内部使用 `fsc-core` 的 `SppManage` 建立 SPP 连接，因此必须同时引入 `fsc-core` 与 `fsc-debug-utils`。

---

## 3. 快速开始（Kotlin）

```kotlin
val ota = UartDfuSppOta_1026.getInstance()
ota.setOtaCallback(object : UartDfuOtaCallback_1026 {
    override fun onDfuReady() { /* 设备已进入 DFU */ }
    override fun onSuccess() { /* 升级成功 */ }
    override fun onFailure(erro: String) { /* 升级失败 */ }
    override fun onProgress(p: Float) { /* 0f..100f */ }
    override fun onDeviceRestart() { /* 设备重启中 */ }
})
ota.setOtaFile(File(firmwarePath))
ota.startOta(bluetoothDevice)          // 升级后不恢复出厂
// ota.startOta(bluetoothDevice, true) // 升级后恢复出厂设置
```

---

## 4. API

### 4.1 UartDfuSppOta_1026

| 方法 | 说明 |
| --- | --- |
| `getInstance()` | 获取单例 |
| `setOtaCallback(callback)` | 设置升级回调 |
| `setOtaFile(File)` | 设置固件文件，返回是否有效 |
| `setOtaFile(byte[])` | 直接设置固件字节 |
| `startOta(BluetoothDevice)` | 开始升级（不恢复出厂） |
| `startOta(BluetoothDevice, boolean)` | 开始升级，可指定升级后是否恢复出厂 |
| `isUpgrading()` | 是否正在升级 |

### 4.2 回调 UartDfuOtaCallback_1026

| 方法 | 说明 |
| --- | --- |
| `onDfuReady()` | 设备进入 DFU 模式 |
| `onProgress(float)` | 升级进度 0~100 |
| `onSuccess()` | 升级成功 |
| `onFailure(String)` | 升级失败及原因 |
| `onDeviceRestart()` | 设备重启（可能短暂断连） |

### 4.3 常量

- `Stage`：`IDLE/SYNC/START/START_DATA/DATA_TRANSFER/VALIDATION/TRANSFER_COMPLETE/IN_PROGRESS/COMMIT/COMPLETED/ABORTED`
- `OtaError`：`FILE_INVALID/FILE_READ_FAILED/NOT_CONNECTED/ALREADY_UPGRADING/DEVICE_DISCONNECTED`

---

## 5. 混淆说明

AAR 自带 `consumer-rules.pro`，开启 R8 时自动保留公开类与回调接口。

---

## 6. 更新记录

| 版本 | 说明 |
| --- | --- |
| 1.0.0 | 首个独立发布版本 |