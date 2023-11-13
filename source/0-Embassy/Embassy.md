# Embassy

原文：[https://embassy.dev/book/dev/index.html](https://embassy.dev/book/dev/index.html)

在嵌入式开发中，Embassy项目的目标是让`async/await`成为首选的实现方式。

## 什么是异步？

在执行I/O操作时，程序会在调用I/O相关操作的功能处阻塞，直到I/O操作完成。在比如Linux这样的操作系统环境中，这样的功能一般会交给操作系统内核来控制，使得其他任务，通常是另一个线程，能够得到执行，或者是CPU进入睡眠模式直到有任务进入可执行状态。操作系统不能假设线程会主动协作，线程相对来说需要较多的资源，在分配的时间内，如果线程没有将控制权交还给内核，就可能会被强制中断。但如果假设任务会主动协作，或者至少不是恶意抢占资源，那么与传统操作系统线程相比，创建线程的资源消耗就可能达到几乎可以忽略不计的程度。这种被某些编程语言称之为协程或*goroutines*(Go语言)的轻量任务，在Rust中使用异步来实现。

在Rust中，非阻塞操作可以通过async/await来实现。Async-wait将async函数传入一个叫做future的对象，当future对象被I/O操作阻塞时，调度器(通常称为执行器，executor)将选择其他future对象来执行。执行器不需要检测future对象什么时候能执行，所以相对RTOS等方案而言，async有更好的切换性能和更低的消耗，但程序大小通常会比非异步程序更大。这在资源有限，比如内存很少的设备环境中可能是个问题。在Embassy支持的设备，比如stm32和nrf中，内存通常足以容纳规模适度的程序运行。

## Embassy是什么？

Embassy项目包含以下crate，它们可以自由组合使用，没有任何限制。

- **执行器** [embassy-executor](https://docs.embassy.dev/embassy-executor/)是一个异步执行器。它在启动时分配固定的任务数，同时允许后续添加。对于支持硬件抽象层（HAL）的硬件，比如USART, UART, I2C, SPI, CAN, 和USB等，Embassy同时提供了支持同步和异步的版本。像直接内存访问（DMA）这样的场景适合使用异步接口，GPIO则更适合使用同步接口。执行器还提供了定时器功能，可以方便地实现同步或异步的延迟。对于小于一微秒的延迟要求，建议使用阻塞延迟，因为上下文切换的成本太高，执行器将无法提供准确的定时。

- **硬件抽象层（HAL）** HAL提供了安全易用的接口来操作硬件，不需要操作原始寄存器。Embassy维护了下列硬件的HAL，但它并不局限于此，您可以在任何使用Embassy的项目中使用HAL。

  - [embassy-stm32](https://docs.embassy.dev/embassy-stm32/) 支持STM32微控制器系列

  - [emmbassy-nrf](https://docs.embassy.dev/embassy-nrf/) 支持北欧半导体公司(the Nordic Semiconducotr)的nRF52、nRF53、nRF91系列
  
  - [embassy-rp](https://docs.embassy.dev/embassy-rp/), 支持树莓派RP2040。
  
  - [esp-rs](https://github.com/esp-rs), Espressif Systems ESP32系列芯片

> **注意**  
> 有很多人关心是否可以单独使用Embassy硬件抽象层（Embassy HALs）？答案当然是可以! Embassy硬件抽象层并不依赖执行器，甚至可以在没有异步的情况下使用它们，它们实现了[Embedded HAL](https://github.com/rust-embedded/embedded-hal)阻塞和异步特性。|

- **网络** [embassy-net](https://docs.embassy.dev/embassy-net/)实现了常用的网络功能，包括以太网、IP、TCP、UDP、ICMP和DHCP。在管理超时和同时为多个连接提供服务方面，Async做了高度简化，提供了很大的便利。

- **蓝牙** [nrf-softdevice](https://github.com/embassy-rs/nrf-softdevice)为nRF52微控制器提供蓝牙低能耗4.x和5.x支持。

- **LoRa** [embassy-lora](https://docs.embassy.dev/embassy-lora/)支持STM32WL无线微控制器和Semtech SX127x收发器上的LoRa网络。

- **USB** [embassy-usb](https://docs.embassy.dev/embassy-usb/)实现了设备侧的USB堆栈。实现了通用类（如USB串行（CDC ACM），支持USB HID，并且提供丰富的构建器API方便构建您自己的实现。

- **Bootloader和DFU** [embassy-boot](https://github.com/embassy-rs/embassy/tree/master/embassy-boot)是一个轻量级的启动器，它支持固件加载，支持以电源故障安全的方式升级固件，支持试用引导和回滚。

## 相关资源

关于异步Rust和Embassy的更多资料:

- [对比FreeRTOS和Embassy](https://tweedegolf.nl/en/blog/65/async-rust-vs-rtos-showdown)

- [Embassy应用实例](https://dev.to/apollolabsbin/series/20707)

- [通过Embassy升级固件](https://blog.drogue.io/firmware-updates-part-1/)
