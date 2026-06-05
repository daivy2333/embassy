# Spec: M5.4 LoRa 远程通信说明文档

> Version: 1.0
> Last updated: 2026-06-05
> 适用: 短篇说明文档(本 fork 无 embassy-lora crate,不做实现分析)

## ADDED Requirements

### Requirement: 文档明确说明本 fork 不做 LoRa 实现分析的原因
docs/20-lora.md MUST 在文档开头说明本 fork 缺失 `embassy-lora` crate,故仅写概念入门与集成模式参考,不做源码级分析。

#### Scenario: 读者知道本 fork 的 LoRa 文档范围
- **WHEN** 读者阅读文档 §1 概述后
- **THEN** 读者 MUST 看到"本 fork 不做 LoRa 实现分析"的明确说明及原因(本 fork 无 embassy-lora crate)

### Requirement: 文档覆盖 LoRa 物理层与扩频概念
docs/20-lora.md MUST 涵盖 LoRa 物理层(调制方式 / 扩频因子 SF / 带宽 BW / 编码率 CR)与 MAC 层(LoRaWAN Class A/B/C)的基本概念。

#### Scenario: 读者能解释 LoRa 物理层参数
- **WHEN** 读者阅读 §2 物理层章节
- **THEN** 读者能描述扩频因子 / 带宽 / 编码率对速率、灵敏度、传输距离的影响

#### Scenario: 读者能区分 LoRaWAN Class A/B/C
- **WHEN** 读者阅读 §3 LoRaWAN 章节
- **THEN** 读者能描述三个 Class 的接收窗口差异与功耗特性

### Requirement: 文档描述 Embassy 中可能的 LoRa 集成模式
docs/20-lora.md MUST 描述 Embassy 中集成 LoRa 的常见模式(SX127x over SPI + LoRaWAN 状态机),并以伪代码示例呈现。

#### Scenario: 读者能找到 SX127x 集成模式
- **WHEN** 读者查阅 §4 集成模式章节
- **THEN** 读者能找到 SX127x 驱动通过 SPI 外设 + LoRaWAN 任务在 embassy-executor 中运行的伪代码

### Requirement: 文档给出后续建议
docs/20-lora.md MUST 给出后续推进 LoRa 文档的具体建议(等待 upstream embassy-lora 同步 / 自行 vendor / 参考 stm32/esp32 实现等)。

#### Scenario: 读者获得明确的下一步建议
- **WHEN** 读者阅读 §5 后续建议章节
- **THEN** 读者能选择 3 种可行路径之一(等同步 / 自行 vendor / 参考其他平台)

### Requirement: 文档长度与质量基线
docs/20-lora.md MUST 达到 ~200 行(说明性文档,不与 M3/M4 深度对齐),0 emoji,包含 1 个 LoRaWAN Class 对比表。

#### Scenario: 文档达到说明文档基线
- **WHEN** 文档完成后
- **THEN** 文档行数 ≥ 150, Mermaid 图 ≥ 0(可选),对比表 ≥ 1(LoRaWAN Class A/B/C),0 emoji
