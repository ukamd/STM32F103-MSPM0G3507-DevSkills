# F103 时钟、引脚和 AFIO

以 F103C8 数据手册、RM0008、当前 `.ioc` 与电路为准。LQFP48 的可用引脚和外设实例必须逐一核对；同系列大封装示例不能直接移用。

## 时钟检查

- 最大 SYSCLK/HCLK 72 MHz；APB1 最大 36 MHz，APB2 最大 72 MHz，ADC 时钟最大 14 MHz。PLL、分频与 Flash wait state 需一起校验。
- APB 分频为 1 时对应 TIM 时钟等于 PCLK；APB 分频不为 1 时通常为 2×PCLK。UART 按所属 APB 的实际时钟计算波特率，不能用 TIM 时钟代替。
- **有真实 8 MHz HSE 的条件示例**：HSE×9 → SYSCLK 72 MHz，AHB /1、APB1 /2、APB2 /1、ADC /6 得 12 MHz。该频率工作点配置 Flash 2 wait states，并按供电条件复核。USB 若启用，校验独立要求的 48 MHz 时钟和实际电路。
- 内部 HSI 路线按其 PLL 输入约束计算，不假定 HSI 可按 HSE 的方式达到 72 MHz。HSE 晶体与外部有源时钟分别对应 oscillator/bypass 配置。
- 改时钟后核对 `HSE_VALUE`、`SystemCoreClock`、HAL tick、定时器、UART 与所有依赖时间的应用；读回 RCC 配置，必要时通过 MCO 或外设波形验证。

## AFIO 与调试引脚

F1 通过 AFIO 重映射选择外设引脚，不使用新系列常见的 GPIO Alternate 数字表。启用 AFIO 时钟并检查 MAPR。PA13/PA14 是 SWDIO/SWCLK；PA15/PB3/PB4 涉及 JTAG。需要释放 JTAG 引脚时选择保留 SWD 的模式，不一并关闭所有调试接口。不要用整寄存器常量写 MAPR 丢失其他重映射配置。

USART1 未重映射时 PA9/PA10 是 TX/RX；这只是选项，不是所有板卡的固定接线。I2C1、SPI1、TIM 通道需核对 remap、封装引出与板上占用。GPIO 输出先设安全初始电平再切输出；I2C 开漏和外部上拉按电路确定。EXTI 同一线路号只能选一个端口来源，例如 PA0/PB0 不能同时映射到 EXTI0。

不因 LED 常接 PC13 就默认该引脚；PC13 等引脚的驱动限制及电平容忍条件查数据手册。模拟引脚不应笼统视为 5 V 容忍。

输出一张本任务资源表：功能、引脚、复用/重映射、时钟、IRQ/DMA、板上占用。冲突时指出冲突资源并给出满足约束的选择，不清空整个工程重配。

依据：[F103C8 数据手册](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)、[RM0008](https://www.st.com/resource/en/reference_manual/cd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)。
