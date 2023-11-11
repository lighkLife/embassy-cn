快速开始
======================================

.. toctree::
   :maxdepth: 1

   Basic-application


So you want to try Embassy, great! To get started, there are a few tools you need to install:

- [rustup](https://rustup.rs/) - the Rust toolchain is needed to compile Rust code.

- [probe-rs](https://crates.io/crates/probe-rs) - to flash the firmware on your device. If you already have other tools like `OpenOCD` setup, you can use that as well.

If you don’t have any supported board, don’t worry: you can also run embassy on your PC using the `std` examples.

Getting a board with examples
-------------------------------


Embassy supports many microcontroller families, but the easiest ways to get started is if you have one of the more common development kits.

nRF kits
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- [nRF52 DK](https://www.nordicsemi.com/Products/Development-hardware/nrf52-dk)

- [nRF9160 DK](https://www.nordicsemi.com/Products/Development-hardware/nRF9160-DK)

STM32 kits
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


- [STM32 Nucleo-144 development board with STM32H743ZI MCU](https://www.st.com/en/evaluation-tools/nucleo-h743zi.html)

- [STM32 Nucleo-144 development board with STM32F429ZI MCU](https://www.st.com/en/evaluation-tools/nucleo-f429zi.html)

- [STM32L4+ Discovery kit IoT node, low-power wireless, BLE, NFC, WiFi](https://www.st.com/en/evaluation-tools/b-l4s5i-iot01a.html)

- [STM32L0 Discovery kit LoRa, Sigfox, low-power wireless](https://www.st.com/en/evaluation-tools/b-l072z-lrwan1.html)

- [STM32 Nucleo-64 development board with STM32WL55JCI MCU](https://www.st.com/en/evaluation-tools/nucleo-wl55jc.html)

- [Discovery kit for IoT node with STM32U5 series](https://www.st.com/en/evaluation-tools/b-u585i-iot02a.html)

RP2040 kits
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/)

ESP32
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- [ESP32C3](https://github.com/esp-rs/esp-rust-board)

Running an example
-------------------------------

First you need to clone the [github repository];

```bash
git clone https://github.com/embassy-rs/embassy.git
cd embassy
```

You can run an example by opening a terminal and entering the following commands:

```bash
cd examples/nrf52840
cargo run --bin blinky --release
```

What’s next?
-------------------------------

Congratulations, you have your first Embassy application running! Here are some alternatives on where to go from here:

- Read more about the [executor](https://embassy.dev/book/dev/runtime.html).

- Read more about the [HAL](https://embassy.dev/book/dev/hal.html).

- Start [writing your application](https://embassy.dev/book/dev/basic_application.html).

