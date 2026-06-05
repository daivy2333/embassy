# Spec: M5.2 embassy-usb 学习文档

> Version: 1.0
> Last updated: 2026-06-05
> 适用平台: STM32 USB / nRF USBD / RP USB / ESP32 USB / 任何 embassy-usb 兼容 USB IP

## ADDED Requirements

### Requirement: 文档覆盖 embassy-usb 整体架构
docs/18-usb.md MUST 涵盖 embassy-usb crate 的核心组件(Builder / Device / DeviceClass / Config)及其相互关系。

#### Scenario: 读者能描述 embassy-usb 模块结构
- **WHEN** 读者阅读文档 §1-§3 后
- **THEN** 读者能解释 Builder 模式、Device 抽象、Class 注册机制

#### Scenario: 文档包含架构 Mermaid 图
- **WHEN** 读者查阅 §1 架构图
- **THEN** 图中 MUST 包含 Host/Device/Endpoint/Control/Bulk/Interrupt/ISO 等核心对象

### Requirement: 文档覆盖 USB 描述符体系
docs/18-usb.md MUST 涵盖 Device Descriptor / Configuration Descriptor / Interface Descriptor / Endpoint Descriptor / String Descriptor 的层级关系与字段含义。

#### Scenario: 读者能描述 USB 描述符树
- **WHEN** 读者查阅 §2 描述符章节
- **THEN** 读者能画出 Device → Configuration → Interface → Endpoint 的层级图

#### Scenario: 读者能配置复合设备描述符
- **WHEN** 读者查阅 §2 + §10 实战示例
- **THEN** 读者能找到 CDC + MSC 复合设备的描述符配置代码

### Requirement: 文档覆盖端点机制
docs/18-usb.md MUST 涵盖 Control / Bulk / Interrupt / Isochronous 四种端点类型的特性与 embassy-usb API。

#### Scenario: 读者能为每种端点类型选择 API
- **WHEN** 读者查阅 §3 端点章节
- **THEN** 读者能找到 EndpointOut::write / EndpointIn::read 各端点类型的 API 路径

### Requirement: 文档覆盖设备类(Device Class)
docs/18-usb.md MUST 涵盖 embassy-usb 内置的 CDC(ACM)/MSC/HID 等设备类的使用方式,以及如何自定义 Class。

#### Scenario: 读者能实现 CDC 串口
- **WHEN** 读者查阅 §4 CDC 章节 + §10 实战示例
- **THEN** 读者能找到 CdcAcmClass 的实例化与读写示例

#### Scenario: 读者能自定义设备类
- **WHEN** 读者查阅 §4 自定义 Class 章节
- **THEN** 读者能找到实现 Handler trait 的最小自定义 Class 示例

### Requirement: 文档覆盖跨 USB IP 平台对照
docs/18-usb.md MUST 包含跨 USB IP 平台的对比表(STM32 USB / nRF USBD / RP USB / ESP32 USB),覆盖至少 10 维指标(USB 速率 FS/HS/SS、端点数、DMA、中断占用、bus reset 处理、SOF 支持、文档质量、社区活跃度、license、稳定性)。

#### Scenario: 读者能选择适合的 USB 平台
- **WHEN** 读者查阅 §11 平台对比表
- **THEN** 读者能基于速率/端点数/HS 支持做出平台选型决策

### Requirement: 文档长度与质量基线
docs/18-usb.md MUST 达到 ~1000 行,包含至少 1 个 Mermaid 架构图,引用至少 20 处 embassy-usb 源码,0 emoji。

#### Scenario: 文档达到 M3/M4 质量基线
- **WHEN** 文档完成后
- **THEN** 文档行数 ≥ 900, Mermaid 图 ≥ 1, 源码引用 ≥ 20, 0 emoji
