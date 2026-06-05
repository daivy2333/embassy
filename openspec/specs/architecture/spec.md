## Purpose

定义 Embassy 项目的架构决策和设计原则，指导开发过程中的技术选型和系统设计。

## Requirements

### Requirement: 架构决策记录

所有重要的架构决策 SHALL 以 ADR（Architecture Decision Record）形式记录，包含决策内容、原因、影响和替代方案。

#### Scenario: 记录新决策

- **WHEN** 开发者做出影响系统架构的决策（如选择数据库、设计模块边界、确定通信协议）
- **THEN** 必须创建新的 ADR 条目，包含：决策标题、决策内容、决策原因、影响范围、替代方案

#### Scenario: 查询已有决策

- **WHEN** 开发者需要了解某个架构选择的原因
- **THEN** 可以通过 grep 搜索 decision 标题或关键词快速定位相关 ADR

### Requirement: 架构原则遵循

所有架构设计 MUST 遵循项目定义的架构原则。

#### Scenario: 评估设计方案

- **WHEN** 开发者提出新的设计方案
- **THEN** 方案必须符合架构原则（如 SOLID、DRY、关注点分离），不符合时需说明理由

### Requirement: 模块边界清晰

系统模块之间 MUST 有清晰的边界和接口定义。

#### Scenario: 新增模块依赖

- **WHEN** 模块 A 需要依赖模块 B
- **THEN** 必须通过明确定义的接口交互，禁止直接访问内部实现

### Requirement: 跨平台兼容性

HAL 层实现 MUST 保持跨平台兼容性。

#### Scenario: 添加新硬件支持

- **WHEN** 开发者添加新的硬件平台支持
- **THEN** 必须实现 embassy-hal-internal 定义的标准接口，确保上层代码可移植

---

## 架构决策记录

### ADR-001: 使用 async/await 替代 RTOS

**决策**: 使用 Rust async/await 机制替代传统 RTOS 的任务调度

**原因**:
- 编译时状态机转换，零运行时开销
- 无需动态内存分配
- 单栈执行，无栈大小调优问题
- 更小的代码体积和更低的功耗

**影响**: 所有 embassy-executor 和任务调度代码

**替代方案**: FreeRTOS、Zephyr、裸机轮询

### ADR-002: 多 crate 工作空间架构

**决策**: 采用 40+ 独立 crate 的工作空间结构

**原因**:
- 细粒度的依赖管理
- 用户只需引入需要的功能
- 支持不同硬件平台独立发布
- 便于社区贡献

**影响**: 项目结构、CI/CD、版本管理

**替代方案**: 单一巨型 crate、feature flags 控制

### ADR-003: HAL 层抽象设计

**决策**: 每个硬件平台实现独立的 embassy-{platform} crate

**原因**:
- 平台特有优化空间
- 独立的发布周期
- 清晰的维护责任

**影响**: embassy-stm32、embassy-nrf、embassy-rp 等

**替代方案**: 统一 HAL 层、使用 embedded-hal trait
