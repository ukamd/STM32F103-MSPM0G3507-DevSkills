# HAL/LL 编码与移植

## 确定驱动和生命周期

```text
rg -n "USE_HAL_DRIVER|STM32F103xB|HAL_Init|SystemClock_Config|MX_.*_Init" .
rg -n "HAL_UART_Receive_DMA|HAL_UARTEx_ReceiveToIdle_DMA|HAL_ADCEx_Calibration_Start" Drivers
```

按真实目录调整搜索范围；HAL 在外部包中时定位实际引用路径。CubeMX 的 F103C8 常用器件宏为 `STM32F103xB`，它是 CMSIS 的 medium-density 分类，不表示将 Flash 扩为 128 KB。

典型启动顺序：复位启动文件初始化 data/bss → `SystemInit` → C 运行库 → `main` → `HAL_Init` → 系统时钟 → GPIO/DMA/外设初始化 → 应用启动。以当前启动文件为准。生成器可能为 DMA 生成特定初始化顺序，不重新随意排列。

## 应用代码边界

- HAL 使用 `UART_HandleTypeDef` 等 F1 类型与 `HAL_*_Init(handle)` 接口。先看当前版本返回值、回调注册开关与签名，避免重复定义弱回调或复制 HAL2 API。
- 应用模块独立实现；初始化和回调入口只放必要连接代码。添加源文件同步 CMake 目标，删除源文件同步链接清单。
- 对 IT/DMA 明确数据、长度、所有者和完成通知；错误码区分 BUSY、TIMEOUT、ERROR。重启接收前确认旧传输状态，不无限重试掩盖硬件错误。
- ISR 只清除/转发事件并通知主循环；解析与日志在非中断上下文。没有调度器的裸机可用状态机或环形缓冲区，不能假设已存在 RTOS API。
- HAL 与 LL 可按外设分工，但同一外设状态机要有明确所有者；直接改 HAL 管理的 DMA/寄存器可能使其 handle 状态失真。

移植时分别检查芯片资源、总线时钟、GPIO/remap、启动/链接、中断向量、HAL API、时序与电气接口。来自大容量 F103 或其他 STM32 的代码先删去不可用资源引用，不能通过定义另一型号宏让编译勉强通过。

完成后构建并列明功能验证办法：输入/输出、所需仪器、预期频率/数据与错误条件；实际未测则标明。新建空工程不添加与用户任务无关的演示业务。

依据：[STM32CubeF1 官方源码](https://github.com/STMicroelectronics/STM32CubeF1)、工程当前驱动头文件。
