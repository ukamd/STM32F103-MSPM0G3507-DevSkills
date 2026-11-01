---
name: stm32f103c8t6-dev
description: 开发和维护 STM32F103C8T6 固件，覆盖工具定位、CubeMX 配置、HAL/LL 编码、CMake/Ninja/GCC 构建、ST-Link 下载与 GDB 调试。适用于该芯片的新建与已有工程、外设故障和 FreeRTOS 接入；不用于 STM32C5 或 TI MSPM0。
license: MIT
---

# STM32F103C8T6 开发

默认路线是传统 CubeMX `.ioc`、STM32CubeF1 HAL、CMake、Ninja、Arm GNU Toolchain，裸机优先。接手工程保留其驱动与构建方式；SPL、Keil 工程先按原方式建立基线，不自动迁移。该芯片是 Cortex-M3、无 FPU、64 KB Flash、20 KB SRAM、LQFP48；不套用 C5 的 HAL2、CubeMX2 或 `.ioc2` 操作。

## 工作流程

1. 识别工程根目录及已有修改，查 `.ioc`、构建配置、启动文件、链接脚本和 HAL 版本。型号不符时说明适用边界。
2. 从工程、原理图和任务确定引脚、晶振、外设与调试连接；只询问当前操作必需且无法查到的信息。不把 Blue Pill 的 LED、HSE 或 USB 接法当作芯片默认值。
3. 按下表只读相关参考。配置查询得到结果即结束；修改任务只改需求及必要依赖。重新生成先暂存、比较再合并，不覆盖已有业务代码。
4. 配置修改后检查生成文件及初始化调用链，再编译；核对目标架构、链接容量和 map。下载按用户需求执行，不因编译成功自行扩展为烧录任务。
5. 汇报实际版本、修改点、验证证据及剩余条件，分别标明配置/生成、构建、下载校验、实板功能状态。没有工具或硬件时完成能做的分析，不声称验证成功。

## 按任务读取

| 当前任务 | 参考资料 |
| --- | --- |
| 找工具、依赖、版本或处理环境失败 | [环境](references/environment.md) |
| 新建、接手、例程、重新生成 | [工程](references/project.md) |
| 时钟树、引脚冲突、AFIO、SWD | [时钟与引脚](references/clocks-pins.md) |
| GPIO/EXTI、UART、I2C、SPI | [通信与 GPIO](references/gpio-comms.md) |
| PWM、ADC、DMA 和中断 | [定时器、ADC 与 DMA](references/timers-adc-dma.md) |
| HAL/LL 调用、异步缓冲区、移植 | [固件实现](references/firmware.md) |
| 编译链接、ELF/map、容量超限 | [构建](references/build.md) |
| ST-Link 下载、启动、Fault、GDB | [下载与调试](references/flash-debug.md) |
| 已有或明确要求的 FreeRTOS | [FreeRTOS](references/freertos.md) |
| 核对依据、版本和许可 | [来源](references/sources.md) |

示例中的路径、频率、通道和引脚是说明条件，不是自动配置。具体 HAL 签名、CubeMX 命令和下载器选项以工程头文件、本机帮助及对应版本文档为准。各参考均随本目录分发，不依赖其他 Skill。
