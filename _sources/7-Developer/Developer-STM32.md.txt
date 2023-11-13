# 开发者文档：STM32

原文： https://embassy.dev/book/dev/developer_stm32.html
## 了解外设元信息（meta-pac）

当一个项目编译导入`embassy-stm32` `crate` 时，该项目正在使用哪个的芯片，该项目就会使用哪个芯片对应的功能。根据这个特性，`embassy-stm32` 会选择芯片所支持的 [IP](https://anysilicon.com/ip-intellectual-property-core-semiconductors/) （半导体知识产权），并启用相应的硬件抽象层实现（HAL）。我们支持数百种芯片，但是`embassy-stm32` 是如何知道哪个芯片包含哪个 IP 呢？ 说来话长，这就算一个关于`stm32-data-sources` 的故事了。

> 注：**半导体知识产权**
> 
> 半导体中的知识产权 (IP) 核心是可重用的逻辑或功能单元或单元或布局设计，通常是按照向多个供应商授权，以用作不同芯片设计中的构建块的想法来开发的。

## stm32-data-sources

[`stm32-data-sources`](https://github.com/embassy-rs/stm32-data-sources) 是一个比较“荒凉”的仓库。它没有说明书，没有文档，关注的人很少。但它是`embassy-stm32` 成功的关键。在这个仓库你，我们支持的芯片都有一个相关的 XML 文件，比如 [`STM32F051K4Ux.xml`](https://github.com/embassy-rs/stm32-data-sources/blob/b8b85202e22a954d6c59d4a43d9795d34cff05cf/cubedb/mcu/STM32F051K4Ux.xml)。在这个文件中，你可以看到如下的内容：

```xml
    <IP InstanceName="I2C1" Name="I2C" Version="i2c2_v1_1_Cube"/>
    <!-- snip  -->
    <IP ConfigFile="TIM-STM32F0xx" InstanceName="TIM1" Name="TIM1_8F0" Version="gptimer2_v2_x_Cube"/>   ## 
```

这些内容说明这个芯片有一个 i2c（一种同步串行总线）, 它的版本是"v1_1"。同时，说明它还有一个通用的定时器（timer），版本是 "v2_x"。通过这些数据，就可以决定`embassy-stm32` 应该被包含哪些实现。但实际上，这些工作就是另一回事了。

## stm32-data

虽然该项目的所有用户都熟悉`embassy-stm32`，但很少有人熟悉为其提供支持的项目：`stm32-data`。这个项目旨在为`embassy-stm32`生成和使用必要的数据。为此，通过组合和解析`stm32-data-sources`项目下多个文件，来得到相关信息，再按所支持芯片的 IP ，来分配寄存器块实现。这些操作主要在`chips.rs`中：

```rust
    (".*:I2C:i2c2_v1_1", ("i2c", "v2", "I2C")),
    // snip
    (r".*TIM\d.*:gptimer.*", ("timer", "v1", "TIM_GP16")),
```

在这种情况下，i2c 的版本对于我们实现的 "v2"版本，通用定时器（timer）版本对应于我们的“v1”版本。因此，`i2c_v2.yaml`和`timer_v1.yaml`寄存器块实现就对应 i2c v2 和 timer v1 这些 IP。`STM32F051K4.json`就是这些内容生的结果：

```json
    {
        "name": "I2C1",
        "address": 1073763328,
        "registers": {
            "kind": "i2c",
            "version": "v2",
            "block": "I2C"
        },
        // snip
    }
    // snip
    {
        "name": "TIM1",
        "address": 1073818624,
        "registers": {
            "kind": "timer",
            "version": "v1",
            "block": "TIM_ADV"
        },
        // snip
    }
```

除了寄存器块之外，芯片引脚和复位与时钟（RCC）映射的数据也由`embassy-stm32`生成和使用。`stm32-metapac-gen`用于将数据打包并发布为 crate。

## embassy-stm32

在`embassy-stm32`的根目录`lib.rs`文件中，您将看到这一行：

```rust
#[cfg(i2c)]
pub mod i2c;
```

在`mod.rs` 中的 i2c 模块中，您将看到这一行：

```rust
#[cfg_attr(i2c_v2, path = "v2.rs")]
```

由于 STM32F051K4 支持 i2c，并且其版本对应于我们的“v2”，所以将出现`i2c`和`i2c_v2`配置指令。上述这些文件都在`embassy-stm32`中。根据芯片数据，可以生成配置指令和配置表，根据这些生成数据，`embassy-stm32` 就可以明确且清晰适配每个芯片所需要的逻辑和实现。与嵌入式生态系统中的其他项目相比，`embassy-stm32`是唯一一个项目：可以在整个 stm32 系列中重用代码，并删除了硬件抽象层中难以实现的不安全逻辑。
