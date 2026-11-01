# 新建、接手与生成边界

## 新建默认路线

1. 传统 CubeMX 选择完整器件 `STM32F103C8T6`；先确认板卡供电、时钟源及任务引脚。没有 HSE 依据时保留可用内部时钟起点，不为追求 72 MHz 假设有 8 MHz 晶振。
2. 按需保留 Serial Wire、设置时钟及外设，检查冲突与时钟校验。空工程不自动增加 LED、UART 或 RTOS。
3. Project Manager 选择 CMake（需本机支持），设置工程路径与 STM32CubeF1 版本；生成到新目录。保存 `.ioc`、生成器版本、固件包版本及依赖获取方式。
4. 检查生成的 CMake、GCC 工具链文件、F103xB 启动文件和 C8 链接容量，再按[构建](build.md)完成基线。保留生成的 toolchain/preset，不用通用模板覆盖。

## 接手已有工程

先识别 `.ioc`、`CMakePresets.json`、Makefile、`.uvprojx`、CubeIDE 元数据，以及 HAL/LL/SPL/寄存器实现。先按现有方式构建；记录已有失败，随后修改本次需求。没有 `.ioc` 时维护现有初始化，不重建一个猜测的配置。移植官方例程先查芯片、封装、时钟、LED/串口、内存容量差异，不把 ST 评估板设置直接当作通用板设置。

## 重新生成

把当前工作树和未提交修改保留在原位置；将 `.ioc` 副本及需要的附属配置放到暂存工程。确保副本的输出路径确实指向暂存目录，再生成，检查日志与文件差异。只合并相关配置、初始化、IRQ、构建清单改动。生成前后比较 `USER CODE BEGIN/END` 区块；用户代码优先位于独立应用文件。更改 pin/clock/DMA 后同步回调、handle、IRQ 及应用引用，防止仅 `.ioc` 更新。

传统 CubeMX 的 `-q` 可从脚本读取命令。下例为 UM1718 的脚本路线，仅用于**已配置好的暂存副本**；先确认版本支持及退出行为，生成日志和实际文件都要检查：

```text
config load <暂存工程的绝对路径.ioc>
project generate
exit
```

不要把尖括号占位内容直接执行。Windows 在 CubeMX 安装目录使用自带 Java：

```powershell
# $mxRoot 与 $scriptFile 均为已核实的绝对路径；脚本编码按本机支持确定
Push-Location $mxRoot
try { & './jre/bin/java.exe' -jar './STM32CubeMX.exe' -q $scriptFile }
finally { Pop-Location }
```

Linux/macOS 按安装说明运行 CubeMX 入口及 `-q`。不拼造 `cube mx` 子命令，也不假定所有 CubeMX 版本共享参数。脚本入口不适配时用 GUI 完成同样配置并记录步骤。

依据：[UM1718](https://www.st.com/resource/en/user_manual/um1718-stm32cubemx-for-stm32-configuration-and-initialization-c-code-generation-stmicroelectronics.pdf)。
