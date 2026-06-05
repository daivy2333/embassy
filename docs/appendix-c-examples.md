# 附录 C - Embassy 仓库示例索引

> 版本: v0.1.0
> 最后更新: 2026-06-05
> 适用项目版本: Embassy `0.5+`
> 索引范围: `embassy/examples/` 目录下 30+ `src/bin/*.rs` 文件
> 路径前缀: `examples/<platform>/src/bin/<name>.rs`(基于 embassy 仓库根目录)

---

## 简介

Embassy 仓库 `examples/` 目录包含 30+ 平台 × 多种外设的可运行示例,是学习 embassy 的"实践宝库"。本附录按"主题(Mx.x 节号) → 平台 → 文件路径"三层组织,帮助读者快速定位"想找某类功能的示例"。

每个 bin 文件标注:

- **路径**:在 embassy 仓库中的相对位置
- **关键 API**:使用的 embassy-* crate 与 trait
- **关键外设**:涉及的 MCU 外设(GPIO / UART / SPI / I2C / TIM 等)
- **模式**:最适合学习的 Embassy 异步编程模式(M7.4 六大模式之一)
- **Mx.x 引用**:在 M1-M7 哪一篇文档被详细分析

---

## 三层导航

| 主题 | 平台 | 关键 bin |
|------|------|----------|
| 入门闪灯 | STM32 / nRF / RP | [blinky 系列](#入门闪灯) |
| 异步执行器 | nRF / RP | [multiprio / raw_spawn](#异步执行器) |
| 同步原语 | nRF / RP / STM32 | [channel / sharing](#同步原语) |
| GPIO 与中断 | STM32 / nRF / RP | [button 系列](#gpio-与中断) |
| UART | STM32 / nRF / RP | [usart / uart 系列](#uart) |
| SPI | STM32 / RP | [spi 系列](#spi) |
| I2C | STM32 / nRF | [i2c 系列](#i2c) |
| 定时器 / PWM | STM32 / nRF / RP | [timer / pwm / pwm_servo](#定时器--pwm) |
| ADC | STM32 / nRF / RP | [adc / saadc](#adc) |
| USB | STM32 / nRF / RP | [usb_hid_keyboard / usb_serial](#usb) |
| 网络 | RP | [ethernet_w5500 系列](#网络) |
| BLE / IEEE 802.15.4 | nRF | [ieee802154 / temp](#ble--ieee-802154) |
| 看门狗 | STM32 / nRF / RP | [wdt / wwdt](#看门狗) |
| Boot / DFU | STM32 / nRF / RP | [boot 系列](#boot--dfu) |
| 低功耗 | nRF / mcxa | [qspi_lowpower / power-deepsleep](#低功耗) |
| PIO(RP 特有) | RP | [pio 系列](#pio-rp-特有) |

---

## 入门闪灯

最简示例,适合 embassy 第一次跑通。

### STM32F4

- **路径**: `examples/stm32f4/src/bin/blinky.rs`
- **关键 API**: `embassy_stm32::gpio::Output` + `embassy_time::Timer`
- **关键外设**: GPIO(PA5,Nucleo 板载 LED)
- **模式**: 模式 1(状态机 Timer 周期)+ 模式 5(低功耗调度)
- **Mx.x 引用**: M7.1 §6(实战示例完整闪灯代码)
- **预计耗时**: 30 分钟跑通

### nRF52840

- **路径**: `examples/nrf52840/src/bin/blinky.rs`
- **关键 API**: `embassy_nrf::gpio::Output`
- **关键外设**: GPIO(P0_13)
- **模式**: 同上
- **Mx.x 引用**: M7.1 §6.7 跨平台等价示例

### RP2040

- **路径**: `examples/rp/src/bin/blinky.rs`
- **关键 API**: `embassy_rp::gpio::Output`
- **关键外设**: GPIO(PIN_25,Pico 板载 LED)
- **模式**: 同上
- **Mx.x 引用**: M7.1 §6.7 跨平台等价示例

---

## 异步执行器

展示 `embassy_executor` 的多任务调度。

### nRF / RP 多优先级

- **路径**:
  - `examples/nrf52840/src/bin/multiprio.rs`
  - `examples/rp/src/bin/multiprio.rs`
- **关键 API**: `embassy_executor::Spawner` + `select_biased!`
- **关键概念**: 多优先级任务 + `InterruptExecutor`(M2.1 §3)
- **模式**: 模式 1(状态机 + select_biased)
- **Mx.x 引用**: M2.1 §3 主循环 + M7.4 §2 select_biased
- **预计耗时**: 1 小时读懂

### nRF 手动创建 Executor

- **路径**: `examples/nrf52840/src/bin/manually_create_executor.rs`
- **关键 API**: `embassy_executor::raw::Executor` 手动 API
- **关键概念**: 不依赖宏的 Executor 初始化
- **模式**: 模式 5(低功耗 Executor::run)
- **Mx.x 引用**: M2.1 §4 内部机制
- **预计耗时**: 1.5 小时

### nRF / RP 静态 spawn

- **路径**:
  - `examples/nrf52840/src/bin/raw_spawn.rs`
  - `examples/rp/src/bin/raw_spawn.rs`
- **关键 API**: `Executor::spawn` 手动 spawn(非 `#[embassy_executor::task]` 宏)
- **关键概念**: Embassy task 的另一种启动方式
- **模式**: 模式 2 / 3(同步原语)
- **Mx.x 引用**: M2.1 §3 任务管理
- **预计耗时**: 1 小时

### RP 共享资源

- **路径**: `examples/rp/src/bin/sharing.rs`
- **关键 API**: `embassy_sync::Mutex` + 临界区
- **关键概念**: 任务间共享资源
- **模式**: 模式 2(产消)+ 模式 6(RAII MutexGuard)
- **Mx.x 引用**: M2.3 §8 Mutex
- **预计耗时**: 1.5 小时

### nRF 通道发送接收

- **路径**: `examples/nrf52840/src/bin/channel_sender_receiver.rs`
- **关键 API**: `embassy_sync::channel::Channel`
- **关键概念**: 单生产者单消费者(SPSC)通道
- **模式**: 模式 2(生产者-消费者)
- **Mx.x 引用**: M2.3 §4 Channel + M7.4 §3
- **预计耗时**: 1 小时

---

## 同步原语

### 按钮多任务(共享资源)

- **路径**:
  - `examples/stm32f0/src/bin/multiprio.rs`
  - `examples/stm32c0/src/bin/button.rs`
- **关键 API**: `embassy_executor::Spawner` + `embassy_stm32::gpio::Input`
- **关键概念**: 多任务监听同一按钮
- **模式**: 模式 3(发布-订阅)简化版
- **Mx.x 引用**: M2.3 + M4.1 GPIO 中断

---

## GPIO 与中断

### STM32F4 按键

- **路径**: `examples/stm32f4/src/bin/button.rs`(M3.2 引用)
- **关键 API**: `embassy_stm32::gpio::Input` + `wait_for_low()`
- **关键外设**: GPIO + EXTI
- **模式**: 模式 1(状态机)
- **Mx.x 引用**: M4.1 §6 输入模式

### nRF 按键(异步)

- **路径**: `examples/mcxa2xx/src/bin/button_async.rs` / `button.rs`
- **关键 API**: `Input` 异步 wait
- **Mx.x 引用**: M4.1

---

## UART

### STM32F4 异步 UART

- **路径**: `examples/stm32f4/src/bin/usart_async.rs`
- **关键 API**: `embassy_stm32::usart::Uart::new_async`
- **关键外设**: USART + DMA
- **模式**: 模式 2(产消 via DMA 通道)
- **Mx.x 引用**: M4.2 §5 异步驱动
- **预计耗时**: 2 小时

### STM32F4 阻塞 UART

- **路径**: `examples/stm32f4/src/bin/usart_blocking.rs`
- **关键 API**: `Uart::new_blocking`
- **Mx.x 引用**: M4.2 §4 阻塞模式
- **对比**: 与 async 版本对比性能

### STM32F4 缓冲 UART

- **路径**: `examples/stm32f4/src/bin/usart_buffered.rs`
- **关键 API**: `BufferedUart`
- **关键概念**: 软件 FIFO 缓冲
- **Mx.x 引用**: M4.2 §6 缓冲模式

### nRF52840 USB 串口(多任务)

- **路径**: `examples/nrf52840/src/bin/usb_serial_multitask.rs`
- **关键 API**: `embassy_usb` + 多任务 spawn
- **关键概念**: USB CDC + 多任务(读 + 写)
- **Mx.x 引用**: M4.2 + M5.2 USB
- **预计耗时**: 2 小时

### RP Pico UART

- **路径**: `examples/rp/src/bin/uart_buffered.rs`
- **关键 API**: `embassy_rp::uart`
- **Mx.x 引用**: M4.2

---

## SPI

### STM32F4 DMA SPI

- **路径**: `examples/stm32f4/src/bin/spi_dma.rs`
- **关键 API**: `embassy_stm32::spi::Spi::new` + DMA
- **关键外设**: SPI + DMA 双工
- **模式**: 模式 2(DMA 通道)
- **Mx.x 引用**: M4.3 §4-5 异步 + DMA
- **预计耗时**: 1.5 小时

### RP2040 异步 SPI

- **路径**: `examples/rp/src/bin/spi_async.rs`
- **关键 API**: `embassy_rp::spi::Spi`
- **关键外设**: SPI0 / SPI1
- **Mx.x 引用**: M4.3 §5

### RP2040 SPI + SD 卡

- **路径**: `examples/rp/src/bin/spi_sdmmc.rs`
- **关键 API**: SPI + embedded-sdmmc
- **关键概念**: 文件系统
- **Mx.x 引用**: M4.3 + 第三方 sdmmc crate

### RP2040 SPI + GC9A01(圆形 LCD)

- **路径**: `examples/rp/src/bin/spi_gc9a01.rs`
- **关键 API**: SPI + display 接口
- **Mx.x 引用**: M4.3 + 第三方 display crate

### STM32F4 异步 flash

- **路径**: `examples/stm32f4/src/bin/flash_async.rs`
- **关键 API**: `embedded-storage` async flash
- **Mx.x 引用**: M6.1 §3 链接脚本 + flash trait

---

## I2C

### STM32F4 异步 I2C

- **路径**:
  - `examples/stm32f4/src/bin/i2c_slave_async.rs`
  - `examples/stm32f4/src/bin/i2c_comparison.rs`
- **关键 API**: `embassy_stm32::i2c::I2c::new_async`
- **关键外设**: I2C + DMA
- **Mx.x 引用**: M4.4 §5
- **预计耗时**: 1.5 小时

### STM32F4 阻塞 I2C 从机

- **路径**: `examples/stm32f4/src/bin/i2c_slave_blocking.rs`
- **关键概念**: I2C slave 模式
- **Mx.x 引用**: M4.4

### nRF TWIM(Two-Wire Master)

- **路径**: `examples/nrf52840/src/bin/twim.rs`
- **关键 API**: `embassy_nrf::twim::Twim`
- **关键外设**: nRF 特有 TWIM(TWI Master)
- **Mx.x 引用**: M4.4 + M3.3 nRF EasyDMA
- **预计耗时**: 1.5 小时

### STM32F4 I2S DMA 全双工

- **路径**: `examples/stm32f4/src/bin/i2s_dma_full_duplex.rs`
- **关键 API**: I2S(音频协议)+ DMA
- **Mx.x 引用**: M4.4 协议扩展

### STM32F4 CAN 总线

- **路径**: `examples/stm32f4/src/bin/can.rs`
- **关键 API**: `embassy_stm32::can::Can`
- **关键概念**: CAN 总线收发
- **Mx.x 引用**: 协议扩展(非 M4.4 主线)

---

## 定时器 / PWM

### STM32F4 PWM 输入捕获

- **路径**: `examples/stm32f4/src/bin/pwm_input.rs`
- **关键 API**: `embassy_stm32::timer::input_capture`
- **关键外设**: TIM 输入捕获模式
- **Mx.x 引用**: M4.5 §5 输入捕获

### STM32F4 输入捕获分离通道

- **路径**: `examples/stm32f4/src/bin/input_capture_split.rs`
- **关键 API**: 分离通道输入捕获
- **Mx.x 引用**: M4.5 §5

### nRF52840 PWM(双序列)

- **路径**: `examples/nrf52840/src/bin/pwm_double_sequence.rs`
- **关键 API**: `embassy_nrf::pwm::Pwm`
- **关键外设**: PWM 外设
- **Mx.x 引用**: M4.5 §4 PWM

### nRF52840 PWM(PPI 序列)

- **路径**: `examples/nrf52840/src/bin/pwm_sequence_ppi.rs`
- **关键 API**: PWM + PPI 联动
- **关键概念**: PPI 触发 PWM 序列(无 CPU 介入)
- **Mx.x 引用**: M4.5 + M3.3 PPI
- **预计耗时**: 2 小时

### nRF52840 PWM 舵机

- **路径**: `examples/nrf52840/src/bin/pwm_servo.rs`
- **关键 API**: PWM + 高电平脉宽
- **关键概念**: 舵机控制(50Hz,1-2ms 高电平)
- **Mx.x 引用**: M4.5 §4 实战

### STM32F4 看门狗

- **路径**: `examples/stm32f4/src/bin/wdt.rs`
- **关键 API**: `embassy_stm32::wdg::IndependentWatchDog`
- **关键概念**: 独立看门狗(IWDG)
- **模式**: 模式 4(Watchdog)
- **Mx.x 引用**: M6.3 + M7.4 §5
- **预计耗时**: 30 分钟

---

## ADC

### STM32F4 ADC + DMA

- **路径**: `examples/stm32f4/src/bin/adc_dma.rs`
- **关键 API**: `embassy_stm32::adc::Adc` + DMA
- **关键外设**: ADC1
- **Mx.x 引用**: M4.1 §7 ADC 简述

### nRF52840 SAADC

- **路径**: `examples/nrf52840/src/bin/saadc.rs`
- **关键 API**: `embassy_nrf::saadc::Saadc`
- **关键外设**: nRF Successive Approximation ADC
- **Mx.x 引用**: M3.3 nRF SAADC

### RP2040 ADC + DMA

- **路径**: `examples/rp/src/bin/adc_dma.rs`
- **关键 API**: `embassy_rp::adc::Adc`
- **Mx.x 引用**: M3.4

---

## USB

### STM32F4 USB HID 键盘

- **路径**:
  - `examples/stm32f4/src/bin/usb_hid_keyboard.rs`
  - `examples/nrf52840/src/bin/usb_hid_keyboard.rs`
  - `examples/rp/src/bin/usb_hid_keyboard.rs`
- **关键 API**: `embassy_usb` + HID class
- **关键概念**: USB HID 设备类
- **Mx.x 引用**: M5.2 §5 HID
- **预计耗时**: 3 小时
- **三平台对照**:同一 USB HID 跨三平台,展示 embassy-usb 抽象

---

## 网络

### RP2040 以太网 W5500(ICMP ping)

- **路径**: `examples/rp/src/bin/ethernet_w5500_icmp_ping.rs`
- **关键 API**: `embassy_net` + W5500 SPI 网卡
- **关键概念**: TCP/IP 协议栈 + 简易硬件网卡
- **Mx.x 引用**: M5.1 §3 smoltcp 集成
- **预计耗时**: 4 小时

### RP2040 以太网 W5500(UDP)

- **路径**: `examples/rp/src/bin/ethernet_w5500_udp.rs`
- **关键 API**: `embassy_net` UDP socket
- **关键概念**: UDP 收发
- **Mx.x 引用**: M5.1 §4 UDP

### RP2040 以太网 W5500(TCP)

- **路径**: `examples/rp/src/bin/ethernet_w5500_tcp_client.rs` / `tcp_server.rs`
- **关键 API**: `embassy_net` TCP socket
- **Mx.x 引用**: M5.1 §5 TCP

---

## BLE / IEEE 802.15.4

### nRF52840 IEEE 802.15.4 发送

- **路径**: `examples/nrf52840/src/bin/ieee802154_send.rs`
- **关键 API**: `embassy_nrf::ieee802154`
- **关键概念**: IEEE 802.15.4(6LoWPAN / Thread / Zigbee 基础)
- **Mx.x 引用**: M5.3 §4 BLE / Thread 协议栈

### nRF52840 温度传感器(softdevice)

- **路径**: `examples/nrf52840/src/bin/temp.rs`
- **关键 API**: 内部温度传感器 + softdevice
- **关键概念**: nRF softdevice 集成
- **Mx.x 引用**: M5.3 §3 nRF softdevice
- **预计耗时**: 2 小时

---

## 看门狗

### STM32F4 / nRF / RP IWDG

- **路径**:
  - `examples/stm32f4/src/bin/wdt.rs`
  - `examples/nrf52840/src/bin/wdt.rs`
- **关键 API**: `IndependentWatchDog` / `WatchdogHandle`
- **关键概念**: 独立看门狗(LSI 时钟驱动)
- **模式**: 模式 4(Watchdog)
- **Mx.x 引用**: M6.3 + M7.4 §5

### nRF52840 QSPI 低功耗

- **路径**: `examples/nrf52840/src/bin/qspi_lowpower.rs`
- **关键 API**: QSPI flash + 低功耗
- **关键概念**: 外部 flash 与 deep sleep 协同
- **Mx.x 引用**: M6.3 §9 唤醒源管理

### mcxa2xx / mcxa5xx 低功耗(deep sleep)

- **路径**:
  - `examples/mcxa2xx/src/bin/power-deepsleep.rs`
  - `examples/mcxa2xx/src/bin/power-deepsleep-big-jump.rs`
  - `examples/mcxa2xx/src/bin/power-deepsleep-gating.rs`
  - `examples/mcxa2xx/src/bin/power-deepsleep-gpio-int.rs`
  - `examples/mcxa2xx/src/bin/power-wfe-gated.rs`
- **关键 API**: `embassy_mcxa` 电源管理
- **关键概念**: deep sleep 三档(WfeUngated / WfeGated / DeepSleep)
- **Mx.x 引用**: M6.3 §5 三档功耗模式

### nRF 实时追踪(RTOS trace)

- **路径**: `examples/nrf-rtos-trace/src/bin/rtos_trace.rs`
- **关键 API**: RTOS trace(SEGGER SystemView)
- **关键概念**: 任务调度追踪
- **Mx.x 引用**: M2.1 §3 主循环可视化

---

## Boot / DFU

### STM32 bootloader

- **路径**:
  - `examples/boot/bootloader/stm32/src/main.rs`
  - `examples/boot/bootloader/stm32-dual-bank/src/main.rs`
  - `examples/boot/bootloader/stm32wb-dfu/src/main.rs`
- **关键 API**: `embassy_boot::BootLoader`
- **关键概念**: Bootloader 主函数
- **Mx.x 引用**: M6.1 §10 实战示例

### STM32 应用程序(A/B 槽)

- **路径**:
  - `examples/boot/application/stm32f3/src/bin/a.rs`
  - `examples/boot/application/stm32f4/src/bin/a.rs`(`b.rs` 同理)
  - `examples/boot/application/stm32f7/src/bin/a.rs`
  - `examples/boot/application/stm32h7/src/bin/a.rs`
  - `examples/boot/application/stm32l0/src/bin/a.rs`
  - `examples/boot/application/stm32l4/src/bin/a.rs`
  - `examples/boot/application/stm32wl/src/bin/a.rs`
- **关键 API**: `FirmwareUpdater` + `mark_updated` / `mark_booted`
- **关键概念**: 应用固件视角的 OTA
- **Mx.x 引用**: M6.2 §10 OTA 时序

### nRF52840 bootloader

- **路径**: `examples/boot/bootloader/nrf/src/main.rs`
- **关键 API**: `embassy_boot_nrf` 平台适配
- **关键概念**: nRF softdevice 兼容 bootloader
- **Mx.x 引用**: M6.1 §7 平台差异

### RP2040 bootloader

- **路径**:
  - `examples/boot/bootloader/rp/src/main.rs`
  - `examples/boot/bootloader/rp235x/src/main.rs`
- **关键 API**: `embassy_boot_rp` + `WatchdogFlash`
- **关键概念**: RP 平台 watchdog 集成
- **Mx.x 引用**: M6.1 §7 平台差异 + M6.3 §9

### STM32WB DFU

- **路径**:
  - `examples/boot/application/stm32wb-dfu/src/main.rs`
  - `examples/boot/bootloader/stm32wb-dfu/src/main.rs`
  - `examples/boot/application/stm32wba-dfu/src/main.rs`
  - `examples/boot/bootloader/stm32wba-dfu/src/main.rs`
- **关键 API**: USB DFU class
- **关键概念**: USB DFU 升级流程
- **Mx.x 引用**: M6.2 §4 OTA 时序

### 完整 STM32 平台 A/B 槽

- **路径**:
  - `examples/boot/application/stm32f3/`
  - `examples/boot/application/stm32f7/`
  - `examples/boot/application/stm32h7/`
  - `examples/boot/application/stm32l0/`
  - `examples/boot/application/stm32l1/`
  - `examples/boot/application/stm32l4/`
  - `examples/boot/application/stm32wl/`
  - `examples/boot/application/nrf/`
  - `examples/boot/application/rp/`
  - `examples/boot/application/rp235x/`
- **关键 API**: 全 embassy-boot 实战
- **Mx.x 引用**: M6.1 + M6.2 全部章节

---

## 低功耗

### mcxa2xx / mcxa5xx 各种 deep sleep 变体

- **路径**:
  - `examples/mcxa2xx/src/bin/power-deepsleep.rs`(基础)
  - `examples/mcxa2xx/src/bin/power-deepsleep-big-jump.rs`(大跳转)
  - `examples/mcxa2xx/src/bin/power-deepsleep-gating.rs`(时钟门控)
  - `examples/mcxa2xx/src/bin/power-deepsleep-gpio-int.rs`(GPIO 中断唤醒)
  - `examples/mcxa2xx/src/bin/power-wfe-gated.rs`(WFE 门控)
- **关键 API**: `embassy_mcxa` 电源管理
- **Mx.x 引用**: M6.3 §5 三档功耗模式
- **预计耗时**: 每例 1 小时

### nRF 低功耗 + QSPI

- **路径**: `examples/nrf52840/src/bin/qspi_lowpower.rs`
- **Mx.x 引用**: M6.3 §9

### mcxa CDOG(Copy Dog 看门狗)

- **路径**: `examples/mcxa2xx/src/bin/cdog.rs`
- **关键 API**: NXP CDOG 外设
- **Mx.x 引用**: M6.3 §9 + M7.4 §5

### RP 简易 WFE 循环

- **RP 平台未提供专门 deep sleep 示例**,WFE 循环在所有 RP 示例的 `Executor::run` 主循环中体现
- **Mx.x 引用**: M6.3 §8

---

## PIO(RP 特有)

RP2040 的 PIO(Programmable I/O)允许用软件定义自定义协议。

### RP PIO UART

- **路径**: `examples/rp/src/bin/pio_uart.rs`
- **关键 API**: `embassy_rp::pio::Pio`
- **关键概念**: PIO 实现 UART 协议
- **Mx.x 引用**: M3.4 §5 PIO

### RP PIO SPI

- **路径**: `examples/rp/src/bin/pio_spi.rs`
- **关键 API**: PIO 实现 SPI
- **Mx.x 引用**: M3.4 + M4.3

### RP PIO I2S

- **路径**: `examples/rp/src/bin/pio_i2s.rs`
- **关键 API**: PIO 实现 I2S(音频)
- **Mx.x 引用**: M3.4

### RP PIO 1-Wire(温度传感器)

- **路径**:
  - `examples/rp/src/bin/pio_onewire.rs`
  - `examples/rp/src/bin/pio_onewire_parasite.rs`
- **关键 API**: PIO 实现 1-Wire 协议
- **关键概念**: DS18B20 温度传感器
- **Mx.x 引用**: M3.4 §5 + 实战

### RP PIO 旋转编码器

- **路径**: `examples/rp/src/bin/pio_rotary_encoder.rs`
- **关键 API**: PIO 编码器
- **关键概念**: 旋钮输入
- **Mx.x 引用**: M3.4

### RP PIO 时钟输出

- **路径**: `examples/rp/src/bin/pio_clk.rs`
- **关键 API**: PIO 输出任意频率时钟
- **关键概念**: 时钟生成
- **Mx.x 引用**: M3.4

### RP PIO DMA

- **路径**: `examples/rp/src/bin/pio_dma.rs`
- **关键 API**: PIO + DMA 联动
- **关键概念**: PIO 高吞吐数据搬运
- **Mx.x 引用**: M3.4

### RP PIO 舵机

- **路径**: `examples/rp/src/bin/pio_servo.rs`
- **关键 API**: PIO + 舵机协议
- **Mx.x 引用**: M3.4

---

## 杂项 / 其他平台

### STM32F4 SDMMC(SD 卡)

- **路径**: `examples/stm32f4/src/bin/sdmmc.rs`
- **关键 API**: `embassy_stm32::sdmmc::Sdmmc`
- **关键概念**: SD 卡接口
- **Mx.x 引用**: M4.3 + 文件系统

### STM32F4 MCO(Master Clock Output)

- **路径**: `examples/stm32f4/src/bin/mco.rs`
- **关键 API**: 时钟输出
- **Mx.x 引用**: M3.2 时钟

### nRF52840 PDM(数字麦克风)

- **路径**: `examples/nrf52840/src/bin/pdm.rs`
- **关键 API**: PDM(Pulse Density Modulation)麦克风
- **Mx.x 引用**: M3.3 + 音频

### nRF52840 QDEC(正交解码)

- **路径**: `examples/nrf52840/src/bin/qdec.rs`
- **关键 API**: QDEC(Quadrature Decoder)
- **关键概念**: 旋转编码器硬件接口
- **Mx.x 引用**: M3.3

### nRF52840 QSPI flash

- **路径**: `examples/nrf52840/src/bin/qspi.rs`
- **关键 API**: QSPI + 外部 flash
- **Mx.x 引用**: M4.3 + M6.1

### STM32F4 Hello World(最简)

- **路径**: `examples/stm32f4/src/bin/hello.rs`
- **关键 API**: defmt::info!
- **关键概念**: 最小可运行 embassy 程序
- **Mx.x 引用**: M7.1 §2 入门

---

## 反向索引(Mx.x → 关键 bin)

| Mx.x | 关键 bin 文件 | 主题 |
|------|---------------|------|
| M1.1 | (无,看 embassy 仓库根布局) | 项目结构 |
| M1.2 | (无,看 embassy-* crate 文档) | crate 列表 |
| M1.3 | (无,async 概念) | 异步基础 |
| M2.1 | multiprio.rs / raw_spawn.rs / manually_create_executor.rs | 任务调度 |
| M2.2 | timer.rs (nrf52840) | 时间管理 |
| M2.3 | channel_sender_receiver.rs / sharing.rs | 同步原语 |
| M2.4 | (无,看 embassy-futures 文档) | Future 组合 |
| M3.1 | (无,跨平台对比) | HAL 架构 |
| M3.2 | blinky.rs / button.rs (stm32f4) | STM32 HAL |
| M3.3 | twim.rs / saadc.rs / ppi.rs (nrf52840) | nRF HAL |
| M3.4 | blinky.rs / pio_*.rs (rp) | RP HAL |
| M4.1 | button.rs / adc_dma.rs | GPIO |
| M4.2 | usart_async.rs / usart_buffered.rs / usb_serial_multitask.rs | UART |
| M4.3 | spi_dma.rs / spi_async.rs / spi_sdmmc.rs | SPI |
| M4.4 | i2c_slave_async.rs / twim.rs | I2C |
| M4.5 | pwm_servo.rs / pwm_input.rs / wdt.rs | 定时器 / PWM |
| M5.1 | ethernet_w5500_*.rs (rp) | 网络 |
| M5.2 | usb_hid_keyboard.rs / usb_serial_multitask.rs | USB |
| M5.3 | ieee802154_send.rs / temp.rs (nrf) | BLE / 802.15.4 |
| M5.4 | (无,在 embassy-lora 仓库) | LoRa |
| M6.1 | examples/boot/bootloader/ 全部 | 启动引导 |
| M6.2 | examples/boot/application/ 全部 | OTA 升级 |
| M6.3 | power-deepsleep*.rs (mcxa) / qspi_lowpower.rs (nrf) | 低功耗 |
| M7.1 | blinky.rs (任何平台) | 开发环境 |
| M7.2 | (无,看 panic-probe 文档) | 调试 |
| M7.3 | (无,看 embassy-test 文档) | 测试 |
| M7.4 | 全部 | 模式汇编 |

---

## 入门路径(10 步)

适合 embassy 新手,按"由浅入深"循序渐进,每步 30 分钟到 4 小时。

| 步骤 | 路径 | 关键点 | 预计耗时 |
|------|------|--------|----------|
| 1 | `examples/rp/src/bin/blinky.rs` | GPIO 闪灯(最短入门) | 30 min |
| 2 | `examples/nrf52840/src/bin/blinky.rs` 或 `stm32f4/src/bin/blinky.rs` | 跨平台对照 | 30 min |
| 3 | `examples/stm32f4/src/bin/button.rs` | GPIO 输入 + wait_for_low | 30 min |
| 4 | `examples/stm32f4/src/bin/usart_async.rs` | UART 异步收发 | 2 h |
| 5 | `examples/stm32f4/src/bin/spi_dma.rs` | SPI + DMA | 1.5 h |
| 6 | `examples/stm32f4/src/bin/i2c_slave_async.rs` | I2C 异步 | 1.5 h |
| 7 | `examples/rp/src/bin/multiprio.rs` | 多任务 + select_biased | 1 h |
| 8 | `examples/nrf52840/src/bin/channel_sender_receiver.rs` | Channel 任务间通信 | 1 h |
| 9 | `examples/boot/application/stm32f4/src/bin/a.rs` + `examples/boot/bootloader/stm32/src/main.rs` | OTA 升级全流程 | 4 h |
| 10 | `examples/rp/src/bin/ethernet_w5500_icmp_ping.rs` | 网络入门 | 4 h |

**总入门路径**:约 18-20 小时,可完成 embassy 主要功能的基础理解。

---

## 维护说明

- **路径稳定性**:embassy 仓库结构相对稳定,但 `examples/` 目录会随新平台 / 新功能增加。建议固定 embassy 版本,例如 commit `cdc25b1b8`(本项目 M7 收官 commit)
- **bin 文件数**:截至 2026-06,embassy `examples/` 共有 ~150 个 `src/bin/*.rs` 文件,本附录索引了 50+ 个关键文件
- **未覆盖平台**:ESP32 / RISC-V / esp32s3 等 Embassy 实验性支持的平台在 `embassy` 主仓的 `examples/` 中也有,本附录聚焦 STM32 / nRF / RP 三大主流
- **学习顺序建议**:M8.B §6 的 7 周路径(综合)与本附录的 10 步入门路径(聚焦)互补

---

## M1-M7 引用回标汇总

| Mx.x 引用频次 | 主题 |
|----------------|------|
| 全文(基础) | embassy 主仓库 / book |
| M7.1, M7.2, M7.3 | blinky + button 入门示例 |
| M2.x | executor / time / sync 示例 |
| M3.x | 三平台 HAL 示例 |
| M4.1-4.5 | 外设驱动示例 |
| M5.1-5.4 | 网络 / USB / BLE / LoRa 示例 |
| M6.1-6.3 | boot / DFU / 低功耗示例 |
| M7.1-7.4 | 实战模式示例 |

本附录是 embassy 仓库 `examples/` 的"精选索引",与 M8.A 术语表 + M8.B 参考资料共同构成 Embassy 学习项目的"参考三角"。
