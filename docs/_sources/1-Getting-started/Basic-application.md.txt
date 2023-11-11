# A basic Embassy application

So you’ve got one of the [examples](https://embassy.dev/book/dev/examples.html) running, but what now? Let’s go through a simple Embassy application for the nRF52 DK to understand it better.

## Main

The full example can be found [here](https://github.com/embassy-rs/embassy/tree/master/docs/modules/ROOT/examples/basic).

### Rust Nightly

The first thing you’ll notice is a few declarations stating that Embassy requires some nightly features:

```rust
#![no_std]
#![no_main]
#![feature(type_alias_impl_trait)]
```

### Dealing with errors

Then, what follows are some declarations on how to deal with panics and faults. During development, a good practice is to rely on `defmt-rtt` and `panic-probe` to print diagnostics to the terminal:

```rust
use {defmt_rtt as _, panic_probe as _}; // global logger
```

### Task declaration

After a bit of import declaration, the tasks run by the application should be declared:

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

An embassy task must be declared `async`, and may NOT take generic arguments. In this case, we are handed the LED that should be blinked and the interval of the blinking.

|     | Notice that there is no busy waiting going on in this task. It is using the Embassy timer to yield execution, allowing the microcontroller to sleep in between the blinking. |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### Main

The main entry point of an Embassy application is defined using the `#[embassy_executor::main]` macro. The entry point is also required to take a `Spawner` and a `Peripherals` argument.

The `Spawner` is the way the main application spawns other tasks. The `Peripherals` type comes from the HAL and holds all peripherals that the application may use. In this case, we want to configure one of the pins as a GPIO output driving the LED:

```rust
#[embassy_executor::main]
async fn main(spawner: Spawner) {
    let p = embassy_nrf::init(Default::default());

    let led = Output::new(p.P0_13, Level::Low, OutputDrive::Standard);
    unwrap!(spawner.spawn(blinker(led, Duration::from_millis(300))));
}
```

What happens when the `blinker` task has been spawned and main returns? Well, the main entry point is actually just like any other task, except that you can only have one and it takes some specific type arguments. The magic lies within the `#[embassy_executor::main]` macro. The macro does the following:

1. Creates an Embassy Executor

2. Initializes the microcontroller HAL to get the `Peripherals`

3. Defines a main task for the entry point

4. Runs the executor spawning the main task

There is also a way to run the executor without using the macro, in which case you have to create the `Executor` instance yourself.

## The Cargo.toml

The project definition needs to contain the embassy dependencies:

```toml
embassy-executor = { version = "0.3.0", path = "../../../../../embassy-executor", features = ["defmt", "nightly", "integrated-timers", "arch-cortex-m", "executor-thread"] }
embassy-time = { version = "0.1.4", path = "../../../../../embassy-time", features = ["defmt", "nightly"] }
embassy-nrf = { version = "0.1.0", path = "../../../../../embassy-nrf", features = ["defmt", "nrf52840", "time-driver-rtc1", "gpiote", "nightly"] }
```

Depending on your microcontroller, you may need to replace `embassy-nrf` with something else (`embassy-stm32` for STM32. Remember to update feature flags as well).

In this particular case, the nrf52840 chip is selected, and the RTC1 peripheral is used as the time driver.
