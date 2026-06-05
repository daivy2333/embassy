# m74-patterns Specification

## Purpose
TBD - created by archiving change add-m7-dev-practice-docs. Update Purpose after archive.
## Requirements
### Requirement: 模式 1 — 状态机(select_biased)

文档 MUST 介绍状态机模式在 Embassy 中的实现:`embassy_futures::select` / `select_biased` / `select_all` 三种多路复用宏的差异,以及在"等待多个事件源"场景中的应用。

#### Scenario: 选型决策

- **WHEN** 读者需要"等待多个异步事件中任一完成"
- **THEN** 文档 SHALL 给出决策树:`select_biased`(优先级固定,确定性强,默认推荐) / `select`(随机轮询,公平但非确定) / `select_all`(等待全部,用于扇入合并)
- **AND** 用"按钮 + 定时器"双事件源的闪灯示例展示 3 种用法的差异

#### Scenario: 状态机实现

- **WHEN** 读者实现"按钮短按 / 长按 / 双击"三态检测
- **THEN** 文档 SHALL 给出:`enum State { Idle, WaitingDoubleClick, LongPress }` + `loop { match self.state { ... } }` + 用 `Timer::after(500.millis()).await` 作超时转移
- **AND** 解释为何 Embassy 适合此模式(`Future` 自然表达"等待条件",无显式状态变量线程)

### Requirement: 模式 2 — 生产者-消费者(Channel)

文档 MUST 介绍生产者-消费者模式在 Embassy 中的实现:`embassy_sync::Channel` / `blocking_mutex::Channel` / `pipe` 三种形式的差异,以及在"中断 → 主循环"、"任务 → 任务"通信场景中的应用。

#### Scenario: Channel 选型

- **WHEN** 读者需要在任务间传递数据
- **THEN** 文档 SHALL 给出:`embassy_sync::Channel<M, T, N>`(动态大小,通用) vs `embassy_sync::pipe::Pipe<M, N>`(字节流,适合串口数据) vs `embassy_sync::zerocopy_channel::Channel`(零拷贝,适合大块数据)
- **AND** 用"按键中断 → LED 任务"完整示例展示 Channel 用法

#### Scenario: 中断上下文生产者

- **WHEN** 读者从 GPIO 中断发送数据到 Channel
- **THEN** 文档 SHALL 给出:`#[interrupt] fn EXTI0() { CHANNEL.send_immediate(ButtonEvent::Press); }` + `#[embassy_executor::task] async fn led() { loop { let ev = CHANNEL.receive().await; } }`
- **AND** 解释 `send_immediate` 与 `try_send` 的差异(`send_immediate` 在中断中安全,`try_send` 需短临界区)

### Requirement: 模式 3 — 发布-订阅(Signal / PubSubChannel)

文档 MUST 介绍发布-订阅模式:1-to-1 用 `embassy_sync::Signal`、1-to-N 用 `embassy_sync::pubsub::PubSubChannel`、动态订阅者用 `embassy_sync::watch::Watch`。

#### Scenario: Signal 用法

- **WHEN** 读者需要 1-to-1 事件通知(完成任务 → 启动下一个任务)
- **THEN** 文档 SHALL 给出:`static SIGNAL: Signal<CriticalSectionRawMutex, ()> = Signal::new();` + `SIGNAL.signal(())` + `SIGNAL.wait().await`
- **AND** 与 `Channel` 区分(Channel 携带数据,Signal 仅触发)

#### Scenario: PubSubChannel 用法

- **WHEN** 读者需要 1-to-N 广播(传感器数据 → 多个消费者)
- **THEN** 文档 SHALL 给出:`static PUBSUB: PubSubChannel<CriticalSectionRawMutex, SensorData, 4, 4, 4> = PubSubChannel::new();` + 发布者 `publish_immediate(data)` + 订阅者 `subscriber().next().await`
- **AND** 解释泛型参数(M 互斥类型 / T 数据类型 / N 发布者容量 / N 订阅者数 / N 消息队列深度)

#### Scenario: Watch 用法

- **WHEN** 读者需要"最后值语义"(持续覆盖,新订阅者立即拿到最新值)
- **THEN** 文档 SHALL 给出:`static WATCH: Watch<CriticalSectionRawMutex, u32, 4> = Watch::new();` + `WATCH.send(42)` + `let mut sub = WATCH.did_change(|v| *v).await;`
- **AND** 解释与 PubSubChannel 的差异(Watch 是"状态",PubSub 是"事件流")

### Requirement: 模式 4 — Watchdog 模式

文档 MUST 介绍 Watchdog 模式:硬件 watchdog(独立看门狗)+ 软件 watchdog(任务心跳检测)+ Embassy 中 `embassy_time::Timer` 与 watchdog 的协作。

#### Scenario: 硬件 watchdog 集成

- **WHEN** 读者配置 STM32 IWDG
- **THEN** 文档 SHALL 给出:`let mut wdt = IndependentWatchDog::new(p.IWDG, 1_000.millis());` + 主循环中 `wdt.feed()` 周期性喂狗
- **AND** 解释 `cortex_m::interrupt::free(|cs| ...)` 临界区中能否喂狗(可以,但建议在任务上下文中喂)

#### Scenario: 软件 watchdog 模式

- **WHEN** 读者需要检测任务卡死
- **THEN** 文档 SHALL 给出:用 `embassy_time::Timer::after(100.millis())` + `select` 实现"心跳超时则报警"模式
- **AND** 与硬件 watchdog 配合使用(硬件兜底,软件精细控制)

### Requirement: 模式 5 — 低功耗调度(Executor + WFE)

文档 MUST 综合 M2.1 `embassy-executor` 与 M6.3 低功耗两篇文档,介绍"主循环用 WFE 等待 + 唤醒源(中断/定时器/任务)协程"的低功耗调度模式。

#### Scenario: Executor::run 主循环

- **WHEN** 读者理解 Embassy 如何实现低功耗
- **THEN** 文档 SHALL 解释:`Executor::run` 调用 `poll` → 任务可运行时 `wake()` 触发 `__sev()` 指令 → 主循环 `__wfe()` 等待事件 → Cortex-M `WFE` 指令在无事件时进入 sleep,被中断或事件唤醒
- **AND** 引用 `embassy-executor/src/executor.rs` 的 `Executor::run` 实现(M2.1 已分析)

#### Scenario: critical_section + deep sleep

- **WHEN** 读者实现深度睡眠
- **THEN** 文档 SHALL 给出:`cortex_m::interrupt::free(|cs| { SCB.set_sleepdeep(); __wfi(); })` 模式 + Embassy `Executor::run` 在 STM32 / RP / nRF 三平台的实现差异
- **AND** 引用 M6.3 `23-low-power.md` §5 三档功耗模式对比

### Requirement: 模式 6 — RAII 资源管理(Drop + on_drop)

文档 MUST 介绍 RAII 模式在 Rust(无传统析构,只有 `Drop` trait)中的实现,以及 Embassy 中 `on_drop` future 的应用。

#### Scenario: Drop 自动清理

- **WHEN** 读者管理外设资源(GPIO 拉高、UART 发送完)
- **THEN** 文档 SHALL 给出:`struct LedOn<'a>(&'a mut Output<'a, PA5>); impl Drop for LedOn<'_> { fn drop(&mut self) { self.0.set_low(); } }`
- **AND** 解释 RAII 在 Embassy 中的"作用域"语义(`async` 块结束时 Drop)

#### Scenario: on_drop 异步清理

- **WHEN** 读者需要在 future 取消时执行清理(发送"任务取消"通知)
- **THEN** 文档 SHALL 给出:`use embassy_futures::on_drop; let cleanup = on_drop(|| { defmt::info!("cancelled"); });` 在 future 任意分支结束时自动触发
- **AND** 解释与同步 `Drop` 的差异(on_drop 显式注册,不会与原 Drop 冲突)

### Requirement: 模式间协作与决策矩阵

文档 MUST 给出 6 大模式之间的协作关系(可组合 vs 互斥)、典型应用场景决策矩阵(根据"事件数 / 订阅者数 / 数据语义"选模式)。

#### Scenario: 决策矩阵

- **WHEN** 读者面对"如何处理事件"的设计选择
- **THEN** 文档 SHALL 给出决策表(行:场景,列:模式,值:推荐/不推荐/可选):
  - "中断 → 单任务处理" → 推荐 Signal
  - "多生产者 → 单消费者" → 推荐 Channel
  - "单生产者 → 多消费者" → 推荐 PubSubChannel
  - "持续覆盖状态" → 推荐 Watch
  - "等待多个事件源" → 推荐 select_biased
  - "任务卡死检测" → 推荐 软件 watchdog

#### Scenario: 模式组合

- **WHEN** 读者实现"按钮事件 → 状态机 → 多个 LED 任务响应"
- **THEN** 文档 SHALL 给出:Signal(按钮 → 状态机)+ 状态机(状态切换)+ PubSubChannel(状态变更 → LED 任务)
- **AND** 解释组合的合理性(每层用最适合的模式,避免"一刀切用 Channel")

### Requirement: 实战综合案例

文档 MUST 给出一个综合运用 6 模式的完整 Embassy 应用案例(完整可编译,200-300 行):电池供电的 IoT 节点,具备按钮控制 / LED 状态指示 / 周期上报 / 低功耗睡眠 / Watchdog 保护。

#### Scenario: 电池供电 IoT 节点

- **WHEN** 读者按综合案例从零开始
- **THEN** 文档 SHALL 给出:完整 `src/main.rs`(6 模式各用 1 次以上)+ `Cargo.toml`(依赖列表)+ `memory.x`(链接脚本)
- **AND** 代码 MUST 涵盖:Signal(按钮)/ Channel(上报队列)/ PubSubChannel(状态广播)/ select_biased(主事件循环)/ 软件 watchdog(任务心跳)/ RAII(外设清理)
- **AND** 给出预期功耗估算(睡眠态 < 10μA,唤醒间隔 1s,平均电流计算)

### Requirement: 模式未覆盖与推荐阅读

文档 MUST 在文末明确标注"本文未覆盖的模式"清单(Strategy / Observer / Factory / Builder / Actor 等通用模式),并给出推荐阅读资源(《Programming Rust》/《Async Rust》/ Embassy 官方 examples/)。

#### Scenario: 模式外清单

- **WHEN** 读者寻找特定模式
- **THEN** 文档 SHALL 给出:本文 6 模式聚焦 Embassy 异步并发场景,未覆盖"配置模式(Builder)"、"序列化模式(Adapter)"、"错误处理模式(Question Mark)"等通用模式
- **AND** 给出 Embassy 官方 `examples/` 目录中相关案例的路径与用途说明

