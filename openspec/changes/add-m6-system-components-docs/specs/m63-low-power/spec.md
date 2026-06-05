# Spec: M6.3 低功耗设计模式学习文档

> Version: 1.0
> Last updated: 2026-06-05
> 适用平台: embassy-stm32 / embassy-rp / embassy-mcxa(等所有自定义 Executor 实现)

## ADDED Requirements

### Requirement: 文档覆盖 Executor::run 主循环
docs/23-low-power.md MUST 涵盖 Executor::run 的主循环骨架(poll → check pending → WFE → 唤醒)及其三个平台变种。

#### Scenario: 读者能描述主循环的三个状态
- **WHEN** 读者阅读文档 §3 主循环章节
- **THEN** 读者能解释"有任务待运行 → poll;无任务且无 pending → WFE 休眠;中断/SEV 触发 → 退出 WFE 进入下一轮"

#### Scenario: 文档包含主循环 Mermaid 流程图
- **WHEN** 读者查阅 §3 主循环流程图
- **THEN** 图中 MUST 展示 poll → go_around 检查 → WFE/CoreSleep → wakeup 完整循环

### Requirement: 文档覆盖 WFE/DSB 双指令对
docs/23-low-power.md MUST 解释 `cortex_m::asm::dsb(); cortex_m::asm::wfe();` 双指令对的必要性,以及裸 WFE 的潜在问题。

#### Scenario: 读者能理解为何用 DSB+WFE 而非裸 WFE
- **WHEN** 读者查阅 §3 双指令对章节
- **THEN** 文档 MUST 解释 DSB(Data Synchronization Barrier)确保 WFE 前所有内存访问完成,避免 SEV-on-pend 与 WFE 的竞态

### Requirement: 文档覆盖 TASKS_PENDING + __pender 协议
docs/23-low-power.md MUST 涵盖 TASKS_PENDING AtomicBool 与 #[unsafe(export_name = "__pender")] 函数的协作机制。

#### Scenario: 读者能追踪一次任务唤醒
- **WHEN** 读者阅读 §4 __pender 协议章节
- **THEN** 读者能描述"Waker::wake → __pender 调用 → TASKS_PENDING.store(true) + SEV → Executor::run 在 WFE 后退出 → swap(false) → 进入 poll" 完整通路

### Requirement: 文档覆盖三档功耗模式
docs/23-low-power.md MUST 详细解释 embassy-mcxa 的 CoreSleep 三档(WfeUngated / WfeGated / DeepSleep)。

#### Scenario: 读者能选择合适的功耗档位
- **WHEN** 读者查阅 §5 三档功耗章节
- **THEN** 文档 MUST 提供"档位 × 唤醒延迟 × 功耗 × 唤醒源" 决策表

#### Scenario: 文档包含三档状态机 Mermaid 图
- **WHEN** 读者查阅 §5 状态机图
- **THEN** 图中 MUST 标注 WfeUngated/WfeGated/DeepSleep 三档以及彼此之间的迁移条件

### Requirement: 文档覆盖 deep_sleep_if_possible + critical_section
docs/23-low-power.md MUST 解释 deep_sleep_if_possible 在 critical_section 内执行的安全性原因。

#### Scenario: 读者能理解 critical_section 持锁安全性
- **WHEN** 读者查阅 §6 critical_section 持锁分析章节
- **THEN** 文档 MUST 解释"deep sleep 期间持锁安全是因为唤醒中断会先打开 critical_section,无死锁风险"

#### Scenario: 文档包含 critical_section 持锁时序 Mermaid 图
- **WHEN** 读者查阅 §6 持锁时序图
- **THEN** 图中 MUST 展示 enter cs → check inhibit → deep sleep → wakeup IRQ → exit cs → resume 完整时序

### Requirement: 文档覆盖 go_around 优化
docs/23-low-power.md MUST 解释 go_around 函数的 SEV pending 检测机制,以及为何能避免双 WFE 浪费。

#### Scenario: 读者能理解 go_around 的优化原理
- **WHEN** 读者查阅 §7 go_around 章节
- **THEN** 文档 MUST 解释"TASKS_PENDING.swap(false) 检测到 pending → 先 wfe 立即清掉 SEV 标志 → 继续 poll(避免下次 WFE 立即返回浪费)"

### Requirement: 文档覆盖三平台对比
docs/23-low-power.md MUST 包含 embassy-mcxa / embassy-rp / embassy-stm32 三平台的低功耗对比,覆盖至少 8 维指标(代码行数、功耗档位数、是否 critical_section、是否 perf_counters、是否 debug pin、唤醒延迟、典型电流、复杂度评分)。

#### Scenario: 读者能选择平台与功耗策略
- **WHEN** 读者查阅 §8 三平台对比 / §12 平台对比表
- **THEN** 读者能基于平台与功耗需求做出选型决策(mcxa 三档最完整 / rp 最简单 WFE / stm32 介于两者之间)

### Requirement: 文档覆盖唤醒源管理
docs/23-low-power.md MUST 涵盖常见唤醒源(RTC、外部中断、watchdog、UART RX)及其在不同功耗档位下的有效性。

#### Scenario: 读者能配置 RTC 周期唤醒
- **WHEN** 读者查阅 §9 RTC 唤醒章节
- **THEN** 文档 MUST 提供 stm32 low_power feature + RTC 周期唤醒示例(stop_with_rtc 模式)

### Requirement: 文档包含完整实战示例
docs/23-low-power.md MUST 提供完整的低功耗 task + main loop 节流示例。

#### Scenario: 读者能复制示例实现低功耗设计
- **WHEN** 读者查阅 §10 实战示例
- **THEN** 示例 MUST 包含 sensor task(周期采样)+ main task(等待外部中断)+ low_power feature 配置 + 实测电流标注

### Requirement: 文档长度与质量基线
docs/23-low-power.md MUST 达到 ≥ 1200 行,包含至少 1 个 Mermaid 图,引用至少 20 处 embassy-mcxa/embassy-rp/embassy-stm32 源码(file:line 格式),0 emoji。

#### Scenario: 文档达到 M5 质量基线
- **WHEN** 文档完成后
- **THEN** 文档行数 ≥ 1200, Mermaid 图 ≥ 1, 源码引用(file:line 格式) ≥ 20, 0 emoji
