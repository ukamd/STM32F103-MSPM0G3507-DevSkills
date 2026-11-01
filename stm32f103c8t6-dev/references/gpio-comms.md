# GPIO、EXTI 与通信

接口名用于 STM32CubeF1 HAL1 路线；先从当前 SDK 的 `stm32f1xx_hal_*.h` 核对签名。初始化先检查生成器已经完成的部分，避免重复重置外设。

## GPIO / EXTI

`HAL_GPIO_WritePin`、`HAL_GPIO_ReadPin`、`HAL_GPIO_TogglePin` 以实际 port/pin 工作。按键依据有效电平设置上下拉，轮询或计时消抖；不在 EXTI ISR 中阻塞延时。EXTI 需同时满足 GPIO/AFIO 来源、边沿、屏蔽及 NVIC 设置；实际向量处理函数调用 `HAL_GPIO_EXTI_IRQHandler(pin)`，应用回调为 `HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)`。共享 EXTI 向量中处理本项目启用的各条线。

## UART：先基本链路，再 IT/DMA

确认 TX/RX 交叉、共地、电平、引脚重映射、波特率/格式、所用 PCLK。使用 `UART_HandleTypeDef` 与 `HAL_UART_Init`，不使用 HAL2 的 handle/SetConfig 模式。轮询传输有真实超时；中断/DMA 接收必须先启动，回调后按模式重装。

- `HAL_UART_Receive_IT` 的缓冲区在完成前保持有效，不能指向已经返回的栈对象。
- 固定长度 DMA：查 `HAL_UART_Receive_DMA`、`HAL_UART_RxCpltCallback`；正常模式接收完成后需要重启，循环模式需管理读写位置。
- 变长帧：**头文件确有接口时**才使用 `HAL_UARTEx_ReceiveToIdle_DMA` 与 `HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *, uint16_t)`。HT/TC/IDLE 事件可能分别调用回调；`Size` 按当前驱动语义处理，循环接收按位置差计算新数据，不能每次将其当成全新帧长度。
- 旧 HAL 缺该 API 时，沿用官方例程的 IDLE+DMA 方案或与用户讨论版本更新；不自行发明替代函数。
- STM32F103 无 DMAMUX；DMA 通道由器件映射决定。USART 的 IRQ 与 DMA IRQ 都按需要接入 HAL，检查 ORE/FE/NE、错误回调及恢复策略。

## I2C / SPI

I2C：核对 SCL/SDA remap、开漏上拉、目标电压与地址约定。F1 HAL 常规 7 位地址参数使用地址左移一位，按实际接口确认；总线 ACK、BUSY 卡住与软件超时分开诊断。恢复前确认总线归属，不直接对共享总线盲发时钟。

SPI：核对 CPOL/CPHA、帧宽、位序、分频及 CS 极性；软件 NSS 模式由应用管理 CS，传输完成后才释放。16 位帧的 `Size` 不能想当然按字节解释。IT/DMA 的 TX 缓冲区直到驱动完成前不复用，ISR 不做协议解析或阻塞打印。

依次验证静态接线、时钟、简单轮询、异步传输。逻辑分析仪/示波器只在实际使用后报告波形结论。

API 依据：[ST 官方 F1 HAL 驱动](https://github.com/STMicroelectronics/stm32f1xx-hal-driver)，定位 `Inc/stm32f1xx_hal_uart.h`、`Src/stm32f1xx_hal_uart.c` 及所需外设文件；在线 main 不替代工程锁定版本。
