# m53-ble-protocol Specification

## Purpose
TBD - created by archiving change add-m5-network-comms-docs. Update Purpose after archive.
## Requirements
### Requirement: 文档覆盖 BLE 协议架构
docs/19-ble.md MUST 涵盖 BLE 协议栈分层(Physical / Link / HCI / L2CAP / ATT / GATT / GAP / SMP)及各层职责。

#### Scenario: 读者能描述 BLE 协议栈分层
- **WHEN** 读者阅读文档 §1-§2 后
- **THEN** 读者能画出 7 层 BLE 协议栈图并解释每层职责

#### Scenario: 文档包含 BLE 协议栈 Mermaid 图
- **WHEN** 读者查阅 §1 架构图
- **THEN** 图中 MUST 包含 Controller(Link Layer + HCI)/ Host(L2CAP + ATT + GATT + GAP + SMP) 分层

### Requirement: 文档覆盖 GAP 角色与状态
docs/19-ble.md MUST 涵盖 GAP 层 4 种角色(Broadcaster / Observer / Peripheral / Central)与连接状态机。

#### Scenario: 读者能区分 4 种 GAP 角色
- **WHEN** 读者查阅 §3 GAP 章节
- **THEN** 读者能描述每个角色的广告/扫描/连接行为与典型应用场景

#### Scenario: 读者能描述连接状态机
- **WHEN** 读者查阅 §3 状态机章节
- **THEN** 读者能画出 Standby → Advertising → Connecting → Connected 状态转移图

### Requirement: 文档覆盖 GATT 服务/特征/描述符
docs/19-ble.md MUST 涵盖 GATT 的 Service / Characteristic / Descriptor 三层模型,以及 Read/Write/Notify/Indicate 四种操作。

#### Scenario: 读者能设计 GATT 数据模型
- **WHEN** 读者查阅 §4 GATT 章节
- **THEN** 读者能画出自定义服务的 Service → Characteristic → Descriptor 层级

#### Scenario: 读者能选择通知方式
- **WHEN** 读者查阅 §4 Notify vs Indicate 章节
- **THEN** 读者能描述两者的确认机制差异与适用场景

### Requirement: 文档覆盖广播(Advertising)
docs/19-ble.md MUST 涵盖广播包结构(ADV_IND / ADV_DATA / SCAN_RSP)、广播间隔、广播信道(37/38/39)。

#### Scenario: 读者能配置广播数据
- **WHEN** 读者查阅 §5 Advertising 章节
- **THEN** 读者能找到广播数据字段(Service UUID / Local Name / Manufacturer Data)的配置示例

### Requirement: 文档覆盖连接参数
docs/19-ble.md MUST 涵盖连接间隔(Connection Interval)、从机延迟(Slave Latency)、监督超时(Supervision Timeout)三个核心参数及其权衡。

#### Scenario: 读者能权衡功耗与吞吐
- **WHEN** 读者查阅 §6 连接参数章节
- **THEN** 读者能基于功耗/吞吐/响应延迟三个维度选择合适的连接参数

### Requirement: 文档覆盖 MTU 与数据分片
docs/19-ble.md MUST 涵盖 ATT MTU 协商、数据分片(Data Fragmentation)与单包最大 20 字节的限制。

#### Scenario: 读者能配置 MTU
- **WHEN** 读者查阅 §7 MTU 章节
- **THEN** 读者能找到 MTU Exchange Request 与协商后的有效载荷计算

### Requirement: 文档覆盖加密与配对
docs/19-ble.md MUST 涵盖 BLE 配对(Just Works / Passkey / OOB)与加密(LE Secure Connections / legacy pairing)概念。

#### Scenario: 读者能为产品选择配对模式
- **WHEN** 读者查阅 §8 安全章节
- **THEN** 读者能基于安全等级/UX 复杂度为产品选择合适的配对模式

### Requirement: 文档覆盖 Embassy 中的 BLE 抽象
docs/19-ble.md MUST 描述 BLE 协议到 Embassy 抽象的映射(Controller trait / Host 任务 / GATT 事件流),不深入特定平台。

#### Scenario: 读者能理解 BLE 任务在 Embassy 中如何运行
- **WHEN** 读者查阅 §9 Embassy 抽象章节
- **THEN** 读者能描述 BLE Host 任务如何通过 embassy-executor 调度、与 PHY Controller 的事件交互

### Requirement: 文档长度与质量基线
docs/19-ble.md MUST 达到 ~800 行,包含至少 1 个 Mermaid 协议栈图,引用至少 15 处 BLE spec 关键概念与 Embassy 抽象对应,0 emoji。

#### Scenario: 文档达到 M3/M4 质量基线
- **WHEN** 文档完成后
- **THEN** 文档行数 ≥ 700, Mermaid 图 ≥ 1, 协议概念引用 ≥ 15, 0 emoji

