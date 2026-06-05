# 20. LoRa 远程通信(说明文档)

> 本篇是 **说明性文档**。本 fork 缺失 `embassy-lora` crate,故本篇**不做** LoRa 实现源码分析,仅介绍 LoRa 物理层 / LoRaWAN 概念、Embassy 中的可能集成模式,以及后续推进建议。

---

## 目录

1. 本 fork 为何不做 LoRa 实现分析
2. LoRa 物理层概念
3. LoRaWAN Class A / B / C 对比
4. Embassy 中的可能集成模式
5. 后续推进建议
6. 总结

---

## 1. 本 fork 为何不做 LoRa 实现分析

### 1.1 缺失的 crate

本 fork(`embassy-rs/embassy` 的学习研究项目副本)在 `embassy-net/`, `embassy-usb/`, `embassy-stm32-wpan/` 等 40+ 子 crate 中,**不包含** `embassy-lora` crate(也未在 `Cargo.toml` workspace 列出)。

社区中 `embassy-rs/embassy-lora` 由其他维护者独立管理,提供 Semtech SX1276 / SX126x / LoRaWAN 协议栈支持。本 fork 未 vendor 该 crate。

### 1.2 影响

- 无 LoRa 源码可分析
- 无 LoRa 任务示例可参考
- 强行写 LoRa 实现分析会变成"空谈"或"参考未集成的 crate"

### 1.3 本篇目的

1. **入门 LoRa 概念**:为后续真要写 LoRa 应用的用户提供协议入门
2. **集成模式参考**:用伪代码展示 Embassy 中集成 LoRa 驱动的模式(与 SX127x 通信的 SPI 任务 + LoRaWAN 状态机任务)
3. **后续路径建议**:3 种可行路径(等上游同步 / 自行 vendor / 参考其他平台)

### 1.4 本 fork 的范围声明

本 fork 后续如需深化 LoRa 学习,推荐路径:
- 等待 upstream `embassy-rs/embassy` 合并 `embassy-lora` 子 workspace
- 自行 vendor `embassy-rs/embassy-lora` 到本 fork
- 参考 stm32 / esp32 / nrf 的 LoRa 应用层代码

---

## 2. LoRa 物理层概念

LoRa(Long Range)是 Semtech 公司的**扩频调制技术**,工作于 sub-GHz 频段(433 / 868 / 915 MHz)。LoRaWAN 是基于 LoRa 调制的 MAC 层协议。

### 2.1 物理层参数

| 参数 | 范围 | 含义 | 权衡 |
|------|------|------|------|
| 扩频因子(SF) | 6-12 | 每 bit 用 2^SF 个 chirp 表示 | 大 SF → 高灵敏度 + 低速率 + 长距离 |
| 带宽(BW) | 7.8-500 kHz | 占用频谱宽度 | 大 BW → 高速率 + 低灵敏度 + 短距离 |
| 编码率(CR) | 1 / 2 / 3 / 4 | 前向纠错冗余 | 大 CR → 高抗干扰 + 低速率 |

**灵敏度** ≈ -137 dBm(SF=12, BW=125 kHz),可达 10+ km 距离(空旷环境)。

**数据速率** ≈ 0.3 - 50 kbps(由 SF / BW / CR 组合决定)。

### 2.2 LoRa 调制原理(简述)

LoRa 用 **Chirp Spread Spectrum(CSS)** 扩频:
- 1 bit 信息 = 多个 chirp 频率(线性扫频)
- chirp 数量 = 2^SF
- 解调靠 chirp 起点相位差
- 抗干扰能力强(频率选择性衰落 + 多径效应不敏感)

**与 FSK / OOK 对比**:
- FSK:功耗低,距离短(~100 m)
- OOK:最简单,距离短
- LoRa:扩频,距离长,功耗适中

### 2.3 频段与地区法规

| 地区 | ISM 频段 | 占空比限制 | 输出功率限制 |
|------|----------|------------|--------------|
| 欧洲(ETSI) | 868 MHz | 1% (Duty Cycle) | +14 dBm |
| 美国(FCC) | 915 MHz | 无硬限制 | +20 dBm |
| 中国 | 470-510 MHz / 779-787 MHz | 10% | +17 dBm |
| 亚洲(部分) | 433 MHz | 1-10% | +10 dBm |

**关键**:不同地区法规不同,LoRaWAN 协议栈会根据地区自动调整占空比与最大 EIRP。

### 2.4 常见 LoRa 芯片

| 芯片 | 厂商 | 频段 | 调制 | 备注 |
|------|------|------|------|------|
| SX1276 | Semtech | 137-1020 MHz | LoRa + FSK + OOK | 经典款 |
| SX1277 | Semtech | 137-1020 MHz | LoRa + FSK + OOK | 较新 |
| SX1278 | Semtech | 137-525 MHz | LoRa + FSK + OOK | 433/470 MHz |
| SX1261 | Semtech | 150-960 MHz | LoRa + (G)FSK | 低功耗 |
| SX1262 | Semtech | 150-960 MHz | LoRa + (G)FSK | 主流 |
| SX1268 | Semtech | 410-810 MHz | LoRa + (G)FSK | 中国频段 |
| LLCC68 | Semtech | 150-960 MHz | LoRa | 廉价 |
| RA-01 | Ai-Thinker | 410-525 MHz | LoRa | 基于 SX1278 |
| RA-02 | Ai-Thinker | 868 / 915 MHz | LoRa | 基于 SX1276 |

### 2.5 链路预算示例

假设发射功率 +14 dBm,接收灵敏度 -137 dBm:

```
链路预算 = +14 - (-137) = 151 dB
```

LoRa 路径损耗模型(空旷室外):
- 自由空间损耗(dB) = 20 log10(d_km) + 20 log10(f_MHz) + 32.4
- 868 MHz, 1 km:约 91 dB 损耗
- 868 MHz, 10 km:约 111 dB 损耗
- 868 MHz, 15 km:约 116 dB 损耗(余 35 dB 余量)

实际距离受地形 / 阻挡 / 天线影响很大。城市环境可能只有 1-3 km。

---

## 3. LoRaWAN Class A / B / C 对比

LoRaWAN 是 LoRa Alliance 制定的 MAC 层协议,定义设备 Class 与网络架构。

### 3.1 网络拓扑

```
LoRaWAN Network:
  ├─ End Device(传感器 / 标签,Class A/B/C)
  ├─ Gateway(LoRaWAN 集中器,8+ 通道同时接收)
  │    └─ Network Server(LoRaWAN 服务器)
  │         └─ Application Server(应用数据)
```

**End Device ↔ Gateway**:LoRa 无线
**Gateway ↔ Network Server**:IP 网络(以太网 / WiFi / 蜂窝)
**Network Server ↔ Application Server**:IP 网络

### 3.2 3 种 Class 对比

| 维度 | Class A | Class B | Class C |
|------|---------|---------|---------|
| 接收窗口 | 2 个(上行后 1s / 2s) | 同步信标 + 周期窗口 | 持续(除发射时) |
| 下行延迟 | 1-3 s(无 beacon) | < 4 s | < 1 s |
| 功耗 | 极低 | 中(需 beacon sync) | 高(常开) |
| 应用 | 电池传感器(1 Hz 上报) | 智能路灯 / 智能农业 | 实时控制 / 智能家居 |
| 复杂度 | 最简单 | 中(需网络同步) | 简单(除发射外常开) |

### 3.3 Class A(必选,最常用)

Class A 设备:
- 每次上行(Uplink)后开 2 个接收窗口(RX1 / RX2)
- RX1 紧接上行后 ~1 秒(频率与上行同)
- RX2 紧接 RX1 后 ~1 秒(频率固定,默认 869.525 MHz EU)
- 其他时间深度睡眠

**典型应用**:
- 温湿度传感器
- 烟雾报警器
- 智能水表 / 电表
- 农业土壤传感器

**优势**:最省电(可工作 5-10 年于 AA 电池)
**劣势**:下行命令有 1-3 秒延迟

### 3.4 Class B(同步信标)

Class B 设备:
- 在 Class A 基础上增加**同步信标**(beacon)接收
- Gateway 周期性(每 128 s)发 beacon,设备同步
- 设备在 beacon 间隙打开 ping slot 接收下行
- 下行延迟可控 < 4 s

**实现要求**:
- Network Server 需支持 beacon scheduling
- Gateway 需 GPS 同步(beacon 帧有时间戳)
- 设备需保留 ~5% 接收时间(ping slot 数量决定)

**典型应用**:
- 智能路灯(集中控制开关 + 调光)
- 智能农业阀门
- 城市停车传感器

### 3.5 Class C(持续接收)

Class C 设备:
- 除发射时,持续打开接收窗口
- 下行延迟 < 1 s
- 功耗高(通常 mains-powered)

**实现要求**:
- RX2 窗口持续(直到下次发射)
- 发射时关闭接收(避免自干扰)
- 不需要 beacon sync

**典型应用**:
- 智能家居(智能锁、插座)
- 实时工业控制
- 路灯控制(无电池限制)

### 3.6 Class 选择决策表

| 应用 | 电池供电 | 下行延迟需求 | 推荐 Class |
|------|----------|--------------|-----------|
| 温湿度传感器(1/小时) | AA / 锂电池 | 无所谓 | A |
| 烟雾报警 | 长寿命电池 | < 10 s | A |
| 智能门锁 | 锂电池 | < 1 s | C |
| 智能路灯 | mains | < 4 s | B / C |
| 农业阀门 | 锂电池 | < 4 s | B |
| 实时工业传感器 | mains | < 1 s | C |
| 宠物追踪 | 锂电池 | 无所谓 | A |

---

## 4. Embassy 中的可能集成模式

虽然本 fork 无 `embassy-lora` crate,但可以基于 Embassy 现有的 SPI / executor 抽象推测集成模式。

### 4.1 硬件连接(典型 SX127x + STM32)

```
MCU (STM32)          SX127x
  ├── SPI_MOSI ──────→ MOSI
  ├── SPI_MISO ←────── MISO
  ├── SPI_SCK ───────→ SCK
  ├── CS   ──────────→ NSS
  ├── INT  ←────────── DIO0 (TX done / RX done)
  ├── RST  ──────────→ RESET
  └── VCC             3.3V
```

SPI 速率:典型 1-10 MHz,SX1276 上限 10 MHz。

### 4.2 伪代码:SX127x 驱动 + LoRaWAN 任务

```rust
// 伪代码 - 非可运行代码,展示 Embassy 集成模式

use embassy_executor::Spawner;
use embassy_stm32::spi::{Config, Spi};
use embassy_stm32::gpio::{Input, Output, Level, Pull, Speed};
use embassy_time::Duration;

// SX127x 驱动 crate(假设)
use sx127x_drv::{Sx127x, LoraConfig, OpMode};

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    let p = embassy_stm32::init(Default::default());

    // 1) 初始化 SPI
    let spi_config = Config::default();
    let spi = Spi::new(
        p.SPI1, p.PA5, p.PA6, p.PA7, p.PA4,
        p.DMA1_CH3, p.DMA1_CH2,
        spi_config,
    );

    // 2) 初始化控制引脚
    let cs = Output::new(p.PA4, Level::High, Speed::VeryHigh);
    let reset = Output::new(p.PA0, Level::High, Speed::Low);
    let dio0 = Input::new(p.PA1, Pull::Down);

    // 3) 初始化 SX127x
    let mut lora = Sx127x::new(spi, cs, reset, dio0).await.unwrap();
    lora.set_opmode(OpMode::LongRangeMode).unwrap();

    let lora_config = LoraConfig {
        frequency: 868_100_000,  // 868.1 MHz EU
        bandwidth: Bandwidth::Bw125kHz,
        spreading_factor: SpreadingFactor::Sf7,
        coding_rate: CodingRate::Cr45,
        tx_power: 14,  // dBm
    };
    lora.apply_config(&lora_config).unwrap();

    // 4) 启动 LoRaWAN 任务(假设有 lorawan crate)
    spawner.spawn(lora_rx_task(lora)).unwrap();
    spawner.spawn(lora_tx_task(...)).unwrap();
    spawner.spawn(lorawan_state_machine(...)).unwrap();
}

#[embassy_executor::task]
async fn lora_tx_task(...) {
    let mut counter = 0u32;
    loop {
        // 1) 周期性上报数据
        embassy_time::Timer::after(Duration::from_secs(60)).await;

        // 2) 构造上行 payload(温度、湿度等)
        let payload = encode_sensor_data(read_sensors());

        // 3) 发送到 SX127x
        lora.transmit(&payload).await.unwrap();
        counter += 1;
    }
}

#[embassy_executor::task]
async fn lora_rx_task(...) {
    loop {
        // 1) 等待 DIO0 中断(SX127x TX done / RX done)
        dio0.wait_for_high().await;

        // 2) 检查中断源
        let flags = lora.read_irq_flags().unwrap();
        if flags.rx_done() {
            // 3) 读取 RX FIFO
            let len = lora.get_packet_length().unwrap();
            let mut buf = [0u8; 255];
            lora.read_fifo(&mut buf[..len]).unwrap();
            // 4) 处理下行(LoRaWAN 命令)
            handle_downlink(&buf[..len]);
        }
        // 5) 清中断
        lora.clear_irq_flags(flags).unwrap();
    }
}
```

### 4.3 典型 Embassy 抽象层

LoRa 集成需要 Embassy 提供的:
- `embassy-time::Timer` / `Duration` — 周期上报
- `embassy-time::Instant` — 时间戳(LoRaWAN 需要)
- `embassy-stm32::spi` — SX127x 通信
- `embassy-stm32::gpio` — CS / RST / DIO0 控制
- `embassy-executor` — 多任务调度
- `embassy-sync::Channel` — RX 任务到 LoRaWAN 任务传递下行包

### 4.4 LoRaWAN 状态机任务(伪代码)

```rust
// 伪代码 - LoRaWAN 状态机(简化)
#[embassy_executor::task]
async fn lorawan_state_machine() {
    let mut state = LorawanState::Joined;
    let mut uplink_seq: u16 = 0;
    loop {
        match state {
            LorawanState::Joined => {
                // 1) 周期上行
                Timer::after(Duration::from_secs(60)).await;
                let payload = read_sensors();
                let mac = MacPayload { f_port: 1, frm_payload: payload, f_cnt: uplink_seq };
                lorawan_send_uplink(&mac).await;
                uplink_seq += 1;
                // 2) 等待 RX1 / RX2 窗口
                Timer::after(Duration::from_millis(1000)).await;
                let rx1 = lorawan_check_rx1().await;
                if rx1.is_none() {
                    Timer::after(Duration::from_millis(1000)).await;
                    let rx2 = lorawan_check_rx2().await;
                    if let Some(downlink) = rx2 {
                        handle_downlink(downlink);
                    }
                } else if let Some(downlink) = rx1 {
                    handle_downlink(downlink);
                }
            }
            LorawanState::NotJoined => {
                // 触发 Join procedure
                lorawan_join().await;
                state = LorawanState::Joined;
            }
        }
    }
}
```

### 4.5 Embassy 集成挑战

- **精确时序**:LoRaWAN 的 RX1 / RX2 窗口要求毫秒级精度,embassy-time 可满足
- **中断响应**:SX127x DIO0 触发后需在数十 us 内读 RX FIFO,embassy-stm32 EXTI 满足
- **状态机共享**:LoRaWAN 状态需要跨任务共享,embassy-sync::Mutex / Channel 适用
- **无 dynamic memory**:LoRaWAN 内部 MAC 状态需静态分配,LoRaWAN crate 通常支持
- **OTA**:LoRaWAN 支持 FUOTA(Firmware Update Over The Air),需 Flash 写保护

### 4.6 已知的 embassy-lora 实现

虽然本 fork 不含,但社区有:

- `embassy-rs/embassy-lora`:基于 Semtech SX127x 驱动 + LoraWan Rust 实现
- `lora-rs/lora-rs`:LoRaWAN 协议栈(纯 Rust)
- `lora-device`:LoRa 设备抽象 trait

集成模式大同小异,都是 SPI 驱动 + 异步任务 + 状态机。

---

## 5. 后续推进建议

如需在本 fork 深化 LoRa 学习,有 3 种可行路径。

### 5.1 路径 A:等 upstream 同步

**步骤**:
1. 关注 `embassy-rs/embassy` 仓库 PR / Issue 中关于 `embassy-lora` 的讨论
2. 等待 embassy 主仓库合并 LoRa workspace 成员
3. Fork 同步 upstream(本 fork 是 Embassy 主仓库的 fork)
4. 拉取后开始 LoRa 源码分析

**优点**:
- 与社区代码一致
- 维护负担小
- 自动获得后续更新

**缺点**:
- 时机不可控
- upstream 可能不合并(若 embassy-lora 继续独立维护)

### 5.2 路径 B:自行 vendor

**步骤**:
1. 复制 `embassy-rs/embassy-lora` 源码到本 fork 的 `embassy-lora/` 目录
2. 在 root `Cargo.toml` workspace 成员中加入
3. 修复可能的 build 错误(API 兼容性)
4. 写 docs/20-lora.md 实现分析章节(替代本说明文档)

**优点**:
- 立即可用
- 完全控制代码
- 适配本 fork 的代码风格

**缺点**:
- 维护负担(需要手动 sync upstream 更新)
- 失去与社区同步

### 5.3 路径 C:参考其他平台实现

**步骤**:
1. 学习 stm32 / esp32 / nrf 上的 LoRa 应用(github 上搜索 `stm32 lora rust` 等)
2. 不直接使用 embassy-lora,而是学习 LoRa 协议 + Embassy 集成模式
3. 写"协议+集成模式"的中性文档(不深入某 crate)

**优点**:
- 无 vendor 依赖
- 协议知识中立(可迁移到其他 LoRa crate)

**缺点**:
- 无具体代码可分析
- 文档偏概念性

### 5.4 推荐路径

短期:路径 C(本篇就是这种做法,中性概念文档)
中期:路径 A(等 upstream 合并)
长期:路径 B(若本 fork 持续学习 LoRa,自行 vendor)

### 5.5 优先级建议

如要继续 M5 之外的工作,推荐先完成:
1. M5.1-5.4 commit + push(已完成 3 篇)
2. M6 embassy-boot 启动引导(2026-06-05 之后)
3. M5.4 LoRa 等路径 A / B 触发后,扩展本篇为完整实现分析

---

## 6. 总结

本 fork 不含 `embassy-lora` crate,故本篇**不进行** LoRa 源码分析,仅作为说明性文档:

1. 解释本 fork 不做实现分析的原因
2. 介绍 LoRa 物理层参数(SF / BW / CR)
3. 对比 LoRaWAN Class A / B / C(应用 + 功耗)
4. 展示 Embassy 中可能的集成模式(SPI + GPIO + 异步任务)
5. 给出 3 种后续推进路径(等同步 / vendor / 概念)

如需深化,推荐路径 A → B(等 upstream,再 vendor)。本篇作为入门 + 决策依据,后续若切换到完整实现分析,可在本篇基础上扩展。

---

## 附:本篇章节索引

1. 本 fork 为何不做 LoRa 实现分析(缺失 crate / 范围声明 / 后续入口)
2. LoRa 物理层概念(参数 / 调制 / 频段法规 / 常见芯片)
3. LoRaWAN Class A / B / C 对比(网络拓扑 + 3 Class 详细 + 决策表)
4. Embassy 中的可能集成模式(硬件 / 伪代码 / 状态机)
5. 后续推进建议(3 路径 + 推荐 + 优先级)
6. 总结

---

**文档结束。本 fork 后续若需完整 LoRa 实现分析,建议以本篇为入口,补充 docs/20-lora-impl.md 或扩展本篇。**
