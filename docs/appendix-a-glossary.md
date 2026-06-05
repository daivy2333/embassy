# 附录 A - Embassy 学习项目术语表

> 版本: v0.1.0
> 最后更新: 2026-06-05
> 适用项目版本: Embassy `0.5+`
> 收录范围: M1-M7 27 篇文档 + embassy 通用术语

---

## 简介

本术语表收录 Embassy 学习项目 M1-M7 27 篇文档中出现的关键专业术语,按 A-Z 字母索引。每条术语给出 1-2 句中文定义,并标注分类标签(如 `M2.1` / `ARM` / `工具` / `crate` 等),便于快速定位"该术语在我关心的主题吗"以及"在 M1-M7 的哪里有详细分析"。

**术语分类标签说明**:

- `M1.x` / `M2.x` / `M3.x` / `M4.x` / `M5.x` / `M6.x` / `M7.x`:主里程碑节号引用
- `ARM`:ARM 架构相关术语
- `Cortex-M`:Cortex-M 处理器术语
- `工具`:工具链(probe-rs / defmt / QEMU 等)
- `crate`:embassy-* crate 名
- `概念`:通用嵌入式 / 异步概念
- `外设`:MCU 外设术语
- `协议`:通信协议术语

---

## A-Z 字母索引

| A | B | C | D | E | F | G | H | I | J | K | L | M |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| [A](#a) | [B](#b) | [C](#c) | [D](#d) | [E](#e) | [F](#f) | [G](#g) | [H](#h) | [I](#i) | [J](#j) | [K](#k) | [L](#l) | [M](#m) |

| N | O | P | Q | R | S | T | U | V | W | X | Y | Z |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| [N](#n) | [O](#o) | [P](#p) | [Q](#q) | [R](#r) | [S](#s) | [T](#t) | [U](#u) | [V](#v) | [W](#w) | [X](#x) | [Y](#y) | [Z](#z) |

---

## A

- **ADC** (M4.1 / 外设):模数转换器,将模拟电压转换为数字值,Embassy 提供异步 API `embassy_stm32::adc::Adc`。
- **ADR-004** (M3.1 / 概念):本项目的架构决策记录编号,定义 M3+ 文档的术语与章节模板。
- **API** (概念):应用程序编程接口,Rust 中通过 `trait` 表达,Embassy 用 `async fn in trait` 扩展为异步 API(M1.3)。
- **Async** (M1.3 / 概念):Rust 异步编程范式关键字,与 `await` 配合使用。Embassy 是单线程协作式异步。

## B

- **Bank0** (M3.4 / 外设):RP2040 的 DMA 数据通道,0 号 bank 包含 12 个 channel,2 号 bank 包含 0 个(全部发往 1 号 bank)。
- **Binary Semaphore** (M2.3 / 概念):二值信号量,资源计数为 0 或 1,用于互斥访问。Embassy 提供 `embassy_sync::Semaphore`。
- **Bit-bang** (M4.1 / 概念):通过软件控制 GPIO 翻转模拟协议(I2C / SPI / UART),通常在硬件外设不足时使用。
- **Bootloader** (M6.1 / 概念):MCU 启动时运行的第一段固件,负责加载应用固件。Embassy 提供 `embassy-boot` crate。
- **Bounded** (M2.3 / 概念):有界,指数据结构容量编译期固定(如 `Channel<M, T, 4>` 中的 4)。Embassy 全部使用 bounded,无堆分配。

## C

- **Cargo** (工具):Rust 包管理与构建工具,Embassy 项目用 `cargo build --target thumbv7em-none-eabihf` 编译。
- **cargo-generate** (工具):从 Git 模板生成项目的工具,Embassy 推荐用 `cargo generate --git embassy-rs/embassy-template`。
- **Channel** (M2.3 / M3.x / crate):跨任务传递数据的通道,API 为 `send().await` / `receive().await`,容量编译期固定。
- **CMSIS** (ARM):Cortex Microcontroller Software Interface Standard,ARM 提供的标准外设访问层。
- **CMSIS-DAP** (工具):ARM 的调试探针协议,probe-rs 与 Raspberry Pi Debug Probe 均支持。
- **Channel** (M2.3 / M3.x / crate):同 C 段,跨任务传递数据的 MPSC 通道(更详细见 M2.3 §4 与 M7.4 模式 2)。
- **Cooperative Scheduling** (M1.3 / 概念):协作式调度,任务主动让出(`await`)才切换。Embassy 单线程模型属于此类。
- **CoreSleep** (M6.3 / 概念):embassy-mcxa 的低功耗主循环三档(WfeUngated / WfeGated / DeepSleep),M6.3 §5 详述。
- **Cortex-M** (ARM):ARM Cortex-M 系列 MCU,主流嵌入式 MCU 类别。Embassy 支持 Cortex-M0 / M0+ / M3 / M4 / M7 / M33 / M55。
- **CPU** (概念):中央处理器,Embassy 单线程模型下唯一活跃 CPU。
- **critical_section** (M6.3 / crate):临界区抽象,确保一段代码不被中断打断。`cortex_m::interrupt::free` 是其 ARM 平台实现。

## D

- **D-Cache** (M3.2 / ARM):数据缓存,某些 Cortex-M7 平台有,与 DMA 共存时需手动维护一致性。
- **Data Race** (M1.3 / 概念):数据竞争,多线程同时访问同一数据。Embassy 单线程避免大部分数据竞争。
- **debug** (工具):probe-rs 子命令,提供 REPL 风格调试接口。
- **defmt** (工具 / M7.2):Embassy 生态的零成本日志框架,编译期编码节省 5-10 倍体积。
- **defmt-print** (工具):defmt 的主机端解码器,配合 RTT 链路输出可读日志。
- **defmt-rtt** (工具):defmt 的 RTT 后端,M7.2 §2.5 详述。
- **DeepSleep** (M6.3 / 概念):深度睡眠,功耗 < 1mA,仅 wakeup 源(中断 / RTC / watchdog)能唤醒。
- **DFU** (M6.2 / 概念):Device Firmware Upgrade,USB 协议标准,Embassy `embassy-boot` 实现 DFU 入口。
- **DMA** (M4.x / 外设):直接内存访问,无需 CPU 介入的数据搬运。Embassy 异步驱动通常基于 DMA。
- **DPPI** (M3.3 / 外设):nRF 的"分布式 PPI",跨外设事件路由系统,降低 CPU 中断负担。
- **Drive Mode** (M4.1 / 概念):GPIO 输出驱动模式(推挽 / 开漏),STM32 平台用 `OutputSpeed` 控制。

## E

- **EasyDMA** (M3.3 / 外设):nRF 外设的 DMA 抽象,简化外设与 RAM 之间的数据传输。
- **embedded-hal** (M1.2 / crate):Rust 嵌入式 HAL 标准 trait 集合,提供 `digital::OutputPin` / `spi::SpiBus` 等。
- **embedded-hal-async** (M3.x / crate):embedded-hal 的异步扩展,Rust 异步 trait 稳定前用 `local-ringbuf` 等 workaround。
- **Embassy** (crate):下一代 Rust 嵌入式异步框架,本文档学习对象。
- **embassy-boot** (M6.1 / crate):Embassy 官方 bootloader 框架,实现 `BootLoader` + `FirmwareUpdater` 双 API。
- **embassy-executor** (M2.1 / crate):Embassy 自研的异步执行器,单线程协作式调度。
- **embassy-futures** (M2.4 / crate):Future 组合宏库,提供 `select` / `select_biased` / `join` / `on_drop` 等。
- **embassy-net** (M5.1 / crate):Embassy 异步网络栈,基于 smoltcp。
- **embassy-sync** (M2.3 / crate):同步原语库,提供 `Channel` / `Signal` / `PubSubChannel` / `Watch` / `Mutex` / `Semaphore`。
- **embassy-time** (M2.2 / crate):时间管理库,提供 `Timer` / `Instant` / `with_timeout` 等抽象。
- **embassy-usb** (M5.2 / crate):Embassy USB 设备栈,实现 USB 2.0 device 类。
- **ErrorType** (M3.x / concept):embedded-hal 标准的错误 trait,所有外设 trait 关联类型 `type Error`。
- **Event Loop** (M5.2 / 概念):事件循环,USB 任务通过 `embassy_usb::Builder` 注册事件处理。
- **EXTI** (M3.2 / 外设):External Interrupt,STM32 的外部中断控制器,用于 GPIO 中断。
- **Executor** (M2.1 / crate):异步执行器,Embassy `embassy-executor::Executor::run` 主循环。

## F

- **Fall-through** (M4.1 / 概念):GPIO 模式选项,引脚未配置为输出时输入仍生效(STM32 特有)。
- **FIFO** (M4.x / 概念):First In First Out 队列,UART / SPI 接收常用。
- **Flash** (M6.1 / 外设):MCU 非易失存储,擦写次数有限(~10K-100K cycles)。
- **Flash Trait** (M6.1 / concept):`embedded_storage::nor_flash::NorFlash`,定义擦 / 写 / 读的 trait。
- **Future** (M1.3 / 概念):Rust 异步的"未完成值",`async fn` 返回 `impl Future`。
- **FutureUnfinished** (M2.1 / concept):`Poll::Pending` 时返回的状态,任务需要被再次 `poll`。

## G

- **GPIOTE** (M3.3 / 外设):nRF 的 GPIO Task and Event 系统,4 个 channel 可在无 CPU 介入下处理 GPIO 事件。
- **GPIO** (M4.1 / 外设):通用输入输出,MCU 最基础的外设,Embassy 异步 API 见 M4.1。
- **GPIO_Input** (M4.1 / 外设):GPIO 输入模式,`embassy_stm32::gpio::Input::new()`。
- **GPIO_Output** (M4.1 / 外设):GPIO 输出模式,`Output::new()` / `set_high()` / `set_low()`。
- **GPIOTE** (M3.3 / 外设):同前,nRF 平台 GPIO 中断子系统。

## H

- **HAL** (M3.1 / crate):Hardware Abstraction Layer,硬件抽象层,提供 `embassy_stm32::init()` 等高级 API。
- **HardFault** (M5 / 概念):Cortex-M 异常,空指针 / 未对齐 / 栈溢出等导致。`panic-probe` crate 打印栈回溯(M7.2 §7)。
- **HFSR** (M7.2 / ARM):HardFault Status Register,记录 HardFault 类别。
- **HRTIM** (M4.5 / 外设):High-Resolution Timer,STM32 高级定时器,提供 ns 级 PWM。

## I

- **I-Cache** (M3.2 / ARM):指令缓存,某些 Cortex-M7 平台有。
- **I2C** (M4.4 / 协议):两线制串行协议(SCL + SDA),主从架构,100kHz / 400kHz / 1MHz。
- **IWDG** (M6.3 / M7.4 / 外设):独立看门狗(Independent Watchdog),使用 LSI 时钟,主时钟失效仍能复位。
- **ISR** (M4.1 / 概念):中断服务例程,`#[interrupt]` 函数。Embassy 的 `InterruptExecutor` 可在 ISR 内运行 task(M3.2)。

## J

- **J-Link** (工具):SEGGER 调试探针,商业产品,probe-rs 兼容。
- **JLINK-OB** (工具):Nordic nRF DK 上的板载 J-Link,默认探针。
- **join** (M2.4 / 概念):`embassy_futures::join` 宏,等待多个 future 全部完成(类似 `select_all`)。

## K

- **Knob** (M4.1 / 概念):旋钮输入,通过 ADC 读取电压分压,通常用 `EmbassyChannel` 实现。

## L

- **Level** (M4.1 / 概念):GPIO 输出电平,`Level::High` / `Level::Low`。
- **Linker Script** (M7.1 / 工具):`memory.x`,告诉 linker FLASH / RAM 地址。STM32 手写,RP 平台由 HAL 提供。
- **LoRa** (M5.4 / 协议):远距离低功耗广域网协议,Embassy 通过 `embassy-lora` crate 集成 Semtech SX127x 系列。
- **LPUART** (M3.2 / 外设):Low-Power UART,STM32L0 / STM32WL 等低功耗平台的 UART 外设。
- **LSI** (M7.4 / 概念):Low-Speed Internal oscillator,STM32 内部 32kHz 时钟,IWDG 默认时钟源。
- **LTO** (M7.1 / 工具):Link-Time Optimization,链接期优化。`[profile.release] lto = true` 启用。

## M

- **main** (M7.1 / 概念):`#[embassy_executor::main]` 宏,展开为 `cortex-m-rt` 入口 + Executor::run。
- **Magic 字节** (M6.1 / 概念):bootloader 状态分区的特殊字节,标识当前是否"已交换" / "需要回滚" 等状态。
- **MCU** (概念):微控制器(Microcontroller Unit),嵌入式系统核心芯片。
- **Memory.x** (M7.1 / 工具):链接脚本,定义 FLASH / RAM 地址。
- **Mock** (M7.3 / 概念):测试中替代真实硬件的对象,通过 trait 抽象 + 手写或 `mockall` 生成。
- **MPSC** (M2.3 / 概念):Multi-Producer Single-Consumer,多生产者单消费者模式,`Channel` 是典型实现。

## N

- **NVIC** (ARM):Nested Vectored Interrupt Controller,Cortex-M 中断控制器。
- **Nominated** (M3.4 / 概念):RP2040 PIO 程序"被提名",等待主 SM 接受(详见 M3.4 §5)。

## O

- **OTA** (M6.2 / 概念):Over-The-Air 升级,无线固件升级。Embassy `FirmwareUpdater` 是核心 API。
- **on_drop** (M2.4 / M7.4 / 概念):`embassy_futures::on_drop`,在 future 任意结束路径触发的清理闭包。

## P

- **PAC** (M3.1 / crate):Peripheral Access Crate,MCU 外设寄存器直接映射,Embassy HAL 在 PAC 之上构建。
- **Panic Handler** (M7.2 / 概念):`#[panic_handler]` 函数,panic 时调用,通常用 `panic-probe` 打印栈回溯。
- **PendSV** (ARM):Cortex-M 异常,用于任务上下文切换(RTOS 常用)。
- **Pico** (工具):Raspberry Pi Pico 开发板,RP2040 芯片。
- **Pin** (M4.1 / 概念):GPIO 引脚,Embassy 用 `Output<'a, PA5>` 形式标注引脚。
- **PIO** (M3.4 / 外设):Programmable I/O,RP2040 特有的可编程 IO 子系统,支持自定义协议。
- **PPI** (M3.3 / 外设):Programmable Peripheral Interconnect,nRF 外设间事件路由。
- **Probe-rs** (工具):Rust 嵌入式一体化烧录 / 调试 / RTT 工具,替代 OpenOCD + GDB。
- **PubSubChannel** (M2.3 / M7.4 / crate):1-to-N 发布订阅通道,每个订阅者独立消费。
- **PWM** (M4.5 / 外设):Pulse Width Modulation,脉冲宽度调制,常用于 LED 调光 / 电机控制。

## Q

- **QEMU** (M7.3 / 工具):开源硬件模拟器,`qemu-system-arm` 模拟 STM32 / nRF / RP 平台。
- **QSPI** (M3.4 / 协议):Quad SPI,4 线 SPI 协议,提升 flash 读写速度。

## R

- **RAII** (M7.4 / 概念):Resource Acquisition Is Initialization,Rust 通过 `Drop` trait 实现 RAII 守卫。
- **RAM** (概念):随机访问存储器,运行时数据存储,容量 16-512KB(典型 Cortex-M)。
- **RISC-V** (ARM):开源指令集架构,Embassy 实验性支持。
- **RTT** (M7.2 / 工具):Real-Time Transfer,SEGGER 设计的日志传输协议,通过内存区段 + 调试器读取。
- **RTIC** (M2.x / crate):Real-Time Interrupt-driven Concurrency,另一款 Rust 嵌入式并发框架,Embassy 的"兄弟"项目。
- **RTOS** (M1.3 / 概念):Real-Time Operating System,实时操作系统。Embassy 与 RTOS 的对比见 M1.3 §5。
- **Rust** (工具):系统级编程语言,Embassy 项目基础。
- **rust-toolchain.toml** (M7.1 / 工具):声明 Rust 工具链(channel / targets / components)的 TOML 文件。

## S

- **SCB** (ARM):System Control Block,Cortex-M 系统控制块,`SCB::sys_reset()` 触发软件复位。
- **Select** (M2.4 / M7.4 / 概念):`embassy_futures::select` 宏族,等待多个 future 任一 ready。
- **select_biased** (M2.4 / M7.4 / 概念):`select_biased!` 宏,按声明顺序优先检查 future。
- **Semaphore** (M2.3 / crate):信号量,资源计数。`embassy_sync::Semaphore::acquire().await`。
- **Signal** (M2.3 / M7.4 / crate):1-to-1 触发事件,`signal.signal()` / `signal.wait()`。
- **Smoltcp** (M5.1 / crate):Rust 异步 TCP/IP 协议栈,embassy-net 集成它。
- **SPI** (M4.3 / 协议):Serial Peripheral Interface,主从串行协议(SCK / MOSI / MISO / CS)。
- **STM32** (M3.2 / crate):STMicroelectronics 32 位 MCU 系列,Embassy 主力支持平台。
- **Stop Mode** (M6.3 / 概念):STM32 平台深度睡眠模式之一,保留 RAM 与部分外设。
- **Stub** (M7.3 / 概念):测试中返回固定值的占位对象,与 Mock 类似但通常更简单。
- **SysTick** (ARM):Cortex-M 系统定时器,24-bit 递减计数器,常用于 OS tick。
- **SystemView** (工具):SEGGER 的运行时分析工具,Embassy 通过 `embassy-executor` 的 `SEV` 指令协同。

## T

- **TASK** (M2.1 / 概念):`#[embassy_executor::task]` 宏标记的异步函数,Embassy 调度的基本单元。
- **Task Storage** (M2.1 / 概念):task 需要的静态内存,用 `static_cell::make_static!` 分配。
- **Test** (M7.3 / 概念):`#[test]`(host)或 `#[embassy_test::test]`(QEMU)测试函数。
- **Timer** (M2.2 / M7.4 / crate):`embassy_time::Timer::after()` 异步定时器。
- **TIM** (M4.5 / 外设):Timer 通用定时器,STM32 有 TIM1-TIM17(基本 / 通用 / 高级三类)。
- **TLM** (工具):Test Logic Module,FPGA 调试接口。
- **Trait** (M1.2 / 概念):Rust 接口定义,Embassy 用 trait 抽象外设(如 `OutputPin` / `SpiBus`)。

## U

- **UART** (M4.2 / 协议):Universal Asynchronous Receiver/Transmitter,异步串口,无时钟线。
- **USART** (M4.2 / 外设):Universal Synchronous/Asynchronous Receiver/Transmitter,STM32 的串口外设。
- **USB** (M5.2 / 协议):Universal Serial Bus,Embassy 通过 `embassy-usb` 设备栈实现。
- **USB DFU** (M6.2 / 协议):USB Device Firmware Upgrade class,标准 USB DFU 1.1 协议。

## V

- **VCP** (M4.2 / 概念):Virtual COM Port,USB CDC 类模拟的虚拟串口。
- **VTOR** (ARM):Vector Table Offset Register,设置中断向量表位置,bootloader 切换应用时用。
- **Volatile** (M4.1 / 概念):`core::ptr::read_volatile` / `write_volatile`,防止编译器优化掉硬件寄存器访问。

## W

- **WASM** (M1.3 / 概念):WebAssembly,Embassy 实验性支持的目标平台之一。
- **Watch** (M2.3 / M7.4 / crate):"最后值"语义的发布订阅,新订阅者立即拿到当前值。
- **Watchdog** (M7.4 / 概念):看门狗定时器,超时未喂狗则复位 MCU,`IWDG` 是独立看门狗。
- **WFE** (M6.3 / M7.4 / ARM):Wait For Event 指令,无事件时 MCU 进入 sleep,被中断或 `__sev()` 唤醒。
- **WFI** (M6.3 / ARM):Wait For Interrupt 指令,无中断时进入 sleep,仅中断能唤醒。
- **WWDG** (外设):Window Watchdog,窗口看门狗,必须在指定时间窗口内喂狗。
- **Waker** (M1.3 / 概念):`core::task::Waker`,future 唤醒机制,`wake()` 通知 executor 任务可运行。

## X

- **XIP** (M6.1 / 概念):Execute In Place,直接在 flash 中执行代码,无需拷贝到 RAM。

## Y

- **Yield** (M1.3 / 概念):协作式调度中的"主动让出"操作,Embassy 中由 `await` 实现。

## Z

- **Zephyr** (工具):另一款嵌入式 RTOS,与 Embassy 互为补充(参考资料见 M8.B)。

---

## M1-M7 引用位置汇总

| 术语类别 | 主要 Mx.x 引用 |
|----------|----------------|
| 异步基础 | M1.3 §2-3, M2.1 §3 |
| embassy-* crate | M1.2, M2.1-2.4 |
| 工具链 | M7.1-7.2, M7.3 §4-6 |
| 外设 | M4.1-4.5, M3.2-3.4 |
| 协议 | M4.2 (UART), M4.3 (SPI), M4.4 (I2C), M5.1 (smoltcp), M5.2 (USB), M5.3 (BLE), M5.4 (LoRa) |
| ARM / Cortex-M | M6.3 (WFE/WFI), M2.1 (NVIC), M7.2 (HardFault) |
| 设计模式 | M7.4 §2-§7 |
| 系统组件 | M6.1 (boot), M6.2 (DFU) |

---

## 检索提示

- **按"中文"找**:多数术语用英文,首次出现的中文释义在本表中
- **按"缩写"找**:如 WFE / IWDG / NVIC 等,直接查字母索引
- **按"crate 名"找**:embassy-* crate 全部以 `embassy-` 开头
- **按"模式"找**:M7.4 六大模式(状态机 / 产消 / 发布订阅 / Watchdog / 低功耗 / RAII)在 M7.4 §2-§7

本表为索引型文档,详细分析与代码示例请跳转对应 Mx.x 节号。
