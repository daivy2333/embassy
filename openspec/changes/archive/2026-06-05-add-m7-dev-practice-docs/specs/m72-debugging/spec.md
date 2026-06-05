# m72-debugging: 调试与日志最佳实践规格

> Version: 0.1.0
> Last updated: 2026-06-05
> 适用硬件平台: STM32 (F4/H7/L4 等) / nRF52/53 / RP2040 / RP235x
> 关联文档: `docs/25-debugging.md`
> 关联里程碑: M7.2
> 关联 OpenSpec change: `add-m7-dev-practice-docs`

---

## ADDED Requirements

### Requirement: defmt 日志框架

文档 MUST 介绍 defmt 框架的核心组件(`defmt::info!` / `defmt::warn!` / `defmt::error!` / `defmt::panic!`)、格式化占位符(`{=u32}` / `{=str}` / `{}` 推断式)与二进制大小影响(零成本 vs `core::fmt` 体积对比)。

#### Scenario: defmt 基础使用

- **WHEN** 读者在 Embassy 应用中调用 `defmt::info!("LED toggled, count = {}", count)`
- **THEN** 文档 SHALL 说明:需在 `Cargo.toml` 引入 `defmt = "0.3"`、调用 `defmt::timestamp!()` 提供时间戳(默认无时间)、`defmt::println!` 与 `defmt::info!` 区别
- **AND** 解释 RTT 链路下 `defmt-print` 解码输出的格式

#### Scenario: defmt 与 panic_handler 集成

- **WHEN** 读者注册 `#[panic_handler]`
- **THEN** 文档 SHALL 给出:`#[panic_handler] fn panic(_info: &core::panic::PanicInfo) -> ! { defmt::panic!("{_info}"); cortex_m::asm::udf() }`
- **AND** 说明 `defmt::panic!` 与 `defmt::info!` 的差异(panic 标记为 CRITICAL 级别)

### Requirement: RTT(Real-Time Transfer)链路

文档 MUST 解释 RTT 协议的工作原理(MCU 内存区段 buffer + 主机轮询)、`rtt-target` 与 `defmt-rtt` 的关系、`defmt-print` 与 `probe-rs rtt` 命令的差异。

#### Scenario: 启动 RTT 监听

- **WHEN** 读者需要查看运行中 MCU 的日志
- **THEN** 文档 SHALL 给出两步:`probe-rs run --chip <chip>` 启动应用 → 在另一终端 `probe-rs rtt`(或 `defmt-print probe-rs run ...` 一体化命令)
- **AND** 解释 `--rtt-scan-memory` 选项的用途

#### Scenario: RTT 与 ITM/SWO 对比

- **WHEN** 读者选择日志传输方案
- **THEN** 文档 SHALL 给出 RTT vs ITM/SWO vs UART 对比表(带宽、引脚占用、阻塞性、跨调试器兼容性)

### Requirement: probe-rs 调试会话

文档 MUST 介绍 `probe-rs debug` 子命令的常用操作(continue / step / break / info breakpoints / print),以及与 `gdb-multiarch` + `probe-rs gdb` 模式的对比。

#### Scenario: 设置断点并观察

- **WHEN** 读者在源码某行设置断点
- **THEN** 文档 SHALL 给出:`probe-rs debug --chip <chip>` 进入调试器 → `break src/main.rs:42` → `continue` → `print` 局部变量
- **AND** 说明 `#[defmt::dbg]` 宏与 `probe-rs` 的协同(无需手写断点,自动打印变量)

#### Scenario: 异常与 hard fault 排查

- **WHEN** 读者遇到 MCU 进入 HardFault
- **THEN** 文档 SHALL 解释:HardFault 的常见原因(空指针、未对齐访问、栈溢出)、如何用 `cortex-m-rt` 的 panic handler 打印栈回溯、`probe-rs reset` 与 `probe-rs attach` 的差异

### Requirement: 故障排查清单

文档 MUST 提供分场景的故障排查清单(至少 8 个),覆盖:链接错误、烧录失败、运行时崩溃、RTT 无输出、`defmt::unwrap!` 失败、内存不足、Watchdog 复位、外设配置错。

#### Scenario: 链接错误

- **WHEN** 读者遇到 `rust-lld: error: undefined symbol`
- **THEN** 文档 SHALL 给出常见原因(遗漏 `memory.x`、缺 PAC feature、未启用 `rt` 特性)+ 排查步骤(检查 `.cargo/config.toml` 的 `rustflags`)

#### Scenario: 烧录失败

- **WHEN** 读者遇到 `probe-rs: failed to find a probe`
- **THEN** 文档 SHALL 给出排查步骤(检查 USB 连接、驱动安装、权限 udev 规则、`probe-rs list` 是否识别)+ 跨平台差异(Linux udev vs Windows WinUSB vs macOS 无需驱动)

#### Scenario: RTT 无输出

- **WHEN** 读者运行 `probe-rs rtt` 看不到日志
- **THEN** 文档 SHALL 给出排查清单(检查 `defmt-rtt` 是否在依赖、初始化顺序、buffer 大小、芯片是否启用 RTT 时钟)

### Requirement: 实战示例

文档 MUST 给出完整可运行的调试场景示例:在 Embassy 闪灯项目基础上,加入"按按钮切日志级别"和"hard fault 时打印栈回溯"两个实战功能。

#### Scenario: 动态日志级别

- **WHEN** 读者在代码中切换日志级别
- **THEN** 文档 SHALL 给出:使用 `static mut LOG_LEVEL: AtomicU8` 配合 `defmt::info!` 宏的过滤实现,或 `defmt::println!` 的运行时过滤变体

#### Scenario: HardFault 栈回溯

- **WHEN** 读者 MCU 崩溃
- **THEN** 文档 SHALL 给出 `cortex-m-rt` 的 `#[inline(never)] #[no_mangle] fn hard_fault(_ef: &cortex_m::scb::HardFault)` + 栈回溯打印代码 + `defmt::error!("backtrace: {:?}", backtrace)` 完整示例
