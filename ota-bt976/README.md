# Feasycom BT976 BLE OTA SDK

BT976 系列模组的蓝牙（BLE）固件升级 SDK。通过 BLE GATT 将固件文件分包写入设备，设备端完成 CRC / SHA-256 校验后自动跳转到新固件。

- 版本 / Version：**1.0.0**
- 适用模组 / Modules：BT976 系列（BLE）
- 最低系统 / minSdk：API 24（Android 7.0）
- 编译环境 / Build：JDK 11、Android Gradle Plugin 7.4+ / 8.x

---

## 1. 特性

- **标准 BLE GATT 升级**：无需一次性大通知，逐包写入并按扇区校验，失败可重试
- **SHA-256 摘要下发**：升级前先下发固件 SHA-256（分两包），设备端据此校验固件完整性
- **扇区级校验**：每扇区（4 KB）写入完成后读取设备校验结果，异常可提前中断
- **写失败自动重发**：`writeCharacteristic` 返回失败时回滚包序号并重发，最多 5 次
- **连接优先级自动提升**：MTU 协商成功后请求 `CONNECTION_PRIORITY_HIGH`，提升吞吐
- **节奏参数保守化**：内置适配多数手机的包间隔/校验等待参数（见第 7 节）
- **内置 ProGuard 规则**：使用方开启 R8 时 SDK 公开 API 自动保留

---

## 2. 集成方式

### 2.1 使用预编译 AAR

1. 将 `aar/ota-bt976-1.0.0.aar` 复制到 App 模块的 `libs/` 目录
2. 在 `app/build.gradle` 中添加：

```gradle
android {
    // ...
}

dependencies {
    implementation files('libs/ota-bt976-1.0.0.aar')
}
```

> AAR 内已包含 `proguard.txt`（consumer 规则），App 开启 `minifyEnabled true` 时无需额外配置。

### 2.2 所需权限（AndroidManifest.xml）

```xml
<!-- Android 12 (API 31) 及以上 -->
<uses-permission
    android:name="android.permission.BLUETOOTH_SCAN"
    android:usesPermissionFlags="neverForLocation" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />

<!-- Android 11 (API 30) 及以下 -->
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" android:maxSdkVersion="30" />
```

> `BLUETOOTH_SCAN` / `BLUETOOTH_CONNECT` 需要运行时动态申请。扫描在 Android 11 及以下需要定位权限。

---

## 3. 快速开始

```java
import com.feasycom.ota_976_lib.sdk.Ota976Sdk;
import com.feasycom.ota_976_lib.sdk.OtaBt976Api;
import com.feasycom.ota_976_lib.sdk.OtaCallBack;

OtaBt976Api ota = Ota976Sdk.getInstance();

// 1) 设置固件文件（完整升级镜像，.bin）
ota.setOtaFile(new File("/sdcard/Download/BT976_Update.bin"));

// 2) 开始升级：newSdkVer 为 true 表示使用新版本 SDK 升级流程
ota.StartOta(bluetoothDevice, /* newSdkVer = */ true, new OtaCallBack() {

    @Override
    public void onSuccess() {
        // 升级成功
    }

    @Override
    public void onFailure(String err) {
        // 升级失败，err 为原因描述
    }

    @Override
    public void onProgress(int progress) {
        // 进度：0 ~ 100
    }

    @Override
    public void onTips(String tips) {
        // 过程提示（连接中、校验中、写入扇区等）
    }
});
```

Kotlin：

```kotlin
val ota = Ota976Sdk.getInstance()
ota.setOtaFile(File("/sdcard/Download/BT976_Update.bin"))
ota.StartOta(device, true, object : OtaCallBack {
    override fun onSuccess() { /* 成功 */ }
    override fun onFailure(err: String?) { /* 失败 */ }
    override fun onProgress(progress: Int) { /* 进度 */ }
    override fun onTips(tips: String?) { /* 提示 */ }
})
```

### 3.1 自行管理连接（可选）

SDK 默认在 `StartOta()` 内部完成连接、服务发现、MTU 协商。若你的 App 已有统一的 BLE 连接管理，也可以自行连接并使用底层写入接口：

```java
ota.init(context);
ota.ConnectBleDev(context, device, new FscBleConnectCallback() {
    @Override public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) { }
    @Override public void onPrepared(int mtu) { }          // 服务发现 + MTU 完成
    @Override public void onpacketSend(byte[] value) { }    // 每包发送回调
    @Override public void onpacketReceived(byte[] value) { }// 每包接收回调
    @Override public void onErro(String erro) { }
});

// 手动写入（writeNoResponse 必须为 true，见第 8 节）
ota.sendBleValue(data, true);
```

---

## 4. API 参考

### 4.1 `Ota976Sdk`

| 方法 | 说明 |
| --- | --- |
| `static OtaBt976Api getInstance()` | 获取 SDK 单例（双重检查锁，线程安全） |

### 4.2 `OtaBt976Api`

| 方法 | 说明 |
| --- | --- |
| `void init(Context context)` | 初始化，保存上下文（建议传 `getApplicationContext()`） |
| `void setOtaFile(File otaFile)` | 设置待升级的固件文件（完整 `.bin` 镜像） |
| `void StartOta(BluetoothDevice device, boolean newSdkVer, OtaCallBack callback)` | 开始升级；`newSdkVer=true` 使用新流程（目标地址 `0x18061000`，升级完成后下发完成命令） |
| `void ConnectBleDev(Context, BluetoothDevice, FscBleConnectCallback)` | 自行发起连接（可选，用于自管连接场景） |
| `void sendBleValue(byte[] value, boolean writeNoResponse)` | 写入特征值（OTA 数据必须 `writeNoResponse = true`） |
| `void sendBleValue(byte[], boolean, OtaBt976imp.BleWriteCallback)` | 带写入结果回调的写入 |
| `BluetoothDevice getConnectedBleDevice()` | 获取当前已连接的 BLE 设备 |
| `void Disconnect()` | 断开连接并释放资源 |

### 4.3 `OtaCallBack`

| 回调 | 说明 |
| --- | --- |
| `void onSuccess()` | 升级成功（设备已返回校验成功） |
| `void onFailure(String str)` | 升级失败，`str` 为失败原因 |
| `void onProgress(int progress)` | 升级进度 `0~100` |
| `void onTips(String str)` | 过程提示文本 |

### 4.4 `FscBleConnectCallback`

| 回调 | 说明 |
| --- | --- |
| `void onConnectionStateChange(BluetoothGatt gatt, int status, int newState)` | 连接状态变化 |
| `void onPrepared(int mtu)` | 服务发现与 MTU 协商完成（MTU 通常为 517） |
| `void onpacketSend(byte[] value)` | 每包发送后回调（调试用） |
| `void onpacketReceived(byte[] value)` | 每包接收后回调（调试用） |
| `void onErro(String erro)` | 错误回调 |

---

## 5. 升级流程（协议摘要）

服务 UUID `00002600-0000-1000-8000-00805f9b34fb`

| 特征值 | UUID | 属性 |
| --- | --- | --- |
| 控制 | `00007000-...` | Write / Notify |
| 数据 | `00007001-...` | Write Without Response |

| 步骤 | 指令 | 说明 |
| --- | --- | --- |
| 1 | `0x01 0x00` + SHA-256[0..15] | 下发固件摘要前半段 |
| 2 | `0x01 0x01` + SHA-256[16..31] | 下发固件摘要后半段 |
| 3 | `0x02` + 地址(4, LE) + 大小(4, LE) + 包长(4, LE) | 开始升级；包长 = MTU − 4（默认 256） |
| 4 | `0x04` + 扇区号(2, LE) | 请求写第 N 扇区（每扇区 4 KB = 16 包） |
| 5 | `SEGMENTID` + 256 B 负载 | 数据包，共 257 B |
| 6 | `0x05` | 请求校验固件完整性 |
| 7 | 设备返回 `0x06 0x00 …` / `0x06 0x80 …` | `0x00` 成功，`0x80` 失败 |
| 8 | `0x07 0x07` + 大小(3, LE) + 源地址 + 目标地址 + 启动地址 | 新版本流程的完成命令 |

关键约定：

- **每扇区固定 16 包**，每包 1 B 段号（0~15）+ 256 B 数据
- 数据必须使用 **Write Without Response**（默认写会被固件拒绝，返回 `GATT_WRITE_NOT_PERMITTED`）
- 目标地址：`newSdkVer=true` → `0x18061000`；旧流程为 `0x18069000`

---

## 6. 升级耗时参考

以 105 KB 固件（420 包 / 27 扇区）为例：

| 场景 | 总耗时 |
| --- | --- |
| 默认保守参数 | 约 25 ~ 28 秒 |
| 网络/手机较慢 | 30 ~ 40 秒 |
| 历史版本（250 ms/包） | 约 125 秒 |

---

## 7. 节奏参数与调优

SDK 内部使用以下保守参数（源码 `OtaBt976imp.java` 中的 `DELAY_*` 常量），在多数手机上稳定通过：

| 常量 | 默认值 | 含义 |
| --- | --- | --- |
| `DELAY_SEGMENT_MS` | 15 ms | 两包之间的间隔 |
| `DELAY_SECTOR_CMD_MS` | 60 ms | 扇区命令应答后开始发数据 |
| `DELAY_BEFORE_SECTOR_CHECK_MS` | 200 ms | 扇区发完、读取校验前的等待（设备算 CRC 时间） |
| `DELAY_NEXT_SECTOR_MS` | 80 ms | 校验通过后发下一个扇区命令 |
| `DELAY_SECTOR_END_MS` | 100 ms | 扇区边界等待 |
| `DELAY_BEFORE_FINAL_CHECK_MS` | 400 ms | 整片完整性校验前等待 |

调优建议：

- 某类手机升级失败（校验不通过）→ 先增大 `DELAY_SEGMENT_MS`（15 → 25 → 30）
- 设备算 CRC 慢 → 增大 `DELAY_BEFORE_SECTOR_CHECK_MS`（200 → 300）
- 追求速度且设备稳定 → 可将 `DELAY_SEGMENT_MS` 降到 0（极限约 18 秒），但兼容性下降

> 以上参数为编译期常量，修改后需重新编译 SDK。

---

## 8. 常见问题（FAQ）

**Q1：升级最后返回 "Firmware integrity verification failed"，设备回 `06 80`？**

按以下顺序排查：

1. **固件文件是否与设备匹配**（型号、分区、Bootloader 版本）。实测中某些固件包本身就通不过设备端校验，换官方匹配的固件即可。
2. **包序号是否错乱**：数据包段号必须严格 `0~15` 循环，每扇区恰好 16 包。SDK 已修复"写回调早于计数器自增导致多发一包"的竞态问题。
3. **包间隔过小**：设备端来不及处理导致丢包，调大 `DELAY_SEGMENT_MS`。

**Q2：第一包就写入失败，返回 `status=3`（GATT_WRITE_NOT_PERMITTED）？**

固件只接受 **Write Without Response**。请确认使用 `sendBleValue(data, true)`，或由 SDK 内部写入（默认即为无响应写）。

**Q3：连接后立即断开，`onConnectionStateChange` 返回 `status=133`？**

通常为升级流程参数不匹配（如旧流程地址 `0x18069000` 用于新固件）。请使用 `StartOta(device, true, callback)`。

**Q4：部分手机（如部分 OPPO / 小米机型）升级不稳定？**

1. 确认已请求高优先级连接（SDK 在 MTU 协商后自动请求 `CONNECTION_PRIORITY_HIGH`）
2. 提示用户不要同时进行其它 BLE 操作
3. 适当增大包间隔

**Q5：可以多设备同时升级吗？**

SDK 为单例，同一时刻只支持一个升级任务。多设备请串行执行。

---

## 9. 版本记录

见 [CHANGELOG.md](CHANGELOG.md)。

---

## 10. 技术支持

- 官网：https://www.feasycom.com
- 问题反馈：请在本仓库提交 Issue

© Feasycom. All rights reserved.
