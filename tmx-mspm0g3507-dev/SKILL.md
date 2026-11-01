---
name: tmx-mspm0g3507-dev
description: 开发和维护立创天猛星 MSPM0G3507 固件，覆盖板级适配、CCS/TI Arm Clang 环境、SysConfig/DriverLib 配置与编码、构建、下载和 Cortex-M0+ 调试。适用于该板的新建与已有工程、外设与 DMA 故障及按需 FreeRTOS 接入；不用于 STM32 或直接套用 LaunchPad 接线。
license: MIT
---

# 天猛星 MSPM0G3507 开发

默认 CCS、TI Arm Clang、MSPM0 SDK、SysConfig、DriverLib，裸机优先。目标为立创天猛星，不是 TI LaunchPad 或立创地猛星。MSPM0G3507 为 Cortex-M0+、无 FPU、128 KB Flash、32 KB SRAM，最高频率 80 MHz；实际频率、封装和接线由板卡与工程确定。接手 Keil 等工程时保持已有工具链，不自动迁移。

## 工作流程

1. 读取已有修改及 `.syscfg`、CCS 工程元数据、`.projectspec`、`.ccxml`、启动与链接文件；确认实际 SDK、SysConfig、编译器版本及板卡身份。
2. 涉及硬件先读[板级参考](references/board.md)，从原理图、现有工程和用户接线核实资源；只对影响本次操作且查不到的条件提问。板卡版本未确认时不将参考接线当作实物事实。
3. 按下表读取所需资料，查询不扩展为修改。以 `.syscfg` 为配置来源，业务逻辑位于生成目录外。重新生成先在暂存副本比较，不能通过改生成文件长期维持配置。
4. 按本机 CCS 版本建立构建基线，再完成需求并检查目标架构、链接区域、map。下载与调试仅按用户本次需求执行；不因构建成功自动启动探针操作。
5. 输出工具与 SDK 版本、板级条件、改动、构建和验证证据。分别报告生成、构建、下载/校验、实板功能；缺工具或硬件时标明限制，不声称功能已验证。

## 按任务读取

| 当前任务 | 参考资料 |
| --- | --- |
| 板载 LED/按键、串口、SWD、版本不明 | [天猛星板级参考](references/board.md) |
| CCS、编译器、SDK、SysConfig 定位 | [环境](references/environment.md) |
| 建项、导入、例程迁移、生成边界 | [工程与 SysConfig](references/project.md) |
| 时钟、电源、IOMUX/PINCM | [时钟与引脚](references/clocks-pins.md) |
| GPIO、UART、I2C、SPI、DriverLib 编码 | [通信与固件](references/gpio-comms.md) |
| Timer/PWM、ADC、DMA、共享中断 | [定时器、ADC 与 DMA](references/timers-adc-dma.md) |
| CCS 命令行构建、map、内存错误 | [构建](references/build.md) |
| 探针下载、启动失败、HardFault | [下载与调试](references/flash-debug.md) |
| 已有或明确要求的 RTOS | [FreeRTOS](references/freertos.md) |
| 用户提供手册、官方资料与版本 | [来源](references/sources.md) |

DriverLib、SysConfig 字段、设备 ID 与命令以本机 SDK 的头文件/元数据及对应版本帮助为准。示例宏名须从生成头文件取得，不能猜测。普通下载不修改 NONMAIN、BSL/保护配置或执行解锁、恢复出厂及全片擦除；确需这些操作时单独说明实际影响并取得对应授权。本目录自包含，不依赖其他 Skill。
