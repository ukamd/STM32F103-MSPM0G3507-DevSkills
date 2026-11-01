# 环境定位与版本

适用传统 STM32CubeMX 6.x、STM32CubeF1 1.8.x、GNU Arm Embedded/Arm GNU Toolchain。版本范围表示参考路线，不保证每个组合都测试过。新建要求 CubeMX 实际提供 CMake 导出；先检查安装版本和 Project Manager 选项。无需 CubeMX2、`cube mx` 或 C5 Pack。

## 只读检查

在目标工程根目录执行：

```powershell
git status --short
rg --files -g '*.ioc' -g '*Presets.json' -g 'CMakeLists.txt' -g '*.cmake' -g '*.ld' -g '*.uvprojx' -g '.project' -g '*startup*'
Get-Command arm-none-eabi-gcc,arm-none-eabi-gdb,cmake,ninja,STM32_Programmer_CLI -ErrorAction SilentlyContinue | Select-Object Name,Source
```

非 Git 工程跳过 Git 命令。PATH 未找到不等于未安装；检查用户提供目录、STM32CubeCLT/CubeIDE 安装目录、CubeMX 自带 JRE、用户 `STM32Cube/Repository`。只在这些已知候选内搜索，不扫描整盘。完整路径调用带空格的程序使用 `& $toolPath ...`。

```powershell
arm-none-eabi-gcc --version
cmake --version
ninja --version
```

确认 GCC 是 `arm-none-eabi` 交叉编译器；Windows 本机 gcc、Keil armclang 不替代该路线。检查固件包中的 `package.xml`/Release Notes、`Drivers/CMSIS`、`Drivers/STM32F1xx_HAL_Driver`；子模块工程检查是否完整初始化。仅阅读任务不更新子模块、软件包或全局 PATH。

CubeMX 单独安装也可；已有手写工程仅编译时无需启动生成器。旧版本不支持 CMake 时报告缺口，提供支持该导出的传统 CubeMX 官方组件选择；不悄悄改用 Keil 或 CubeIDE 工程。已有项目缺某依赖时优先恢复锁定版本，避免连带升级 HAL 和生成器。

## 平台差异与完成条件

Linux/macOS 用 `command -v` 定位，检查工具包主机架构与执行权限。CubeMX 入口和 bundled JRE 布局随安装变化；Linux 的 USB 权限需按 ST 驱动说明设置，不能将权限错误当成芯片连接故障。环境调整尽量限于当前终端。

输出工具路径、版本、SDK 根目录及缺失项。`--version` 成功只证明程序可运行；完整构建还需通过[构建检查](build.md)。

官方入口：[CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)、[CubeF1](https://www.st.com/en/embedded-software/stm32cubef1.html)、[Arm GNU Toolchain](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain)。
