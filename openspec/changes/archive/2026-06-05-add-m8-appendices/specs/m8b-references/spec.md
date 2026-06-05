# m8b-references: 参考资料汇总规格

> Version: 0.1.0
> Last updated: 2026-06-05
> 关联文档: `docs/appendix-b-references.md`
> 关联里程碑: M8.B
> 关联 OpenSpec change: `add-m8-appendices`

---

## ADDED Requirements

### Requirement: 5 大类组织

文档 MUST 按 5 大类组织参考资料:

1. **Embassy 官方**:embassy-rs/embassy 仓库 / embassy-book / examples 目录 / Awesome-Embassy
2. **ARM & Cortex-M 规范**:ARM Architecture Reference Manual / Cortex-M 编程手册 / CMSIS 标准
3. **Rust 资源**:The Rust Programming Language / Rust Async Book / The Embedded Rust Book / std 文档
4. **工具链**:probe-rs / defmt / QEMU / cargo-generate / rustup
5. **学习社区**:This Week in Rust / Embedded Working Group / r/rust / embassy Matrix 频道

#### Scenario: 类别完整性

- **WHEN** 读者需要某类资源
- **THEN** 文档 SHALL 在对应类别列出 3-10 条权威来源
- **AND** 每条 MUST 包含 URL / 仓库地址
- **AND** 简短说明(1 句话)在 M1-M7 的引用位置

### Requirement: URL 引用规范

文档 MUST 标注每个外部资源:

- 完整的 URL(优先使用永久链接,避免 404)
- 访问日期(2026-06)
- embassy 版本(0.5+)

#### Scenario: 链接稳定性

- **WHEN** 文档引用 `https://github.com/embassy-rs/embassy`
- **THEN** 文档 SHALL 标注"参考日期:2026-06,embassy 版本:0.5+"
- **AND** 优先使用永久链接(如 embassy-book 的 gh-pages 分支)

### Requirement: 必含资源清单

文档 MUST 至少包含以下资源:

- Embassy 官方仓库:`https://github.com/embassy-rs/embassy`
- Embassy Book:`https://embassy.dev/book/`
- Embassy examples:仓库内 `examples/` 目录(M8.C 详述)
- ARM ARM v7-M / v8-M Architecture Reference Manual
- The Rust Programming Language(TRPL):`https://doc.rust-lang.org/book/`
- Rust Async Book:`https://rust-lang.github.io/async-book/`
- The Embedded Rust Book:`https://docs.rust-embedded.org/book/`
- probe-rs:`https://probe.rs/`
- defmt:`https://defmt.ferrous-systems.com/`
- QEMU 文档:`https://www.qemu.org/docs/master/`

#### Scenario: 必含资源验证

- **WHEN** 读者按文档链接访问资源
- **THEN** SHALL 能正常打开
- **AND** 内容与 M1-M7 引用的 API 名称 / 用法一致

### Requirement: M1-M7 引用回标

文档 MUST 在每条参考资料后标注"M1-M7 引用示例"。

#### Scenario: 引用回标

- **WHEN** 文档列出 ARM ARM
- **THEN** 文档 SHALL 标注"M6.3 §6(低功耗 WFE/DSB)+ M5.3 §2(BLE 7 层模型)"
- **AND** 读者能跳转到对应 Mx.x 节号查看详细引用

### Requirement: 学习路径建议

文档 MUST 末尾提供"按 M1-M7 顺序的学习路径建议",告诉初学者"先读什么,后读什么"。

#### Scenario: 学习路径

- **WHEN** 读者是 embassy 新手
- **THEN** 文档 SHALL 给出:Week 1 读 embassy-book 前 5 章 / Week 2 跑通 embassy-template 闪灯 / Week 3-4 读 M1-M2 docs / Week 5-7 读 M3-M4 / ...
- **AND** 标注每阶段预计耗时
