# 从裸机到异步Rust

作为Embassy新手，可能会觉得所有的术语和概念都很难理解。本章旨在阐明Embassy中的不同层次，以及每一层次为应用程序编写者解决的问题。

本例中使用的是STM32 IOT01A开发板，很容易转换为任何STM32芯片。对于nRF，它的PAC本身并未在Embassy项目中维护，但概念和层次结构是类似的。

我们将编写的应用程序非常简单，它实现的功能仅仅是：按下按钮，让LED闪烁。它的输入输出处理非常典型，对于后续要讨论的所有示例都非常有参考价值。我们将从外设访问包(PAC)示例开始，异步示例结束。

## PAC版本

如果不直接从内存读取的话，PAC（Peripheral Access Crate，外设访问包）是访问外设和寄存器的最底层API。它提供了不同的类型来使访问外设寄存器更容易，但它并不能阻止你编写不安全的代码。

因此，直接使用PAC编写应用程序并不推荐，建议仅在上层没有暴露您想使用的接口时使用。

使用PAC的示例程序如下：

```rust
#![no_std]
#![no_main]

use pac::gpio::vals;
use {defmt_rtt as _, panic_probe as _, stm32_metapac as pac};

#[cortex_m_rt::entry]
fn main() -> ! {
    // Enable GPIO clock
    let rcc = pac::RCC;
    unsafe {
        rcc.ahb2enr().modify(|w| {
            w.set_gpioben(true);
            w.set_gpiocen(true);
        });

        rcc.ahb2rstr().modify(|w| {
            w.set_gpiobrst(true);
            w.set_gpiocrst(true);
            w.set_gpiobrst(false);
            w.set_gpiocrst(false);
        });
    }

    // Setup button
    let gpioc = pac::GPIOC;
    const BUTTON_PIN: usize = 13;
    unsafe {
        gpioc.pupdr().modify(|w| w.set_pupdr(BUTTON_PIN, vals::Pupdr::PULLUP));
        gpioc.otyper().modify(|w| w.set_ot(BUTTON_PIN, vals::Ot::PUSHPULL));
        gpioc.moder().modify(|w| w.set_moder(BUTTON_PIN, vals::Moder::INPUT));
    }

    // Setup LED
    let gpiob = pac::GPIOB;
    const LED_PIN: usize = 14;
    unsafe {
        gpiob.pupdr().modify(|w| w.set_pupdr(LED_PIN, vals::Pupdr::FLOATING));
        gpiob.otyper().modify(|w| w.set_ot(LED_PIN, vals::Ot::PUSHPULL));
        gpiob.moder().modify(|w| w.set_moder(LED_PIN, vals::Moder::OUTPUT));
    }

    // Main loop
    loop {
        unsafe {
            if gpioc.idr().read().idr(BUTTON_PIN) == vals::Idr::LOW {
                gpiob.bsrr().write(|w| w.set_bs(LED_PIN, true));
            } else {
                gpiob.bsrr().write(|w| w.set_br(LED_PIN, true));
            }
        }
    }
}
```

正如你所看到的，PAC层次的抽象中，启用外设时钟，配置应用程序的输入引脚和输出引脚，都需要大量的代码。

它的另一个缺点是：在轮询按钮状态时会忙等待。这阻止了微控制器利用睡眠模式来节省能源。

## HAL版本

为了简化我们的应用程序，我们可以选择使用HAL（硬件抽象层）。HAL提供了更高级别的API抽象，它可以处理以下细节：

- 使用外设时，自动启用外设时钟
- 在更高层次的类型中派生并应用寄存器配置
- 实现嵌入式硬件抽象层(embedded-hal)特性，使外设能用于第三方驱动程序

下面是采用HAL的示例程序：

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;
use embassy_stm32::gpio::{Input, Level, Output, Pull, Speed};
use {defmt_rtt as _, panic_probe as _};

#[entry]
fn main() -> ! {
    let p = embassy_stm32::init(Default::default());
    let mut led = Output::new(p.PB14, Level::High, Speed::VeryHigh);
    let button = Input::new(p.PC13, Pull::Up);

    loop {
        if button.is_low() {
            led.set_high();
        } else {
            led.set_low();
        }
    }
}
```

正如你所看到的，即使没有使用任何异步代码，应用程序也变得简单了很多。输入和输出类型隐藏了访问GPIO寄存器的所有细节，并允许你使用更简单的API来查询按钮的状态和切换LED输出。

然而，PAC示例中的缺点在这里仍然存在：应用程序在忙等待，消耗的能源比必要的多。

## 中断驱动

为了节约能源，需要支持处理器睡眠，需要通过配置应用程序，使其能够在按钮被按下时通过中断得到通知时才执行相关任务。

一旦配置了中断，应用程序就可以指示微控制器进入睡眠模式，消耗的能源显著减少。

考虑到Embassy的关注重点是异步Rust（我们将在本示例之后回到这个问题），示例应用程序必须使用HAL和PAC的组合才能使用中断。因此，应用程序还包含一些访问PAC的辅助函数（下面没有列出）。

```rust
#![no_std]
#![no_main]

use core::cell::RefCell;

use cortex_m::interrupt::Mutex;
use cortex_m::peripheral::NVIC;
use cortex_m_rt::entry;
use embassy_stm32::gpio::{Input, Level, Output, Pin, Pull, Speed};
use embassy_stm32::peripherals::{PB14, PC13};
use embassy_stm32::{interrupt, pac};
use {defmt_rtt as _, panic_probe as _};

static BUTTON: Mutex<RefCell<Option<Input<'static, PC13>>>> = Mutex::new(RefCell::new(None));
static LED: Mutex<RefCell<Option<Output<'static, PB14>>>> = Mutex::new(RefCell::new(None));

#[entry]
fn main() -> ! {
    let p = embassy_stm32::init(Default::default());
    let led = Output::new(p.PB14, Level::Low, Speed::Low);
    let mut button = Input::new(p.PC13, Pull::Up);

    cortex_m::interrupt::free(|cs| {
        enable_interrupt(&mut button);

        LED.borrow(cs).borrow_mut().replace(led);
        BUTTON.borrow(cs).borrow_mut().replace(button);

        unsafe { NVIC::unmask(pac::Interrupt::EXTI15_10) };
    });

    loop {
        cortex_m::asm::wfe();
    }
}

#[interrupt]
fn EXTI15_10() {
    cortex_m::interrupt::free(|cs| {
        let mut button = BUTTON.borrow(cs).borrow_mut();
        let button = button.as_mut().unwrap();

        let mut led = LED.borrow(cs).borrow_mut();
        let led = led.as_mut().unwrap();
        if check_interrupt(button) {
            if button.is_low() {
                led.set_high();
            } else {
                led.set_low();
            }
        }
        clear_interrupt(button);
    });
}
//
//
//
```

简单的程序变得越来越复杂，主要是我们需要在全局范围内保存按钮和LED的状态，以便于主应用循环能像中断一样访问到它们。

要做到这一点，类型必须用锁来保护，并且当我们通过全局状态获得了外设的访问权限期间，中断必须被暂时屏蔽。

幸运的是，在Embassy中，有一种很优雅的方式来解决这个问题。

## 异步版本

现在是时候充分利用Embassy的功能了。最核心的内容是，Embassy有一个异步执行器，或者你可以理解为异步任务的运行时，执行器轮询一组任务（在编译时定义），每当一个任务被阻塞时，`执行器`将运行另一个任务，或者让微控制器进入睡眠状态。

```rust
#![no_std]
#![no_main]
#![feature(type_alias_impl_trait)]

use embassy_executor::Spawner;
use embassy_stm32::exti::ExtiInput;
use embassy_stm32::gpio::{Input, Level, Output, Pull, Speed};
use {defmt_rtt as _, panic_probe as _};

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_stm32::init(Default::default());
    let mut led = Output::new(p.PB14, Level::Low, Speed::VeryHigh);
    let mut button = ExtiInput::new(Input::new(p.PC13, Pull::Up), p.EXTI13);

    loop {
        button.wait_for_any_edge().await;
        if button.is_low() {
            led.set_high();
        } else {
            led.set_low();
        }
    }
}
```

异步版本看起来与HAL版本非常相似，除了一些小细节：

- 主入口点用不同的宏进行注释，并具有异步类型签名。此宏创建并启动Embassy运行时实例，并启动主应用程序任务。应用程序可以使用Spawner实例生成其他任务。
- 外设初始化由main宏完成，并传递到主任务中。
- 在检查按钮状态之前，应用程序会一直等待引脚状态的转换（ 低→高 或 高→低 ）。

当`button.await_for_any_edge().await`被调用，如果没有其他任务可以执行，微控制器会暂停主任务并进入睡眠状态。Embassy HAL配置了按钮（在`ExtiButton`中）的中断处理程序，当中断发生时，等待按钮的任务都将被唤醒。

执行器的开销最小化、“并发”执行多个任务、极大简化应用程序等特点，让`异步`非常适用于嵌入式开发。

## 总结

本章我们讲述了在不同的抽象层次中，如何用Embassy实现相同的应用程序。我们从PAC层次开始，然后是HAL，再是使用中断，最后是通过异步间接使用中断。
