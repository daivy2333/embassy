# m71-dev-setup: 开发环境配置指南规格

> Version: 0.1.0
> Last updated: 2026-06-05
> 适用硬件平台: STM32 (F4/H7/L4 等) / nRF52/53 / RP2040 / RP235x
> 关联文档: `docs/24-dev-setup.md`
> 关联里程碑: M7.1
> 关联 OpenSpec change: `add-m7-dev-practice-docs`

---

## ADDED Requirements

### Requirement: 工具链组成

文档 MUST 列明 Embassy 开发必需的 5 类核心工具(rustup / cargo / probe-rs / defmt 工具链 / cargo-generate),每类工具给出用途、安装命令、最小版本号和验证命令。

#### Scenario: 工具链完整

- **WHEN** 读者按文档"快速安装"路径操作
- **THEN** 系统具备:`rustc --version` ≥ 1.75、`probe-rs --version` ≥ 0.24、`cargo generate --version` ≥ 0.18、`defmt-print --version` ≥ 0.14
- **AND** `cargo build --target thumbv7em-none-eabihf` 能成功链接一个空白 Embassy 项目

#### Scenario: 工具链缺失检测

- **WHEN** 读者缺少 probe-rs
- **THEN** 文档 SHALL 提供"故障排查"小节给出安装命令(各平台:apt/dnf/brew/winget/scoop)
- **AND** 提供验证命令:`probe-rs list` 列出支持芯片

### Requirement: 项目脚手架

文档 MUST 详细说明 `cargo generate` + `embassy-rs/embassy-template` 的两种使用方式(交互式 + 非交互式),并展示生成项目后的目录结构。

#### Scenario: 生成 STM32 项目

- **WHEN** 读者运行 `cargo generate --git https://github.com/embassy-rs/embassy-template --name my-app`
- **THEN** 文档 SHALL 说明:交互式选择 chip / 工具链 / HAL 配置后的产出目录结构(`.cargo/config.toml`、`src/main.rs`、`src/bin/`、`memory.x` 等)
- **AND** 提供"build 验证"步骤:`cargo build --release` 成功链接出 `.elf`

#### Scenario: 非交互式生成

- **WHEN** 读者需要 CI 自动化生成项目
- **THEN** 文档 SHALL 给出 `cargo generate` 的 `--define` 参数用法(`--define mcu=STM32F411CEUx --define toolchain=stable` 等)

### Requirement: 平台烧录命令速查表

文档 MUST 给出 STM32 / nRF / RP 三平台的 `probe-rs run` / `probe-rs flash` / `probe-rs attach` 命令对比表,涵盖 `--chip` 参数和常见的 `--probe-clockspeed` 调整。

#### Scenario: STM32 烧录

- **WHEN** 读者在 STM32F411 Nucleo 板上运行 Embassy 应用
- **THEN** 文档 SHALL 给出:`probe-rs run --chip STM32F411CEUx --protocol swd target/thumbv7em-none-eabihf/release/my-app`

#### Scenario: nRF 烧录

- **WHEN** 读者在 nRF52840 DK 板上运行 Embassy 应用
- **THEN** 文档 SHALL 给出:`probe-rs run --chip nRF52840_xxAA --protocol swd target/thumbv7em-none-eabihf/release/my-app`

#### Scenario: RP 烧录

- **WHEN** 读者在 Raspberry Pi Pico 板上运行 Embassy 应用
- **THEN** 文档 SHALL 给出:`probe-rs run --chip RP2040 --protocol swd target/thumbv6m-none-eabi/release/my-app`(RP235x 类似,替换 chip 名)

### Requirement: 构建配置

文档 MUST 解释 `rust-toolchain.toml` 和 `.cargo/config.toml` 的关键字段(target / runner / rustflags / 链接脚本),并给出三平台的推荐配置。

#### Scenario: rust-toolchain.toml 字段

- **WHEN** 读者查看 `rust-toolchain.toml`
- **THEN** 文档 SHALL 解释 `channel`(stable/nightly)、`targets`(列表)、`components`(rustfmt/clippy)、`profile`(minimal/default)
- **AND** 给出 Embassy 项目最小配置示例

#### Scenario: .cargo/config.toml 链接脚本

- **WHEN** 读者配置 `.cargo/config.toml` 的 `[build]` 和 `[target.<triple>]`
- **THEN** 文档 SHALL 解释:`target = "thumbv7em-none-eabihf"`(以 STM32 为例)、`runner = "probe-rs run --chip ..."`、`rustflags = ["-C", "link-arg=-Tmemory.x"]`
- **AND** 说明 `memory.x` 的 FLASH / RAM 地址含义及如何从芯片参考手册获取

### Requirement: 实战示例

文档 MUST 提供一个最小可运行 Embassy 闪灯示例(`#[embassy_executor::main] + #[embassy_executor::task]` + GPIO toggle),覆盖从生成项目到烧录 + 串口日志验证的完整流程。

#### Scenario: 端到端闪灯

- **WHEN** 读者按文档示例从零开始
- **THEN** 文档 SHALL 给出步骤:生成项目 → 修改 `src/main.rs`(完整 30-50 行代码)→ `cargo build --release` → `probe-rs run` → 验证 LED 闪烁
- **AND** 代码示例 MUST 可编译(标注 Embassy 版本依赖 `embassy-executor = "0.5"`、`embassy-stm32 = "0.1"` 等)
- **AND** 包含 defmt 日志初始化与 `defmt::info!("Hello")` 的使用
