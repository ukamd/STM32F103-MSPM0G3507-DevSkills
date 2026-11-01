# 下载、启动与 Cortex-M0+ 调试

## 探针与连接

先确认实际探针（如 XDS110 或被当前 CCS 支持的 J-Link）、支持版本、序列号与 `.ccxml`。立创教程列举 XDS110/J-Link；这不是任意探针/固件组合都可用的保证。SWD 协议相同也不说明 ST-Link 在当前 TI 工具链里可用。

按[板级参考](board.md)检查 GND、DIO、CLK、电压参考和必要的 NRST。探针的 Vref 检测与供电输出要区分；USB 已供电时避免直接并联第二路电源。多个探针时在目标配置中指定实际连接，不能默认选第一个。

## CCS 下载

1. 构建当前项目与配置，保留 `.out`/map 和日志，检查下载镜像没有意外 NONMAIN section。
2. 核对 target configuration 的芯片/连接，按用户要求启动 Debug Project；该动作会连接、可能复位并下载到硬件，属于实际设备操作。
3. 确认下载日志及工具提供的 program verification 结果。只有看到了明确校验证据才能写“下载并校验成功”。
4. 需要时断点到 main，检查 `SYSCFG_DL_init()`，恢复运行；验证 LED/UART/波形并单独验证复位/断电启动。

需 CLI 自动下载时，按本机 CCS Debug Server Scripting 或 UniFlash 的**实际受支持器件和连接**配置，先查看随安装示例与帮助，再复用其明确的调用。首版不提供猜测的 MSPM0 通用烧录命令。没有支持的 CLI 时如实使用 GUI 路线。串口 BSL 是独立路线，必须确认启动条件和已支持工具，不把 CH340 枚举等同于已具备 BSL 烧录条件。

普通应用下载不执行 NONMAIN 写入、BSL 重配置、debug lock、factory reset 或全片恢复。若目标受保护，记录错误并停止自动恢复；只有用户要求恢复且了解数据丢失/配置影响时再按当前 TI 文档执行相应操作。

## 不运行或异常

- 到不了 main：核对实际 vector/reset、初始 SP、startup/链接区域、BSL invoke 条件、复位源；把“停在调试器入口”与“脱离调试器不能启动”分开。
- `SYSCFG_DL_init` 中卡住：检查外部时钟存在性、PLL 锁定、供电/Flash 配置、外设电源/复位与等待顺序；保留出错寄存器，不循环重初始化。
- HardFault：保存 PC/LR/xPSR/MSP/PSP；从 EXC_RETURN 和栈帧确定异常前 PC/LR，结合 `.out`、map、反汇编排查非法地址/指令、坏函数指针、栈溢出或不支持的对齐访问。栈损坏时不盲信回溯。
- Cortex-M0+ 不具备 Cortex-M3 完整 CFSR/HFSR/MMFAR/BFAR 诊断集合；不要读不存在的寄存器或照搬 M3 Fault 脚本。使用本器件有的系统/复位/外设状态信息与异常堆栈。
- IRQ 消失：检查分组/向量、清标志、DMA 请求及 IIDX 读取副作用；关闭会周期读取此类寄存器的 debugger watch，保存一次读取结果分析。
- 暂停后串口/ADC 行为变化可能是 FIFO 溢出、时间基准停顿或 DMA 仍运行，先核对调试冻结机制再推断运行态故障。

依据：[立创 CCS 环境与调试教程](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/ccs-beginner/install.html)、[TI TRM](https://www.ti.com/lit/pdf/SLAU846)、[TI UniFlash](https://www.ti.com/tool/UNIFLASH)。
