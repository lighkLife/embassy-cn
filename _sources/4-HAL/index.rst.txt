硬件抽象层 HAL
======================================

.. toctree::
   :maxdepth: 1

   nRF
   STM32




Embassy 为多个单片机系列提供了硬件抽象层：

- ``embassy-nrf``：北欧半导体公司的 nRF 微控制器

- ``embassy-stm32``： 意法半导体的 STM32 微控制器

- ``embassy-rp`` 树莓派的 RP2040 微控制器


这些硬件抽象层为大多数外设实现了异步等待（``async/await``）功能，同时，在``embedded-hal``和``embedded-hal-async``中实现了异步接口（async traits）。也你也可以选择其他执行器（即异步运行时）来使用这些硬件抽象层提供的函数。

针对 ESP32 系列的微控制器，可以使用 `esp-hal <https://github.com/esp-rs/esp-hal>`_ 库。
