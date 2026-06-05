# m8c-examples Specification

## Purpose
TBD - created by archiving change add-m8-appendices. Update Purpose after archive.
## Requirements
### Requirement: 三层分类结构

文档 MUST 按"主题(Mx.x) → 平台 → 文件路径"三层组织示例索引。

#### Scenario: 主题 → 平台 → 文件

- **WHEN** 读者想找"状态机"相关示例
- **THEN** 文档 SHALL 给出 `M2.4 / M7.1` 主题 → `nrf52840 / rp / stm32f4` 平台 → `examples/nrf52840/src/bin/raw_spawn.rs` 等具体文件
- **AND** 每条 1 句用途说明 + Mx.x 节号引用

### Requirement: 覆盖 embassy/examples/ 主要 bin 文件

文档 MUST 至少覆盖 embassy 仓库 `examples/` 目录中 30 个以上 `src/bin/*.rs` 文件,按平台分布:

- STM32:`stm32f3` / `stm32f4` / `stm32f7` / `stm32h7` / `stm32l0` / `stm32l4` / `stm32wb-dfu` / `stm32wba-dfu` / `stm32wl`
- nRF:`nrf51` / `nrf52810` / `nrf52840`
- RP:`rp` / `rp235x`
- 其他:`lpc55s69` / `mcxa2xx` / `mcxa5xx` / `mspm0g3507` / `mspm0l1306` / `mimxrt1062-evk` / `mps3-an536`

#### Scenario: bin 文件覆盖

- **WHEN** 读者想找 STM32F4 的 I2C 异步示例
- **THEN** 文档 SHALL 给出 `stm32f4/src/bin/i2c.rs`(M4.4 §xx 引用)
- **AND** 标注关键 API:`embassy_stm32::i2c::I2c::new` / `embedded-hal-async` 等

### Requirement: 主题映射表

文档 MUST 提供"主题 → 平台 → 文件"主表,以及"Mx.x → 关键示例"反向索引。

#### Scenario: 反向索引

- **WHEN** 读者在 M2.4 看到 `select_biased` 的引用
- **THEN** 文档 SHALL 在反向索引中给出 `M2.4 → examples/nrf52840/src/bin/multiprio.rs:59` + `examples/rp/src/bin/multiprio.rs:59`
- **AND** 简短说明此示例展示 select_biased 优先级调度

### Requirement: 每个 bin 文件的关键 API 标注

文档 MUST 为每个索引的 bin 文件标注:

- 使用的 embassy-* crate + API
- 关键外设(GPIO / UART / I2C / SPI / TIM 等)
- 引脚 / DMA 通道
- 该示例最适合学习的"模式"(模式 1-6 之一)

#### Scenario: 模式标注

- **WHEN** 文档列出 `examples/rp/src/bin/blinky.rs`
- **THEN** 文档 SHALL 标注:`模式 5(低功耗调度)+ 模式 1(状态机 Timer 周期)`
- **AND** 与 M7.4 模式章节号对应

### Requirement: 入门路径建议

文档 MUST 末尾提供"按 M1-M7 顺序的入门路径",告诉初学者"先跑哪个示例"。

#### Scenario: 入门路径

- **WHEN** 读者是 embassy 新手
- **THEN** 文档 SHALL 给出:
  - 第 1 步:`examples/rp/src/bin/blinky.rs`(最简 GPIO toggle)
  - 第 2 步:`examples/nrf52840/src/bin/button.rs`(按钮输入)
  - 第 3 步:`examples/stm32f4/src/bin/uart.rs`(UART 异步)
  - 第 4 步:`examples/rp/src/bin/multiprio.rs`(select_biased 多任务)
  - ...
- **AND** 每个示例标"预计耗时 30 分钟跑通"

