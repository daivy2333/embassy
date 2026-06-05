## Context

Embassy 学习研究项目已完成 M1-M6 六个里程碑(23 篇文档),覆盖基础架构、核心组件、HAL 层、外设驱动、网络通信栈和系统组件。当前需要推进 M7 开发实践(4 篇文档:开发环境 / 调试 / 测试 / 设计模式),是收官里程碑,完成从"读懂源码"到"独立开发 Embassy 应用"的知识闭环。

**当前状态**:
- M1-M6 共 23 篇文档已建立稳定的 11-12 节骨架模板
- M6 已收官(`docs/21-23` + OpenSpec change `add-m6-system-components-docs` 已归档)
- CodeGraph 索引健康(46966 节点,1967 文件)
- `embassy-futures/` / `embassy-sync/` / `embassy-executor/` / `embassy-time/` 等核心 crate 已有深度分析(M2 系列)
- `embassy-template`(官方项目模板)、`cargo-generate`、probe-rs、defmt、RTT 等工具链已被 M1-M6 多处引用

**约束**:
- 本项目是学习研究项目,不做开发,只做分析
- 模板复用 M1-M6 章节骨架,按 M7 主题精简(工具型)或深化(综合型)
- 深度灵活:工具型 ~500-800 行,综合型 ~1000+ 行(用户决策"看情况精简就好")
- Phase 3 一篇一篇来(7.1 → 7.2 → 7.3 → 7.4)
- 文档撰写禁止使用 emoji(CLAUDE.md 规则)
- 所有产出物用简体中文,技术术语保持英文
- M7.2/M7.3 都依赖 M7.1 的工具链配置,故 M7.1 先做

## Goals / Non-Goals

**Goals:**
- 写完 M7.1 dev-setup(~600-800 行,工具速查型)并 commit
- 写完 M7.2 debugging(~600-800 行,工具速查型)并 commit
- 写完 M7.3 testing(~600-800 行,工具速查型)并 commit
- 写完 M7.4 patterns(~1200-1500 行,综合分析型)并 commit
- M7.4 综合 M2-M6 已分析的核心 crate(executor/sync/futures/time),呈现平台中立的设计模式
- 每篇配 Mermaid 图(开发流程图 / 调试时序图 / 测试架构图 / 模式时序图)
- 用 CodeGraph 加速源码理解,引用具体行号(每篇 ≥ 15 处)

**Non-Goals:**
- 不修改 Embassy 任何源码
- 不深入特定芯片的 datasheet 章节(probe-rs 协议、RTT 协议仅做引用)
- 不在 Phase 3 自动 push(等用户决定)
- 不建立新的 spec 领域(用现有 learned spec 记录新发现)
- 不覆盖 ESP-IDF / Zephyr 等 Embassy 之外的嵌入式框架
- 不做 nrf softdevice / embassy-boot 的二级深入(已在 M6 覆盖)
- 不做附录 A/B/C(术语表 / 参考资料 / 示例索引)— 留作后续可选增量

## Decisions

### D1: 模板复用 M1-M6 章节骨架,按 M7 主题精简或深化

**决策**:复用 M1-M6 的 11-12 节结构(背景/概念/架构/...),但 M7.1-M7.3(工具型)精简为 6-8 节,M7.4(综合型)保持 11-12 节。

**理由**:
- M1-M6 11-12 节模板经 23 篇文档验证,稳定性高
- 工具型主题(dev-setup/debugging/testing)读者期望"速查表",过长会失去焦点
- 综合型主题(patterns)需要"从历史到现代"的纵向展开,11-12 节合理
- 一致的节序号框架便于 MkDocs 导航和读者跨篇对照

**M7 通用章节模板**:
1. 主题在 Embassy 学习中的位置(角色 + 上下游 + 工具/模式索引)
2. 核心概念与原理(工具型:原理 1-2 段;综合型:6 大模式)
3. 关键工具/源码/模式(工具型:工具链组成;综合型:模式实现 crate)
4. 平台差异(STM32 / nRF / RP 三平台差异)
5. 实战示例(工具型:完整命令/配置;综合型:6 模式各 1 个完整代码片段)
6. 踩坑与最佳实践(工具型:常见错误;综合型:模式陷阱)
7. 速查表(工具型) / 模式决策树(综合型)

**M7.4 综合型额外节**:
8. 模式间的协作(模式组合 vs 模式冲突)
9. 演进与变体(从 embassy-futures 到 embassy-sync)
10. 性能与资源(各模式的内存/任务开销)
11. 实战案例(完整应用模板)
12. 总结与决策矩阵

**备选方案**:
- 全 4 篇统一深度 → 工具型被强行拉长,失去速查价值
- 全 4 篇精简 → M7.4 综合型深度不足

### D2: 实施顺序 7.1 → 7.2 → 7.3 → 7.4

**决策**:Phase 3 按 7.1 → 7.2 → 7.3 → 7.4 顺序执行,每篇单独 commit。

**理由**:
- 7.2 调试依赖 7.1 的工具链(probe-rs、defmt-print),7.3 测试依赖 7.1 的工具链(test runner)
- 7.1 是入门必备,先做便于后续所有文档的引用一致
- 7.4 设计模式是 M2-M6 知识综合,放最后能引用所有前序文档
- 4 篇单独 commit 便于 review、revert

**备选方案**:
- 7.4 优先 → 综合性内容依赖 M2-M6 引用,放最后最自然
- 一次性 4 篇 commit → 3000-4000 行不便 review

### D3: 工具型 vs 综合型深度差异化

**决策**:M7.1-M7.3 工具型 ~500-800 行,聚焦"命令 + 配置 + 输出 + 故障排查";M7.4 综合型 ~1200-1500 行,6 模式各 ~200 行展开。

**理由**:
- 用户决策"看情况精简就好",且工具型主题本质是"用法速查",不需长篇理论
- M7.4 是 M2-M6 知识综合,需要深度展开才能体现"模式"价值
- 工具型 + 综合型混合呈现,使 4 篇整体工作量合理(9h 估计)
- 速查表型便于初学者快速上手,综合型便于资深开发者查阅

**备选方案**:
- 4 篇统一 1000+ 行 → 工具型硬拉长,信息密度低
- 4 篇统一 500-800 行 → M7.4 深度不够

### D4: M7.4 模式选择与案例来源

**决策**:M7.4 docs/27-patterns.md 选 6 大模式,每模式以 embassy-* 已有 crate 为案例:

| 模式 | 案例 crate | M2-M6 交叉引用 |
|------|-----------|----------------|
| 状态机 | embassy-futures::select_biased | M2.4 `07-futures.md` §7 |
| 生产者-消费者 | embassy-sync::Channel | M2.3 `06-sync.md` §4 |
| 发布-订阅 | embassy-sync::Signal + PubSubChannel | M2.3 `06-sync.md` §5-6 |
| Watchdog | embassy-time::Timer + 平台 watchdog | M2.2 `05-time.md` + M6.3 `23-low-power.md` |
| 低功耗调度 | embassy-executor::Executor + WFE | M2.1 `04-executor.md` §3 + M6.3 `23-low-power.md` |
| RAII 资源管理 | embassy-* 的 Drop + on_drop 回调 | M2.4 `07-futures.md` + M6.1 `21-boot.md` |

**理由**:
- 6 模式覆盖 Embassy 异步编程的全部典型场景
- 每个模式都有现成 crate 可引用,避免"凭空构造"
- 与 M2-M6 已分析的内容形成"模式视角"的二次综合,完成知识闭环
- 用户能直接定位"我的场景该用哪个模式"

**备选方案**:
- 选经典 OOP/函数式模式(Strategy/Observer/Factory)→ 不贴 Embassy 实际使用,泛泛而谈
- 选 10+ 模式 → 覆盖广但每模式浅,失去综合价值

### D5: M7.1-M7.3 跨平台呈现策略

**决策**:M7.1/M7.2/M7.3 中,跨平台命令/配置以 STM32 / nRF / RP 三平台分别给出,使用统一的命令模板(`probe-rs run --chip STM32F411CEUx` vs `--chip nRF52840_xxAA` vs `--chip RP2040`)。

**理由**:
- M3-M6 文档已建立"三平台对比表"的稳定呈现方式
- 工具链命令与平台强相关,不能简化为"通用命令"
- 表格对比利于快速定位本平台的命令
- 减少读者"我平台怎么用?"的反复查找

**备选方案**:
- 只给 STM32 命令 → 其他平台读者不适应
- 给出所有 embassy-* 支持平台 → 命令膨胀(8+ 平台)

## Risks / Trade-offs

| 风险 | 影响 | 缓解 |
|------|------|------|
| 工具型 3 篇总长 ~1800-2400 行,内容可能稀疏 | 速查价值未达预期 | 用"速查表 + 故障排查清单"充实,避免硬拉理论 |
| 工具链命令随 embassy/probe-rs 版本变化 | 文档半年后可能过时 | 标注版本依赖,提供"参考日期"和"upgrade path"小节 |
| probe-rs 协议/RTT 协议跨 crate 复杂 | M7.2 调试内容易超界 | 仅讲用法,协议层引用 probe-rs / rtt-target 文档 |
| embassy-test 框架在 Embassy 生态尚在演进 | M7.3 测试内容可能需调整 | 标注"以 embassy-test 当前 master 为准",同时介绍原生 cargo test + QEMU 路径 |
| M7.4 综合 M2-M6 引用密集 | 与前序文档重复内容多 | 用"前序索引 + 模式视角"避免重复展开,聚焦模式本身的决策点 |
| 6 模式选择可能不全面 | 用户场景未覆盖 | 在文末给出"未覆盖模式"清单 + 推荐阅读 |
| cargo-generate / embassy-template 在 embassy-* crate 列表中位置不固定 | M7.1 引用路径可能漂移 | 引用 embassy-rs/embassy GitHub 主页 + 具体 commit |
| 4 篇 + mkdocs.yml nav + archive 一致性 | nav 漏配会导致 GitHub Pages 404 | 写完 24 后立即追加 24 到 nav,后续随写随追加 |
| 测试覆盖率工具(llvm-cov/tarpaulin)对嵌入式目标支持弱 | M7.3 实战价值打折 | 明确区分"host 测试覆盖率 vs target 测试覆盖率",只演示 host |
| RAII 模式在 Rust 无传统析构语义(只有 Drop) | 模式命名易误解 | 强调"Rust RAII = 资源获取即初始化 + Drop 即释放"的 Rust 等价 |
| Mermaid 时序图对 6 模式过多会碎片化 | 阅读疲劳 | 6 模式用统一的 sequenceDiagram 模板,只在参数和事件上差异 |
| 4 篇文档零散 commit 多,易冲突 | 暂存区混乱 | 每篇独立 commit,文件路径完全独立(docs/24-27.md) |

## Migration Plan

M7 是纯文档增量变更,无运行时/构建系统影响。完成顺序:

1. 写完 M7.1 → commit(不 push)
2. 写完 M7.2 → commit(不 push)
3. 写完 M7.3 → commit(不 push)
4. 写完 M7.4 → commit(不 push)
5. 更新 `mkdocs.yml` nav 加入 24-27
6. `openspec validate --change add-m7-dev-practice-docs`
7. 等待用户决策 push 时机
8. `openspec archive add-m7-dev-practice-docs`
9. 更新 `tasks.md` 和 `SNAPSHOT.md`(M7 全部完成 → 27/27 = 100% 收官)
10. push(由用户决定)

**回滚策略**:
- 每篇独立可回滚:`git revert <commit>` + 撤销 `mkdocs.yml` nav 中的对应行
- 整个 change 回滚:删除 `docs/24-27.md` + 撤销 mkdocs.yml 4 行 nav + 删除 `openspec/changes/add-m7-dev-practice-docs/`
