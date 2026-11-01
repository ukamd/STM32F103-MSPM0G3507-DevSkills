# 时钟、电源与 IOMUX

MSPM0G3507 的实际资源以数据手册 SLASEX6、对应 TRM、封装和当前 SDK 为准。用户提供 SLAU846A Rev. A 的相关章节为 2.4（SYSCTL）、3（CPU subsystem）、8（IOMUX）、9（GPIO）。参考手册较旧时，当前器件勘误和项目支持版本优先，不能据此删除已存在的新版本配置。

## 时钟

区分 SYSOSC、SYSPLL、MCLK、ULPCLK、LFCLK 及外设实际时钟源。最大主频 80 MHz 不等于默认 80 MHz，也不说明外部晶振存在。读 `.syscfg` 和生成的 SYSCTL 初始化、时钟宏，逐段记录源、倍分频和外设选择。按当前器件工作点检查 Flash wait state、供电/电源策略和 PLL 锁定顺序。

改变 MCLK 后核对 Timer、UART、ADC 和延时计算。生成宏 `CPUCLK_FREQ` 仅在本工程确有定义且语义匹配时使用；`delay_cycles(32000000)` 不能固定解释为一秒。测试时优先用可观测的 Timer/串口/时钟输出交叉验证，不凭单个配置宏宣称实际晶振正常。

低功耗会改变时钟可用性、外设运行与唤醒路径；运行模式正常、睡眠后停止的外设要核对时钟源及电源域。不要套用 STM32 APB 定时器自动倍频公式。

## PINCM / IOMUX

GPIO 端口位、封装管脚号、PINCM 索引和排针位置是不同编号。使用 SysConfig 的 pinmux 约束和 SDK 的 IOMUX 常量，不用 GPIO 编号计算 PINCM。读生成代码确认输入使能、上下拉、模拟/数字模式、输出连接和复用功能。某脚有 5 V 容忍能力不代表所有模拟/数字引脚都可以接 5 V。

保护板载 UART0 PA10/PA11 占用、SWD 和 BSL；确需重分配时先确认电路与本次需求。GPIO 输出先设安全电平，I2C 用适合的开漏与外部上拉；避免为解决一个冲突重置所有 pinmux。

## 外设复位与上电

很多 MSPM0 外设需要模块复位、电源使能及规定的启动等待，之后才能配置寄存器。从生成的 `SYSCFG_DL_init` 及 `SYSCFG_DL_*_init` 追踪这条链；不是 STM32 的 `__HAL_RCC_*` 模式。运行时重新复位会清除先前配置和传输状态，不能把 SysConfig 初始化当作随时安全的重试函数。

输出本次使用资源的端口/位、pinmux、时钟源/频率、电源、IRQ/DMA 及板载冲突；生成后确认实际代码与配置一致。

依据：[TI 数据手册](https://www.ti.com/lit/ds/symlink/mspm0g3507.pdf)、[最新 TRM](https://www.ti.com/lit/pdf/SLAU846)、[SDK DriverLib](https://github.com/TexasInstruments/mspm0-sdk)。
