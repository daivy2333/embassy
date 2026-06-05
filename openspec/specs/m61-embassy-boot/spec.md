# m61-embassy-boot Specification

## Purpose
TBD - created by archiving change add-m6-system-components-docs. Update Purpose after archive.
## Requirements
### Requirement: 文档覆盖 embassy-boot 整体架构
docs/21-boot.md MUST 涵盖 embassy-boot crate 的核心组件(BootLoader / BootLoaderConfig / FirmwareUpdater / FirmwareState / digest_adapters)及其相互关系。

#### Scenario: 读者能描述 embassy-boot 模块结构
- **WHEN** 读者阅读文档 §1-§2 后
- **THEN** 读者能解释 BootLoader 与 FirmwareUpdater 的职责分离(bootloader 二进制 vs 应用二进制视角)

#### Scenario: 文档包含架构 Mermaid 图
- **WHEN** 读者查阅 §1 架构图
- **THEN** 图中 MUST 包含 ACTIVE/DFU/STATE 三分区、BootLoader、FirmwareUpdater、应用四大对象及其交互

### Requirement: 文档覆盖三分区设计与字节级布局
docs/21-boot.md MUST 解释 ACTIVE / DFU / STATE 三分区的角色,以及 STATE 分区的字节级布局(Magic / Progress validity / Progress index)。

#### Scenario: 读者能理解 STATE 分区布局
- **WHEN** 读者查阅 §3 STATE 分区章节
- **THEN** 文档 MUST 提供字节范围表(0..1 Magic / 1..2 Progress validity / 2..(2+2N) swap progress / (2+2N)..(2+4N) revert progress),与 boot_loader.rs:136-144 的注释一致

#### Scenario: 读者能理解 DFU 至少比 ACTIVE 大 1 PAGE 的原因
- **WHEN** 读者查阅 §3 分区大小约束
- **THEN** 文档 MUST 解释 swap 算法需要额外 1 PAGE 周转空间

### Requirement: 文档覆盖 State 状态机与 Magic 字节协议
docs/21-boot.md MUST 涵盖 State enum(Boot / Swap / DfuDetach / Revert)与 4 个 Magic 常量(BOOT_MAGIC / SWAP_MAGIC / DFU_DETACH_MAGIC / REVERT_MAGIC)及其状态转换图。

#### Scenario: 读者能追踪一次完整升级的状态变迁
- **WHEN** 读者查阅 §3 状态机 Mermaid 图
- **THEN** 图中 MUST 标注 Boot → Swap → Boot(成功路径)与 Boot → Swap → Revert → Boot(失败路径)的完整路径

### Requirement: 文档覆盖 swap / revert 算法
docs/21-boot.md MUST 详细解释 swap 与 revert 两个算法,以及 progress index 的断电续传机制。

#### Scenario: 读者能理解 copy_page_once_to_active / copy_page_once_to_dfu
- **WHEN** 读者阅读 §4 swap 算法章节
- **THEN** 读者能描述每个 page 的双向拷贝(active 暂存到 dfu 下一 page,dfu 写回 active),以及 current_progress 在断电后的恢复逻辑

#### Scenario: 文档包含 swap 流程 Mermaid 图
- **WHEN** 读者查阅 §4 swap 流程图
- **THEN** 图中 MUST 展示 page 0..N 的双向拷贝顺序与 progress 更新点

### Requirement: 文档覆盖链接器集成
docs/21-boot.md MUST 解释 `from_linkerfile_blocking` 如何通过 `__bootloader_*_start/_end` 符号自动分区。

#### Scenario: 读者能配置自己的 memory.x
- **WHEN** 读者查阅 §5 链接器集成章节
- **THEN** 读者能找到 6 个 extern "C" 符号(active_start/end, dfu_start/end, state_start/end)及其在 memory.x 中的典型布局示例

### Requirement: 文档覆盖错误处理
docs/21-boot.md MUST 涵盖 BootError enum(Flash / BadMagic)与 assert_partitions 的运行时检查。

#### Scenario: 读者能诊断 BadMagic 错误
- **WHEN** 读者查阅 §6 错误处理章节
- **THEN** 文档 MUST 解释 BadMagic 发生的场景(STATE 分区被擦除/损坏)与恢复策略

### Requirement: 文档覆盖三平台薄壳差异
docs/21-boot.md MUST 包含 embassy-boot-stm32 / embassy-boot-nrf / embassy-boot-rp 三薄壳的对比,覆盖至少 10 维指标(代码行数、是否支持 softdevice、watchdog 集成、load 实现方式、PAGE_SIZE 默认值、特殊依赖等)。

#### Scenario: 读者能选择适合的平台
- **WHEN** 读者查阅 §7 平台差异 / §12 平台对比表
- **THEN** 读者能基于硬件平台(STM32 / nRF / RP)与是否需要 softdevice 等约束做出选型决策

### Requirement: 文档覆盖与 watchdog 协作
docs/21-boot.md MUST 解释 nrf 与 rp 平台的 WatchdogFlash 设计(为何需要、何时 pet/feed)。

#### Scenario: 读者能避免 swap 期间 watchdog 复位
- **WHEN** 读者查阅 §7 nrf/rp WatchdogFlash 子章节
- **THEN** 读者能理解 WatchdogFlash 在每次 erase/write/read 时 pet/feed watchdog 的原因(swap 整个 ACTIVE 分区可能耗时数秒)

### Requirement: 文档包含完整实战示例
docs/21-boot.md MUST 提供完整的 bootloader main 函数示例,基于 examples/boot 目录。

#### Scenario: 读者能复制示例搭建自己的 bootloader
- **WHEN** 读者查阅 §10 实战示例
- **THEN** 示例 MUST 包含 flash 实例化、BootLoaderConfig::from_linkerfile_blocking、BootLoader::prepare、unsafe load 完整链路

### Requirement: 文档长度与质量基线
docs/21-boot.md MUST 达到 ≥ 1200 行,包含至少 1 个 Mermaid 图,引用至少 20 处 embassy-boot/embassy-boot-* 源码(file:line 格式),0 emoji。

#### Scenario: 文档达到 M5 质量基线
- **WHEN** 文档完成后
- **THEN** 文档行数 ≥ 1200, Mermaid 图 ≥ 1, 源码引用(file:line 格式) ≥ 20, 0 emoji

