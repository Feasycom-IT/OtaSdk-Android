# Feasycom BT671 / BT3721Vx OTA SDK

BT671、BT3721Vx 系列模组的 BLE 固件升级 SDK（67x 协议）。

- 版本 / Version：**1.0.0**
- 适用模组 / Modules：BT671 系列、BT3721Vx 系列
- 通道 / Transport：BLE
- 最低系统 / minSdk：API 24（Android 7.0）
- 依赖 AAR：`fsc-core-0.0.0.6.aar`、`fsc-debug-utils-1.0.0.aar`

---

## 1. 特性

- **两套入口**：`FeasyOta67xSdk`（BT671 系列）与 `FeasyOta3721VxSdk`（3721Vx 系列）
- **快速模式**：`fastMode=true` 提升吞吐，适合信号稳定的场景
- **进度回调**：`onProgress(double)` 0~100
- **断开控制**：升级结束或取消时主动断开
- **内置 ProGuard 规则**

---

## 2. 集成方式

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/fsc-core-0.0.0.6.aar')
    implementation files('libs/ota-bt671-1.0.0.aar')
}
```

---

## 3. 快速开始

```kotlin
val ota = FeasyOta67xSdk.getInstance()      // 3721Vx 用 FeasyOta3721VxSdk.getInstance()
ota.setOtaFile(File(firmwarePath))
ota.startOta(context, true, device, object : OtaCallback {
    override fun onStart() {}
    override fun onSuccess() {}
    override fun onFail(erro: String) {}
    override fun onProgress(p: Double) {}
})

// 取消/退出页面
ota.disconnect()
```

---

## 4. API

### 4.1 FeasyOta67x

| 方法 | 说明 |
| --- | --- |
| `startOta(Context, boolean fastMode, BluetoothDevice, OtaCallback)` | 开始升级 |
| `setOtaFile(File)` | 设置固件文件 |
| `disconnect()` | 断开连接 |

### 4.2 OtaCallback

| 方法 | 说明 |
| --- | --- |
| `onStart()` | 开始升级 |
| `onProgress(double)` | 进度 0~100 |
| `onSuccess()` | 升级成功 |
| `onFail(String)` | 升级失败及原因 |

### 4.3 入口

| 类 | 适用 |
| --- | --- |
| `FeasyOta67xSdk.getInstance()` | BT671 系列 |
| `FeasyOta3721VxSdk.getInstance()` | BT3721Vx 系列 |

---

## 5. 混淆说明

AAR 自带 `consumer-rules.pro`，开启 R8 时公开 API 自动保留。

---

## 6. 更新记录

| 版本 | 说明 |
| --- | --- |
| 1.0.0 | 首个独立发布版本 |