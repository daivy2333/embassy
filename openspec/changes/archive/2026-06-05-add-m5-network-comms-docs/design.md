## Context

Embassy 学习研究项目已完成 M1-M4 三个里程碑(16 篇文档),覆盖基础架构、核心组件、HAL 层与外设驱动。当前需要推进 M5 网络与通信栈(4 篇文档:embassy-net / embassy-usb / BLE 协议 / LoRa)。

**当前状态**:
- `embassy-net/` 与 `embassy-usb/` crate 存在且有完整源码可分析
- `embassy-stm32-wpan/` crate 存在(BLE 实现)
- 本 fork **无** `embassy-lora` crate(M5.4 仅能写说明文档)
- M3/M4 11 篇文档已建立稳定的 11 节骨架模板
- CodeGraph 索引健康(46966 节点),可加速源码探索

**约束**:
- 本项目是学习研究项目,不做开发,只做分析
- 模板复用 M3/M4 11 节骨架,节标题按 M5 主题调整
- Phase 3 一篇一篇来(M5.1 → 5.2 → 5.3 → 5.4)
- 文档撰写禁止使用 emoji(CLAUDE.md 规则)
- 所有产出物用简体中文,技术术语保持英文

## Goals / Non-Goals

**Goals:**
- 写完 M5.1 embassy-net(~1000 行 11 节)并 commit
- 框架化 M5.2/5.3/5.4 的章节结构(为后续撰写准备)
- 建立 M5 系列文档的术语一致性(网络/通信共用词汇表)
- 用 CodeGraph 加速源码理解,引用具体行号
- 每篇配 Mermaid 架构图(网络栈、协议状态机、缓冲区流)

**Non-Goals:**
- 不修改 Embassy 任何源码
- 不实现 LoRa 学习分析(本 fork 无 crate)
- 不做 BLE 平台特定实现分析(用户决策:协议中立)
- 不在 Phase 3 自动 push(等用户决定)
- 不建立新的 spec 领域(用现有 learned spec 记录)

## Decisions

### D1: 模板复用 M3/M4 11 节骨架,节标题按 M5 调整

**决策**:复用 M3.2/3.3/3.4 / M4.1-4.5 的 11 节结构(位置/概念/架构/...)但节标题按 M5 主题特化。

**理由**:
- M3/M4 11 节模板经 11 篇文档验证,稳定性高
- M5 是"协议类"内容(网络/USB/BLE/LoRa 都是协议栈),需要"协议状态机"、"缓冲区管理"、"错误恢复"、"MTU/帧"、"跨平台 PHY 集成" 等 M3/M4 没有的节
- 保持模板骨架便于 MkDocs 导航和读者习惯

**M5 通用 11 节模板**:
1. 协议在 Embassy 中的位置(角色 + 上下游)
2. 核心 trait / 类型体系(协议抽象)
3. 协议状态机(连接/断开/重试/超时)
4. 缓冲区管理(TX/RX ring buffer、池化、零拷贝)
5. 异步 API 路径(`async fn` 设计与 waker 链)
6. 错误处理与恢复(超时、重传、复位)
7. 跨平台 PHY 集成(STM32/nRF/RP 各自的 PHY 驱动)
8. 性能与资源(CPU/内存/带宽)
9. 与其他子系统的协作(executor、time、sync)
10. 实战示例(完整可运行片段)
11. 平台对比表 + 总结(10 维矩阵)

**备选方案**:
- 沿用 M3/M4 11 节原标题 → 协议类内容用"输入模式/输出模式"不贴切
- 不限定节数 → 用户决策保证必含节,但无骨架不便于导航

### D2: M5.1 优先,一篇一篇来

**决策**:Phase 3 按 5.1 → 5.2 → 5.3 → 5.4 顺序执行,每篇单独 commit。

**理由**:
- 每篇 ~1000 行,一次性 4 篇共 ~3000 行,质量难保证
- 用户明确"一篇一篇来"
- 单独 commit 便于 review、revert

**备选方案**:
- 一次性 4 篇 commit → 违反用户决策,且 1 commit 含 4000 行不便 review

### D3: M5.3 BLE 协议中立(不绑特定平台)

**决策**:docs/19-ble.md 写 BLE 协议本身(GAP/GATT/广播/连接/MTU),不深入 stm32-wpan 或 nrf softdevice 平台特定实现。

**理由**:
- 用户决策"BLE 协议本身(平台中立)"
- BLE 协议知识相对独立,适合协议中立学习
- 平台特定实现可在后续 M6/M7 系统组件或附录中补

**备选方案**:
- stm32-wpan 为主 → 失去协议中立性
- 两者对比 → 篇幅爆炸,且与 M3 平台分析重叠

### D4: M5.4 LoRa 仅写说明文档

**决策**:docs/20-lora.md 短篇说明文档(目标 ~200 行),不写实现分析。

**理由**:
- 本 fork 无 `embassy-lora` crate,无法做源码分析
- 用户决策"说明不重要,单独说明没有做文档的原因就好"
- 短文档避免空白占位,提供概念入门 + 集成模式 + 后续建议

**包含内容**:
- 1-2 节 LoRa 概念入门(物理层、扩频、LoRaWAN)
- 1 节 Embassy 中可能的集成模式(SX127x over SPI + LoRaWAN 状态机)
- 1 节 本 fork 不做实现分析的原因(无 embassy-lora crate)
- 1 节 后续建议(等 upstream embassy-lora 同步,或参考 stm32/esp32 实现)

### D5: CodeGraph 优先,禁止 grep/read 兜底

**决策**:文档撰写期间,所有源码探索通过 `codegraph_explore` / `codegraph_node` / `codegraph_callers` / `codegraph_callees` 完成,禁止用 grep 反向验证。

**理由**:
- CLAUDE.md 明确:CodeGraph 是预建索引,grep/read 重复更慢更贵
- M3.2-3.4 与 M4.1-4.5 期间已验证 CodeGraph 健康(46966 节点)

## Risks / Trade-offs

| 风险 | 影响 | 缓解 |
|------|------|------|
| 4 篇文档共 ~3000 行,工作量大 | 单次 Phase 3 可能需要分多次 commit | 一篇一篇来,每篇单独 commit |
| embassy-net 代码量大,源码探索耗时 | M5.1 可能超时 | 用 codegraph_explore 一次取多模块,按文件分组阅读 |
| smoltcp 是独立 crate(本 fork 可能有 vendor) | smoltcp 内部细节可能不完整 | 引用 smoltcp 官方文档,标注"外部依赖" |
| M5.4 缺失实现分析,文档偏薄 | 与其他 3 篇不一致 | 显式说明原因,给出后续建议,避免"假装完整" |
| Mermaid 复杂度过高导致渲染失败 | 文档站异常 | 复用 M3/M4 已验证的 mermaid 模板,测试简单图 |
| BLE 协议中立可能导致内容空泛 | 失去"代码引用"支撑 | 用 BLE spec 关键概念(GAP/GATT)对照 Embassy 抽象,提供伪代码示例 |
| mkdocs.yml 缺 M5 nav 阻塞 push | push 失败 | 写完 17 后立即追加 17 到 nav,后续随写随追加 |
| 跨 PHY 平台(STM32 ETH / nrf radio / RP CYW43)差异大 | 跨平台对比表难写 | 抽 10 维共性指标,不深入 PHY 特定细节 |
