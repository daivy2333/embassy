# 附录 B - Embassy 学习项目参考资料汇总

> 版本: v0.1.0
> 最后更新: 2026-06-05
> 参考日期: 2026-06
> 适用项目版本: Embassy `0.5+`

---

## 简介

本附录汇总 Embassy 学习项目 M1-M7 27 篇文档引用的所有外部权威资源,按 5 大类组织。每条资源标注完整 URL、访问日期(2026-06)、embassy 版本(0.5+),并给出 M1-M7 引用示例,帮助读者快速定位"在哪一篇文档用到了这个资源"。

文档末尾提供"按 M1-M7 顺序的 7 周学习路径建议",适合 embassy 新手循序渐进。

---

## 5 大类导航

| 类别 | 主要内容 | 跳转 |
|------|----------|------|
| Embassy 官方 | embassy 仓库 / book / examples / awesome | [§1](#1-embassy-官方) |
| ARM & Cortex-M | ARM ARM / Cortex-M 编程手册 / CMSIS | [§2](#2-arm--cortex-m-规范) |
| Rust 资源 | TRPL / Async Book / Embedded Rust Book / std | [§3](#3-rust-资源) |
| 工具链 | probe-rs / defmt / QEMU / cargo-generate / rustup | [§4](#4-工具链) |
| 学习社区 | TWiR / Embedded WG / Matrix / Reddit | [§5](#5-学习社区) |
| 学习路径 | 7 周入门路径 | [§6](#6-学习路径建议) |

---

## 1. Embassy 官方

### 1.1 embassy-rs/embassy 主仓库

- **URL**: `https://github.com/embassy-rs/embassy`
- **访问日期**: 2026-06
- **版本**: 0.5+
- **M1-M7 引用**: M1.1 §1(项目定位)+ M1.2 §2(crate 列表)+ M2.1-M2.4(全部 executor / time / sync / futures 源码)+ M3.1-3.4(三平台 HAL)+ M4.1-4.5(外设驱动)+ M5.1-5.4(网络 / USB / BLE / LoRa)+ M6.1-6.3(boot / DFU / 低功耗)
- **用途**: Embassy 的"参考手册"级权威,所有 M1-M7 文档的源码引用都来自此仓库

### 1.2 Embassy 官方文档(embassy-book)

- **URL**: `https://embassy.dev/book/`
- **访问日期**: 2026-06
- **版本**: 0.5+
- **M1-M7 引用**: M7.1 §1(开发环境基础)+ M7.2 §2(defmt 入门)+ M7.3 §5(embassy-test 入门)
- **用途**: 入门教程,涵盖工具链、embassy-executor / embassy-time / embassy-sync 的基本用法

### 1.3 embassy-template 模板

- **URL**: `https://github.com/embassy-rs/embassy-template`
- **访问日期**: 2026-06
- **版本**: 0.5+
- **M1-M7 引用**: M7.1 §3(项目脚手架)
- **用途**: `cargo generate` 用的官方模板,生成的 `Cargo.toml` / `.cargo/config.toml` / `memory.x` 模板

### 1.4 embassy-rs/embassy 仓库 examples/ 目录

- **路径**: `https://github.com/embassy-rs/embassy/tree/main/examples`
- **访问日期**: 2026-06
- **M1-M7 引用**: M7.1 §1.3(三平台支持现状)+ M7.3 §8(端到端测试案例)+ M8.C(本附录 C,完整索引)
- **用途**: 30+ 平台 × 多种外设的可运行示例,M8.C 详述

### 1.5 embassy-rs/awesome-embassy

- **URL**: `https://github.com/embassy-rs/awesome-embassy`
- **访问日期**: 2026-06
- **M1-M7 引用**: 全文(参考资料)
- **用途**: 第三方库 / 教程 / 工具的 curated 列表

---

## 2. ARM & Cortex-M 规范

### 2.1 ARM Architecture Reference Manual (ARM ARM)

- **URL(ARMv7-M)**: `https://developer.arm.com/documentation/ddi0403/latest/`
- **URL(ARMv8-M)**: `https://developer.arm.com/documentation/ddi0553/latest/`
- **访问日期**: 2026-06
- **M1-M7 引用**: M6.3 §6(WFE / WFI / SEV 指令)+ M2.1 §5(中断控制器)+ M7.2 §5(HardFault / CFSR / HFSR)
- **用途**: ARM 架构权威,涉及异常模型、指令集、内存模型

### 2.2 Cortex-M 编程手册

- **URL(Generic)**: `https://developer.arm.com/documentation/dui0553/latest/`
- **URL(M7 Devices)**: `https://developer.arm.com/documentation/dui0646/latest/`
- **访问日期**: 2026-06
- **M1-M7 引用**: M6.3 §3(主循环 WFE 模式)+ M7.2 §5(异常处理)
- **用途**: Cortex-M 处理器级别的程序员手册,比 ARM ARM 更友好

### 2.3 CMSIS 标准

- **URL**: `https://developer.arm.com/tools-and-software/embedded/cmsis`
- **访问日期**: 2026-06
- **M1-M7 引用**: M3.1 §3(embedded-hal 与 CMSIS 的关系)
- **用途**: ARM 的"嵌入式 C 库"标准,Embassy 不直接用,但理解背景有帮助

### 2.4 ARM Cortex-M 异常清单

- **URL**: `https://developer.arm.com/documentation/dui0553/latest/the-cortex-m4-processor/fault-handling/fault-types`
- **M1-M7 引用**: M7.2 §5(异常与 HardFault)
- **用途**: 各种 Fault 的根因分类表

---

## 3. Rust 资源

### 3.1 The Rust Programming Language(TRPL, "the Book")

- **URL**: `https://doc.rust-lang.org/book/`
- **访问日期**: 2026-06
- **M1-M7 引用**: M1.3 §2(async/await 入门)
- **用途**: Rust 入门必读,前 10 章覆盖所有权 / 借用 / trait / 泛型,Ch 17 覆盖 async

### 3.2 The Rust Async Book

- **URL**: `https://rust-lang.github.io/async-book/`
- **访问日期**: 2026-06
- **M1-M7 引用**: M1.3 §3(异步机制详解)+ M2.1 §2(executor 概念)
- **用途**: Rust 异步机制权威,从 `Future` trait 到 `Waker` 深入讲解

### 3.3 The Embedded Rust Book

- **URL**: `https://docs.rust-embedded.org/book/`
- **访问日期**: 2026-06
- **M1-M7 引用**: M1.1 §1(嵌入式 Rust 生态)+ M3.1 §2(embedded-hal 标准)
- **用途**: 嵌入式 Rust 入门,涵盖工具链 / QEMU 模拟 / 常用 crate

### 3.4 Rust Standard Library 文档

- **URL**: `https://doc.rust-lang.org/std/`
- **访问日期**: 2026-06
- **M1-M7 引用**: 全文(API 参考)
- **用途**: `core::ptr` / `core::task` / `core::sync::atomic` 等核心 API

### 3.5 Embassy Async 教程(Tokio 对比)

- **URL**: `https://tokio.rs/tokio/tutorial/async`
- **M1-M7 引用**: M1.3 §5(Embassy vs Tokio)
- **用途**: Tokio 的 async 教程,与 Embassy 对比理解

### 3.6 The Cargo Book

- **URL**: `https://doc.rust-lang.org/cargo/`
- **M1-M7 引用**: M7.1 §5(`Cargo.toml` 字段解释)
- **用途**: Cargo 工具参考

---

## 4. 工具链

### 4.1 probe-rs

- **URL**: `https://probe.rs/`
- **GitHub**: `https://github.com/probe-rs/probe-rs`
- **访问日期**: 2026-06
- **M1-M7 引用**: M7.1 §2.3(工具链组成)+ M7.2 §3(RTT 链路)+ M7.2 §5(probe-rs debug)+ M7.3 §6(CI 集成)
- **用途**: Rust 嵌入式一体化烧录 / 调试 / RTT 工具,Embassy 项目的"标配"

### 4.2 defmt

- **URL**: `https://defmt.ferrous-systems.com/`
- **GitHub**: `https://github.com/ferrous-systems/defmt`
- **访问日期**: 2026-06
- **M1-M7 引用**: M7.2 §2(defmt 框架)+ M7.2 §3(RTT 链路)+ M7.3 §5(test-log 集成)
- **用途**: 零成本嵌入式日志框架

### 4.3 QEMU 文档

- **URL**: `https://www.qemu.org/docs/master/`
- **访问日期**: 2026-06
- **M1-M7 引用**: M7.3 §4(QEMU 模拟支持的目标)+ M7.3 §5(QEMU 集成测试)
- **用途**: 开源硬件模拟器,Embassy 测试的核心

### 4.4 cargo-generate

- **URL**: `https://github.com/cargo-generate/cargo-generate`
- **访问日期**: 2026-06
- **M1-M7 引用**: M7.1 §2.5(cargo-generate 安装)+ M7.1 §3(项目脚手架)
- **用途**: 从 Git 模板生成项目

### 4.5 rustup

- **URL**: `https://rustup.rs/`
- **访问日期**: 2026-06
- **M1-M7 引用**: M7.1 §2.1(rustup 安装 + 目标)
- **用途**: Rust 工具链管理器,跨平台标准安装方式

### 4.6 GitHub Actions 文档

- **URL**: `https://docs.github.com/en/actions`
- **M1-M7 引用**: M7.3 §6(CI 集成完整配置)
- **用途**: CI 配置参考,Embassy CI 常用 Swatinem/rust-cache + dtolnay/rust-toolchain

### 4.7 常用 GitHub Actions

- **Swatinem/rust-cache**: `https://github.com/Swatinem/rust-cache`(Rust 编译缓存)
- **dtolnay/rust-toolchain**: `https://github.com/dtolnay/rust-toolchain`(Rust 工具链安装)
- **actions/checkout**: `https://github.com/actions/checkout`

---

## 5. 学习社区

### 5.1 This Week in Rust(TWiR)

- **URL**: `https://this-week-in-rust.org/`
- **访问日期**: 2026-06
- **M1-M7 引用**: 全文(更新资讯)
- **用途**: Rust 每周更新,Embassy 新版本动态通常在此首发

### 5.2 Embedded Working Group

- **URL**: `https://github.com/rust-embedded/wg`
- **访问日期**: 2026-06
- **M1-M7 引用**: M1.1 §1(嵌入式 Rust 社区)
- **用途**: Rust 嵌入式工作组,标准与生态的协调者

### 5.3 Embassy Matrix 频道

- **URL**: `https://matrix.to/#/#embassy-rs:matrix.org`
- **M1-M7 引用**: 全文(社区)
- **用途**: Embassy 官方实时聊天,问题求助与新功能讨论

### 5.4 Reddit r/rust 与 r/embedded

- **URL**: `https://www.reddit.com/r/rust/`
- **URL**: `https://www.reddit.com/r/embedded/`
- **M1-M7 引用**: 全文(社区)
- **用途**: 综合性 Rust 与嵌入式讨论

### 5.5 Rust 语言中文社区

- **URL**: `https://rustcc.cn/`
- **M1-M7 引用**: 全文(中文资源)
- **用途**: Rust 中文论坛

### 5.6 Embassy 中文教程

- **搜索建议**: 在 `https://github.com/embassy-rs/embassy` 的 issues / discussions 中搜索 "中文" 关键词
- **M1-M7 引用**: 本项目本身即贡献之一

---

## 6. 学习路径建议

以下 7 周路径适合 embassy 新手,基于 M1-M7 27 篇文档的"由浅入深"顺序,每阶段标注预计耗时与产出。

### Week 1:Rust 基础 + 嵌入式 Rust 入门

- **目标**:掌握 Rust 语法 + 嵌入式 Rust 工具链基础
- **任务**:
  - 读完 TRPL Ch 1-10(`https://doc.rust-lang.org/book/`)
  - 读完 Embedded Rust Book Ch 1-4(`https://docs.rust-embedded.org/book/`)
  - 安装 rustup + probe-rs(参见 M7.1 §2)
- **产出**:`rustc --version` / `probe-rs --version` 验证
- **预计耗时**: 8-12 小时

### Week 2:Embassy 入门 + 第一盏灯

- **目标**:跑通 embassy-template 闪灯项目
- **任务**:
  - 读 embassy-book 前 3 章(`https://embassy.dev/book/`)
  - 跑通 `examples/rp/src/bin/blinky.rs`(参见 M8.C 入门路径第 1 步)
  - 读 M1.1 + M1.3(项目结构 + async 基础)
- **产出**:开发板 LED 闪烁
- **预计耗时**: 6-8 小时

### Week 3-4:M1-M2(架构 + 核心组件)

- **目标**:理解 Embassy 的执行器 / 时间 / 同步 / future
- **任务**:
  - 读 M1.1(项目结构)+ M1.2(crate 架构)+ M1.3(async 基础)
  - 读 M2.1(执行器)+ M2.2(时间)+ M2.3(同步)+ M2.4(future)
  - 跑 `examples/nrf52840/src/bin/multiprio.rs`(多优先级 select_biased)
- **产出**:能解释"任务如何被调度"
- **预计耗时**: 20-25 小时

### Week 5-6:M3-M4(HAL + 外设)

- **目标**:理解三平台 HAL 架构 + 常用外设异步驱动
- **任务**:
  - 读 M3.1(HAL 通论)+ M3.2-3.4(三平台 HAL)
  - 读 M4.1(GPIO)+ M4.2(UART)+ M4.3(SPI)+ M4.4(I2C)+ M4.5(定时器)
  - 跑 `examples/stm32f4/src/bin/uart.rs` + `examples/rp/src/bin/i2c.rs`
- **产出**:能编写 GPIO + UART 异步应用
- **预计耗时**: 25-30 小时

### Week 7:M5-M6(网络 / 系统)

- **目标**:理解网络栈 / USB / BLE / LoRa / 启动引导 / 低功耗
- **任务**:
  - 读 M5.1(embassy-net)+ M5.2(embassy-usb)+ M5.3(BLE)+ M5.4(LoRa)
  - 读 M6.1(embassy-boot)+ M6.2(DFU)+ M6.3(低功耗)
- **产出**:能描述 embassy 全栈架构
- **预计耗时**: 18-22 小时

### 后续:M7(开发实践)+ M8(附录,本项目产出)

- **目标**:掌握实战工具与查阅参考体系
- **任务**:
  - 读 M7.1(开发环境)+ M7.2(调试)+ M7.3(测试)+ M7.4(模式)
  - 查阅 M8.A 术语表 + M8.B 参考资料 + M8.C 示例索引(本项目产出)
- **产出**:能独立开发 embassy 应用 + 查阅体系完整
- **预计耗时**: 12-15 小时

**总学习路径**: 约 90-120 小时,适合有 Rust 基础者系统学习 embassy。

---

## 资源维护说明

- **链接稳定性**:本表 URL 在 2026-06 测试可达;半年后可能有变更,优先通过 embassy-book 入口访问
- **版本漂移**:embassy 处于活跃开发,API 可能在 v1.0 前调整,建议固定到 0.5+ 某个具体次版本
- **本地镜像**:embassy 主仓库可 `git clone` 到本地,`cargo doc --open` 在 `embassy-executor` / `embassy-sync` 等目录下生成离线 API 文档
- **社区贡献**:发现本表遗漏的优质资源,欢迎 PR 补充

---

## M1-M7 引用回标汇总

| 资源类别 | 关键引用节号 |
|----------|--------------|
| embassy 主仓库 | 全文(基础) |
| embassy-book | M7.1 §1, M7.2 §1, M7.3 §1 |
| embassy-template | M7.1 §3 |
| ARM ARM | M2.1 §5, M6.3 §6, M7.2 §5 |
| Embedded Rust Book | M1.1 §1, M3.1 §2 |
| Rust Async Book | M1.3 §3, M2.1 §2 |
| probe-rs | M7.1 §2.3, M7.2 全篇, M7.3 §6 |
| defmt | M7.2 §2-§3, M7.3 §5 |
| QEMU | M7.3 §4-§5 |
| cargo-generate | M7.1 §2.5, §3 |
| TWiR / Matrix | 社区(全文) |

本附录是"参考体系"维度的入口,与 M8.A 术语表 + M8.C 示例索引共同构成 Embassy 学习项目的"参考三角"。
