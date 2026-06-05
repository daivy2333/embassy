## Why

本项目为 Embassy 嵌入式异步框架的学习研究项目,目标是系统性分析其架构与实现细节。已完成的 M1-M5 五个里程碑(20 篇文档)覆盖了基础架构、核心组件、HAL 层、外设驱动和网络通信栈。M6 系统组件(3 篇文档)是下一个关键里程碑,涵盖 embassy-boot 启动引导 / 固件升级与 DFU 机制 / 低功耗设计模式,补完 Embassy "运行时 + 网络 + 系统" 全栈分析的最后一块。本次按 M5 风格一篇一篇来。

## What Changes

- 新增 `docs/21-boot.md`(M6.1 embassy-boot 启动引导分析,目标 ~1200-1500 行 11-12 节,含 Mermaid 状态机图与三平台对比表)
- 新增 `docs/22-dfu.md`(M6.2 固件升级与 DFU 机制,目标 ~1200-1500 行 11-12 节,含完整 swap/revert 状态机)
- 新增 `docs/23-low-power.md`(M6.3 低功耗设计模式,目标 ~1200-1500 行 11-12 节,含 Executor::run 主循环与 WFE/DSB 双指令分析)
- 不做的事:不修改 Embassy 任何源码(本项目是学习项目,不做开发);不改 mkdocs.yml 之外的其他配置文件;不深入特定芯片的 datasheet 章节
- 模板:复用 M5 12 节骨架,节标题按 M6 主题调整(如"分区布局"、"swap 状态机"、"签名验证"、"WFE/DSB 指令对"、"deep sleep 时钟切换"等)

## Capabilities

### New Capabilities

- `m61-embassy-boot`: docs/21-boot.md 的内容契约,定义 embassy-boot 启动引导分析的范围(三分区设计、State 状态机、Magic 字节协议、swap/revert 算法、平台薄壳差异、链接器集成)
- `m62-firmware-updater`: docs/22-dfu.md 的内容契约,定义 FirmwareUpdater 分析的范围(async + blocking 双 API、write_firmware 流程、mark_updated/mark_booted 状态切换、签名验证 digest_adapters、prepare_update 优化路径)
- `m63-low-power`: docs/23-low-power.md 的内容契约,定义低功耗设计模式分析的范围(Executor::run 主循环、WFE/DSB 指令对、TASKS_PENDING 协议、SEV 唤醒、三档功耗模式、deep sleep + critical_section、平台差异)

### Modified Capabilities

(无 — 不修改现有 spec)

## Impact

- **代码**:零影响(本项目不做开发)
- **API**:零影响
- **依赖**:零影响
- **文档**:新增 3 篇 Markdown 文档到 `docs/`(累计 ~3600-4500 行)
- **OpenSpec**:`openspec/specs/learned/` 增量条目(M6 系列踩坑/技巧/速查待 M6 各篇完成后追加)
- **GitHub Pages**:新增后需更新 `mkdocs.yml` nav 加入 21-23 项
- **回滚方案**:删除 `docs/21-*.md ~ 23-*.md` + 撤销 mkdocs.yml nav 改动。每个文档独立可回滚。

## 跨平台兼容性

M6.1 embassy-boot 跨 STM32/nRF/RP 三平台薄壳以"核心 + 平台适配 + WatchdogFlash"统一抽象呈现。M6.2 FirmwareUpdater 平台中立(基于 embedded-storage::NorFlash trait),签名验证可选 digest_adapters。M6.3 低功耗跨 embassy-stm32 / embassy-rp / embassy-mcxa 三种 Executor 实现以"WFE 简单循环 vs CoreSleep 三档"对比呈现。

## Phase 3 实施顺序

按 M5 经验"一篇一篇来":
1. M6.1 embassy-boot 启动引导(本次)
2. M6.2 FirmwareUpdater + DFU 机制
3. M6.3 低功耗设计模式

每篇完成后做一次 commit(不自动 push,等用户决定时机)。
