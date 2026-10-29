# Feasycom OTA SDK for Android

Android OTA (Over-The-Air / Firmware Upgrade) SDK collection for Feasycom Bluetooth / Wi-Fi modules.

Android 平台 OTA（固件升级）SDK 合集，适用于各系列蓝牙 / Wi-Fi 模组。

---

## SDK 列表 / Available SDKs

| SDK | 适用模组 / Devices | 通道 / Transport | 版本 / Version | 文档 / Docs |
| --- | --- | --- | --- | --- |
| [ota-bt976](ota-bt976/) | BT976 系列（BLE） | BLE GATT | 1.0.1 | [中文](ota-bt976/README.md) · [English](ota-bt976/README.en.md) |
| [fsc-core](fsc-core/) | 基础蓝牙能力（BLE / SPP）/ Core Bluetooth stack | BLE + SPP | 0.0.0.3 | [中文](fsc-core/README.md) · [English](fsc-core/README.en.md) |

> 后续 SDK 会陆续加入本仓库，目录命名规则：`ota-<模组型号>`（模组 SDK）、`fsc-<功能>`（基础能力 SDK）。
> More SDKs will be added over time. Directory naming convention: `ota-<module-model>` for module SDKs, `fsc-<feature>` for base SDKs.

---

## 仓库结构 / Repository Layout

```
OtaSdk-Android/
├── README.md                     本文件 / this index
├── ota-bt976/
│   ├── README.md                 SDK 文档（中文）
│   ├── README.en.md              SDK documentation (English)
│   └── aar/
│       └── ota-bt976-1.0.1.aar   预编译 AAR / prebuilt AAR
└── fsc-core/
    ├── README.md                 SDK 文档（中文）
    ├── README.en.md              SDK documentation (English)
    └── aar/
        └── fsc-core-0.0.0.3.aar  预编译 AAR / prebuilt AAR
```

---

## 通用说明 / General Notes

- **最低系统版本 / minSdk**：API 24（ota-bt976）/ API 28（fsc-core），以各 SDK 的 README 为准
  API 24 for ota-bt976, API 28 for fsc-core; see each SDK's README
- **语言 / Language**：Java + Kotlin 均可调用 / callable from both Java and Kotlin
- **权限 / Permissions**：各 SDK 的 README 中列出了所需蓝牙权限
  Bluetooth permissions are listed in each SDK's README.
- **混淆 / Obfuscation**：AAR 内置 consumer ProGuard 规则，使用方开启 R8 时 SDK 公开 API 会自动保留
  Each AAR ships consumer ProGuard rules, so the public API is kept automatically when R8 is enabled by the app.

---

## 问题反馈 / Issues

请在本仓库提交 Issue。
Please open an Issue in this repository.

