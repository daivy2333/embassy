## Context

Embassy 学习研究项目已完成 M1-M5 五个里程碑(20 篇文档),覆盖基础架构、核心组件、HAL 层、外设驱动和网络通信栈。当前需要推进 M6 系统组件(3 篇文档:embassy-boot 启动引导 / FirmwareUpdater DFU / 低功耗设计模式)。

**当前状态**:
- `embassy-boot/` crate 存在且有完整源码(BootLoader + FirmwareUpdater 双核心 + digest_adapters + test_flash)
- 三平台薄壳 `embassy-boot-stm32` / `embassy-boot-nrf` / `embassy-boot-rp` 均存在
- 低功耗实现分布在 `embassy-stm32/src/executor.rs` / `embassy-rp/src/executor.rs` / `embassy-mcxa/src/executor.rs` 等
- M3-M5 共 14 篇文档已建立稳定的 11-12 节骨架模板
- CodeGraph 索引健康(46966 节点),Phase 1 已用 codegraph_explore 拉取 BootLoader/FirmwareUpdater/Executor::run 核心源码

**约束**:
- 本项目是学习研究项目,不做开发,只做分析
- 模板复用 M5 12 节骨架,节标题按 M6 主题调整
- Phase 3 一篇一篇来(M6.1 → 6.2 → 6.3)
- 文档撰写禁止使用 emoji(CLAUDE.md 规则)
- 所有产出物用简体中文,技术术语保持英文

## Goals / Non-Goals

**Goals:**
- 写完 M6.1 embassy-boot(~1200-1500 行 12 节)并 commit
- 写完 M6.2 FirmwareUpdater(~1200-1500 行 12 节)并 commit
- 写完 M6.3 低功耗设计(~1200-1500 行 12 节)并 commit
- 建立 M6 系列文档的术语一致性(分区/状态机/Magic/WFE/critical_section 共用词汇表)
- 用 CodeGraph 加速源码理解,引用具体行号(每篇 ≥ 20 处)
- 每篇配 Mermaid 图(boot 状态机、DFU 时序、Executor 主循环)

**Non-Goals:**
- 不修改 Embassy 任何源码
- 不深入特定芯片的 datasheet 章节(WFE 指令、SCB->VTOR、SCB->SCR 等仅引用 ARM 通用文档)
- 不在 Phase 3 自动 push(等用户决定)
- 不建立新的 spec 领域(用现有 learned spec 记录新发现)
- 不分析 nrf softdevice 的 sd_mbr_command_irq_forward_address_set 内部(平台特定 ABI,标注引用即可)
- 不深入 cortex_m::asm::bootload 的汇编细节(标注用途,链接到 cortex-m crate)

## Decisions

### D1: 模板复用 M5 12 节骨架,节标题按 M6 调整

**决策**:复用 M5.1/5.2/5.3 的 12 节结构(位置/概念/架构/...)但节标题按 M6 主题特化。

**理由**:
- M5 12 节模板经 3 篇主流文档验证,稳定性高
- M6 是"系统层"内容(boot / DFU / 低功耗都是系统级关注),需要"分区布局"、"状态机"、"Magic 协议"、"flash 算法"、"中断时序"、"功耗权衡" 等 M5 没有的节
- 保持模板骨架便于 MkDocs 导航和读者习惯

**M6 通用 12 节模板**:
1. 系统组件在 Embassy 中的位置(角色 + 上下游)
2. 核心类型与 trait 体系(BootLoader/FirmwareUpdater/Executor 等)
3. 数据/状态结构(分区布局、状态机、Magic 字节、TASKS_PENDING 协议)
4. 关键算法(swap/revert/copy_page_once、WFE/DSB 双指令、deep_sleep_if_possible)
5. 异步/同步 API 路径(async + blocking 双 API 对比,或 main loop 节流)
6. 错误处理与恢复(BootError、FirmwareUpdaterError、唤醒源管理)
7. 平台差异(STM32 / nRF / RP 三薄壳;executor 三档功耗实现)
8. 与其他子系统的协作(flash trait、watchdog、时钟、digest_adapters、interrupt)
9. 性能与资源(代码量、RAM、erase/write cycles、休眠唤醒延迟)
10. 实战示例(完整可运行片段:bootloader main / DFU 升级 / 低功耗 idle task)
11. 踩坑与最佳实践(WRITE_SIZE 对齐、DFU 容量、watchdog 持锁、SEV 双唤醒)
12. 平台对比表 + 总结(多维矩阵)

**备选方案**:
- 沿用 M3/M4 11 节原标题 → 系统类内容用"外设/中断"不贴切
- 不限定节数 → 用户决策保证必含节,但无骨架不便于导航

### D2: M6.1 优先,一篇一篇来

**决策**:Phase 3 按 6.1 → 6.2 → 6.3 顺序执行,每篇单独 commit。

**理由**:
- 每篇 ~1200-1500 行,一次性 3 篇共 ~3600-4500 行,质量难保证
- M5 已验证"一篇一篇 commit"模式有效
- 单独 commit 便于 review、revert
- M6.1 是 M6.2 的基础(FirmwareUpdater 依赖 BootLoader 的状态分区),需先写

**备选方案**:
- 一次性 3 篇 commit → 4500 行不便 review

### D3: M6.2 与 M6.1 内容边界

**决策**:M6.1 docs/21-boot.md 聚焦 `BootLoader` 一侧(bootloader 二进制视角:prepare_boot/read_state/swap/revert/copy_page_once 内部算法);M6.2 docs/22-dfu.md 聚焦 `FirmwareUpdater` 一侧(应用二进制视角:write_firmware/mark_updated/mark_booted/verify_and_mark_updated/digest_adapters)。共享部分(分区布局、State enum、Magic 协议)在 M6.1 详述,M6.2 标注交叉引用。

**理由**:
- 两侧 API 都需要单独深入,合并会撑爆篇幅
- bootloader 视角与应用视角是 Embassy 升级体系的天然边界
- 用户能清晰区分"我在写 bootloader"还是"我在写应用"

**备选方案**:
- 合并 1 篇(boot + DFU) → 一篇 3000 行不便阅读
- 按 async/blocking 分 → 两侧都有 async/blocking,会更乱

### D4: M6.3 跨平台呈现策略

**决策**:M6.3 docs/23-low-power.md 以 embassy-mcxa 的 CoreSleep 三档实现(WfeUngated / WfeGated / DeepSleep + critical_section)为主线案例(最完整),embassy-rp 与 embassy-stm32 的简化版本作为对比展示。

**理由**:
- embassy-mcxa 实现最完整(CoreSleep 三档 + go_around 优化 + critical_section 保护 deep_sleep + perf_counters)
- embassy-rp / embassy-stm32 的版本是 mcxa 的"子集",作对比能突出共性与差异
- 用户能理解"为何不同平台需要不同的低功耗策略"

**备选方案**:
- 以 embassy-stm32 为主 → 实现最简但缺乏深度
- 平行呈现三平台 → 重复内容多,无主线

### D5: CodeGraph 优先,禁止 grep/read 兜底

**决策**:文档撰写期间,所有源码探索通过 `codegraph_explore` / `codegraph_node` / `codegraph_callers` / `codegraph_callees` 完成,禁止用 grep 反向验证。

**理由**:
- CLAUDE.md 明确:CodeGraph 是预建索引,grep/read 重复更慢更贵
- M5 期间已验证 CodeGraph 健康(46966 节点)

## Risks / Trade-offs

| 风险 | 影响 | 缓解 |
|------|------|------|
| 3 篇文档共 ~3600-4500 行,工作量大 | 单次 Phase 3 可能需要分多次 commit | 一篇一篇来,每篇单独 commit |
| embassy-boot 代码精炼但概念密集(分区/状态机/Magic/progress) | 读者门槛高 | 用 Mermaid 状态机图 + 字节级布局表降低认知负担 |
| FirmwareUpdater async/blocking 双 API 对称 | 容易重复描述 | 用"对称表"集中展示差异,正文只详述 async,blocking 仅标注差异点 |
| digest_adapters 涉及密码学(ed25519、salty) | 学习曲线陡 | 仅介绍接口契约与典型用法,密码学原理引用外部资料 |
| 低功耗内容跨平台 + 跨指令集 + 跨时钟域 | 容易碎片化 | 以 embassy-mcxa 主线 + 三平台对比表 + 一张 Mermaid 主循环图整合 |
| WFE/DSB 指令对涉及 ARM 架构细节 | 非 ARM 用户读不懂 | 给出 ARM Architecture Reference Manual 章节链接,本文聚焦"为何用 DSB+WFE 而非裸 WFE" |
| critical_section 在 deep sleep 中持锁 | 不熟悉的人会担心死锁 | 单独小节解释"为何 deep_sleep_if_possible 内持锁安全" |
| Mermaid 状态机图复杂度过高 | 渲染失败 | 用简单的 stateDiagram-v2 + 每个状态附简短描述,避免嵌套 |
| nrf softdevice 内嵌汇编 | 平台特定不易解释 | 标注"softdevice 专属"+ 提供 nRF MBR 链接,本文只覆盖 boot 入口 |
| mkdocs.yml 缺 M6 nav 阻塞 push | push 失败 | 写完 21 后立即追加 21 到 nav,后续随写随追加 |
