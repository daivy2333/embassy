# Embassy 学习笔记

> 系统性源码分析 · 中文技术文档 · 基于 [embassy-rs/embassy](https://github.com/embassy-rs/embassy) fork 而来

---

## 📖 这是什么

这是一份 **学习研究型笔记**,目标是把 [Embassy](https://embassy.dev/)(Rust 嵌入式异步框架)的架构设计与核心组件**讲清楚**。

- **不是教程** — 不教你怎么用 Embassy 写程序
- **不是 API 文档** — 不替代 [docs.embassy.dev](https://docs.embassy.dev/)
- **是源码分析笔记** — 关注"为什么这样设计"、"内部怎么工作"

适合 **想理解 Embassy 内部机制** 的读者:已经会用 Rust + async/await,想知道执行器、时间驱动、同步原语在嵌入式无堆环境下的实现取舍。

---

## 🗺️ 阅读路径

按里程碑分组,**建议顺序阅读**。每篇 500-700 行,带源码引用、对比表、Mermaid 流程图。

### M0 项目入口

| 篇章 | 看什么 |
|---|---|
| [项目概览 embassy.md](embassy.md) | Embassy 是什么、解决什么问题、与 RTOS 的本质差异 |

### M1 基础架构 ✅

| 篇章 | 看什么 |
|---|---|
| [01 项目结构](01-overview.md) | 40+ crate 工作空间布局,各 crate 的角色定位 |
| [02 crate 架构](02-architecture.md) | 依赖拓扑图、分层模型、跨平台策略 |
| [03 异步基础](03-async-fundamentals.md) | Rust async/await 在嵌入式无堆场景的落地,与 stack-RTOS 的对比 |

### M2 核心组件 ✅

| 篇章 | 看什么 |
|---|---|
| [04 执行器 embassy-executor](04-executor.md) | 主循环、6 状态转移、ThreadMode 与 InterruptExecutor |
| [05 时间管理 embassy-time](05-time.md) | integrated vs generic 时间驱动、12 步 wake 链 |
| [06 同步原语 embassy-sync](06-sync.md) | Channel / Signal / Mutex / Pipe / Watch / Pubsub 的实现与决策树 |
| [07 异步工具 embassy-futures](07-futures.md) | select! / join! / yield_now / poll_once 的零开销实现 |

### M3 HAL 层架构 🚧(规划中)

> 待补:`08-hal-architecture.md` · `09-stm32.md` · `10-nrf.md` · `11-rp.md`

### M4 外设驱动 / M5 网络通信 / M6 系统组件 / M7 开发实践 ⏳

> 待启动,见 [项目仓库 tasks.md](https://github.com/daivy2333/embassy/blob/main/.claude/docs/tasks.md)

---

## ✨ 文档特色

- **源码引用准确** — 每个关键说明都给出 `file:line` 定位,可直接跳到源代码
- **图表优先** — Mermaid 流程图、状态机图、依赖图比文字更直观
- **对比表统一格式** — 相同维度横向对比,避免"含糊其辞"
- **术语首次定义** — 不假设你知道 PAC / metapac / RIOT 等专有词
- **不裁剪难点** — 实现细节、踩坑、设计权衡完整呈现

---

## 🛠️ 反馈与贡献

发现错误、有疑问、想补充?欢迎:

- 在 [GitHub Issues](https://github.com/daivy2333/embassy/issues) 提 issue
- 直接 PR 改进文档

---

## 📜 版权

本笔记基于 [embassy-rs/embassy](https://github.com/embassy-rs/embassy)(Apache-2.0 OR MIT)的源码分析撰写。原项目所有权归 Embassy 项目所有,本笔记仅为学习用途。
