# 03 Rust 异步基础与 Embassy 适配

> 本文档是 M1 收官之作，把 Rust 标准 async 模型与 Embassy 的具体适配点连接起来。
> 读者已具备 Rust 异步基础（已在 §1 5 分钟过一遍），重点在 §4 Embassy 适配与 §5 嵌入式特定考虑。

---

## 1. 5 分钟回顾：Rust async 模型

### 1.1 `async fn` → 状态机

```rust
async fn read_with_timeout() -> Result<u8, Error> {
    let data = read_uart().await?;       // await 点 1
    Timer::after_millis(100).await;       // await 点 2
    Ok(data)
}
```

`async fn` 编译后等价于一个**实现 `Future` trait 的匿名的零大小状态机**。每个 `.await` 点是该状态机的一个状态。函数体是状态转移函数。**Future 是惰性的**：不 `.await` 不会执行。

### 1.2 `Future` trait

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(T),   // 完成，带结果
    Pending,    // 未就绪，注册 cx.waker()，等待 wake 后被重新 poll
}
```

`poll` 协议的本质：**未来要么完成，要么把"何时重试"的决定权交给 waker**。

### 1.3 执行器（Executor）的角色

执行器就是**一个循环**：不断从待处理任务队列里取出 task，调用它的 `poll`。task 返回 `Ready` 就丢弃；返回 `Pending` 就保留等下次唤醒。

```rust
loop {
    if let Some(task) = queue.dequeue() {
        task.poll();  // 重新 poll
    } else {
        wait_for_wake();  // 队列空时休眠
    }
}
```

### 1.4 与 tokio / async-std 的关键差异（前置概念）

| 维度 | std async | tokio / async-std |
|------|-----------|-------------------|
| 抽象 | 只定义 `Future` trait | 提供完整运行时 |
| Waker | 用户需自实现 | `tokio::spawn` 包装好 |
| 时间 | 无 | 内置 `tokio::time` |
| 多线程 | 平台线程 | tokio 调度 + 工作窃取 |
| 阻塞 I/O | 不允许 | 不允许 |

**Embassy = tokio 的设计哲学 + 嵌入式 zero-alloc 约束**。本节重点在 Embassy 怎么"塞"进 MCU。

---

## 2. Waker 与 Executor 协作

### 2.1 `Waker` 的本质

```rust
pub struct Waker {
    waker: RawWaker,
}

pub struct RawWaker {
    data: *const (),         // 用户数据（Embassy 用 TaskRef 指针）
    vtable: &'static RawWakerVTable,  // 4 个函数指针
}

pub struct RawWakerVTable {
    clone: fn(*const ()) -> RawWaker,
    wake: fn(*const ()),
    wake_by_ref: fn(*const ()),
    drop: fn(*const ()),
}
```

`Waker` 就是**一个函数指针表 + 一个数据指针**。`wake()` 调用 = 执行 `vtable.wake(data)`。

### 2.2 Embassy 的 waker 实现（极致精简）

`embassy-executor/src/raw/waker.rs` 完整实现（50 行）：

```rust
use core::task::{RawWaker, RawWakerVTable, Waker};
use super::{TaskHeader, TaskRef, wake_task};

static VTABLE: RawWakerVTable = RawWakerVTable::new(clone, wake, wake, drop);

unsafe fn clone(p: *const ()) -> RawWaker {
    RawWaker::new(p, &VTABLE)
}

unsafe fn wake(p: *const ()) {
    wake_task(TaskRef::from_ptr(p as *const TaskHeader))
}

unsafe fn drop(_: *const ()) {
    // nop
}

pub(crate) unsafe fn from_task(p: TaskRef) -> Waker {
    Waker::from_raw(RawWaker::new(p.as_ptr() as _, &VTABLE))
}
```

**4 个惊人简化**：
1. **`clone` 和 `wake` 共用** —— Embassy 的 waker 是单所有者模型，clone 只是复制指针
2. **`wake_by_ref` 直接是 `wake`** —— 借用调用无需区分
3. **`drop` 是 noop** —— waker 不持有资源
4. **`data` 直接是 `*const TaskHeader`** —— 反查任务状态，无需任何查找表

总开销：**2 个字（指针 + vtable）**。比标准 `Waker`（通常 16+ 字节）紧凑 8 倍。

### 2.3 `from_waker` 反查：`task_from_waker` 优化

```rust
pub fn task_from_waker(waker: &Waker) -> TaskRef {
    unwrap!(
        try_task_from_waker(waker),
        "Found waker not created by the Embassy executor. ..."
    )
}
```

`AtomicWaker` 等通用原语可以**只存 1 个字（TaskRef）**而不是完整 `Waker`（2 个字）。这是 Embassy 嵌入式友好的关键优化。

---

## 3. Pin 与自借用 future

### 3.1 为什么需要 Pin

如果一个 future 内部有**自借用**（self-reference），它一旦被 `mem::swap` 或 `mem::replace` 移动，内部的引用就指向旧地址，**未定义行为**。

```rust
async fn self_referential() {
    let data = [1, 2, 3];
    let slice = &data;     // 借用 data
    yield_now().await;     // ← 如果 future 在此被移动，slice 悬垂
    println!("{:?}", slice);
}
```

### 3.2 `Pin<&mut T>` 的保证

`Pin` 包装的类型**承诺不会被移动**，直到被 drop。配合 `Unpin` trait：
- **`Unpin`**：大多数类型自带（`i32`、`String`、`Vec` 等），可以安全 `Pin::get_mut`
- **`!Unpin`**（默认）：自借用 future 必须 `Pin` 才能 poll

### 3.3 Embassy 的处理策略（对用户隐藏 Pin）

Embassy 的 `#[task]` 宏**自动 Pin 住 future**：

```rust
// 用户代码（不需要 Pin）
#[task]
async fn my_task() {
    Timer::after_millis(100).await;
}

// 宏展开后（用户在编译产物里看到 Pin）
unsafe fn poll(p: TaskRef) {
    let this = &*p.as_ptr().cast::<TaskStorage<F>>();
    let future = Pin::new_unchecked(this.future.as_mut());  // ← 内部 Pin
    // ...
}
```

**来源**：`embassy-executor/src/raw/mod.rs:249`

```rust
unsafe fn poll(p: TaskRef) {
    let this = &*p.as_ptr().cast::<TaskStorage<F>>();

    let future = Pin::new_unchecked(this.future.as_mut());   // ← Pin!
    let waker = waker::from_task(p);
    let mut cx = Context::from_waker(&waker);
    match future.poll(&mut cx) {
        Poll::Ready(_) => {
            this.future.drop_in_place();
            // ...
        }
        Poll::Pending => {}
    }
    mem::forget(waker);
}
```

**关键点**：
- `Pin::new_unchecked` 是 `unsafe` —— 编译器无法证明这个 `Pin` 是安全的，**靠 TaskStorage 的固定地址保证**
- `mem::forget(waker)` 避免编译器插入 virtual call（已知 waker drop 是 noop，省一个间接调用）
- 整个 poll 过程**用户完全不接触 Pin**

### 3.4 vs tokio

| 维度 | Embassy | tokio |
|------|---------|-------|
| 用户接触 Pin | 否（宏包装） | 否（运行时包装） |
| Future 存储 | 静态槽位 `TaskStorage<F>` | 堆 `Box<dyn Future>` |
| `Pin` 安全保证 | 静态地址（无堆移动） | 堆固定（Box 已 pin） |
| 自借用 future | 安全 | 安全 |

---

## 4. Embassy 的关键适配

### 4.1 静态任务分配（cordyceps intrusive list）

任务不存堆，而是存放在编译期固定的 `TaskStorage<F>` 槽位里。**空闲/运行中**两种状态。

```rust
// 用户代码
#[embassy_executor::main]
async fn main(spawner: Spawner) {
    spawner.spawn(blink(p.PIN_25, 500)).unwrap();
}

// 宏展开：TaskStorage 静态数组
static TASKS: [Task<BlinkFuture>; 1] = [Task::new()];

// spawner.spawn 的核心：在空槽里写 future
let task = TASKS.iter().find(|t| t.is_available());
if let Some(task) = task {
    task.initialize(future);
    // 把 task 指针塞入执行器的 run_queue
    run_queue.enqueue(task_ref);
}
```

**关键数据结构**：执行器用 **cordyceps**（crates.io 上的 intrusive 数据结构库）实现 `IntrusiveList` —— task 通过指针自引用，**加入/移出 run_queue 不需要堆分配**。

```toml
# embassy-executor/Cargo.toml
cordyceps = { version = "0.3.4", features = ["no-cache-pad"] }
```

### 4.2 自实现 Waker（waker.rs / waker_turbo.rs）

| 实现 | 文件 | 适用场景 |
|------|------|----------|
| `waker.rs` | 标准版 | 大多数平台（基于 TaskRef 指针） |
| `waker_turbo.rs` | 优化版 | 需 nightly + 自定义 core 补丁（`turbowakers` feature） |

`waker_turbo.rs` 把 waker 压缩到 **1 个字**（直接 TaskRef，无 vtable）—— 进一步省内存，但需要 nightly Rust 配合修改 `core` 源码。

### 4.3 跨平台状态管理（state_atomics vs critical_section）

任务状态机（WAITING / RUNNING / ...）的存储有两种实现：

| 文件 | 平台 | 状态字段 |
|------|------|----------|
| `state_atomics.rs` | 有 32-bit 原子的平台 | `AtomicUsize`（无锁） |
| `state_critical_section.rs` | 没有原子的平台（AVR 等） | 用 `critical-section` 保护的 `Cell` |
| `state_atomics_arm.rs` | ARM 特殊优化 | 用 LDREX/STREX 指令 |

**典型状态机**（以 `state_atomics.rs` 为例，简化）：

```rust
const RUNNING: usize = 1 << 0;
const NOTIFY_QUEUED: usize = 1 << 1;
const SCHEDULED: usize = 1 << 2;

impl State {
    fn poll(&self) -> bool {
        // 用 CAS 原子地 "clear notify, set scheduled"
        self.0.fetch_and(!NOTIFY_QUEUED, AcqRel) & NOTIFY_QUEUED != 0
    }
}
```

**这种设计的好处**：调度器可以从任何上下文（线程/中断）安全地修改状态，因为都是原子操作。

### 4.4 平台相关唤醒（pender 回调）

```rust
// embassy-executor/src/platform/cortex_m.rs（节选）
unsafe impl InterruptExecutorImpl for crate::raw::Executor {
    fn sys_tick(&self) {
        self.on_wake();
    }
    fn pend_interrupt(&self) {
        cortex_m::peripheral::SCB::set_pendsv();  // 触发 PendSV 中断
    }
}
```

**关键设计**：
- `sys_tick` 是 SysTick 中断处理入口（由用户绑定到 `embassy-executor`）
- `pend_interrupt` 触发 PendSV —— Cortex-M 的最低优先级异常，**专用于 context switch**
- 在 PendSV handler 里执行 `poll` —— 避免在中断上半部做重活

**完整调用链**：
```
Timer 唤醒 (中断)
  → Waker::wake() → wake_task(task_ref)
  → 任务进入 run_queue
  → pend_interrupt() 触发 PendSV
  → PendSV handler: 取出 task，调用 poll
  → 如果 Ready 移出；Pending 保留
```

---

## 5. 与 tokio 的关键差异

| 维度 | Embassy | tokio |
|------|---------|-------|
| **Future 存储** | 静态槽位 `TaskStorage<F>` | 堆 `Box<dyn Future>` |
| **任务 spawn** | `pool_size` 编译期固定 | 运行时无限制（受 RAM 约束） |
| **多任务同一 future** | 同一 `#[task]` 函数可 spawn 多次（pool_size 限制） | 同一 `async fn` 可 spawn 无限次 |
| **状态机内存** | 编译期已知（`size_of::<F>()` 静态可查） | 动态堆分配 |
| **Wake 路径** | 原子操作（无锁，零延迟） | 通过 Arc + 调度队列 |
| **执行器类型** | thread-mode / interrupt-mode（编译期选） | current-thread / multi-thread（运行时） |
| **时间抽象** | trait 注入（HAL 提供 Driver） | 内置（tokio 运行时自带） |
| **取消** | 没有（必须 `select!` 等待） | `JoinHandle::abort()` |
| **`Pin` 责任** | `#[task]` 宏处理 | tokio 运行时处理 |

**核心取舍**：Embassy 用编译期信息换取了**内存零不确定性**和**零分配**，代价是**灵活性低**（不能动态创建 N 个任务、不能取消）。

---

## 6. 嵌入式特定考虑

### 6.1 中断与任务的桥接

**问题**：硬件中断（UART 接收完成、SPI TX 完成）需要唤醒等待中的任务。`Waker` 是任务的"门铃"。

**Embassy 模式**：用 `AtomicWaker` 把中断 handler 和任务 waker 解耦。

```rust
// UART 接收驱动（简化）
struct UartRx<'d> {
    waker: AtomicWaker,
    buf: &'d mut [u8],
}

impl Future for UartRxFuture<'_> {
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        // 1. 注册 waker
        self.uart.waker.register(cx.waker());

        // 2. 检查硬件状态
        if self.uart.is_rx_done() {
            return Poll::Ready(Ok(()));
        }

        // 3. 启动接收（DMA / 中断）
        unsafe { self.uart.start_rx(self.buf) };
        Poll::Pending
    }
}

// 中断 handler
#[interrupt]
fn USART1() {
    // 取出 waker，唤醒任务
    UART1_RX_WAKER.wake();
}
```

**`AtomicWaker` 状态机**（`embassy-sync/src/waitqueue/atomic_waker.rs`，4 状态）：

```rust
const WAITING:     usize = 0b00;  // 空闲
const REGISTERING: usize = 0b01;  // register() 持有 waker cell
const WAKING:      usize = 0b10;  // wake() 持有 waker cell
const BOTH:        usize = 0b11;  // 竞争状态，register() 必须补 wake
```

**为什么用 4 状态而不是 Mutex**：
- Mutex 进入临界区 → **中断延迟增加** → 不适合高优先级中断
- 状态机用 `fetch_or` → **无锁，零延迟** → 适合硬实时场景

### 6.2 临界区与无锁 wake

Embassy 抽象的 `critical-section` crate（嵌入式通用）：

```rust
critical_section::with(|cs| {
    // 在临界区内安全访问共享数据
    *SHARED.borrow_ref_mut(cs) = new_value;
});
```

在 wake 路径中：
- **不能**在临界区里调用 `waker.wake()` —— 可能嵌套临界区/死锁
- **应该**用原子状态机 + 后置 wake（参考上面的 4 状态机）

### 6.3 `yield_now` 的实现与代价

完整源码（`embassy-futures/src/yield_now.rs`）：

```rust
pub fn yield_now() -> impl Future<Output = ()> {
    YieldNowFuture { yielded: false }
}

struct YieldNowFuture {
    yielded: bool,
}

impl Future for YieldNowFuture {
    type Output = ();
    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        if self.yielded {
            Poll::Ready(())
        } else {
            self.yielded = true;
            cx.waker().wake_by_ref();   // ← 关键：自己 wake 自己
            Poll::Pending
        }
    }
}
```

**使用场景**（官方文档）：

```rust
while !some_condition() {
    yield_now().await;   // 让出 CPU，但不阻塞
}
```

**代价（必须警惕）**：
- 注:**busy-loop**：每次 `yield_now().await` 都会让出，但下次 poll 立即返回（除非被其他任务抢占），**100% CPU**
- 适合轮询简单条件（几微秒内变化）
- 适合需要等待**毫秒级以上**事件的场景 —— 用 `Timer::after()` 替代

### 6.4 `select!` 的工作原理

`embassy-futures/src/select.rs` 提供了 `select` 宏（78 符号），允许同时等待多个 future，**任一完成则返回**：

```rust
let result = select(
    uart.read(&mut buf),
    Timer::after_millis(100),
).await;

match result {
    Either::First(n) => /* UART 完成 */,
    Either::Second(_) => /* 超时 */,
}
```

**实现机制**（简化）：
- 创建一个 wrapper future，poll 时**poll 所有内部 future**
- 任一返回 `Ready` → select 自身也返回 `Ready`，并丢掉其他 future
- 全部 `Pending` → 返回 `Pending`
- 关键是**每个内部 future 都用同一个 `cx.waker()`** —— 任何一个唤醒都会重新 poll select

### 6.5 `JoinSet` / `JoinHandle` 没有的取舍

tokio 提供 `JoinSet` 让你收集任意数量的 future 并等待任一/全部。Embassy **没有**这个：
- 原因：JoinSet 内部需要堆分配 Box
- 替代：所有任务在编译期固定数量，**没有"任意集合"的概念**

---

## 7. 实战：带超时的外设读

```rust
use embassy_time::{with_timeout, Duration, Timer};
use embassy_stm32::usart::UartRx;

async fn read_with_timeout(
    uart: &mut UartRx<'static, DMA>,
    buf: &mut [u8],
) -> Result<usize, Error> {
    // 用 embassy_time::with_timeout 包装任意 future
    match with_timeout(Duration::from_millis(100), uart.read(buf)).await {
        Ok(Ok(n)) => Ok(n),                          // 读成功
        Ok(Err(e)) => Err(e),                        // 硬件错误
        Err(_) => Err(Error::Timeout),               // 超时
    }
}
```

**步骤解读**：

1. `uart.read(buf)` 返回一个 future，poll 时会注册到 `AtomicWaker` 并启动 DMA
2. `with_timeout(d, fut)` 创建一个 wrapper future，内部**同时 poll** `fut` 和 `Timer::after(d)`
3. 内部任一完成 → wrapper 返回，丢弃另一个
4. 关键：`Timer::after` 用的 time driver 由 HAL 提供（RP2040 用 TIMER，nRF 用 RTC）
5. **零堆**：所有状态机大小在编译期固定

**对比 tokio**：

```rust
// tokio 版
tokio::time::timeout(Duration::from_millis(100), uart.read(buf)).await
```

API 几乎一样，但内部：
- tokio：tokio 运行时提供 `timeout` 和时间
- Embassy：`with_timeout` 在 `embassy-futures`，时间来自 HAL 注入的 driver

---

## 8. 推荐阅读源码顺序

```
embassy-executor/src/raw/waker.rs            (50 行，waker 极简实现)
embassy-executor/src/raw/mod.rs (poll fn)    (~40 行，poll 核心)
embassy-executor/src/raw/state_atomics.rs    (无锁状态机)
embassy-futures/src/yield_now.rs             (49 行，自己 wake 自己的范式)
embassy-futures/src/select.rs                (78 符号，select 实现)
embassy-sync/src/waitqueue/atomic_waker.rs   (4 状态无锁 waker)
embassy-sync/src/channel.rs                  (跨任务通信)
embassy-executor-macros/src/macros/task.rs   (#[task] 宏展开)
```

按这个顺序读，能在 **~600 行代码**里理解 Embassy 异步核心。

---

## 9. 关键设计模式总结

| 模式 | 实现位置 | 收益 |
|------|----------|------|
| 静态 waker（2 字） | `executor/raw/waker.rs` | 比 std Waker 紧凑 8 倍 |
| TaskRef 反查 | `executor/raw/waker.rs` | 通用原语只存 1 字 |
| 自 wake 范式 | `futures/yield_now.rs` | 简单轮询无需自实现 wake |
| 4 状态机 waker | `sync/waitqueue/atomic_waker.rs` | 无锁 wake，零中断延迟 |
| cordyceps intrusive list | `executor/raw/run_queue.rs` | 任务入队出队零分配 |
| PendSV context switch | `executor/platform/cortex_m.rs` | 中断下半部 poll |
| `#[task]` 自动 Pin | `executor/raw/mod.rs` poll | 用户无感 |

---

## 10. M1 收官总结

经过 3 篇文档，我们从宏观到微观逐步深入 Embassy：

```
01-overview.md     → 是什么 / 为什么 / 怎么用（255 行）
02-architecture.md → crate 如何连接 / 依赖与 feature（485 行）
03-async.md        → async/await 怎么运行 / Embassy 适配（本文）
```

**下一步推荐路径**：
- **M2.1 立即进入**：`docs/04-executor.md`（任务调度原理）—— 把本文的 `poll` 函数展开
- **M2.2 同步视角**：`docs/05-time.md`（`Timer` 内部）—— 了解时间驱动注入
- **M2.3 通信视角**：`docs/06-sync.md`（Channel/Signal）—— 跨任务协作

---

## 11. 参考

- **本仓库**：
  - `docs/01-overview.md` · `docs/02-architecture.md`
  - `docs/04-executor.md`（M2.1，紧接本篇）
  - `docs/05-time.md` · `docs/06-sync.md` · `docs/07-futures.md`
- **官方**：
  - [Asynchronous Programming in Rust](https://rust-lang.github.io/async-book/) — std async 圣经
  - [std::task::Waker](https://doc.rust-lang.org/std/task/struct.Waker.html) — Waker 文档
  - [Pin and Suffering](https://rust-lang.github.io/rustdoc-quizes/about-this-quiz/about.html) — Pin 理解测试
- **上游 crate**：
  - [cordyceps](https://crates.io/crates/cordyceps) — intrusive 数据结构
  - [critical-section](https://crates.io/crates/critical-section) — 跨平台临界区
  - [futures::task::AtomicWaker](https://docs.rs/futures/latest/futures/task/struct.AtomicWaker.html) — 4 状态机原版（Embassy 移植自此）
