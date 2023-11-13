
# 基本应用示例

我们已经正常运行了一个[示例程序](https://embassy.dev/dev/examples.html)。接下来，我们通过一个运行在nRF52套件环境下的Embassy程序来更深入地了解它。

## 详细内容

本章的代码都在[这里](https://github.com/embassy-rs/embassy/tree/master/docs/modules/ROOT/examples/basic)

### Rust Nightly

首先，需要Rust工具链的Nightly版本。Embassy的一些状态定义用到了nightly中的某些特性，比如：

```rust
#![no_std]
#![no_main]
#![feature(type_alias_impl_trait)]
```

### 错误处理

接下来是关于错误和恐慌(panic)的处理，在开发过程中，比较好的做法是依靠defmt_rtt和panic_probe将诊断信息输出到终端：

```rust
use {defmt_rtt as _, panic_probe as _}; // global logger
```

### 任务描述

通过添加属性宏的方式，任务定义如下：

```rust
#[embassy_executor::task]
async fn blinker(mut led: Output<'static, P0_13>, interval: Duration) {
    loop {
        led.set_high();
        Timer::after(interval).await;
        led.set_low();
        Timer::after(interval).await;
    }
}
```

Embassy中，任务必须被定义为异步的，**不能带有泛型参数**。在本例中，我们传入了两个参数：要操作的LED设备和闪烁的间隔时间。

> **注意**  
>Embassy并没有采用空转等待的方式，而是采用内部的计时器来实现出让执行(yield)，在闪烁周期内允许微控制器进入睡眠状态。(译者注：busy waiting going no，这里结合下文按意思来译的)

### main函数

Embassy应用的入口函数用`#[embassy_executor::main]`宏来定义，要求传入两个参数：`Spawner`、`Peripherals`。Spawner用于应用创建任务，`Spawner`是主任务创建其他任务的途径。 `Peripherals`来自HAL，它负责沟通可能用到的外设。本例中我们配置其中一根引脚，把它当作GPIO输出来驱动LED。

```rust
#[embassy_executor::main]
async fn main(spawner: Spawner) {
    let p = embassy_nrf::init(Default::default());

    let led = Output::new(p.P0_13, Level::Low, OutputDrive::Standard);
    unwrap!(spawner.spawn(blinker(led, Duration::from_millis(300))));
}
```

在blinker任务已经生成、main函数返回时，究竟发生了什么？主入口看起来跟其他任务没什么不同，除了它只能存在一个并且需要一些特殊类型的参数。主要的魔法来自`#[embassy_executor::main]`宏，它做了以下的事情：

1. 创建一个Embassy执行器。
2. 初始化硬件层并通过Peripherals参数提供交互。
3. 为程序入口定义主任务。
4. 运行执行器生成主任务。

不使用宏也可以运行执行器。这种情况下，需要您手动实现`执行器`实例。

## Cargo.toml

项目定义需要引入embassy相关的依赖：

```toml
embassy-executor = { version = "0.3.0", path = "../../../../../embassy-executor", features = ["defmt", "nightly", "integrated-timers", "arch-cortex-m", "executor-thread"] }
embassy-time = { version = "0.1.4", path = "../../../../../embassy-time", features = ["defmt", "nightly"] }
embassy-nrf = { version = "0.1.0", path = "../../../../../embassy-nrf", features = ["defmt", "nrf52840", "time-driver-rtc1", "gpiote", "nightly"] }
```

根据您的微控制器型号，选择相应的crate替换`embassy-nrf`，比如STM32用`embassy-stm32`，记得设置合适的`features`标记。

在上面的示例中，使用了nrf52840芯片，选择了RTC1作为时间驱动器。
