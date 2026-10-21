# Feasycom OTA SDK for Android

Android OTA (Over-The-Air / Firmware Upgrade) SDK collection for Feasycom Bluetooth / Wi-Fi modules.

Android 平台 OTA（固件升级）SDK 合集，适用于各系列蓝牙 / Wi-Fi 模组。

---

## SDK 列表 / Available SDKs

| SDK | 适用模组 / Devices | 通道 / Transport | 版本 / Version | 文档 / Docs |
| --- | --- | --- | --- | --- |
| [ota-bt976](ota-bt976/) | BT976 系列（BLE） | BLE GATT | 1.0.1 | [中文](ota-bt976/README.md) · [English](ota-bt976/README.en.md) |

> 后续 SDK 会陆续加入本仓库，目录命名规则：`ota-<模组型号>`。
> More SDKs will be added over time. Directory naming convention: `ota-<module-model>`.

---

## 仓库结构 / Repository Layout

```
OtaSdk-Android/
├── README.md                    本文件 / this index
├── CHANGELOG.md                 仓库级变更 / repository-level changes
└── ota-bt976/
    ├── README.md                SDK 文档（中文）
    ├── README.en.md             SDK documentation (English)
    ├── CHANGELOG.md             SDK 变更记录 / SDK changelog
    └── aar/
        └── ota-bt976-1.0.1.aar  预编译 AAR / prebuilt AAR
```

---

## 通用说明 / General Notes

- **最低系统版本 / minSdk**：Android 7.0 (API 24)
- **语言 / Language**：Java + Kotlin 均可调用 / callable from both Java and Kotlin
- **权限 / Permissions**：各 SDK 的 README 中列出了所需蓝牙权限
  Bluetooth permissions are listed in each SDK's README.
- **混淆 / Obfuscation**：AAR 内置 consumer ProGuard 规则，使用方开启 R8 时 SDK 公开 API 会自动保留
  Each AAR ships consumer ProGuard rules, so the public API is kept automatically when R8 is enabled by the app.

---

## 问题反馈 / Issues

请在本仓库提交 Issue。
Please open an Issue in this repository.

