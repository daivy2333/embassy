## Why

本项目为 Embassy 嵌入式异步框架的学习研究项目,已完成 M1-M7 七个里程碑(27 篇文档,合计约 24300 行),覆盖基础架构、核心组件、HAL 层、外设驱动、网络通信栈、系统组件和开发实践。本次 M8 附录(3 篇文档)是项目最终收官,补完"参考体系"维度:术语表统一跨文档词汇、参考资料汇总外部权威来源、示例索引按主题分类 embassy 仓库 `examples/` 可运行案例,形成从"读懂"到"查阅"再到"动手试"的知识闭环。

## What Changes

- 新增 `docs/appendix-a-glossary.md`(M8.A 术语表,~400-600 行,A-Z 字母索引,覆盖 M1-M7 出现的所有专业术语)
- 新增 `docs/appendix-b-references.md`(M8.B 参考资料汇总,~400-600 行,分 5 类:官方文档/标准规范/Rust 资源/工具文档/学习社区)
- 新增 `docs/appendix-c-examples.md`(M8.C 示例索引,~400-600 行,按主题分类 embassy `examples/` 目录中的可运行案例)
- 不做的事:不修改 Embassy 任何源码(本项目是学习项目,不做开发);不重复 M1-M7 已分析内容(术语表是索引式,非教程式);不做 Rust 教程(假设读者已有 Rust 基础)
- 模板:简明条目式,每术语 1-2 句定义,参考资料带 URL 链接,示例带路径与用途
- 0 emoji(与 M1-M7 一致)

## Capabilities

### New Capabilities

- `m8a-glossary`: docs/appendix-a-glossary.md 的内容契约,定义术语表的范围(A-Z 字母索引,分类标签:M1/M2/M3/M4/M5/M6/M7 + 类型:术语/缩写/概念/工具)
- `m8b-references`: docs/appendix-b-references.md 的内容契约,定义参考资料的范围(5 大类:embassy 官方 / ARM & Cortex-M 规范 / Rust 资源 / 工具链(probe-rs/defmt/QEMU)/ 学习社区)
- `m8c-examples`: docs/appendix-c-examples.md 的内容契约,定义示例索引的范围(按 M1-M7 主题分类 embassy `examples/` 中可运行 bin 文件,标注每例的 chip / 平台 / 关键 embassy API)

### Modified Capabilities

(无 — 不修改现有 spec)

## Impact

- **代码**:零影响(本项目不做开发)
- **API**:零影响
- **依赖**:零影响
- **文档**:新增 3 篇 Markdown 文档到 `docs/appendix-*.md`(累计 ~1200-1800 行)
- **OpenSpec**:本次是项目最终增量,完成后无新 spec 领域
- **GitHub Pages**:需更新 `mkdocs.yml` nav 加入 3 个附录项
- **回滚方案**:删除 3 个 `docs/appendix-*.md` + 撤销 mkdocs.yml nav 改动

## 跨平台兼容性

附录 A 术语表包含三平台相关术语(STM32 / nRF / RP 各自的 MCU 名、引脚命名、调试探针)。附录 B 参考资料列三平台官方链接。附录 C 示例按平台分类(STM32F4 / STM32H7 / STM32L4 / nRF52 / nRF5340 / RP2040 / RP235x 等)。

## Phase 3 实施顺序

按"参考价值从高到低"排序:
1. M8.A 术语表(其他文档查阅的最高频入口)
2. M8.B 参考资料(为术语表与示例提供外部链接)
3. M8.C 示例索引(综合 M8.A 术语 + M8.B 参考 + M1-M7 已分析 API)

每篇完成后做一次 commit(不自动 push,等用户决定时机)。
