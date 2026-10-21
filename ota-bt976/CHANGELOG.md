# Changelog — ota-bt976

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
