# m62-firmware-updater Specification

## Purpose
TBD - created by archiving change add-m6-system-components-docs. Update Purpose after archive.
## Requirements
### Requirement: 文档覆盖 FirmwareUpdater 整体架构
docs/22-dfu.md MUST 涵盖 FirmwareUpdater / FirmwareState / FirmwareUpdaterConfig / FirmwareUpdaterError 四大核心类型及其关系。

#### Scenario: 读者能描述 Updater 与 State 的分离
- **WHEN** 读者阅读文档 §1-§3 后
- **THEN** 读者能解释 FirmwareUpdater 持有 DFU 与 FirmwareState,FirmwareState 独立管理 STATE 分区(支持单独使用做更细粒度控制)

#### Scenario: 文档包含与 BootLoader 共享状态的说明
- **WHEN** 读者查阅 §3 状态共享章节
- **THEN** 文档 MUST 解释 BootLoader 与 FirmwareUpdater 共用 STATE 分区与 Magic 字节协议,且 M6.1 已详述,本篇仅交叉引用

### Requirement: 文档覆盖完整 OTA 状态机
docs/22-dfu.md MUST 包含完整的 OTA 升级流程状态机:write_firmware → mark_updated(SWAP_MAGIC) → reset → BootLoader.swap → 应用启动 → verify_and_mark_updated → mark_booted(BOOT_MAGIC)。

#### Scenario: 文档包含完整 OTA 时序 Mermaid 图
- **WHEN** 读者查阅 §4 完整 OTA 时序图
- **THEN** 图中 MUST 展示应用、FirmwareUpdater、Flash(DFU/STATE)、BootLoader、reset 路径的完整时序,以及失败回滚分支

#### Scenario: 读者能理解 mark_booted 未调用的回滚机制
- **WHEN** 读者查阅 §4 回滚路径
- **THEN** 读者能描述"应用启动后未及时 mark_booted → 再次 reset → bootloader 看到 SWAP_MAGIC 但 is_swapped=true → 执行 revert"的完整逻辑

### Requirement: 文档覆盖 async + blocking 双 API 对称
docs/22-dfu.md MUST 提供 FirmwareUpdater 与 BlockingFirmwareUpdater 的 API 对称表,覆盖 get_state/mark_updated/mark_booted/mark_dfu/verify_and_mark_updated/write_firmware/prepare_update。

#### Scenario: 读者能选择 async 还是 blocking 版本
- **WHEN** 读者查阅 §5 双 API 对称章节
- **THEN** 文档 MUST 解释 async 适用于 async flash(如 nrf softdevice)、blocking 适用于一般 NorFlash + 嵌入式同步 flash 驱动

### Requirement: 文档覆盖错误处理
docs/22-dfu.md MUST 涵盖 FirmwareUpdaterError enum 与 verify_booted 的状态检查机制。

#### Scenario: 读者能理解 verify_booted 接受的状态集
- **WHEN** 读者查阅 §6 verify_booted 章节
- **THEN** 文档 MUST 解释 verify_booted 接受 Boot/DfuDetach/Revert 三种状态(拒绝 Swap 状态以避免覆盖正在 swap 的固件)

### Requirement: 文档覆盖签名验证
docs/22-dfu.md MUST 涵盖 verify_and_mark_updated + digest_adapters(ed25519_dalek 与 salty 二选一)的签名验证流程。

#### Scenario: 读者能选择签名后端
- **WHEN** 读者查阅 §7 签名验证决策树 Mermaid
- **THEN** 图中 MUST 标注 ed25519_dalek(性能 / std-friendly) vs salty(no_std / 代码量小)的选型权衡

#### Scenario: 读者能复制签名验证调用示例
- **WHEN** 读者查阅 §7 实战代码
- **THEN** 文档 MUST 提供 verify_and_mark_updated 调用示例,包含 public_key、signature、update_len 三参数

### Requirement: 文档覆盖关键优化
docs/22-dfu.md MUST 解释 last_erased_dfu_sector_index 去重与 prepare_update 一次性 erase 两个优化路径。

#### Scenario: 读者能理解 last_erased_dfu_sector_index 的作用
- **WHEN** 读者查阅 §8 sector 去重章节
- **THEN** 读者能解释"分块 write_firmware 时,同一 sector 已 erase 不再重复 erase"的优化原理

#### Scenario: 读者能选择 prepare_update 还是 write_firmware
- **WHEN** 读者查阅 §8 两种 API 对比章节
- **THEN** 文档 MUST 解释 prepare_update(一次性 erase DFU 区返回 partition,适合高速 USB DFU)vs write_firmware(增量 erase,适合 OTA 慢速下发)

### Requirement: 文档包含完整实战示例
docs/22-dfu.md MUST 提供完整的 OTA 升级示例,基于 examples/boot/application 目录。

#### Scenario: 读者能复制示例实现 OTA
- **WHEN** 读者查阅 §10 实战示例
- **THEN** 示例 MUST 包含 FirmwareUpdaterConfig::from_linkerfile、FirmwareUpdater::new、write_firmware 分块写、mark_updated、cortex_m::peripheral::SCB::sys_reset 完整链路

### Requirement: 文档覆盖与 USB DFU class 集成
docs/22-dfu.md MUST 在 §12 总结章节标注 DFU_DETACH_MAGIC 与 USB DFU class 的集成入口(mark_dfu → next boot 进入 USB DFU mode)。

#### Scenario: 读者能理解 USB DFU mode 入口
- **WHEN** 读者查阅 §12 USB DFU 集成章节
- **THEN** 文档 MUST 解释 mark_dfu 写入 DFU_DETACH_MAGIC,下次 boot 时 bootloader 进入 USB DFU mode(具体 USB stack 集成参见 docs/18-usb.md)

### Requirement: 文档长度与质量基线
docs/22-dfu.md MUST 达到 ≥ 1200 行,包含至少 1 个 Mermaid 图,引用至少 20 处 embassy-boot 源码(file:line 格式),0 emoji。

#### Scenario: 文档达到 M5 质量基线
- **WHEN** 文档完成后
- **THEN** 文档行数 ≥ 1200, Mermaid 图 ≥ 1, 源码引用(file:line 格式) ≥ 20, 0 emoji

