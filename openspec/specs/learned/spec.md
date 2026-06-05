## Purpose

记录项目开发过程中学到的知识，避免重复探索，加速问题解决。

## Requirements

### Requirement: API 路径记录

项目中使用的关键 API 路径 SHALL 记录，包含用途和使用示例。

#### Scenario: 发现新 API

- **WHEN** 开发者发现或使用了新的 API 端点
- **THEN** 必须记录到 learned/spec.md，包含：API 路径、用途、请求/响应格式

### Requirement: 踩坑经验记录

遇到的技术陷阱和解决方案 MUST 记录，防止重复踩坑。

#### Scenario: 解决棘手问题

- **WHEN** 开发者花费大量时间解决了一个技术问题
- **THEN** 必须记录踩坑档案，包含：症状、根因、解决方案、预防措施

### Requirement: 技巧模式记录

有效的开发技巧和模式 SHALL 记录，促进知识共享。

#### Scenario: 发现高效做法

- **WHEN** 开发者发现了一种高效的开发技巧或模式
- **THEN** 必须记录到技巧模式区，包含：技巧名称、适用场景、使用方法

### Requirement: 文件速查表

关键文件和目录的位置 SHALL 记录，加速代码导航。

#### Scenario: 定位关键文件

- **WHEN** 开发者频繁访问某些文件或目录
- **THEN** 必须记录到文件速查表，包含：文件路径、用途、关键内容

---

## API 路径速查

### embassy-executor

| API | 用途 | 示例 |
|-----|------|------|
| `Spawner::spawn()` | 异步任务派发 | `spawner.spawn(task_name(arg)).unwrap()` |
| `#[embassy_executor::task]` | 任务宏 | 标记异步函数为 embassy 任务 |
| `#[embassy_executor::main]` | 主函数宏 | 标记入口点 |

### embassy-time

| API | 用途 | 示例 |
|-----|------|------|
| `Timer::after_millis()` | 延时等待 | `Timer::after_millis(100).await` |
| `Instant::now()` | 获取当前时间 | `let now = Instant::now()` |
| `Duration::from_millis()` | 创建时长 | `let d = Duration::from_millis(50)` |

### embassy-nrf (nRF 平台)

| API | 用途 | 示例 |
|-----|------|------|
| `Output::new()` | GPIO 输出 | `Output::new(pin, Level::Low, OutputDrive::Standard)` |
| `Input::new()` | GPIO 输入 | `Input::new(pin, Pull::Up)` |
| `Uarte::new()` | UART 初始化 | 见 examples/nrf52840 |

---

## 踩坑档案

### MkDocs `not_in_nav` vs `exclude_docs` 混淆(2026-06-05)

**症状**:`mkdocs build --strict` 在 GitHub Actions 失败:

```
WARNING - Excluding 'README.md' from the site because it conflicts with 'index.md'.
Aborted with 1 warnings in strict mode!
```

**根因**:
- 配了 `not_in_nav: |\n  /README.md` 想"排除 README.md"
- `not_in_nav` 只控制"不放进侧边栏导航",**不阻止 build**
- MkDocs 仍 build `docs/README.md`,在 `use_directory_urls: true` 下它和 `docs/index.md` **都映射到 `/index.html`**
- MkDocs 检测到 URL 冲突 → 发 WARNING → `--strict` 把 WARNING 当 Error → 失败

**解决**:用 `exclude_docs`(MkDocs 1.5+),build 阶段就剔除文件:

```yaml
# 错误:仍会 build,只是不进导航
not_in_nav: |
  /README.md

# 正确:build 阶段就跳过,不读不渲染
exclude_docs: |
  README.md
```

**预防**:
- `not_in_nav` 适用场景:**文件需要存在且需要被构建**(比如有内部链接指向),但不希望出现在侧栏顶级导航
- `exclude_docs` 适用场景:**文件完全不参与文档站**(比如 README、CHANGELOG、内部草稿)
- 两者 pattern 语法不同:`not_in_nav` 用 gitignore 风格(带前导 `/` 表示根),`exclude_docs` 用 gitignore-style 但相对于 `docs_dir`(`README.md` 即可)
- 一旦开了 `--strict`,任何 WARNING 都会 fail,所以必须用正确 API 而不是"压住报错"

### MkDocs `custom_dir: null` 触发 TypeError(2026-06-05)

**症状**:GitHub Actions 跑 `mkdocs build --strict` 报错:

```
File "mkdocs/config/config_options.py", line 844, in run_validation
    if 'custom_dir' in theme_config and not os.path.isabs(theme_config['custom_dir']):
TypeError: expected str, bytes or os.PathLike object, not NoneType
```

**根因**:`mkdocs.yml` 写了 `theme.custom_dir: null`(本想"显式声明不用自定义模板")。
MkDocs validator 用 `if 'custom_dir' in theme_config` 判断 key 是否存在 — `null` 也算"存在",然后 `os.path.isabs(None)` 直接炸。

**解决**:**删除** `custom_dir: null` 这一行。MkDocs 默认就没这个 key,根本不需要"显式 null"。

**预防**:
- 不要在 `mkdocs.yml` 里写"显式 null"占位(`key: null` / `key: ~` / `key:` 留空均不可)
- "我不用这个特性"= 不写,**不是**写 null
- 这与 Python `dict.get(k, None)` 的容忍语义不同,YAML schema validator 严格区分"key 存在但值是 None" vs "key 不存在"

**适用范围**:同样的雷区可能存在于其它 MkDocs config options(只要 validator 写了 `if 'X' in cfg:` 后直接对值操作)。配置项若官方文档说"optional",意思是"可省略",不是"可写 null"。

---

## 技巧模式

### M3 HAL 三平台设计哲学对比(2026-06-05)

**适用场景**:理解 Embassy 跨平台 HAL 架构、为新平台 HAL 设计借鉴、评估某平台是否适合特定应用。

**M3.1~3.4 串讲后的三平台核心对比**:

| 维度 | stm32 | nrf | rp |
|------|-------|-----|----|
| 设计哲学 | 外设全 + 世代多 + 抽象厚 | 集成度高 + 互联强 + 外设异步原生 | 极简 + PIO 灵活补外设短板 |
| chip 数量 | 800+ | 24 | 3 |
| 元数据机制 | stm32-data 全自动(YAML→build.rs 3042 行) | chips/{chip}.rs 手写(16 文件) | rp-pac 半自动(svd2rust) |
| time-driver feature | 18(底层 2 实现) | 2(rtc1/grtc) | 0(单一 TIMER 硬件) |
| 优先级位 | 4(16 级) | 3(8 级) | 2(4 级,M0+) |
| 外设互联 | 无 | **PPI/DPPI**(外设事件 → 任务,无 CPU) | 无硬件 + **PIO**(可编程 IO 状态机) |
| DMA 抽象 | DMA/BDMA/GPDMA 三栈 + aligned | **EasyDMA**(每外设自带) | 单 DMA(12/16 ch) |
| 集成无线 | 仅 STM32WB | nrf52/53/54L 全集成 BLE/802.15.4 | 无(需外挂 cyw43) |
| 多核 | 仅 H7/WB 未原生集成 | nrf53 双核 + IPC | **原生 spawn_core1** |
| Boot 链 | 简单 | 简单 | rp2040 stage2 + rp235x IMAGE_DEF |
| 硬件原子 | RMW + cs | RMW + cs | **xor/set/clear 寄存器别名映射** |

**三平台独有创新**:

- **nrf PPI/DPPI**:`GPIOTE event → PPI → SAADC task`,μs 级响应 + CPU 全程旁观(本项目 `embassy-nrf/src/ppi/`)
- **rp PIO**:用户编 PIO 汇编实现任意 IO 协议(WS2812/I2S/1-Wire/SDIO),3 PIO × 4 SM(rp235x),配 `pio_programs/` 11 个预制库
- **stm32 stm32-data**:一份源码支持 800+ chip,芯片差异下沉到元数据(本项目 `embassy-stm32/build.rs`)

**关键决策选型**(选哪个平台):

| 应用场景 | 推荐 |
|---------|------|
| 外设种类多(SPI/I2C/CAN/Ethernet 全要) | stm32 |
| BLE / Thread / 802.15.4 应用 | nrf52/53/54L |
| 蜂窝 LTE-M / NB-IoT | nrf9160/9151/9161 |
| 双核并行计算 | rp(原生 spawn_core1)/ nrf53(IPC) |
| 需要不存在的 IO 协议(WS2812、HD44780、HDMI 等) | rp(PIO) |
| 极致低功耗 + RTC 唤醒 | nrf(LFCLK 天然支持) / stm32-L 系列(low-power feature) |
| 最低成本 + Pico 生态 | rp2040 |

**参考**:`docs/08-hal-architecture.md`(M3.1)+ `docs/09-stm32.md`(M3.2)+ `docs/10-nrf.md`(M3.3)+ `docs/11-rp.md`(M3.4)+ ADR-004。

### embassy-stm32 元数据驱动架构(2026-06-05)

**适用场景**:理解 Embassy 如何用一份源码支持 800+ STM32 型号、给其它芯片厂商 HAL 设计参考。

**核心机制(stm32-data 三层链路)**:

```
ST 官方 SVD
  → embassy-rs/stm32-data(YAML 元数据,人工归一化)
  → embassy-rs/stm32-data-generated(Rust 代码,CI 自动)
  → stm32-metapac crate(单 crate + feature 选 chip)
  → embassy-stm32/build.rs(3042 行,读 METADATA 生成 _macros.rs + _generated.rs)
  → embassy-stm32 源码(用 #[cfg(spi_v3)] 等表达版本差异)
```

**关键设计**:

| 决策 | 做法 | 收益 / 代价 |
|------|------|-------------|
| 单 metapac vs 800+ PAC | 单 crate + feature 选型号 | 收益:统一编译路径;代价:metapac crate 大但只编译选中的 |
| 元数据驱动 cfg | build.rs 自动 enable spi_v3/usart_v2 等 | 收益:HAL 源码用 cfg 分支表达差异;代价:cfg 分支多,代码可读性下降 |
| Config 不统一 | 每个时钟世代独立 Config(14 个) | 收益:表达力贴合硬件;代价:换芯片家族要改代码 |
| time-driver 18 feature 但 2 实现 | gp16.rs + lptim.rs + mod.rs 用 cfg_attr path 路由 | 收益:用户感知"选项丰富";代价:无 — 接口被很好抽象 |
| DMA 三栈共存 | DMA / BDMA / GPDMA 接同一 Channel | 收益:用户跨芯片代码不变;代价:Channel 抽象需做最大公约数 |

**stm32 独有 feature(对照 ADR-004 矩阵)**:`aligned`(DMA 缓冲对齐,H7 必需)+ `prio-bits-4`(16 级中断)+ `embedded-hal-nb`(其它 3 平台部分支持) + `low-power`(完整低功耗集成)。

**`bind_interrupts!` 宏不在 hal-internal 的原因**:`$crate` 宏卫生限制 — 在哪个 crate 定义,展开后 `$crate::interrupt::typelevel::Binding` 就指向那个 crate;用户写 `embassy_stm32::bind_interrupts!`,展开需指向 `embassy_stm32::interrupt`,所以每个平台 HAL 自己定义一份(语义一致)。

**vector 折叠**:STM32 部分 chip(F0/G0)有 `I2C2_3` 等共享 vector,`bind_interrupts!` 支持一行注册 N 个 handler:`I2C2_3 => h1, h2, h3, h4;` 展开后按序调用 `on_interrupt()`,每个 handler 自检事件位 early return。

**参考**:`docs/09-stm32.md`(M3.2)+ `embassy-stm32/build.rs` + `embassy-stm32/Cargo.toml`。

### Embassy HAL 跨平台骨架核心符号(2026-06-05)

**适用场景**:研究 Embassy HAL 层架构、添加新平台 HAL 支持、理解 1+N 跨平台抽象。

**核心符号清单(M3.1 探索证据)**:

| 符号 | 路径 | 角色 |
|------|------|------|
| `Peri<'a, T>` | `embassy-hal-internal/src/peripheral.rs:17` | 零成本"借用式外设引用",PhantomData 表达生命周期,不实际持有引用。空间和代码体积都比 `&mut T` 优 |
| `PeripheralType` | `embassy-hal-internal/src/peripheral.rs:107` | marker trait,仅 `Copy + Sized` |
| `interrupt::typelevel::{Interrupt, Handler, Binding}` | `embassy-hal-internal/src/interrupt.rs:38/117/140` | 中断接线三件套,`Binding` 是 unsafe trait,`bind_interrupts!` 宏生成 |
| `InterruptExt` | `embassy-hal-internal/src/interrupt.rs:148` | 运行时 NVIC 操作 trait(enable/disable/priority) |
| `interrupt_mod!` 宏 | `embassy-hal-internal/src/interrupt.rs:11` | 平台 HAL 用它生成 `mod interrupt` |
| `OnDrop` | `embassy-hal-internal/src/drop.rs:7` | RAII guard,可 `defuse()` 拆掉 |
| `Driver` trait | `embassy-time-driver/src/lib.rs:118` | 时间驱动接口:`now()` + `schedule_wake()` |
| `time_driver_impl!` 宏 | `embassy-time-driver/src/lib.rs:161` | 用 `extern "Rust" + no_mangle` 做链接器替换 |
| `SetConfig` / `GetConfig` | `embassy-embedded-hal/src/lib.rs:21/33` | 运行时配置 trait,供 `shared_bus` 使用 |

**`embassy-hal-internal` 公开 API 表面(极简)**:`Peri` + `PeripheralType` + 5 个 mod(`aligned`、`atomic_ring_buffer`、`drop`、`ratio`、`interrupt`)。

**3 平台 embedded-hal 矩阵**:

| crate | stm32 | nrf | rp |
|-------|-------|-----|----|
| `embedded-hal` 0.2 | 是 | 是 | 是 |
| `embedded-hal` 1.0 | 是 | 是 | 是 |
| `embedded-hal-async` 1.0 | 是 | 是 | 是 |
| `embedded-hal-nb` 1.0 | 是 | **否** | 是 |
| `prio-bits-N` | 4 (16级) | 3 (8级) | 2 (4级) |
| `aligned` feature(DMA) | 是 | 否 | 否 |

**参考**:`docs/08-hal-architecture.md`(本项目 M3.1)+ ADR-004(`openspec/specs/architecture/spec.md`)。

### `Driver` trait 链接器替换技巧(2026-06-05)

**适用场景**:实现全局唯一的服务接口,且需要"零或多个实现"在链接阶段就被发现。

**核心做法**:
1. 库定义抽象:`pub trait Driver { fn now(&self) -> u64; }`
2. 库内通过 `extern "Rust" fn _xxx() -> u64;` 声明外部符号
3. 用户用宏 `time_driver_impl!` 提供 `#[unsafe(no_mangle)] fn _xxx() -> u64`
4. 链接器自动连接

**为何这样设计**:
- 全局可达:不需要把 `Driver` 实例通过泛型参数往下传(`embassy-net` 在所有 HAL 上跑就靠这个)
- 全局唯一:0 个实现 → 链接 unresolved,>1 个 → 链接 duplicate
- `Instant` 跨调用可比(多 driver 会让 `now()` 来源不同,比较失去意义)

**代价**:trait 调用变成 `extern "Rust"` 函数调用,失去内联机会。开销在嵌入式可忽略(函数体短 + LTO)。

**参考**:`embassy-time-driver/src/lib.rs:87-103, 141-178` + `docs/08-hal-architecture.md` §6.2。

### MkDocs Material 中文技术文档站(2026-06-05)

**适用场景**:中文 + 大量代码片段 + Mermaid 图 + 表格的技术文档,部署到 GitHub Pages。

**核心配置要点**:

| 维度 | 配置 |
|------|------|
| 主题 | `theme.name: material` + `language: zh` |
| Mermaid 渲染 | `pymdownx.superfences` + `custom_fences: name=mermaid` 一行搞定(Material 9.5+ 内建 Mermaid.js) |
| 中文搜索 | `plugins.search.lang: [zh, en]`(无需 jieba 插件,Material 内建 Lunr 中文分词) |
| 暗色跟随系统 | `palette` 两段配置 + `media: "(prefers-color-scheme: ...)"` |
| 字体 | `theme.font.text: Noto Sans SC`(中文) + `code: JetBrains Mono` |
| 代码复制按钮 | `features: content.code.copy` 一行 |
| 排除文件 | `not_in_nav: \|\n  /README.md`(避免 README.md 和 index.md 双首页冲突) |

**GitHub Pages 部署模式选择(易踩坑)**:

```
错误:Settings → Pages → Source = "Deploy from a branch" + main/docs
  → GitHub 用默认 Jekyll,只渲染 README,看不到导航

正确:Settings → Pages → Source = "GitHub Actions"
  → 让 .github/workflows/docs.yml 控制构建产物,部署到 gh-pages 环境
```

**Workflow 关键设置**:
- `permissions: pages: write + id-token: write`(部署 Pages 必需)
- `concurrency.group: pages` + `cancel-in-progress: false`(避免并发部署冲突)
- `mkdocs build --strict`(broken link 直接失败,质量门)
- `fetch-depth: 0`(若用 git-revision-date 插件需要完整 history)

**预防项**:
1. `.gitignore` 必须加 `site/`(MkDocs build 输出,不要 commit)
2. `Settings → Actions → General → Workflow permissions = Read and write`(否则部署 403)
3. 本地 `python3 -c "yaml.unsafe_load(open('mkdocs.yml'))"` 会因为 `!!python/name:` tag 报错,这是预期(需先装 mkdocs-material),CI 里没问题

**参考来源**:本项目 `mkdocs.yml` + `.github/workflows/docs.yml`(2026-06-05 搭建)。

---

## 文件速查表

| 路径 | 用途 |
|------|------|
| `embassy-executor/src/` | 异步执行器核心 |
| `embassy-time/src/` | 时间管理 |
| `embassy-nrf/src/` | nRF 平台 HAL |
| `embassy-stm32/src/` | STM32 平台 HAL |
| `embassy-rp/src/` | RP2040/235x 平台 HAL |
| `embassy-net/src/` | 网络栈 |
| `embassy-usb/src/` | USB 栈 |
| `examples/` | 示例代码 |
| `tests/` | 集成测试 |
