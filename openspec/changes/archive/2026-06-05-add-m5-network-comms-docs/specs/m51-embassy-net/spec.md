# Spec: M5.1 embassy-net 学习文档

> Version: 1.0
> Last updated: 2026-06-05
> 适用平台: STM32 ETH / nRF radio / RP CYW43 / ESP32 / W5500 SPI / 任何 embassy-net 兼容 PHY

## ADDED Requirements

### Requirement: 文档覆盖 embassy-net 整体架构
docs/17-net.md MUST 涵盖 embassy-net crate 的核心组件(Stack / Driver trait / Runner / Resources / Config)及其相互关系。

#### Scenario: 读者能描述 embassy-net 模块结构
- **WHEN** 读者阅读文档 §1-§3 后
- **THEN** 读者能解释 Stack、Driver、Runner 三者职责及生命周期

#### Scenario: 文档包含架构 Mermaid 图
- **WHEN** 读者查阅 §1 架构图
- **THEN** 图中 MUST 包含 Stack/Device/SocketPool 三大核心对象及其交互

### Requirement: 文档解释 smoltcp 集成方式
docs/17-net.md MUST 解释 embassy-net 如何基于 smoltcp 实现协议栈,以及与裸 smoltcp 的差异。

#### Scenario: 读者能说明 smoltcp 在 embassy-net 中的角色
- **WHEN** 读者阅读 §2 smoltcp 集成章节后
- **THEN** 读者能描述 smoltcp 由 embassy executor 驱动而非独立线程,以及 wire 设备抽象如何映射到 Driver trait

#### Scenario: 文档标注 smoltcp 为外部依赖
- **WHEN** 读者查阅 §2 依赖关系
- **THEN** 文档 MUST 标注 smoltcp 为独立 crate(本 fork vendor 或外部依赖)

### Requirement: 文档覆盖 TCP/UDP Socket API
docs/17-net.md MUST 涵盖 embassy-net 提供的 TCP/UDP socket 创建、连接、收发、关闭的完整 API 路径。

#### Scenario: 读者能编写 TCP 客户端代码
- **WHEN** 读者查阅 §3 TCP API 章节 + §10 实战示例
- **THEN** 读者能找到创建 TcpSocket、connect、read/write 的完整示例

#### Scenario: 读者能编写 UDP 收发代码
- **WHEN** 读者查阅 §3 UDP API 章节 + §10 实战示例
- **THEN** 读者能找到创建 UdpSocket、bind、send/recv 的完整示例

### Requirement: 文档覆盖 DHCP/DNS 集成
docs/17-net.md MUST 涵盖 embassy-net 内置 DHCP client 与 DNS resolver 的使用方式。

#### Scenario: 读者能配置 DHCP 自动获取 IP
- **WHEN** 读者查阅 §4 DHCP 章节
- **THEN** 读者能找到 DhcpConfig::default()、Stack::set_config_up、poll 触发 IP 分配的完整流程

#### Scenario: 读者能使用 DNS 解析域名
- **WHEN** 读者查阅 §4 DNS 章节 + §10 实战示例
- **THEN** 读者能找到 dns_query 异步调用的使用方式

### Requirement: 文档覆盖跨 PHY 平台对照
docs/17-net.md MUST 包含跨 PHY 平台的对比表(STM32 ETH / nRF radio / RP CYW43 / ESP32 / W5500 SPI),覆盖至少 10 维指标(初始化方式、速率、链接检测方式、DMA、中断占用、内存占用、稳定性、文档质量、社区活跃度、license)。

#### Scenario: 读者能选择适合的 PHY 平台
- **WHEN** 读者查阅 §11 平台对比表
- **THEN** 读者能基于速率/内存/复杂度权衡做出平台选型决策

### Requirement: 文档解释异步 API 路径与 waker 链
docs/17-net.md MUST 解释 embassy-net 的 async socket API 如何通过 waker 链与 executor 协作。

#### Scenario: 读者能追踪 waker 唤醒路径
- **WHEN** 读者阅读 §5 异步 API 章节
- **THEN** 读者能描述 PHY 中断 → smoltcp poll → socket 状态变更 → Waker::wake → executor 调度 完整路径

### Requirement: 文档覆盖错误处理与恢复
docs/17-net.md MUST 涵盖 embassy-net 的错误处理模型(超时、断开、连接失败)与恢复策略(重试、退避)。

#### Scenario: 读者能处理 socket 断开
- **WHEN** 读者查阅 §6 错误处理章节
- **THEN** 读者能找到如何检测对端关闭(socket 状态查询)与重连策略

### Requirement: 文档覆盖缓冲区管理
docs/17-net.md MUST 解释 embassy-net 的 RX/TX 缓冲区设计(Static/Dynamic 池化、零拷贝)。

#### Scenario: 读者能配置 socket 缓冲区
- **WHEN** 读者查阅 §4 缓冲区管理章节
- **THEN** 读者能找到 TcpSocket::new 时的 rx_buffer/tx_buffer 参数配置

### Requirement: 文档长度与质量基线
docs/17-net.md MUST 达到 ~1000 行,包含至少 1 个 Mermaid 架构图,引用至少 20 处 embassy-net 源码,0 emoji。

#### Scenario: 文档达到 M3/M4 质量基线
- **WHEN** 文档完成后
- **THEN** 文档行数 ≥ 900, Mermaid 图 ≥ 1, 源码引用(file:line 格式) ≥ 20, 0 emoji
