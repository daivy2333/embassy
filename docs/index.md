# Embassy 学习笔记

> 系统性源码分析 + 开发实战 · 中文技术文档 · 30 篇文档覆盖 M1-M7 + 附录 · 基于 [embassy-rs/embassy](https://github.com/embassy-rs/embassy) fork 而来

---

## 这是什么

这是一份 **学习研究型笔记**,目标是把 [Embassy](https://embassy.dev/)(Rust 嵌入式异步框架)的架构设计、核心组件、开发实践**讲清楚**。

- **不是教程** — 不教你怎么用 Embassy 写程序
- **不是 API 文档** — 不替代 [docs.embassy.dev](https://docs.embassy.dev/)
- **是源码分析 + 实战笔记** — 关注"为什么这样设计"、"内部怎么工作"、"怎么动手试"

适合 **想理解 Embassy 内部机制** 的读者:已经会用 Rust + async/await,想知道执行器、时间驱动、同步原语在嵌入式无堆环境下的实现取舍,以及如何用 probe-rs / defmt / QEMU 工具链开发与测试 embassy 应用。

---

## 项目状态

**当前进度: 30/30 = 100% 收官**(M1-M7 主文档 27 篇 + 附录 3 篇,合计约 25700 行)

| 维度 | 详情 |
|------|------|
| 主里程碑 | M1 基础架构 · M2 核心组件 · M3 HAL 层 · M4 外设驱动 · M5 网络通信 · M6 系统组件 · M7 开发实践 |
| 附录 | 术语表(A)· 参考资料(B)· 示例索引(C) |
| 支持平台 | STM32(F4 / H7 / L4 等)· nRF52 / 53 · RP2040 / RP235x |
| OpenSpec 归档 | 4 个 change 全部归档(`add-m5-network-comms-docs` / `add-m6-system-components-docs` / `add-m7-dev-practice-docs` / `add-m8-appendices`) |
| GitHub Pages | 已部署,见 [daivy2333.github.io/embassy](https://daivy2333.github.io/embassy/) |

详细进度与状态见 [.claude/docs/SNAPSHOT.md](https://github.com/daivy2333/embassy/blob/main/.claude/docs/SNAPSHOT.md) 与 [.claude/docs/tasks.md](https://github.com/daivy2333/embassy/blob/main/.claude/docs/tasks.md)。

---

## 阅读路径

按里程碑分组,**建议顺序阅读**。主文档每篇 500-1500 行,带源码引用、对比表、Mermaid 流程图;附录 3 篇是"参考体系"维度入口。

### M0 项目入口

| 篇章 | 看什么 |
|---|---|
| [项目概览 embassy.md](embassy.md) | Embassy 是什么、解决什么问题、与 RTOS 的本质差异 |

### M1 基础架构(已完成 · 3 篇)

| 篇章 | 看什么 |
|---|---|
| [01 项目结构](01-overview.md) | 40+ crate 工作空间布局,各 crate 的角色定位 |
| [02 crate 架构](02-architecture.md) | 依赖拓扑图、分层模型、跨平台策略 |
| [03 异步基础](03-async-fundamentals.md) | Rust async/await 在嵌入式无堆场景的落地,与 stack-RTOS 的对比 |

### M2 核心组件(已完成 · 4 篇)

| 篇章 | 看什么 |
|---|---|
| [04 执行器 embassy-executor](04-executor.md) | 主循环、6 状态转移、ThreadMode 与 InterruptExecutor |
| [05 时间管理 embassy-time](05-time.md) | integrated vs generic 时间驱动、12 步 wake 链 |
| [06 同步原语 embassy-sync](06-sync.md) | Channel / Signal / Mutex / Pipe / Watch / PubSub 的实现与决策树 |
| [07 异步工具 embassy-futures](07-futures.md) | select! / join! / yield_now / poll_once 的零开销实现 |

### M3 HAL 层架构(已完成 · 4 篇)

| 篇章 | 看什么 |
|---|---|
| [08 HAL 通论](08-hal-architecture.md) | embedded-hal 4 大 trait 体系、PAC/HAL/应用层 7 层架构、ADR-004 术语与模板 |
| [09 STM32 平台](09-stm32.md) | embassy-stm32 11 节、3 大 timer 类别、interrupt 抽象、EXTI 实战 |
| [10 nRF 平台](10-nrf.md) | embassy-nrf 11 节、EasyDMA + PPI + DPPI、softdevice 集成 |
| [11 RP 平台](11-rp.md) | embassy-rp 11 节、PIO 可编程 IO、Multicore 实战 |

### M4 外设驱动(已完成 · 5 篇)

| 篇章 | 看什么 |
|---|---|
| [12 GPIO 与中断](12-gpio.md) | 异步 GPIO、EXTI/IRQ 链、165 处异步命中 |
| [13 UART 串口](13-uart.md) | 异步 / 阻塞 / 缓冲三种模式、DMA 实战 |
| [14 SPI 总线](14-spi.md) | SPI 全双工/半双工、DMA、TX-only/RX-only 模式 |
| [15 I2C 总线](15-i2c.md) | I2C master/slave、地址扫描、DMA、错误恢复 |
| [16 定时器与 PWM](16-timer.md) | HRTIM / TIM 输入捕获 / PWM 输出 / 舵机控制 |

### M5 网络与通信(已完成 · 4 篇)

| 篇章 | 看什么 |
|---|---|
| [17 embassy-net 网络栈](17-net.md) | smoltcp 集成、DriverAdapter、TCP/UDP socket |
| [18 embassy-usb 设备栈](18-usb.md) | USB 描述符、端点、HID/CDC/DFU 类 |
| [19 BLE 协议](19-ble.md) | 平台中立 BLE 7 层模型、5 平台 Controller 对比 |
| [20 LoRa 远程通信](20-lora.md) | SX127x 驱动、LoRaWAN 入网说明 |

### M6 系统组件(已完成 · 3 篇)

| 篇章 | 看什么 |
|---|---|
| [21 embassy-boot 启动引导](21-boot.md) | 三分区设计、State 状态机、Magic 字节协议、swap/revert 算法 |
| [22 FirmwareUpdater 与 DFU](22-dfu.md) | write_firmware 流程、签名验证、跨平台 OTA 时序 |
| [23 低功耗设计模式](23-low-power.md) | Executor::run 主循环、WFE/DSB 指令对、TASKS_PENDING 协议 |

### M7 开发实践(已完成 · 4 篇)

| 篇章 | 看什么 |
|---|---|
| [24 开发环境配置](24-dev-setup.md) | rustup / probe-rs / defmt / cargo-generate 工具链,三平台烧录命令速查 |
| [25 调试与日志](25-debugging.md) | defmt 框架 + RTT 链路 + probe-rs debug,HardFault 栈回溯实战 |
| [26 测试策略](26-testing.md) | host 单元测试 + QEMU 集成 + GitHub Actions 完整配置 |
| [27 设计模式](27-patterns.md) | 6 大 Embassy 异步模式汇编,电池供电 IoT 节点综合实战 |

### 附录(已完成 · 3 篇)

| 篇章 | 看什么 |
|---|---|
| [附录 A 术语表](appendix-a-glossary.md) | 121 术语 A-Z 索引,覆盖 M1-M7 关键概念,每条带 Mx.x 节号引用 |
| [附录 B 参考资料](appendix-b-references.md) | 38 URL 分 5 大类(Embassy 官方 / ARM / Rust / 工具链 / 社区),含 7 周学习路径 |
| [附录 C 示例索引](appendix-c-examples.md) | embassy `examples/` 108 个 bin 文件按 16 主题分类,含 10 步入门路径 |

---

## 文档统计

| 维度 | 数值 |
|------|------|
| 主文档(M1-M7) | 27 篇 |
| 附录(A/B/C) | 3 篇 |
| 合计 | 30 篇 |
| 总行数 | 约 25700 行 |
| 总 Mermaid 图 | 60+ 张 |
| 总源码引用 | 600+ 处 |
| 跨文档交叉引用 | 100+ 处 |
| 0 emoji | 全项目遵守 |

---

## 快速跳转

| 我想... | 看哪 |
|---------|------|
| 理解 Embassy 的本质 | [embassy.md](embassy.md) |
| 跑通第一个 embassy 项目 | [24 开发环境配置](24-dev-setup.md) + [附录 C 入门路径](appendix-c-examples.md#入门路径10-步) |
| 学习 embassy 的执行器怎么工作 | [04 执行器](04-executor.md) |
| 排查 HardFault | [25 调试与日志 §7](25-debugging.md#7-实战hardfault-栈回溯--panic_handler-集成) |
| 写 OTA 升级 | [22 FirmwareUpdater](22-dfu.md) + [21 embassy-boot](21-boot.md) |
| 找某个术语解释 | [附录 A 术语表](appendix-a-glossary.md) |
| 找某类 embassy 示例 | [附录 C 示例索引](appendix-c-examples.md) |
| 学习 embassy 异步编程模式 | [27 设计模式](27-patterns.md) |

---

## 反馈与贡献

发现错误、有疑问、想补充?欢迎:

- 在 [GitHub Issues](https://github.com/daivy2333/embassy/issues) 提 issue
- 直接 PR 改进文档
- 参考 [附录 B 参考资料](appendix-b-references.md) 的 Embassy Matrix 频道

---

## 版权

本笔记基于 [embassy-rs/embassy](https://github.com/embassy-rs/embassy)(Apache-2.0 OR MIT)的源码分析撰写。原项目所有权归 Embassy 项目所有,本笔记仅为学习用途。
