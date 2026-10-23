# Feasycom Fsc Core SDK

Core Bluetooth SDK. It wraps BLE / SPP scanning, connection management and characteristic read/write, adds Xmodem-based firmware upgrade over both BLE and SPP, and provides firmware parsing, CRC calculation and encryption helpers.

- Version: **0.0.0.2**
- minSdk: 29 (Android 10)
- Toolchain: JDK 11, Android Gradle Plugin 8.x
- Bundled native libs: `libFeasyBlueLibrary.so` (arm64-v8a, armeabi-v7a)

---

## 1. Features

- **BLE** — scan, connect, service discovery + MTU negotiation, characteristic read/write, connection callbacks
- **SPP** — scan, connect, data transfer, device information readout
- **BLE OTA** — `XmodemBleOta` upgrades firmware over BLE using the Xmodem protocol
- **SPP OTA** — `XmodemSppOta` upgrades firmware over the serial (SPP) channel
- **Firmware parsing** — `TeaCode` / `FileUtil` decode DFU image information (bootloader, app version, module model)
- **CRC / encryption helpers** — `CRC16`, `FACP`, `EncryptInfo`, `EncryptedUtil`, `FscEncryptApi`
- **Bundled consumer ProGuard rules** — the public API and JNI symbols survive R8 in the consuming app

---

## 2. Integration

### 2.1 Add the AAR

1. Copy `aar/fsc-core-0.0.0.2.aar` into your app module's `libs/` directory
2. Add the dependency:

```gradle
dependencies {
    implementation files('libs/fsc-core-0.0.0.2.aar')
}
```

### 2.2 Required permissions (AndroidManifest.xml)

```xml
<!-- Android 12 (API 31) and above -->
<uses-permission
    android:name="android.permission.BLUETOOTH_SCAN"
    android:usesPermissionFlags="neverForLocation" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />

<!-- Android 11 (API 30) and below -->
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" android:maxSdkVersion="30" />
```

> SPP scanning requires location permission. `BLUETOOTH_SCAN` / `BLUETOOTH_CONNECT` must be requested at runtime.

### 2.3 About the native libraries (important)

The AAR bundles `libFeasyBlueLibrary.so`. However the following classes call `System.loadLibrary("JniLibrary")`:

- `com.feasycom.ota.Teacode.bean.EncryptInfo`
- `com.feasycom.ota.Teacode.controler.FscEncryptApiImp`
- `com.feasycom.ota.Teacode.utils.FACP`

The AAR does **not** contain `libJniLibrary.so`. If you use these encryption / checksum capabilities, the host app must supply that library itself (`app/src/main/jniLibs/<abi>/`), otherwise a `UnsatisfiedLinkError` is thrown at runtime.

---

## 3. Quick start

### 3.1 BLE firmware upgrade

```java
import com.feasycom.ota.XmodemBleOta;
import com.feasycom.ota.callback.XmodemOtacallback;

XmodemBleOta ota = XmodemBleOta.getInstance();
ota.setOtaFile(new File("/sdcard/Download/firmware.bin"));
ota.startOta(bluetoothDevice, new XmodemOtacallback() {

    @Override
    public void onsuccess() {
        // upgrade completed
    }

    @Override
    public void onfailure(String err) {
        // upgrade failed
    }

    @Override
    public void onprogress(float progress) {
        // 0.0f ~ 1.0f
    }
});
```

> Cancel with `ota.cancel();`

### 3.2 SPP firmware upgrade

```java
import com.feasycom.ota.XmodemSppOta;

XmodemSppOta ota = XmodemSppOta.getInstance();
ota.setOtaFile(new File("/sdcard/Download/firmware.bin"));
ota.startOta(bluetoothDevice, new XmodemOtacallback() { /* same as above */ });
```

### 3.3 Managing the connection yourself

```java
FscCoreSdk core = FscCoreSdk.getInstance();
core.Init(getApplicationContext());

// BLE scan
core.BleScan(10000L, new FscBleScanCallback() {
    @Override public void onBleScanStop() { }
    @Override public void onScanResult(int callbackType, ScanResult result) { }
    @Override public void onBatchScanResults(List<ScanResult> results) { }
    @Override public void onScanFailed(int errorCode) { }
});

// BLE connect
core.FscBleConnect(device, new FscBleConnectCallback() {
    @Override public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) { }
    @Override public void onPrepared(int mtu) { }          // services + MTU ready
    @Override public void onpacketSend(byte[] value) { }
    @Override public void onpacketReceived(byte[] value) { }
    @Override public void onErro(String erro) { }
});

// read / write
core.sendBleValue(data, true);   // true = write without response
core.readBleCharacteristicValues(serviceUuid, charUuidList, readCallback);
```

---

## 4. API reference

### 4.1 `FscCoreSdk` (singleton, implements `FscCoreApi`)

| Method | Description |
| --- | --- |
| `static FscCoreSdk getInstance()` | Returns the singleton |
| `void Init(Context)` | Initialize; pass `getApplicationContext()` |
| `BluetoothAdapter getBluetoothAdapter()` | Underlying Bluetooth adapter |
| `Context getAppContext()` | Stored application context |
| `boolean isConnected()` | True when either BLE or SPP is connected |
| constants `SPP` / `BLE` | Transport identifiers |

### 4.2 BLE

| Method | Description |
| --- | --- |
| `void BleScan(long timeoutMs, FscBleScanCallback)` | Start BLE scan |
| `void StopBleScan()` | Stop scanning |
| `boolean isBleScaning()` | Scanning state |
| `void FscBleConnect(BluetoothDevice, FscBleConnectCallback)` | Connect a device |
| `boolean isBleconnected()` | Connection state |
| `void FscBleDisconnect()` | Disconnect |
| `void FscBleDisconnectImmediately()` | Disconnect immediately |
| `void sendBleValue(byte[] value, boolean writeNoResponse)` | Write a characteristic |
| `void sendBleValue(byte[], boolean, BleManage.BleWriteCallback)` | Write with result callback |
| `void readBleCharacteristicValues(UUID service, List<UUID> chars, BleManage.BleReadCallback)` | Read characteristics |
| `void Blerelease()` | Release BLE resources |

### 4.3 SPP

| Method | Description |
| --- | --- |
| `void SppScan(long timeoutMs, FscSppScanCallback)` | Start SPP scan |
| `void StopSppScan()` | Stop scanning |
| `boolean isSppScaning()` | Scanning state |
| `void FscSppConnect(BluetoothDevice, FscSppConnectCallback)` | Connect a device |
| `boolean isSppconnected()` | Connection state |
| `void FscSppDisconnect()` | Disconnect |
| `void sendSppValue(byte[] value, SppManage.SppSendCallback)` | Send data |
| `void Spprelease()` | Release SPP resources |

### 4.4 Callbacks

| Interface | Methods |
| --- | --- |
| `FscBleScanCallback` | `onScanResult(int, ScanResult)`, `onBatchScanResults(List<ScanResult>)`, `onScanFailed(int)`, `onBleScanStop()` |
| `FscBleConnectCallback` | `onConnectionStateChange(BluetoothGatt,int,int)`, `onPrepared(int mtu)`, `onpacketSend(byte[])`, `onpacketReceived(byte[])`, `onErro(String)` |
| `FscSppScanCallback` | `onScanResult(FscSppBean)`, `onSppScanStop()` |
| `FscSppConnectCallback` | `onConnected()`, `onDisConnected()`, `onpacketReceived(byte[])`, `onErro(String)`, `onDeviceInfoReady(HashMap<DeviceInfo,String>)` |
| `BleManage.BleReadCallback` | `onReadComplete(HashMap<UUID,String>)`, `onReadFailed(String)` |
| `BleManage.BleWriteCallback` | `onWriteSuccess()`, `onWriteFailed(String)` |
| `SppManage.SppSendCallback` | `onSendSuccess()`, `onSendFailed(String)` |
| `XmodemOtacallback` | `onsuccess()`, `onfailure(String)`, `onprogress(float)` |

### 4.5 OTA and utilities

| Class | Key methods |
| --- | --- |
| `XmodemBleOta` | `getInstance()`, `setOtaFile(File)`, `startOta(BluetoothDevice, XmodemOtacallback)`, `cancel()` |
| `XmodemSppOta` | `getInstance()`, `setOtaFile(File)`, `startOta(BluetoothDevice, XmodemOtacallback)` |
| `XmodemUtils` | constructor `XmodemUtils(byte[] data)`, `getData(byte)`, `setCompleteCallback`, `setProgressUpdateCallback`; `OTA_STATUS_*` constants |
| `TeaCode` | `encryptBitstream(byte[])`, `decryptBitstream(byte[])`, `feasycomDecryption(byte[])`, `getDfuFileInformation(byte[])` (all native) |
| `FileUtil` | `readFile(String)`, `readFileToByte(String)`, `readFileSize(String)`, `bytesToHex(...)`, `hexToByte(String)`, `getDfuFileInformation(byte[])`, `getModelName(String)`, `getAppVersion(String)`, `getBootLoaderVersion(String)`, `getFileAbsolutePath(Context, Uri)` |
| `CRC16` | `crc16_ccitt(byte[], int)` |
| `FACP` | `cacChecksum(byte[], int)` (native) |
| `EncryptInfo` | `create(String, String, String)`, `createRandom(String)` (native), `isEncryptAlgorithmUniversal()`, `isEncryptAlgorithmBeacon()`, `getEncryptAlgorithm()`, `getPassword()`, `getRandomNumber()`; constants `ENCRYPT_WAY`, `ENCRYPT_BEACON`, `ENCRYPT_UNIVERSAL` |
| `EncryptedUtil` | `covAssicAndByte(String, String)`, `hexToByte(String)` |
| `FscEncryptApi` / `FscEncryptApiImp` | `getEncrypted()`, `getBeaconEncrypted(String, String)`, `verifyEncrypted(byte[])` |
| `DfuFileInfo` | fields `bootloader`, `versionStart`, `version_soft_end`, `type_model` with getters/setters |
| `BleUuids` | `SERVICE_UUID`, `NOTIFY_UUID`, `WRITE_UUID`, `DEVICE_INFO_SERVICE_UUID`, `MODEL_NUMBER_UUID`, `SERIAL_NUMBER_UUID`, `FIRMWARE_REVISION_UUID` … |

---

## 5. Xmodem OTA status codes

| Constant | Meaning |
| --- | --- |
| `OTA_STATUS_BEGIN` | Upgrade started |
| `OTA_STATUS_PROCESSING` | Upgrade in progress |
| `OTA_STATUS_FINISH` | Upgrade finished |
| `OTA_STATUS_FAILED` | Upgrade failed |
| `OTA_STATUS_VERIFY_FAILED` | Verification failed |
| `OTA_STATUS_VERIFY_MODEL_FAILED` | Module model verification failed |
| `OTA_STATUS_VERIFY_APP_VERSION_FAILED` | App version verification failed |
| `OTA_STATUS_AT_FAILED` | AT command interaction failed |

---

## 6. FAQ

**Q1. `UnsatisfiedLinkError: dlopen failed: library "libJniLibrary.so" not found`**

The AAR ships only `libFeasyBlueLibrary.so`. When using the encryption / checksum features (`EncryptInfo`, `FACP`, `FscEncryptApiImp`) the host app must provide `libJniLibrary.so` (`app/src/main/jniLibs/arm64-v8a/` and `armeabi-v7a/`).

**Q2. No devices found while scanning**

1. Make sure Bluetooth permissions are granted (Android 12+ needs `BLUETOOTH_SCAN` / `BLUETOOTH_CONNECT`)
2. Android 11 and below also need location permission with location enabled
3. Confirm `Init()` was called

**Q3. BLE writes fail**

- `sendBleValue(data, true)` is write-without-response, `false` is write-with-response
- Some firmware only accepts write-without-response
- Wait for MTU negotiation (`onPrepared(int mtu)`) before writing

**Q4. Range of `onprogress(float)`**

`0.0f ~ 1.0f`; multiply by 100 for a percentage.

**Q5. Can BLE and SPP be connected at the same time?**

The SDK tracks both transports independently; `isConnected()` is true when either is connected. For upgrades, use one transport at a time.

---

## 7. Support

Please open an issue in this repository.
