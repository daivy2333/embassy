## Why

本项目为 Embassy 嵌入式异步框架的学习研究项目,目标是系统性分析其架构与实现细节。已完成的 M1-M6 六个里程碑(23 篇文档)覆盖了基础架构、核心组件、HAL 层、外设驱动、网络通信栈和系统组件。本次 M7 开发实践(4 篇文档)是收官里程碑,聚焦"动手写代码"维度:开发环境、调试方法、测试策略和设计模式,补完从"读懂源码"到"独立开发 Embassy 应用"的知识闭环。

## What Changes

- 新增 `docs/24-dev-setup.md`(M7.1 开发环境配置指南,目标 ~600-800 行,工具链速查型:rustup / probe-rs / cargo-binutils / defmt-print / cargo-generate / embassy-template)
- 新增 `docs/25-debugging.md`(M7.2 调试与日志最佳实践,目标 ~600-800 行,工具速查型:defmt 日志框架 / RTT / panic handler / probe-rs attach / 常见故障排查)
- 新增 `docs/26-testing.md`(M7.3 测试策略与方法,目标 ~600-800 行,工具速查型:单元测试(#[embassy_executor::task] mock)/ 集成测试(QEMU/simulator)/ test-log / CI 集成)
- 新增 `docs/27-patterns.md`(M7.4 常见设计模式总结,目标 ~1200-1500 行,综合分析型:状态机 / 生产者-消费者 / 发布-订阅 / Watchdog / 低功耗调度 / RAII 资源管理 6 大模式)
- 不做的事:不修改 Embassy 任何源码(本项目是学习项目,不做开发);不改 mkdocs.yml 之外的其他配置文件;不深入特定芯片的 datasheet 章节;不做附录 A/B/C(术语表 / 参考资料 / 示例索引)— 留作后续可选增量
- 模板:复用 M1-M6 章节骨架(项目背景 → 核心概念 → 关键源码 → 实战示例 → 对比/总结),按 M7 主题精简或深化;Mermaid 图 + 源码引用;0 emoji

## Capabilities

### New Capabilities

- `m71-dev-setup`: docs/24-dev-setup.md 的内容契约,定义开发环境配置指南的范围(工具链安装/版本、cargo generate 脚手架、embassy-template 模板、rust-toolchain.toml 配置、.cargo/config.toml 链接脚本、probe-rs 烧录、构建产物)
- `m72-debugging`: docs/25-debugging.md 的内容契约,定义调试方法与日志最佳实践的范围(defmt 框架 / RTT 链路 / panic_handler / probe-rs attach & step / defmt-print 解析 / 故障排查清单)
- `m73-testing`: docs/26-testing.md 的内容契约,定义测试策略的范围(单元测试结构 / 集成测试 / QEMU 模拟器 / test-log / CI(github actions)/ 测试覆盖率 / mock 与 stub)
- `m74-patterns`: docs/27-patterns.md 的内容契约,定义设计模式汇编的范围(状态机(embassy-futures::select_biased)/ 生产者-消费者(embassy-sync::Channel)/ 发布-订阅(Signal)/ Watchdog 模式 / 低功耗调度(Executor::run + WFE)/ RAII 资源管理(Drop + on_drop))

### Modified Capabilities

(无 — 不修改现有 spec)

## Impact

- **代码**:零影响(本项目不做开发)
- **API**:零影响
- **依赖**:零影响
- **文档**:新增 4 篇 Markdown 文档到 `docs/`(累计 ~3000-4000 行,深度灵活)
- **OpenSpec**:`openspec/specs/learned/` 增量条目(M7 系列踩坑/技巧/速查待 M7 各篇完成后追加)
- **GitHub Pages**:新增后需更新 `mkdocs.yml` nav 加入 24-27 项
- **回滚方案**:删除 `docs/24-*.md ~ 27-*.md` + 撤销 mkdocs.yml nav 改动。每个文档独立可回滚。

## 跨平台兼容性

M7.1 dev-setup 跨平台中立(工具链配置/项目脚手架/烧录命令以 STM32 / nRF / RP 三平台分别给出)。M7.2 debugging 同样三平台分别给出 RTT 配置、probe-rs chip 名、attach 命令差异。M7.3 testing 聚焦 QEMU(支持 stm32/rp 等),不依赖物理硬件,跨平台测试能力一致。M7.4 patterns 是平台中立的设计模式抽象,不绑定具体芯片。

## Phase 3 实施顺序

按 M1-M6 经验"一篇一篇来":
1. M7.1 开发环境配置(本次先做,铺垫后续调试/测试)
2. M7.2 调试与日志最佳实践(依赖 M7.1 的工具链)
3. M7.3 测试策略与方法(依赖 M7.1 的工具链)
4. M7.4 设计模式总结(综合性,放最后)

每篇完成后做一次 commit(不自动 push,等用户决定时机)。
