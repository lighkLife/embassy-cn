硬件抽象层 HAL
======================================

.. toctree::
   :maxdepth: 1

   nRF
   STM32


Embassy provides HALs for several microcontroller families:

- `embassy-nrf` for the nRF microcontrollers from Nordic Semiconductor

- `embassy-stm32` for STM32 microcontrollers from ST Microelectronics

- `embassy-rp` for the Raspberry Pi RP2040 microcontrollers

These HALs implement async/await functionality for most peripherals while also implementing the async traits in `embedded-hal` and `embedded-hal-async`. You can also use these HALs with another executor.

For the ESP32 series, there is an [esp-hal](https://github.com/esp-rs/esp-hal) which you can use.