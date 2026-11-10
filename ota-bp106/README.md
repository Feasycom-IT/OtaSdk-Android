# Feasycom BP106 SUOTA SDK

BP106 系列模组的 SUOTA（Software Update Over The Air）BLE 固件升级 SDK。

- 版本 / Version：**1.0.0**
- 适用模组 / Modules：BP106 系列
- 通道 / Transport：BLE
- 最低系统 / minSdk：API 24（Android 7.0）
- 依赖 AAR：无（独立 AAR）

---

## 1. 特性

- **标准 SUOTA 流程**：读取设备信息 → 选择镜像 → 分块上传 → 结束信号 → 重启
- **多格式固件头**：支持 58x / 68x / 69x `HeaderInfo`
- **速度统计**：`updateSpeedStatistics` / `updateCurrentSpeed` 可用于界面展示
- **块/分块进度**：`onChunkSend`、`onBlockSent`、`onUploadProgress`
- **存储检查**：`SuotaManager.checkStorage()`、`getFilePathUri()` 适配 SAF
- **内置 ProGuard 规则**

---

## 2. 集成方式

```gradle
dependencies {
    implementation files('libs/ota-bp106-1.0.0.aar')
}
```

---

## 3. 快速开始

```kotlin
val manager = SuotaManager(context, device, object : SuotaManagerCallback() {
    override fun onDeviceReady() {}
    override fun onUploadProgress(percent: Float) {}
    override fun onSuccess(totalElapsedSeconds: Double, imageUploadElapsedSeconds: Double) {}
    override fun onFailure(errorCode: Int) {}
})

manager.setSuotaFile(suotaFile)   // SuotaFile.listFiles(...) 选择固件
manager.startUpdate()

// 退出页面
manager.disconnect()
```

---

## 4. API

### 4.1 SuotaManager

| 方法 | 说明 |
| --- | --- |
| `SuotaManager(Context, BluetoothDevice, SuotaManagerCallback)` | 构造并绑定设备 |
| `setSuotaFile(SuotaFile)` | 设置待升级固件 |
| `startUpdate()` | 开始升级 |
| `disconnect()` | 断开连接 |
| `getState()` | 当前状态（`ManagerState`） |
| `checkStorage(Context)` / `getFilePathUri()` / `setFilePathUri()` | SAF 目录检查与设置 |

### 4.2 回调（ISuotaManagerCallback）

`onConnectionStateChange`、`onServicesDiscovered`、`onDeviceInfoReadCompleted`、`onDeviceReady`、`onChunkSend`、`onBlockSent`、`onUploadProgress`、`updateSpeedStatistics`、`updateCurrentSpeed`、`onSuccess`、`onFailure`、`onRebootSent` 等。

### 4.3 SuotaFile

| 方法 | 说明 |
| --- | --- |
| `listFiles(List<String>, String, boolean, boolean)` | 从路径列表筛选固件 |
| `listFiles(Context, DocumentFile, String, boolean, boolean)` | 从 SAF 目录筛选 |
| `createDirectory()` | 创建默认目录 |

---

## 5. 混淆说明

AAR 自带 `consumer-rules.pro`，开启 R8 时公开 API 与回调类自动保留。

---

## 6. 更新记录

| 版本 | 说明 |
| --- | --- |
| 1.0.0 | 首个独立发布版本 |