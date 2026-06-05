## Purpose

记录项目中发现的优化点和改进方向，持续提升代码质量和性能。

## Requirements

### Requirement: 优化点记录

发现的性能瓶颈、代码异味、技术债务 SHALL 记录，包含当前影响和建议方案。

#### Scenario: 发现优化机会

- **WHEN** 开发者发现代码中存在性能问题、重复代码、过度复杂设计等
- **THEN** 必须记录到 optimization/spec.md，包含：问题描述、当前影响、建议方案、优先级

#### Scenario: 评估优化价值

- **WHEN** 开发者需要决定是否进行某项优化
- **THEN** 可以参考 optimization/spec.md 中的记录，评估影响范围和收益

### Requirement: 优化完成追踪

已完成的优化 SHALL 记录完成状态，保留历史记录。

#### Scenario: 完成优化

- **WHEN** 开发者完成了某项优化工作
- **THEN** 必须更新 optimization/spec.md，标记为已完成，记录完成日期和实际效果

### Requirement: 优化优先级管理

优化点 SHALL 有优先级排序，合理安排优化顺序。

#### Scenario: 规划优化计划

- **WHEN** 开发者制定优化计划时
- **THEN** 可以参考 optimization/spec.md 中的优先级标注，优先处理高优先级优化点

---

## 优化点列表

### 高优先级

（待填充）

### 中优先级

（待填充）

### 低优先级

（待填充）

---

## 已完成优化

| 日期 | 优化项 | 效果 |
|------|--------|------|
| 2026-06-05 | **OPT-001 全项目历史文档 emoji 清理**:批量处理 21 个项目自有 `.md`(`CLAUDE.md` + `docs/01-11` + `.claude/docs/*` + `openspec/specs/*` + 2 plan),消除全部 emoji,代之以文字状态(`(已完成)` / `(规划中)` / `(待启动)` / `禁止:` / `必须:` 等)。同时修正 53 处 `learn/` → `docs/` 路径残留;CLAUDE.md 文档体系段重写;SNAPSHOT.md 项目结构刷新;optimization spec OPT-001 移到本区。规则与现实一致,杜绝破窗效应。 | 项目规范一致性达成,grep 检索友好度提升,新 agent 看 CLAUDE.md 即可全面执行 |
