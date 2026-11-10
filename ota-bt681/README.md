# Feasycom BT681 OTA SDK

BT681（Airoha）系列模组的 BLE Mesh 固件升级 SDK。

- 版本 / Version：**1.0.0**
- 适用模组 / Modules：BT681 系列
- 通道 / Transport：BLE
- 最低系统 / minSdk：API 24（Android 7.0）
- 依赖 AAR：`fsc-debug-utils-1.0.0.aar`

---

## 1. 特性

- **Airoha 升级协议**：分块写入 + 校验，支持断点/重试
- **连接状态回调**：`onConnectSuccess` / `onConnectFailure`
- **进度回调**：`onProgress(int)` 0~100
- **错误码**：通过 `ErrorCodeManager` 转成可读信息
- **内置 ProGuard 规则**

---

## 2. 集成方式

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/ota-bt681-1.0.0.aar')
}
```

---

## 3. 快速开始

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

// 退出页面
ota.disconnect()
```

---

## 4. API

### 4.1 FeasyOta68x

| 方法 | 说明 |
| --- | --- |
| `startOta(Context, BluetoothDevice, FeasyMeshUpdateCallbacks)` | 开始升级 |
| `setOtaFile(File)` | 设置固件文件 |
| `disconnect()` | 断开连接 |

### 4.2 FeasyMeshUpdateCallbacks

| 方法 | 说明 |
| --- | --- |
| `onStart()` | 升级开始 |
| `onConnectSuccess()` | 设备连接成功 |
| `onConnectFailure()` | 设备连接失败 |
| `onProgress(int)` | 进度 0~100 |
| `onSuccess()` | 升级成功 |
| `onFail(String)` | 升级失败及原因 |

---

## 5. 混淆说明

AAR 自带 `consumer-rules.pro`，开启 R8 时公开 API 与回调接口自动保留。

---

## 6. 更新记录

| 版本 | 说明 |
| --- | --- |
| 1.0.0 | 首个独立发布版本 |