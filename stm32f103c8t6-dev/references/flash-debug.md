# 下载、启动与 GDB 调试

## ST-Link / CubeProgrammer

工具安装不代表探针已连接。检查供电、电压参考、GND、PA13/SWDIO、PA14/SWCLK；需要复位连接时检查 NRST。Vref 与供电脚功能依探针说明区分，避免两个电源同时驱动目标。连接多个探针时必须选择序列号。

CubeProgrammer 2.x 常用 CLI 形式如下，先用本机 `--help` 确认参数；变量必须是已核实路径/序列号：

```powershell
& $programmer --help
& $programmer -l
& $programmer -c port=SWD "sn=$probeSerial"
```

连接可能复位或暂停目标，应纳入本次调试/烧录授权。读取识别信息后核对芯片族、Flash 容量、连接电压，注意 F103 系列 device ID 不能单独区分全部容量型号。按用户要求下载当前 ELF（自带地址）并校验：

```powershell
& $programmer -c port=SWD "sn=$probeSerial" -w $elfPath -v -rst
if ($LASTEXITCODE -ne 0) { throw '下载或校验失败，保留日志' }
```

原始 BIN 必须显式提供真实加载地址；应用有 bootloader 时不能一律写入 `0x08000000`。保留现有参数区，普通下载不使用全片擦除、读保护解除或 option bytes 修改。连接失败先检查接线/供电/占用，再按需求降低 SWD 频率或连接复位；不能自动跳到解锁擦除。

## GDB

使用本机已配置且支持该目标的 ST-Link GDB Server，或经验证的 OpenOCD 配置。启动前确认参数、探针序列号、端口与复位行为；不假定服务器端口固定。

```text
# 启动 arm-none-eabi-gdb <当前 ELF> 后：
target extended-remote localhost:<实际端口>
# monitor 命令依服务器帮助确认；已下载固件的只读诊断不要再 load
break main
continue
info registers
bt
```

## 启动与 Cortex-M3 Fault

- 断点到不了 main：检查 BOOT0/BOOT1、向量首地址、初始 MSP 是否在 SRAM、Reset_Handler Thumb 地址、data/bss 初始化及 `SystemInit`。复位测试区分软件复位与断电启动。
- 停在 HardFault：先保存 PC/LR/xPSR/MSP/PSP 和现场。以 EXC_RETURN 判断异常前栈，检查堆栈中的 R0–R3、R12、LR、PC、xPSR；坏 SP 时不要继续盲读。
- Cortex-M3 可读取 CFSR/HFSR/SHCSR；MMFAR/BFAR 仅在对应 valid 位有效时解释。结合 ELF/map/反汇编定位越界、非法访问、栈溢出和未对齐访问，避免仅凭 `bt` 推断原因。
- 外设无信号依次检查引脚、时钟、使能、状态标志、DMA、IRQ、回调和缓冲区，必要时查看[DMA 排查](timers-adc-dma.md)。

下载校验通过仅证明写入内容可比对；运行/断电启动、UART 数据、PWM 波形等须分别验证。

依据：[CubeProgrammer 文档](https://dev.st.com/stm32cube-docs/prog/latest/en/index.html)、[PM0056 Cortex-M3 编程手册](https://www.st.com/resource/en/programming_manual/cd00228163-stm32f10xxx20xxx21xxl1xxxx-cortexm3-programming-manual-stmicroelectronics.pdf)。
