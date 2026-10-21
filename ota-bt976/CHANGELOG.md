# Changelog — ota-bt976

## 1.0.1 — 2026-10-21

- AAR 使用 R8 重新构建（混淆 + 裁剪），公开 API 保持不变
  AAR rebuilt with R8 (obfuscation + shrinking); public API unchanged.
- 内部实现类（`OtaBt976imp` 及其内部类）被重命名；`sdk` 包公开 API 与回调接口按内置 consumer 规则保留
  Internal classes are renamed; the public `sdk` API and callback interfaces are kept by the bundled consumer rules.
- `classes.jar` 由 32.4 KB 缩小至 27.4 KB
  `classes.jar` shrinks from 32.4 KB to 27.4 KB.

## 1.0.0 — 2026-10-21

首个公开版本。First public release.

### 功能 / Features
- BLE GATT 固件升级（服务 `00002600`，控制 `00007000`，数据 `00007001`）
- SHA-256 固件摘要下发（分两包）
- 扇区级校验（每扇区 4 KB / 16 包）
- 写失败自动回滚重发（最多 5 次）
- 连接优先级自动提升（`CONNECTION_PRIORITY_HIGH`）
- 保守节奏参数（420 包 / 105 KB 约 25~28 秒）
- 内置 consumer ProGuard 规则

### 修复 / Fixes
- 修复分包计数竞态：写回调早于计数器自增时可能多发一包，导致扇区错位、最终完整性校验失败
- 修复包序号越界导致的 `ArrayIndexOutOfBoundsException`

### 已知限制 / Known limitations
- 单例，一次仅支持一个升级任务
- 升级过程需独占 BLE 连接
