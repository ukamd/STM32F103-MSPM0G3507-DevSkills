# 工程与 SysConfig

## 新建或导入

从本机匹配版本 SDK 的 MSPM0G3507 **nortos / driverlib / empty** 类 TI Arm Clang 例程建立最小基线。目录名随 SDK 变化，先搜索 `.projectspec` 和器件信息，不假定固定文件路径。使用 CCS Import Project(s) 导入到项目副本，不在 SDK 安装目录直接开发。

导入 LaunchPad 例程时，明确它只提供芯片/工具链起点。按[板级参考](board.md)调整 device/package、LED、按键、串口、时钟、调试连接，删除不需要的板级配置；不因芯片同名就保留 LaunchPad 全部引脚。

已有 Keil、GCC、冻结生成文件或手写初始化工程先保留原方式建立基线。没有 `.syscfg` 不说明工程损坏，不自动反推新文件覆盖初始化。

## 配置与生成

`.syscfg` 是配置来源；查看文件头中的 SDK/product/device/package/CLI 信息。优先按当前 SDK 的 SysConfig 界面修改：查询资源 → 指定实际引脚 → 设置时钟/模式 → 解决冲突 → 保存 → 重新加载检查。脚本属性名取该版本元数据和已有文件，不从旧教程猜属性。

CCS 通常在构建时调用 SysConfig，生成 `ti_msp_dl_config.c/.h`，目录可能在 build configuration 下的 `syscfg` 或其他已配置路径。先查实际 prebuild/生成日志；不要再手工生成一份同名文件加入目标，造成重复符号或过期头文件被优先包含。

需独立生成时，先看 CCS 实际构建调用及 SysConfig 帮助。Windows 常见入口为 `sysconfig_cli.bat`，形式示意：

```powershell
# 所有变量均为已核实的实际路径；输出到暂存空目录
& $sysconfigCli --output $stageDir --product $sdkProductJson --compiler ticlang $syscfgFile
if ($LASTEXITCODE -ne 0) { throw 'SysConfig 生成失败，查看诊断' }
```

`$sdkProductJson` 指本工程 SDK 的 `.metadata/product.json`。**仅在该版本帮助支持 `ticlang` 取值及所需 device/package 参数已具备时使用**；优先复制本工程构建日志的完整调用并更换输出目录。Linux/macOS 入口按安装包调整。不使用 STM32 的 `.ioc` 或 CubeMX 命令。

## 业务代码和重生成

先在暂存工程副本运行生成，对比 `.syscfg`、生成文件、构建元数据；输出路径必须留在暂存副本中。应用、协议、状态机、IRQ 的业务处理位于稳定源目录，`main` 调用 `SYSCFG_DL_init()` 后启动需要的外设活动；先读已有调用链，避免重复初始化。

不要把长期修改写入 `Debug/syscfg` 等构建产物目录。确需冻结生成代码时显式采用 TI 指南的禁用/重新启用 SysConfig 流程，调整源清单和 include 路径，并保证仅一套初始化参与编译。不能一边保留 `.syscfg` 自动生成、一边手工维护另一份同名实现。

已对照 [TI 官方 G3507 empty/ticlang makefile](https://github.com/TexasInstruments/mspm0-sdk/blob/main/examples/nortos/LP_MSPM0G3507/driverlib/empty/ticlang/makefile)（blob `8cf7abed824111446a77ad47037ae157c1508e86`）核对 `--compiler ticlang --product` 的形式；本机 SDK 仍优先。

依据：[TI SysConfig 指南](https://software-dl.ti.com/msp430/esd/MSPM0-SDK/latest/docs/english/tools/sysconfig_guide/doc_guide/doc_guide-srcs/sysconfig_guide.html)、本机 SDK 示例。
