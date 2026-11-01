# GPIO、通信与 DriverLib 固件

先定位本机 `source/ti/driverlib` 的 `dl_gpio.h`、UART/I2C/SPI 头文件及当前生成的 `ti_msp_dl_config.h`。DriverLib 的 `DL_*` 接口不是 STM32 HAL handle 模型；不同 UART 类型有不同包装，按本器件头文件选择，不靠前缀规律发明函数。

```text
rg -n "SYSCFG_DL_init|_INST|_IRQn|_PORT|_PIN" <工程实际生成目录>
rg -n "DL_UART_Main_|DL_I2C_|DL_SPI_|DL_GPIO_" <实际SDK的source/ti/driverlib目录>
```

替换尖括号路径后执行，不把文件搜索当成配置修改。

## GPIO 与按键

`DL_GPIO_setPins`/`clearPins`/`togglePins` 使用真实端口和掩码；宏来自本项目生成头文件。天猛星 LED/按键参考见[板级表](board.md)，不要复制教程里互相不一致的 `PIN_14`/`PIN_22` 宏名。按键轮询返回位掩码需与目标 bit 比较，计时消抖；GPIO ISR 检查实际共享向量和 pending，不阻塞等待按键松开。

## UART

先核对 UART 实例、PA10/PA11 或用户实际引脚、方向、时钟、波特率与格式。板载 CH340 的串口可枚举不证明固件已运行。`SYSCFG_DL_init()` 生成配置后，按例程启用所需 IRQ、DMA 或接收流程；先用短帧轮询验证，再做异步链路。

读当前 SDK 的 `DL_UART_Main_transmitData*`、`DL_UART_Main_receiveData*` 或对应 UART 变体，确认阻塞/FIFO/超时语义。非阻塞写入前检查容量，接收时处理 FIFO 和 overrun/error，不能一次中断只读一个字节就假定 FIFO 已空。ISR 不做阻塞串口日志。

UART DMA 的触发源、FIFO 阈值、数据寄存器地址、传输模式和完成通知从同版本官方示例获得。明确一次触发搬一个元素还是一个块，禁止把未到达的 UART 数据按整块强行读取。协议帧边界由长度/分隔符/计时等具体设计决定，不假设有 STM32 ReceiveToIdle 回调。缓冲区在 DMA 完成前保持有效，normal 模式重装和重复模式消费策略分开实现。

## I2C / SPI

I2C 核对 controller/target 角色、7/10 位地址、上拉、电压、时钟和 FIFO。TI 接口的地址含义按当前函数确认，不能沿用 STM32 的“地址左移一位”惯例。处理 NACK、仲裁丢失、BUSY 和完成事件；有限超时后保存状态，不在 ISR 无限等待。

SPI 核对 controller/peripheral 角色、PICO/POCI 方向、CPOL/CPHA、帧宽、时钟与 CS。TX FIFO 接收了数据不等于最后一位已移出；根据实际 busy/完成语义释放 CS。帧宽、DMA 宽度、传输计数与缓冲区元素保持一致。

## 模块所有权

应用与驱动放在非生成目录；向量处理函数使用 startup 中真实符号，避免与官方例程重复定义。一个外设的初始化、传输状态和缓冲区由一个明确模块负责。所有错误恢复有停止条件，避免无限重试和重复复位掩盖问题。

依据：[TI 官方 SDK DriverLib 与例程](https://github.com/TexasInstruments/mspm0-sdk)、项目对应版本 DriverLib API Guide。
