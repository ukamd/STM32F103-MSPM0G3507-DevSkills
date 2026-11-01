# 天猛星板级参考

核对日期：2026-11-01。下表来自立创天猛星官方 CCS 教程及 TI 芯片资料，**未对用户实物及完整板卡原理图逐网核验**。教程示例宏名偶有不一致，使用生成头文件中的实际名称。教程是板级线索；实物版本、改板或外接扩展板以其原理图和连接为准。

| 资源 | 可核实的参考接法/状态 | 依据与限制 |
| --- | --- | --- |
| 用户 LED | PB22，高电平亮 | [立创 LED 教程](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/ccs-beginner/led.html) 的“LED灯原理图”描述；不是 LaunchPad LED 引脚 |
| 用户按键 B21 | PB21，接地按下，输入上拉 | [立创按键教程](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/ccs-beginner/key.html)；软件消抖，ISR 不阻塞等待 |
| BSL 按键 | PA18，为默认 BSL invoke 相关连接 | 同上；教程描述上电拉高进入 BSL。具体有效条件受 NONMAIN/启动配置影响，不自动重配该脚 |
| 复位按键 | RST，为复位功能 | 同上；不当普通 GPIO 按键 |
| USB 串口 | CH340E，连 UART0 的 PA10/PA11 | [立创 UART 教程](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/ccs-beginner/uart.html)；接口为板载 USB 转串口，不等于 MCU 自带 USB |
| UART0 方向 | PA10 为 MCU TX，PA11 为 MCU RX | 对照 [TI 数据手册 Pin Attributes](https://www.ti.com/lit/ds/symlink/mspm0g3507.pdf) 复用选项；CH340 的 RX/TX 对向连接 |
| SWD | 板丝印 DIO/CLK，分别接 SWDIO/SWCLK；GND 共地，3V3 接法依探针供电/参考要求 | [立创环境教程](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/ccs-beginner/install.html)；TI 数据手册给出 PA19/SWDIO、PA20/SWCLK |
| MCU 封装/完整订货号 | 尚未从完整原理图核实 | 配置前从实物丝印、官方原理图及当前 SysConfig package 确认；不根据“G3507”省略 package 选择 |
| 外部晶振与元件装配 | 尚未核实，不能假定 HFXT/LFXT 已装 | 教程代码中的 delay 常数也不能证明当前主频；查 SYSCTL 配置和原理图 |
| PCB 版本、扩展排针物理编号 | 尚未核实 | 不提供猜测的排针序号。需要时从对应版本原理图/引脚图建立映射 |

## 实际使用顺序

识别板卡与 PCB 版本 → 获取对应原理图 → 对照资源表及实际外接线 → 查看 `.syscfg` 的 device/package、pinmux、clock → 生成后回查 `ti_msp_dl_config.h/.c`。若来源冲突，记录冲突并以能确认的实物电路为准；不要静默选一个教程值。

UART0 PA10/PA11 已接板载 USB 串口，改作其他外设前核实其电气占用。保留 SWD 引脚和 BSL 启动相关条件；电机等外接负载必须有合适的外部驱动，不能因引脚可输出 PWM 就直接连接负载。

本包不捆绑厂商原理图或用户手册。后续资料入口：[官方页面](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/)、[下载中心](https://wiki.lckfb.com/zh-hans/tmx-mspm0g3507/download-center.html)。
