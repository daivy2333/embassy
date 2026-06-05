# Tasks: M7 开发实践学习文档

> 适用范围: docs/24-dev-setup.md, docs/25-debugging.md, docs/26-testing.md, docs/27-patterns.md
> 受影响 crate: 工具链(rustup/probe-rs/cargo-generate/defmt)/ embassy-template(参考)/ embassy-executor(M7.4 模式 5)/ embassy-futures(M7.4 模式 1)/ embassy-sync(M7.4 模式 2-3)/ embassy-time(M7.4 模式 4)
> 依赖: openspec/changes/add-m7-dev-practice-docs/{proposal,design,specs}/*.md
> 验收基线:
>   - 工具型 3 篇(24/25/26):~500-800 行 / ≥ 1 Mermaid / ≥ 15 源码引用 / 0 emoji
>   - 综合型 1 篇(27):~1200-1500 行 / ≥ 2 Mermaid / ≥ 25 源码引用 / 0 emoji

## 1. M7.1 开发环境配置(docs/24-dev-setup.md,工具速查型)

- [ ] 1.1 CodeGraph 探索工具链组成(`codegraph_explore` 覆盖 probe-rs cli / defmt-print / cargo-generate)
- [ ] 1.2 探索 embassy-template(`codegraph_files` 列目录,定位 `Cargo.toml` / `.cargo/config.toml` / `memory.x` 模板结构)
- [ ] 1.3 撰写 §1 主题位置(开发环境在 Embassy 学习中的角色,链接 M1-M6 已有工具链引用)
- [ ] 1.4 撰写 §2 工具链原理(rustup / cargo / probe-rs / defmt / cargo-generate 五件套)
- [ ] 1.5 撰写 §3 项目脚手架(`cargo generate` + embassy-template,交互式 + 非交互式)
- [ ] 1.6 撰写 §4 三平台烧录命令速查表(STM32 / nRF / RP × `run` / `flash` / `attach`)
- [ ] 1.7 撰写 §5 构建配置(`rust-toolchain.toml` + `.cargo/config.toml` + `memory.x`)
- [ ] 1.8 撰写 §6 实战示例(完整闪灯项目,30-50 行可编译代码)
- [ ] 1.9 撰写 §7 速查表 + 故障排查(常见 8 类问题及解决)
- [ ] 1.10 添加 Mermaid 图(§3 脚手架流程图 / §4 烧录命令决策树)
- [ ] 1.11 自审:行数 600-800 / 源码引用 ≥ 15 / Mermaid ≥ 1 / 0 emoji
- [ ] 1.12 追加 mkdocs.yml nav 加入 24-dev-setup.md
- [ ] 1.13 Commit docs/24-dev-setup.md + mkdocs.yml(不 push)

## 2. M7.2 调试与日志最佳实践(docs/25-debugging.md,工具速查型)

- [ ] 2.1 CodeGraph 探索 defmt 框架(`codegraph_explore` 覆盖 defmt / defmt-rtt / defmt-print)
- [ ] 2.2 探索 probe-rs 调试子命令(`codegraph_explore probe-rs debug attach step rtt`)
- [ ] 2.3 撰写 §1 主题位置(调试在 Embassy 开发中的角色,与 M7.1 工具链的衔接)
- [ ] 2.4 撰写 §2 defmt 原理 + 用法(`defmt::info!` / `warn!` / `error!` + 格式化占位符)
- [ ] 2.5 撰写 §3 RTT 链路(rtt-target / defmt-rtt / `probe-rs rtt` 协议)
- [ ] 2.6 撰写 §4 平台差异(STM32 / nRF / RP 三平台 RTT 配置差异)
- [ ] 2.7 撰写 §5 probe-rs debug 会话(断点 / 单步 / 变量观察 + `#[defmt::dbg]` 宏)
- [ ] 2.8 撰写 §6 故障排查清单(链接错 / 烧录失败 / RTT 无输出 / HardFault / 内存不足 / WDT 复位 / 外设错 等 8+ 项)
- [ ] 2.9 撰写 §7 实战(HardFault 栈回溯 + `cortex-m-rt` 集成)
- [ ] 2.10 添加 Mermaid 图(§3 RTT 时序图 / §5 调试会话流程图)
- [ ] 2.11 自审:行数 600-800 / 源码引用 ≥ 15 / Mermaid ≥ 1 / 0 emoji
- [ ] 2.12 追加 mkdocs.yml nav 加入 25-debugging.md
- [ ] 2.13 Commit docs/25-debugging.md + mkdocs.yml(不 push)

## 3. M7.3 测试策略与方法(docs/26-testing.md,工具速查型)

- [ ] 3.1 CodeGraph 探索 embassy-test / test-log(`codegraph_explore` 覆盖 embassy-test 框架 + test-log crate)
- [ ] 3.2 探索 QEMU 集成测试路径(`codegraph_files embassy-test/qemu` + `codegraph_explore qemu-system-arm embassy-test runner`)
- [ ] 3.3 撰写 §1 主题位置(测试在 Embassy 开发中的角色,与 M7.1/M7.2 的依赖)
- [ ] 3.4 撰写 §2 测试金字塔(单元 70% / 集成 20% / E2E 10% + 嵌入式取舍)
- [ ] 3.5 撰写 §3 host 端单元测试(`#[cfg(test)]` + `embassy_futures::block_on` 适配)
- [ ] 3.6 撰写 §4 平台差异(target 端 `#[embassy_executor::task]` mock + QEMU 支持的 target 列表)
- [ ] 3.7 撰写 §5 集成测试 + QEMU + embassy-test 框架(`#[embassy_test::main]` + `#[embassy_test::test]`)
- [ ] 3.8 撰写 §6 CI 集成(GitHub Actions 完整 `tests.yml` + Swatinem/rust-cache)
- [ ] 3.9 撰写 §7 Mock 与 Stub 模式(trait 抽象 + `mockall` + embassy-sync 测试钩子)
- [ ] 3.10 撰写 §8 实战(端到端:按钮事件 → 任务处理 → LED 响应,完整代码 + 测试 + CI)
- [ ] 3.11 添加 Mermaid 图(§2 测试金字塔 / §8 测试架构图)
- [ ] 3.12 自审:行数 600-800 / 源码引用 ≥ 15 / Mermaid ≥ 1 / 0 emoji
- [ ] 3.13 追加 mkdocs.yml nav 加入 26-testing.md
- [ ] 3.14 Commit docs/26-testing.md + mkdocs.yml(不 push)

## 4. M7.4 常见设计模式总结(docs/27-patterns.md,综合分析型,~1200-1500 行)

- [ ] 4.1 CodeGraph 综合探索 6 模式相关源码(executor::Executor::run / select_biased / Channel::send_immediate / Signal::signal / PubSubChannel::publish / Watch::send / embassy_time::Timer / on_drop)
- [ ] 4.2 撰写 §1 主题位置(模式在 Embassy 学习中的角色,作为 M2-M6 知识综合)
- [ ] 4.3 撰写 §2 状态机模式(embassy-futures::select / select_biased / select_all 决策树 + 按钮三态检测完整示例,引用 M2.4 `07-futures.md` §7)
- [ ] 4.4 撰写 §3 生产者-消费者(embassy-sync::Channel / pipe / zerocopy_channel 选型 + 中断 → 主循环示例,引用 M2.3 `06-sync.md` §4)
- [ ] 4.5 撰写 §4 发布-订阅(Signal / PubSubChannel / Watch 三种形式 + 泛型参数解释,引用 M2.3 `06-sync.md` §5-6)
- [ ] 4.6 撰写 §5 Watchdog 模式(STM32 IWDG + 软件 watchdog 心跳检测,引用 M2.2 + M6.3)
- [ ] 4.7 撰写 §6 低功耗调度(Executor::run 主循环 + WFE/DSB 双指令对 + 跨平台差异,引用 M2.1 §3 + M6.3 §5)
- [ ] 4.8 撰写 §7 RAII 资源管理(Rust Drop + embassy-futures::on_drop 异步清理)
- [ ] 4.9 撰写 §8 模式间协作(6 模式可组合 vs 互斥关系 + 决策矩阵表)
- [ ] 4.10 撰写 §9 演进与变体(从 embassy-futures 到 embassy-sync 的演进路径)
- [ ] 4.11 撰写 §10 性能与资源(各模式 RAM/任务开销对比表)
- [ ] 4.12 撰写 §11 实战综合案例(电池供电 IoT 节点,完整 200-300 行,6 模式各用 1+ 次,含功耗估算)
- [ ] 4.13 撰写 §12 总结 + 未覆盖模式清单(Strategy/Observer/Factory/Builder/Actor 等通用模式,推荐阅读资源)
- [ ] 4.14 添加 Mermaid 图(§2 状态机状态图 / §4 PubSub 拓扑 / §6 WFE 时序 / §8 决策矩阵 / §11 IoT 节点架构)
- [ ] 4.15 自审:行数 1200-1500 / 源码引用 ≥ 25 / Mermaid ≥ 2 / 0 emoji
- [ ] 4.16 追加 mkdocs.yml nav 加入 27-patterns.md
- [ ] 4.17 Commit docs/27-patterns.md + mkdocs.yml(不 push)

## 5. 收尾

- [ ] 5.1 确认 mkdocs.yml 4 篇 nav 都已加入(24/25/26/27)
- [ ] 5.2 更新 .claude/docs/SNAPSHOT.md 反映 M7 进度
- [ ] 5.3 更新 .claude/docs/tasks.md M7 4 篇状态 + 验收标准勾选
- [ ] 5.4 追加 learned/spec.md 记录 M7 系列踩坑/技巧/速查(若有新发现)
- [ ] 5.5 验证:4 篇 commit 都在本地,未 push(由用户决定 push 时机)
- [ ] 5.6 openspec validate --change add-m7-dev-practice-docs
- [ ] 5.7 /opsx:archive add-m7-dev-practice-docs(全 4 篇完成后)
