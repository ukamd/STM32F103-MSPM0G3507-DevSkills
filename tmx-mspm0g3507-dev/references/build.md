# CCS 命令行构建与内存检查

CCS 20.x/Theia 默认 TI Arm Clang；已有工程选择的 SDK、编译器与 SysConfig 路径应保持一致。先在独立 workspace 导入项目副本，确认 GUI/CLI 对应同一配置，保留生成与编译完整日志。

## CCS 20.5 文档形式

下列形式依据 TI CCS 20.5 文档；本机版本先核对随安装文档与 CLI 帮助。Windows/Linux 常见入口位于 CCS 的 `ccs/eclipse` 下，macOS 在其 app bundle 内；以实际安装为准。

```powershell
# $ccsCli、$workspace、$projectDir 均为实际绝对路径；workspace 不与已打开会话争用
& $ccsCli -noSplash -workspace $workspace -application com.ti.ccs.apps.importProject -ccs.location $projectDir
if ($LASTEXITCODE -ne 0) { throw 'CCS 导入失败' }
& $ccsCli -noSplash -workspace $workspace -application com.ti.ccs.apps.buildProject -ccs.projects $projectName -ccs.configuration Debug
if ($LASTEXITCODE -ne 0) { throw 'CCS 构建失败' }
```

`Debug` 仅适用于实际存在的配置；`$projectName` 是 CCS 元数据中的项目名称，不一定等于文件夹名。已导入项目不重复导入；导入同名冲突时检查 workspace 状态或选择新 workspace，不删除原项目。

上述 import 用于已具备 CCS 项目元数据的目录。对于 SDK 的 `.projectspec`，使用该版本支持的创建入口（先将例程复制到工程工作区）：

```powershell
& $ccsCli -noSplash -workspace $workspace -application com.ti.ccs.apps.createProject -ccs.projectSpec $projectSpecPath
if ($LASTEXITCODE -ne 0) { throw '从 projectspec 创建工程失败' }
```

创建后从实际元数据读取项目名，再调用 buildProject；不要对同一个项目同时 create 与 import。

如果本机只有 Eclipse CCS 12.x 等旧版，不直接运行上述 application ID；改查该版本官方 headless 文档和工程生成 makefile。可先使用 GUI 建立基线并记录 CLI 缺口，不编造兼容命令。SysConfig 报错应先修复生成阶段，不能只编译上次残留的配置文件。

## 产物与架构

- 确认工程实际编译器为 TI Arm Clang，目标 Cortex-M0+、Thumb、软件浮点 ABI；不启用硬件 FPU 或混链接其他 ABI 的库。
- startup、device 宏、链接 `.cmd` 与 MSPM0G3507 匹配，不能沿用 G3505 的容量或不同系列内存布局。器件 Flash 总量 128 KB，SRAM 32 KB；官方 G3507 empty/ticlang 例程的普通区域为 Flash `0x00000000`/`0x20000` 字节、SRAM `0x20200000`/`0x8000` 字节。实际工程还需扣除保留区并核对向量位置，不套 STM32 的 `0x08000000`。
- 检查应用主存储区域与 NONMAIN/BSL 配置区域分开；普通业务构建不主动新增 NONMAIN 初始化数据或改变调试/启动保护。
- CCS 固件输出通常为 ELF 格式 `.out`，名称从真实链接日志读取；不能因扩展名不是 `.elf` 就误判格式。map 用于检查 `.text/.const`、初始化数据、`.bss`、`.stack`、heap；具体 section 名依链接器版本。

## 常见失败

| 表现 | 定位方向 |
| --- | --- |
| 缺 `ti_msp_dl_config.h` | SysConfig 未运行、输出目录/包含路径错误、SDK metadata 不匹配 |
| 重复初始化/向量符号 | 两套生成文件、例程 ISR 与用户 ISR 重复、源码清单重复 |
| 缺 DriverLib 符号 | 头文件/库版本错配、device 宏、源/库未加入目标；部分 API 为 inline |
| RAM/Flash 超限 | map 中大对象、RTOS stack/heap、库和浮点格式化成本；不能扩大声明容量掩盖 |
| CLI 与 GUI 结果不同 | workspace 配置、编译器/产品路径、SysConfig 版本或残留产物 |

构建成功后提供具体 `.out`/map 路径与容量，检查它们确为本次生成。没有 CCS/SDK/编译器时提供缺失项和可复现命令条件，停止于静态审查状态，不声称构建成功。

依据：[TI CCS CLI 文档](https://software-dl.ti.com/ccs/esd/documents/ccs_project-command-line.html)、[TI 官方 G3507 device_linker.cmd](https://github.com/TexasInstruments/mspm0-sdk/blob/main/examples/nortos/LP_MSPM0G3507/driverlib/empty/ticlang/device_linker.cmd)（核对 blob `7d80655bd9a0e9a230de55352fb535488c2d0c73`）、本机 SDK 与编译器手册。
