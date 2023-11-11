# Embassy nRF HAL

[Embassy nRF HAL](https://github.com/embassy-rs/embassy/tree/master/embassy-nrf) 基于 [nrf-rs](https://github.com/nrf-rs/) 中的 PACs（外设访问模块）。

## 定时器驱动

nRF 定时器驱动的默认工作频率为 32768 Hz。

## 外设

以下外设目前采用 HAL 实现

- PWM（脉宽调制）

- SPIM（串行外设接口）

- QSPI（四线串行外设接口）

- NVMC（非易失性内存控制器）

- GPIOTE（通用输入输出任务）

- RNG（随机数发生器）

- TIMER（定时器）

- WDT（看门狗定时器）

- TEMP（温度传感器）

- PPI（可编程外设接口）

- UARTE（通用异步接收/发送传输引擎）

- TWIM（双线串行外设接口）

- SAADC（模拟到数字转换器）

## 蓝牙

对于蓝牙，你可以使用 [nrf-softdevice](https://github.com/embassy-rs/nrf-softdevice) crate.
