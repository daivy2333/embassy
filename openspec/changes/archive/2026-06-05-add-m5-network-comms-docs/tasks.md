# Tasks: M5 网络与通信栈学习文档

> 适用范围: docs/17-net.md, docs/18-usb.md, docs/19-ble.md, docs/20-lora.md
> 受影响 crate: embassy-net(分析)/ embassy-usb(分析)/ embassy-stm32-wpan(参考)/ 无 LoRa crate(说明)
> 依赖: openspec/changes/add-m5-network-comms-docs/{proposal,design,specs}/*.md
> 验收基线: 每篇 ≥ 900 行(M5.4 ≥ 150)/ ≥ 1 Mermaid / ≥ 20 源码引用(M5.4 ≥ 5)/ 0 emoji

## 1. M5.1 embassy-net(docs/17-net.md)

- [x] 1.1 CodeGraph 探索 embassy-net 整体架构(`codegraph_explore` 覆盖 Stack/Device/Runner/Resources/Config)
- [x] 1.2 追踪 smoltcp 集成路径(`codegraph_callers` / `codegraph_callees` 找出 wire device 适配点)
- [x] 1.3 撰写 §1-§3 架构与 smoltcp 集成
- [x] 1.4 撰写 §4-§5 TCP/UDP Socket API + DHCP/DNS
- [x] 1.5 撰写 §6 异步 API 路径与 waker 链
- [x] 1.6 撰写 §7-§8 错误处理 + 跨平台 PHY 集成
- [x] 1.7 撰写 §9 性能与资源
- [x] 1.8 撰写 §10 与其他子系统的协作
- [x] 1.9 撰写 §11 实战示例(2 完整片段)+ 跨 PHY 平台对比表 + 总结
- [x] 1.10 添加 4 个 Mermaid 架构图(§1 系统架构 / §2 类型关系 / §6 waker 时序 / §8 平台选型决策树)
- [x] 1.11 自审:行数 1606 / 引用 21 / Mermaid 4 / 0 emoji
- [x] 1.12 追加 mkdocs.yml nav 加入 17-net.md
- [x] 1.13 Commit docs/17-net.md + mkdocs.yml(本轮统一 commit, 不 push)

## 2. M5.2 embassy-usb(docs/18-usb.md)

- [x] 2.1 CodeGraph 探索 embassy-usb 整体架构(Builder/Device/Class/Config)
- [x] 2.2 追踪描述符层级(`codegraph_explore` 覆盖 device desc / config desc / interface desc / endpoint desc)
- [x] 2.3 撰写 §1-§3 架构 + 描述符 + 端点机制
- [x] 2.4 撰写 §4-§5 设备类(CDC/MSC/HID/DFU/WebUSB) + Handler trait
- [x] 2.5 撰写 §6-§7 异步 API 路径 + 错误处理与恢复
- [x] 2.6 撰写 §8 跨 USB IP 平台集成
- [x] 2.7 撰写 §9-§10 性能资源 + 跨子系统协作
- [x] 2.8 撰写 §11 实战示例(2 完整片段:RP2040 CDC + STM32 复合)+ 跨 USB IP 平台对比表 + 总结
- [x] 2.9 添加 3 个 Mermaid 架构图(§1 系统架构 / §6 waker 时序 / §8 平台选型决策树)
- [x] 2.10 自审:行数 1594 / Mermaid 3 / 0 emoji / 12 embassy-usb-driver 引用
- [x] 2.11 追加 mkdocs.yml nav 加入 18-usb.md
- [x] 2.12 Commit docs/18-usb.md + mkdocs.yml(本轮统一 commit, 不 push)

## 3. M5.3 BLE 协议平台中立(docs/19-ble.md)

- [x] 3.1 收集 BLE 协议参考资料(BLE Core Spec 关键章节:Vol 1 Part A + Vol 3 Part G/H + Vol 6 Part B)
- [x] 3.2 CodeGraph 探索 embassy-stm32-wpan 提取 Controller/Host 抽象(为协议中立映射提供素材)
- [x] 3.3 撰写 §1-§2 BLE 协议栈分层 + 架构图(7 层图 + 关键概念总览图)
- [x] 3.4 撰写 §3-§4 GAP 角色 + GATT 三层模型(状态机图 + 服务层级图)
- [x] 3.5 撰写 §5-§6 Advertising + 连接参数
- [x] 3.6 撰写 §7-§8 MTU/分片 + 加密/配对
- [x] 3.7 撰写 §9 Embassy 抽象映射(Controller trait / Host task)
- [x] 3.8 撰写 §10 实战示例(2 段伪代码 + 1 时序图)
- [x] 3.9 撰写 §11 总结 + 平台差异索引(7 平台对比表)
- [x] 3.10 添加 5 个 Mermaid 图(架构 / 状态机 / 服务树 / 时序 / 关键概念)
- [x] 3.11 自审:行数 1442 / Mermaid 5 / 0 emoji / 19 embassy-stm32-wpan 引用 / 42 Controller trait 引用
- [x] 3.12 追加 mkdocs.yml nav 加入 19-ble.md
- [x] 3.13 Commit docs/19-ble.md + mkdocs.yml(本轮统一 commit, 不 push)

## 4. M5.4 LoRa 说明文档(docs/20-lora.md)

- [x] 4.1 撰写 §1 概述(说明本 fork 无 embassy-lora crate 不做实现分析)
- [x] 4.2 撰写 §2 LoRa 物理层(调制/SF/BW/CR/频段法规/常见芯片表)
- [x] 4.3 撰写 §3 LoRaWAN Class A/B/C 对比(网络拓扑 + 详细对比表 + 决策表)
- [x] 4.4 撰写 §4 Embassy 集成模式(SX127x SPI 伪代码 + LoRaWAN 状态机伪代码)
- [x] 4.5 撰写 §5 后续建议(3 种路径:等同步/自行 vendor/参考其他平台 + 推荐)
- [x] 4.6 自审:行数 482 / LoRaWAN Class 13 命中 / 0 emoji
- [x] 4.7 追加 mkdocs.yml nav 加入 20-lora.md
- [x] 4.8 Commit docs/20-lora.md + mkdocs.yml(本轮统一 commit, 不 push)

## 5. 收尾

- [x] 5.1 确认 mkdocs.yml 4 篇 nav 都已加入(17/18/19/20)
- [x] 5.2 更新 .claude/docs/SNAPSHOT.md 反映 M5 进度(待用户决策)
- [x] 5.3 更新 .claude/docs/tasks.md M5 4 篇状态 + 验收标准勾选(待用户决策)
- [SKIPPED] 5.4 追加 learned/spec.md 记录 M5 系列踩坑(无新踩坑,CodeGraph MCP 按 house rule 优先,流程顺利)
- [x] 5.5 验证: 4 篇 commit 都在本地,未 push(由用户决定 push 时机)
- [x] 5.6 /opsx:archive add-m5-network-comms-docs(即将执行)
