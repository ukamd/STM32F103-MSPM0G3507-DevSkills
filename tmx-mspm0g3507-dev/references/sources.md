# 资料来源与版本边界

核对日期：2026-11-01。本技能参考 [STM32C5DevSkills](https://github.com/keysking/STM32C5DevSkills/tree/82e8bff032c95eae85592d82dec5dfac8fa50454) 的能力划分，针对 TI 重新编写，不复用 CubeMX/HAL2 命令。MIT 声明见本目录 LICENSE。厂商 SDK、手册与工具各自许可有效，不随包重分发。

## 用户提供的参考手册

文件显示名：`mspm0g3507用户手册.pdf`。实际标题是 **MSPM0 G-Series 80-MHz Microcontrollers Technical Reference Manual (Rev. A)**，文献号 **SLAU846A**，2023-10 修订，英文，1607 页。已读取封面、目录及所需 CPU/SYSCTL/DMA/GPIO 相关内容。这里保留文献号和节号，Skill 安装后不依赖用户原来的本地路径，也不要求整本加载。

| 任务 | 手册定位 |
| --- | --- |
| 配置存储及 NONMAIN 边界 | §1.4.1、§1.5 |
| SYSCTL、复位、时钟与电源 | §2.4；目录起始印刷页 135 |
| Cortex-M0+、中断体系 | §3.2、§3.3；章节起始页 214 附近 |
| DMA 通道类型、触发、完成与错误 | §4.2，尤其 §4.2.3、§4.2.9–4.2.13；页 269–277 |
| PINCM / GPIO、读 IIDX 的副作用 | §8、§9 |

它是芯片通用 TRM，不能证明天猛星板上装配、封装或具体接线。新资料/SDK 与 Rev. A 有差异时查当前 TRM、datasheet 与勘误；不要把旧版文字当作所有版本的唯一行为定义。

## 公开一手入口

- [TI MSPM0G3507 数据手册 SLASEX6](https://www.ti.com/lit/ds/symlink/mspm0g3507.pdf)：容量、封装、管脚、时钟与电气限制。
- [TI 最新 SLAU846](https://www.ti.com/lit/pdf/SLAU846)：更新版 TRM，链接可能随修订变化。
- [MSPM0 SDK](https://www.ti.com/tool/MSPM0-SDK)、[官方 SDK 源码](https://github.com/TexasInstruments/mspm0-sdk)：DriverLib、器件头文件、startup/linker、例程与版本说明。
- [TI SysConfig 指南](https://software-dl.ti.com/msp430/esd/MSPM0-SDK/latest/docs/english/tools/sysconfig_guide/doc_guide/doc_guide-srcs/sysconfig_guide.html)：按实际 SDK 对应版本查阅，`latest` 不作为项目锁定版本。
- [CCS 命令行构建说明](https://software-dl.ti.com/ccs/esd/documents/ccs_project-command-line.html)：本技能 CLI 例子依据页面标示的 CCS 20.5；旧 CCS 单独查询。
- [立创天猛星官方入口](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/)、[资料下载](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/download-center.html)：板级来源，详细链接与核实状态见 [board.md](board.md)。

未取得用户实物 PCB 版本及完整原理图，封装/晶振/排针编号仍需按具体任务核实；未宣称对该板完成实测。
