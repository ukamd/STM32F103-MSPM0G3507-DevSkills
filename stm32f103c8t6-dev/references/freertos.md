# FreeRTOS 按需接入

只在用户要求或已有项目使用时读取。裸机工程不自动添加调度器。优先使用工程已锁定的 FreeRTOS/CMSIS-RTOS 版本，区分 CMSIS-RTOS v1/v2 和 native API，不混用不同对象类型。

1. 选择与编译器和 Cortex-M3 匹配的 port、一个 heap 实现；确认 SVC/PendSV/SysTick 向量唯一归属。CMSIS 适配层和手写 handler 不可重复定义。
2. 检查 CPU/tick 频率和 `configPRIO_BITS` 是否对应芯片。NVIC 数字越小优先级越高；区分 FreeRTOS 的已移位优先级宏与库级未移位值。调用 `FromISR` 的 IRQ 必须处于允许调用内核的优先级范围。
3. 检查 HAL tick 与 RTOS tick 的所有权；CubeMX 可将 HAL 时间基准配置到空闲定时器，具体选哪个由资源表决定。不得同时让两个实现覆盖 SysTick，不能让 HAL timeout 停止推进。
4. 20 KB SRAM 下先做任务栈、heap、data/bss、DMA 缓冲预算。原生 FreeRTOS 任务栈参数通常以 StackType_t 元素数计，CMSIS v2 `stack_size` 以字节计；按实际层确认。开启适用的栈溢出与 malloc 失败钩子。
5. ISR 使用明确支持的 FromISR 通知/队列并按 port 要求请求切换；不阻塞。任务间用队列、通知或互斥管理外设，DMA 缓冲区所有权在完成后转移。

构建后检查向量和 RAM；实机再验证调度、HAL 时间基准、最大负载下 UART/DMA 与栈水位。没有硬件时这些项目保持待验证状态。

依据：[FreeRTOS Cortex-M3/M4 中断优先级说明](https://www.freertos.org/Documentation/02-Kernel/03-Supported-devices/04-Demos/ARM-Cortex/RTOS-Cortex-M3-M4)、工程所用内核与 CMSIS 适配层源码。
