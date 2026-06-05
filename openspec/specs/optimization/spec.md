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

#### OPT-001: 全项目历史文档 emoji 清理(2026-06-05)

**问题描述**:CLAUDE.md 2026-06-05 新增"文档撰写规范"(`二、务实编码原则`)明确禁用 emoji,但项目内多份历史文档仍带 emoji:

- `.claude/docs/tasks.md`:多处 `✅ 完成` 状态、`🚧`/`⏳` 等
- `.claude/docs/SNAPSHOT.md`:`✅ healthy`、`🚀 已配置` 等
- `openspec/specs/*/spec.md`:已完成区表格 `✅` 标记
- 部分 ADR 内 emoji 装饰(如有)

**当前影响**:
- 规则与现实不一致,新 agent 看 CLAUDE.md 又看不到全面执行,容易混淆
- 长期会让"禁用 emoji"规则形同虚设(破窗效应)

**建议方案**:
1. 全项目 grep 扫描列出所有命中
2. 按文件批量替换:`✅ 完成` → `(已完成)`、`✅ healthy` → `(健康)`、`🚀` → 删除或文字描述、`⚠️` → `(警告)`、`❌` → `(违规)` 等
3. 一次性 commit:`docs: cleanup historical emoji to comply with CLAUDE.md docs spec`

**优先级**:中。不影响功能,但影响项目规范一致性。建议在下次"文档大整理"时合并处理。

**估时**:30-45 分钟。

### 低优先级

（待填充）

---

## 已完成优化

| 日期 | 优化项 | 效果 |
|------|--------|------|
| - | - | - |
