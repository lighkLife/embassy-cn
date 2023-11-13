快速开始
======================================

.. toctree::
   :maxdepth: 1

   Basic-application


.. seealso:: 原文：https://embassy.dev/book/dev/getting_started.html

已经迫不及待要上手了吗？稍等，要正式开始工作前，我们需要做一些准备。

在您的开发环境中，需要确保以下软件已正确安装：

- `rustup <https://rustup.rs/>`_ Rust工具链，参考官方链接或各发行版的安装文档
- `probe-run <https://crates.io/crates/probe-run>`_ 一款固件写入工具，在将固件写入设备时需要。您也可以使用其他工具来做同样的事情，比如`OpenOCD`。

不用担心没有已支持的开发板，您可以在PC上，通过使用`std`来运行Embbassy应用。

支持的型号
-------------------------------------------

Embassy支持很多型号的微控制器，不过最容易的方式就是拥有一块更常见的开发板套件。

nRF套件
^^^^^^^^^^^

- `nRF52 Dk <https://www.nordicsemi.com/Products/Development-hardware/nrf52-dk>`_
- `nRF9160 DK <https://www.nordicsemi.com/Products/Development-hardware/nRF9160-DK>`_

STM32套件
^^^^^^^^^^^

- `STM32_Nucleo-144(STM32H743ZI MCU) <https://www.st.com/en/evaluation-tools/nucleo-h743zi.html>`_ 开发板
- `STM32_Nucleo-144(STM32F429ZI MCU) <https://www.st.com/en/evaluation-tools/nucleo-f429zi.html>`_ 开发板
- `STM32L4+ <https://www.st.com/en/evaluation-tools/b-l4s5i-iot01a.html>`_ loT开发套件,支持低功耗无线, BLE, NFC, WiFi
- `STM32L0 <https://www.st.com/en/evaluation-tools/b-l072z-lrwan1.html>`_ 开发套件，支持LoRa, Sigfox, 低功耗无线
- `STM32_Nculeo-64(STM32WL55JCI) <https://www.st.com/en/evaluation-tools/nucleo-wl55jc.html>`_ 开发版
- `STM32U5 <https://www.st.com/en/evaluation-tools/nucleo-wl55jc.html>`_ loT开发套件

RP2040套件
^^^^^^^^^^^

- `树莓派Pico <https://www.raspberrypi.com/products/raspberry-pi-pico/>`_

运行实例
-------------------------------------------

首先从git仓库克隆代码到本地：

::

    git clone https://github.com/embassy-rs/embassy.git
    cd embassy
    git submodule update --init


执行下列命令运行示例程序：

::

    cd examples/nrf
    cargo run --bin blinky --release


下一章节内容预告
-------------------------------------------

恭喜，您已经成功运行了第一个Embassy应用程序！接下来您可以：

- 了解 `执行器 <https://embassy.dev/dev/runtime.html>`_ 的更多细节
- 了解 `HAL <https://embassy.dev/dev/hal.html>`_ 的更多细节
- 开始 `编写自己的应用程序 <https://embassy.dev/dev/basic_application.html>`_


