# CMake / Ninja / GCC 构建与容量

下列命令适用 CMake 3.x/4.x 常见 CLI；具体最低版本取工程 `cmake_minimum_required`。使用 preset 前先读文件，检查生成器、工具链路径和主机相关变量。

```powershell
cmake --list-presets
# 只有工程确有 Debug preset 时：
cmake --preset Debug
# 只有存在同名 build preset 时：
cmake --build --preset Debug
```

没有 preset 时，可用实际存在的工具链文件配置独立 build 目录：

```powershell
# 在工程根目录；$toolchain 为实际 GCC .cmake 文件，$buildDir 为本次构建目录
cmake -S . -B $buildDir -G Ninja "-DCMAKE_TOOLCHAIN_FILE=$toolchain" -DCMAKE_BUILD_TYPE=Debug
if ($LASTEXITCODE -ne 0) { throw 'CMake 配置失败' }
cmake --build $buildDir --parallel
if ($LASTEXITCODE -ne 0) { throw '固件构建失败' }
```

已有生成器不同的 build 目录不要复用；选择新的构建目录，保留旧日志。编译器已被 CMake 缓存时，改 PATH 不会自动替换编译器。

## 必查项

| 项目 | F103C8 约束 |
| --- | --- |
| 编译/链接架构 | Cortex-M3、Thumb；无 `-mfpu`/硬浮点 ABI，保持对象 ABI 一致 |
| GCC 常见参数 | `-mcpu=cortex-m3 -mthumb -mfloat-abi=soft`，由工程工具链统一管理 |
| 宏与驱动 | 通常 `STM32F103xB`、HAL 工程 `USE_HAL_DRIVER`，源码与 HAL 配置一致 |
| 启动 | 匹配 F103 medium-density 的 GCC startup，一份向量表，入口可到 main |
| Flash | `ORIGIN=0x08000000, LENGTH=64K`；已有 bootloader 时扣除其保留区并匹配向量位置 |
| RAM | `ORIGIN=0x20000000, LENGTH=20K`；栈、堆和静态数据总预算不能超限 |

最小常规工程依赖 startup、system、CMSIS、HAL、应用、链接脚本；缺 `HAL_*` 符号先查源文件、宏和目标链接，不用空实现绕过。重复符号查重复 startup、回调和生成文件。`Reset_Handler`/向量丢失查入口及 linker `KEEP`。硬件 FPU 相关选项错误必须回到 toolchain 修正。

## 产物检查

```powershell
# $elfPath 为实际固件 ELF
arm-none-eabi-size $elfPath
arm-none-eabi-objdump -h $elfPath
arm-none-eabi-nm --print-size --size-sort $elfPath
```

确保链接器生成 map（GNU ld 常见 `-Wl,-Map=...`），分析 Flash 的 text/rodata 和 data 加载副本、RAM 的 data/bss 加上运行时 heap/stack。size 总计不是运行时内存峰值。超限先定位大缓冲区、浮点 printf、库重复和栈/堆需求，不把 C8 改成 128 KB 掩盖问题。报告 ELF/map 路径、有效内存占用、构建配置和退出码；ELF 需与本次源码一致。

依据：[CMake 手册](https://cmake.org/cmake/help/latest/manual/cmake.1.html)、[Arm GNU Toolchain 文档](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain)。
