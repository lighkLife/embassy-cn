# Embassy nRF HAL

The [Embassy nRF HAL](https://github.com/embassy-rs/embassy/tree/master/embassy-nrf) is based on the PACs (Peripheral Access Crate) from [nrf-rs](https://github.com/nrf-rs/).

## Timer driver

The nRF timer driver operates at 32768 Hz by default.

## Peripherals

The following peripherals have a HAL implementation at present

- PWM

- SPIM

- QSPI

- NVMC

- GPIOTE

- RNG

- TIMER

- WDT

- TEMP

- PPI

- UARTE

- TWIM

- SAADC

## Bluetooth

For bluetooth, you can use the [nrf-softdevice](https://github.com/embassy-rs/nrf-softdevice) crate.
