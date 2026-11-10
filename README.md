# OTA SDK for Android

Android OTA (Over-The-Air / Firmware Upgrade) SDK collection for Feasycom Bluetooth modules.

Android 平台 OTA（固件升级）SDK 合集，适用于各系列蓝牙模组。

---

## SDK 列表 / Available SDKs

### 基础能力 / Base SDKs

| SDK | 说明 / Description | 通道 / Transport | 版本 / Version | 文档 / Docs |
| --- | --- | --- | --- | --- |
| [fsc-core](fsc-core/) | 基础蓝牙能力（BLE / SPP 扫描、连接、读写、Xmodem 升级）/ Core Bluetooth stack | BLE + SPP | 0.0.0.6 | [中文](fsc-core/README.md) · [English](fsc-core/README.en.md) |
| [fsc-debug-utils](fsc-debug-utils/) | 远程排查日志（FeasyLog）/ Remote diagnostics logging | - | 1.0.0 | [中文](fsc-debug-utils/README.md) · [English](fsc-debug-utils/README.en.md) |

### 模组 OTA / Module OTA SDKs

| SDK | 适用模组 / Modules | 通道 / Transport | 版本 / Version | 文档 / Docs |
| --- | --- | --- | --- | --- |
| [ota-bt976](ota-bt976/) | BT976 系列 | BLE | 1.0.3 | [中文](ota-bt976/README.md) · [English](ota-bt976/README.en.md) |
| [ota-bt1026](ota-bt1026/) | BT1026 系列 | SPP | 1.0.0 | [中文](ota-bt1026/README.md) · [English](ota-bt1026/README.en.md) |
| [ota-bt1035](ota-bt1035/) | BT1035 系列 | SPP + BLE | 1.0.0 | [中文](ota-bt1035/README.md) · [English](ota-bt1035/README.en.md) |
| [ota-bt671](ota-bt671/) | BT671 / BT3721Vx 系列 | BLE | 1.0.0 | [中文](ota-bt671/README.md) · [English](ota-bt671/README.en.md) |
| [ota-bt681](ota-bt681/) | BT681 系列 | BLE | 1.0.0 | [中文](ota-bt681/README.md) · [English](ota-bt681/README.en.md) |
| [ota-bp106](ota-bp106/) | BP106 系列（SUOTA） | BLE | 1.0.0 | [中文](ota-bp106/README.md) · [English](ota-bp106/README.en.md) |
| [ota-bt2044gi](ota-bt2044gi/) | BT2044GI 系列 | BLE | 1.0.0 | [中文](ota-bt2044gi/README.md) · [English](ota-bt2044gi/README.en.md) |

> 目录命名规则：`ota-<模组型号>`（模组 SDK）、`fsc-<功能>`（基础能力 SDK）。
> Directory naming convention: `ota-<module-model>` for module SDKs, `fsc-<feature>` for base SDKs.

---

## AAR 依赖关系 / AAR Dependencies

| AAR | 必须一起引入 / Must be added together |
| --- | --- |
| `fsc-core` | `fsc-debug-utils` |
| `ota-bt976` | `fsc-debug-utils` |
| `ota-bt1026` | `fsc-core` + `fsc-debug-utils` |
| `ota-bt1035` | `fsc-core` + `fsc-debug-utils` |
| `ota-bt671` | `fsc-core` + `fsc-debug-utils` |
| `ota-bt681` | `fsc-debug-utils` |
| `ota-bp106` | 无 / none |
| `ota-bt2044gi` | `fsc-debug-utils` |

> 所有 AAR 均为 `implementation files(...)` 方式引入，不通过 Maven 传递依赖；第三方库（`appcompat`、`material`、`core-ktx`）由使用方 App 自行提供。
> All AARs are consumed via `implementation files(...)`; third-party libraries (`appcompat`, `material`, `core-ktx`) are provided by the host app.

---

## 仓库结构 / Repository Layout

```
OtaSdk-Android/
├── README.md                     本文件 / this index
├── fsc-core/
│   ├── README.md                 中文文档 / Chinese docs
│   ├── README.en.md              English docs
│   └── aar/fsc-core-0.0.0.6.aar
├── fsc-debug-utils/
│   ├── README.md · README.en.md
│   └── aar/fsc-debug-utils-1.0.0.aar
├── ota-bt976/
│   ├── README.md · README.en.md
│   └── aar/ota-bt976-1.0.3.aar
├── ota-bt1026/  ota-bt1035/  ota-bt671/  ota-bt681/  ota-bp106/  ota-bt2044gi/
│   └── README.md · README.en.md · aar/<sdk>-1.0.0.aar
└── ...
```

---

## 通用说明 / General Notes

- **最低系统 / minSdk**：API 24（多数模组 SDK）/ API 28（fsc-core），以各 SDK 的 README 为准
  API 24 for most module SDKs, API 28 for fsc-core; see each SDK README
- **语言 / Language**：Java + Kotlin 均可调用 / callable from both Java and Kotlin
- **权限 / Permissions**：各 SDK 的 README 中列出了所需蓝牙权限
  Bluetooth permissions are listed in each SDK's README
- **混淆 / Obfuscation**：所有 AAR 均已开启 R8 混淆并内置 consumer ProGuard 规则，使用方开启 R8 时 SDK 公开 API 会自动保留
  All AARs are built with R8 minification enabled and ship consumer ProGuard rules, so the public API is kept automatically when the app enables R8

---

## 问题反馈 / Issues

请在本仓库提交 Issue。
Please open an Issue in this repository.