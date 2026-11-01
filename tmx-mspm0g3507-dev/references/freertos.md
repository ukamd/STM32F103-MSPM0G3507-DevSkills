# FreeRTOS 按需路线

仅在工程已经使用或用户要求时接入；默认保留裸机。先查**同版本 MSPM0 SDK** 提供的 G3507 FreeRTOS 示例及其 supported compiler/port。示例若只有 GCC 等其他工具链，不能将其中汇编 port 直接放入 TI Arm Clang 工程，也不能静默迁移整个工程。优先使用已明确支持的组合，缺支持时报告具体条件。

## 集成检查

- port 与 Cortex-M0+、编译器、内核版本相匹配；不要复制 STM32 Cortex-M3 的 PendSV/SVC、BASEPRI 临界区实现或编译宏。
- 中断向量唯一归属，明确 SysTick、PendSV、SVC 与 startup 的连接。MSPM0 共享中断内只使用该 port 明确支持的 ISR API。
- Cortex-M0+ 没有 M3 的 BASEPRI，常见 port 以 PRIMASK 保护临界区；不能机械添加 M3 的 `configMAX_SYSCALL_INTERRUPT_PRIORITY` 规则。实际 API 与约束依据所用 port，不自造优先级策略。
- tick 时钟与当前主频、低功耗策略匹配，阻塞 UART/软件延时不能妨碍调度；`delay_cycles` 不替代任务延时。
- 32 KB SRAM 内同时预算 data/bss、DMA buffer、各任务 stack、内核 heap 和中断栈；内核原生 stack depth 与 CMSIS 包装的字节单位按接口区分。开启适用的 malloc/栈失败检测。
- 外设传输由任务/模块单一管理，ISR 用通知/队列等移交缓冲区；不在 ISR 阻塞，不在 DMA 完成前释放或修改缓冲区。

验收：构建 map 和向量正确；实板验证 tick、任务切换、最大负载下 DMA/IRQ、栈水位及复位启动。未接硬件时只报告静态/构建结果。

依据：[TI MSPM0 SDK](https://www.ti.com/tool/MSPM0-SDK) 对应版本示例及 Release Notes、[FreeRTOS 官方内核源码](https://github.com/FreeRTOS/FreeRTOS-Kernel) 中实际采用的 portable 目录。
