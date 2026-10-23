# Feasycom Fsc Core SDK

基础蓝牙能力 SDK。封装 BLE / SPP 的扫描、连接、特征值读写，并提供基于 Xmodem 协议的固件升级能力（BLE 通道与 SPP 通道），以及固件解析、CRC 校验、加密校验等工具类。

- 版本 / Version：**0.0.0.2**
- 最低系统 / minSdk：API 29（Android 10）
- 编译环境 / Build：JDK 11、Android Gradle Plugin 8.x
- 内含原生库 / Native libs：`libFeasyBlueLibrary.so`（arm64-v8a、armeabi-v7a）

---

## 1. 特性

- **BLE 能力**：扫描、连接、服务发现与 MTU 协商、特征值读写、连接状态回调
- **SPP 能力**：扫描、连接、数据收发、设备信息读取
- **BLE OTA**：`XmodemBleOta` 通过 BLE 通道使用 Xmodem 协议升级固件
- **SPP OTA**：`XmodemSppOta` 通过串口（SPP）通道升级固件
- **固件解析**：`TeaCode` / `FileUtil` 解析 DFU 文件信息（Bootloader、固件版本、模块型号）
- **CRC / 加密校验**：`CRC16`、`FACP`、`EncryptInfo`、`EncryptedUtil`、`FscEncryptApi`
- **内置 consumer ProGuard 规则**：使用方开启 R8 时自动保留对外 API 与 JNI 符号

---

## 2. 集成方式

### 2.1 引入 AAR

1. 将 `aar/fsc-core-0.0.0.2.aar` 复制到 App 模块的 `libs/` 目录
2. 在 `app/build.gradle` 中添加：

```gradle
dependencies {
    implementation files('libs/fsc-core-0.0.0.2.aar')
}
```

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

> SPP 扫描需要定位权限；`BLUETOOTH_SCAN` / `BLUETOOTH_CONNECT` 需要运行时动态申请。

### 2.3 关于原生库（重要）

AAR 内打包了 `libFeasyBlueLibrary.so`。但以下类中调用了 `System.loadLibrary("JniLibrary")`：

- `com.feasycom.ota.Teacode.bean.EncryptInfo`
- `com.feasycom.ota.Teacode.controler.FscEncryptApiImp`
- `com.feasycom.ota.Teacode.utils.FACP`

AAR 内**没有** `libJniLibrary.so`。如果使用到上述加密 / 校验相关能力，请确认宿主工程自行提供该原生库（放入 `app/src/main/jniLibs/<abi>/`），否则会在运行时抛出 `UnsatisfiedLinkError`。

---

## 3. 快速开始

### 3.1 BLE 固件升级

```java
import com.feasycom.ota.XmodemBleOta;
import com.feasycom.ota.callback.XmodemOtacallback;

XmodemBleOta ota = XmodemBleOta.getInstance();
ota.setOtaFile(new File("/sdcard/Download/firmware.bin"));
ota.startOta(bluetoothDevice, new XmodemOtacallback() {

    @Override
    public void onsuccess() {
        // 升级成功
    }

    @Override
    public void onfailure(String err) {
        // 升级失败，err 为原因
    }

    @Override
    public void onprogress(float progress) {
        // 进度：0.0 ~ 1.0
    }
});
```

> 取消升级：`ota.cancel();`

### 3.2 SPP 固件升级

```java
import com.feasycom.ota.XmodemSppOta;

XmodemSppOta ota = XmodemSppOta.getInstance();
ota.setOtaFile(new File("/sdcard/Download/firmware.bin"));
ota.startOta(bluetoothDevice, new XmodemOtacallback() { /* 同上 */ });
```

### 3.3 自行管理连接

```java
FscCoreSdk core = FscCoreSdk.getInstance();
core.Init(getApplicationContext());

// BLE 扫描
core.BleScan(10000L, new FscBleScanCallback() {
    @Override public void onBleScanStop() { }
    @Override public void onScanResult(int callbackType, ScanResult result) { }
    @Override public void onBatchScanResults(List<ScanResult> results) { }
    @Override public void onScanFailed(int errorCode) { }
});

// BLE 连接
core.FscBleConnect(device, new FscBleConnectCallback() {
    @Override public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) { }
    @Override public void onPrepared(int mtu) { }         // 服务发现 + MTU 协商完成
    @Override public void onpacketSend(byte[] value) { }
    @Override public void onpacketReceived(byte[] value) { }
    @Override public void onErro(String erro) { }
});

// 读写
core.sendBleValue(data, true);   // true = Write Without Response
core.readBleCharacteristicValues(serviceUuid, charUuidList, readCallback);
```

---

## 4. API 参考

### 4.1 `FscCoreSdk`（单例，实现 `FscCoreApi`）

| 方法 | 说明 |
| --- | --- |
| `static FscCoreSdk getInstance()` | 获取单例 |
| `void Init(Context)` | 初始化，建议传 `getApplicationContext()` |
| `BluetoothAdapter getBluetoothAdapter()` | 获取蓝牙适配器 |
| `Context getAppContext()` | 获取已保存的上下文 |
| `boolean isConnected()` | BLE 或 SPP 是否任一处于连接状态 |
| 常量 `SPP` / `BLE` | 通道类型标识 |

### 4.2 BLE 相关

| 方法 | 说明 |
| --- | --- |
| `void BleScan(long timeoutMs, FscBleScanCallback)` | 开始 BLE 扫描 |
| `void StopBleScan()` | 停止扫描 |
| `boolean isBleScaning()` | 是否正在扫描 |
| `void FscBleConnect(BluetoothDevice, FscBleConnectCallback)` | 连接设备 |
| `boolean isBleconnected()` | 是否已连接 |
| `void FscBleDisconnect()` | 断开连接（正常流程） |
| `void FscBleDisconnectImmediately()` | 立即断开 |
| `void sendBleValue(byte[] value, boolean writeNoResponse)` | 写特征值 |
| `void sendBleValue(byte[], boolean, BleManage.BleWriteCallback)` | 写特征值（带结果回调） |
| `void readBleCharacteristicValues(UUID service, List<UUID> chars, BleManage.BleReadCallback)` | 读特征值 |
| `void Blerelease()` | 释放 BLE 资源 |

### 4.3 SPP 相关

| 方法 | 说明 |
| --- | --- |
| `void SppScan(long timeoutMs, FscSppScanCallback)` | 开始 SPP 扫描 |
| `void StopSppScan()` | 停止扫描 |
| `boolean isSppScaning()` | 是否正在扫描 |
| `void FscSppConnect(BluetoothDevice, FscSppConnectCallback)` | 连接设备 |
| `boolean isSppconnected()` | 是否已连接 |
| `void FscSppDisconnect()` | 断开连接 |
| `void sendSppValue(byte[] value, SppManage.SppSendCallback)` | 发送数据 |
| `void Spprelease()` | 释放 SPP 资源 |

### 4.4 回调接口

| 接口 | 方法 |
| --- | --- |
| `FscBleScanCallback` | `onScanResult(int, ScanResult)`、`onBatchScanResults(List<ScanResult>)`、`onScanFailed(int)`、`onBleScanStop()` |
| `FscBleConnectCallback` | `onConnectionStateChange(BluetoothGatt,int,int)`、`onPrepared(int mtu)`、`onpacketSend(byte[])`、`onpacketReceived(byte[])`、`onErro(String)` |
| `FscSppScanCallback` | `onScanResult(FscSppBean)`、`onSppScanStop()` |
| `FscSppConnectCallback` | `onConnected()`、`onDisConnected()`、`onpacketReceived(byte[])`、`onErro(String)`、`onDeviceInfoReady(HashMap<DeviceInfo,String>)` |
| `BleManage.BleReadCallback` | `onReadComplete(HashMap<UUID,String>)`、`onReadFailed(String)` |
| `BleManage.BleWriteCallback` | `onWriteSuccess()`、`onWriteFailed(String)` |
| `SppManage.SppSendCallback` | `onSendSuccess()`、`onSendFailed(String)` |
| `XmodemOtacallback` | `onsuccess()`、`onfailure(String)`、`onprogress(float)` |

### 4.5 OTA 与工具类

| 类 | 主要方法 |
| --- | --- |
| `XmodemBleOta` | `getInstance()`、`setOtaFile(File)`、`startOta(BluetoothDevice, XmodemOtacallback)`、`cancel()` |
| `XmodemSppOta` | `getInstance()`、`setOtaFile(File)`、`startOta(BluetoothDevice, XmodemOtacallback)` |
| `XmodemUtils` | 构造 `XmodemUtils(byte[] data)`、`getData(byte)`、`setCompleteCallback`、`setProgressUpdateCallback`；提供 `OTA_STATUS_*` 状态常量 |
| `TeaCode` | `encryptBitstream(byte[])`、`decryptBitstream(byte[])`、`feasycomDecryption(byte[])`、`getDfuFileInformation(byte[])`（均为 native） |
| `FileUtil` | `readFile(String)`、`readFileToByte(String)`、`readFileSize(String)`、`bytesToHex(...)`、`hexToByte(String)`、`getDfuFileInformation(byte[])`、`getModelName(String)`、`getAppVersion(String)`、`getBootLoaderVersion(String)`、`getFileAbsolutePath(Context, Uri)` 等 |
| `CRC16` | `crc16_ccitt(byte[], int)` |
| `FACP` | `cacChecksum(byte[], int)`（native） |
| `EncryptInfo` | `create(String, String, String)`、`createRandom(String)`（native）、`isEncryptAlgorithmUniversal()`、`isEncryptAlgorithmBeacon()`、`getEncryptAlgorithm()`、`getPassword()`、`getRandomNumber()`；常量 `ENCRYPT_WAY`、`ENCRYPT_BEACON`、`ENCRYPT_UNIVERSAL` |
| `EncryptedUtil` | `covAssicAndByte(String, String)`、`hexToByte(String)` |
| `FscEncryptApi` / `FscEncryptApiImp` | `getEncrypted()`、`getBeaconEncrypted(String, String)`、`verifyEncrypted(byte[])` |
| `DfuFileInfo` | `bootloader`、`versionStart`、`version_soft_end`、`type_model` 字段及 getter/setter |
| `BleUuids` | `SERVICE_UUID`、`NOTIFY_UUID`、`WRITE_UUID`、`DEVICE_INFO_SERVICE_UUID`、`MODEL_NUMBER_UUID`、`SERIAL_NUMBER_UUID`、`FIRMWARE_REVISION_UUID` 等常量 |

---

## 5. Xmodem OTA 状态码

`XmodemUtils` 提供如下状态常量：

| 常量 | 含义 |
| --- | --- |
| `OTA_STATUS_BEGIN` | 升级开始 |
| `OTA_STATUS_PROCESSING` | 升级进行中 |
| `OTA_STATUS_FINISH` | 升级完成 |
| `OTA_STATUS_FAILED` | 升级失败 |
| `OTA_STATUS_VERIFY_FAILED` | 校验失败 |
| `OTA_STATUS_VERIFY_MODEL_FAILED` | 模块型号校验失败 |
| `OTA_STATUS_VERIFY_APP_VERSION_FAILED` | App 版本校验失败 |
| `OTA_STATUS_AT_FAILED` | AT 指令交互失败 |

---

## 6. 常见问题（FAQ）

**Q1：运行时报 `UnsatisfiedLinkError: dlopen failed: library "libJniLibrary.so" not found`？**

AAR 内只包含 `libFeasyBlueLibrary.so`。使用加密 / 校验能力（`EncryptInfo`、`FACP`、`FscEncryptApiImp`）时，需要宿主工程自行提供 `libJniLibrary.so`（放入 `app/src/main/jniLibs/arm64-v8a/` 与 `armeabi-v7a/`）。

**Q2：扫描不到设备？**

1. 检查蓝牙权限是否已授权（Android 12+ 需 `BLUETOOTH_SCAN` / `BLUETOOTH_CONNECT`）
2. Android 11 及以下需要定位权限且定位开关已打开
3. 确认 `Init()` 已调用

**Q3：BLE 写入返回失败？**

- 确认特征值属性：`sendBleValue(data, true)` 用于 Write Without Response，`false` 用于 Write
- 部分固件只接受 Write Without Response
- 检查 MTU 是否已协商完成（`onPrepared(int mtu)` 回调之后再进行写入）

**Q4：升级进度 `onprogress(float)` 的值域？**

`0.0 ~ 1.0`，乘以 100 即为百分比。

**Q5：SPP 与 BLE 可以同时连接吗？**

SDK 内部各自维护连接状态，`isConnected()` 表示任一通道已连接。建议同一时刻只使用一个通道进行升级。

---

## 7. 技术支持

请在本仓库提交 Issue。
