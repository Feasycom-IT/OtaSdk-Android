# Feasycom BT1035 OTA SDK

BT1035 系列模组的 Fmodem 协议固件升级 SDK，同时支持 SPP 与 BLE 两种通道。

- 版本 / Version：**1.0.0**
- 适用模组 / Modules：BT1035 系列
- 通道 / Transport：SPP + BLE
- 最低系统 / minSdk：API 24（Android 7.0）
- 依赖 AAR：`fsc-core-0.0.0.6.aar`、`fsc-debug-utils-1.0.0.aar`

---

## 1. 特性

- **双通道**：SPP（`FmodemSppOta`）与 BLE（`FmodemBleOta`）任选
- **DFU 进入**：自动发送 `EnterDFU` 并等待 `FS0000000000` 握手
- **断线重连**：最多 3 次自动重连后续传
- **进度回调**：`onProgress(float)` 0~100
- **可选升级后恢复出厂**
- **内置 ProGuard 规则**

---

## 2. 集成方式

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/fsc-core-0.0.0.6.aar')
    implementation files('libs/ota-bt1035-1.0.0.aar')
}
```

---

## 3. 快速开始

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

| 方法 | 说明 |
| --- | --- |
| `getInstance()` | 单例 |
| `setOtaCallback(callback)` | 设置回调 |
| `setOtaFile(File)` / `setOtaFile(byte[])` | 设置固件 |
| `startOta(device, callback)` | 开始升级（不恢复出厂） |
| `startOta(device, callback, boolean)` | 可指定升级后恢复出厂 |
| `onConnected()` / `onpacketReceived(byte[])` / `onDisConnected()` / `onErro(String)` | 通道事件回调（由宿主转发） |

### 4.2 FmodemBleOta

| 方法 | 说明 |
| --- | --- |
| `getInstance()` | 单例 |
| `setOtaFile(File)` | 设置固件 |
| `startOta(device, callback)` | 开始升级 |
| `startOta(device, callback, boolean)` | 可指定升级后恢复出厂 |
| `disconnect()` | 断开连接 |

### 4.3 回调

`FmodemSppOtaCallBack`：`onDfuReady()`、`onSuccess()`、`onFailure(String)`、`onProgress(float)`、`onDeviceRestart()`  
`FmodemBleOtaCallBack`：`onSuccess()`、`onFailure(String)`、`onProgress(float)`、`onDeviceRestart()`

---

## 5. 混淆说明

AAR 自带 `consumer-rules.pro`，开启 R8 时公开 API 与回调接口自动保留。

---

## 6. 更新记录

| 版本 | 说明 |
| --- | --- |
| 1.0.0 | 首个独立发布版本（SPP + BLE） |