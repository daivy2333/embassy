# Tasks: M6 系统组件学习文档

> 适用范围: docs/21-boot.md, docs/22-dfu.md, docs/23-low-power.md
> 受影响 crate: embassy-boot(分析)/ embassy-boot-stm32 / embassy-boot-nrf / embassy-boot-rp(参考)/ embassy-stm32 / embassy-rp / embassy-mcxa 的 executor(参考)
> 依赖: openspec/changes/add-m6-system-components-docs/{proposal,design,specs}/*.md
> 验收基线: 每篇 ≥ 1200 行 / ≥ 1 Mermaid / ≥ 20 源码引用 / 0 emoji

## 1. M6.1 embassy-boot 启动引导(docs/21-boot.md)

- [x] 1.1 CodeGraph 探索 embassy-boot 整体架构(`codegraph_explore` 覆盖 BootLoader/BootLoaderConfig/State/BootError/digest_adapters)
- [x] 1.2 追踪 prepare_boot 主流程(`codegraph_callees prepare_boot` + `codegraph_node read_state/swap/revert/is_swapped/copy_page_once_to_active/copy_page_once_to_dfu/current_progress/update_progress` 全部拉源码)
- [x] 1.3 探索三平台薄壳(`codegraph_files embassy-boot-stm32/src`, `embassy-boot-nrf/src`, `embassy-boot-rp/src` + `codegraph_node` 拉 3 个 prepare/load 实现)
- [x] 1.4 撰写 §1-§3 架构 + 三分区设计 + State 状态机 + Magic 字节协议
- [x] 1.5 撰写 §4 swap/revert 算法详解(progress index 断电续传机制)
- [x] 1.6 撰写 §5 链接器集成(from_linkerfile_blocking + __bootloader_*_start/_end 符号)
- [x] 1.7 撰写 §6 错误处理(BootError enum + assert_partitions)
- [x] 1.8 撰写 §7 平台差异(STM32 最薄 / nRF + softdevice / RP + WatchdogFlash)
- [x] 1.9 撰写 §8-§9 与子系统协作 + 性能资源(flash erase/write cycles)
- [x] 1.10 撰写 §10 实战示例(完整 bootloader main 函数,基于 examples/boot)
- [x] 1.11 撰写 §11 踩坑(PAGE_SIZE 对齐、DFU 至少比 ACTIVE 大 1 个 PAGE)
- [x] 1.12 撰写 §12 平台对比表(3 平台 × 10 维)+ 总结
- [x] 1.13 添加 Mermaid 图(§3 State 状态机 / §4 swap 流程 / §7 平台架构对比)
- [x] 1.14 自审:行数 1211 / 源码引用 25 / Mermaid 4 / 0 emoji
- [x] 1.15 追加 mkdocs.yml nav 加入 21-boot.md
- [ ] 1.16 Commit docs/21-boot.md + mkdocs.yml(不 push)

## 2. M6.2 FirmwareUpdater + DFU 机制(docs/22-dfu.md)

- [x] 2.1 CodeGraph 探索 FirmwareUpdater 整体架构(`codegraph_explore` 覆盖 FirmwareUpdater/FirmwareState/FirmwareUpdaterConfig/FirmwareUpdaterError)
- [x] 2.2 追踪关键方法(`codegraph_node` 拉 write_firmware/mark_updated/mark_booted/mark_dfu/verify_booted/verify_and_mark_updated/prepare_update/set_magic 源码)
- [x] 2.3 探索 digest_adapters(`codegraph_explore digest_adapters ed25519_dalek salty hash verify` + `codegraph_node` 拉 verify/hash 实现)
- [x] 2.4 撰写 §1-§3 架构 + FirmwareUpdater/FirmwareState 分离 + 与 BootLoader 状态分区共享
- [x] 2.5 撰写 §4 完整状态机(write_firmware → mark_updated → reset → bootloader.swap → mark_booted)+ 失败回滚路径
- [x] 2.6 撰写 §5 async + blocking 双 API 对称表(get_state/mark_*/verify_*/write_firmware 全对比)
- [x] 2.7 撰写 §6 错误处理(FirmwareUpdaterError + verify_booted 检查 Boot/DfuDetach/Revert)
- [x] 2.8 撰写 §7 签名验证(verify_and_mark_updated + ed25519_dalek/salty 二选一)
- [x] 2.9 撰写 §8 关键优化(last_erased_dfu_sector_index 去重、prepare_update 一次性 erase)
- [x] 2.10 撰写 §9-§10 跨平台 + 实战示例(完整 OTA 升级流程,基于 examples/boot/application)
- [x] 2.11 撰写 §11 踩坑(WRITE_SIZE 对齐 aligned buffer、verify_booted 拒绝 Swap 状态)
- [x] 2.12 撰写 §12 总结 + 与 USB DFU class 集成入口(DFU_DETACH_MAGIC 协议)
- [x] 2.13 添加 Mermaid 图(§4 完整 OTA 时序图 / §5 双 API 对称结构 / §7 签名验证决策树)
- [x] 2.14 自审:行数 1200 / 源码引用 23 / Mermaid 5 / 0 emoji
- [x] 2.15 追加 mkdocs.yml nav 加入 22-dfu.md
- [ ] 2.16 Commit docs/22-dfu.md + mkdocs.yml(本轮不 commit,M6 全部完成后统一 commit)

## 3. M6.3 低功耗设计模式(docs/23-low-power.md)

- [x] 3.1 CodeGraph 探索 Executor::run 主循环(`codegraph_explore Executor::run wfe do_wfe go_around TASKS_PENDING __pender` 覆盖 mcxa/rp/stm32 三实现)
- [x] 3.2 追踪 mcxa CoreSleep 三档(`codegraph_node CoreSleep deep_sleep_if_possible with_clocks core_sleep` + 完整拉 mcxa executor.rs)
- [x] 3.3 探索 low_power feature(stm32 的 stop_with_rtc / wakeup 源等,`codegraph_explore low_power stop_with_rtc Rtc wakeup`)
- [x] 3.4 撰写 §1-§3 架构 + Executor::run 主循环骨架 + WFE/DSB 双指令对
- [x] 3.5 撰写 §4 TASKS_PENDING + __pender 协议(协程唤醒通路)
- [x] 3.6 撰写 §5 三档功耗模式详解(WfeUngated / WfeGated / DeepSleep,以 mcxa 为主)+ STM32 STOP 子模式
- [x] 3.7 撰写 §6 deep_sleep_if_possible + critical_section 持锁安全性分析
- [x] 3.8 撰写 §7 go_around 优化(SEV pending 检测,避免双 WFE 浪费)
- [x] 3.9 撰写 §8 三平台对比(mcxa 完整三档 / rp 简单 WFE loop / stm32 WFE for ultra low power)
- [x] 3.10 撰写 §9 唤醒源管理(RTC、外部中断、watchdog、UART RX)
- [x] 3.11 撰写 §10 实战示例(完整 low-power task + main loop 节流模式)+ §10.5 测量调优工具
- [x] 3.12 撰写 §11 踩坑(SEV 双唤醒、critical_section 嵌套、deep sleep 时钟切换风险等 15 条)
- [x] 3.13 撰写 §12 平台对比表 + 总结(功耗档位 × 唤醒延迟 × 实现复杂度)
- [x] 3.14 添加 Mermaid 图(§3 主循环流程图 / §4 唤醒时序 / §5 三档状态机 / §6 critical_section 持锁时序 / §12 哲学图)
- [x] 3.15 自审:行数 1201 / 源码引用 20 / Mermaid 5 / 0 emoji
- [x] 3.16 追加 mkdocs.yml nav 加入 23-low-power.md
- [ ] 3.17 Commit docs/23-low-power.md + mkdocs.yml(本轮 M6 全部完成后统一 commit)

## 4. 收尾

- [ ] 4.1 确认 mkdocs.yml 3 篇 nav 都已加入(21/22/23)
- [ ] 4.2 更新 .claude/docs/SNAPSHOT.md 反映 M6 进度
- [ ] 4.3 更新 .claude/docs/tasks.md M6 3 篇状态 + 验收标准勾选
- [ ] 4.4 追加 learned/spec.md 记录 M6 系列踩坑(若有新发现)
- [ ] 4.5 验证: 3 篇 commit 都在本地,未 push(由用户决定 push 时机)
- [ ] 4.6 /opsx:archive add-m6-system-components-docs(全 3 篇完成后)
