# Feasycom BT2044GI OTA SDK

BT2044GI 系列模组的 BLE 固件升级 SDK，支持 zip 固件包自动解压。

- 版本 / Version：**1.0.0**
- 适用模组 / Modules：BT2044GI 系列
- 通道 / Transport：BLE
- 最低系统 / minSdk：API 24（Android 7.0）
- 依赖 AAR：`fsc-debug-utils-1.0.0.aar`

---

## 1. 特性

- **zip 固件支持**：自动解压并定位固件文件
- **CRC32 校验**：升级前后校验固件完整性
- **进度与提示回调**：`onProgress(int)`、`onTips(String)`
- **内置 ProGuard 规则**

---

## 2. 集成方式

```gradle
dependencies {
    implementation files('libs/fsc-debug-utils-1.0.0.aar')
    implementation files('libs/ota-bt2044gi-1.0.0.aar')
}
```

---

## 3. 快速开始

```kotlin
val ota = Ota2044Sdk.getInstance()
ota.init(context)
ota.setOtaFile(File(firmwarePath))   // 支持 .bin / .zip
ota.StartOta(device, object : Ota2044CallBack {
    override fun onSuccess() {}
    override fun onFailure(str: String) {}
    override fun onTips(str: String) {}
    override fun onProgress(progress: Int) {}
})

// 退出页面
ota.Disconnect()
```

---

## 4. API

### 4.1 OtaBt2044Api

| 方法 | 说明 |
| --- | --- |
| `init(Context)` | 初始化（Application 或页面创建时） |
| `setOtaFile(File)` | 设置固件文件（bin/zip） |
| `StartOta(BluetoothDevice, Ota2044CallBack)` | 开始升级 |
| `Disconnect()` | 断开连接 |

### 4.2 Ota2044CallBack

| 方法 | 说明 |
| --- | --- |
| `onSuccess()` | 升级成功 |
| `onFailure(String)` | 升级失败及原因 |
| `onTips(String)` | 过程提示信息 |
| `onProgress(int)` | 进度 0~100 |

---

## 5. 混淆说明

AAR 自带 `consumer-rules.pro`，开启 R8 时公开 API 与回调接口自动保留。

---

## 6. 更新记录

| 版本 | 说明 |
| --- | --- |
| 1.0.0 | 首个独立发布版本 |