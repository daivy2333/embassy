## Purpose

记录项目依赖和外部参考资源，确保依赖可追溯，资源可获取。

## Requirements

### Requirement: 依赖版本锁定

所有项目依赖 SHALL 记录版本信息，确保构建可重现。

#### Scenario: 添加新依赖

- **WHEN** 开发者引入新的外部依赖
- **THEN** 必须记录到 references/spec.md，包含：依赖名称、版本、官方链接、用途说明

#### Scenario: 更新依赖版本

- **WHEN** 开发者升级或降级依赖版本
- **THEN** 必须更新 references/spec.md 中的版本记录，标注更新原因

### Requirement: 外部资源记录

项目使用的外部资源和文档 SHALL 记录，方便查阅。

#### Scenario: 参考外部文档

- **WHEN** 开发者参考了重要的外部文档或资源
- **THEN** 必须记录到 references/spec.md，包含：资源名称、链接、关键内容摘要

### Requirement: 项目分析文档索引

深度分析文档 SHALL 建立索引，方便查找。

#### Scenario: 生成分析文档

- **WHEN** openspec-explorer 生成了项目分析文档
- **THEN** 必须在 references/spec.md 中注册索引条目，包含：主题、路径、内容概要

---

## 核心依赖

| 依赖 | 版本 | 用途 | 链接 |
|------|------|------|------|
| defmt | - | 嵌入式日志框架 | https://github.com/knurling-rs/defmt |
| probe-rs | - | 调试/烧录工具 | https://probe.rs |
| rust-toolchain | stable | Rust 编译器 | https://rustup.rs |

## 硬件平台支持

| 平台 | Crate | 状态 |
|------|-------|------|
| STM32 | embassy-stm32 | 活跃 |
| nRF52/53/54/91 | embassy-nrf | 活跃 |
| RP2040/235x | embassy-rp | 活跃 |
| ESP32 | esp-hal (外部) | 活跃 |
| NXP MCX-A | embassy-mcxa | 活跃 |
| TI MSPM0 | embassy-mspm0 | 活跃 |
| Microchip | embassy-microchip | 活跃 |

## 外部资源

| 资源 | 链接 | 说明 |
|------|------|------|
| Embassy Book | https://embassy.dev/book/ | 官方文档 |
| API Reference | https://docs.embassy.dev/ | API 文档 |
| Matrix Chat | https://matrix.to/#/#embassy-rs:matrix.org | 社区聊天 |
| GitHub | https://github.com/embassy-rs/embassy | 源码仓库 |

---

## 项目分析文档索引

（待填充 - 由 openspec-explorer 生成）
