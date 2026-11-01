# CCS / SDK / SysConfig 环境

默认 TI Arm Clang（ticlang），不将 GCC 或 Keil armclang 路径当作同一种编译器。CCS 20.x/Theia 与旧 Eclipse CCS 的 CLI 不同。SDK、SysConfig、编译器、CCS 之间按该 SDK Release Notes 和工程锁定版本匹配，版本号不要求数值相同，也不默认全部升级。

## 发现与记录

```powershell
git status --short
rg --files -g '*.syscfg' -g '*.projectspec' -g '.project' -g '.cproject' -g '*.ccsproject' -g '*.ccxml' -g '*.cmd' -g '*startup*' -g '*ti_msp_dl_config*'
Get-Command ccs-server-cli,tiarmclang,sysconfig_cli -ErrorAction SilentlyContinue | Select-Object Name,Source
```

非 Git 目录跳过第一条。工具往往不在 PATH，先查用户提供路径、CCS 安装目录及 TI 产品搜索路径。Windows 常见安装根为 `C:/ti`，但不能硬编码为实际目录；按当前已知安装位置定向搜索。

| 组件 | 必须核对的内容 |
| --- | --- |
| MSPM0 SDK | `source/ti/driverlib`、`source/ti/devices/msp`、`examples`、`.metadata/product.json`、Release Notes |
| SysConfig | 实际版本、CLI 入口、是否使用同一 SDK 的 product metadata |
| CCS | About/安装版本、工程支持组件、MSPM0 设备支持、产品发现设置 |
| 编译器 | 实际 `tiarmclang` 路径与版本、当前工程选择的 toolchain |
| 探针 | 实际型号、驱动、固件/支持版本与 `.ccxml` 连接类型 |

通过完整路径调用带空格的程序：`& $compiler --version`；CLI 帮助参数先查随安装文档，不用执行构建来探测版本。运行 SysConfig 时必须指向本工程 SDK；另一个版本能打开 `.syscfg` 不代表可以无损升级保存。

缺组件时报告具体项及官方获取位置；已有工程优先恢复其锁定工具。环境修复不擅自更改全局 PATH、工程编译器或 SDK 版本。创建/修改构建配置前保存可比较的基线。

Linux/macOS 需使用对应主机包，脚本扩展名/CLI 路径按安装布局调整；探针支持与 USB 权限按 TI/探针厂商说明处理。版本查询成功、SysConfig 成功生成、完整构建、硬件连接是不同完成状态。

依据：[TI SDK](https://www.ti.com/tool/MSPM0-SDK)、[CCS](https://www.ti.com/tool/CCSTUDIO)、[SysConfig 指南](https://software-dl.ti.com/msp430/esd/MSPM0-SDK/latest/docs/english/tools/sysconfig_guide/doc_guide/doc_guide-srcs/sysconfig_guide.html)。
