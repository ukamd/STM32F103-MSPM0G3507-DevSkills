# 验证记录

交付版本：0.1.0。日期：2026-11-01（Asia/Shanghai）。本记录区分文件验证、人工流程审查与真实固件/硬件测试。

## 文件验证

- 两个 Skill 均通过 skill-creator 原版 `quick_validate.py`，未修改校验器；执行时使用 UTF-8 和工作目录内的 PyYAML 6.0.3。
- YAML 可解析；名称、目录、中文 UI 简介和 `$skill-name` 默认调用一致，自动发现保持开启。
- 对本地 Markdown 引用及代码围栏做完整检查，引用必须存在且留在当前 Skill 内；每个目录单独复制到临时目录后再次检查并运行原版 validator。
- ZIP 检查 CRC、文件清单、重复项及每个文件的逐字节一致性；只包含两个 Skill、INSTALL.md、VALIDATION.md。每个 Skill 保留上游 MIT LICENSE。
- 未将中间脚本、PyYAML、SDK、固件工具或用户 PDF 纳入交付压缩包。

这些检查验证分发结构与引用完整性，不证明固件或工具命令已经在硬件上运行。

## 人工场景流程审查

以下为沿入口和参考资料进行的逐场景审查，未运行独立代理，也未模拟为实板通过。每个场景均检查 STM32 与 MSPM0 路线是否分离。

| 场景 | STM32F103C8T6 的处理 | 天猛星 MSPM0G3507 的处理 | 审查结论 |
| --- | --- | --- | --- |
| 新建空工程 | `.ioc`、传统 CubeMX CMake 导出；检查 GCC startup/linker，不预设 LED/HSE | SDK nortos/ticlang 起点；核对 package 与天猛星接线，区分 projectspec 创建和现有工程导入 | 路线明确；实际构建待工具齐备 |
| 接手 Keil/SPL 或手写工程 | 先保留原构建/驱动建立基线，不强迁 HAL/CMake | 保留原工具链或冻结的初始化，不自动重建 `.syscfg` | 未要求无关迁移 |
| UART DMA 仅收首帧 | 核对固定 DMA 通道、normal 重装、USART/DMA IRQ；ReceiveToIdle 先查版本 | 核对触发粒度、通道能力、完成/错误与重装，不套用 HAL 回调 | 未混用 API；运行时待测 |
| PWM 与 ADC | 检查 APB/TIM 时钟、计数模式、ADC 14 MHz 上限及采样/DMA宽度 | 检查实际 TIMA/TIMG 模式、CCP/pinmux、事件路由、ADC 参考和 DMA | 未套用另一架构的时钟/计数模型 |
| Flash/SRAM 超限 | C8 64 KB/20 KB，map 定位，不扩大容量 | G3507 128 KB/32 KB，使用 TI 链接布局，不写入 STM32 地址 | 容量与区域边界明确 |
| 重新生成可能覆盖应用 | `.ioc` 暂存副本、核对输出路径、差异合并、USER CODE及独立模块 | `.syscfg` 暂存副本、只保留一套生成文件，应用在构建目录外 | 生成与业务边界明确 |
| 工具或版本缺失 | PATH 外定向查安装，CMake 不支持则报告缺组件，不更换路线 | 区分 CCS 版本/CLI，匹配 SDK/SysConfig，不猜测旧版命令 | 有停止条件与可行动缺口 |
| 无探针或无硬件 | 软件任务可继续，下载/功能状态保持未验证 | 不把串口枚举当下载，校验需明确日志，不自动改 NONMAIN | 完成状态没有夸大 |

另行核对了天猛星 LED/按键/CH340/SWD 的官方教程、TI 数据手册中的 UART0 引脚方向、官方 SDK 的 TI Arm Clang/SysConfig 参数及内存布局。板卡完整原理图与用户 PCB 版本未取得；封装、晶振与排针编号保留待核实标记。

## 构建及硬件边界

本机只读检查发现传统 CubeMX 安装和 STM32CubeF1 1.8.7，已在本地 HAL 头文件核对 `HAL_UARTEx_ReceiveToIdle_DMA`、`HAL_UARTEx_RxEventCallback` 的存在与签名。CubeMX 安装数据库含 CMake 工具链项，但本次没有生成固件工程。

在 PATH 与定向检查的常见工具位置中未定位本路线所需 Arm GNU GCC/CMake/Ninja，也未定位完整 TI CCS/MSPM0 SDK/TI Arm Clang 工具链。已有 Keil 安装不替代已选的两条默认路线。未进行固件交叉编译、SDK 安装、软件升级、探针连接、下载或实板测试；不能据此断言机器其他位置绝无这些软件。

用户提供的 PDF 已核对为 SLAU846A Rev. A，并提取所需 SYSCTL、CPU、中断、DMA、GPIO 章节核查；其版本边界及公开替代入口已写入 TI Skill 的来源页。

## 后续实板验收

工具齐备后，各平台先构建最小裸机工程，再验证实际 LED/按键/UART；随后按任务加入 PWM、ADC 与 DMA，记录波形/数据及错误恢复结果。下载前核对器件、探针和镜像；分别保存构建日志、下载/校验日志、复位和断电启动结果。FreeRTOS 仅在需要时增加调度、IRQ约束与栈水位测试。
