# TIM/PWM、ADC、DMA 与 NVIC

## 定时器与 PWM

先从 RCC 确认实际 TIM 输入时钟。边沿对齐向上计数时：`f_update = f_TIM / ((PSC+1)×(ARR+1))`。PWM 频率还取决于计数模式，不能把该式直接用于中心对齐。占空比结合 PWM1/2、极性和 CCR/ARR 解释；高低有效、0%/100% 边界需实测。

配置 GPIO 复用、AFIO remap、通道后，使用 `HAL_TIM_PWM_Start` 启动实际通道；基准定时器用 `HAL_TIM_Base_Start_IT`，不要认为初始化即已启动。TIM1 高级定时器还要检查主输出使能及 Break，互补输出有单独启动和死区要求。运行时更新 PSC/ARR/CCR 注意预装载和 update event，不在计数中途造成意外脉冲。

条件示例：TIM 时钟 72 MHz、PSC=71、ARR=999 时，边沿对齐周期为 1 ms；只有时钟链满足这些条件才可采用。

## ADC

确认 VDDA/VREF、模拟输入范围、对应 ADC 通道、模拟 GPIO、ADC 时钟（不超过 14 MHz）和源阻抗所需采样时间。校准按当前 HAL 的 `HAL_ADCEx_Calibration_Start` 生命周期执行。先以单通道轮询确认结果，再加入扫描、定时触发或 DMA。

检查规则序列、触发边沿、连续模式、DMA 模式和缓冲区大小是否一致。`HAL_ADC_Start_DMA` 的 C 参数类型并不代表每个样本必须用 32 位保存；DMA 外设/内存宽度、地址对齐及样本类型要一致。多通道缓冲区按扫描顺序解释，半满与全满通知后由主循环或任务消费，避免边采边读同一半区。

## F103C8 的 DMA 约束

无 DMAMUX，使用 RM0008 的 DMA1 请求映射表。常见候选：USART1 TX/RX 对应 DMA1 Ch4/Ch5，ADC1 对应 Ch1；仍需检查整个工程是否被其他请求占用，不只看本外设。一个 DMA 通道不能同时承担冲突请求。配置方向、增量、宽度、长度、优先级及 normal/circular。

HAL 异步调用之前完成外设与 DMA handle 关联（检查 `__HAL_LINKDMA`）、启用 IRQ 并在实际向量中调用 `HAL_DMA_IRQHandler`。UART 可能还需要 USART 全局中断。回调不触发时沿“触发源→DMA 计数/错误→标志→NVIC→向量名→HAL handler→回调”检查。`volatile` 不解决异步缓冲区所有权与并发协议。

NVIC 数字越小优先级越高；区分抢占优先级和子优先级。ISR 内不调用依赖更低优先级 tick 的阻塞 HAL API。FreeRTOS 下额外遵守[RTOS 中断约束](freertos.md)。

依据：[RM0008](https://www.st.com/resource/en/reference_manual/cd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)、当前 F1 HAL 的 TIM/ADC/DMA 源码。
