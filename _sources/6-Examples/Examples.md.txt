# 例子

Embassy provides examples for all HALs supported. You can find them in the `examples/` folder.
Embassy 提供了所有支持的 HAL 的示例。你可以在`examples/` 目录下找到它们

主循环例子

```rust
#![feature(type_alias_impl_trait)]

use embassy_executor::Spawner;
use embassy_time::Timer;
use log::*;

#[embassy_executor::task]
async fn run() {
    loop {
        info!("tick");
        Timer::after_secs(1).await;
    }
}

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    env_logger::builder()
        .filter_level(log::LevelFilter::Debug)
        .format_timestamp_nanos()
        .init();

    spawner.spawn(run()).unwrap();
}
```
