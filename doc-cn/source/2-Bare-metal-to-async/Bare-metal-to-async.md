# From bare metal to async Rust

If you’re new to Embassy, it can be overwhelming to grasp all the terminology and concepts. This guide aims to clarify the different layers in Embassy, which problem each layer solves for the application writer.

This guide uses the STM32 IOT01A board, but should be easy to translate to any STM32 chip. For nRF, the PAC itself is not maintained within the Embassy project, but the concepts and the layers are similar.

The application we’ll write is a simple 'push button, blink led' application, which is great for illustrating input and output handling for each of the examples we’ll go through. We’ll start at the Peripheral Access Crate (PAC) example and end at the async example.

## PAC version

The PAC is the lowest API for accessing peripherals and registers, if you don’t count reading/writing directly to memory addresses. It provides distinct types to make accessing peripheral registers easier, but it does not prevent you from writing unsafe code.

Writing an application using the PAC directly is therefore not recommended, but if the functionality you want to use is not exposed in the upper layers, that’s what you need to use.

The blinky app using PAC is shown below:

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

As you can see, a lot of code is needed to enable the peripheral clocks and to configure the input pins and the output pins of the application.

Another downside of this application is that it is busy-looping while polling the button state. This prevents the microcontroller from utilizing any sleep mode to save power.

## HAL version

To simplify our application, we can use the HAL instead. The HAL exposes higher level APIs that handle details such as:

- Automatically enabling the peripheral clock when you’re using the peripheral

- Deriving and applying register configuration from higher level types

- Implementing the embedded-hal traits to make peripherals useful in third party drivers

The HAL example is shown below:

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

As you can see, the application becomes a lot simpler, even without using any async code. The `Input` and `Output` types hide all the details of accessing the GPIO registers and allow you to use a much simpler API for querying the state of the button and toggling the LED output.

The same downside from the PAC example still applies though: the application is busy looping and consuming more power than necessary.

## Interrupt driven

To save power, we need to configure the application so that it can be notified when the button is pressed using an interrupt.

Once the interrupt is configured, the application can instruct the microcontroller to enter a sleep mode, consuming very little power.

Given Embassy focus on async Rust (which we’ll come back to after this example), the example application must use a combination of the HAL and PAC in order to use interrupts. For this reason, the application also contains some helper functions to access the PAC (not shown below).

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

The simple application is now more complex again, primarily because of the need to keep the button and LED states in the global scope where it is accessible by the main application loop, as well as the interrupt handler.

To do that, the types must be guarded by a mutex, and interrupts must be disabled whenever we are accessing this global state to gain access to the peripherals.

Luckily, there is an elegant solution to this problem when using Embassy.

## Async version

It’s time to use the Embassy capabilities to its fullest. At the core, Embassy has an async excecutor, or a runtime for async tasks if you will. The executor polls a set of tasks (defined at compile time), and whenever a task `blocks`, the executor will run another task, or put the microcontroller to sleep.

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

The async version looks very similar to the HAL version, apart from a few minor details:

- The main entry point is annotated with a different macro and has an async type signature. This macro creates and starts an Embassy runtime instance and launches the main application task. Using the `Spawner` instance, the application may spawn other tasks.

- The peripheral initialization is done by the main macro, and is handed to the main task.

- Before checking the button state, the application is awaiting a transition in the pin state (low → high or high → low).

When `button.await_for_any_edge().await` is called, the executor will pause the main task and put the microcontroller in sleep mode, unless there are other tasks that can run. Internally, the Embassy HAL has configured the interrupt handler for the button (in `ExtiButton`), so that whenever an interrupt is raised, the task awaiting the button will be woken up.

The minimal overhead of the executor and the ability to run multiple tasks "concurrently" combined with the enormous simplification of the application, makes `async` a great fit for embedded.

## Summary

We have seen how the same application can be written at the different abstraction levels in Embassy. First starting out at the PAC level, then using the HAL, then using interrupts, and then using interrupts indirectly using async Rust.
