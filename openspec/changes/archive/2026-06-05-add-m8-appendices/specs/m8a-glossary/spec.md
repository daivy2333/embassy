# m8a-glossary: 术语表规格

> Version: 0.1.0
> Last updated: 2026-06-05
> 关联文档: `docs/appendix-a-glossary.md`
> 关联里程碑: M8.A
> 关联 OpenSpec change: `add-m8-appendices`

---

## ADDED Requirements

### Requirement: 术语覆盖范围

文档 MUST 覆盖 M1-M7 27 篇文档中出现的所有关键专业术语,按 A-Z 字母索引,每条术语 1-2 句定义 + 分类标签 + M1-M7 节号引用(可选)。

#### Scenario: 术语完整

- **WHEN** 读者查阅某 embassy / 嵌入式 / 异步编程术语
- **THEN** 文档 SHALL 包含该术语的 A-Z 字母位置定义
- **AND** 标注分类标签(如 `M2.1` / `M4.1` / `ARM` / `工具` / `概念` 等)
- **AND** 给出 M1-M7 详细分析的节号引用(若适用)

#### Scenario: 跨主题术语

- **WHEN** 某术语在 M1-M7 多处出现(如 Channel / Signal / Timer)
- **THEN** 文档 SHALL 在该字母位置给出 1 句统一定义 + 列出所有相关 Mx.x 节号引用

### Requirement: 分类标签体系

文档 MUST 标准化分类标签,标签类型 SHALL 包括:

- `M1.x` / `M2.x` / ... / `M7.x`:主里程碑引用
- `ARM`:ARM 架构相关术语
- `Cortex-M`:Cortex-M 处理器术语
- `工具`:工具链(probe-rs / defmt / QEMU 等)
- `概念`:通用嵌入式 / 异步概念
- `crate`:embassy-* crate 名

#### Scenario: 标签使用

- **WHEN** 文档中出现 `**WFE** (M6.3 / ARM / 指令)`
- **THEN** 读者 SHALL 知道 WFE 详见 M6.3 §6,属于 ARM 架构的指令集术语

### Requirement: 最小术语集

文档 MUST 至少包含 100 个术语,覆盖以下 8 个主题:

- 异步编程基础:Future / async / await / Poll / Waker / Executor
- Embassy crate:embassy-executor / embassy-time / embassy-sync / embassy-futures / embassy-stm32 / embassy-nrf / embassy-rp
- 工具链:probe-rs / defmt / defmt-rtt / QEMU / cargo-generate
- 通信协议:UART / SPI / I2C / CAN / USB / BLE
- ARM / Cortex-M:WFI / WFE / SVC / PendSV / NVIC / SysTick
- 外设:GPIO / DMA / ADC / PWM / IWDG / WWDG
- 协议栈:smoltcp / USB DFU / LoRa
- 系统概念:OTA / WFE / sleep / WFI / bootloader / firmware updater

#### Scenario: 术语数量

- **WHEN** 文档完成
- **THEN** 至少 100 个 unique 术语
- **AND** 每个术语至少有 1 句中文定义
- **AND** 至少 50% 术语带 Mx.x 节号引用

### Requirement: 索引可导航

文档 MUST 在开头提供 A-Z 字母快速跳转锚点链接(类似 Wikipedia 索引)。

#### Scenario: 字母跳转

- **WHEN** 读者查找以 "S" 开头的术语(Signal / SPI / Sync / Semaphore)
- **THEN** 文档 SHALL 在开头索引中点击 "S" 直接跳转到该字母段
