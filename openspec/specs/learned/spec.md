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

（待填充）

---

## 技巧模式

（待填充）

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
