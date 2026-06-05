# m73-testing: 测试策略与方法规格

> Version: 0.1.0
> Last updated: 2026-06-05
> 适用硬件平台: STM32 (F4/H7/L4 等) / nRF52/53 / RP2040 / RP235x + QEMU 模拟
> 关联文档: `docs/26-testing.md`
> 关联里程碑: M7.3
> 关联 OpenSpec change: `add-m7-dev-practice-docs`

---

## ADDED Requirements

### Requirement: 单元测试策略

文档 MUST 解释 Embassy 项目中单元测试的两条路径(host 端原生 Rust 测试 vs target 端 `#[embassy_executor::task]` mock 测试),给出各路径的适用场景与 cargo 命令。

#### Scenario: host 端单元测试

- **WHEN** 读者测试纯逻辑代码(不涉及外设)
- **THEN** 文档 SHALL 给出:`#[cfg(test)] mod tests { #[test] fn my_logic() { assert_eq!(2+2, 4); } }` + `cargo test --lib` 运行
- **AND** 解释 Embassy 异步函数在 host 端测试的方法(`embassy_futures::block_on` 或 `tokio::test` + 适配)

#### Scenario: target 端任务测试

- **WHEN** 读者测试 `#[embassy_executor::task]` 任务的逻辑
- **THEN** 文档 SHALL 给出:将任务逻辑拆为"接受 `embassy_sync::Channel` 的纯函数",host 端测试纯函数逻辑 + QEMU 集成测试任务流程
- **AND** 给出 `static_cell::make_static!` 模式初始化 task storage 的 mock 写法

### Requirement: 集成测试与 QEMU 模拟

文档 MUST 介绍 QEMU 在 Embassy 集成测试中的作用(`qemu-system-arm` 模拟 STM32/RP 目标)、`embassy-test` 框架的用法、test runner 协议。

#### Scenario: QEMU 集成测试

- **WHEN** 读者需要在 CI 中测试 Embassy 应用而无需物理硬件
- **THEN** 文档 SHALL 给出:安装 `qemu-system-arm` → `cargo test --target thumbv7em-none-eabihf` → test runner 通过 RTT 收集 defmt 日志 + exit code 报告 PASS/FAIL
- **AND** 列出支持 QEMU 模拟的 Embassy 目标(STM32F4 / STM32H7 / RP2040)

#### Scenario: embassy-test 框架

- **WHEN** 读者用 `embassy-test` 框架
- **THEN** 文档 SHALL 解释:`#[embassy_test::main]` 宏 + `#[embassy_test::test]` 宏的语义(在 QEMU 中顺序执行)+ `test-log` crate 的日志集成
- **AND** 给出 5-10 个 `#[embassy_test::test]` 函数的完整示例

### Requirement: test-log 与日志集成

文档 MUST 介绍 `test-log` / `defmt-test` 与标准测试框架的集成方式(自动捕获 `defmt::info!` 并在测试失败时显示上下文)。

#### Scenario: 失败时显示日志

- **WHEN** 读者某个 embassy-test 失败
- **THEN** 文档 SHALL 说明:`test-log = { version = "0.2", default-features = false, features = ["defmt"] }` 配置 + `RUST_LOG=debug` 启用详细输出
- **AND** 给出 `cargo test -- --nocapture` 的用法

### Requirement: CI 集成

文档 MUST 提供 GitHub Actions 配置文件示例(`tests.yml`),覆盖:多平台 build 检查、QEMU 集成测试、rustfmt / clippy 静态分析、文档构建(mkdocs)。

#### Scenario: GitHub Actions 完整配置

- **WHEN** 读者设置 Embassy 项目的 GitHub Actions CI
- **THEN** 文档 SHALL 给出 `.github/workflows/tests.yml` 完整配置:actions/checkout → dtolnay/rust-toolchain(stable + thumbv 目标) → Swatinem/rust-cache → cargo fmt --check + cargo clippy --target thumbv7em-none-eabihf -- -D warnings + cargo test --target thumbv7em-none-eabihf(qemu 模拟)
- **AND** 解释每个 step 的作用与失败时定位

#### Scenario: cache 与并发

- **WHEN** 读者优化 CI 时间
- **THEN** 文档 SHALL 给出:`Swatinem/rust-cache` 缓存 `~/.cargo/registry` 和 `target/` 的配置 + `concurrency: group: ${{ github.ref }}, cancel-in-progress: true` 取消旧运行

### Requirement: 测试金字塔与覆盖率

文档 MUST 解释嵌入式项目中的测试金字塔(单元测试 70% / 集成测试 20% / 端到端硬件测试 10%)与各层测试的取舍。

#### Scenario: 覆盖率工具

- **WHEN** 读者评估测试覆盖率
- **THEN** 文档 SHALL 给出:`cargo llvm-cov` 用于 host 端测试覆盖率(不支持 cross-compile target) + `tarpaulin` 仅支持 host + STM32 覆盖率需借助 `OpenOCD` + `gcov` 工具链(超出本文范围,标注引用)
- **AND** 明确"嵌入式覆盖率的目标值因实时性约束而灵活,不追求 100%"

### Requirement: Mock 与 Stub 模式

文档 MUST 介绍 Embassy 测试中常用 mock 模式:用 trait 抽象外设、用 `mockall` 或手写 mock struct、用 `embassy-sync` 同步原语作为测试钩子。

#### Scenario: 抽象外设 trait

- **WHEN** 读者需要测试使用 GPIO 的代码
- **THEN** 文档 SHALL 给出:定义 `trait GpioToggle { fn toggle(&mut self); }` + 实际实现 `RealGpio` + 测试用 `MockGpio { toggled: Cell<u32> }`
- **AND** 解释为何在 Embassy 中 mock 比传统 RTOS 简单(无共享全局状态,所有依赖通过泛型注入)

#### Scenario: embassy-sync 测试钩子

- **WHEN** 读者需要测试任务间通信
- **THEN** 文档 SHALL 给出:在测试代码中创建 `Channel::new()` 共享给两个任务,用 `block_on` 执行 `select` 触发场景
- **AND** 给出"生产者-消费者"完整测试代码

### Requirement: 实战示例

文档 MUST 提供一个端到端测试示例:实现"按钮事件 → 任务处理 → LED 响应"的完整功能,配套 host 单元测试 + QEMU 集成测试 + GitHub Actions 配置。

#### Scenario: 端到端测试

- **WHEN** 读者按文档示例实现"按钮事件 → LED 响应"功能
- **THEN** 文档 SHALL 给出:完整 `src/main.rs`(主程序)+ `tests/integration.rs`(QEMU 集成测试)+ `tests/unit.rs`(host 单元测试)+ `.github/workflows/tests.yml`(CI)
- **AND** 所有代码 MUST 可编译(标注 embassy 版本)
- **AND** 给出预期 CI 输出示例(全 PASS 截图描述)
