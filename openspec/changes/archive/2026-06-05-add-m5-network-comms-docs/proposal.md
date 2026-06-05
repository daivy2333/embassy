## Why

本项目为 Embassy 嵌入式异步框架的学习研究项目,目标是系统性分析其架构与实现细节。已完成的 M1-M4 三个里程碑(16 篇文档)覆盖了基础架构、核心组件、HAL 层和外设驱动。M5 网络与通信栈(4 篇文档)是下一个关键里程碑,涵盖 embassy-net / embassy-usb / BLE 协议 / LoRa 远程通信,补完 Embassy "网络+无线+长距" 全栈分析。本次先做 M5.1 embassy-net,后续 M5.2-M5.4 一篇一篇来。

## What Changes

- 新增 `docs/17-net.md`(M5.1 embassy-net 网络栈,目标 ~1000 行 11 节,含 Mermaid 架构图与跨 PHY 平台对比表)
- 新增 `docs/18-usb.md`(M5.2 embassy-usb 设备栈,目标 ~1000 行 11 节)
- 新增 `docs/19-ble.md`(M5.3 BLE 协议平台中立,目标 ~800 行 11 节)
- 新增 `docs/20-lora.md`(M5.4 LoRa 远程通信,目标 ~200 行说明文档,因本 fork 无 `embassy-lora` crate 不做实现分析,仅说明原因与替代方案)
- 不做的事:不修改 Embassy 任何源码(本项目是学习项目,不做开发);不改 mkdocs.yml 之外的其他配置文件
- 模板:复用 M3/M4 11 节骨架,节标题按 M5 主题调整(如"协议状态机"、"缓冲区管理"、"错误恢复"、"跨平台 PHY 集成"等)

## Capabilities

### New Capabilities

- `m51-embassy-net`: docs/17-net.md 的内容契约,定义 embassy-net 网络栈分析的范围(架构、smoltcp 集成、Driver trait、TCP/UDP/DHCP/DNS、跨 PHY 平台对照)
- `m52-embassy-usb`: docs/18-usb.md 的内容契约,定义 embassy-usb 设备栈分析的范围(描述符体系、端点机制、device class、builder 模式)
- `m53-ble-protocol`: docs/19-ble.md 的内容契约,定义 BLE 协议平台中立分析的范围(GAP/GATT/广播/连接/MTU/加密)
- `m54-lora-note`: docs/20-lora.md 的内容契约,定义 LoRa 文档结构(说明本 fork 无 embassy-lora crate 不做实现分析,给出概念入门 + 集成模式 + 后续建议)

### Modified Capabilities

(无 — 不修改现有 spec)

## Impact

- **代码**:零影响(本项目不做开发)
- **API**:零影响
- **依赖**:零影响
- **文档**:新增 4 篇 Markdown 文档到 `docs/`(累计 ~3000 行)
- **OpenSpec**:`openspec/specs/learned/` 增量条目(M5 系列踩坑/技巧/速查待 M5.1 完成后追加)
- **GitHub Pages**:新增后需更新 `mkdocs.yml` nav 加入 17-20 项
- **回滚方案**:删除 `docs/17-*.md ~ 20-*.md` + 撤销 mkdocs.yml nav 改动。每个文档独立可回滚。

## 跨平台兼容性

M5.1 embassy-net 跨 PHY 平台(STM32 ETH / nrf radio / RP CYW43 / ESP32 / W5500 SPI)以"Driver trait + PHY 适配"统一抽象呈现。M5.2 USB 跨 STM32/nRF/RP USB IP 差异以"USB IP 适配层"呈现。M5.3 BLE 协议中立,不绑特定平台。

## Phase 3 实施顺序

按用户决策"一篇一篇来":
1. M5.1 embassy-net(本次)
2. M5.2 embassy-usb
3. M5.3 BLE 协议中立
4. M5.4 LoRa 说明文档

每篇完成后做一次 commit(不自动 push,等用户决定时机)。
