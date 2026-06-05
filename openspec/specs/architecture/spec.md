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

### ADR-004: M3 文档系列(HAL 层)统一规划

**日期**: 2026-06-05

**决策**: 为 M3 学习文档系列(`08-hal-architecture` + `09-stm32` + `10-nrf` + `11-rp`)制定统一的术语表、章节模板、对比维度与 CodeGraph 入口清单,确保 1+3 结构(架构总览 + 3 平台篇)的横向可比性。

**背景**:
- M1/M2 各篇是独立组件,无需前置规划
- M3 是 1+3 结构,若不前置规划,3 平台篇会风格漂移、读者无法横向对比
- 已通过 codegraph 探索 `embassy-hal-internal` 骨架(10 核心符号)和三平台 `Cargo.toml` 矩阵,具备制定规划的条件

**决策内容**:

#### 1. 术语统一(三平台篇共用,首次出现给定义)

| 术语 | 定义 |
|------|------|
| PAC | Peripheral Access Crate,寄存器层面的最薄抽象(stm32-metapac / nrf-pac / rp-pac) |
| HAL | Hardware Abstraction Layer,本项目指 `embassy-{platform}` |
| `embedded-hal` | 跨厂商生态 trait,本项目同时支持 0.2 / 1.0 / async / nb 四套 |
| `embassy-hal-internal` | 跨平台复用骨架(`Peri`、`PeripheralType`、`interrupt::typelevel`、`OnDrop`) |
| `Peri<'a, T>` | embassy 自有"借用式外设引用",零成本替代 `&mut T` |
| `typelevel::{Interrupt, Handler, Binding}` | 类型级中断绑定三件套,编译期保证"中断已绑定 handler" |
| `bind_interrupts!` | 用户面向宏,生成 `Binding` impl |
| `Driver` trait | `embassy-time-driver` 时间驱动接口(`now` + `schedule_wake`),链接器替换实现 |

#### 2. 章节模板(M3.2~3.4 沿用,7 节固定结构)

```
1. 平台概览(芯片家族、外设矩阵、典型应用场景)
2. PAC 来源与生成方式(stm32-metapac / nrf-pac / rp-pac)
3. HAL crate 入口与 init() 流程
4. 中断模型(平台 NVIC + bind_interrupts! 落实)
5. 时间驱动绑定(time-driver-* feature 与 hardware timer 选择)
6. 外设抽象映射(以 GPIO 为代表,呼应架构篇统一模式)
7. 平台独有特性(DMA / WiFi / USB / Bluetooth 等差异化)
```

#### 3. 横向对比 4 维度(放在 ADR,各平台篇引用)

| 维度 | 详情 |
|------|------|
| 外设异步化策略 | 中断 + waker / DMA 完成 / 状态轮询 |
| 时钟与时间驱动 | hardware timer 选择 / overflow 处理 / tick rate 配置 |
| DMA 抽象 | 是否有统一 channel 抽象 / DMA buf 对齐(STM32 用 `aligned` feature) |
| PAC 生成方式 | stm32-data 元数据驱动 / nrf-pac 手写 / rp-pac 半自动 |

#### 4. embedded-hal 矩阵(三平台支持差异)

| crate | stm32 | nrf | rp |
|-------|-------|-----|-----|
| `embedded-hal-02` | 是 | 是 | 是 |
| `embedded-hal-1` | 是 | 是 | 是 |
| `embedded-hal-async` | 是 | 是 | 是 |
| `embedded-hal-nb` | 是 | 否 | 是 |
| `prio-bits-N`(NVIC 优先级位) | 4(16 级) | 3(8 级) | 2(4 级) |
| `aligned` feature(DMA 对齐) | 是 | 否 | 否 |

#### 5. CodeGraph 入口清单(M3.1~3.4 复用,避免重复探索)

| 关注点 | CodeGraph 入口 | 文件 |
|--------|-----------------|------|
| 外设引用骨架 | `codegraph_node Peri / PeripheralType` | `embassy-hal-internal/src/peripheral.rs` |
| 中断三件套 | `codegraph_node InterruptExt`,Read `interrupt.rs` | `embassy-hal-internal/src/interrupt.rs` |
| RAII guard | `codegraph_node OnDrop` | `embassy-hal-internal/src/drop.rs` |
| 时间驱动 | Read `embassy-time-driver/src/lib.rs` | `embassy-time-driver/src/lib.rs` |
| 平台中断 mod | `codegraph_search "interrupt_mod!"` | 各平台 `src/lib.rs` |
| 平台 init | `codegraph_explore "<platform>::init"` | 各平台 `src/lib.rs` |
| GPIO 统一模式 | `codegraph_explore "<platform>::gpio"` | 各平台 `src/gpio.rs` |

**影响**:

- 改变 M3.1~3.4 的撰写顺序和模板,统一术语首次定义位置
- 提供 codegraph 入口清单,后续 3 平台篇可直接复用,减少各自从零探索的开销
- 后续若加 M3.5+ 平台(mcxa/mspm0/imxrt),沿用本模板即可

**替代方案**:

| 方案 | 弃用原因 |
|------|----------|
| 不前置规划,各平台篇独立撰写 | 风格漂移、读者无法横向对比 |
| 每平台篇独立规划 | 重复劳动,术语可能冲突 |
| 把对比表分散到各平台篇 | 读者需在 3 篇之间跳转,难形成整体认知 |

**参考**:

- CodeGraph 探索证据:本次会话 Step A1(10 核心符号清单)
- Cargo.toml 矩阵:`embassy-{stm32,nrf,rp}/Cargo.toml`
- 时间驱动设计:`embassy-time-driver/src/lib.rs`
