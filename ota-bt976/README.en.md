# Feasycom BT976 BLE OTA SDK

Over-the-air (BLE) firmware upgrade SDK for the BT976 module family. The SDK writes a firmware image to the device over a BLE GATT connection; the device verifies the image (CRC / SHA-256) and boots into the new firmware.

- Version: **1.0.1**
- Supported modules: BT976 series (BLE)
- minSdk: 24 (Android 7.0)
- Toolchain: JDK 11, Android Gradle Plugin 8.x

---

## 1. Features

- **Standard BLE GATT upgrade** — writes the image packet by packet, verifying every flash sector
- **SHA-256 digest provisioning** — the firmware digest is sent to the device (two packets) before the transfer
- **Per-sector verification** — after each 4 KB sector a status read is performed, so errors surface early
- **Automatic write retry** — a failed `writeCharacteristic` rolls the sequence back and retries (up to 5 times)
- **High connection priority** — `CONNECTION_PRIORITY_HIGH` is requested after MTU negotiation
- **Conservative pacing defaults** — tuned to work across most Android devices (see section 7)
- **Bundled ProGuard rules** — the public API is kept automatically when the app enables R8

---

## 2. Integration

### 2.1 Use the prebuilt AAR

1. Copy `aar/ota-bt976-1.0.1.aar` into your app module's `libs/` directory
2. Add the dependency:

```gradle
dependencies {
    implementation files('libs/ota-bt976-1.0.1.aar')
}
```

> The AAR bundles `proguard.txt` (consumer rules), so no extra configuration is needed when `minifyEnabled true` is enabled in your app.

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

> `BLUETOOTH_SCAN` and `BLUETOOTH_CONNECT` must be requested at runtime. Scanning requires location permission on Android 11 and below.

---

## 3. Quick start

```java
import com.feasycom.ota_976_lib.sdk.Ota976Sdk;
import com.feasycom.ota_976_lib.sdk.OtaBt976Api;
import com.feasycom.ota_976_lib.sdk.OtaCallBack;

OtaBt976Api ota = Ota976Sdk.getInstance();

// 1) Provide the firmware file (complete .bin image)
ota.setOtaFile(new File("/sdcard/Download/BT976_Update.bin"));

// 2) Start the upgrade; newSdkVer = true selects the new upgrade flow
ota.StartOta(bluetoothDevice, /* newSdkVer = */ true, new OtaCallBack() {

    @Override
    public void onSuccess() {
        // upgrade completed
    }

    @Override
    public void onFailure(String err) {
        // upgrade failed; err describes the reason
    }

    @Override
    public void onProgress(int progress) {
        // 0 ~ 100
    }

    @Override
    public void onTips(String tips) {
        // progress hints (connecting, verifying, writing sector, ...)
    }
});
```

Kotlin:

```kotlin
val ota = Ota976Sdk.getInstance()
ota.setOtaFile(File("/sdcard/Download/BT976_Update.bin"))
ota.StartOta(device, true, object : OtaCallBack {
    override fun onSuccess() { /* success */ }
    override fun onFailure(err: String?) { /* failure */ }
    override fun onProgress(progress: Int) { /* progress */ }
    override fun onTips(tips: String?) { /* hints */ }
})
```

### 3.1 Managing the connection yourself (optional)

`StartOta()` handles connecting, service discovery and MTU negotiation internally. If your app already owns BLE connection management, you can connect and write manually:

```java
ota.init(context);
ota.ConnectBleDev(context, device, new FscBleConnectCallback() {
    @Override public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) { }
    @Override public void onPrepared(int mtu) { }           // services + MTU ready
    @Override public void onpacketSend(byte[] value) { }     // per-packet sent
    @Override public void onpacketReceived(byte[] value) { } // per-packet received
    @Override public void onErro(String erro) { }
});

// writeNoResponse MUST be true (see section 8)
ota.sendBleValue(data, true);
```

---

## 4. API reference

### 4.1 `Ota976Sdk`

| Method | Description |
| --- | --- |
| `static OtaBt976Api getInstance()` | Returns the thread-safe singleton |

### 4.2 `OtaBt976Api`

| Method | Description |
| --- | --- |
| `void init(Context context)` | Initialize; pass `getApplicationContext()` |
| `void setOtaFile(File otaFile)` | Sets the firmware image (complete `.bin` file) |
| `void StartOta(BluetoothDevice device, boolean newSdkVer, OtaCallBack callback)` | Starts the upgrade; `newSdkVer=true` uses the new flow (target address `0x18061000` and the completion command) |
| `void ConnectBleDev(Context, BluetoothDevice, FscBleConnectCallback)` | Connect manually (optional) |
| `void sendBleValue(byte[] value, boolean writeNoResponse)` | Writes to the data characteristic (must be `true` for OTA data) |
| `void sendBleValue(byte[], boolean, OtaBt976imp.BleWriteCallback)` | Same as above with a write result callback |
| `BluetoothDevice getConnectedBleDevice()` | Returns the currently connected device |
| `void Disconnect()` | Disconnects and releases resources |

### 4.3 `OtaCallBack`

| Callback | Description |
| --- | --- |
| `void onSuccess()` | Upgrade succeeded (device verified the image) |
| `void onFailure(String str)` | Upgrade failed; `str` describes the reason |
| `void onProgress(int progress)` | Progress `0~100` |
| `void onTips(String str)` | Progress hint text |

### 4.4 `FscBleConnectCallback`

| Callback | Description |
| --- | --- |
| `void onConnectionStateChange(BluetoothGatt gatt, int status, int newState)` | Connection state changed |
| `void onPrepared(int mtu)` | Service discovery and MTU negotiation finished (MTU is usually 517) |
| `void onpacketSend(byte[] value)` | Called after each packet is written (debugging) |
| `void onpacketReceived(byte[] value)` | Called after each packet is received (debugging) |
| `void onErro(String erro)` | Error callback |

---

## 5. Upgrade flow (protocol summary)

Service UUID `00002600-0000-1000-8000-00805f9b34fb`

| Characteristic | UUID | Properties |
| --- | --- | --- |
| Control | `00007000-...` | Write / Notify |
| Data | `00007001-...` | Write Without Response |

| Step | Command | Description |
| --- | --- | --- |
| 1 | `0x01 0x00` + SHA-256[0..15] | First half of the firmware digest |
| 2 | `0x01 0x01` + SHA-256[16..31] | Second half of the firmware digest |
| 3 | `0x02` + address(4 LE) + size(4 LE) + packet size(4 LE) | Start upgrade; packet size = MTU − 4 (default 256) |
| 4 | `0x04` + sector(2 LE) | Request sector N (4 KB per sector = 16 packets) |
| 5 | `SEGMENTID` + 256 B payload | Data packet, 257 bytes total |
| 6 | `0x05` | Request full-image verification |
| 7 | Device replies `0x06 0x00 …` / `0x06 0x80 …` | `0x00` success, `0x80` failure |
| 8 | `0x07 0x07` + size(3 LE) + src + dst + boot address | Completion command (new flow) |

Key rules:

- **Exactly 16 packets per sector**; each packet carries a 1-byte segment id (0–15) plus 256 bytes of payload
- Data writes **must** use Write Without Response (default writes are rejected with `GATT_WRITE_NOT_PERMITTED`)
- Target address: `newSdkVer=true` → `0x18061000`; legacy flow → `0x18069000`

---

## 6. Upgrade duration

Measured with a 105 KB image (420 packets / 27 sectors):

| Scenario | Duration |
| --- | --- |
| Default conservative settings | ~25–28 s |
| Slow phone / congested link | 30–40 s |
| Legacy build (250 ms per packet) | ~125 s |

---

## 7. Pacing parameters

The SDK ships with conservative defaults (`DELAY_*` constants in `OtaBt976imp.java`):

| Constant | Default | Meaning |
| --- | --- | --- |
| `DELAY_SEGMENT_MS` | 15 ms | Gap between two data packets |
| `DELAY_SECTOR_CMD_MS` | 60 ms | Wait after the sector command acknowledgement |
| `DELAY_BEFORE_SECTOR_CHECK_MS` | 200 ms | Wait before reading the sector verification result |
| `DELAY_NEXT_SECTOR_MS` | 80 ms | Wait before sending the next sector command |
| `DELAY_SECTOR_END_MS` | 100 ms | Sector boundary wait |
| `DELAY_BEFORE_FINAL_CHECK_MS` | 400 ms | Wait before the full-image verification |

Tuning hints:

- Upgrade fails on some phone models → raise `DELAY_SEGMENT_MS` (15 → 25 → 30)
- Device computes CRC slowly → raise `DELAY_BEFORE_SECTOR_CHECK_MS` (200 → 300)
- Speed-critical and the device is stable → lower `DELAY_SEGMENT_MS` to 0 (~18 s), at the cost of compatibility

> These are compile-time constants; rebuild the SDK after changing them.

---

## 8. FAQ

**Q1. The upgrade ends with "Firmware integrity verification failed" and the device replies `06 80`.**

Check, in order:

1. **Firmware matches the device** (model, partition layout, bootloader version). Some firmware packages simply fail the device-side verification; use the matching official image.
2. **Packet sequence** — segment ids must cycle 0–15 and every sector must contain exactly 16 packets. The SDK already fixes a race where a fast write callback could emit an extra packet.
3. **Packet gap too small** — the device may drop packets; increase `DELAY_SEGMENT_MS`.

**Q2. The very first packet fails with `status=3` (GATT_WRITE_NOT_PERMITTED).**

The firmware only accepts **Write Without Response**. Use `sendBleValue(data, true)`, or let the SDK write internally (it always uses write-without-response).

**Q3. The connection drops immediately with `status=133`.**

Usually a flow mismatch (e.g. the legacy address `0x18069000` used with new firmware). Call `StartOta(device, true, callback)`.

**Q4. The upgrade is unstable on some phones (some OPPO / Xiaomi models).**

1. Make sure high connection priority is requested (the SDK does this after MTU negotiation)
2. Ask the user not to run other BLE operations concurrently
3. Increase the packet gap

**Q5. Can several devices be upgraded at the same time?**

The SDK is a singleton and supports one upgrade at a time. Upgrade devices sequentially.

---

---

## 9. Support

- Issues: please open an issue in this repository

