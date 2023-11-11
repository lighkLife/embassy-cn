硬件抽象层 HAL
======================================

.. toctree::
   :maxdepth: 1

   nRF
   STM32


Embassy 为多个微控制器系列提供硬件抽象层：

- `embassy-nrf` 用于 Nordic Semiconductor 的 nRF 微控制器

- `embassy-stm32` 用于意法半导体的 STM32 微控制器

- `embassy-rp` embassy-rp 用于 Raspberry Pi RP2040 微控制器

这些硬件抽象层为大多数外设实现了 async/await 功能，同时还实现了 `embedded-hal` 和 `embedded-hal-async` 中的异步特性。你也可以将这些 HAL 用于其他执行器。

对于 ESP32 系列，你可以使用 [esp-hal](https://github.com/esp-rs/esp-hal) 。