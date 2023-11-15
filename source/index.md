# Embassy 下一代嵌入式应用框架
通过使用 Rust 编程语言、其异步功能以及 Embassy 库，更快地编写安全、正确且节能的嵌入式代码。

## [马上开始](https://embassy.dev/book/dev/getting_started.html)

## Rust + async ❤️ 嵌入式

[Rust编程语言](https://www.rust-lang.org/)具有极快的执行速度和高效的内存利用，且不依赖运行时、垃圾回收器或操作系统。得益于其完善的内存与线程安全性以及表达性强大的类型系统，它在编译时就能够捕捉到多种类型的错误。

Rust 的 [异步/等待(async/await)](https://rust-lang.github.io/async-book/) 机制在嵌入式系统中提供了前所未有的轻松且高效的多任务处理方式。任务会在编译时被转换成能够协作运行的状态机。同时，它不需要动态内存分配，且能在单一堆栈上运行，因此不需要对每个任务的堆栈大小进行调整。它淘汰了传统实时操作系统（RTOS）对内核上下文切换的的需求，并且在速度和占用空间上都[更快更小](https://tweedegolf.nl/en/blog/65/async-rust-vs-rtos-showdown)。

```rust
use defmt::info;
use embassy::executor::Spawner;
use embassy::time::{Duration, Timer};
use embassy_nrf::gpio::{AnyPin, Input, Level, Output, OutputDrive, Pin, Pull};
use embassy_nrf::Peripherals;

// Declare async tasks
#[embassy::task]
async fn blink(pin: AnyPin) {
    let mut led = Output::new(pin, Level::Low, OutputDrive::Standard);

    loop {
        // Timekeeping is globally available, no need to mess with hardware timers.
        led.set_high();
        Timer::after(Duration::from_millis(150)).await;
        led.set_low();
        Timer::after(Duration::from_millis(150)).await;
    }
}

// Main is itself an async task as well.
#[embassy::main]
async fn main(spawner: Spawner, p: Peripherals) {
    // Spawned tasks run in the background, concurrently.
    spawner.spawn(blink(p.P0_13.degrade())).unwrap();

    let mut button = Input::new(p.P0_11, Pull::Up);
    loop {
        // Asynchronously wait for GPIO events, allowing other tasks
        // to run, or the core to sleep.
        button.wait_for_low().await;
        info!("Button pressed!");
        button.wait_for_high().await;
        info!("Button released!");
    }
}
```

# 包含的组件

### 硬件抽象层

HAL提供了安全易用的接口来操作硬件，无需再对原始寄存器进行操作。

Embassy 维护了下列硬件的 HAL ，但它并不局限于此，您可以在任何使用 Embassy 的项目中使用 HAL 。

* [embassy-stm32](https://docs.embassy.dev/embassy-stm32/) , 支持 STM32 微控制器系列
* [embassy-nrf](https://docs.embassy.dev/embassy-nrf/) , 支持北欧半导体公司 \(the Nordic Semiconducotr\) 的 nRF52、nRF53、nRF91 系列

### 时间处理

不用再因硬件定时器而烦恼。 

 [embassy::time](https://docs.embassy.dev/embassy-time/) 提供了全局可用且不会溢出的 Instant、Duration 和 Timer 类型。

### 实时性

相同的异步执行器上的任务会以相互协调的方式运行。

但您可以创建多个具有不同优先级的执行器，使更高优先级的任务可以抢占较低优先级任务的运行。具体请参见[示例](https://github.com/embassy-rs/embassy/blob/master/examples/nrf52840/src/bin/multiprio.rs)。

### 低能耗

轻松构建电池寿命更长的设备。

异步执行器闲置时自动将核心置于睡眠状态。任务可通过中断将其唤醒，无需在等待时进行忙等轮询（busy-loop polling）。

### 网络

[embassy-net](https://docs.embassy.dev/embassy-net/) 实现了常用的网络功能，包括以太网、IP、TCP、UDP、ICMP和DHCP。在管理超时和同时为多个连接提供服务方面，Async 做了高度简化，提供了很大的便利。

### 蓝牙

[nrf-softdevice](https://github.com/embassy-rs/nrf-softdevice) 为 nRF52 微控制器提供蓝牙低能耗 4.x 和 5.x 支持。

### LoRa

[embassy-lora](https://docs.embassy.dev/embassy-lora/) 支持 STM32WL 无线微控制器和 Semtech SX127x 收发器上的 LoRa 网络。

### USB

[embassy-usb](https://docs.embassy.dev/embassy-usb/) 实现了设备侧的USB堆栈。实现了通用类（如USB串行（CDC ACM），支持 USB HID ，并且提供丰富的构建器API方便构建您自己的实现。

### Bootloader and DFU

[embassy-boot](https://github.com/embassy-rs/embassy/tree/master/embassy-boot) 是一个轻量级的启动器，它支持固件加载，支持以电源故障安全的方式升级固件，支持试用引导和回滚。

* * *

Copyright © 2019-2023 Embassy project contributors

[Privacy](https://embassy.dev/privacy/)   [Sitemap](https://embassy.dev/sitemap.xml)