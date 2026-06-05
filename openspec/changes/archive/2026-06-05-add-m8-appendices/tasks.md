# Tasks: M8 附录文档

> 适用范围: docs/appendix-a-glossary.md, docs/appendix-b-references.md, docs/appendix-c-examples.md
> 依赖: openspec/changes/add-m8-appendices/{proposal,design,specs}/*.md
> 验收基线: 各篇 ~400-600 行 / 0 emoji / 0 Mermaid(附录索引型,非分析型)

## 1. M8.A 术语表(docs/appendix-a-glossary.md,索引型)

- [ ] 1.1 用 grep 提取 M1-M7 加粗术语作为基础词表(`grep -oE "\*\*[^*]+\*\*" docs/*.md | sort -u`)
- [ ] 1.2 整理 A-Z 字母索引,补充 embassy 通用术语(Executor / Future / Waker / WFI / WFE 等)
- [ ] 1.3 撰写文档头部(简介 + A-Z 字母跳转索引)
- [ ] 1.4 撰写 A-G 段术语(异步基础 / Embassy crate / ARM 指令)
- [ ] 1.5 撰写 H-N 段术语(外设 / 工具链 / 协议)
- [ ] 1.6 撰写 O-Z 段术语(操作系统 / 时序 / 看门狗)
- [ ] 1.7 撰写文档尾部(M1-M7 引用位置汇总 + 检索提示)
- [ ] 1.8 自审:行数 400-600 / 术语 ≥ 100 / 0 emoji / ≥ 50% 术语带 Mx.x 节号
- [ ] 1.9 追加 mkdocs.yml nav 加入 appendix-a-glossary.md
- [ ] 1.10 Commit docs/appendix-a-glossary.md + mkdocs.yml(不 push)

## 2. M8.B 参考资料(docs/appendix-b-references.md,索引型)

- [ ] 2.1 收集 5 大类资源(Embassy 官方 / ARM / Rust / 工具链 / 社区)
- [ ] 2.2 撰写文档头部(简介 + 5 大类导航)
- [ ] 2.3 撰写 Embassy 官方类(embassy 仓库 / book / examples / awesome-embassy)
- [ ] 2.4 撰写 ARM & Cortex-M 规范类(ARM ARM / Cortex-M 编程手册 / CMSIS)
- [ ] 2.5 撰写 Rust 资源类(TRPL / Async Book / Embedded Rust Book / std 文档)
- [ ] 2.6 撰写工具链类(probe-rs / defmt / QEMU / cargo-generate / rustup)
- [ ] 2.7 撰写学习社区类(TWiR / Embedded WG / Matrix 频道)
- [ ] 2.8 撰写 M1-M7 引用回标(每条资源后标注相关 Mx.x 节号)
- [ ] 2.9 撰写学习路径建议(Week-by-Week 7 周路径)
- [ ] 2.10 自审:行数 400-600 / 0 emoji / 每条带 URL + 访问日期
- [ ] 2.11 追加 mkdocs.yml nav 加入 appendix-b-references.md
- [ ] 2.12 Commit docs/appendix-b-references.md + mkdocs.yml(不 push)

## 3. M8.C 示例索引(docs/appendix-c-examples.md,索引型)

- [ ] 3.1 用 codegraph_files 列出 embassy/examples/ 所有 bin 文件(已探查,直接用结果)
- [ ] 3.2 按 Mx.x 主题分类:状态机 / Channel / 中断 / 协议 / OTA / 低功耗 / 模式
- [ ] 3.3 撰写文档头部(简介 + 主题 → 平台 → 文件三层导航)
- [ ] 3.4 撰写 M1 主题示例(M1.x 主要为项目结构,引用 embassy 根仓库布局)
- [ ] 3.5 撰写 M2 主题示例(执行器 / 时间 / 同步 / future,30+ bin)
- [ ] 3.6 撰写 M3-M4 主题示例(三平台 HAL 初始化 / GPIO / UART / SPI / I2C / TIM,40+ bin)
- [ ] 3.7 撰写 M5 主题示例(网络 / USB / BLE / LoRa,20+ bin)
- [ ] 3.8 撰写 M6 主题示例(boot / DFU / 低功耗,15+ bin)
- [ ] 3.9 撰写反向索引表(Mx.x → 关键 bin 文件)
- [ ] 3.10 撰写入门路径(8-10 步,每步标注预计耗时)
- [ ] 3.11 自审:行数 400-600 / bin 文件 ≥ 30 / 0 emoji / 每个 bin 标注 embassy API + 模式
- [ ] 3.12 追加 mkdocs.yml nav 加入 appendix-c-examples.md
- [ ] 3.13 Commit docs/appendix-c-examples.md + mkdocs.yml(不 push)

## 4. 收尾

- [ ] 4.1 确认 mkdocs.yml 3 附录 nav 都已加入
- [ ] 4.2 更新 .claude/docs/SNAPSHOT.md 反映 M8 进度(项目最终 100%)
- [ ] 4.3 更新 .claude/docs/tasks.md M8 3 篇状态 + 验收标准勾选
- [ ] 4.4 验证:3 篇 commit 都在本地,未 push(由用户决定 push 时机)
- [ ] 4.5 openspec validate add-m8-appendices
- [ ] 4.6 /opsx:archive add-m8-appendices(全 3 篇完成后)
- [ ] 4.7 学习项目 M1-M8 全部 30 篇文档收官(27 主 + 3 附录)
