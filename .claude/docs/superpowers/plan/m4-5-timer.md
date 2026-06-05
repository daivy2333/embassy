# Plan: M4.5 Timer + PWM

> Created: 2026-06-05
> Skill: ai-engineer-workflow-v5(Auto Mode)
> 流程裁剪:跳过 OpenSpec 变更 + 跳过 Gate 2 用户 approve(Auto Mode)
> 前置:M4.1 GPIO + M4.2 UART + M4.3 SPI + M4.4 I2C
> 模板:docs/15-i2c.md 11 节结构

---

## 1. 概览

| 项 | 内容 |
|---|------|
| 产出 | `docs/16-timer.md`(700-900 行,11 节) |
| 不在范围 | 网络 / USB(M5);M3 §10 平台 timer 硬件 |
| 依赖 | M4.1 GPIO + M3.1 §5 中断 + M3.1 §6 time-driver |
| 主题焦点 | PWM 频率/占空比 + Input Capture + Counter + QEI |
| CodeGraph | 健康 |
| 预估 | A: 40m · B: 2.5h · C: 10m |

### 1.1 11 节大纲

| § | 标题 | 主题 |
|---|------|------|
| 1 | Timer 在 Embassy 中的位置 | PWM / Counter / Capture 角色 |
| 2 | Timer trait 体系 | `Pwm` / `SetDutyCycle` / `embedded_hal::Pwm` |
| 3 | 跨平台统一抽象 | `Pwm` / `SimplePwm` / `Counter` |
| 4 | PWM 配置 | 频率 / 占空比 / 对齐模式 |
| 5 | 异步 set_duty / capture waker 机制 | 状态机 |
| 6 | DMA 模式(Input Capture) | capture + DMA |
| 7 | 平台差异 | stm32 TIM / nrf PWM+ PPI / rp PWM Slice + PIO |
| 8 | 实战 1:LED 调光 | 占空比渐变 |
| 9 | 实战 2:舵机 + 电机 | 50Hz PWM + QEI 测速 |
| 10 | 跨平台对比 + 调试 | 10 维矩阵 + 5 步排查 |
| 11 | 总结 + M5 网络导览 | 引向 M5 |

预估 ~1000 行,1 Mermaid (in §5),20+ 源码引用,0 emoji。

---

## 2. 探索清单(8 项)

| # | 关注点 | 工具 |
|---|--------|------|
| 1 | stm32 `Pwm` / `SimplePwm` + TIM 复杂分类 | codegraph_explore "embassy_stm32::timer" |
| 2 | nrf `Pwm` / `PwmSeq` + PPI | codegraph_explore "embassy_nrf::pwm" |
| 3 | rp `Pwm` / `Slice` / `Channel` | codegraph_explore "embassy_rp::pwm" |
| 4 | `embedded_hal::Pwm` trait | codegraph_search "Pwm" |
| 5 | Input Capture | codegraph_search "capture" |
| 6 | examples/pwm | ls `examples/` |
| 7 | QEI / Encoder | codegraph_search "qei" / "encoder" |
| 8 | Counter | codegraph_search "Counter" |

**预算**:codegraph_explore ≥3 次、codegraph_node ≥3 次。

---

## 3-7. Phase A/B/C 流程

同 M4.1-M4.4 模板,不再详列。

### Verify A 关键:
- 8 项全覆盖
- 关键符号 file:line 定位
- 三平台对照表已草拟

### Verify B 关键:
- R1: 700-900 行
- R2: 严格 11 节
- R3: 源码引用 ≥20
- R4: 1 Mermaid
- R5: 0 emoji

---

## 6. Requirements Traceability Matrix

| Req | 描述 | Status |
|-----|------|--------|
| R1 | 行数 700-900 | (计划) |
| R2 | 严格 11 节模板 | (计划) |
| R3 | 源码引用 ≥20 | (计划) |
| R4 | ≥1 Mermaid (§5 异步 waker) | (计划) |
| R5 | 0 emoji | (计划) |
| R6 | 不重复 M3 §10,仅引用 | (计划) |
| R7 | 引用 M4.1 GPIO | (计划) |
| R8 | 实战 2 个真实 example | (计划) |
| R9 | 10 维跨平台对比矩阵 | (计划) |
| R10 | tasks/SNAPSHOT/learned 同步 | (计划) |

---

## 7. 风险与应对

| ID | 风险 | 应对 |
|----|------|------|
| RK1 | stm32 TIM 复杂分类(tim v1/v2/v3/PWM/Counter) | §7 重点 v2,其他简略 |
| RK2 | nrf PWM 复杂(PWM + PWMSeq + PPI)| §7 重点概念,§9 实战 |
| RK3 | rp PWM Slice(每个 Slice 4 channel)| §3 重点 channel A/B |
| RK4 | PWM 频率 / 占空比计算 | §4 给公式 + 例子 |
| RK5 | QEI / Encoder 三平台差异大 | §9 实战简略 |

---

## 8. Phase 边界

推迟到 M4 全部完成,5 docs + 5 plans 一次性 commit。

---

## 9-11. 附录(简版,同 M4.3 模板)
